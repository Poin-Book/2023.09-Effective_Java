# 이왕이면 제네릭 타입으로 만들라

> 작성자: {루카}

- JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다.

## Object 기반 Stack

- 아래의 Stack 클래스는 원래 제네릭 타입이어야 한다.
  - why? 클라이언트가 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임에 오류가 날 수 있기 때문이다.

```java
public class Stack {
  private Object[] elements;
  private mnt size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
  }

  public boolean isEmpty() {
    return size == 0;
  }

  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);
  }
}
```

### 일반 클래스 to 제네릭 클래스

- 일반 클래스를 제네릭 클래스로 만드는 첫 단계는 클래스 선언에 타입 매개변수를 추가하는 것이다.
  - 이때 타입 매개변수의 이름은 관례에 따라 대문자 알파벳 한 글자로 짓는다.
  - 보통은 E, T, S, U, V 등을 사용한다.
  - 지금은 E를 사용할 것이며, E와 같은 실체화 불가 타입으로는 `배열을 만들 수 없다.`

```java
public class Stack<E> {
  // Object[] -> E[]
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    // Object -> E
    elements = new E[DEFAULT_INITIAL_CAPACITY]; // 컴파일 오류 발생
  }

  // Object -> E
  public void push(E e) {
    ensureCapacity();
    elements[size++] = e;
  }

  // Object -> E
  public E pop() {
    if (size == 0)
      throw new EmptyStackException();

    // Object -> E
    E result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
  }

  ...
}
```

## 배열 with 제네릭

배열을 사용하는 코드를 제네릭으로 만들 때 문제가 발생하게 되고, 이를 해결하기 위해 두 가지 방법이 있다.

### 1. 제네릭 배열 생성을 금지하는 제약 우회

- 컴파일러는 오류 대신 경고를 내보낼 것이다.
- 타입 안전하지 않다는 경고
- 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음 스스로 확인해야 한다.
- 비검사 형변환이 안전함을 직접 증명했다면 범위를 최소로 좁혀 @SuppressWarnings 어노테이션으로 해당 경고를 숨긴다.
  - 생성자가 비검사 배열 생성 말고는 하는 일이 없으니 생성자 전체에서 경고를 숨겨도 좋다.
- 어노테이션을 달면 Stack을 깔끔하게 컴파일 되고, 명시적으로 형변환하지 않아도 ClassCastException 걱정 없이 사용할 수 있게 된다.

```java
public class Stack<E> {
    // ...

    // 배열 elements 는 push(E)로 넘어온 E 인스턴스만 담는다.
    // 따라서 타입 안전성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]이다.
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
    // ...
}
```

### 2. elements 필드의 타입을 E[]에서 Object[]로 변경

- 첫 번째와는 다른 오류가 발생한다.
- 배열이 반환한 원소를 E로 형변환하면 오류 대신 경고가 뜬다.
  - E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.
- pop 메서드 전체에서 경고를 숨기지 말고, 비검사 형병환을 수행하는 할당문에서만 숨긴다.

```java
public class Stack<E> {
  ...

    // 비검사 경고를 적절히 숨긴다.
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        @SuppressWarnings("unchecked")
        E result = (E) elements[--size];

        elements[size] = null;
        return result;
    }
    ...
}
```

### 두 가지 방법의 차이

1. 제네릭 배열 생성을 금지하는 제약 우회
   - 가독성이 좋다.
   - 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실하게 명시한다.
   - 형변환을 배열 생성 시 단 한번만 해주면 된다.
   - 힙 오염을 이르킨다.
     - 배열의 런타임 타입이 컴파일 타임 타입과 다르기 때문

2. elements 필드의 타입을 E[]에서 Object[]로 변경
   - 배열에서 원소를 읽을 때 마다 형변환을 해줘야 한다.
     - 현업에서는 첫 번째 방식을 더 선호하며 자주 이용한다.
   - 힙 오염을 이르키지 않는다.
     - 힙 오염이 걱정 된다면 두 번째 방식을 사용한다.

## 제네릭 Stack을 사용하는 예시

- 명령줄 인수들을 역순으로 바꿔 대문자로 출력하는 프로그램
  - Stack 에서 꺼낸 원소에서 String의 toUpperCase 메서드를 호출할 때 명시적 형변환을 수행하지 않는다.
    - 컴파일러에 의해 자동 생성
  - 이는 형병환이 항상 성공함을 보장한다.

```java
public class Stack<T> {
  public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for (String arg : args)
      stack.push(arg);
    while (!stack.isEmpty())
      System.out.println(stack.pop().toUpperCase());
  }
}
```

- "배열보다는 리스트를 우선하라"는 아이템 28과 모순되어 보인다.
  - 사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다.
  - 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국 기본 타입인 배열을 사용해 구현해야 한다. 또한 HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

- Stack 예처럼 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다.
  - Stack<Object>, Stack<int[]>, Stack<List<String>>, Stack 등 어떤 참조 타입으로도 Stack을 만들 수 있다.
    - 단, 기본 타입은 사용할 수 없다.
    - Stack<int>나 Stack<double>을 만들려고 하면 컴파일 오류가 난다.
  - 이는 자바 제네릭 타입 시스템의 근본적인 문제이나, 박싱된 기본타입을 사용해 우회할 수 있다.

- 타입 매개변수에 제약을 두는 제네릭 타입도 있다.
  - java.util.current.DelayQueue
  - 타입 매개변수 목록인 는 java.util.concurrent.Delayed 의 하위 타입만 받는다는 것이다.
  - DelayQueue 자신과 DelayQueue를 사용하는 클라이언트는 DelayQueue의 원소에서 곧바로 Delayed 클래스의 메서드를 호출할 수 있다.
  - ClassCastException 걱정은 할 필요가 없다.
  - 이러한 타입 매개변수 E를 한정적 타입 매개변수(bounded type parameter)라 한다.
  - 모든 타입은 자기 자신의 하위 타입이므로 DelayQueue로도 사용할 수 있다.

## 정리

- 클래이언트에서 직접 형변환해야 하는 타입보다는 제네릭 타입이 더 안전하며 쓰기도 쉽다.
  - 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라.
  - 그런 경우 제네릭 타음으로 만들어야 하는 경우가 많다.
  - 기존 타입 중 제네릭이었어야하는 부분이 있다면 변경하자.
  - 기존 클라이언트에는 아무 영향을 끼치지 않으면서, 새로운 사용자에게는 좋은 API를 제공할 수 있다.