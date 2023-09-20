# 다 쓴 객체 참조를 해제하라

> 작성자: 워니

## 목차
### 1. 메모리 누수 문제가 있는 스택 클래스
### 2. 메모리 누수를 일으키는 주범
### 참고). 레퍼런스 종류

---

### 1. 메모리 누수 문제가 있는 스택 클래스

자바는 가비지 컬렉터를 지원하는 언어로, 다 쓴 객체를 가비지 컬렉터가 알아서 회수해준다.
그렇다고 개발자가 메모리 관리에 아예 신경을 쓰지 않아도 된다는 것은 아니다.

**ex). 스택을 구현한 코드**

```java
class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object object) {
        ensureCapacity();
        elements[size++] = object;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

- 위의 코드는 메모리 누수 문제가 있다.
- pop 메서드를 통해 elements 배열에서 꺼내진 객체는 가비지 컬렉터가 회수하지 않는다. 스택이 다 쓴 참조(obsolete reference)를 가지고 있기 때문이다.
<br>*다 쓴 참조: 더 이상 쓰지 않을 참조*

<img src="https://github.com/Poin-Book/2023.09-Effective_Java/assets/116738827/d8c1e506-f4a1-42a7-bb22-7737890e82fb" width="700" height="300"/>

---

### 2. 메모리 누수를 일으키는 주범
메모리 관리를 GC(Garbage Collector)가 마법처럼 다 해줄 것 같지만, 메모리 누수는 생각보다 종종 발생한다.
그 원인을 크게 3가지로 나누면 다음과 같다.

- 1️⃣ 직접 메모리를 관리하는 경우
- 2️⃣ 캐시 (Cache)
- 3️⃣ 리스너/콜백


> **1️⃣ 직접 메모리를 관리하는 경우**
>
- 1의 Stack 예시처럼 직접 메모리를 관리하는 경우에는, 해당 참조를 다 썼을 때 명시적으로 null 처리(참조 해제)를 해주면 된다.
```java
class Stack {
    // ...
    public Object pop() {
      if (size == 0)
        throw new EmptyStackException();
        
      Object result = elements[--size];
      elements[size] = null; // 다 쓴 참조 해제
      return result;
    }
    // ...
}
```
- 스택 클래스에서 각 원소의 참조가 더 이상 필요하지 않는 시점은 스택에서 원소가 꺼내지는 시점이다.
- null 처리를 함으로써 가비지 컬렉터에 더 이상 쓰지 않을 것임을 알려줘야 한다. 
가비지 컬렉터 입장에서는 활성/비활성 객체 둘 다 Stack에서 사용중인 것으로 판단하기 때문이다.
- 이렇게 다 쓴 참조를 null 처리 해주면, NullPointerException에도 대응할 수 있다.


❓   **그렇다면 개발자는 자바에서 사용한 모든 객체에 null 처리를 해줘야 할까?**
<br>
매번 null 처리를 반드시 할 필요는 없다. 객체 참조에 null 처리를 하는 경우는 예외적이며, 코드를 복잡하게 만들 뿐이다.
다 쓴 객체 참조를 해제하는 가장 좋은 방법은 참조를 유효 범위(scope) 밖으로 밀어내는 것이다.

앞에서 살펴봤던 Stack 클래스는 배열을 통해 원소를 관리하고, 배열은 size라는 인덱스 값을 통해 원소에 접근한다. 원소를 사용하지 않더라도 가비지 컬렉터는 이를 알 수 없다. 그렇기 때문에 비활성 영역에서 참조하는 객체는 가비지 컬렉터가 해당 객체를 회수하지 못한다. 이처럼 메모리를 직접 관리하는 클래스인 경우에 null 처리가 적합하다.


> **2️⃣ 캐시 (Cache)**
>
객체 참조를 캐시에 넣고 나서, 객체 사용 후 객체 참조 해제하는 것을 잊어버리고 둔다면 메모리 누수가 발생할 수 있다.
이와 같은 상황에서 사용할 수 있는 방법은 다음과 같다.

1. 캐시 외부에서 Key를 참조하는 동안에만 Value가 필요한 경우, WeakHashMap을 사용한다.
   
💡 WeakHashMap이란?

- 더 이상 사용하지 않는 객체를 가비지 컬렉션 프로세스를 돌 때 자동으로 삭제해주는 Map으로 WeakReference를 사용하여 구현된 Map이다.
- WeakReference 타입을 키 값으로 사용한다.
- Key를 통해 캐시에 접근하여 필요한 로직을 모두 수행한 뒤, Key를 null로 초기화 하면 GC가 돌 때 자동으로 Map에서 삭제가 된다.

``` java
package reference;

import java.util.Map;
import java.util.WeakHashMap;

public class TestMain {

    public static void main(String[] args) {
        Map<Key, String> map = new WeakHashMap<>();

        Key key = new Key("key");

        // Map에 새로운 엔트리 추가
        map.put(key, "value");

        // Key[key] = value
        mapPrint(map);

        // Key 객체 참조 null 처리
        key = null;

        // 강제 GC
        System.gc();

        // 빈 값 출력
        mapPrint(map);
    }

    private static void mapPrint(Map<?, ?> map) {
        map.entrySet().stream().forEach(System.out::println);
    }
}

class Key {

    private String name;

