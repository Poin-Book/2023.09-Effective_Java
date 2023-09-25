# equals는 일반 규약을 지켜 재정의하라

> 작성자: 다나

## 목차
+ equals를 재정의 하지 않아야 할 상황
+ equals를 재정의해야 할 상황
+ equals 메서드를 재정의할 경우 따라야 할 일반 규약
+ 양질의 equals 메서드 구현 방법
+ equals 메서드 구현 시 주의사항
+ 핵심 정리

`equals` 메서드를 적합하게 재정의하는 것은 까다로워 최대한 재정의 하지 않는 것이 좋지만,
`equals` 메서드를 재정의하지 않고 그냥 두면, 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

## ☑️ equals를 재정의 하지 않아야 할 상황

### 1. 각 인스턴스가 본질적으로 고유한 경우

값 클래스(`Integer`, `String`)가 아닌 `Thread` 와 같이 동작하는 개체를 표현하는 클래스라면, 재정의할 필요가 없다.

### 2. 인스턴스의 논리적 동치성(logical equality)를 검사할 일이 없는 경우

`java.util.regex.Pattern` 은 `equals`를 재정의해서 두 `Pattern`의 인스턴스가 같은 정규표현식을 나타내는지를 검사한다.

하지만, 클라이언트가 필요하다 판단하지 않을 수 있기 때문에 재정의 하지 않고 `Object`의 기본 `equals`만으로 해결된다.

### 3. **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우**

ex) Set - AbstarctSet, List - AbstractList, Map - AbstractMap 

```java
public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
     public boolean equals(Object o) {
          if (o == this)
              return true;

          if (!(o instanceof Set))
              return false;
          Collection<?> c = (Collection<?>) o;
          if (c.size() != size())
              return false;
          try {
              return containsAll(c);
          } catch (ClassCastException | NullPointerException unused) {
              return false;
          }
      }
 }
```

### 4. 클래스가 private이거나 package-private이고 *equals* 메서드를 호출할 일이 없는 경우

```java
// equals 함수 호출을 막고 싶다면 다음과 같이 구현하자
@Override public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```

## ☑️ equals를 재정의해야 할 상황

> **object identity(객체 식별성)이 아니라 logical equality(논리적 동치성)을 확인해야**는데 
상위 클래스의 `equals`가 **논리적 동치성**을 비교하도록 재정의되지 않았을 경우
→ **즉 객체가 같은지가 아니라 값이 같은지를 확인해야할 경우**
> 

주로 값 클래스들이 해당된다. ex) `Integer`, `String`

값 클래스라 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(*아이템 1*) 라면 `equals`를 재정의하지 않아도 된다. `Enum`과 같은 ..

## ☑️ equals 메서드를 재정의할 경우 따라야 할 일반 규약

> `equals` 메서드는 동치관계를 구현하며, 다음을 만족한다.
> 
1. **refelexivity(반사성)** : null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 **true**이다.
2. **symmetry(대칭성)** : null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`가 true면 `y.equals(x)`도 **true**이다.
3. **transitivity(추이성)** : null이 아닌 모든 참조 값 x, y, z에 대해, `x.equals(y)`가 **true**이고 `y.equals(z)`도 **true**면 `x.equals(z)`도 **true**다.
4. **consistency(일관성)** : null이 아닌 모든 참조 값 x,y에 대해 , `x.equals(y)`를 반복해서 호출하면 항상 **true**를 반환하거나 항상 **false**를 반환한다.
5. **not-null** : null이 아닌 모든 참조 값 x에 대해, `x.equals(null)`는 **false**다.

`equals`메서드가 쓸모 있으려면 **동치류**에 속한 어떤 원소와도 서로 교환할 수 있어야 한다고 한다.

> 🔖 **동치 관계**: 집합을 서로 같은 원소들로 이뤄진 부분 집합으로 나누는 연산이다. 이 부분 집합을 동치류라 한다.
![image](https://github.com/Poin-Book/2023.09-Effective_Java/assets/85955988/1fb9140b-488e-4b84-b8f7-930f8398c4de)


### 1. 반사성 : 객체는 자기 자신과 같아야 한다.

 너무나 당연함.

### 2. 대칭성 : 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

```java
public final class CaseInsensitiveString {
	private final String s;

