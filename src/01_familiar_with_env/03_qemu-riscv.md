## 交叉编译
首先就是要获得交叉编译器，gentoo可以直接用crossdev，先创建一个本地overlay，然后crossdev会帮你收拾一切，也可以参考crossdev的脚本写个自己的wrapper处理环境变量。参考<https://wiki.gentoo.org/wiki/Embedded_Handbook/General/Creating_a_cross-compiler>

非gentoo系统懒得自己编译的话，也可以直接去[bootlin](https://toolchains.bootlin.com/releases_riscv64.html)下载编译好的工具包，当然gentoo用crossdev还是最方便的。

gentoo上编译内核直接使用genkernel就行，注意加入virtio支持，才能正常使用qemu。有个依赖需要perl，但perl支持riscv的multilib的ebuild似乎还有问题，需要打一个补丁，参考<https://bugs.gentoo.org/794463>

接下来gentoo上qemu配置可以参考<https://wiki.gentoo.org/wiki/QEMU/Linux_guest> 和 <https://wiki.gentoo.org/wiki/QEMU>，主要就是QEMU_USER_TARGETS和QEMU_SOFTMMU_TARGETS两个变量。

## 编译环境的隔离
建议试试lxc或者更高级一点的docker来隔离环境，同时管理也方便。但不是必须的，遇到的把host环境错误地引入目标系统的情况不多。

## qemu
gentoo的qemu自己安装了riscv的sbi。如果qemu没有带sbi的话（qemu会报错没有bios），可以去自己编译然后使用`-bios`指定。使用<https://lib.rs/crates/rustsbi> 或者<https://github.com/riscv/opensbi>或者 u-boot都行,关于sbi还是参考标准文档 <https://github.com/riscv/riscv-sbi-doc>，大致理解成一个简化统一的启动固件就行了,这里有一篇讲的比较清楚[目前D1芯片引导启动流程过长的问题](https://bbs.aw-ol.com/topic/165/%E6%B7%B1%E5%BA%A6%E8%AE%A8%E8%AE%BA-%E7%9B%AE%E5%89%8Dd1%E8%8A%AF%E7%89%87%E5%BC%95%E5%AF%BC%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E8%BF%87%E9%95%BF%E7%9A%84%E9%97%AE%E9%A2%98-%E4%BB%A5%E5%8F%8A%E5%AF%B9risc-v%E4%B8%8B%E5%BC%95%E5%AF%BC%E7%A8%8B%E5%BA%8F%E7%8E%AF%E5%A2%83%E7%9A%84%E6%80%9D%E8%80%83)，虽然本身是讨论d1开发板的。

可能提示找不到/dev/rtc0，检查内核一般都是包括实时时钟的，这就说明没有硬件时钟，此时可以使用软件时钟。

另外也可以使用virish和virt-manager来管理qemu的虚拟机。

## stage3
应该直接用dlan的stage4包的，花费了挺多时间在补全dilfridge的stage3包（其实没啥好补的，当时对linux系统可以做到的简洁程度一无所知，通过不停试错花了很多时间）。结果发现[raspberry pi的gentoo wiki](https://wiki.gentoo.org/wiki/Raspberry_Pi)包括了大部分遇到的问题。。。算是learn it the hard way了。想让系统启动可用，最小的可以用带dhcp的busybox，或者把openrc udev dhcpcd都编译进去也行。

## 编译linux嵌入式系统入门
资源有很多，可惜当时是自己把坑踩遍了，才发现一堆资源的。入门级讲的比较好的<https://www.youtube.com/watch?v=LSQf3iuluYo> (口音挺重，不过很清晰)

