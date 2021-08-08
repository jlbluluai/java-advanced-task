# Table of Contents

* [Week7-作业-9](#week7-作业-9)
  * [题目](#题目)
  * [解题](#解题)


# Week7-作业-9

## 题目

> 读写分离 - 动态切换数据源版本 1.0

## 解题

[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-multi-data-source)

**测试用例**

```java
@Test
public void testUser(){
    User user = new User();
    user.setUserName("test");
    user.setPassword("123");
    user.setNickname("测试");
    user.setPhone("13676456567");
    user.setMail("test@qq.com");
    user.setCreateTime(System.currentTimeMillis());
    user.setUpdateTime(System.currentTimeMillis());
    userService.add(user);
    System.out.println(user.getId());

    User slaveUser = userService.query(user.getId());
    System.out.println(slaveUser);
}
```

**结果**

主库插入user，id为1，读取从库的id为1的user

```
2021-08-07 21:48:45.604  INFO 42242 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-08-07 21:48:46.011  INFO 42242 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
1
2021-08-07 21:48:46.067  INFO 42242 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Starting...
2021-08-07 21:48:46.092  INFO 42242 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Start completed.
User(id=1, userName=test, password=123, nickname=测试, phone=13676456567, mail=test@qq.com, createTime=1628344125577, updateTime=1628344125577)
```
