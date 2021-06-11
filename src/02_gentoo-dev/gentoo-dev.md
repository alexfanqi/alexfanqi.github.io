 - EUSE很杂，不过官网有些不错的参考，<https://packages.gentoo.org/useflags>
 - slot和subslot，可以允许多个版本共存
 - sandbox机制，主要是保护文件系统，防止写的有毛病的ebuild不小心修改了它不应该修改的文件
 - 中途失败的话可以去/var/tmp/portage/下面查看log和编译文件
 
 - 管理仓库的时候，profile下没有 package.unmask，只能在package.mask里加-package，管理系统的时候在/etc/portage/下则可以用package.unmask，优先级比仓库中的mask高，所以可以取消mask
 
 - accepted-keyword和arch use flag是不同的东西
 - eclasses, optfeatures?
 - <https://devmanual.gentoo.org/ebuild-writing/variables/>
