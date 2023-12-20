# Item 90 - 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

> 작성자: 워니

## 목차
**개요  
직렬화 프록시 패턴(serialization proxy pattern)  
역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다  
한계점  
📌 요약**

---

### 개요
이전 아이템에서도 소개했듯이 Serializable을 구현하는 순간 클래스에는 바이트 스트림을 매개변수로 받는 또 하나의 생성자가 생긴다고 볼 수 있다.  
그렇기에 생성자 이외의 방법으로 인스턴스를 생성할 수 있게 되어 버그나 보안 문제가 발생할 가능성이 높아질 수 있는데, 
이를 직렬화 프록시 패턴(serialization proxy pattern)을 이용해 어느정도 위험을 해소할 수 있다는 점을 소개한다.  
_(직렬화 프록시는 일반적으로 이전 아이템에서 나온 `readObject`의 방어적 복사보다 강력하다.)_

---

### 직렬화 프록시 패턴(serialization proxy pattern)
바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 `private static`으로 선언한다. 이 중첩 클래스가 바깥 클래스의 직렬화 프록시다. 

#### 특징
- 중첩 클래스의 생성자는 단 하나여야 한다.
- 바깥 클래스를 매개변수로 받아야 한다. 
- 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사한다. 
- 바깥 클래스, 직렬화 프록시 모두 Serializable을 구현한다고 선언해야 한다.

##### 예제 코드 - Period 클래스에 적용한 직렬화 프록시 패턴

``` java
public final class Period implements Serializable {

  private final Date start;
  private final Date end;

  public Period(Date start, Date end) {
    if (this.start.compareTo(this.end) > 0) {
      throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    }
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
  }

  // 자바의 직렬화 시스템이 바깥 클래스의 인스턴스 말고 SerializationProxy의 인스턴스를 반환하게 하는 역할
  // 이 메서드 덕분에 직렬화 시스템은 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다.
  // "프록시야 너가 대신 직렬화되라"
  private Object writeReplace() {
    return new SerializationProxy(this);
  }

  // 불변식을 훼손하고자 하는 시도를 막을 수 있는 메서드
  // "Period 인스턴스로 역직렬화를 하려고 해? 안 돼!"
  private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다");
  }

  // 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스(Period의 직렬화 프록시)
  private static class SerializationProxy implements Serializable {

    private static final long serialVersionUID = 234098243823485285L;

    private final Date start;
    private final Date end;

    // 생성자는 단 하나여야 하고, 바깥 클래스의 인스턴스를 매개변수로 받고 데이터를 복사해야 함
    SerializationProxy(Period p) {
      this.start = p.start;
      this.end = p.end;
    }

    // 역직렬화 시 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다.
    // 역직렬화는 불변식을 깨뜨릴 수 있다는 불안함이 있는데, 
    // 이 메서드가 불변식을 깨뜨릴 위험이 적은 정상적인 방법(생성자, 정적 팩터리, 다른 메서드를 사용)으로 역직렬화된 인스턴스를 얻게 한다.
    // "역직렬화 하려면 이거로 대신하라"
    private Object readResolve() {
      return new Period(start, end);
    }
  }
}
```

- `writeReplace` 메서드는 자바의 직렬화 시스템이 바깥 클래스의 인스턴스 대신 SerializationProxy의 인스턴스를 반환하게 하는 역할을 한다.  
즉, 이 메서드로 인해 직렬화 시스템은 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.

- `readResolve` 메서드는 공개된 API만을 사용해 바깥 클래스의 인스턴스를 생성하는데, 이 패턴은 직렬화가 생성자를 이용하지 않고 인스턴스를 생성하는 부분을 해결해준다. 

---

### 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다

➡️ 쉽게 말하면,

- A 클래스의 인스턴스를 직렬화했다.
- 그 데이터를 가지고 역직렬화를 수행해 B 클래스의 인스턴스를 만들었다. (역직렬화 했다.)
- 근데 정상작동 한다!

</br>

> _**예시를 살펴보자.**_

#### EnumSet (Item 36)

 - `EnumSet`은 생성자 없이 정적 팩터리들만 제공
 - 단순히 생각하면 EnumSet 인스턴스를 반환하는 것 같지만, 열거 타입의 크기에 따라 다르다.  
   열거 타입의 원소의 개수가
    - 64개 이하면, `RegularEnumSet`을 반환
    - 그보다 크면, `JumboEnumSet`을 반환

![image](https://github.com/Poin-Book/2023.09-Effective_Java/assets/116738827/0ba22094-fa28-4775-ae98-28218e255c96)

#### 시나리오
```
- 63개의 열거타입을 가진 EnumSet이 있다.
- 이를 직렬화 하자.
- 그리고 원소 5개를 추가해 역직렬화 하자.
```

- 당연히 처음엔 `RegularEnumSet` 인스턴스 였다가, 나중에는 `JumboEnumSet`으로 하는 것이 더 효율적이고 좋을 것이다.
- 직렬화 프록시를 이용하면, 원하는 방향대로 사용이 가능하다.

##### EnumSet의 실제 코드 - 직렬화 프록시 패턴을 이용

``` java
private static class SerializationProxy <E extends Enum<E>> implements java.io.Serializable
{
    // EnumSet의 원소 타입
    private final Class<E> elementType;

    // EnumSet 내부 원소
    private final Enum<?>[] elements;

    SerializationProxy(EnumSet<E> set) {
        elementType = set.elementType;
        elements = set.toArray(ZERO_LENGTH_ENUM_ARRAY);
    }

    // 새롭게 원소의 크기에 맞는 EnumSet 생성
    @SuppressWarnings("unchecked")
    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType);
        for (Enum<?> e : elements)
            result.add((E)e);
        return result;
    }

    private static final long serialVersionUID = 362491234563181265L;
}

Object writeReplace() {
    return new SerializationProxy<>(this);
}

private void readObject(java.io.ObjectInputStream stream)
    throws java.io.InvalidObjectException {
    throw new java.io.InvalidObjectException("Proxy required");
}
```

---

### 한계점
직렬화 프록시 패턴에도 한계점이 있다. 

1. 클라이언트가 마음대로 확장할 수 있는 클래스에는 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다. 
3. 방어적 복사를 할 때보다 성능 부분(속도)에서 떨어진다.

---

### 📌 요약
제3자가 확장할 수 없는 클래스라면 가능한 한 직렬화 프록시 패턴을 사용하자.
