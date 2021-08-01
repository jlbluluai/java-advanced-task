# Table of Contents

* [Week6-作业-6](#week6-作业-6)
  * [题目](#题目)
  * [解题](#解题)

# Week6-作业-6

## 题目

> 基于电商交易场景（用户、商品、订单），设计一套简单的表结构。

## 解题

```SQL
CREATE TABLE `t_user`
(
  `id`          bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'PK',
  `user_name`   varchar(32)  NOT NULL DEFAULT '' COMMENT '用户名',
  `password`    varchar(32)  NOT NULL DEFAULT '' COMMENT '密码',
  `nickname`    varchar(32)  NOT NULL DEFAULT '' COMMENT '昵称',
  `phone`       varchar(11)  NOT NULL DEFAULT '' COMMENT '手机号',
  `mail`        varchar(64)  NOT NULL DEFAULT '' COMMENT '邮箱',
  `avatar`      varchar(128) NOT NULL DEFAULT '' COMMENT '头像',
  `sex`         char(1)      NOT NULL DEFAULT '' COMMENT '性别 M:男 W:女',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`    tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标识 0:未删除 1:删除',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='用户表';


CREATE TABLE `t_product`
(
  `id`             bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'PK',
  `product_name`   varchar(32)   NOT NULL DEFAULT '' COMMENT '商品名',
  `product_type`   tinyint(4) NOT NULL DEFAULT '0' COMMENT '商品类型',
  `product_price`  bigint(20) NOT NULL DEFAULT '0' COMMENT '商品单价 单位:分',
  `product_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '商品状态 0:创建 1:上架 2:下架',
  `carousel_pic`   varchar(1024) NOT NULL DEFAULT '' COMMENT '商品轮播图 json',
  `description`    varchar(1024) NOT NULL DEFAULT '' COMMENT '商品描述',
  `create_time`    bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time`    bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`       tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标识 0:未删除 1:删除',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='商品表';

CREATE TABLE `t_product_stock`
(
  `product_id`          bigint(20) NOT NULL COMMENT '商品id',
  `stock_amount`        int(11) NOT NULL DEFAULT '0' COMMENT '库存总数',
  `remain_stock_amount` int(11) NOT NULL DEFAULT '0' COMMENT '剩余库存数量',
  `create_time`         bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time`         bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`            tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标识 0:未删除 1:删除',
  PRIMARY KEY (`product_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='商品库存表';

CREATE TABLE `t_product_stock_order_usage`
(
  `id`           bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'PK',
  `product_id`   bigint(20) NOT NULL COMMENT '商品id',
  `order_id`     bigint(20) NOT NULL COMMENT '订单id',
  `stock_amount` int(11) NOT NULL DEFAULT '0' COMMENT '占用库存数量',
  `usage_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '使用状态 0:占位 1:完成 2:取消',
  `create_time`  bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time`  bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`     tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标识 0:未删除 1:删除',
  PRIMARY KEY (`id`),
  KEY            `idx_product_id` (`product_id`),
  KEY            `idx_order_id` (`order_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='商品库存订单使用情况表';


CREATE TABLE `t_order`
(
  `id`               bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'PK',
  `uid`              bigint(20) NOT NULL DEFAULT '0' COMMENT '用户id',
  `product_id`       bigint(20) NOT NULL DEFAULT '0' COMMENT '商品id',
  `product_snapshot` text NOT NULL COMMENT '下单时商品信息快照 json',
  `product_amount`   int(11) NOT NULL DEFAULT '0' COMMENT '商品数量',
  `order_status`     tinyint(4) NOT NULL DEFAULT '0' COMMENT '订单状态 0:待支付 1:待发货 2:待收货 3:已完成 4:已取消',
  `order_price`      bigint(20) NOT NULL DEFAULT '0' COMMENT '订单金额 单位:分',
  `create_time`      bigint(20) NOT NULL DEFAULT '0' COMMENT '添加时间',
  `update_time`      bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `del_flag`         tinyint(4) NOT NULL DEFAULT '0' COMMENT '删除标识 0:未删除 1:删除',
  PRIMARY KEY (`id`),
  KEY                `idx_uid` (`uid`),
  KEY                `idx_product_id` (`product_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='订单表';
```
