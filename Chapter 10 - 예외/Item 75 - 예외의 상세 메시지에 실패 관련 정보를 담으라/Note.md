# Item 75 - 예외의 상세 메시지에 실패 관련 정보를 담으라

> 작성자: 피터
> 

실패 순간을 포착하려면 발생한 예외에 관여한 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다. 

## Stack Trace

프로그램이 실패하면 예외의 stack trace 정보를 출력한다. stack trace는 예외 객체의 `toString()`메서드를 호출해서 얻는 문자열로 반환한다. 따라서 `toString()`메서드에 실패 원인에 대한 정보를 가능한한 많이 담아 반환해야 한다. 

## 예제

```java
/**
 * IndexOutOfBoundsException을 생성한다.
 *
 * @param lowerBound 인덱스의 최솟값
 * @param upperBound 인덱스의 최댓값 + 1
 * @param index 인덱스의 실젯값
 */
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
   // 실패를 포착하는 상세 메시지를 생성한다.
   super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
   
   // 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
   this.lowerBound = lowerBound;
   this.upperBoudn = upperBound;
   this.index = index;
}
```

- 다음과 같이 범위의 최소, 최대, 범위를 벗어난 인덱스등 실패 원인과 관련된 모든 정보를 담아라
- 필요 정보를 예외 생성자로 받아 생성해놓는것도 하나의 방법이다

## 접근자 메서드

- 예외는 실패와 관련한 정보를 얻을 수 있는 접근자 메서드를 적절히 제공하는 것이 좋다.
- 포착한 실패 정보는 예외 상황을 복구하는 데 유용할 수 있으므로 접근자 메서드는 비검사 예외보다 검사 예외가 더 적합하다
