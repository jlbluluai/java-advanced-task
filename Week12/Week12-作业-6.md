# Table of Contents

* [Week12-作业-6](#week12-作业-6)
    * [题目](#题目)
    * [解题](#解题)
        * [代码](#代码)
        * [测试](#测试)
            * [测试queue的生产消费](#测试queue的生产消费)
            * [测试topic的发布订阅](#测试topic的发布订阅)


# Week12-作业-6

## 题目

> 搭建 ActiveMQ 服务，基于 JMS，写代码分别实现对于 queue 和 topic 的消息生产和消费。

## 解题

[代码地址](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-common/src/main/java/com/xyz/study/common/jike/week12)

### 代码

区别queue和topic只是定义的`Destination`不同类型，代码是一套。

统一的消息监听器
```
@Slf4j
public class TextMessageListener implements MessageListener {

    private String listenerName;

    public TextMessageListener(String listenerName) {
        this.listenerName = listenerName;
    }

    @Override
    public void onMessage(Message message) {
        try {
            TextMessage textMessage = (TextMessage) message;
            log.info("{}接收到消息 message={}", listenerName, textMessage.getText());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

实现消息生产消费的demo
```
public class ActiveMqDemo {

    public static void main(String[] args) {
        // 定义Destination
//        Destination destination = new ActiveMQQueue("xyz.queue");
        Destination destination = new ActiveMQTopic("xyz.topic");

        // 测试
        testDestination(destination);
    }

    public static void testDestination(Destination destination) {
        try {
            // 创建连接和会话
            ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory("tcp://127.0.0.1:61616");
            ActiveMQConnection conn = (ActiveMQConnection) factory.createConnection();
            conn.start();
            Session session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);

            // 创建两个消费者 并 绑定消息监听器
            MessageConsumer consumer1 = session.createConsumer(destination);
            MessageConsumer consumer2 = session.createConsumer(destination);
            consumer1.setMessageListener(new TextMessageListener("consumer1"));
            consumer2.setMessageListener(new TextMessageListener("consumer2"));

            // 模拟生产消息
            MessageProducer producer = session.createProducer(destination);
            int index = 0;
            while (index++ < 20) {
                TextMessage message = session.createTextMessage(index + " message.");
                producer.send(message);
            }

            Thread.sleep(20000);
            session.close();
            conn.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


### 测试

#### 测试queue的生产消费

![](http://img.yelizi.top/63d905e0-aae3-4141-8ecf-135bfe03d33e.jpg$xyz)

```
10:51:07.960 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=1 message.
10:51:07.977 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=2 message.
10:51:07.981 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=3 message.
10:51:07.988 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=4 message.
10:51:07.994 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=5 message.
10:51:07.999 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=6 message.
10:51:08.004 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=7 message.
10:51:08.008 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=8 message.
10:51:08.012 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=9 message.
10:51:08.024 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=10 message.
10:51:08.028 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=11 message.
10:51:08.034 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=12 message.
10:51:08.038 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=13 message.
10:51:08.049 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=14 message.
10:51:08.054 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=15 message.
10:51:08.059 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=16 message.
10:51:08.063 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=17 message.
10:51:08.069 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=18 message.
10:51:08.074 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=19 message.
10:51:08.082 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=20 message.
```

结果分析：

1. 成功生产了20条消息并消费完成
2. 两个消费者分别消费了10条没有重复，符合queue的定义

#### 测试topic的发布订阅

![](http://img.yelizi.top/8f8f9e0b-470a-4fa2-8054-7da57ffed246.jpg$xyz)

```
10:54:17.444 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=1 message.
10:54:17.445 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=1 message.
10:54:17.445 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=2 message.
10:54:17.447 [ActiveMQ Session Task-1] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=2 message.
10:54:17.450 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=3 message.
10:54:17.451 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=3 message.
10:54:17.454 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=4 message.
10:54:17.455 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=4 message.
10:54:17.459 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=5 message.
10:54:17.460 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=5 message.
10:54:17.464 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=6 message.
10:54:17.465 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=6 message.
10:54:17.468 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=7 message.
10:54:17.469 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=7 message.
10:54:17.473 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=8 message.
10:54:17.474 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=8 message.
10:54:17.478 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=9 message.
10:54:17.479 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=9 message.
10:54:17.487 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=10 message.
10:54:17.488 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=10 message.
10:54:17.492 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=11 message.
10:54:17.493 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=11 message.
10:54:17.497 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=12 message.
10:54:17.499 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=12 message.
10:54:17.502 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=13 message.
10:54:17.504 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=13 message.
10:54:17.511 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=14 message.
10:54:17.512 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=14 message.
10:54:17.517 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=15 message.
10:54:17.518 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=15 message.
10:54:17.521 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=16 message.
10:54:17.523 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=16 message.
10:54:17.526 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=17 message.
10:54:17.527 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=17 message.
10:54:17.530 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=18 message.
10:54:17.531 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=18 message.
10:54:17.537 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=19 message.
10:54:17.538 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=19 message.
10:54:17.542 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer1接收到消息 message=20 message.
10:54:17.543 [ActiveMQ Session Task-2] INFO com.xyz.study.common.jike.week12.TextMessageListener - consumer2接收到消息 message=20 message.
```

结果分析：

1. 生产了20条消息，出有40条
2. 两个订阅者因为都订阅了，20条消息都是各自取到，也是为啥显示的Dequeued是40条