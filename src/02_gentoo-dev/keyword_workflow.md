# Gentoo Testing and Keywording Workflow
## 如何寻找需要keyword(testing)的包
gentoo的包对各个架构的支持保存在KEYWORD这个变量里面，可以在ebuild中看到，也可以通过`equery -q keywords xxx_package` 查询。对于一个比较新的架构如riscv，很多包都是没有keyword的，可以先从`/var/db/repos/gentoo/profiles/arch/riscv/`下的`package.use.mask`和`use.mask`开始，里面主要包括了 
1. 别的开发者暂时没有测试的依赖包给屏蔽掉的，可能是riscv已经支持的 
2. 开发者遇到过的，riscv明确还没有支持的软件 
3. 其他一些别的原因引入的。这两个文件也是之后keyword工作中有时要修改的。

当然还有更多的这两个文件中没有提到的其他需要测试的包，开发者可能没遇到过。所以也可以根据**自己的兴趣**寻找需要keyword的包。

还有一个途径是借助`qkeyword -n riscv`会列出所有没有riscv keyword的包。利用一些简单的bash和eix，可以列出一个category里没有riscv keyword的包。
```
for pkg in $(eix -C -c net-analyzer|head -n -3|cut -d ' ' -f2); do qkeyword -p $pkg -n riscv; done
```
当然equery，eix，qkeyword都是直接查询你的机器上的repo，所以假设你的repo是比较新的。

## keyword某个包的要求
当然需要在gentoo riscv环境中进行测试，最基本的钢性要求是__通过emerge编译和安装成功__。然后最好portage的测试阶段\(通过`FEATURES="test" USE="test"`打开，有可能引入更多的包\)也可以通过，但不太关键的小包或许可以懒一点跳过。然后就是功能测试，自己试着使用一下这个软件，看会不会遇到运行时错误，或者不正常的行为，比如启动gnome，然后发现gnome压根显示不了桌面。

如何定义__通过emerge编译和安装成功__呢？gentoo的软件通过一系列USE flag来控制各种功能和特性。理想的是每种USE flags的组合都测试一遍，但很麻烦。建议先作为试探把默认的或者常用比较重要的USE的flag的组合打开来测试是否能够安装成功，然后再把所有use flag都打开测试一遍。对于比较耗时的软件，我喜欢直接一波把所有use flags都打开测试。

某些情况下某个USE flag需要被mask掉才能keyword一个包，不过遇到的不多。
1. 启用这个USE会引入很多其他没有keyword的依赖，开发者比较懒暂时不测
2. 启用这个USE会引入某个没有keyword的关键包，要么一并测试了，要么暂时不测
3. 启用这个USE导致运行时/测试出错，不启用就没问题，这个要慎重一点，去irc核实去bugzilla提bug
4. 这个USE flag对应于一个硬件，开发者暂时无法测试
5. riscv暂时不支持该USE flag对应的依赖或者功能

还有其他一些状况，非常建议去**irc沟通核实**，尤其是关键的包。

## workflow
**大致流程**：在独立测试环境中测试 -> {通过? 在本地git仓库中更新 -> 给官方仓库提pr; 不通过? 去社区询问 并 报bug}
### 测试环境
测试环境搭建可以参考<https://wiki.gentoo.org/wiki/Test_environment> <https://wiki.gentoo.org/wiki/Package_testing>。zfs btrfs 都有快照功能，可以方便地回退。一些容器可以隔离测试环境，最简易的chroot -> systemd-nspawn -> lxc -> docker 复杂。

我的配置是，利用libvirtd+qemu来管理riscv虚拟机，可以通过`virsh list`查看已有的虚拟机。储存管理用的是zfs volume，因为libvirtd自带支持。通过ssh连接控制，出了问题也可以进console `virsh console ANY_VM_DOMAIN`查看。gentoo系统repo通过squashfs来锁定某个日期的版本。另外虚拟机通过distcc将编译转移到主机上的交叉编译器，并用ccache加速。网络是libvirtd默认的NAT，通过vnet，iptables masquerade和libvirt内置的dns实现。

### 在测试环境的测试流程
主要通过修改`/etc/portage/`下的make.conf(portage的环境变量)、package.acceptkeywords/ (覆盖系统repo给没有keyword的软件加上keyword)、env/和 package.env/（设置某个软件的环境变量）、package.use（调整USE flag），profiles/(覆盖系统profile设置)等来，不建议直接修改系统源repo。当然也有把一个本地git仓库目录设置为系统源repo的操作，就能直接操作系统源了。

