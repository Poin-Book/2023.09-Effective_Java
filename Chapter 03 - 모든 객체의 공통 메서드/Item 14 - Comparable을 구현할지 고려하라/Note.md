# Comparable을 구현할지 고려하라

> 작성자: 캐슬

## 목차
- Comparable의 compareTo?
- compareTo 규약
- compareTo 작성 요령
- 정적 compare 메서드를 활용한 비교자
- 비교자 생성 메서드를 활용한 비교자

들어가며,

compareTo는 Object의 메서드가 아니다.

성격은 두가지만 빼면 Object의 equals와 같다.

**다른점**

comapreTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.

<aside>
💡 제네릭? <br>
데이터 형식에 의존하지 않고, 하나의 값이 여러 다른 데이터 타입들을 가질 수 있도록 한다.

</aside>

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다. 따라서 Comparable을 구현한 객체들의 배열은 아래와 같이 손쉽게 정렬할 수 있다.

```java
Arrays.sort(a);
```

검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게할 수 있다.

아래 프로그램은 String이 Comparable을 구현해서 명령줄 인수들을 (중복제거)알파벳순으로 출력한다.

```java
public class WordList{
	public static void main(String[] args){
		Set<String> s = new TressSet<>();
		Collections.addAll(s, args);
		System.out.println(s);
	}
}
```

Comparable을 구현하여 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있다.

(자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현했다.)

**알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable을 구현하자.**

```java
public interface Comparable<T>{
	int compareTo(T t);
}
```

compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다.

> 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같다면 0을, 크면 양의 정수를 반환한다. 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
>

모든 객체에 대해 전역 동치관계를 부여하는 equals메서드와 달리, compareTo는 타입이 다른 객체를 신경 쓰지 않아도된다. 타입이 다른 객체가 주어지면 ClassCastException을 던져도 되기 때문이다.

### compareTo 규약

첫 번째 규약 : 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다. 즉 첫 번째 객체가 두 번째 객체보다 작으면 두 번째 객체가 첫 번째 객체보다 커야한다.

두 번째 규약 : 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세번째 보다 커야한다는 것이다.

세 번째 규약 : 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 크기가 같아야 한다.

이 세 규약은 compareTo 메서드로 수행하는 동치성 검사도 eqauls 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 한다는 것을 뜻한다.

주의사항

기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다.

⇒ 우회법 : Comparable 을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두어야 한다. 그 후에 내부 인스턴스를 반환하는 ‘뷰’ 메서드를 제공하면 된다.

### compareTo 메서드 작성 요령

compareTo 작성요령은 equals와 비슷하지만 몇가지 차이점이 존재한다.

1. Comparable은 타입을 인수로 받는 제너릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다. (입력 인수의 타입을 확인하거나 형변환할 필요가 없다. ⇒ 인수의 타입이 다르면 컴파일이 되지 않기 때문)
2. null을 인수로 넣어 호출하면 NullPointerException을 던져야한다.
3. compareTo는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교한다.
4. 객체 참조 필드를 비교하려면 compareTo를 재귀적으로 호출한다.
5. Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야한다면 비교자(Comparator)를 대신 사용한다.
6. 비교자는 직접 만들거나 자바가 제공한 것 중에 골라 사용한다.

### 정적 compare 메서드를 활용한 비교자

코드 - 객체 참조 필드가 하나뿐인 비교자

```java
public final class CaseInsensitiveString 
		implements Comparable<CaseInsensitiveString> {
	public int compareTo(CaseInsensitiveString cis) {
		return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
	}
}
```

- 자바가 제공하는 비교자를 사용하고 있다.
- 자바 7 부터는 박싱된 기본 타입 클래스에 새로 추가된 정적 메서드인 compare을 이용하면 된다.

클래스에 핵심 필드가 여러개라면 어느 것을 먼저 비교하느냐가 중요해진다.

1. 가장 핵심 필드부터 비교해나가자.
2. 비교 결과가 0이 아니라면, 즉 순서가 결정되면 그 결과를 곧장 반환하자.
3. 가장 핵심이 되는 필드가 똑같다면 그다음으로 중요한 필드를 비교해 나가자.

