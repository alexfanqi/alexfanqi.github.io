# Gentoo AMD64在VirtualBox上的安装

因为[官方文档handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage)会更新和变化并且有中文翻译，所以本文档不具体讨论安装流程，只在这里补充一些细节和遇到的问题。

_______
**注意！！** Gentoo的安装需要下载livecd和stage3两个包，___一定要看清楚架构！！！___ 否则就会出现[我碰到的第三个问题](#碰到的问题)。gentoo的keyword x86指的是32位的，对应livecd和stage3安装包标记是\*-i686-\*；amd64才是如今基本普及了的64位架构（因为是amd主导的标准嘛，intel的IA64凉了），对应的标记是\*-amd64-\*。**中途没报错不代表选对了包和参数！！！** ，因为64位架构amd64能够兼容旧的架构32位的x86,你可能不经意间在你的64位机子上安了32位的系统，直到某天碰到某个软件不支持32位才发现，毕竟32位的x86芯片慢慢要被淘汰了。目前来看**从32位改成64位只能重装Gentoo**，[参考](https://wiki.gentoo.org/wiki/AMD64/FAQ#Can_I_upgrade_from_my_x86_system_to_AMD64_by_doing_emerge_-e_.40world.3F)。
_______

还有gentoo默认使用的是openrc管理之前的systemv的init。目前用户量比较多的发行版比如ubuntu fedora opensuse都默认的是systemd。我还是选的默认openrc。**一定注意安装包的标记**，带systemd是systemd的，不带是openrc的。之后选profile的时候也要保持一致。安装完成之后还是可以切换openrc和systemd。

先说明我的配置，VirtualBox上分了50GB，2个分区,一个2GB给swap，剩下的都给另一个分区刷成btrfs。然后在第二个分区可以用btrfs建立subvolume，我使用了平铺布局，在顶层目录直接建了 @root, @home, @var, 挂载的时候使用参数-o subvol=@root 挂在对应目录就好了，之后gentoo全部安装好之后再搭配[snapper](https://wiki.gentoo.org/wiki/Snapper)就能方便地给分区进行快照和备份了。

因为Vbox虚拟机默认是BIOS模式，所以干脆用MBR格式的分区表，这里官方文档讲不需要/boot分区，可是唯一的可引导分区刷成了btrfs，开始还有点害怕grub2不支持btrfs想着要多划分一个ext格式的/boot，结果完全没问题，确实不需要独立划分/boot分区。

之所以不在实体机上直接安装gentoo是因为VirtualBox的快照功能，可以随时暂停拍个快照，之后直接恢复快照就能立刻继续工作，非常方便，不用重新吟唱咒语了。

cli安装Linux发行版流程基本都是先在某个现成的linux环境上把目标系统的文件和硬盘分区准备好，然后把proc下初始化好的硬件信息给mount到目标系统对应/proc下，然后chroot进目标系统，准备好内核再配置好字符集和时区等信息，最后安装grub等引导程序就完成了。Gentoo基本也是这个流程。注意有时候chroot之后会出现”无法找到某文件“但ls一下文件确实存在的情况，很有可能是这个文件是个软链接接并且指向的是chroot之外的文件系统。

Gentoo需要使用`eselect profile`选择profile，建议选择最精简的那个，方便快速且不容易有问题，之后完全安装成功再换profile是一样的。当然如果使用systemd要选带systemd标识的，一定要看清楚。

## 碰到的问题
前两个问题都是因为我开始选的不是最精简的profile，而是带desktop版的，如果选精简的就没事，不过都还容易解决。第三个问题乃是操作失误，不过对新手来说不容易及时意识到。
1. cpu的编译FLAG
> The following REQUIRED_USE flag constraints are unsatisfied: cpu_flags_x86_sse? ( cpu_flags_x86_mmxext ) .

这个的原因大概是某些软件需要某个cpu指令集，但CPU FLAG里没加就没有编译对应支持。解决方法参考 [CPU flags unsatisfied](https://forums.gentoo.org/viewtopic-t-1061352-start-0.html), 大概就是多安装一个cpuid2cpuflags工具，帮你检测出cpu支持的指令集对应的FLAG，然后在make.conf加入CPU_FLAGS_X86="\<flags\>"里面就是了。

2. 软件编译的循环依赖 linux包管理器的日常~~~
> Error: circular dependencies:
> (dev-libs/libical-3.0.8:0/3::gentoo, ebuild scheduled for merge) depends on
> (dev-libs/icu-68.2:0/68.2::gentoo, ebuild scheduled for merge) (buildtime_slot_op)
> (dev-lang/python-3.9.0-r1:3.9/3.9::gentoo, ebuild scheduled for merge) (buildtime)
> (net-wireless/bluez-5.55:0/3::gentoo, ebuild scheduled for merge) (buildtime)
> (dev-libs/libical-3.0.8:0/3::gentoo, ebuild scheduled for merge) (buildtime_slot_op) 

我的报错和最个不太一样，但是样子差不多。解决方法一般就是修改USE FLAG去掉其中一个包对另外一个包的支援，先编译一个出来。之后再把USE FLAG改回去，把那个支持给加上，重新编译就成了。 参考[Circular problem on fresh install](https://forums.gentoo.org/viewtopic-t-1128355-start-0.html)，[不同循环依赖的类型](https://devmanual.gentoo.org/general-concepts/dependencies/index.html#circular-dependencies)

3. 64位的cpu手动编译内核提示不支持选64位指令拒绝编译 

这个问题就是选错了安装包，纯粹操作失误，64位amd64选成了32位i686了，当时被自己蠢哭了，不过还好也有其他人犯错（窃喜），[CPU you selected does not support x86-64](https://forums.gentoo.org/viewtopic-t-958268-start-0.html)。发现这个问题还是我使用第一个办法手动编译内核，cpu类型选择的generic-x86-64, 64位系统，emerge --info，/proc/cpuinfo, uname -a全部有显示的64位，结果还是提示`cpu you selected does not support x86-64`拒绝编译。用wiki上第2和第3个安装内核的方法都可能让你忽视这个失误。不过仔细看emerge --info还是能够发现问题的
> Portage 2.1.11.55 (default/linux/x86/13.0, gcc-4.6.3, glibc-2.15-r3, 3.4.42-std3 51-amd64 x86_64

注意到虽然后面有`x86_64`，但是前面的`default/linux/x86`就说明了是32位的了，正确的应该是`default/linux/amd64`。
