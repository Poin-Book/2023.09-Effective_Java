# Item 76 - 가능한 한 실패 원자적으로 만들라

> 작성자: 캐슬

## 목차
1. 실패 원자적이란
2. 실패 원자적으로 만드는 방법들
3. 주의 사항

## 실패 원자적?

> 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다.
>

## 메서드를 실패 원자적으로 만드는 방법

1. 불변 객체로 설계하기
    - 불변 객체는 태생적으로 실패 원자적 - 상태가 생성 시점에 고정되어 절대 변하지 않기 때문


2-1. 가변 객체의 경우 작업 수행에 앞서 매개 변수의 유효성 검사하기

- 상태를 변경 하기 전에 잠재적 예외의 가능성 대부분을 걸러낼 수 있다.

코드7-2 Stack.pop

```java
public Object pop(){
	if(size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null;  // 다 쓴 참조 해제
	return result;
```

- if 문에서 size를 확인하여 0이면 예외를 던진다.

2-2. 실패할 가능성이 있는 모든 코드를 객체의 상태를 바꾸는 코드보다 앞에 배치하기

- 앞서의 방식에 덧붙여 쓸 수 있는 기능

ex) TreeMap

- 원소를 추가하려면 그 원소는 TreeMap의 기준에 따라 비교할 수 있는 타입이여야 한다.
- 엉뚱한 원소를 추가하려 들면 트리를 변경하기에 앞서 해당 원소가 들어갈 위치를 찾는 과정에서 ClassCastException을 던진다.

1. 객체의 임시 복사본에서 작업을 수행한 다음, 작업이 성공적으로 완료되면 원래 객체와 교체하기
    - 데이터를 임시자료 구조에 저장해 작업하는게 더 빠를때 적용하기 좋은 방식

   ex) 어떤 정렬 메서드에서는 정렬을 수행하기 전에 입력 리스트의 원소들을 배열로 옮겨 담는다.

    - 배열을 사용하면 정렬 알고리즘의 반복문에서 원소들에 훨씬 빠르게 접근할 수 있기 때문이다.
    - 성능을 높이고자 하는 결정이지만, 정렬에 실패하더라도 입력 리스트는 변하지 않는 효과를 덤으로 얻는다.

1. 작업 도중 실패를 가로채는 복구 코드를 작헝하여 작업 전 상태로 되돌리기
    - 주로 내구성을 보장해야 하는 자료구조에 사용 (자주 사용되지는 않음)

## 주의 사항

- 실패 원자성은 일반적으로 권장되는 덕목이지만 항상 달성할 수 있는 것은 아니다.

ex) 두 스레드가 동기화 없이 같은 객체를 동시에 수정한다면 그 객체의 일관성이 깨질 수 있다. → concurrentModificationException을 잡아냈다고 해서 그 객체가 여전히 쓸 수 있는 상태라고 가정해서는 안된다.

- Error는 복구할 수 없음으로 AssertionError에 대해서는 실패 원자적으로 만드려는 시도조차 할 필요 없다.
- 실패 원자적으로 만들 수 있다고 하더라도 항상 그리 해야하는 것은 아니다.

  → 달성하기 위해 비용이나 복잡도가 아주 큰 연산도 있기 때문이다.


> 메서드 명세에 기술한 예외라면 설혹 예외가 발생하더라도 객체의 상태는 메서드 호출 전과 똑같이 유지돼야 한다는 것이 기본 규칙이다. 이 규칙을 지키지 못한다면 실패 시 객체의 상태를 API 설명에 명세해야 한다.
>