---
layout: post
title: "[Kafka] 8장. 신뢰성 있는 데이터 전달"
description: " "
date: 2021-01-15
tags: [Kafka]
comments: true
share: true
---


# Kafka 기초 다지기

 **출처 : [카프카 핵심 가이드 (O'Reilly)](https://book.naver.com/bookdb/book_detail.nhn?bid=14093855)**

#### 목차

1. [카프카 훑어보기](https://github.com/3457soso/TIL/blob/master/Kafka/01_Introduction.md)
2. [범용 메시지 큐와 비교하기](https://github.com/3457soso/TIL/blob/master/Kafka/02_compare.md)
3. [카프카 프로듀서 : 카프카에 메시지 쓰기](https://github.com/3457soso/TIL/blob/master/Kafka/03_producer.md)
4. [카프카 컨슈머 : 중요 개념](https://github.com/3457soso/TIL/blob/master/Kafka/04_consumer_core.md)
5. [카프카 컨슈머 : 카프카에서 데이터 읽기](https://github.com/3457soso/TIL/blob/master/Kafka/05_consumer_use.md)
6. [스키마 레지스트리](https://github.com/3457soso/TIL/blob/master/Kafka/06_schema_registry.md)
7. [카프카 내부 메커니즘](https://github.com/3457soso/TIL/blob/master/Kafka/07_inside.md)
8. **신뢰성 있는 데이터 전달**
9. [데이터 파이프라인 구축하기](https://github.com/3457soso/TIL/blob/master/Kafka/09_data_pipeline.md)

- [confluent 예제](https://github.com/3457soso/TIL/blob/master/Kafka/99_confluent_example)
- [schema registry 예제](https://github.com/3457soso/TIL/blob/master/Kafka/99_schema_registry_example)



------

## 신뢰성 있는 데이터 전달

### 1. 

