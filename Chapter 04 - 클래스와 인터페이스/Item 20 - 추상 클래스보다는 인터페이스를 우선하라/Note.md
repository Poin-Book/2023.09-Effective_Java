# 제목

> 작성자: 밀리

## 목차
- [자바의 다중 구현 메커니즘](#자바의_다중_구현_메커니즘)
- [인터페이스 장점](#인터페이스_장점)
  - [기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.](#기존_클래스에도_손쉽게_새로운_인터페이스를_구현해넣을_수_있다.)
  - [인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.](#인터페이스는_믹스인(mixin)_정의에_안성맞춤이다.)
  - [인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.](#인터페이스로는_계층구조가_없는_타입_프레임워크를_만들_수_있다.)
  - [래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.](#래퍼_클래스_관용구와_함께_사용하면_인터페이스는_기능을_향상시키는_안전하고_강력한_수단이_된다.)
  - [디폴트 메서드](#디폴트_메서드)
 - [골격 구현](#골격_구현)
   - [추상 골격 구현(skeletal implementation) 클래스](#추상_골격_구현(skeletal_implementation)_클래스)
   - [단순 구현](#단순_구현)  
 - [결론](#결론)

### 자바의 다중 구현 메커니즘
> 다중 구현 메커니즘이란, 여러 인터페이스를 구현할 수 있도록 허용하는 방법을 의미
- 자바가 제공하는 다중 구현 메커니즘은 인터페이스, 추상 클래스 총 2가지
- 자바 8부터 인터페이스도 디폴트 메서드(default method)를 제공할 수 있게 됨
  - 따라서, 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있음
  #### 추상클래스 vs 인터페이스
  - 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 함
  - 자바는 단일 상속만 지원하므로, 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약을 안게 됨
  - 반면, 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급됨

### 인터페이스 장점
#### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.  
- 인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 implements 구문만 추가하면 끝이다.
- 반면, 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어려운 게 일반적이다.
  - 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 그 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 함
  - 이 방식은 그렇게 하는 것이 적절하지 않은 상황에서도 새로 추가된 추상 클래스의 모든 자손이 이를 상속하게 된다.

#### 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.
> 믹스인이란, 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.

즉, 믹스인은 클래스에 필요한 특정 기능을 선택적으로 추가하여 클래스가 다양한 기능을 혼합하고 사용할 수 있게 해주는 것.  
- 대상 타입의 주된 기능에 선택적 기능을 '혼합(mixed in)'한다고 해서 믹스인이라 부른다.
- 추상 클래스로는 믹스인을 정의할 수 없다. 이유는 기존 클래스에 덧씌울 수 없기 때문이다. 클래스는 두 부모를 섬길 수 없고, 클래스 계층구조에는 믹스인을 삽입하기에 합리적인 위치가 없기 때문이다.

#### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
- 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있으나 현실에는 계층을 엄격히 구분하기 힘든 개념도 있다.  
```java
public interface Singer {
    void sing();
}

public interface Songwriter {
    void compose();
}
```
- 위 코드처럼 타입을 인터페이스로 정의하면 가수 클래스가 Singer와 Songwriter 모두를 구현해도 전혀 문제되지 않는다.
- Singer와 Songwriter 모두를 확장하고 새로운 메서드까지 추가한 제 3의 인터페이스를 정의할 수도 있다.
```java
public interface SingerSongwriter extends Singer, Songwriter {
    void strum();

    void actSensitive();
}
```
- 같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어질 것.
- 속성이 n개라면 지원해야 할 조합의 수는 2^n 개나 될 것이다. 이러한 현상을 조합 폭발(combinatorial explosion)이라 부른다.
- 거대한 클래스 계층구조에는 공통 기능을 정의해놓은 타입이 없으니, 자칫 매개변수 타입만 다른 메서드들을 수없이 많이 가진 거대한 클래스를 낳을 수 있다.


#### 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.
- 타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속뿐이다.
- 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기는 더 쉽다.  
  1. 강한 의존성: 상속을 사용하면 서브클래스가 슈퍼클래스에 강하게 의존하게 되기 때문에 슈퍼클래스의 변경이 서브클래스에 영향을 미칠 수 있다.
  2. 깨지기 쉬운 기반 클래스: 기존의 슈퍼클래스를 수정하면 해당 슈퍼클래스를 상속받는 모든 서브클래스에 영향을 미칠 수 있기 때문에 안정성에 문제가 생긴다.

#### 디폴트 메서드
- 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해줄 수 있다. 이는 프로그래머의 일을 상당히 덜어준다.
- 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 @implSpec 자바독 태그를 붙여 문서화하면 된다.

  ##### 디폴트 메서드의 제약
  - 많은 인터페이스가 equals와 hashCode 같은 Object의 메서드를 정의하고 있지만, 이들은 디폴트 메서드로 제공해서는 안 된다.
    - equals()와 hashCode() 메서드는 클래스의 동등성 및 해시 코드 생성과 관련된 중요한 개념들이며, 각 클래스마다 그 의미와 구현이 다를 수 있기 때문에 인터페이스에서 디폴트 메서드로 제공하면 인터페이스를 구현하는 모든 클래스에서 동일한 의미를 제공하기 어렵다.
    - 리스코프 치환 원칙 위반 때문인데 이 치환 원칙은 하위 타입(subtype)은 상위 타입(supertype)으로 치환 가능해야 한다는 원칙이다. equals()와 hashCode()는 각 클래스의 동작을 특정하므로, 이러한 동작을 인터페이스의 디폴트 메서드로 제공하면 치환 가능성이 저해될 수 있다.
  - 인터페이스는 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다. (단, private 정적 메서드는 예외다).

### 골격 구현(skeletal implementation)
> 인터페이스나 추상 클래스에서 일부 기능을 미리 구현하여, 구현 클래스에서 이를 상속받아 재사용할 수 있는 기본적인 구현을 제공하는 것
#### 추상 골격 구현(skeletal implementation) 클래스
- 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다.
  - 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다.
  - 골격 구현 클래스는 나머지 메서드들까지 구현한다.
  - 이렇게하면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료되는데 이를 템플릿 메서드 패턴이라 한다.
  - 관례상 인터페이스 이름이 Interface라면 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는다.
    - ex) AbstractCollection, AbstractSet, AbstractList, AbstractMap
##### 골격 구현을 사용해 온성한 구체 클래스
```java
public class AbstractSkeletalConcreteClass {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 추정하자.
        return new AbstractList<>() {
            @Override
            public Integer get(int index) {
                return a[index]; // 오토박싱
            }

            @Override
            public Integer set(int index, Integer element) {
                int oldElement = a[index];
                a[index] = element; // 오토언박싱
                return oldElement; // 오토박싱
            }

            @Override
            public int size() {
                return a.length;
            }
        };
    }
}
```
- 골격 구현 클래스는 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다.
- 골격 구현을 확장하는 것으로 인터페이스 구현이 거의 끝나지만, 반드시 이렇게 해야하는 것은 아니다.
- 구조상 골격 구현을 확장하지 못한다면 인터페이스를 직접 구현해야 한다. 그래도 여전히 디폴트 메서드의 이점을 누릴 수 있다.
- 골격 구현 클래스를 우회적으로 이용할 수도 있다.
  - 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하면 된다.
  - 래퍼 클래스와 비슷한 이 방식을 시뮬레이트한 다중 상속(simulated multiple inheritance)이라 하며, 다중 상속의 많은 장점을 제공하면서 단점은 피하게 해준다.
##### 골격 구현 클래스
```java
public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {

    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override
    public boolean equals(Object obj) {
        if (obj == this) {
            return true;
        }
        if (!(obj instanceof Map.Entry)) {
            return false;
        }
        Map.Entry<?, ?> e = (Map.Entry) obj;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override
    public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override
    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```
- Map.Entry 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다.  
  - 디폴트 메서드는 equals, hashCode, toString 같은 Object 메서드를 재정의할 수 없기 때문이다.
- 골격 구현은 기본적으로 상속해서 사용하는 걸 가정한다. 따라서 설계 및 문서화 지침을 모두 따라야 한다.
- 인터페이스에 정의한 디폴트 메서드든 별도의 추상 클래스든, 골격 구현은 반드시 그 동작 방식을 잘 정리해 문서로 남겨야 한다.
#### 단순 구현
- 단순 구현(simple implementation)은 골격 구현의 작은 변종으로, AbstractMap.SimpleEntry가 좋은 예다.
```java
public static class SimpleEntry<K, V> implements Entry<K, V>, java.io.Serializable {
    private static final long serialVersionUID = -8499721149061103585L;

    private final K key;
    private V value;

    /**
     * Creates an entry representing a mapping from the specified
     * key to the specified value.
     *
     * @param key the key represented by this entry
     * @param value the value represented by this entry
     */
    public SimpleEntry(K key, V value) {
        this.key = key;
        this.value = value;
    }

    /**
     * Creates an entry representing the same mapping as the
     * specified entry.
     *
     * @param entry the entry to copy
     */
    public SimpleEntry(Entry<? extends K, ? extends V> entry) {
        this.key = entry.getKey();
        this.value = entry.getValue();
    }

    /**
     * Returns the key corresponding to this entry.
     *
     * @return the key corresponding to this entry
     */
    public K getKey() {
        return key;
    }

    /**
     * Returns the value corresponding to this entry.
     *
     * @return the value corresponding to this entry
     */
    public V getValue() {
        return value;
    }
}
```
- 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니란 점이 다르다.
- 쉽게 말하면, 동작하는 가장 단순한 구현이다. 이러한 단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.

### 결론
- 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
- 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해본다.
- 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.
- 골격 구현은 인터페이스에 걸려 있는 구현상의 제약 때문에 추상 클래스로 제공하는 경우가 더 흔하다.  



 

