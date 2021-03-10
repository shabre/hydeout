---
layout: post
title: kotlin coroutine
categories:
    - kotlin
excerpt_separator: "<!--more-->"
---

## coroutine 이란?

co(협력) + routine(루틴) 을 합친 뜻으로 코틀린 공식 라이브러리에서 제공하는 비동기 방식의 라이브러리이다. 
기존의 자바 환경에서 친숙한 async, await 또는 future, promise 방식보다 좀더 가볍게 비동기 프로그래밍을 할 수 있도록 한다.

코루틴을 실행하는 방법은 간단하다. 비동기로 실행시키고 싶은 부분을 코루틴 scope로 선언하고 그 scope 안에 로직을 작성하면 된다.


## 코루틴 동작 방식

아래 코드는 코루틴을 간단히 동작시킨 코드이다.
GlobalScope.launch 로 선언된 코루틴이 신규 코루틴을 전역스코프에 생성되었다. delay 를 통해 지연이 발생하면 코루틴은 해당 상태를 잠시 걸어(suspend)두고 스코프를 빠져나와서 다음 명령을 수행하게 된다.

```kt
fun main() {
    GlobalScope.launch { // 전역스코프에서 신규 코루틴 생성
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    Thread.sleep(2000L)
}
```

결과값은 Hello, World! 가 될 것이다.


## Scope builder

위에서의 코드는 전역 스코프에서 선언을 진행하였다. 이번에는 전역이 아닌 구조화된 스코프를 선언해 보겠다. 아래 코루틴이 동작하는 방식을 순서대로 작성하였다.

1. runBlocking(블럭 스레드)로 하나의 코루틴 스코프가 선언되었다. 해당 스코프 내부의 코루틴들이 모두 종료될 때 까지 스레드는 block 될 것이다.
2. launch 로 코루틴이 실행되었다. 200L 의 지연이 생겨 suspend 되고 스코프 밖의 다른 명령을 수행한다.
3. coroutineScope가 신규 생성되었다. launch로 실행된 코루틴에서 500L delay를 보게 되어 suspend 되어 밖으로 나온다.
4. 100L delay 후 `Task from coroutine scope` 를 처음 출력한다.
5. 2번에서 suspend 되었던 delay가 해제가 되어 `Task from runBlocking` 가 실행된다.
6. coroutineScope 내부의 코루틴이 종료되지 않아 종료될때까지 기다린다. delay(500L) 이 끝나면 `Task from nested launch` 를 출력 후 스코프를 종료한다.
7. 마지막으로 `Coroutine scope is over` 이 출력되면서 코루틴 스코프가 종료된다.

1.
```kt
fun main() = runBlocking { // 코루틴 스코프
    launch { //코루틴 실행
        delay(200L)
        println("Task from runBlocking") //두번째 출력
    }
    
    coroutineScope { // 신규 코루틴 스코프
        launch {
            delay(500L) 
            println("Task from nested launch") //세번째 출력
        }
    
        delay(100L)
        println("Task from coroutine scope") // 첫번째 출력
    }
    
    println("Coroutine scope is over") // 마지막 출력
}
```

- runBlocking: 해당 스코프 내부는 코루틴으로 동작 하나 내부의 코루틴 동작이 모두 끝날 때 까지 스레드가 스코프에서 블럭 상태로 유지됨.
- coroutineScope: 스레드가 블럭 상태로 유지되지 않고 비동기로 로직이 수행됨. delay와 같이 block 되는 코드가 발생하면 block 이 아닌 suspend 상태로 되고 스레드를 풀게 된다.

두개는 겉으로 실행되는것을 보기엔 비슷한 것 같으나 큰 차이점이 있다. 스코프 내부를 기점으로 스레드가 block 되느냐 non-block 이 되느냐의 차이점이다. coroutineScope   내부에서 suspend이 되면 그 스레드는 잠시 해당 스코프 밖으로 빠져나와 실행 가능한 곳에서 다른 작업을 수행하게 될 것이다.

runBlocking은 코루틴 스코프가 아닌 block 환경에서 새로운 코루틴 scope 를 생성하는 용도로 사용하면 된다.

## 스코프 특징

- 자식들의 코루틴 스코프들을 모두 담당하며, 자식 코루틴의 lifetime 은 자식 스코프에 해당된다.
- 자신의 코루틴에 문제가 생겨서 종료될 시 자식 코루틴의 동작도 취소된다.
- 자식 코루틴에 대해서 책임을 진다. 자식 코루틴들이 모두 종료되어야 자신도 종료된다.

## launch vs async

두개 모두 코루틴을 시작할때 사용하는 함수이다. 차이점은 크게 아래와 같다.

- launch: 결과를 return 하지 않고 job을 반환한다.
- async: 결과를 return 한다(Deffered\<T\> 타입)