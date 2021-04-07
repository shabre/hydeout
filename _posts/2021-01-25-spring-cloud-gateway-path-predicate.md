---
layout: post
title: spring cloud gateway path predicate 작성 시 주의사항
categories:
    - Development
excerpt_separator: "<!--more-->"
---

## 결론부터

현재 저희 MSA 프로젝트에서 스프링 클라우드 게이트웨이의 Path predicate를 통해 라우팅을 하고 있는데요, 보통 1depths 이후에 `'**'` 와일드카드를 많이 사용중에 있습니다. 이걸 사용할때 주의해야 할 점 이 있는데요,

우선 결론부터 말씀드리면 Path route 규칙을 설정할 때 `'**'` 와일드 카드는 Path 의 **마지막 패턴 부분**에서만 사용하셔야 합니다.

- 올바른 예: /shop/payments/**
- 틀린 예: /shop/payments/**/callback

path 중간에 패턴을 적용을 시키려면 한 세그먼트용의 와일드카드 `'*'` 를 여러번 사용하시는것을 추천드립니다. (ex> /shop/\*/\*/callback)

```
Note: In contrast to AntPathMatcher, ** is supported only at the end of a pattern. For example /pages/{**} is valid but /pages/{**}/details is not. The same applies also to the capturing variant {*spring}. The aim is to eliminate ambiguity when comparing patterns for specificity.
```
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html


## 라우팅이 제대로 안된 케이스

결제수단을 가상계좌로 하여 PG 주문 요청을 한 케이스에 사용자가 가상 계좌로 입금을 하게 되면 PG사에서 저희 NCP 로 입금이 완료되었다는 Callback을 날려주는 과정이 있습니다.

이때 callback을 저희 주문에서는 shop 모듈로 받고 있었는데요, server to server 통신이여서 기존 shop 에서 필요한 토큰 검증 로직을 탈 필요가 없습니다.

따라서 spring cloud config에 ShopClientId 를 검증하는 필터를 포함하지 않는 라우팅 규칙을 별도로 정의해서 가지고 있었습니다.
```yml
id: order-shop-redirect
uri: lb://order-shop
predicates:
 - Host=url.test.com
 - Path=/fruit/**/callback
filters:
 - RewritePath=/(?<segment>.*),     /shop/$\{segment}

id: order-shop
uri: lb://order-shop
predicates:
  - Host=url.test.com
  - Path=/payments/**
filters:
  - name: ShopClientId
  - RewritePath=/(?<segment>.*),     /shop/$\{segment}
```

`'**'` 와일드카드는 여러 뎁스의 path를 포함합니다. 따라서 아래 두 콜백 url 모두 `order-shop-redirect`로 라우팅 되는것이 이상하지 않습니다.
- https://url.test.com/payments/abc/callback
- https://url.test.com/payments/abc/123/callback

첫번째 url은 의도대로 order-shop-redirect로 잘 라우팅 되었습니다. 하지만 두번째 url은 order-shop 으로 라우팅 되어 callback 응답을 제대로 못받는 현상이 발생했습니다.

## 원인 분석

아무래도 path 패턴 매칭의 우선순위 문제이지 않나 싶은데  spring cloud gateway 문서에서는 이 이슈 관련해서 자세한 설명이 없었습니다.

그래서 큰마음을 먹고 소스를 열어봅니다.

org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory 클래스를 열어보면 아래와 같은 소스가 있습니다.

![image.png]({{ site.baseurl }}/images/PathRoutePredicateFactory.png)

PathPattern 이라는 클래스를 이용해서 매칭을 하는것 같네요! PathPattern 클래스를 한번 볼까 하고 열어봤지만 현기증이 날것 같아서 잠시 닫고 docs를 열어봤습니다.

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html

못보고 놓칠 뻔 했는데 아래에 요런 글이 있습니다. 모호성을 해소하기 위해 `'**'` 와일드카드는 마지막 패턴에서만 사용하라~ 라고 되어있네요.
![image.png]({{ site.baseurl }}/images/PathDocs.png)


## 결과

path predicates 에 `/payments/*/*/callback` 를 추가하였더니 이제 라우팅이 잘 됩니다.

요번 일을 통해서 세가지 정도를 알게 되었습니다.

- 스프링 클라우드 게이트웨이 path 라우팅에선 PathPattern 클래스를 사용한다.(이건 mvc의 path 매칭에서도 사용된다고 합니다)
- 와일드카드와 같은 모호성을 불러일으킬 수 있는건 정의하는 곳 마다 조금씩 다를 수 있기 때문에 한번 더 확인해보자
- **문서의 Note 부분을 유심히 보자**