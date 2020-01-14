---
layout: post
title: elastic search 데이터 집계(2)
categories:
    - elastic search
excerpt_separator: "<!--more-->"
---

# 데이터 집계(2)

## 버킷 집계
버킷 집계는 메트릭 집계와는 다르게 메트릭을 계산하지 않고 버킷을 생성한다. 생성되는 버킷은 또 다시 하위 집계를 한번 더 수행해서 집계된 결과에 대해 중첩된 집계를 수행하는것이 가능하다.

버킷을 생성한다는 것은 결과 데이터 집합을 메모리에 저장한다는 의미이기 때문에 중첩 단계가 깊어질수록 메모리 사용량은 증가한다. 엘라스틱서치에서는 기본적으로 사용 가능한 최대 버킷 수가 미리 정의되어 있고, search.max_buckets 값을 수정해 조정할 수 있다.

### 범위 집계

아래와 같이 apache-web-log의 필드 수를 범위로 집계가 가능하며, range를 여러 구간으로 배열입력하여 다중 결과를 얻어낼 수 있다.
```json
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "bytes_range" : {
            "range": {
              "field": "bytes",
              "ranges": [
                {
                  "to": 1000
                },
                {
                  "from": 1000,
                  "to": 2000
                },
                {
                  "from": 2000,
                  "to": 3000
                }
              ]
            }
        }
    }
}

// result
"aggregations" : {
    "bytes_range" : {
      "buckets" : [
        {
          "key" : "*-1000.0", //범위가 수행될 범위
          "to" : 1000.0, 
          "doc_count" : 666
        },
        {
          "key" : "1000.0-2000.0",
          "from" : 1000.0, //범위의 시작
          "to" : 2000.0, //범위의 끝
          "doc_count" : 754 // 갯수
        },
        {
          "key" : "2000.0-3000.0",
          "from" : 2000.0,
          "to" : 3000.0,
          "doc_count" : 81
        }
      ]
    }
  }
```

### 날짜 범위 집계

범위 집계와 유사하게 날짜를 기준으로 범위 집계를 수행한다.
```json
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "request count with date range" : {
            "date_range": {
              "field": "timestamp",
              "ranges": [
                {
                  "from": "2015-01-04T05:14:00.000Z",
                  "to": "2015-01-04T05:16:00.000Z"
                }
              ]
            }
        }
    }
}

// result
"aggregations" : {
    "request count with date range" : {
      "buckets" : [
        {
          "key" : "2015-01-04T05:14:00.000Z-2015-01-04T05:16:00.000Z",//집계에 대한 날짜 범위
          "from" : 1.42034844E12,//시작 날짜 밀리초
          "from_as_string" : "2015-01-04T05:14:00.000Z",//시작 날짜
          "to" : 1.42034856E12,//마지막 날짜 밀리초
          "to_as_string" : "2015-01-04T05:16:00.000Z",//마지막 날짜
          "doc_count" : 0//갯수
        }
      ]
    }
  }
```

### 히스토그램 집계

숫자 범위를 처리하기 위한 집계이다. 범위 집계와는 다르게 지정한 수치가 간격을 나타내고, 이 간격의 범위 내에서 집계를 수행한다.

```json
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "bytes_histogram" : {
            "histogram" : {
                "field" : "bytes",
                "interval": 10000
            }
        }
    }
}

//result
"aggregations" : {
    "bytes_histogram" : {
      "buckets" : [
        {
          "key" : 0.0,//0~10000
          "doc_count" : 4196
        },
        {
          "key" : 10000.0,//10000~20000
          "doc_count" : 1930
        },
        ...
        {
          "key" : 6.919E7,//마지막
          "doc_count" : 2
        }
      ]
    }
}
```

### 날짜 히스토그램 집계

히스토그램 집계와 유사하다. interval 에 year, quarter, month, week, day, hour, minute, second 표현식을 사용할 수 있다.

