这部分可以参考目录下的我做报告时的pdf slide。

用emerge交叉编译的时候可以把SYSROOT和ROOT放在一个目录，并起用pkgbuild生成安装包，之后通过安装包指定安装到正确的ROOT。

首先要理清楚各种变量，sysroot, root, broot, cbuild,chost等等，这些在<https://wiki.gentoo.org/wiki/Embedded_Handbook> <https://devmanual.gentoo.org/ebuild-writing/eapi/index.html> 以及mgorny大佬的文档都有<https://mgorny.pl/articles/the-ultimate-guide-to-eapi-7.html> <https://mgorny.pl/articles/the-ultimate-guide-to-eapi-8.html>

## binfmt magic string

riscv64的
`:riscv64:M::\x7f\x45\x4c\x46\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xf3\x00:\xff\xff\xff\xff\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xff\xff:[binary_to_run]:`

riscv32的
`:riscv32:M::\x7f\x45\x4c\x46\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x03\x00\xf3\x00:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/opt/qemu-static/qemu-riscv32:`

替换\[binary_to_run\]为你想用来跑riscv的软件，比如qemu-riscv user。这些玩意是怎么来的,可以参考<https://en.wikipedia.org/wiki/Executable_and_Linkable_Format>以及内核文档<https://www.kernel.org/doc/html/latest/admin-guide/binfmt-misc.html>。对比一下上面的64和32位，你会发现第5个字节的一个bit不同，那个其实就是32和64位的标志啦。

TODO:写个自动生成binfmt magic string的python脚本


[crosstool-ng](https://crosstool-ng.github.io/)

catalyst参考<https://wiki.gentoo.org/wiki/Porting> (这个文档也有一些时间了，里面关于boostrap portage的内容已经过时太久了，不过还是有参考意义的)

interpreter: /usr/bin/qemu-riscv64
