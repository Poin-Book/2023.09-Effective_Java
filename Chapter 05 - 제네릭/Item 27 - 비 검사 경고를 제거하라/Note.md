# 비 검사 경고를 제거하라

> 작성자: 루카

## 비 검사 경고

`제네릭`을 사용하기 시작하면 수많은 컴파일러 경고를 보게 된다.

| 경고 | 설명 |
|:---:|:---|
| 비검사 형변환 경고 | 프로그래머가 제네릭을 사용할 때마다 컴파일러는 형변환을 수행하는데, 이때 형변환을 검사하지 않는다. |
| 비검사 메서드 호출 경고 | 제네릭 타입이 아닌 메서드를 호출할 때 발생한다. |
| 비검사 매개변수화 가변인수 타입 경고 | 제네릭 가변인수 메서드에 잘못된 타입의 매개변수가 전달될 때 발생한다. |
| 비검사 변환 경고 | 제네릭 타입이 아닌 타입을 제네릭 타입으로 변환할 때 발생한다. |

대부분의 비검사 경고는 쉽게 제거 가능

```java
Set<Lark> exaltation = new HashSet();

Venery.java:4: warning: [unchecked] unchecked conversion
    Set<Lark> exaltation = new HashSet();
                            ^
```

컴파일러가 알려준 대로 수정하면 경고가 사라진다.

```java
// <> 다이아몬드 연산자를 사용하면 컴파일러가 타입을 추론해준다.
Set<Lark> exaltation = new HashSet<>();
```

## @SuppressWarnings("unchecked")

> 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있을 때 사용

경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 에노테이션을 달아 경고를 숨기자

안전하다고 검증된 비검사 경고를 (숨기지 않고)그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다.

하지만 @SuppressWarnings 에너테이션은 항상 가능한 한 좁은 범위에 적용하자. 자칫 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안 된다.

한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 에너테이션을 발견하면 지역 변수 선언 쪽으로 옮기자.

```java
public <T> T[] toArray(T[] a) {
    if(a.length < size) {
    // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로
    // 올바른 형변환이다.
    @SuppressWarnings("unchecked") T[] result =
        (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    ...
}
```

위 처럼 @SuppressWarnings("unchecked") 에너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.

### SuppressWarnings 종류

| 종류 | 설명 |
|:---:|:---|
| all | 모든 경고를 억제 |
| cast | 캐스트 연산자 관련 경고를 억제 |
| dep-ann | 사용하지 말아야 할 주석 관련 경고를 억제 |
| deprecation | 사용하지 말아야 할 메서드 관련 경고를 억제 |
| fallthrough | switch문에서 break 누락 관련 경고를 억제 |
| finally | 반환하지 않는 finally 블록 관련 경고를 억제 |
| null | null 분석 관련 경고를 억제 |
| rawtypes | 제네릭 타입을 사용하지 않고, 제네릭 타입이 지정된 클래스나 인터페이스를 참조할 때 관련 경고를 억제 |
| unchecked | 검사되지 않은 연산자 관련 경고를 억제 |
| unused | 사용되지 않는 코드 관련 경고를 억제 |

## 요약

`비검사 경고`는 중요하니 무시하지 말자. 모든 비검사 경고는 `런타임`에 ClassCastException을 일으킬 수 있는 `잠재적 가능성`을 뜻하니 최선을 다해 제거하라. 경고를 없앨 방법을 찾지 못하겠다면, 그 코드가 `타입 안전함을 증명`하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 에너테이션으로 경고를 숨겨라. 그런 다음 경고를 `숨기기로 한 근거를 주석`으로 남겨라