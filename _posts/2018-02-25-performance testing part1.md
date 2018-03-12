---
layout: post
title: "[part 1 ch1,2] Introduction to Performance Testing"
categories:
    - Performance testing
excerpt_separator: "<!--more-->"
---

## Chapter 1 - Fundamentals of Web Application Performance Testing

### 성능 테스트란?
성능 테스트(performance testing)는 일반적으로 시스템의 병목 현상을 파악하고 향후 테스트를위한 기준을 수립하고 성능 튜닝 노력을 지원하며 성과 목표 및 요구 사항을 준수하는지 확인하고 기타 성과 관련 데이터를 수집하여 이해 관계자가 정보에 근거한 의사 결정을 내릴 수 있도록 돕기 위해 수행된다. 성능 테스트는 총 7개의 단계(Activity)로 구성된다.
- Activity 1. 테스트 환경을 확인한다
- Activity 2. 성능 수용 기준을 확인한다.
- Activity 3. 테스트를 계획하고 디자인 한다.
- Activity 4. 테스트 환경을 구성한다.
- Activity 5. 테스트 디자인을 구축한다.
- Activity 6. 테스트를 실행한다.
- Activity 7. 테스트 결과를 분석하고, 보고하고, 테스트를 재진행한다.

### 그렇다면 성능 테스트를 하는 이유는 무엇일까?
성능 테스트를 진행하는데엔 여러가지 이유가 존재한다. 성능은 비용, 기회비용, 기업의 평판을 결정지을 수 있는 중요한 사항이다. 성능테스트는 크게 4가지로 분류할 수 있다.
- 출시 여부 준비 평가
- 인프라 적합성 평가
- 개발된 소프트웨어의 적절성 평가
- 개선된 성능의 효율성 평가

### 성공적인 테스팅을 위한 요소들에는 무엇이 있을까요?
1. 프로젝트 문서(Project context): 프로젝트 문서는 테스팅을 수행하는데 있어서 테스트의 방향성과 이해를 돕는 중요한 장치이다. 프로젝트 문서 없이는 팀원간의 갈등도 생길 수 있다.
2. 성능 테스트와 tuning process 간의 협업: 튜닝은 성능 테스터와 직접적인 연관은 없어 보일수도 있다. 하지만 성능 테스트 결과는 튜닝의 방향을 제시하기 때문에 성능 테스트와 튜닝은 긴밀한 협업이 필요하다.
3. 기준선(Baseline): 기준선은 시스템, 프로그램의 성능 향상을 위한 후속 변경의 효과를 평가할 목적으로, 테스트 결과값을 평가하는 기준이다. 기준선을 제시하기 위해선 프로그램의 동작과정을 완벽히 이해하야 한다. 또한 프로그램의 진화에 따른 새로운 기준선을 정의해야 한다. 기준선은 재사용이 가능해야 하지만 지나치게 일반화되어서는 안된다.
4. 벤치마킹(Benchmarking): 벤치마킹은 시스템 성능을 내부적으로 작성한 기준 또는 다른 조직에서 승인한 업계 표준과 비교하는 프로세스이다. 벤치마킹을 통해 투명한 성능 비교결과를 경쟁사와 대조할 수 있다. 신뢰할 수 있는 결과를 보장하기 위해 엄격한 표준을 사용하게 된다.

### 테스팅 종류
- Performance testing: 시스템 또는 응용 프로그램의 속도, 확장성 또는 안정성 특성을 결정하거나 유효성을 검사한다. 성능은 프로젝트 또는 제품의 성능 목표를 충족시키는 응답 시간, 처리량 및 자원 활용 수준과 관련된다.
- Load testing: 이 테스트는 작업 중에 발생하는 작업로드나 로드 량에 관련된 성능을 결정, 검증하는 테스팅을 말한다.
- Stress testing: 작업중에 예상되는 조건을 초과하는 경우 시스템, 응용 프로그램의 성능, 유효성을을 검증하는 테스트이다.

## Chapter 2 - Types of Performance Testing

성능 테스팅에는 chapter 1에서 정리한 내용과 같이 크게 performance testing, Load testing, Stress testing 에 Capacity testing 을 추가해 크게 4가지로 구성이 된다. 이번 chapter 에서는 성능 테스팅에 대해 좀더 자세히 정리해 보겠다.

### Performance test
- - -
Performance test 의 목적은  속도, 확장성, 안정성을 검증하기 위함이다. 시스템의 사용자가 응용 프로그램의 성능에 만족하는지 판단하는데 중점을 두고, 기대와 결과의 불일치를 확인하여 튜닝, 용량 계획 및 최적화 작업을 지원한다.

Performance test 는 전체적인 성능을 검증하는데 사용된다. 따라서 각각의 세부 기능별 검증은 하지 못할 수 있다. 또한 철저히 테스팅 모델을 디자인하거나 검증하지 않을경우 매우 적은 부분에서만 성능적 특징을 보일 수 있다. 그리고 소프트웨어적 테스트는 하드웨어에서 수행되지 않는 한 실행에 대한 불확실성이 있다.

