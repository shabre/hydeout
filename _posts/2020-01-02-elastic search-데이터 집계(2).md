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

파이프라인 집계를 수행할 때는 buckets_path 파라미터를 사용해 참조할 집계의 경로를 지정함으로써 체인 형식으로 집계 간의 연산이 이뤄진다.

### 형제 집계(Sibling Aggregation)

형제 집계는 동일 선상의 위치에서 수행되는 새 집계를 의미한다. 형제 집계에는 아래와 같은 집계가 포함된다.

- 평균 버킷 집계(Avg Bucket Aggregation)
- 최대 버킷 집계(Max Bucket Aggregation)
- 최소 버킷 집계(Min Bucket Aggregation)
- 합계 버킷 집계(Sum Bucket Aggregation)
- 통계 버킷 집계(Stats Bucket Aggregation)
- 확장 통계 버킷 집계(Extended Stats Bucket Aggregation)
- 백분위수 버킷 집계(Percentiles Bucket Aggregation)
- 이동 평균 집계(Moving Average Bucket Aggregation)

```json
GET /apache-web-log/_search?size=0
{
  "aggs": {
    "histo": {
      "date_histogram": {//분단위 집계
        "field": "timestamp",
        "interval": "minute"
      },
      "aggs": {//분단위 집계 내부에서 byte sum
        "bytes_sum": {
          "sum": {
            "field": "bytes"
          }
        }
      }
    },
    "max_bytes": {//파이프라인 집계의 이름
      "max_bucket": {//합산된 데이터량 가운데 가장 큰 값을 구하기 위해 max_bucket 사용(min_bucket, avg_bucket, stats_bucket, extended_stats_bucket, percentiles_bucket, moving_avg_bucket 사용가능)
        "buckets_path": "histo>bytes_sum"//참조할 버킷 path
      }
    }
  }
}

//result
 "aggregations" : {
    "histo" : {
      "buckets" : [
        {
          "key_as_string" : "2015-05-17T10:05:00.000Z",
          "key" : 1431857100000,
          "doc_count" : 74,
          "bytes_sum" : {
            "value" : 5185322.0
          }
        },
        ...
        {
          "key_as_string" : "2015-05-20T21:05:00.000Z",
          "key" : 1432155900000,
          "doc_count" : 86,
          "bytes_sum" : {
            "value" : 4127318.0
          }
        }
      ]
    },
    "max_bytes" : {
      "value" : 2.06109322E8,
      "keys" : [
        "2015-05-18T21:05:00.000Z"
      ]
    }
  }
```

### 부모 집계(Parent Aggregation)

부모 집계는 집계를 통해 생성된 버킷을 사용해 계산을 수행하고, 결과를 기존 집계에 반영한다. 부모 집계에 해당하는 집계는 다음과 같다.

- 파생 집계 (Derivative Aggregation)
- 누적 집계 (Cumulative Sum Aggregation)
- 버킷 스크립트 집계 (Bucket Script Aggregation)
- 버킷 셀렉터 집계 (Bucket Selector Aggregation)
- 시계열 차분 집계 (Serial Differencing Aggregation)

파생 집계는 부모 히스토그램 또는 날짜 히스토그램 집계에서 지정된 메트릭의 파생 값을 계산하는 상위 파이프라인 집계다. 이는 부모 히스토그램 집계의 측정 항목에 대해 동작하고, 히스토그램 집계에 의한 각 버킷의 집계 값을 비교해서 차이를 계산한다. 지정된 메트릭은 숫자여야 하고, 상위에 해당하는 집계의 min_doc_count가 0보다 큰 값으로 설정되는 경우 일부 간격이 결괴에서 생략될 수 있기 때문에 min_doc_count 값을 0으로 설정해야 한다.

파생집계의 경우에는 선행되는 데이터가 존재하지 않으면 집계를 수행할 수가 없다. 데이터 중간에 노이즈가 존재하거나 필드에 값이 없을 수 있는데 이것을 갭(gap) 이라 한다. 갭은 여러가지 이유로 발생할 수 있으며, 일반적인 이유는 다음과 같다.

- 어느 하나의 버킷 안으로 포함되는 문서들에 요청된 필드가 포함되지 않은 경우
- 하나 이상의 버킷에 대한 쿼리와 일치하는 문서가 존재하지 않는 경우
- 다른 종속된 버킷에 값이 누락되어 계산된 메트릭이 값을 생성할 수 없는 경우

이러한 경우의 처리를 하는 메커니즘이 필요한데 이를 갭 정책(gap_policy) 라 한다. 모든 파이프라인 집계에서는 gap_policy 파라미터를 허용한다. 갭 정책은 두가지가 존재한다.

-skip: 누락된 데이터를 버킷이 존재하지 않는 것으로 간주하여 다음 파이프라인으로 건너뛴다.
-insert_zeros: 누락된 값을 0으로 대체하여 파이프라인 집계를 진행한다.

