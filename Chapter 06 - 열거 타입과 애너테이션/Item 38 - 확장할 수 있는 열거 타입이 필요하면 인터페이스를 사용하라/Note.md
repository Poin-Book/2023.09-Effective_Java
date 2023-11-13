# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

> 작성자: 루카

## 목차

## 1. 열거 타입의 확장 불가능

열거 타입은 확장할 수 없다. <br>
즉, 열거한 값들을 그대로 가져온 다음 값을 더 추가할 수 없다.

열거 타입이 확장 불가능하도록 설계한 이유는 다음과 같다.

1. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않는 것은 이상하다.
2. 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법이 마땅치 않다.
3. 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 더 복잡해진다

## 2. 인터페이스를 통한 확장 가능 열거 타입 흉내내기

그렇다면 확장할 수 있는 열거 타입이란 구현할 수 없는 것인가? <br>
아니다. 인터페이스를 정의하고, 열거 타입이 인터페이스를 구현하도록 하면 열거 타입을 확장한 효과를 낼 수 있다.

### 1) 열거 타입 확장 예제

[인터페이스 정의]
  
  ```java
  public interface Operation {
      double apply(double x, double y);
  }
  ```

[기본 열거 타입]

  ```java
  public enum BasicOperation implements Operation {
      PLUS("+") {
          public double apply(double x, double y) { return x + y; }
      },
      MINUS("-") {
          public double apply(double x, double y) { return x - y; }
      },
      TIMES("*") {
          public double apply(double x, double y) { return x * y; }
      },
      DIVIDE("/") {
          public double apply(double x, double y) { return x / y; }
      };
  
      private final String symbol;
  
      BasicOperation(String symbol) {
          this.symbol = symbol;
      }
  
      @Override
      public String toString() {
          return symbol;
      }
  }
  ```

[확장된 열거 타입]

  ```java
  public enum ExtendedOperation implements Operation {
      EXP("^") {
          public double apply(double x, double y) { return Math.pow(x, y); }
      },
      REMAINDER("%") {
          public double apply(double x, double y) { return x % y; }
      };
  
      private final String symbol;
  
      ExtendedOperation(String symbol) {
          this.symbol = symbol;
      }
  
      @Override
      public String toString() {
          return symbol;
      }
  }
  ```

새로 추가된 연산은 Operation 인터페이스를 사용하도록 작성되어 있기만 하면 어디든 사용할 수 있다.

또한 apply가 인터페이스에 선언되어 있기 때문에 열거 타입에 따로 추상 메서드로 선언하지 않아도 된다는 점에서 아이템 34의 상수별 메서드 구현과 차이가 있다.

### 2) 사용 예제

1. 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예

  ```java
  public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
  }
  private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
  }
  ```

- 위의 예제에서는 ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들을 모두 출력한다. 여기서 class 리터럴은 `한정적 타입 토큰` 역할을 한다

> opEnumType 매개변수
>
> `<T extends Enum<T> & Operation> Class<T>`의 의미 <br>
> = Class 객체가 `enum`인 동시에 `Operation`의 하위 타입이어야 한다.

2. 컬렉션 인스턴스를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예

  ```java
  public static void main(String[] args) {
      double x = Double.parseDouble(args[0]);
      double y = Double.parseDouble(args[1]);
      test(Arrays.asList(ExtendedOperation.values()), x, y);
  }
  private static void test(Collection<? extends Operation> opSet,
                           double x, double y) {
      for (Operation op : opSet)
          System.out.printf("%f %s %f = %f%n",
                  x, op, y, op.apply(x, y));
  }
  ```

- 여기서는 Class 객체 대신에 `한정적 와일드카드` 타입인 `Collection<? extends Operation>`을 사용하였다.
- 이 예제의 test 메서드가 조금 더 유연하지만 특정 연산에서는 EnumSet과 EnumMap을 이용하지 못한다는 문제가 있다.

### 3) 문제점 - 구현 상속 불가능

열거 타입끼리 구현을 상속할 수 없다.

[해결 방법]

1. 아무 상태에도 의존하지 않는 경우
   - 인터페이스에 디폴트 메서드를 추가하는 방법을 이용할 수 있다.
2. 의존적인 경우
   - 공유하는 기능을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식을 사용한다.
