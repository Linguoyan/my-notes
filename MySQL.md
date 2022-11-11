# MySQL 



## MySQL 简介



数据库：专门用于存放数据地方。sqlServer，mysql，sqlite。

数据库分类：关系型数据库（mysql)，非关系型数据库（nosql,mongodb）,图谱数据库（大数据建立知识图谱）



### MySQL 下载与安装



1. Mysql下载：https://dev.mysql.com/downloads/

2. 选择 MySQL Community Server
3. 下载页面：https://dev.mysql.com/downloads/windows/installer/8.0.html
4. 安装MySQL
   1. 仅安装server-only；
   2. 选择mysql.5x密码验证
5. 测试是否安装成功
   1. 打开mysql8.0 cline client
   2. 输入账号密码能够进入数据库
6. 安装navicat



### MySQL 基本命令



1. 查看数据库

~~~js
show databases
~~~



2. 使用数据库

~~~js
use database-name
~~~



## Nodejs 连接 MySQL



### 一、连接数据库

~~~js
const mysql = require('mysql');

// 创建与数据库连接的对象
const con = mysql.createConnection({
  host     : 'localhost', // 主机名
  port     : '3306', // 端口 默认3306
  user     : 'root', // 用户
  password : '123456', // 密码
  database : 'test', // 连接的数据库名
});
 
// 建立连接
con.connect((err) => {
  if (err) {
    console.log("连接失败");
  } else {
    console.log("连接成功");
  }
});
~~~



### 二、增删改查



1. 查询数据

~~~js
// 查询数据表 user 中的所有数据
const sql = "SELECT * FROM user";
con.query(sql, function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err.message);
    return;
  }
  console.log(result);
});
~~~



2. 创建库

~~~js
// 创建数据库 user
const sql = "create database user";
con.query(sql, function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err.message);
    return;
  }
  console.log(result);
});
~~~



3. 删除库

~~~js
// 删除数据库 user
const sql = "drop database user";
con.query(sql, function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err.message);
    return;
  }
  console.log(result);
});
~~~



4. 创建表

~~~js
// 创建数据表 user
const sql = 'CREATE TABLE `user` (id INT(11), name VARCHAR(25)...)'
con.query(sql, function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err);
    return;
  }
  console.log(result);
});
~~~



5. 删除表

~~~js
// 删除数据表 user
const sql = "drop table user";
con.query(sql, function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err.message);
    return;
  }
  console.log(result);
});
~~~



6. 插入数据

~~~js
// 向数据表 user 插入数据
const sql =
  'insert into user (id, username, password) values (1, "ming", "123456")';
con.query(sql, function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err);
    return;
  }
  console.log(result);
});

// 不指定id id会自动递增
const sql =
  'insert into user (username, password, email) values ("dong", "123456", "dong@qq.com")';
con.query(sql, function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err);
    return;
  }
  console.log(result);
});

// 另一种写法
const sql = "insert into user (username, password, email) values (?,?,?)";
con.query(sql, ["hong", "123456", "hong@qq.com"], function (err, result) {
  if (err) {
    console.log("[SELECT ERROR] - ", err);
    return;
  }
  console.log(result);
});
~~~



7. 更新数据

~~~js
var modSql = 'UPDATE user SET name = ?,address = ? WHERE Id = ?';
var modSqlParams = ['xiaohong', 'xiamen', 6];
connection.query(modSql, modSqlParams, function (err, result) {
   if(err){
         console.log('[UPDATE ERROR] - ',err.message);
         return;
   }        
  console.log('UPDATE affectedRows',result.affectedRows);
});
~~~



8. 删除数据

~~~js
// 删除数据表中 id=6 的数据
var delSql = 'DELETE FROM user where id=6';
connection.query(delSql,function (err, result) {
    if(err){
        console.log('[DELETE ERROR] - ',err.message);
        return;
    }        
 
    console.log('DELETE affectedRows',result.affectedRows);
});
~~~



## 数据库基本操作



### 一、条件查询(where)



1. 使用 **where** 对表中的数据筛选，获取结果为 true 的数据

```
select * from 表名 where 条件;
```



2. 比较运算符：等于**=**、大于**>**、大于等于**>=**、小于**<**、小于等于**<=**、不等于**!=**或**<>**

```mysql
-- 查询年龄大于12岁的学生
select * from students where age > 12;

-- 查询年龄不大于20的学生
select * from students where age <= 20;

-- 查询家庭住址不是“福州”的学生
select * from students where address != '福州';

-- 查询没被删除的学生
select * from students where isdelete = 0;
```



3. 逻辑运算符：**and or not**

```mysql
-- 查询编号大于3的女同学
select * from students where id>3 and gender=0;

-- 查询编号小于4或没被删除的学生
select * from students where id<4 or isdelete=0;
```



4. 模糊查询：**like**、**%**(表示任意多个任意字符)、**_**(表示一个任意字符)

