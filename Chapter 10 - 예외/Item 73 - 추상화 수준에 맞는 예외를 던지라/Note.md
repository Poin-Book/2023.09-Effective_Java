# Item 73 - 추상화 수준에 맞는 예외를 던지라

> 작성자: 워니

## 목차
**저수준 예외의 발생  
예외 번역(exception translation)  
예외 연쇄(exception chaining)  
예외 번역도 남용해선 안 된다.  
📌 요약**  

---

### 저수준 예외의 발생
현재 로직과 전혀 상관없고 유추할 수 없는 예외가 나올 경우 당황스러울 것이다.  
이는 메서드를 호출했을 때 메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버릴 때 종종 일어나는 상황이다.  
이러한 방식은 내부 구현 방식을 드러내 윗 레벨 API를 오염시킬 수 있다. 또 구현 방식이 바뀌게 되면 다른 예외가 전파되어 프로그램을 깨지게 할 수도 있다.  

> 👉 **위와 같은 문제를 피하기 위해서 예외 번역 (exception translation)을 사용할 수 있다.**

---

### 예외 번역(exception translation)
저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔서 던지는 것

##### 예외 번역의 예시
``` java
try {
	... // 저수준 추상화 이용
} catch(LowLevelException e) {
	// 추상화 수준에 맞게 번역
	throw new HigherLevelException(...);
}
```

##### AbstractSequentialList 예제
``` java
public E get(int index) {
    ListIterator<E> i  = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException();
    }
}
```

##### SQLException
- `SQLException`은 대표적인 체크 예외로, 대부분 복구가 불가능하다.  
  즉, 예외를 잡아도 거의 처리해 줄 수 없다.
- 따라서 아래의 예시와 같이 처리해줄 수 있는 것은 처리하고, 
  처리할 수 없는 것은 런타임 에러로 포장하여 호출하는 쪽에서 무차별 `throw`를 선언하지 않도록 해주는 것이 좋다.

``` java
try {
    ...
} catch (SQLException e) {
    if(e.getErrorCode()) {
        // 처리가 가능한 경우
    } else {
        // 처리가 불가능한 경우, 에러를 전송
        throw new RuntimeException(e);
    }
}
```

</br>

> 💡 **하위 계층에서 발생한 예외 정보가 상위 계층 예외를 발생시킨 문제를 디버깅하는데 유용할 수도 있다.**  
**그럴 때는 예외 연쇄(exception chaining)를 사용하면 좋다.**

---

### 예외 연쇄(exception chaining)
문제의 원인인 저수준 예외를 고수준 예외에 실어 보내는 것

##### 상위 계층에서는 필요하면 저수준 예외를 꺼내어 확인할 수 있다.  

``` java
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}

class ChildClass {
    public void publicAPIMethod() {
	try {
	    ... // 저수준 추상화 이용
	} catch (LowLevelException e) {
	    // 저수준 예외(e)를 고수준 예외에 실어 보낸다. 
	    throw new HigherLevelException(e);
        }
    }
}
```
- 상위 계층 예외에서 접근자 메서드(`Throwable.getCause`)를 통해 저수준 예외를 꺼내볼 수 있다.  
- 즉, 이렇게 예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며, 원인과 고수준 예외의 스택 추적 정보를 통합해준다.

---

### 예외 번역도 남용해선 안 된다. 
예외 번역으로 저수준의 예외를 무작정 바깥으로 전파해서 생기는 문제는 해결할 수 있지만, 그렇다고 이를 남용해서는 안 된다.  
전파 혹은 번역을 하기 전에 적절한 수준에서 메서드가 성공하도록 하여 하위 계층에서 예외가 발생하지 않도록 만드는 것이 최선이다. 

이를 위해서, 다음과 같은 방법을 사용할 수 있다.

- 상위 계층에서 인수를 전달하기 전에 미리 검사한다.  
  ex). `validator` 이용
- `java.util.logging`의 로깅 기능을 이용해 기록해두어 클라이언트에게 전파하지 않고 프로그래머가 로그 분석 및 조치를 취할 수 있게 한다.

---

### 📌 요약
- 어쨌든 제일 좋은 방법은 당연한 말이지만 에러 없는 코딩을 하는 것.
- 하위 계층에서 발생한 에러는 하위 계층에서 처리하고, 전달해야 하는 경우에는 적절한 예외로 변환해서 보내자.
- 이때, 예외 연쇄를 이용하면 상위 계층 맥락의 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다.
