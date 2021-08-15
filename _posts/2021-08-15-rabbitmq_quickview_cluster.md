---
layout: post
title:  "RabbitMQ 살펴보기 - RabbitMQ Cluster"
date:   2021-08-15 15:30:00 +0900
description: "RabbitMQ 살펴보기 - RabbitMQ Cluster"
categories: develop
---

### RabbitMQ 살펴보기 - RabbitMQ Cluster

#### RabbitMQ Cluster 구성

https://www.rabbitmq.com/clustering.html

<br>

#### erlang 의 역할

erlang VM의 native IPC (TCP/IP) 로 메세지/상태/설정 공유 및 동기화  

![rabbitmq_cluster]({{"/img/rabbitmq_quickview/rabbitmq_cluster.png"| relative_url}})


<br>

#### RabbitMQ Cluster 구성 요소

![rabbitmq_cluster_comp]({{"/img/rabbitmq_quickview/rabbitmq_cluster_comp.png"| relative_url}})

rabbitmpctl
- rabbitmq 클러스터 cil
- 노드 추가 / 삭제, Disk 노드 <-> RAM 노드 간 전환
- 보안을 위해 erlang 의 cookie 를 사용하여 클러스터와 통신 ( rabbitmqctl 이 erlang cookie 경로 접근 권한 있어야 함 )

<br>

Node (Disk Node, RAM Node)
- 노드 유형 : DISK 노드, RAM 노드
- 노드 유형에 따라 클러스터 런타임 상태를 DISK 또는 RAM 에 저장 ( 메세지는 노드 유형이 아닌 delivery-mode 에 따라 DISK 또는 RAM 에 저장 )
- 런타임 상태 : 익스체인지, 큐, 바인딩, 가상호스트, 사용자, 정책 등의 정보
- 클러스터에는 하나 이상의 DISK 노드 유지 ( 디스크 노드 개수 == 장애 복원력 )
- 복수개의 DISK 노드는 서로간 동기화 설정 하는 것을 권장

<br>

Management Node
- 통계 노드
- 클러스터 노드에서 통계 및 상태 데이터 수집
- 전용 노드를 통계 노드로 설정 가능 ( default: 클러스터 내 하나의 노드가 특정시간대에 통계 노드 역할 )
- rabbitmq-management 플러그인을 사용하여 통계 노드 시각화

<br>

#### RabbitMQ Cluster 최적화

Cluster 구성 및 권장 운영 방식
- 노드간 동기화 성능을 위해 동일 LAN 환경에서 클러스터 운영을 권장
- 최대 32-64 노드가 최적

<br>

Cluster & Consumer 구성 최적화

![rabbitmq_cluster_perf]({{"/img/rabbitmq_quickview/rabbitmq_cluster_perf.png"| relative_url}})

<br>

- 큐가 존재하는 노드로 Consumer 가 바로 연결되어야 메세지 전달 성능 최적화 가능
- HA 큐를 사용할때만 적합한 노드에 직접 접속하여 노드간 통신 및 메세지 소비 성능 최적화가 가능
- 큐가 존재하는 노드로 Consumer 가 연결되지 않으면 노드 끼리 통신이 증가하여 성능 저하

<br>
<br>

#### RabbitMQ Federation

![rabbitmq_federation]({{"/img/rabbitmq_quickview/rabbitmq_federation.png"| relative_url}})
*Cluster A -> Cluster B 로 Federation 구성*

<br>

Exchange
- 동일한 이름의 Exchange 를 up-stream, down-stream 에 생성하고 Exchange policy 를 down-stream 에 설정해야 federation 이 구성된다

<br>

ClusterB 의 Consumer
- down-stream Federation Plugin
- Consumer 와 비슷하게 동작
- Exchange Policy (정책)에 따라 up-stream 노드에 연결 및 메세지를 수신하는 작업 큐를 생성
