---
layout: post
title: "错误码规范"
description: "定义好错误码规范，使得用户或调用方在提供错误码的情况下，就能准确定位系统问题所在。"
summary: "定义好错误码规范，使得用户或调用方在提供错误码的情况下，就能准确定位系统问题所在。"
tags:
---

# 规范
## 总规
错误代码为 7 位数字，第 1 位代表错误类型，第 2-4 位代表业务编码，第 5-7 位代表具体错误原因。


## 第 1 位：错误类型
数字越小，严重性越高，影响的范围越大。
示例：
- 1 开头：数据库错误，数据不存在、关联异常等
- 2 开头：系统错误（内部组件错误），依赖的 Redis、MySQL、CMQ、RMQ、加密等基础组件异常
- 3 开头：第三方服务异常，如 PUSH SDK、微信 API、mesh 异常及非 0 返回码
- 4 开头：参数错误，参数类型/参数内容不合法
- 5 开头：业务逻辑错误，如活动、订单等场景、无操作权限


## 第 2-4 位：业务编码
000：公共错误编码


## 第 5-7 位：具体错误码
如果某个服务中存在需要通过错误码来进行逻辑处理的场景，那就各自单独增加。


# 常见错误原因（全局错误码）

| 错误码 | 错误代码 | 错误描述 |
| --- | --- | --- |
| 1000001 | DB_OP_ERROR | 数据库操作失败 |
| 1000002 | DATA_ALREADY_DELETED | 数据已被删除，请勿重复操作 |
| 1000003 | DB_NO_RECORD | 未找到记录 |
| 2000001 | INTERNEAL_ERROR | 内部错误 |
| 3000001 | THIRD_SERVICE_ERROR | 调用第三方服务出错 |
| 3000002 | ENCRYPT_ERROR | 加密错误 |
| 3000003 | DECRYPT_ERROR | 解密错误 |
| 4000001 | PARAM_INVALID | 参数无效 |
| 4000002 | ENUM_INVALID | 枚举值无效 |
| 6000001 | BIZ_ERROR | 业务逻辑错误 |
| 6000002 | PERMISSION_DENY | 无权限 |


# 错误码要素
- 错误码
- 现象&原因
- 解决方法