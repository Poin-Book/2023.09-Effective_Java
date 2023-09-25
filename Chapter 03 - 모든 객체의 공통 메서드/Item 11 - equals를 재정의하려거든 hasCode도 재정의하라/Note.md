# equals를 재정의하려거든 hashCode도 재정의하라

> 작성자: 루카
>
> 요약: equals를 정의할 떄는 hashCode도 반드시 재정의해야 한다.

- [equals를 재정의하려거든 hashCode도 재정의하라](#equals를-재정의하려거든-hashcode도-재정의하라)
  - [Object의 일반 규약](#object의-일반-규약)
  - [논리적으로 같은 객체는 같은 해시코드를 반환해야 한다](#논리적으로-같은-객체는-같은-해시코드를-반환해야-한다)
  - [적절한 hashCode 메서드](#적절한-hashcode-메서드)
    - [적법하지만 사용해서는 안되는 hashCode 메서드 - 사용금지](#적법하지만-사용해서는-안되는-hashcode-메서드---사용금지)
    - [전형적인 hashCode 메서드](#전형적인-hashcode-메서드)
    - [한 줄짜리 hashCode 메서드 - 성능이 아쉬움](#한-줄짜리-hashcode-메서드---성능이-아쉬움)
    - [해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성 고려해야함](#해시코드를-지연-초기화하는-hashcode-메서드---스레드-안정성-고려해야함)
  - [hashCode 메서드 작성시 주의점](#hashcode-메서드-작성시-주의점)
- [부록](#부록)
  - [부록 A. Hash란?](#부록-a-hash란)
  - [부록 B. AutoValue 프레임워크](#부록-b-autovalue-프레임워크)
  - [부록 C. 구아바의 com.google.common.hash 패키지](#부록-c-구아바의-comgooglecommonhash-패키지)

equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

## Object의 일반 규약

Object 명세에는 다음과 같이 써 있다.

- equals 비교에 사용되는 `정보가 변경되지 않았다`면, 그 객체의 hashCode 메서드는 `몇 번을 호출해도 항상 같은 값을 반환`해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- [중요] equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 `똑같은 값을 반환`해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

## 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다

위의 Object의 일반 규약에서 hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째 조항이다. 즉, `논리적으로 같은` 객체는 같은 해시코드를 반환해야 한다.  

- 아이템 10에서 보았듯이 equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있다.
- 하지만 Object의 기본 hashCode 메서드는 이 둘이 전혀 다른 객체라고 판단한다.

ex) PhoneNumber(hashCode를 제정의하지 않은) 클래스와 hashMap

- 아래의 코드에서 m.get(new PhoneNumber(707, 867, 5309))는 null을 반환한다.
  - 이는 PhoneNumber의 hashCode를 재정의하지 않았기 때문에 논리적으로 동치인 두 인스턴스가 서로 다른 해시코드를 반환하여 생긴 문제이다.

  ```java
  Map<PhoneNumber, String> m = new HashMap<>();
  m.put(new PhoneNumber(707, 867, 5309), "제니");
  m.get(new PhoneNumber(707, 867, 5309)); // null
  ```

## 적절한 hashCode 메서드

다음은 좋은 hashCode를 작성하기 위한 요령이다.

1. int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 `2.i` 방식으로 계산한 해시코드다. (핵심 필드란 equals 비교에 사용되는 필드를 말한다.)
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 c를 반환한다.
      1. 기본 타입 필드라면 Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다. (ex. Integer)
      2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 복잡해질까봐 걱정된다면, 이 필드의 표준형(canonical representation)을 만들어 이 표준형의 hashCode를 반환한다. 필드의 값이 null이면 0을 사용한다(다른 상수도 문제없음).
      3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시 코드를 계산한 다음, 단계 `2.ii` 방식으로 갱신한다. 배열의 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 반대로 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
   2. 단계 `2.i`에서 계산한 해시코드 c로 result를 갱신한다. <br> `result = 31 * result + c;`
3. result를 반환한다.

### 적법하지만 사용해서는 안되는 hashCode 메서드 - 사용금지

아래의 코드는 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법하다. 하지만 다음과 같은 문제를 지닌다.

- 모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마지 Linked List처럼 되어버린다.
- 그 결과 평균 수행시간이 O(1)이 되어야 할 해시테이블이 O(n)이 되어버린다.

```java
@Override public int hashCode() { return 42; }
```

### 전형적인 hashCode 메서드

> 고려 가능 상황: 대부분의 경우

아래의 코드는 PhoneNumber 클래스의 hashCode 메서드를 재정의한 것이다. PhoneNumber 인스턴스의 핵심 필드 3개만 사용해 간단한 계산만 수행하였으며 다음과 같은 장점을 지닌다.

- 비결정적 요소가 없음으로 동치인 두 인스턴스는 같은 해시코드를 반환한다.
- 단순하며, 충분히 빠르다.
- 서로 다른 인스턴스들을 다른 해시 버킷들로 제법 훌륭히 분배해준다.

```java
@Override public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```

### 한 줄짜리 hashCode 메서드 - 성능이 아쉬움

> 고려 가능 상황: 속도가 중요하지 않을 경우

아래의 코드는 Object 클래스의 hash 메서드를 이용한 것이다. hash 메서드란 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적인 메서드이다. 이 메서드를 활용해 hashCode 메서드를 작성하면 다음과 같은 장/단점이 있다.

[장점]

- 코드가 간결하다.

[단점]

- 속도가 더 느리다.
  - 입력 인수를 담기위한 배열이 만들어지고, 입력 중 기본타입이 있다면 박싱과 언박싱을 거쳐야한다.

```java
@Override public int hashCode() { 
  return Objects.hash(lineNum, prefix, areaCode);
}
```

### 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성 고려해야함

> 고려 가능 상황: 클래스가 불변이고 해시코드를 계산하는 비용이 큰 경우

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기보다는 캐싱하는 방식을 고려해야한다. 때문에 아래의 코드는 이 문제를 지연 초기화 전략을 통해 해결하고자 했으며 다음과 같은 주의점을 지닌다.

- 스레드 안정성을 고려해야한다.
  - 여러 스레드가 동시에 이 메서드를 호출하면 hashCode 필드가 아직 초기화되지 않은 상태일 수 있으니, 이를 방지하기 위해 동기화해야한다.
  - 동기화는 성능을 떨어뜨릴 수 있으니, 성능이 중요한 상황이라면 지연 초기화 전략을 사용하지 말아야 한다.
- hashCode 필드의 초기값을 흔히 생성되는 개체의 해시코드와는 달라야한다.

```java
private int hashCode; // 자동으로 0으로 초기화된다.

@Override public int hashCode() {
  int result = hashCode;
  if (result == 0) {
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    hashCode = result;
  }
  return result;
}
```

## hashCode 메서드 작성시 주의점

- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후 계산 방식을 바꿀 수도 있다.

# 부록

<!--
TODO: 부록 A. Hash란? 작성
TODO: 부록 B. AutoValue 프레임워크 작성
TODO: 부록 C. 구아바의 com.google.common.hash 패키지 작성
-->

## 부록 A. Hash란?

## 부록 B. AutoValue 프레임워크

## 부록 C. 구아바의 com.google.common.hash 패키지
