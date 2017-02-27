# Git学习笔记

## Git初始化

查看git版本。

```bash
# > git --version
# git version 2.7.4 (Apple Git-66)
git --version
```

切换到`${WORKSPACE_HOME}`目录。

```bash
cd /Users/axu/code/axuProject
```

创建学习git目录`${LEARN_GIT_HOME}`。

```bash
mkdir learn_git
```

进入目录。

```bash
cd learn_git
```

初始化git（执行后会在 `${LEARN_GIT_HOME}` 目录中创建.git文件）。

```bash
# > git init 
# Initialized empty Git repository in /Users/axu/code/axuProject/learn_git/.git/
git init 
```

查看生成的`${LEARN_GIT_HOME}/.git`文件。

```bash
# > pwd
# /Users/axu/code/axuProject/learn_git
pwd

# > ls -al
# total 0
# drwxr-xr-x   3 axu  staff  102  9 13 10:41 .
# drwxr-xr-x  13 axu  staff  442  9 13 10:41 ..
# drwxr-xr-x  10 axu  staff  340  9 13 10:41 .git 
ls -al
```

创建一个测试文件。

```bash
echo "Hello." > welcome.txt
```

添加测试文件到`缓存区`中。

```bash
git add welcome.txt
```

提交`welcome.txt`文件到`版本库`中。

```bash
# > git ci -m "initialized."
# [master (root-commit) 1d20a87] initialized.
# 1 file changed, 1 insertion(+)
# create mode 100644 welcome.txt
git ci -m "initialized."
```

切换到`${LEARN_GIT_HOME}`目录，并创建`${LEARN_GIT_HOME}/a/b/c`文件夹。

```bash
cd /Users/axu/code/axuProject/learn_git
mkdir -p a/b/c
cd /Users/axu/code/axuProject/learn_git/a/b/c
```

通过命令显示`.git`目录所在位置。

```bash
# > git rev-parse --git-dir
# /Users/axu/code/axuProject/learn_git/.git
git rev-parse --git-dir
```

通过命令显示`${LEARN_GIT_HOME}`目录。

```bash
# > git rev-parse --show-toplevel
# /Users/axu/code/axuProject/learn_git
git rev-parse --show-toplevel
```

通过命令显示当前目录针对`${LEARN_GIT_HOME}`目录的相对路径。

```bash
# > git rev-parse --show-prefix
# a/b/c/
git rev-parse --show-prefix
```

通过命令显示从当前目录如何退回`${LEARN_GIT_HOME}`目录。

```bash
# > git rev-parse --show-cdup
# ../../../
git rev-parse --show-cdup
```

设置git别名，用户，颜色等（只针对当前用户）。

```bash
# > cat ~/.gitconfig 
# [user]
#     name = anxu
#     email = axu.home@gmail.com
# [color]
#     ui = auto
# [filter "lfs"]
#     clean = git-lfs clean %f
#     smudge = git-lfs smudge %f
#     required = true
# [alias]
#     st = status
#     ci = commit
#     co = checkout
#     br = branch
#     pl = pull
#     ph = push
cat ~/.gitconfig 
```

为了测试上文中针对git设置的提交用户。

```bash
# > git ci --allow-empty -m "who does commit?"
# [master 39bf0ad] who does commit?
git ci --allow-empty -m "who does commit?"
```

查看提交日志（到目前为止，一共提交了两次）。

```bash
# > git log --pretty=fuller
# commit 39bf0ad897e23a03879ce612c95ba62b033f5f47
# Author:     anxu <axu.home@gmail.com>
# AuthorDate: Tue Sep 13 11:06:18 2016 +0800
# Commit:     anxu <axu.home@gmail.com>
# CommitDate: Tue Sep 13 11:06:18 2016 +0800
# 
#     who does commit?
# 
# commit 1d20a87c0708cfa73052c1fa081859f2c64c99ba
# Author:     anxu <axu.home@gmail.com>
# AuthorDate: Tue Sep 13 10:50:55 2016 +0800
# Commit:     anxu <axu.home@gmail.com>
# CommitDate: Tue Sep 13 10:50:55 2016 +0800
# 
#     initialized.
git log --pretty=fuller
```

备份项目。

