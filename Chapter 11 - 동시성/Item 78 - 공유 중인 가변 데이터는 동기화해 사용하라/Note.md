# 공유 중인 가변 데이터는 동기화해 사용하라

> 작성자: 럭키

## Synchronized
`synchronized` 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다. 많은 프로그래머가 동기화를 **배타적 실행** 용도로만 생각한다. 

**배타적 실행**이란, 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 것을 의미한다.

자바 언어의 명세상으로 long과 double 를 제외한 변수를 읽고 쓰는 것은 원자적이다. 즉, 동기화 없이 여러 스레드가 같은 변수를 수정하더라도 항상 어떤 스레드가 정상적으로 저장한 값을 읽어오는 것을 보장한다는 것이다.

그러나 동기화에는 중요한 기능이 하나 더 있다. 자바 언어 명세상 long과 double 를 제외한 변수를 읽고 쓰는 것은 원자적이다.

즉, 동기화 없이 여러 스레드가 같은 변수를 수정하더라도 항상 어떤 스레드가 정상적으로 저장한 값을 읽어오는 것을 보장한다. 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장한다, 그러 **한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다**. 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.

따라서 원자적 데이터를 쓸 때도 동기화해야 한다. 동기화는 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

**동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.**

싱글 스레드 기반 프로그램이라면 동기화를 고려하지 않아도 되지만 멀티 스레드 기반이라면 객체를 공유할 때 동기화를 고민해야 한다.

이는 한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 때문이다.

동기화가 잘못 되었을 때는 어떤 일이 발생하는지 코드로 살펴보자. 아래 코드는 얼마나 오랫동안 실행될까?

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

스레드가 `start` 되고 1초 동안의 sleep이 끝나면 boolean 변수의 값이 true가 되어 루프를 빠져나올 것으로 예상된다. 

하지만 실제로 코드를 수행해보면 프로그램은 종료되지 않는다. 동기화를 하지 않았기 때문에 메인 스레드가 수정한 boolean 변수의 값이 백그라운드 스레드에게 언제 변경된 값으로 보일지 모른다. 또한 동기화 코드가 없다면 JVM에서 아래와 같은 최적화를 할 수도 있다.

```java
// 원래 코드
while (!stopRequested)
    i++;

// 최적화한 코드
if (!stopRequested)
    while (true)
        i++;
```

이는 JVM이 실제로 적용하는 **끌어올리기(hoisting, 호이스팅)** 라는 최적화 기법이 사용된 것이다.

## 호이스팅이란?

자바 스크립트에서 사용되는 개념으로 여기서는 개념적으로 while 문의 내용을 if문으로 끌어올려 JVM이 최적화를 진행하였다 정도로 이해하면 좋을 듯 합니다

호이스팅 개념: 함수 안에 있는 선언들을 모두 끌어올려서 해당 함수 유효 범위의 최상단에 선언하는 것

### 대상
- var 변수 선언과 함수 선언문
- let/const 변수 선언과 함수 표현식은 대상X

### var와 let/const 사이의 차이가 발생하는 이유

- 이유를 알기 위해 변수가 메모리에 저장될 때 과정을 이해해야 한다.
    - 변수가 메모리에 저장되는 과정
        1. 선언 단계: 실행 컨텍스트 변수 객체에 등록
        2. 초기화 단계: 변수 객체에 대한 메모리 할당 및 변수를 undefined로 초기화
        3. 할당 단계: undefined로 초기화 된 변수에 값 할당
- 위 3단계 과정에서 var와 const/let 사이에 차이가 발생하는데 var는 1,2단계가 동시에 진행되면서 호이스팅이 되고, const/let은 1,2단계가 분리되어 진행되므로 1단계만 진행되면서 호이스팅이 된다. 호이스팅이 된 시점을 봐보면 var의 경우 변수에 undefined가 할당되어 있는 상태고, const/let은 undefined로 초기화되지 않았으므로 ReferenceError가 발생한다.

결과적으로 응답 불가(liveness failure) 상태가 되어 더 이상 진행되는 코드가 없다. 다시 기존 코드로 돌아와서 생각해보면, 공유하는 변수를 다룰 때 동기화하는 코드를 넣으면 된다.

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

이처럼 동기화는 읽기와 쓰기에 대해 모두 필요하다. 위 코드처럼 공유 필드에 대한 읽기/쓰기 메서드 모두를 동기화 처리하면 문제는 해결된다.

## Volatile

- `volatile` keyword는 Java 변수를 Main Memory에 저장하겠다라는 것을 명시하는 것
- 매번 변수의 값을 Read할 때마다 CPU cache에 저장된 값이 아닌 Main Memory에서 읽는 것
- 또한 변수의 값을 Write할 때마다 Main Memory에 까지 작성하는 것이다.

배타적 수행과는 상관이 없지만 항상 가장 최근에 저장된 값을 읽어온다. 이론적으로는 CPU 캐시가 아닌 컴퓨터의 메인 메모리로부터 값을 읽어온다. 그렇기 때문에 읽기/쓰기 모두가 메인 메모리에서 수행된다.

```java
public class stopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

위 코드처럼 `volatile`을 사용하면 동기화를 생략해도 된다. 다만 주의해서 사용해야 한다. 아래와 같은 예제에서 문제점을 찾아볼 수 있다.

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

코드상으로 증가 연산자(++)는 하나지만 실제로는 **volatile 필드에 두 번 접근** 한다. 

먼저 값을 읽고, 그 다음에 1을 증가한 후 새로운 값을 저장하는 것이다. 따라서 두 번째 스레드가 첫 번째 스레드의 연산 사이에 들어와 공유 필드를 읽게 되면, 첫 번째 스레드와 같은 값을 보게될 것이다.

이처럼 잘못된 결과를 계산해내는 오류를 **안전 실패(safety failure)** 라고 한다. 이 문제는 메서드에 `synchronized`를 붙이고 `volatile` 키워드를 공유 필드에서 제거하면 해결된다.

# **atomic 패키지**

`java.util.concurrent.atomic` 패키지에는 락 없이도 thread-safe한 클래스를 제공한다. `volatile`은 동기화의 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다. 게다가 성능도 동기화 버전보다 우수하다.

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

## 결론

가변 데이터를 공유하지 않는 것이 동기화 문제를 피하는 가장 좋은 방법이다. 

즉, 가변 데이터는 단일 스레드에서만 사용하자. 한 스레드가 데이터를 수정한 후에 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 다른 스레드에 이런 객체를 건네는 행위를 **안전 발행(safe publication)** 이라고 한다. 

클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드 혹은 보통의 락을 통해 접근하는 필드 그리고 동시성 컬렉션에 저장하면 안전하게 발행할 수 있다.