---
title:  "[번역] Introduction to DevSecOps"
search: true
categories: 
  - 번역
tags:
  - DevSecOps
  - Security
  - dzone
classes: wide
---

* 이 포스팅은 Jeff Williams의 [글](https://dzone.com/refcardz/introduction-to-devsecops)을 번역한 것입니다.
* 번역 내용에 대한 조언 및 의견은 작성자에게 큰 도움이 됩니다
* 모든 저작권과 권리는 Jeff Williams에게 있습니다. 

## 1) DevSecOps 란 무엇인가?

DevSecOps는 IT 보안을 DevOps 원칙에 기반하여 접근한 방식이다. 빠른 파이프라인 속도를 유지하며 보안(Sec)을 달성하도록 DevOps의 방법(practices)들을 통해 접근하고 있으며, 현재도 발전 중에 있다.

- **DevSecOps Is Full Stack**: DevSecOps는 IT 스택 전체에 걸쳐 있으며 네트워크, 호스트, 컨테이너, 서버, 클라우드, 모바일 및 응용 프로그램의 보안을 포함한다.
- **DevSecOps Is Full SLC**: DevSecOps는 또한 개발 및 운영을 포함하여 전체 software lifecycle에 걸쳐 있다. 개발 과정에서는 취약점을 식별 및 방지하고, 운영 과정에서는 애플리케이션을 모니터링하고 방어한다.

DevSpsOps를 DevOps 화 되지 않은 프로젝트들에 적용할 수 있을까? 물론 적용할 수 있다. 이 문서의 아이디어는 거의 모든 소프트웨어 프로젝트에 적용할 수 있고, 만약 가능한 한 비용에 효과적으로 고도의 안전한 소프트웨어를 생산하는 것이 목표라면? DevSecOps가 바로 그 답이다.

Gartner는 DevSecOps를 가장 빠르게 성장하는 분야 중 하나라고 명명했으며, 2021 년까지 빠른 개발팀(rapid development teams) 중 80%가 DevSecOps를 활용할 것으로 예측했다. DevSecOps를 시행하는 조직은 인상적인 결과를 보여주고 있는데, 이들은 보안 테스트를 자주 수행하여 애플리케이션을 자주 업데이트하고 취약점을 수정하는 시간을 2 배 단축할 수 있었다고 한다.

성공적인 DevSecOps 도입을 위해서는 여러 유형의 보안 작업들, 그리고 그것들이 조직에 미치는 가치를 이해하는 것이 중요하다. 보안 작업들에 대해 진정으로 이해하기 전에는 DevSecOps의 효과를 전달하기 어렵기 때문이다. 만일 보안 작업들에 대해 깊이 알고 싶으면 The Phoenix Project와 The DevOps Handbook과 같은 책을 읽는 것을 추천한다.

![](https://dzone.com/storage/temp/9612769-picture1.png)



## 2) DevSecOps의 핵심 테마

이 글을 통해 DevSecOps를 적용하는 과정들을 살펴볼 수 있다. 아래의 내용들은 DevSecOps를 도입하는 과정에서 결정을 내리는 데 도움을 주는 지침이 될 것이다.

|                              |                                                                                                                                                                                                                                                                                         |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Empowering Engineering Teams |  개발 조직과 운영 조직은 안전한 응용 프로그램을 프로덕션에 제공할 수 있는 권한과 책임이 있고, 보안 전문가는 도구 제작과 코칭을 통한 지원을 하지만 보안에 대한 주요한 책임은 없다. 따라서 도구들과 프로세스들이 보안 전문가가 아닌 개발자와 운영 조직을 위해 설계되었는지 확인해야 한다. |
| Making Security Visible      | 많은 조직에서 보안 작업은 숨겨져 있고, 추적되지 않기 때문에 보안의 가치는 일반적으로 이해하기 쉽지가 않다. DevSecOps에서는 다른 유형의 작업들처럼 보안 작업들을 추적, 작업 및 측정할 수 있도록 해야 한다.                                                                               |
| Shift Left                   | 보안을 "왼쪽으로" 옮긴다는 의미는 보안 활동을 개발 초기부터,생산까지 모든 단계에서 지속적인 피드백을 통해 수행하는 것을 의미한다. 즉 software lifecycle 전체에 걸쳐 확장된다는 것을 의미하며, 왼쪽으로 이동한다고 해서 개발 중에 보안이 완료되는 것은 아니다.                           |
| Security as Code             | 보안 전문가에 의해 수동으로 수행했던 보안 활동(특히 테스트)들을 test cases, tools 그리고 configurations로 변화 시킨다. 이것들은 보안이 software lifecycle 전역에서 지속적으로 검증될 수 있도록 한다.                                                                                    |
| Continuous Security          | Continuous Security는 CI/CD와 마찬가지로 지속적인 보안 활동을 통해 지속적인 위협에 대응하는 것을 의미하며, 개발 및 운영 프로세스의 일부로 팀 구성원이 이미 사용하고 있는 도구에 통합한다.                                                                                               |
| Prevent and Protect          | 모든 공격자를 탐지하거나 막을 수 없고, 100% 완벽한 코드를 개발할 수는 없을 것이다. 따라서 보안 코딩과 (DevSec)과 런타임 보호 (SecOps)를 조화롭게 적용하는 전략이 필요하다.                                                                                                              |

  
  
## * 번역 예정 *
### 3) The "Three Ways" of Security
### 4) "Hello, World!" – A First Step Towards DevSecOps
### 5) Creating Your DevSecOps Pipeline
### 6) Choosing Security Tools and Technologies

