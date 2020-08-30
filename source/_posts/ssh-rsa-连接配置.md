---
layout: _post
title: ssh rsa 连接配置
date: 2020-08-29 16:01:57
tags:
---

印象笔记的VIP到期了，还是得自己搞个小站记录下，最方便

```
# 客户端
# 查看下本地 ~/.ssh 下是否有自己常用的 id_rsa id_rsa.pub 私钥跟公钥

# 默认生成路径~/.ssh
$ ssh-keygen -t rsa -C "邮箱地址" # 不写-C也没事，生成私钥公钥

$ chmod 700 ~/.ssh
$ cd ~/.ssh
$ chmod 600 ~/.ssh/id_rsa

id_rsa，私钥，必要

id_rsa.pub，公钥，必要，留待以后添加到其他远程主机

known_hosts，记录本机ssh免密登陆的远程主机列表，可选

-------------------------------------------------------------------------

# 服务端
$ vim /etc/ssh/sshd_config 
# 增加下列配置
# RSAAuthentication yes 
# PubkeyAuthentication yes 
# AuthorizedKeysFile .ssh/authorized_keys 

$ systemctl restart sshd   # 重启ssh服务
$ cd ~/.ssh
# 复制客户端的公钥
# 手动把公钥复制到远程主机，添加到~/.ssh/authorized_keys文件中
$ chmod 700 ~/.ssh
$ cd ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys

可在客户端~/.ssh/下创建config文件，添加如下配置，直接通过别名就能登陆了
Host            alias            #自定义别名
HostName        hostname         #替换为你的ssh服务器ip或domain
Port            port             #ssh服务器端口，默认为22
User            user             #ssh服务器用户名
IdentityFile    ~/.ssh/id_rsa    #第一个步骤生成的公钥文件对应的私钥文件


```