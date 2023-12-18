# 스레드 안정성 수준을 문서화하라

> 작성자: 밀리

## 목차
- [API 문서 synchronized 한정자](#_API_문서_synchronized_한정자)
- [스레드 안정성 수준(높은 수준부터 나열!)](#스레드_안정성_수준(높은_수준부터_나열!))
- [동기화에 대한 문서화](#동기화에_대한_문서화)
- [외부에서 사용할 수 있는 Lock](#외부에서_사용할_수_있는_Lock)
- [결론](#결론)  
### API 문서 synchronized 한정자
- 메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다.  

    - API 문서에 synchronized 한정자가 보인다고해서 이 메서드가 스레드 안전하다고 믿기 어렵다.  

- 멀티 스레드 환경에서 안전하게 사용하게 하려면 지원하는 스레드 안전성 수준을 정확히 명시해야한다.

### 스레드 안정성 수준(높은 수준부터 나열!)
> 불변(immutable)
- 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화가 필요없다.  
- String , Long, BigInteger  
- ```@Immutable```  
 
> 무조건적 스레드 안전(unconditionally thread-safe)
- 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다.
- AtomicLong ,ConcurrentHashMap
- ```@ThreadSafe```
 
> 조건부 스레드 안전(conditionally thread-safe)
- 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다.  
- Collections.synchronized 래퍼 메서드가 반환한 컬렉션들  
- ```@ThreadSafe```
 
> 스레드 안전하지 않음(not thread-safe)
- 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야한다.  
- ArrayList , HashMap  
- ```@NotThreadSafe``` 
 
> 스레드 적대적(thread-hostil)
- 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다.  
- 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정한다.  
- 문제를 고쳐 재배포 하거나 사용자제 (deprecated) API로 지정한다.  

### 동기화에 대한 문서화
📌 조건부 스레드 안전한 클래스는 주의하여 문서화해야 한다. 어떠한 순서로 호출할 때 외부 동기화 로직이 필요한지 그리고 그 순서대로 호출하려면 어떤 락 혹은 락을 얻어야만 하는지 알려주어야 한다.  

예를 들면 Collections.synchronizedMap의 API의 문서에는 아래와 같이 명시되어 있다.  
```java
/**
 * It is imperative that the user manually synchronize on the returned
 * map when iterating over any of its collection views
 * 반환된 맵의 콜렉션 뷰를 순회할 때 반드시 그 맵으로 수동 동기화하라
 * 
 *  Map m = Collections.synchronizedMap(new HashMap());
 *      ...
 *  Set s = m.keySet();  // Needn't be in synchronized block
 *      ...
 *  synchronized (m) {  // Synchronizing on m, not s!
 *      Iterator i = s.iterator(); // Must be in synchronized block
 *      while (i.hasNext())
 *          foo(i.next());
 *  }
 */
```  
- 반환 타입만으로 명확히 알 수 없는 정적 팩토리 메서드라면 위의 예시처럼 자신이 반환하는 객체에 대한 스레드 안전성을 문서화해야 한다.  

### 외부에서 사용할 수 있는 Lock
💡 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 메서드 호출을 원자적으로 수행할 수 있다.  
- 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용이 안된다.(CurrentHashMap과는 사용 X)  
- 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격 (denail-of-service attack)을 수행할 수도 있다.  
    - 이를 막기 위해 비공개 락 객체를 사용해야한다!  
- synchronized 메서드도 공개된 락에 속한다.  

#### 비공개 락 객체 관용구 - 서비스 거부 공격을 막아준다.
```java
// 비공개 락 객체, final 선언!
private final Object lock = new Object();

public void foo() {
  synchronized(lock) {
    ...
  }
}
```  
#### 락필드는 항상 final로 선언하라

👉 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용가능하다.  

👉 조건부 스레드 안전클래스에서는 특정 호출 순서에 필요한 락을 알려줘야하므로 이 관용구를 사용할 수 없다.

### 결론
- 모든 클래스가 자신의 스레드 안정성 정보를 명확히 문서화해야 한다.  
    - synchronized 한정자는 문서화와 관련이 없다.  
- 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할 때 외부 동기화가 요구되고, 그때 어떤 락을 얻어야 하는지도 알려줘야 한다.  
- 무조건적 스레드 안전 클래스를 작성할 때는 synchronized 메서드가 아닌 비공개 락 객체를 사용하자.
