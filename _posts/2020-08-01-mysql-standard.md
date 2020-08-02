---
layout: post
title: "数据库表设计规范"
description: "统一数据库表的命名规范。包括库名、表名、字段名和类型、索引等规范。"
summary: "统一数据库表的命名规范。包括库名、表名、字段名和类型、索引等规范。"
tags:
---

**数据库表设计规范**

# 变更历史

| 日期 | 变更人 | 变更说明 |
| ---- | -------- | --- |
| 2019-12-25 | truesnow | 第一版 |



# 目标

统一数据库表的命名规范。避免前端、测试、业务方等在对接不同后端开发人员开发的服务时，对相同、相似的概念使用不一样的名称，造成疑惑，进而增加对接难度。



# 规范

## 数据库命名规范

- 使用 `部门或业务线名称 + 服务名 + _db` 命名数据库。

> 例如：
- 支付中心：使用 `pay_` 为前缀
- 用户中心：使用 `user_` 为前缀
    - 用户触达服务：`user_push_db`
    - 白名单服务：`user_white_list_db`
- 活动中心：使用 `activity_` 为前缀
    - 拼团服务： `activity_group_buy_db`
    - 特价促销服务：`activity_special_price_db`
    - 秒杀服务：`activity_seckill_db`


## 数据表命名规范

- 统一以 `t_` 作为表名前缀
- 表名以名词单数命名

> 如：
- 活动表：`t_activity`
- 订单表：`t_order`

- 表名必须有注释
- 一般数据表必须为 InnoDB 类型，编码默认为 `utf8mb4`

```sql
ENGINE=InnoDB CHARSET=utf8mb4 COMMENT='表名';
```


## 索引规范

- 普通索引使用 `idx_` 前缀，后跟字段名，如 `idx_order_id`，如为联合索引，字段间加下划线，且按先后顺序书写（最左原则），如：`idx_type_name`
- 唯一索引使用 `uniq_` 前缀
- 如大数据团队会同步表数据，需设置更新时间为索引

```sql
INDEX `idx_update_time` (`update_time`)
```

- 在 `varchar` 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度即可

> 说明：索引的长度与区分度是一对矛盾体，一般对字符串类型数据，长度为 20 的索引，区分度会高达 90% 以上， 可以使用 `count(distinct left(列名, 索引长度))/count(*)` 的区分度来确定。


## 字段规范

### 字段命名规范

