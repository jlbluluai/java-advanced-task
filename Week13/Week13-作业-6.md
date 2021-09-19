# Week13-作业-6

## 题目

> 思考和设计自定义 MQ 第二个版本或第三个版本，写代码实现其中至少一个功能点

## 解题

实现了第二个版本：

去掉内存 Queue，设计自定义 Queue，实现消息确认和消费 offset

- 自定义内存 Message 数组模拟 Queue。
- 使用指针记录当前消息写入位置。
- 对于每个命名消费者，用指针记录消费位置。

[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/kmq-core)

**定义了自定义的MyQueue**

通过数组实现，并通过一个ConcurrentHashMap标记每个消费者的读指针位置

```
@Slf4j
public class MyQueue<T> {

    private T[] queue;

    private int capacity;

    /**
     * 当前写位置
     */
    private int currentWriteIndex = 0;

    private ConcurrentHashMap<String, Integer> consumers = new ConcurrentHashMap<>();

    /**
     * 写锁
     */
    private ReentrantLock lock = new ReentrantLock();

    private ConcurrentHashMap<String, ReentrantLock> readLockMap = new ConcurrentHashMap<>();


    public MyQueue(int capacity) {
        if (capacity < 1) {
            throw new IllegalArgumentException("队列容量不得为空");
        }
        this.capacity = capacity;
        this.queue = (T[]) new Object[capacity];
    }

    /**
     * 增加消费者
     */
    public void addConsumer(String consumerId) {
        consumers.put(consumerId, 0);
        readLockMap.put(consumerId, new ReentrantLock());
    }

    /**
     * 队列入值
     */
    public boolean offer(T message) {
        lock.lock();
        try {
            // 容量已满
            if (currentWriteIndex == capacity) {
                return false;
            }

            // 容量不满，写入
            queue[currentWriteIndex++] = message;

            return true;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 队列出值
     */
    public T poll(String consumerId) {
        ReentrantLock lock = readLockMap.get(consumerId);
        lock.lock();
        try {
            // 拿出当前消费者读指针
            int currentReadIndex = consumers.get(consumerId);

            // 无数据 当前读追上了当前写，无数据可读
            if (currentReadIndex == currentWriteIndex) {
                return null;
            }

            // 容量有，读取
            T t = queue[currentReadIndex];
            consumers.put(consumerId, ++currentReadIndex);

            return t;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 队列出值，带超时
     */
    public T poll(String consumerId, long timeout, TimeUnit timeUnit) throws InterruptedException {
        ReentrantLock lock = readLockMap.get(consumerId);
        try {
            boolean flag = lock.tryLock(timeout, timeUnit);

            if (!flag) {
                return null;
            }

            // 拿出当前消费者读指针
            int currentReadIndex = consumers.get(consumerId);

            // 无数据 当前读追上了当前写，无数据可读
            if (currentReadIndex == currentWriteIndex) {
                return null;
            }

            // 容量有，读取
            T t = queue[currentReadIndex];
            consumers.put(consumerId, ++currentReadIndex);
            return t;
        } finally {
            lock.unlock();
        }
    }

}
```