```mysql
-- 查询书名第一个字是"大"的书
select * from book where sname like '大%';

-- 查询书名第一个字是"大"且书名只有两个字的书
select * from book where sname like '大_';

-- 查询书名第一个字是"黄"或叫书名包含"靖"的书
select * from book where sname like '黄%' or sname like '%靖%';
```



5. 范围查询：**in** (表示在一个非连续的范围内)、**between ... and ...**(表示在一个连续的范围内)

```mysql
-- 查询编号是1或3或8的学生
select * from students where id in(1,3,8);

-- 查询学号是3至8的学生
select * from students where id between 3 and 8;

-- 查询学生是3至8的男生
select * from students where id between 3 and 8 and gender=1;
```



6. 空判断

- 注意：null 与 "" 是不同的

```mysql
-- 查询没有填写地址的学生 判空 is null
select * from students where hometown is null;

-- 查询填写了地址的学生 判非空 is not null
select * from students where hometown is not null;

-- 查询填写了地址的女生
select * from students where hometown is not null and gender=0;
```



7. 优先级



- 排序：小括号，not，比较运算符，逻辑运算符
- and 比 or 优先，如果同时出现并希望先算 or，需要结合()使用



### 二、排序(order by)



- **asc** 从小到大排列，即升序
- **desc** 从大到小排序，即降序



~~~mysql
-- 将行数据按照列1进行排序，如果某些行列1的值相同时，则按照列2排序，以此类推...
select * from database_name order by column1 asc|desc, column2 asc|desc,...

-- 查询所有男生的信息，并按学号降序
select * from students where gender=1 order by id desc;

-- 查询姓"李"的所有学生，按名称升序
select * from students where name like "李%" order by name;
~~~



### 三、聚合



1. **count**(*) ：表示计算总行数，括号中可以是 * 与列名，结果是相同的

~~~mysql
-- 查询学生总数
select count(*) from students;
~~~



2. **max**(列)：表示求此列的最大值

~~~mysql
-- 查询女生的编号最大值
select max(id) from students where gender = 0;
~~~



3. **min**(列)：表示求此列的最小值

~~~mysql
-- 查询男学生年龄中的最小值
select min(age) from students where isdelete=0;
~~~



4. **sum**(列)：表示求此列的和

~~~mysql
-- 查询男生分数的总和
select sum(score) from students where gender=1;
~~~



5. **avg**(列)：表示求此列的平均值

~~~mysql
-- 查询班级学生数学考试的平均分
select avg(score) from students where subject = "math";
~~~



### 四、分组(group by)



将查询结果按照1个或多个字段进行分组，字段值相同的为一组。一般来说，你按照什么分组，就只查询什么东西。



基本语法 **GROUP BY** 

~~~mysql
-- 根据 gender 字段来分组，gender 字段的全部值只有两个('男'和'女')
-- 当 group by 单独使用时，只显示出每一组的第一条记录，所以 group by 单独使用时的实际意义不大
SELECT gender FROM employee GROUP BY gender;
~~~



对分组后的结果查询：**group_concat()**

~~~mysql
-- 将职员表，按照部门分组，查询每个部门职员的姓名
select department, group_concat(name) from employee group by department;
~~~



GROUP BY + 聚合函数

~~~mysql
-- 将职员表 按照部门分组 查询每个部门职员的薪水 和 薪水总数
select department,group_concat(salary),sum(salary) from employee group by department;

-- 查询每个部门的名称 以及 每个部门的人数
select department,group_concat(name),count(*) from employee group by department;

-- 查询每个部门的部门名称 以及 每个部门工资大于1500的人数
-- 先把大于1500的人查出来 再做分组
select department,group_concat(salary),count(*) from employee group by department;
~~~



group by + having

- 用来分组**查询后**制定一些条件来输出查询结果

- having 的作用和 where 一样，但 **having 只能用于 group by**

- having 和 where 的区别：

  - having 是在分组后对数据进行过滤，where 是在分组前对数据进行过滤；
  - having 后面可以使用聚合函数（统计函数)，where 后面不可以使用分组函数；

  

~~~mysql
-- 查询工资总和大于9000的部门名称以及工资和
SELECT department, GROUP_CONCAT(salary), SUM(salary) FROM employee GROUP BY department HAVING SUM(salary) > 9000;
~~~



### 五、连接



使用 JOIN 在两个或多个表中查询数据。



JOIN 按照功能大致分为如下三类：

- **INNER JOIN（内连接,或等值连接）**：获取两个表中字段匹配关系的记录。
- **LEFT JOIN（左连接）：**获取左表所有记录，即使右表没有对应匹配的记录。
- **RIGHT JOIN（右连接）：** 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。



**INNER JOIN...ON**

~~~mysql
SELECT a.id, a.category, b.title FROM book a INNER JOIN category b ON a.category_id = b.id;

-- 使用 WHERE 等价
SELECT a.id, a.title, b.category FROM book a, category b WHERE a.category_id = b.id
~~~





