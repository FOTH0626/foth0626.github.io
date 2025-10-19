---
title: ProGit Learn Record 2：Git基础
description: learn progit book record 2
slug: progit-book-2
image:
draft: false
categories:
  - Programming
date: 2025-10-18 15:47:07+08:00
tags:
  - Git
weight:
---
>本系列文章基于《Pro Git》一书，作为本人学习记录
# 获取Git仓库
1. 将一个没有版本控制的本地目录转换为Git仓库
2. 从服务器克隆(clone)一个已存在的仓库
## 从本地目录初始化
首先，`cd /path/to/your/project` ，然后执行 `git init` ，这会创建 `.git` 的一个子目录。  
如果此时这个目录非空，那么应该开始追踪并进行初始提交。 使用 `git add` 指定需要追踪的文件，然后 `git commit`  
```bash
git add *.cpp
git add LICENSE
git commit -m 'inital project version' 
```

## 克隆已有仓库
从服务器上克隆已有的数据（包括所有的历史数据） 
`git clone https://github.com/libgit2/libgit2`    
这会在当前目录创建一个libgit2的目录，并创建 `.git` 目录，将数据存放进去。  
如果你想要使用自定义的目录名那么  
`git clone https://github.com/libgit2/libgit2 mylibgit`  
除了使用https，git也可以使用git:// 或者 ssh 协议。  

# 记录更新到仓库
通常，你会对文件做些修改，并在完成修改后提交到仓库以记录下修改。  
工作目录下的所有文件只存在两种状态： **已跟踪** 和 **未跟踪** 。
已跟踪的文件就是那些已经被纳入版本控制的文件，它们的状态是未修改，已修改或已暂存。  
所以有如下关系:
1. **未跟踪**文件被添加到版本控制后，会进入**已暂存**状态。
2. **未修改**文件在修改后变为**已修改**文件，移除文件变为**未跟踪** 。
3. **已修改**文件在暂存后变为**已暂存**文件
4. 暂存文件在commit变为**未修改**文件
## 检查文件状态
使用 `git status` 查看文件处于什么状态。对于刚克隆的仓库，应该是"everything is up to date" 的。  
新建一个README文件，再使用`git status`可以看到输出结果多了`Untracked Files`，包含了README文件
```bash
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README
```
## 跟踪新文件
使用`git add FILENAME` 开始跟踪文件，此时再`git status` 则发现 `Changes to be committed` 这表明文件处于 **已暂存** 状态。   
`git add` 接受文件或者目录，如果是目录则会递归跟踪该目录下的所有文件。  
```bash
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   README
```
## 暂存已修改文件 
假设现在有一个文件 `aaa.md` ，且已被跟踪。你对它进行了修改，此时运行`git status` 则会有：
```bash
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   aaa.md

no changes added to commit (use "git add" and/or "git commit -a")
```
`Changes not staged for commit` 表明已跟踪文件发生了变化，但没有暂存。如果需要**暂存**这次更新，则运行`git add` ，当这个命令接受的文件是已跟踪文件，则功能变化为将已跟踪的文件放入暂存区。将这个命令理解为“将内容添加到下一次的提交中”是比较合适的说法。  
```bash
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   aaa.md
```
注意：如果你现在对`aaa.md`进行修改，再运行`git status` 
```bash
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   README
	modified:   aaa.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   aaa.md
```
现在`aaa.md` 同时存在暂存区和非暂存区，这是因为Git记录的是你最后一次`git add` 的文件版本，所以如果你需要提交最新文件那么，你得再次`git add` 
```bash
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   README
	modified:   aaa.md
```
## 状态简略输出
使用`git status -s` 或者 `git status --short` 可以获得一个更为紧凑的输出
```bash
A  README
M  aaa.md
```
事实上，输出类型共有三种分别为`?`,`A`,`M`,分别指代**未跟踪**，**新添加**，**已修改**。 不仅如此，输出结果事实上有两栏，我现在再对`aaa.md` 修改。
```bash
A  README
MM aaa.md
```
**左栏代表着暂存区状态，右栏表示工作区状态。**  
所以`MM`表明文件修改有一部分是暂存的，还有一部分没有被暂存。
## 忽略文件（.gitignore)
有些文件我们不希望它进入Git的管理中，我们也不希望它们出现在未跟踪列表。比如编译缓存，日志等。此时我们可以创建`.gitignore` 文件。下面是一个示例
```.gitignore
*.[oa]
*~
```
第一行表明忽略所有 `.o` 和 `.a` 文件。第二行表明 忽略所有最后以`~`结尾的文件。    
`.gitginore` 的规则如下：
1. 所有空行或 以`#` 开头的行会被忽略
2. 使用glob模式匹配，并且递归的应用于整个工作区中。
3. 匹配模式可以使用`/` 开头防止递归（使用`/temp`这种写法，只会匹配根目录的temp文件夹，而不会匹配其他目录里的文件夹，如果使用`temp/`则会匹配所有temp文件夹）
4. 匹配模式可以使用`/` 结尾指定目录（如`build/`）
5. 一些符合匹配忽略规则的，但是我们希望加入版本管理的文件，使用`!` 取反。  