	public CaseInsensitiveString(String s){
			this.s = Object.requiredNonNull(s)
}

// 대칭성 위배!
@Override public boolean equals(Object o){
	if (o instanceOf CaseInsensitiveString)
			return s.equalsIgnoreCase(
					((CaseInsensitiveString) o).s);
	if (o instanceOf String) // 한 방향으로만 작동
			return s.equalsIgnoreCase((String) o);
	return false;
	}
}
```

```java
CaseInsensitiveString cis = new CaseInsensitiveString(”Polish”);
String s = “polish”;
```

`cis.equals(s)`는 **true**를 반환하지만 `String`의 `equals` 는 `CaseInsensitiveString` 을 모르기 때문에, `s.equals(cis)` 에서 `false` 가 나오게 되어 대칭성에 위반된다.

컬렉션에 `cis`를 넣어 `list.contains(c)`를 호출하면 **false**를 반환한다. 하지만 OpenJDK  버전이 바뀌거나 다른 JDK에서는 **true**를 반환하거나 런타임 예외를 던질 수 있다.

### **해결 방법 → `CaseInsensitiveString`끼리만 비교하도록 한다.**

```java
 @Override public boolean equals(Object o) { // 빼주면 정상 동작
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s); 
								// equalsIgnoreCases는 대소문자 구분없이 비교
}
```

### 3. 추이성 : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다.

해당 속성은 상위 **클래스에 없는 새로운 필드를 하위 클래스에 추가**하는 상황에서 어기기 쉽다. 미리 결론부터 말하지만, 어떤 방식을 해봐도 상속을 통해서는 `equals` 조건을 충족시킬 수 없다.

```java
// Point 클래스와 이를 확장하여 색상 필드를 추가한 ColorPoint 클래스
public class Point {  // 부모
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}
```

```java
public class ColorPoint extends Point {  // 자식
	private final Color color;
    
    public ColorPoint(int x, int y, Color color){
    	super(x,y);
        this.color = color;
    }
```

**1) 잘못된 코드 - 대칭성 위배**

이 경우에 `equals` 메서드를 그대로 둔다면 `Point`의 구현이 상속되어 색상 정보는 무시한 채 비교를 수행한다. 

```java
@Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
             return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

```java
 Point p = new Point(1,2);
      ColorPoint cp = new ColorPoint(1,2, Color.RED);
      p.equals(cp);    // true
      cp.equals(p);    // false
```

위처럼 비교 대상이 `ColorPoint` 객체이고, 위치와 색상이 같을 때만 `true`가 반환되도록 `equals`를 작성하였다.
하지만 `Point`와 `ColorPoint`를 비교한 결과와, 둘을 바꿔서 비교한 결과가 다르기 때문에 **대칭성**에 위배된다.
`ColorPoint.equals`가 `Point`와 비교할 때는 색상을 무시하도록 `equals`를 작성했을 때를 생각해보자.

**2) 잘못된 코드 - 추이성 위배**

```java
@Override public boolean equals(Object o) {
  if (!(o instanceof Point))
   return false;

  //o가 일반 Point면 색상을 무시하고 비교한다.
  if (!(o instanceof ColorPoint))
    return o.equals(this);

  // o가 ColorPoint면 색상까지 비교한다.
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2);    // true 
p2.equals(p3);    // true 
p1.equals(p3);    // false
```

수정한 `equals` 메서드로 p1과 p2, p2와 p3를 각각 비교하면 **true**지만  p1과 p3를 비교한 경우 **false**를 반환하므로 **추이성**을 위반하게 된다. 
p1과 p2, p2와 p3를 비교할 땐 색상을 무시했지만 p1과 p3는 색상까지 비교했기 때문이다.

```java
 //SmellPoint.java의 equals
    @Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof SmellPoint))
        return o.equals(this);
      return super.equals(o) && ((SmellPoint) o).color == color;
    }
```

