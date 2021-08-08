# Week7-作业-9

## 题目

> 读写分离 - 数据库框架版本 2.0

## 解题


[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-shardingsphere)

**测试用例**

```java
@Test
public void testUser(){
    User user = new User();
    user.setUserName("test2");
    user.setPassword("123");
    user.setNickname("测试2");
    user.setPhone("13676456568");
    user.setMail("test2@qq.com");
    user.setCreateTime(System.currentTimeMillis());
    user.setUpdateTime(System.currentTimeMillis());
    userService.add(user);
    System.out.println(user.getId());

    User slaveUser = userService.query(user.getId());
    System.out.println(slaveUser);
}
```

**结果**

日志中insert用的ds0即主库，select走的ds1即从库
```
2021-08-08 13:12:31.218  INFO 45222 --- [           main] ShardingSphere-SQL                       : Logic SQL: insert into t_user (user_name, `password`, nickname,
                            phone, mail, create_time,
                            update_time)
        values (?, ?, ?,
                ?, ?, ?,
                ?)
2021-08-08 13:12:31.218  INFO 45222 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLInsertStatement(setAssignment=Optional.empty, onDuplicateKeyColumns=Optional.empty)
2021-08-08 13:12:31.218  INFO 45222 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds0 ::: insert into t_user (user_name, `password`, nickname,
                            phone, mail, create_time,
                            update_time)
        values (?, ?, ?,
                ?, ?, ?,
                ?) ::: [test2, 123, 测试2, 13676456568, test2@qq.com, 1628399550311, 1628399550311]
39
2021-08-08 13:12:31.364  INFO 45222 --- [           main] ShardingSphere-SQL                       : Logic SQL: select
         
        
        id, user_name, `password`, nickname, phone, mail, create_time, update_time
     
        from t_user
        where id = ?
2021-08-08 13:12:31.364  INFO 45222 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLSelectStatement(limit=Optional.empty, lock=Optional.empty)
2021-08-08 13:12:31.364  INFO 45222 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: select
         
        
        id, user_name, `password`, nickname, phone, mail, create_time, update_time
     
        from t_user
        where id = ? ::: [39]
User(id=39, userName=test2, password=123, nickname=测试2, phone=13676456568, mail=test2@qq.com, createTime=1628399550311, updateTime=1628399550311)

```
