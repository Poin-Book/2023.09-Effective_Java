# 가변인수는 신중히 사용하라

> 작성자: 워니

## 목차
#### 1. 가변인수(varargs)란?
#### 2. 문제가 되는 경우
#### 3. 성능 문제와 해결책
#### 📌 요약

---

### 가변인수(varargs)란?
매개변수를 동적으로 받을 수 있는 방법으로 Java 5부터 지원되기 시작한 기능

``` java
public void printArgs(String ... strs){
	for(String str : strs){
		System.out.println(str);
	}
}
```

매개변수에 `...` 을 사용하여 가변인수를 설정할 수 있고, 0개 이상의 인수를 받을 수 있다.
<br></br>

> #### 가변인수 메서드의 흐름
1. 인수의 개수와 길이가 같은 배열을 생성한다.  
2. 생성된 배열에 인수들을 저장하여 메서드에 전달한다.  
3. 메서드에서는 해당 배열을 사용한다.

이처럼 메서드를 호출 할 때마다 새로 배열을 생성해서 전달해주는데, 이 부분도 결국 비용이고, 낭비일 수 있다. 

---

### 문제가 되는 경우
인수의 개수가 1개 이상이어야 하는 경우에는 메서드에서 인수의 개수를 검증해야 한다.

``` java
public void min(int... args) {
	if (args.length == 0) {
		throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
	}
	int min = args[0];
	for (int i = 1; i < args.length; i++) {
		if (args[i] < min) {
			min = args[i];
		}
	}
	System.out.println(min);
}
```

다음과 같이 최솟값을 구하는 메서드의 경우, 인수가 0개라면 최솟값을 구할 수 없다.  
그렇기에 상단에 가변인수 배열의 길이를 검증하는 로직이 들어가야하는데, 이 경우 컴파일 시점이 아닌 런타임 시점에 예외가 발생할 수 있다. 
<br></br>

> #### 💡 1개 이상의 인수를 받아야 하는 경우 해결책
➡️ 평범한 매개변수를 첫번째로 받고, 가변인수를 두번째로 받는다.

``` java
public void min(int firstArg, int... args) {
	int min = firstArg;
	for(int arg : args) {
		if(arg < min) {
			min = arg;
		}
	}
	System.out.println(min);
}
```

이렇게 하면 첫번째 인수를 최솟값으로 지정하여 가변인수 배열을 비교해 최솟값을 구할 수 있다. 

---

### 성능 문제와 해결책
가변인수는 인수 개수가 정해지지 않고 가변적일 때 유용한 방식이다. 
하지만, 매번 배열을 새로 생성해서 할당해야 하는 비용적인 문제가 있다.  
그렇기에 가변인수를 무작정 쓰기보다는, 메서드의 호출 패턴을 분석하여 오버로딩을 활용할 수도 있다.  

만약 가변인수를 받는 메서드에서 메서드 호출의 95% 이상이 인수의 개수를 3개 이하로 받을 경우, 다음과 같이 다중정의를 이용해 배열이 새로 생성되는 비용을 줄일 수 있다. 

``` java
public void method() { ... }
public void method(int a1) { ... }
public void method(int a1, int a2) { ... }
public void method(int a1, int a2, int a3) { ... }
public void method(int a1, int a2, int a3, int... args) { ... }
```

이렇게 오버로딩을 할 경우, 메서드 호출의 95%는 가변인수를 사용하지 않고 5%만이 가변인수를 사용해 배열을 새로 생성할 것이다. 
따라서 유연성을 지키면서도 성능 이슈를 최소화 할 수 있다.
<br></br>

> **_대표 예시) EnumSet_**

![image](https://github.com/kiwijomn/test-repo/assets/116738827/792b3cc4-e4e9-4629-9844-dd25a22d888a)

---

### 📌 요약
- 인수의 개수가 유동적일 경우 가변인수를 사용하되, 필수 인수가 있을 경우 가변인수와 함께 선언하자.
- 성능 문제가 있을 경우 다중 정의와 함께 사용할 수 있다.
