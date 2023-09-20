# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> 작성자: 럭키

## 목차
- [본문](#본문)
- [팩토리 메소드 패턴](#팩토리-메소드-패턴)

## 본문

**사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**

클래스가 여러 자원의 인스턴스를 지원해야 하고, 클라이언트가 원하는 자원을 사용해야 한다면, **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**이 효과적

- 사례 1

```java
public class SpellChecker {
	private static final Lexicon dictionary = ~;

	private SpellChecker() {}

	public static boolean isValid(String word) {~}
	public static List<String> suggestions(String typo) {~}
}
```

→ 사전을 static final로 구현해 한 사전으로만 사용 가능

- 사례 2

```java
public class SpellChecker {
	private final Lexicon dictionary = ~;
	
	private SpellChecker(~) {}
	public static SpellChecker INSTANCE = new SpellChecker(~);
	
	public boolean isValid(String word) {~};
	public List<String> suggestions(String typo) {~};
}
```

→ 위의 경우도 SpellChecker를 싱글톤으로 구현을 했지만 사전을 유연하게 변경X

—> 위의 사례들 모두 사전이라는 자원의 의존적이지만, 단 한 종류의 사전 만을 사용할 수 있는 문제를 지님.

- Sol

```java
public class SpellChecker {
	private final Lexicon dictionary;

	public SpellChecker(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}

	public boolean isValid(String word) {~};
	public List<String> suggestions(String typo) {~};
}
```

→ 사용하는 사전(자원)에 따라 동작이 달라져야 하는 경우에
인스턴스를 생성할 때, 생성자에 필요한 자원을 넘겨주는 방식이 유용하다. 위처럼 **의존 객체를 주입 해주는 방식**을 사용함으로써 유연성과 테스트 용이성을 높일 수 있다.

의존 객체 주입은 생성자, 정적 팩토리, 빌더 모두에 똑같이 응용이 가능하다.


이 패턴을 변형한 게 **팩토리 메소드 패턴**

## 팩토리 메소드 패턴

![image1](https://github.com/Poin-Book/2023.09-Effective_Java/assets/110045522/fdd570fb-8e22-4831-997c-35ab1d57026b)

**팩토리 메서드 패턴**은 

팩토리를 사용해서 인스턴스를 만드는 과정 중에 비슷한 부분은 재사용을 하고, 다른 부분은 공장마다 각각 다른 공정을 거쳐 만들게 합니다.

구체적인 인스턴스를 구체적인 팩토리 클래스에서 만들어주는 패턴입니다.

팩토리도 인터페이스가 있고, 그를 구현한 구체적인 클래스가 있고, 제품도 인터페이스가 있고 그를 구현한 구체적인 클래스가 있습니다.

장점은 **클라이언트 코드에 변화 없이** 새로운 팩토리, 새로운 Product를 추가할 수 있습니다.

Dictionary로 인터페이스를 사용하고 DictionaryFactory로 인터페이스를 사용해서 새로운 팩토리, 제품이 들어와도 클라이언트의 코드는 변화가 없습니다.

이러한 구조를 객체지향 원칙적으로 “**확장에 열려 있고 변경에 닫혀 있다**”고 말합니다.

이 팩토리 메서드 패턴을 일반적으로 확장하면 스프링 IOC 핵심인 빈 팩토리가 됩니다.
빈 팩토리가 바로 팩토리 메서드 패턴의 대표적인 예제입니다.

팩토리 메소드 사용 예

```java
public FactoryEx (DictionFactory dictionaryFactory) {
    this.dictionary = dictionaryFactory.get();
}
```

자바 8에서 소개한 Supplier<T>인터페이스도 팩토리 메소드의 대표 예.

![image2](https://github.com/Poin-Book/2023.09-Effective_Java/assets/110045522/67cdcf1b-d144-423d-994d-f216c759c9aa)

Supplier\<T>의 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입을 사용 → 팩터리 타입 매개변수 제한 → 클라이언트는 자신이 명시한 타입의 하위 타입이면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.

의존성이 많아 질 경우 코드의 복잡하게 만들 수 있다. 그래서 우리는 의존 객체 주입 프레임워크인 스프링을 통해 문제를 해결

DI라는 큰 장점을 가지고 있어 스프링을 사용하는 것

Q. 스프링을 사용하는 이유? 장점? → 중 하나인 **DI**