---
layout: post
title: elastic search 시작
categories: Elastic-search
excerpt_separator: "<!--more-->"
---
## elastic search 란?

elastic search 는 Elasticsearch B.V. 에서 개발중인 오픈소스 프로젝트이다. elastic `search` 라는 이름에서 볼 수 있듯이 검색을 도와주는 검색 엔진이다. 대용량 데이터의 검색, 집계를 빠르고 정확하게 할 수 있는것이 elasticsearch가 자랑하는 장점이다. 일반적인 데이터 뿐만 아니라 위치, 날짜와 같은 특별한 데이터 기준으로도 검색이 가능하다.

엘라스틱서치의 특징은
- 엘라스틱서치는 실시간, 분산형, 분석 엔진이다.
- 오픈 소스이며, 자바로 개발되었다.
- 테이블과 스키마 대신에 문서 구조로 된 데이터를 사용한다.

## 엘라스틱 서치 구성요소

엘라스틱 서치는 인덱스, 타입, 문서, 필드 구조로 구성된다. RDBS 에서의 스키마, 테이블 구조와는 다른 문서 방식의 구조로 형성되어 있다.
엘라스틱 서치와 RDBMS의 구조를 비교 하면 아래의 테이블과 비슷하게 생각할 수 있다.

|엘라스틱서치|RDBMS|
|-|-|
|인덱스(index)|데이터베이스|
|샤드(shard)|파티션|
|~~타입(type)~~|테이블|
|문서(document)|행|
|필드(field)|열|
|매핑(mapping)|스키마|
|Query DSL | SQL|

### 인덱스(index)
인덱스는 데이터 저장 공간이다. 인덱스는 하나의 타입만 가지며, 한개의 노드에 여러개의 인덱스가 생성될 수 있다. 검색 시 인덱스 이름으로 문서 데이터를 검색하며, 여러개의 인덱스를 동시에 검색하는것도 가능하다.
인덱스를 여러개의 노드에 분산 저장시켜 분산환경으로 구성을 할 수도 있다.(샤드)
인덱스의 이름은 모두 소문자여야 하며 추가, 수정, 삭제, 검색은 RESTful API로 수행 가능하다.
만약 인덱스가 없는 상태에서 데이터가 추가된다면 인덱스가 자동 생성된다.

### 샤드(shard)
인덱스를 여러개의 노드에 분산 저장시켜 저장할 때, 이 분산 저장된 부분을 샤드라 한다. 데이터의 양이 많아 한개의 물리 저장공간에서 분리시켜야 할 때나, 검색 성능 향상을 시킬 때 샤드를 적절히 사용하면 된다.

### 타입(type)
타입은 인덱스의 논리적 구조를 의미한다. 6.0 버전까지만 인덱스 하나에 여러개의 타입을 두는것이 가능했다. 하지만 6.1버전부터 1인덱스에 1타입만 설정이 가능했고, 7.0 부터는 타입 개념이 제거되었다.

### 문서(document)
데이터가 저장되는 최소 단위이다. RDBMS 테이블의 행과 비슷하다.

### 필드(field)
문서를 구성하기 위한 속성으로 RDBMS 테이블의 열 속성과 비슷하다.
하나의 필드는 다수의 데이터 타입을 가질 수 있다.

### 매핑(mapping)
매핑은 문서와 데이터를 가지고 있는 필드가 어떻게 저장되고 색인될지에 대한 프로세스다. 매핑 정보에 여러가지 데이터 타입 지정이 가능하지만, 필드명 자체는 중복해서 사용할 수 없다.


## 노드의 종류
### 마스터(master) 노드
- 클러스터 관리.
- 노드 추가, 제거와 같은 전반적인 클러스터 관리 담당.
- 여러 개의 마스터 노드를 설정하면 하나만 마스터 노드로 작동.
- 네트워크가 빠른 노드로 구성하는것이 권장됨.

### 데이터(data) 노드
- 데이터(문서)가 저장되는 노드.
- 검색과 통계 같은 작업이 진행됨.
- 마스터와는 별개로 구성하는것이 권장됨.

### 인제스트(injest) 노드
- 데이터 전처리를 위한 노드.
- 인덱스 생성 전 문서의 형식을 다양하게 변경할 수 있음.

### 코디네이팅(coordinating) 노드
- 사용자의 요청만 받아서 리바운드식으로 처리.

설정에 따라 노드는 한 가지 유형으로 동작할 수도 있고 여러개의 유형을 겸해서 동작될 수 있음.

## elasticsearch & kibana 설치

kibana는 엘라스틱 서치를 시각화 시켜주고, 개발을 용이하게 하므로 엘라스틱서치와 같이 설치를 진행한다.

엘라스틱서치와 키바나는 모두 엘라스틱서치 공식 홈페이지에서 다운로드 할 수 있다.
각 파일들을 다운로드 후 압축 해제만 하면 /bin 폴더 안에있는 elasticsearch, kibana 로 실행이 가능하다. 환경 변수로 설정을 해 놓으면 더욱 빠른 실행이 가능하다.

