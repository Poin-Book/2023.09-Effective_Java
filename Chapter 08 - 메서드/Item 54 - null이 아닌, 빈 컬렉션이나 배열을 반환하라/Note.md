# Item 54 null이 아닌, 빈 컬렉션이나 배열을 반환하라

> 작성자: @destiny3912

## 목차

- [Item 54 null이 아닌, 빈 컬렉션이나 배열을 반환하라](#item-54-null이-아닌-빈-컬렉션이나-배열을-반환하라)
  - [목차](#목차)
  - [본문](#본문)
    - [0. Java에서의 null이란? - 링크 참고](#0-java에서의-null이란---링크-참고)
    - [1. 메서드가 널을 반환한다면?](#1-메서드가-널을-반환한다면)
    - [2. 컨테이너 생성 비용은?](#2-컨테이너-생성-비용은)
    - [3. 요약](#3-요약)
  
## 본문

### 0. Java에서의 null이란? - 링크 참고

---

[☕ 개발자들을 괴롭히는 자바 NULL 파헤치기](https://inpa.tistory.com/entry/JAVA-☕-개발자들을-괴롭히는-NULL-파헤치기)

### 1. 메서드가 널을 반환한다면?

---

> Optional
> 
- Java에서는 Optional로 랩핑해서 null의 핸들링이 가능하다
- 다음과 같이
    
    ```java
    Optional<String> optionalValue = Optional.ofNullable(getValue());
    optionalValue.ifPresent(value -> System.out.println(value));
    ```
    
- 이러한 것을 흔히 방어 코드라고 한다.
- 이 코드를 빼먹으면 컴파일 타임에 에러를 잡지 못하므로 NullPointerException을 던지면서 런타임에 프로그램이 뻗어버리는 것을 볼 수 있다.
- 따라서 빈 컬랙션이나 배열을 반환하는 것이 여러모로 좋음

### 2. 컨테이너 생성 비용은?

---

> 불변의 빈 컨테이너(컬랙션, 배열)을 생성해두고 돌려쓰자!
> 
- 불변 객체는 자유롭게 공유해서 사용해도 안전함 (Item 17)
- 아니면 다음과 같이 사용해도 된다.
    
    ```java
    // 차례로 빈 리스트, 빈 맵, 빈 셋
    Collections.emptyList();
    Collections.emptyMap();
    Collections.emptySet();
    //그냥 빈 배열
    new int[0];
    // 빈 배열 생성 비용도 아깝다 -> 미리 상수로 생성해서 돌려쓰자
    private static final int[] EMPTY_INT_ARR = new int[0]
    ```
    
- 위 코드를 리턴할 배열이나 컬랙션이 비었는지 검사하는 코드에서 사용하면 된다.

### 3. 요약

---

- null이 아닌, 빈 배열이나 컬렉션을 반환하라
- null을 반환하는 메서드는 사용하기 굉장히 귀찮고 까다롭다 (NullPointerException)\
- 또한 성능이 좋지도 않다.
