---
title: spring事务失效的几个场景
catalog: true
header-img: /img/header_img/lml_bg.jpg
date: 2024-01-16 16:44:20
subtitle:
tags: 
- Spring
- Spring事务
- Java
categories:
- Java
- Spring
---


# spring事务失效的几个场景
## 1、抛出检查异常
如下代码会抛出检查异常，spring事务不会回滚
```java  
@Transactional
public void transactionTest() throws IOException{
    User user = new User();
    UserService.insert(user);
    throw new IOException();
}

```
如果@Transactional 没有特别指定，Spring 只会在遇到运行时异常RuntimeException或者error时进行回滚，而IOException等检查异常不会影响回滚。
```java 
// org.springframework.transaction.interceptor.DefaultTransactionAttribute#rollbackOn
public boolean rollbackOn(Throwable ex) {
	return (ex instanceof RuntimeException || ex instanceof Error);
}
```
## 2. 业务方法本身捕获了异常
spring事务管理通过AOP对方法进行增强，如果业务方法本身捕获了异常，spring事务管理不会进行回滚。  

```java 
@Transactional(rollbackFor = Exception.class)
public void transactionTest() {
    try {
        User user = new User();
        UserService.insert(user);
        int i = 1 / 0;
    }catch (Exception e) {
        e.printStackTrace();
    }
}
```
## 3. 同一类中的方法调用
这也是一个容易出错的场景。事务失败的原因也很简单，因为Spring的事务管理功能是通过动态代理实现的，而Spring默认使用JDK动态代理，而JDK动态代理采用接口实现的方式，通过反射调用目标类。简单理解，就是saveUser()方法中调用this.doInsert(),这里的this是被真实对象，所以会直接走doInsert的业务逻辑，而不会走切面逻辑，所以事务失败。

```java
@Service
public class DefaultTransactionService implement Service {

    public void saveUser() throws Exception {
        //do something
        doInsert();
    }

    @Transactional(rollbackFor = Exception.class)
    public void doInsert() throws IOException {
        User user = new User();
        UserService.insert(user);
        throw new IOException();

    }
}

```

## 4. 方法使用 final 或 static关键字
   如果Spring使用了Cglib代理实现（比如你的代理类没有实现接口），而你的业务方法恰好使用了final或者static关键字，那么事务也会失败。更具体地说，它应该抛出异常，因为Cglib使用字节码增强技术生成被代理类的子类并重写被代理类的方法来实现代理。如果被代理的方法的方法使用final或static关键字，则子类不能重写被代理的方法。
   如果Spring使用JDK动态代理实现，JDK动态代理是基于接口实现的，那么final和static修饰的方法也就无法被代理。
   总而言之，方法连代理都没有，那么肯定无法实现事务回滚了。

## 5. 方法不是public
   如果方法不是public，Spring事务也会失败，因为Spring的事务管理源码AbstractFallbackTransactionAttributeSource中有判断computeTransactionAttribute()。如果目标方法不是公共的，则TransactionAttribute返回null。

```java
// Don't allow no-public methods as required.
if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
  return null;
}

```

## 6. 多线程执行

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {

        userMapper.insertUser(userModel);
        new Thread(() -> {
             try {
                 test();
             } catch (Exception e) {
                roleService.doOtherThing();
             }
        }).start();
    }
}

@Service
public class RoleService {

    @Transactional
    public void doOtherThing() {
         try {
             int i = 1/0;
             System.out.println("保存role表数据");
         }catch (Exception e) {
            throw new RuntimeException();
        }
    }
}


```

我们可以看到事务方法add中，调用了事务方法doOtherThing，但是事务方法doOtherThing是在另外一个线程中调用的。
这样会导致两个方法不在同一个线程中，获取到的数据库连接不一样，从而是两个不同的事务。如果想doOtherThing方法中抛了异常，add方法也回滚是不可能的。
我们说的同一个事务，其实是指同一个数据库连接，只有拥有同一个数据库连接才能同时提交和回滚。如果在不同的线程，拿到的数据库连接肯定是不一样的，所以是不同的事务。
