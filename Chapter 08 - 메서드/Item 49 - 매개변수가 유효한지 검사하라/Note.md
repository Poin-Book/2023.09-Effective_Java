# 매개변수가 유효한지 검사하라

> 작성자: {다나}

## 목차
* 오류를 즉시 잡아라
* 예외의 문서화
* private 메서드의 유효성 검사
* 나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라
* 매개변수 검사의 예외사항
* 핵심 정리
## 오류를 즉시 잡아라

- 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기도 어렵고, 정확히 어디서 발생했는지 알기도 어렵다.
- 매개변수 검사를 제대로 하지 못했다면 메서드가 수행되다가 모호한 예외를 던지며 실패한다.
- 메서드가 잘 수행되지만 잘못된 결과를 반환할 수도 있고, 사용한 다른 객체를 이상한 상태로 만들어 미래의 알 수 없는 시점에 문제가 발생한다.

## 예외의 문서화

- `public`과 `protected` 메서드는 매개변수 값이 잘못 됐을 때 던지는 예외를 문서화 해야 한다.
- 자바독의 `@throws` 태그를 이용해서 가능하다.
- `IllegalArgumentException`, `IndexOutOFBoundsException`, `NullPointerException` 중 하나가 될 것이다.

### ■ 예외 문서화 코드의 예시

```java
**/**
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 * @param  m 계수 (양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */**
public BigInteger mod(BigInteger m) {
		// 매개변수 검사
    if (m.signum <= 0)
        throw new ArithmeticException("계수(m)은 양수여야 합니다");
		
		// 계산 수행
    BigInteger result = this.remainder(m);
    return (result.signum >= 0 ? result : result.add(m));
}
```

- `@throws` 태그는 0보다 작거나 같은 값의 나머지를 구하려 하면, `ArithmeticException`이 발생한다고 친절하게 알려주고 있다.
- 이 메서드는 m이 null이면 m.signum() 호출 때 `NullPointerException`을 던진다.
    - 그런데 "m이 null일 때 NullPointerException을 던진다"라는 말은 메서드 설명 어디에도 없다.
    - 그 이유는 이 설명을 개별 메서드가 아닌 BigInteger 클래스 수준에서 기술했기 때문이다.
    - 클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔한 방법이다.
    
    <aside>
    📎 Null에 대한 검사
    
    ```java
    this.strategy = Objects.requireNonNull(strategy, "전략");
    ```
    
    - 자바 7에 추가된 java.util.Objects.requireNonNull 메서드는 null이 들어오면 NullPointerException 예외를 던진다. 원하는 예외 메시지도 지정할 수 있다.
    - 자바 9에서는 Objects 에 범위 검사 기능도 더해졌다.
        - `checkFromIndexSize`, `checkFromToIndex`, `checkIndex`
        - 예외 메세지 지정할 수 없고, 리스트와 배열 전용으로 설계됐다.
        - 또한 닫힌 범위는 다루지 못한다.
    </aside>
    

## private 메서드의 유효성 검사

- private 메서드라면 패키지 제작자인 프로그래머가 메서드가 호출되는 상황을 통제 할 수 있다.
- 오직 유효한 값만이 메서드에 넘겨지리라는 것을 보증할 수 있고, 그렇게 해야한다.
- public이 아닌 메서드라면 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있다.

```java
private static void sort(long a[], int offset, int length) {
		assert a!=null;
    assert offset>=0 && offset<=a.length;
    assert length>=0 && length<=a.length;
    // 계산 수행..
}
```

- 여기서 핵심은 이 단언문들이 자신이 단언한 조건이 무조건 참이라고 선언한다는 것이다.
- 이 단언문은 몇가지 면에서 일반적인 유효성 검사와 다르다.
    - 첫 번째, 실패하면 AssertionError를 던진다.
    - 두 번째, 런타임에 아무런 효과도, 아무런 성능 저하도 없다.

## **나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라**

- 메서드가 직접 사용하지 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써서 검사해야 한다.
    
    ex ) 입력받은 int 배열의 List 뷰를 반환하는 메서드는 null 검사를 수행하므로 클라이언트가 null을 건네면 NullPointerExcpetion을 던진다.
    
    - 만약 이 검사를 생략했다면 새로 생성한 List 인스턴스를 반환하는데, 클라이언트가 돌려받은 List를 사용하려 할 떄 비로소 `NullPointerException`이 발생한다.
    - 이때가 되면 이 List가 어디서 가져왔는지 추적하기 어려워 디버깅이 상당히 괴로워진다.
- 생성자는 “나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라”는 원칙의 특수한 사례이다.
    - 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다.

## 매개변수 검사의 예외사항

- 유효성 검사 자체가 비용이 지나치게 높거나 실용적이지 않을 때, 계산 과정에서 암묵적으로 검사가 수행될 때다.
    - `Collections.sort(List)`의 경우 상호 비교될 수 없는 타입의 객체가 들어있다면 그 객체와 비교할 때 `ClassCastException`을 던질 것이다.
    - 따라서 미리 리스트 안의 모든 객체에 대해 상호 검사해봐야 실익이 없다.
- 유효성 검사에 너무 의존하면 실패 원자성(아이템76)을 해칠 수도 있다.
    - 실패 원자성이란, 유효성 검사에 실패한 이후에 객체의 상태가 영구적으로 바뀌어 다음 실행 결과가 다른 경우를 의미한다.
    - 메서드가 여러번 실패해도 매번 같은 결과를 반환해야 실패 원자성이 지켜지는 것이다.
- 계산 중 던져지는 예외와 API에서 던지기로 한 예외가 다를 수 있다.
  - 이 경우 저수준의 예외를 잡아 고수준의 사용자정의 예외로 바꿔주는 예외 번역이 필요할 수 있다.

## 핵심 정리

- “매개변수에 제약을 두는게 좋다”고  해석하면 안된다. 오히려 그 반대인 “메서드는 최대한 범용적으로 설계해야 한다.”이다.
- 제약이 있다면 메서드 코드 시작 부분에서 명시적으로 검사해야 한다.
