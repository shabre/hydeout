---
layout: post
title: elastic search 데이터 모델링(2)
categories:
    - elastic search
excerpt_separator: "<!--more-->"
---

## 엘라스틱서치 분석기

엘라스틱서치의 분석기는 입력된 문서를 검색에 응용될 수 있도록 문장을 분석하고 저장하는 역할을 한다. 분석기의 설정에 따라 검색 엔진의 기능과 성능을 결정지을 수 있기 때문에 분석기를 잘 설정하는 부분은 중요하다.

텍스트 타입의 문서가 입력되었을 때 character fileter -> tokenizer filter -> token filter 의 세가지 과정을 거쳐서 문장 구조가 분석되어 저장되어 진다.

### character filter

문장을 분석하기 전에 입력 텍스트에 대해 특정한 단어를 변경하거나 HTML 태그와 같은 요소를 제거하는 전처리 역할을 하는 필터이다.

### tokenizer filter

tokenizer filter는 분석기를 구성할 때 하나만 사용할 수 있으며, 텍스트를 어떻게 나눌지에 대해 정의한다.

### token filter

토큰화된 단어를 하나씩 필터링해서 사용자가 원하는 토큰으로 변환한다. 불필요한 단어를 제거하거나 동의어 사전을 만들어 단어를 추가하는 기능을 수행할 수 있다. token filter는 여러개 지정할 수 있으며, 지정 순서에따라 저장되는 데이터가 달라질 수 있다.


## 분석기 사용법

엘라스틱서치는 형태소가 어떻게 분석되는지를 확인할 수 있는 _analyze API를 제공한다. 미리 정의된 분석기의 경우 이를 이용해 쉽게 테스트해볼 수 있다. 아래와 같이 `The quick brown fox.` 문장을 standard, whitespace 필터를 각각 적용해 보았다.

```json
POST _analyze
{
  "analyzer": "standard",
  "text":     "The quick brown fox."
}
>>>
{
  "tokens" : [
    {
      "token" : "the",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "quick",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "brown",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "fox",
      "start_offset" : 16,
      "end_offset" : 19,
      "type" : "<ALPHANUM>",
      "position" : 3
    }
  ]
}

POST _analyze
{
  "analyzer": "whitespace",
  "text":     "The quick brown fox."
}
>>>
{
  "tokens" : [
    {
      "token" : "The",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "quick",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "brown",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "fox.",
      "start_offset" : 16,
      "end_offset" : 20,
      "type" : "word",
      "position" : 3
    }
  ]
}
```
standard filter는 토큰을 ALPHANUM 타입으로 저장하고 fox 뒤의 마침표를 제거하였으며, whitespace filter는 토큰을 word 타입으로 저장하고, 마침표 제거를 하지 않은점을 볼 수 있다.

아래와 같이 직접 인덱스 정의 시 custom 분석기를 설정하여 사용할 수도 있다.
```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_folded": { 
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_text": {
        "type": "text",
        "analyzer": "std_folded" 
      }
    }
  }
}
```

### built-in analyzer

엘라스틱서치에서 기본적으로 제공하는 분석기가 있다. 아래 리스트와 같은 분석기가 현재 기본으로 제공되고 있다. 각 분석기는 character filter, tokenizer filter, token filter 로 구성되어 있다. 각각 analyzer에서는 옵션 파라미터에 값을 입력하여 추가적으로 정의할 수 있다.

- Fingerprint Analyzer
- Keyword Analyzer
- Language Analyzer
- Pattern Analyzer
- Simple Analyzer
- Standard Analyzer
- Stop Analyzer
- Whitespace Analyzer
- Custom Analyzer

대표적으로 사용되는 Standard Analyzer에 대해 알아보면, Standard Analyzer는 아래와 같은 구성으로 설정되어 있다.

#### Tokenizer
- Standard Tokenizer

#### Token Filters
- Lower case Token Filter
- Stop Token Filter
  
아래 옵션을 추가 설정할 수 있다.

#### Configuration
- max_token_length : 최대 토큰 길이를 추가하는 경우 length 길이만큼으로 분할 저장한다. 기본 255
- stopword: 사전 정의된 불용어 사전을 사용한다. 기본값은 사용하지 않음.
- stopword_path: stopword 가 저장된 파일의 경로

