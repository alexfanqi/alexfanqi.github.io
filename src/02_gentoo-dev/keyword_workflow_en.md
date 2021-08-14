# Gentoo Testing and Keywording Workflow
It is a simple




## test on VMs and devices



## Commit to the tree
### prepare the git tree


For a long time (actually only a few weeks), I have been using `repoman commit` and `repoman -d full` to do keywording. It is a simple tool to use, yet takes a long time to run on my ancient laptop. It is an excellent tool for people like me who like to take one or two hours' break and enjoy life. Because if you feed the whole portage tree into it, it kindly leaves you plenty of time finishing one anime episode or reading scifi book series. Therefore, despite its complaint, I used to only give it a single package directory to check, convincing myself the rest of tree is okay.

### old repoman workflow
My old workflow with repoman is 
1. ekeyword the toplevel package first and let repoman commit to fail and tell me all its dependencies.  

### new pkgdev pkgcheck workflow
pkgcheck

## Edit the commits
As I am new to the Gentoo world, I always make mistakes like forgetting signoff, wrong commit sequences, missing bug reference in commit message that I need to come back in my commit tree to correct them. It is pretty simple if the commit to be correted is just the HEAD commit, then a single `git reset HEAD~1` or `git ammend` will suffice. But if it is a few commits old in the tree, I cannot simply reset all commits till it and redo them all manually. 

Before I get more familiar with git, I did the hardest way. Run `git format-patch #commit_to_modify` to dump all commits after the to-be-modified commit to patches, do some modification on it and apply the patches back with `git am`.

A slightly better way is using rebase. Run `git rebase --interative #commit_to_modify^` to enter an interative rebase procedure. Modify the `pick` to `edit` in that commit line. Now, the commit to modify becomes the topmost commmit, so I can make changes easily to it with `git commit --amend`. After everything is done, `git rebase --continue` to go back to apply all the rest commits.

### introduce stgit
More flexible approach is using stgit, ie. stack based git. 


### Add signoff to multiple commmits


--------------
## Reference
1. <https://blogs.gentoo.org/mgorny/2019/12/12/a-better-ebuild-workflow-with-pure-git-and-pkgcheck/>
2. <https://github.com/mgorny/mgorny-dev-scripts>
3. <http://gitready.com/>
