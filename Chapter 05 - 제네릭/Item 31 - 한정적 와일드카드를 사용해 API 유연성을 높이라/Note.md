# Item 31 - 한정적 와일드카드를 사용해 API 유연성을 높이라

> 작성자: 워니

## 목차
#### 한정적 와일드카드의 사용으로 유연성 높이기
#### 상위 객체에 대한 유연성을 늘리는 와일드카드 타입
#### PECS
#### 조금 더 복잡한 한정적 와일드카드 타입
#### 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라
#### 정리

---

>[Item 28. 배열보다는 리스트를 사용하라]에서 언급했듯, 매개변수화 타입은 불공변(invariant)이다.  
이를 좀 더 풀어보자면, `List<String>`은 `List<Object>`가 하는 일을 제대로 수행하지 못하므로 
`List<String>`은 `List<Object>`의 하위 타입이 아니라는 의미가 된다.  

❓ 이렇게 매개변수화 타입은 불공변이지만, public API처럼 유연한 방식으로 설계하고 싶다면 어떻게 해야할까? 
 
### 한정적 와일드카드의 사용으로 유연성 높이기

- 와일드카드 타입을 사용하지 않은 코드

``` java
public class Stack<E> {
	...
	public void pushAll(Iterable<E> src) {
		for(E e : src) push(e);
	}
}
```
> Stack 클래스에 일련의 원소를 스택에 넣는 pushAll() 메서드를 추가

pushAll 메서드는 잘 컴파일 되고 동작도 되지만, 유연성이 많이 떨어진다. 

- pushAll() 메서드의 결함

```java
Stack<Number> stack = new Stack<>();
List<Integer> integers = Arrays.asList(1, 2, 3, 4);

//java: incompatible types: java.util.List<java.lang.Integer> 
//      cannot be converted to java.lang.Iterable<java.lang.Number>
stack.pushAll(integers); 
```

Integer는 Number의 하위 타입이니 논리적으로 문제가 없어야 할 것 같지만 실제로는 컴파일 에러가 뜬다.  
매개변수화 타입이 불공변이라서 Number 타입으로 변환할 수 없기 때문이다. 
Iterable의 원소 타입이 스택의 원소 타입과 일치하면 잘 작동하지만 타입이 다를 경우 매개변수화 타입이 불공변이기 때문에 컴파일 에러가 발생하게 되는 것이다.  
즉, 매개변수화 타입 자체를 독립적으로 보면 하위 객체이고 공변이 되지만 매개변수화 타입에서는 불공변이기에 서로 호환성이 떨어지게 된다.   

➡️ 그래서 이런 경우 **한정적 와일드카드 타입**이라는 특별한 매개변수화 타입으로 문제를 해결할 수 있다!

- 한정적 와일드카드 타입을 적용한 코드

``` java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) push(e);
}
```

매개변수의 타입을 `Iterable<E>` 에서 `Iterable<? extends E>` 으로 와일드카드 타입을 적용해준 뒤 위의 에러가 났던 코드를 다시 실행시켜보면 이제 문제없이 동작할 것이다.  
이는, pushAll() 입력 매개변수 타입은 `E의 Iterable`이 아니라 `E의 하위타입의 Iterable`이라고 변경해주어 타입을 안전하게 사용할 수 있게 됨을 의미한다. 

-----

### 상위 객체에 대한 유연성을 늘리는 와일드카드 타입
> 하위 객체에 대한 유연성을 한정적 와일드카드 타입(`<? extends E>`)로 해결을 했다. 

❓ 그렇다면 상위 객체에 대한 유연성은 어떻게 확보해야 할까?

사실 이 질문에 대한 답변은 하나의 키워드로 해결할 수 있다.  
💡 **하위 객체로 확장이 extends라면, 상위 객체로의 확장은 super다.**  

<br>

- Stack 객체에서 모든 값을 꺼내는 popAll 메서드

``` java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) dst.add(pop());
}

...
public static void main(String[] args) {
    Stack<Number> stack = new Stack<>();
    List<Integer> integers = Arrays.asList(1, 2, 3, 4);

    stack.pushAll(integers);

    Collection<Object> objects = new ArrayList<>();
    stack.popAll(objects);

    System.out.println("objects = " + objects);
}
```

popAll() 메서드의 매개변수 타입으로 와일드카드 타입을 적용했다.  
이는, 매개변수 Collection의 매개변수화 타입은 `E의 상위 타입`이어야 한다는 의미이다.  
그래서 Number의 상위 타입인 Object로 매개변수화 타입을 작성해도 에러가 나지 않고 동작한다. 

-----

### PECS

- 이렇게 하위 타입, 상위 타입에 대한 유연성을 높이는 방법들은 모두 하나의 메세지를 던진다.  

>💡 유연성을 극대화 하기 위해 원소의 생산자나 소비자용 입력 매개변수에 **와일드카드 타입을 사용**하라.

(물론, 매개변수가 생산자와 소비자 역할을 동시에 한다면 타입을 정확히 지정해야 하기에 와일드카드 타입 사용을 하면 안 된다.)