옵션으로 time_zone, offset을 설정하여 타임존, 시작 시간에 대한 상세 설정도 할 수 있다.

```json
GET /apache-web-log/_search?size=0
{
    "aggs" : {
        "daily_request_count" : {
            "date_histogram": {
              "field": "timestamp",
              "interval": "day",
              "time_zone": "+09:00", //한국 타임존 설정
              "offset": "+3h" //3시 이후부터 집계 시작
            }
        }
    }
}

//result
"aggregations" : {
    "daily_request_count" : {
      "buckets" : [
        {
          "key_as_string" : "2015-05-17T03:00:00.000+09:00", //3시 부터 하루 집계(한국 타임존 기준)
          "key" : 1431799200000,
          "doc_count" : 912
        },
        {
          "key_as_string" : "2015-05-18T03:00:00.000+09:00",
          "key" : 1431885600000,
          "doc_count" : 2903
        },
        {
          "key_as_string" : "2015-05-19T03:00:00.000+09:00",
          "key" : 1431972000000,
          "doc_count" : 2860
        },
        {
          "key_as_string" : "2015-05-20T03:00:00.000+09:00",
          "key" : 1432058400000,
          "doc_count" : 2888
        },
        {
          "key_as_string" : "2015-05-21T03:00:00.000+09:00",
          "key" : 1432144800000,
          "doc_count" : 436
        }
      ]
    }
  }
  }
  ```

### 텀즈 집계

keyword를 집계 하여 숫자가 높은 순으로 결과를 반환한다.
```json

GET /apache-web-log/_search?size=0
{
  "aggs" : {
    "request count by country" : {
      "terms" : {
        "field" : "geoip.country_name.keyword"
      }
    }
  }
}

//result
 "aggregations" : {
    "request count by country" : {
      "doc_count_error_upper_bound" : 47,//문서 수에 대한 오류의 상한선(각 샤드별로 계산되는 집계의 성능을 고려해 근사치를 계산하기 때문에 문서 수가 정확하지 않을 수 있음)
      "sum_other_doc_count" : 2334,//결과에 포함되지 않은 문서의 수
      "buckets" : [//최상위 버킷 목록
        {
          "key" : "United States",//질의에 해당하는 필드
          "doc_count" : 3974//갯수
        },
        {
          "key" : "France",
          "doc_count" : 855
        },
        {
          "key" : "Germany",
          "doc_count" : 510
        },
        ...
      ]
    }
  }
  ```


텀즈 집계 시 size 값을 줄 수 있는데, 이것은 각 샤드의 상위 n개의 집계 결과만 반환한 후 그것을 다시 합쳐서 보여주게 된다.

예를들어 샤드의 문서가 아래와같이 색인되어 있다고 가정한다.

A=[A(30), B(20), C(10), D(5), E(3)]

B=[A(10), B(6), C(3), D(2)]

C=[A(15), C(6), D(13), F(2)]

여기서 size=3 파라미터를 주게 되면 아래와 같이 각각 샤드의 상위 3개 결과를 가져오게 된다.

A=[A(30), B(20), C(10)]

B=[A(10), B(6), C(3)]

C=[A(15), C(6), D(13)]

결과값들을 합쳐서 다시 상위 3개만 최종 결과값으로 도출하면 아래와 같다.

[A(45), B(26), C(19)]

실제로는 D(20) 이 3번째로 많은 값이지만 size 제한으로 인해 위와 같은 결과가 나타났다.

위 결과를 보면 알 수 있듯이 **size 값을 작게 주면 성능은 향상되나 정확도는 떨어질 수 있다.**

## 파이프라인 집계

파이프라인 집계는 다른 집계와 달리 쿼리 조건에 부합하는 문서에 대해 집계를 수행하는 것이 아니라 다른 집계로 생성된 버킷을 참조하여 집계를 수행한다. 파이프라인 집계에서는 부모(parent), 형제(sibling) 두가지 유형이 있다.

