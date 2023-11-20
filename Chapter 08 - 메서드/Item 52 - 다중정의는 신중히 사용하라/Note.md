# 다중정의는 신중히 사용하라

> 작성자: {루카}

"재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다." 따라서 다중 정의는 신중하게 사용해야 한다.

## Override

메서드 재정의란 상위 클래스가 정의한 것과 같은 시그니처의 메서드를 하위 클래스에서 다시 정의하는 것을 의미한다.

메서드를 재정의한 다음 '하위 클래스 인스턴스'에서 그 메서드를 호출하면 재정의한 메서드가 실행되며, 컴파일 타임의 인스턴스 타입을 신경쓰지 않는다.

[재정의 된 메서드 호출 예제]

```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override
    String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override
    String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
            new Wine(), new SparklingWine(), new Champagne()
        );

        for (Wine wine : wineList) {
            System.out.println(wine.name()); // 포도주, 발포성 포도주, 샴페인
        }
    }
}
```

오버라이딩을 하면 for문에서의 컴파일 타입이 모두 Wine인 것에 무관하게 항상 `"가상 하위에서 정의한" 재정의 메서드가 실행`된다.

## Overload

다중정의 메서드 사이에서는, 객체의 런타임 타입은 전혀 중요하지 않고 오직 컴파일 타임에 매개변수의 컴파일 타입에 의해 선택이 이루어진다.

[컬렉션 분류기]

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c)); // "그 외", "그 외", "그 외"
    }
}
```

코드의 결과가 "집합", "리스트" "그 외" 가 아닌 "그 외"만 3번 출력하는 것은 세가지의 classfiy 메서드 중에 어느 메서드를 호출할지가 컴파일 타임에 정해졌기 때문이다. 즉, for 문 안의 c는 항상 `Collection<?>` 타입이기 때문에 `classify(Collection<?> c)` 만 호출되는 것이다.

올바르게 사용하려면 다음과 같이 classify 메서드를 하나로 합친 후 `instanceof`로 명시적으로 검사하면 말끔히 해결된다.

```java
public static String classify(Collection<?> c) {
    return c instanceof Set ? "집합" :
           c instanceof List ? "리스트" : "그 외";
}
```

### 다중정의 잘 사용하기

1. 매개변수 수가 같은 다중정의는 만들지 말아야 한다.
2. 가변인수(varargs)를 사용하는 메서드라면 다중정의를 아예 사용하면 안된다.
3. 기능이 똑같은 다중정의 메서드는 더 특수한 다중정의 메서드에서 덜 특수한(더 일반적인) 메서드로 일을 넘겨버리자.

[인수를 포워드하여 두 메서드가 동일한 일을 하도록 보장]

```java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```

해당 규칙을 따루면 어떤 다중정의 메서드가 호출될지 막을 수 있지만, 다중정의 대신 메서드의 이름을 다르게 지어주는 방법도 있다.

> ObjectOutputStream 클래스
>
>   - writeBoolean(boolean)
>   - writeInt(int)
>   - writeLong(long)

## 다중정의 주의 사항

### 생성자와 다중정의

- 생성자는 이름을 다르게 지을 수 없으니 두번째 생성자부터는 다중정의가 되지만, 대신 정적팩터리를 사용할 수 있다.(아이템 1)

### 오토박싱과 다중정의

- 매개변수 수가 같은 다중정의 메서드가 많더라도, 매개변수중 하나 이상이 "근본적으로 다르다"면 헷갈일 일은 없다. 다중정의 메서드를 호출할지가 매개변수들의 컴파일이 아닌 런타임 타입만으로 결정되기 때문이다.

> "근본적으로 다르다"는 의미
>   - 두 타입의(null이 아닌) 값을 서로 어느 쪽으로든 형변환 불가능한 경우를 의미
>   - ex) Object 외의 클래스 타입과 배열 타입 / Serializable과 Cloneable 외의 인터페이스 타입과 배열 타입 / 상하 관계가 아닌 관련 없는(unrelated) 타입들
>     - (ex)String/Throwable)

- 하지만 자바 5이후로 오토박싱이 도입되고, 기본 타입과 참조 타입이 근본적으로 다름을 보장할 수 없게 되면서 문제가 발생했다.
  - 예시

    ```java
    public class SetList {
      public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list); 
        // [-3, -2, -1] [-2, 0, 2] 
      }
    }
    ```

- 출력이 이상하게 되는 이유는 바로 `List<E>` 인터페이스 가 `remove(Object)`와 `remove(int)`를 다중정의 했기 때문이다.
- `set.remove(i)`의 시그니처는 `remove(Object)`이며, 다중정의된 다른 메서드가 없으니 기대대로 동작한다.
- 하지만 `list.remove(i)`는 다중정의된 `remove(int idex)`를 선택하여 원소가 아닌 "지정한 위치"의 원소를 제거하니, 다른 결과를 내게 되는 것이다. Integer로 형변환하여 올바른 다중정의 메서드를 선택하게 하면 해결된다.
  - 예시

    ```java
    for (int i = 0; i < 3; i++) {
        set.remove(i);
        list.remove((Integer) i);
    }
    ```

### 람다/메서드 참조와 다중정의

- 다중정의 메서드들(혹은 생성자)들이 함수형 인터페이스를 인수로 받을 때, 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다. 달라 보이는 함수형 인터페이스도 근본적으로 다르지 않기 때문이다.
  - 예시

    ```java
    // 1. Thread의 생성자 호출
    new Thread(System.out::println).start();

    // 2. ExecutorService의 submit 메서드 호출
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.submit(System.out::println);
    ```

## 정리

- 일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 것이 좋다.
- 하지만 생성자와 같이 상황에 따라 조언을 따르기 불가능 할 때는, 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야한다.
- 이것 또한 불가능하다면, 기존 클래스를 수정해 새로운 인터페이스를 구현해야 할때는 같은 객체를 입력받는 다중정의 메서드들이 모두 동일하게 동작하도록 해야한다.