```bash
cd /Users/axu/code/axuProject

# > git clone learn_git learn_git-step-1
# Cloning into 'learn_git-step-1'...
git clone learn_git learn_git-step-1

# > ls -l
# [...]
# drwxr-xr-x   5 axu  staff       170  9 13 10:56 learn_git
# drwxr-xr-x   4 axu  staff       136  9 13 11:10 learn_git-step-1
# [...]
ls -l
```

`-EOF-`

***

## Git暂存区

切换到`${LEARN_GIT_HOME}`目录，并通过命令查看git日志。

> 可以从日志输入中看到，在`initialized`有一个文件提交。

```bash
cd /Users/axu/code/axuProject/learn_git

# > git log --stat
# commit 39bf0ad897e23a03879ce612c95ba62b033f5f47
# Author: anxu <axu.home@gmail.com>
# Date:   Tue Sep 13 11:06:18 2016 +0800
# 
#     who does commit?
# 
# commit 1d20a87c0708cfa73052c1fa081859f2c64c99ba
# Author: anxu <axu.home@gmail.com>
# Date:   Tue Sep 13 10:50:55 2016 +0800
# 
#     initialized.
# 
#  welcome.txt | 1 +
#  1 file changed, 1 insertion(+)
git log --stat
```

修改`welcome.txt`文件内容。

```bash
echo "Nice to meet you." >> welcome.txt 
```

通过命令对比`welcome.txt`文件内容变化。

```bash
# > git diff
# diff --git a/welcome.txt b/welcome.txt
# index 18832d3..fd3c069 100644
# --- a/welcome.txt
# +++ b/welcome.txt
# @@ -1 +1,2 @@
#  Hello.
# +Nice to meet you.
git diff
```

直接提交。

```bash
# > git ci -m "Append a nice line."
# On branch master
# Changes not staged for commit:
#     modified:   welcome.txt
# 
# no changes added to commit
git ci -m "Append a nice line."
```

通过命令查看提交日志（并没有之前的提交）。

```bash
# > git log --pretty=oneline
# 39bf0ad897e23a03879ce612c95ba62b033f5f47 who does commit?
# 1d20a87c0708cfa73052c1fa081859f2c64c99ba initialized.
git log --pretty=oneline
```

通过命令查看当前文件状态。

```bash
# > git st -s 
#  M welcome.txt
git st -s 
```

> 因为git需要先将修改文件添加到`缓存区`然后才可以提交到`版本库`中。
> 所以需要先执行`add`命令。

```bash
git add welcome.txt
```

通过命令对比文件内容区别，发现没有任何输出。
因为`git diff`是对比`工作区`的文件差异，刚刚已经将文件`add`到`缓存区`了，所以没有发现差异。

> 个人理解`git diff`命令是对比`工作区`和`缓存区`的文件差异（因为刚才将差异`add`到`缓存区`，所以现在`工作区`和`缓存区`没有差异，所以没有显示）。
> 
> `#!# 具体需要验证`

```bash
git diff
```

> 个人理解是对比`工作区`和`版本库`的差异。
> 
> `#!# 具体需要验证`

```bash
# > git diff HEAD
# diff --git a/welcome.txt b/welcome.txt
# index 18832d3..fd3c069 100644
# --- a/welcome.txt
# +++ b/welcome.txt
# @@ -1 +1,2 @@
#  Hello.
# +Nice to meet you.
git diff HEAD
```

查看状态。

```bash
# > git st
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
# 
#     modified:   welcome.txt
git st

# > git st -s
# M  welcome.txt
git st -s
```

再次修改`工作区`中`welcome.txt`文件内容。

```bash
echo "Bye-Bye." >> welcome.txt 
```

查看状态。

> 从状态输出结果可以看出现在`工作区`和`缓存区`都有内容变更。

```bash
# > git st
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
# 
#     modified:   welcome.txt
# 
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
# 
#     modified:   welcome.txt
git st

# > git st -s
# MM welcome.txt
git st -s
```

再次调用`git diff`命令对比`工作区`和`缓存区`的内容。

```bash
# > git diff
# diff --git a/welcome.txt b/welcome.txt
# index fd3c069..51dbfd2 100644
# --- a/welcome.txt
# +++ b/welcome.txt
# @@ -1,2 +1,3 @@
#  Hello.
#  Nice to meet you.
# +Bye-Bye.
git diff
```

