---
layout: post
title:  "ElasticSearch 살펴보기 - Aggregation"
date:   2021-08-16 11:00:00 +0900
description: "ElasticSearch 살펴보기 - Aggregation"
categories: develop
---

### ElasticSearch 살펴보기 - Aggregation

#### Aggregation
- 인덱스를 활용한 분산 처리 가능
- 데이터가 양이 많을 수록 CPU/메모리 사용률이 높아짐 -> ES 캐시를 사용하여 최적화 가능
- 특정 필드에서 aggregation 연산을 할 때, 모든 필드 값을 메모리에 로드 -> 리소스 사용율 증가

![query_type]({{"/img/elasticsearch_quickview/query_type.png"| relative_url}})

<br>

| ES cache 종류 |   |
|--------------|---|
| node query cache | - 노드내 모든 샤드가 공유 <br> - LRU 캐시 <br> - elasticsearch.yml 에 ’index.queries.cache.enabled: true’ 설정하여 캐시 활성화 |
| shard request cache | - 샤드에서 수행된 쿼리 결과를 캐싱 <br> - 샤드의 내용이 변경되면 캐시가 삭제됨 -> 업데이트가 많은 샤드에서는 shard request cache 적용이 성능 저하를 유발 |
| field data cache | aggregation 동인 필드 값을 메모리에 캐시 |

<br>

#### Aggregation API 동작 실행 방식


쿼리 수행 결과에 대해 aggregation 결과 수행

```
  "query" : { ... } // 생략시 match._all
  "aggs" : { ... }
}
```

![agg_api_flow]({{"/img/elasticsearch_quickview/agg_api_flow.png"| relative_url}})

<br>

- bucket : 쿼리 결과로 grouping 된 문서들의 모음

<br>

분산환경에서 Aggregation 동작 방식

![agg_flow_distributed]({{"/img/elasticsearch_quickview/agg_flow_distributed.png"| relative_url}})

- coordinate node 는 각 shard 결과를 취합해서 size 만큼 최종결과 반환
- 각 shard 에 shard_size 크기만큼 1차 집계 결과 요청
- shard_size 외 결과는 누락되며, 최종 결과의 정확도에 영향


<br>

#### Aggregation Query Request / Response 예제

Request

```
{
  "query" : { ... } // 생략시 match._all
  "aggs" : {
    "<aggregation_name1>": {
      "<aggregation_type>": {
        "field": "{필드명}"
      }
    }
    [,"meta": { [<meta_data_body>] } ]?
    [,"aggregations": { [<sub_aggregations>] } ]?

    "<aggregation_name2>": {
      "global": {},  // (bucket 외에) 전체 문서에 대해서 aggregation 수행할 때 설정
      "aggs": {
        "<sub_aggregation_name1>": {
          "<sub_aggregation_type>": {
            "field": "{필드명}"
          }
        }
      }
    }
  }
}
```

<br>

Response

```
{
  "took" : 2, // 소요시간
  "timed_out" : false,  // timeout 발생 여부
  "_shards" : {             // 검색에 영향 받은 shard 정보
    "total": 5,                // - 영향받은 shard 총 개수
    "success": 3,          // - 검색이 처리된 shard 개수
    "skipped": 2,          // - 검색이 처리되지 않은(skip) shard 개수
    "failed": 0,              // - 검색이 처리 실패한 shard 개수
  },
  "hits": {                     // 검색 결과
    "total": 100,            // 검색 쿼리가 일치한 문서 총 개수
    "max_score": 100,  // 검색 결과에 포함된 문서의 score 최대값
    "hits": {}                  // 검색 결과 문서 목록 (default: 10개만 반환)
  }
  "aggregations" : {
    "<aggregation_name>": {
        // 집계 결과
    }
  }
}
```

<br>

