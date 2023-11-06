# 아이템 33. 타입 안전 이종 컨테이너를 고려하라

> 작성자: 럭키

## 목차
- [본문](#본문)
- [타입 안전 이종 컨테이너는 완벽하지 않다](#타입-안전-이종-컨테이너는-완벽하지-않다)
- [정리](#정리)

## 본문
제네릭은 Set<E>, Map<K, V> 등의 컬렉션과 ThreadLocal<T>, AtomicReference<T> 등의 단일 원소 컨테이너에도 흔히 쓰인다. 예컨대 Set에는 원소의 타입을 뜻하는 단 하나의 타입 매개변수만 있으면 되며, Map에는 키와 값의 타입을 뜻하는 2개만 필요한 식이다.

### 하지만 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한 된다. 따라서, 더 유연한 수단이 필요할 때도 종종 있다.
데이터베이스의 행(row)은 임의 개수의 열(column)을 가질 수 있는데, 모든 열을 타입 안전하게 이용할 수 있다면 좋을 것이다.<br>

``` java
public static void main(String[] args) {

  Map<Class<?>,Object> favorites = new HashMap<>();

  favorites.put(String.class,"밥");
  favorites.put(Integer.class,"이것도 된다.");

  Integer o = (Integer) favorites.get(Integer.class); // ClassCastException
}
```
혹은 타입을 \<Object>로 사용하면 괜찮지 않냐는 질문이 나올수 있다. 그런데 이는 컴파일 타임에 타입을 체크하려는 제네릭의 의도를 무시한 행위이며, 타입 안정성이 떨어지고, 사용자의 잘못된 입력시 런타임 오류를 발생하는 문제를 야기한다.

e.g. put(Integer.class, 1) 이렇게 넣을 경우, get 조회시 ClassCastException이 발생. 

이는 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 만족할 수 있게 된다. 이런 방식을 **타입 안전 이종 컨테이너 패턴**이라 한다.
``` java
public class Favorite {
	private Map<Class<?>, Object> favorites = new HashMap<>();

	public <T> void putFavorite(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), type.cast(instance));
	}
	public <T> T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type));
	}
}

public static void main(String[] args) {
	Favorites f = new Favorites();
	
	f.putFavorite(String.class, "Java");
	f.putFavorite(Integer.class, 0xcafebabe);
	f.putFavorite(Class.class, Favorites.class);

	String favoriteString = f.getFavorite(String.class);
	int favoriteInteger = f.getFavorite(Integer.class);
	Class<?> favoriteClass = f.getFavorite(Class.class);

	System.out.printf("%s %x %s\\n", favoriteString, favoriteInteger, favoriteClass.getName());
}
```
이렇게 타입 안전 이종 컨테이너를 사용하게 되면 Favorites 인스턴스는
- 컴파일 타임에 타입안정성을 보장
- map에서 꺼내올 때, 타입캐스팅을 클라이언트 쪽에서 해주지 않아도 되서 깔끔

비한정적 와일드카드 타입이라 이 맵 안에 아무것도 넣을 수 없다고 생각할 수 있지만, 와일드카드 타입이 중첩 되었다기 때문에 가능한 일이다.

### 와일드 타입이 중첩되었다는 말은 무슨 뜻일까?

이는 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻으로, 첫 번째는 Class<String>, 두 번째는 Class<Integer>식으로 될 수 있다.

## 타입 안전 이종 컨테이너는 완벽하지 않다
- 악의적인 클라이언트가 Class 객체를 로 타입으로 넘기면 타입 안전성이 쉽게 깨진다. 하지만 이는 클라이언트 코드에서 컴파일할 때 비검사 경고가 뜬다.
Favorites가 타입 불변식을 어기는 일이 없도록 보장하려면 putFavorite 메서드와 같이 instance의 타입이 type으로 명시한 타입과 같은지 확인하면 된다.
```java
// 동적 형변환으로 런타임 타입 안전성 확보
public <T> void putFavorite(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), type.cast(instance));
	}
```
java.util.Collections에는 checkedSet, checkedList, checkedMap 같은 메서드가 있는데 바로 이 방식을 적용한 컬렉션 래퍼들.

- Favorites 클래스는 **실체화 불가 타입에는 사용할 수 없다**. String이나 String[]은 저장할 수 있어도 List<String>은 저장할 수 없다. List<String>과 List<Integer>는 List.class라는 객체를 공유하기 때문이다.<br>
이는 한정적 타입 토큰을 활용할 수 있다 한정적 타입 토큰이란 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 **표현 가능한 타입을 제한**하는 타입 토큰이다.
애너테이션 API는 한정적 타입 토큰을 적극적으로 사용한다.<br>
**AnnotatedElement.<T extends Annotation> T getAnnotation(Class<T> annotationClass);**<br>
여기서 annotationType인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰이다. 이 메서드는 토큰으로 명시한 타입의 에너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환하고 없다면 null을 반환한다. 즉, 애너테이션된 요소는 그 키가 애너테이션 타입인, 타입 안전 이종 컨테이너이다.
+ 슈퍼 타입 토큰을 사용하는 방식도 있으나 이도 완벽한 방법이 되진 못한다.
### Class<?> 타입의 객체가 있고, 이를 (getAnnotation처럼) 한정적 타입 토큰을 받는 메서드에 넘기려면 어떻게 해야 할까?
객체를 Class<? extends Annotation>으로 형변환할 수도 있지만 이 형변환은 비검사이므로 컴파일하면 경고가 뜰 것이다. 운 좋게도 Class 클래스가 이런 형변환을 안전하게 수행해주는 인스턴스 메서드를 제공.<br>
-> **asSubClass** 메서드. 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다. 여기서 asSubClass는 특정 클래스의 하위 타입의 한정적 타입 토큰으로 캐스팅해주는 역할을 한다.

```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) { 
	Class<?> annotationType = null; // 비한정적 타입 토큰
	try {
		annotationType = Class.forName(annotationTypeName);
	} catch (Exception ex) {
		throw new IllegalArgumentException(ex);
	} 
	return element.getAnnotation(
		annotationType.asSubclass(Annotation.class));
}
```



## 정리
- 유연한 컨테이너, 타입 안전성 -> 타입 안전 이종 컨테이너
- Class를 키로 쓰며, 이렇게 쓰이는 Class 객체를 타입 토큰
- 타입 안전 이종 컨테이너는 제약이 있으니 이를 숙지하고 주의해서 사용하자

## 참고
https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD-%EC%99%80%EC%9D%BC%EB%93%9C-%EC%B9%B4%EB%93%9C-extends-super-T-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4#%EB%B9%84%ED%95%9C%EC%A0%95%EC%A0%81_%EC%99%80%EC%9D%BC%EB%93%9C_%EC%B9%B4%EB%93%9C<br>
https://ojt90902.tistory.com/1418<br>
https://hwan33.tistory.com/27<br>
https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/04/24/%ED%83%80%EC%9E%85-%EC%95%88%EC%A0%84-%EC%9D%B4%EC%A2%85-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88/
