---
sidebar_position: 2
title: "关于MQTT"
---

## 为什么在物联网中使用 MQTT 协议？与 HTTP 协议相比有哪些优势？

物联网设备通常需要发送和接收小数据包，例如传感器读数、设备状态等。MQTT（消息队列遥测传输）是一种基于发布/订阅模式的轻量级通信协议，专为低带宽
和不可靠网络环境设计，非常适合物联网应用。

与 HTTP 协议相比，MQTT 协议具有以下优势：

- **更轻量级**：与 HTTP 笨重的协议头相比，MQTT 协议非常简洁，其消息体比 HTTP 小，能够减轻网络传输负担。
- **更低功耗**：MQTT 协议的低功耗特性使其非常适合物联网设备，例如传感器、智能家居等，这些设备的电池寿命通常非常有限。
- **更高效**：MQTT 协议采用推送方式传递消息，可以大大减少客户端和服务器之间的通信开销。同时，MQTT 协议的 QoS 机制还能确保消息的可靠传输，
  使通信更加稳定高效。
- **更具扩展性**：MQTT 协议采用主题的发布/订阅模式，使系统具有更高的扩展性，能够快速适应新设备和应用场景。相比之下，HTTP 协议需要在服务器端
  部署 RESTful 接口，限制了系统的扩展性。

总的来说，MQTT 协议适用于需要低功耗、小数据包和高效传输的物联网设备，而 HTTP 协议更适用于需要大数据量传输和更复杂请求/响应的应用场景。

---

## 如何处理客户端与服务器之间的异常断开？

鉴于物联网设备经常工作在不稳定的网络环境中，MQTT 协议和常见的开源客户端实现对异常断开都有相应的设计和处理。

请为客户端设置自动重连功能，并在客户端负责订阅主题的情况下，设置重连后自动重新订阅。

如果不希望在断开期间丢失重要消息，可以在订阅设置中将 `cleanSession` 设置为 `False`。但请谨慎使用此设置，并合理设置 `topicfilter`，
将重要消息与大量普通消息分开，仅对最必要的消息订阅使用。大量不合理使用 `cleanSession=False` 的设置会带来严重的性能压力和更大的消息丢失风险。

如果出现意外的高频率断开现象，请检查从客户端到服务器的网络链接。

---

## 为什么会出现消息丢失，订阅者未收到成功发送的消息？

消息丢失可能有多种原因：

- **网络不稳定导致数据包丢失**：根据物联网设备的具体网络环境，可能会发生不同程度的数据包丢失。如果需要确保消息到达，可以使用 `qos=1` 
  设置发送和订阅消息，但这也会消耗更多性能。
- **订阅者断开连接**：如果某个主题未被订阅，该消息会被直接丢弃。
- **单连接收发消息超出限制**：默认情况下，单个客户端每秒限制为 200 条消息，带宽为 512MB。如果是发送方，超出限制的消息将发送失败；
  如果是订阅者，超出限制且 `cleanSession=True` 的情况下，消息将被丢弃。
- **消息发送频率过高导致下游溢出**：可能出现 `cleanSession=True` 下的订阅溢出、规则引擎下游溢出、`cleanSession=False` 
  下的收件箱消息缓存溢出等情况。可以合理拆分主题，降低每个订阅者接收的消息频率，或使用共享订阅。
- **资源不足**：如果 BifroMQ 服务部署的内存或 CPU 资源不足以支撑业务规模，也可能出现这种情况，需要合理评估和扩展业务规模。

---

## 订阅者接收的消息顺序是否与发送顺序一致？

根据 MQTT 协议，我们保证消息顺序。

特别是在 QoS 0 下，单个发送者发送的消息必须按照发送顺序到达。在 QoS 1 下，由于存在消息重发的情况，可能会偶尔出现顺序混乱，但在消息重发后，
根据协议，重发消息后的消息会再次按顺序发送，确保客户端接收的消息序列中至少有一个子序列是完全有序的。

---

## 为什么会出现连接-断开-重连-断开的重复现象？

通常是由于设备间的互相踢出导致的。`ClientID` 必须唯一，否则已经连接的冲突设备会被断开。如果两个设备都设置了自动重连，
那么它们会不断互相踢出，从而导致此问题。
