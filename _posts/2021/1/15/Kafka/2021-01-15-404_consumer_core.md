---
layout: post
title: "[Kafka] 4장. 카프카 컨슈머 : 중요 개념"
description: " "
date: 2021-01-15
tags: [Kafka]
comments: true
share: true
---

# Kafka 기초 다지기

 **출처 : [카프카 핵심 가이드 (O'Reilly)](https://book.naver.com/bookdb/book_detail.nhn?bid=14093855)**

#### 목차

1. [카프카 훑어보기](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-01_Introduction.md)
2. [범용 메시지 큐와 비교하기](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-02_compare.md)
3. [카프카 프로듀서 : 카프카에 메시지 쓰기](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-03_producer.md)
4. **카프카 컨슈머 : 중요 개념**
   - [**컨슈머와 컨슈머 그룹**](#1-컨슈머와-컨슈머-그룹)
   - [**컨슈머 그룹과 리밸런싱**](#2-컨슈머-그룹과-리밸런싱)
   - [**독자 실행 컨슈머**](#3-독자-실행-컨슈머)
   - [**커밋과 오프셋**](#4-커밋과-오프셋)
5. [카프카 컨슈머 : 카프카에서 데이터 읽기](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-05_consumer_use.md)
6. [스키마 레지스트리](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-06_schema_registry.md)
7. [카프카 내부 메커니즘](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-07_inside.md)
8. [신뢰성 있는 데이터 전달](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-08_reliability.md)
9. [데이터 파이프라인 구축하기](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-09_data_pipeline.md)

- [confluent 예제](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-99_confluent_example)
- [schema registry 예제](https://github.com/colinch4/colinch4.github.io/blob/master/_posts/2021/1/15/Kafka/2021-01-15-99_schema_registry_example)



------

## **카프카 컨슈머 : 중요 개념**

### 1. 컨슈머와 컨슈머 그룹

#### 1) 컨슈머 그룹이란?

- 카프카의 데이터 읽기는 **다른 메시지 시스템의 읽기와 약간 다르다**

- 여러 프로듀서들이 해당 토픽에 메시지를 쓰는 속도가 **컨슈머가 메시지를 처리하는 속도보다 빠르다면**

  - 하나의 컨슈머만으로 처리한다면 **추가되는 메시지 속도를 따라갈 수 없다**
  - 당연히 토픽을 소비하는 **컨슈머의 수를 늘려야 한다**

  > 다수의 프로듀서들이 같은 토픽의 메시지들을 쓸 수 있는 것과 마찬가지로
  >
  > **다수의 컨슈머들이 같은 토픽의 메시지들을 분담해 읽을 수 있어야 한다**

- 카프카 컨슈머들은 **컨슈머 그룹**에 속한다.

  - 다수의 컨슈머가 같은 토픽을 소비하며 같은 컨슈머 그룹에 속하면
  - 각 컨슈머가 해당 토픽의 **서로 다른 파티션을 분담해** 메시지를 읽을 수 있다.



#### 2) 컨슈머 그룹의 컨슈머 수

- **파티션 수 > 컨슈머 수 (1)**

  ![one](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in01.png)

  - C1 컨슈머는 T1 토픽의 **네 개 파티션 모두에 있는 메시지들을 읽을 것**이다.

- **파티션 수 > 컨슈머 수**

  ![two](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in02.png)

  - G1 컨슈머 그룹에 또 다른 C2 컨슈머를 추가한다면, **각 컨슈머는 두 개의 파티션에서만** 읽어들이면 됨!

- **파티션 수 = 컨슈머 수**

  ![equal](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in03.png)

  - 만일 G1 컨슈머 그룹에 네 개의 컨슈머가 있다면, **각 컨슈머는 하나의 파티션에서만** 읽으면 된다!

- **파티션 수 < 컨슈머 수**

  ![over](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in04.png)

  - 만약 컨슈머 그룹에 더 많은 수의 컨슈머를 추가한다면, **일부 컨슈머가 놀게 된다!**

  

#### 3) 용도 및 주의사항

- 이런 방법은 **대기 시간이 긴 작업**을 수행하는 컨슈머에 많이 쓰인다.
  - 하나의 컨슈머로는 데이터가 추가되는 속도를 따라잡을 수 없기 때문!
  - Ex) 토픽의 데이터를 DB에 쓰거나, 시간 소요가 많은 연산을 수행하는 것
- **많은 수의 파티션을 갖도록 토픽을 생성하는 이유**이기도 하다.
- **[주의]** 한 토픽의 파티션 개수보다 더 많은 수의 컨슈머를 추가하는 건 **의미가 없다.**



#### 4) 컨슈머 그룹 추가하기

- 같은 토픽의 데이터를 **다수의 애플리케이션이 읽어야 하는 경우**

  - 각 애플리케이션이 토픽의 일부 메시지가 아닌 **모든 메시지를 읽어야** 한다.
  - 이렇게 하려면 각 애플리케이션이 **자신의 컨슈머 그룹을 가지도록** 해야 한다.

  ![add](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in05.png)

  - 새로운 컨슈머 그룹인 **G2**를 추가하면, **G1 그룹과 무관하게** T1 토픽의 **모든 메시지를 읽는다**
  - **G1**의 경우처럼 각 컨슈머가 파티션들을 **분담**하게 된다.



#### 5) 정리하면

- 각 애플리케이션에서 하나 이상의 토픽에 저장된 **모든 메시지**를 읽어야 할 땐 **애플리케이션마다 컨슈머 그룹 생성**
- 토픽의 메시지 소비를 확장할 때는 **기존 컨슈머 그룹에 새 컨슈머 추가**
- 해당 그룹의 각 컨슈머는 **토픽의 일부 파티션에 있는 메시지만** 읽으면 된다.



___

### 2. 컨슈머 그룹과 리밸런싱

#### 1) 리밸런싱이란

- **파티션 소유권 (ownership)** : 각 컨슈머가 특정 파티션에 대응되는 것
  - 컨슈머 그룹의 컨슈머들은 자신들이 읽는 **토픽 파티션의 소유권을 공유**한다.
  - 이는 **특정 컨슈머가 중단될 때도** 마찬가지다!
    - 그 컨슈머가 읽던 파티션은 **남은 컨슈머 중 하나가 재할당**받아 읽는다.
    - 받아서 읽어야 하는 **토픽들에 변경사항**이 생길 때에도 재할당이 일어날 수 있다.
- **리밸런싱 (rebalancing)** : 한 컨슈머부터 다른 컨슈머로 **파티션 소유권을 이전하는 것**
  - 컨슈머 그룹의 **가용성과 확장성을 높여주므로** 중요하다!
  - **쉽고 안전하게** 컨슈머를 추가하고 삭제할 수 있다.



#### 2) 무조건 좋은가?

- 리밸런싱은 정상적인 처리에서는 **바람직하지 않다**
- 리밸런싱을 하는 동안 **컨슈머들은 메시지를 읽을 수 없어 해당 컨슈머 그룹 전체가 사용 불가 상태**가 된다.
- 한 컨슈머로부터 다른 컨슈머로 파티션이 이전될 때, **해당 컨슈머의 이전 파티션에 대한 정보가 삭제**된다.



#### 3) 안전한 리밸런싱

- **그룹 조정자 (group coordinator)** : `GroupCoordinator` 클래스의 인스턴스로 생성되어,
  - 백그라운드 프로세스로 실행되는 카프카 브로커...

- **그룹 조정자**로 지정된 카프카 브로커에게 컨슈머가 **하트비트 (heartbeat)** 전송
  - 자신이 속한 컨슈머 그룹의 멤버십과 자신에게 할당된 **파티션 소유권 유지** 가능!
  - 컨슈머가 일정 간격으로 하트비트를 보내면 **잘 동작한다는 신호**로 감지!
  - 타임아웃 시간이 경과될 때까지 하트비트 전송을 하지 않으면 **리밸런싱 시작!**



___

### 3. 독자 실행 컨슈머

#### 1) 하나의 컨슈머만 사용하는 이유

- 경우에 따라서는 훨씬 간단하게, **하나의 컨슈머만 필요**할 때도 있다.
- 이 때는 한 토픽의 **모든 파티션**이나 하나의 특정 파티션 데이터를 **항상 하나의 컨슈머**가 읽으면 된다.
  - **컨슈머 그룹**이나 **리밸런싱**이 필요 없게 된다!
- *해당 컨슈머 전용의 토픽과 파티션을 할당*하고, *메시지를 읽고 오프셋을 커밋*하면 된다.
- **할당 (assign)** : 컨슈머가 어떤 파티션을 읽어야 하는지 **정확히 알고 있을 때**는, 토픽을 구독하지 말고 **할당**한다.
  - 컨슈머는 토픽을 구독할 때 다음 중 하나을 택한다.
    1. 컨슈머 그룹의 일원이 된다.
    2. 스스로 파티션을 할당한다.



#### 2) 파티션 할당 예제

```java
List<PartitionInfo> partitionInfos = null;
partitionInfos = consumer.partitionsFor("TOPIC");
	// 특정 토픽의 사용 가능한 파티션을 모두 가져온다.

if (partitionInfos != null) {
    for (PartitionInfo partition : partitionInfos)
    	partitions.add(new TopicPartition(partition.topic(), partition.partition()));
    	
    consumer.assign(partition); // 이 컨슈머에게 해당 파티션들이 할당됨!
    
    while (true) {
        ConsumerRecord<String, String> records = consumer.poll(1000);
        
        for (ConsumerRecords<String, String> record : records) {
            // 할 일 해주기...
        }
        cosumer.commitSync();
    }
}
```



___

### 4. 커밋과 오프셋

- 카프카는 다른 **JMS (Java Message Service) 시스템**과 다른 방법으로 **컨슈머가 읽는 레코드를 추적 관리**한다.
- **커밋 (commit)** : 파티션 내부의 현재 위치는 변경하는 것.



#### 1) 컨슈머는 어떻게 오프셋을 커밋할까?

- 컨슈머가 오프셋을 커밋하면 내부적으로 `__consumer_offsets`라는 이름의 **특별한 토픽에 메시지를 쓴다**

  - 이 토픽은 모든 컨슈머의 오프셋을 가진다.

  - 정상적으로 굴러갈 땐 오프셋을 커밋해도 **아무 일도 일어나진 않는다**

  - **비정상적(?) 일 때**, 오프셋 커밋은 **리밸런싱을 유발**한다.

    1. 기존 컨슈머가 **비정상적으로 종료**되었을 때
    2. **새로운 컨슈머**가 컨슈머 그룹에 추가됐을 때

    

- **두 번 처리되는 메시지**

  ![two](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in06.png)

  - 마지막으로 커밋된 오프셋이 컨슈머가 가장 최근에 읽고 처리한 메시지의 오프셋보다 **작으면**
  - 그 사이의 메시지들이 **두 번 처리된다**

  

- **오프셋 간 누락되는 메시지**

  ![leave](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in07.png)

  - 마지막으로 커밋된 오프셋이 가장 최근에 읽고 처리한 메시지의 오프셋보다 **크다면**
  - 그 사이의 메시지들은 **컨슈머 그룹에서 누락된다**

- **[정리]** *오프셋 관리는 컨슈머 클라이언트 애플리케이션에 큰 영향을 준다*



#### 2) 자동 커밋

- 가장 쉬운 오프셋 커밋 방법... `KafkaConsumer` 객체가 자동으로 오프셋을 커밋해준다.
- 자동 커밋도 **폴링 루프에서 처리**된다.
  - 매번 폴링할 때마다 `KafkaConsumer` 객체가 커밋한 시간이 되었는지 확인!
- **[주의]** *자동 커밋을 가끔 해주면... 레코드가 자주 중복되어 처리될 수 있다*
  - `poll()` 메소드에서 반환된 모든 메시지는 다시 `poll()`을 호출하기 전에 **처리가 끝나도록** 하는 게 중요!
  - 결국... 편하긴 한데 **중복 메시지 방지를 제어**하기에는 충분하지 않다!
  - 대부분 쓸 때는 **직접 오프셋이 커밋되는 시간을 제어**하려고 노력하게 된다.

