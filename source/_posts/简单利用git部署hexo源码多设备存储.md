---
layout: _post
title: 简单利用git部署hexo源码多设备存储
date: 2020-08-30 13:29:34
tags:
---

#### node安装

```shell
wget https://nodejs.org/dist/v10.16.0/node-v10.16.0-linux-x64.tar.xz
mkdir /opt/software/ && cd /opt/software/

tar -xvf node-v10.16.0-linux-x64.tar.xz

mv node-v10.16.0-linux-x64 nodejs

# 建立软连接，变为全局

ln -s /opt/software/nodejs/bin/npm /usr/local/bin/

ln -s /opt/software/nodejs/bin/node /usr/local/bin/

```



#### hexo安装

```shell
npm install hexo-cli g
```

**hexo 源码项目构建**

* 首先静态源码文件夹，以及主题，markdown文档，配置文件

  * **scaffolds**
  * **source**
  * **themes**

  * **_config.yml**

  > 基本这四个文件跟文件夹配置涵盖了写作文章跟个性化配置，把他们做成git 仓库就可以多设备部署，并没有利用github进行托管，用的是自己服务器搭建，需要集成CI/CD托管的绕道

* hexo 项目本地初始化

  ```shell l
  mkdir xxx.github.io && cd xxx.github.io
  hexo init
  
  # 再进行git 仓库初始化(git自行安装)
  git init
  
  # 在github上创建新的仓库跳过
  # 本地文件夹进行关联
  git remote add origin https://github.com/xxxxx/xxx.github.io.git 
  
  git remote -v # 可以查看关联记录
  # xxx.github.io.git 为仓库名 xxxxx为账户名 直接找到仓库地址添加就OK
  
  git add scaffolds source themes _config.yml # 只需要提交这四个源文件就行
  git commit -am 'first commit'
  git push origin master 
  
  # 源码仓库完成
  # 以后写作只需要提交这四个源文件下面的变动就好了,其他依赖不用管
  ```

* 远程服务器部署

  ```shell
  # 首先创建文件夹
  mkdir xxx.github.io && cd xxx.github.io
  hexo init
  
  # 文件夹下的四类文件删除scaffolds source themes _config.yml
  rm -rf scaffolds source themes _config.yml
  
  # 手动关联远程仓库
  git remote add origin https://github.com/xxxxx/xxx.github.io.git
  git pull origin master  # 默认放到了master
  
  # 后面正常进行编译部署就OK了,并不需要关联_config.yml 里面的deploy,不要绑定仓库
  hexo d 
  hexo g
  hexo s
  
  ```

* 缺点发布需要自己手动, 不能CI/CD, 主要是不需要GitHub托管