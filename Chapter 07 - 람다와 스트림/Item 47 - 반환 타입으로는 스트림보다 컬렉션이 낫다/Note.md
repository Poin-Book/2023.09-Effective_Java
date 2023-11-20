# 반환 타입으로는 스트림보다 컬렉션이 낫다

> 작성자: 워니

## 목차
**1. 반복을 지원하지 않는 Stream**  
**2. Stream과 Iterable 간의 호환성을 고려하자**  
**3. 공개 API에서의 Collection 반환**  
**4. 전용 컬렉션 구현을 고려하라**  
**5. 요약**

---

> ### Java 8 등장

- **Java 8 이전**까지는 Stream이 없었기 때문에 원소 시퀀스(일련의 원소)를 반환하는 메서드는 보통 컬렉션 인터페이스나 Iterable, 배열을 사용하곤 했는데,  
- **Java 8 이후** Stream이 등장하며 고려해야 할 부분들이 생겨났다. 

---

### 반복을 지원하지 않는 Stream
스트림은 반복(iteration)을 지원하지 않는다.  

##### for-each를 활용할 수 없는 Stream
![image](https://github.com/kiwijomn/test-repo/assets/116738827/ad1eeb14-925e-453c-b19a-7bfa7622e842)

##### ➡️ 메서드 참조를 매개변수화된 Iterable로 적절히 형변환 해주기
![image](https://github.com/kiwijomn/test-repo/assets/116738827/b3bd709e-2eed-4f99-b837-47e69ab45fd9)

##### BaseStream interface
``` java
public interface BaseStream<T, S extends BaseStream<T, S>>
        extends AutoCloseable {
    /**
     * Returns an iterator for the elements of this stream.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">terminal
     * operation</a>.
     *
     * @return the element iterator for this stream
     */
    Iterator<T> iterator();

		...
```

Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고 Iterable 인터페이스가 정의한 방식대로 동작한다.  
하지만 Stream과 Stream의 상위 객체인 BaseStream 모두 Iterable을 extends 하지 않으므로 for-each로 stream을 반복할 수 없다.
</br>

---

### Stream과 Iterable 간의 호환성을 고려하자
Stream을 반복하기 위해서 제공되는 우회로는 적절한 것이 없다.  

##### 타입을 중개해주는 어댑터 메서드
``` java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
	return stream::iterator;
}
```
``` java
public class Test {
  public static void main(String[] args) {
    Stream<String> fruits = Stream.of("사과", "바나나", "귤");

    for (String fruit : iterableOf(fruits)) { // 어댑터 메소드 사용
      System.out.println(fruit);
    }
  }
}
```

이렇게 어댑터를 사용해서 반복할 수 있지만, 이왕이면 이런 어댑터를 구현하지 않는 것이 비용적으로 더 절약될 것이다. 

---

### 공개 API에서의 Collection 반환
- 해당 메서드가 오직 stream pipe line에서만 쓰이는 걸 안다면, stream을 반환해주고
- 반환된 객체들이 반복문에서만 쓰일 걸 알면, Iterable을 반환하면 된다.

하지만 공개 API의 경우, stream pipe line을 사용하려는 사용자와 반복문을 사용하려는 사용자 둘 다 배려해야 한다.

##### Collection interface
``` java
public interface Collection<E> extends Iterable<E> {
	...
	default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
  }
}
```

➡️ Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공한다. 즉, 반복과 stream을 동시 지원한다.  
따라서 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.

---

### 전용 컬렉션 구현을 고려하라
데이터가 충분히 적다면 표준 컬렉션 구현체(ex: ArrayList, HashSet)를 사용해도 되지만, 
그게 아니라면 전용 컬렉션을 구현하는 것을 고려할 수 있다.  
_(단지 collection을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.)_

멱집합은 원소의 개수가 n개일 때 2^n개가 된다는 점을 알고 있기에 AbstractList를 이용해 효율성을 높인 전용 컬렉션을 반환할 수 있다. 

##### 입력 집합의 멱집합을 전용 컬렉션에 담아 반환
> - 멱집합: 한 집합의 모든 부분집합을 원소로 하는 집합으로, 원소의 개수가 n개일 때 2^n개가 된다.  
> - {a, b, c}의 멱집합 : {{a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}

``` java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
       List<E> src = new ArrayList<>(s);
       if(src.size() > 30) { // (1)
           throw new IllegalArgumentException("집합에 원소가 너무 많습니다.(최대 30개): " + s);
       }

       return new AbstractList<Set<E>>() {
           @Override
           public int size() {
               return 1 << src.size();
           }

           @Override
           public boolean contains(Object o) {
               return o instanceof Set && src.containsAll((Set) o);
           }

           @Override
           public Set<E> get(int index) {
               Set<E> result = new HashSet<>();
               for (int i = 0; index != 0; i++, index >>= 1) { // (2) 
                   if((index & 1) == 1) {
                       result.add(src.get(i));
                   }
               }
               return result;
           }
       };
    }
}
```

- (1) size() 메서드의 리턴 타입은 `int`이므로 최대 길이가 `2^31 - 1` 또는 `Integer.Max_Value`로 제한되기에 입력 집합의 원소 수가 30을 넘으면 예외를 던진다.
- (2) 원소의 인덱스를 비트 백터로 사용하기 때문에, 메모리에 거대한 컬렉션을 올리지 않고 효율적으로 사용이 가능하다.
- AbstractCollection을 활용하여 Collection 구현체를 리턴할 때는 Iterator용 메서드 외에 contains와 size 메서드만 더 구현하면 된다.
- 하지만 contains와 size를 구현할 수 없는 경우에는, Collection보다는 Stream, Iterable을 반환하는 것이 낫다.

---

📌 요약
- Stream API는 Iterable 인터페이스가 정의한 추상 메서드를 모두 포함하고 해당 인터페이스가 정의한 방식대로 동작하지만, Iterable을 extends 하지는 않았다. 
- 원소 시퀀스를 반환하는 API를 정의할 때 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자 모두를 위해서, 양쪽을 다 만족시키려 노력하자.
- Collection 인터페이스는 Iterable의 하위 타입이면서 stream 메서드도 제공하므로 반복과 스트림을 동시 지원한다.  
  그러니 원소 시퀀스를 반환하는 공개 API는 Collection(혹은 그 하위 객체)을 사용하는게 좋다. 
- 반환 전부터 이미 원소들을 컬렉션으로 관리하고 있거 원소의 수가 적다면, 표준 컬렉션(ex: ArrayList)에 담아 반환하자.
- 그렇지 않다면, 전용 컬렉션을 구현할지 고민하자. (ex: 멱집합)
- 컬렉션을 반환할 수 없다면 Stream과 Iterable 중 자연스러운 것을 반환하자. 
