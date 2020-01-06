---
layout: post
title: elastic search 데이터 집계
categories:
    - elastic search
excerpt_separator: "<!--more-->"
---

# 집계

## 엘라스틱서치와 데이터분석

일반적인 통계 프로그램은 배치 방식으로 데이터를 처리한다. 대용량 데이터를 하둡이나 RDBMS에 저장하고 배치로 처리하는 방식이다. 반면 엘라스틱서치는 많은양의 데이터를 조각내어 관리한다. 그 덕분에 문서의 수가 늘어나도 배치 처리보다 실시간에 가깝게 문서를 처리할 수 있다.


일반적인 SQL과 엘라스틱서치의 Query DSL 을 비교하면 아래와 같다.

```
SELECT SUM(ratings) FROM movie_review GROUP BY movie_no;
```
이를 엘라스틱서치로 표현하면 아래와 같다.
```json
{
  "aggs":{
    "movie_no_agg":{
      "terms":{
        "field":"movie_no"
      
      },
      "aggs":{
        "ratings_agg":{
          "sum":{
            "field":"ratings"
          }
        }
      }
    }
  }
}
```

## 엘라스틱서치가 집계에 사용하는 기술

데이터를 분석하는데 있어 집계는 검색보다 더 많은 리소스를 사용한다. 

### 캐시

집계 쿼리로 값을 조회하면 마스터 노드가 여러 노드에 있는 데이터를 집계해 질의에 답변한다. 데이터의 양이 클수록 많은 양의 CPU와 메모리 자원이 소모된다. 엘라스틱서치에서는 자체 캐시를 제공하여 이러한 문제를 어느정도 해결해준다. 질의의 결과를 캐시에 두고 같은 질의에 대한 결과는 캐시에서 반환한다. 캐시를 적용하는 것만으로 인덱스의 성능을 대폭 향상시킬 수 있다. 일반적으로 메모리 크기의 1%를 할당하고 설정에서 변경 가능하다.

엘라스틱서치에서는 3가지의 캐시를 제공한다.

### Node query Cache
노드의 모든 샤드가 공유하는 LRU 캐시이다. 캐시 용량이 가득차면 사용량이 가장 적은 데이터를 삭제하고 새로운 결과를 캐싱한다. 기본적으로 10%의 필터 캐시가 메모리를 제어하고 쿼리 캐싱을 사용할지 여부는 elasticsearch.yml 파일에 옵션으로 추가하면 된다. 기본값은 true 이다.

### Shard request Cache
엘라스틱서치는 인덱스의 수평적 확산과 병렬 처리를 통한 서능 향상을 위해 고안된 캐시이다. 샤드는 데이터를 분산 저장하기 위한 단위로서, 그 자체가 온전한 기능을 가진 독립적인 인덱스라고 할 수 있다. Shard request Cache는 바로 이 샤드에서 수행된 쿼리의 결과를 캐싱한다. 샤드의 내용이 변경되면 캐시가 삭제되기 때문에 업데이트가 빈번한 인덱스에서는 성능 저하를 일으킬 수 있다.

### Field data Cache
엘라스틱서치가 필드에서 집계 연산을 수행할 때는 모든 필드 값을 메모리에 로드한다. 이러한 이유로 엘라스틱서치에서 계산되는 집계 쿼리는 성능적인 측면에서 비용이 많이 든다. Field data Cache는 집계가 계산되는 동안 필드의 값을 메모리에 보관한다.
 
## Aggregation API 이해하기

엘라스틱서치에서는 크게 4가자의 집계 기능을 제공한다.

- 버킷 집계: 쿼리 결과로 도출된 도큐먼트 집합에 대해 특정 기준으로 나눈 다음 나눠진 도큐먼트들에 대한 산술 연산을 수행한다. 이 때 나눠진 도큐먼트들의 모음들이 각 버킷에 해당한다.
- 메트릭 집계: 쿼리 결과로 도출된 도큐먼트 집합에서 필드의 값을 더하거나 평균을 내는 등의 산술 연산을 수행한다.
- 파이프라인 집계: 다른 집계 또는 메트릭 연산 결과를 집계한다.
- 행렬 집계: 버킷 대상이 되는 도큐먼트의 여러 필드에서 추출한 값으로 행렬 연산을 수행한다.

엘라스틱서치가 강력한 이유는 집계를 중첩해 사용할 수 있다는 점이다. 하위 집계가 상위 집계를 다시 집계하는 식이다. 중첩 횟수에 제한은 없지만 중첩할수록 성능은 하락한다.

### 집계 구문의 구조

기본적인 집계 구조는 아래와 같다.
```json
{
"aggregation": { //집계를 하기위해 명시
  "<aggregation_name>":{ //하위 집계의 이름
    "<aggregation_type>":{ //집계의유형(terms, date_histogram, sum ...)
      <aggregatrion_body> //type에 맞춰 내용 작성
    }
    [,"meta":{[<meta_data_body>]}]? //메타 필드 사용
    [,"aggregation":{[<sub_aggregation>]+ }]? //중첩 집계
  }
  [,"<aggregation_name_2>":{...}]* //같은 레벨에서의 또 다른 집계 정의
}
```

### 집계 영역

집계와 질의를 함께 수행하면 질의의 결과 영역 안에서 집계가 수행된다. 만약 질의가 생략된다면 내부적으로 match_all 쿼리로 수행되어 전체 문서에 대해 집계가 수행된다.

