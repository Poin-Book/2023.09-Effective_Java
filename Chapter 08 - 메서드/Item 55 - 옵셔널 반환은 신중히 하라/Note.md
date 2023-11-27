# 옵셔널 반환은 신중히 하라

> 작성자: { 루카 }

- [옵셔널 반환은 신중히 하라](#옵셔널-반환은-신중히-하라)
  - [자바 8 이전 vs 이후](#자바-8-이전-vs-이후)
    - [자바 8 이전](#자바-8-이전)
    - [자바 8 이후](#자바-8-이후)
  - [옵셔널 반환의 장점과 주의점](#옵셔널-반환의-장점과-주의점)
    - [사용하면 안되는 경우](#사용하면-안되는-경우)
    - [사용하면 좋은 경우](#사용하면-좋은-경우)
  - [결론](#결론)

## 자바 8 이전 vs 이후

> 자바 8 이후 부터는 `Optional` 클래스가 추가되었다.

### 자바 8 이전

자바 8 이전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 두가지 선택지가 있었다.

- 예외를 던진다.
  - 예외는 진짜 예외적인 상황에서만 사용해야 한다.
  - 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용이 많이 든다.

- 반환 타입이 객체 참조라면 null을 반환한다.
  - 클라이언트에서 null을 반환하는 메서드를 사용할 때마다 null을 처리하는 코드를 추가해야 한다.

```java
// 컬렉션에서 최댓값을 구한다. - 컬렉션이 비었으면 예를 던진다.
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```

### 자바 8 이후

자바 8 이후에는 `Optional`이 생기면서 null이 아닌 타입을 참조하거나 아무것도 담지 않을 수 있게 되었다.

- Optional 객체는 원하는 값이 들어있을 수도, 아닐 수도 있다.
- 보통은 T를 반환하지만, 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 Optional을 반환한다.
  - 유효한 반환값이 없을 때는 Optional.empty()를 반환한다.
- Optional을 반환하는 메서드가 예외를 던지거나 null을 반환하는 것 보다는 오류가 적다.
- Optional을 반환하는 메서드에서는 절대 null을 반환하지 말아야 한다.

```java
// 컬렉션에서 최대값 구하기 - Optional<E> 반환
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}
```

```java
// steam version
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

## 옵셔널 반환의 장점과 주의점

Optional은 검사 예외와 취지가 비슷하다.

- 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.
- 검사 예외를 던진다면 클라이언트는 이에 대한 처리 코드를 작성할 수 있다.

```java
// Optional 활용 1 - 기본값 지정
String lastWordInLexicon = max(words).orElse("단어 없음...");

// Optional 활용 2 - 예외 던지기
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);

// Optional 활용 3 - 항상 값이 채워져 있다고 가정
Element lastNobleGas = max(Elements.NOBLE_GASES).get();

// Optional 활용 4 - 기본값 설정 비용이 큰 경우
public static String orElseGetBenchmark() {
    return Optional.of("fruit").orElseGet(() -> getRandomName());
}

// Optional 활용 5 - filter, map, flatMap, ifPresent

// Optional 활용 6 - isPresent 메서드
// Optional이 채워있으면 true, 비어있으면 false를 반환한다.
public class ParentPid {
    public static void main(String[] args) {
        ProcessHandle ph = ProcessHandle.current();

        // isPresent() 메서드를 이용해 Optional 객체가 비어있는지 확인한다.
        Optional<ProcessHandle> parentProcess = ph.parent();
        System.out.println("부모 PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));
    
        // map 메서드 활용
        System.out.println("부모 PID: " + ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));

        // stream 활용
        streamOfOptionals
            .filter(Optional::isPresent)
            .map(Optional::get);

        // 자바 9부터 Optional.stream() 메서드가 생겼다.
        // 옵셔널에 값이 있으면 그 값을 포함하는 스트림을 반환하고, 값이 없으면 빈 스트림을 반환한다.
        // flatMap 메서드를 이용하면 더욱 간결하게 표현할 수 있다.
        streamOfOptionals.flatMap(Optional::stream);
    }
}
```

### 사용하면 안되는 경우

- collection, stream, array, optional 같은 컨테이너 타입은 Optional로 감싸면 안된다.
- int, double, long 같은 기본 타입도 Optional로 감싸면 안된다.
  - 전용 옵셔널 클래스(OptionalInt, OptionalDouble, OptionalLong)를 사용하자.
- 옵셔널을 컬렉션의 키, 값, 원소, 배열의 원소로 사용하지 말자.
  - 복잡하고 오류가 생길 가능성이 크다.

### 사용하면 좋은 경우

- 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional을 반환한다.
- 단, 성능에 민감한 메서드에서는 Optional을 반환하지 말자.
  - Optional을 반환하는 메서드는 반환 전에 Optional을 만들어야 하고, 이 과정에서 객체를 생성하고 초기화해야 한다.
  - 성능이 중요한 상황에서는 어울리지 않는다.

## 결론

- 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 한다면 Optional을 반환할 수 있다.
- 단, Optional에는 성능 저하가 뒤따르니 성능에 민감한 메서드에서는 Optional을 반환하지 말자.
- Optional을 반환값 이외의 용도로 쓰는 경우는 드물다.