<br>

- 언제, 어떤 와일드 카드 타입을 사용해야 하는지는 공식을 통해 쉽게 기억할 수 있다.  

>💡 **PECS: producer-extends, consumer-super**

→ 즉, 생산자라면 extends로 하위 타입 유연성을 높이고 소비자라면 super로 상위 타입 유연성을 높인다.  
앞서 사용했던 Stack 코드로 다시 예를 들면, 
1) pushAll() 메서드에서 매개변수(src)는 생산자로 사용되므로, 하위 호환 유연성을 높는 <? extends E> 와일드 카드 타입이 적절하고 
2) popAll() 메서드의 매개변수(dst)는 Stack이라는 객체로부터 E 인스턴스를 소비하므로 <? super E>가 적절하다. 

<i> (나프탈리(Naftalin)과 와들러(Wadler)는 이를 겟풋원칙(Get and Put Priciple)이라 부른다.)</i>

- 참고) 반환 타입은 한정적 와일드카드 타입을 사용하면 안 된다.  
→ 유연성을 높여주기는 커녕 클라이언트 코드에서도 와일드카드 타입을 써야하기 때문이다.  
클라이언트가 와일드카드 타입을 신경써야 하는 상황이 오면 그 API에 문제가 있을 가능성이 크다.

----

### 조금 더 복잡한 한정적 와일드카드 타입
- 하나의 와일드카드 타입이 아닌 두 개 이상의 와일드카드 타입을 사용하기

> 아래의 max 메서드는 아이템 29에서 작성했던 코드이다.

```java
public static <E extends Comparable<E>> Optional<E> max(List<E> list){...}
```

- PECS 공식을 한 번 더 적용한 코드 
``` java
public static <E extends Comparable<? super E>> Optional<E> max(List<? extends E> list){...}
```

변경된 부분은 다음과 같다.  
(1) 입력 매개변수는 E 인스턴스를 생산하므로 원래의 `List<E>`를 `List<? extends E>`로 수정했다.  
(2) 원래 선언 쪽 E는 `Comparable<E>`를 확장한다고 했는데, 여기서 `Comparable<E>`는 E 인스턴스를 소비한다. 따라서 `Comparable<? super E>`로 변환했다.  
> <i> Comparable은 언제나 소비자이기에 보통 `Comparable<E>`보단 `Comparable<? super E>`가 낫다. </i>

-----

### 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라
❓ 메서드를 정의할 때 매개변수의 타입으로 타입 매개변수와 와일드카드 중 무엇을 선택해야 할까?  
→ 타입 매개변수와 와일드카드에는 공통되는 부분들이 있어서 둘 중 무엇을 사용해도 괜찮은 경우가 많다. 

``` java
public static <E> void swap(List<E> list, int i, int j); // 타입 매개변수
public static void swap(List<?> list, int i, int j); // 와일드카드
```

- 둘 중 어떤 선언이 더 나을까? 그리고 더 나은 이유는 무엇일까?
 
public API라면 간단한 두 번째가 낫다.  
어떤 리스트든 이 메서드에 리스트를 넘기면 인덱스의 원소들을 교환해준다. 이때, 우리가 신경써야 할 타입 매개변수도 없다.  

<br>

기본 규칙은 다음과 같다.  
>💡 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.

또한 위 조건을 충족한다면 다음과 같은 타입도 바꿔주면 된다.  
>▪️ 비한정적 타입 매개변수 ➡️ 비한정적 와일드카드  
▪️ 한정적 타입 매개변수 ➡️ 한정적 와일드카드  

<br>

- 두번째 방법인 와일드카드 방식의 결함 
``` java
public static void swap(List<?> list, int i, int j) {
		list.set(i, list.set(j, list.get(i)));
}
```

이 코드는 문제가 없어 보이지만 컴파일 시 오류가 발생한다.  
그 이유는 List<?>에는 null 외에는 어떤 값도 넣을 수 없기 때문이다.  

- 이를 해결하기 위해 와일드카드 타입의 실제 타입을 알려주는 private helper method 작성

``` java
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

헬퍼 메서드인 swapHelper는 List<E> 매개변수로 이 리스트에서 꺼낸 값이 항상 E라는 것을 알고 있다.  
그래서 set으로 List에서 값을 꺼내 다시 넣는 과정도 문제 없이 수행될 수 있다. 

➡️ 와일드카드 방식을 사용하면 이렇게 약간 복잡한 우회 방법(제네릭 메서드)을 사용해야 하지만, 
덕분에 외부에서는 와일드카드 기반의 메서드를 유지할 수 있고 해당 메서드의 로직 내부에서 swapHelper라는 헬퍼 메서드의 동작에 대해서도 알 필요가 없다. 

---

### 📌 정리
- 와일드카드 타입을 적용하면 API가 유연해진다. 
- PECS: Producer - Extends, Consumer - Super
- Comparable과 Comparator는 모두 소비자다. 
- 메서드 선언 타입에 타입 매개변수가 한 번만 등장하면 와일드카드를 사용하자. 
