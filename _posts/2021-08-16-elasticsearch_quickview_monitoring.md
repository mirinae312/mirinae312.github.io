---
layout: post
title:  "ElasticSearch 살펴보기 - Monitoring"
date:   2021-08-16 11:00:00 +0900
description: "ElasticSearch 살펴보기 - Monitoring"
categories: develop
---

### ElasticSearch 살펴보기 - Monitoring

#### ES health check

![health_status]({{"/img/elasticsearch_quickview/health_status.png"| relative_url}})

status
- GREEN
  - 모든 shard 가 정상힐 때
- YELLOW
  - primary shard 정상
  - replica shard 비정상 ( node 재부팅시 shard 가 초기화 될때 잠시 yellow 상태가 될 수 있다)
- RED
  - primary shard 일부 비정상

<br>

cluster health check API
- ```GET /_cluster/health``` : ES 클러스터 상태 조회
- ```GET /_cluster/health?level=indices``` : 클러스터의 모든 index 상태 조회
- ```GET /_cluster/health?level=shards``` : 클러스터의 모든 index + shard 상태 조회
- ```GET /_cluster/health/{index-name}``` : 클러스터의 특정 index 상태 조회 (기본 정보만 출력)
- ```GET /_cluster/health/{index-name}?level=indices``` : 클러스터의 특정 index 상태 조회 (index 상세 정보 출력)
- ```GET /_cluster/health/{index-name}?level=shards``` : 클러스터의 특정 index 상태 조회 (index + shard 상세 정보 출력)

<br>

#### Cluster Status

cluster/node 등의 상태 정보 및 설정 내역 확인 API
- ```GET /_cluster/state``` : cluster 상태/설정 정보 조회
- ```GET /_nodes``` : 전체 node 상태/설정 정보 조회
- ```GET /_nodes/{node ip} ```, ```GET /_nodes/{node id} ```, ```GET /_nodes/_local``` : 특정 node 상태/설정 정보 조회
- ```GET _cluster/health```
  - cluster.health.status : 클러스터 상태 (green, yellow, red)
  - cluster.health.number_of_nodes : 노드 개수
  - cluster.health.initializing_shards : 초기화 중인 샤드 개수
  - cluster.health.unassigned_shard : 미할당 샤드 개수

<br>

#### Cluster Monitoring 통계 조회

cluster/node/index 등의 모니터링 통계 출력
- ```GET /_cluster/stats``` : cluster 전체 통계 조회 , cluster, node 정보 (role, OS, cpu usage, mem usage 등)
- ```GET _stats``` : 인덱스 통계 조회

<br>

#### Cluster Monitoring (_cat API)

linux shell 형태로 api 결과 출력
- ```curl http://localhost:9200/_cat``` : 사용가능한 _cat API 목록 출려

_cat API 파라미터
- ```?v``` : 결과 header 출력
- ```?help``` : 각 컬럼(header) 상세 정보 출력
- ```?h={column name},{column name}``` : 선택 컬럼만 출력

<br>

#### 성능 지표 조회

검색 성능

```
// node 통계
GET _nodes/stats
GET _nodes/{node name}/stats
GET _nodes/{node name}/stats/indices/search

// index 통계
GET stats
GET {node name}/stats
```

- indices.search.query_total : 실행 쿼리 수
- indices.search.query_time_in_millis : 쿼리 실행 시간 합계
- indices.search.query.current : 실행중인 쿼리 수
- indices.search.fetch_total : 조회 수
- indices.search.fetch_time_in_millis : 조회 시간 합계
- indices.search.fetch_current : 실행중인 조회 수

<br>

색인 성능

```
// node 통계
GET _nodes/stats
GET _nodes/{node name}/stats
GET _nodes/{node name}/stats/indices/indexing,refresh,flush

// index 통계
GET stats
GET {node name}/stats
```

- indices.indexing.index_total : 색인된 문서 총 개수
- indices.indexing.index_time_in_millis : 색인 작업 소요 시간 합
- indices.indexing.index_current : 현재 색인중인 문서 개수
- indices.refresh.total : refresh 발생 총 횟수
- indices.refresh.total_time_in_millis : refresh 작업 총 시간 합
- indices.flush.total : 디스크 flush 작업 총 횟수
- indices.flush.total_time_in_mills : 디스크 flush 작업 총 시간 합


<br>

HTTP 성능

```
// node 통계
GET _nodes/stats
GET _nodes/{node name}/stats
GET _nodes/{node name}/stats/http
```

- http.current_open : 현재 open 된 connection 개수
- http.total_opened : connection 총 개수

<br>

JVM 성능 (imx 기반 도구를 사용하는 것을 권장)

```
// node 통계
GET _nodes/stats
GET _nodes/{node name}/stats
GET _nodes/{node name}/stats/jvm
```

- jvm.mem.heap_used_percent : heap 사용률
- jvm.mem.heap_committed_in_bytes : heap 실제 사용 크기

<br>

thread pool

```
// node 통계
GET _nodes/stats
GET _nodes/{node name}/stats
GET _nodes/{node name}/stats/indices/thread_pool
```

- thread pool 에서 생성된 스레드 수
  - thread_pool.bulk.queue 
  - thread_pool.index.queue
  - thread_pool.search.queue 
  - thread_pool.merge.queue
- thread pool 에서 제거된 스레드 수
  - thread_pool.bulk.rejected
  - thread_pool.index.rejected 
  - thread_pool.search.rejected 
  - thread_pool.merge.rejected

<br>

cache 상태 측정

```
// node 통계
GET _nodes/stats
GET _nodes/{node name}/stats
GET _nodes/{node name}/stats/indices/query_cache,fileddata
```

- indices.query_cache.memory_size_in_byte : 쿼리 캐시 크기
- indices.query_cache.total_count : 쿼리 캐시 개수
- indices.query_cache.hit_count : 캐시 적중률
- indices.query_cache.miss_count : 미스 율
- indices.query_cache.evictions : eviction 횟수
- indices.fielddata.memory_size_in_bytes : fielddata 캐시 크기
- indices.fielddata.evictions : fielddata 캐시 eviction 횟수

<br>

#### 모니터링 대상이 되는 OS 주요지표
- I/O 사용률
- CPU 사용률
- Send/Receive byte 크기
- File Descriptor 사용률
- swap 사용률
- disk 사용률
