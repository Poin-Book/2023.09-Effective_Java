# 변경 가능성을 최소화하라

> 작성자: 루카

불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, `오류가 생길 여지도 적고 훨씬 안전하다.`

## 불변 클래스

클래스를 불변으로 만들려면 다음 다섯 규칙을 따르면 된다.

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 `final`로 선언한다.
- 모든 필드를 `private`으로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근하지 못하도록 한다.
  - 이런 필드는 절대 클라이언트가 제공한 객체나 배열 참조를 저장해서는 안 된다.
  - 생성자, 접근자, `readObject` 메서드 모두에서 `방어적 복사`를 수행해야 한다.

### 불변 클래스의 장점

불변 복소수 클래스

- `함수형 프로그래밍`의 패러다임을 따르는 방식
  - 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 방식

```java
public final class Complex {
  private final double re;
  private final double im;

  public Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }

  public double realPart() { return re; }
  public double imaginaryPart() { return im; }

  public Complex plus(Complex c) {
    return new Complex(re + c.re, im + c.im);
  }

  public Complex minus(Complex c) {
    return new Complex(re - c.re, im - c.im);
  }

  public Complex times(Complex c) {
    return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
  }

  public Complex dividedBy(Complex c) {
    double tmp = c.re * c.re + c.im * c.im;
    return new Complex((re * c.re + im * c.im) / tmp, 
                       (im * c.re - re * c.im) / tmp);
  }
}
```

[불변 객체의 장점]

- 근본적으로 스레드 안전하여 따로 동기화할 필요 없기 때문에 안심하고 공유할 수 있다.
- 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.
  - 새로운 클래스를 설계할 때 public 생성자 대신 정적 팩터리를 만들어두면, 클라이언트를 수정하지 않고도 필요에 따라 캐시 기능을 나중에 덧붙일 수 있다.
- 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
  - 때문에 불변 클래스는 clone 메서드나 복사 생성자를 제공하지 않는 것이 좋다.
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
  - 불변 객체는 맵의 키와 집합(Set)의 원소로 쓰기에 안성맞춤이다. 맵이나 집합은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정은 하지 않아도 된다.
- 불변 객체는 그 자체로 실패 원자성을 제공한다.
  - 생성자나 메서드가 실패하면 생성하려는 객체가 유효하지 않은 상태로 남는 문제를 막아준다.

### 값이 다르면 반드시 독립된 객체로 만들어야 한다

> 불변 클래스(객체)의 단점

- 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용이 든다.
- 이런 경우는 가변 동반 클래스를 만들어두면 된다.

ex) 불변 객체
  
  ```java
  /**
   * 예를 들어 백만 비트짜리 BigInteger에서 비트 하나를 바꾼다고 하자.
   * 이때 flipBit 메서드는 새로운 BigInteger 인스턴스를 만들어 반환한다.
   * 즉, 해당 방식은 BigInteger의 크기에 비례해 시간과 공간을 소비한다.
   */

  BigInteger moby = ...;
  mody = mody.flipBit(0);
  ```

이 문제를 해결하기 위한 두가지 방법

1. 다단계 연산들을 예측하여 기본기능으로 제공

    - 다단계 연산을 기본으로 제공한다면 더 이상 각 단계마다 새로운 객체를 만들 필요가 없어진다.
    - ex) BigInteger는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스를 package-private으로 제공한다.

2. `package-private`의 가변 동반 클래스

    - 클라이언트들이 원하는 복잡한 연산들을 정확히 예측할 수 있다면 해당 방법으로 충분하다.
    - 만약, 그렇지 않다면 해당 클래스를 public으로 제공하는 것이 최선이다.

## 불변 클래스 설계

> 클래스가 불변임을 보장하기 위해서는 자신을 상속하지 못하게 해야 한다.

[클래스를 불변으로 만드는 방법]

1. `final` 클래스로 선언하기
2. 모든 생성자를 `private` 혹은 `package-private`으로 만들고, `public 정적 팩터리`를 제공하기
   - 이 방법이 조금 더 유연함

EX)

```java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public static Complex valueOf(double re, double im) {
        return new Complex(re,im);
    }
    ...
}
```

패키지 바깥의 클라이언트에서 바라본 이 불변 객체는 사실상 final이다. public 이나 protected 생성자가 없으니 다른 패키지에서는 이 클래스를 확장하는 게 불가능하기 때문이다.

## private 생성자로 구현된 클래스를 상속하게 되면 어떻게 될까?

게터(getter)가 있다고 해서 무조건 세터(setter)를 만들지는 말자. 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.

PhoneNumber와 Complex 같은 단순한 값 객체는 항상 불변으로 만들자. 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소환으로 줄이자. 객체가 가질 수 있는 상태의 수를 줄이면 그 객체를 예측하기 쉬어지고 오류가 생길 가능성이 줄어든다.

java.util.concurrent 패키지의 CountDownLatch 클래스가 이상의 원칙을 잘 방증한다. 비록 가변 클래스지만 가질 수 있는 상태의 수가 많지 않다.
