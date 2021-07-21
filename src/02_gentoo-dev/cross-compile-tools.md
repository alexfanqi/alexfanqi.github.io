# catalyst

# ccache
ccache会注入错误的libstdc++库，造成某些诡异的错误比如`/libstdc++.so.6: version 'GLIBCXX_3.4.26' not found (required by x86_64-pc-linux-gnu-g++)`，strings一下你会发现确实不包含那个symbol。编译ccache的时候建议把libstdc++编成static的，用USE="static-c++"。当然有时候也不一定是ccache的锅，llvm的gold linker也会导致这个问题。参考[Bug 761220](https://bugs.gentoo.org/761220),[Bug 644772](https://bugs.gentoo.org/644772),[GLIBCXX_3.4.28 not found requird by libQt5Core.so.5](https://forums.gentoo.org/viewtopic-p-8513692.html?sid=7dd5835a7e60ae87ba41e58d942a7ff6)。类似的问题被报了很多次了，好多人都遇到过，不过原因都不太一样。

# distcc
参照gentoo的wiki就行。注意wiki里面对/etc/conf.d的修改是只会对openrc生效的，如果用的systemd有点麻烦。
1. systemd的env里不能直接用PATH=/home/xxx:$PATH这种操作，需要写个wrapper或者修改service的execstart,比如这样<https://askubuntu.com/questions/1014480/how-do-i-add-bin-to-path-for-a-systemd-service>。
2. env-update只会更新/etc/environment,进而影响systemd的systemd不会承认里面的环境变量，所以不要像wiki那样修改env.d。全部在/etc/systemd/system/distccd.service.d/中设置。这其实也是env-update的一个坑，参见<https://bugs.gentoo.org/704416>
3. 还要注意发送到remote的任务是以一个non-login的distcc用户运行的，注意设置好权限，尤其是和ccache搭配的话，一定要覆盖ccache，否则会出现蜜汁的failed to mkdir /dev/null/.cache/。
4. 还有没必要改DISTCC_DIR，wiki上会造成权限问题，就储存着lock和state。只不过distccmon用不成而已，其实也不需要，在journalctl里都能看到。
5. 做测试的时候建议`export DISTCC_FALLBACK="0"`,`export DISTCC_VERBOSE="1"`，确认是真的成功在remote上编译了而不是fallback到了本地。`export DISTCC_SAVE_TEMPS=1`， 可以去distcc tmp目录 (默认/tmp)去看看distcc_*.stderr。
6. 不建议把distcc暴露到外网去，别人能够通过distcc用户权限任意执行代码。
7. 有个zeroconf的use flag，host多的话可以试试。
8. client端空间紧张的话可以不用在client端设置ccache，wiki里面client的设置里有关ccache的全忽略就行了。
9. 某些包用distcc编译会失败，可以试试no-distcc-env
