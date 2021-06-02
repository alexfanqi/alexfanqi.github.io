# 熟悉Gentoo环境
## Gentoo AMD64在VirtualBox上的安装

因为[官方文档handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage)会更新和变化并且有中文翻译，所以本文档不具体讨论安装流程，只在这里补充一些细节和遇到的问题。

先说明我的配置，VirtualBox上分了50GB，2个分区,一个2GB给swap，剩下的都给另一个分区刷成btrfs。然后在第二个分区可以用btrfs建立subvolume，我使用了平铺布局，在顶层目录直接建了 @root, @home, @var, 挂载的时候使用参数-o subvol=@root 挂在对应目录就好了，之后gentoo全部安装好之后再搭配[snapper](https://wiki.gentoo.org/wiki/Snapper)就能方便地给分区进行快照和备份了。

因为Vbox虚拟机默认是BIOS模式，所以干脆用MBR格式的分区表，这里官方文档讲不需要/boot分区，可是唯一的可引导分区刷成了btrfs，开始还有点害怕grub2不支持btrfs想着要多划分一个ext格式的/boot，结果完全没问题，确实不需要独立划分/boot分区。

之所以不在实体机上直接安装gentoo是因为VirtualBox的快照功能，可以随时暂停拍个快照，之后直接恢复快照就能立刻继续工作，非常方便，不用重新吟唱咒语了。

cli安装Linux发行版流程基本都是先在某个现成的linux环境上把目标系统的文件和硬盘分区准备好，然后把proc下初始化好的硬件信息给mount到目标系统对应/proc下，然后chroot进目标系统，准备好内核再配置好字符集和时区等信息，最后安装grub等引导程序就完成了。Gentoo基本也是这个流程。注意有时候chroot之后会出现”无法找到某文件“但ls一下文件确实存在的情况，很有可能是这个文件是个软链接接并且指向的是chroot之外的文件系统。

Gentoo需要使用`eselect profile`选择profile，建议选择最精简的那个，方便快速且不容易有问题，之后完全安装成功再换profile是一样的。当然如果使用systemd要选带systemd标识的，一定要看清楚。

### 碰到的问题
前两个问题都是因为我开始选的不是最精简的profile，而是带desktop版的，如果选精简的就没事，不过都还容易解决。第三个问题我直接闪过了，嘿嘿。
1. cpu的编译FLAG
> The following REQUIRED_USE flag constraints are unsatisfied: cpu_flags_x86_sse? ( cpu_flags_x86_mmxext ) .

这个的原因大概是某些软件需要某个cpu指令集，但CPU FLAG里没加就没有编译对应支持。解决方法参考 [CPU flags unsatisfied](https://forums.gentoo.org/viewtopic-t-1061352-start-0.html), 大概就是多安装一个cpuid2cpuflags工具，帮你检测出cpu支持的指令集对应的FLAG，然后在make.conf加入CPU_FLAGS_X86="\<flags\>"里面就是了。

2. 软件编译的循环依赖 linux包管理器的日常~~~
> * Error: circular dependencies:
> (dev-libs/libical-3.0.8:0/3::gentoo, ebuild scheduled for merge) depends on
> (dev-libs/icu-68.2:0/68.2::gentoo, ebuild scheduled for merge) (buildtime_slot_op)
> (dev-lang/python-3.9.0-r1:3.9/3.9::gentoo, ebuild scheduled for merge) (buildtime)
> (net-wireless/bluez-5.55:0/3::gentoo, ebuild scheduled for merge) (buildtime)
> (dev-libs/libical-3.0.8:0/3::gentoo, ebuild scheduled for merge) (buildtime_slot_op) 

我的报错和最个不太一样，但是样子差不多。解决方法一般就是修改USE FLAG去掉其中一个包对另外一个包的支援，先编译一个出来。之后再把USE FLAG改回去，把那个支持给加上，重新编译就成了。 参考[Circular problem on fresh install](https://forums.gentoo.org/viewtopic-t-1128355-start-0.html)

3. 64位的cpu手动编译内核提示不支持选64位指令拒绝编译

不清楚我的是什么原因造成的，不过其他人用vbox也遇到了，[CPU you selected does not support x86-64](https://forums.gentoo.org/viewtopic-t-958268-start-0.html)。
我使用第一个办法手动编译内核，cpu类型选择的generic-x86-64, emerge --info，/proc/cpuinfo, uname -a全部显示的64位，可是仍旧提示不支持x86-64。此时就体现了Virtualbox的好处，当时的快照我还有，不知那个大佬能告诉我如何解决。不过，解决不了还躲不了吗？最后用最省事的方法3,用installkernel-gentoo把内核给搞好了。