### Load test
- - -
Load test의 목적은 정상 및 최대의 로딩 환경에서 프로그램의 정상 동작 여부를 확인하는 것이다. Load test는 응용 프로그램이 원하는 성능 목표를 충족시킬 수 있는지 확인하기 위해 수행되는데, 성능 목표는 service level agreement(SLA)에서 지정되는 경우가 많다. Load test를 사용하면 응답 시간, 처리속도 및 자원활용 수준을 측정가능하고, 최대로드 조건에서의 측정 또한 가능하다.

Load test는 다음과 같은곳에 활용할 수 있다.
- 예상되는 생산로드를 지원하는 데 필요한 처리량을 결정
- 로드밸런서의 적합성 평가
- 동시성 문제를 감지
- 로드 중 기능 오류 감지
- 확장성, 용량계획을 위한 데이터 수집
- 최대 리소스 사용률을 알아보기 위한 테스트

Load test에 대해 알아둬야할 점은 속도 측정을 위한 테스트가 아니란 점이다. 또한 결과는 다른 load test와 비교할 때만 사용해야 한다.

### Stress test
- - -
Stress test는 응용 프로그램이 최대로드 조건을 초과하여 실행될 때 응용프로그램의 동작 상태를 확인하는 테스트이다.
이 테스트의 목적은 높은 부하 조건에서만 나타나는 응용 프로그램의 버그를 밝히는 것이다. 버그의 종류는 동기화 문제, 공유자원 경쟁, 메모리 릭 같은 상황이 있다. stress test를 통해 응용 프로그램의 약점을 파악하고 검증할 수 있다.

Stress test는 다음과 같은 곳에 활용할 수 있다.
- 시스템에 과부하로 인해 데이터가 손상 될 수 있는지를 판별한다.
- 오류를 유발하기 전에 응용 프로그램이 얼마나 많이 부하를 초과하는지 확인한다.
- 임박한 장애를 경고하기 위해 응용 프로그램 모니터링 트리거를 설정할 수 있다.
- 스트레스가 많은 상황에서 보안 취약점이 열리지 않도록 한다.
- 공통 하드웨어 또는 지원 응용 프로그램 오류의 부작용을 판별한다
- 어떤 종류의 실패가 가장 중요한 계획인지 결정하는 데 도움이 된다.

Stress test는 비현실적인 디자인이기 때문에 어느정도의 현실적인 적용이 가능한지에 대한 가치에 대한 의문점이 존재한다. 또한 만약 실제 서비스와 분리를 하지 않을 시 프로그램과 네트워크에 심각한 문제를 발생시킬 수 있다.

### Capacity test
- - -
Capacity test는 용량 테스트가 아니라, 주어진 시스템이 얼마나 많은 사용자나 트랜잭션을 지원하는지에 대한 성능을 테스트하는 것이다. 용량 계획과 함께 수행되며 사용자, 데이터 증가를 기반으로 미래 성장을 계획하는데 사용된다. Capacity test를 통해 현재 상황에서 scale up을 해야하는지, scale out을 해야하는지 파악하는데 도움이 될 수 있다.

Capacity test는 다음과 같은 곳에 활용할 수 있다.
- 비즈니스 요구 사항을 충족하기 위해 workload 처리하는 방법에 대한 정보를 제공한다.
- 용량 계획자가 모델 예측을 검증하거나 향상시키는 데 사용할 수있는 실제 데이터를 제공한다.
- 용량 계획 모델 예측을 비교하기위한 다양한 테스트를 수행 할 수 있다.
- 용량 계획을 돕기 위해 기존 시스템의 현재 사용량 및 용량을 결정한다.

Capacity test는 작성하기가 복잡하고, 모델의 모든 측면이 가치가 가장 높은 시점에 테스트를 통해 검증될 수 있는 것이란 보장이 없다.


큰 4개의 Test 외에도 여러가지 테스트 기법이 존재합니다.
- Component test: 응용 프로그램의 아키텍쳐 구성 요소를 대상으로 하는 성능 테스트이다. Component에는 서버, 데이터베이스, 네트워크, 방화벽 등이 있다.
- Investigation: 성능을 향상시킬 수 있는 데이터 수집 기반을 목적으로 한 활동이다.
- Smoke test: 정상 부하 상태에서 프로그램이 작동을 수행할 수 있는지 확인하기 위한 초기 테스트 과정이다.
- Unit test: 코드 모듈 단위별로 테스트를 진행하는 테스트입니다. 모듈에는 함수, 객체, 메소드, 클래스 등이 포함된다.
- Validation test: 테스트 중인 제품의 성능을 추정된 기대치와 비교하는 테스트이다.