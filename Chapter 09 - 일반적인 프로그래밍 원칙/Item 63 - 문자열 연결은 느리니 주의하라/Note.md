# 문자열 연결은 느리니 주의하라

> 작성자: 밀리

## 목차
1. [문자열 예시](#_문자열_예시)
2. [결론](#결론)


#### 📌 문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이지만, 성능 저하가 상당하다. 문자열 연결 연산자로 문자열 n개를 잇는 작업은 n^2에 비례한다.

### 문자열 예시
#### 문자열을 잘못 사용한 예 - 느리다!
```java
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
        result += lineForItem(i); //문자열 연결
    }
    return result;
}

```  

각 item의 원소 개수만큼 문자열을 잇는다. 품목의 개수가 많아지면 많아질수록 성능 저하가 심해진다.

🤔 성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자!

#### StringBuilder 예시. 이를 사용하면 문자열 연결 성능이 크게 개선된다.
```java
public String statement2() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        b.append(lineForItem(i));
    }
    return b.toString();
}
```
statement() 메서드 수행 시간은 Item 수의 제곱에 비례해 늘어나고, statement2() 메서드의 수행 시간은 선형으로 늘어나므로 Item이 많아질수록 그 성능 차이도 심해질 것이다.

### 결론
- 성능에 신경 써야한다면 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자.  
- 대신에 StringBuilder의 append 메서드를 사용하거나 문자열을 연결하지 않고 하나씩 처리하자.