其中glob模式是指：
1. 使用`*`作为通配符，匹配任意多个任意字符。
2. 使用`[]`匹配方括号内任意一个字符。如`[abc]`可以为a,b,c任意一个字符
3. `？` 匹配任意**一个**字符
4. 使用`**`匹配任意中间目录，如`src/**/*.js`则会匹配所有子文件的`js`文件    

## 查看已暂存和未暂存的修改
`git status`只能查看哪些文件有改变，但是不能查看具体修改了些什么，此时可以使用`git diff`，`git diff`能通过文件补丁的方式具体显示哪些行发生了变化。这个指令展示的是工作区和暂存区之间的差异，也就是修改后没有暂存的内容。  
如果想要查看已暂存的文件和最后一次提交（commit）的区别，那么使用`git diff --staged` 或者 `git diff --cached`  
## 提交更新（commit）
在提交之前，请确保没有已经修改的文件却没有`git add`过，否则提交时不会记录这些文件的变化。所以使用 `git status` 确保所有文件已暂存。
使用 `git commit` 指令提交更新，这是会启用你的文本编辑器来输入提交说明。  
`git commit` 会默认使用注释格式显示 `git status` 的内容，这有助你确认修改内容，如果你希望查看 `git diff` 的输出内容，那么commit时添加额外的参数 `-v` 。  
除此之外，如果你希望直接在命令里输入提交说明，那么，使用参数 `-m` 并附带消息，如  `git commit -m "God Bless You!"`   
## 跳过暂存区
如果你不希望添加暂存并提交，这样略繁琐的步骤，那么，直接使用参数 `-a` 可以直接把所有已跟踪文件暂存并提交  
`git commit -a -m "add: new feature"`   
## 移除文件
如果想要从Git中移除文件，那么就也得从暂存区中移除并提交。如果直接在本地手动删除文件，那么会导致这个文件仍然记录在未暂存区域。需要使用 `git rm FILENAME` 来移除一个未修改文件。   
如果一个文件已经保留在暂存区中，或者处于已修改的文件，那么需要使用 `git rm -f <filename>` ，这是为了确保尚未添加到快照中的文件不被误删除。或者使用 `git rm --cached <filename>` 指令，这会使文件在Git仓库中被删除，但是保留本地文件，这是为了处理误添加了一堆不想跟踪的文件进暂存区的情况 ，如没有使用 `.gitignore` 导致的一堆编译文件。   
当然 `git rm ` 指令后也可以使用glob模式匹配文件，如 `git rm log/\*.log` 注意，`\`是用于展开通配符 `*` 的。    
## 移动文件  
在Git中对文件改名，使用 `git mv file_from file_to` 这等价于   
```bash
mv file_from file_to
git rm file_from
git add file_to
``` 
# 查看提交历史
使用 `git log` 指令，可以查看所有提交，默认最近的更新在最上方，并会给出每次提交的校验，作者的姓名和邮箱，提交时间，提交说明。  

`git log` 有许多选项，其中 `-p` 或者 `--patch` ，这会按照补丁（类似于`git diff`)的格式输出，也可以使用 `-2` 这类命令限制一次输出的日志数量。`--stat` 会按照简略形式表示更改。  
```bash
commit ded2ebf137ddc0b1f60e09fe3c96a3b88f491033 (HEAD -> master)
Author: foth <fothli0626@gmail.com>
Date:   Sat Oct 18 19:34:56 2025 +0800

    God Bless You

 README | 1 +
 aaa.md | 2 ++
 2 files changed, 3 insertions(+)

commit 42a782e56593d75b86ef8f3d52309dc09ed66406
Author: foth <fothli0626@gmail.com>
Date:   Sat Oct 18 16:25:11 2025 +0800

    aaa.md

 aaa.md | 1 +
 1 file changed, 1 insertion(+)
