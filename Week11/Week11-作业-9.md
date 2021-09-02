# Week11-作业-9

## 题目

> 基于 Redis 的 PubSub 实现订单异步处理

## 解题

[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-common/src/main/java/com/xyz/study/common/jike/week11/task9)

**模拟订单发布**

```java
@RestController
public class OrderPub {

    @Resource
    private MyRedisTemplate myRedisTemplate;

    @PostMapping("/order/pub")
    public void pubOrder(@RequestBody OrderAddVO orderAddVO){
        myRedisTemplate.originalTemplate().convertAndSend("order.create", JSON.toJSONString(orderAddVO));
    }
}
```

**模拟订单订阅**

订单监听器配置
```java
@Configuration
public class OrderSubConfig {

    @Resource
    private OrderSub orderSub;

    @Bean
    RedisMessageListenerContainer orderCreateListener(final RedisConnectionFactory factory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(factory);
        container.addMessageListener(new MessageListenerAdapter(orderSub), new ChannelTopic("order.create"));
        return container;
    }

}
```

监听处理
```java
@Service
@Slf4j
public class OrderSub implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] bytes) {
        String s = new String(message.getBody());
        OrderAddVO orderAddVO = JSON.parseObject(s, OrderAddVO.class);
        log.info("添加订单 param={}", orderAddVO);
    }
}
```

**测试用例**

```
以 POST 方式访问 localhost:8080/order/pub

数据体
{
    "productId":10001,
    "amount":1
}
```

结果
```
2021-09-02 19:56:25.026  INFO 96934 --- [reateListener-2] c.x.s.common.jike.week11.task9.OrderSub  : 添加订单 param=OrderAddVO(productId=10001, amount=1)
```


