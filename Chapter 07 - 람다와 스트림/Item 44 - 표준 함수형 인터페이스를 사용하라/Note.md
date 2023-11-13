# 표준 함수형 인터페이스를 사용하라

> 작성자: 워니

## 목차
**1. 람다의 등장으로 바뀐 개발 방향  
2. 표준 함수형 인터페이스  
3. 직접 함수형 인터페이스를 작성해야 하는 경우  
4. 📌 요약**

---

### 람다의 등장으로 바뀐 개발 방향
자바 8에 람다가 등장하며 상위 클래스의 기본 메서드를 재정의하는 템플릿 메서드 패턴의 가치가 많이 떨어졌다. 인수로 함수 객체를 전달하면 되기 때문이다.  
즉, 함수 객체를 받는 정적 팩토리나 생성자를 사용하면 되기 때문에 템플릿 메서드 패턴의 효용이 떨어졌다.  
</br>

> ### 💡 `LinkedHashMap`을 예로 들어보자. 

`removeEldestEntry()` 함수는 맵에 새로운 키를 추가할 때의 `put()` 메서드에 의해 호출되는데, 해당 메서드가 `true`를 반환하면 맵에서 가장 오래된 원소를 제거한다.

#### * 람다를 사용하지 않은 경우
``` java
protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
	return size() > 100; // size: 호출된 맵 안의 원소 수를 알아냄
}
```

`removeEldestEntry()`는 인스턴스 메서드라서 바로 `size()` 호출을 통해 맵 안의 원소 수를 알아낼 수 있다.

하지만 만약 함수 객체로 구현한다면, 생성자에 넘기는 함수 객체는 맵의 인스턴스 메서드가 아니기 때문에 자기 자신도 함수 객체에 건네줘야 한다.  
이를 반영하여 함수 객체를 받는 정적 팩터리나 생성자로 구현한다면, 다음과 같다.

#### * 람다를 사용한 경우 - 함수형 인터페이스
``` java
@FunctionalInterface
interface EldestEntryRemovalFunction<K, V> {
	boolean remove(Map<K, V> map, Map.Entry<K, V> eldest); 
}
```

이처럼 함수형 인터페이스를 선언하면 이를 하나의 인수로 취급해 함수 객체로 전달할 수 있다.  
하지만, 굳이 이런 방식을 사용할 이유는 없다. 대부분의 경우 자바 표준 라이브러리에서 함수형 인터페이스를 제공하기 때문이다.

#### * 표준 함수형 인터페이스를 사용한 경우
``` java
BiPredicate<Map<K, V>, Map.Entry<K, V>>
```
``` java
BiPredicate<Map<String, String>, Map.Entry<String, String>> predicate = (k, v) -> k.size() > 100;
```

필요한 용도에 맞는게 있다면 직접 구현하지 말고 표준 함수형 인터페이스를 활용하자!

---

### 표준 함수형 인터페이스
`java.util.function` 패키지 내부에는 다양한 용도의 표준 함수형 인터페이스가 담겨있다. 
그래서 굳이 직접 함수형 인터페이스를 구현하기보단 제공되는 표준 함수형 인터페이스를 활용하는게 좋다.  
표준이기에 다른 개발자와의 소통도 쉽고, API를 다루는 개념의 숫자도 줄어들어서 익히기도 쉽다.  
_(같은 용도의 함수형 인터페이스를 개발자가 필요할 때마다 매번 새로 만든다면 알아야 할 함수형 인터페이스만 늘어나 혼란이 커질 것이다.)_

또한 이러한 표준 함수형 인터페이스는 유용한 디폴트 메서드들도 많이 제공하기에 사용하기에 편하다. 
> ex). `Predicate` 인터페이스는 `predicate`들을 조합하는 메서드를 제공한다.
</br>

`java.util.function` 패키지에는 총 43개의 인터페이스가 담겨져 있는데, 전부 외울 필요는 없다.  
기본 인터페이스 6개만 기억한다면, 나머지 인터페이스는 대부분 이름의 유사성이나 기본 인터페이스명과 비슷하게 특징을 넣은 이름이기에 사용하기 어렵지 않다. 

### 📌 기본 함수형 인터페이스 정리 (표)