#### Metric Aggregation
- 특정 필드에 대한 sum, avg, sort, geo_bounds 계산
- (검색 결과 출력 없이) 집계 결과만 출력해야하는 경우 (size=0 으로 쿼리)
  - ex. ```GET {인덱스명}/_search?size=0```
- 집계 연산간 각 문서에 대해 적용될 script 작성 가능
- query 작성시 constant_score 를 사용하여, score 값을 계산하지 않도록 해서 성능 최적화

<br>

| Metric Aggregation 종류 |   |
|------------------------|---|
| single-value | - 결과 값이 단일값 <br> - sum, avg 등 |
| multi-value | - 결과 값이 여러개가 될 수 있음 <br> - stats, geo_bounds |

<br>

#### Metric Aggregation 유형별 예제

sum / avg / min / max / value_count (count)

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "sum": {   // 또는 avg, min, max, value_count
         "field": "{필드명}"
         "script": { ... }  // 실행할 script (optional)
       }
     }
   }
}
```

<br>

stats
- sum, avg, min, max, value_count 를 한번에 연산

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "stats": {
         "field": "{필드명}"
         "script": { ... }  // 실행할 script (optional)
       }
     }
   }
}
```

<br>

extended_stats
- sum, avg, min, max, value_count 를 한번에 연산
- 추가 응답
  - sum_of_squares : 제곱합 (평균 편차)
  - variance : 분산
  - std_derivation : 표준 편차
  - std_derivation_bound : 표춘편차 범위
    - upper : 표준편차 상한
    - lower : 표준편차 하한

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "extended_stats": {
         "field": "{필드명}"
         "script": { ... }  // 실행할 script (optional)
       }
     }
   }
}
```

<br>

cardinality
- uniq 개수 카운트
- (정확한 값이 아닌) 근사치 반환
- ‘{필드명}’ 에 (분석기를 사용하는) text 타입이 아닌 keyword 타입을 지정해야함 (ex. "field" : xxx.yyy.keyword )
- precision_threshold 값을 지정해 정확도 조정이 가능 (0 ~ 40000 값 설정 가능)
- 정확도를 높일 수록 많은 메모리 필요
- cardinality 는 hash 를 기반으로 계산됨
  - murmur3 플러그인을 사용하면 hash 값을 미리 생성해 성능 향상 효과가 있다.
- 분산환경에서는 각 shard 에서 중복제거한 목록을 coordinate 노드로 전송해야하며, 중간 결과를 임시 저장 및 전송하기 위해 트래픽/메모리 소비가 크다

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "cardinality": {
         "field": "{필드명}"
         "script": { ... }  // 실행할 script (optional)
       }
     }
   }
}
```

<br>

percentiles
- 백분위 구간 별 count
- 근사값으로 계산
- percents 값을 사용해 집계 구간을 지정 가능
- compression 옵션으로 정확도 조절이 가능
  - 정확도를 높일 수록 메모리 사용량 증가
  - 참고: TDigest 알고리즘 (노드를 사용한 근사치 계산)

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "percentiles": {
         "field": "{필드명}"
         "percents": [10, 20, 30, ... , 90]
       }
     }
   }
}
```

percentile_rank
- 백분위 구간/위치 연산
- 근사값으로 계산

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "percentile_rank": {
         "field": "{필드명}"
         "values" : [100, 200]   // 검색하고자하는 필드값
       }
     }
   }
}
```

<br>