```json
GET /apache-web-log/_search?size=0
{
  "aggs": {
    "histo": {
      "date_histogram": {
        "field": "timestamp",
        "interval": "day"
      },
      "aggs": {
        "bytes_sum": {
          "sum": {
            "field": "bytes"
          }
        },
        "sum_deriv": {//일단위로 생성된 byte_sum을 이전값과 현재값 변화 추이 집계(파생)
          "derivative": {//파생 집계
            "buckets_path": "bytes_sum"
          }
        }
      }
    }
  }
}


//result
"aggregations" : {
    "histo" : {
      "buckets" : [
        {
          "key_as_string" : "2015-05-17T00:00:00.000Z",
          "key" : 1431820800000,
          "doc_count" : 1632,
          "bytes_sum" : {
            "value" : 4.14259902E8
          }
        },
        {
          "key_as_string" : "2015-05-18T00:00:00.000Z",
          "key" : 1431907200000,
          "doc_count" : 2893,
          "bytes_sum" : {
            "value" : 7.88636158E8
          },
          "sum_deriv" : {//이전값과의 변화추이
            "value" : 3.74376256E8
          }
        },
        {
          "key_as_string" : "2015-05-19T00:00:00.000Z",
          "key" : 1431993600000,
          "doc_count" : 2896,
          "bytes_sum" : {
            "value" : 6.65827339E8
          },
          "sum_deriv" : {
            "value" : -1.22808819E8
          }
        },
        {
          "key_as_string" : "2015-05-20T00:00:00.000Z",
          "key" : 1432080000000,
          "doc_count" : 2578,
          "bytes_sum" : {
            "value" : 8.78559106E8
          },
          "sum_deriv" : {
            "value" : 2.12731767E8
          }
        }
      ]
    }
  }
```

## 근사값으로 제공되는 집계 연산

대부분의 경우 전체 데이터를 대상으로 집계가 일어나나 일부 집계에서는 근사값을 기반으로 한 결과를 제공한다.

### 집계 연산과 정확도

집계는 크게 버킷 집계, 메트릭 집계, 파이프라인 집계, 행렬 집계 4가지로 분류될수 있다. 이 가운데 실제로 수학적인 계산을 수행하는 것은 메트릭 집계이다. 대부분의 메트릭 집계는 100% 정확한 결과를 제공한다. 하지만 3가지의 집계 연산은 근사값을 기반으로 계산을 한다. 3가지 집계는 아래와 같다.

- 카디널리티 집계
- 백분위 수 집계
- 백분위 수 랭크 집계

백분위 수 집계, 백분위 수 랭크 집계는 사용 빈도가 높은 편은 아니나 카디널리티 집계는 집계 연산 시 중복 제거를 위해 자주 사용되는 연산이다. 따라서 카디널리티 집계를 사용할 때 주의해야 한다. 카디널리티 집계는 왜 근사값으로 계산이 되는지 아래에서 알아본다.

### 분산 환경에서 집계 연산의 어려움

엘라스틱서치는 분산 환경으로 데이터 집계를 할 때 Grouping 과정이 필수적이다. 엘라스틱서치에서는 독자적인 Grouping 기술인 Aggregation API 를 사용한다. 집계 연산 요청이 들어오면 각 샤드에서 집계 작업이 수행되고 중간 결과를 취합해 최종 결과를 도출하는 방식으로 동작한다.

일반적인 메트릭 집계의 경우 이 과정이 잘 이어진다. 하지만 카디널리티 집계의 경우는 조금 특수하다.

카디널리티 집계는 중복을 제거한 데이터 리스트를 코디네이터 노드로 전송한다. 만약 10개의 샤드에서 각각 50MB의 데이터를 전송했다면, 500MB의 데이터를 메모리에 올린 후 추가 중복 제거 작업에 들어가게 된다. 데이터의 양과 샤드의 양은 더 늘어날 수도 있고, 이는 엘라스틱서치 성능에 치명적인 영향을 끼치게 된다.

이러한 분산환경에서의 성능 향상을 위해선 3가지 요소중 하나를 포기해야 한다.

- 분산환경을 포기
  - 정확한 데이터를 실시간으로 제공가능
  - 단일 서버로 서비스를 구성하고 메모리에서 처리
  - 처리가능한 데이터의 크기에 제한이 있다.
  - 복잡한 연산을 수행할때 빠르게 처리가 가능하다.
- 실시간성을 포기
  - 정확한 데이터를 제공할 수 있다.
  - 디스크 기반의 분산환경으로 서비스를 구성한다.
  - 데이터 크기에 제한이 없고 대용량 데이터를 처리 가능하다.
  - 처리 시간이 오래 걸린다
- 정확도를 포기
  - 데이터를 실시간으로 제공 가능하다.
  - 메모리 기반의 분산환경 서비스를 구축한다.
  - 데이터 크기에 제한이 없고 대용량 처리가 가능하다.
  - 근사치를 이용해 빠르게 계산한다.

엘라스틱서치의 경우 정확도를 포기하여 분산환경과 실시간성을 지킨 케이스가 될 수 있다.