```java
ColorPoint p1 = new ColorPoint(1,2, Color.RED);
SmellPoint p2 = new SmellPoint(1,2);
p1.equals(p2);

// 1. ColorPoint의 equals: 2번째 if문 때문에 SmellPoint의 equals로 비교
// 2. SmellPoint의 equals: 2번째 if문 때문에 ColorPoint의 equals로 비교
// 3. 1~2 무한 재귀로 인한 StackOverflow Error
```

새로운 클래스 `SmellPoint`를 만들고 `equals`는 같은 방식으로 구현한 경우 `myColorPoint.equals(mySmellPoint)`를 호출하면 StackOverflowError을 일으킨다.
이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제로, 
**구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**

### 3) 잘못된 코드 - 리스코프 치환 원칙 위배

> 🔖 **리스코프 치환 원칙**
> 
> 
> 부모 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.
> 

```java
public class Point{
  @Override public boolean equals(Object o) {
     if (o == null || o.getClass() != getClass())
       return false;
     Point p = (Point) o;
     return p.x == x && p.y == y;
  }
}
```

객체 지향 추상화인 `instanceof` 검사를 `getClass` 검사로 바꾸면 규약도 지키고 값도 추가하면서 
구체 클래스를 상속 가능한 것 처럼 보이지만, 이는 **리스코프 치환 원칙**을 위반한다. 
같은 구현 클래스의 객체와 비교할때만 **true** 를 반환하고, `Point`의 **하위 클래스**를 비교할때는 항상 **false**를 반환하게 될 것이다.

### 💡문제 해결

**1 ) 상속 대신 컴포지션을 사용하라 (equals 규약을 지키면서 값 추가하기)**

> 컴포지션: 기존 클래스가 새로운 클래스의 구성 요소로 쓰인다.
> 

```java
public class ColorPoint{
  private final Point point;
  private final Color color;

	public ColorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	// 이 ColorPoint의 Point 뷰를 반환한다. 
  public Point asPoint(){ // view 메서드 패턴
    return point;
  }

  @Override public boolean equals(Object o){
    if(!(o instanceof ColorPoint)){
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```

`Point`를 상속하는 대신 `Point`를 `ColorPoint`의 *private* 필드로 두고, `ColorPoint`와 같은 위치의 일반 `Point`를 반환하는 뷰(view 메서드)를 *public*으로 추가하는 식이다.

ex) `java.sql.Timestamp` 는 `java.util.Date` 를 확장한 후 `nanoseconds` 필드를 추가함. 그 결과로 `Timestamp` 의 `equals`는 **대칭성** 위배하며 `Date` 객체와 한 컬렉션에 넣거나 서로 섞어 사용하면 엉뚱하게 동작할 수 있다. 

**2) 추상 클래스의 하위 클래스 사용하기**

추상 클래스의 하위 클래스에서는 `equals` 규약을 지키면서도 값을 추가할 수 있다.

상위 클래스의 인스턴스를 직접 만드는 게 불가능하기 때문에, 하위 클래스끼리의 비교가 가능하다.

### 4. 일관성 : 두 객체가 같다면 ( 어느 하나 혹은 두 객체가 모두 수정되지 않는 한 ) 앞으로도 영원히 같아야 한다.

가변 객체의 경우 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있다.

불변 객체는 한번 다르면 끝까지 달라야 한다.

**클래스가 불변이든 가변이든 `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.**

ex) `java.net.URL` 의 `equals` 는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다. 호스트의 이름을 IP 주소로 바꾸려면 네트워크를 통과해야 하는데, 이는 외부 요인이므로 결과를 신뢰할 수 없다. 따라서 **equals는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행**해야 한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/66cd63ec-50d7-4011-9e2a-0ccf0042806a/92072329-eb0b-48b6-84b6-9a4d419e6b43/Untitled.png)

## 5. Not Null : 모든 객체가 null이 아니어야 한다.

**잘못된 명시적 null 검사**

```java
@Override
public boolean equals(Object o) {
  if(o == null) {
      return false;
  }
}
```

**올바른 묵시적 null 검사**

```java
@Override public boolean equals(Object o){
	if (!o instance of MyType))
    	return false
    MyType mt = (MyType) o;
 }