再次调用`git diff HEAD`命令对比`工作区`和`版本库`的内容。

```bash
# > git diff HEAD
# diff --git a/welcome.txt b/welcome.txt
# index 18832d3..51dbfd2 100644
# --- a/welcome.txt
# +++ b/welcome.txt
# @@ -1 +1,3 @@
#  Hello.
# +Nice to meet you.
# +Bye-Bye.
git diff HEAD
```

调用`git diff --cached`可以对比`缓存区`和`版本库`的差异。

```bash
# > git diff --cached
# diff --git a/welcome.txt b/welcome.txt
# index 18832d3..fd3c069 100644
# --- a/welcome.txt
# +++ b/welcome.txt
# @@ -1 +1,2 @@
#  Hello.
# +Nice to meet you.
git diff --cached
```

使用`git ci`命令提交`缓存区`中的内容。

```bash
# > git ci -m "which version checked in?"
# [master 4cf2968] which version checked in?
#  1 file changed, 1 insertion(+)
git ci -m "which version checked in?"
```

查看提交日志。

```bash
# > git log --pretty=oneline
# 4cf2968749c0b86e820b51c963ff80cc54d2519c which version checked in?
# 39bf0ad897e23a03879ce612c95ba62b033f5f47 who does commit?
# 1d20a87c0708cfa73052c1fa081859f2c64c99ba initialized.
git log --pretty=oneline
```

查看状态。

> **可以从结果看出**
> `缓存区`（第一列）的`M`已经没有了，证明`缓存区`的内容已经提交到`版本库`中。
> `工作区`（第二列）的`M`还存在，证明`工作区`的内容现在还处于变更状态而且没有提交到`缓存区`。

```bash
# > git st -s
#  M welcome.txt
git st -s
```

使用`git checkout`命令撤销`工作区`中的变更（修改）。

```bash
git checkout -- welcome.txt
```

执行撤销命令后，查看状态。

> 执行`git st -s`后，没有任何输出，证明之前在`工作区`针对`welcome.txt`的修改已经被撤销。

```bash
git st -s
```

查看`版本库`中目录树。

> `-l`参数可以显示文件大小（单位：字节）。

```bash
# > git ls-tree -l HEAD
# 100644 blob fd3c069c1de4f4bc9b15940f490aeb48852f3c42      25	welcome.txt
git ls-tree -l HEAD
```

通过命令（`git clean -fd`）清除`工作区`中没有加入版本库的文件和目录。
`#!#` 个人理解是删除空的目录。

```bash
# > git clean -fd
# Removing a/
git clean -fd
```

修改和添加部分内容。

> 其中`"Bye-Bye."`内容，是在之前执行 `git checkout -- welcome.txt`取消的。
> 其中`"a/b/c"`目录，是在之前执行`git clean -fd`删除的。

```bash
cd /Users/axu/code/axuProject/learn_git
echo "Bye-Bye." >> welcome.txt
mkdir -p a/b/c
echo "Hello." > a/b/c/hello.txt
```

将新文件`a/b/c/hello.txt`添加到`缓存区`中。

```bash
git add .
```

再次修改`a/b/c/hello.txt`文件内容。

```bash
echo "Bye-Bye." >> a/b/c/hello.txt
```

查看当状态。

> **可以从结果看出**
> `缓存区`（第一列）有个一个文件（`a/b/c/hello.txt`）被添加（`A`）；有一个文件（`welcome.txt`）被修改（`M`）。
> `工作区`（第二列）有一个文件（`a/b/c/hello.txt`）被修改（`M`）

```bash
# > git st -s
# AM a/b/c/hello.txt
# M  welcome.txt
git st -s
```

通过命令查看`缓存区`的目录树。

> `git ls-files -s`命令第三列代表的不是文件大小，而是`缓存区`编号。

```bash
# > git ls-files -s
# 100644 18832d35117ef2f013c4009f5b2128dfaeff354f 0	a/b/c/hello.txt
# 100644 51dbfd25a804c30e9d8dc441740452534de8264b 0	welcome.txt
git ls-files -s
```

若想通过`git ls-tree`命令查看`缓存区`文件，则需要先执行`git write-tree`命令。

