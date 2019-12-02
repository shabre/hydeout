---
layout: post
title: elastic search 데이터 모델링
categories:
    - elastic search
excerpt_separator: "<!--more-->"
---
# 매핑 API 이해하기

엘라스틱서치가 기본적으로 검색 엔진이기 때문에, 매핑 설정에 따라 엔진의 성능이 결정될 수 있다. 각 필드의 속성에는 메타데이터가 포함되고, 이 메타데이터는 문서가 어떻게 역색인으로 변환되는지를 상세하게 정의할 수 있기 때문에 중요하다. 또한 한번 생성된 인덱스의 매핑 타입은 변경할 수 없기 때문에 더더욱 중요하다고 할 수 있다.

매핑 정보를 설정할 때는 다음과 같은 사항을 고민해야 한다.
- 문자열을 분석할 것인가?
- _source에는 어떤 필드를 정의할 것인가?
- 날짜 필드를 가지는 필드는 무엇인가?
- 매핑에 정의됮 않고 유입되는 필드는 어떻게 처리할 것인가?

## 매핑 인덱스 만들기

## 매핑 파라미터

### analyzer

해당 필드의 데이터를 형태소 분석하겠다는 의미의 파라미터. text 타입은 기본적으로 analyzer 매핑 파라미터를 사용해야함. standard analyzer가 기본값임.

### normalizer

term query에 분석기를 사용하기 위해 사용된다. cafe, Cafe, caFe 는 서로 다른 문서로 인식되지만 asciifolding 과 같은 필터를 사용하면 같은 데이터로 인식되게 할 수 있다.

### coerce

색인 시 자동 변환을 허용할지 여부를 설정하는 파라미터다. 사용으로 설정하면 "10" 과 같은 숫자 형태의 문자열이 Integer 필드에 들어온다면 자동으로 숫자로 변환 시켜준다.

### copy_to

매핑 파라미터를 추가한 필드의 값을 지정한 필드로 복사한다. keyword를 text 타입으로 저장하여 형태소 분석을 할 수도 있고, 여러개의 필드의 조합으로 만들 수도 있다.

### fielddata

엘라스틱서치가 힙 공간에 생성하는 메모리 캐시이다. 현재는 반복적인 메모리 부족과 잦은 GC로 자주 사용하지 않고, 기본적으로 비활성화 처리되어 있다.

### doc_value

엘라스틱서치에서 사용하는 기본 캐시다. text 타입을 제외한 모든 타입에서 기본적으로 doc_value 캐시를 사용한다. 운영체제의 파일 시스템 캐시를 사용함으로 힙 사용에 대한 부담을 줄이고, 메모리 연산과 비슷한 성능을 보여준다.

### dynamic

매핑에 필드를 추가할 때 동적으로 생성할지에 대한 여부를 결정한다.
- true: 새로 추가되는 필드를 매핑에 추가한다.
- false: 새로 추가되는 필드를 무시한다. 해당 필드는 색인되지 않아 검색은 안되지만 _source에는 남는다.
- strict: 새로운 필드가 감지되면 예외가 발생되고 문서 자체가 색인되지 않는다.

### enabled

검색 결과에는 포함하지만 색인하고 싶지 않은 경우 false로 설정하면 된다.
이 세팅은 top-level mapping definition과 object 필드에만 적용이 가능하다. 그이유는 엘라스틱 서치가 enable이 켜져 있으면 JSON parsing 을 중단하기 때문이다.

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "user_id": {
        "type":  "keyword"
      },
      "last_updated": {
        "type": "date"
      },
      "session_data": {
        "type": "object",
        "enabled": false
      }
    }
  }
}
>>>
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "my_index"
}


PUT my_index/_doc/session_1
{
  "user_id": "kimchy",
  "session_data": {
    "arbitrary_object": {
      "some_array": [ "foo", "bar", { "baz": 2 } ]
    }
  },
  "last_updated": "2015-12-06T18:20:22"
}
>>>
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "session_1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

PUT my_index/_doc/session_2
{
  "user_id": "jpountz",
  "session_data": "none",
  "last_updated": "2015-12-06T18:22:13"
}
>>> 
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "session_2",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

GET my_index/_doc/session_2
>>>
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "session_2",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "user_id" : "jpountz",
    "session_data" : "none",
    "last_updated" : "2015-12-06T18:22:13"
  }
}


