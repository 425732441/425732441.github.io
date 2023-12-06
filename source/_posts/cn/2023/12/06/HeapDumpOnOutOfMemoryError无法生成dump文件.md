---
title: HeapDumpOnOutOfMemoryError无法生成dump文件
catalog: true 
header-img: /img/header_img/lml_bg.jpg 
date: 2023-12-06 19:55:50 
subtitle:
tags:
- Java
- OOM 
categories:
- Java
- JVM
---

# 关于HeapDumpOnOutOfMemoryError无法生成dump文件的问题

## 问题背景

最近公司的一个项目，因为JVM内存大小设置问题导致应用出现OOM，但看到指定的HeapDumpPath路径中没有最新的dump文件产生，导致无法定位问题。遂决定看看为什么不生成新文件。

## 问题复现

 使用现有项目在本地环境中，设置JVM参数：_**-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/Users/xxx/applog/oom.hprof -Xms5m -Xmx5m**_。因项目是springboot框架，因此启动时迅速出现OOM异常。 第一次启动抛出异常后查看目录下确实生成了oom.hprof文件，但第二次启动后，没有再次生成最新的oom.hprof文件，也没有覆盖旧文件。

## 问题分析

 问题的原因在于，当程序抛出OOM异常时，JVM会自动dump堆内存，由于当前指定的参数到文件名 **_-XX:HeapDumpPath=/Users/xxx/applog/oom.hprof_** 路径下存在旧的oom.hprof文件，无法生成新文件。
![](oom_file_create_failed.png)

## 解决方案

### 方案一：指定HeapDumpPath到目录

 指定HeapDumpPath到目录后，如果出现OOM异常，JVM会根据线程的pid自动生成新的dump文件，每次启动的pid不同则不会出现文件名相同无法生成dump文件的情况。  
![](oom_file_create_pid.png)
**但该方案存在一个问题，假如应用使用docker或者k8s部署，则大概率会出现容器中的Java进程每次启动时的pid相同，则仍然会出现因为pid相同无法生成新dump文件的情况，因此引出方案二**

### 方案二：使用OnOutOfMemoryError参数

 使用 **_-XX:OnOutOfMemoryError='/home/docker/oom-test/heapDump.sh'_** 参数可以在出现OOM且生成dump文件后执行一个脚本对dump文件进行重命名，这样下次再出现OOM，就不会因为文件已存在而生成失败了。
 
#### heapDump.sh文件内容

```bash
#在该脚本下修改hprof的文件名
datetime = $(date "%Y%m%d%H%M%S")
hprofs = `find /applog -name '*.hprof'`
for tmprof in $hprofs 
do
    mv $tmprof `echo "$tmprof.$datetime"`
done
```
#### 效果验证
 JVM根据pid生成了dumpfile，之后运行了heapDump.sh脚本重命名了dumpfile
![img.png](oom_file_create_and_exec_shell.png)