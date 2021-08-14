 - EUSE很杂，不过官网有些不错的参考，<https://packages.gentoo.org/useflags>
 - slot和subslot，可以允许多个版本共存
 - sandbox机制，主要是保护文件系统，防止写的有毛病的ebuild不小心修改了它不应该修改的文件
 - 中途失败的话可以去/var/tmp/portage/下面查看log和编译文件
 - 想要debug的话，可以用package.env，把-g -ggdb给打开，还有一些feature，比如keepwork,keeptmp,splitdebug,etc
 
 - 管理仓库的时候，profile下没有 package.unmask，只能在package.mask里加-package，管理系统的时候在/etc/portage/下则可以用package.unmask，优先级比仓库中的mask高，所以可以取消mask
 
 - accepted-keyword和arch use flag是不同的东西
 - eclasses, optfeatures?
 - <https://devmanual.gentoo.org/ebuild-writing/variables/>

查缺少riscv keyword的dependency
`for pkg in $(qdepends -t xdg-desktop-portal-kde|sed -e 's/^>=/=/g' -e 's/\(\S+\)/"\1"/g') ; do qkeyword -p $pkg -n riscv; done`
查找某个category中缺失riscv keyword的包
`for pkg in $(eix -C -c net-analyzer|head -n -3|cut -d ' ' -f2); do qkeyword -p $pkg -n riscv; done`
以上两个有了emerge --autounmask后就不怎么需要了

mgorny大佬的一些dev用小脚本
<https://github.com/mgorny/mgorny-dev-scripts>

快速改软件源码（不需要用户patch），当然只适合测试和权宜之计
<https://www.reddit.com/r/Gentoo/comments/52qhcn/how_do_i_manually_edit_my_sources_for_emerge/>

在accept keywords里用`<category/PN-9999`，可以避免把live ebuild也给搞进来 

善用btrfs和zfs的snapshot功能，比如测试一个软件之前打个snapshot，然后emerge就不必用oneshot了，测试完之后再restore回去，参考<https://wiki.gentoo.org/wiki/Chroot_for_package_testing>。避免在运行的时候打snapshot，相当于突然关机内存数据全没。要是对这个有要求的话可以参考[Consistently backup your virtual machines using libvirt and zfs](http://www.linuxsystems.it/2018/10/consistently-backup-your-virtual-machines-using-libvirt-and-zfs-part-1/)。

squashfs+overlayfs非常适合gentoo repo这种文件很多的目录，也很适合做测试，临时改一个文件试完马上就恢复回去

KEYWORDS.dropped 是维护者主动drop某个keyword，就是在arch前加个-，比如KEYWORDS="-riscv"，比较重要的包需要维护者再发一个rekeywording request。

对每个包可以设置单独的环境，参考wiki gentoo debug。比如不要在全局设置FEATURES=”test",或者对单个软件限制-j并行数量。

USE debug 不是用来获取debug info的，不要随便启用它，参考gentoo debug的wiki的

注意CFLAGS CXXFLAGS都是会覆盖的，在packege.env设置时要CFLAGS="${CFLAGS} -DSOME_FLAGS", 而FEATURES则是会累加到一起的，所以直接FEATURES="test"就行。

不同文件use flag的优先级，<https://wiki.gentoo.org/wiki/USE_ORDER>, 可以用`portageq envvar USE_ORDER`去查

euse -i cups,可以查询某use具体在那个文件中设置的,make.conf/package.use/...

有时候碰到的use flag加个括号，比如(-cups)，指的是<https://archives.gentoo.org/gentoo-user/message/85a73d3af73e2efb3f28f4bada1aa0b1>

关于git sign off的一个坑，commit-tree不会加signed off by，网上的答案大部分都走偏了,<https://pmhahn.github.io/git-signoff/>

一堆的package.*文件实在头痛，可以试试mgorny大佬的flaggie

除了MAKEOPT=“-jN -lM”外还可以设置EMERGE_DEFAULT_OPTS="--jobs L --load-average H"，能更充分利用cpu。 

keyword中遇到循环依赖处理方式: 比如qtchooser\[test\]->qttest->qtgui-qtchooser，这种情况下只要都测试好了，可以直接keyword，不需要mask test再keyword再unmask。

gui qt5 gtk这几个use flag含义比较模糊，可以参考当初讨论如何设计的邮件<https://archives.gentoo.org/gentoo-dev/message/eecad370248118c474a0d819fa7f3576>

使用xvbf和x11vnc可以在没有显示屏的虚拟机中测试gui软件，参考<https://en.wikipedia.org/wiki/Xvfb>
