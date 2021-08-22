# Table of Contents

* [Week9-作业-3](#week9-作业-3)
   * [题目](#题目)
   * [解题](#解题)
      * [分析](#分析)
      * [系统拆分](#系统拆分)
      * [涉及的分布式事务问题](#涉及的分布式事务问题)
      * [表设计](#表设计)
      * [项目](#项目)


# Week9-作业-3

## 题目

> 结合 dubbo+hmily，实现一个 TCC 外汇交易处理，代码提交到 GitHub:\
\
用户 A 的美元账户和人民币账户都在 A 库，使用 1 美元兑换 7 人民币 ;\
用户 B 的美元账户和人民币账户都在 B 库，使用 7 人民币兑换 1 美元 ;\
设计账户表，冻结资产表，实现上述两个本地事务的分布式事务。

## 解题

### 分析

基于我自己的理解，进行了一些考虑：

1. 目前有家银行开放线上个人外汇交易平台（即我们的Demo系统），用户A和用户B决定在该平台进行外汇交易（即货币交换）；
2. 前提用户A和用户B都均已开通对方币种账户（即人民币和美元）；
3. 交易流程
   1. 用户A对用户B发起交易，金额1美元，系统成功冻结用户A1美元后，向用户B发起交易；
   2. 用户B接收到交易请求，确认交易，按汇率计算应支付7RMB，冻结用户B的资金；
   3. 系统对双方账户进行处理，处理成功后清除冻结资金。


### 系统拆分

- 交易系统（即用户A和用户B操作的平台）
- 账户系统（即存放用户账户信息的系统）


### 涉及的分布式事务问题

交易系统中最终进行交易，无论先操作用户A还是B的账户，都是跨系统的，也就是在交易系统中先一个用户钱扣完也加好了，另一个用户还没开始，这时候交易系统出错了，那在账户系统已经处理完的用户A是无法感知交易系统的本地事务回滚的。


### 表设计

```sql
CREATE TABLE `t_account_rmb` (
  `id`          bigint(20) NOT NULL COMMENT '账户id',
  `uid`         bigint(20) NOT NULL COMMENT '用户id',
  `balance`     bigint(20) NOT NULL COMMENT '余额',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`    tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标志 0:正常 1:删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_uid` (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户人民币账户表';

CREATE TABLE `t_account_dollar` (
  `id`          bigint(20) NOT NULL COMMENT '账户id',
  `uid`         bigint(20) NOT NULL COMMENT '用户id',
  `balance`     bigint(20) NOT NULL COMMENT '余额',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`    tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标志 0:正常 1:删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_uid` (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户美元账户表';


CREATE TABLE `t_freeze_rmb` (
  `id`          bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'PK',
  `uid`         bigint(20) NOT NULL COMMENT '用户id',
  `amount`      bigint(20) NOT NULL COMMENT '金额',
  `exchange_id` bigint(20) NOT NULL COMMENT '交易id',
  `status`      tinyint(4) NOT NULL COMMENT '状态 0:正常 1:完成 2:取消',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`    tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标志 0:正常 1:删除',
  PRIMARY KEY (`id`),
  KEY `idx_uid_exchange` (`uid`,`exchange_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户人民币冻结表';


CREATE TABLE `t_freeze_dollar` (
  `id`          bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'PK',
  `uid`         bigint(20) NOT NULL COMMENT '用户id',
  `amount`      bigint(20) NOT NULL COMMENT '金额',
  `exchange_id` bigint(20) NOT NULL COMMENT '交易id',
  `status`      tinyint(4) NOT NULL COMMENT '状态 0:正常 1:完成 2:取消',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`    tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标志 0:正常 1:删除',
  PRIMARY KEY (`id`),
  KEY `idx_uid_exchange` (`uid`,`exchange_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户美元冻结表';

CREATE TABLE `t_foreign_exchange` (
  `id`                bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'PK',
  `from_uid`          bigint(20) NOT NULL COMMENT '发起交易用户id',  
  `from_out_currency` tinyint(4) NOT NULL COMMENT '发起出币种',
  `from_out_amount`   bigint(20) NOT NULL COMMENT '发起出金额',
  `from_in_currency`  tinyint(4) NOT NULL COMMENT '发起进币种',
  `from_in_amount`    bigint(20) NOT NULL COMMENT '发起进金额',
  `exchange_rate`     varchar(32) NOT NULL COMMENT '交易汇率 只做该笔交易时汇率展示用 按发起者出币种 : 入币种',
  `to_uid`            bigint(20) NOT NULL COMMENT '交易对象用户id',  
  `exchange_status`   tinyint(4) NOT NULL DEFAULT '0' COMMENT '交易状态 0:进行中 1:完成 2:取消',
  `create_time`       bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time`       bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`          tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标志 0:正常 1:删除',
  PRIMARY KEY (`id`),
  KEY `idx_uid` (`from_uid`,`to_uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='外汇交易表';
```

### 项目

[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/foreign-exchange-demo)


**测试**

故意在中间加个1/0触发异常，此时用户A转账结束但是用户B还没开始
```
        // 完成用户A的转账操作
        TransferVO transferVOA = new TransferVO();
        transferVOA.setFromUid(foreignExchange.getFromUid());
        transferVOA.setToUid(foreignExchange.getToUid());
        transferVOA.setCurrency(fromCurrency);
        transferVOA.setAmount(foreignExchange.getFromOutAmount());
        transferVOA.setExchangeId(foreignExchange.getId());
        accountInterface.transfer(transferVOA);
        
        int i=1/0

        // 完成用户B的转账操作
        TransferVO transferVOB = new TransferVO();
        transferVOB.setFromUid(foreignExchange.getToUid());
        transferVOB.setToUid(foreignExchange.getFromUid());
        transferVOB.setCurrency(toCurrency);
        transferVOB.setAmount(inAmount);
        transferVOB.setExchangeId(foreignExchange.getId());
        accountInterface.transfer(transferVOB);
```

**结果**

本地事务就不看了，交易信息肯定是回滚了，重点关注两个远端转账的操作。

看日志，美元的交易也就是用户A的转账被回滚了，人民币交易由于还没开始所以尝试回滚失败。

```
2021-08-21 16:20:04.898  INFO 47987 --- [20880-thread-10] c.x.f.e.a.service.AccountDollarService   : 尝试回滚美元转账 param = TransferRollbackVO(fromUid=10001, toUid=10002, fromCurrency=DOLLAR, toCurrency=RMB, exchangeId=6)
2021-08-21 16:20:04.923  INFO 47987 --- [20880-thread-10] c.x.f.e.a.service.AccountRmbService      : 尝试回滚人民币转账 param = TransferRollbackVO(fromUid=10001, toUid=10002, fromCurrency=DOLLAR, toCurrency=RMB, exchangeId=6)
2021-08-21 16:20:04.925  INFO 47987 --- [20880-thread-10] c.x.f.e.a.service.AccountRmbService      : 尝试回滚人民币转账，无需回滚 param = TransferRollbackVO(fromUid=10001, toUid=10002, fromCurrency=DOLLAR, toCurrency=RMB, exchangeId=6)
```
