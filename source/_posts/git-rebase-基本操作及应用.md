title: git rebase 基本操作及应用
date: 2014-11-03 15:32:49
tags: [git, rebase]
---

下面介绍两种 rebase 的方法， 一种用于同个分支中合并两次提交， 一种用于不同分支的衍合。

###$ git rebase -i XXX

当前我的测试项目下有四个commit， 分别是：

```
test3
test2
test
ssh
first commit
```

用 `git log` 查看当前的 log 信息， 我的测试显示如下：

```
commit 9f85bd47dd6410f0a8c84481690b352e3900f4cd
Author: minary <i@f10.com>
Date:   Mon Nov 3 16:04:37 2014 +0800
    test 3
commit 36a3e92460f9a15d21361dd100fdc51e9c0bdbca
Author: minary <i@f10.com>
Date:   Mon Nov 3 15:40:11 2014 +0800
    test2
commit 375136336ebef674fec77355214e65049fdb5ad8
Author: minary <i@f10.com>
Date:   Mon Nov 3 15:39:05 2014 +0800
    test
commit 890faef7ceeafe3216438355be9e5bf1282c2e33
Author: minary <i@f10.com>
Date:   Thu Oct 16 10:56:49 2014 +0800
    ssh
commit 3b4d793b68d7bbea89e3bc290f25f299984812dc
Author: minary <i@f10.com>
Date:   Thu Oct 16 10:53:51 2014 +0800
    first commit
(END)

```

现在想把 **test 3** , **test2** 合并成一个 commit, 即：

```
new test
test
ssh
first commit
```

首先执行： (-i后面是test的SHA-1) 

```
$ git rebase -i 375136336ebef674fec77355214e65049fdb5ad8 
```

按 enter 进入编辑模式：

```
pick 36a3e92 test2
pick 9f85bd4 test 3

# Rebase 3751363..9f85bd4 onto 3751363
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out


```

上面的两行是执行的指令， 下面的注释是各种指令简单的介绍。

> p, pick = use commit
> s, squash = use commit, but meld into previous commit

所以我们改成：

```
 pick 36a3e92 test2
 squash 9f85bd4 test 3
```

也就是将 9f85bd4 这个版本的commit合并到 36a3e92 的commit中。
编辑完后保存退出。

如果这时候两个版本有冲突，修改完冲突后 `git --continue` 即可。

这时候会进入另一个编辑界面：

```
# This is a combination of 2 commits.
# The first commit's message is:
test2
# This is the 2nd commit message:
test 3
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon Nov 3 15:40:11 2014 +0800
#
# rebase in progress; onto 3751363
# You are currently editing a commit while rebasing branch 'master' on '3751363'.
#
# Changes to be committed:
#       modified:   index.js

```

将注释改成自己想改的，像我在这里注释掉了 test2 和 test 3， 加了 test 。

```
test

# This is a combination of 2 commits.
# The first commit's message is:
# test2
# This is the 2nd commit message:
# test 3
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon Nov 3 15:40:11 2014 +0800
#
# rebase in progress; onto 3751363
# You are currently editing a commit while rebasing branch 'master' on '3751363'.
#
# Changes to be committed:
#       modified:   index.js
#

```

接下来保存退出， 这时候再用	`git log` 查看当前的 commit 是否正确：

```
commit 63bcb30ac082311e1116cd8a96db5ef437bda5b1
Author: minary <i@f10.com>
Date:   Mon Nov 3 15:40:11 2014 +0800
    test
commit 375136336ebef674fec77355214e65049fdb5ad8
Author: minary <i@f10.com>
Date:   Mon Nov 3 15:39:05 2014 +0800
    test
commit 890faef7ceeafe3216438355be9e5bf1282c2e33
Author: minary <i@f10.com>
Date:   Thu Oct 16 10:56:49 2014 +0800
    ssh
commit 3b4d793b68d7bbea89e3bc290f25f299984812dc
Author: minary <i@f10.com>
Date:   Thu Oct 16 10:53:51 2014 +0800
    first commit
(END)

```

可以看出 test2 和 test 3 已经不见， 变成了 test 。

最后一步，将本地修改push到远程就好：

```
$ git push origin master 
```

可能会报下面类似的错误：

```
To ssh://gitlab@gitlab.XXX.com:65422/XXX/tears.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'ssh://gitlab@gitlab.XXX.com:65422/XXX/tears.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

```

解决方法：（强制push）

```
$ git push origin master --force 
```

### 分支的衍合

例如我现在需要将 test 分支 合并到 develop 的分支中：

```
$ git checkout test
$ git rebase develop

```

执行完上面两如果有 **冲突** ，git会停止 rebase 并让你去解决 **冲突** 。
解决完后用 `git add` 命令去更新这些内容的索引， 然后再执行：

```
$ git rebase --continue
```

在任何时候都可以用 `--abort` 来终止 rebase 的行动， 并且 test 分支会回到 rebase 开始前的状态。

```
$ git rebase --abort
```

### 总结

自从开始有了些许的强迫症，就开始喜欢用 rebase ，不过使用有风险，谨慎使用。
