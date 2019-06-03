---
title: github遇到的坑
date: 2017-11-27 00:34:53
tags:
  - hexo
  - git
categories: git
---
部署博客时（hexo d）

遇到错误   
<font color=orange >Error: EACCES: permission denied, unlink '/home/mitrecx/blog/.deploy_git/about/index.html'</font>

遇到这样的权限问题，理所当然的想到
```sh
chmod -R 777 .
```
简单粗暴，解决问题。

**但是，我没有这么做！**

我直接
```sh
sudo hexo d
```
新问题来了
<font color=orange>
ssh: connect to host my.github.com port 22: Connection timed out　　　　
fatal: Could not read from remote repository.　　
Please make sure you have the correct access rights
and the repository exists.　　
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html

</font>

**为什么会ssh　github　会有问题？**　　　

我试了一下：
```sh
ssh -T git@github.com
```
[两个帐号都可以连接到github]()

但是 用sudo就不行  
[sudo ssh连接github提示publickey不行]()

原因很明显了：
**我用的 sshkey　是 mitrecx 用户生成的。sudo执行 命令时，用的是 root 用户的 sshkey，所以github会拒绝接受git请求。**    
sshkey在 __~./ssh__ 目录下，root 可以复制，查看 mitrecx 的 sshkey, 感觉有点不安全.  
但是, root 用户 连 mitrecx 的 登录密码 都能改,  还说啥安全不安全的.....

github权限的坑踩完了。  

为什么最初我在 <code>hexo d</code> 会出出错呢？
**原因是我在 <code>hexo d</code> 执行前，执行的是 <code>sudo hexo g</code> ，而不是 <code>hexo g</code>** 。
