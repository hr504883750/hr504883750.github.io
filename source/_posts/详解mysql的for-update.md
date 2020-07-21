---
title: 详解mysql的for-update
date: 2020-07-21 11:45:40
categories: Mysql
tags: 高并发 锁
---

### 详解mysql的for update

#### 背景

for update是在数据库中上锁用的，可以为数据库中的行上一个排它锁。当一个事务的操作未完成时候，其他事务可以读取但是不能写入或更新。

#### for update的使用场景
如果遇到存在高并发并且对于数据的准确性很有要求的场景，是需要了解和使用for update的。

比如涉及到金钱、库存等。一般这些操作都是很长一串并且是开启事务的。如果库存刚开始读的时候是1，而立马另一个进程进行了update将库存更新为0了，而事务还没有结束，会将错的数据一直执行下去，就会有问题。所以需要for upate 进行数据加锁防止高并发时候数据出错。

记住一个原则：一锁二判三更新

#### for update如何使用
使用姿势
```
SELECT * FROM TABLE WHERE xxx FOR UPDATE
```

#### for update的锁表

InnoDB默认是行级别的锁，当有明确指定的主键时候，是行级锁。否则是表级别。

例子: 假设表foods ，存在有id跟name、status三个字段，id是主键，status有索引。

例1: (明确指定主键，并且有此记录，行级锁)

```
SELECT * FROM foods WHERE id=1 FOR UPDATE;
SELECT * FROM foods WHERE id=1 and name=’咖啡色的羊驼’ FOR UPDATE;
```

例2: (明确指定主键/索引，若查无此记录，无锁)
```
SELECT * FROM foods WHERE id=-1 FOR UPDATE;
```

例3:(无主键/索引，表级锁)
```
SELECT * FROM foods WHERE name=’咖啡色的羊驼’ FOR UPDATE;
```

例4:(主键/索引不明确，表级锁)
```
SELECT * FROM foods WHERE id<>’3’ FOR UPDATE;
SELECT * FROM foods WHERE id LIKE ‘3’ FOR UPDATE;
```

#### for update的注意点
1.for update 仅适用于InnoDB，并且必须开启事务，在begin与commit之间才生效。

2.要测试for update的锁表情况，可以利用MySQL的Command Mode，开启二个视窗来做测试。

#### for update的疑问点
当开启一个事务进行for update的时候，另一个事务也有for update的时候会一直等着，直到第一个事务结束吗？
答：会的。除非第一个事务commit或者rollback或者断开连接，第二个事务会立马拿到锁进行后面操作。

如果没查到记录会锁表吗？
```
会的。表级锁时，不管是否查询到记录，都会锁定表。
```


##### for update 和 for update nowait区别（前者阻塞其他事务，后者拒绝其他事务）

`for update`锁住表或者锁住行，只允许当前事务进行操作（读写），其他事务被阻塞，直到当前事务提交或者回滚，被阻塞的事务自动执行
`for update nowait` 锁住表或者锁住行，只允许当前事务进行操作（读写），其他事务被拒绝，事务占据的statement连接也会被断开



