---
layout: post
title: "[Spring Study] 스프링 코어"
categories:
    - Spring Study
excerpt_separator: "<!--more-->"
---

# 스프링 코어

## IoC와 POJO에 관해
---

### IoC 란
Inversion of Control 의 약자로 제어의 역전이란 뜻. 프로그램이 수동적으로 실행되는것이 아닌 능동적으로 런타임에 제어를 통해 실행됨. 라이브러리와 프레임워크를 구분짓는 가장 결정적 요소이자 스프링의 핵심요소.

### POJO란?

Plain Old Java Object 의 줄임말. 그야말로 오래된 단순 자바 객체이다.

스프링에서는 이 POJO를 스프링 Bean에서 받아와 사용하게 된다. 이때 IoC 컨테이너는 등록된 POJO빈을 주는 창구 역할을 한다. 순서는 다음과 같다.

1. POJO 생성.
2. POJO Bean을 Generating 하는 Config 생성.
3. app-context에 config 등록.
4. context에서 bean을 가져온다.

그림으로 나타내면 대략 이렇다.

![POJO_IoC]({{ site.baseurl }}/assets/image/chap2_1.png)

아래는 코드이다.

```java
/** 
* POJO
*/
public class SequenceGenerator {

    private String prefix;
    private String suffix;
    private int initial;
    private final AtomicInteger counter = new AtomicInteger();

    public SequenceGenerator() {
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }

    public void setInitial(int initial) {
        this.initial = initial;
    }

    public String getSequence() {
        String builder = prefix +
                initial +
                counter.getAndIncrement() +
                suffix;
        return builder;
    }
}
```

```java
/** 
* POJO 생성을 bean으로 등록
*/
@Configuration
public class SequenceGeneratorConfiguration {

    @Bean
    public SequenceGenerator sequenceGenerator() {

        SequenceGenerator seqgen = new SequenceGenerator();
        seqgen.setPrefix("30");
        seqgen.setSuffix("A");
        seqgen.setInitial(100000);
        return seqgen;
    }
}
```

```java
/** 
* POJO 생성을 bean으로 등록
*/
@Configuration
public class SequenceGeneratorConfiguration {

    @Bean
    public SequenceGenerator sequenceGenerator() {

        SequenceGenerator seqgen = new SequenceGenerator();
        seqgen.setPrefix("30");
        seqgen.setSuffix("A");
        seqgen.setInitial(100000);
        return seqgen;
    }
}
```

```java
public class Main {

    public static void main(String[] args) {
        ApplicationContext context =
                new AnnotationConfigApplicationContext(SequenceGeneratorConfiguration.class);
                //ApplicationContext가 IoC 창구가 된다

        SequenceGenerator generator = context.getBean(SequenceGenerator.class);
    }
}
```

## Aspect 지향 프로그래밍
---

### AOP
Aspect Oriented Programming이란 뜻이다. 어떤 기능을 수행할때 전처리, 후처리 템플릿을 생각하면 편리하다. 트랜잭션이 대표적인 예.

### Aspect
어디에서(PointCut) + 무엇을 할 것인지(Advice) 를 합쳐놓은 개념

Advice 종류
- @Before : 실행 이전
- @After : 실행 이후
- @AfterReturning : 조인포인트가 값을 반환할 경우
- @AfterThrowing : 조인포인트가 예외를 던질 경우
- @Around : 모든 조건을 포함

@Around는 가장 넓은 범위를 포함할 수 있지만 최소한의 요건을 충족하면서도 가장 기능이 약한 어드바이스를 쓰는것이 바람직하다.

예제코드

```java
@Component("arithmeticCalculator")
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {
    @Override
    public double add(double a, double b) {
       ...
    }

    @Override
    public double sub(double a, double b) {
        ...
    }
}
```

```java
@Aspect
@Component
public class CalculatorLoggingAspect {

    private Logger log = LoggerFactory.getLogger(this.getClass());

    @Before("execution(* *.*(..))") //포인트컷 - 모든 실행조건
    public void logBefore(JoinPoint joinPoint) {
    }

}
```

```java
@Configuration
@EnableAspectJAutoProxy //Aspect 검색을 가능하게 한다
@ComponentScan
public class CalculatorConfiguration {
}
```

```java
public class Main {

    public static void main(String[] args) {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(CalculatorConfiguration.class);

        ArithmeticCalculator arithmeticCalculator =
                context.getBean("arithmeticCalculator", ArithmeticCalculator.class);
        arithmeticCalculator.add(1, 2);//Advice실행
        arithmeticCalculator.sub(4, 3);//Advice실행
    }
}

```

### Pointcut 재사용

```java

@Aspect
@Component
public class CalculatorLoggingAspect {

    private Log log = LogFactory.getLog(this.getClass());

    @Pointcut("execution(* *.*(..))")
    private void loggingOperation() {
    }

    @Before("CalculatorLoggingAspect.loggingOperation()")
    public void logBefore(JoinPoint joinPoint) {
        ...
    }

    @After("CalculatorLoggingAspect.loggingOperation()")
    public void logAfter(JoinPoint joinPoint) {
        ...
    }

    @AfterReturning(
            pointcut = "CalculatorLoggingAspect.loggingOperation()",
            returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        ...
    }

    @AfterThrowing(
            pointcut = "CalculatorLoggingAspect.loggingOperation()",
            throwing = "e")
    public void logAfterThrowing(JoinPoint joinPoint, IllegalArgumentException e){
        ...
    }

    @Around("CalculatorLoggingAspect.loggingOperation()")
    public Object logAround(ProceedingJoinPoint joinPoint) {
        ...
    }
}
```

### @어노태이션을 이용한 Aspect

1. 어노테이션 선언
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LoggingRequired {
}
```
2. Pointcut 등록
```java
@Aspect
public class CalculatorPointcuts {
    @Pointcut("annotation(com.apress.springrecipes.calculator.LoggingRequired)")
    public void loggingOperation() {
    }
}
```
어노테이션을 이용하면 좀더 깔끔한 aop를 구성 가능하다.


#### 참조
스프링 레시피5 4판