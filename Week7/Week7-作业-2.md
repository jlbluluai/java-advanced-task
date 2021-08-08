# Table of Contents

* [Week7-作业-2](#week7-作业-2)
  * [题目](#题目)
  * [解题](#解题)
    * [方式1-循环单条插入](#方式1-循环单条插入)
    * [方式2-拼装多条sql一次性执行](#方式2-拼装多条sql一次性执行)
    * [方式3-PreparedStatement的batch方式](#方式3-preparedstatement的batch方式)
    * [总结](#总结)


# Week7-作业-2

## 题目

> 按自己设计的表结构，插入 100 万订单模拟数据，测试不同方式的插入效率

## 解题

### 方式1-循环单条插入

**结论**

由于100w次插入每次都要获取连接，执行，再断开连接，非常慢，跑了10分钟都未见结束，效率极其低下。

**测试用例**

``` java
@Slf4j
public class TestLargeDataInsert_1 {

    private Connection getConnection() throws SQLException {
        String url = "jdbc:mysql://127.0.0.1:3306/xyz_shop?characterEncoding=utf8&useSSL=false&serverTimezone=GMT";
        String user = "root";
        String password = "123456";
        return DriverManager.getConnection(url, user, password);
    }


    private static final String INSERT_SQL = "INSERT INTO `t_order`(`uid`, `product_id`, `product_snapshot`, `product_amount`, `order_status`, `order_price`, `create_time`, `update_time`) " +
            "VALUES (%s, %s, '%s', %s, %s, %s, %s, %s);";

    public void insert(Order order) {
        try (Connection con = getConnection()) {
            Statement statement = con.createStatement();
            String sql = String.format(INSERT_SQL,
                    order.getUid(),
                    order.getProductId(),
                    order.getProductSnapshot(),
                    order.getProductAmount(),
                    order.getOrderStatus(),
                    order.getOrderPrice(),
                    order.getCreateTime(),
                    order.getUpdateTime());
            statement.execute(sql);
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }


    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        TestLargeDataInsert_1 o = new TestLargeDataInsert_1();
        for (int i = 0; i < 1000000; i++) {
            Order testOrder = new Order();
            testOrder.setUid(i);
            testOrder.setProductId(i);
            testOrder.setProductSnapshot("");
            testOrder.setProductAmount(1);
            testOrder.setOrderStatus(1);
            testOrder.setOrderPrice(1);
            testOrder.setCreateTime(1);
            testOrder.setUpdateTime(1);
            o.insert(testOrder);
        }

        log.info("cost={}ms", System.currentTimeMillis() - start);
    }

}
```



### 方式2-拼装多条sql一次性执行

**结论**

利用了values可以一次性带入多条的特性，每次插入1000条（放置一次性sql太长超过限制），这样执行下来，只花费了13s左右，基本上是可以接受的时间范围内了。

**测试用例**

```java
@Slf4j
public class TestLargeDataInsert_2 {

    private Connection getConnection() throws SQLException {
        String url = "jdbc:mysql://127.0.0.1:3306/xyz_shop?characterEncoding=utf8&useSSL=false&serverTimezone=GMT";
        String user = "root";
        String password = "123456";
        return DriverManager.getConnection(url, user, password);
    }


    private static final String INSERT_SQL = "INSERT INTO `t_order`(`uid`, `product_id`, `product_snapshot`, `product_amount`, `order_status`, `order_price`, `create_time`, `update_time`) " +
            "VALUES";
    private static final String INSERT_SQL_VALUE = "(%s, %s, '%s', %s, %s, %s, %s, %s)";

    public void insertMulti(List<Order> orders) {
        try (Connection con = getConnection()) {
            Statement statement = con.createStatement();
            StringBuilder builder = new StringBuilder();
            orders.forEach(order -> {
                String sql = String.format(INSERT_SQL_VALUE,
                        order.getUid(),
                        order.getProductId(),
                        order.getProductSnapshot(),
                        order.getProductAmount(),
                        order.getOrderStatus(),
                        order.getOrderPrice(),
                        order.getCreateTime(),
                        order.getUpdateTime());
                builder.append("\n").append(sql).append(",");
            });
            builder.deleteCharAt(builder.length() - 1).append(";");
            statement.executeUpdate(INSERT_SQL + builder.toString());
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }


    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        TestLargeDataInsert_2 o = new TestLargeDataInsert_2();
        List<Order> orders = new ArrayList<>();
        for (int i = 0; i < 1000000; i++) {
            Order testOrder = new Order();
            testOrder.setUid(i);
            testOrder.setProductId(i);
            testOrder.setProductSnapshot("");
            testOrder.setProductAmount(1);
            testOrder.setOrderStatus(1);
            testOrder.setOrderPrice(1);
            testOrder.setCreateTime(1);
            testOrder.setUpdateTime(1);
            orders.add(testOrder);

            if (orders.size() == 1000) {
                o.insertMulti(orders);
                orders.clear();
            }
        }

        if (orders.size() != 0) {
            o.insertMulti(orders);
            orders.clear();
        }

        log.info("cost={}ms", System.currentTimeMillis() - start);
    }

}
```


### 方式3-PreparedStatement的batch方式

**结论**

该方式相当于一次性执行了多条insert，所以在批量插入前关闭自动提交事务，改为主动提交，减少提交事务的性能损耗，最后执行大概在1分钟（不关闭自动事务提交大概3分钟）

**测试用例**

```java
@Slf4j
public class TestLargeDataInsert_3 {

    private Connection getConnection() throws SQLException {
        String url = "jdbc:mysql://127.0.0.1:3306/xyz_shop?characterEncoding=utf8&useSSL=false&serverTimezone=GMT";
        String user = "root";
        String password = "123456";
        return DriverManager.getConnection(url, user, password);
    }

    private static final String INSERT_SQL = "INSERT INTO `t_order`(`uid`, `product_id`, `product_snapshot`, `product_amount`, `order_status`, `order_price`, `create_time`, `update_time`) " +
            "VALUES (?, ?, ?, ?, ?, ?, ?, ?);";

    public void insertBatch(List<Order> orders) {
        try (Connection con = getConnection()) {
            con.setAutoCommit(false);
            PreparedStatement ps = con.prepareStatement(INSERT_SQL);

            for (Order order : orders) {
                ps.setLong(1,order.getUid());
                ps.setLong(2,order.getProductId());
                ps.setString(3,order.getProductSnapshot());
                ps.setInt(4,order.getProductAmount());
                ps.setInt(5,order.getOrderStatus());
                ps.setLong(6,order.getOrderPrice());
                ps.setLong(7,order.getCreateTime());
                ps.setLong(8,order.getUpdateTime());
                ps.addBatch();
            }
            ps.executeBatch();
            con.commit();
            con.setAutoCommit(true);
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }


    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        TestLargeDataInsert_3 o = new TestLargeDataInsert_3();
        List<Order> orders = new ArrayList<>();
        for (int i = 0; i < 1000000; i++) {
            Order testOrder = new Order();
            testOrder.setUid(i);
            testOrder.setProductId(i);
            testOrder.setProductSnapshot("");
            testOrder.setProductAmount(1);
            testOrder.setOrderStatus(1);
            testOrder.setOrderPrice(1);
            testOrder.setCreateTime(1);
            testOrder.setUpdateTime(1);
            orders.add(testOrder);

            if (orders.size() == 1000) {
                o.insertBatch(orders);
                orders.clear();
            }
        }

        if (orders.size() != 0) {
            o.insertBatch(orders);
            orders.clear();
        }

        log.info("cost={}ms", System.currentTimeMillis() - start);
    }

}
```


### 总结

插入的values后面可以带多条数据，尽量利用这个特性一次多插入些数据，效率会更高。若是非要执行多条insert语句，也要关闭自动事务提交，减少每次insert自身都是一个事务的性能损耗。一次连接只执行一次插入这就非常不建议了，连接本身更是一个重的性能损耗。
