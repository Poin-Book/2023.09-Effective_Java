# 스트림에서는 부작용 없는 함수를 사용하라

> 작성자: {피터}

# 스트림

스트림은 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이다.스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 API와, 이 패러다임을 받아들여야한다.스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다.

각 변환 단계는 이전단계의 결과를 받아 처리해야하는 **순수함수**여야 한다.이 순수 함수란 오직 입력만이 결과에 영향을 주고, 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않아야한다. 

이렇게 하려면 스트림 연산에 건네는 함수 객체는 모두 side effect가 없어여한다.

```java
46-1
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
	words.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
	});
}
```

위의 코드는 stream을 사용했지만 올바르게 사용했다고 할수 없다. 단순 반복적 코드로, 스트림 API의 이점을 살리지 못했다. 위 코드의 모든 작업이 forEach에서 일어나는데 이 때 외부 상태를 수정하는 람다가 실행되면서 문제가 생긴다. 앞서 말했듯 순수 함수여야 하는데 그 이상의 일을 한다.

이를 올바르게 작성하면 아래와 같다.

```java
46-2
Map<String, Long> frequency = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    frequency = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

같은 결과를 나타내지만 짧고 명확하다.

forEach 연산은 스트림 계산 결과가 필요할때만 사용하고 계산할 때는 사용하지 말아야한다.

# Collector

스트림을 사용하는데 수집기(Collector)를 사용할 수 있다. 수집기는 축소(reduction) 전략을 캡슐화한 블랙박스 객체라고 볼 수 있으며 때문에 스트림에 자주 사용된다. 
수집기는 총 세가지로, toList(), toMap(), toCollection(collectionFactory)가 있다. 

### **toList**

- 스트림을 컬렉션 List로 변환해주는 메서드이다.
- comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드다.

```java
List<String> topTen = freq.keySet().stream()
	.sorted(comparing(frequency::get).reversed())
	.limit(10)
	.collect(Collectors.toList());
```

### **toMap**

- 스트림을 컬렉션 Map으로 변환해주는 메서드이다.
- 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.
- 스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 IllegalStateException을 던지며 종료될 것이다.

```java
private static final Map<String, Operation> stringToEnum =
	Stream.of(values()).collect(
		toMap(Object::toString, e -> e));
```

# ****Collectors Method: groupingBy****

groupingBy 메서드는 Collectors의 또 다른 메서드로, 입력으로 분류함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환합니다. 그리고 이 카테고리가 해당 원소의 맵 키로 쓰인다.
****groupingBy는 다중정의된 메서드로 총 3가지 메서드가 있다.

### classifier만 사용하는 메서드

- groupingBy 메서드는 가장 간단한 것으로서 분류함수 classfier만 인수로 받고 Map을 반환한다.
- 반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 List이다.

```java
Map<String, List<String>> map = words.collect(groupingBy(word -> alphabetize(word)))
```

### classifier와 downstream을 사용하는 메서드

- groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림(downstream) 수집기도 명시해야 한다.
- 아래와 같이 다운스트림 수집기로 counting()을 건네는 방법도 있다. 이렇게 하면 각 카테고리(키)를 (원소를 담은 컬렉션이 아닌) 해당 카테고리에 속하는 원소의 개수(값)와 매핑한 맵을 얻는다.

```java
Map<String, Long> frequency = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    frequency = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

### classfier와 downstream, mapFactory를 모두 사용하는 메서드

- 이 메서드는 점층적 인수 목록 패턴에 어긋난다.
- 즉 mapFactory 매개변수가 downStream 매개변수보다 앞에 놓인다.
- 이버전의 groupingBy를 사용하면 맵과 그 안에 담긴 컬렉션 타입을 모두 지정 가능하다.
