# Mongodb 



## Mongodb 介绍



### 什么是 NoSql



#### 概念



Not only SQL，即不仅仅是 SQL。非关系型数据库，以键值对（key-value）形式存储。



#### NoSql vs 传统关系型数据库



非结构型数据库：没有行、列的概念，用 JSON 来存储数据；**集合相当于表，文档相当于行**。

非关系型数据库，以键值对（key-value）形式存储。关系型数据库采用结构化的数据。

在处理半结构化/非结构化的大数据，优先考虑 No-SQL；

在考虑数据库的成熟度、管理及商业化应用优先考虑关系型数据库。



### 什么是 MongoDB



MongoDB 是由 C++ 语言编写的，是一个面向文档存储的数据库。

Mongodb 最大的特点是支持的查询语言非常强大，支持对数据建立索引以实现更快的排序。

它的特点是高性能、易部署、易使用，存储数据非常方便。





## MongoDB 增删改查



### 一、连接数据库



~~~js
// 建立连接
mongo
// 清屏
cls
// 查看所有数据库
show dbs
// 使用指定数据库
use database-name
// 查看当前数据库集合列表
show collections
~~~



### 二、创建 查看 删除



1. 使用数据库

~~~js
use database-name
~~~

如果真的想把这个数据库创建成功，那么必须插入一个数据。 

数据库中不能直接插入数据，只能往集合 collections 中插入数据。



2. 插入数据

~~~js
// 插入一条
db.base-name.insert({"key": "value"})
// 插入多条
db.base-name.insert([{key1: value1},{key2: value2}])
~~~

随着插入数据，集合创建成功了，数据库也创建成功了。



3. 查看当前集合（mysql称为表）

 ~~~js
show collections
 ~~~



4. 删除集合

~~~js
db.base-name.drop()
show collections
~~~



5. 删除当前所在数据库

~~~js
db.dropDatabase()
~~~



### 三、查询数据



1. 查询所有记录

~~~js
db.base.find()
~~~



2. 查询集合中某一列的数据，结果以数组形式展示

~~~js
db.base.distinct("name")
~~~



3. 条件查询

~~~js
// 查询 age = 20 的数据
db.base.find({age: 20})

// age > 22
db.base.find({age: {$gt: 22}})

// age < 22
db.base.find({age: {$lt: 22}})

// age >= 25
db.base.find({age: {$gte: 25}})

// age <= 25
db.base.find({age: {$lte: 25}})

// age >= 23 且 age <=26
db.base.find({age: {$gte: 23, $lte: 26}})

...

// 指定字段名称查询：name = ming, age = 22
db.base.find({name: 'ming', age: 22})

// 查询 name 中包含 mongo 的数据 (模糊查询用于搜索)
db.base.find({name: /mongo/})

// 查询 name 以 mongo 开头的数据
db.base.find({name: /^mongo/})

// 查询指定列 name age 的数据，也可用 true/false 表示，false 意味着排除
db.base.find({}, {name:1, age:1})

// 查询指定的列 name age，且 age > 25；前面是条件，后面是指定的字段
db.base.find({age: {$gt: 25}}, {name: 1, age: 1 })

...

// 排序 升序1 降序-1
// 只返回username、age两个列的数据 并按照年龄升序
db.base.find({}, {username: 1, age: 1}).sort({age: 1})

// 按照年龄降序
db.base.find().sort({age: -1})

...

// 查询第1条数据
db.base.findOne()
db.base.find().limit(1)

// 查询前 5 条数据
db.base.find().limit(5)

// 查询 10 条后的数据
db.base.find().skip(10)

// 查询 5-10 之间的数据
db.base.find().skip(5).limit(5)
db.base.find().limit(10).skip(5)

...

// or: 查询 age=12 或 ae=22 的数据
db.base.find({$or: [{age: 12}, {age:22}]})

...

// 查询结果集数据的总数
db.base.find().count()
~~~



### 四、修改数据



修改语句里有查询条件，即告诉数据库你要修改谁，怎么改。

~~~js
// 查找名字为小明的数据，将他的年龄改为16岁 (注意：只会修改匹配到的第一条数据)
db.base.update({name: "小明"}, {$set: {age: 16}})

// 查询数学成绩为 0 的学生，将他的状态改为 warn
db.base.update({score.math: 0}, {$set: {status: 'warn'}})

// 更改所有匹配的数据
db.base.update({name: "小明"}, {$set: {age: 16}}, {multi: true})

// 不使用 $set 修改，则相当于完整替换数据
db.base.update({name: 'ming'}, {name: 'update_ming', age: 21})
~~~



### 五、删除数据



~~~js
// 删除 name = ming 的所有数据
db.base.remove({name: 'ming'})
~~~





## 索引查询



索引是对数据库表中一列或多列的值进行排序的一种结构，可以让数据库查询速度变得更快。MongoDB 的索引几乎与传统的关系型数据库一模一样，这其中也包括一些基本的查询优化技巧。



### 索引创建/查看/删除



1. 创建索引

~~~js
db.base.ensureIndex({name: 1})
~~~



2. 获取当前集合的索引

~~~js
db.base.getIndexes()
~~~



3. 删除索引

~~~js
db.base.dropIndex({name: 1})
~~~



### 复合索引



数字 1 表示 name 键的索引按升序存储，-1 表示 age 键的索引按降序方式存储。



~~~js
db.base.ensureIndex({name: 1, age: -1})
~~~

