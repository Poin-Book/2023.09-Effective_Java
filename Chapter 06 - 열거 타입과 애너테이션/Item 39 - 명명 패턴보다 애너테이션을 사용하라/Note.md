# 명명 패턴보다 애너테이션을 사용하라

> 작성자: {밀리}

## 목차

### 명명 패턴의 단점
#### 1. 오타가 나면 안 된다.
만약 test로 시작되어야 할 메서드 이름이 오타로 인해 tset로 작성되었다면, 명명 패턴에는 벗어나지만 프로그램 상에서는 문제가 없기 때문에 테스트 메서드로 인식하지 못하고 테스트를 수행하지 않는다.

#### 2. 명명 패턴을 의도한 곳에서만 사용할 거라는 보장이 없다.
개발자는 JUnit3의 명명 패턴인 'test'를 메서드가 아닌 클래스의 이름으로 지음으로써 해당 클래스의 모든 테스트 메서드가 수행되길 바랄 수 있다. 하지만 JUnit은 클래스 이름에는 관심이 없다. 따라서 개발자가 의도한 테스트는 전혀 수행되지 않는다.

 

#### 3. 명명 패턴을 적용한 요소를 매개변수로 전달할 마땅한 방법이 없다.
특정 예외를 던져야 성공하는 테스트가 있을 때, 메서드 이름에 포함된 문자열로 예외를 알려주는 방법이 있지만 보기 흉할 뿐 아니라 컴파일러가 문자열이 예외 이름인지 알 도리가 없다.

##### ⚠️ 하지만 애너테이션을 사용하면 위의 단점을 모두 해결할 수 있다.

### 마커 애너테이션
> 아무 매개변수 없이 단순히 대상에 마킹하는 용도로 사용되는 애너테이션
#### 마커 애너테이션 타입 선언 예시
```java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적 메서드 전용
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{}
```
- 위 예제를 보면 애너테이션 안에 또 다른 애너테이션이 달려 있다.  
➡️ 이를 **메타 에너테이션**이라고 한다.
즉 프로그래머가 Test이름에 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내준다.
- ```@Retention(RetentionPolicy.RUNTIME)``` - 보존 정책  
    - ```@Test```가 런타임에도 유지되어야 한다는 표시  
- ```@Target(ElementType.METHOD)``` - 적용 대상  
    - ```@Test```가 반드시 메서드에 선언되어야 한다는 표시  

#### 마커 에너테이션을 사용한 예시
```java
public class Sample {
    @Test
    public static void m1() { }        // 성공해야 한다.
    public static void m2() { }
    @Test public static void m3() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m4() { }  // 테스트가 아니다.
    @Test public void m5() { }   // 잘못 사용한 예: 정적 메서드가 아니다.
    public static void m6() { }
    @Test public static void m7() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m8() { }
}
```  
- ```@Test``` 애너테이션은 Sample 클래스의 의미에 직접적인 영향을 주지 않고, 단지 관심 있는 프로그램에게 추가 정보를 제공한다. 즉 대상 코드의 의미는 그대로 둔 채 그 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 주는 것이다.

#### 마커 에너테이션을 처리하는 프로그램(39-3)
```java
import java.lang.reflect.*;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```  
- 이 테스트 러너는 명령줄로부터 완전 정규화된 클래스 이름을 받아, 클래스에서 ```@Test``` 애너테이션이 달린 메서드를 찾아 차례로 호출한다. 그리고 애너테이션을 잘못 사용해 예외가 발생한다면 오류 메세지를 출력한다.  
  - 여기서 완전 정규화된 클래스란, 예를 들어 ```com.example.TestClass```에서 "com.example"는 패키지 이름이고 "TestClass"는 클래스 이름인데 이 둘을 모두 포함된 정규화된 형태이다.

### 매개변수를 받는 애너테이션 타입
> 만약 특정 예외를 던져야만 성공하는 테스트를 지원하려면, 다음과 같은 새로운 애너테이션 타입이 필요하다.  
#### 매개변수를 하나만 받는 애너테이션
```java
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();  // 매개변수
}
```  
- 이 애너테이션의 매개변수 타입은 ```Class<? extends Throwable>``` 이며(한정적 타입 토큰), 이는 "Throwable을 확장한 클래스의 Class 객체 "라는 뜻이다. 따라서 모든 예외와 오류 타입을 수용한다.