    public Key(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Key[" + name + "]";
    }
}
```

단, 이 방식을 사용한다면 반드시 Map의 Key는 커스텀한 CacheKey 클래스를 만들어 사용해야 한다.
JVM 내부에서 일부의 값들을 캐싱을 하고 있기 때문에 Interger, Long, String 등과 같은 기본 Reference Type 클래스를 Key로 사용하게 되면 null로 초기화 하더라도 Map에서 삭제되지 않는다.

2. 특정 시간이 지나면 캐시 값이 의미가 없어지는 경우에는, 쓰지 않는 캐시 내 엔트리를 청소하는 방법이 있다. 
백그라운드 스레드를 활용해 오래된 캐시를 주기적으로 청소해 줄 수도 있고, 새로운 엔트리를 추가할 때 부수 작업을 수행할 수도 있다.

**ex). ScheduledThreadPoolExecutor 사용**
```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class CacheCleaner {
    public static void main(String[] args) {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

        // 캐시 청소 작업 정의
        Runnable cacheCleaningTask = () -> {
            // 캐시 청소 로직 작성
            System.out.println("캐시 청소 작업 실행");
        };

        // 캐시 청소 작업을 1시간마다 실행
        scheduler.scheduleAtFixedRate(cacheCleaningTask, 0, 1, TimeUnit.HOURS);
    }
}
```

**ex). LinkedHashMap의 removeEldestEntry 메서드**
```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```


> **3️⃣ 리스너/콜백**
>
리스너나 콜백이 객체를 강한 참조로 유지하여 객체가 더 이상 필요하지 않더라도 가비지 컬렉터가 회수하지 않아 메모리 누수가 발생할 수 있다. 또는, 비동기 작업에서 콜백을 사용할 때 콜백이 완료된 후에도 해당 콜백을 참조하고 있는 객체나 스레드가 있다면 해당 객체가 계속 유지되어 메모리 누수가 발생할 수도 있다.

따라서 리스너나 콜백을 WeakHashMap의 키로 저장하는 등, 약한 참조로 유지하면 가비지 컬렉터가 자동으로 수거할 수 있다. 또는 리스너나 콜백을 등록할 때 적절한 해제 메커니즘을 구현하여 객체가 더 이상 필요하지 않을 때 명시적으로 등록을 해제하여 메모리 누수를 방지할 수 있다.

---

### 참고). 레퍼런스 종류
1️⃣ **Strong Reference**
- 우리가 흔히 사용하는 '=' 을 사용하여 할당하는 경우
  ex). String str = new String(“abc”);
- GC의 대상이 되지 않으므로 해당 객체가 GC가 되기 위해서는 null로 초기화 하여 UnReachable 상태로 만들어 줘야 한다.

2️⃣ **Soft Reference**
- 현재 대상 객체의 참조가 SoftReference만 있는 경우
- SoftReference ref = new SoftReference<>(new String(“abc”));와 같은 형태로 사용
- 메모리에 여유가 있다면 GC 대상이 아니지만, out of memory 에러 직전까지 가면 GC 대상이 된다.

```java
public class SoftReferenceExample {  
  
    public static void main(String[] args) throws InterruptedException {  
        Object strong = new Object();  
        SoftReference<Object> soft = new SoftReference<>(strong);  
        strong = null;  
  
        System.gc();  
        Thread.sleep(3000L);  
  
        // 없어지지 않음 (메모리가 충분해서 굳이 제거할 필요가 없으므로)
        System.out.println(soft.get());  
    }  
}
```

3️⃣ **Weak Reference**
- 현재 대상 객체의 참조가 WeakReference만 있는 경우 해당 객체는 GC 대상이 된다.
- WeakReference ref = new WeakReference(new String(“abc”)); 와 같은 형태로 사용

```java
import java.util.Map;
import java.util.WeakHashMap;

class Main  {
    public static void main(String[] args) {
        WeakHashMap<Integer, String> map = new WeakHashMap<>();
        Integer key1 = 1000;
        Integer key2 = 2000;
        map.put(key1, "test a");
        map.put(key2, "test b");
        key1 = null;
        System.gc();  //강제 Garbage Collection
        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
}
```
```
2000=test b
```

4️⃣ **Phantom Reference**
- 바로 지우지 않고, PhantomReference를 ReferenceQueue에 넣고 나중에 정리할 수 있게 한다.
- 자원 정리를 할 때나, 객체가 메모리에서 언제 해제 되는지 알아야 할 때 사용된다.
```java
public class PhantomReferenceExample {  
  
    public static void main(String[] args) throws InterruptedException {  
        BigObject strong = new BigObject();  
        ReferenceQueue<BigObject> rq = new ReferenceQueue<>();  
  
        BigObjectReference<BigObject> phantom = new BigObjectReference<>(strong, rq);  
        strong = null;  
  
        System.gc();  
        Thread.sleep(3000L);  
  
        // 없어지지 않고 큐에 추가
        System.out.println(phantom.isEnqueued());  // true 출력 
  
        Reference<? extends BigObject> reference = rq.poll();  
        BigObjectReference bigObjectCleaner = (BigObjectReference) reference;  
        bigObjectCleaner.cleanUp();  
        reference.clear();  
    }  
}
```

---

✏️ **핵심정리**

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 존재한다. 이런 누수는 철저한 코드리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.
