---
layout: post
title: "[JQuery] JQuery란?"
categories: Development
excerpt_separator: "<!--more-->"
---

## JQuery 에 대해서

웹서비스 개발을 하면서 ajax를 이용한 비동기 통신을 구현하면서 JQuery를 몇번 사용하였다. 하지만 개념을 정확히 알지 못해서 JQuery를 공부하려 한다.

JQuery란 간단히 말하면 1. 엘리먼트를 선택하는 강력한 방법과 2. 선택된 엘리먼트들을 효율적으로 제어할 수 있는 Javascript 라이브러리이다. 

라이브러리는 http://jquery.org 에서 소스를 받아 사용하거나 직접 스크립트에서 호출하여 사용이 가능하다. 하지만 외부 CDN(Content delivery network)을 이용하게되면 웹 페이지의 로딩 속도가 느려질 수 있으니 되도록 소스를 다운로드해서 사용하는것이 좋다.

### JQuery의 문법
$(제어대상).method1().method2();
제어대상은 주어가 되고, 뒤의 메소드는 서술어가 된다. 여기서 $()는 jQuery() 함수의 약자로, JQuery 객체를 반환하는 함수이다.

주어 뒤에 서술어가 꼬리에 꼬리를 무는 것으로 이를 JQuery의 Chain이라 한다. 예를 들면

$('li').css('color', 'red');

라는 명령을 해석하면 li 태그 전체에 대해서 css 메소드의 color를 red로 바꾸도록 하는 명령이 되는 것이다.

기존의 Javascript으로 이 명령을 코딩하려면 li tag를 getElementsByTagName으로 불러온 후 그 객체의 속성값을 변경시켜줘야 하는 코드를 작성해야 한다. 하지만 이 한줄이면 엘리먼트를 아주 쉽게 변경할 수 있게된다. 만약에 $('.class') 이렇게 입력을 하면 해당 클래스의 엘리먼트를 변경하는 코드가 되는 것이다.


### JQuery 응용

#### .ajax


사실 JQuery를 ajax때문에 입문하게 되었다고해도 과언이 아니다. 그만큼 JQuery에서 가장 중요한 library중에 하나이다.
호출 함수는 jQuery.ajax() 또는 $.ajax이다. 

ajax에 사용되는 주요 옵션들은 다음과 같다.

- data: 서버로 데이터를 전송할 때 사용
- dataType: 서버에서 전송한 데이터를 어떤 형식으로 받을지 정한다. 종류는 xml, json, script, html이 있다.
- success: 성공했을 때 호출할 콜백을 지정한다. Function(PlainObject data, String textStatus, jqXHR jqXHR)
- type: 데이터를 전송하는 방법을 지정한다. get, post가 있다.

다음은 ajax를 이용한 웹서버와의 통신 예제이다.

```javascript
    $('#execute').click(function(){
        $.ajax({
            url:'./time3.php',//time3.php 페이지의 값을 얻어온다
            dataType:'json', //얻어오는 데이터 형식은 json이다
            success:function(data){ //통신에 성공하면 json을 객체화 하여 data에 담는다.
                var str = '';
                for(var name in data){ //data 의 key값들을 순서대로 꺼낸다
                    str += '<li>'+data[name]+'</li>';
                }
                $('#timezones').html('<ul>'+str+'</ul>');//timezones id를 가진 태그에 ul태그를 추가한다
            }
        })
    })
```
#### .map
.map 호출된 element들을 각각 호출하여 처리하도록 하는 JQuery의 메소드 이다. 메소드는 함수를 인자로 받도록 약속되어있다. 형식은 다음과 같다.

```javascript
li.map(function(index, elem){// index는 element의 index, elem은 해당 index의 엘리먼트이다
    console.log(index, elem);
    $(elem).css('color', 'red');//해당 엘리먼트의 속성을 변경한다
})
```

이외에 [jquery.org](http://jquery.org) 를 들어가면 JQuery 에 대한 API 문서가 정리되어 있다.

reference: [생활코딩opentutorials](https://opentutorials.org/course/1)