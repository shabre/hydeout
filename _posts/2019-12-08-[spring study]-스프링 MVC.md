---
layout: post
title: "[Spring Study] 스프링 MVC"
categories:
    - Spring Study
excerpt_separator: "<!--more-->"
---

# Spring MVC

### servlet
```
server + let???
server + applet??
```
서버에서 실행되는 프로그램

### HTTP Servlet 동작방식

![POJO_IoC]({{ site.baseurl }}/assets/image/httpservlet.PNG)

1. 사용자(클라이언트)가 URL을 클릭하면 HTTP Request를 Servlet Conatiner로 전송합니다.
2. HTTP Request를 전송받은 Servlet Container는 HttpServletRequest, HttpServletResponse 두 객체를 생성합니다.
3. web.xml은 사용자가 요청한 URL을 분석하여 어느 서블릿에 대해 요청을 한 것인지 찾습니다.
4. 해당 서블릿에서 service메소드를 호출한 후 클리아언트의 POST, GET여부에 따라 doGet() 또는 doPost()를 호출합니다 (HTTP servlet).
5. doGet() or doPost() 메소드는 동적 페이지를 생성한 후 HttpServletResponse객체에 응답을 보냅니다.
6. 응답이 끝나면 HttpServletRequest, HttpServletResponse 두 객체를 소멸시킵니다.

> http://mangkyu.tistory.com/14

#### Model attribute HttpservletRequest attribute 차이점?
```
The difference is, that Model is an abstraction. You could use Spring with servlets, portlets or other frontend technologies and Model attributes will always be available in your respective views.

HttpServletRequest on the other hand is an object specific for Servlets. Spring will also make request attributes available in your views, just like model attributes, so from a user perspective there is not much difference.

Another aspect is that models are more lightweight and more convenient to work with (e.g iterating over all attributes in a model map is easier than in a request).
-stackoverflow 답변 중
```
- - -
### DispatcherServlet

```
요청된 HTTP request 를 처리해주는 중앙dispatcher로, 등록된 controller/hanler로 dispatch 해주는 역할을 함.
```
#### DispatcherServlet 특징
- Java bean configuration 매커니즘을 기초로 함.

- 미리 만들어졌거나 application의 일종으로 request를 Handler 오브젝트에 연결해주는 어떤 HandlerMapping 구현체라도 사용가능하다. Default는 BeanNameUrlHandlerMapping and RequestMappingHandlerMapping 이다. HandlerMapping 오브젝트는 서블릿의 application context에 빈으로 등록이 가능하다(HandlerMapping을 오버라이딩 가능하면). 

- 어떤 HandlerAdapter든 사용이 가능하다. Default는 HttpRequestHandlerAdapter, SimpleControllerHandlerAdapter. HandlerAdapter는 오버라이딩 형식으로 application context 에 빈으로 등록이 가능하다.

- dispathcer의 예외 처리는 등록된 HandlerExceptionResolver로 가능하다.

- View 처리는 ViewResolver 구현체로부터 가능하다. Default는 InternalResourceViewResolver. application context에 오버라이딩 형식으로 빈으로 등록이 가능하다. 만약 view 가 사용자로부터 정의되지 않았다면 설정된 RequestToViewNameTranslator 가 현재의 요청을 view name으로 해석할 것이다. bean 이름은 "viewNameTranslator" 이고 default 클래스는 DefaultRequestToViewNameTranslator.
```java
@Configuration
@ComponentScan("com.apress.springrecipes.court")
public class CourtConfiguration {

    @Bean
    public InternalResourceViewResolver internalResourceViewResolver() {

        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

}
```

- multipart request(ex> 파일업로드) 에 대한 구현은 MultipartResolver 로 가능하다. pache Commons FileUpload 와 Servlet 3 가 포함되어있다. 가장 많이 선택되는 resolver는 CommonsMultipartResolver이다. bean name은 "multipartResolver"이고 default값은 없다.

- LocaleResolver로 다국어처리가 가능하다.

- Themeresolver 구현체로 고정된 theme, cookie, session 설정이 가능하다.
> [springdocs 번역](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)

```java
/**
* DispatcherSerlvet 등록 예제
*/
public class CourtServletContainerInitializer implements ServletContainerInitializer {

    public static final String MSG = "Starting Court Web Application";

    @Override
    public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException {

        System.out.println(MSG);

        ctx.log(MSG);

        AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext();
        applicationContext.register(CourtConfiguration.class);

        // DispatcherServlet 등록
        DispatcherServlet dispatcherServlet = new DispatcherServlet(applicationContext);

        ServletRegistration.Dynamic courtRegistration = ctx.addServlet("court", dispatcherServlet);
        courtRegistration.setLoadOnStartup(1);
        courtRegistration.addMapping("/");
    }
}
```

- - -
### 웹 어플리케이션 배포하기

배포된 war 파일을 시동하려면 META-INF/service/javax.servlet.ServletContainerInitializer를 파일을 함께 만들어야한다. 이 작업은 스프링에서 제공하는 SpringServletContainerInitializer를 이용하면 간단하게 해결된다.
SpringServletContainerInitializer 는 ServletContainerInitializer 의 구현체로 클래스패스에서 WebApplicationInitializer 구현체를 찾게된다. WebApplicationInitializer 구현체는 스프링에서 몇가지가 준비되어있고, AbstractAnnotationConfigDispatcherServletInitializer는 그 중 하나이다.
```java
public class CourtApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[]{ServiceConfiguration.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[]{WebConfiguration.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/", "/welcome"};
    }
}
```

- - -
### 핸들러 인터셉터
핸들러 인터셉터로 서블릿 처리의 전처리, 후처리를 수행할 수 있다.

```java

```

#### 참조
스프링 레시피5 4판