# 다른 타입이 적절하다면 문자열 사용을 피하라

> 작성자 : 다나

## 목차
- 문자열을 쓰지 말아야 할 사례
- 혼합 타입을 문자열로 처리한 부적적한 예
- 권한 표현을 문자열로 처리한 부적절한 예
  - 리팩토링
- 핵심 정리
## ****문자열을 쓰지 말아야 할 사례****

1. 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
    
    > 수치형이라면, int, float, BigInteger 등 적당한 수치 타입을 사용하자.
    > 
2. 문자열은 열거 타입을 대신하기에 적합하지 않다.
    
    > '예/아니오' 질문의 답이라면, 적절한 열거 타입이나 boolean 타입을 사용하자.
    > 
3. 문자열은 혼합 타입을 대신하기에 적합하지 않다.
    
    > 여러 요소가 혼합된 데이터(혼합 타입)인 경우, 전용 클래스를 만들어 사용하자. 하나의 문자열로 표현하는 것은 대체로 좋지 않다.
    > 


## 혼합 타입을 문자열로 처리한 부적절한 예

```java
String compoundKey = className + "#" + i.next();
```

- 두 요소를 구분해주는 문자 #이 두 요소 중 하나에서 쓰였다면 혼란스러운 결과를 초래한다.
- 각 요소를 개별로 접근하려면 문자열을 파싱해야 한다.→ 느리고, 번거롭고, 오류 가능성이 커진다.
- equals, toString, compareTo 메서드를 제공할 수 없으며, String이 제공하는 기능에만 의존해야 한다.
- 그러므로 전용 클래스를 새로 만들자. 보통 private 정적 멤버 클래스로 선언한다.


##  권한 표현을 문자열로 처리한 부적절한 예

- 권한(capacity)을 문자열로 표현하는 경우가 종종 있는데, 문자열은 권한을 표현하기에 적합하지 않다.
- 스레드 지역변수 기능(각 스레드가 자신만의 변수를 갖게 해주는 기능)을 설계한다고 해보자. 이때 클라이언트가 제공한 문자열 키로 스레드별 지역변수를 식별하도록 해보자.

```java
public class ThreadLocal() {
	private ThreadLocal() {} // 객체 생성 불가

    // 현 스레드의 값을 키로 구분해 저장한다.
    public static void set(String key, Object value);

    // (키가 가리키는) 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```

- 이 방식의 문제는 스레드 구분용 문자열 키가 전역 이름공간에 공유된다는 점이다.
- 이 방식대로 하면, 서로 다른 클라이언트가 동일한 키를 사용할 경우 제대로 동작하지 못한다.
- 문제를 해결하려면 문자열 대신 위조할 수 없는 키를 사용하면 된다. 이 키를 권한(capacity)라고 한다.

### 리팩토링 1

```java
public class ThreadLocal {
	private ThreadLocal() {} // 객체 생성 불가

    private static class Key { // 권한
    	Key() {}
	}

    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

- set과 get은 이제 정적 메서드일 이유가 없으니 Key 클래스의 인스턴스 메서드로 바꾸자.
- 톱레벨 클래스인 TreadLocal은 별로 하는 일이 없어지므로 없애고 중첩 클래스 Key의 이름을 ThreadLocal 바꿔버리자.

### 리팩토링 2

```java
public final class ThreadLocal {    
		public ThreadLocal();    
		public void set(Object value);    
		public Object get();
}
```

- 이 API는 get으로 얻은 Object를 실제 타입으로 형변환해 써야 해서 타입 안전하지 않다.
- ThreadLocal을 매개변수화 타입으로 선언하여, 타입 안전하게 만들자.

### 리팩토링 3

```java
public final class ThreadLocal<T> {
	public ThreadLocal();
    public void set(T value);
    public T get();
}
```

- 이제 자바의 java.lang.ThreadLocal과 흡사해졌다.
- 문자열 기반 API의 문제를 해결해주며, 키 기반의 API보다 빠르고 우아하다.


## 📌 핵심 정리

- 더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열을 쓰고 싶은 유혹을 뿌리쳐라.
- 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다.
- 문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입이 있다.
