---
title:  "[Kafka Streams] Streams Concepts"
search: true
categories: 
  - Kafka
tags:
  - Kafka Streams
  - Kafka
  - Stream
classes: wide
---

* confluent의 [Streams Concepts 문서](https://docs.confluent.io/current/streams/concepts.html)를 번역하는 과정입니다.
* 번역 내용에 대한 조언 및 의견은 작성자에게 큰 도움이 됩니다.
* 모든 저작권과 권리는 Confluent, Apache Kafka에 있습니다. 

# Streams Concepts

이 문서는 Kafka Streams의 주요 개념을 요약한다. 더 자세한 내용은, [Streams Architecture](https://docs.confluent.io/current/streams/architecture.html#streams-architecture) 와, [Streams Developer Guide](https://docs.confluent.io/current/streams/developer-guide/index.html#streams-developer-guide) 를 참고할 수 있다.

## Kafka 101

Kafka Streams는 Apache Kafka와 긴밀히 통합(tightly integrated) 되도록 의도적으로 설계되었다. (Kafka Streams의 stateful processing feature, fault tolerance, processing guarantees와 같은 많은 기능들은 Apache Kafka의 storage 및 messaging layer에서 제공하는 기능들을 기반으로 한다.) Kafka Streams을 잘 이해하기 위해서는 [Kafka Document](https://kafka.apache.org/documentation.html)에 있는 [Getting Started](https://kafka.apache.org/documentation.html#gettingStarted), [Design](https://kafka.apache.org/documentation.html#design)와 같은 Kafka의 주요 개념에 익숙해지는 게 중요하다. 특히 생상자, 소비자, 브로커, 데이터, 병렬성 등에 대한 이해가 필요하다.

- 생산자(producer), 소비자(consumer), 브로커(broker)
> 생산자는 kafka 브로커에게 데이터를 전달(publish) 한다. 그리고 소비자는 kafka 브로커로부터 전송된 데이터를 받는다. 생산자와 소비자는 브로커를 통해 완전히 분리되어 있으며, 각각 kafka cluster 외부에서 독립적으로 실행된다.

- 데이터
> 데이터는 토픽(topic)에 저장된다. 토픽은 kafka에서 제공되는 가장 중요한 추상적 개념이고, 생산자가 전달(publish) 하는 데이터의 category나 feed 이름을 의미한다. kafka에 있는 모든 토픽은 하나 또는 그 이상의 파티션(partition)으로 나뉜다. 마찬가지로 kafka streams에서도 프로세싱을 위해 데이터가 파티션으로 나뉜다. 파티션은 데이터를 저장하고, 전송하고, 복제하는데 사용되고, kafka와 kafka Streams 둘 다 모두 파티션을 통해 elasticity, scalability, high performance, fault tolerance 특성을 갖도록 설계되었다.

- 병렬성
> kafka 토픽 파티들은 kafka의 데이터 읽기와 쓰기의 병렬 처리하기 위한 주요 핵심 요소이다. kafka Streams API를 사용하는 응용 프로그램은 이 kafka 토픽, 파티션에 의존하여 병렬 처리를 지원한다.

## Stream

Stream은 kafka Streams에서 제공되는 가장 중요한 추상적 개념이다. kafka에서 topic과 마찬가지로, kafka Streams API는 하나 이상의 stream partition들로 구성된다.

stream partition은 순서가 존재하고, 재생 가능하며, 내결함성(fault tolerance)이 있는 불변(immutable)의 data record의 연속이며, 하나의 data record는 key와 value의 쌍으로 정의된다.

## Stream Processing Application

stream processing application은, kafka stream library를 사용해 만든 프로그램을 말한다. 예를 들어, 우리가 만든 프로그램이 될 수 있을 것이고, 우리의 프로그램은 아마 하나 이상의 [processor topologies](https://docs.confluent.io/current/streams/concepts.html#streams-concepts-processor-topology)를 통한 계산 로직을 정의할 수도 있을 것이다. 우리가 만든 stream processing application은 브로커에서 실행되지 않고, 분리된 JVM instance나 별도의 클러스터에서 동작한다.

application instance은 elasticity, scalability, parallelism, fault tolerance에 도움을 준다.

예를 들어, 우리가 application에서 들어오는 데이터의 처리를 위해 10개의 머신의 파워가 필요한 경우, 우리는 10개의 application instance를 각각 10개의 머신에서 실행하기 위해 선택할 수 있다. 그리고 이 instance들은 실시간 환경에서 새로운 instance 혹은 머신이 추가되거나, 삭제되더라도 자동으로 조정이 가능하다.

## Processor Topology

processor topology(간단히 토폴로지)는 stream processing application에 의한 수행이 필요한 데이터 프로세싱의 계산 로직을 정의한다. 토폴로지는 stream processor들(nodes)의 그래프이고, stream(edge) 들에 의해 연결된다. 개발자는 토폴로지들을 low-level Processor API 또는, Kafka Streams DSL을 통해 정의할 수 있다.

토폴로지에 대한 더 자세한 내용은, [Architecture](https://docs.confluent.io/current/streams/architecture.html#streams-architecture) 문서에서 볼 수 있다.

## Stream Processor

stream processor는 앞의 프로세서 토폴로지 문서의 다이어그램에서 프로세서 토폴로지에 있는 노드라고 생각하면 되는데, 이것은 토폴로지에서 프로세싱 단계를 나타낸다. 예를 들어, 한 stream processor는 데이터를 transform 하는데 사용될 수 있다. 그리고 예를 들면 map, filter, join, aggregation과 같은 일반적인 연산이 Kafka Streams에서 즉시 사용 가능한 stream processor가 된다. 토폴로지에서 stream processor는 upstream processor로부터 하나의 input record를 받는다. 그리고, 그 후에 하나 혹은 그 이상의 output records를 downstream processor로 전달한다.

Kafka Streams는 stream processor를 정의하기 위해 다음의 두 가지 API를 제공한다.
1. declarative 형태의, [functional DSL](https://docs.confluent.io/current/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl)은 대부분의 유저에게 권장되는 API이다. 그리고 특히 초보자에게 추천한다. 왜냐하면 대부분의 데이터 프로세싱 use cases들은 간단한 몇 줄의 DSL 코드로 표현될 수 있기 때문이다. (일반적으로 map, filter 같은 built-in operations를 사용한다.)
2. imperative 형태의, [lower-level Processor API](https://docs.confluent.io/current/streams/developer-guide/processor-api.html#streams-developer-guide-processor-api)는 DSL보다 조금 더 유연함을 제공한다. (하지만 당연히 조금 더 코딩에 대한 노력이 들어간다.) 이를 통해 맞춤 프로세서를 정의하고 연결할 수 있고, [state stores](https://docs.confluent.io/current/streams/architecture.html#streams-architecture-state)에 직접 접근할 수 있다.

## Stateful Stream Processing

몇몇의 stream processing application은 state가 필요하지 않다. 이것들을 stateless라고 표현한다. (이는 한 메세지의 처리가 다른 메세지들의 처리와 독립된 것을 의미한다. 예를 들면, 단순히 한 메세지의 변경이나, 특정 조건에서의 메세지 필터와 같은 연산들이 stateless 한 형태이다.)

하지만, 실 사용에서는 대부분이 application은 state가 필요하다. 이것들을 stateful이라고 표현한다. (제대로 작동하기 위해, 이 state는 내결함적(fault tolerance)으로 관리해야 한다. 예를 들면, 인풋 데이터의 join, aggregate, window를 처리할 때, 우리의 application은 언제든 상태를 유지한다. 그리고 Kafka Streams는, 우리 application에 powerful, elastic, highly scalable, and fault-tolerant 한 상태 저장 프로세싱 능력을 제공한다.)

## Duality of Streams and Tables

실제로 stream 프로세싱의 use cases를 구현할 때, 우리는 일반적으로 stream과 database 둘 다 필요하다. 실제에서 매우 일반적인 한 예를 들면, 고객 트랜잭션으로 들어오는 stream을 database의 최신 고객 정보와 합치는 e-commerce application이 있다. 

그러므로, 어떠한 stream 프로세싱 기술이든 [stream](https://docs.confluent.io/current/streams/concepts.html#streams-concepts-kstream)과 [table](https://docs.confluent.io/current/streams/concepts.html#streams-concepts-ktable)을 위한 first-class 지원을 제공해야 하고, Kafka의 stream API는 stream과 table에 대한 핵심 추상화 기능들을 제공한다. 더 자세한 내용은 조금 뒤 설명하고, 지금 흥미로운 사실은 stream과 table 간에 '[stream-table duality](https://www.confluent.io/blog/introducing-kafka-streams-stream-processing-made-simple/?_ga=2.242863170.1565192945.1585916051-231782923.1581842684&_gac=1.154066250.1585381817.Cj0KCQjw6_vzBRCIARIsAOs54z5Vsb3tCGj1PWwZNDPWDE5dGuPVbiSQEbefQaRTRACusu1G4Z7RNPQaAldcEALw_wcB)'라고 불리는, 매우 긴밀한 관계가 있는 것이다. 그리고 카프카는 이 duality를 다양한 방법으로 활용한다. 예를 들면, 우리의 application을 [탄력 있게](https://docs.confluent.io/current/streams/developer-guide/running-app.html#streams-developer-guide-execution-scaling) 하거나, [내결함성을 가진 stateful 프로세싱](https://docs.confluent.io/current/streams/developer-guide/processor-api.html#streams-developer-guide-state-store-fault-tolerance)을 할 수 있게 하거나, 우리 application의 최신 프로세싱 결과에 대한 [interactive query](https://docs.confluent.io/current/streams/developer-guide/interactive-queries.html#streams-developer-guide-interactive-queries)를 할 수 있게 한다. 그리고, Kafka 내부에서의 사용 이외에도, Kafka Streams API를 통해 우리는 우리가 개발하는 application에서 이 duality를 활용할 수 있다.

우리가 Kafka Streams에서의 [aggregations](https://docs.confluent.io/current/streams/concepts.html#streams-concepts-aggregations)와 같은 개념에 대해 생각하기 전에, 우리는 먼저 tables에 대해 자세히 살펴보고, 앞서 언급한 stream-table duality에 대해 얘기할것이다. 기본적으로, 이 duality는 stream이 table처럼 보여질수 있고, table이 stream처럼 보여질 수 있는 것을 의미한다.

> 우리는 쉽게 설명하기 위해 compound keys나 multisets과 같은 것들에 대해서는 일단은 지나가도록 한다.

아래의 간단한 형태의 table은, 맵 혹은 연관 배열이라고도 불리는 key와 value 쌍의 모음이다.

![streams-table-duality-01](https://docs.confluent.io/current/_images/streams-table-duality-01.jpg){: width="150pt" height="96pt"}

stream-table duality는 stream과 table 간의 가까운 관계를 보여준다.
- Stream as Table - stream은 table의 변경 기록으로 생각될 수 있다. stream의 각각의 데이터 레코드는 table의 상태 변화를 담는다. 따라서 stream은 변경 로그를 처음부터 끝까지 재생시키는 것을 통해 쉽게 table로 재현될 수 있다. 마찬가지로, stream에서 aggregating 데이터 레코드는 table을 만들 수 있다. 예를 들어, 우리는 pageview 이벤트 인풋 stream으로부터의 유저 페이지뷰 총개수를 계산할 수 있다. 그리고 결과는 table key는 유저이고, value는 대응되는 pageview 개수를 갖는 table이 될 수 있다.
- Table as Stream - table은 특정 시점에서 stream의 각 key에 대한 최신의 value의 스냅샷으로 생각될 수 있다. 그리고 table은 각 key-value 항목을 반복하여 쉽게 stream으로 변경될 수 있다.

이것을 예제로 한번 그려보자. 유저 페이지뷰의 총개수를 추적하는 table을 상상해보자. 시간이지나, 새로운 페이지뷰 이벤트가 수행될때마다, table의 상태는 그에 맞춰서 업데이트 된다. 아래에서, table의 시점별 변화된 상태는 변경 stream으로 다시 그려질 수 있다.
![streams-table-duality-02](https://docs.confluent.io/current/_images/streams-table-duality-02.jpg){: width="320pt" height="320pt"}

stream-table duality로 인해, 같은 stream은 원래의 table로 재 생성될 수 있다.
![streams-table-duality-03](https://docs.confluent.io/current/_images/streams-table-duality-03.jpg){: width="460pt" height="399pt"}

예를 들어, change data capture (CDC)를 사용해서 database를 replicate 하는 것과, [fault tolerance](https://docs.confluent.io/current/streams/architecture.html#streams-architecture-fault-tolerance)를 위해 머신 간의 [state stores](https://docs.confluent.io/current/streams/architecture.html#streams-architecture-state)를 replicate 하는 것에는 같은 메커니즘이 사용된다.

stream-table duality는 stream processing applications에서 중요한 개념이고, 실제로 Kafka Streams는 KStream과 KTable 추상화를 통해 이것을 명시적으로 모델링 했다. 더 자세히 다음 섹션에서 설명한다.

## KStream

KStream은 record stream의 추상적 개념이다. 여기서 data records는 table로 생각했을 때 `INSERT`로 해석된다. 예를 들면 신용카드 트랜잭션이나, 페이지 뷰 이벤트 또는 서버 로그 입력들이다. 

아래의 두 data records가 stream으로 보내진다고 생각해보자.

```bash
("alice", 1) --> ("alice", 3)
```

만약 우리의 stream prosesing application이 각 user의 value를 sum 하는 것이라면, `alice`에 대해서는 `4`를 return 하게 될 것이다. 

## KTable

KTable은 changelog stream의 추상적 개념이다. 여기서 각 data record는 table로 생각했을 때 `upsert`를 나타낸다. (만약 키가 아직 존재하지 않는다면, insert로 해석되고, 따라서 upsert로 생각할 수 있다) 또한, `null` values는 해당 key를 제거하는 `DELETE`로 해석 될 수 있다.

마찬가지로, 아래의 두 data records를 stream에 보낸다고 상상해보자.

```bash
("alice", 1) --> ("alice", 3)
```

만약 우리의 stream processing application이 각 user의 value를 sum 하는 것이라면, `alice`에 대해서 `3`을 return 하게 될 것이다. 두 번째 레코드는 이전의 레코드를 update 하기 때문이다.

> Effects of Kafka’s log compaction: 만약 KTable를 Kafka topic에 저장하는 경우, 저장 공간을 절약하기 위해서, kafka의 로그 압축 기능을 사용할 수 있다. 하지만, KStream의 경우 동일한 key에 대한 이전의 값들을 삭제하게 되면, 데이터의 의미를 손상시키기 때문에 로그 압축 기능을 사용하는 것은 올바르지 않다.

우리는 이미 [Duality of Streams and Tables](https://docs.confluent.io/current/streams/concepts.html#streams-concepts-duality)에서 changelog stream 예제를 보았다. 그리고 다른 예제로, relational database에서 각 행의 insert, update, delete에 대한 CDC(change data capture) records가 있다.

KTable은 또한 keys를 통해 data records의 현재 value를 찾는 기능을 제공한다. 이 table-lookup 기능은 [Interactive Queries](https://docs.confluent.io/current/streams/concepts.html#streams-concepts-interactive-queries) 뿐만 아니라, [join operations](https://docs.confluent.io/current/streams/concepts.html#streams-concepts-joins)를 통해서도 사용될 수 있다.

## GlobalKTable
- TODO

## Time
- TODO

## Aggregations
- TODO

## Joins
- TODO

## Windowing
- TODO

## Interactive Queries
- TODO

## Processing Guarantees
- TODO

## Out-of-Order Handling
- TODO
