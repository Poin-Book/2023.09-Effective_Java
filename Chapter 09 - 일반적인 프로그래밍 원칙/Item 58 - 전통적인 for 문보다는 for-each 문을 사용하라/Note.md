# 제목

> 작성자: {밀리}

## 목차
- [전통적인 for문을 사용해서 반복하기](#_전통적인_for문을_사용해서_반복하기)
- [for-each문](#_for-each문)
- [for문을 잘못 사용했을 때 생기는 버그 찾기](#_for문을_잘못_사용했을_때_생기는_버그_찾기)
- [for-each문을 사용할 수 없는 경우](#_for-each문을_사용할_수_없는_경우)
- [Iterable 인터페이스](#_Iterable_인터페이스)
- [결론](#결론)

### 전통적인 for문을 사용해서 반복하기
#### 컬렉션 순회
```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ){
	Element e = i.next();
    ...  //e로 무언가를 한다.
}
```

#### 배열 순회
```java
for (int i = 0; i < a.length; i++) {
    ..// a[i]로 무언가를 한다.
}
```

아이템 57에서 말했던 ```while``` 문 보다는 낫지만, 가장 좋은 방법은 아니다.  
1. 반복자와 인덱스 변수는 코드만 지저분하게 할 뿐 꼭 필요한 것은 원소들뿐이다.  
2. 반복자와 인덱스 '변수'를 잘못 사용할 가능성이 높아진다.  
3. 컬렉션이나 배열이냐에 따라 코드 형태가 달라진다.

➡️ 이러한 문제들은 ```for-each``` 문을 사용하면 모두 해결된다!  

### for-each문

> for-each 문의 정식 명칭은 향상된 for문(enhanced for statement) 이다.

```java
for (Element e: elements) {
  ... // e로 무언가를 한다.
}
```
- 여기서 콜론(:)은 "안의(in)"라고 읽으면 된다.
  - 따라서 이 반복문은 "elements 안의 각 원소 e에 대해"라고 읽는다.  
- 간단하게 반복을 구성할 수 있다.  
  - 그렇다고 속도도 느리지 않고, 최적화된 속도를 따른다.
- 컬렉션 중첩에서는 더욱 더 이점이 커진다.  
  - Iterator를 건드릴 필요도 없기 때문에 실수할 일도 적다.  


### for문을 잘못 사용했을 때 생기는 버그 찾기
#### 58-4 버그를 찾아보자
```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
            NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
    deck.add(new Card(i.next(), j.next())); // 문제 발생
```
- 위 코드에는 어떤 버그가 있을까?  
  - 먼저, 사용자가 ```i.next()``` 와 ```j.next()``` 를 이용하여 작성하고자 한 코드는 이것이 아니었을 것이다.  
  - 외부 루프에 이용된 ```enum Suit```이 ```enum Rank```보다 짧아서 ```NoSuchElementException```을 던지게 된다.


사실 위의 예는 운이 좋은 케이스고 운이 나쁘다면, enum 상수의 개수가 같아서 예외를 던지지 않고 종료할 것이다. 그렇다면 나중에 enum 의 개수가 변경된 후에 없던 에러가 생기는 현상을 겪을 것이다.  
#### 같은 버그, 다른 증상
```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }


Collection<Face> faces = EnumSet.allOf(Face.class);

for (Iterator<Face> i = faces.iterator(); i.hasNext();) {
  for (Iterator<Face> j = faces.iterator(); j.hasNext();) {
    System.out.println(i.next() + ", " + j.next());
  }
}
```
- 주사위를 두 번 굴렸을 때 나올 수 있는 모든 경우의 수를 출력하는 코드인데 가능한 조합을 단 여섯 쌍만 출력하고 끝내버린다.(36개 조합이 나와야 한다.)  
- 즉, 위 코드가 운이 없을 때이다.  
  - 예외가 뜨지 않아서 결과값을 눈으로 확인하기 전까지는 코드가 잘못되었는지 알기 힘들다.


#### 58-4 코드 문제 해결
```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
  Suit suit = i.next();
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
    deck.add(new Card(suit, j.next())); // 문제 해결
}
```
- 코드를 올바르게 동작하게 바꾸었지만, 아직은 코드가 더러워보인다.

```java
for (Suit suit : suits)
	for (Rank rank : ranks)
    	deck.add(new Card(suit, rank));
```
- ```for-each``` 를 통해 훨씬 깔끔한 방향으로 코드를 개선했다.  

```for-each``` 를 이용하면 순회 코드에서 생기는 실수를 미연에 방지할 수 있다.

### for-each문을 사용할 수 없는 경우
1. 파괴적인 필터링 (destructive filtering)  
- 반복을 돌며 원소를 하나씩 지워나가는 필터링을 의미한다.  
- ```Collection``` 의 ```removeIf()``` 를 통해 구현해나가자.  
  - ```Collection.removeIf()``` 메서드는 특정 조건을 만족하는 원소를 지울 수 있다.

2. 변형 (transforming)  
- 리스트 혹은 배열을 순회하며 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용하자.  

3. 병렬 반복 (parallel iteration)  
- 여러 컬렉션을 병렬로 순회해야 한다면, 반복자와 인덱스 변수를 사용해 엄격하게 제어하는 편이 좋다.

### Iterable 인터페이스
```java
public interface Iterable<E> {
  Iterator<E> iterator();
}
```
- ```for-each``` 를 사용하기 위해서는 ```Iterable``` 을 구현해야 한다.  
- ```Iterable``` 만 간단히 구현한다면, ```for-each``` 를 사용할 수 있다.

### 결론
💡 전통적인 for문과 비교했을 때 for-each문은 명료하고, 유연하고, 버그를 예방해준다. 성능 저하도 없으므로 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.