코드 - 기본 타입 필드가 여럿일 때의 비교자

```java
public int compareTo(PhoneNumber pn){
	int result = Short.compare(areaCode, pn.areaCode); // 가장 핵심 필드
	if (result ==0) {
		result = Short.compare(prefix, pn.prefix);       // 두 번째로 중요한 필드
		if (result == 0)
			result = Shoret.compare(lineNum, pn.lineNum);  // 세 번째로 중요한 필드
	}
	return result;
}
```

### 비교자 생성 메서드를 활용한 비교자

자바 8에서는 Comparator 인터페이스가 일련의 비교 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다. 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데 활용할 수 있다.

장점은 간결함.

단점은 약간의 성능저하.

코드 - 비교자 생성 메서드를 활용한 비교자

```java
public static final Comparator<PhoneNumber> COMPARATOR = 
				comparingInt(PhoneNumber pn) -> pn.areaCode)
					.thenComparingInt(pn -> pn.prefix)
					.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn){
	return COMPARATOR.compare(this, pn);
}
```

이 코드는 클래스를 초기화할 때 비교자 생성 메서드 2개를 이용해 비교자를 생성한다.

그 첫번째인 comparingInt는 객체 참조를 int 타입키에 맵핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드이다.

앞의 예에서 comparingInt는 람다를 인수로 받으며, 이 람다는 PhoneNumber에서 추출한 지역 코드를 기준으로 전화번호의 순서를 정하는 Comparator<PhoneNumber> 를 반환한다. 여기서 인수의 타입(PhoneNumber pn) 을 명시한 이유는 자바의 타입 추론 능력이 타입을 알아낼만큼 강력하지 않기에 컴파일 되도록 도와준 것이다.

두 전화번호의 지역 코드가 같을 수 있으니 비교방식을 더 추가해야한다.

해당 작업은 두 번째 비교 생성 메서드인 thenComparingInt가 수행한다. thenComparingInt는 Comparator의 인스턴스 메서드로, int 키 추출자 함수를 입력 받아 다시 비교자를 반환한다. 해당 메서드는 원하는 만큼 호출할 수 있다. 여기서는 인수의 타입을 명시하지 않아도 자바가 타입추론이 가능하다.

### Comparator의 보조 생성자

Comparator는 많은 보조 생성 메서드들을 가지고 있다. long, double 용으로는 comparingInt와 thenComparingInt의 변형 메서드를 가지고 있다. shor처럼 더 작은 정수 타입에는 int형 버전을 사용하면 된다. 마찬가지로 float는 double형 용을 이용해 수행한다.따라서 자바의 모든 숫자용 기본타입을 커버할 수 있다.

### 객체 참조용 비교자 생성 메서드

comparing이라는 정적 메서드 2개가 다중 정의되어있다.

- 첫 번째는 키 추출자를 받아서 그 키의 자연적 순서를 사용한다.
- 두 번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받는다.

또한 thenComparing이라는 인스턴스 메서드가 3개 다중정의 되어 있다.

- 첫 번째는 비교자 하나만 인수로 받아 그 비교자로 부차 순서를 정한다.
- 두 번재는 키 추출자를 인수로 받아 그 키의 자연적 순서로 보조순서를 정한다.
- 세 번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 만든다.

### 사용하면 안되는 예시

**코드 - 해시코드 값의 차를 기준으로 하는 비교자 - 추이성을 위배한다!**

```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
	public int compare(Object o1, Object o2){
		return o1.hashCode() - o2.hashCode();
	}
}
```

이 방식은 정수 오버플로를 일으키거나 IEEE 754 부동 소수점 계산 방식에 따른 오류를 낼 수 있다. 그렇다고 속도가 월등히 빠르지도 않다.

### 결론 : 두 방식 중 하나를 사용하자.

코드 - 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
	public int compare(Object o1, Object o2){
		return Integer.compare(o1.hashCode() - o2.hashCode());
	}
}
```

코드 - 비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = 
			Comparator.comparingInt(o -> o.hashCode());
```