---
title:  "[디자인패턴] Design Patterns for Microservices"
search: true
categories: 
  - 디자인패턴
tags:
  - dzone
  - 디자인패턴
  - design pattern
  - microservice
classes: wide
---

* 이 포스팅은 Rajesh Bhojwani의 [Design Patterns for Microservices](https://dzone.com/articles/design-patterns-for-microservices)을 정리한 것입니다.
* 내용에 대한 조언 및 의견은 작성자에게 큰 도움이 됩니다
* 모든 저작권과 권리는 Rajesh Bhojwani에 있습니다. 


---

# 마이크로서비스 아키텍처 도입

마이크로서비스 아키텍처(Microservice architecture)는 사실상 현대의 응용 프로그램 개발 기술로 자리 잡고 있다. 이 기술에는 몇 가지 단점과 부딫혀야 할 많은 문제들이 있고, 마법같이 쉬운 솔루션은 아니다. 그래서 이 기술의 도입으로 인해 발생할 수 있는 문제들의 패턴들을 학습하고, 재사용 가능한 솔루션으로 만들 필요성, 즉, 마이크로서비스를 위한 디자인 패턴이 논의될 필요성이 있다. 그리고 이 디자인 패턴들에 대해 고민하기 전에 우리는 다음의 마이크로서비스 아키텍처가 구축된 원칙들에 대한 이해가 필요하다.

  1. **Scalability** (확장성)
  2. **Availability** (가용성)
  3. **Resiliency** (탄력성)
  4. **Independent** (독립성), **autonomous** (자율성)
  5. **Decentralized governance** (분산된 관리)
  6. **Failure isolation** (장애 격리)
  7. **Auto-Provisioning** (자동 프로비저닝)
  8. **Continuous delivery through DevOps** (DevOps를 통한 지속적 배포)

위의 모든 원칙을 적용하다 보면 몇 가지 문제점들을 마주친다. 아래에서 마주칠 문제들과 해결책을 논의해보자.  

## 1. Decomposition Patterns
### a. Decompose by Business Capability (비즈니스 기능 분해)
#### Ploblem

> 마이크로 서비스는 단일 책임 원칙을 적용해서 모든 서비스들을 느슨한 관계로 만든다. 하지만 응용 프로그램을 작은 조각으로 나누는 것은 논리적으로 수행돼야 한다. 어떻게 응용프로그램을 작은 서비스로 분해할 수 있을까?

#### Solution

> 한 전략은 비즈니스 기능으로 분리하는 것이다. 가치 창출을 위해 비즈니스가 수행하는 비즈니스 기능들은 비즈니스의 유형에 따라 다를 수 있다. 예를 들어, 보험회사의 기능에는 일반적으로 영업, 마케팅, 가입, 청구 처리, 결제 등이 포함된다. 기술보단 비즈니스 지향적인 기능들을 제외하고, 기술적인 비즈니스 기능들을 각각의 서비스라고 생각할 수 있다.

### b. Decompose by Subdomain (하위도메인 분해)

#### Ploblem

> 응용 프로그램을 비즈니스 기능을 사용해서 분해하는 것은 좋은 시작이 될 수 있다. 하지만 분해하기 쉽지 않은 여러 서비스에서 공통적으로 사용되는 (소위 "God Classes") 영역이 있다. 예를 들어, 주문 클래스는 주문관리, 주문접수, 주문배달 등에 사용된다. 이것들을 어떻게 분리할 수 있을까?


#### Solution

> "God Classes" 같은 문제에 대해서는 DDD(Domain-Driven Design, 도메인 주도 설계)가 해결방안이 될 수 있다. 이 방법은 하위 도메인과 bounded context 개념을 사용한다. DDD는 전체 도메인 모델을 하위 도메인으로 나눈다. 각 하위 도메인에는 모델이 있으며, 해당 모델의 범위를 bounded context라고 한다. 각 마이크로 서비스는 bounded context 중심으로 개발될 것이다. NOTE: 하위 도메인 식별은 쉬운 작업이 아니다. 이는 비즈니스에 대한 이해를 요구한다. 비즈니스 기능과 마찬가지로 하위 도메인은 비즈니스와 조직 구조를 분석하고 전문 영역을 분석하여 식별될 수 있다.

### c. Strangler Pattern (Strangler 패턴) 

#### Ploblem

> 지금까지 우리는 새로 개발되는 응용프로그램에 대한 분해 디자인 패턴에 대해 이야기했다. 하지만, 우리가 수행하는 작업의 80%는 기존에 개발된 큰 모놀리식 응용프로그램이다. 라이브로 사용되고 있는 서비스를 작은 코드로 분할하는 것은 큰 작업이기 때문에, 위에서 다룬 디자인 패턴들을 적용하는 것은 어려울 것이다. 

#### Solution

> Strangler 패턴이 해결방안이 될 수 있다. Strangler 패턴은 감싸 인 나무를 질식시키는 포도나무에 비유해서 만들어졌다. 이 방법은 웹 응용프로그램에서 잘 적용되는데, 각 URI 호출마다 한 서비스는 여러 도메인으로 분리될 수 있고, 별도의 서비스로 호스팅 될 수 있다. 이 아이디어는 한 번에 하나의 도메인을 수행하는 것이다. 동일한 URI 공간에 나란히 두 개의 서로 다른 응용 프로그램을 만들고, 새롭게 리팩터링 된 응용 프로그램은 최종적으로 모놀리식 응용프로그램을 종료할 수 있을 때까지 점진적으로 대체한다.

## 2. Integration Patterns
### a. API Gateway Pattern
### b. Aggregator Pattern
### c. Client-Side UI Composition Pattern

## 3. Database Patterns
### a. Database per Service
### b. Shared Database per Service
### c. Command Query Responsibility Segregation (CQRS)
### d. Saga Pattern

## 4. Observability Patterns
### a. Log Aggregation
### b. Performance Metrics
### c. Distributed Tracing
### d. Health Check

## 5. Cross-Cutting Concern Patterns
### a. External Configuration
### b. Service Discovery Pattern
### c. Circuit Breaker Pattern
### d. Blue-Green Deployment Pattern


