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

在accept keywords里用`< category/PN-9999`，可以避免把live ebuild也给搞进来 

善用btrfs和zfs的snapshot功能，比如测试一个软件之前打个snapshot，然后emerge就不必用oneshot了，测试完之后再restore回去，参考<<https://wiki.gentoo.org/wiki/Chroot_for_package_testing>。避免在运行的时候打snapshot，相当于突然关机内存数据全没。要是对这个有要求的话可以参考[Consistently backup your virtual machines using libvirt and zfs](http://www.linuxsystems.it/2018/10/consistently-backup-your-virtual-machines-using-libvirt-and-zfs-part-1/)。

squashfs+overlayfs非常适合gentoo repo这种文件很多的目录，也适合很做测试，临时改一个文件试完马上就恢复回去

KEYWORDS.dropped 是维护者主动drop某个keyword，需要维护者再发一个rekeywording request，不影响不在这个里面的arch

USE debug 不是用来获取debug info的，不要随便启用它，参考gentoo debug的wiki的
