---
layout: post
title: "[spring] Annotation"
categories:
    - Development
excerpt_separator: "<!--more-->"
---

# Annotation 정리


## @(Annotation) 이란?

어노테이션이란 자바 소스 코드에 추가될 수 있는 Syntatic metadata 이다. 주로 클래스나 메소드 위에 @(annotation) 과 같이 추가될 수 있다.

```java
public class Animal {
    public void speak() {
    }

    public String getType() {
        return "Generic animal";
    }
}

public class Cat extends Animal {
    @Override
    public void speak() { // This is a good override.
        System.out.println("Meow.");
    }

    @Override
    public String gettype() { // Compile-time error due to mistyped name.
        return "Cat";
    }
}
```

위의 소스코드에선 @Override 란 어노테이션이 적용되었는데, 이것을 통해 해당 메소드가 제대로 오버라이딩이 되었는지를 컴파일 시간에 알 수 있다.

어노테이션은 여러 종류가 있으며, 사용자가 직접 어노테이션을 Custom하게 선언하여 사용할 수도 있다.

어노테이션을 사용함으로 어떠한 기능을 명시적으로 보여줄 수 있고, 기능을 추가할 수 있는 장점이 있다.

## Spring 에서의 주요 Annotation

Spring에서 자주 사용하는 Annotation은 다음과 같다.

### @Require

@Require 메소드는 등록된 빈을 주입시키는데 사용된다. name 파라미터에 빈 이름을 입력하면 빈이 주입된다.
```java
@Require(name = "myBean")
public void getBean
```
만약 빈이 없을 경우 BeanInitializationException이 발생된다.

### @Autowired

bean으로 등록된 메소드의 의존성을 주입하는 어노테이션이다. 3가지 방법으로 주입이 가능하다.
- 필드에서의 주입
- 생성자로서의 주입
- setter 메소드로서의 주입
Bean으로 등록된 Person 오브젝트를 주입하는것을 예로 보겠다.
```java
//필드에서의 주입
public class Customer {
    @Autowired                               
    private Person person;                   
    private int type;
}
```
```java
//생성자로 주입
@Component
public class Customer {
    private Person person;
    @Autowired
    public Customer (Person person) {					
      this.person=person;
    }
}
```

```java
//setter 메소드로 주입
@Component
public class Customer {
    private Person person;
    @Autowired
    public Customer (Person person) {					
      this.person=person;
    }
}
```

### @Qualifier
Qualifier는 주로 Autowired와 같이 사용된다. 같은 타입의 메소드가 다른 이름으로 여러개 Bean으로 등록됬을 시 또는 인터페이스가 구현체를 주입 받을 시 사용된다.
```java
@Component
public class BeanB1 implements BeanInterface {
  //구현체1
}
@Component
public class BeanB2 implements BeanInterface {
  //구현체2
}
```
```java
@Component
public class BeanA {
  @Autowired
  @Qualifier("beanB2") //구현체 2로 주입
  private BeanInterface dependency;
  ...
}
```

### @Configuration
Configuration 어노테이션은 Bean을 define하는 클래스일 때 클래스 맨 위에 붙여준다.

### @ComponentScan
@Configuration과 사용되며 Spring에게 annotation 컴포넌트를 스캔하라고 명령한다. @ComponentScan은 basePackages 속성을 통해 기본 패키지를 설정 가능하다. @ComponentScan을 통해 컴포넌트를 스캔하면 베이스 패키지에 포함되어 있는 모든 컴포넌트들을 찾게되나, 포함되지 않은것은 찾지 않는다.

### @Bean
Method 레벨에서 Spring bean을 등록하는 어노테이션이다. @Configuration 과 같이 사용된다.
```java
@Configuration
public class AppConfig{
  @Bean
  public Person person(){
    return new Person(address());
  }
  @Bean
  public Address address(){
    return new Address();
  }
}
```

### @Lazy
보통 Bean은 스프링이 시작할 때 생성된다. Lazy 어노테이션을 추가하면 first request 타임에 bean이 생성되게 된다. Bean 단위가 아니라 Configuration 에 선언할 경우 모든 bean이 늦게 initialize 된다.

### @Value
Bean의 속성에 값을 입력할때 사용될 수 있다. 주로 properties의 속성값을 bean에 입력할 때 사용된다.

## Bean 등록 어노테이션

### @Component
Spring component로 등록하는 annotation으로, 클래스를 Bean으로 등록할때 사용된다.

### @Controller
MVC 모델의 Controller임을 명시하는 어노테이션이다. component와 동일하게 Bean으로 등록된다. 

### @Service
Business logic을 수행하는 자바 클래스를 명시하는 어노테이션이다. Component와 동일하게 Bean으로 등록된다.

### @Repository
Database에 직접적으로 연결되는 클래스에 선언하는 어노테이션이다. DAO(Data access object) 클래스에 선언된다고 생각하면 된다. Repository 어노테이션 안에는 Exception 핸들러가 존재해 try catch 문이 필요하지 않다.


## Rest 관련 어노테이션

### @RequestMapping
Controller에서 요청받을 주소를 설정하는 어노테이션이다. method 설정을 통헤 GET, POST 와 같은 Request 메소드 타입을 설정할 수 있다. 
```java
@Controller
@RequestMapping("/welcome")
public class WelcomeController{
  @RequestMapping(method = RequestMethod.GET)
  public String welcomeAll(){
    return "welcome all";
  }	
}
```
RequestMapping은 메소드 설정을 해주지 않으면 보안상 취약할 수 있다. RequestMapping 대신 `@PostMapping`, `@GetMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`을 사용할 수 있다.

### @CookieValue
RequestMapping 메소드의 파라미터에서 선언할 시 쿠기값을 얻어올 수 있다.
```java
@RequestMapping("/cookieValue")
  public void getCookieValue(@CookieValue "JSESSIONID" String cookie){
}
```

### @CrossOrigin
Cross origin 문제를 해결해주는 어노테이션. Cross origin 이란 도메인과 다른 도메인으로부터 리소스가 요청될 경우를 말한다.

- - -

### @ExceptionHandler
Controller 레벨에서 처리될 예외를 처리하도록 하는 어노테이션으로, 메소드에 선언된다. 예외 값들은 배열로도 넣을 수 있으며, ExceptionHandler와 일치하는 예외가 발생 시 view 단으로 보내서 처리가 가능하다.

### @PathVariable
PathVariable은 RequestMapping에 동적으로 들어오는 값을 파라미터로 받아서 메소드에서 처리하도록 하는 어노테이션이다.
```java
@RequestMapping("/main/{id}")
public String page(@PathVariable String id){
//logic
}
```