GET my_index/_search
{
  "query": {
    "match": {
      "some_data": "none"
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
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}

```

### format

엘라스틱서치에서는 날짜/시간을 문자열로 표시한다. 날짜/시간을 문자열로 변경할 때 엘라스틱서치가 제공하는 미리 구성된 포맷을 사용할 수 있다.


### ignore_above

필드에 저장하는 문자열이 지정한 크기를 넘으면 빈 값으로 저장되게 한다. 지정한 크기만큼이 아니라 빈 값이 들어가게 된다.

### ignore_malformed

잘못 매핑된 필드는 제외하고 다른 필드들은 색인이 가능하게 하는 필드이다.

### index

필드값을 색인할지를 결정한다. 기본값은 true 이며 false로 변경하면 색인하지 않는다.
enable 옵션과 어떤부분이 다른지 확인해 보았다.

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "user_id": {
        "type":  "keyword"
      },
      "last_updated": {
        "type": "date"
      },
      "session_data": {
        "type": "object",
        "index": false
      }
    }
  }
}
>>>
{
  "error": {
    "root_cause": [
      {
        "type": "mapper_parsing_exception",
        "reason": "Mapping definition for [session_data] has unsupported parameters:  [index : false]"
      }
    ],
    "type": "mapper_parsing_exception",
    "reason": "Failed to parse mapping [_doc]: Mapping definition for [session_data] has unsupported parameters:  [index : false]",
    "caused_by": {
      "type": "mapper_parsing_exception",
      "reason": "Mapping definition for [session_data] has unsupported parameters:  [index : false]"
    }
  },
  "status": 400
}
```
object 타입으로 인덱스 생성을 시도할 시 생성이 정상적으로 되지 않았다.

enable과는 반대로 text, keyword 타입으로는 생성이 된다.

```json
PUT my_index/_doc/session_1
{
  "user_id": "kimchy",
  "session_data": "SESSION1",
  "last_updated": "2015-12-06T18:20:22"
}

{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "session_1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

PUT my_index/_doc/session_2
{
  "user_id": "jpountz",
  "session_data": "SESSION2",
  "last_updated": "2015-12-06T18:22:13"
}

{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "session_2",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

GET my_index/_doc/session_1

{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "session_1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "user_id" : "kimchy",
    "session_data" : "SESSION1",
    "last_updated" : "2015-12-06T18:20:22"
  }
}

GET my_index/_search
{
    "query": {
    "match": {
      "session_data": "SESSION1"
    }
  }
}

{
  "error": {
    "root_cause": [
      {
        "type": "query_shard_exception",
        "reason": "failed to create query: {\n  \"match\" : {\n    \"session_data\" : {\n      \"query\" : \"SESSION1\",\n      \"operator\" : \"OR\",\n      \"prefix_length\" : 0,\n      \"max_expansions\" : 50,\n      \"fuzzy_transpositions\" : true,\n      \"lenient\" : false,\n      \"zero_terms_query\" : \"NONE\",\n      \"auto_generate_synonyms_phrase_query\" : true,\n      \"boost\" : 1.0\n    }\n  }\n}",
        "index_uuid": "zkSiCVd5TkGUx4piNpYu-w",
        "index": "my_index"
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "my_index",
        "node": "Xc75QpnGRRC87ZonYfVU3g",
        "reason": {
          "type": "query_shard_exception",
          "reason": "failed to create query: {\n  \"match\" : {\n    \"session_data\" : {\n      \"query\" : \"SESSION1\",\n      \"operator\" : \"OR\",\n      \"prefix_length\" : 0,\n      \"max_expansions\" : 50,\n      \"fuzzy_transpositions\" : true,\n      \"lenient\" : false,\n      \"zero_terms_query\" : \"NONE\",\n      \"auto_generate_synonyms_phrase_query\" : true,\n      \"boost\" : 1.0\n    }\n  }\n}",
          "index_uuid": "zkSiCVd5TkGUx4piNpYu-w",
          "index": "my_index",
          "caused_by": {
            "type": "illegal_argument_exception",
            "reason": "Cannot search on field [session_data] since it is not indexed."
          }
        }
      }
    ]
  },
  "status": 400
}
```
색인이 되지 않은 필드를 검색하려고 하면 위와 같은 예외를 내뱉는다.

### fields

다중 필드를 설정할 수 있는 옵션이다. string 값을 각각 다른 분석기로 처리하도록 설정할 수 있다. 기본 필드는 전문 검색을 하고, 필드안의 추가 필드는 집계용으로 사용할 수 있다.

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "fields": {
          "english": {
            "type":     "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}

PUT my_index/_doc/1
{ "text": "quick brown fox" }

PUT my_index/_doc/2
{ "text": "quick brown foxes" }

GET my_index/_search
{
  "query": {
    "multi_match": {
      "query": "quick brown foxes",
      "fields": [
        "text",
        "text.english"
      ],
      "type": "most_fields"
    }
  }
}
>>>
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.604755,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.604755,
        "_source" : {
          "text" : "quick brown foxes"
        }
      },
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.9116078,
        "_source" : {
          "text" : "quick brown fox"
        }
      }
    ]
  }
}