아래와 같의 인덱스에 정의할 수 있다.
```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english_analyzer": {
          "type": "standard",
          "max_token_length": 5,
          "stopwords": "_english_"
        }
      }
    }
  }
}
```

### Normalizer

Normalizer는 analyzer와 비슷하나 단일 토큰에만 적용된다는 점이 특징이다. Normalizer는 tokenizer를 가지지 않으며 char filter나 token filter에서만 사용 가능하다. 예를들어 lowercase filter에는 적용 가능하나 stemming filter에는 적용 불가능하다. 아래는 지원하는 normalizer 종류이다. 

arabic_normalization, asciifolding, bengali_normalization, cjk_width, decimal_digit, elision, german_normalization, hindi_normalization, indic_normalization, lowercase, persian_normalization, scandinavian_folding, serbian_normalization, sorani_normalization, uppercase.

### Character Filter

문장을 token 화 시키기 전 전처리 필터의 역할을 수행한다. 아래의 3가지 필터가 존재한다.

- HTML Strip Char Filter
- Mapping Char Filter
- Pattern Replace Char Filter

### Tokenizer

토크나이저 필터는 분석기를 구성하는 핵심 구성요소다. 전처리 필터를 거쳐 넘어온 문장은 tokenizer에 의해 token으로 나누어진다. 어떤 tokenizer를 설정했느냐에 따라 분석기의 전체적인 성격이 결정된다.

엘라스틱 서치에서 적용하는 tokenizer들은 다음과 같다

#### word oriented tokenizers

- standard tokenizer: Unicode segmentation 알고리즘에의해 대부분의 기호를 만나면 토큰으로 나눈다. 대부분의 언어에서 가장 좋은 선택이 될 수 있다.
- letter tokenizer: letter가 아닌것을 만나기 전 까지 letter 단위로 분해한다.
- lowercase tokenizer: letter tokenizer + 분해된 토큰들을 소문자로 변환한다.
- whitespace tokenizer: 띄어쓰기 단위로 나눈다.
- UAX URL Email tokenizer: standard tokenizer 기능 + url, email을 하나의 단어로 보도록 함
- Classic tokenizer: 영문법 기반의 토크나이저
- Thai tokenizer: Thai 문장을 토큰화 시켜줌

#### partail word tokenizer

- N-Gram Tokenizer: 단어를 특정 길이만큼 잘라서 토큰화 시킨다. ex) quick → [qu, ui, ic, ck]
- Edge N-Gram Tokenizer: 한 단어를 첫 길이부터 마지막 길이까지 잘라 토큰화 시킨다. ex) quick → [q, qu, qui, quic, quick]

#### structured text tokenizers

구조화된 텍스트(Identifier, 이메일주소, zip 코드, 도로명) 과 같은 곳에 사용된다.

- Keyword Tokenizer: 어떤 특별한 동작을 하지않는 텍스트 그대로 keyword로 저장하는 토크나이저이다.
- Pattern Tokenizer: 정규식을 이용하여 word separator 에 일치하거나 matching text를 찾아 단어를 나눈다.
- Simple Pattern Tokenizer: 정규식을 이용하여 매칭되는 단어만 선별한다. 정규식 subset으로만 제한하여 단어를 추출함으로 pattern tokenizer보다는 성능이 빠르다.
- Char Group Tokenizer: 구성된 char group 내의 단어들만으로 단어를 추출하며, 정규식을 이용한 토크나이저보다 성능이 좋다.
- Simple Pattern Split Tokenizer: Simple pattern tokenizer와 동일하나, term을 반환하진 않는다.
- Path Tokenizer: 파일의 경로를 토큰화 시키는 토크나이저다. ex) /foo/bar/baz → [/foo, /foo/bar, /foo/bar/baz ]

### Token Filter

토큰 필터는 토크나이저에서 분리된 토큰들을 변형하거나 추가, 삭제할 때 사용되는 필터이다.
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html 공식문서에 접속하면 token filter 들을 자세히 알아볼 수 있다. 