---
layout: post
title:  "ElasticSearch 살펴보기 - Lucene"
date:   2021-08-16 11:00:00 +0900
description: "ElasticSearch 살펴보기 - Lucene"
categories: develop
---

### ElasticSearch 살펴보기 - Lucene

#### Lucene
- ES 각 샤드 내에서는 Lucene 을 통해 검색 (ES의 각 shard == 독립된 Lucene 검색 엔진)

![lucene]({{"/img/elasticsearch_quickview/lucene.png"| relative_url}})

<br>

translog
- 샤드의 변경사항을 저장 (redo 로그 역할)
- 장애복구, 데이터 유실방지
- 일정크기 이상이되면 Flush 작업 후 삭제

<br>

index writer
- segment 를 생성하여 색인 처리

<br>

index searcher
- segment 를 이용해 검색 처리

<br>

Segment
- 역색인 정보
- 변경 불가, Immutable
  - Lock 필요없음
  - 시스템 캐시의 효율적 사용으로 성능 향상
- 정기적으로 segment를 병합한다.
  - segment 개수 최적화를 위한 작업

<br>

commit point
- segment 목록 관리
