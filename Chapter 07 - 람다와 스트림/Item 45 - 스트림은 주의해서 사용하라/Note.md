# 스트림은 주의해서 사용하라

> 작성자: @destiny3912

# 목차
- [스트림은 주의해서 사용하라](#스트림은-주의해서-사용하라)
- [목차](#목차)
- [본문](#본문)
  - [Item 45 스트림은 주의해서 사용하라](#item-45-스트림은-주의해서-사용하라)
    - [1. 스트림은 무엇을 하는가](#1-스트림은-무엇을-하는가)
    - [2. 스트림의 가독성](#2-스트림의-가독성)
    - [3. 매핑시 이전 값이 필요할때](#3-매핑시-이전-값이-필요할때)
    - [4. 스트림 사용시 주의사항](#4-스트림-사용시-주의사항)
    - [5. 요약](#5-요약)
# 본문

## Item 45 스트림은 주의해서 사용하라

### 1. 스트림은 무엇을 하는가

> 다량의 데이터 처리 작업
> 
1. 핵심 개념
    - 스트림 (Stream)
        - 데이터 원소의 유한 혹은 무한 시퀀스(sequence)를 의미함
    - 스트림 파이프라인 (Stream pipeline)
        - 스트림의 원소들로 수행하는 연산 단계를 의미함
        - 과정
            1. 소스 스트림에서 시작
            2. 하나 이상의 중간 연산
                - 스트림을 어떠한 방식으로 변환(transform)한다.
                - 변환
                    - 각 원소에 함수를 적용
                    - 특정 조건을 만족 못하는 원소를 필터링
            3. 종단 연산 → 이게 없다면 해당 스트림의 연산은 없는 연산으로 취급한다.
                - 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 추가한다.
                    - 원소를 정렬해 컬랙션에 담는다.
                    - 특정 원소 하나를 선택한다.
                    - 모든 원소를 출력한다. → System.out.println()을 직접사용해도 된다.
                - 또한 스트림 파이프 라인은 지연 평가(lazy evaluation) 된다.
                    - 종단 연산이 호출될때 평가를 진행 (계산을 한다)
                    - 종단 연산에 쓰이지 않는 원소는 계산할때 없는 것으로 취급하고 연산한다
                    - 따라서 무한 스트림을 다룰 수 있게 해준다
        - 기본적으로 위의 과정이 순차적으로 수행 된다
        - 다만 병렬로 사용하고 싶다면 파이프라인을 구성하는 스트림 중 parallel 메서드를 사용하자
            - 그러나 효과를 볼 수 있는 상황은 제한적이다 → Item 48에서 자세히
        - 또한 파이프라인 안에서 a라는 값을 b라는 값으로 매핑했을때 a라는 값을 기억하지 않는다.
            - 따라서 앞단계의 값이 필요할때 사용하기는 좀 까다롭다. → 아래에서 설명
    - 스트림 원소의 출처
        - 컬랙션, 배열, 파일, 정규표현식 패턴 매처(matcher), 난수 생성기, 다른 스트림
        - 객체 참조이거나 기본 타입값이다
            - 기본 타입 → int, long, double의 3가지를 지원 (오직 3가지이다.)

### 2. 스트림의 가독성

> 여러분도 알겠지만 스트림으로만 구성된 코드는 매우 읽기 싫게 생겼다. 예시를 한번 보자.
> 
- 예시는 다음 프로그램이다.
    - 사전 파일에서 단어를 읽어 사용자가 지정한 값보다 원소 수가 많은 아나그램 그룹을 출력
        - 아나그램 → 철자를 구성하는 알파벳이 같고 순서만 다른 단어
    - 이 프로그램은 사용자가 명시한 사전 파일에서 각 단어를 읽어 맵에 저장
        - 이 맵의 키는 그 단어를 구성하는 철자들을 알파벳순으로 정렬한 값이다.
        - 따라서 아나그램끼리는 같은 키를 공유함
        - 즉, 맵의 값은 같은 키를 공유한 단어를 담은 집합이다.
    1. 스트림을 쓰지 않을때
        
        ```java
        public class IterativeAnagrams {
            public static void main(String[] args) throws IOException {
                File dictionary = new File(args[0]);
                int minGroupSize = Integer.parseInt(args[1]);
        
                Map<String, Set<String>> groups = new HashMap<>();
                try (Scanner s = new Scanner(dictionary)) {
                    while (s.hasNext()) {
                        String word = s.next();
        /* 
        	자바 8에서 추가된 computeIfAbsent 메서드를 사용
        	맵 안에 키가 있는지 찾은 다음 있으면 키에 매핑된 값을 반환 한다.
        	키가 없다면 건네진 함수(alphabetize)를 키에 적용해서 연산하고 그 결과를 반환 한다.
        */
                        groups.computeIfAbsent(alphabetize(word),
                                (unused) -> new TreeSet<>()).add(word);
                    }
                }
        				
                for (Set<String> group : groups.values())
                    if (group.size() >= minGroupSize)
                        System.out.println(group.size() + ": " + group);
            }
        
            private static String alphabetize(String s) {
                char[] a = s.toCharArray();
                Arrays.sort(a);
                return new String(a);
            }
        }
        ```
        
    2. 스트림을 쓸때
        
        ```java
        public class StreamAnagrams {
            public static void main(String[] args) throws IOException {
                Path dictionary = Paths.get(args[0]);
                int minGroupSize = Integer.parseInt(args[1]);
        				
        /*
        	이 주석 아래의 코드는 1번 예시 주석 아래의 코드와 동일한 코드이다.
        	그냥 보기만해도 읽기가 싫어진다
        */
                try (Stream<String> words = Files.lines(dictionary)) {
                    words.collect(
                            groupingBy(word -> word.chars().sorted()
                                    .collect(StringBuilder::new,
                                            (sb, c) -> sb.append((char) c),
                                            StringBuilder::append).toString()))
                            .values().stream()
                            .filter(group -> group.size() >= minGroupSize)
                            .map(group -> group.size() + ": " + group)
                            .forEach(System.out::println);
                }
            }
        }
        ```
        
    3. 스트림을 적절히 사용할때
        
        ```java
        public class HybridAnagrams {
            public static void main(String[] args) throws IOException {
                Path dictionary = Paths.get(args[0]);
                int minGroupSize = Integer.parseInt(args[1]);
        /*
        	이 주석 아래의 코드는 2번 예시 주석 아래의 코드와 동일한 코드이다.
        	groupingBy 메서드 안의 람다 함수를 도우미 함수로 빼내어 구현하였음
        	그냥 영어 해석하듯이 해석해보면
        	파일을 읽어서 각 단어에 alphabetize를 적용하여 매핑하고
        	그 맵에 스트림을 열어서
        	필터를 거친 후 나온 각 원소를 출력한다.
        	정도가 되겠다.
        	이렇게 적절히 사용하면 가독성이 나쁘지만은 않다.
        */
                try (Stream<String> words = Files.lines(dictionary)) {
                    words.collect(groupingBy(word -> alphabetize(word)))
                            .values().stream()
                            .filter(group -> group.size() >= minGroupSize)
                            .forEach(g -> System.out.println(g.size() + ": " + g));
                }
            }
        
            private static String alphabetize(String s) {
                char[] a = s.toCharArray();
                Arrays.sort(a);
                return new String(a);
            }
        }
        ```
        

### 3. 매핑시 이전 값이 필요할때

- 예시로 메르센 소수라는 숫자를 찾는 스트림이 있다

```java
public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
								.forEach(System.out::println);
    }
```

- 메르센 소수는 2^p - 1 형태의 수 중에서 p가 소수일때 소수인 경우를 말한다.
- 위 코드는 그 메르센 소수를 20개만 출력하는 코드인데
- 여기서 p 값은 중간에 소실된다.
- 하지만 p 값을 끄집어내고 싶다면
- 다음과 같이 종단 연산을 바꾸면 된다.

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

- 단순히 생각하자면 중간연산에서 수행한 매핑을 반대로 수행한다고 생각하면 된다.

### 4. 스트림 사용시 주의사항

1. 람다 함수의 매개변수 이름
    - 람다에서는 타입 이름을 생략하는 경우가 많음
    - 따라서 매개변수의 이름을 잘 지어야 함
        - 이름은 매개변수가 어떠한 값을 가지고 있는지 이름만 보고 알 수 있어야 한다.
    - 또한 우리가 구현해야하는 핵심 로직은 도우미 매서드로 빼낸다.
        - 스트림 파이프라인에서는 매우 중요함
2. 기본 타입 char에 대한 스트림은 존재하지 않는다.
    - 예를 들어서 “Hello World”.chars()가 반환하는 스트림의 원소는 int 값이다.
    - 따라서 그대로 프린트를 하게 된다면 아스키 코드값이 찍힐것이다.
    - 올바르게 프린트 하려면 명시적으로 형변환이 필요하다.
3. 스트림을 남용하지 말자.
    
    > 스트림을 처음 본다면 신세계를 접한듯 해서 모든 것을 스트림으로 바꾸고 싶을 것이다 (저도 그랬어요 ㅎㅎ)
    > 
    - 다만 앞서 설명한 예시에서 보았듯이 스트림으로만 구성된 코드는 가독성이 매우 좋지 않다.
    - 따라서 다음과 같이 해야한다
        
        > 기존 코드는 스트림을 사용하도록 리팩터링한다면 스트림을 사용하는 코드가 더 나아보일때 사용하자
        > 
        - 여기서 더 나아보인다의 기준은 유지보수성도 포함한 내용이다.
    - 대략적인 기준이 책에 나와있어서 요약해보았다
        1. 반복문을 써야할때 
            - 해당 반복문의 코드 블럭내에서 지역 변수를 읽고 수정할때
                - 람다는 final이거나 사실상 final인 변수만 읽을 수 있음
                - 또한 지역변수 수정도 불가능 함
            - 코드 블럭 내에서 return, break, continue를 사용해야 할때
                - 람다는 모두 불가능 하다.
        2. 스트림을 써야할때
            - 원소들의 시퀀스를 일관되게 변환할때
            - 원소들을 필터링 할때
            - 원소들의 하나의 연산을 사용해 결합할때 (더하기, 빼기, 최대/최소 구하기 등등)
            - 원소들을 하나의 컬랙션에 모을때 (비슷한 속성을 기준으로)
            - 특정 조건을 만족하는 원소를 찾을때

### 5. 요약

> 스트림을 쓸때는 좋다고 막 쓰지 말고 가독성과 유지보수성을 생각해서 사용하라
>