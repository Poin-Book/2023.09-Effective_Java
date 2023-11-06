# Item 32 - 제네릭과 가변인수를 함께 쓸 때는 신중해라

> 작성자: 피터
## 목차
가변인수 메서드를 호출하면 가변인수를 담기위한 배열이 자동으로 하나 만들어진다. 
이때 내부로 감춰야할 이 배열이 클라이언트에 노출하는 문제가 생기고 
그 결과 varargs 매개변수에 제네릭이나 매개변수화타입이 포함되면 컴파일 경고가 발생한다. 

실체화 불가 타입은 런타입에는 컴파일타임보다 타입관련 정보를 적게 담고 있다. 
그리고 거의 모든 제네릭과 매개변수화 타입은 실체화되지 않는다. 
그렇기에 메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다. 
가변인수도 마찬가지로 실체화 불가타입으로 추론되면 아래와 같은 경고를 보낸다. 

```java
warning: [unchecked] Possible heap pollution from parameterized vararg type List<String>
```

매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다. 

다음 메소드로 예를 들어보자

```java
static void dangerous(List<String>... stringLists){
	List<Integer> intList = List.of(42);
	Object[] objects = StringLists;        //얕은 복사
	objects[0] = intList;                  //힙 오염 발생
	String s = stringLists[0].get(0);      //ClassCastException
}
```

**`stringLists`** 는 **`List<String>[]`** 타입으로 처리된다. 이 배열을 **`Object[]`** 에 할당한다. 이렇게 하면 제네릭 타입 정보가 손실된다. 즉, 컴파일러는 **`Object[]`** 에 어떤 객체 타입이 저장되었는지 알 수 없으며 모든 객체를 **`Object`** 로 다룬다. 그러나 **`intList`** 는 **`List<Integer>`** 타입의 객체를 담고 있다. 따라서 **`Object`** 배열에 저장될 때 제네릭 타입 정보를 잃게 되고, 힙오염이 발생한다

**`stringLists`** 는 **`List<String>[]`** 이지만, 실제로는 **`Object[]`** 가 되었으며, 이 배열의 첫 번째 요소는 **`intList`** 인 **`List<Integer>`** 로 변환되어 저장되어있다. 따라서 **`stringLists[0]`** 는 **`List<Integer>`** 객체를 가리키게 되고, **`get(0)`** 에서 **`Integer`** 객체를 반환하려고 하므로 **`ClassCastException`** 가 발생하게된다.

이처럼 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

그렇다면 제네릭 배열을 프로그래머가 직접생성하는 건 허용하지 않으면서 제네릭 varags 매개변수를 받는 메서드를 선언할 수 있게 한 이유는 무엇일까

그 이유는 제네릭이나 배개변수 타입의 varags 매개변수를 받는 메서드가 실무에서 상당히 유용하기 때문이다. 따라서 자바에서 이 모순을 수용하고 자바 라이브러리에서도 이런 메소드를 여럿 제공한다.

- Arrays.asList(T… a)
- Collections.addAll(Collection<? super T> c, T… elemets)
- EnumSet.of(E first, E… rest)

이 메서드는 앞서 예시로 들었던 메소드와 달리 안전하다.

자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해 할 수 있는 일이 없었고 그냥 두거나 @SuppressWarnings(”unchecked”) 어노테이션을 달아 경고를 숨겨야 했다.

자바 7에서는 `@SafeVarags` 어노테이션이 추가되어 경고를 숨길수 있게 되었다.

`@SafeVarags` 어노테이션은 메소드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이다. 
컴파일러는 이 약속을 믿고 그 메소드에 대한 경고를 더이상 하지 않는다. 따라서 메소드가 안전하지 않다면 이 어노테이션을 달아서는 안된다. 

메서드가 안전하다는 확신은 어떻게할까?

메서드가 이 배열에 아무것도 저장하지 않고, 그 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다. 즉 이 매개변수배열이 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.

하지만 varargs 매개변수 배열에 아무것도 저장하지 않고도 타입 안정성을 깰수도 있다.

```java
static <T> T[] toArray(T... args){
	return args
}
```

이 메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일타임에 결정되는데 위 함수를 보면 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다. 즉 
**`T... args`** 는 실제로 **`T[] args`** 로 변환되고 **`T[]`** 로 선언한 배열은 실제로 **`Object[]`** 로 컴파일된다. 즉 결과적으로, 제공된 코드는 배열을 사용하여 제네릭 타입을 표현하려고 하지만 타입 정보가 손실되기 때문에 타입 안정성이 깨진다.

구체적인 예를 보자면

```java
static <T> T[] pickTwo(T a, T b, T c){
	switch(ThreadLocalRadom.current().nextInt(3)){
		case 0: return toArray(a,b);
		case 1: return toArray(a,c);
		case 2: return toArray(b,c);
	}
	throw new AssertionError();
}
```

