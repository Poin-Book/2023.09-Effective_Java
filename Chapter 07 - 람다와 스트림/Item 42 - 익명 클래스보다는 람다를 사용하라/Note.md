# 익명 클래스보다는 람다를 사용하라

> 작성자: {다나}

## 목차
- 함수 객체
- 익명 클래스와 람다를 함수 객체로 사용한 예
- 람다 활용
- 람다 주의점
- 핵심 요약

## 함수 객체

- 예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용했다.
- 이런 인터페이스의 인스턴스를 함수 객체(function object)라고 하여, 특정 함수나 동작을 나타내는 데 썼다.
- 이전에는 함수 객체를 익명 클래스를 사용하여 만들었는데, 자바 8부터 함수형 인터페이스라는 개념이 등장하면서 이 인터페이스들의 인스턴스를 람다식을 사용해 만들 수 있게 되었다

## 익명 클래스와 람다를 함수 객체로 사용한 예

### ■  메서드 파라미터로 함수 객체를 받는 경우: 람다 미사용

```java
Collections.sort(words, new Comparator<String>() {
        public int compare(String o1, String o2) {
            return Integer.compare(o1.length(), o2.length());
        }
```

기존에는 위와 같이 익명 클래스를 이용해 처리했다.

### ■  메서드 파라미터로 함수 객체를 받는 경우: 람다 사용

```java
Collections.sort(words, (o1, o2) -> Integer.compare(o1.length(), o2.length()));

```

- Comparator 타입은 추상메서드 하나만 구현하면 되기 때문에 람다로 대체할 수 있다.
- 타입을 명시해야 코드가 더 명확할 때만 제외하고는 람다의 모든 매개변수 타입을 생략하자. 그래야 코드가 더 간단해진다.
    - 컴파일러가 타입을 알 수 없다는 오류를 내면 그 때 타입을 적어도 된다.
    - 제네릭을 잘 써주는 경우엔 컴파일러의 람다 타입 추론에 더욱 유리해지니 제네릭을 잘 써주자.
    - 잘 수행될 수 있는 코드도 Raw 타입을 활용하면 에러가 날 수 있으니 주의하자.

### ■ 메서드 파라미터로 함수 객체를 받는 경우: 람다 + 비교자 생성 메서드 활용

```java
// 람다 자리에 비교자 생성 메서드 사용
Collections.sort(words,comparingInt(String::length));
// 자바 8 때 List 인터페이스에 추가된 sort 메서드 사용
words.sort(comparingInt(String::length));
```

- 익명클래스를 사용할 때와 비교하면 정말 정말 간결해졌다

## 람다 활용하기 : enum Operation

### ■ 람다 활용 전

```java
enum Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```

### ■ 람다 활용 후

```java
enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator<Double> op;

    Operation(String symbol, DoubleBinaryOperator<Double> op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

- 자바에서 기본으로 제공하는 함수 인터페이스 중 하나인 `DoubleBinaryOperator`를 통해 코드를 개선했다.
    - 이는 double 타입 인수 2개를 받아 double 타입 결과를 돌려준다.
- 람다는 이름도 없고 문서화도 못하기 때문에 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 않는 것이 좋다.
    - 아이템 34에서는 상수별 클래스 몸체를 구현하는 방식 보다는 열거 타입에 인스턴스 필드를 두는 편이 낫다고 했다.
    - 하지만 enum 타입 생성자 안의 람다는 enum 타입의 인스턴스 멤버에 접근할 수 없다.
    - 따라서 상수별 동작을 단 몇줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다. ( 활용 전과 같이 )

## 람다 주의점

1. 람다는 함수형 인터페이스에서만 쓰인다.
    - 함수형 인터페이스란 추상 메서드가 한 개인 인터페이스를 말한다. 즉, 추상 메서드가 하나 이상이라면 람다를 사용할 수 없다.
2. 추상 클래스의 인스턴스를 만들 때는 익명 클래스를 사용해야 한다.
    - 추상 클래스에서는 람다를 사용할 수 없기 때문에 익명 클래스를 사용해야 한다
3. 람다의 `this` 키워드는 바깥을 가리키므로 주의해야 한다.
    - 반면 익명 클래스의 `this` 키워드는 인스턴스 자신을 가리키므로 함수 객체가 자신을 참조해야 한다면 익명 클래스를 사용하자.
4. 람다는 가상머신별로 `직렬화` 형태가 다를 수 있기 때문에 주의해야 한다.

## 핵심 요약

> 익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하자.
>
