# private 생성자나 열거 타입으로 싱글톤을 보증하라

> 작성자: 피터

## 목차
### 1. public static final 필드 방식의 싱글톤
### 2. 정적 팩터리 방식의 싱글톤
### 3. 열거 타입 방식의 싱글톤 - 바람직한 방법

<br/>

싱글톤이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 

ex) 함수와 같은 무상태(stateless)객체, 유일해야하는 시스템 컴포넌트  등

즉 객체를 최초 한 번만 메모리에 할당해두고 그 후 생성해둔 객체를 참조하여 사용하는 것을 말한다.

목차에서 3번의 경우 바람직한 방법이라고 나와있는 그 이유가 class를 싱글톤 패턴으로 만들면 이를 사용하는 클라이언트가 테스트하기 어려워질 수 있다. private 생성자를 갖고 있어 상속이 불가능하고, 생성 방식이 제한적이기 때문에 Mock 객체로 대체하기가 어려워 테스트하기 힘들다.
<br/>

1. public static final 필드 방식의 싱글톤
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() {...}
    	public void leaveTheBuilding() {...}
    }
    ```
    
    private 생성자는 public static field인 Elvis.INSTANCE를 초기화할때 딱 한번만 호출된다. 생성자는 private 말고 없으므로 만들어진 instance가 전체 시스템에서 하나뿐임이 보장된다. 왜냐하면 static 변수 선언과 함께 인스턴스가 생성되었고 생성자는 private이기 때문에 모든 외부 class에서 접근이 불가능하기 때문이다.강제로 리플렉션 API AccessibleObject.setAccessible을 사용할 수 있지만 그럴땐 예외를 던지게 하면 된다. 물론 일반적인 상황은 아니다. 
    
    장점
    
    - 싱글텀임이 명백히 드러난다.
    - 간결하다
2. 정적 팩터리 방식의 싱글톤
    
    ```java
    public class Elvis {
    	private static final Elvis INSTANCE = new Elvis();
    	private Elvis() {...}
    	public static Elvis getInstance() { return INSTANCE; }
    	public void leaveTheBuilding() {...}
    }
    ```
    
    정적 팩토리 메서드를 public static으로 제공하는 것이다. 생성자와 객체 모두 private이므로 외부 class에서 접근이 불가능하고 오직 객체 참조만 가능하다. 1번과 마찬가지로 리플렉션 예외는 똑같이 있다.
    
    장점
    
    - API 수정 없이 싱글톤임이 아니게 변경할 수 있다.
    - 정적 팩토리를 제네릭 싱글톤 팩토리로 만들 수 있다
    - 정적 팩토리의 메서드 참조를 supplier로 사용할 수 있다.
3. 열거 타입 방식의 싱글톤 - 바람직한 방법
    
    ```java
    public enum Elvis {
    	INSTANCE;
    		public void leaveTheBuilding(){...}
    }
    ```
    
    원소가 하나인 enum을 선언하는 것. 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글톤을 만드는 가장 좋은 방법이다. enum이 다른 interface를 구현하도록 선언할 수 없으므로 만드려는 싱글톤이 enum 외의 클래스를 상속해야한다면 다른 방법을 사용해야 한다. 
    
    장점
    
    - 간결
    - 직렬화
    - 리플렉션과 복잡한 직렬화 상황에서도 인스턴스 추가 생성을 막아준다.

1번과 2번의 경우 직렬화하려면 `Serializable`을 `implements`하는 것만으로 부족하다.  모든 instance를 `transient`로 선언하고 `readResolve`메소드를 제공해야한다. 그렇지 않으면 역직렬화 할때 새로운 인스턴스가 만들어진다.

### appendix

1. 리플렉션
구체적인 클래스 타입을 알지 못하더라도 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API

2. supplier
Java8에서는 **@FuncionalInterface** 어노테이션을 가진 다양한 함수형 인터페이스를 제공하며 supplier는 그 중 하나 이다. 함수형 인터페이스는 1개의 추상 method를 가지고 있는 인터페이스를 의미하며 supplier의 경우 get method가 추상화 되어있다. 
3. 직렬화
현재 데이터(structure, object)의 상태를 영속적으로 저장하거나 다른 환경으로 전달(네트워크 통신 등)하기 위해 바이트 스트림(stream of bytes) 형태로 연속전인(serial) 데이터로 변환하는 포맷 변환 기술
    
4. transient
Serialize하는 과정에 제외하고 싶은 경우 선언하는 키워드
