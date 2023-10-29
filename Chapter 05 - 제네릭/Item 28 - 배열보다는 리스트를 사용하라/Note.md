# 배열보다는 리스트를 사용해라

> 작성자: {캐슬}

## 목차
1. 배열과 제네릭 타입의 차이점
2. 배열과 제네릭은 잘 어우러지지 못한다
3. 실체화 불가 타입
4. 배열대신 컬렉션을 사용하자
5. 핵심 정리

# 배열과 제네릭 타입의 중요한 차이점 두가지

## 첫 번째 차이점 (공변 / 불공변)

---

- 배열에서 Sub가 Super의 하위 타입이라면 배열 `Sub[]` 는 배열 `Super[]`의 하위 타입이 된다.(**공변, 함께 변한다**는 뜻)
- **제네릭에서는 불공변**이다. 서로 다른 타입 type1, type2가 있을 때 `List<type1>` 은 `List<type2>` 의 하위타입도 상위 타입도 아니다.

**문법상 허용되는 코드**

*코드 28-1 런타임에 실패*

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다"; //ArrayStoreException을 던진다.
```

**문법에 맞지 않는 코드**

*코드28-2 컴파일 불가*

```java
List<Object> ol = new ArrayList<Long>(); //호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```

두 코드 모두  Long용 저장소에 String을 넣을 수 없다. 이때 배열은 그 실수를 런타임에서 발견하게 되지만, 리스트를 사용하면 컴파일할 때 바로 알 수 있다.

> Java의 Collection들은 컴파일 시간에 제네릭을 통해 타입을 체크하고, 이렇게 타입을 체크하여 런타임에 발생할 수 있는 오류를 미연에 방지할 수 있다.
>

## 두 번째 차이점

---

- **배열은 실체화(reify)**된다**.** → 배열은 **런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인**한다. (28-1 코드에서 Long배열에 String을 넣으려 하면 `ArrayStoreException`이 발생한다.)
- **제네릭은 타입 정보가 런타임에는 소거**된다. → **원소 타입을 컴파일타임에만 검사하며 런타임에는 알 수 없다**.

## 배열과 제네릭은 잘 어우러지지 못한다.

---

**배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.**

- `new List<E> []`, `new List<String>[]`, `new E[]` 식으로 작성하면 컴파일 시에 제너릭 배열 생성 오류 발생

> **제네릭 배열을 만들지 못하게 막은 이유는 타입이 안전하지 않기 때문**이다. 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 `ClassCastException`이 발생할 수 있다. 이는 런타임에 `ClassCastException` 발생을 막아주자는 제너릭 타입 시스템의 취지에 어긋난다.
>

**EX) 제네릭 배열을 생성하는 (1) 이 허용된다면?**

*코드 28-3 제너릭 배열 생성을 허용하지 않는 이유 - 컴파일 되지 않는다.*

```java
List<String>[] stringLists = new List<String>[1]  // (1)
List<Integer> intList = List.of(42);              // (2)
Object[] objects = stringLists;                   // (3)
objects[0] = intList;                             // (4)
String s = stringLists[0].get(0);                 // (5)
```

위 상황에서 (2)는 원소가 하나인 `List<Integer>` 를 생성한다. (3)은 (1)에서 생성한 `List<String>`의 배열을 Object에 할당한다. 배열은 공변이니 아무 문제 없다. (4)는 (2)에서 생성한 `List<Integer>`의 인스턴스를 Object 배열의 첫 원소로 저장한다. 제너릭은 소거 방식으로 구현되어서 이 역시 성공한다. 즉 런타임에는 `List<Integer>` 인스턴스의 타입은 단순히 List가 되고, `List<Integer>[]` 인스턴스 타입은 List[]가 된다. 따라서 (4)에서도 `ArrayStroreException`이 발생하지 않는다.

**여기서 부터 문제가 생긴다.**

> List<String> 인스턴스만 담겠다고 선언한 stringLists 배열에는 지금 List<Integer>이 저장되어 있다. 그리고 (5)에서 이 배열의 처음 리스트에서 첫 원소를 꺼내려한다. 컴파일에서 꺼낸 원소를 자동으로 String으로 형변환하는데 이 원소는 Integer이므로 런타임에 `ClassCastException`이 발생한다.
>

