---
sidebar_position: 5
---

# API服务
## 简介
API服务在设备管理和确保最佳性能方面发挥着至关重要的作用。它使用户能够通过一组API执行管理操作。这些操作包括代理发布、订阅、取消订阅消息和设备踢出。
此外，用户可以使用API清除不必要的inbox中的cleanSession = false的消息，从而提高整体系统性能。
## API列表
### 订阅(`/sub`)
#### 描述
用户可以调用 `/sub` 来做代理订阅。订阅中有三种类型的inbox，即 cleanSession = true、cleanSession = false 和外部inbox。

每种inbox类型的行为略有不同。前两种inbox是BifroMQ内部的，而第三种是外部的，满足用户的自定义业务。
#### 关键参数
* `tenant_id`：用户的tenantId。
* `topic_filter`：用户想要订阅的topicFilter。
* `sub_qos`：topicFilter的订阅QoS。
* `inbox_id`：用户想要进行代理订阅的inboxId。
* `subbroker_id`：inbox类型，即cleanSession = true、cleanSession = false和外部类型。
* `deliverer_key`：用于消息传递的键值，对于外部inbox而言该参数是必需的。
#### 返回值
如果订阅成功，将返回请求的QoS。如果订阅失败，返回的QoS将为`128`。
#### 注意点
* 对于外部inbox，API服务只在BifroMQ中记录感兴趣的topicFilter。每个外部inbox必须维护其感兴趣的topicFilter列表。
* `deliverer_key`参数对于外部inbox是必需的。相同`deliverer_key`的inbox将使用相同的RPC通道进行消息传递，因此用户应谨慎设置该值。
* 对于cleanSession = false的inbox，它们可能当前没有连接到BifroMQ，因此订阅只处理记录topicFilter。如果有任何保留的消息，它们不会被推送。
然而，对于cleanSession = true的inbox，保留的消息将被推送到相应的设备。
### 取消订阅(`/unsub`)
#### 描述
取消订阅API允许用户取消订阅特定的topicFilter。与订阅过程类似，外部inbox负责管理其感兴趣的topicFilter列表，用户应有效地管理外部inbox的
`deliverer_key`。
#### 关键参数
* `tenant_id`: 用户的tenantId。
* `topic_filter`: 用户想要取消订阅的topicFilter。
* `inbox_id`: 用户想要进行代理取消订阅的inboxId。
* `subbroker_id`: inbox类型，即cleanSession = true、cleanSession = false和外部类型。
* `deliverer_key`: 用于消息传递的键值，对于外部inbox而言该参数是必需的。
#### 返回值
如果取消订阅成功，将返回`200`状态码。否则，如果订阅失败，将返回`NOT_FOUND`，如果topicFilter检查失败，将返回`FORBIDDEN`。
### 发布(`/pub`)
#### 描述
用于消息发布的API（`/pub`），允许用户向指定主题发布消息。它还支持保留消息。
#### 关键参数
* `tenant_id`: 用户的tenantId。
* `topic`: 发送消息对应的主题。
* `client_type`: 发送消息的客户端类型。
* `retain`: 是否需要保留消息，若是则将其置为true。
#### 注意点
在消息发布阶段，使用名为`pub_qos`的参数来表示发布者所需的服务质量（QoS）。需要注意的是，由于HTTP协议缺乏原生的QoS支持，因此该参数并不指定发
布者与BifroMQ之间的消息传递的QoS。如果API服务强制将QoS降级为`AT_MOST_ONCE`，则可能会间接影响BifroMQ和订阅者之间的QoS。因此，API服务保留
了QoS信息。此外，用户（特别是发布者）需要管理维护自己与BifroMQ之间的实际QoS。
### 互踢(`/kill`)
#### 描述
用户可以使用该API完成相同clientId的设备的互踢。
#### 关键参数
* `tenant_id`: 用户的tenantId。
* `user_id`: 将被踢下线的设备的user_id。
* `client_type`: 主动发起互踢的设备类型。
### 清理