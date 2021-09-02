# Week11-作业-8

## 题目

> 基于 Redis 封装分布式数据操作：\
在 Java 中实现一个简单的分布式锁；\
在 Java 中实现一个分布式计数器，模拟减库存。

## 解题

### 在 Java 中实现一个简单的分布式锁

[代码地址](https://github.com/jlbluluai/xyz-study/blob/master/xyz-study-common/src/main/java/com/xyz/study/common/jike/week11/task8/SimpleRedisLock.java)

关键点在于释放锁时得判断锁是否还是那个锁，所以得先取值匹配再删除，为了保证原子性，通过lua脚本实现。

```java
@Component
@Slf4j
public class SimpleRedisLock {

    @Resource
    private MyRedisTemplate myRedisTemplate;

    /**
     * 获取锁
     *
     * @param key         key
     * @param value       value
     * @param exSeconds   锁住时长 单位：s
     * @param waitSeconds 若未获取锁，尝试时长 单位：s 没秒尝试一次
     * @return true/false
     */
    public boolean lock(String key, String value, int exSeconds, int waitSeconds) {
        int wait = 0;
        do {
            boolean lock = myRedisTemplate.setexnx(key, value, exSeconds);
            if (lock) {
                return true;
            } else {
                if (waitSeconds > 0) {
                    try {
                        Thread.sleep(1000);
                    } catch (Exception e) {
                        log.error("lock wait for sleep failed.", e);
                    }
                    wait++;
                }
            }
        } while (wait < waitSeconds);

        return false;
    }

    /**
     * 释放锁
     *
     * @param key   key
     * @param value value
     */
    public boolean unlock(String key, String value) {
        // 通过lua脚本，（获取锁值匹配再删除，做成原子操作），防止释放的锁有误
        String script = "if redis.call(\"get\",KEYS[1]) == ARGV[1] then \n" +
                "    return redis.call(\"del\",KEYS[1]) \n" +
                "else\n" +
                "    return 0 \n" +
                "end";
        Long result = myRedisTemplate.eval(script, Long.class, Collections.singletonList(key), new String[]{value});
        return result != null && result == 1;
    }

}
```

**测试用例**

```java
    @Resource
    private SimpleRedisLock simpleRedisLock;

    @Test
    public void test1() {
        // 锁key
        String key = "lock.10001";

        // A获取锁 锁住5s
        long a = System.currentTimeMillis();
        boolean lock = simpleRedisLock.lock(key, a + "", 5, 0);
        System.out.println("A获取锁：" + lock + " ，当前时间：" + new Date());

        // B获取锁 等待2s 理论上B是获取不到锁的
        long b = System.currentTimeMillis();
        lock = simpleRedisLock.lock(key, b + "", 5, 2);
        System.out.println("B获取锁：" + lock + " ，当前时间：" + new Date());

        // 粗略来算 B等待了2s 离A自动释放锁还有3s
        // C获取锁 等待4s 理论上C会获取锁并且时间在B那条日志时间后的不到3s点
        long c = System.currentTimeMillis();
        lock = simpleRedisLock.lock(key, c + "", 5, 4);
        System.out.println("C获取锁：" + lock + " ，当前时间：" + new Date());

        // C获取锁 理论上5s后才释放 我们主动释放锁
        simpleRedisLock.unlock(key, c + "");

        // D获取锁 由于C主动释放锁 D应该是立马拿到锁
        long d = System.currentTimeMillis();
        lock = simpleRedisLock.lock(key, d + "", 5, 0);
        System.out.println("D获取锁：" + lock + " ，当前时间：" + new Date());
    }
```

**结果**

```
A获取锁：true ，当前时间：Thu Sep 02 14:53:55 CST 2021
B获取锁：false ，当前时间：Thu Sep 02 14:53:57 CST 2021
C获取锁：true ，当前时间：Thu Sep 02 14:54:00 CST 2021
D获取锁：true ，当前时间：Thu Sep 02 14:54:00 CST 2021
```


### 在 Java 中实现一个分布式计数器，模拟减库存

[代码地址](https://github.com/jlbluluai/xyz-study/blob/master/xyz-study-common/src/main/java/com/xyz/study/common/jike/week11/task8/SimpleRedisCounter.java)

关键点在于计数器减操作时会不会触碰底线（比如库存的0），得先取值比较再减，为了保证原子性，通过lua脚本实现。

```java
@Component
public class SimpleRedisCounter {

    @Resource
    private MyRedisTemplate myRedisTemplate;

    /**
     * 初始化
     *
     * @param key   key
     * @param quota 配额数量
     */
    public void init(String key, long quota) {
        myRedisTemplate.set(key, String.valueOf(quota));
    }

    /**
     * 获取当前计数
     *
     * @param key key
     */
    public long get(String key) {
        String data = myRedisTemplate.get(key);
        return Long.parseLong(data);
    }

    /**
     * 加
     *
     * @param key   key
     * @param quota 配额数量
     * @return true/flase
     */
    public boolean plus(String key, long quota) {
        Long increment = myRedisTemplate.originalTemplate().opsForValue().increment(key, quota);
        return increment != null;
    }

    /**
     * 减（默认最低限制为0，即减完不得低于0）
     *
     * @param key   key
     * @param quota 配额数量
     * @return true/false
     */
    public boolean minus(String key, long quota) {
        String script = "local value = redis.call(\"get\",KEYS[1])\n" +
                "if(value == nil or value == 0 or value - ARGV[1] < 0) \n" +
                "then \n" +
                "    return nil\n" +
                "else \n" +
                "    return redis.call(\"decrby\",KEYS[1],ARGV[1])\n" +
                "end";
        Long result = myRedisTemplate.eval(script, Long.class, Collections.singletonList(key), new String[]{String.valueOf(quota)});
        return result != null;
    }

    /**
     * 减（自定义最低限制，即减完不低于该值即可）
     *
     * @param key   key
     * @param quota 配额数量
     * @param min   最低限制
     * @return true/false
     */
    public boolean minus(String key, long quota, long min) {
        String script = "local value = redis.call(\"get\",KEYS[1])\n" +
                "if(value == nil or value == " + min + " or value - ARGV[1] < " + min + ") \n" +
                "then \n" +
                "    return nil\n" +
                "else \n" +
                "    return redis.call(\"decrby\",KEYS[1],ARGV[1])\n" +
                "end";
        Long result = myRedisTemplate.eval(script, Long.class, Collections.singletonList(key), new String[]{String.valueOf(quota)});
        return result != null;
    }
}
```

**测试用例**

```java
    @Resource
    private SimpleRedisCounter simpleRedisCounter;

    @Test
    public void test2() {
        // 库存key
        String key = "inventory.10001";

        // 设定库存初始为10
        simpleRedisCounter.init(key, 10);

        // A使用库存4
        boolean flag = simpleRedisCounter.minus(key, 4);
        System.out.println("A：" + flag);
        System.out.println("剩余库存：" + simpleRedisCounter.get(key));

        // 此时平台调整控制库存最少5 B使用库存3
        flag = simpleRedisCounter.minus(key, 3, 5);
        System.out.println("B：" + flag);
        System.out.println("剩余库存：" + simpleRedisCounter.get(key));

        // 平台取消库存限制 C使用库存5
        flag = simpleRedisCounter.minus(key, 5);
        System.out.println("C：" + flag);
        System.out.println("剩余库存：" + simpleRedisCounter.get(key));

        // D使用库存2
        flag = simpleRedisCounter.minus(key, 2);
        System.out.println("D：" + flag);
        System.out.println("剩余库存：" + simpleRedisCounter.get(key));
    }
```

**结果**

```
A：true
剩余库存：6
B：false
剩余库存：6
C：true
剩余库存：1
D：false
剩余库存：1
```