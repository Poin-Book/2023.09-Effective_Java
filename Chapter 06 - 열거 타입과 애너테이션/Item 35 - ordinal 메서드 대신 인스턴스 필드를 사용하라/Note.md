# ordinal 메서드 대신 인스턴스 필드를 사용하라

> 작성자: {다나}

## 목차
- ordinal 이란
- ordinal을 잘못 사용한 예
- 해결책
# [아이템 35] ordinal 메서드 대신 인스턴스 필드를 사용하라

### ■ enum type의 ordinal

Enum 추상 클래스의 내부를 보면 ordinal이라는 정수형 필드가 존재한다.

ordinal 필드는 열거 타입 상수가 열거 타입에서 선언된 순서를 가지고 있다. 이 ordinal 필드의 값을 얻기 위해서는 ordinal() 메서드를 호출하면 된다.

### ■ ordinal을 잘못 사용한 예

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians(){
        return ordinal() + 1;
    }

}
```

문제점

- 상수 선언 순서가 바뀔 경우 정해놓은 값과 달라진다.
- 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.
- 값을 중간에 비워둘 수 없다. 더미 상수를 추가할 수도 있지만 이는 코드가 깔끔하지 못하고 실용성이 떨어진다.

### ■ 해결책

> **열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하자.**
> 

```java
public enum Ensembel{

    SOLO(1), DUET(2), TRIO(3) ...
    ;

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size;}
    public int numberOfMusicians() { return numberOfMusicians; }

}
```

- Enum의 API 문서를 봐도 ordinal 필드의 주석을 보면
**'대부분 프로그래머는 이 메소드를 쓸 일이 없다. 이 메소드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다'** 라고 쓰여있다.
- 이러한 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자.
