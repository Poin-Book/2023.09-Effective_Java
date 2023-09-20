# 생성자 대신 정적 팩터리 메서드를 고려하라

> 작성자: @hw130

## 목차
- [생성자 및 정적 팩터리 메서드 소개](#생성자_및_정적_팩터리_메서드_소개)  
- [정적 팩터리 메서드 장점]  
  - [장점 1. 이름을 가질 수 있다.](#장점_1._이름을_가질_수_있다.)  
  - [장점 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.](#장점_2._호출될_때마다_인스턴스를_새로_생성하지_않아도_된다.)
  - [장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.](#장점_3._반환_타입의_하위_타입_객체를_반환할_수_있는_능력이_있다.)
  - [장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.](#장점_4._입력_매개변수에_따라_매번_다른_클래스의_객체를_반환할_수_있다.)
  - [장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.](#장점_5._정적_팩터리_메서드를_작성하는_시점에는_반환할_객체의_클래스가_존재하지_않아도_된다.)  
- [정적 팩터리 메서드 단점]  
  - [단점 1. 상속을 하려면 public이나 protected 생성자가 필요하니, 정적 팩터리 메소드만 제공하면 하위 클래스를 만들 수 없다.](#단점_1._상속을_하려면_public이나_protected_생성자가_필요하니,_정적_팩터리_메소드만_제공하면_하위_클래스를_만들_수_없다.)  
  - [단점 2. 정적 팩터리 메소드는 프로그래머가 찾기 어렵다.](#단점_2._정적_팩터리_메소드는_프로그래머가_찾기_어렵다.)  
- [결론](#결론)



### 생성자 및 정적 팩터리 메서드 소개  
---
- 생성자(new)는 JAVA의 가장 기본적인 문법이고 클래스의 인스턴스를 만들 때 일반적으로 생성자를 사용한다. 하지만 개발자는 클래스에 별도의 정적 팩토리 메소드 (Static Factory Method)를 제공할 수 있다. 클래스의 인스턴스를 반환하는 정적 팩토리 메소드를 제공하면 생성자 대신 사용할 수 있다.
```java
public class TestClass {
	// 생성자를 통한 인스턴스 생성
	public TestClass() {}
	
	// 정적 팩토리 메소드를 통한 인스턴스 생성
	public static TestClass getInstance() {
		return new TestClass();
	}
}
```
- 위와 같이 간단한 예제 코드를 보면, TestClass는 생성자를 통해서 인스턴스를 만들 수도 있고 getInstance라는 정적 팩토리 메소드를 통해서도 인스턴스를 만들 수 있다.
- 이번 챕터에서는 정적 팩터리 방식의 장점과 단점에 대해 알아보도록 하겠다.  
  



### 장점 1. 이름을 가질 수 있다.
---
> 용도를 명확히 표현할 수 있다.
- 생성자의 이름은 클래스의 이름을 그대로 사용한다.
  - 파라미터를 다르게하여 여러개의 생성자를 만들 수 있지만, 각각의 생성자가 어떤 용도로 존재하는지는 명확히 알 수 없다.  
- 하지만 정적 팩토리 메소드는 메소드의 이름을 자유롭게 지을 수 있어서 그 용도를 명확히 표현할 수 있다.
```java
// BigInteger 클래스의 probablePrime 메소드
// 메소드의 이름으로 PrimeNumber, 즉 소수를 반환한다는 사실을 알 수 있다.
public static BigInteger probablePrime(int bitLength, Random rnd)

// probablePrime를 생성자로 만든다고 했을 때, 소수를 만든다는 사실을 쉽게 알 수 없다.
public BigInteger(int bitLength, Random rnd) 
```  


### 장점 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.  
---
- new 키워드를 사용하면, 객체는 무조건 새로 생성된다. 만약, 자주 생성될 것 같은 인스턴스는 클래스 내부에 미리 생성해 놓은 다음 반환한다면 코드를 최적화할 수 있을 것이다.   
   java.lang.Interger 클래스의 정적 메서드인 valueOf 의 구현을 한번 살펴보자.
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
- 전달된 i 가 캐싱된 숫자의 범위내에 있다면, 객체를 새로 생성하지 않고 **미리 생성된** 객체를 반환한다. 그렇지 않을 경우에만 new 키워드를 사용하여 객체를 생성하는 것을 확인할 수 있다.  

- 이렇게 인스턴스의 생성에 관여하여, 생성되는 인스턴스의 수를 통제할 수 있는 클래스를 **인스턴스 통제 (instance-controlled) 클래스**라고 한다. 인스턴스 통제를 하면, 클래스를 싱글턴(Singleton) 또는 인스턴스화 불가 (Noninstantiable) 클래스로 만들 수 있다. 또한 불변 값 클래스에서 동일한 값을 가지고 있는 인스턴스를 단 하나 뿐임을 보장할 수 있다. (a == b 일 때만, a.equals(b). 즉, a 와 b 가 같은 메모리 주소를 갖을 때만 둘의 '값'도 같을 수 있다.)

  - 여기서 싱글턴이란, 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말하며, 인스턴스화 불가 클래스란 생성자의 접근제어자를 private 으로 설정하여, 외부에서 new 키워드로 새로운 인스턴스를 생성할 수 없게 할 수 있다.  
      
 

### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.  
---
- 생성자를 사용하면 생성되는 객체의 클래스가 하나로 고정된다. 하지만 정적 팩터리 메서드를 사용하면, 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 갖을 수 있다.
```java
interface Computer {
    void powerOn();
}

class BasicComputer implements Computer {
    public void powerOn() {
        System.out.println("컴퓨터 입니다.");
    }
}

class StandardComputer implements Computer {
    public void powerOn() {
        System.out.println("표준 컴퓨터 입니다.");
    }
}

class PremiumComputer implements Computer {
    public void powerOn() {
        System.out.println("최고급 컴퓨터 입니다.");
    }
}

class Computers {
    public static Computer getBasicComputer() {
        return new BasicComputer();
    }

    public static Computer getStandardComputer() {
        return new StandardComputer();
    }

    public static Computer getPremiumComputer() {
        return new PremiumComputer();
    }
}
```
- 프로그래머는 Computer 의 하위 타입인 BasicComputer, StandardComputer, PremiumComputer 의 구현체를 직접 알 필요 없이, Computer 이라는 인터페이스를 사용하면 된다. 여기서 Computers 를 Computer 의 동반 클래스(Companion Class) 라고 한다.
- 하지만, 자바 8버전 부터는 인터페이스가 정적 메서드를 가질 수 있게 되어 동반 클래스는 더이상 필요 없어졌다. 인터페이스가 정적 메서드를 갖는 형태로 코드를 작성하면 아래와 같다.  
```java
interface Computer {
    static Computer createBasicComputer() {
        return new BasicComputer();
    }

    static Computer createStandardComputer() {
        return new StandardComputer();
    }

    static Computer createPremiumComputer() {
        return new PremiumComputer();
    }

    void powerOn();
}

class BasicComputer implements Computer {
    public void powerOn() {
        System.out.println("컴퓨터 입니다.");
    }
}

class StandardComputer implements Computer {
    public void powerOn() {
        System.out.println("표준 컴퓨터 입니다.");
    }
}

class PremiumComputer implements Computer {
    public void powerOn() {
        System.out.println("최고급 컴퓨터 입니다.");
    }
}
```
- 이렇게 구체적인 구현체를 사용자에게 공개하지 않고, 반환 타입을 인터페이스로 두게된다면 API의 개념적인 무게가 가벼워진다. 개발자는 API를 사용하기 위해 많은 개념을 익히지 않아도 된다. 인터페이스에 명세된 대로 동작한 객체를 얻을 것을 알기 때문이다.  
  


### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
---
- 클래스의 생성자는 해당 클래스의 인스턴스만 만들 수 있다. 하지만 정적 팩터리 메소드를 사용하면 같은 메소드라도 상황에 따라 다른 클래스 인스턴스를 반환할 수 있다. 예를 들어 적은 메모리를 사용해야하는 경우와 그 반대의 경우에 따라 다른 클래스 인스턴스를 반환함으로써 자원을 효율적으로 사용할 수 있다.
```java
// EnumSet의 정적 팩터리 메소드는 경우에 따라 RegularEnumSet, JumboEnumSet 두개의 클래스의 인스턴스를 반환한다.
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```
- 6번째 줄 부터 살펴보면, 원소의 수가 64개 이하라면 RegularEnumSet 클래스의 인스턴스를, 65개 이상이라면 JumboEnumSet 클래스의 인스턴스를 반환하는 것을 확인할 수 있다.  
  


### 장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. 
---
- 클래스가 존재해야 생성자가 존재할 수 있다. 하지만 정적 팩터리 메소드는 메소드와 반환할 타입만 정해두고 실제 반환될 클래스는 나중에 구현하는게 가능하다. 최근 프로그램의 규모는 점점 거대해지고 있고 여러 개발자를 포함한 팀들이 협업하여 하나의 프로그램을 완성한다. 원활한 협업을 위해 인터페이스까지 먼저 합의하여 만들고, 실제 구현체는 추후 만드는 식으로 업무가 진행된다. 이때 정적 팩터리 메소드를 사용할 수 있다.  
 
```java
// nextProviderClass 메소드 작성 시점에는 클래스가 존재하지 않아도 된다.
// 클래스가 작성되면 그때 클래스의 이름을 전달하면 된다. 
private Class<?> nextProviderClass() {
	...
    return Class.forName(cn, false, loader);
    ...
}
```  
  
  


### 단점 1. 상속을 하려면 public이나 protected 생성자가 필요하니, 정적 팩터리 메소드만 제공하면 하위 클래스를 만들 수 없다.
---
정적 팩터리 메소드만을 사용하게 하려면 기존 생성자는 private으로 해야하고 상속을 할 수 없게된다. 하지만 이 단점은 상속보다 컴포지션 사용을 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 장점으로 받아들일 수도 있다. 이 부분에 대한 자세한 내용은 이펙티브 자바 3판 아이템 18과 17 에 설명되어 있다. 이 내용은 해당 아이템을 공부할 때 더 자세히 다뤄보도록 하겠다.  

  


### 단점 2. 정적 팩터리 메소드는 프로그래머가 찾기 어렵다.
---
생성자는 기본 문법이기 때문에 개발자가 새로운 공부없이 바로 사용할 수 있다. 하지만 정적 팩터리 메소드는 개발자가 해당 클래스에 정적 팩토리 메소드가 있다는 사실을 알아야 한다. 이 단점을 완화하기 위해서는 널리 쓰이는 이름을 사용하여 정적 팩터리 메소드를 만들어야 한다.  
```java
// from: 매개변수 하나를 받아서 해당 타입의 인스턴스를 반환
Date d = Date.from(param);

// of: 매개변수 여러개를 받아서 적합한 타입의 인스턴스를 반환
Set<Rank> faceCards = EnumSet.of(param, param, param);

// valueOf: from, of와 유사
BigInteger bi = BigInteger.valueOf(param)

// instance, getInstnce: 매개변수에 맞는 인스턴스를 반환, 같은 인스턴스 보장 X
StackWalker luke = StackWalker.getInstance(param);

// create, newInstance: 매개변수에 맞는 인스턴스를 반환, 같은 인스턴스 보장 O
Object newArray = Array.newInstance(param)

// getType: getInstnce와 같으나 다른 클래스의 인스턴스를 반환
FileStore fs = Files.getFileStore(param);

// newType: newInstnce와 같으나 다른 클래스의 인스턴스를 반환
BufferedReader br = Files.newBufferedReader(param);

// type: getType, newType와 유사
List list = Collections.list(param);
```


### 결론
> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그래도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.



#### 📌 참고
> public 생성자를 쓰는 게 더 나은 경우

1. 간단한 객체 생성:  
간단하게 객체를 생성할 때는 생성자를 사용하는 것이 더 직관적일 수 있음.  

2. 단일 클래스의 경우:
특정 클래스에 대해 하나의 생성자만 필요하거나 객체 생성 로직이 단순한 경우에는 생성자를 사용하는 것이 간편하며 더 직관적일 수 있음.

3. 불변 객체 (Immutable Objects):
불변 객체를 생성할 때는 생성자를 통해 초기화하는 것이 일반적이고, 불변 객체는 한 번 생성되면 내부 상태를 변경할 수 없어야 하므로, 생성자를 통해 초기화하는 것이 안전하고 적절
 