```  
## 格式参数
使用 `pretty` 参数可以改变提交显示的格式。有 `oneline` `short` `full` `fuller` 等内置参数，还可以使用 `pretty=format"THIS IS CUSTOM FORMAT"` 自定义输出格式。关于format具体有哪些格式和参数，这里不做介绍。  
 `--graph` 能够是用`ascii`字符画形式，可视化的展现仓库的提交历史（俗称画电路板（笑）。）  
 ## 时间参数
 上面我们提到可以展示任意条数量记录的 `-<n>` 参数，也可以使用 `--since` 或者 `--until` 这种形式 `--since=2.weeks` 表示最近两周的提交，这类格式不仅可以使用绝对日期，也可以使用相对时间如 `"2 years 1 day 3 minutes ago"`   
 
 ## 过滤与搜索
 使用 `--author` , `--commiter`  `--grep` 可以搜索作者，提交者和提交信息中的关键词。  
 另一个有用的过滤条件是参数 `-S` ，这个参数需要一个字符串参数，这会搜索出所有修改的文件内容中包含该字符串的提交。比如你想寻找添加或者删除了某一个函数的调用的提交那么可以使用 `git log -S "function_name"` 。  
 最后，如果想要搜索某个有关特定文件的提交，可以使用 `-- path/to/file` 这个参数，注意，这个参数**必须**为 `git log` 的最后一个参数。
不过注意，当有多个参数搜索时，任一条件满足即会打印，加上 `--all-match` 参数才会打印出所有条件都满足的提交。（即 `||` 逻辑和 `&&` 逻辑）  
# 撤销操作
这一节会有一些撤销操作的教学，**注意**部分操作是不可逆的。  
## 替换commit
`git commit --amend` 会“覆盖”你的上一次提交，通常我们会用于修改一些小错误或者重写提交信息。如果只需要更改消息，那么直接运行该指令即可，如果需要修改文件或者提交漏提交的文件，那么可以这样：
```bash
git commit -m "a mistake commit" 
git add missing_file
git commit --amend
```
这个指令会使新的提交替换掉旧的提交，所以旧的提交完全不存在仓库的历史中。这能使你的提交历史被“修正笔误”或者“忘记提交一个文件”之类的记录弄乱你的记录。  
l## 撤销已暂存的文件 
如果你修改了两个文件，想分两次提交，但是已经一次性提交了，应该如何操作？  
在 `git status` 中，已有提示: `git reset HEAD <filename>` 来取消暂存。  
如何取消已修改但为未暂存的文件？此事在 `git status` 中亦有记载：  
使用 `git checkout -- <filename>` 可以撤销修改，不过这是一个危险的命令，因为实质上是使用上一次提交里的文件版本覆盖掉当前文件。  
# 远程仓库（remote）
远程仓库指托管在网络中的仓库，与他人协作时需要管理远程仓库。  
>远程仓库可以在本地的主机上，这里的远程表明的是仓库在别处而已。
## 查看远程仓库  
使用 `git remote` 查看远程仓库的简写，通常来说，你应该能看到 `origin` ，这是克隆的仓库的默认名字。  
使用 `git remote -v` 则会额外列出仓库简写的对应的URL。这个指令也会列出全部的远程仓库（如果不止一个）。  
## 添加远程仓库  
使用 `git remote add <shrotname> <url>` 的格式添加远程仓库并指定简写。  
现在可以使用简写代替这个仓库的URL，如拉取远程仓库的更新 `git fetch origin`   
## 从远程仓库抓取与拉取
就如刚才所提：使用 `git fetch <remote>` 可以获取远程仓库中的数据，并将新数据抓取到本地，值得注意的是，这不会合并到你当前的工作区，所以需要手动合并。  
一般来说， `git clone` 克隆一个仓库，默认会将这个仓库设置为远程仓库，并添加简写为 `origin` 。  
如果当前分支已经设置了跟踪远程分支（请等待git分支一节），那么使用 `git pull` 即可自动抓取远程并合并。另外， `git clone` 命令会默认设置本地的 `master` 分支跟踪远程的 `master` 分支。 运行 `git pull` 会从克隆的服务器上抓取并自动尝试合并到当前分支。 
## 推送到远程仓库 
使用 `git push <remote> <branch>` 即可， 通常来说，有：  
`git push origin master` （这是克隆的默认名称）  
并且，只有你拥有远程服务器的写入权限，并且没有其他人推送过，这才是有效的。否则，你需要将他人的工作合并到本地后才能推送。  
## 查看远程仓库
使用 `git remote show <remote>` ,如此博客： 
```bash
git remote show origin
* remote origin
  Fetch URL: https://github.com/FOTH0626/foth0626.github.io.git
  Push  URL: https://github.com/FOTH0626/foth0626.github.io.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```
