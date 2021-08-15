# Table of Contents

* [Week8-作业-2](#week8-作业-2)
  * [题目](#题目)
  * [解题](#解题)


# Week8-作业-2

## 题目

> 设计对前面的订单表数据进行水平分库分表，拆分 2 个库，每个库 16 张表。并在新结构在演示常见的增删改查操作。

## 解题

[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-db-sharding)

**插入**

```java
    @Test
    public void testOrder(){
        Order order = new Order();
        order.setUid(100001L);
        order.setProductId(1L);
        order.setProductAmount(1);
        order.setOrderStatus(0);
        order.setOrderPrice(1L);
        order.setCreateTime(System.currentTimeMillis());
        order.setUpdateTime(System.currentTimeMillis());
        orderService.add(order);

        System.out.println(order);
    }
```

用户id取模是ds1库，雪花算法id分配在0表
```
2021-08-14 18:27:29.999  INFO 9683 --- [           main] ShardingSphere-SQL                       : Logic SQL: insert into t_order (`uid`, product_id, product_amount,
                             order_status, order_price, create_time,
                             update_time)
        values (?, ?, ?,
                ?, ?, ?,
                ?)
2021-08-14 18:27:29.999  INFO 9683 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLInsertStatement(setAssignment=Optional.empty, onDuplicateKeyColumns=Optional.empty)
2021-08-14 18:27:29.999  INFO 9683 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: insert into t_order_0 (`uid`, product_id, product_amount,
                             order_status, order_price, create_time,
                             update_time, id)
        values (?, ?, ?, ?, ?, ?, ?, ?) ::: [100001, 1, 1, 0, 1, 1628936849215, 1628936849215, 633370312214679552]
Order(id=633370312214679552, uid=100001, productId=1, productAmount=1, orderStatus=0, orderPrice=1, createTime=1628936849215, updateTime=1628936849215)
```


**查询**

```java
    @Test
    public void testOrder(){
        Order order = orderService.query(100001, 633370312214679552L);
        System.out.println(order);
    }
```

查询刚才那条数据，命中1库和0表
```
2021-08-14 18:29:44.342  INFO 10234 --- [           main] ShardingSphere-SQL                       : Logic SQL: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order
        where del_flag = 0
        and id = ?
        and uid = ?
2021-08-14 18:29:44.342  INFO 10234 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLSelectStatement(limit=Optional.empty, lock=Optional.empty)
2021-08-14 18:29:44.342  INFO 10234 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order_0
        where del_flag = 0
        and id = ?
        and uid = ? ::: [633370312214679552, 100001]
Order(id=633370312214679552, uid=100001, productId=1, productAmount=1, orderStatus=0, orderPrice=1, createTime=1628936849215, updateTime=1628936849215)
```


**更新**

```java
    @Test
    public void testOrder(){
        Order order = orderService.query(100001, 633370312214679552L);
        System.out.println(order);

        order.setOrderPrice(10000L);
        orderService.update(order);

        order = orderService.query(100001, 633370312214679552L);
        System.out.println(order);
    }
```

查询刚才那条数据，然后修改金额，再查寻，数据变更完成
```
2021-08-14 18:33:37.074  INFO 11160 --- [           main] ShardingSphere-SQL                       : Logic SQL: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order
        where del_flag = 0
        and id = ?
        and uid = ?
2021-08-14 18:33:37.074  INFO 11160 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLSelectStatement(limit=Optional.empty, lock=Optional.empty)
2021-08-14 18:33:37.075  INFO 11160 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order_0
        where del_flag = 0
        and id = ?
        and uid = ? ::: [633370312214679552, 100001]
Order(id=633370312214679552, uid=100001, productId=1, productAmount=1, orderStatus=0, orderPrice=1, createTime=1628936849215, updateTime=1628936849215)
2021-08-14 18:33:37.162  INFO 11160 --- [           main] ShardingSphere-SQL                       : Logic SQL: update t_order
        set `uid`          = ?,
            product_id     = ?,
            product_amount = ?,
            order_status   = ?,
            order_price    = ?,
            create_time    = ?,
            update_time    = ?
        where id = ?
        and uid = ?
2021-08-14 18:33:37.162  INFO 11160 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLUpdateStatement(orderBy=Optional.empty, limit=Optional.empty)
2021-08-14 18:33:37.162  INFO 11160 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: update t_order_0
        set `uid`          = ?,
            product_id     = ?,
            product_amount = ?,
            order_status   = ?,
            order_price    = ?,
            create_time    = ?,
            update_time    = ?
        where id = ?
        and uid = ? ::: [100001, 1, 1, 0, 10000, 1628936849215, 1628936849215, 633370312214679552, 100001]
2021-08-14 18:33:37.175  INFO 11160 --- [           main] ShardingSphere-SQL                       : Logic SQL: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order
        where del_flag = 0
        and id = ?
        and uid = ?
2021-08-14 18:33:37.175  INFO 11160 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLSelectStatement(limit=Optional.empty, lock=Optional.empty)
2021-08-14 18:33:37.175  INFO 11160 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order_0
        where del_flag = 0
        and id = ?
        and uid = ? ::: [633370312214679552, 100001]
Order(id=633370312214679552, uid=100001, productId=1, productAmount=1, orderStatus=0, orderPrice=10000, createTime=1628936849215, updateTime=1628936849215)
```

**更新**

```java
    @Test
    public void testOrder(){
        Order order = orderService.query(100001, 633370312214679552L);
        System.out.println(order);

        orderService.delete(633370312214679552L);

        order = orderService.query(100001, 633370312214679552L);
        System.out.println(order);
    }
```

查询刚才那条数据，然后删除（该删除未指定用户id，因此两个库的0表都尝试删除了），再查寻，查找不到
```
2021-08-14 18:38:00.639  INFO 12215 --- [           main] ShardingSphere-SQL                       : Logic SQL: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order
        where del_flag = 0
        and id = ?
        and uid = ?
2021-08-14 18:38:00.640  INFO 12215 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLSelectStatement(limit=Optional.empty, lock=Optional.empty)
2021-08-14 18:38:00.640  INFO 12215 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order_0
        where del_flag = 0
        and id = ?
        and uid = ? ::: [633370312214679552, 100001]
Order(id=633370312214679552, uid=100001, productId=1, productAmount=1, orderStatus=0, orderPrice=10000, createTime=1628936849215, updateTime=1628936849215)
2021-08-14 18:38:00.718  INFO 12215 --- [           main] ShardingSphere-SQL                       : Logic SQL: delete
        from t_order
        where id = ?
2021-08-14 18:38:00.718  INFO 12215 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLDeleteStatement(orderBy=Optional.empty, limit=Optional.empty)
2021-08-14 18:38:00.718  INFO 12215 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds0 ::: delete
        from t_order_0
        where id = ? ::: [633370312214679552]
2021-08-14 18:38:00.718  INFO 12215 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: delete
        from t_order_0
        where id = ? ::: [633370312214679552]
2021-08-14 18:38:00.749  INFO 12215 --- [           main] ShardingSphere-SQL                       : Logic SQL: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order
        where del_flag = 0
        and id = ?
        and uid = ?
2021-08-14 18:38:00.750  INFO 12215 --- [           main] ShardingSphere-SQL                       : SQLStatement: MySQLSelectStatement(limit=Optional.empty, lock=Optional.empty)
2021-08-14 18:38:00.750  INFO 12215 --- [           main] ShardingSphere-SQL                       : Actual SQL: ds1 ::: select
         
        
        id, `uid`, product_id, product_amount, order_status, order_price, create_time, update_time
     
        from t_order_0
        where del_flag = 0
        and id = ?
        and uid = ? ::: [633370312214679552, 100001]
null

```
