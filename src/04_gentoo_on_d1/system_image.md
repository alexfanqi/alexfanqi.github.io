2021-08-13
目前有参考意义的镜像包括，韦东山老师做的buildroot <https://gitee.com/weidongshan/neza-d1-buildroot>，smael holland分离出来的boot0以及还在开发中的mainline uboot和kernel 14 <https://linux-sunxi.org/Allwinner_Nezha>, 以及 tekkamaninja傅玮做的fedora镜像<https://koji.oepkgs.net/koji/taskinfo?taskID=986>。其中文档最友好的还是smael的。

官方的tina是基于openwrt的，可惜我对openwrt还不熟。

bootloader一般都是分成两个img，姑且叫boot0和boot_package吧。但具体放什么好像不太一样。
不过偏移都默认的一样，扇区大小512byte，boot0在16扇区，boot_package在32800扇区（大约16Mib的偏移，浪费空间！）
这就造成分区的时候要注意，不能让分区表和这两个img重叠了。

1. mbr的header只有在开始第一个扇区，因此不必关心header，但第一个分区最好在16Mib之后，考虑到对齐或许放到20Mib?
2. gpt的header默认在2-34扇区（从1开始数，一号扇区是mbr保留的），必然和boot0打架。但可以用gpart进入expert模式 j命令，移动到之后，别撞到那两个img就成。所以就是，扇区1:protected mbr, 扇区2:gpt header， 扇区M到M+31分区表。另外gpt在磁盘/sdcard的末尾还有备份，所以直接dd和sd卡大小不同镜像会有警告，直接忽略不会影响使用或者用gpart修复都行。

boot_package 32800是可以修改的，可以通过spl(secondary program loader)也就是boot0的代码的`include/spare_head.h`文件中宏`UBOOT_START_SECTOR_IN_SDMMC`修改，比如调整到2*1024*2扇区也就是2Mib的位置。

## smael
<https://linux-sunxi.org/Allwinner_Nezha>流程写的非常详细，里面讲的boot_package是toc1格式的。kernel似乎只能用smael做的，fedora的kernel就不显示输出。
本来以为用这个就能手动搞出来一个完整的systemd镜像。可惜似乎由于timer相关的问题，systemd启动后会挂掉，进入opensbi unhandled trap。另外一个问题是genkernel生成的initramfs用不成。。。

还有可能遇到uImage过大的情况，uboot默认的kernel上限是8Mb，可以在uboot代码的`include/configs/sun20iw1p1.h`文件单独改，也可以在`./common/bootm.c`中全局改，把宏`CONFIG_SYS_BOOTM_LEN`调高一点。

还有就是driver初始化可能有延时，造成kernel的mmc还没好就要运行init了，可以加内核参数rootwait或者init-delay。

另外启动时console是ttyS0，earlyprintk可以查看device tree文件，可以找到uart0被alias成了serial0,所以earlyprintk=serial0就成。


## fedora的镜像

tekkamaninja提供了boot0和fex格式的boot_package（因为社区逆向的sunxi-tools中的fex反编译工具还不支持d1,所以我也不知道fex里面是啥，strings看一下有一些环境变量，但也不止是环境变量）。这个fex文件是sunxi的特色，不得不尝。

然后他的kernel dtb 和initramfs是做在一个boot.img里的，（试过了，boot分区其他文件都是可有可无的，包括extlinux，sunxi的uboot压根不认），应该是通过安卓的工具链mkbootimg做的，韦东山老师的buildroot的`board/Neza/d1/`下的post-build.sh脚本写的是`mkbootimg --kernel  Image  --ramdisk  ramdisk.img --board  d1-nezha_min --base  0x40200000 --kernel_offset  0x0 --ramdisk_offset  0x01000000 -o  boot.img`，不过我试着调整调整，没成。。

听tekkamaninja说以后还要用uboot chainload grub，所以才多留一个分区。。。我查了查目前grub还是虽然支持了riscv但仍旧处于啥也不能启动的状态。

在tg d1群里也有其他分析sunxi boot.img格式的人，repo <https://github.com/wargio/d1-nezha-tools>。

## sunxi的tina
关键的内核和bootloader都在`./lichee/`下面，里面有个脚本可以参考。makefile流程都在`./build`目录，配置在`./device`目录下，包括device tree。另外readme可以看官方文档<https://d1.docs.aw-ol.com/>。

还有这个tina对编译工具链挺敏感，官方是建议用他们的ubuntu的镜像，我在自己的ubuntu 20.04上编译按官方流程都会出问题（当然依赖都安装好了），似乎是由于一些32位的包。gentoo上会出ncurses的问题，用不成menuconfig，不过拷贝个config过去就好。不过又会由于工具链过新出现下面说的vdso问题。

## genekernel+crossdev交叉编译内核
genkernel看gentoo wiki就行，交叉编译不支持zfs，不过d1的那小内存用个啥zfs。。。

注意某些sunxi的module不是随便选就成的，会编译失败。genkernel有时候模组编译失败不会停止，导致最后initramfs里一个模组也没有，需要看清楚或者用lsinitramfs核实。

vector指令拓展需要sunxi的打过补丁的gcc，然而我们要用gentoo系统的genkernel和工具链，所以很可惜要禁用vector拓展。

用高版本binutil交叉编译内核会出vdso的bug，在5.12内核中都还有，d1的5.4内核当然也受影响，需要一个新patch <https://github.com/0day-ci/linux/commit/dfdcaf93f40f0d15ffc3f25128442c1688e612d6>。或许需要把这个也加上<https://patchwork.kernel.org/project/linux-riscv/patch/20201017002500.503011-1-palmerdabbelt@google.com/>
