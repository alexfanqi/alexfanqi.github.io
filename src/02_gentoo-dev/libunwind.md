# on libunwind
libunwind 是一个编程获取call stack的API标准，为什么要是API标准而不是单独的库呢，可以参考这个[What kind of stack unwinding libraries do exist and what's the difference?](https://stackoverflow.com/questions/25356691/what-kind-of-stack-unwinding-libraries-do-exist-and-whats-the-difference)

比较好的解释资料：
- <https://eli.thegreenplace.net/2015/programmatic-access-to-the-call-stack-in-c/>
- 中文的
 <https://wdv4758h-notes.readthedocs.io/zh_TW/latest/libunwind.html>

strace如果编译带了libunwind，那么可以用-k来运行时查看stack trace,<https://www.programmersought.com/article/6034185246/>
 
## 有关libunwind的一些内容
1. <https://forums.gentoo.org/viewtopic-t-1068838-start-0.html> 这个是 static-libs use冲突，和libunwind没太大关系，但也挺有意思的。

<https://bugs.gentoo.org/612602>
这个主要是默认的use flag起了冲突，libunwind默认不启用static-libs，而libcxxabi默认要static-libs，有其他的包又依赖着libcxxabi，然后就打架了。可以修改其他的所有包去掉static-libs但是这样好像说会影响应该默认就能跑的clang -static，所以得把libcxxabi的static-libs留着，因此 默认给libunwind启用static-lib。

然后还没完，几年后又出了<https://bugs.gentoo.org/768978>，这次是因为zlib又依赖于libunwind了，然后社区觉得应该避免默认启用static-lib，同时又感觉人们不在乎libcxx被编成static还是不static (<https://bugs.gentoo.org/760504>)，于是就把unwind和llvm-unwind的默认static-libs又给去掉了。结果和之前一样，libcxx默认启用static-libs，要依赖libcxxabi，然后又依赖于libunwind。（没想到吧，我又改了回去，\#推一推眼镜）。于是这次终于从上层把libcxx的static-libs给去掉了。

2. 具体到riscv上的问题
<https://github.com/libunwind/libunwind/pull/267>
有待未来研究具体细节
