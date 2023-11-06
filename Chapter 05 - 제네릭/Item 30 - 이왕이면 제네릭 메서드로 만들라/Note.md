# 이왕이면 제네릭 메서드로 만들라

> 작성자: {밀리}

## 목차
- [제네릭 메서드 작성 방법](#제네릭_메서드_작성_방법)
- [제네릭 싱글턴 팩터리](#제네릭_싱글턴_메서드)
- [재귀적 타입한정 이용하기](#재귀적_타입한정_이용하기)  
- [핵심 정리](#핵심_정리)

### 제네릭 메서드 작성 방법
- 매개변수화 타입을 받는 정적 유틸리티 메서드(static)는 보통 제네릭이다.  
    - ```Collections```의 ```binarySearch```, ```sort``` 등 알고리즘 메서드는 모두 제네릭이다.  
- 제네릭 메서드 작성법은 제네릭 타입 작성법과 비슷하다.  

```java
public static Set union(Set s1, Set s2) {
   Set result = new HashSet(s1);
   result.addAll(s2);
   return result;
}
```
Set result = new HashSet(s1);
result.addAll(s2);
- 컴파일은 되지만 위 두 부분에서 경고가 두 개 발생한다. 
- 경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다. 메서드 선언에서의 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다. (타입 매개변수들을 선언하는)
- 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.  

```java
public static <E/*타입 매개변수 목록*/> Set<E/*반환 타입*/> union(Set<E/*파라미터 타입*/> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
``` 
- 순서대로 타입 매개변수 목록, 반환 타입, 파라미터 타입 3가지를 메서드 시그니처에 입력할 수 있다.  

#### 활용 예시
```java
@Test
public void unionTest() {
    Set<String> guys = Set.of("톰", "딕", "헤리");
    Set<String> stooges = Set.of("래리", "모에", "컬리");
    Set<String> aflCio = union(guys, stooges);
    System.out.println("aflCio = " + aflCio);
}
```  
- ```Set<String>```타입 2개를 합쳤다.  
    - 입력 2개와 반환 1개 타입이 모두 일치했다.  
    - 이는 사실 한정적 와일드카드 타입(아이템 31)을 사용하여, 더 유연하게 개선할 수 있다.  


### 제네릭 싱글턴 팩터리
- 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.  
    - 이를 이용해 불변 객체를 여러 타입으로 이용할 수 있게 만들 수도 있다.  
- 객체를 매개변수화하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리가 필요하다.  

- 이 정적 팩터리를 제네릭 싱글턴 팩터리라고 한다.  
```java
public static final <T> Set<T> emptySet() {
    return (Set<T>) EMPTY_SET;
}
```  
- 제네릭 싱글턴 팩터리의 예이다. EMPTY_SET은 Set 로타입 불변객체이다.  
```java
private static final UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```  
- 위는 제네릭 싱글턴 팩터리로 항등함수를 작성한 예이다.  
    - 제네릭으로 수용 가능한 어떤 타입이든 작동하는 항등함수이다.  
        - UnaryOperator를 어떤 타입이든 계속해서 이용할 수 있다.  

```java
@Test
public void identityFunctionTest() {
    String[] strings = { "삼베", "대마", "나일론" };
    UnaryOperator<String> sameString = identityFunction();
    for (String string : strings) {
        System.out.println(sameString.apply(string));
    }

    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number number : numbers) {
        System.out.println(sameNumber.apply(number));
    }
}
```  

### 재귀적 타입한정 이용하기
- 제네릭의 타입 범위를 한정하는 것이다.  
- 이런 재귀적 타입 한정은 주로 타입의 자연적 순서를 지정해주는 ```Comparable```과 함께 사용된다.  


```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```  
- E로 받을 타입은 오직 ```Comparable<E>```를 구현한 타입만 가능하다는 뜻이다.
    - 즉, ```Comparable```을 구현한 타입만 가능하다는 뜻이다.  


### 핵심 정리
- 클라이언트에서 입력 매개변수 및 반환값을 명시적으로 형변환하는 메서드보다 제네릭 메서드가 더 안전하고 사용하기도 쉽다.  
- 형 변환은 런타임 시에 에러를 동반하기 쉬우므로 제네릭을 사용하자.  