이 메서드를 본 컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varags 매개변수 배열을 만드는 코드를 생성하는데 이때 배열의 타입은 **`Object[]`** 이다. 그리고 toArray가 돌려준 배열이 그대로 클라이언트까지 전달된다. 즉 pickTwo는 항상 **`Object[]`** 타입을 반환한다. 

```java
public static void main(String[] args){
	String[] attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

아무런 경고없이 컴파일 되지만 실제 실행하면 ClassCastException을 던진다. pickTwo변환 값을 attributes 저장하기 위해 String[]로 형변환하는 코드를 컴파일러가 자동 생성하고 Object[]는 Stirng[]의 하위 타입이 아니므로 형변환에 실패한다. 

즉 제네릭 varags 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다

하지만 예외가 두가지 있는데 

1. @SafeVarargs로 제대로 어노테이션된 다른 varags메서드에 넘기는 것은 안전하다.
2. 이 배열 내용의 일부 함수를 호출하는 일반메서드에 넘기는 것도 안전하다. 

다음 예는 제네릭 varags 매개변수를 안전하게 사용하는 메서드이다.

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
	List<T> result = new ArrayList<>();
	for(List<? extends T> list: lists){
		result.addAll(list);
	return result;
}
```

기존 리스트를 변경하지 않고 단순히 새로운 리스트에 담아 전달하는 역할만 하니 안전한 코드이고 @SafeVarargs를 달고 있어 경고또한 표시하지 않는다.

@SafeVarargs는 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 매서드에 달아야한다.또한 SafeVarags 어노테이션은 재정의할 수 없는 메서드에만 달아야한다. final ,private 인스턴스 메소드에 붙일 수 있다.
만약 제네릭 varargs 매개변수를 사용하는 메서드 중 힙 오염 경고가 뜨는 메소드가 있다면 실제로 안전한지 점검을 해야할 것이다. 

다음 두 조건을 만족하는 제네릭 varargs 메서드는 안전하다.

1. varargs 매개변수 배열에 아무것도 저장하지 않는다. 
2. 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다. 

아래와 같이 @SafeVarargs를 달지 않고 varargs 매개변수를 List로 바꿀수 있다.

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
	List<T> result = new ArrayList<>();
	for(List<? extends T> list: lists){
		result.addAll(list);
	return result;
}
```

또한 정적 팩터리 메서드인 List.of를 활용하면 임의개수의 인수를 넘길수도 있다. 

`audience = flatten(List.of(friends, romans, countrymen));`
이 방식의 장점은 컴파일러가 이 메서드의 타입안정성을 검증할 수 있다. 단점이라면 클라이언트 코드가 약간 지저분해지고 속도가 조금 느려질 수 있다.

또한 아까 예시로 들었던 toArray처럼 varargs 메서드를 안전하기 작성하는게 불가능한 상황에서도 쓸 수 있다. 

```java
static <T> List<T> pickTwo(T a, T b, T c){
	switch(ThreadLocalRadom.current().nextInt(3)){
		case 0: return List.of(a,b);
		case 1: return List.of(a,c);
		case 2: return List.of(b,c);
	}
	throw new AssertionError();
}
```

```java
public static void main(String[] args){
	List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

배열 없이 제니릭만 사용하므로 타입 안전하다.
### 정리
가변인수와 제네릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 않고, 배열은 제네릭과 타입 규칙이 서로 다르다. 
하지만 실제로 사용하기에 유용하기에 제네릭 varargs 매개변수는 타입 안전하지 않지만 허용된다. 
메서드에 제네릭 varargs 매개변수를 사용하고자 한다면 타입 안전한지 확인한 후 @SafeVarargs 어노테이션을 달자

### appendix

1. 가변인수 메서드

    기존 메서드의 매개변수 개수가 고정적이었으나 자바 5때 동적으로 지정할 수 있는 기능이 추가 되었으며 이를 variable arguments라고 한다. 
    가변 인자는 `Type… name` 으로 선언한다. 
    가변 인자 외에도 매개변수가 더 있다면 가장 마지막에 선언해야한다.

2. 제네릭
클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법을 의미한다. 
꺽쇠 괄호로 사용한다. 
`List<T>
List<String> list = new ArrayList<>();`
꺾쇠 괄호 안에 식별자 기호를 지정함으로써 파라미터화 할 수 있다.
메소드가 매개변수를 받아 사용하는 것과 비슷하여 제네릭의 타입 매개변수(parameter) / 타입 변수 라고 부른다.
3. 힙오염
JVM의 힙(Heap) 메모리 영역에 저장되어있는 특정 변수(객체)가 불량 데이터를 참조함으로써, 만일 힙에서 데이터를 가져오려고 할때 얘기치 못한 런타임 에러가 발생할 수 있는 오염 상태
