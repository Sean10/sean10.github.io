---
title: git笔记
date: 2017-02-21 00:12:09
updated:
tags: [git]
categories: [专业]
---

# 常用命令

## 切换分支
`git reset --hard`
## 查看操作历史，寻找之前记录的分支
`git reflog`
## 查找指定行的commit log
`git log -S "void ss()" /path/file`

## 查看未提交的文件修改

``` bash
git show
```
###  show not staged
``` bash
git diff
```

## 查看指定commit存在哪些分支
``` bash
git branch -r --contains xxx
```

## 查找指定commit后面的commit

``` bash
git log --oneline --reverse master..REF | head -n 10
```

## 查看被删除的文件相关的commit

```

git log --all -- src/mgr/PyModule.cc
```

## 查看tag之间差异的commit
``` bash
git log v12.2.12..v13.0.2 src/mgr
```

## tag功能

### 如何找到tag是从哪个分支的哪个commit打出?


## 子模块
### git subtree(第三方, 独立仓库子树合并)


### git submodule(独立的子仓库)

1. .gitmodules修改为mirror地址
2. git submodule sync 
3. git submodule init
4. git submodule update


或者  git submodule update --init --recursive

#### 添加git submodule

``` bash
git submodule add <url> <path>
git diff --cached
git commit -m "xxx"
```

#### submodule的submodule

``` bash
git submodule update --init --recursive
```

理论上这个可以, 那我编译时还触发了git clone的应该是因为代码并不是完全用submodule维持的关系了把?

# 回滚. `git reset --hard`即可. 


# git diff 与vscode对接

git支持通过difftool对接一些外部插件

``` bash
git config --global difftool.code.cmd 'code --wait --diff $LOCAL $REMOTE'

git config --global mergetool.code.cmd 'code --wait $MERGED'

git config --global -l

git difftool dev
git difftool -t code dev


git mergetool -t code dev

```

### 潜在问题: 好像在difftool这个交互进程退出前, 会在dock上弹出很多个vscode的图标


# 记录

git的快照流是指什么？
(参见 Git 内部原理 来了解更多关于到底 .git 文件夹中包含了哪些文件的信息。)
事实上，如果你的服务器的磁盘坏掉了，你通常可以使用任何一个克隆下来的用户端来重建服务器上的仓库（虽然可能会丢失某些服务器端的挂钩设置，但是所有版本的数据仍在，详见 在服务器上搭建 Git ）。

<!--more-->

`$ git clone https://github.com/libgit2/libgit2 mylibgit`

这将执行与上一个命令相同的操作，不过在本地创建的仓库名字变为 mylibgit。

Git 支持多种数据传输协议。 上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用 SSH 传输协议，比如 user@server:path/to/repo.git 。 在服务器上搭建 Git将会介绍所有这些协议在服务器端如何配置使用，以及各种方式之间的利弊。

最后，该命令还显示了当前所在分支，并告诉你这个分支同远程服务器上对应的分支没有偏离。 现在，分支名是 “master”,这是默认的分支名。 我们在 Git 分支 会详细讨论分支和引用。


忽略文件
一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 .gitignore 的文件，列出要忽略的文件模式。 来看一个实际的例子：

```
$ cat .gitignore
*.[oa]
*~
```

第一行告诉 Git 忽略所有以 .o 或 .a 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的。 第二行告诉 Git 忽略所有以波浪符（~）结尾的文件，许多文本编辑软件（比如 Emacs）都用这样的文件名保存副本。 此外，你可能还需要忽略 log，tmp 或者 pid 目录，以及自动生成的文档等等。 要养成一开始就设置好 .gitignore 文件的习惯，以免将来误提交这类无用的文件。

文件 .gitignore 的格式规范如下：

所有空行或者以 ＃ 开头的行都会被 Git 忽略。

可以使用标准的 glob 模式匹配。

匹配模式可以以（/）开头防止递归。

匹配模式可以以（/）结尾指定目录。

要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

