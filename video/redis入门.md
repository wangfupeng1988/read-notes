# 《redis入门》视频教程学习笔记

-----------

## 前言

作为前端工程师，日常开发并不太多接触 redis 。但是想要全面发展就得眼观六路，通用的软件都要了解。此前也零零散散看过一些博客介绍，不过没有拿出专门时间演练过，本次就做一个演练的总结。

也推荐其他想要了解 redis 的同仁一起看这个视频教程，比看书效率高很多，地址 https://www.imooc.com/learn/839

当然了，如果你日常用 redis 频率很高或者想深入研究，看这种介绍视频肯定是不够的，老老实实买本几书仔细研究吧。

----------

## mac os 安装 redis

使用 homebrew 可以很简单的安装，首先要安装 homebrew 以及更改镜像源（下载快），具体方法参见我写的[与mysql零距离接触]()与mysql零距离接触.md)中的内容。

- 执行`brew install redis`，很快就能安装完成
- 执行`redis-server &`（后台运行）启动服务
- 执行`ps aux | grep redis`可查看服务是否运行
- 执行`redis-cli`可启动客户端，可执行`set name xxx`测试
- 退出客户端，执行`redis-cli shutdonwn`可关闭服务

-----------

## NoSQL

首先，NoSQL 表示的是 Not only SQL ，并不是`no` `sql`，概念要搞清楚。与 NoSQL 对应的就是常见的关系型数据库，例如 mysql 。关系型数据库需要先定义表结构，然后按照表结构的要求存储数据，而 NoSQL 数据库就没有数据表，只需要按照规定的数据格式，讲数据直接写入即可。

正如视频中提到的，这种存储方式更加灵活，对于目前互联网大数据、复杂数据的处理和扩展有很大的好处。

-----------

## redis 概述

redis 首先肯定是一个 NoSQL 的数据库，其次最重要的它是一个内存数据库，相比于硬盘数据库有天生的性能优势，因此也被用来做缓存数据库使用。当然，它也有成熟的数据持久化方案，课程最后讲到了。

-----------

## redis 数据结构

### string

这是最基本的数据类型，也是项目中最常用的。基础的命令就是`set key val`和`get key`，例如

```
set name wangfupeng
get name
```

也许你能看出来，其他数据库或者编程语言都会有数字类型，为何 redis 这里没有呢？这个问题我没有深入的看，视频中也没有专门讲解。我当前的理解是，redis 没有专门的数字类型、都是 string 类型，但是它却能识别出数字类型然后进行操作。例如`incr` `decr` `incrby` `decrby`等操作。

```
set age 30
get age
INCR age
DECR age
INCRBY age 5
DECRBY age 5
```

### hash

hash 类型就可以理解为 JS 中的 object ，即一个键值对的组合，是一种无序的数据结构。一个 hash 中可以包含很多很多个键，多到你日常使用可以忽略。

基本的使用命令有`hset` `hget` `hmset` `hgetall`等（前端都带有`h`，就是`hash`的简写），示例如下：

```
hset userinfo name wangfupeng
hset userinfo age 30
hmset userinfo otherkey1 value1 otherkey2 value2
hget userinfo name
hgetall userinfo
hdel userinfo k2
hexits userinfo k2
hlen userifo
hkeys userinfo
hvals userinfo
```
其他命令可查询相关文档

### list

list 是一个按照插入顺序排序的数据集合，就是一个链表，也相当于 JS 中的数组。redis 操作 list 的命令都以`l`开头，如`lpush` `lrange` `lpop`等。命令演示如下：

```
lpush mylist 1 2 3
llen mylist
lpop mylist
lrange mylist 0 2
lpushx mylist x
rpushx mylist y
```
其他命令可查询相关文档

### set

set 和 ES6 中的 set 概念基本一样，也是一个按照插入顺序排序的字符集合，只不过不能重复。它不是一个数组或者 list ，因为后者是允许元素相互重复的。

其实可以参考 hash 或者 JS object 来理解。hash 中 key 是不能重复的，set 中元素不能重复，这是一个概念。只不过，set 是一个 key-val 集合，而 set 只是一个 key 集合，这个区别。

set 的使用场景是很多的，因为去重操作就用到项目的各个地方。第一，如果要存储一些唯一 ID 的集合（如身份证号、车牌号等），用 set 合适。第二，多个 set 之间可以进行集合的并集、交集、差异的查找，而数组就不行，因为数组没有去重。

set 命令都带有`s`开头，常用命令如下：

```
sadd myset a b c
sadd myset c d e
smembers myset
sismember myset a

sdiff myset1 myset2
sinter myset1 myset2
sunion myset1 myset2
```

其他命令查询相关文档即可

### sorted set

顾名思义，就是经过大小排序的 set 集合，有序集合在查找的时候是非常快的，二分查找即可，时间复杂度 O(logN) 。在一些需要排名、排序的场景中可以应用。

既然是按照大小排序，在添加元素时肯定要给一个可依据排序的数字，例如要添加`a` `b` `c`三个元素进去，使用的命令就是`zadd 10 a 20 b 30 c`。

----------

## 事务

事务是一个数据库必须具备的功能，redis 肯定也支持。具体操作，通过`multi`命令开启事务，然后输入一系列命令（不会立即执行，而是入队列），最后通过`exec`命令来提交事务（提交之后所有命令才被统一执行）。执行`discard`回滚事务。

```
multi
incr num
incr num
incr num
exec
```
----------

## 持久化

redis 是一个内存数据库，因此速度很快。但是内存的数据毕竟是不稳定的，因此要持久化到硬盘中。redis 提供了 RDB 和 AOF 两种持久化方式，两者可分开使用也可结合使用。

- RDB，默认支持，按照间隔时间将内存快照写入硬盘
- AOF, 需要自行配置，记录 redis 的操作日志，redis 启动时会根据记录的日志重新加载数据

---------

## 总结

通过看视频，以及自己实际安装了 redis 敲了一些命令，算是了解 redis 的一个不错的过程。比看那些零散的博客效果好很多。

