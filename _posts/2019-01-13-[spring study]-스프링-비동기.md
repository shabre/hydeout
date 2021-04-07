---
layout: post
title: "[Spring Study] 스프링 비동기"
categories: Spring-study
excerpt_separator: "<!--more-->"
---

## Spring 비동기

### CompletableFuture

Java 8에서 지원하는 비동기 프로그래밍 클래스. Concurrent 패키지에 존재하며 Future와 CompletionStage 인터페이스를 구현한다.

장점: 기존에 Spring framework에서 사용하던 ListenableFuture를 가독성 좋게 프로그래밍 가능하다.(콜백 오버라이딩에서 함수형 프로그래밍으로)


ListenalbeFuture를 이용한 구현
```java
@author jibumjung */ public class ListenableFutureTest {

Callable task = () -> { try { Thread.sleep(5 * 1000L); } catch (InterruptedException e) { e.printStackTrace(); } System.out.println("TASK completed"); return null; };
 
@Test public void listenableFuture() throws Exception { ListenableFutureTask listenableFutureTask = new ListenableFutureTask(task); listenableFutureTask.addCallback(new ListenableFutureCallback() { @Override public void onFailure(Throwable throwable) { System.out.println("exception occurred!!"); }

 
     @Override
     public void onSuccess(Object o) {
         ListenableFutureTask listenableFuture = new ListenableFutureTask(task);
         listenableFuture.addCallback(new ListenableFutureCallback() {
             @Override
             public void onFailure(Throwable throwable) {
                 System.out.println("exception occurred!!");
             }
         @Override
         public void onSuccess(Object o) {
             System.out.println("all tasks completed!!");
         }
     });
     listenableFuture.run();
 }

}); listenableFutureTask.run();


Thread.sleep(11 * 1000L); 
} 
} 
```

CompletableFuture를 이용한 구현
```java
public class completablefuture {

Runnable task = () -> { try { Thread.sleep(5 * 1000L); } catch (InterruptedException e) { e.printStackTrace(); } System.out.println("TASK completed"); };

 
@Test public void completableFuture() throws Exception {
    CompletableFuture
     .runAsync(task)
     .thenCompose(aVoid -&gt; CompletableFuture.runAsync(task))
     .thenAcceptAsync(aVoid -&gt; System.out.println("all tasks completed!!"))
     .exceptionally(throwable -&gt; {
         System.out.println("exception occurred!!");
         return null;
     });
Thread.sleep(11 * 1000L); 

} } 

```
`출처 https://www.hungrydiver.co.kr/bbs/detail/develop?id=2`

#### runAsync, supplyAsync

비동기로 실행할 task를 등록한다. runAsync 의 경우 return 값이 없을때 사용, supplyAsync의 경우 return값이 존재할 때 사용하면 된다.

CompletableFuture을 실행하는 방법엔 여러가지 종류가 있다.

1. CompletableFuture().supplyAsync().thenApply().get()

supplyAsync에서 반환된 값을 이용해 일련의 처리과정을 거치고, 처리된 값을 다시 반환한다.
```java
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<String> future = completableFuture
  .thenApply(s -> s + " World");
 
assertEquals("Hello World", future.get());
```

2. CompletableFuture.supplyAsync().thenAccept().get();
   
supplyAsync에서 반환된 값을 이용해 일련의 처리과정을 거치고, 반환을 하고싶지 않으면 사용
```java
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<Void> future = completableFuture
  .thenAccept(s -> System.out.println("Computation returned: " + s));
 
future.get();
```

3. CompletableFuture.supplyAsync().thenRun().get();

supplyAsync() 처리 후 아무런 과정을 수행하지 않을 때 사용.
```java
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<Void> future = completableFuture
  .thenRun(() -> System.out.println("Computation finished."));
 
future.get();
```