**⇒ 이런 일을 방지하기 위해서는 제너릭 배열이 생성되지 않도록 (1)에서 오류를 발생시켜야 한다.**

## 실체화 불가 타입

---

`E`, `List<E>`, `List<String>` 같은 타입을 실체화 불가 타입(non-reifiable type)이라고 한다.

> **실체화 불가 타입 :** 실체화 되지 않아서 런타임에는 컴파일 타임보다 타입 정보를 적게 가지는 타입
>
- 소거 매커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 `List<?>`와 `Map<?,?>` 같은 비한정적 와일드 카드 타입 뿐이다. → 배열을 비한정적 와일드 카드 타입으로 만들 수는 있지만 유용하게 쓰일 일은 거의 없다.

## @SafeVarargs

---

제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는 게 보통은 불가능하다. 또한 제네릭 타입과 가변인수 메서드를 함께 쓰면 해석하기 어려운 경고메시지를 받게된다. 가변 인수 메서드를 호출할 때마다 가변 인수 매개변수를 담을 배열이 하나 만들어지는데, 이때 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생한다. 이 문제는 `@SafeVarargs` 애너테이션으로 대체할 수 있다.

> **@SafeVarags 는 메서드 작성자가 해당 메서드 타입이 안전하다는 것을 보장하는 장치**
>

## 배열대신 컬렉션을 사용하자

---

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다.

> 코드가 복잡해지고 성능이 나빠질 수 있지만 타입의 안전성과 상호운용성이 좋아진다.
>

EX) 생성자에서 컬랙션을 받는 Chooser 클래스

컬랙션 안의 원소 중 하나를 무작위로 선택해 반환하는 choose 메서드를 제공한다. 생성자에 어떤 컬렉션을 넘기느냐에 따라 이 클래스를 다양한 데이터 소스로 활용할 수 있다.

**제네릭을 쓰지않고 간단하게 구현한 코드**

*코드 28-4 Chooser - 제네릭 적용이 시급하다.*

```java
public class Chooser{
	public final Object[] choiceArray;

	public Chooser(Collection choices){
		choiceArray = choices.toArray();
	}
	public Object choose(){
		Random rnd = ThreadLocalRandom.current()
		return choiceArray[rnd.nextInt(choiceArray.length);
	}
}
```

- 이 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 object를 원하는 타입으로 형변환해야 한다. 혹시 다른 타입의 원소가 들어있다면 런타임에 형변환 오류가 발생한다.

**제네릭으로 변환하기 위한 첫 코드**

*코드 28-5 Chooser를 제네릭으로 만들기 위한 첫 시도 - 컴파일 실패*

```java
public class Chooser<T>{
	private final T[] choiceArray;
	
	public Chooser(Collection<T> choices){
		choiceArray = Choices.toArray();       // 컴파일 오류 발생
	}
	public Object choose(){
		Random rnd = ThreadLocalRandom.current()
		return choiceArray[rnd.nextInt(choiceArray.length);
	}
}
```

- 이 클래스를 컴파일하면 오류가 발생한다. 아래와 같이 Object 배열을 T배열로 형변환 하면된다.

```java
choiceArray = (T[]) choices.toArray();
```

- 코드를 고친 후에는 경고 메시지가 발생한다. (T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다는 메시지)

> **제너릭에서는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입인지 알 수 없음을 기억!!**
>
- 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 사용하면 된다.

**오류나 경고 없이 컴파일 되는 코드**

*코드 28-6 리스트 기반 Chooser - 타입 안정성 확보!*

```java
public class Chooser<T>{
	private final List<T> choiceList;
	
	public Chooser(Collection<T> choices){
		choiceList = new ArrayList<>(choices);
	}
	public T choose(){
		random rnd = ThreadLocalRandom.current();
		return choiceList.get(rnd.nextInt(choiceList.size()));
	}
}
```

## 핵심 정리

---

배열과 제네릭에는 매우 다른 타입 규칙이 적용된다.

배열은 공변이고 실체화 되는 반면, (런타임에는 안전하지만 컴파일러는 그렇지 않다.)

제네릭은 불공변이고 타입 정보가 소거된다. (배열의 반대이다.)

따라서 둘을 섞어 쓰기는 쉽지 않다.

> 둘을 섞어쓰다 컴파일 오류나 경고를 만다면 가장 먼저 배열을 리스트로 대체하는 방법을 적용하자.
>

