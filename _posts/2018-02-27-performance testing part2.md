---
layout: post
title: "[part 2] Exemplar Performance
Testing Approaches"
categories:
    - Performance testing
excerpt_separator: "<!--more-->"
---

## Chapter 4 Web Application Performance Testing Core Activities

이번 챕터의 목적은 시스템, 어플리케이션에서 공통적으로 사용되는 7개의 코어 테스팅 액티비티에 대해서 알아볼 것이다. 어떠한 프로그램에서든지 필요한 공통적인 코어 테스트이기 때문에 그 테스트는 곧 대부분 또는 모든 프로그램에 적용될 것이다.

7개의 코어 액티비티는 다음과 같다.
- Activity 1. Identify the Test Environment.
- Activity 2. Identify Performance Acceptance Criteria.
- Activity 3. Plan and Design Tests.
- Activity 4. Configure the Test Environment.
- Activity 5. Implement the Test Design.
- Activity 6. Execute the Test.
- Activity 7. Analyze Results, Report, and Retest.

Activity 1,2 는 처음 프로젝트를 시작하는 단계에서 중요하게 고려된다. 프로젝트에서 어떠한 사항을 기준으로 세워야하는지 모르기 때문에 project context를 이해하고 반복적으로 activity를 수행하게 된다. 그렇게 project context에 익숙하게 되면 프로젝트의 테스트를 계획하게 된다. 이것이 Activity 3,4 이다. Test가 어느정도 완성이 되면 Activity 5~7 이 반복적으로 수행되게 된다.

아래애서 액티비티별로 하는 테스팅을 자세하게 알아보겠다.

### Activity1. Identify the test environment

성능 테스트 환경을 결정하는것은 중요한 사안이다. 실제 환경과 동일한 환경을 구축하는것이 가장 좋은 방법이겠지만, 여러가지 리소스 모니터링 프로그램과 같은 보조 프로그램이 들어가다보면 실제 환경과는 다르게 테스트 환경이 만들어질 가능성이 높다. 따라서 테스트 환경과 실제 환경에 어떠한 차이점이 존재하는지 아는것은 매우 중요하다.

환경을 구성하는 요소들은 다음과 같은 요인들이 있다.
- 하드웨어
- 네트워크
- Tools
- 소프트웨어
- 외부 요인

### Activity 2. Identify Performance Acceptance Criteria

어떤 성능까지 합격 기준으로 할 것인지에 대한 기준은 개발 초기에 정하는것이 좋다. 테스트를 수행할때 위의 기준에 따라 테스트 결과가 달라지기도 하기 때문이고, 우수한 성능으로 갈 가능성이 높아지기 때문이다.
성능 기준을 정하는 사항들은 다음과 같은 것들이 있을 수 있다.
- 비즈니스 요구 사항
- 사용자 기대치
- 규정 준수 기준 및 업계 표준
- 서비스 수준 계약 (SLA)
- 자원 활용 목표
- 다양하고 현실적인 워크로드 모델

### Activity 3. Plan and Design Tests

위의 2개의 액티비티를 수행하고 나면 여기서부터 본격적으로 테스트 환경을 구축하기 시작한다. 성능 테스트 계획 및 설계에는 주요 사용 시나리오 식별, 사용자 간의 적절한 가변성 결정, 테스트 데이터 식별 및 생성, 수집 할 메트릭 지정이 포함된다.
테스트를 디자인 하기위해선 실제 환경과 비슷한 시뮬레이션을 만들어야 한다. 이 시뮬레이션은 테스트의 지표가 될 수 있기 때문에 중요하며, 명시적으로 결정하는것이 중요하다. 시뮬레이션에 가능한 시나리오는 여러가지가 있다. 대표적으로 5가지를 꼽자면 다음과 같다.
- 계약상 의무가 있는 시나리오
- 성능 테스트 목표 및 목적에 의해 묵시적으로 또는 요구되는 시나리오
- 가장 일반적인 시나리오
- 업무상 중요한 시나리오
- 성능 집약적 시나리오

### Activity 4. Configure the Test Environment

테스트 수행 전에 테스트 환경을 준비하게 되면 테스트 기간동안 환경 구축에 시간을 허비하지 않을 것이고, 이것은 곧 수행할 수 있는 테스트가 많아지는데 기여할 것이다. 따라서 테스트 환경을 미리 준비하는것은 서비스 품질 향상에서도 중요하다고 할 수 있다. 또한 테스트 환경을 구축했다고 끝이 아니다. 변화하는 테스트 상황에 맞추어 지속적으로 환경을 수정해야 할 것이다. 따라서 어느정도 재구성, 추가, 업데이트 할 계획을 미리 세워놓는것도 중요하다.

### Activity 5. Implement the Test Design

### Activity 6. Activity 6. Execute the Test

### Activity 7. Activity 7. Analyze Results, Report, and Retest