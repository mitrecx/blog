---
title: git简单教程
date: 2017-11-26 20:04:51
tags:
  - git
categories: git
---
# 从零开始－－简单使用
首先, [强烈推荐这个网站](https://git-scm.com/book/zh/v2).    


```shell
git init #初始化(新建)一个 本地 git 仓库

#第一次把本地仓库提交到 github
#添加到 暂存区，不添加的话 filename是 untracked ,untracked files不能 commit
git add <filename>  
#把暂存区的内容提交到 HEAD, 仍然在 本地git仓库
git commit -m "关于本次提交的说明信息"
#本地仓库连接远程仓库 ，连接名为 origin
git remote add origin  git@github.com:username/repository.git
#把 HEAD 里的内容提交到 github 上的 master 分支
git push origin master

#再次操作
#现在有本地仓库，也和github连接过（有连接名origin），
#只要三步就能push到github
git add <filename>
git commit -m "commit info"
git push origin <branch-name>

#
#分支
git branch  #查看分支， -a查看所有分支
git branch -d  branchname  #有些时候可能会删除失败,比如如果branchname分支的代码还没有合并到master, -D 可以强制删除
git branch new-branch  #创建名为new-branch的分支
git checkout new-branch  #切换到new-branch分支
#
git checkout -b new-branch  #简化：创建并切换

#
#频繁使用的一个命令－－查看状态
git status

```
如果git add一个不想要的提交的file，可以用以下命令清除
```shell
git rm --cached <filename> #只是删除 暂存区 中的文件
# 不想 追踪 track 的文件 要 添加到 .gitignore 文件中呀  

git rm -f <filename> #删除 暂存区 中的文件 同时 删除 磁盘上本地文件, 不可恢复
```
# 从一开始
## 工作流
你的本地仓库由 git 维护的三棵“树”组成。第一个是你的 **工作目录**，它持有实际文件；第二个是 **暂存区（Index)**，它像个缓存区域，临时保存你的改动；最后是 **HEAD**，它指向你最后一次提交的结果。


## 创建新仓库
创建新文件夹，打开，然后执行
```sh
git init
```
创建新的 git 仓库

## 添加和提交
提出更改（把它们添加到暂存区），使用如下命令：
```sh
git add <filename>
```
举一个例子，
- 如果想把当前目录下的所有文件添加到 __暂存区INDEX__ ,执行：
```sh
git add .
```
- 如果想把当前目录下的某几个文件添加到 __暂存区INDEX__ ,执行：
```sh
git add <filename1,2,3>
```
还有个方法，在当前目录下建立一个名为
<font color=blue size=3>.gitignore</font> 的文件。然后把想要忽略的文件名添加在<font color=blue size=3>.gitignore</font>中,  这样这些文件就不会 出现在未跟踪文件列表.  

.gitignore 用 glob 模式匹配 (git 命令 也可以使用 glob 模式).  
所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。   
星号（\*）匹配零个或多个任意字符；  
[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；  
问号（?）只匹配一个任意字符；  
如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。  
使用两个星号（\*) 表示匹配任意中间目录，比如 a/\*\*/z 可以匹配 a/z , a/b/z 或 a/b/c/z 等。  
<font color=blue size=3>.gitignore</font> [示例](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%AE%B0%E5%BD%95%E6%AF%8F%E6%AC%A1%E6%9B%B4%E6%96%B0%E5%88%B0%E4%BB%93%E5%BA%93):  
 ```
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

撤消对文件的修改:  
如果 修改 了 已追踪文件 README (未提交), 但是 后来又想 撤回 修改(撤回到暂存区的状态):    
```
git checkout -- README
```
如果 暂存区 的想 撤回 到 已提交的本地git库:  
```
# 先 移除 暂存区. 也可以使用: git rm --cached README
git reset README
# 再 checkout 检查撤回
git checkout -- README
```

放弃 本地git仓库 提交:  
```
git fetch --all
git reset --hard origin/master
```

注意:  
```
# 远程仓库中抓取 数据, 它并不会自动合并或修改你当前的工作
git fetch [remote-name]