> 所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（\*）匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 使用两个星号（*) 表示匹配任意中间目录，比如a/**/z 可以匹配 a/z, a/b/z 或 a/b/c/z等。

我们再看一个 .gitignore 文件的例子：
``` bash
# no .a files
*.a

# but do track lib.a, even though you're ignoring .a files above
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
/TODO

# ignore all files in the build/ directory
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```

TIP
GitHub 有一个十分详细的针对数十种项目及语言的 .gitignore 文件列表，你可以在 https://github.com/github/gitignore 找到它.

查看已暂存和未暂存的修改



git .gitignore 我的powershell不能调用，待会看看






另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 --cached 选项：
```
$ git rm --cached README
git rm 命令后面可以列出文件或者目录的名字，也可以使用 glob 模式。 比方说：

$ git rm log/\*.log
```
注意到星号 * 之前的反斜杠 \， 因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用 shell 来帮忙展开。 此命令删除 log/ 目录下扩展名为 .log 的所有文件。 类似的比如：
```
$ git rm \*~
```
该命令为删除以 ~ 结尾的所有文件。.


Git 保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。


http://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF

远程分支不太能够理解，比如远程仓库和本地仓库分支之间fetch什么的
还有origin/master怎么操作
现在，可以运行 git fetch teamone 来抓取远程仓库 teamone 有而本地没有的数据。 因为那台服务器上现有的数据是 origin 服务器上的一个子集，所以 Git 并不会抓取数据而是会设置远程跟踪分支 teamone/master 指向 teamone 的 master 分支。

这里有些工作被简化了。 Git 自动将 serverfix 分支名字展开为 refs/heads/serverfix:refs/heads/serverfix，那意味着，“推送本地的 serverfix 分支来更新远程仓库上的 serverfix 分支。” 我们将会详细学习 Git 内部原理 的 refs/heads/ 部分，但是现在可以先把它放在儿。 你也可以运行 git push origin serverfix:serverfix，它会做同样的事 - 相当于它说，“推送本地的 serverfix 分支，将其作为远程仓库的 serverfix 分支” 可以通过这种格式来推送本地分支到一个命名不相同的远程分支。 如果并不想让远程仓库上的分支叫做 serverfix，可以运行 git push origin serverfix:awesomebranch 来将本地的 serverfix 分支推送到远程仓库上的 awesomebranch 分支。

如何避免每次输入密码

如果你正在使用 HTTPS URL 来推送，Git 服务器会询问用户名与密码。 默认情况下它会在终端中提示服务器是否允许你进行推送。

如果不想在每一次推送时都输入用户名与密码，你可以设置一个 “credential cache”。 最简单的方式就是将其保存在内存中几分钟，可以简单地运行 git config --global credential.helper cache 来设置它。

想要了解更多关于不同验证缓存的可用选项，查看 凭证存储。




git branch 如何切换一个本地分支跟踪远程仓库的分支
git checkout

哑（Dumb） HTTP 协议
如果服务器没有提供智能 HTTP 协议的服务，Git 客户端会尝试使用更简单的“哑” HTTP 协议。 哑 HTTP 协议里 web 服务器仅把裸版本库当作普通文件来对待，提供文件服务。 哑 HTTP 协议的优美之处在于设置起来简单。 基本上，只需要把一个裸版本库放在 HTTP 跟目录，设置一个叫做 post-update 的挂钩就可以了（见 Git 钩子）。 此时，只要能访问 web 服务器上你的版本库，就可以克隆你的版本库。 下面是设置从 HTTP 访问版本库的方法：

```
$ cd /var/www/htdocs/
$ git clone --bare /path/to/git_project gitproject.git
$ cd gitproject.git
$ mv hooks/post-update.sample hooks/post-update
$ chmod a+x hooks/post-update
```

这样就可以了。 Git 自带的 post-update 挂钩会默认执行合适的命令（git update-server-info），来确保通过 HTTP 的获取和克隆操作正常工作。 这条命令会在你通过 SSH 向版本库推送之后被执行；然后别人就可以通过类似下面的命令来克隆：

