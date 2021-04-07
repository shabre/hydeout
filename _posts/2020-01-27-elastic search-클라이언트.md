---
layout: post
title: elastic search 클라이언트
categories: Elastic-search
excerpt_separator: "<!--more-->"
---

# 엘라스틱서치 클라이언트

엘라스틱서치에서는 다양한 언어별 클라이언트 라이브러리를 제공하고 있다.
아래 링크에 접속하면 각 언어별로 제공하는 라이브러리에 대한 정보를 얻을 수 있다.

https://www.elastic.co/guide/en/elasticsearch/client/index.html 


## 자바 클라이언트 모듈

자바에서는 크게 두가지 모듈이 존재한다.

- REST 클라이언트
- Transport 클라이언트

엘라스틱서치 초기에는 Transport 모듈만 제공되었다. Transport 모듈은 직접 소켓 통신을 하여 성능이 REST 모듈보다 좋았으나, 업데이트 할 때 클래스나 메소드가 변경되는 점이 있었고, 현재는 REST 모듈의 성능이 개선되어서 Transport와 크게 차이가 없어졌다. 또한 7버전 이상부터는 Transport가 deprecated 되어서 앞으로는 REST 클라이언트만 사용하면 될 것이다.

## High Level REST 클라이언트

Transport 클라이언트와 REST 클라이언트의 가장 큰 차이점은 HTTP로 요청을 보내느냐, 객체로 요청을 하느냐의 차이이다. 객체로 요청을 하는것이 성능상으로 더 유리할 수 밖에 없지만, 현재는 두 통신 방법에 대한 성능 차이를 무시할 수 있을 정도이고, 무엇보다도 HTTP 통신을 할 경우 HTTPS 환경을 구축하여 사용하는것이 가능하기 때문에 Transport는 더이상 지원을 하지 않게 되었다.

아래와 같이 메이븐으로 모듈을 가져올 수 있다.
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.5.2</version>
</dependency>
```

자바의 경우 가져온 모듈을 아래와같이 클라이언트를 선언하여 사용할 수 있다.
```java
RestHighLevelClient client = new RestHighLevelClient(
        RestClient.builder(
                new HttpHost("localhost", 9200, "http"),
                new HttpHost("localhost", 9201, "http")));
```


## 클라이언트 API 작성 방법

## Index API

대표적으로 Index API 가 자바 REST 클라이언트에서 보내지게 되는 폼은 아래와 같다.

```java
IndexRequest request = new IndexRequest("posts"); //IndexRequest 객체 생성
request.id("1"); //id 설정
String jsonString = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";//body를 jsonString으로
request.source(jsonString, XContentType.JSON);
```

위의 방법은 아래의 3가지의 형태로 표현될 수 있다.

- Map을 이용하여 body 생성
```java
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("user", "kimchy");
jsonMap.put("postDate", new Date());
jsonMap.put("message", "trying out Elasticsearch");
IndexRequest indexRequest = new IndexRequest("posts")
    .id("1").source(jsonMap); 
```

- XContentBuilder를 이용하여 body 생성
```java
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();//첫번째 scope
{ //scope 인식을 쉽게 하기 위해 추가
    builder.field("user", "kimchy");//필드 입력
    builder.timeField("postDate", new Date());
    builder.field("message", "trying out Elasticsearch");
}
builder.endObject();//scope 종료
IndexRequest indexRequest = new IndexRequest("posts")
    .id("1").source(builder); 
```

- source 파라미터에 직접 입력
```java
IndexRequest indexRequest = new IndexRequest("posts")
    .id("1")
    .source("user", "kimchy",
        "postDate", new Date(),
        "message", "trying out Elasticsearch"); 
```

이 외에도 엘라스틱 서치에서 제공하는 JAVA High Level Rest Client API 에 대한 정보는 아래 공식문서에서 확인 가능하다.

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html