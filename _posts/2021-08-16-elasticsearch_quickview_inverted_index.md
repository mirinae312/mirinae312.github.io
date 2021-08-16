---
layout: post
title:  "ElasticSearch 살펴보기 - 역색인 (Inverted Index)"
date:   2021-08-16 11:00:00 +0900
description: "ElasticSearch 살펴보기 - 역색인 (Inverted Index)"
categories: develop
---

### ElasticSearch 살펴보기 - 역색인 (Inverted Index)

#### 역색인이 되는 순서

HTML 문서가 역색인 되는 Flow

![inverted_index_flow]({{"/img/elasticsearch_quickview/inverted_index_flow.png"| relative_url}})

<br>

Character Filter
- 분석전 전처리 과정
- html_strip 등 처리

Tokenizer Filter
- 형태소 분석
- 토큰 분리

Token Filter
- 토큰 후처리
- 불필요한 단어 제거, 소문자 변환등 등 처리


<br>

#### Index 내 역색인된 데이터

![inverted_index_data]({{"/img/elasticsearch_quickview/inverted_index_data.png"| relative_url}})
