# 예외를 무시하지 말라

> 작성자: 루카

## 목차

- [예외를 무시하지 말라](#예외를-무시하지-말라)
  - [목차](#목차)
  - [예외를 무시하는 방법](#예외를-무시하는-방법)
  - [부적절한 예외 처리](#부적절한-예외-처리)
  - [예외를 무시해도 되는 경우](#예외를-무시해도-되는-경우)

## 예외를 무시하는 방법

```java
try {
  ...
} catch (SomeException e) {
  // 아무것도 하지 않으면 예외가 무시된다.
}
```

- catch 블록을 비워두면 예외가 존재할 이유가 없어진다.
  - 화재 경보를 무시하는 수준이 아니라 그냥 꺼버린 거나 다름없다.
  - 빈 catch 문을 보거든 당신의 머릿속의 사이렌이 돌아가야 한다.
- API 설계자가 메서드 선언에 예외를 명시한 까닭은, 적절한 조치를 취해달라고 말하는 것이다.

## 부적절한 예외 처리

1. 냅다 무시

```java
try {
  ... // 예외가 발생할 수 있는 코드
} catch (SomeException e) {
}
```

2. 예외를 기록만 하고 끝내기

```java
try {
  ... // 예외가 발생할 수 있는 코드
} catch (SomeException e) {
  logger.log(e.getMessage()); // or e.printStackTrace();
}
```

3. 무책임한 throws 선언
     - 앞선 두 방식보단 나은 편이다. 적어도 예외를 숨기진 않는다.
     - 추상화 수준에 맞게 던질 수 있다면 더 나은 선택지가 될 수도 있다. (Item 73)

```java
void someMethod() throws Exception {
  ... // 예외가 발생할 수 있는 코드
}
```

## 예외를 무시해도 되는 경우

> 적어도 무시하기로 했다면, 그 이유를 주석으로 남기고 예외 변수의 이름도 `ignored`로 바꿔놓자.

```java
public class FileInputStreamEx {

  private static final File defaultFile = new File("defaultFilePath");

  public static void main(String[] args) {
    FileInputStreamEx fx = new FileInputStreamEx();

    FileInputStream fileInputStream = fx.openFile();
    close(fileInputStream);
  }

  public FileInputStream openFile() {
    String filePath = (new Scanner(System.in)).nextLine();
    File file = new File(filePath);

    try {
      return new FileInputStream(file);
    } catch (FileNotFoundException e) {
      return openFile();
    }
  }

  public static void close(FileInputStream fileInputStream) {
    try {
      fileInputStream.close();
    } catch (IOException ignored) { 
      // 아래의 이유로 예외를 무시한다.
      ignored.printStackTrace(); // 예외가 주기적으로 발생한다면 조사하기 좋게 로그를 남긴다.
    }
  }
}
```

- FileInputStream의 close 메서드가 IOException을 던질 수는 있으나 닫을 때는 딱히 복구할 게 없다.
  - 입력 전용 Stream 이므로 파일의 상태를 변경하지 않는다.
  - Stream을 닫는 다는 것은 필요한 정보를 이미 다 읽었다는 의미라 남은 작업 또한 없다.
- 혹시나 파일이 닫히지 못하는 경우가 주기적으로 발생한다면, 그 사실을 로그로 남기는 방법도 좋다.

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // 기본값
try {
  numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
  // 기본값을 사용한다. (색상 수를 최소화하면 좋으나 필수는 아니다.)
}
```

- 검사와 비검사 예외에 똑같이 적용하라.
- 예외를 적절히 처리한다면 오류를 완전히 피할 수도 있다.
- 적어도 바깥으로 전파될 수 있도록 놔두기만 해도 최소한 디버깅 정보를 남긴채 프로그램이 죽는 경우는 방지할 수 있다.