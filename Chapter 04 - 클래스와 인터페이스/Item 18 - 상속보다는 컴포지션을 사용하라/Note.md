# 상속보다는 컴포지션을 사용하라

> 작성자: 럭키

## 목차
- [상속은 왜 사용할까](#상속은-왜-사용할까)
- [상속의 단점](#상속은-왜-사용할까)
- [그러면 상속은 언제 사용 가능할까?](#그러면-상속은-언제-사용가능한가)
- [상속으로 인한 오작동 예시](#상속으로-인한-오작동-예시)
- [오버라이드를 안한다면?](#오버라이드를-안-한다면)
- [상속 대신 컴포지션!](#상속-대신-컴포지션을-사용해보자)
- [데코레이터 패턴](#데코레이터-패턴)
- [랩퍼클래스 사용시 주의사항](#랩퍼클래스-활용에-주의할-점)
- [핵심 정리](#핵심-정리)

## 상속은 왜 사용할까?
- 코드를 재사용함으로써 중복을 줄일 수 있다.
- 변화에 대한 유연성 및 확장성이 증가한다.
- 개발 시간이 단축된다.

## 상속의 단점
상속은 코드를 재사용하기 가장 쉬운 방법이다. (extends를 의미)<br> 
하지만 다른 패키지에 있는 클래스를 상속 받는 것은 위험하다. **상속은 캡슐화를 깨뜨리기 때문이다.**<br>

상위 클래스가 어떻게 구현되어 있느냐에 따라 하위 클래스의 동작에 이상이 발생할 수 있다. 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 안 건드려도 하위 클래스가 오작동 할 수 있다. 이를 해결하기 위해 모든 하위 클래스에서 일일이 수정을 해주는 작업을 하게 된다.<br>

하위 클래스가 상위 클래스에 강하게 결합, 의존을 의미 => **변화에 유연하게 대처하기 어려워진다!**

## 그러면 상속은 언제 사용가능한가?
- 두 클래스가 "is-a" 관계일 때 클래스 B를 클래스 A의 서브 클래스로 확장(상속)해야 합니다. 만일 클래스 B를 클래스 A의 서브 클래스로 만들고 싶다면 "모든 B 객체가 진정한 A인가?" 라는 질문을 던져봐야 합니다. 자신있게 "예" 라고 대답할 수 없다면 상속을 하지 말아야 합니다.

- 수퍼클래스의 API 에 결함은 없는지, 결함이 있다면 그 결함을 그대로 상속받을 것인지를 생각해보아야 합니다. 결함이 있고, 그 결함을 받아들일 수 없다면 사용해서는 안됩니다.

## 상속으로 인한 오작동 예시
한 예를 들면, 상위 클래스 중 특정 필드에 대한 보안 대책이 필요해 모든 변경자를 override해 보안 대책 로직을 거친 후 변경하게 하는 자식 클래스를 만들었다고 생각해보자. 다음 릴리즈에서 상위 클래스에 새로운 변경자가 생긴다면 자식 클래스는 보안 대책을 거치지 않고 필드가 변경되는 경우가 생기게 된다.<br>

아래는 책의 예시이다.

```Java
public class InstrumentedHashSetUseExtends<E> extends HashSet {
    private int addCount = 0; // 추가된 원소의 수

    public InstrumentedHashSetUseExtends() {
    }

    public InstrumentedHashSetUseExtends(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(Object o) {
        return super.add(o);
    }

    @Override
    public boolean addAll(Collection c) {
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

}

public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {
    // ...

    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size() / .75f) + 1, 16));
        addAll(c); // super의 addAll 메서드를 호출한다.
    }

    // ...
}

public abstract class AbstractCollection<E> implements Collection<E> {
    // ...

    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    // ...
}
```
- HashSet의 addAll 메서드는 add 메서드를 사용해서 구현되어 있다.
- InstrumentedHashSetUseExtends의 addAll은 addCount를 더한 후, HashSet의 addAll을 호출한다.
- HashSet의 addAll은 각 원소를 add 메서드를 호출해 추가하는데, 이때 불리는 add는 InstrumentedHashSetUseExtends에서 재정의한 메서드다.
- 따라서 addCount의 값이 중복으로 더해지게 된다.<br>

이 경우 하위 클래스에서 addAll 메서드를 재정의하지 않으면 문제를 고칠 수 있다. 하지만 HashSet의 addAll이 add 메서드를 이용해 구현했음을 가정한 해법이라는 한계를 가진다. 이처럼 자신의 다른 부분을 사용하는 '자기 사용(self-use)' 여부는 해당 클래스의 내부 구현 방식에 해당한다. 따라서 이런 가정에 기댄 InstrumentedHashSetUseExtends도 깨지기 쉽다.

addAll 메서드를 다른 식으로 재정의할 수도 있다. 하지만 상위 클래스의 메서드 동작을 다시 구현하는 것은 어렵고, 시간도 더 들고, 오류를 내거나 성능을 떨어뜨릴 수도 있다. 또한 하위 클래스에서는 접근할 수 없는 private 필드를 써야 하는 상황이라면 이 방식으로는 구현자체가 불가능하다.

## 오버라이드를 안 한다면?
위 사례는 메서드 재정의 때문에 생긴 일이므로, 메서드 재정의를 안 하면 해결되지 않을까? 새로운 메서드를 생성한다 해도 문제는 생긴다. 추후 부모 객체에 내가 자식 객체에 새롭게 만들었던 메서드와 반환 타입만 다른 메서드를 추가하면 내 자식 객체는 컴파일 에러가 나게 된다. 반환타입까지 똑같다면, 의도치 않게 오버라이드를 한 것이 된다.

## 상속 대신 컴포지션을 사용해보자
기존 클래스를 상속 하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스를 포함하자. 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 점에서, 이 기법은 **조합(composition)** 이라고 부른다.<br>
새로운 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해준다. 이 방식은 **전달(forwarding)** 이라고 한다. 이렇게 하면 새로운 클래스는 기존 클래스의 내부 구현 영향으로부터 벗어나며, 새로운 메서드가 추가되어도 영향을 받지 않는다. 또한 기존 클래스의 api를 호출하여 소통하므로 캡슐화를 깨지도 않는다.<br>

다음은 조합 방식으로 구현한, 추가된 요소의 개수를 계측할 수 있는 InstrumentedSet의 예다.
```Java
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {

 private int addCount = 0;

 public InstrumentedSet(Set<E> s) {
	 super(s);
 }

 @Override public boolean add(E e) {
	 addCount++;
	 return super.add(e);
 }

 @Override public boolean addAll(Collection<? extends E> c) {
	 addCount += c.size();
	 return super.addAll(c);
 }

 public int getAddCount() {
	 return addCount;
 }
}
```
이와 같이 동일한 인터페이스를 감싸고 있는 클래스를 **랩퍼 클래스**라고 부른다

```Java
// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {

 private final Set<E> s;

 public ForwardingSet(Set<E> s) { this.s = s; }

 public void clear() { s.clear(); }
 public boolean contains(Object o) { return s.contains(o); }
 public boolean isEmpty() { return s.isEmpty(); }
 public int size() { return s.size(); }
 public Iterator<E> iterator() { return s.iterator(); }
 public boolean add(E e) { return s.add(e); }
 public boolean remove(Object o) { return s.remove(o); }
 public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
 public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
 public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
 public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
 public Object[] toArray() { return s.toArray(); }
 public <T> T[] toArray(T[] a) { return s.toArray(a); }

 @Override public boolean equals(Object o) { return s.equals(o); }
 @Override public int hashCode() { return s.hashCode(); }
 @Override public String toString() { return s.toString(); }
}
```
이처럼 상위 객체를 인터페이스를 조합 방식으로 전달해주는 객체를 **포워딩 클래스**라고 부른다.

## 데코레이터 패턴
객체들을 새로운 행동들을 포함한 특수 래퍼 객체들 내에 넣어서 위 행동들을 해당 객체들에 연결시키는 구조적 디자인 패턴을 **데코레이터 패턴**이라고 부른다.

### 데코레이터 패턴은 이 객체들을 사용하는 코드를 훼손하지 않으면서 런타임에 추가 행동들을 객체들에 할당할 수 있어야 할 때 사용하자.

 데코레이터는 비즈니스 로직을 계층으로 구성하고, 각 계층에 데코레이터를 생성하고 런타임에 이 로직의 다양한 조합들로 객체들을 구성할 수 있도록 합니다. 이러한 모든 객체가 공통 인터페이스를 따르기 때문에 클라이언트 코드는 해당 모든 객체를 같은 방식으로 다룰 수 있습니다.

### 이 패턴은 상속을 사용하여 객체의 행동을 확장하는 것이 어색하거나 불가능할 때 사용하자.

![image](https://refactoring.guru/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb)<br>
위의 사례를 상속을 통해 해결하기 위해서는 여러 자식 클래스를 합성해야 해 클래스 수가 폭발적으로 늘어나게 된다.

 많은 프로그래밍 언어에는 클래스의 추가 확장을 방지하는 데 사용할 수 있는 final 키워드가 있다. Final 클래스의 경우 기존 행동들을 재사용할 수 있는 유일한 방법은 데코레이터 패턴을 사용하여 클래스를 자체 래퍼로 래핑하는 것입니다.

### 장점
 - 새 자식 클래스를 만들지 않고도 객체의 행동을 확장할 수 있다.
 - 런타임에 객체들에서부터 책임들을 추가하거나 제거할 수 있다.
 - 객체를 여러 데코레이터로 래핑하여 여러 행동들을 합성할 수 있다.
 - 단일 책임 원칙. 다양한 행동들의 여러 변형들을 구현하는 모놀리식 클래스를 여러 개의 작은 클래스들로 나눌 수 있다.

### 단점
 - 래퍼들의 스택에서 특정 래퍼를 제거하기가 어렵다.
 - 데코레이터의 행동이 데코레이터 스택 내의 순서에 의존하지 않는 방식으로 데코레이터를 구현하기가 어렵다.
 - 계층들의 초기 설정 코드가 보기 흉할 수 있다.

자세한 내용은 아래 링크를 참조해주세요!<br>
https://refactoring.guru/ko/design-patterns/decorator

## 랩퍼클래스 활용에 주의할 점
콜백 프레임워크와 함께 쓰는 경우엔 주의해야 한다. 콜백 프레임워크는 자기 자신의 참조를 넘겨 다음 호출 때 사용되도록 하는데, 내부 객체는 자신을 감싸고 있는 객체의 존재를 모르기 때문에 대신 자신의 참조(SELF)를 넘기고 이 때문에 오동작 할 수 있다.<br>
콜백 프레임워크는 대부분의 GUI 프레임워크, SAX(스트리밍 XML) XML 파서같이 콜백을 사용하는 것을 말한다. 기본 패턴은 다른 클래스를 초기화할 때 핸들러 클래스의 인스턴스(또는 때로는 메소드 참조만)를 전달하는 방식을 사용

## 핵심 정리
상속은 강력하지만 *캡슐화를 해친다*는 문제가 있다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다. is-a 관계여도, 패키지가 다르고 상위 객체가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다.
거기에 API에 아무런 결함이 없는 경우, 결함이 있다면 하위 클래스까지 전파돼도 괜찮은 경우여야 한다.<br>

그렇지만 실은 이런 조건을 만족한 경우에도 상속은 조합과 달리 캡슐화를 깨뜨리기 때문에 100% 정답은 없다.<br>

상속을 코드 재사용만을 위한 수단으로 사용하면 안 된다. 상속은 반드시 확장이라는 관점에서 사용해야 한다.
상황에 맞는 최선의 방법을 선택하면 된다. 다만, 애매할 때는 상속 대신 **컴포지션과 전달을 사용하자.**


## 참고
https://tecoble.techcourse.co.kr/post/2020-05-18-inheritance-vs-composition/<br>
https://aroundck.tistory.com/617<br>
https://refactoring.guru/ko/design-patterns/decorator