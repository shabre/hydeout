---
layout: post
title: elastic search 데이터 검색
categories:
    - elastic search
excerpt_separator: "<!--more-->"
---

## 엘라스틱서치 분석기

엘라스틱서치에서는 인덱스에 저장된 문서를 검색할 수 있도록 다양한 기능을 제공한다. 문서는 색인 시 분석기에 의해 분석 과정을 거쳐 토큰으로 분리되어 저장된다. 분석기는 저장 뿐 아니라 문서를 검색할 때에도 분석기를 통해 문장을 분석 후 score에 따라 결과를 찾을 수 있다.

이번 포스트에선 데이터 검색에 사용되는 API, URI 검색, QUERY DSL에 대해서 공부해보겠다.

### URI 검색

URI 검색은 request body 검색에 비해 간결하고 작성하는데 걸리는 시간이 짧은 장점이 있다. 하지만 복잡한 쿼리를 작성 할 경우 URI 검색에는 모든 쿼리를 다 넣기가 힘들어진다. 따라서 URI 검색은 간단한 쿼리를 통해 확인작업을 하는 용도로 사용하는것이 좋다.

movie_search 인덱스에 movieNmEn 값이 Family 인 값을 검색하려면 아래와 같은 쿼리를 작성하면 된다
```
POST movie_search/_search?q=movieNmEn:Family
```

URI 검색에서 자주 사용하는 파라미터는 아래와 같다.
|파라미터|기본값|설명|
|-|-|-|
|q|-|검색을 수행할 쿼리 문자열 조건을 정한다.|
|df|-|쿼리에 검색을 수행할 필드가 지정되지 않았을 경우 기본값으로 검색할 필드를 지정한다.|
|analyzer|검색 대상 필드에 설정된 형태소 분석기|쿼리 문자열을 형태소 분석할 때 사용할 형태소 분석기를 지정한다.|
|analyzer_wildcard|false|접두어/와일드카드 검색 활성화 여부를 지정한다.|
|default_operator|OR|두 개 이상의 검색 조건이 쿼리 문자열에 포함된 경우 검색 조건 연산자를 설정한다.|
|_source|true|검색 결과에 문서 본문 포함 여부를 지정한다.|
|sort|-|검색 결과의 정렬 기준 필드를 지정한다.|
|from|-|검색을 시작할 문서의 위치를 설정한다.|
|size|-|반환할 검색 결과 개수를 설정한다.|


### RequestBody 검색

