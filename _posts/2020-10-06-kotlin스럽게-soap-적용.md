---
layout: post
title: kotlin 스럽게 soap 통신 적용기
categories:
    - Development
excerpt_separator: "<!--more-->"
---

# Kotlin 에서 soap 통신

자바 코드를 코틀린 코드로 전환하는 업무를 진행 중에 아주 신기한 통신방식을 발견했습니다. 바로 SOAP 이란 방식인데요, 저랑 비슷한 주니어 개발자들은 이 통신방식을 접해본 경험이 없지 않을까 싶습니다. 저도 들어만 봤지 실제로 보게 된 건 이번이 처음이었네요!

이제는 잘 사용하지 않는 SOAP 을 비교적 현대 언어라 불릴 수 있는 코틀린에 자연스럽게 녹일 수 있는 방법에 대해 고민한 과정을 나누고자 합니다.

### 목차

1. SOAP 이란?
2. 자바에서 SOAP 통신방법
3. 자바에서 코틀린으로 옮겨왔을 때의 문제점
4. ADAPTER 클래스로 코틀린스럽게 해결



## 1. SOAP 이란?

Simple Object Access Protocol의 약자인 SOAP은 플랫폼에 의존적이지 않게 네트워크 통신을 통해 데이터를 주고받을 수 있도록 xml 기반으로 제작된 프로토콜입니다. HTTP 바디에 xml 형식의 데이터를 넣어 주고받는 형식인데요 프로토콜까지 정의했으면서 왜 요즘 쓰지 않을까요?

복잡하고 무겁습니다. 아래 간단하게 SOAP 요청을 통한 HTTP 응답 예제를 보겠습니다.
```http
HTTP/1.1 200 OK
Content-Type: text/xml; charset="utf-8"
Content-Length: nnnn

<SOAP-ENV:Envelope
  xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
  SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"/>
   <SOAP-ENV:Body>
       <m:GetLastTradePriceResponse xmlns:m="Some-URI">
           <Price>34.5</Price>
       </m:GetLastTradePriceResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```
네 34.5 데이터 하나 받으려고 저렇게 많은 xml들이 생성되었습니다. 게다가 저런 통신을 하라면 위와 같은 xml에 데이터들을 serialize/deserialze 해줘야 합니다. 가슴이 먹먹해집니다!!

## 2. 자바에서 SOAP 통신방법