- 字段名必须是 `order_id` 格式，全小写，以 `_` 连接，若含数字，数字与单词连接在一起，如 `level3_name`
- 禁止使用保留字命名，如 `desc`、`range`、`match`、`delayed` 等，参考：[MySQL 关键字和保留字](https://dev.mysql.com/doc/refman/8.0/en/keywords.html)
- 字段以名词单数命名
- 字段必须有注释
- 主键 ID：命名为 `id`
- 状态字段以 `_status` 为后缀，非真即假字段命名不要以 `is` 开头，使用 `_status` 后缀（避免自动生成的 POJO 生成类似 `isIsDeleted` 这种代码），如：`review_status`、`pay_status`、`use_status`、`del_status`
- 时间相关字段以 `_time` 为后缀，如：`start_time`、`end_time`、`pay_time`
- 操作人相关字段以 `_user` 为后缀，如：`create_user`、`offline_user`
- 金额相关字段以 `_amount` 为后缀，如：`order_amount`
- 价格相关字段以 `_price` 为后缀，如：`sale_price`、`cost_price`
- 数量相关字段以 `_qty` （Quantity 简写）为后缀，如：`buy_qty`、`sold_qty`
- 统计数量相关字段以 `_count` 为后缀


### 字段注释规范

- 字段注释如有标点符号全部使用英文半角标点
- 枚举类型注释格式，在字段名称后面，枚举值使用小括号 `()` 包含，枚举值后面用冒号 `:` 与枚举名称分隔，每个枚举后面用分号 `;` 分隔，示例 ：`审核状态(1:待提交;2:审核中;3:审核通过;4:审核驳回)`
- 其他含有特殊说明的注释使用小括号 `()` 说明，如：`活动结束时间(时间戳)`、`告警通知邮箱(多个用,分隔)`
- 如字段已废弃或未使用等特殊情况，在注释最前面使用中括号 `[]` 写明，如：`[已废弃]`、`[该字段暂不使用]`
- 字段注释发生变更时必须提 SQL 单跟随服务上线（包括新增枚举值、状态废弃等情况）


### 字段类型规范

- 所有字段必须有 `NOT NULL DEFAULT`
- 任何字段如果为非负数，必须是 `unsigned`

> 反例：库存未设置 `unsigned`，库存可为负，商品超卖；订单金额未设置 `unsigned`，订单为负仍下单成功

- text 类型无法设置 `NOT NULL`，但在程序中应默认写入空字符串
- 枚举值字段定义为 `tinyint(4) UNSIGNED` 类型
    - 接口中会作为查询参数的字段：不能以 `0` 作为枚举起始值，`0` 只能作为字段默认值

> 理由是与业务或前端交互时，例如，在 PB 中，即使前端未传 status 字段，PB 也会将参数 status 值设为默认值 0，如果此时 status 为 0 有实际意义，那么这里查询就有歧义了，无法查询所有状态，而如果 status 为 0 无意义，后端可判断 status 为 0 时不按 status 搜索。

- 接口中不会使用到的字段，只在服务内部交互的非真即假字段，使用 0、1 作为枚举值

> 如：删除状态字段 `del_status`，0 表示未删除，1 表示已删除，前端与接口中不会按删除状态查询；库存释放状态字段 `stock_release_status`，0 表示未释放，1 表示已释放，这个状态是服务内部标识字段，只有服务内部会使用，前端与接口不会关心。

- 时间字段
  - 除创建时间、更新时间使用 `timestamp` 类型，其余时间类型定义为 `int(11) UNSIGNED`，如精度为微秒定义为 `bigint(20) UNSIGNED`，代码中定义为 `Long` 类型。

> 如：活动开始时间、活动结束时间等，因为所有字段都必须为 NOT NULL，当时间字段为非必填时，timestamp 不好设置默认值。例如，下线操作需记录下线时间，数据创建时下线时间应为空，如为 timestamp 类型则不得不设置一个默认值，而设置为 int 类型，则可设置其默认值为 0。

- 当 `varchar` 长度超过 4096，定义字段类型为 `text`，独立出来一张表，用主键来对应，避免影响其它字段索引效率


### 字段设计规范

- 字段允许适当冗余，以提高查询性能，但必须考虑数据一致。冗余字段应遵循：

1）不是频繁修改的字段
2）不是 `varchar` 超长字段，更不能是 `text` 字段

> 正例：商品类目名称使用频率高，字段长度短，名称基本一成不变，可在相关联的表中冗余存储类目名称，避免关联查询。


### 公共字段定义

即每个表都应设置的字段，包括：

- 主键：主键使用雪花算法

```sql
`id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
PRIMARY KEY (`id`),
```

- 删除状态

```sql
`del_status` tinyint(4) UNSIGNED NOT NULL DEFAULT '0' COMMENT '删除状态(0:未删除;1:已删除)',
```

- 创建时间

```sql
`create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
```

- 更新时间

```sql
`update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
```

示例：

```sql
CREATE TABLE `t_activity` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(128) NOT NULL DEFAULT '' COMMENT '活动名称',
  `del_status` tinyint(4) UNSIGNED NOT NULL DEFAULT '0' COMMENT '删除状态(0:未删除,1:已删除)',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='活动表';
```


### 常用字段定义

- 时间类型

```sql
`start_time` int(11) UNSIGNED NOT NULL DEFAULT '0' COMMENT '活动开始时间',
`end_time` int(11) UNSIGNED NOT NULL DEFAULT '0' COMMENT '活动结束时间',
```

- 排序值

```sql
`ordinal` int(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '排序值',
```