这能显示每个分支会推送的远程分支，哪些远程分支不在本地等等信息。  
## 远程仓库重命名 
使用 `git remote rename <old> <new>` 这也会改变你远程跟踪的分支名字。  
如果你需要移除一个远程仓库（如你不再使用服务器/不使用某个镜像/某个维护者不再维护），那么可以使用 `git remote remove <remote>` ，这也会删除远程仓库的跟踪和配置信息。-  
# 标签
## 列出标签
使用 `git tag` 命令（可加上 `-l` 或者 `--list` 按照通配符查找)  
## 创建标签
Git 存在两种标签：轻量（lightweight）与附注（annotated）。  
轻量标签只是某个提交的引用，它是不会改变的。  
附注标签是一个完整的，存储在Git数据库的对象，它是可以被校验的，包含标签创建者的姓名、电子邮件、日期时间、标签信息，并且可以使用GNU Privacy Guard(GPG)进行签名并校验。  
## 附注标签
使用 `-a` 参数即可创建附注标签，如 `git tag -a <tag> -m <message>` ，如果没有显式地使用 `-m` 那么，将如提交时一样，需要在文本编辑器内输入标签的信息。  
使用 `git show <tag>` 可以查看该标签的详细信息。有标签创建者信息，标签日期，附注，提交信息。（这里我的显示与书中不同，我额外显示了这次commit的修改文件的具体信息，很奇葩。）  
## 轻量标签
不额外附带参数即可创建轻量标签，如 `git tag <tag>` 。  
这时的标签只会展现出提交信息。（不过我还是显示了一堆消息，可能是 `git show` 的问题？） 
## 后期打标签（对过去的提交打标签）
对于一个已经提交的记录，可以使用 `git tag -a <tag> <checksum>` 
```bash
$ git log --pretty=oneline
33849c071fbd57dbefcb279b97255b24c794f319 (HEAD -> master, tag: 1.0, tag: 0.1) test
ded2ebf137ddc0b1f60e09fe3c96a3b88f491033 God Bless You
42a782e56593d75b86ef8f3d52309dc09ed66406 aaa.md
```
可以使用 `git tag -a v0.8 ded2eb` 在 `God Bless You` 打上tag。  
## 共享标签
默认情况下， `git push` 并不会推送标签到远程服务器，必须显式地使用 `git push origin <tag>` 推送标签，或者使用 `git push origin --tags` 附带所有不在远程仓库的所有tag提交。

>没有简单的方法可以只推送附注标签或轻量标签。

## 删除标签 
使用 `git tag -d <tagname>` 。   
例如 `git tag -d 0.8` ,与push不会推送到远程一样，这个指令不会删除远程仓库，需要使用 `git push <remote> :refs/tags/<tag>` 才能更新远程仓库。  
另外一种方式是使用指令： `git push origin --delete <tag>` 

## 检出标签
如果想要查看某个标签指向的版本，可以使用 `git checkout <tag>` ，但是这会让你出于“分离头指针”状态，这会导致不好的副作用。  
这个状态下，你的新提交不会属于任何分支，并且无法访问，除非通过确切的提交哈希。因此，需要进行更改时，需要新建一个分支。（总之我也没看懂，尽量不要使用该该功能？欢迎指正）
# Git别名
可以使用 `git config` 为命令设置一个别名。如
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
```
当使用 `git commit` 时， 只需要输入 `git ci` 。  
别名是非常有用的，所以创建时不要犹豫，比如取消暂存文件的命令可以输入 `git config --global alias.unstage 'reset HEAD --'` 这让两个命令等价。  
```bash
git unstage <file>
git reset HEAD -- <file>
```
或者有： `git config --global alias.last 'log -1 HEAD'`  
这样可以快速查看最后一次提交。  
如果想使用外部程序，可以添加 `!`，如  
`git config --global alias.visual '!gitk'` ，可以使用 `git visual` 调用 `gitk` 程序。（这GUI真tm的丑。2000年前后的风格） 
# 总结
这一节我们学习了基础操作，下一节我们来学习Git杀手级特性：Git分支模型。