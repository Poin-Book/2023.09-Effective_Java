# 태그 달린 클래스보다는 클래스 계층구조를 활용하라

> 작성자: 워니

## 목차
### 태그 달린 클래스
### 태그 달린 클래스는 단점이 많다. 
### 제안 - 서브타이핑

---

### 태그 달린 클래스
#### 태그 달린 클래스란?
: 두 가지 이상의 의미를 표현할 수 있으며 현재 표현하는 의미를 태그 값으로 알려주는 클래스

``` java
public class Figure {
    enum Shape {
        RECTANGLE, CIRCLE
    };

    private final Shape shape; // 태그 필드

    private double length;
    private double width;

    private double radius;

    public Figure(double radius) {
        this.radius = radius;
        shape = Shape.CIRCLE;
    }

    public Figure(double length, double width) {
        this.length = length;
        this.width = width;
        shape = Shape.RECTANGLE;
    }
    
    public double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return  Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

➡️ 사각형(rectangle)과 원(circle)을 나타내는 클래스이고, 하나의 클래스에서 두가지 타입에 대응이 가능하다.  
(area() 메서드 하나로 타입에 따라 area 크기를 계산해 반환하고 있어서 편해보일 수도 있으나, 사실상 문제가 많은 코드이다.)

### 태그 달린 클래스는 단점이 많다. 
1. 열거 타입, 태그 필드, switch문 등의 부가적인 코드가 너무 많다.  
2. 하나의 메서드에 여러 구현이 들어가 있기에 가독성도 떨어지고 SRP 지침에도 어긋난다.
3. 하나의 타입으로 정해지면 그 외의 타입을 위한 코드는 모두 불필요한 코드가 되지만 항상 함께 있기에 메모리 낭비가 된다.
4. 필드가 final일 경우, 필드 초기화시 매번 불필요한 필드도 초기화 해줘야 한다. 
5. 새로운 타입이 추가될 때마다 분기가 필요한 모든 메서드에 새로운 타입에 대응하는 코드를 작성해야 한다.
6. 인스턴스의 타입만으로 의미를 알기 쉽지 않다.

### 제안 - 서브타이핑
#### 클래스 계층구조를 활용하는 서브타이핑(subtyping)을 해보자. 
1. 계층 구조의 루트 추상 클래스를 정의한 뒤, 태그에 따라 동작이 달라지는 메서드들을 추상 메서드로 선언한다.
2. 태그 값에 상관없이 동일한 동작을 하는 메서드를 루트 클래스에 일반 메서드로 추가한다.
3. 루트 클래스를 확장한 구체 클래스를 의미별로 정의한다. 
4. 구체 클래스에서 루트 클래스의 추상 메서드를 각각의 의미에 맞게 구현한다.

➡️ 다음 코드는 계층 구조화가 된 Figure 클래스다.
``` java
public abstract class Figure {
    abstract double area();
}

public class Circle extends Figure {
    final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return  Math.PI * (radius * radius);
    }
}

public class Rectangle extends Figure {
    private final double length;
    private final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

- 기존 구조의 단점을 모두 없앴다. 
- switch 구문, 열거 타입, 불필요한 필드 정보들이 모두 분리돼서 사라졌다.  
(⇒ switch 문에서 default로 Assertion Error가 발생할 일도 없어졌다.)
- 타입 사이의 계층 관계를 반영할 수 있어 유연성 및 컴파일 타임 검사 능력도 높여준다.

---

### 📌 정리
- 태그 달린 클래스가 사용되어야 하는 상황은 거의 없고, 대부분 계층 구조로 해결이 된다. 
- 기존 클래스가 태그 필드를 사용해 구분하고 있다면 리팩터링을 고려할 필요가 있다. 
