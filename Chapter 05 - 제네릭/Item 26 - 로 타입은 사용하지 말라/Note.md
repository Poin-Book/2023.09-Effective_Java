# Item 26 - 로 타입은 사용하지 말라

> 작성자: {루키}

## 목차
- [Item 26 - 로 타입은 사용하지 말라](#item-26---로-타입은-사용하지-말라)
  - [목차](#목차)
  - [본문](#본문)
    - [1. 들어가며](#1-들어가며)
    - [2. 왜 로 타입을 사용하지 말라는 것인가?](#2-왜-로-타입을-사용하지-말라는-것인가)
    - [3. 로 타입과 제네릭의 차이](#3-로-타입과-제네릭의-차이)
    - [4. 와일드카드](#4-와일드카드)
    - [5. 로 타입의 사용처](#5-로-타입의-사용처)
## 본문

### 1. 들어가며

우선 용어 정리부터

- 제네릭 클래스 / 제네릭 인터페이스
    - 클래스와 인터페이스 선언에 타입 매개변수(type parameter)가 쓰이는 것
- 매개변수화 타입
    - 예를 들어서 List<String> 이라는 정의에서 String을 의미한다.
- 제네릭 타입을 하나 정의하면 그에 딸려오는 로(raw)타입도 함께 정의된다.
    - 예를 들어서 List<E>의 로 타입은 List이다.
        - 제네릭 추가 이전 코드와의 하위 호환성을 위함

### 2. 왜 로 타입을 사용하지 말라는 것인가?

> 타입 안전성
> 
- 예를 들어서 컬랙션의 로 타입을 사용한다고 해보자
    
    ```java
    private final Collection stamps = ...;
    
    stamps.add(new Coin());
    ```
    
- 위 코드처럼 스탬프에 코인을 넣어도 컴파일이 잘 된다.
- 다만 경고 메시지가 뜨긴 할건데 (unchecked call) 모호하다
- 이러한 오류는 넣은 동전을 다시 꺼내기 전까지는 알아채는 것이 불가능하다
    
    ```java
    Stamp stamp = stamps.get(0); // ClassCastExcetion이 발생
    ```
    
- 이러한 오류는 즉시 발견이 어렵고 발생한다 하더라도 발생한 위치와 원인이 각각 다른 위치이기 때문에 수정이 어려움
- 따라서 매개변수를 넣어 주어야 한다
    
    ```java
    private final Collection<Stamp> stamps = ...;
    ```
    
- 이렇게 하면 다른 타입을 집어 넣으려 하면 컴파일러가 오류를 낸다.

### 3. 로 타입과 제네릭의 차이

- 예를 들어보자
    - List는 그냥 제네릭과는 아무 상관이 없다
    - 또한 List<Object>는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달 하는 것
    - 여기서 List<String>을 List를 받는 메서드에는 넘길 수 있지만 List<Object>를 받는 메서드에는 못쓴다.
        - 왜냐하면 제네릭의 하위 타입 규칙 때문이다
        - List<String>은 List의 하위 타입이지 List<Object>의 하위 타입이 아니기 때문이다.

```java
List<String> strings = new ArrayList<>();

unsafeAdd(strings, Integer.valueOf(42));
String s = strings.get(0);

private static void unsafeAdd(List list, Object o) {
	list.add(o);
}
```

- 위 코드를 실행하면 unchecked call 경고가 뜨게된다
- 만약 그 오류를 무시하면 strings.get(0)의 결과를 형변환 할때 오류를 낸다 (ClassCastException)
- 보통은 형변환은 컴파일러가 알아서 하는거라 오류를 내지는 않지만 경고를 무시한 것

### 4. 와일드카드

> 타입을 지정하지 않고 받고 싶어요
> 
- 그러면 로 타입을 쓰면 되는거 아닌가?
- 쓸수는 있다만 상술한 이유 때문에 문제다.
- 따라서 와일드카드 타입을 사용하자
1. 비한정적 와일드 카드 타입
    
    > ?
    > 
    - Set에 실제 타입 매개변수가 뭔지 신경안쓰고 싶다면
        
        ```java
        Set<?>
        ```
        
    - 이것과 같이 쓰자
2. Set<?>와 Set의 차이는 무엇인가?
    
    > 전자는 안전하고 후자는 안전하지 않다.
    > 
    - 로 타입 컬랙션에는 아무 원소나 넣을 수 있어서 타입 불변식을 훼손하기 쉬움
    - Collection<?>에는 null 이외의 어떠한 원소도 넣을 수 없음
        - 이러한 이유는 unknown type 때문이다.
        - 값을 추가하기 위해서는 어떤 타입인지를 알아야 하는데 그렇지 못하기 때문이다.
3. 한정적 와일드 카드
    - src: https://mangkyu.tistory.com/241
    
    > 특정 타입을 기준으로 하자
    > 
    - 예시는 다음 클래스 구조를 기반으로 한다
        
        ```java
        class MyGrandParent {
        
        }
        
        class MyParent extends MyGrandParent {
        
        }
        
        class MyChild extends MyParent {
        
        }
        ```
        
    - 상한 경계 와일드카드
        
        > 와일드 카드 타입에 extends를 사용해서 와일드카드 타입의 최상위 타입을 정의
        > 
        - 원소 꺼내기
            
            ```java
            void printCollection(Collection<? extends MyParent> c) {
                // 컴파일 에러
                for (MyChild e : c) {
                    System.out.println(e);
                }
            
                for (MyParent e : c) {
                    System.out.println(e);
                }
            
                for (MyGrandParent e : c) {
                    System.out.println(e);
                }
            
                for (Object e : c) {
                    System.out.println(e);
                }
            }
            ```
            
            - 상한 경계를 MyParent로 주었을때
            - MyChild 타입으로 꺼낼때 컴파일 에러가 발생
        - 원소 추가
            
            ```java
            void addElement(Collection<? extends MyParent> c) {
                c.add(new MyChild());        // 불가능(컴파일 에러)
                c.add(new MyParent());       // 불가능(컴파일 에러)
                c.add(new MyGrandParent());  // 불가능(컴파일 에러)
                c.add(new Object());         // 불가능(컴파일 에러)
            }
            ```
            
            - 이 경우는 모든 경우에 대해서 에러가 발생
            - <? extends MyParent>로 가능한 타입을 MyParent와 모르는 모든 자식 클래스 인데
            - 우리는 c가 MyParent의 하위 타입중에서 어떤 타입인지 모르기 때문이다.
            - 또한 상위 타입들은 그냥 MyParent가 아니기 때문에 불가능 하다
            - 이때는 하한 경계를 지정해야 함 (Consume)
    - 하한 경계 와일드 카드
        
        > super를 사용해 와일드 카드의 최하위 타입을 정의
        > 
        - Consume
            
            ```java
            void addElement(Collection<? super MyParent> c) {
                c.add(new MyChild());
                c.add(new MyParent());
                c.add(new MyGrandParent());  // 불가능(컴파일 에러)
                c.add(new Object());         // 불가능(컴파일 에러)
            }
            ```
            
            - c가 원하는 타입은 MyParent의 부모 타입들이다.
            - 따라서 MyParent의 자식 타입이면 안전하게 추가가 가능하고
            - 부모 타입인 경우에만 에러가 발생한다.
        - Produce
            
            ```java
            void printCollection(Collection<? super MyParent> c) {
                // 불가능(컴파일 에러)
                for (MyChild e : c) {
                    System.out.println(e);
                }
            
                // 불가능(컴파일 에러)
                for (MyParent e : c) {
                    System.out.println(e);
                }
            
                // 불가능(컴파일 에러)
                for (MyGrandParent e : c) {
                    System.out.println(e);
                }
            
                for (Object e : c) {
                    System.out.println(e);
                }
            }
            ```
            
            - <?, super Myparent>로 가능한 타입은 MyParent와 그 부모 타입들이므로 부모 타입을 특정할 수 없어서 모든 부모 타입에 에러가 발생
            - 다만 Object는 언어가 보장하는 최상위 타입이기 때문에 오류가 발생하지 않음
            - 하위 타입이어도 문제인게 경계가 MyParent이므로 이 경계 아래의 타입은 추가가 불가능
    - 그래서 언제 사용하는가
        
        > PECS(Producer-Extends, Consumer-Super)
        > 
        
        ```java
        void printCollection(Collection<? extends MyParent> c) {
            for (MyParent e : c) {
                System.out.println(e);
            }
        }
        
        void addElement(Collection<? super MyParent> c) {
            c.add(new MyParent());
        }
        ```
        
        - 와일드 카드 타입의 객체를 생성하면 (produce) extends
        - 가지고 있는 객체를 컬랙션에 사용하면 (consume) super

### 5. 로 타입의 사용처

> class 리터럴
> 
- 예를 들어서 int.class

> instanceof 연산자
> 
- 런타임에는 제네릭 타입 정보가 지워짐 (소거)
- 또한 로 타입이든 비한정적 와일드카드 타입이든 똑같이 동작함
- 예시
    
    ```java
    if (o instanceof Set) {
    	Set<?> s = (Set<?>) o;
    }
    ```
    
    - o의 타입이 Set임을 확인한 다음 와일드카드 타입으로 형변환
    - 이것은 검사 형변환(checked cast)이므로 경고가 뜨지 않는다.