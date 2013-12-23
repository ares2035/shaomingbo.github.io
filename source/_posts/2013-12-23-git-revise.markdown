---
layout: post
title: "git 常用命令复习"
date: 2013-12-23 11:13
comments: true
categories: 码农工具
keywords: git,命令,版本控制,终端
description: 总结了git使用过程中常用的命令
---

最近打算对[帝傲狮](/soulsaunter)进行重构，觉得应该是时候启用版本控制工具来管理代码了。想当初为了学习这个，我还专门看了几天的[《Git版本控制管理》][1]，可是长期在公司使用clearcase，现在我连它长什么样都记不起来了-_-# 。悔恨当初做的笔记太渣，现在重新把书啃一遍那基本上又不太现实。好在这两年网上积累了好多有关的优秀资源，以下便是我这两天通过学习&使用这些网络资源而整理的**git 常用命令**的使用说明


###git clone
1.  clone 操作将会复制一个已经存在的git仓库
2.  clone 操作会自动的创建一个名叫`origin`的远程连接指向原始的仓库
3.  This makes collaborating with Git fundamentally different than with SVN

###git config
1.  `git config user.name`设置用户名
2.  `git config user.email`设置电子邮箱
3.  默认情况下是对当前仓库进行设置，加上`--global`标签之后，则为全局设置


###git add
1.  直到运行`git commit`之前，`git add`并没有直接影响仓库
2.  `git add <file>`stage 单个文件的改变
3.  `git add <directory>`stage 整个目录的改变
4.  `git add -p`交互式的add

###git commit
1.  将处于stage 状态的该表保存到仓库
2.  `git commit -a`会将工作区的所有改变保存到仓库
3.   除非你真的准备好了，否则git 不会强迫你与中心仓库记性交互
4.   就像stage 空间是工作目录与代码仓库之间的缓存一样，本地仓库是开发者的代码贡献与中心仓库之间的缓存
5.   `git commit --amend`修改之前的commit。这将导致当前的stage区间与前一个commit的状态融合。
6.   和`git reset`一样，`git commit --amend`不应该发生在发布后的`<commit>`上。
7.   `--no-edit`可以使`git commit --amend`使用上一次commit的注释


###git status
用来显示工作区的状态

