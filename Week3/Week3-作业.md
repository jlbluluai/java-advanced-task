# Table of Contents

* [Week3-作业](#week3-作业)
  * [题目](#题目)
  * [通过httpclient访问server](#通过httpclient访问server)
  * [通过netty访问server](#通过netty访问server)
  * [实现过滤器](#实现过滤器)


# Week3-作业

## 题目

> 通过httpclient访问server、通过netty访问server、实现过滤器

[项目地址](https://github.com/jlbluluai/xyz-gateway)

## 通过httpclient访问server

[代码](https://github.com/jlbluluai/xyz-gateway/blob/master/src/main/java/com/xyz/gateway/outbound/httpclient)

## 通过netty访问server

[代码](https://github.com/jlbluluai/xyz-gateway/blob/master/src/main/java/com/xyz/gateway/outbound/netty)

## 实现过滤器

[代码](https://github.com/jlbluluai/xyz-gateway/tree/master/src/main/java/com/xyz/gateway/filter)

- ApiHttpRequestFilter: 对入站api进行过滤
- LogHttpResponseFilter: 对出站内容打印日志
