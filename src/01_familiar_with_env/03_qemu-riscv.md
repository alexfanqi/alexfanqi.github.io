## 交叉编译
首先就是要获得交叉编译器，gentoo可以直接用crossdev，先创建一个本地overlay，然后crossdev会帮你收拾一切，也可以参考crossdev的脚本写个自己的wrapper处理环境变量。参考<https://wiki.gentoo.org/wiki/Embedded_Handbook/General/Creating_a_cross-compiler>

懒得自己编译的话，也可以直接去[crossdev](https://toolchains.bootlin.com/releases_riscv64.html)下载编译好的工具包。

gentoo上编译内核直接使用genkernel就行，注意加入virtio支持，才能正常使用qemu。有个依赖需要perl，但perl支持riscv的multilib的ebuild似乎还有问题，需要打一个补丁，参考<https://bugs.gentoo.org/794463>

接下来gentoo上qemu配置可以参考<https://wiki.gentoo.org/wiki/QEMU/Linux_guest> 和 <https://wiki.gentoo.org/wiki/QEMU>，主要就是QEMU_USER_TARGETS和QEMU_SOFTMMU_TARGETS两个变量。

## qemu
gentoo的qemu自己安装了riscv的sbi。如果qemu没有带sbi的话（qemu会报错没有bios），可以去自己编译然后使用`-bios`指定。使用<https://lib.rs/crates/rustsbi> 或者<https://github.com/riscv/opensbi>或者 u-boot都行,关于sbi还是参考标准文档 <https://github.com/riscv/riscv-sbi-doc>，大致理解成固件就行了。

可能提示找不到/dev/rtc0，检查内核一般都是包括实时时钟的，这就说明没有硬件时钟，此时可以使用软件时钟。

另外也可以使用virish和virt-manager来管理qemu的虚拟机。

## stage3
应该直接用dlan的stage4包的，花费了挺多时间在补全dilfridge的stage3包。结果发现[raspberry pi的gentoo wiki](https://wiki.gentoo.org/wiki/Raspberry_Pi)包括了大部分遇到的问题。。。算是learn it the hard way了。

