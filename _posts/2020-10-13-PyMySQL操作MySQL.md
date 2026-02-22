---
layout: post
title: 'PyMySQL操作MySQL'
subtitle: 'Python操作数据库'
date: 2020-10-13
categories: tech
author: yucol


tags: Python PyMySQL MySQL 数据库
music-id: 536870202
---

PyMySQL是在 Python3.x 版本中用于连接 MySQL 服务器的一个库，可以简单地操作MySQL数据库。安装方法：```pip install pymysql```，使用方法```import pymysql```。使用之前需要先安装并配置好MySQL，具体方法可以参考[网络教程](https://www.runoob.com/mysql/mysql-install.html)。

> python及sql代码地址：[点我](https://github.com/JMbaozi/absorb/tree/master/program/python_use_mysql)

以下操作内容为学生信息数据库data_school中的class_data表，表建立方式：
```sql
CREATE TABLE `class_data`(
    `id` INT UNSIGNED auto_increment,
    `number` int(10) not null,
    `name` VARCHAR(50) not NULL,
    `class_name` varchar(50) not NULL,
    PRIMARY KEY(`id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
数据库内容：
```
+----+--------+------+------------+
| id | number | name | class_name |
+----+--------+------+------------+
|  1 |   1001 | 张三 | MySql      |
|  2 |   1002 | 李四 | MySql      |
|  3 |   1003 | 王五 | MySql      |
|  4 |   1003 | 王五 | Python     |
|  6 |   1004 | 赵六 | Python     |
+----+--------+------+------------+
```

#### 连接数据库

```python
def connect_mysql():
    db = pymysql.connect(host="localhost",user="root",password="baozi",database="data_school",port=3306)
    cursor = db.cursor()
    cursor.execute("select * from class_data;")
    data = cursor.fetchall()
    print(f"data:{data}")
    # print("data:",data)
    db.close()
```

|connect()方法的常用参数|
|------------------------------------------------|
| host：数据库服务器所在的主机。|
|user：已登录身份登录的用户名。|
|password：要使用的密码。|
|database：用使用的数据库，设置为None，则表示不使用特定的数据库。|
|port：要使用的MySQL端口，默认为3306|

|其他常用方法|
|-------------------|
|close()：发出退出消息并关闭套接字。|
|commit()：提交更改到稳定存储。|
|cursor()：创建一个新游标以执行查询。|
|rollback()：回滚当前事务。|

#### 插入操作

```python
def insert_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = """
        INSERT INTO class_data(number,name,class_name)
        VALUES(1004,"赵六","Python");"""
    try:
        cursor.execute(sql) #执行SQL语句
        db.commit() #提交到数据库执行
        print("插入记录完成")
    except err:
        print("插入记录错误")
        db.rollback()   #发生错误时回滚
    db.close()
```

#### 查询操作

```python
def select_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    s_number = 1003
    sql = "select * from class_data where number = {}".format(s_number)
    try:
        cursor.execute(sql)
        data = cursor.fetchall()
        print(data)
    except err:
        print("查询错误")
    db.close()
```

#### 更新操作

```python
#更新操作
def update_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = "update class_data set class_name = '{}' where class_name = '{}'".format("MySql","Mysql")
    try:
        cursor.execute(sql)
        db.commit()
        print("更新完成")
    except err:
        db.rollback()
        print("更新失败")
    db.close()
```

#### 删除操作

```python
#删除操作
def delete_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = "delete from class_data where name='{}'".format("test")
    try:
        cursor.execute(sql)
        db.commit()
        print("删除成功")
    except err:
        db.rollback()
        print("删除失败")
    db.close()
```

#### 创建新表（学生地址）

```python
def create_table():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = """
        create table st_addr(
            id int unsigned auto_increment,
            number int(10) not null,
            address varchar(100) not null,
            primary key(id)
            )ENGINE=InnoDB DEFAULT CHARSET=UTF8"""
    cursor.execute(sql)
    print("创建新表成功")
    db.close()
```
数据库内容：
```
+----+--------+------------+
| id | number | address    |
+----+--------+------------+
|  1 |   1001 | 艾欧尼亚    |
|  2 |   1002 | 弗雷尔卓德  |
|  3 |   1003 | 德玛西亚    |
|  4 |   1004 | 祖安       |
+----+--------+------------+
```

#### 查询两张表

```python
def query_tables(num):
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = "select a.number,a.name,b.address from class_data a "\
        "join st_addr b on a.number=b.number and a.number={}".format(num)
    try:
        cursor.execute(sql)
        results = cursor.fetchall()
        for r in results:
            number = r[0]
            name = r[1]
            address = r[2]
            print(f"学号为{num}的学生信息：number = {number},name = {name},address = {address}")
        print("查询完毕")
    except err:
        print("查询失败")
    db.close()
```

#### 完整代码

```python
from os import name
import pymysql
from pymysql import cursors
from pymysql import err


#连接数据库
def connect_mysql():
    db = pymysql.connect("localhost","root","baozi","data_school",3306)
    cursor = db.cursor()
    cursor.execute("select * from class_data;")
    data = cursor.fetchall()
    print(f"data:{data}")
    # print("data:",data)
    db.close()

#插入操作
def insert_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = """
        INSERT INTO class_data(number,name,class_name)
        VALUES(1004,"赵六","Python");"""
    try:
        cursor.execute(sql) #执行SQL语句
        db.commit() #提交到数据库执行
        print("插入记录完成")
    except err:
        print("插入记录错误")
        db.rollback()   #发生错误时回滚
    db.close()


#查询操作
def select_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    s_number = 1003
    sql = "select * from class_data where number = {}".format(s_number)
    try:
        cursor.execute(sql)
        data = cursor.fetchall()
        print(data)
    except err:
        print("查询错误")
    db.close()

#更新操作
def update_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = "update class_data set class_name = '{}' where class_name = '{}'".format("MySql","Mysql")
    try:
        cursor.execute(sql)
        db.commit()
        print("更新完成")
    except err:
        db.rollback()
        print("更新失败")
    db.close()

#删除操作
def delete_record():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = "delete from class_data where name='{}'".format("test")
    try:
        cursor.execute(sql)
        db.commit()
        print("删除成功")
    except err:
        db.rollback()
        print("删除失败")
    db.close()

#创建新表(学生地址)
def create_table():
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = """
        create table st_addr(
            id int unsigned auto_increment,
            number int(10) not null,
            address varchar(100) not null,
            primary key(id)
            )ENGINE=InnoDB DEFAULT CHARSET=UTF8"""
    cursor.execute(sql)
    print("创建新表成功")
    db.close()

#查询两张表
def query_tables(num):
    db = pymysql.connect("localhost","root","baozi","data_school")
    cursor = db.cursor()
    sql = "select a.number,a.name,b.address from class_data a "\
        "join st_addr b on a.number=b.number and a.number={}".format(num)
    try:
        cursor.execute(sql)
        results = cursor.fetchall()
        for r in results:
            number = r[0]
            name = r[1]
            address = r[2]
            print(f"学号为{num}的学生信息：number = {number},name = {name},address = {address}")
        print("查询完毕")
    except err:
        print("查询失败")
    db.close()


if __name__ == "__main__":
    # connect_mysql()
    # insert_record()
    # select_record()
    # update_record()
    # delete_record()
    # create_table()
    query_tables(1004)
```

![](https://lz.sinaimg.cn/orj1080/ebeef3aaly3gjnz4dq31wj21f10tztxu.jpg)