tips: 为单独的包调整单独的环境变量可以参考<https://wiki.gentoo.org/wiki/Debugging>

1. 在package.acceptkeywords中加入最顶层的要keyword的xxx包，加入在package.env中给他起用test，调整好USE
2. emerge --autounmask --verbose xxx包。这里有个hack，autounmask对比根据字典序文件名排最后的那个配置文件 来更新unmask，所以可以故意加入一个空的zzz文件，就能得到一个单独的叫._zzz_XXXX的文件，包含缺失keyword的依赖。
3. 利用这个文件批量更新package.env和package.use
4. 重复2. 3.多次把所有依赖都找到。一口气用emerge全部测好
5. 如果某个use有问题，在profiles/package.use.mask中调整
也可以用git管理`/etc/portage/`

### 更新到独立git仓库
首先clone一份官方git仓库到本地<https://github.com/gentoo/gentoo>。根据<https://wiki.gentoo.org/wiki/Gentoo_git_workflow> <https://devmanual.gentoo.org/ebuild-maintenance/git/index.html>的提示来配好gpg和git，最下面也有很多非常棒的git教程，比如<http://gitready.com/>。注意每个commit都要是atomic完整独立的，commit 信息的格式可以参考repo上的，一般是:
```
影响到的文件/包: 简明的更新信息
空行
更多解释（如果需要的话）
空行
一些bug: closes: acked: 等tag信息

例子：
sys-apps/util-linux: keyword 1.2.3-r4 for ~riscv

nothing to say here

bug: 12345
closes: 233434
Signed-off-by: <your email>
```
**Signoff-by是必须的**，不过如果配置好git和portage就可以自动sign。有时候repoman也会加入 portage version的信息

___！！！！注意只能rebase，不要有merge commit！！！！____

- 具体到加入keyword的流程:

比较传统的可以使用repoman，也可以用pkgdev pkgcheck。借助bash的小工具pushd和popd可以自顶向下深度优先or广度优先commit，也可以从底往上。

更改keyword不要手动了，直接`ekeyword ~riscv xxx_package`

`repoman commit`可以自动帮你填入部分commit message并做一些单独的依赖检查。全部commit完后 `repoman full -d`可以包含dev branch做QA检查，不过repoman很慢。

pkgcore是个比repoman快很多的工具。`pkgdev commit -s` 可以检查依赖，并自动识别更新填写commit message，如果git配置好的话还会填上signed-off-by。`pkgcheck scan --commit official_repo..your_branch`可以只检查新的commit而不用像repoman full一样全扫一遍。另外还有限制profile的功能。`pkgdev showkw`也可以快速查看本地git仓库某个包的keyword。参考<https://blogs.gentoo.org/mgorny/2019/12/12/a-better-ebuild-workflow-with-pure-git-and-pkgcheck/>

更多的dev script可以看看<https://github.com/mgorny/mgorny-dev-scripts>，这个也是个包 app-portage/mgorny-dev-scripts 。

头几次经常容易犯错，需要更改旧的commit。针对我犯过的错误的解决方法,参考<https://stackoverflow.com/questions/1186535/how-to-modify-a-specified-commit>，<https://stackoverflow.com/questions/13043357/git-sign-off-previous-commits>,批量修改可以借助format-patch sed am <https://mijingo.com/blog/creating-and-applying-patch-files-in-git>

- stgit:stack based git也是改commit很方便的工具。官方tutorial很好<https://stacked-git.github.io/>

`stg init`初始化stg分支，`stg uncommit -n/--to`取消一些commits，`stg series`查看stack，`stg pop push goto`在stack上移动，`stg commit`把stack重新commit回git。一般在用stg的时候，stg的commit之前不要动git的tree，会出现错误还可能弄坏commit。不过也有hack，比如不用rebase改commit的技巧：stg回退到恰好要修改的commit之前，pop/goto/commit保证目标commit之前的commit都不在stack上（否则stg会出错），这就相当于之后的commit都被移除了（在stg的stack中），于是就可以随便用git添加和修改新的commit了，或者stg还可以把某个patch转化到working tree，之后stg再push回来。还有一些hack可以分割commit，参考<https://stackoverflow.com/questions/22702642/how-to-split-a-patch-in-stgit>