#### 매개변수 하나짜리 애너테이션을 사용한 프로그램
```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
```  
#### 39-3 테스트 도구 수정 코드
```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
         try {
               m.invoke(null);
               System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
         } catch (InvocationTargetException wrappedEx) {
               Throwable exc = wrappedEx.getCause();
               Class<? extends Throwable> excType =
               m.getAnnotation(ExceptionTest.class).value();
                if (excType.isInstance(exc)) {
                     passed++;
                } else {
                     System.out.printf(
                                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                }
           } catch (Exception exc) {
                System.out.println("잘못 사용한 @ExceptionTest: " + m);
           }
}
```
💡 앞의 코드(39-3)와 한 가지 차이라면, 이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는데 사용한다.  

#### 마커 애너테이션과 매개변수 하나짜리 애너태이션을 처리하는 프로그램
```java
 if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedEx) {
                    Throwable exc = wrappedEx.getCause();
                    Class<? extends Throwable> excType =
                            m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf(
                                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
```  
여기서 더 나아가 예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다. 기존 애너테이션에 Class 객체를 아래와 같이 배열로 수정해보자.  
#### 배열 매개변수를 받는 애너테이션 타입
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```  
- 원소가 여럿인 배열일 지정할 때는 다음과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해주기만 하면 된다.  
#### 배열 매개변수를 받는 애너테이션을 사용하는 코드
```java
@ExceptionTest({ IndexOutOfBoundsException.class,
                     NullPointerException.class })
public static void doublyBad() {   // 성공해야 한다.
     List<String> list = new ArrayList<>();

     // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
     // NullPointerException을 던질 수 있다.
     list.addAll(5, null);
}

```
다음은 이 새로운 @ExceptionTest를 지원하도록 테스트 러너를 수정한 코드이다. 
#### 배열 매개변수를 받는 애너테이션을 처리하는 코드
```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
             m.invoke(null);
             System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
         } catch (Throwable wrappedExc) {
             Throwable exc = wrappedExc.getCause();
             int oldPassed = passed;
             Class<? extends Throwable>[] excTypes =
                     m.getAnnotation(ExceptionTest.class).value();
             for (Class<? extends Throwable> excType : excTypes) {
                 if (excType.isInstance(exc)) {
                       passed++;
                       break;
                 }
              }
              if (passed == oldPassed)
                  System.out.printf("테스트 %s 실패: %s %n", m, exc);
              }
          }
 }
```  
- 하지만 위의 코드를 더 간단하게 개선하고 싶다면, 자바 8에서는 여러개의 값을 받는 애너테이션을, 배열 매개변수를 사용하는 대신 ```@Repeatable``` 메타 애너테이션을 다는 방식을 선택하여 코드 가독성을 높일 수 있다.
- @Repeatable를 단 애너테이션은 하나의 프로그램 요소에 여러번 달 수 있다.

📌 주의 사항
> 1) @Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
> 2) 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
> 3) 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.(그렇지 않으면 컴파일 X)  

#### 반복 가능한 애너테이션 타입
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)  //컨테이너 애너테이션 class 객체
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

//컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();     // value 메서드 정의
}

```

#### 반복 가능한 애너테이션을 두 번 단 코드
```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
      List<String> list = new ArrayList<>();

      // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
      list.addAll(5, null);
 }
```  
- 반복 가능 애터네이션은, 처리할 때도 주의 사항이 존재한다. 
    - 애너테이션을 여러개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용되기 때문에 m isAnnotationPresent() 에서 둘을 명확히 구분하고 있는 것을 볼 수 있다.

    - 하지만, 해당 메서드로 반복 가능 애너테이션이 달렸는지 검사한다면 검사에 실패할 것이고(애너테이션을 여러 번 단 메서드들을 무시) 컨테이너 애너테이션이 달렸는지만 검사하여도 반복 가능 애너테이션을 한번만 단 메서드를 무시하고 지나치기 때문에 둘을 따로따로 확인해야 한다.  

#### 반복 가능 애너테이션 다루기
```java
if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
        tests++;
        try {
             m.invoke(null);
             System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (Throwable wrappedExc) {
             Throwable exc = wrappedExc.getCause();
             int oldPassed = passed;
             ExceptionTest[] excTests =
                       m.getAnnotationsByType(ExceptionTest.class);
             for (ExceptionTest excTest : excTests) {
                 if (excTest.value().isInstance(exc)) {
                     passed++;
                     break;
                 }
             }
             if (passed == oldPassed)
                 System.out.printf("테스트 %s 실패: %s %n", m, exc);
            }
      }
}
```
- 이러한 방법은 코드 가독성을 높일 수 있지만 애너테이션을 처리하는 부분의 코드가 늘어나고 복잡해서 오류가 발생할 확률이 커진다는 것을 명심하자. 하지만 이번 예제는 이를 감수하더라도 명명 패턴보다 애너테이션이 좋다는 것을 확실히 보여준다.


### 결론
> 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없으며, 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.

