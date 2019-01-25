---
layout: post
title:  "MongoDB 数据操作总结"
date:   2016-05-01 19:37:48 +0800
categories: No SQL
tags: [MongoDB]

---

## 操作MONGODB数据库

1.创建数据库
  语法：use 数据库名

  > 注意：如果数据库不存在则创建数据库，否则切换到指定的数据库

  > 另外：如果刚刚创建的数据库不再列表内，如果要显示它，我们需要向刚刚创建的数据库中插入一些数据

  `db.student.insert({name:”tom”,age:18,gender:1,address:”上海”,isDeleted:0})`

2.删除数据库

> 前提：使用当前数据库(use 数据库名)

  `db.dropDatabase()`

3.查看所有数据库

  `Show dbs`

4.查看当前正在使用的数据库

  `db`

  `db.getName()`

5.断开连接

  `exit`

6.查看命令api帮助

  `help`

## 集合操作

1.查看当前数据库下有哪些集合：`show collections`

2.创建集合：

  - 语法：`db.createCollection(“集合名”) 实例：db.createCollection(“student”)`

  - 语法:`db.student.insert(document)`

> 区别：前者创建一个空的集合，后者创建一个空的集合并添加一个文档

3.删除当前数据库中的集合

  语法：db.集合名.drop()  实例：`db.student.drop()`

## 文档操作

1.插入文档

  - 使用insert()方法插入文档
    语法：db.集合名.insert(文档)
    + 插入一个：`db.student.insert({name:”Jack”,age:18,gender:1,address:”上海”,isDeleted:0})`

    + 插入多个：用逗号分隔,前后加入中括号，即变为列表，语法：db.集合名.insert([文档1,文档2,….,]):

``` mongodb

db.student.insert([{name:”Jack”,age:18,gender:1,address:”上海”,isDeleted:0},
{name:”Rose”,age:18,gender:0,address:”北京”,isDeleted:0}]`
)

```

- 使用save()方法插入文档

  语法：db.集合名.save(文档)
  注意：如果指定_id字段则更新数据否则执行插入数据

实例1：`db.student.save({name:”Jack”,age:18,gender:1,address:”上海”,isDeleted:0})`

实例2：`db.student.save({_id:ObjectId(“55656565656ac565d17”),name:”Jack”,age:18,gender:1,address:”上海”,isDeleted:0})`

2.文档更新

  - update()方法用于更新已存在的文档

    语法：`db.集合名.update([<query>],[<update>],{upset:<boolean>,multi:<boolean>,writeConcern:<document>})`

      参数说明：query是update的查询条件，类似sql的update语句where后面的内容

        Update：update的对象和一些更新的操作符($set,$inc)等，$set直接更新，$inc在原有基础上累加后更新。

        Upset：可选，如果不存在update的记录，是否当新数据插入，true为插入，false为不插入，默认为false

        Multi：可选，mongodb默认为false,只更新找到的第一条记录，如果为true则按照条件查找出来的记录全部更新

        Writeconcern：可选，抛出异常的级别

  以下实例我们将 Jack 的年龄更新为25：

`db.student.update({name:”jack”},{$set:{age:25}})`

`db.student.update({name:”jack”},{$inc:{age:25}}) //累加`

`db.student.update({name:”jack”},{$set:{age:25}},{multi:true}) //全改`


- save()通过传入的文档替换已存在的文档

  语法：`db.集合名.save(<document>,{ Writeconcer)`

3.文档删除

> 说明：在执行remove()函数前，先执行find()命令来判断执行的条件是否存在是一个良好习惯。Remove函数已经过时，不推荐使用，请使用deleteOne和deleteMany方法

语法：
``` mongodb 
db.集合名.remove(Query,  #可选，删除文档的条件
{
	Justone:<boolean>, #为可选ture或1则只删除一个文档
		writeConcern:<document> #可选，抛出异常的级别
})

```

  实例：`db.student.deleteMany({}) #删除所有`

`Db.student.deleteMany({name:”Jack”}) #按照条件删除多条记录`

`Db.student.deleteOne({name:”Jack”}) #删除满足条件的一条记录`

4.文档查询

  - Find()方法：查询机会中所有文档数据 ，语法：`db.集合名.find()`

  - find()方法查询指定列，语法：`db.集合名.find(query,{<key>:1,<key>:1})`

    参数说明：query是查询条件，可以是要显示的列(同时为1或同时为0不允许有交叉设置)，1代表显示，0为不显示

实例：`db.student.find({name:”tom”},{name:1,age:1}) #查询所有则query为{}`

  - pretty()方法以格式化的方式显示文档实例：`db.student.find().pretty()`

  - findOne()方法查询匹配结果的第一条记录实例：`db.student.findOne({gender:0})`


5.查询条件操作符

> 作用：条件操作符用于比较两个表达式并从MongoDb集合中获取数据

|语法|意义|
------------- | --------
|$gt|大于|
|$gte|大于等于|
|$lte|小于等于|
|:|等于|
|$lt|小于|

实例：`db.student.find({age:{$gt:20}})`

`db.student.find({age:{$gte:10,$lte:50}})  #并且关系：大于等于且小于等于`

`db.student.find({“_id”:ObjectId(“Id值”)}) #使用id进行查询`

`db.student.find().count() #查询某个结果集的数据条数`

`db.student.find({name:/ile/})#查询某个字段的值是否包含另一个值`

`db.student.find({name:/^li/})字段的值是否以另一个值开头`


6.条件查询 and 和or

  - AND 条件：`db.student.find({gender：0,age:{$gt:16}})`

  - OR 条件：`db.student.find({$or:[{age:17},{age:{$gte:20}}]})`

  - AND 和OR联合使用：`db.student.find({条件1，条件2，$or:[{条件3},{条件4}]})`


7.Limit /skip

  - limit():读取指定数量的数据记录:` db.student.find().limit(3)`

  - skip():跳过指定数量的数据：`db.students.find().skip(3)`

  - limit和skip联合使用实现分页：`db.students.find().skip(3).limit(3)`


8.排序

  语法：`db.集合名.find().sort({<key>:1|-1})`
    参数：1是升序，-1是降序

实例：`db.student.find().sort({age:1})`
9.创建索引

> 说明：索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构

  语法：`db.student.createIndex(keys, options)`
    参数：Key 要创建的索引字段，1 为指定按升序创建索引，按降序来创建索引指定为 -1,options中的background参数为true则创建在后台执行

实例：`db.student.createIndex({name:1,age:-1}, {background: true})`