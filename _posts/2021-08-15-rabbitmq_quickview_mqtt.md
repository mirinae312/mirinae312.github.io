---
layout: post
title:  "RabbitMQ 살펴보기 - MQTT"
date:   2021-08-15 15:30:00 +0900
description: "RabbitMQ 살펴보기 - MQTT"
categories: develop
---

### RabbitMQ 살펴보기 - MQTT

#### MQTT - Message Format

![mqtt_msg_format]({{"/img/rabbitmq_quickview/mqtt_msg_format.png"| relative_url}})

Message Type
- 메세지 유형(method)
- CONNECT, PUBLISH, SUBSCRIBE

DUP
- 재전송 메세지인지 여부

QoS
- 메세지 전송 품질 설정
- Onec-At-Most, At-Least-Once, Exactly-Once

Retain
- 발행한 최신 메세지를 Broker 에 유지하여, 신규 Subscribe 가 최신 정보(메세지)를 동기화 할 수 있게 해준다.

Message ID
- MQTT 연결 식별자, 16bit unsigned Int
- Message Payload 내의 Topic Name, MessageID 는 ‘PUBLISH’ 메세지 일때의 가변 헤더를 나타낸다.