```

`instance of` 를 사용하면, `NullPointerException` 과 잘못된 타입이 들어왔을 때 일어날 수 있는 `ClassCastException` 까지 방지해주므로 명시적으로 `null` 검사를 하지 않아도 된다.

## 💡 양질의 equals 메서드 구현 방법

### 1. 연산자를 사용해 입력이 자기 자신의 참조인지 확인

성능 최적화용으로, 자기 자신이면 **true** 를 반환한다.

```java
 if (o == this)
      return true;
```

### 2. instanceof 연산자로 입력이 올바른 타입인지 확인

가끔 해당 클래스가 구현한 특정 인터페이스를 비교할 수도 있다.
이런 인터페이스를 구현한 클래스라면 equals에서 (클래스가 아닌) 해당 인터페이스를 사용해야한다.

`Set`, `ListMapMap.Entry` 등의 컬렉션 인터페이스들이 여기 해당한다.

```java
 if (!(o instanceof PhoneNumber))
      return false;
```

### 3. 입력을 올바른 타입으로 형변환

```java
 PhoneNumber pn = (PhoneNumber)o;
```

### 4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사

모든 필드가 일치하면 `true`를, 하나라도 다르면 `false` 를 반환한다. 
2단계에서 인터페이스를 사용했다면, 필드 값을 가져올때 해당 인터페이스의 메서드를 사용해야 한다.

```java
 return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
```

### 5. **타입에 따른 비교 방법**

1. 기본 타입 필드 : `==` 연산자
2. 참조 타입 필드 : 각각의 `equals` 메서드
    
    `float` / `double` : `Float.compare(float, float)` / `Double.compare(double, double)` 로 각각 비교 ( 특수한 부동소수 값등을 다뤄야하기 때문 ) 
    
3. 배열 : 모든 원소가 핵심 필드라면 `Arrays.equals()` 사용
4. `null` 정상 값 취급 참조 타입 필드 : `Object.equals(Object, Object)` 사용
5. 비교하기 복잡한 필드를 가진 클래스(CaseInsensitiveString과 같은)는 그 필드의 표준형을 저장해둔 후 표준형끼리 비교하자.  

> 🔖 **equals의 성능을 향상시키는 방법**
> 
> 
> 다를 가능성이 크거나 비교하는 비용이 싼 필드를 먼저 비교한다. 
> **동기화용 락 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다**.
> 

## ⚠️ 주의사항

1. `equals`를 재정의 할 땐 `hashcode`도 반드시 재정의하자 *(아이템 11)*
2. 필드들의 동치성만 검사해도 `equals`규약을 잘 지킬 수 있으니 복잡하게 해결하려 들지 말자.
3. `Object` 외의 타입을 매개변수로 받는 `equals` 메서드는 선언하면 안된다.
    
    ```java
    public boolean equals(MyClass o){  // 입력 타입은 반드시 Object
       ...
    }
    ```
    
    해당 코드는 `Object.equals`를 재정의 한것이 아니라 다중정의한 것이 된다. 이 메서드는 하위 클래스에서의 `@Override` 애너테이션이 긍정 오류(false positive: 거짓 양성)을 내게하고 보안 측면에서도 잘못된 정보를 준다.
    
    `@Override` 애너테이션을 일관되게 사용한다면, 실수를 예방할 수 있다.*(아이템 40)* 아래 코드는 컴파일되지 않기 때문에 무엇이 문제인지 정확히 알 수 있다.
    
    ```java
    @Override public boolean equals(Myclass o){  // 컴파일 되지 않음
    	...
    }
    ```
    
    구글이 만든 `AutoValue` 오픈소스 프레임워크를 사용하면 `equals`를 작성하고 테스트하는 작업을 대신해준다.
    
## 📌 핵심 정리 
**꼭 필요한 경우가 아니면 `equals`를 재정의하지 말자.** 
많은 경우에 `Object.equals`가 원하는 비교를 정확히 수행해주기 때문이다. 재정의해야 할 때는, 그 클래스의 핵심 필드를 모두 빠짐없이 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.