该索引被创建后，基于 name 和 age 的查询将会用到该索引，或者是基于 name 的查询也会用到该索引；

但基于 age 的查询将不会用到该复合索引；

因此，如果想用到复合索引，**必须在查询条件中包含复合索引中的前 N 个索引列**。



### 唯一索引



在缺省情况下创建的索引均不是唯一索引。下面的示例将创建唯一索引：

~~~js
db.user.ensureIndex({userid: 1}, {unique: true})
~~~



如果再次插入 userid 重复的文档时，MongoDB 将报错，以提示插入重复键；

如果插入的文档中不包含 userid 键，那么该文档中该键的值为 null。

~~~js
db.user.insert({"userid": 5})
db.user.insert({"userid": 5})
~~~



### 使用 explain



explain 是非常有用的工具，会帮助你获得查询方面诸多有用的信息。只要对游标调用该方法，就可以得到查询细节。explain 会返回一个文档，而不是游标本身。

~~~js
db.base.find({name: 'ming'}).explain()
~~~



查询具体执行时间

~~~js
db.base.find().explain("executionStats")
~~~



## 账户权限配置



### 超级管理用户的创建



1. 创建超级管理用户

~~~js
use admin
db.createUser({
    user: 'admin',
    pwd: '123456',
    roles: [{role: 'root', db: 'admin'}]
})
~~~



2. 修改 mongodb 配置文件 mongod.cfg

~~~js
security: authorization: enabled
~~~



3. 重启 mongodb 服务



4. 用超级管理员账户连接数据库

~~~js
mongo admin -u '用户名' -p '密码'

mongo 192.168.1.200:27017/test -u user -p password
~~~



5. 给创建的数据库 eggcms 创建一个用户，只能访问 eggcms，不能访问其他

~~~js
use eggcms
db.createUser(
    {
    	user: "eggadmin", 
        pwd: "123456", 
        roles: [ { role: "dbOwner", db: "eggcms" } ]
    }
)
~~~



### Mongodb 配置常用命令



~~~js
// 查看所有用户
show users

// 删除用户
db.dropUser("egg_admin")

// 修改用户密码
db.updateUser("admin", {pwd: "new_password"})

// 密码认证
db.auth("amdin", "password")
~~~



### mongodb 数据库角色



- 数据库用户角色：read、readWrite;
- 数据库管理角色：dbAdmin、dbOwner、userAdmin； 
- 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager； 
- 备份恢复角色：backup、restore； 
- 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、 dbAdminAnyDatabase 
- 超级用户角色：root



## 高级查询



使用聚合管道可以对集合中的文档进行变换和组合。

使用场景：表关联查询、数据统计。



### 管道操作符与表达式



**操作符**

- $project：筛选字段 
- $match：条件匹配，只满足条件的文档才能进入下 一阶段 
- $limit：限制结果的数量 
- $skip：跳过文档的数量 
- $sort：条件排序
- $group：条件组合结果 统计 
- $lookup：用以引入其它集合的数 据 （表关联查询



**常用表达式**

$addToSet：将文档指定字段的值去重 

$max：文档指定字段的最大值 

$min：文档指定字段的最小值 

$sum：文档指定字段求和 

$avg：文档指定字段求平均 

$gt：大于给定值 

$lt：小于给定值 

$eq：等于给定值



### aggregate 聚合管道



1. $match：用于过滤文档，类似 find()

~~~js
// 查找出 all_price >= 90 的数据，且只显示 trade_no、all_price 字段 
db.order.aggregate([
    {
    	$project:{ trade_no:1, all_price:1 }
    }, {
    	$match:{"all_price":{$gte:90}}
    }
])
~~~



2. $group：将集合中的文档进行分组，可用于统计结果

~~~js
// 统计每个订单的总数，按订单号分组
db.order_item.aggregate([
    {
    	$group: {_id: "$order_id", total: {$sum: "$num"}}
    }
])
~~~



3. $sort：将集合中的文档进行排序，1升序，2降序

~~~js
db.order.aggregate([
    {
        $project:{ trade_no:1, all_price:1 }
    }, {
        $match:{"all_price":{$gte:90}}
    }, {
        $sort:{"all_price":-1}
    }
])
~~~



4. $limit：限制返回数据条目
5. $skip：跳过多少个数据后返回最终结果

6. $lookup：表关联

~~~js
db.order.aggregate([
    {
    	$lookup:{
        	from: "order_item", // 关联表名称
        	localField: "order_id", // 源数据表关联字段
            foreignField: "order_id",  // 外部表关联字段
            as: "items" // 将外部表关联数据放在 items 字段下
        }
    },
    ...
])
~~~



## 备份与还原



### 数据库导出备份



注意：是在新建的终端操作 而不是在连接数据库后操作

~~~js
mongodump -h dbhost -d dbname -o dbdirectory
~~~

dbhost：所在服务器地址

dbname：需要备份的数据库

dbdirectory：备份的数据存放路径



### 数据库恢复导入



注意：是在新建的终端操作 而不是在连接数据库后操作

~~~js
mongorestore -h dbhost -d dbname dbdirectory
~~~

dbname：需要恢复的数据库实例

dbdirectory：备份数据路径



有用户名密码认证需要参考如下命令：

~~~js
mongodump -h localhost:27017 -d test -u admin -p 123456 -o ...
mongorestore -h localhost:27017 -d test -c order --dir ....
~~~

