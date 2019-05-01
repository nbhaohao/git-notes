# git-notes
记录 git 的一些常用操作

**资料主要来源于这本书 [为你自己学 Git](<https://kaochenlong.com/2017/11/28/new-git-book/>)**

**图形化工具使用 SourceTree**

**工作区、暂存区、存储库分别代表：WorkingDirectory、Staging Area、Repository**

## 目录：
  * [.git 里面有什么](<https://github.com/nbhaohao/git-notes/issues/1>)

  * [设置用户身份](#设置用户身份)

  * [追加改动到最后一次的 commit](#追加改动到最后一次的-commit)

  * [忽略某些文件](#忽略某些文件)

  * [检查某个文件的 commit 记录](#检查某个文件的-commit-记录)

  * [检查文件每行代码的作者](#检查文件每行代码的作者)

  * [丢弃还没提交到暂存区的改动](#丢弃还没提交到暂存区的改动)

  * [提交完 commit 后后悔了，想要重新来过](#提交完-commit-后后悔了想要重新来过)

  * [reset 后悔了，想要回到原来的 commit](#reset-后悔了想要回到原来的-commit)

  * [HEAD 是什么](#head-是什么)

  * [Fast Forward](#fast-forward)

  * [不小心删除分支后还可以找回来吗](#不小心删除分支后还可以找回来吗)

  * [rebase 合并分支](#rebase-合并分支)

  * [修改历史信息](#修改历史信息)

  * [使用标签](#使用标签)

  * [设置远程服务器](#设置远程服务器)

## 设置用户身份

设置全局信息：

```bash
git config --global user.name 'zhangzihao'
git congig --global user.email 'nbhaohao@163.com'
```

查看当前设定：

```bash
git config --list
```

为当前的 git 仓库设置信息，因为有可能公司一个信息，然后自己的项目一个信息：

```bash
git config --local user.name 'zhangzihao'
git congig --local user.email 'nbhaohao@163.com'
```

## 追加改动到最后一次的 commit

当我们完成一次 commit 后，突然想起来有些改动忘记提交了，我们就可以用到 `git commit —amend`, 它会将这次 commit 的内容合并到最后一次的 commit 中，并重新生成一个 commit id.

```bash
git log --oneline
5b7bcfc (HEAD -> master) normal commit
4267830 init commit
git commit --amend -m 'use amend'

git log --oneline
ec425d6 (HEAD -> master) use amend
4267830 init commit
```

## 忽略某些文件

在 git 仓库下新建一个 `.gitignore` 文件，在里面写入想要忽略的文件名即可。

```bash
# 忽略 dist 文件夹
dist 
# 忽略 apiSecret 文件
apiSecret.json
```

但是这里会有一个问题，如果你的文件在 `.gitignore` 创建之前就已经提交了，那之后的改动还是不会被忽略。

可以使用 `git rm fileNamed --cached` 把这个文件移出 git 追踪 (这里的 rm 不是真正意义上的删除，只是将 git 中关于这个文件的记录删除掉而已)。

如果喜欢暴力一点的，可以一下次删除掉所有这种文件的，可以使用 `git clean -fx`

## 检查某个文件的 commit 记录

SourceTree 右键点击文件 -> Log Selected

## 检查文件每行代码的作者

SourceTree 右键点击文件 -> Annotate Selected

## 丢弃还没提交到暂存区的改动

我们使用 `git checkout` 这个命令，如果指定的参数是分支的名字的话，会切换到对应分支，如果是文件名字的话，git 会从暂存区拿这个文件的内容来覆盖当前工作目录的文件内容。

`git checkout test.txt`

我们还可以指定一个 commit 版本号，表示用指定的版本的文件内容覆盖当前的文件内容，同时更新暂存区

`git checkout ec425d6 test.txt`

我们也可以使用 `git checkout ec425d6 .` 这个命令，将当前工作区的文件内容都改为指定的版本，同时更新暂存区

## 提交完 commit 后后悔了，想要重新来过

我们使用 `git reset` 这个命令，在后面直接加想要回退到指定的版本号：`git reset 7ca9348`

不过要注意：reset 有 3 种模式：

- --mixed: 默认值，会丢弃掉暂存区的改动，但不会影响到工作区的文件 (即保留当前的改动，就算已经 git add 了，也只是取消暂存的状态，并不会影响到本地的文件)，所以从 commit 里拆出来的文件会留在工作区，不会存在暂存区。
- —soft: 工作区和暂存区的文件都不会丢掉，所以 commit 里拆出来的文件会放在暂存区。
- —hard: 工作区和暂存区的文件都会丢掉。 

什么是 **commit 里拆出来的文件**？因为你使用 reset 命令回到之前的版本，那么你回到的版本的文件和你当前工作区的文件肯定会有差异。模式决定了如何处理这些改动。

```bash
# 假设我们有一个 test.txt 文件
# 创建的时候内容是空
# 第一次修改的时候内容是：'第一次修改'
# 第二次修改的时候内容是：`第一次修改\n第二次修改`
git log --oneline
a92bee4 (HEAD -> master) 第二次修改
5410577 第一次修改
58eba40 创建 test.txt
```

如果我们使用 `git reset 5410577 `, 默认使用了 mixed 模式，那么我们对于 `test.txt` 的修改将会出现在工作区中：

## reset 后悔了，想要回到原来的 commit

我们可以使用 `git reflog` 这个命令来看 git HEAD(指针) 变化的记录。然后再通过 reset 命令回到我们较新的 commit 即可。

## HEAD 是什么

HEAD 是一个指针，指向某一个分支，可以把 HEAD 当做当前分支来看待，可以查看 `.git/HEAD` 

```bas
.git git:(master) cat HEAD
ref: refs/heads/master
```

如果我们再看下 `refs/heads/master` 的内容：

```bash
a92bee4ec0a9ec1dcda66f189dc3f7387fbd5729
```

可以发现，`master` 这个文件的内容只是记录了一个 commit 的 id.

## Fast Forward

Fast Forward 的意思是：假设有 A, B 两个分支，B 是基于 A 的，领先 A 2 次 commit, 当在 A 分支执行 `git merge B`时，Git 发现，B 分支只是比 A 多了 2 次 commit, 其他都是一样的，所以会使用 Fast Forward 合并分支，即不会产生额外的 commit, 在 history 上面也是一条直线。

反之，如果现在有 B 和 C 两个分支，它们是基于 A 的，然后分别做了 2 次 commit, 现在在 B 分支上执行 `git merge C`, 这样就无法使用 Fast Forward 来合并了，所以会产生一个新的 commit, 同时，这个 commit 的 parents 有 2 个，分别是 B 和 C 的最后一次 commit.

## 不小心删除分支后还可以找回来吗

因为分支只是指向 commit 的一个标签，也就是说删除分支只是删除了标签，并不是删除了 commit, 所以你可以通过 SHA-1 值来创建基于那个提交的分支：`git branch new_branch b174a5a` 类似这样的命令即可。如果你忘记了 SHA-1, 可以通过 `git reflog` 来查找。

## rebase 合并分支

rebase 也可以起到合并分支的作用，不过它不会产生新的 commit, 假设我们有 A, B 两个分支，我们在 A 分支使用 `git rebase B`, 意思是：A 分支想要重新定义它的 commit 的 parent, 并且以 B 作为 commit 新的 parent. 所以如果使用 rebase, 提交历史将会是一条直线，就好像 A 分支就是基于 B 分支一样，在 B 分支的基础上多了几次 commit.

## 修改历史信息

之前我们提到，可以使用 `git commit --amend` 来修改最后一次 commit 信息，那如果要修改最后第二次的，或者其他的就要使用别的方法了，我们可以使用 `git rebase -i bb0c9c2`, `-i` 表示互动模式，`bb0c9c2` 表示指定这次 rebase 的应用范围从现在到 bb0c9c2 这个 commit.

使用这种互动模式，要注意的是：**commit 的排列顺序是从旧到新**。而且还可以合并多个 commit.

## 使用标签

轻量标签：`git tag tagName 51d54ff`

附注标签：`git tag tagName 51d54ff -a -m 'tag description'`, `-a` 表示要建立一个附注标签。

## 设置远程服务器

`git remote add orgin git@github.com:nbhaohao/git-notes.git`

其中 `git remote` 表示和远程有关的操作，`add` 表示加入一个远程节点，`origin` 是一个代名词。
