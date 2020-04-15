---
title:  "[ElasticSearch] Magic WAND: Faster Retrieval of Top Hits in Elasticsearch"
search: true
categories: 
  - ElasticSearch
tags:
  - 검색엔진
  - ElasticSearch
  - ES
  - Magic WAND
  - Retrieval
  - Block-Max WAND
classes: wide
---

* 이 포스팅은 Adrien Grand의 [Magic WAND: Faster Retrieval of Top Hits in Elasticsearch](https://www.elastic.co/blog/faster-retrieval-of-top-hits-in-elasticsearch-with-block-max-wand)을 정리한 것입니다.
* 내용에 대한 조언 및 의견은 작성자에게 큰 도움이 됩니다.
* 모든 저작권과 권리는 Elasticsearch, Adrien Grand에 있습니다.

---

## 옛날이야기 ...

2012년에 베를린 Buzzword에서, Stefan Pohl은 1995년에 H. Turtle and J. Flood이 소개했던 [MAXSCORE](https://dl.acm.org/citation.cfm?id=220057) 알고리즘을 발표했다. 이 알고리즘은 예를 들어, "elasticsearch or kibana"와 같은 요청에서 상위 매칭을 최적화할 때 도움이 된다. 이 아이디어는 매우 간단한데, 만약 우리가 10개의 최상위 결과를 얻고 싶다고 하자. elasticsearch 검색에 대한 최고 점수가 3.0이고, kibana에 대한 최고 점수가 5.0라고 할 때, 문서를 모으는 동안 특정 시점에서 만약 TOP 10번째 문서의 점수가 3.0점 이상이라면, 더 이상 elasticsearch만 포함하는 문서를 검색할 필요가 없게 된다. 결과적으로 이런 경우 최소한의 경쟁력 있는 점수는 3.0 이상이기 때문에, kibana를 포함하는 문서만 찾고, 그 문서들에 elasticsearch가 포함되어 있는지만 확인하고 계산해도 충분하다. 이 아이디어는 후보를 찾기 위한 용어들과, 점수를 계산하는데 사용하는
용어들을 관리함으로써 임의의 수의 용어로 쉽게 일반화할 수 있다. 더 많은 문서를 수집하고, 문서가 상위 10개의 일치 항목 목록에 들어가는 최소 점수가 높아짐에 따라, 최대 점수가 가장 낮은 첫 번째 세트의 용어가 두 번째 세트로 이동하여 쿼리 속도가 향상된다. 이 과정에서 후보를 찾기 위한 쿼리가 더 선택적이게 되므로, 쿼리 실행 속도가 빨라지게 된다.

Stefan은 단순히 설명만 하지 않고, 컨퍼런스 며칠 전에 Lucene에 티켓을 만들고 프로토타입을 공유했다. 이 컨트리뷰션은 흥미 있었지만, 인덱스에 모든 항에 대한 최대 점수를 계산할 방법이 필요했기 때문에, 통합하기에 어려운 부분이 있었다.

특히, score는 문서 빈도(주어진 용어를 포함하는 총 문서 수)와 같은 인덱스 통계에 따라 달라지기 때문에, 인덱스에 새로운 문서들을 추가하게 되면, 기존 세그먼트의 최대 점수도 달라지게 된다.
Stefan의 프로토타입은 이 문제를 인덱스를 rewrite 하도록 했고, 이 최적화는 실제로 정적인 인덱스에서만 적용된다. 이 한계는 많은 노력이 필요했고, 이 문제는 5년간 정지되었다.

## 마침내 ...

5년은 긴 시간이고, 그동안 많은 변화가 생겼다. 특히 한 가지 흥미로운 최적화가 있었다. Lucene이 TF-IDF에서 BM25로 기본 점수 계산 모델을 변경했다. 이 변화는 MAXSCORE에서 매우 중요하다. 왜냐하면 BM25 점수는 자연스럽게 제한되기 때문에, 최대 점수를 기록하지 않고도 이 최적화를 구현할 수 있다. 물론 이 상한은, 모든 문서에서 각 용어의 최대 점수를 계산하는 것 만큼 좋지는 않다. 하지만, 이 최적화 문제를 다시 시도하기에는 충분하다. 그리고 몇 가지 hacks을 통해, 최적화에 사용하기 위한 최소 점수에 대해서도 쿼리 할 수 있었다. 우리는 다이나믹 인덱스 환경에서, 상위 결과를 매칭하는 것을 최적화할 수 있었고, Lucene의 벤치마크에서 약 13배 빨라졌다.

사실, 이 패치는 Stefan의 MAXSCORE 구현이 아니라 Broder와 al이 2003년에 발표한 알고리즘인 WAND이 맞다. WAND는 Weak AND의 줄임말인데, MAXSCORE보다 조금 더 세밀하다. WAND는 쿼리의 항 두 세트를 관리하는 방법 대신에, 하나의 세트를 유지하고, 각 항의 가중치로 최대 점수를 할당한다. 그리고 가중치의 합계가 특정 숫자보다 커야 한다는 사실을 활용해서 모든 용어에 대한 매치 작업을 수행하지 않는다. 이 알고리즘은 "minimum_should_match"이 1보다 큰 boolean query와 같고, 이미 Lucene과 Elasticsearch이 사용 중인 알고리즘이다. 이 알고리즘은 모든 가중치가 1이기 때문에, 조금 더 간단하다.

하지만, 일은 여기서 멈추지 않는다. 이 최적화가 정확한 top hits를 줄 수 있긴 하지만, 이것이 몇 문서를 지나치는 것은 total hit count가 더 이상 계산되지 않는다는 것을 의미하고 이것은 큰 변화가 될 것이다. 또한 이 최적화는 더 많은 항의 매칭은 단순히 점수를 증가시킨다는 추정에 근거한다. 그래서 이것은 몇몇 항이 마이너스 점수를 갖는다면 이 항은 무시된다.

## 추가 개선

그리고 우리는, 이것이 더 개선될 수 있다고 생각했다. 점수의 상한선을 사용하는 것은 모든 문서의 모든 term의 최고 점수를 계산하는 것보다 좋지 않다. 우리가 아직 실제 모든 term에 대한 최고 점수를 계산하고 있진 않지만, 우리는 매우 높은 점수를 갖는 하나의 outlier가 최대 점수를 증가시키고, 최적화 품질을 낮춘다는 사실을 좋아하지 않는다. 더 분석을 해서, 우리는 2011년에 발표한 [S. Ding과 T. Suel](http://engineering.nyu.edu/~suel/papers/bmw.pdf)의 block-max 인덱스와 [block-max WAND](https://issues.apache.org/jira/browse/LUCENE-8135)를 소개한 논문을 찾았다. 이 논문의 기반 아이디어는, 게시물을 고정된 사이즈의 블록으로 분할하고, 각 블록에 대한 분리된 maximum impact score를 기록하는 것이다. 이것은 outlier에 대한 이슈를 완화시키는 데 도움을 준다. 이 impact score는 문서 별 용어 빈도(문서에서 용어가 몇 개나 있는지)와 문서 길이 통계 점수에 기여한다. Block-max WAND는 각 블록이 다른 최대 점수를 갖는다는 이점을 갖는 WAND의 변형이다. Block-max 인덱스는 또한 최고 점수의 합이 경쟁력 있지 않은 블록들을 건너뛰어 term query와 연결의 속도를 향상시키는 데 도움을 준다.

우리는 아주 살짝 수정하는 이 아이디어를 구현했다. impact score를 인덱스에 저장하는 대신에, 우리는 용어 빈도와 문서 길이의 쌍을 기록한다. impact score는 문서마다의 통계들을 통해서 쉽게 유도될 수 있다. 그리고 impact score를 기록하는 것보다, 두 가지 중요한 장점이 있다.

- 이것은 랭킹 함수가 용어별로, 문서별로 분해할 필요 없도록 하고, 이 최적화 작업은 Lucene이 제공하는 TF-IDF, BM25와 같은 모든 점수 계산 함수와 잘 동작한다.
- 유저들이 검색할 때 문서의 길이를 동일한 방식으로 encoding 하는 한, 여전히 다른 similarity로 변경할 수 있다.

인덱스의 impact를 기록하고, block-max WAND를 구현하는 것은 Lucene 벤치마크에서 term query를 약 3배에서 7배 빠르게 만들었다. 그리고 disjunctions은 -8% (약간 늦어짐) ~ 15배 빨라졌다. 이 쿼리들의 일부는 [Lucene's nightly benchmarks](http://people.apache.org/~mikemccand/lucenebench/)에서 추적된다. 예를 들어, [term queries](http://people.apache.org/~mikemccand/lucenebench/Term.html)와 [disjunctions](http://people.apache.org/~mikemccand/lucenebench/OrHighMed.html)에서 CJ로 찾아볼 수 있다. 또한 [phrases](http://people.apache.org/~mikemccand/lucenebench/Phrase.html)나 [prefix queries](http://people.apache.org/~mikemccand/lucenebench/Prefix3.html)같은 쿼리 들도 속도 향상이 있었지만, 이것들은 완전히 다른 메커니즘으로 향상되었다.

## Lucene과 Elasticsearch에서의 실제 의미

Block-max WAND는 Lucene 8.0과 Elasticsearch 7.0에서 통합될 것이다.

이 최적화의 큰 속도 향상을 고려했을때, 우리는 [first-class citizen](https://en.wikipedia.org/wiki/First-class_citizen)에 노출 될 수 있는 두 가지 큰 변화를 만들어 내는것에 대해 결정했다.

- 스코어는 더 이상 마이너스가 아니다.
- total hit count는 더 이상 항상 정확하지 않다.

Lucene은 여전히 정확한 total hit count를 볼 수 있는 옵션을 제공한다. 하지만 이것은 모든 매칭 결과를 수집하는 퍼포먼스 패널티를 발생시킨다.
더 나아가, 더 이상 hit counts가 항상 정확하지 않기 때문에, 응답 형식이 변경되었다. 기존에 숫자로 제공되던 hits.total이 다음과 같은 오브젝트로 제공된다.

```json
{
  "value": 1000,
  "relation": "gte" // can only be "eq" if equal of "gte" if `value` is a lower bound of the hit count
}
```

Elasticsearch에 정확한 total count hits를 요청하기 위해, `track_total_hits` 파라미터를 사용할 수 있다. 만약 실제 개수가 `track_total_hits` 숫자보다 작다면, 응답은 실제 total hit count를 응답할 것이다. 아니라면, hits에 설정한 `track_total_hits` 숫자와 "relation"에 "gte" 값을 응답할 것이고, 이 의미는 실제 total hit count가 응답된 숫자보다 크거나 같음을 의미한다. 그리고 track_total_hits가 낮을수록 쿼리는 빠를 것이다.

만약 우리가 우리의 검색 요청에 aggregations를 포함한다면, 이 최적화는 적용되지 않는다. Elasticsearch는 aggregations를 계산하기 위해 여전히 모든 매치를 검색해야 한다. 이것은 우리에게 이 최적화가 유용하지 않다는 것을 의미하진 않는다. 예를 들어, 만약 우리가 e-commerce 웹사이트를 운영 중인 경우, 상품과 검색 결과에 대한 양상을 노출한다고 할 때, 먼저 상위 결과를 표시하는 요청을 처리하고, 검색 결과에 통계에 대해 따로 요청해서 아직 통계가 수집되지 않은 상황에도 UI에서 상품들을 노출을 시킬 수 있다.

## 결론

몇 배 더 빠른 성능을 만드는 업그레이드는 자주 있는 일은 아니고, 아마 지난 몇 년간 Lucene에 만들었던 가장 흥미로운 변화였다. 그리고 우리는 우리 유저들에게 시일 내에 공개될 일이 매우 기대된다.
7.0 프리뷰 릴리즈를 다운로드하고, 새로운 기능을 활용하여 우리에게 의견을 알려주길 바란다. Elastic Pioneer가 되고, Elastic swag를 얻을 수 있다.

## Reference

* <https://www.elastic.co/blog/faster-retrieval-of-top-hits-in-elasticsearch-with-block-max-wand>
* <http://engineering.nyu.edu/~suel/papers/bmw.pdf>
* <https://issues.apache.org/jira/browse/LUCENE-8135>
* <https://dl.acm.org/citation.cfm?id=220057>
* <http://people.apache.org/~mikemccand/lucenebench>
* <http://people.apache.org/~mikemccand/lucenebench/Term.html>
* <http://people.apache.org/~mikemccand/lucenebench/OrHighMed.html>
* <http://people.apache.org/~mikemccand/lucenebench/Phrase.html>
* <http://people.apache.org/~mikemccand/lucenebench/Prefix3.html>
* <https://en.wikipedia.org/wiki/First-class_citizen>
