---
layout: post
title:  "ElasticSearch 살펴보기 - API"
date:   2021-08-16 11:00:00 +0900
description: "ElasticSearch 살펴보기 - API"
categories: develop
---

### ElasticSearch 살펴보기 - API

### Document API

#### Document API 공통

- ID : 문서 추가시 id 를 제공하지 않으면, UUID 로 _id 값을 생성
- Version : Update API 사용시 ES 내부에서는 snapshot 을 생성하여 문서 수정 후 재색인을 진행 (문서 갱신/삭제 마다 version 이 증가함)
- op_type : ES 에 문서 저장 기본 동작은 upsert. ( op_type=create 파라미터를 추가하면 insert 동작만 허용한다)
- timeout : 기본 API 타임아웃은 1분 (timeout=5m 으로 조정 가능)
- source_exclude : 문서 조회시 특정 필드 (대용량 필드)를 제외하고 결과를 반환 (ex. source_exclude={필드명} )

<br>

#### Document API 종류

| API |   | 주의 사항 |
|-----|---|---------|
| INDEX | | - 문서 생성 <br> - 응답의 _shards 필드에 복제 성공/실패 샤드 개수 출력 | |
| GET | - 문서 조회 <br> - 조회되는 문서내용은 _source 필드에 출력 | - 대용량 필드 출력 방지를 위해 source_exclude 파라미터 사용을 권장 |
| DELETE | - 문서/인덱스 삭제 | - 인덱스 삭제시 모든 문서 삭제되고 복구 불가 |
| _delete_by_query | - 조건에 맞는 문서 삭제 | - 대량 수정 작업중에 삭제 API 호출시 version 이 일치하지 않아 version_conflicts 항목을 통해 삭제 실패 건수 확인 필요 |
| UPDATE | - 인덱스에서 문서를 가져와 스크립트 수행후 재색인(reIndex)하는 방식 <br> - 스크립트 사용하여 문서 업데이트 가능 | |
| _bulk | - 대량 색인 <br> - CUD 연산을 혼합하여 수행가능 | - API 실행중 실패가 발생해도 이미 갱신/수정된 결과는 롤백 불가 (항상 처리 결과를 확인해야 함) |
| _reindex | - 한 인덱스에서 다른 인덱스로 문서 복사 | - sort 성능 최적화 등을 위해 활용 가능 |

<br>

### 검색 API

#### 검색API 동작 방식

![search_api]({{"/img/elasticsearch_quickview/search_api.png"| relative_url}})

<br>

- 검색 요청은 모든 shard 에 broadcasting 됨 
- broadcasting 방식 선택가능 
  - round-robin (default)   
  - 동적 분배 방식     
    - 검색 응답시간, Thread Pool 크기 등에 adaptive 하게 샤드를 동적 결정
- 각 샤드의 결과값을 조합하여 최종 결과를 출력

<br>

#### 검색 API 사용

```
// 검색 대상 shard 선택을 adaptive 하게 클러스터 설정
PUT _cluster/settings
{
  “transient” : {
    “cluster.routing.use_adaptive_replica_selection”: true
  }
}
```

```
// 검색 API 기본 타임아웃 클러스터 설정
PUT _cluster/settings
{
  “transient” : {
    “search.default_search_timeout”: “1s”
  }
}
```

<br>

### 기타 API

_shard_shards
- ```POST {인덱스}/_shard_search```
- 검색이 수행되는 노드/샤드 정보 출력
- 쿼리 최적화 및 오류 분석에 활용

_msearch
- ```POST _msearch```
  ```
  POST _msearch
  {“index”: “{인덱스}” }
  {“query”: … }
  {“index”: “{인덱스}” }
  {“query”: … }
  ```
- 여러 검색 요청을 한번에 요청
- 페이지에 필요한 데이터를 한번에 요청/구성할 때 활용

_count
- ```POST {인덱스}/_count```
- 검색된 문서의 개수만 출력

_validate
- ```POST {인덱스}/_validate```
- 쿼리의 유효성/syntax 체크
- ```?rewrite=true``` 파라미터를 추가하여 valid 실패 원인 출력 가능

_explain
- ```POST {인덱스/_doc/{_id}/_explain```
- 방금 조회한 문서({_id})의 _score 값이 계산된 이유를 출력

Profile API
- 쿼리 실행 계획, 실행 계획별 소요시간 출력
- 각 샤드별 수행 시간 출력
- 성능 튜닝/디버깅에 활용
- 예
  ```
  POST {인덱스}/_search
  {
    “profile”: true,
    “query”: { … }
  }
  ```

<br>

#### Script 기능
- script 를 사용하여 문서(doc) 수정 가능
- script 작성/사용 방식
  - config 디렉토리에 스크립트를 저장하고, 이름을 지정해서 호출하는 방식
  - in-request : API 호출시 스크립트를 정의해서 실행하는 방식
- script.disable_dynamic: false 설정 필요 (elasticsearch.yml)
- script 는 Painless 로 작성

<br>

doc 에 필드 추가 예제
- ctx._source 로 doc 에 접근

```
POST {인덱스}/_doc/{id}/_update
{
  “script”: “ctx._source.{doc내 path}.{추가할 field 명} = {값}”
}
```

<br>

doc 에 필드 제거 예제
- ctx._source 로 doc 에 접근

```
POST {인덱스}/_doc/{id}/_update
{
  “script”: “ctx._source.{doc내 path}.remove(“{삭제할 field 명}”)”
}
```

<br>

#### Template 기능
- 검색 쿼리를 template 으로 ES 에 등록해두고 사용가능하다. (RDB의 stored-procedure 같은 느낌)
- template 은 mustache 로 작성

<br>

template 생성 예제

```
POST _script/{template 이름}
{
  “script”: {
    “lang”: “mustache”
    “source”: {
      “query”: {
        “match”: {
          “{필드명}”: “{{ field_value }}”
        }
      }
    }
  }
}
```

<br>

template 실행 예제

```
POST {인덱스}/_doc/_search/template
{
  “id”: “{template 이름}”
  “params”: {
     “field_value” : “{값}”
  }
}
```
