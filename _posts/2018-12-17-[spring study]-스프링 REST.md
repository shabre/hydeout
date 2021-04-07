---
layout: post
title: "[Spring Study] 스프링 REST"
categories: Spring-study
excerpt_separator: "<!--more-->"
---

## REST
REST란 REpresentational State Transfer 의 약자.
- 하이퍼미디어 기반의 소프트웨어 아키텍쳐 형식(사상)
- HTTP 프로토콜 상에서 URI 주소와 CRUD 조작을 통해 서비스를 제공받는 아키텍쳐 개념
- XML나 HTTP같은 간단한 웹 인터페이스로 통신

#### RESTFul 작성 원칙
- Client-Server: 
REST서버는 API를 제공합니다.
하나 혹은 여러 클라이언트에서 하나의 REST API를 이용할 수 있습니다.
REST 서버에서 비즈니스 로직 처리 및 저장을 책임집니다.
클라이언트의 경우 사용자 인증이나 컨택스트(세션, 로그인 정보) 등을 직접 관리하고 책임지는 구조로 REST 서버와 역할이 나뉘어 지고 있습니다.

- Stateless:
무상태성이라고도 합니다.
상태 정보를 저장하지 않고 각 API서버는 들어오는 요청만을 들어오는 메시지로만 처리하면 됩니다.
세션과 같은 컨텍스트 정보를 신경쓸 필요가 없기 때문에 구현이 단순해집니다.

- Cache: 
캐쉬를 사용할 수 있습니다..
HTTP 프로토콜 표준에서 사용하는 Last-Modified태그나 E-Tag를 이용하면 구현할 수 있습니다.
캐쉬를 사용하게 되면 응답시간 뿐 아니라 REST 서버 트랜잭션이 발생하지 않기 때문에 전체 응답시간, 성능, 서버의 자원 사용률을 향상 시킬 수 있습니다.

- Uniform Interface: 
REST는 HTTP 표준에만 따른다면, 어떤 기술이라던지 사용이 가능한 인터페이스 스타일입니다.


- Layered System: 
클라이언트는 직접 최종서버에 붙었는지 등을 알수가 없습니다.
intermediary 서버 등을 통해서 로드밸런싱/공유 캐시등을 통해 확장성과 보안성을 향상 가능 합니다.

- Code-On-Demand: 
서버로 부터 스크립트를 받아서 Client에서 실행하는 것입니다.
반드시 충족할 필요는 없습니다.


출처: https://blog.sonim1.com/105 [Kendrick's Blog]

- - -

## Spring REST

스프링에서도 REST를 구현하기 위한 기능을 제공한다. 기능 설명 전 필요한 용어를 간단히 정리해 보았다.

### Marshalling 이란?
객체의 메모리에서의 표현방식을 저장 또는 전송에 적합한 다른 데이터 형식으로 변환하는 과정이다.

### Serialization 이란?
객체의 상태를 저장하기 위해서 객체를 byte stream 으로 변환하는 것을 의미한다.

### Marshalling 과 Serialization의 차이점

두개는 서로 비슷하나 자바에서 차이점이 있다.
자바는 marshalling에 코드베이스(코드 위치, 상태 정보)를 기록하게 된다.

스프링 REST에서 받은 데이터를 간단히 처리하기 위해 마샬러를 통한 마샬링 과정을 거치게 된다. 아래는 그 기능과 과정을 구현한 간단한 예제이다.

### 마샬링 과정

1. 컨트롤러
```java
@Controller
public class RestMemberController {
    ...
    @RequestMapping("/members")
    public String getRestMembers(Model model) {
        // Return view membertemplate. Via resolver the view
        // will be mapped to a JAXB Marshaller bound to the Member class

        Members members = new Members();
        members.addMembers(memberService.findAll());
        model.addAttribute("members", members);
        return "membertemplate";
    }
}
```
2. Bean name view resolver 선언. Member class를 선언된 마샬러로 보냄.
```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.apress.springrecipes.court")
public class CourtRestConfiguration {
    @Bean
    public View membertemplate() {
        return new MarshallingView(jaxb2Marshaller());
    }
    @Bean
    public Marshaller jaxb2Marshaller() {
        Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
        marshaller.setClassesToBeBound(Members.class, Member.class);
        marshaller.setMarshallerProperties(Collections.singletonMap(javax.xml.bind.Marshaller.JAXB_FORMATTED_OUTPUT, Boolean.TRUE));
        return marshaller;
    }
    @Bean
    public ViewResolver viewResolver() {
        return new BeanNameViewResolver();
    }
}
```

3. Member, Members 클래스
```java
@Data
@XmlRootElement
public class Member {
    private String name;
    private String phone;
    private String email;
}
@XmlRootElement
@XmlAccessorType(XmlAccessType.FIELD)
public class Members {
    @XmlElement(name="member")
    private List<Member> members = new ArrayList<>();

    public void addMembers(Collection<Member> members) {
        this.members.addAll(members);
    }
}
```
위 과정을 거치면 members url 을 호출하면 Members 객체 값을 XML 페이로드로 변경 후 클라이언트에게 반환한다.

**주의점: 콘텐트 협상이 없을 경우, 확장자는 반드시 .xml을 붙여야 정상적으로 xml 마샬링이 처리가 된다**

- - -
### @ResponseBody 사용하기
@ResponseBody 어노테이션을 사용하여 객체를 반환하면 위의 마샬링이 수행해준 방식을 동일하게 수행해준다.

위의 긴 코드를 아래의 @ResponseBody 어노테이션 하나를 추가하면 동일하게 처리해준다.
```java
@Controller
public class RestMemberController {
    @RequestMapping("/members")
    @ResponseBody
    public Members getRestMembers() {
        Members members = new Members();
        members.addMembers(memberService.findAll());
        return members;
    }
}
```

### @PathVariable 사용하기
pathvariable로 특정 path의 값을 변수로 받아와 특정 값의 정보만 호출하는 rest api를 구성이 가능하다.
```java
@Controller
public class RestMemberController {
    @RequestMapping("/member/{memberid}")
    @ResponseBody
    public Member getMember(@PathVariable("memberid") long memberID) {
        return memberService.find(memberID);
    }
}
```

### @InitBinder
InitBinder를 이용하면 특정 Path 변수 값을 알맞는 오브젝트로 매핑을 할 수 있다.
```java
@InitBinder
public void initBinder(WebDataBinder binder) {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
}

@RequestMapping("/reservations/{date}")
public void getReservation(@PathVariable("date") Date date) {...}
```

- - -
### RPC
RPC 란? Remote Procedure call 의 약자로 원격지에 위치한 프로그램을 로컬에서 동작시키는것저럼 수행하는 동작. 네트워크에 대한 세부사항을 몰라도 함수를 실행해 사용자는 로직에만 신경쓸수있도록 함. 함수적 호출을 이용한 분산 프로그래밍.

### SOAP
Simple Object Aceess Protocol 로 XML기반의 문서를 http, https, smtp 등을 통해 통신하는 기법. RPC 이용한 프로토콜 기법.
정형화된 틀에 데이터를 전송해서 여러 플랫폼에서 연동하기 쉬운 장점이 있으나 REST에 비해 상대적으로 많은 데이터를 전송해야하고, SOAP 을 기본적으로 이해하고 있어야하는 고난이도 프로그래밍이 들어가야하는 단점이 존재.