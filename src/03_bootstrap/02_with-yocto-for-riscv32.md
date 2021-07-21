借花献佛。因为qemu riscv32 user static有问题，portage的multi processing会陷入死循环等待某个子进程结束，所以用yocto来做一个能用的系统，然后在里面bootstrap gentoo。之所以不用buildroot，是因为buildroot不支持安装guest里面的native compiler，并且会把header和static lib什么的都删掉,(试了下修改makefile把这步骤跳过但还是有问题)，这样做让系统非常轻量，但不反而不是我们想要的。

另外推荐bootlin的buildroot和yocto的training workshop，<https://bootlin.com/training/>,lab slides比看它们的manual可清楚多了，yocto的文档2。还有oe的wiki挺旧了，还停留在说不支持python3的时代（实际当然支持啦）。yocto官方给的30分钟的简介也很不错<https://vimeo.com/36450321>

buildroot和yocto的许多概念和gentoo都很像，倍感亲切啊。



