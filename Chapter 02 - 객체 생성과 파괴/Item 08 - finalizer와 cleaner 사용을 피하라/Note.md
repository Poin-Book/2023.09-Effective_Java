# finalizer와 cleaner 사용을 피하라

> 작성자: 루키

## 목차
- [finalizer와 cleaner 사용을 피하라](#finalizer와-cleaner-사용을-피하라)
  - [목차](#목차)
  - [본문](#본문)
    - [1. Java의 객체 소멸자](#1-java의-객체-소멸자)
    - [2. 객체 소멸자를 사용하면 안되는 이유](#2-객체-소멸자를-사용하면-안되는-이유)
    - [3. 그러면 자원 회수를 명시적으로 해줄때는 어떻게 하나](#3-그러면-자원-회수를-명시적으로-해줄때는-어떻게-하나)
    - [4. 그럼 객체 소멸자는 언제 사용하는 것이 적절한가](#4-그럼-객체-소멸자는-언제-사용하는-것이-적절한가)
    - [5. Cleaner 예시](#5-cleaner-예시)

## 본문

### 1. Java의 객체 소멸자

1. finalizer
    - Garbage Collector 스레드가 객체의 자원을 회수할 때 (메모리를 회수할때) 호출하는 메서드
    - 객체는 JVM에 어떠한 타입의 참조가 남아있지 않을때 GC의 대상이 된다. (팬텀 도달 가능)
        
        [http://www.cs.fsu.edu/~jtbauer/cis3931/tutorial/refobjs/about/phantom.html#:~:text=An object is phantomly reachable,been finalized%2C but not reclaimed](http://www.cs.fsu.edu/~jtbauer/cis3931/tutorial/refobjs/about/phantom.html#:~:text=An%20object%20is%20phantomly%20reachable,been%20finalized%2C%20but%20not%20reclaimed).
        
    - Object라는 객체 최상위 클래스에 정의되어있으며 각 객체는 이를 override하여 사용함
    - 최신 JVM에서는 deprecate되어 사용되지 않음
    - 코드
        
        ```java
        @Override  protected void finalize() throws Throwable {
              try {
        					... // cleanup subclass state      
        			} finally {
                  super.finalize();
              }  
        }
        ```
        
2. cleaner
    - Java 9에서 finalizer대신에 도입 된 소멸자이다.
    - 아래의 코드처럼 AutoClosable을 구현하여 (implements)하여 같이 사용함
        
        ```java
        public class CleaningExample implements AutoCloseable {
                 // A cleaner, preferably one shared within a library
                 private static final Cleaner cleaner = Cleaner.create();
        
                 static class State implements Runnable {
        						 //상태
                     State(...) {
                       // initialize State needed for cleaning action
                   }
                     public void run() {
                       // cleanup action accessing State, executed at most once
                   }
                 }
                 private final State state;
        
                 private final Cleaner.Cleanable cleanable;
        
                 public CleaningExample() {
                   this.state = new State(...);
        					 //등록
                   this.cleanable = cleaner.register(this, state);
        	       }
        
                 public void close() {
                   cleanable.clean();
                 }
           }
        ```
        
    - 보통 위와 같이 명시적으로 사용할 때에는 try-with-resource와 함께 사용한다.

### 2. 객체 소멸자를 사용하면 안되는 이유

1. 수행 시점이 보장되지 않음
    - 얘네들은 즉시 수행된다는 보장이 없다.
    - 즉 finalizer가 원하는 시간에 실행이 된다는 보장이 없다.
    - 따라서 제 시간에 실행되어야 하는 작업은 할 수 없다.
        - 예를 들어 파일 닫기를 얘네들에게 맡기면 시스템이 열 수 있는 파일의 갯수를 넘어가는 상황에서도 여전히 파일을 닫아주지 않아 시스템 전체가 망가지는 경우가 있을 수 있다.
    - 다만 cleaner는 수행할 스레드를 통제할 수 있다는 점에서 그나마 낫지만
    - 여전히 백그라운드 작업이며 GC의 통제 아래에 있기 때문에 즉각 수행은 보장할 수 없다.
        
        > 따라서 상태를 영구적으로 수정하는 작업에서는 절대 사용하면 안된다. (ex. 공유 자원의 lock 해제)
        > 
    - finalizer가 제때 자원을 회수하지 않아 계속 쌓이게 되면 Out Of Memory Error를 벹으며 프로그램이 뻗을 수 있다.
    - 왜냐하면 finalizer 스레드는 그 우선순위가 낮기 때문에 계속 후순위로 밀려 오래 대기하기 때문이다.
    - 이러한 면에서 Out Of Memory Error가 안나더라도 자원을 계속 잡아먹기 때문에 성능 저하가 있을 수 있다.
    - 혹자는 아래와 같이 말 할 수 있다.
        
        > System.gc나 System.runFinalization 메서드는 무너졌냐?
        > 
        - 맞다 무너졌다. finalizer와 cleaner가 실행될 가능성은 높여주지만 보장해주지는 않음
        - 이 문제를 해결해보겠다고 나온게 다음 메서드들이지만…
            - System.runFinalizersOnExit
            - Runtime.runFinalizersOnExit
        - 이 2개의 메서드는 심각한 결함이 있음
            - ThreadStop
                - 클라이언트와 finalize에서 동시에 자원에 접근하여
                - 두개의 스레드가 충돌이나게 되며 synchroized(resource)를 해야한다.
                - 자세한 내용은 링크 참고
                    
                    http://mkseo.pe.kr/blog/?p=1029
                    
2. 예외 처리 에서의 문제
    - finalizer 작동시에 발생한 예외는 무시되어 (왜?) → 아래 코드를 실행시켜 보자 그럼 이해가 된다
        
        ```java
        public class ExceptionInFinalizeTest {
        
        	public static void main(String[] args) throws Throwable {
        		ExceptionInFinalizeTest exceptionInFinalizeTest = new ExceptionInFinalizeTest();
        		exceptionInFinalizeTest = null;
        
        		// System.gc does not guarantee finalize, but generally works fine.
        		System.gc();
        	}
        
        	@Override
        	protected void finalize() throws Throwable {
        		System.out.println("The finalize method start");
        
        		// Exceptions are ignored.
        		System.out.println(2 / 0);
        
        		super.finalize();
        
        		System.out.println("The finalize method end"); // not printed
        	}
        }
        ```
        
        - 실행 시켜보면 알겠지만 예외를 의도적으로 냈음에도 불구하고 StackTrace가 뜨지 않음
    - 처리할 작업이 남아있더라도 그 순간 종료 됨
    - 따라서 이러한 잡지 못한(처리하지 못한) 예외들 때문에 객체가 불완전해진다
    - 이러한 훼손된 객체를 이용하면 예상치 못한 버그가 발생함
    - StackTrace도 출력하지 않음
    - 그나마 cleaner는 자신의 스레드를 통제하기 때문에 이러한 문제에서는 자유로움
3. 성능 이슈
    - 책에서 나온 예시는 다음과 같다
        - AutoCloseable 객체를 생성해서 GC가 수거하기 까지 12ns가 걸렸고
        - finalizer를 사용하면 550ns가 걸렸다
    - 따라서 finalizer를 사용하는 것이 엄청 느리다는 것을 확인 할 수 있다.
    - 이 이유는 finalizer가 GC의 효율을 떨어뜨리기 때문이다(왜?)
    - cleaner도 클래스내의 모든 인스턴스를 수거하는 방식으로 사용하면 비슷하다.
    - 다만 안전망 방식으로는 훨씬 빠르다
4. 보안 이슈
    - 생성자나 직렬화 과정에서 (readObject, readResolve 메서드) 예외가 발생하면
    - 이 불완전한 객체에서 악의적인 하위 클래스의 finalizer를 실행하게 한다.
    - 또한 이 finalizer는 정적 필드에 본인의 참조를 할당하여 GC의 수집을 막는다
    - 따라서 악성 객체가 계속 메모리에 남게 된다.
    - 이 공격을 방어하는 방법은
        - 아무일도 안하는 finalizer를 구현하고 final로 만들어버리면 된다.
5. JVM 구현에 따른 동작의 가변성
    - finalizer나 cleaner를 언제 실행할지는 JVM마다 다른 GC 알고리즘에 달려있음
    - 따라서 환경마다 작동 여부가 달라질 수 있다.

### 3. 그러면 자원 회수를 명시적으로 해줄때는 어떻게 하나

- AutoCloseable 인터페이스를 implements해주면 된다. (Cleaner를 사용)
- 이후 클라이언트가 인스턴스를 다 쓰고 나서 close 메서드를 호출하면 끝이다.
    - 이때 예외가 발생해도 재대로 종료되도록 try-with-resource를 사용해야 한다.
    - 또한 각 인스턴스는 자신이 닫혔는지 확인을 하는 것이 좋음
        - close 메서드에서 해당 객체가 더 이상 유효하지 않음을 필드에 기록하고
        - 다른 메서드들은 해당 필드를 검사하여 유효하지 않으면 IllegalStateException을 던지면 된다.

### 4. 그럼 객체 소멸자는 언제 사용하는 것이 적절한가

1. 최종적인 자원 회수에 대한 안전망
    
    > 자원의 소유자가 close 메서드를 호출하지 않을때를 대비한 안전망
    > 
    - cleaner나 finalizer가 즉시 호출되지는 않겠지만
    - 적어도 아예 닫지 않는 것 보다는 이렇게라도 닫는 것이 좋기 때문이다.
    - 예를 들어 다음 3가지가 있다.
        - FileInputStream
        - FileOutputStream
        - ThreadPoolExecutor
    - 코드
        
        ```java
        public class FileInputStream extends InputStream
        {
            ...
        
            /**
             * Ensures that the <code>close</code> method of this file input stream is
             * called when there are no more references to it.
             *
             * @exception  IOException  if an I/O error occurs.
             * @see        java.io.FileInputStream#close()
             */
            protected void finalize() throws IOException {
                if ((fd != null) &&  (fd != FileDescriptor.in)) {
                    /* if fd is shared, the references in FileDescriptor
                     * will ensure that finalizer is only called when
                     * safe to do so. All references using the fd have
                     * become unreachable. We can call close()
                     */
                    close();
                }
            }
        }
        ```
        
2. 네이티브 자원 회수
    
    > 네이티브 자원이란? (네이티브 피어)
    > 
    - 참고
        - https://stackoverflow.com/questions/48260485/what-is-a-native-peer
        - https://www.infoworld.com/article/2077520/java-tip-23--write-native-methods.html
    - Java로 작성된 것이 아닌 C나 어셈블리로 작성된 코드 (네이티브 메서드)
    - 이 코드들은 GC의 대상이 되지 않아 소멸자로 일일히 회수해야 함
        - 왜? 자바 객체가 아니기 때문에
        - 다만 성능 저하를 감당 할 수 있고 네이티브 피어가 중요한 자원을 가지고 있을 않을때에 한한다.
        - 그렇지 않다면 앞의 close 메서드를 사용해야 한다.
    - 자바 객체의 네이티브 부분을 네이티브 피어라고 한다.
        - 다른 언어로 작성된 프로그램을 실행/사용 하기 위한 객체
        - 이해하기 어려우면 다른 언어로 작성된 프로그램을 사용하기 위한 인터페이스라고 생각하자
        - 일반 자바 객체가 아님

### 5. Cleaner 예시

- Room.java
    
    ```java
    public class Room implements AutoCloseable {
    		//클리너를 만들자
        private static final Cleaner cleaner = Cleaner.create();
    
        // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
        private static class State implements Runnable {
            int numJunkPiles; // Number of junk piles in this room
    
            State(int numJunkPiles) {
                this.numJunkPiles = numJunkPiles;
            }
    
            // close 메서드나 cleaner가 호출한다.
            @Override public void run() {
                System.out.println("Cleaning room");
                numJunkPiles = 0;
            }
        }
    
        // 방의 상태. cleanable과 공유한다.
        private final State state;
    
        // cleanable 객체. 수거 대상이 되면 방을 청소한다.
        private final Cleaner.Cleanable cleanable;
    		
    		//Room의 생성자
        public Room(int numJunkPiles) {
            state = new State(numJunkPiles);
    				//클리너 등록
            cleanable = cleaner.register(this, state);
        }
    		
    		//객체 회수시 호출되는 close 메서드
        @Override public void close() {
            cleanable.clean();
        }
    }
    ```
    
- Adult.java
    
    ```java
    // cleaner 안전망을 갖춘 자원을 제대로 활용하는 클라이언트 (45쪽)
    public class Adult {
        public static void main(String[] args) {
            try (Room myRoom = new Room(7)) { //try-with-resource
                System.out.println("안녕~");
    						//잘 보면 Room 객체를 안닫았음
            }
        }
    }
    ```
    
    - 출력
        
        ![image](https://github.com/Poin-Book/2023.09-Effective_Java/assets/74547868/135fefad-1c14-448a-8576-26e5735f3d8b)
        
        - run() 메서드가 호출 됨
    - 이 예시는 try-with-resource를 활용했으므로 실행이 끝나고 알아서 close 메서드를 호출
        - Room 객체를 닫지 않았더라도 말이다. (안전망 역할)
- Teenager.java
    
    ```java
    // cleaner 안전망을 갖춘 자원을 제대로 활용하지 못하는 클라이언트 (45쪽)
    public class Teenager {
        public static void main(String[] args) {
            new Room(99);
            System.out.println("Peace out");
    
            // 다음 줄의 주석을 해제한 후 동작을 다시 확인해보자. (Cleaning Room 이 나온다)
            // 단, 가비지 컬렉러를 강제로 호출하는 이런 방식에 의존해서는 절대 안 된다!
    //      System.gc();
        }
    }
    ```
    
    - 출력
        
        ![image](https://github.com/Poin-Book/2023.09-Effective_Java/assets/74547868/631fa898-b894-4a55-92b1-7dedb9de94be)
        
        - run() 메서드가 호출 X
        - 이 예시는 try-with-resource를 활용하지 않아 close 메서드 호출 X
    - 출력 (주석 풀고)
        
        ![image](https://github.com/Poin-Book/2023.09-Effective_Java/assets/74547868/5630f41c-1b43-406b-bb37-0f6803a91700)
        
        - run() 메서드가 호출 됨
        - 이 예시는 System.gc로 가비지 컬랙터를 강제로 호출하여 close 메서드 호출
        - 이렇게 하면 절대 안된다
    
    ### 6. Reference
    
    - https://camel-context.tistory.com/43