| 인터페이스 | 함수 시그니처 | 의미 |  예 |
| - | - | - | - |
| `UnaryOperator<T>` | `T apply(T t)` | 반환 값과 인수의 타입이 같은 함수, 인수는 1개 | `String::toLowerCase` |
|`BinaryOperator<T>` | `T apply(T t1, T t2)` | 반환 값과 인수의 타입이 같은 함수, 인수는 2개 | `BigInteger::add` |
| `Predicate<T>` | `boolean test(T t)` | 인수 하나를 받아 boolean을 반환하는 함수 | `Collection::isEmpty` |
| `Function<T, R>` | `R apply(T t)` | 인수와 반환 타입이 다른 함수 | `Arrays::asList` |
| `Supplier<T>` | `T get()` | 인수를 받지 않고 값을 반환(혹은 제공)하는 함수 | `Instant::now` |
| `Consumer<T>` | `void accept(T t)` | 인수 하나 받고 반환 값은 없는 함수 | `System.out::println` |

- **Operator**: 인수 타입과 반환 타입이 동일한 인터페이스
- **UnaryOperator**: 인수가 1개인 인터페이스
- **BinaryOperator**: 인수가 2개인 인터페이스
- **Predicate**: 인수 하나를 받아 `boolean`을 반환하는 인터페이스
- **Function**: 인수와 반환 타입이 다른 인터페이스
- **Supplier**: 인수를 받지 않고 값을 반환(혹은 제공)하는 인터페이스
- **Consumer**: 인수를 하나 받고 반환값은 없는(특히 인수를 소비하는) 인터페이스
</br>

이런 기본 인터페이스에서 `int`, `long`, `double` 용으로 각 3개씩 변형이 생기는데 접두사로 해당 타입이 붙는다. 
- **Predicate** → `IntPredicate`, `LongPredicate` , `DoublePredicate`
- **BinaryOperator** → `IntBinaryOperator`, `LongBinaryOperator`, `DoubleBinaryOperator`

이러한 변형된 함수형 인터페이스는 제공할 인수 타입이 기본 타입(`int`, `long`, `double`)일 때 좀 더 편하게 사용하도록 명시된 인터페이스로, 
실제 타입 매개변수를 작성해야 하는 수고를 덜어준다. 

##### ex). 숫자가 짝수인지 검증하는 함수형 인터페이스
``` java
// before
Predicate<Integer> isEvenV1 = value -> value % 2 == 0;

// after
IntPredicate isEvenV2 = value -> value % 2 == 0;
```
</br>

그리고 그 밖에 다음과 같은 변형 인터페이스가 있다. 

#### ➕ 인수 타입을 두 개(혹은 세 개)씩 받도록 변형된 함수형 인터페이스  
인수 개수가 하나 혹은 두개로 부족한 경우를 대비하기 위해 Bi라는 접두사가 붙은 인수 개수를 늘린 세 가지 인터페이스를 제공한다. 

- Predicate<T> → BiPredicate<T, U>
- Function<T, R> → BiFunction<T, U, R>
- Consumer<T> → BiConsumer<T, U>

#### ➕ 기본 타입을 반환하는 BiFunction 변형 모델
BiFunction 인터페이스가 만약 기본 타입(`int`, `long`, `double`)을 반환한다면 이를 지원하는 변형 인터페이스가 또 있다. 

- Integer 타입을 반환하는 경우 → ToIntBiFunction<T, U>  
- Long 타입을 반환하는 경우 → ToLongBiFunction<T, U>  
- Double 타입을 반환하는 경우 → ToDoubleBiFunction<T, U>

#### ➕ 기본 타입을 받는 BiConsumer 변형 모델
인수 개수를 하나가 아닌 두 개를 받도록 한 BiConsumer 인터페이스에서 하나의 인수 타입이 기본 타입인 경우 이를 지원하는 변형 모델도 있다. 

- Integer 타입을 받는 경우 → ObjIntConsumer<T>  
- Long 타입을 받는 경우 → ObjLongConsumer<T>  
- Double 타입을 받는 경우 → ObjDoubleConsumer<T>

#### ➕ Boolean 타입을 반환하는 Supplier
인수를 받지 않고 값을 반환 혹은 제공하는 Supplier에서 만약 반환 타입이 Boolean일 경우 이를 따로 실제 타입 매개변수를 작성하지 않아도 되도록 **BooleanSupplier**라는 인터페이스를 제공한다. 
- `Supplier<Boolean>` → BooleanSupplier
</br>

