# Week14-作业-1

## 题目

> 自己动手设计一个高并发的秒杀系统

## 解题

![](http://img.yelizi.top/ee8e9b54-3d5a-4552-9a42-d6641100d84f.jpg$xyz)

**前端**

1. 秒杀按钮设置防重复点击，大量先排除大量正常用户不停点击带来的重复流量
2. 参与秒杀前必须做一个人机交互的验证，比如说秒杀前1分钟就可以做了，做了有效半小时这样，大量排除无脑刷单的机器


**后端**

1. nginx网关限制流量进入，以防并发量太大，撑爆服务器
2. 服务器网关做一个负载均衡，将流量打散到多个秒杀系统去，视秒杀的并发量，准备秒杀系统的资源数量（若量预估就很大，也需要支撑，但不能很好预估大到哪去，也做好备用资源准备，随时加机器）
3. 流量进入秒杀系统后，上redis分布式锁，检验redis中计数是否达到max，若达到，直接打回请求，若没有，在redis中计数+1（假设秒杀量一人一个），释放分布式锁（若可能，单独搭建秒杀redis集群服务秒杀系统）
4. 订单直接下单，无论库存是否真实扣减，塞入队列通知库存系统，然后直接返回客户端下单成功