geo_bounds
- 지형 경계 box 계산
- geo_point 타입의 field 에 대해서만 연산 가능

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "geo_bounds": {
         "field": "{geo_point 타입의 필드명}"
         "percents": [10, 20, 30, ... , 90]
       }
     }
   }
}
```

<br>

geo_centroid
- 지형 중심 계산
- geo_point 타입의 field 에 대해서만 연산 가능

```
{
  "query" : {
    "constant_score": {
      "filter": {
        "match" : {
          "{필드명}": "{검색어}".
        }
      }
    }
  },
  "aggs": {
     "{agg_name}" : {
       "geo_centroid": {
         "field": "{geo_point 타입의 필드명}"
       }
     }
   }
}
```

<br>

#### Bucket Aggregation
- metric 계산없이 bucket 생성
- bucket 집계 결과로 bucket 을 만들고 계속 중첩된 집계 가능
  - 중첩이 많을 수록 메모리 사용률 증가
  - ES에 최대 허용 bucket 수 제한 있음 ( search.max_buckets 에 설정 가능)
- bucket
  - 집계된 결과 데이터
  - 메모리에 저장

<br>

#### Bucket Aggregation 동작 방식

![bucket_agg_flow]({{"/img/elasticsearch_quickview/bucket_agg_flow.png"| relative_url}})

<br>

- 검색 처리 노드에서 각 shard 결과를 취합해서 size 만큼 최종결과 반환
- 각 shard 에 shard_size 크기의 집계 결과 요청
- shard_size 외 결과는 누락되며, 최종 결과의 정확도에 영향

<br>

#### Bucket Aggregation 응답 포맷

```
{
 "aggregations": {
     "{agg_name}" : {
       "doc_count_error_upper_bound": 10,   // 집계 오류 상한
       "sum_other_doc_count": 1111, // 결과에 누락된 문서 개수 (남은 결과가 더 있는지 여부 판단용)
       "buckets" : [
          // 집계 결과
       ]
     }
   }
}
```

<br>

#### Bucket Aggregation 유형혈 예제

range
- from 부터 to 미만의 범위에 대한 집계 연산

```
{
 "aggs": {
     "{agg_name}" : {
       "range": {
         "field": "{필드명}"
         "ranges": [
            {
              "key": "{범위 이름}", // optional
              "from" : 1,
              "to": 100" // 100은 연산범위에 미포함, 100 미만
            },
            { "from" : 1000 },
            ...
         ]
       }
     }
   }
}
```

<br>

date_range
- 날짜 범위(from ~ to 미만)의 집계 연산
- 날짜 포맷은 ES 지원 형식만 사용 가능 (UTC)

```
{
  "aggs": {
     "{agg_name}" : {
       "range": {
         "field": "{필드명}"
         "ranges": [
            {
              "key": "{범위 이름}", // optional
              "from" : 1,
              "to": 100" // 100은 연산범위에 미포함, 100 미만
            },
            { "from" : 1000 },
            ...
         ]
       }
     }
   }
}
```

<br>

histogram
- 일정 간격 단위 집계 연산

```
{
 "aggs": {
     "{agg_name}" : {
       "histogram": {
         "field": "{필드명}"
         "interval": 1000  // 1000 간격으로 집계
         "min_doc_count": 1  
           // 최소 1개의 결과가 있을때만 출력 (optional)
       }
     }
   }
}
```

<br>

date_histogram
- year, quarter, month, week, day, hour, minute, second 등 시간 간격으로 집계 연산
- "30m" (30분 간격), "1.5h" (90분 간격) 등 지정 가능
- 날짜 포맷은 ES 지원 형식만 사용 가능 (UTC)

```
{
 "aggs": {
     "{agg_name}" : {
       "date_histogram": {
         "field": "{필드명}"
         "interval": "day"  // 일 간격으로 집계
         "format": "yyyy-MM-dd" // 집계 결과에서 시간 출력 포맷
                                                 // ( yyyy-MM-dd’T’HH:mm:ss.SSS지정 가능)
         "time_zone": "+09:00"    // 날짜를 한국시간(+09:00) 으로 변환하여 연산
         "offset": "+1h"                 // 01 시부터 다음날 01까지 day 간격으로 집계
       }
     }
   }
}
```

<br>

terms
- ‘{필드명}’ 필드에 대해 빈도수가 높은 term 순위의 집계
- bucket 이 동적으로 생성되는 다중 bucket 집계 연산
- ‘{필드명}’ 에는 (형태소분석이 필요없는) keyword 데이터 타입을 명시해야 함
- (정확한 값이 아닌) 근사치를 계산
  - size, shard_size 지정으로 정확도 향상 가능
  - 값이 클수록 메모리 사용량 및 연산 사간 증가

```
{
 "aggs": {
     "{agg_name}" : {
       "terms": {
         "field": "{필드명, xxx.keyword}". // keyword 타입을 명시해야 함
         "size":  5 // 반환할 결과값 크기 (default: 10), 값이 클수록 정확도 향상
         "shard_size": 100  // 각 샤드에 최초 수행되는 집계 결과 크기
      }
     }
   }
}
```


<br>

#### Pipeline Aggregation
- 다른 집계 결과로 생성된 bucket 에 대해 추가적인 집계 연산 수행
- Parent, Sibling 두 종류의 파이프라인 집계 방식 있음
- buckets_path 로 참조할 집계 결과의 경로를 지정

<br>

#### Pipeline Aggregation 유형별 예제

Pipeline Aggregation - Sibling

```
{
 "aggs": {
     "{agg_name1}" : {
       "{agg_type1}": {
           ...
       },
       "aggs": {
         "{agg_name2}" : {
           "{agg_type2}": {
              ...
           }
         }
       },
       "{pipeline_agg_name}": {
         "{pipeline_agg_type}": {   // ex. max_bucket (가장 큰 값을 가진 bucket 출력)
           "buckets_path": "{agg_name1} > {agg_name2}.{metric name(optional)}"
         }
       }
     }
   }
}
```

- pipeline_agg_type
  - min_bucket
  - avg_bucket
  - sum_bucket
  - stats_bucket
  - extend_stats_bucket
  - percentiles_bucket
  - moving_avg_bucket

<br>

Pipeline Aggregation - Parent

```
{
 "aggs": {
     "{agg_name1}" : {
       "{agg_type1}": {
           ...
       },
       "aggs": {
         "{agg_name2}" : {
           "{agg_type2}": {
              ...
           }
         },
         "{pipeline_agg_name}": {
           "{pipeline_agg_type}": {   // ex. derivative (차이/변화량 계산하여 집계 결과에 추가)
             "buckets_path": "{agg_name2}"
           }
         }
       }
     }
   }
}
```

- 집계 결과로 생성된 bucket 을 이용해 계산 후, 기존 집계 결과에 계산 결과를 추가
  - ex. bucket 간의 수치 차이(변화량) 등 계산
- (histogram 등) 집계 결과에 누락되는 구간이 없도록 min_doc_count 값을 0으로 설정
  - gap : 누락되는 구간
  - gap_policy : 누락되는 구간의 처리 방식
    - skip : 누락된 bucket 을 skip 하고 pipeline 집계
    - insert_zeros: 누락된 값을 0으로 대체 후, pipeline 집계
- pipeline_agg_type
  - derivative (변화량)
  - cumulative_sum (누적)
  - bucket_script
  - bucket_selector
  - serial_diff (시계열 차분)

<br>

Pipeline Aggregation - Parent : bucket_script 사용예

```
{
      ...
    "{pipeline_bucket_name}" {
      "bucket_script": {
         "buckets_path": {
           "val1": "{agg_name1}",
           "val2": "{agg_name2}",
         },
         "script": "params.val1 / params.val2"
       }
    }
}
```

<br>

Pipeline Aggregation - Parent : bucket_selector 사용예

```
{
  ...
  "{pipeline_bucket_name}" {
  "bucket_selector": {
     "buckets_path": {
       "val1": "{agg_name1}",
       "val2": "{agg_name2}",
     },
     "script": "params.val1 > params.val2"
   }
  }
}
```

<br>

Pipeline Aggregation - Parent : serial_diff 사용예

```
{
      ...
    "{pipeline_bucket_name}" {
      "serial_diff": {
         "buckets_path": "{agg_name1}"
         "lag": "7"
       }
    }
}
```