```

### norms

문서의 _score 값 계산에 필요한 정규화 인수를 사용할지 여부를 설정한다.

### null_value

기본적으로 문서에 필드가 없거나 값이 null이면 색인 시 필드를 생성하지 않는다.
이 경우 null_value를 설정하면 값이 null이더라도 필드를 생성하고 그에 해당하는 값으로 저장한다.
이때 null 값을 분명하게 지정 해줘야 설정해놓은 값으로 저장이 된다.

**변환되는 null_value 는 그 필드의 타입과 동일한 타입으로 설정을 해야한다.**
```json
PUT my_index
{
  "mappings": {
    "properties": {
      "status_code": {
        "type":       "keyword",
        "null_value": "NULL"
      }
    }
  }
}

//null -> "NULL"로 치환되어 저장
PUT my_index/_doc/1
{
  "status_code": null
}

//null 값이 명시적으로 없기 때문에 치환되지 않고 저장됨
PUT my_index/_doc/2
{
  "status_code": []
}

GET my_index/_search
{
  "query": {
    "term": {
      "status_code": "NULL"
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "status_code" : null
        }
      }
    ]
  }
}
```

### position_increment_gap

배열 형태의 데이터를 색인할 대 검색의 정확도를 높이기 위해 제공되는 옵션. 단어와 단어 사이의 간격을 허용할지를 설정한다.

### properties

오브젝트 타입이나 중첩 타입의 스키마를 정의 할 때 사용되는 옵션.

### search_analyzer

일반적으로 색인과 검색 시 같은 분석기를 사용하나, 다른 분석기를 적용하고 싶은 경우 설정할 수 있다.

### similarity

유사도 측정 알고리즘을 지정한다.

### store

필드의 값을 저장해 검색 결과에 값을 포함하기위한 매핑 파라미터이다.
기본적으로 엘라스틱서치에선 _source에 색인된 문서가 저장된다. 하지만 store 매핑 파라미터를 사용하면 해당 필드를 자체적으로 저장할 수 있다.

필요한 파라미터만 불러와서 검색할 수 있기 때문에 효율적일 수 있으나 디스크를 더 많이 차지하게 된다.

### term vector

루씬에서 분석된 용어의 정보를 포함할지 여부를 결정하는 매핑 파라미터이다.

## 메타 필드

메타 필드는 엘라스틱서치에서 생성한 문서에서 제공하는 특별한 필드다.
이것은 메타데이터를 저장하는 특수 목적의 필드로서 이를 이용하면 검색 시 문서를 다양한 형태로 제어하는것이 가능해진다.

### _index

해당 문서가 속한 인덱스의 이름을 담고 있다. 이를 이용해 검색된 문서의 인덱스명을 알 수 있으며, 해당 인덱스에 몇개의 문서가 있는지 확인할 수 있다.

### _type

해당 문서가 속한 매핑 타입 정보를 담고 있다. 이를 이용해 해당 인덱스 내부에서 타입별로 몇 개의 문서가 있는지 확인할 수 있다.
6.0 이상 버전에서 사용되지 않는다.

### _id

_id 메타 필드는 문서를 식별하는 유일한 키 값이다.

### _uid

_uid 메타 필드는 특수한 목적의 식별키로 # 태그를 사용해 _type 과 _id 값을 조합해 사용한다.
7 이상 버전에서는 지원되지 않는다.

### _source

문서의 원본 데이터를 제공한다. 내부에는 색인 시 전달된 원본 JSON 문서의 본문이 저장되어 있다.

storage overhead가 발생하면 enable 기능을 이용해 disable 시킬 수 있다.

만약 생각없이 _source 필드를 disable 시킨다면 나중에 후회할 만한 일이 생길 수 있다. _source field 를 사용하지 못한다면 아래와 같은 작업을 수행할 수 없다.

- update, update_by_query, reindex API
- highlighting
- 엘라스틱서치로부터 다른 저장소로의 이관, mapping 변경, 새로운 버전 업그레이드
- debug query, 색인 시점에서의 원본 데이터 aggregation
- 미래애 있을 인덱스 손상 자동 복구 기능

### _routing

특정 문서를 특정 샤드에 저장하기 위해 사용자가 지정하는 메타 필드.

### _ignored (책에 없음)

ignore_malformed 기능이 켜져있어서 무시된 모든 필드의 이름들을 색인하고 저장한다.

term, terms, exists로 검색이 가능하다.