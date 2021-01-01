# 参考资料
[Git-Book](https://git-scm.com/book/zh/v2)

# Git 起步
略
## 关于版本控制
## Git 简史
## Git 基础
## 命令行
## 安装 Git
## 初次运行 Git 前的配置
## 获取帮助
## 总结

# Git 基础 ！！！
## 获取仓库（repository）
### 本地初始化仓库 git init
```
$ git init
$ git add *.c
$ git add LICENSE
$ git commit -m 'initial project version'
```

### 克隆现有的仓库 git clone [url] [repository-name]
```
$ git clone https://github.com/libgit2/libgit2
```

可以自定义本地仓库的名字。
```
$ git clone https://github.com/libgit2/libgit2 mylibgit
```

## 基础操作
工作区下的每一个文件有两种状态：未跟踪（untracked）或已跟踪。

已跟踪的文件的状态：未修改(unmodified)，已修改(modified)或已暂存(staged)。

**文件示例周期：**
- 初次克隆某个仓库的时候，工作区中的所有文件都属于已跟踪文件，并处于未修改状态(unmodified)。
- 编辑某些文件之后，git将文件标记为已修改文件(modified)。
- 然后将这些文件放入暂存区，git将文件标记为已暂存文件(staged)。
- 然后提交所有暂存了的修改，git将文件重新标记为未修改文件(unmodified)。
- 跟踪新文件，git将文件标记为已暂存文件(staged)。

### 查看状态 git status
```
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
	
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md	
```
- Untracked files：未跟踪文件。Git 在之前的快照（提交）中没有这些文件，且Git不会自动将之纳入跟踪范围。
- Changes not staged for commit：已修改文件。说明已跟踪文件的内容发生了变化，但还没有放到暂存区。 要暂存这次更新，需要运行 git add 命令。
- Changes to be committed： 已暂存文件。如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中。

### 状态简览 git status -s
git status 命令的输出十分详细，但其用语有些繁琐。如果你使用 git status -s 命令或 git status --short 命令，你将得到一种更为紧凑的格式输出。 
```
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```
新添加的未跟踪文件前面有 ?? 标记。
新添加到暂存区中的文件前面有 A 标记。
修改过的文件前面有 M 标记。
M 有两个可以出现的位置，出现在右边的 M 表示该文件被修改了但是还没放入暂存区，出现在靠左边的 M 表示该文件被修改了并放入了暂存区

### 跟踪新文件（暂存已修改文件） git add
git add 命令是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。
git add 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。
```
$ git add CONTRIBUTING.md
```

### 查看未暂存的修改 git diff
git diff  查看尚未暂存的改动，而不是自上次提交以来所做的所有改动。
此命令比较的是工作区中当前文件和暂存区域快照之间的差异， 也就是修改之后还没有暂存起来的变化内容。
```
$ git diff
```

### 查看已暂存的修改 git diff --cached | git diff --staged
git diff --cached 查看已暂存的将要添加到下次提交里的内容。
```
$ git diff --cached
```

### 提交更新 git commit -m ""
每次准备提交前，先用 git status 看下，是不是都已暂存起来了， 然后再运行提交命令 git commit。
```
$ git commit
```
这种方式会启动文本编辑器以便输入本次提交的说明。

另外，你也可以在 commit 命令后添加 -m 选项，将提交信息与命令放在同一行。
```
$ git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed
 2 files changed, 2 insertions(+)
 create mode 100644 README
```

### 跳过暂存直接提交 git commit -a -m ""
Git 提供了一个跳过使用暂存区域的方式， 只要在提交的时候，给 git commit 加上 -a 选项。
```
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
 1 file changed, 5 insertions(+), 0 deletions(-)
```

### 移除文件（不在跟踪）git rm [file-name]
要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。

可以用 git rm 命令完成此项工作，并连带从工作区中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。
```
$ rm PROJECTS.md
$ git rm PROJECTS.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    PROJECTS.md
```

如果只是简单地从工作区中手工删除文件，运行 git status 时就会在 “Changes not staged for commit” 部分（也就是 未暂存清单）。
```
$ rm PROJECTS.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    PROJECTS.md

no changes added to commit (use "git add" and/or "git commit -a")
```

如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f（译注：即 force 的首字母）。
```
$ git rm -f PROJECTS.md
```

如果想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作区中，使用 --cached 选项。 
```
$ git rm --cached README
```

### 移动文件 git mv [from-file-name] [to-file-name]
```
$ git mv README.md README
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

其实，运行 git mv 就相当于运行了下面三条命令。
```
$ mv README.md README
$ git rm README.md
$ git add README
```

### 忽略文件（不在显示，不标记为未跟踪）
创建一个名为 .gitignore 的文件，列出要忽略的文件模式。 
```
*.[oa]
*~
```

文件 .gitignore 的格式规范如下：
- 所有空行或者以 ＃ 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配。
- 匹配模式可以以（/）开头防止递归。
- 匹配模式可以以（/）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

GitHub 有一个十分详细的针对数十种项目及语言的 .gitignore 文件列表，参考[https://github.com/github/gitignore](https://github.com/github/gitignore)。

## 撤消操作
### 重新提交 git commit --amend
例如，你提交后发现忘记了暂存某些需要的修改，可以像下面这样操作。
```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```
最终你只会有一个提交——第二次提交将代替第一次提交的结果。

### 撤消已修改文件 git checkout -- [file-name]
```
$ git checkout -- CONTRIBUTING.md
```

### 取消已暂存文件 git reset HEAD [file-name]
```
$ git reset HEAD CONTRIBUTING.md
```

## 历史操作
### 查看提交历史 git log

git log 有许多选项可以帮助你搜寻你所要找的提交， 接下来我们介绍些最常用的。

一个常用的选项是 -p，用来显示每次提交的内容差异。 你也可以加上 -2 来仅显示最近两次提交。
```
$ git log -p -2
```

定制要显示的记录格式。
```
git log --pretty=format:"%h %s" --graph
```
或
```
$ git log --pretty=oneline
```

## 远程仓库
远程仓库是指托管在因特网或其他网络中的你的项目的版本库。 
你可以有好几个远程仓库，通常有些仓库对你只读，有些则可以读写。 
与他人协作涉及管理远程仓库以及根据需要推送或拉取数据。
管理远程仓库包括了解如何添加远程仓库、移除无效的远程仓库、管理不同的远程分支并定义它们是否被跟踪等等。

### 查看所有远程仓库 git remote
如果想查看你已经配置的远程仓库服务器，可以运行 git remote 命令。 它会列出你指定的每一个远程服务器的简写。 
如果你已经克隆了自己的仓库，那么至少应该能看到 origin ——这是 Git 给你克隆的仓库服务器的默认名字。
```
$ git clone https://github.com/schacon/ticgit
Cloning into 'ticgit'...
remote: Reusing existing pack: 1857, done.
remote: Total 1857 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (1857/1857), 374.35 KiB | 268.00 KiB/s, done.
Resolving deltas: 100% (772/772), done.
Checking connectivity... done.
$ cd ticgit
$ git remote
origin
```

### 查看某个远程仓库 git remote show [remote-name]
如果想要查看某一个远程仓库的更多信息，可以使用 git remote show [remote-name] 命令。
```
$ git remote show origin
* remote origin
  URL: https://github.com/my-org/complex-project
  Fetch URL: https://github.com/my-org/complex-project
  Push  URL: https://github.com/my-org/complex-project
  HEAD branch: master
  Remote branches:
    master                           tracked
    dev-branch                       tracked
    markdown-strip                   tracked
    issue-43                         new (next fetch will store in remotes/origin)
    issue-45                         new (next fetch will store in remotes/origin)
    refs/remotes/origin/issue-11     stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev-branch merges with remote dev-branch
    master     merges with remote master
  Local refs configured for 'git push':
    dev-branch                     pushes to dev-branch                     (up to date)
    markdown-strip                 pushes to markdown-strip                 (up to date)
    master                         pushes to master                         (up to date)
```
这个命令列出了
当你在特定的分支上执行 git push 会自动地推送到哪一个远程分支。
当你执行 git pull 时哪些分支会自动合并。 
它也同样地列出了哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了。

可以指定选项 -v，会显示需要读写远程仓库使用的 Git 保存的**简写**与其对应的 **URL**。
```
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
```

如果你的远程仓库不止一个，该命令会将它们全部列出。
```
$ git remote -v
bakkdoor  https://github.com/bakkdoor/grit (fetch)
bakkdoor  https://github.com/bakkdoor/grit (push)
cho45     https://github.com/cho45/grit (fetch)
cho45     https://github.com/cho45/grit (push)
defunkt   https://github.com/defunkt/grit (fetch)
defunkt   https://github.com/defunkt/grit (push)
koke      git://github.com/koke/grit.git (fetch)
koke      git://github.com/koke/grit.git (push)
origin    git@github.com:mojombo/grit.git (fetch)
origin    git@github.com:mojombo/grit.git (push)
```

### 添加远程仓库 git remote add [remote-name] [url]
添加一个新的远程 Git 仓库，同时指定一个你可以轻松引用的简写。
```
$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

### 移除远程仓库 git remote rm [remote-name]
```
$ git remote rm paul
$ git remote
origin
```

### 重命名远程仓库 git remote rename [old-remote-name] new-[remote-name]
```
$ git remote rename pb paul
$ git remote
origin
paul
```

### 从远程仓库拉取 git fetch [remote-name] | git pull [remote-name] [branch-name]
git fetch 命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。
```
$ git fetch [remote-name]
```
注意 git fetch 命令会将数据拉取到你的本地仓库——它并不会自动合并或修改你当前的工作区。当准备好时你必须手动将其合并入你的工作区。
如果你有一个分支设置为跟踪一个远程分支（阅读下一节与 Git 分支 了解更多信息），可以使用 git pull 命令来自动的抓取然后合并远程分支到此分支。 
默认情况下，git clone 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 master 分支。

运行 git pull 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。

针对refusing to merge unrelated histories问题，设置允许合并不相关的内容。
```
$ git pull --allow-unrelated-histories
```

### 推送至远程仓库 git push [remote-name] [branch-name]
当你想要将 master 分支推送到 origin 服务器时，那么运行这个命令就可以将你所做的备份到服务器。
```
$ git push origin master
```
只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 
当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝，你必须先将他们的工作拉取下来并将其合并进你的工作后才能推送。

针对non-fast-forward问题，可以强行覆盖远程仓库。
```
$ git push -f
```

## Git 标签
### 列出标签 git tag
```
$ git tag
v0.1
v1.3
```

### 查看信息 git show

### 创建标签 git tag -a [tag-name] -m ["tag-desc"] [commit-id]
Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。

轻量标签很像一个不会改变的分支——它只是一个特定提交的引用。
附注标签是存储在 Git 数据库中的一个完整对象。
它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；
并且可以使用 GNU Privacy Guard （GPG）签名与验证。 通常建议创建附注标签，这样你可以拥有以上所有信息；
但是如果你只是想用一个临时的标签，或者因为某些原因不想要保存那些信息，轻量标签也是可用的。

**创建轻量标签**，不需要使用 -a、-s 或 -m 选项，只需要提供标签名字。
```
$ git tag v1.4-lw
$ git tag
v0.1
v1.3
v1.4
v1.4-lw
v1.5
```

**创建附注标签**，是当你在运行 tag 命令时指定 -a 选项。
```
$ git tag -a v1.4 -m "my version 1.4"
$ git tag
v0.1
v1.3
v1.4
```

**补充标签**，可以对过去的提交打标签，需要在命令的末尾指定提交的校验和（或部分校验和）。
```
$ git log --pretty=oneline
15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme
$ git tag -a v1.2 9fceb02
```

### 推送标签 git push origin [tag-name]
默认情况下，git push 命令并不会传送标签到远程仓库服务器上。在创建完标签后你必须显式地推送标签到共享服务器上。
```
$ git push origin v1.5
```

想要一次性推送很多标签，也可以使用带有 --tags 选项的 git push 命令。
```
$ git push origin --tags
```

### 删除标签 git tag -d [tag-name]
删除本地仓库上的标签，可以使用命令 git tag -d <tagname>。
```
$ git tag -d v1.4-lw
Deleted tag 'v1.4-lw' (was e7d5add)
```

注意上述命令并不会从任何远程仓库中移除这个标签，必须使用 git push <remote> :refs/tags/<tagname> 来更新你的远程仓库。
```
$ git push origin :refs/tags/v1.4-lw
To /git@github.com:schacon/simplegit.git
 - [deleted]         v1.4-lw
```

### 检出标签 git checkout
如果你想查看某个标签所指向的文件版本，可以使用 git checkout 命令，虽然说这会使你的仓库处于“分离头指针（detacthed HEAD）”状态。
```
$ git checkout 2.0.0
Note: checking out '2.0.0'.
```

在“分离头指针”状态下，如果你做了某些更改然后提交它们，标签不会发生变化，
但你的新提交将不属于任何分支，并且将无法访问，除非确切的提交哈希。
因此，如果你需要进行更改——比如说你正在修复旧版本的错误——这通常需要创建一个新分支。
```
$ git checkout -b version2 v2.0.0
Switched to a new branch 'version2'
```

## Git 别名
略

## 总结
略

# Git 分支
**“origin”并无特殊含义**
远程仓库名字 “origin” 与分支名字 “master” 一样，在 Git 中并没有任何特别的含义一样。
同时 “master” 是当你运行 git init 时默认的起始分支名字，原因仅仅是它的广泛使用，“origin” 是当你运行 git clone 时默认的远程仓库名字。
如果你运行  git clone -o booyah，那么你默认的远程分支名字将会是 booyah/master。

## 分支简介
由于 Git 的分支实质上仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。 
创建一个新分支就相当于往一个文件中写入 41 个字节（40 个字符和 1 个换行符），如此的简单能不快吗？

这与过去大多数版本控制系统形成了鲜明的对比，它们在创建分支时，将所有的项目文件都复制一遍，并保存到一个特定的目录。 
完成这样繁琐的过程通常需要好几秒钟，有时甚至需要好几分钟。所需时间的长短，完全取决于项目的规模。而在 Git 中，任何规模的项目都能在瞬间创建新分支。 
同时，由于每次提交都会记录父对象，所以寻找恰当的合并基础（译注：即共同祖先）也是同样的简单和高效。
这些高效的特性使得 Git 鼓励开发人员频繁地创建和使用分支。

## 分支管理 
### 查看分支 git branch | git branch -v
git branch 命令不只是可以创建与删除分支。 如果不加任何参数运行它，会得到当前所有分支的一个列表。
```
$ git branch
  iss53
* master
  testing
```
注意 master 分支前的 * 字符：它代表现在检出的那一个分支（也就是说，当前 HEAD 指针所指向的分支）。 这意味着如果在这时候提交，master 分支将会随着新的工作向前移动。

如果需要查看每一个分支的最后一次提交，可以运行 git branch -v 命令
```
$ git branch -v
  iss53   93b412c fix javascript issue
* master  7a98805 Merge branch 'iss53'
  testing 782fd34 add scott to the author list in the readmes
```

--merged 与 --no-merged 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支。

如果要查看哪些分支已经合并到当前分支，可以运行 git branch --merged
```
$ git branch --merged
  iss53
* master
```
因为之前已经合并了 iss53 分支，所以现在看到它在列表中。 

查看所有包含未合并工作的分支，可以运行 git branch --no-merged
```
$ git branch --no-merged
  testing
```

查看远程仓库分支，可以运行 git branch -a
```
$ git branch -a
```

### 新建分支 git branch [branch] | git checkout -b [branch] | git checkout -b [branch] [remote/branch]
想要新建一个分支并同时切换到那个分支上，你可以运行一个带有 -b 参数的 git checkout 命令。
```
$ git checkout -b iss53
Switched to a new branch "iss53"
```
它是下面两条命令的简写。
```
$ git branch iss53
$ git checkout iss53
```

### 删除分支 git branch -d [branch]
删除无效的 hotfix 分支。
```
git branch -d hotfix
```

### 切换分支 git checkout [branch]
```
$ git checkout master
Switched to branch 'master'
```

### 合并分支 git merge [target-branch]
将 hotfix 分支合并回你的 master 分支，需要先切换至 master 分支。
```
$ git checkout master
$ git merge hotfix
```
遇到冲突时时，在你解决了所有文件里的冲突之后，对每个文件使用 git add 命令来将其标记为冲突已解决，全部冲突解决后输入 git commit 来完成合并提交。


## 远程分支 ！！！
**远程分支**是远程仓库的分支。

**远程引用**是对远程仓库的引用（指针），包括分支、标签等等。

**远程分支引用**是远程分支的引用。它们是你不能移动的本地引用，当你做任何网络通信操作时，它们会自动移动。

远程分支引用像是你上次连接到远程仓库时，保存在本地的远程分支所处状态的信息记录。

### 推送（新建远程分支） git push [remote] [branch]
本地的分支并不会自动与远程仓库同步，你必须显式地推送想要分享的分支。首次推送至远程仓库时，会创建远程分支。
```
$ git push origin serverfix
Counting objects: 24, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (24/24), 1.91 KiB | 0 bytes/s, done.
Total 24 (delta 2), reused 0 (delta 0)
To https://github.com/schacon/simplegit
 * [new branch]      serverfix -> serverfix
```

如果并不想让远程仓库上的分支叫做 serverfix，可以运行 git push origin serverfix:awesomebranch ，
来将本地的 serverfix 分支推送到远程仓库上的 awesomebranch 分支。
```
git push origin serverfix:awesomebranch
```

之后其他协作者从服务器上抓取数据时，他们会在本地生成一个远程分支 origin/serverfix，指向服务器的 serverfix 分支的引用。
```
$ git fetch origin
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 3 (delta 0)
Unpacking objects: 100% (3/3), done.
From https://github.com/schacon/simplegit
 * [new branch]      serverfix    -> origin/serverfix
```

要特别注意的一点是当抓取到新的远程跟踪分支时，本地不会自动生成一个新的 serverfix 分支，只有一个不可以修改的 origin/serverfix 指针（远程分支引用）。

此时可以运行 git merge origin/serverfix 将此远程分支合并到当前所在的分支进行工作。 

也可以新建自己的 serverfix 分支，同时跟踪此远程分支，进行工作。
```
$ git checkout -b serverfix origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

### 拉取 git fetch | git fetch [remote] | git pull [remote] [branch]
git fetch 命令从服务器上抓取本地没有的数据，它并不会修改工作区中的内容。它只会获取数据，然后让你自己合并。
```
$ git fetch origin
$ git merge origin/serverfix
```

git pull 命令在大多数情况下它的含义是一个 git fetch 紧接着一个 git merge 命令。
如果当前分支有一个设置好的跟踪分支，git pull 都会查找当前分支所跟踪的服务器与分支，从服务器上抓取数据，然后尝试合并那个远程分支至当前分支。

git pull 的魔法经常令人困惑所以通常单独显式地使用 fetch 与 merge 命令会更好一些。

### 删除远程分支
假设你已经通过远程分支做完所有的工作了，也就是说你和你的协作者已经完成了一个特性并且将其合并到了远程仓库的 master 分支（或任何其他稳定代码分支）。 
可以运行带有 --delete 选项的 git push 命令来删除一个远程分支。
```
$ git push origin --delete serverfix
To https://github.com/schacon/simplegit
 - [deleted]         serverfix
```

## 跟踪分支 ！！！
跟踪分支是与远程分支有直接关系的本地分支。

如果在一个跟踪分支上输入 git pull，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。

从一个远程跟踪分支检出（clone）一个本地分支会自动创建所谓的“跟踪分支”（它跟踪的分支叫做“上游分支”）。

也可以设置其他的跟踪分支，或是一个在其他远程仓库上的跟踪分支。

### 新建分支设置跟踪分支：git checkout -b [branch-name] [remote-name/branch-name] 或者 git checkout --track [remote-name/branch-name]
```
$ git checkout --track origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```
```
$ git checkout -b sf origin/serverfix
Branch sf set up to track remote branch serverfix from origin.
Switched to a new branch 'sf'
```

### 已有分支修改跟踪分支：git branch -u [remote-name]/[branch-name] | git branch ---set-upstream-to [remote-name]/[branch-name]
```
$ git branch -u origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
```

### 查看所有跟踪分支信息：git branch -vv
```
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
  testing   5ea463a trying something new
```
这会将所有的本地分支列出来并且包含更多的信息，如每一个分支正在跟踪哪个远程分支，本地分支是否是领先或落后等。

**需要重点注意的一点是**这些数字的值来自于你从每个服务器上最后一次抓取的数据。
这个命令并没有连接服务器，它只会告诉你关于本地缓存的服务器数据。
如果想要统计最新的领先与落后数字，需要在运行此命令前抓取所有的远程仓库 git fetch --all 。
```
$ git fetch --all
$ git branch -vv
```

## 分支开发工作流
- 特性分支
- 长期分支

## 变基
略

## 总结
略

# 服务器上的 Git
略
## 协议
## 在服务器上搭建 Git
## 生成 SSH 公钥
## 配置服务器
## Git 守护进程
## Smart HTTP
## GitWeb
## GitLab
## 第三方托管的选择
## 总结

# 分布式 Git
## Git 工作流程
### 集中式工作流
![](image/centralized_workflow.png)
集中式系统中通常使用的是单点协作模型——集中式工作流。 
一个中心集线器，或者说仓库，可以接受代码，所有人将自己的工作与之同步。 
若干个开发者则作为节点——也就是中心仓库的消费者——并且与其进行同步。

这意味着如果两个开发者从中心仓库克隆代码下来，同时作了一些修改，那么只有第一个开发者可以顺利地把数据推送回共享服务器。 
第二个开发者在推送修改之前，必须先将第一个人的工作合并进来，这样才不会覆盖第一个人的修改。
这和 Subversion （或任何 CVCS）中的概念一样，而且这个模式也可以很好地运用到 Git 中。

### 集成管理者工作流
![](image/integration-manager.png)
Git 允许多个远程仓库存在，使得这样一种工作流成为可能：每个开发者拥有自己仓库的写权限和其他所有人仓库的读权限。 
“官方”项目的权威的仓库。 要为这个项目做贡献，你需要从该项目克隆出一个自己的公开仓库，然后将自己的修改推送上去。 
接着你可以请求官方仓库的维护者拉取更新合并到主项目。 
维护者可以将你的仓库作为远程仓库添加进来，在本地测试你的变更，将其合并入他们的分支并推送回官方仓库。 

这一流程的工作方式如下所示（见 集成管理者工作流。）：
- 项目维护者推送到主仓库。
- 贡献者克隆此仓库，做出修改。
- 贡献者将数据推送到自己的公开仓库。
- 贡献者给维护者发送邮件，请求拉取自己的更新。
- 维护者在自己本地的仓库中，将贡献者的仓库加为远程仓库并合并修改。
- 维护者将合并后的修改推送到主仓库。

这是 GitHub 和 GitLab 等集线器式（hub-based）工具最常用的工作流程。
人们可以容易地将某个项目派生成为自己的公开仓库，向这个仓库推送自己的修改，并为每个人所见。 
这么做最主要的优点之一是你可以持续地工作，而主仓库的维护者可以随时拉取你的修改。 
贡献者不必等待维护者处理完提交的更新——每一方都可以按照自己的节奏工作。

### 司令官与副官工作流
![](image/benevolent-dictator.png)
这其实是多仓库工作流程的变种。 一般拥有数百位协作开发者的超大型项目才会用到这样的工作方式，例如著名的 Linux 内核项目。 
被称为副官（lieutenant）的各个集成管理者分别负责集成项目中的特定部分。 
所有这些副官头上还有一位称为司令官（dictator）的总集成管理者负责统筹。 
司令官维护的仓库作为参考仓库，为所有协作者提供他们需要拉取的项目代码。 

整个流程看起来是这样的（见 司令官与副官工作流。 ）：
- 普通开发者在自己的特性分支上工作，并根据 master 分支进行变基。 这里是司令官的 master 分支。
- 副官将普通开发者的特性分支合并到自己的 master 分支中。
- 司令官将所有副官的 master 分支并入自己的 master 分支中。
- 司令官将集成后的 master 分支推送到参考仓库中，以便所有其他开发者以此为基础进行变基。

## 向一个项目贡献
略

## 维护项目
略

## 总结
略

# GitHub
6.1 账户的创建和配置
6.2 对项目做出贡献
6.3 维护项目
6.4 管理组织
6.5 脚本 GitHub
6.6 总结

# Git 工具
## 选择修订版本
## 交互式暂存
## 储藏与清理
## 签署工作
## 搜索
## 重写历史
## 重置揭密 ！！！
先讨论一下 reset 与 checkout。在你初次遇到的 Git 命令中，这两个是最让人困惑的。

### 三棵树
理解 reset 和 checkout 的最简方法，就是以 Git 的思维框架（将其作为内容管理器）来管理三棵不同的树。 
“树” 在我们这里的实际意思是 “文件的集合”，而不是指特定的数据结构。

Git 作为一个系统，是以它的一般操作来管理并操纵这三棵树的：
- HEAD：上一次提交的快照，下一次提交的父结点
- Index：预期的下一次提交的快照
- Working Directory：沙盒

### 头指针 HEAD
HEAD 是当前分支引用的指针，它总是指向该分支上的最后一次提交。 
这表示 HEAD 将是下一次提交的父结点。 通常，理解 HEAD 的最简方式，就是将它看做 你的上一次提交 的快照。

其实，查看快照的样子很容易。 下例就显示了 HEAD 快照实际的目录列表，以及其中每个文件的 SHA-1 校验和：
```
$ git cat-file -p HEAD
tree cfda3bf379e4f8dba8717dee55aab78aef7f4daf
author Scott Chacon  1301511835 -0700
committer Scott Chacon  1301511835 -0700

initial commit

$ git ls-tree -r HEAD
100644 blob a906cb2a4a904a152...   README
100644 blob 8f94139338f9404f2...   Rakefile
040000 tree 99f1a6d12cb4b6f19...   lib
```

### 暂存区 Index
索引是你的 预期的下一次提交。 我们也会将这个概念引用为 Git 的“暂存区域”，这就是当你运行 git commit 时 Git 看起来的样子。

Git 将上一次检出到工作区中的所有文件填充到索引区，它们看起来就像最初被检出时的样子。 
之后你会将其中一些文件替换为新版本，接着通过 git commit 将它们转换为树来用作新的提交。
```
$ git ls-files -s
100644 a906cb2a4a904a152e80877d4088654daad0c859 0	README
100644 8f94139338f9404f26296befa88755fc2598c289 0	Rakefile
100644 47c6340d6459e05787f644c2447d2595f5d3a54b 0	lib/simplegit.rb
```

### 工作区 Working Directory
Working Directory即工作区。 另外两棵树以一种高效但并不直观的方式，将它们的内容存储在 .git 文件夹中。 
工作区会将它们解包为实际的文件以便编辑。你可以把工作区当做沙盒。
在你将修改提交到暂存区并记录到历史之前，可以随意更改。
```
$ tree
.
├── README
├── Rakefile
└── lib
    └── simplegit.rb

1 directory, 3 files
```

**Tips：当检出一个分支时，git会修改 HEAD 指向新的分支引用，将 Index 填充为该次提交的快照，然后将 Working Directory 的内容复制到 工作区 中。**

### 重置操作 git reset [commit-id]
```
$ git reset 9e5e6a4
```

reset实际上做了三个操作。

**第 1 步：移动 HEAD**
reset 做的第一件事 是移动 HEAD 的指向。这与改变 HEAD 自身不同（checkout 所做的）；reset 移动 HEAD 指向的分支。
这意味着如果 HEAD 设置为 master 分支（例如，你正在 master 分支上），运行 git reset 9e5e6a4 将会使 master 指向 9e5e6a4。
它本质上是撤销了上一次 git commit 命令，就是把该分支改了，而不会改变索引和工作区。
无论你调用了何种形式的带有一个提交的 reset，它首先都会尝试这样做。 使用 reset --soft，它将仅仅停在那儿。
```
$ git reset --soft HEAD~
```

**第 2 步：更新 Index（reset默认执行到第2步）**
reset 做的第二件事是 用HEAD 指向的当前快照的内容来更新Index。
这也是默认行为，所以如果没有指定任何选项（在本例中只是 git reset HEAD~），这就是命令将会停止的地方。
```
$ git reset --mixed HEAD~
```
或
```
$ git reset HEAD~
```

**第 3 步：更新 Working Directory**
reset 做的的第三件事情是 让工作区看起来像暂存区。
如果使用 --hard 选项，它将会继续这一步。
那么可以运行 git reset --soft HEAD~2 来将 HEAD 分支移动到一个旧一点的提交上（即你想要保留的第一个提交）。
然后只需再次运行 git commit。

```
$ git reset --hard HEAD~
```

**回顾**
reset 命令会以特定的顺序重写这三棵树，在你指定以下选项时停止：
- 移动 HEAD 分支的指向 （若指定了 --soft，则到此停止）
- 使 暂存区 看起来像 HEAD （若未指定 --hard，则到此停止）
- 使 工作区 看起来像 暂存区

### 通过路径重置 git reset [commit-id] file.txt
前面讲述了 reset 基本形式的行为，不过你还可以给它提供一个作用路径。 
若指定了一个路径，reset 将会跳过第 1 步，并且将它的作用范围限定为指定的文件或文件集合。
这样做自然有它的道理，因为 HEAD 只是一个指针，你无法让它同时指向两个提交中各自的一部分。 
不过暂存区和工作区 可以部分更新，所以重置会继续进行第 2、3 步。

现在，假如我们运行 git reset file.txt （这其实是 git reset --mixed HEAD file.txt 的简写形式，
因为你既没有指定一个提交的 SHA-1 或分支，也没有指定 --soft 或 --hard），它会：
- 移动 HEAD 分支的指向 （已跳过）
- 让索引看起来像 HEAD （到此处停止）
所以它本质上只是将 file.txt 从 HEAD 复制到索引中。有 **取消暂存文件** 的实际效果。
```
$ git reset --mixed HEAD file.txt
```
或
```
$ git reset file.txt
```

我们可以不让 Git 从 HEAD 拉取数据，而是通过具体指定一个提交来拉取该文件的对应版本。 
```
$ git reset eb43bf file.txt
```

### 压缩提交
假设你有一个项目，第一次提交中有一个文件，第二次提交增加了一个新的文件并修改了第一个文件，第三次提交再次修改了第一个文件。 
由于第二次提交是一个未完成的工作，因此你想要压缩它。
```
$ git reset --soft HEAD~2
$ git commit
```

### 检出


## 高级合并
## Rerere
## 使用 Git 调试
## 子模块
## 打包
## 替换
## 凭证存储
## 总结

# 自定义 Git
8.1 配置 Git
8.2 Git 属性
8.3 Git 钩子
8.4 使用强制策略的一个例子
8.5 总结

# Git 与其他系统
9.1 作为客户端的 Git
9.2 迁移到 Git
9.3 总结

# Git 内部原理
10.1 底层命令和高层命令
10.2 Git 对象
10.3 Git 引用
10.4 包文件
10.5 引用规格
10.6 传输协议
10.7 维护与数据恢复
10.8 环境变量
10.9 总结