```bash
# > git write-tree
# 9431f4a3f3e1504e03659406faa9529f83cd56f8
git write-tree

# > git ls-tree -l -r -t 9431f4a
# 040000 tree 53583ee687fbb2e913d18d508aefd512465b2092       -	a
# 040000 tree 514d729095b7bc203cf336723af710d41b84867b       -	a/b
# 040000 tree deaec688e84302d4a0b98a1b78a434be1b22ca02       -	a/b/c
# 100644 blob 18832d35117ef2f013c4009f5b2128dfaeff354f       7	a/b/c/hello.txt
# 100644 blob 51dbfd25a804c30e9d8dc441740452534de8264b      34	welcome.txt
git ls-tree -l -r -t 9431f4a
```

再次通过命令查看`版本库`中目录树。

```bash
# > git ls-tree -l HEAD
# 100644 blob fd3c069c1de4f4bc9b15940f490aeb48852f3c42      25	welcome.txt
git ls-tree -l HEAD
```

通过命令查看`工作区`中目录树。

```bash
# > ls -lR
# total 8
# drwxr-xr-x  3 axu  staff  102  9 13 15:08 a
# -rw-r--r--  1 axu  staff   34  9 13 15:08 welcome.txt
# 
# ./a:
# total 0
# drwxr-xr-x  3 axu  staff  102  9 13 15:08 b
# 
# ./a/b:
# total 0
# drwxr-xr-x  3 axu  staff  102  9 13 15:08 c
# 
# ./a/b/c:
# total 8
# -rw-r--r--  1 axu  staff  16  9 13 15:09 hello.txt
ls -lR
```

对比上述`工作区`、`缓存区`、`版本库`中文件大小等到下表。

文件名|`工作区`|`缓存区`|`版本库`（`HEAD`）
---|---|---|---
`welcome.txt`|34字节|34字节|25字节
`a/b/c/hello.txt`|16字节|7字节|-

再次通过`git diff`命令查看各个库中的差异。

使用`git diff`查看`工作区`和`缓存区`的差异。

```bash
# > git diff
# diff --git a/a/b/c/hello.txt b/a/b/c/hello.txt
# index 18832d3..e8577ea 100644
# --- a/a/b/c/hello.txt
# +++ b/a/b/c/hello.txt
# @@ -1 +1,2 @@
#  Hello.
# +Bye-Bye.
git diff
```

使用`git diff --cached`查看`缓存区`和`版本库`的差异。

```bash
# > git diff --cached
# diff --git a/a/b/c/hello.txt b/a/b/c/hello.txt
# new file mode 100644
# index 0000000..18832d3
# --- /dev/null
# +++ b/a/b/c/hello.txt
# @@ -0,0 +1 @@
# +Hello.
# diff --git a/welcome.txt b/welcome.txt
# index fd3c069..51dbfd2 100644
# --- a/welcome.txt
# +++ b/welcome.txt
# @@ -1,2 +1,3 @@
#  Hello.
#  Nice to meet you.
# +Bye-Bye.
git diff --cached
```

使用`git diff HEAD`查看`工作区`和`版本库`的差异。

```bash
# > git diff HEAD
# diff --git a/a/b/c/hello.txt b/a/b/c/hello.txt
# new file mode 100644
# index 0000000..e8577ea
# --- /dev/null
# +++ b/a/b/c/hello.txt
# @@ -0,0 +1,2 @@
# +Hello.
# +Bye-Bye.
# diff --git a/welcome.txt b/welcome.txt
# index fd3c069..51dbfd2 100644
# --- a/welcome.txt
# +++ b/welcome.txt
# @@ -1,2 +1,3 @@
#  Hello.
#  Nice to meet you.
# +Bye-Bye.
git diff HEAD
```

查看当前状态。

```bash
# > git st
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
# 
#     new file:   a/b/c/hello.txt
#     modified:   welcome.txt
# 
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
# 
#     modified:   a/b/c/hello.txt
git st
```

通过命令`git stash`将现有状态缓存。

```bash
# > git stash
# Saved working directory and index state WIP on master: 4cf2968 which version checked in?
# HEAD is now at 4cf2968 which version checked in?
git stash
```

再次查看当前状态。

