---
layout: post
title:  "Redis 数据操作总结"
date:   2017-01-01 18:37:48 +0800
categories: technology
tags: [Redis,No SQL]

---

## String
----
> 概述：string是redis的最基本类型，最大能存储512MB，string类型是类型安全的。既可以存储任何数据、比如数字、图片、序列号对象等。

1.设置
- 存键值-Set key value：`set x "hello,world"`
- 存键值并设置其过期时间，以秒为单位：`setex c 10  good`
- 存多个键值:mset key value [key value…]：`mset a 1 b 2`

2.获取
- 根据键获取值，如果键不存在则返回None(null 0 nil)：`get key`
- 根据多个键获取多个值：`mget a b d`

3.运算
- 值是数字类型的字符串，将key对应的值加1：`incr key`；减一：`decr key`
- 将key对应的加整数：`incrby key intnum` ；减整数：`decrby key intnum`

4.其他
- 在指定的键上追加值，如该键不存在则新建:`append key value`
- 获取值的长度：`strlen key`

## Key
----

1.查找键，参数支持正则:`keys pattern`

2.判断键是否存在，存在返回1，否则返回0:`exists key`

3.查看键对应的值类型:`type key`

4.删除键及对应的值:`del key [key….]`

5.设置过期时间，以秒为单位:`expire key seconds`

6.查看有效时间，以秒为单位:`ttl key`

## Hash
----
> 概述：hash用于存储对象如：{name：”tom”,age:18}

1.设置
- 设置单个值hset key field value：`hset p1 name tom;hset p1 age 18`
- 设置多个值`hmset key field value [field value…]` 

2.获取
- 获取一个属性的值:hget key field：`hget p1 name`
- 获取多个属性的值`hmget key field [field…]`
- 获取所有属性和值`hgetall key`
- 获取所有的属性`hkeys key`
- 获取所有的值`hvals key`
- 返回包含属性的个数`hlen key`

3.其他
- 判断属性是否存在，没有返回0，存在返回1`hexists key field`
- 删除属性及值`hdel key field [field…]`
- 返回值的字符串长度`hstrlen key field`

## List
----
> 概述：列表的元素的类型为string，按照插入的顺序排序，在列表的头部或尾部添加元素，本质上可以说是一个队列

1．设置
- 在头部插入`lpush key value [value…]`
- 在尾部插入`rpush key value [value…]`

- 在一个元素的前或后插入新元素`linsert key before|after pivot value`

- 设置指定索引的元素值，index从0开始，可以是负数表示偏移量是从list尾部开始，如-1表示最后一个元素`lset key index value`

2．获取 
- 移除并返回key对应的list的第一个元素：`lpop key`
- 移除并返回key对应的list的最后一个元素：`rpop key`
- 返回key对应的列表中的指定范围的元素，start和end都是从0开始，偏移量可以是负数：`lrange key start end `

3．其他
- 裁剪列表，改为原列表的一个子集`ltrim key start end`
- 返回存储在key里的list的长度`llen key`
- 返回列表中索引对应的值`lindex key index`

## Set
----
> 概述：无序集合，元素类型为string类型，元素具有唯一性，不重复

1.设置
- 添加元素:sadd key member [member…]: `sadd s1 1 2 3 4`

2.获取
- 返回key集合中所有的元素:`smembers key`
- 返回key对应的集合元素个数:`scard key`

3.其他
- 求多个集合的交集:`sinter key [key…]`
- 求多个集合的差集:`sdiff key [key…]`
- 求多个集合的合集:`sunion key [key…]`
- 判断元素是否在集合中，存在返回1，没有返回0:`sismember key member`

## Zset
----
> 概述：（1）有序集合，元素类型为string，元素具有唯一性，不能重复。

（2）每个元素都有一个关联的double类型的score（表示权重），通过权重的大小排序，元素的score可以相同

1.设置
- 添加元素:`zadd key score member [score member]`

2.获取
- 返回指定范围内的元素:`zrange key start end`
- 返回元素个数:`zcard key`
- 返回有序集合key中，score在min 和max之间的元素个数:`zcount key min max`
- 返回有序计划key中，成员member的score值:`zscore key member`
