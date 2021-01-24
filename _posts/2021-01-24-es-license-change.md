---
title: "[이슈정리] ES의 라이센스 ALV2 -> SSPL 변경(aim at AWS)"
search: true
categories:
  - 이슈정리
tags:
  - 이슈정리
  - Elasticsearch
  - ES
classes: wide
---

## 요약
>
  ES가 라이센스를 기존 ALV2에서 SSPL로 변경한다고 밝혔다.  
  클라우드 업체(AWS)에서 ES를 동의없이 유료 서비스로 직접 제공할 수 없게되었다. **(AWS 싸우자!)**  
  AWS가 자신들이 Elasticsearch를 진정한 오픈소스 ALV2로 지속적으로 운영할 것이라고 밝혔다. **(그래 ES 싸우자!)**  
  앞으로 Elastic의 Elasticsearch와 AWS의 Elasticsearch의 움직임에서 결국 winner는 누가 될까? **(관전포인트!)**  

## ES 라이센스 변경 (ALV2 → SSPL)

- ES가 AWS를 비난하며 라이센스를 ALV2에서 SSPL로 변경했다.
  - ES는 AWS가 상표권을 침해했다고 주장
  - "AWS의 Open Distro for ES가 커뮤니티를 더욱 분열, 혼란시켰다"
  - "AWS가 2019년부터 ES를 fork해서 운영하고 있으며, ES의 독점 코드의 저작권을 침해했다."

## AWS의 대응 (Open Distro ES, ALV2 유지하겠다)

- AWS는 이제 자신들이 진정한(truly) 오픈소스 Elasticsearch(ALV2) 라고 했다.
  - AWS는 오픈소스 개발 관행 [upstream first](https://www.redhat.com/en/blog/what-open-source-upstream)을 지켜왔다 주장
  - "ES 변경 사항들에 대해 Upstream Pull Request를 만들어 왔다."
    - [#42066](https://github.com/elastic/elasticsearch/pull/42066), [#42658](https://github.com/elastic/elasticsearch/pull/42658), [#43284](https://github.com/elastic/elasticsearch/pull/43284), [#43839](https://github.com/elastic/elasticsearch/pull/43839), [#53643](https://github.com/elastic/elasticsearch/pull/53643), [#57271](https://github.com/elastic/elasticsearch/pull/57271), [#59563](https://github.com/elastic/elasticsearch/pull/59563), [#61400](https://github.com/elastic/elasticsearch/pull/61400), [#64513](https://github.com/elastic/elasticsearch/pull/64513)
  - "upstream 개발자, 관리자와 협력해왔고, 포크를 만들지 않았다."
  - "2020년에만 ES, Lucene에 230개 이상의 기여를 했다."
  - "AWS가 이제 Elasticsearch ALV2를 지켜나가겠다."

## 관전 포인트

- Elasticsearch vs Open Distro ES의 미래는?
- 오픈소스(호스팅 기반 비지니스 모델을 갖는 일부) vs 클라우드 업체(AWS + 일부 업체)의 최종 승자는?

## 과거 유사 사례

- **MongoDB (SSPL)**
  - [https://www.mongodb.com/licensing/server-side-public-license/faq](https://www.mongodb.com/licensing/server-side-public-license/faq)
- **Redis (RSAL)**
  - [https://techcrunch.com/2019/02/21/redis-labs-changes-its-open-source-license-again/](https://techcrunch.com/2019/02/21/redis-labs-changes-its-open-source-license-again/)

## 관련 문서

- [https://www.elastic.co/kr/blog/why-license-change-AWS](https://www.elastic.co/kr/blog/why-license-change-AWS)
- [https://www.elastic.co/kr/blog/dear-search-guard-users-including-amazon-elasticsearch-service-open-distro-and-others](https://www.elastic.co/kr/blog/dear-search-guard-users-including-amazon-elasticsearch-service-open-distro-and-others)
- [https://opendistro.github.io/for-elasticsearch/blog/meetings/2021/01/changes/](https://opendistro.github.io/for-elasticsearch/blog/meetings/2021/01/changes/)
- [https://aws.amazon.com/ko/blogs/opensource/stepping-up-for-a-truly-open-source-elasticsearch/](https://aws.amazon.com/ko/blogs/opensource/stepping-up-for-a-truly-open-source-elasticsearch/)
