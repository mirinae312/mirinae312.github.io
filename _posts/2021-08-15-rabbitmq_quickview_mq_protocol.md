---
layout: post
title:  "RabbitMQ 살펴보기 - MQ Protocol"
date:   2021-08-15 21:30:00 +0900
description: "RabbitMQ 살펴보기 - MQ Protocol"
categories: develop
---

### RabbitMQ 살펴보기 - MQ Protocol

여러개의 MQ Protocol 을 RabbibMQ 에서 지원하며 각각의 protocol 은 아래와 같은 장단점이 있음

#### AMQP
- 안정적 네트워크 환경에 적합
- Pub/Sub + Exchange + Queue 로 구성
- 최대 16 EB 메세지 크기 제한 (RabbitMQ 에서는 2GB 로 제한)
- 연결 유지 프로토콜

#### MQTT
- 안정적이지 못한 네트워크 환경에 적합 (ex. 모바일 환경)
- Pub/Sub 으로만 구성됨
- 최대 256MB 메세지 크기 제한 (RabbitMQ 에서는 2GB 로 제한)
- 연결 유지 프로토콜

#### STOMP
- 스트림 기반 처리가 가능한 구조
- 메세지 프레임이 null(0x00) 바이트로 끝나는 명령과 payload 가 텍스트로 표현됨 (HTTP와 유사)
- 연결 유지 프로토콜

#### statelessd
- 고성능 메세지 발행 가능
- 연결 유지X(stateless) 프로토콜
- http 프로토콜 사용