키바나에는 여러 기능이 존재하지만, 연습을 편하게 해줄 RESTFul API 호출 콘솔을 사용 가능하다. 왼쪽 탭의 아래서 세번째 스패너 모양의 아이콘을 클릭하면 사용 가능하다.

![kibana]({{ site.baseurl }}/images/kibana_install.PNG)

## 주요 API

엘라스틱서치에서는 아래와 같은 주요 API를 제공한다.
- 인덱스(Indices) 관리 API
- 문서(document) 관리 API
- 검색(search) API
- 집계(aggregation) API

|HTTP Method|기능| RDBMS 질의|
|-|-|-|
|GET|데이터 조회|SELECT|
|PUT|데이터 생성|INSERT|
|POST|인덱스 업데이트, 데이터 조회|UPDATE,SELECT|
|DELETE|데이터 삭제|DELETE|
|HEAD|인덱스 정보 확인|-|

### 인덱스 관리 API

인덱스 생성 요청은 PUT 메소드로, 아래와 같이 요청을 하였다.
위키북스 출판의 엘라스틱서치 실무 가이드(ver 6.4.2)를 참고해 아래 인덱스 생성 요청을 했는데 실패가 떨어졌다.

```json
PUT /customer
{
  "settings": {
    "number_of_shards":3,
    "number_of_replicas": 2
  },
  "mappings": {
    "_doc": {
      "properties": {
        "name": {"type": "text"},
        "age": {"type": "integer"},
        "gender": {"type": "keyword"},
        "birthday": {"type": "date"}
      }
    }
  }
}
>>>
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "The mapping definition cannot be nested under a type [_doc] unless include_type_name is set to true."
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "The mapping definition cannot be nested under a type [_doc] unless include_type_name is set to true."
  },
  "status": 400
}
```
에러 메시지를 보니 mapping type 을 둘 수 없다는 메시지이다. 위에서 7.0 이상 버전에서는 타입이 제거되었다는것을 깨닫게 되었다. 엘라스틱 홈페이지를 찾아보니 아래와 같은 내용이 있었다.
https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html
```
Elasticsearch 7.x
Specifying types in requests is deprecated. For instance, indexing a document no longer requires a document type. The new index APIs are PUT {index}/_doc/{id} in case of explicit ids and POST {index}/_doc for auto-generated ids. Note that in 7.0, _doc is a permanent part of the path, and represents the endpoint name rather than the document type.

The include_type_name parameter in the index creation, index template, and mapping APIs will default to false. Setting the parameter at all will result in a deprecation warning.

The _default_ mapping type is removed.
```
 따라서 type인 [_doc] 을 제거 후 다시 생성을 시도했다. 그 결과 아래와 같이 성공하였다.
```json
PUT /customer
{
  "settings": {
    "number_of_shards":3,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "name": {"type": "text"},
      "age": {"type": "integer"},
      "gender": {"type": "keyword"},
      "birthday": {"type": "date"}
    }
  }
}
>>>
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "customer"
}
```

인덱스 삭제 또한 진행해 보았다. 아래와 같이 응답이 내려왔다.
```json
DELETE /customer
>>>
{
  "acknowledged" : true
}
```
### 문서 관리 API

POST 로 문서 생성 요청을 해보았다.
```json
POST /customer/_doc/1
{
  "name": "sk ha",
  "age": "20",
  "gender": "male",
  "birthday": "2000-01-01"
}
>>>
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

PUT으로 요청을 다시 해보았다. PUT은 INSERT 역할이라고 하는데, 과연 기존 데이터가 있는데 어떻게 요청 응답이 내려올지 궁금했다. 그 결과 오류를 뱉을거라는 기대와 달리 업데이트를 시켜줬다.
POST, PUT 모두 동일하게 생성 및 수정을 하는것을 확인할 수 있었다.
```json
PUT /customer/_doc/1
{
  "name": "sk haha",
  "age": "21",
  "gender": "female",
  "birthday": "2000-01-02"
}
>>>
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```
GET 으로 조회를 하면 아래와 같이 응답이 내려온다.
```json
GET /customer/_doc/1
>>>
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "sk haha",
    "age" : "21",
    "gender" : "female",
    "birthday" : "2000-01-02"
  }
}
```

DELETE 로 삭제를 진행해 보았다.
```json
DELETE /customer/_doc/1
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 4,
  "result" : "deleted",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

