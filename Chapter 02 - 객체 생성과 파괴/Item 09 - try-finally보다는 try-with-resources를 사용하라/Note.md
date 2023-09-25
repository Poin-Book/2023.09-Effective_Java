# try-finally보다는 try-with-resources를 사용하라

> 작성자: 피터

## 목차
- [try finally보다는 try with resources를 사용하라](#try-finally보다는-try-with-resources를-사용하라)
  - [1. try-finally](#1-try-finally)
  - [2. try-with-resources](#2-try-with-resources)
  - [정리](#정리)

 <br/>
자바 라이브러리에는 InputStream, OutputStream, java.sql.Connection 등 사용 후 직접 닫아 줘야 하는 자원이 있다. 클라이언트가 놓치는 경우가 많아 보통 finalizer를 많이 사용한다. 하지만 item 8에서 언급한 대로 문제가 많았고 결국 JDK 9에서 Deprecated됐다.

### 1. try-finally

그동안 전통적으로 자원이 제대로 닫힘을 보장하기 위해 try-finally를 일반적으로 사용하였습니다.자원을 한 개만 사용하면 단순하지만 두 개 이상 사용 시 코드가 지저분해지고 가독성이 떨어진다는 단점이 있습니다. 또한 아래 firstLoneOfFile 안에 readLine()으로 파일을 읽던 중 물리적인 오류가 생길 경우 예외를 던지게 될 것이다. 이때 마찬가지의 이유로 br.close()또한 예외가 발생할 수 있다. 예를 들어 파일이 존재하지 않는데, FileReader와 BufferedReader가 생성되기 때문에 발생한다. finally 블록이 실행되기 전에 해당 예외가 상위 호출 스택으로 전파됩니다. 따라서 br.close()가 실행되지 않을 수 있다. 

 한 개의 try catch문에 예외가 여러 번 발생하는 경우, 마지막에 발생한 예외가 이전 예외를 덮어쓰기 때문에 close() 예외가 readLine()예외를 넘기게 되고 stack trace 내역에 첫 번째 기록은 남지 않게 되어, 디버깅이 어려워 진다. 

```java
static String firstLineOfFile(String path) throws IOXception {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
			br.close();
	}
}
```

### 2. try-with-resources

try-finally에서 발생한 문제들은 java7이 제시한 try-with-resources가 해결하였다.

Try-with-resources가 모든 객체의 close()를 호출해주지는 않는다. try-with-resources를 사용하려면  사용하려는 자원이 AutoCloseable interface를 구현해야 한다. 예를 들어 inputstream의 경우 AutoCloseable를 상속한 Closeable interface를 만들고 그 Closeable을 상속하였다.

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

```java
public abstract class InputStream extends Object implements Closeable {
    ...
}

public interface Closeable extends AutoCloseable {
    void close() throws IOException;
}
```

try-finally를 사용한 코드를 try-with-resource를 사용해 재작성하게 되면 아래와 같다. 

```java
static String firstLineOfFile(String path) throws IOXception {
	try (BufferedReader br = new BufferedReader(new FileReader(path))) {
		return br.readLine();
	}
}
```

코드가 단순해지고 가독성이 증가했다. readLine()과 close() 호출 양쪽에서 예외가 발생하면, close()에서 발생한 예외는 숨겨지고 readLine()에서 발생한 예외가 기록된다. 이때 숨겨진 예외다. stack trace에서 suppressed 달고 출력된다. 또한 Throwable의 getSuppressed()를 사용하여 코드에서도 가져올 수 있다. 또한 try-finally도 catch절을 사용하여 다수의 예외를 처리할 수 있다. 

여기서 try-finally와 달리 덮여지지 않고 예외가 숨겨져서 stack-trace 되는 이유는 블록 내에서 발생한 예외와 Closeable를 구현한 class에서 발생한 예외가 서로 다른 stack trace를 갖고 있기 때문이다.

close()메서드 구현시 구체적인 exception을 throw 하고, close()동작이 전혀 실패할 리가 없을 때는 exception을 throw 하지 않도록 구현하는 것을 강력하게 권고하고 있다. 특히 close()메서드에서 InterruptedException을 던지지 않는 것을 강하게 권고한다.  InterruptedException은 쓰레드의 인터럽트 상태와 상호작용 하므로 interruptedException이 억제되었을 때 런타임에서 잘못된 동작이 발생할 수 있기 때문이다.

### 정리

꼭 회수해야 하는 자원을 다룰 때는 try-finally 대신 try-with-resources를 사용해야 한다. 코드가  간결해지고 가독성도 올라가며 stack trace에 추적되는 예외 또한 매우 유용하다. 자원 또한 정확하고 쉽게 회수 할 수 있다.
