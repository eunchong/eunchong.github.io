---
title:  "[Elasticsearch] Easier Relevance Tuning in Elasticsearch 7.0"
search: true
categories: 
  - Elasticsearch
tags:
  - 검색엔진
  - Elasticsearch
  - ES
  - rank_feature
  - rank_features
  - relevance
classes: wide
---

* 이 포스팅은 Mayya Sharipova, Adrien Grand의 [Easier Relevance Tuning in Elasticsearch 7.0](https://www.elastic.co/kr/blog/easier-relevance-tuning-elasticsearch-7-0)을 정리한 것입니다.
* 내용에 대한 조언 및 의견은 작성자에게 큰 도움이 됩니다.
* 모든 저작권과 권리는 Elasticsearch, Mayya Sharipova, Adrien Grand에 있습니다.

---

## Elasticsearch 7.0에서 더 쉬워진 Relevance 튜닝

Elasticsearch에서 검색을 튜닝하다 보면, relevance를 향상시키는 것이 정말 어려운 일이라는 것을 알게 된다.
가끔씩 수시로 변경이 필요한 다양한 파라미터와, 순위 요소들이 있다. 예를 들면, 아래와 같은 것들이 잠재적으로 더 좋은 결과를 얻기 위해 조작할 수 있는 변수들이다.

* 글의 제목 필드가 본문보다 얼마나 가중치를 갖는 게 좋을까?
* 만약 카테고리 필드에 가중치를 준다면, 얼마나 주는 게 좋을까?

이상적으로는, relevance 지표를 향상시키기 위해 많은 직원들을 고용하고 최대한 많은 수의 쿼리에 대한 result sets에 태그들을 지정해서 relevance에 영향을 미치는 변수들을 최적화하는 것이 좋지만, relevance을 향상시키기 위해 traning set를 만드는 작업은 많은 수작업을 필요로 하기 때문에 일반적으로 시도하기 어렵다. 때문에, 일반적으로 소규모 조직에서는 도메인에 대한 지식을 바탕으로 점수 공식을 대강 예측하는 경우가 많다. 뭐 어떤 경우든, 검색을 튜닝하다 보면 문장의 relevance를 popularity 또는 authority와 같은 다른 relevance와 결합해야 하는 상황에 놓인다. Elasticsearch 7.0에서는 이 작업을 조금 더 쉽게 해주는 몇 가지 새로운 방법을 제공한다.

BM25(Elasticsearch의 기본 랭킹 방식)를 포함해서 대부분의 문장 랭킹 방식은, 합계를 통해 여러 조건의 점수 기여도를 하나의 점수로 합친다. 하지만, 이런 선형의 조합은 각 점수의 특징들이 비슷한 분포를 따르는 경우에만 잘 동작한다. 이런 이유로, 사용자들은 Elasticsearch에서 정규화된 점수를 얻는 방법을 필요로 하는데, 정규화된 점수는 모든 점수를 맞춰보고 계산하기 전에는 얻을 수 없고, 여러 가지 [단점](https://cwiki.apache.org/confluence/display/LUCENEJAVA/ScoresAsPercentages)을 갖고 있다.

2005년의 한 논문([Relevance Weighting for Query Independent Evidence](http://www.hugo-zaragoza.net/academic/pdf/craswell_sigir05.pdf))의 엔지니어들은 BM25 점수와 비슷하게 보이는 다른 방법을 찾아냈다. 특히 용어 빈도를 BM25로 통합하기 위해 satu(saturation)라고 불리는 제안된 한 function을 사용했다. Elasticsearch 7.0은 이 논문에 영감을 받아서 두 가지 새로운 필드인 rank_feature와 rank_features를 개발했고, 이 두 필드에서 작동하는 rank_feature라는 새로운 쿼리를 개발했다.

## 새로운 New rank_feature와 rank_features 필드

rank_feature 필드는 일반적인 float 필드와 유사하게 동작하지만, 이 필드가 랭킹에 사용될 때, 엘라스틱서치에서 효과적으로 쿼리 할 수 있는 방식으로 데이터를 색인한다. rank_feature는 일반적으로 인기도, 권한, URL 길이 등의 관련성 측정에 사용된다. rank_features는 마찬가지로 유사하지만, 가중치 태그나 카테고리같이 sparse한 feature들에 더 적합하다.

다음의 예제를 살펴보자, 웹 페에지를 인덱싱하는 경우 `score = bm25_score + satu(pagerank)`를 계산하기 위해 우리는 다음과 같이 rank_feature 쿼리를 사용할 수 있다.

```json
PUT test
{
  "mappings": {
    "properties": {
      "pagerank": {
        "type": "rank_feature"
      },
      "content": {
        "type": "text"
      },
      "url": {
        "type": "keyword"
      }
    }
  }
}

PUT test/_doc/1
{
  "url": "http://en.wikipedia.org/wiki/2016_Summer_Olympics",
  "content": "Rio 2016",
  "pagerank": 50
}

PUT test/_doc/2
{
  "url": "http://en.wikipedia.org/wiki/2016_Brazilian_Grand_Prix",
  "content": "Formula One motor race held on 13 November 2016 at the Autódromo José Carlos Pace in São Paulo, Brazil",
  "pagerank": 20
}

PUT test/_doc/3
{ 
  "url": "http://en.wikipedia.org/wiki/Deadpool_(film)",
  "content": "Deadpool is a 2016 American superhero film",
  "pagerank": 35
}

POST test/_refresh

GET test/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "2016"
          }
        }
      ],
      "should": [
        {
          "rank_feature": {
            "field": "pagerank"
          }
        }
      ]
    }
  }
}
```

참고할 부분은, rank_feature 필드는 검색이나 aggregate를 할 수 없다. 만약 검색이나 aggregate를 하고 싶다면, 다중 필드를 사용해서 float와 rank_feature 필드로 매핑해야 한다. 아래의 인덱스 생성 요청은 pagerank 필드를 float로, pagerank.feature를 rank_feature 필드로 매핑한다.

```json
PUT test
{
  "mappings": {
    "properties": {
      "pagerank": {
        "type": "float",
        "fields": {
          "feature": {
            "type": "rank_feature"
          }
        }
      }
    }
  }
}
```

[rank_feature 쿼리](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/query-dsl-rank-feature-query.html)는 score를 계산하는 function을 변경하는 각기 다른 3가지 옵션을 제공하는데, 모두 튜닝할 수 있는 파라미터들이 있다. 만약 충분한 테스트를 할 수 있는 학습 셋을 갖고 있지 않다면, 쿼리 부스트만 사용해서 features의 가중치를 변경하고, 인덱스 통계로부터 자동으로 계산된 기본값인 saturation 함수와 파라미터는 유지하는 것을 추천한다.

이 필드들은 유연하지는 않지만 Elasticsearch 7.0에 새로 적용된 [top-k retrieval optimizations](https://www.elastic.co/blog/faster-retrieval-of-top-hits-in-elasticsearch-with-block-max-wand)와 함께 사용될 수 있는 큰 장점이 있다. 이를 통해 전체 결과에 하나 또는 두 개의 features를 추가하기 위해 필요 이상의 성능을 낭비할 필요가 없다. 만약, 더 많은 유연성이 필요하고 성능을 희생할 수 있다면, 스크립팅을 사용할 수 있다.

## script_score를 사용해서 더 유연하게 점수 조정하기

Function Score 쿼리를 대체하도록 만들어진 Script Score 쿼리를 사용하면, painless 스크립트를 통해 문서의 점수 공식을 정의할 수 있다. Script Score 쿼리는 relevance 점수를 합치는데 사용할 수 있는 함수들을 제공한다. 그 기능들 중 두 가지가 saturation와 sigmoid다. 예를 들어, 이전의 rank_feature 예제는 다음의 script_score 쿼리와 같이 만들 수 있다.

```json
GET /_search
{
    "query" : {
        "script_score" : {
            "query" : {
                "match": { "content": "2016"}
            },
            "script" : {
                "source" : "_score + saturation(doc['pagerank'].value, 50))"
            }
        }
     }
}
```

여기서, 우리는 문서의 pagerank가 포함된 문장 점수의 조합으로 랭킹을 정의한다. pagerank 점수는 saturation 함수를 통해 0~1의 값으로 변환되어 제공된다. script_score 쿼리에서 numeric, geo 또는 date 필드를 위한 decay function은 relevance signal을 변환하는데 사용할 수 있다. 만약 relevance signals가 [dense](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/dense-vector.html) 혹은, [sparse](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/sparse-vector.html) vector 필드인 경우, script_score 쿼리에 vector function을 추가하고 바로 이 필드를 scoring에 사용할 수 있다. [blog about better result sorting](https://www.elastic.co/blog/better-than-average-sort-by-best-rating-with-elasticsearch)에서처럼 스크립트를 사용하면, 도메인에 가장 적합한 점수 공식을 정의할 수도 있다. 이 문서에서는 온라인 리테일에서 유저의 평가 기준으로 제품 순위를 매기는 스크립트를 볼 수 있다.

## 결론

아직 유저가 해야 할 일이 많지만, 우리는 이 새로운 도구를 통해 relevance를 향상시키기가 조금 더 쉬워졌길 바란다. 특히 상위의 결과를 검색할 때, 모든 매칭 결과를 확인하지 않아도 되기 때문에 쿼리의 결과가 훨씬 빨라질 수 있다. 또한 우리는 시일 내에 최신성이나, 가까운 거리같이 바로 검색을 해야 하는 dynamic features를 위한 기능들도 제공하길 희망하고 있다.

## Reference

* <https://en.wikipedia.org/wiki/PageRank>
* <https://www.elastic.co/kr/blog/easier-relevance-tuning-elasticsearch-7-0>
* <https://cwiki.apache.org/confluence/display/LUCENEJAVA/ScoresAsPercentages>
* <https://www.elastic.co/guide/en/elasticsearch/reference/7.0/query-dsl-rank-feature-query.html>
* <https://www.elastic.co/blog/faster-retrieval-of-top-hits-in-elasticsearch-with-block-max-wand>
* <https://www.elastic.co/guide/en/elasticsearch/reference/7.0/dense-vector.html>
* <https://www.elastic.co/guide/en/elasticsearch/reference/7.0/sparse-vector.html>
* <https://www.elastic.co/blog/better-than-average-sort-by-best-rating-with-elasticsearch>