한 번의 집계를 통해 질의에 해당하는 문서들 내에서도 집계를 수행하고 전체 문서에 대해서도 집계를 수행해야 할 경우 global 버킷을 사용하면 된다.
```json
{
"query": { //질의
  "constant_score":{
    "filter":{
      "match": <필드 조건>
    }
  },
  "aggs": {
    "<집계 이름>": {
      "<집계 타입>": {
        "field": "<필드명>"
      }
    },
    "<집계 이름>": {
      "global": {},//질의를 하지 않은 문서를 대상으로 집계 수행
      "aggs": {
        "<집계 타입>": {
        "field": "<필드명>"
        }
      }
    },
  }
}
```

## 메트릭 집계

메트릭 집계를 사용하면 특정 필드에 대해 합이나 평균을 계산하거나 다른 집계와 중첩해서 결과에 대해 특정 필드의 _score 값에 따라 정렬을 수행하거나 지리 정보를 통해 접위 계산을 하는 등의 다양한 집계를 수행할 수 있다. 일반적으로 필드 데이터를 이용해 집계가 이뤄지지만 스크립트를 통해 조금 더 유연하게 집계를 수행할 수도 있다.

메트릭 집계 내에서도 단일 숫자 메트릭 집계와 다중 숫자 메트릭 집계로 나뉘는데, 단일 숫자 메트릭 집계는 집계를 수행한 결괏값이 하나라는 의미로서 sum과 avg등이 이에 속한다. 다중 숫자 메트릭 집계를 수행한 결괏값이 여러개가 될 수 있고, stats나 geo_bounds가 이에 속한다.

### 합산 집계

단일 메트릭 집계 연산으로 `sum` 집계 타입을 이용하여 합산한 값을 나타낼 수 있다. 아래는 total_byte를 집계하는 여러 방법이다.
```json
GET /apache-web-log/_search?size=0
{
  "query" : { //질의를 통해 선 필터링
    "constant_score" : {
      "filter" : {
          "match" : { "geoip.city_name" : "Paris" }
      }
    }
  },
  
  "aggs": {
    "total_bytes": {
      "sum": {
        "field": "bytes"
      }
    },
    "total_bytes": {
      "sum": {//합산 집계
        "script": { //스크립트 컨텍스트
          "lang": "painless",
          "source": "doc.bytes.value"//파리에 유입된 데이터의 총량. 위 집계와 동일
        }
      }
    },
    "total_bytes": {
      "sum": {
        "script": { 
          "lang": "painless",
          "source": "doc.bytes.value / params.divide_value", //아래 파라미터 값으로 나눈 값. 정수형으로 나누어서 나머지는 버림처리
          "params": {
            "divide_value": 1000
          }
        }
      }
    },
    "total_bytes": {
      "sum": {
        "script": { 
          "lang": "painless",
          "source": "doc.bytes.value / (double)params.divide_value",//나누는 값을 double로 캐스팅하여 소숫점 아랫자리 살리기
          "params": {
            "divide_value": 1000
          }
        }
      }
    }
  }
}
```

### 평균 집계

단일 메트릭 집계 연산으로 데이터의 평균값을 반환해준다.
```json
GET _search?size=0
{
  "query" : {
    "constant_score" : {
      "filter" : {
          "match" : { "geoip.city_name" : "Paris" }
      }
    }
  },
  "aggs": {
    "avg_bytes": {
      "avg": {//평균 집계
        "field": "bytes"
      }
    }
  }
}
```

### 최솟값 집계

단일 메트릭 집계 연산으로 가장 작은갑을 반환해준다.
```json
  
GET /apache-web-log/_search?size=0
{
  "query" : {
    "constant_score" : {
      "filter" : {
          "match" : { "geoip.city_name" : "Paris" }
      }
    }
  },
  "aggs": {
    "min_bytes": {
      "min": {//최솟값 집계
        "field": "bytes"
      }
    }
  }
}
```

### 최댓값 집계

단일 메트릭 집계 연산으로 가장 큰 값을 반환한다. 최솟값 집계에 min => max로 명시하면 된다.

### 개수 집계
개수 집계(Value Count Aggs) 는 단일 숫자 메트릭 집계이다. 데이터의 갯수를 셀 수 있다.
```json
GET /apache-web-log/_search?size=0
{
  "query" : {
    "constant_score" : {
      "filter" : {
          "match" : { "geoip.city_name" : "Paris" }
      }
    }
  },
  "aggs": {
    "bytes_count": {
      "value_count": {//갯수 집계
        "field": "bytes"
      }
    }
  }
}
```

### 통계 집계

통계 집계는 결과값이 여러 개인 다중 숫자 메트릭 집계이다. 통계 집계를 사용하면 앞서 살펴 본 합, 평균, 최대/최소, 갯수를 한번의 쿼리로 집계할 수 있다.
```json
GET /apache-web-log/_search?size=0
{
  "query" : {
    "constant_score" : {
      "filter" : {
          "match" : { "geoip.city_name" : "Paris" }
      }
    }
  },
  "aggs": {
    "bytes_stats": {
      "stats": {//통계 집계
        "field": "bytes"
      }
    }
  }
}
```

### 확장 통계 집계

확장 통계 집계는 통계 집계에 추가로 표준편차와와 관련된 값들을 제공한다. stats => extended_stats 를 사용하면 된다.

### 카디널리티 집계

카디널리티 집계는 단일 숫자 메트릭 집계에 해당한다. 갯수 집계와 유사하게 횟수를 계산하는데, 중복된 값은 제외한 고유한 값에 대한 집계를 수행한다. 하지만 모든 문서에 대해 중복된 값을 집계하는 것은 성능에 큰 영향을 줄 수 있기 때문에 근사치를 통해 집계를 수행한다.

### 백분위 수 집계
