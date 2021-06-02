# Gentoo使用
emerge是e-merge，emaint是e-maintanence

### 一些有用的提示
[cheatsheat](https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet#Overlays)
* equery, eix，euse
* bash-completion, gentoo-bashcomp, tldr， 自动补全和太长不读提示，让生活更美好
* porthole： portage的一个gui前端
* layman：管理overlay仓库的
* 查找其他的overlay，[](http://gpo.zugaina.org/Overlays) [](https://overlays.gentoo.org/)

## 探索实践问题
最后留一些问题方便自学探索熟悉gentoo，感谢mentor提供，我自己的回答放在文档最后对应章节。
1. Keyword,USE,FEATURES, /etc/portage/make.conf都是什么。EAPI和GLEP是什么的缩写，是干什么的？
2. 如何根据某个文件寻找这个文件属于的包名（package name）？比如那个包安装了/usr/bin/emerge这个文件。如何列出一个包的依赖？如何通过名字/功能/包描述查找一个包？
3. 还是如何根据某个文件寻找这个文件属于的包名？但这次这个包并没有安装在系统上。
4. overlay是什么
5. 怎么配置和管理服务，启用禁用，runlevel？openRC和systemd是什么？
6. 谁是willikins？  
