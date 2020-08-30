---
layout: _post
title: Redis 操作记录日记
date: 2020-08-06 11:36:11
tags: redis 运维
categories: redis
---

> 之前偷懒不想总结，老是炒剩饭，现在记录下，帮助自己快速复习用

##### 一些常用的非（CURD）操作指令

```Linux
> redis-cli 
flushall 							# 刷盘 清数据
dbsize                # 查询redis使用占用情况
info                  # 查询启动redis服务所用的配置情况
info Replication      # 查询主从详细信息，info + 配置说明
config get *          # 查询单条配置信息 *可以替换为任何查询字段

```

##### redis 安装

> 自己根据情况选择合适的下载版本，我这边选的是3.0.6的版本

```shell
> cd /usr/local/src
> wget http://download.redis.io.releases/redis-3.0.6.tar.gz
> tar zxvf redis-3.0.6.tar.gz
> cd redis-3.0.6
```

> 对源码编译安装选择合适的安装路径，这边选的安装路径/usr/local/redis

```shell
> cd /usr/local/src/redis-3.0.6
> make
> make install PREFIX=/usr/local/redis
```

> 创建自己习惯的redis使用目录(根据喜好自行选择), 将原生配置模版copy过来

```shell
> mkdir -p /usr/local/redis/{conf,log,data,var}
> cp /usr/local/src/redis-3.0.6/redis.conf /usr/local/redis/conf
```

> 对配置文件进行修改

```shell
> vi /usr/local/redis/conf/redis.conf
conf> daemonize yes		# 调整yes后默认后台启动
conf> pidfile /usr/local/redis/var/redis.pid  # 将这里调整为我们的指定路径
conf> port 6379  			# 端口使用默认
conf> logfile "/usr/local/redis/log/redis.log" # 日志文件存放位置
conf> dbfilename dump.rdb  # 数据文件名
conf> dir /usr/local/redis/data/  # 数据文件存放目录
```

>指定配置文件进行服务启动

```
> /usr/local/redis/bin/redis-server /usr/local/redis/conf/redis.conf
```

> 查看端口占用情况

```
>lsof:6379 # 查看redis服务是否监听端口
```

> 一些重要配置的说明

```
timeout 300   		# 客户端闲置超时时间 默认单位为秒
loglevel verbose  # 四个日志级别 debug verbose notice warning
databases 16   		# 数据库数量 初始连接选择0库 

save 900 1   			# 几个持久化同步数据文件的选项 可配合多个策略
save 300 10     	# 900秒内一个更改，300秒内10个更改，60秒内10000个更改
save 60 10000 

rdbcompression yes  # 存储到本地时数据文件是否压缩, 数据量小CPU处理不够可以关闭

slaveof  <masterip> <masterport>  # 主从配置时的选项，设置当本机为slav服务时, 通过配置好的master地址以及端口, 在slave启动时, 它会自动从master进行同步

maxmemory <bytes>   # 指定Redis最大内存限制, redis启动之后会把数据加载到内存，如果达到最大
值，但未加载完成会清理过期key, 默认0为不限制

appendonly no 		 # 指定是否在每次更新操作后进行日志记录，看数据的重要性，如果一条都不允许丢的话，建议开启

include /parh/to/local.conf  # 指定包含其他的配置文件
```

---

#####  关于RDB的备份恢复

> 优先准备备用副本夹(也可以是云端存储) 定期将RDB文件进行拷贝放入
>
> 备份前
>
> * 停掉redis服务
> * 将redis.conf 中的appendonly 改为 no，主要是防止再启动时重载AOF的持久化文件，会进行覆盖。内存恢复的时候优先读取AOF的持久化文件， 再读取RDB文件，在恢复的时候需要先删除现存的AOF，RDB文件。
> * 将备份的RDB文件放入数据读取文件路径下
> * 重启redis服务
> * 若需要AOF持久化，则需要再启动后再进行配置变更, 将appendonly 改为yes

##### 关于AOF的备份恢复

> 步骤上同RDB恢复, 区别在于appendonly 在启动的时候需要改为yes， 并保持恢复数据文件夹数据为恢复文件，移除非恢复AOF文件，防止数据覆盖。

---

##### 关于redis 结合lua脚本的简要记录

* 简要说明redis处理lua脚本

  > Redis 使用单个的Lua解释器去运行所有脚本，并且Redis也保证脚本会以原子性的方式执行
  >
  > 当某个脚本正在运行的时候，不会有其他脚本或者命令去执行，在别的客户端看来，脚本效
  >
  > 果要么是不可见的，要不就是已经完成的。所以脚本执行的时候尽量不要做sleep缓慢操
  >
  > 作，如果堆积的话，其他客户端会因为等待而无法处理其他脚本或命令请求。
  >
  > 当Redis执行命令的过程中发生错误时，脚本会停止执行，并返回一个脚本错误，错误的输
  >
  > 出信息会说明造成错误的原因。

**EVAL命令的使用**

```shell
> redis-cli --eval path/to/redis.lua KEYS[1] KEYS[2] , ARGV[1] ARGV[2]

# eval 的作用是告诉redis-cli 读取后面的Lua脚本
# path/to/redis.lua  是lua的脚本路径
# KEYS[1] KEYS[2] 是要操作的键，可以指定多个，
# ARGV[1] ARGV[2] 参数， 在lua脚本中通过 ARGV[1] ARGV[2] 获取
# 注意：KEYS 和 ARGV , 中间的空格不能省略
```

> test.lua

```
return {KEYS[1], KEYS[2], ARGV[1], ARGV[2]}
```

```
> redis-cli --eval path/to/test.lua username age , regalbrown 26
< "username"
< "age"
```



**EVALSHA 与脚本**

可以把脚本内容利用缓存存储到redis，返回指定sha1内容值，通过指定的sha1值去调用不同的脚本

```shell l
# 例如现有脚本 test.lua
# 脚本内容
> cat test.lua
< return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}

# 生成sha1值
> /usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 6379 script load test.lua
< "a42059b356c875f0717db19a51f6aaca9ae659ea"

127.0.0.1:6379> evalsha a42059b356c875f0717db19a51f6aaca9ae659ea 2 name age regalbrown 26
1) "name"
2) "age"
3) "regalbrown"
4) "26"

# 等价于
127.0.0.1:6379> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 name age regalbrown 26

```

**script脚本管理**

* 判断脚本是否在内存中
* 清理内存脚本
* 脚本出现阻塞

```shell
127.0.0.1:6379> script exists a42059b356c875f0717db19a51f6aaca9ae659ea
1) (integer) 1
```