ID 값을 주지 않고 데이터를 생성해 보았다.
여기서 주의해야할 점은 ID를 명시하지 않고 문서를 생성할 때 반드시 POST 메소드만 사용해야 한다.
PUT 메소드를 사용할 시 아래와 같이 오류를 받았다.
```json
PUT /customer/_doc
{
  "name": "sk haha",
  "age": "21",
  "gender": "female",
  "birthday": "2000-01-02"
}
>>>
{
  "error": "Incorrect HTTP method for uri [/customer/_doc?pretty] and method [PUT], allowed: [POST]",
  "status": 405
}
````
POST를 사용할 시 정상적으로 문서가 생성된 것을 확인할 수 있다. 여기서 확인해야할 점은 UUID를 통해 생성된 id 값이다. id가 임의로 생성된 것을 확인할 수 있는데, 업데이트를 고려해서 데이터베이스 테이블의 식별 값과 일치하게 id를 생성해주는것이 좋다.
```json
POST /customer/_doc
{
  "name": "sk haha1",
  "age": "21",
  "gender": "female",
  "birthday": "2000-01-02"
}
>>>
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "6YabqG4BFH0mFiOnAd5Y",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```
### 검색 API

검색 API 사용방식은 두가지가 존재한다.
- HTTP URI 형태의 파라미터를 추가해 검색하는 방법.
- RESTful API 방식인 Query DSL을 사용해 request body에 추가해 요청하는 방법.

검색 실습을 위해 엘라스틱서치 공식 레퍼런스에서 제공하는 더미 데이터를 bulk API를 이용하여 입력해 보았다.
```
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@/Users/shabre/Downloads/accounts.json"

curl "localhost:9200/_cat/indices?v"
>>>
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank                     LrOXnLvERUSPM1AFXZouQg   1   1       1000            0    414.2kb        414.2kb
green  open   .kibana_task_manager_1   M8qPWIC-Sh2j11Gc9u8UZg   1   0          2            0     12.5kb         12.5kb
green  open   .apm-agent-configuration UEPZ1IcDSHyHa40y7Z43Gg   1   0          0            0       283b           283b
green  open   .kibana_1                wc7Nd7NxT0OS7cjIvw6nDw   1   0          9            0       32kb           32kb
yellow open   customer                 nBgWIm5gR_q1Wi6AcdhRdw   3   2          6            0     12.5kb         12.5kb
```

bank 인덱스에 1000개의 데이터가 정상 입력된것을 확인할 수 있었다.

#### HTTP URI 검색
age = 30 인 회원을 검색해 보았다. 그 결과 47개의 결과를 얻어낼 수 있었다.
```json
GET /bank/_search?q=age:30&pretty=true

{
  "took" : 22,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 47,
      "relation" : "eq"
    },
    "max_score" : 1.0,
  ...
  }
}
```

#### request body 방식 질의
교재에서는 필드 검색을 term으로 검색을 하고, 공식 레퍼런스에선 match로 검색을 하여서 둘의 차이가 뭔지 살펴보니 공식 레퍼런스에서 다음과 같은 설명이 있었다.

```
Term-level queries
You can use term-level queries to find documents based on precise values in structured data. Examples of structured data include date ranges, IP addresses, prices, or product IDs.
```

bank 인덱스의 매핑을 따로 설정하지 않고 bulk입력을 진행하였더니 모든 타입 정보가 text로 설정이 되어버렸다. 따라서 term 으로 검색을 할 경우 필드명을 gender.keyword로 설정하면 match 에서 gender로 검색한 결과와 동일하게 나오게 된다.
데이터 검색에 여러가지 옵션이 있는데, 이것은 교재 4장 데이터 검색을 공부할때 제대로 알아보려고 한다.

```json
POST /bank/_search
{
  "query": {
    "match": {
      "gender": "M"
    }
  }
}

>>>
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 507,
      "relation" : "eq"
    },
    "max_score" : 0.67925805,
    ...
  }
}
```

### 집계 API
아래 집계 쿼리에 대해 설명을 하면, state기준으로 group by 를 진행한다.
그렇게 되면 group by된 데이터들이 buckets에 담겨 출력되게 된다.

이때 쿼리를 자세히 보면 aggs 안에 aggs가 중첩으로 되어있는데, 이것은 group by된 한 bucket의 average를 집계하는 쿼리이다.
이처럼 집계 쿼리를 중첩적으로 두어 버킷 안에 버킷을 두는 것이 가능하다.

```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
>>>
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30,
          "average_balance" : {
            "value" : 26073.3
          }
        },
        {
          "key" : "MD",
          "doc_count" : 28,
          "average_balance" : {
            "value" : 26161.535714285714
          }
        },
        {
          "key" : "ID",
          "doc_count" : 27,
          "average_balance" : {
            "value" : 24368.777777777777
          }
        },
        {
          "key" : "AL",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 25739.56
          }
        },
        {
          "key" : "ME",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 21663.0
          }
        },
        {
          "key" : "TN",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 28365.4
          }
        },
        {
          "key" : "WY",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 21731.52
          }
        },
        {
          "key" : "DC",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 23180.583333333332
          }
        },
        {
          "key" : "MA",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 29600.333333333332
          }
        },
        {
          "key" : "ND",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 26577.333333333332
          }
        }
      ]
    }
  }
}
```

### 스키마리스
엘라스틱 서치는 사용 편의성을 위해 스키마리스 기능을 제공한다. 스키마리스란 인덱스를 생성하는 과정 업이 문서가 추가되어도 문서가 인덱싱 되도록 하는 기능이다. 하지만 스키마리스 기능은 데이터 필드가 자동으로 정의되기 때문에 검색이 원하는 대로 작동하지 않을 가능성이 있고, 이는 검색 품질의 저하로 이어지게 된다. 따라서 특수한 경우가 아니면 스키마리스 기능은 사용하지 않는것이 좋다.