###git log
1.  该命令被用来显示commit 历史
2.  `git log`将使用默认格式显示，通常会输出多于一屏的内容。使用空格翻页，使用`q`退出。
3.  `git log -n <limit>`限制输出的条数
4.  `git log -oneline`高度抽象概括提交情况
5.  `git log --stat`同时呈现相关文件、相关行的信息
6.  `git log --author=<pattern>`搜索指定作者的commit。pattern可以是包含正则表达式的字符串
7.  `git log <since>..<until>`搜索两者之间的commit。since、until既可以是commitID,也可以是branchName或者是其他的[reference](http://www.kernel.org/pub/software/scm/git/docs/gitrevisions.html)
8.  `git log <file>`查看指定文件的提交情况
9.  `git log --graph --decorate ﻿--oneline`--graph 顾名思义，用来图形化提交状态；——decorate用来显示分支或者便签的名称；

###git checkout
1.  该命令有三种功能：检出file，检出commit，检出branch
2.  `git checkout <existing-branch>`更新本地仓库与`<exsiting-branch>`匹配，之后的操作都将记录到该分支上。检出分支后，不必担心之前的最新代码会遭到污染，任何改变如果没有`commit`，只存在于stage状态
3.  `git checkout <commit>`检出commit。会使得工作目录的所有代码变成该commit的状态(`<commit>`可以为commit hash或者是tag)。检出一个旧的commit，并不会造成代码丢失，开发状态将会变为`detached HEAD`。值得注意的是，开发应当发生在真正的分支上，否则代码融合的时候会遇到不小的麻烦。
4.  `git checkout <commit> <file>`检出文件。和之前的检出不同，这种检出会影响到当前工程，使得该文件的状态变为**Change to be committed**,若希望还原回主线状态，应使用`git checkout HEAD <file>`
5.  `git checkout -b <new-branch>`创建分支，并切换到该分支去
6.  `git checkout -b <new-branch> <exsiting-branch>`与上条一样，不过创建的分支是基于<existing-branch>，而不是当前分支


###git revert
1.  作为undo 操作存在
2.  并没有删除任何commit 结点，而是将之前的结点append 到HEAD
3.  比reset 更安全

###git reset
1.  是一个可能影响仓库安全的命令
2.  和`git checkout`一样，根据目的不同，有多种用法
3.  `git reset <file>`清除该文件的stage状态，保留工作区间的文件状态
4.  `git reset`将stage恢复到最近一次提交，保留工作区间的文件状态
5.  `git reset --hard`将stage和工作区间的文件都恢复到最近一次提交
6.  `git reset <commit>`将stage恢复到<commit>的状态，保留工作区间的状态，以便re-commit的时候有一个更干净的stage
7.  `git reset --hard <commit>`将stage和工作区间的文件恢复到<commit>状态
8.  需要注意的是：如果<commit>之后的仓库被发布过，那么你不应该使用`git reset --hard <commit>`，因为有可能其他开发者对这个`<commit>`及之后的代码有依赖


###git clean
1.  该命令通常和`git reset --hard`一起使用
2.  主要目的是用来删除工作区间里没有被追踪的文件
3.  `git clean -n`并不会真正执行清除，而是告诉你将会清除哪里文件
4.  `git clean -f`将会清理当前文件夹下的未被跟踪的文件，但并不会清理文件夹以及`.gitignore`中提及的文件
5.  `git clean -df`清理当前文件夹下的未被跟踪的文件和目录
6.  `git clean -xf`清理当前文件夹下的未被跟踪的文件以及那些git 通常忽略的文件

###git branch
1.  每一个branch都可以看做是独立的开发流水线，是上述所有操作组合形成的抽象引用
2.  `git branch`支持创建，罗列，删除以及重命名操作，通常需要配合`git checkout`进行分支切换，配合`git merge`进行代码融合
3.  分支不仅可以使开发任务并行，还可以保证主线代码永远处于可用状态
4.  `git branch`罗列当前仓库中所有的分支，带上`-r`可以罗列远程分支
5.  `git branch <branch-name>`创建分支，但并没有切换过去
6.  `git branch -d <branch-name>`如果代码已经merge回主线，则安全的删除该分支，否则给予警告，且不执行删除操作
7.  `git branch -D <branch-name>`强行删除分支
8.  `git branch -m <branch-name>`修改当前分支的名字

###git merge
1.  通常来说merge 有两种算法来执行：`fast-foward merge`以及`3-way merge`
2.  当前分支与目标分支之间是**linear path**，则执行`fast-forwad merge`，如图1所示，git 只需要将当前分支的HEAD移过来即可（不走回头路）
3.  如果当前分支与目标分支之间有分叉,如图2所示，则执行`3-way merge`。
4.  git 使用`edit/stage/commit`的工作流程来解决merge 冲突：使用edit来解决冲突，使用add 来闭合修改，使用commit 来完成merge
  
###git rebase
1.  `git rebase <base>`将当前分支rebase为`<base>`，其中`<base>`可以为commit、tag或者是分支名称
2.  使用rebase主要目的是保持项目仓库的linear状态，以便分支融合的时候可以使用`fast-forward merge`
3.  和`git reset`一样，不要对以发布的仓库部分执行rebase
4.  `git rebase -i`以交互的形式进行rebase


###git remote
1.  该命令可以让你创建、查看、删除与其他仓库的连接，这些连接更像是一个个书签
2.  `git remote`呈现你的所有连接
3.  `git remote -v`和上一条命令一样，但是要多显示URL
4.  `git remote add <name> <url>`创建一个连接，在此之后，可以直接使用`<name>`来访问这个连接了
5.  `git remote rm <name>`删除这个连接
6.  `git remote rename <old-name> <new-name>`修改连接名称

###git fetch
1.  `git fetch <remote>`将该连接的所有分支全部导入本地
2.  `git fetch <remote> <branch>`仅导入该连接的特定分支
3.  我们可以checkout 远程分支，但此刻会处于一个HEAD detached状态；可以想象这些远程分支为只读的，我们能做的是review他们的代码以及**merge**到我们的本地分支上来

###git pull
1.  `git pull <remote>`将该连接的当前分支导入本地，并进行融合操作
2.  `git pull --rebase <remote>`和上述操作效果一样，但使用rebase而不是融合
3.  `git pull`与SVN的`update`类似，作用都是保持本地代码最新
4.  `--rebase`可以避免不要的merge操作
5.  使用`git config --global branch.autosetuprebase always`将pull的默认动作设置为rebase

###git push
1.  `git push <remote> <branch>`将<branch>推送到远程仓库去。如果本地仓库和远程仓库之间的代码不能执行`fast-forward merge`，则git 会拒绝该次推送
2.  `git push <remote> --force`和上一条效果一样，只不过会强制执行
3.  `git push <remote> --tags`将本地标签全部推送到远程仓库去

##参考资料
-   [Git Tutorials](https://www.atlassian.com/git/tutorial/)
-   [Learn git branching](http://pcottle.github.io/learnGitBranching/)

##附图
-   图1 a fast-forward merge

![fast-forward merge 示意图](/images/20131223/git-revise-merge-fast-forward.png)

-	图2 a 3-way merge

![3-way merge 示意图](/images/20131223/git-revise-merge-three-way.png)



[1]: http://book.douban.com/subject/5311565/