> 可以出结果得出，现在没有任何文件被修改或者提交。

```bash
# > git st
# On branch master
# nothing to commit, working directory clean
git st
```

`-EOF-`

***

## Git对象库

切换到`${LEARN_GIT_HOME}`目录，并通过命令查看git日志。

> `-1`（数字`1`，不是字母`l`）参数是只显示最后一次提交的信息。

类型|ID（SHA1）|说明
---|---|---
commit|4cf2968749c0b86e820b51c963ff80cc54d2519c|本次提交的唯一标识（ID）
tree|f58da9a820e3fd9d84ab2ca2f1b467ac265038f9|本次提交对应的目录树ID
parent|39bf0ad897e23a03879ce612c95ba62b033f5f47|父（上一次）提交的唯一标识（ID）

```bash
# > git log -1 --pretty=raw
# commit 4cf2968749c0b86e820b51c963ff80cc54d2519c
# tree f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
# parent 39bf0ad897e23a03879ce612c95ba62b033f5f47
# author anxu <axu.home@gmail.com> 1473747936 +0800
# committer anxu <axu.home@gmail.com> 1473747936 +0800
# 
#     which version checked in?
git log -1 --pretty=raw
```

可以通过`git cat-file -t`命令查看`ID`对应的类型。

```bash 
# > git cat-file -t 4cf2968749c0b86e820b51c963ff80cc54d2519c
# commit
git cat-file -t 4cf2968749c0b86e820b51c963ff80cc54d2519c

# > git cat-file -t f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
# tree
git cat-file -t f58da9a820e3fd9d84ab2ca2f1b467ac265038f9

# > git cat-file -t 39bf0ad897e23a03879ce612c95ba62b033f5f47
# commit
git cat-file -t 39bf0ad897e23a03879ce612c95ba62b033f5f47
```

可以通过`git cat-file -p`命令配合`ID`查看，每个`ID`对应的内容。

查看`commit`类型的`ID`的内容。

```bash
# > git cat-file -p 4cf2968749c0b86e820b51c963ff80cc54d2519c
# tree f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
# parent 39bf0ad897e23a03879ce612c95ba62b033f5f47
# author anxu <axu.home@gmail.com> 1473747936 +0800
# committer anxu <axu.home@gmail.com> 1473747936 +0800
# 
# which version checked in?
git cat-file -p 4cf2968749c0b86e820b51c963ff80cc54d2519c
```

查看`tree`类型的`ID`的内容。

```bash
# > git cat-file -p f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
# 100644 blob fd3c069c1de4f4bc9b15940f490aeb48852f3c42	welcome.txt
git cat-file -p f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
```

查看`blob`类型的`ID`的内容。

```bash
# > git cat-file -p fd3c069c1de4f4bc9b15940f490aeb48852f3c42
# Hello.
# Nice to meet you.
git cat-file -p fd3c069c1de4f4bc9b15940f490aeb48852f3c42
```

> git对象的保存方式：`${LEARN_GIT_HOME}/.git/objects/${ID的前2位}/${ID后38位}`
> 
> 以`commit`类型的`ID`（`4cf2968749c0b86e820b51c963ff80cc54d2519c`）为例。
> 那么它的存储目录应为：`.git/objects/4c/f2968749c0b86e820b51c963ff80cc54d2519c`

检查git对象保存方式。

```bash
# > ls -l .git/objects/4c/f2968749c0b86e820b51c963ff80cc54d2519c
# -r--r--r--  1 axu  staff  166  9 13 14:25 .git/objects/4c/f2968749c0b86e820b51c963ff80cc54d2519c
ls -l .git/objects/4c/f2968749c0b86e820b51c963ff80cc54d2519c
```

因为`commit`类型对象有`parent`属性，所以就有了针对`commit`类型的提交`跟踪链`。
使用`--graph`参数可以查看某个`commit`类型的提交的`跟踪链`。

