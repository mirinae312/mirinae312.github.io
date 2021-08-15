---
layout: post
title:  "RabbitMQ 살펴보기 - Queue, Exchange, Message 처리"
date:   2021-08-15 15:30:00 +0900
description: "RabbitMQ 살펴보기 - AMQP"
categories: develop
---

### RabbitMQ 살펴보기 - Queue, Exchange, Message 처리

#### Queue 종류

자동삭제 큐

![queue_auto_del]({{"/img/rabbitmq_quickview/queue_auto_del.png"| relative_url}})

- 소비자가 모두 제거(채널 종료)되면 큐도 삭제됨
- 소비자의 수는 제한 없음

<br>

독점 큐

![queue_monopoly]({{"/img/rabbitmq_quickview/queue_monopoly.png"| relative_url}})

- 소비자가 제거(채널 종료)되면 큐도 삭제됨
- 단일 소비자만 허용

<br>

자동 만료 큐

![queue_ttl]({{"/img/rabbitmq_quickview/queue_ttl.png"| relative_url}})

- 큐가 미사용 상태(소비자가 없거나, 메세지 요청이 없는) 가 TTL 로 지정된 시간동안 지속되면 큐 삭제

<br>

영구적인 큐

![queue_perm]({{"/img/rabbitmq_quickview/queue_perm.png"| relative_url}})

- Queue.Delete 명령 호출 전까지 유지됨
- 서버 재시작시에도 메세지 유지됨
- 메세지의 만료(x-message-ttl) 지정 가능
- 메세지의 최대 보관 개수(x-max-length) 지정 가능


<br>
<br>

#### Exchange

Exchange 란?
- 메세지 브로커에서 큐에 메세지를 전달하는 컴포넌트
- 메세지 데이터/속성을 분석하여 메세지에 라우팅 동작을 정의할 수 있다.
- RabbitMQ 내에 서로다른 라우팅 동작을 처리하는 여러 유형의 Exchange 가 존재
- (플러그인을 사용해) 커스텀 Exchange 정의 가능

<br>

#### Exchange 유형 : exchange_type = direct

![exchange_direct]({{"/img/rabbitmq_quickview/exchange_direct.png"| relative_url}})

- routinng-key 에 바인딩된 모든 queue 에 메세지를 전달한다

<br>

#### Exchange 유형 : exchange_type = fanout

![exchange_fanout]({{"/img/rabbitmq_quickview/exchange_fanout.png"| relative_url}})

- Exchange 에 바인딩된 모든 queue 에 메세지를 전달한다
- routing-key를 확인할 필요가 없다 —> 성능 향상

<br>

#### Exchange 유형 : exchange_type = topic

![exchange_topic]({{"/img/rabbitmq_quickview/exchange_topic.png"| relative_url}})

- routing-key 에 매칭되는 패턴에 Bind 되어있는 큐로 메세지가 전송된다.

<br>

#### Exchange 유형 : exchange_type = x-consistent-hash

![exchange_hash]({{"/img/rabbitmq_quickview/exchange_hash.png"| relative_url}})

- 메세지의 routing-key 또는 ‘hash-header’ 로 지정된 header의 값을 hash 하여 여러 큐에 분산 (load-balance)
- 큐에 가중치를 설정하여 load-balance 정도를 조절할 수 있음
- RabbitMQ 에서 plugin 형식으로 배포되는 Exchange












#### Reject Message 처리

![reject_msg_proc]({{"/img/rabbitmq_quickview/reject_msg_proc.png"| relative_url}})

- requeue 플래그가 설정된 경우, 거부된 메세지가 다시 큐로 들어간다
- x-dead-letter-exchange 가 설정되어있는 경우, 거부된 메세지는 dead letter exchange 로 라우팅되고, 연동된 큐로 들어가게 된다
- (rpc 응답) Basic.Reject : 한번에 하나의 메세지만 거부 가능 (AMQP 스펙)
- (rpc 응답) Basic.Nack : 한번에 여러 메세지 거부 가능
