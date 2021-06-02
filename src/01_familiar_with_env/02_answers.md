 最后留一些问题方便自学探索熟悉gentoo，感谢mentor提供，我自己的回答放在文档最后对应章节。
1. Keyword,USE,FEATURES, /etc/portage/make.conf都是什么。EAPI和GLEP是什么的缩写，是干什么的？
    * Keyword 是每个文件包对各个架构的支持情况 稳定？测试中？ known to broken?
    * USE就是标记软件包特性支持和依赖的flag,每个软件包可以在/etc/portage/packege.use改，优先级比make.conf高
    * make.conf是用来设置portage的全局环境的，比如USE，编译选项，CPU_FLAGS等等等等
    * FEATURES是portage的一些不会那么影响编译出的软件包的配置
    * [EAPI](https://devmanual.gentoo.org/ebuild-writing/eapi/) 大概就是包管理器标准的版本号
    * [GLEP](https://www.gentoo.org/glep/) gentoo linux enhancement proposals 改善gentoo的一些书面的提议吧
2. 如何根据某个文件寻找这个文件属于的包名（package name）？比如那个包安装了/usr/bin/emerge这个文件。如何列出一个包的依赖？如何通过名字/功能/包描述查找一个包？
    * `equery belongs`
    * `equery depends|depgraph`
    * `emerge --search|--searchdesc` 或者 `equery list` 
3. 还是如何根据某个文件寻找这个文件属于的包名？但这次这个包并没有安装在系统上。
    * app-portage/pfl 或者 [Portage File List](https://www.portagefilelist.de/site/query)
4. overlay是什么
    * 大致就是指的ebuild软件仓库
5. 怎么配置和管理服务，启用禁用，runlevel？openRC和systemd是什么？
    * [openrc & systemd](https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet#OpenRC)
    * 如今日常发行版基本默认都用的systemd，Gentoo、默认的openRC
    * [各种init系统的比较](https://wiki.gentoo.org/wiki/Comparison_of_init_systems)
6. 谁是willikins？
    * [willikins](https://wiki.gentoo.org/wiki/Willikins) 是gentoo irc上的bot
