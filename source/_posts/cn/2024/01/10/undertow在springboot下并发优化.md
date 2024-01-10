---
title: undertow在springboot下并发优化
catalog: true
header-img: /img/header_img/lml_bg.jpg
date: 2024-01-10 15:15:36
subtitle:
tags:
- Java
- Springboot
- Undertow
categories:
- Java
---

# 关于Springboot的Undertow并发优化
项目中遇到的问题：某个服务在并发较高且接口响应时间较长时，经常会出现没有足够的线程可用，体现在前端就是接口超时。
## 问题分析
undertow在处理请求方面主要有两个配置一个是io线程数，一个是worker线程数。具体配置如下(以springboot配置为例)：
```yaml
server:
  port: 8888
  undertow:
    threads:
      #阻塞任务线程池, 当执行类似servlet请求阻塞IO操作, undertow会从这个线程池中取得线程，默认为io线程数*8
      worker: 96
      #设置IO线程数, 它主要执行非阻塞的任务,它们会负责多个连接，默认为CPU核数
      io: 12
```
io线程数用于接受请求，并分发给worker线程进行具体的接口逻辑处理,若worker线程数不够，会导致请求阻塞，新请求进入会阻塞。
## 解决方案
基于以上分析，根据项目需求进行调整，io线程数一般不是瓶颈，根据项目服务的并发量和接口响应时间来确定worker线程数大小，保证有足够的线程提供响应。
