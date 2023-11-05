# 제목

> 작성자: 루키

## 목차
- [제목](#제목)
  - [목차](#목차)
  - [본문](#본문)
    - [Item 34 int 상수 대신 열거 타입을 사용하라](#item-34-int-상수-대신-열거-타입을-사용하라)
    - [1. 열거 타입이란?](#1-열거-타입이란)
    - [2. 열거 타입의 예시](#2-열거-타입의-예시)
    - [3. 열거 타입 동작의 분리](#3-열거-타입-동작의-분리)
## 본문
### Item 34 int 상수 대신 열거 타입을 사용하라
---
### 1. 열거 타입이란?

> 일정 개수의 상수 값을 정의한 다음 그 외의 값은 허용하지 않는 타입
> 
- 예를 들어서 태양계의 행성, 카드게임의 카드 종류 같은 것이 있다.
- 열거 패턴을 타입으로 올려버린 것
    - 정수 열거 패턴
        
        ```java
        public static final int APPLE_FUJI = 0;
        public static final int APPLE_PIPPIN = 1;
        
        public static final int ORANGE_NAVEL = 0;
        public static final int ORANGE_TEMPLE = 1;
        ```
        
        - 타입 안전을 보장할 수 없다.
            - 오렌지를 Pass 해야할 메서드에 사과를 Pass 해도 컴파일러는 경고하지 않음
        - 가독성도 그렇게 좋지 않다.
            - 사과용 상수는 모두 APPLE로 시작 오렌지용 상수는 ORANGE로 시작
                
                > 정수 열거 패턴을 위한 별도의 namespace를 지원하지 않기 때문이다.
                > 
                - namespace란?
                    - 객체 또는 모듈을 구분할 수 있는 범위
                    - 하나의 네임 스페이스 안에서는 하나의 이름이 하나의 객체를 가리킨다.
                    - Java에서는 패키지 정의를 통해서 관리를 함
                    - 예를 들어서
                        - java.util.List 와 java.awt.List는 다른 객체를 가리킨다.
            - 따라서 구분하기 위해서 저렇기 네이밍을 해주는 것
        - 프로그램이 깨지기 쉬움
            - 컴파일하면 상수에 그 값이 각인이 되어버림
            - 따라서 그 값이 바뀌면 컴파일을 다시 해야하는데
            - 서버야 상대적으로 교체가 용이하지만 클라이언트의 경우 사용자가 직접 해줘야 하므로
            - 클라이언트측에서 오작동을 유발할 가능성이 크다.
    - 문자열 열거 패턴
        - 정수 대신 문자열 상수를 사용
        - 다만 이건 더 좋지 않음
            - 문자열 상수의 이름 대신 문자열 값을 그대로 하드 코딩하기 때문
            - 저 문자열에 오타가 있어도 확인할 길이 없다 → 런타임 버그
            - Java에서 문자열 비교는 equals() 메서드를 사용하므로 이에 따른 성능 저하
- 따라서 등장한 것이 열거 타입이다.
    
    ```java
    public enum Apple {FUJI, PIPPIN}
    public enum Orange {NAVEL, TEMPLE}
    ```
    
    - 완전한 형태의 클래스 이다.
        - 따라서 단순한 정수인 다른 언어의 열거 타입보다 강력함
    - 열거 타입의 사상
        - 그 자체로 클래스.
        - 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static 필드로 공개함
        - 외부에서 접근 가능한 생성자를 제공하지 않아서 사실상 final이라고 할 수 있다
            - 외부에서 값을 수정할 길이 없다.
        - 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없음
            - 열거 타입 선언으로 만들어진 인스턴스는 오직 하나만 존재 함.
        
        > 열거 타입은 인스턴스 통제된다.
        > 
        - 싱글턴은 원소가 하나뿐인 열거 타입 열거 타입은 싱글턴을 일반화 시킨 것이다.
    - 다음의 장점이 있음
        - 컴파일 타임 안전성
            - 위 코드에서 Apple 열거 타입을 매개 변수로 받는다고 했을때
            - 다른 타입의 값을 넘기면 오류가 난다.
        - Namespace 존재
            - 이름이 같은 상수도 공존이 가능
        - 상수의 변경에 로버스트 하다
            - 추가나 순서를 바꾸는 등등
            - 공개되는 것이 오직 필드의 이름뿐
            - 따라서 컴파일시 그 값이 각인되지 않음
        - toString 메서드는 알아서 잘 구현이 되어있음
        - 임의의 메서드나 필드의 추가가 가능
        - 임의의 인터페이스를 구현하게 할 수 있음
        - 또한 Object 메서드 (equals, toStirng …)를 잘 구현해둠
        - Comparable과 Serializable을 구현했고 그 직렬화 형태도 변형에 로버스트함

### 2. 열거 타입의 예시

---

```java
package effectivejava.chapter6.item34;

// 코드 34-3 데이터와 메서드를 갖는 열거 타입 (211쪽)
public enum Planet {
		// 괄호 안의 숫자는 생성자에 넘겨지는 매개 변수 (mass, radius)
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);
		
		// 열거 타입은 불변 따라서 모든 필드는 final이어야 함
    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
				// 최적화
        surfaceGravity = G * mass / (radius * radius);
    }
		
		// 접근자 메서드
    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
		
		// 무게를 측정하고 싶은 물체의 질량을 받아서 무게를 반환
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

> 열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 제공하는 values 메서드를 제공
> 

```java
// 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력한다. (212쪽)
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
													// 여기서 p는 p.toString()과 동일하게 본다
   }
}
```

- 물론 여기서 toString을 재정의 하면 출력되는 이름을 바꿀 수 있다.
- 만약 Planet에서 상수 하나를 제거한다면?
    - 그냥 출력되는 문자열 한줄이 줄어든다.
    - 클라이언트는 다시 컴파일하면 유의미한 에러를 던져 주므로 수정이 용이함
- 열거 타입을 선언한 클래스 또는 해당 패키지에서만 유효한 것들은 private나 package-private로 구현
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고 특정 톱레벨 틀래스에서만 쓴다면 해당 클래스의 멤버 클래스로 만든다

### 3. 열거 타입 동작의 분리

---

> 타입 마다 동일한 메서드가 다른 동작을 해야할 때
> 
1. 우선 간단히 생각을 해봐서 switch를 써보자
    - 사실 코드 안봐도 상수의 갯수가 많아지면 코드 더러워지고 유지보수가 어려워진다는 것에는 공감을 할 것
    - 또한 새로운 상수가 추가된다면 case문도 추가를 해야해서 까먹는다면 코드가 깨진다.
2. 상수별 메서드 구현
    
    ```java
    public enum Operation {
    		// 추상 메서드의 구현을 상수마다 해준다
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
    		
    		// 추상 메서드
    		public abstract double apply(double x, double y);
    }
    ```
    
    - 이러면 재정의를 안했을때 컴파일 오류가 나므로 코드가 깨지지 않는다.

1. 상수변 메서드 구현 + 상수별 데이터
    
    ```java
    public enum Operation {
    		// 추상 메서드의 구현을 상수마다 해준다
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
    
        Operation(String symbol) { this.symbol = symbol; }
    
        @Override 
    		public String toString() { return symbol; }
    		// 추상 메서드
    		public abstract double apply(double x, double y);
    }
    
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    									// op = op.toString()
    }
    ```
    
    - 또한 toString이 있다면 문자열에서 열거 타입으로 바꿔주는 fromString도 있다
        
        ```java
        // 코드 34-7 열거 타입용 fromString 메서드 구현하기 (216쪽)
        // Operation 상수가 stringToEnum 맵에 추가되는 시점은
        // 열거 타입 상수 생성 후 정적 필드가 초기화 될때이다.
        private static final Map<String, Operation> stringToEnum =
        				// Stream 프레임워크를 사용 values가 반환한 배열을 순회
                Stream.of(values()).collect(
                        toMap(Object::toString, e -> e));
        
        // 지정한 문자열에 해당하는 Operation을 (존재한다면 -> Optional) 반환한다.
        public static Optional<Operation> fromString(String symbol) {
            return Optional.ofNullable(stringToEnum.get(symbol));
        }
        ```
        

1. 상수별 메서드 구현에서 코드의 공유
    
    ```java
    enum PayrollDay {
    	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    	private static final int MINS_PER_SHIFT = 8 * 60;
    	
    	int pay(int minuteWorked, int payRate) {
    		int basePay = minuteWorked * payRate;
    		
    		int overtimePay;
    		
    		// 상수가 추가된다면 여기를 무조건 바꿔주어야 함
    		switch(this) {
    			case SATURDAY: case SUNDAY: // 주말
    				overtimePay = basePay / 2;
    				break;
    			default:
    				overtimePay = minuteWorked <= MINS_PER_SHIFT ?
    					0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
    		}
    		return basePay + overtimePay;
    	}
    }
    ```
    
    - 이걸 어떻게 바꿀까?
        1. 잔업 수당을 계산하는 코드를 모든 상수에 넣기
        2. 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성 후 각 상수가 호출
    - 다만 두 방식 모두 코드가 길어져서 가독성이 떨어짐 또한 오류 발생 가능성도 높아짐
    - 또한 PayrollDay에 평일 잔업수당 계산용 메서드 overtimePay를 구현해놓고 주말 상수에서만 재정의해 쓰면 길어지는 건 막을 수 있음
    - 다만 switc문을 썼을때와 같은 단점이 나타남
        - 새로운 상수를 추가하면서 overtimePay 메서드를 재정의하지 않으면 평일용 코드를 그대로 물려 받는다.
    - 그래서 어떻게하면 되냐
        
        > 전략 열거 타입 패턴
        > 
        
        ```java
        enum PayrollDay {
            MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
            THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
            SATURDAY(WEEKEND), SUNDAY(WEEKEND);
        
            private final PayType payType;
        
            PayrollDay(PayType payType) {
        				// 잔업 수당 전략 
        				this.payType = payType; 
        		}
            
            int pay(int minutesWorked, int payRate) {
                return payType.pay(minutesWorked, payRate);
            }
        
            // 전략 열거 타입 (맴버 열거 타입)
            enum PayType {
                WEEKDAY {
                    int overtimePay(int minsWorked, int payRate) {
                        return minsWorked <= MINS_PER_SHIFT ? 0 :
                                (minsWorked - MINS_PER_SHIFT) * payRate / 2;
                    }
                },
                WEEKEND {
                    int overtimePay(int minsWorked, int payRate) {
                        return minsWorked * payRate / 2;
                    }
                };
        
                abstract int overtimePay(int mins, int payRate);
                private static final int MINS_PER_SHIFT = 8 * 60;
        
                int pay(int minsWorked, int payRate) {
                    int basePay = minsWorked * payRate;
                    return basePay + overtimePay(minsWorked, payRate);
                }
            }
        
            public static void main(String[] args) {
                for (PayrollDay day : values())
                    System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
            }
        }
        ```
        
    
    - 그러면 switch 문은 쓰면 안되나?
        
        ```java
        public class Inverse {
            public static Operation inverse(Operation op) {
                switch(op) {
                    case PLUS:   return Operation.MINUS;
                    case MINUS:  return Operation.PLUS;
                    case TIMES:  return Operation.DIVIDE;
                    case DIVIDE: return Operation.TIMES;
        
                    default:  throw new AssertionError("Unknown op: " + op);
                }
            }
        
            public static void main(String[] args) {
                double x = Double.parseDouble(args[0]);
                double y = Double.parseDouble(args[1]);
                for (Operation op : Operation.values()) {
                    Operation invOp = inverse(op);
                    System.out.printf("%f %s %f %s %f = %f%n",
                            x, op, y, invOp, y, invOp.apply(op.apply(x, y), y));
                }
            }
        }
        ```
        
        - 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이다.
        - 추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면
            - 직접 만든 열거 타입이라도 위 방식을 적용
        - 열거 타입의 성능은 정수 상수와 다르지 않음
            - 물론 열거 타입을 메모리에 올리는 공간과 초기화 시간이 있지만 유의미하지는 않다.
    
    ### 4. 그래서 열거 타입을 언제 쓰라는 말이에요
    
    ---
    
    > 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 써라
    > 
    - 예시로 태양계 행성, 한 주의 요일 등등이 있다.
    
    > 허용하는 값 모두를 컴파일 타임에 알고 있을때 사용 가능
    > 
    - 메뉴 아이템, 연산 코드, 명령줄 플래그 등
    
    > 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요 없다
    > 
    - 나중에 상수가 추가되어도 바이너리 수준에서 호환이 가능
    
    ### 5. 요약
    
    ---
    
    1. 열거 타입은 정수 상수보다 좋다
    2. 열거 타입이 명시적 생성자나 메서드 없이 쓰임
        - 다만 다음의 경우에는 필요
            - 각 상수를 특정 데이터와 연결 지을때
            - 상수마다 다르게 동작하게 할때
    3. 하나의 메서드가 상수별로 다르게 동작할때
        - 상수별 메서드 구현을 사용하라
    4. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하라