다행히 serialize/deserialize를 쉽게 해주는 라이브러리가 있습니다. API 제공 업체에서는 SOAP 통신을 쉽게 해주도록 문서를 WSDL(Web Services Description Language. Web Service가 제공하는 서비스에 대한 정보를 기술하기 위한 XML 기반의 마크업 언어) 이라는 형식으로 제공을 해주는데요, [wsimport](https://docs.oracle.com/javase/7/docs/technotes/tools/share/wsimport.html) 를 사용하면 자바 클래스들을 생성해 줍니다.

예로 `wsimport -keep -verbose -p com.ncp.api.order.model.naver.wsdl ./MallService5.wsdl`  명령어를 입력만 하면  com.ncp.api.order.model.naver.wsdl 패키지 안에 MallService5.wsdl에 저술된 xml 데이터를 자바 파일로 만들어 줍니다. 무려 JDK에서 제공하는 기능이라 별도 라이브러리를 설치할 필요도 없고요!


```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "GetCustomerInquiryListResponseType", propOrder = {
    "customerInquiryList",
    "returnedDataCount",
    "hasMoreData"
})
public class GetCustomerInquiryListResponseType
    extends BaseCheckoutResponseType
{

    @XmlElement(name = "CustomerInquiryList", namespace = "")
    protected CustomerInquiryList customerInquiryList;
    @XmlElement(name = "ReturnedDataCount", namespace = "")
    protected Integer returnedDataCount;
    @XmlElement(name = "HasMoreData", namespace = "")
    protected Boolean hasMoreData;
    
    ...

```
생성된 자바 파일들을 보면 JAXB 라이브러리에 대한 의존성을 갖습니다. 프로젝트에 사용하는 빌드 툴(maven, gradle...)에 JAXB를 추가해 주시면 끝입니다.

## 3. 자바에서 코틀린으로 옮겨왔을 때의 문제점

문제는 코틀린으로 옮겨올 때 시작되었습니다. 새로운 마음으로 시작하는 코틀린 프로젝트에 자바 코드가 남아있는 걸 볼 수 없어서 저는 XML 바인딩 클래스들을 코틀린 코드로 옮기는 작업을 시작합니다. 한 개의 클래스를 코틀린 코드로 수정하고 실행을 해보았습니다. 그런데 콘솔 로그에 이런 메시지가 올라옵니다. `default constructor가 없다. default constructor가 있어야 객체 만들 수 있다!` 아.. 이 라이브러리는 기본 생성자로 객체를 생성 후에 setter로 데이터를 주입해주나 봅니다. 그런데... 코틀린에서는 기본 생성자를 제공하지 않습니다!

### 데이터 클래스를 코틀린으로 표현할때의 강력함

코틀린에서는 데이터만을 가지는 클래스를 위해 data class라는 특수한 클래스를 제공합니다. 또한 코틀린에서는 자바와는 다르게 생성자에서 변수를 선언할 수 있도록 하는 강력한 기능을 제공하는데요, 자바와는 다르게 생성자 파라미터 값을 필드값에 매핑해줄 필요도 없고 클래스의 프로퍼티와 기능을 분리하여 가독성을 좋게 해줍니다.

예를들어 아래 자바 클래스를 코틀린으로 표현하게되면
```java
public class User{
    @Getter
    private final String name;
    @Getter
    private final int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
아래와 같이 간단하게 정의될 수 있습니다.
```kotlin
data class User(val name: String, val age: Int)
```

### 그렇다면 무엇이 문제인가?

문제는 코틀린에서는 디폴트 생성자를 제공하지 않는다는 점입니다. 물론 여러 가지 방법을 통해 디폴트 생성자처럼 행동하는 생성자를 만들 순 있으나 기본적으로 지원하지 않는 기능을 비슷하게 구현하는 것은 코틀린스럽지 않은 표현법이라 할 수 있습니다.

제 경우가 기본으로 제공하지 않는 디폴트 생성자를 만들고 인스턴트 생성 이후 시점에 값을 세팅할 수 있는 setter 메서드를 제공해야 했습니다. 이를 코틀린으로 표현하면 아래와 같습니다.

```kotlin
data class User(
    var name: String? = null, 
    var age: Int? = null
)
```
처음 정의한 데이터 클래스와 달라진 점을 나열하면
- val(immutable) -> var (mutable)
- 각 프로퍼티들을 `?`(nullable)로
- null 을 디폴트 값으로 지정

이렇게 됩니다. 보통 data class 들의 프로퍼티는 val 로 선언을 하여 불변성을 깔끔하게 보장해 주는데 var로 선언하니 몸이 불편해지기 시작합니다. 게다가 코틀린의 강력한 기능 중 하나인 null 미 허용 기능을 못쓴다니요! 이거 코틀린으로 옮기는 의미가 있는지 의문이 들기 시작합니다. 노가다는 노가다대로 하고 결과물은 코틀린과 어울리지 않는 null 허용과 default null 이 덕지덕지 붙어있는 결과물이 예상이 됩니다.

## 4. ADAPTER 클래스로 코틀린스럽게 해결

위의 과정까지 겪고 난 저는 그냥 항복하고 자바 클래스를 사용하고 편해질지?(전환할 클래스가 너무나 많았습니다) 고민을 하게 됩니다. 껍데기만 코틀린이지 자바랑 다를 바가 없는 코드를 만드는 것은 정말로 비효율적일 것이란 생각이 들었습니다.

접근 방법이 잘못되었다고 생각되어 혹시 다른 방법이 없나 생각하던 도중 예전에 스프링 스터디를 진행하며 보았던 adapter 클래스가 생각이 났습니다. Spring framework configuration을 정의하기 위해 interface를 구현하게 될 경우 수많은 기능들을 모두 구현해야 하는데, adapter 클래스는 interface에 대한 기본 구현체는 제공해 주고 내가 필요한 설정만 설정할 수 있도록 해주는 중간다리 역할을 해주는 클래스입니다. 참 고마운 클래스입니다. 여기에 영감을 받아 자바 코드를 코틀린으로 전환하는 것이 아니라 코틀린<->자바 변환이 가능한 중간다리(adapter) 클래스를 생성하기로 결정합니다.


### 작업 결과
- - -

### 자바 클래스
```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "getProductOrderIdListRequest", propOrder = {"orderID", "mallID"})
public class GetProductOrderIdListRequest extends BaseCheckoutRequestType {

