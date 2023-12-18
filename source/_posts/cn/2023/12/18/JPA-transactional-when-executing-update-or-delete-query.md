---
title: JPA执行update/delete语句时要显性添加事务
catalog: true
header-img: /img/header_img/lml_bg.jpg
date: 2023-12-18 19:06:40
subtitle:
tags:
 - spring-data
 - jpa
categories:
 - Java
 - Spring
---
# JPA事务问题Executing an update/delete query

当使用repository执行update语句时除了加@Query和@Modifying注解外，还需要显式地使用@Transactional注解，否则会抛出异常。
异常错误截图：
![error_trace.png](error_trace.png)

# 解决方法

在repository中添加@Transactional注解，如下：

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Transactional
    @Modifying
    @Query("update User u set u.name =?1 where u.id =?2")
}