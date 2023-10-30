# 아이템 24. 멤버 클래스는 되도록 static으로 만들라

> 작성자: 럭키

## 목차
 - [정적 멤버 클래스 vs 비정적 멤버 클래스](#정적-멤버-클래스-vs-비정적-멤버-클래스)
 - [익명 클래스](#익명-클래스)
 - [지역 클래스](#지역클래스)

중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 한다.
그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

중첩 클래스의 종류는 다음과 같다.

- 정적 멤버 클래스
- 비정적 멤버 클래스
- 익명 클래스
- 지역 클래스

이중 정적 멤버 클래스를 제외한 클래스를 내부 클래스(inner class)라고 부른다.

## **정적 멤버 클래스 vs 비정적 멤버 클래스**

`정적 멤버 클래스`는 다른 클래스 안에 선언되며 바깥 클래스의 private 멤버에도 접근 가능한 것을 제외하면 일반 클래스와 동일하다. 정적 멤버 클래스와 비정적 멤버 클래스는 코드 상에서 static의 유무만 보일 수 있으나 의미상의 차이는 더 크다.

```java
public class Calculator {

    public static enum Operation {
        PLUS((number1, number2) -> number1 + number2),
        MINUS((number1, number2) -> number1 - number2),
        MULTIPLY((number1, number2) -> number1 * number2),
        DIVIDE((number1, number2) -> number1 / number2);

        private final BiFunction<Double, Double, Double> formula;

        Operation(BiFunction<Double, Double, Double> formula) {
            this.formula = formula;
        }

        public double calculate(double number1, double number2) {
            return formula.apply(number1, number2);
        }
    }
}

public class Item24 {
    public static void main(String[] args) {
        System.out.println(Calculator.Operation.DIVIDE.calculate(10, 8)); // 1.25
    }
}
```

`비정적 멤버 클래스`의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다<br> 그래서 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 통해 바깥 인스턴스의 메서드를 호출한다거나 바깥 인스턴스를 참조할 수 있다.

여기서 정규화된 this란, `클래스명.this` 형태로 바깥 클래스의 이름을 명시하는 용법을 말한다.

```java
public class Member {
    private MemberSupport memberSupport;
    private String name;

    class MemberSupport {
        void printName() {
            System.out.println(name); // 가능한 문법
        }
    }

    public static void main(String[] args) {
        Member member = new Member();
        member.name = "hyun";
        member.memberSupport = member.new MemberSupport(); // 이렇게 인스턴스 생성
        member.memberSupport.printName();
    }
}
```

- Map 인터페이스의 구현체 : 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용
- 컬렉션 인터페이스 구현체 : 자신의 반복자 구현

에서 비정적 멤버 클래스가 활용되었다.

**멤버 클래스가 바깥 클래스의 인스턴스에 독립적이라면(접근할 필요가 없다면) 무조건 static을 추가하여 정적 멤버 클래스로 만들.** 

→ static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 되는데, 이 참조를 저장하려면 시간과 공간적인 리소스가 소비된다. 뿐만 아니라 GC가 바깥 클래스의 인스턴스를 정리하지 못하게 되어 메모리 누수를 일으킬 수 있다.

## 익명 클래스
- 이름 X
- 바깥 클래스의 멤버 X
- 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다
- 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스 참조가 가능하다
- 상수 정적변수 (`static final`) 외에는 정적 변수를 가질 수 없다
- `intanceof` 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다. 
- 인터페이스를 구현할 수도 없고 다른 클래스를 상속할 수도 없다.
- 익명 클래스는 표현식 중간에 등장함으로 짧지 않으면 가독성이 떨어진다.
- 익명 클래스는 자바 8 이전에는 함수 객체를 표현하는 데에 사용하거나 처리 객체(process object)를 만드는데 사용했다.
- 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 주로 사용 → 람다로 대체
- 정적 팩터리 메소드 구현 시 사용

## **지역클래스**

거의 사용 X. 지역 변수를 선언할 수 있는 곳이면 어디든 선언 가능하다. 유효 범위도 지역변수와 같다.

1. 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다.
2. 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버를 가질 수 없다.
3. 가독성을 위해 짧게 작성해야 한다.
