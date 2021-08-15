---
layout: post
title:  "RabbitMQ 살펴보기 - AMQP"
date:   2021-08-15 15:30:00 +0900
description: "RabbitMQ 살펴보기 - AMQP"
categories: develop
---

### RabbitMQ 살펴보기 - AMQP

#### AMQP 란?

AMQP : Advanced Message Queing Protocol

오픈소스 기반 MQ 표준 프로토콜
- 효율적 메세지 전송을 위한 MQ 구성요소 및 메세지 포맷 등을 정의

<br>

#### AMQP 구성요소

![amqp_comp]({{"/img/rabbitmq_quickview/amqp_comp.png"| relative_url}})

<br>

Queue
- 메세지를 저장하는 디스크/메모리 상 자료구조
- 메세지를 메모리 혹은 디스크에 저장할지 큐별로 설정 가능

Binding
- (Binding 을 사용해) 큐와 Exchange 의 관계 정의
- Binding과 Binding-key 로 Exchange 에 전달된 메세지가 어느 큐에 저장돼야 하는지 정의

Exchange
- 메세지 브로커에서 큐에 메세지를 전달하는 컴포넌트
- 메세지 데이터/속성을 분석하여 메세지에 라우팅 동작을 정의할 수 있다.
- RabbitMQ 내에 서로다른 라우팅 동작을 처리하는 여러 유형의 Exchange 가 존재
- (플러그인을 사용해) 커스텀 Exchange 정의 가능


<br>

#### AMQP - Message Publish

![amqp_msg_publish]({{"/img/rabbitmq_quickview/amqp_msg_publish.png"| relative_url}}){: width="500"}

AMQP 명령은 class 와 method 로 구성됨
- ex. ‘Connection.Start’ 명령은 ‘Connection’ class 와 ‘Start’ method 로 구성된 것

Routing key 를 전달하여 Exchange 와 Queue 를 연결한다.
- Queue.BindOk 단계에서 연결이 완료된다.

<br>

#### AMQP - Message Consume

![amqp_msg_consume]({{"/img/rabbitmq_quickview/amqp_msg_consume.png"| relative_url}}){: width="500"}

Consumer Tag
- Consumer 애플리케이션을 식별하는 고유한 문자열
- Basic.Consume 명령 실행시 생성된다
- 메세지 수신 작업을 취소할 때 사용할 수 있다

AMQP 명령은 class 와 method 로 구성됨
- ex. ‘Connection.Start’ 명령은 ‘Connection’ class 와 ‘Start’ method 로 구성된 것

Basic.Consume: Message Consume 방식
- Basic.Consume 은 push 기반 메세지 소비 모델 (Basic.Get 이라는 polling 모델도 있다)
- Basic.Consume 이 Basic.Get 대비 2배 빠르다.
- 비동기로 소비자 애플리케이션에 메세지 전달


<br>

#### AMQP - Commamd Format

AMQP 모든 명령은 프레임 형태로 전송됨

![amqp_cmd_format]({{"/img/rabbitmq_quickview/amqp_cmd_format.png"| relative_url}})

<br>

프레임유형
- Protocol Header Frame : RabbitMQ 연결시 한번만 사용됨
- Method Frame : RPC 요청이나 응답을 의미
- Content Header Frame : 메세지 크기와 속성을 포함
- Body Frame : 메세지 내용을 포함
- Heartbeat Frame : 클라이언트/서버의 heartbeat 확인 용도

채널번호
- 하나의 AMQP connection 에서 여러개의 channel 을 처리할 수 있음  (하나의 connection 내에 여러 channel 을 이용한 multiplexing 처리)
- 하나의 connection 에서 여러 channel 을 처리할수록 메모리 사용률 증가

프레임크기
- 최대 Frame 크기 : 31 bits 로 표현가능한 범위

<br>

#### AMQP - Message Format

RabbitMQ 메세지 발행은 ‘Method Frame (Basic.Publish) + Content Header Frame + Body Frame (1개 이상)’ 으로 구성된다

![amqp_msg_format]({{"/img/rabbitmq_quickview/amqp_msg_format.png"| relative_url}})

<br>


#### AMQP - Message Format : Basic.Properties

![amqp_msg_basic_proerties]({{"/img/rabbitmq_quickview/amqp_msg_basic_proerties.png"| relative_url}})