HTTP 본문에 JSON 형태로 검색 조건을 기록해 검색을 요청한다.
전에 진행했던 URI 검색을 RequestBody 형태로 변경하면 아래와 같다.
```
POST
{
  "query": {
    "query_string": {
      "default_field":"movieNmEn",
      "query": "Family"
    }
  }
}
````

DSL 쿼리 구조는 아래와 같다

```
{
  "size": 리턴받는 결과의 개수. 기본 10
  "from": 몇번째 문서부터 가져올지 결정. 페이징.
  "timeout": 타임아웃 설정. 기본값은 무한대.

  "_source":{필요한 필드만 출력하고 싶을 때 사용}
  "query":{검색 조건문}
  "aggs":{통계 및 집계 데이터 사용 시}
  "sort":{문서 결과 정렬 조건}
}
```

### Query DSL 쿼리와 필터

엘라스틱서치에서는 질의를 작성할 때 실제 분석기에 의해 전문 분석이 필요한 경우와 단순히 참/거짓을 판단할 수 있는 조건 검색으로 구분을 지을 수 있다. 전자는 쿼리 컨텍스트라 하고, 후자를 필터 컨텍스트라고 한다.
||쿼리 컨텍스트|필터 컨텍스트|
|-|-|-|
|용도|전문 검색 시 사용|조건 검색 시 사용|
|특징|분석기에 의해 분석이 수행됨.<br> 연관성 관련 Score 계산.<br>루씬 레벨에서 분석 과정을 거쳐야하므로 상대적으로 느림.|Yes/No 단순 판별.<br> 연관성 관련 계산을 하지않음.<br> 엘라스틱서치 레벨에서 처리가 가능하여 상대적으로 빠름.|
|사용 예|"harry potter"과 같은 문장 분석|"create_year"필드에 2018년인지 여부|

### Query DSL 주요 파라미터

#### Multi index 검색

검색 요청을 여러 인덱스에서 진행할 수 있다.
아래와같이 index를 쉼표로 구분하면 가능하다.
```
POST movie_search,movie_auto/_search
{
  "query":{
    "term":{
      "repGenreNm":"다큐멘터리"
    }
  }
}
```

#### 쿼리 결과 페이징

쿼리 from, size 파라미터 값 조정을 통해 쿼리 결과를 페이징할 수 있다. 다만 from 값이 커지면 그부분만 검색하는것이 아닌 앞의 값들을 모두 검색하여 가져오기 때문에 from값이 커지면 그만큼 검색 비용이 높아진다.
```
POST movie_search,movie_auto/_search
{
  "from":5, #5번째부터
  "size::5, #5개 가져옴
  "query":{
    "term":{
      "repGenreNm":"다큐멘터리"
    }
  }
}
```

#### 쿼리 결과 정렬

sort 파라미터를 이용해 결과를 정렬할 수 있다.
```
POST movie_search,movie_auto/_search
{
  "query":{
    "term":{
      "repGenreNm":"다큐멘터리"
    }
  },
  "sort": {
    "prdtYear": {
      "order":"asc"
    }
  }
}
```

#### _source 필드 필터링
기본적으로는 모든 필드를 검색하나, _source에서 노출될 필드값만을 지정할 수 있다.

#### 범위 검색
숫자나 날짜 데이터를 Range검색을 진행할 수 있다. 범위 연산자는 아래와 같다.

|문법 | 연산자 | 설명 |
| - | -| -|
|lt|<|피연산자보다 작음|
|gt|>|피연산자보다 큼|
|lte|<=|피연산자보다 작거나 같음|
|gte|>=|피연산자보다 크거나 같음|

2016년부터 2017년까지의 데이터를 조회하는 쿼리는 아래와 같다.
```
POST movie_search,movie_auto/_search
{
  "query":{
    "term":{
      "repGenreNm":"다큐멘터리"
    }
  },
  "range": {
    "prdtYear": {
      "gte":2016,
      "lte":2017
    }
  }
}
```

#### operator 설정

엘라스틱서치는 검색 시 문장이 들어올 경우 기본적으로 OR 연산으로 동작한다. 만약 AND 연산으로 해야 할 경우 operator 옵션에 명시를 할 수 있다.

#### mininum_should_match 설정

텀의 갯수가 n개 이상 매칭될 때만 검색 결과로 나오게 할 수 있게하는 옵션이다.

#### fuzziness 설정

유사한 검색 결과를 찾기 위한 설정이다. fuziness 옵션에는 0, 1, 2, AUTO값을 줄 수 있으며 설정한 숫자만큼 오타를 보정하여 검색을 가능하게 해준다.

#### boost 설정
검색을 여러 필드에서 진행할 경우, 특정 필드명에 `^n` 을 덧붙여 가중치를 설정해줄 수 있다.

### Query DSL의 주요 쿼리

#### Match All Query

match_all 파라미터를 사용하는 Match all query는 색인에 모든 문서를 검색하는 쿼리이다. 가장 단순한 쿼리로서 일반적으로 색인에 저장된 문서를 확인할 때 사용한다.
```json
POST movie/_search
{
  "query":{
    "match_all":{}
  }
}
```

#### Match Query

Match Query는 텍스트, 숫자 날짜 등이 포함된 문장을 형태소 분석을 통해 Term으로 분리한 후 이 Term들을 이용해 검색 질의를 수행한다. 그러므로 검색어가 분석되어야 할 경우에 사용해야 한다.

```json
POST movie/_search
{
  "query":{
    "match": {
      "movieNm":"겨울 왕국"
    }
  }
}
```

#### Multi Match Query

multi_match 파라미터를 이용하여 Multi Match Query를 할 수 있다. 단일 필드가 아닌 여러개의 필드를 대상으로 검색할 때 사용한다.
```json
POST movie/_search
{
  "query":{
    "multi_match": {
      "query":"겨울 왕국",
      "fields":["movieNm", "movieNmEn"]
    }
  }
}
```

#### Term Query

Term Query는 별도의 분석 작업을 수행하지 않고 입력된 텍스트가 존재하는 문서를 찾는다. 따라서 keyword 데이터 타입을 사용하는 필드를 검색하려면 Term Query를 사용해야 한다.


```json
POST movie/_search
{
  "query":{
    "term": {
      "genre":"애니메이션"
    }
  }
}
```

#### Bool Query

엘라스틱서치에서는 하나의 쿼리나 여러개의 쿼리를 조합해서 더 높은 스코어를 가진 쿼리 조건으로 검색을 수행할 수 있다.
```json
{
  "query":{
    "bool": {
      "must": [], //반드시 조건에 만족되는 문서만 검색된다.
      "must_not" : [], //조건을 만족하지 않는 문서가 검색된다.
      "should": [], //여러 조건 중 하나 이상을 만족하는 문서가 검색된다.
      "filter": [] //조건을 포함하고 있는 문서를 출력한다. 해당 파라미터를 사용하면 스코어별로 정렬되지는 않는다.
    }
  }
}
```

이 bool query를 이용하여 복잡한 조건의 쿼리를 가능하게 한다.

#### Prefix Query

해당 접두어가 있는 모든 문서를 검색하는데 사용된다. 아래와 같이 prefix로 쿼리를 수행하면 겨울 단어로 시작되는 데이터를 찾아준다.
```json
POST movie/_search
{
  "query":{
    "prefix": {
      "movieNm":"겨울"
    }
  }
}
```

#### Exist Query
필드 값의 존재 여부를 쿼리로 사용할 수 있다. 아래와 같은 쿼리를 사용하면, movieNm 값이 null이 아닌 문서만 가져오게 된다.
```json
POST movie/_search
{
  "query":{
    "exist": {
      "field":"movieNm"
    }
  }
}
```

#### Wildcard query
검색어가 wildcard와 일치하는 구문을 찾는다.
와일드카드 옵션은 아래 두가지가 있다.
- *: 문자의 길이와 상관없이 와일드카드와 일치하는 모든 문서를 찾는다.
- ?: 지정된 위치의 한 글자가 다른 경우의 문서를 찾는다.
```json
POST movie/_search
{
  "query":{
    "wildcard": {
      "typeNm":"장?" //장편, 장르 ...
    }
  }
}
```

#### Nested Query

SQL 에서의 Join과 유사한 기능을 수행한다. 데이터 모델링(1) 마지막에서 설명한 Nested 데이터 타입의 필드를 검색할 때 사용된다.