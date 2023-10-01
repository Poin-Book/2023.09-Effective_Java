# 제목

> 작성자: 다나

## 목차
## 인터페이스에 메서드 추가

- 자바 8 전에는 기존 구현체를 깨뜨리지 않고 인터페이스에 메서드를 추가할 방법은 존재하지 않았다. 자바 8부터 **default 메서드**를 통해서 기존 인터페이스에 메서드를 추가할 수 있게 되었다.
- 디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 **default** 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다.
- 단, 이렇게 **default** 메서드를 추가한다고해도 기존 구현체들과 매끄럽게 연동된다는 보장은 없다.

## **생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어렵다.**

> `Çollection`의 `removeIf` 디폴트 메서드
> 

```java
default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean result = false;
        for (Iterator<E> it = iterator(); it.hasNext();){
            if(filter.test(it.next())){
                it.remove();
                result = true;
            }
        }
        return result;
}
```

- 이 코드는 범용적으로 작성되었지만 현존하는 모든 `Collection` 구현체와 잘 어우러지는것은 아니다. 대표적인 예가 `SyncronizedCollection`이다.

```java
static class SynchronizedCollection<E> implements Collection<E>, Serializable {

    ...
    public int size() {
        synchronized (mutex) {return c.size();}
    }
    ...
}

...

default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```

- `SyncronizedCollection`은 '한번에 한 쓰레드만 해당 오퍼레이션'을 실행해야한다. 이걸 보장하기 위해 **synchronized** 키워드를 사용해서 동기화를 보장한다.
- `removeIf`은 내부적으로 어떠한 동기화 처리도 되어 있지 않은 것을 볼 수 있다. 그렇지만 **default** 메서드이기 때문에 `SyncronizedCollection` 인스턴스에서 호출해서 사용할 수 있는 메서드가 된다.
- 만약 누군가가 이 사실을 모르고 `SyncronizedCollection`을 호출한다면`ConcurrentModificationException`이 발생할 수 있다.

## 인터페이스 설계 시에 default 메서드를 주의하자.

- **default** 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬수 있다. 때문에 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는일은 피해야한다.  
    ex)  
    ✅ **자바의 메서드 접근 규칙은 클래스가 메서드를 가지고 있다면 먼저 접근한다.**
    ![image](https://github.com/Poin-Book/2023.09-Effective_Java/assets/85955988/9c49093b-b852-4cac-9a20-fa11fdf52c99)

    ```java
    Exception in thread "main" java.lang.IllegalAccessError: class 
    org.example.item21.SubClass tried to access private method 
    org.example.item21.SuperClass.hello()V 
    (org.example.item21.SubClass and org.example.item21.SuperClass are in unnamed module of loader 'app')
    at org.example.item21.SubClass.main(SubClass.java:6)
    ```
    
- 반면, 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는데 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다.

## 인터페이스 릴리즈 전에 테스트를 거치자.

- 새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야한다. 서로 다른 방식으로 최소 세가지는 구현해보는것을 추천한다. 또한 다양한 클라이언트도 만들어 보는것이 좋다.
- 이런 테스트 과정을 통해 결함을 찾아내야 한다. **인터페이스를 릴리스 한 후라도 결함을 수정하는게 가능할 경우도 있겠지만, 그 가능성에 기대서는 안된다.**