    @XmlElement(name = "OrderID", required = true)
    protected String orderID;
    @XmlElement(name = "MallID")
    protected String mallID;

    //getter, setter ...
}
```
### 코틀린 클래스
```kotlin
data class GetProductOrderIdListRequestAdapter(
    val orderID: String,
    val mallID: String? = null,

    override val requestID: String? = null
) : BaseCheckoutRequestAdapter(
    requestID,
    version = Constant.getVersion(ApiType.GET_PRODUCT_ORDER_ID_LIST)
) {
    fun toJavaInstance(
        properties: Properties,
        timestamp: String
    ): GetProductOrderIdListRequest {
        return GetProductOrderIdListRequest().also {
            it.orderID = orderID
            it.mallID = mallID

            it.accessCredentials = AccessCredentialsUtil.toMallServiceJavaInstance(
                properties,
                ApiType.GET_PRODUCT_ORDER_ID_LIST,
                timestamp
            )
            it.requestID = requestID
            it.detailLevel = detailLevel
            it.version = version
        }
    }
}
```

작업 결과 코드만 보면 코틀린 클래스가 뭔가 코드도 길고 복잡해 보입니다. 하지만 이 코드들은 기존 자바 코드의 서비스 로직에 존재하던 부분을 도메인 영역으로 가져오게 된 것입니다. 자바 서비스 로직에서의 복잡한 값 세팅 로직들이 코틀린 서비스 로직에서는 .toJavaInstance() 호출 한 번이면 자바 클래스로 바꿔줄 수 있는 것이죠. 이제 코틀린의 장점을 살릴 수 있게 되었고, 자바 코드는 데이터 xml mapper 클래스로만 사용되어 다른 로직적인 부분에 신경을 쓰지 않아도 됩니다.

이렇게 함으로써 외부 서비스와의 의존성을 덜 수 있는 추가적인 장점이 있습니다. wsdl로 생성된 자바 클래스는 어디까지나 외부 서비스에 의존적인 클래스입니다. 만약 서비스 제공 업체가 이 버전에 대해 수정을 하게 될 경우 자바 클래스도 그에 맞게 수정이 되어야 할 텐데요, 자바 클래스 내부를 수정했거나 서비스 로직에서 데이터모델의 값을 바꾸는 작업을 했다면 서비스 로직을 일일이 뒤져보아야 할 것입니다. 위와 같이 코틀린 클래스를 adapter로 두면 해당 클래스만 살짝 수정하면 해결될 것으로 보입니다!

## 마무리

특별한 기술에 대해 소개하는 글은 아니었지만 자바 -> 코틀린 전환 작업을 진행하며 맞닥뜨린 레거시 기술을 코틀린스럽게 해결을 위해 고민한 과정을 정리해 보았습니다.

자바 -> 코틀린 전환 작업을 진행 중 저와 비슷하게 자바 코드를 부득이하게 살려두어야 하지만 코틀린의 장점 버리고 싶지 않을 때 위와 같은 방법을 사용해 보는것도 좋은 방법인 것 같습니다. 저 개인적으로도 이번에 작업을 진행하면서 무작정 전환하는 게 능사가 아니라는 점을 깨닫게 된 좋은 경험이 되었습니다.