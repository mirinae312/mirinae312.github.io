---
layout: post
title:  "RabbitMQ 간단히 살펴보기"
date:   2021-08-15 21:30:00 +0900
description: "RabbitMQ 살펴보기 - AMQP"
categories: develop
---

### RabbitMQ 살펴보기 - 성능

#### Message Publish Performance vs. Delivery Guarantee

![msg_publish_perf]({{"/img/rabbitmq_quickview/msg_publish_perf.png"| relative_url}})

<br>

Notification on failure
- mandatory = true : 메세지를 라우팅할 수 없는 경우 publisher 에게 메세지를 다시 반환한다.

Publisher confirms
- 큐 소비자가 메세지를 사용하거나, 메세지의 디스크 저장이 성공하면 브로커가 publisher 에 Basic.Ack 전송
- 메세지 라우팅이 불가한 경우 브로커가 publisher 에 Basic.Nack 를  전송
- ‘Publisher confirms’ 기능은 트랜잭션과 함께 사용할 수 없으며, Basic.Ack/Nack 처리를 비동기로 수행된다

Alternate exchanges
- 처음 Exchange 선언시, 대체 Exchange 를 함께 선언할 수 있다.
- Exchange 가 라우팅할 수 없는 경우, 메세지는 대체 Exchange 로 전달된다.

HA Queues
- AMQP 스펙 아님, RabbitMQ 만의 기능
- RabbitMQ 클러스터 모든/특정 노드에 저장되는 큐를 정의하여 가용성을 보장할 수 있다.

Transactions
- AMQP 의 Transaction, Commit/Rollback 스펙을 따른다.
- RabbitMQ는 모든 명령이 단일 큐에 영향을 줄때만 트랜잭션 원자성(atomicity) 을 보장해준다.
- delivery-mode: 2 인 경우, 디스크에 저장하는 I/O 로 인해 성능 문제가 발생할 수 있다.

HA Queues w/ Transactions
- HA 큐에서 트랜잭션 처리를 해줌

Persisted Messages
- delivery-mode: 2 로 메세지를 디스크에 저장
- 클러스터 node fail 에도 메세지를 유지하도록 해준다
- I/O 과부하/병목이 RabbitMQ 성능 저하로 이어질 수 있다


<br>

#### Message Get / Consume Performance

![msg_consume_perf]({{"/img/rabbitmq_quickview/msg_consume_perf.png"| relative_url}})

Consuming with acks and QoS > 1
- 한번에 여러개의 메세지를 가져온다 (prefetch)
- 한번에 가져온 (prefetch 된) 메세지에 대해 하나의 ack를 전송한다 (가져온 모든 메세지가 확인되었음을 알림)
- Rabbit MQ 의 QoS == Message Prefetch Count

Consuming with no-acks mode enabled
- 메세지를 소비자에게 보내는 가장빠른 방법
- 소비자의 메세지 수신여부를 확인(ack)하지 않는다
- RabbitMQ 는 수신자의 소켓 버퍼(rmem_default, rmem_max)가 다 찰때까지 메세지를 계속 전송한다

Consuming and using transactions
- 대부분의 경우 성능이 좋지 않다
- (QoS 설정을 사용하지 않는 경우에도) 트랜잭션을 사용해 메세지 확인 응답(ack)을 일괄 처리하면 성능이 향상될 수 있다
- 메세지 수신이 비활성화된(no_ack=true) 경우에는 트랜잭션이 동작하지 않는다
