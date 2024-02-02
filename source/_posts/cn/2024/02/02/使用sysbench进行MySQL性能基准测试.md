---
title: 使用sysbench进行MySQL性能基准测试
catalog: true
header-img: /img/header_img/lml_bg.jpg
date: 2024-02-02 16:49:38
subtitle:
tags:
- mysql优化
categories:
- mysql
- DB
---


# 使用sysbench进行MySQL性能基准测试

## 安装sysbench
```shell
sudo yum install sysbench 
```

## 测试数据准备

### 创建数据库
```sql
create database sbtest;
```
### 创建数据
```shell
# prepare 命令
sysbench oltp_common.lua --time=300 --mysql-host=xx.xx.xx.xx --mysql-port=3306 --mysql-user=root --mysql-password=1111  --mysql-db=sbtest  --table-size=1000000 --tables=10 --threads=32 --events=999999999   prepare
```
![prepare.png](prepare.png)

## 开始测试
```shell
# run 命令
sysbench oltp_read_write.lua --time=300 --mysql-host=xx.xx.xx.xx --mysql-port=3306 --mysql-user=root --mysql-password=1111 --mysql-db=sbtest --table-size=1000000 --tables=10 --threads=32 --events=999999999  --report-interval=10  run
```

### 32线程测试
![test32thread.png](test32thread.png)
### 64线程测试
![test64thread.png](test64thread.png)
### 128线程测试
![test128thread.png](test128thread.png)
### 128线程测试结果汇总
![summary128.png](summary128.png)

## 清理测试数据
```shell
# cleanup命令
sysbench oltp_read_write.lua --time=300 --mysql-host=xx.xx.xx.xx --mysql-port=3306 --mysql-user=root --mysql-password=1111 --mysql-db=sbtest --table-size=1000000 --tables=10 --threads=32 --events=999999999  --report-interval=10  cleanup
```
![cleanup.png](cleanup.png)

## 总结
随着线程数从32-->64-->128，TPS和QPS呈上升趋势，延迟也越来越高，说明随着并发量的增加，性能逐渐出现瓶颈。