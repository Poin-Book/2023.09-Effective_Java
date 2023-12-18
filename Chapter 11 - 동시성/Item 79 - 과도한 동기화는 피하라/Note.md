# Item 79 - 과도한 동기화는 피하라

> 작성자: @destiny3912

## 목차
- [Item 79 - 과도한 동기화는 피하라]([#98](https://github.com/Poin-Book/2023.09-Effective_Java/issues/98)item-79---과도한-동기화는-피하라)
  - [목차](#목차)
  - [본문](#본문)
    - [1. 들어가며](#1-들어가며)
    - [2. 과도한 동기화의 예시](#2-과도한-동기화의-예시)
    - [3. 성능 측면에서의 동기화](#3-성능-측면에서의-동기화)
    - [4. 정리](#4-정리)
## 본문
### 1. 들어가며

---

> 과도한 동기화는 안하느니만 못하다
> 
- 오히려 성능을 떨어뜨리고
- 교착상태(Deadlock)에 빠뜨리고
- 이상하게 동작한다.
- 따라서
    - 응답 불가와 안전 실패를 피하려면
    - **동기화 메서드나 블록 안에서는 제어를 클라이언트에 양도하지 말라**

### 2. 과도한 동기화의 예시

---

> 다음 코드는 집합에 원소가 추가되면 알림을 보내는 관찰자 패턴 코드이다.
> 

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

		// 코드 79-1 잘못된 코드. 동기화 블록 안에서 외계인 메서드를 호출한다. (420쪽)
    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }
}
```

1. addObserver와 removeObserver를 통해 구독을 하거나 취소한다.
    - 두 경우 모두 다음의 콜백 인터페이스의 인스턴스를 메서드에 pass 함
        
        ```java
        @FunctionalInterface
        public interface SetObserver<E> {
        	void added(ObserverableSet<E> set, E element);
        }
        ```
        
        - BiConsumer<ObserverableSet<E>><E>와 같음
        - 다만 다중 콜백을 지원하도록 확장이 가능해 사용함
2. 사용 예시 코드
    - 정수값을 순회하다가 23이 된다면 본인을 제거하는 코드
        
        ```java
        public static void main(String[] args) {
        	ObserverableSet<Integer> set = new ObserverableSet<>(new HashSet<>());
        	
        	set.addObserver((s, e) -> System.out.println(e));
        	
        	set.addObserver(new SetObserver<>() {
        		public void added(Observerable<integer> s, Integer e) {
        			System.out.orintln(e);
        	
        					if(e == 23) 
        						s.removeObserver(this);
        		}
        	});
        
        	for(int i = 0; i < 100; i++) {
        		set.add(i);
        	}
        }
        ```
        
        - 정상적으로 제거하지 않고 ConcurrentModificationException을 던짐
            - 관찰자의 added 메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이고
            - added메서드는 removeObserver 메서드를 거처 최종적으로 observers.remove 메서드를 호출함
                - 여기서 리스트에서 원소를 제거하려고 하는데 notifyElementAdded 메서드에서 순회를 진행하고 있음
                - 해당 순회는 동기화 블록 안에 있으므로 동시 수정은 방지를 하지만
                - 자신의 콜백이 수정하는 것은 막지 못함
    - 이번에는 백그라운드 스레드를 사용해보자
        
        ```java
        public static void main(String[] args) {
                ObservableSet<Integer> set =
                        new ObservableSet<>(new HashSet<>());
        
        				// 코드 79-2 쓸데없이 백그라운드 스레드를 사용하는 관찰자 (423쪽)
                set.addObserver(new SetObserver<>() {
                    public void added(ObservableSet<Integer> s, Integer e) {
                        System.out.println(e);
                        if (e == 23) {
                            ExecutorService exec =
                                    Executors.newSingleThreadExecutor();
                            try {
                                exec.submit(() -> s.removeObserver(this)).get();
                            } catch (ExecutionException | InterruptedException ex) {
                                throw new AssertionError(ex);
                            } finally {
                                exec.shutdown();
                            }
                        }
                    }
                });
        
                for (int i = 0; i < 100; i++)
                    set.add(i);
            }
        ```
        
        - 이 코드는 Deadlock에 빠진다.
        - s.removeObserver를 호출하면 lock을 하려고 하지만
        - 그 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하는 것을 대기중이다.
        - 따라서 Deadlock이다.
        
    - 같은 상황에서 불변식이 잠깐 깨진경우
        - 자바의 lock는 재진입이 가능하므로 Deadlock에 빠지진 않는다.
        - 위 예시에서 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고있어 다음번 락의 획득도 문제 없음
            - 락이 보호하는 데이터에 대해 관계 없는 다른 작업을 진행중인데도…
            - 따라서 락이 제 역할을 못하여 데이터 훼손까지 이어지는 경우가 생길 수 있다.
        - 이러한 부분은 다음과 같이 해결한다.
            
            ```java
            // 코드 79-3 외계인 메서드를 동기화 블록 바깥으로 옮겼다. - 열린 호출 (424쪽)
                private void notifyElementAdded(E element) {
                    List<SetObserver<E>> snapshot = null;
                    synchronized(observers) {
                        snapshot = new ArrayList<>(observers);
                    }
                    for (SetObserver<E> observer : snapshot)
                        observer.added(this, element);
                }
            ```
            
            - 외계인 메서드 호출을 동기화 블록 밖으로 빼면 된다.
            - 동기화 블록 밖으로 빼는 방법은 하나가 더 있다
                - CopyOnWriteArrayList를 사용하면 된다.
                - ArrayList를 구현한 클래스
                - 내부를 변경하는 작업은 복사본을 만들어 진행함
                - 따라서 내부를 수정하더라도 락이 필요 없음
            - 코드는 다음과 같다.
                
                ```java
                private final List<SetObserver<E>> observers =
                            new CopyOnWriteArrayList<>();
                
                    public void addObserver(SetObserver<E> observer) {
                        observers.add(observer);
                    }
                
                    public boolean removeObserver(SetObserver<E> observer) {
                        return observers.remove(observer);
                    }
                
                    private void notifyElementAdded(E element) {
                        for (SetObserver<E> observer : observers)
                            observer.added(this, element);
                    }
                
                    @Override public boolean add(E element) {
                        boolean added = super.add(element);
                        if (added)
                            notifyElementAdded(element);
                        return added;
                    }
                
                    @Override public boolean addAll(Collection<? extends E> c) {
                        boolean result = false;
                        for (E element : c)
                            result |= add(element);  // notifyElementAdded를 호출한다.
                        return result;
                    }
                }
                ```
                
            - 이렇게 동기화 블록 밖에서 호출하는 외계인 메서드를 open call이라고 한다.
            - 동기화 호출로 인한 대기를 없애줘서 효율을 크개 개선해 준다.

### 3. 성능 측면에서의 동기화

---

> 멀티코어가 일반화된 요즘 자바에서 동기화에 드는 시간은 Lock의 획득을 두고 경재하는 시간이다.
> 
- 다시 모든 코어가 메모리를 일관되게 보기 위한 (데이터 정합성) 지연시간이 진짜 비용이라고 할 수 있다.
- 또한 가상 머신의 코드 최적화를 제한한다는 점도 또 하나의 비용이 될 수 있겠다.
- 가변 클래스를 작성할때 다음을 따라야 한다.
    1. 동기화를 하지말고 그 클래스를 동시에 사용하는 클래스가 동기화하게 하라
    2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자
        - 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것 보다 동시성을 월등히 개선가능할 때만 이 방법을 사용한다.
        - 스레드 내부에서 동기화하기로 했다면 다음의 기법으로 동시성을 높일 수 있다
            - 락 분할
            - 락 스트라이핑
            - 비차단 동시성 제어
- 여기서 Java.util은 첫번째 방식을 java.util.concurrent는 두번째 방식을 취한다.
- StringBuffer는 동기화하지 않는 버전인 StringBuilder로, Random은 스레드 안전한 ThreadLocalRandom으로 바뀌었다.
- 여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 수정 대상인 필드를 사용하기 직전에 반드시 동기화 해야함
- 다만 클라이언트가 여러 스레드로 복제되어 동작한다면 다른 클라이언트에서 메서드를 호출하는 것을 막을 수 없음
- 따라서 외부에서 동기화 할 방법이 없다.
- 결과적으로 해당 정적 필드가 접근제어자와 상관 없이 전역 변수와 동일하게 쓰인다.

### 4. 정리

---

1. 교착 상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드 호출을 피하라
2. 동기화 영역 안에서의 작업은 최소한만 하라
3. 클래스가 가변이라면 동기화의 주체를 누가 가져갈지 고민하라
    - 합당한 이유가 있을때만 내부에서 동기화하고, 동기화 여부를 문서에 잘 적어두어라
4. 멀티코어 환경이라면 동기화를 피하는 것이 더더욱 중요하다.