#### _암튼 모두 외울 필요는 없다._  
`java.util.function`의 기본적인 6가지 표준 함수형 인터페이스와 여기서 편의를 위해 변형할 수 있는 모델에 대해서는 변형된 인터페이스를 만들어 제공해주고 있다.  
그렇게 만들어진 함수형 인터페이스가 43개인데, 모두 외울 필요도 없고 이름이 명시적이기에 필요할때마다 찾아서 써도 어렵지 않다. 

_**다만,**_ 표준 함수형 인터페이스는 대부분 기본 타입만 지원한다는 점은 기억해야 한다. 
물론 래퍼 클래스(Wrapper Class)로 사용을 해도 동작은 하지만 계산량이 많아질수록 성능이 처참히 떨어진다. 

---

### 직접 함수형 인터페이스를 작성해야 하는 경우
그렇다고 항상 기본으로 제공해주는 함수형 인터페이스로 모든 상황을 헤쳐나갈 수 있는 것은 아니다.  
예를 들어, 매개변수의 개수가 3개를 넘는 `Predicate`는 `java.util.function`에서 제공하지 않는다.  
그리고 구조적으로 똑같은 표준 함수형 인터페이스가 이미 제공되고 있더라도 직접 작성해야 하는 경우도 있다. 
<br></br>

> #### 💡 Comparator<T>

비교를 하기 위해 제공되는 이 인터페이스는 로직만 파악하자면 `ToIntBiFunction<T, U>`와 같다.  
그럼에도 불구하고 `Comparator<T>`를 사용해야만 하는 이유가 있는데 그 이유는 다음과 같다. 

- 사용 빈도가 높으며 이름 자체로 용도를 명확히 설명해준다. 
- 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다. 
- 비교자들을 변환하고 조합해주는 유용한 디폴트 메세드를 가지고 있다.

``` java
// Comparator
@FunctionInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}

// ToIntBiFunction
@FunctionalInterface
public interface ToIntBiFunction<T, U> {
    int applyAsInt(T t, U u);
}
```
</br>

> #### 💡 @FunctionalInterface

이 애노테이션은 인터페이스가 함수형 인터페이스로 사용됨을 알려주는 애노테이션으로, 자체적으로 어떤 특별한 동작을 수행하는 것은 아니다. 그렇기에 해당 애노테이션이 없어도 인터페이스가 함수형으로 하나의 추상 메서드만을 가진다면 정상적으로 동작을 한다.  
하지만 이 애노테이션을 사용하는 이유는 `@Override`를 사용하는 것과 같다.  
프로그래머의 의도를 명시하는 것으로 다음과 같은 목적이 있다.  

- 사용자에게 이 인터페이스가 람다용으로 설계된 것임을 알려준다. 
- 추상 메서드가 하나만 있어야 함을 컴파일러에게 알려줘 추상 메서드가 하나일 때 컴파일이 된다.
- 차후 유지보수 과정에서 다른 개발자가 실수로라도 메서드 추가를 하지 못하도록 막아준다.

➡️ 이런 이유로 함수형 인터페이스를 직접 만들 때는 항상 @FunctionalInterface 애노테이션을 사용하자.

#### 주의사항  
서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의 해서는 안 된다. 
클라이언트에게 불필요한 모호함을 안겨주고 이런 모호함으로 문제가 발생하기도 쉽다.

##### ex).
``` java
public interface ExecutorService extends Executor {
    // Callable<T>와 Runnable을 각각 인수로 받게 Overloading
    // submit 메서드를 사용할 때마다 형변환 필요
    <T> Future<T> submit(Callback<T> task);
    Future<?> submit(Runnable task);
}
```

---

### 📌 요약
- 자바 8 이후 람다가 등장했으며, 개발에 람다를 고려하는 것이 좋다. 
- 대부분의 경우, 함수형 인터페이스를 구현하기보다 표준 함수형 인터페이스를 사용하는게 좋다.  
> `java.util.function` 패키지에서 제공하는 인터페이스
- 표준 함수형 인터페이스 대부분은 기본 타입만 지원한다.  
> 기본 함수형 인터페이스에 박싱된 기본 타입(래퍼 클래스)를 사용하지 말자. 
- 다음과 같은 경우에만 함수형 인터페이스 설계를 고려하라.
> - 자주 사용되며, 이름 자체가 용도를 명확하게 설명하는 경우
> - 반드시 따라야 하는 규약이 있는 경우
> - 유용한 디폴트 메서드를 제공할 수 있는 경우 
- 직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애너테이션을 사용하라.
- 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의하지 말아라.
