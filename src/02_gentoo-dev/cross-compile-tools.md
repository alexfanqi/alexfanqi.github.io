# catalyst

# ccache
ccache会注入错误的libstdc++库，造成某些诡异的错误比如`/libstdc++.so.6: version 'GLIBCXX_3.4.26' not found (required by x86_64-pc-linux-gnu-g++)`，strings一下你会发现确实不包含那个symbol。编译ccache的时候建议把libstdc++编成static的，用USE="static-c++"。当然有时候也不一定是ccache的锅，llvm的gold linker也会导致这个问题。参考[Bug 761220](https://bugs.gentoo.org/761220),[Bug 644772](https://bugs.gentoo.org/644772),[GLIBCXX_3.4.28 not found requird by libQt5Core.so.5](https://forums.gentoo.org/viewtopic-p-8513692.html?sid=7dd5835a7e60ae87ba41e58d942a7ff6)。类似的问题被报了很多次了，好多人都遇到过，不过原因都不太一样。