```bash
# > git log --pretty=raw --graph 4cf2968749c0b86e820b51c963ff80cc54d2519c
# * commit 4cf2968749c0b86e820b51c963ff80cc54d2519c
# | tree f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
# | parent 39bf0ad897e23a03879ce612c95ba62b033f5f47
# | author anxu <axu.home@gmail.com> 1473747936 +0800
# | committer anxu <axu.home@gmail.com> 1473747936 +0800
# | 
# |     which version checked in?
# |  
# * commit 39bf0ad897e23a03879ce612c95ba62b033f5f47
# | tree 190d840dd3d8fa319bdec6b8112b0957be7ee769
# | parent 1d20a87c0708cfa73052c1fa081859f2c64c99ba
# | author anxu <axu.home@gmail.com> 1473735978 +0800
# | committer anxu <axu.home@gmail.com> 1473735978 +0800
# | 
# |     who does commit?
# |  
# * commit 1d20a87c0708cfa73052c1fa081859f2c64c99ba
#   tree 190d840dd3d8fa319bdec6b8112b0957be7ee769
#   author anxu <axu.home@gmail.com> 1473735055 +0800
#   committer anxu <axu.home@gmail.com> 1473735055 +0800
#   
#       initialized.
git log --pretty=raw --graph 4cf2968749c0b86e820b51c963ff80cc54d2519c
```

使用`git branch`命令可以查看当前分支。

```bash
# > git branch
# * master
git branch
```

在当前工作状态下 `HEAD` = `master` = `refs/heads/master`。

```bash
# > git log -1 HEAD
# commit 4cf2968749c0b86e820b51c963ff80cc54d2519c
# Author: anxu <axu.home@gmail.com>
# Date:   Tue Sep 13 14:25:36 2016 +0800
# 
#     which version checked in?
git log -1 HEAD

# > git log -1 master
# commit 4cf2968749c0b86e820b51c963ff80cc54d2519c
# Author: anxu <axu.home@gmail.com>
# Date:   Tue Sep 13 14:25:36 2016 +0800
# 
#     which version checked in?
git log -1 master

# > git log -1 refs/heads/master
# commit 4cf2968749c0b86e820b51c963ff80cc54d2519c
# Author: anxu <axu.home@gmail.com>
# Date:   Tue Sep 13 14:25:36 2016 +0800
# 
#     which version checked in?
git log -1 refs/heads/master
```

查看`HEAD`、`master`、`refs/heads/master`相关内容。
后面使用`git cat-file -t`命令查看`ID`的类型。
使用`git cat-file -p`命令查看`ID`的内容。
 
> **结论:** 
> 1. `HEAD`、`master`、`refs/heads/master`均指向一个`提交ID`。
> 2. `.git/refs`目录保存引用的命令空间。
> 3. `.git/refs/heads`目录保存分支名称。
> 4. 可以使用`长格式`（`refs/heads/master`）表示分支，也可以直接使用`refs/heads`目录中的分支名称（`master`）表示分支。
> 5. 可以使用`git rev-parse`命令，查看某一个引用或者分支指向的`提交ID`

```bash
# > find .git -name HEAD -o -name master
# .git/HEAD
# .git/logs/HEAD
# .git/logs/refs/heads/master
# .git/refs/heads/master
find .git -name HEAD -o -name master

# > cat .git/HEAD
# ref: refs/heads/master
cat .git/HEAD

# > cat .git/refs/heads/master
# 4cf2968749c0b86e820b51c963ff80cc54d2519c
cat .git/refs/heads/master

# > git cat-file -t 4cf2968749c0b86e820b51c963ff80cc54d2519c
# commit
git cat-file -t 4cf2968749c0b86e820b51c963ff80cc54d2519c

# > git cat-file -p 4cf2968749c0b86e820b51c963ff80cc54d2519c
# tree f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
# parent 39bf0ad897e23a03879ce612c95ba62b033f5f47
# author anxu <axu.home@gmail.com> 1473747936 +0800
# committer anxu <axu.home@gmail.com> 1473747936 +0800
# 
# which version checked in?
git cat-file -p 4cf2968749c0b86e820b51c963ff80cc54d2519c

# > git rev-parse master
# 4cf2968749c0b86e820b51c963ff80cc54d2519c
git rev-parse master

# > git rev-parse refs/heads/master
# 4cf2968749c0b86e820b51c963ff80cc54d2519c
git rev-parse refs/heads/master

# > git rev-parse HEAD
# 4cf2968749c0b86e820b51c963ff80cc54d2519c
git rev-parse HEAD
```




