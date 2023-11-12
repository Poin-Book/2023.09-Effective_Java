# @Override 애너테이션을 일관되게 사용하라

> 작성자: 피터

## 목차
- [@Override 애너테이션을 일관되게 사용하라](#Override-애너테이션을-일관되게-사용하라)
  - [목차](#목차)
  - [본문](#@Override)
  - [정리](#정리)
## @Override
---

프로그래머가 메서드를 재정의 할때 자바에서 제공해주는 `@Override`어노테이션을 달고 재정의를 한다. 이는 상위 타입의 메서드를 재정의 했음을 컴파일러에게 알리는 역할을 한다.

이 어노테이션은 여러 버그들을 예방해 주는데 아래와 같은 상황이 있다.
```java
 public class Bigram {
	private final char first;
	private final char second;
	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}
	public boolean equals(Bigram b) {
		return b.first = first && b.second = second;
	}
	public int hashcode() {
		return 31 * first + second;
	}
	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<>();
		for(int i=0; i < 10; i++;) 
			s.add(new Bigram(ch, ch));
		System.out.println(s.size());
	}
}
```
main 메서드를 보면 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복하여 Set에 추가한 후 그 집합의 크기를 출력한다. Set은 중복을 허용하지 않으니 26이 출력될 거 같지만, 실제로는 260이 출력된다.

Bigram class에서 equals와 hashcode 메서드를 재정의 한것으로 보이지만 실제로는 재정의(override) 한것이 아니라 다중정의(overloading)를 했다. 

Bigram의 equals를 재정의하려면 매개변수 타입을 Object로 해야 하는데 그렇지 않아 equals를 새로 정의한 상황이 되었고, 결국 equals는 ==연산자와 같이 객체 실별성(indetify)만을 확인한다.

결국 main에서 set은 10개의 bigram을 서로 다른 객체로 인식하였고, 결국 260을 출력했다.

이러한 에러는 @Override를 달아 작성자의 의도를 컴파일러가 알아차리고 오류를 발생시킨다.

즉 @Override를 위의 equals에 달아주면 아래와 같이 오류가 발생한다.
```java
Bigram.java.10: method does not override or implement a method from a supertype
@Override public boolean equals(bigram b){
^
```
```java
public boolean equals(Object b) {
	if(!(o instanceOf Bigram))
		return false;
	Bigram b = (Bigram) 0;
	return b.first == first && b.second == second;
}
```
따라서 상위 클래스의 메서드를 재정의하려는 모든 메서드에는 @Override 어노테이션을 달아야 한다. 하지만 상위클래스의 추상 메서드를 재정의할 때는 굳이 @Override를 달지 않아도 된다. 구현하지 않은 추상메서드가 남아 있다면 컴파일러가 그 사실을 바로 알려주기 때문이다. 

IDE는 @Override를 일관되게 사용하도록 부추긴다. IDE 설정에 따라 @Override가 달려있지 않은 메서드가 실제로는 재정의를 했다면 경고를 준다. 또한 대부분의 IDE에서는 재정의할 메서드를 선택하면 @Override를 자동으로 붙여준다.

@Override는 인터페이스 메서드를 재정의할 때도 사용할 수 있지만 구현한 인터페이스에 디폴트 메서드가 없다면 구현한 메서드에서는 @Override를 생략해도 된다.

## 정리

재정의한 모든 메서드에 @Override 어노테이션을 의식적으로 달아야 한다.
실수했을 경우 컴파일러가 알려준다.
상위 클래스의 추상 메서드를 재정의한 경우엔 달지 않아도 된다.(단다고 해서 해로울 것은 없다.)