$ git clone https://example.com/gitproject.git
这里我们用了 Apache 里设置了常用的路径 /var/www/htdocs，不过你可以使用任何静态 web 服务器 —— 只需要把裸版本库放到正确的目录下就可以。 Git 的数据是以基本的静态文件形式提供的（详情见 Git 内部原理）。

通常的，会在可以提供读／写的智能 HTTP 服务和简单的只读的哑 HTTP 服务之间选一个。 极少会将二者混合提供服务。

git pull返回错误“You asked to pull from the remote 'origin', but did not specify a branch. Because this is not the default configured remote for your current branch, you must specify a branch on the command line.”

http://stackoverflow.com/questions/3133387/confusing-error-message-from-git


refusing to merge unrelated histories

强制`git merge --allow-unrelated-histories yourbranch`

w强制回滚，用来撤销失败的调试
`git reset --hard HEAD`


>Auto packing the repository in background for optimum performance.
>See "git help gc" for manual housekeeping.
>error: The last gc run reported the following. Please correct the root cause
>and remove .git/gc.log.
>Automatic cleanup will not be performed until the file is removed.
>
>warning: There are too many unreachable loose objects; run 'git prune' to remove them.

>Git的底层并没有采用 CVS、SVN 底层所采用的那套增量式文件系统，而是采用一套自行维护的存储文件系统。当文件变动发生提交时，该文件系统存储的不是文件的差异信息，而是文件快照，即整个文件内容，并保存指向快照的索引。这种做法，提高 Git 分支的使用效率；但也容易导致代码仓库中内容重复程度过高，从而仓库体积过大。当遇到这种情况时，或者需要将仓库推送到远程主机时，就需要Git中的gc（garbage collect）功能，也就是垃圾回收功能。
>
>大体来说，当运行 "git gc" 命令时，Git会收集所有松散对象并将它们存入 packfile，合并这些 packfile 进一个大的 packfile，然后将不被任何 commit 引用并且已存在一段时间 (数月) 的对象删除。 此外，Git还会将所有引用 (references) 并入一个单独文件

`git prune`功能，似乎是在遇到大文件的问题,有意思，这个git的垃圾回收问题以前倒是没遇到过呢

# git status显示utf-8中文

```
git config --global core.quotepath false
```

# gitlab/github免密

使用ssh密钥时, .ssh/config里的host和对应的`IdentifyFile`也要指定.

然后git config 看到的username和credential也要对应.

# git merge
## 从指定分支合入log到当前分支


# git pull 和git pull --rebase



# git rebase



``` bash
➜  markless git:(dev) git push origin dev
To https://github.com/sean10/markless.git
 ! [rejected]        dev -> dev (non-fast-forward)
error: failed to push some refs to 'https://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@github.com/sean10/markless.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

可以用下述强制命令.

``` bash
git push --force-with-lease origin dev  
```

# gh (github CLI)

``` bash

# 登录, 使用token生成
gh auth login 
# 配置该token缺失的权限
gh auth refresh -h github.com -s admin:public_key
gh repo set-default
gh pr view 18276
```


# gitlens 使用

## search commits 唯一遇到问题是有多个repo时, 每次都需要选择一下

手动在git的`remotes`界面右键关闭Repo, 下次搜索就不用手动选择了, 不会搜那个repo了.
# 参考
1. [git gc功能](http://blog.csdn.net/lihuanshuai/article/details/37345565)
2. [Git子仓库深入浅出 \- 知乎](https://zhuanlan.zhihu.com/p/100214931)****



# 快速在commit间切换  [hutusi/git\-paging: Treat git log as a book, exec \`git next\` or \`git prev\` to checkout the next or the previous commit\.](https://github.com/hutusi/git-paging)

### 查看上次和本次之间的diff
``` bash
git diff HEAD^ HEAD
```


[阅读开源代码小技巧 \| 胡涂说](https://hutusi.com/articles/git-paging)