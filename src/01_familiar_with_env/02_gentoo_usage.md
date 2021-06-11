# Gentoo使用

## Big Picture
Gentoo和arch差不多都是从仓库取得源代码，然后在用户机子上编译，然后安装的。gentood的包管理系统是portage，对应的cli命令是emerge，就像arch的pacman。所谓e-merge就像是git merge，把repo仓库的代码给merge到用户机子上，当然作为包管理器emerge也有其他功能。除了portage当然也有其他的包管理器。每个包都有一个ebuild文本文件来记录包的相关信息，gentoo的repo就是一堆ebuild文本文件和某些其他信息的集合，也叫overlay。要管理这些包就要一些flag，包括keyword，use flag和各种[variable]( https://devmanual.gentoo.org/ebuild-writing/variables/)等，gentoo的包管理大体也就基于这些flag。

刚使用emerge可能很迷，与apt和dnf会有些不同。首先，@world登记了系统中明确要保留的程序，包括用户安装的包和系统的@profile及gentoo必需的@system。删除一个包要先deselect，把包移除@world，声明我不需要这个包了系统没必要为我保留了，但包还没被删除（还可以--select给加回来），之后某天可以统一depclean计算各种依赖，如果没有别的软件依赖它，就一并完全删除。这个@world机制就motivate了--oneshot存在的意义。

***oneshot的use case***: --oneshot就是安装这个包但不加入@world里。典型的use case是，你只想重新编译一下某个包，但不确定这个包是不是其他包的依赖，也不确定这个包是否加入到@world里了。想象一下如果此时不用oneshot，这个包就会自动加入@world里，如果这个包之前不在@world里而只是某个软件的依赖，那它就会被声明为系统明确要保留，即使依赖它的软件被卸载了，它也不会被depclean给删掉，过个几天你也忘了这事了，于是一个不需要的包以及它自己的依赖就可能永远被留在系统里了。。。。

另外，portage安装软件会先进入一个临时目录/tmp/portage/category/package下进行编译安装，包括应用一些用户patch，然后再应用到用户系统。portage对某些文件有修改保护，比如/etc，安装包修改时会创建一个cfg_num原文件名副本，之后用户可以手动或者用etc-update自动替换，这样也就不用担心装软件中途停止搞乱系统了。

理解了这些感觉就好了，openrc和systemd 服务管理各个系统大同小异。

## 一些有用的提示
[cheatsheat](https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet#Overlays)
* equery, eix，euse，eselect，emaint，portageq
* bash-completion, gentoo-bashcomp, tldr， 自动补全和太长不读提示，让生活更美好
* porthole： portage的一个gui前端
* layman vs eselect repository：管理不同overlay仓库的
* 查找更多overlay (repo)，<https://overlays.gentoo.org/>,<http://gpo.zugaina.org/Overlays>
* ccache: 大概是可以cache编译过的obj，之后直接拿来用，节省时间
* 完成后提醒：emerge 有一个-A选项，可以在命令完成后“滴”响下铃提醒你，这样就不用坐在那里盯着了。更复杂的提醒可以通过配置portage的log直接发email，这个还是在irc上学到的，我还没试过不过应该可行，参考 <https://wiki.gentoo.org/wiki/Portage_log#Configuring_for_e-mail>
* genlop: 估计编译时间
* 把编译目录通过tmpfs放到ram上: <https://wiki.gentoo.org/wiki/Portage_TMPDIR_on_tmpfs>

## 探索实践问题
最后留一些问题方便自学探索熟悉gentoo，感谢mentor提供，我自己的回答放在文档最后对应章节。
1. Keyword,USE,FEATURES, /etc/portage/make.conf都是什么。EAPI和GLEP是什么的缩写，是干什么的？
2. 如何根据某个文件寻找这个文件属于的包名（package name）？比如那个包安装了/usr/bin/emerge这个文件。如何列出一个包的依赖？如何通过名字/功能/包描述查找一个包？
3. 还是如何根据某个文件寻找这个文件属于的包名？但这次这个包并没有安装在系统上。
4. overlay是什么
5. 怎么配置和管理服务，启用禁用，runlevel？openRC和systemd是什么？
6. 谁是willikins？  