# 自动的抓取然后合并远程分支到当前分支
git pull [remote-name] [branch-name]
```
git add是 git 基本工作流程的第一步；使用如下命令以实际提交改动 到本地仓库：
```shell
 git commit -m "代码提交信息"
```
现在，你的改动已经提交到了 HEAD，但是还没到你的远端仓库

## 推送改动
你的改动现在已经在本地仓库的 **HEAD** 中了。执行如下命令以将这些改动提交到远端仓库：
```sh
git push origin master
```
可以把 master 换成你想要推送的任何分支。

如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：
```sh
git remote add origin <server>
```
如此你就能够将你的改动推送到所添加的服务器上去了

## 分支
分支是用来将特性开发绝缘开来的。在你创建仓库的时候，master 是“默认的”分支。在其他分支上进行开发，完成后再将它们合并到主分支上。
```sh
git branch hexo  //新建hexo分支
git checkout hexo  //切换到hexo分支上
```
```sh
git checkout -b feature_x  #创建一个叫做“feature_x”的分支，并切换过去：
git checkout master  #切换回主分支

git branch -d feature_x  #再把新建的分支删掉
git push origin <branch>  #将分支推送到远端仓库
```

## 更新与合并
A同学在a分支代码写的不亦乐乎,终于他的功能完工了,并且测试也都ok了,准备要上线了,这个时候就需要把他的代码合并到主分支master上来,然后发布。git	merge	就是合并分支用到的命令,针对这个情况,需要先做两步,第一步是切换到	master	分支,如果你已经在了就不用切换了,第二步执行	git	merge	a	,意思就是把a分支的代码合并过来,不出意外,这个时候a分支的代码就顺利合并到	master	分支来了。  
git pull命令的作用是：取回远程主机某个分支的更新，再与本地的指定分支合并


## git clone

```shell
#1.最简单直接的命令
git clone xxx.git

#2.如果想clone到指定目录
git clone xxx.git "指定目录"

#3.clone时创建新的分支替代默认Origin HEAD（master）
git clone -b [new_branch_name]  xxx.git
```
git clone 命令默认的只会建立master分支.  


1  查看所有分支(包括隐藏的)  git branch -a 显示所有分支，如：　　　
```
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
```
2  在本地新建同名的("dev")分支，并切换到该分支  
```
git checkout -t origin/dev
#等同于：
git checkout -b dev origin/dev
```
执行完2，发现dev分支的内容已经存在于本地仓库了。  

# 其他常用 git命令
假设 远程分支如下:  
```
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/mitrecx/design-pattern.git
  Push  URL: https://github.com/mitrecx/design-pattern.git
  HEAD branch: master
  Remote branches:
    develop  tracked
    master   tracked
    test-old new (next fetch will store in remotes/origin)
    testing  tracked
  Local branches configured for 'git pull':
    develop merges with remote develop
    master  merges with remote master
    testing merges with remote testing
  Local refs configured for 'git push':
    develop pushes to develop (up to date)
    master  pushes to master  (up to date)
    testing pushes to testing (fast-forwardable)

```
**远程仓库名:** origin.   
**远程分支** 有 4 个: master, develop, testing, test-old.  
**本地分支** 有 3 个: master, develop, testing.  
三个远程分支, 都被追踪了(tracked) . 同时 <code> git pull </code> 自动合并:  
develop--> origin/develop;   
master--> origin/master.  
所以, 要想合并 origin/testing 分支 到 testing, 要手动 merge.  
但也可以用命令 <code>git branch -u origin/testing</code> 实现自动合并映射 (设置跟踪分支) .     

```
# 获得远程分支信息
git remote show (remote)

# 抓取 远程仓库origin 的数据 到本地
git fetch orign
# 把抓取到的 远程仓库 master 分支数据 和 本地仓库的 master分支合并
git checkout master
git merge origin/master
# 上述 抓取合并 操作 一般等价于 git pull origin master

# 查看设置的所有跟踪分支
git branch -vv

# 查看 分叉历史
git log --oneline --graph --all

```
