---
title: mysql索引类型
catalog: true
header-img: /img/header_img/lml_bg.jpg
date: 2024-01-16 17:11:13
subtitle:
tags:
- mysql
- index
categories:
- 数据库
- mysql
---

# mysql索引类型

## 1. 普通索引
是最基本的索引，它没有任何限制。它有以下几种创建方式：
###（1）直接创建索引

```sql92
CREATE INDEX index_name ON table(column(length))
```

###（2）修改表结构的方式添加索引

```sql92
ALTER TABLE table_name ADD INDEX index_name ON (column(length))
```
###（3）创建表的时候同时创建索引
```sql92
CREATE TABLE `table` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`title` char(255) CHARACTER NOT NULL ,
`content` text CHARACTER NULL ,
`time` int(10) NULL DEFAULT NULL ,
PRIMARY KEY (`id`),
INDEX index_name (title(length))
)
```
 
###（4）删除索引

```sql92
DROP INDEX index_name ON table
```

## 2.唯一索引
与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。它有以下几种创建方式：
###（1）创建唯一索引
```sql92
CREATE UNIQUE INDEX indexName ON table(column(length))
```
###（2）修改表结构
```sql92
ALTER TABLE table_name ADD UNIQUE indexName ON (column(length))
```
###（3）创建表的时候直接指定
```sql92
CREATE TABLE `table` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`title` char(255) CHARACTER NOT NULL ,
`content` text CHARACTER NULL ,
`time` int(10) NULL DEFAULT NULL ,
UNIQUE indexName (title(length))
);
```
## 3.主键索引
是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引：
```sql92
CREATE TABLE `table` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`title` char(255) NOT NULL ,
PRIMARY KEY (`id`)
);
```
## 4.组合索引
指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀集合
```sql92
ALTER TABLE `table` ADD INDEX name_city_age (name,city,age);
```
## 5.全文索引
主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。fulltext索引配合match against操作使用，而不是一般的where语句加like。它可以在create table，alter table ，create index使用，不过目前只有char、varchar，text 列上可以创建全文索引。值得一提的是，在数据量较大时候，现将数据放入一个没有全局索引的表中，然后再用CREATE index创建fulltext索引，要比先为一张表建立fulltext然后再将数据写入的速度快很多。
###（1）创建表的适合添加全文索引
```sql
CREATE TABLE `table` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`title` char(255) CHARACTER NOT NULL ,
`content` text CHARACTER NULL ,
`time` int(10) NULL DEFAULT NULL ,
PRIMARY KEY (`id`),
FULLTEXT (content)
);
```
###（2）修改表结构添加全文索引
```sql
ALTER TABLE article ADD FULLTEXT index_content(content)
```
###（3）直接创建索引
```sql
CREATE FULLTEXT INDEX index_content ON article(content)
```