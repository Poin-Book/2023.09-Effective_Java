# 톱레벨 클래스는 한 파일에 하나만 담으라

> 작성자: {밀리}

## 목차


### 제목의 의미
톱레벨 클래스란, 소스파일에서 가장 바깥에 존재하는 클래스를 말한다. 일반적으로 하나의 소스파일에서는 하나의 톱 레벨 클래스를 가진다. 예로 파일 이름이 SimpleService.java라면 클래스 이름도 SimpleService로 하나의 톱 레벨 클래스를 갖는다. 쉽게 말하면  
> 하나의 java 파일에는 하나의 class를 생성하자 라는 의미이다.


### 왜 한파일에 담아야 될까?
- 톱레벨 클래스를 여러개 선언하면 이득은 없고 심각한 위험만 발생할 수 있다.
  - 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하느냐에 따라 달라지기 때문이다.


 
#### Main.java
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```
#### Utensil.java
```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```
#### Dessert.java
```java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```
- 위 코드들은 ```Utensil.java```와 ```Dessert.java```라는 두 개의 파일에 중복으로 정의된 2개의 클래스의 예이다.
- 위와 같은 경우 javac 명령어에 들어가는 인수에 따라 실행결과가 달라진다.
```
javac Main.java Dessert.java: 컴파일 에러, Utensil과 Dessert 클래스가 중복 정의되었습니다.
javac Main.java // pancake 출력
javac Main.java Utensil.java // pancake 출력
javac Dessert.java Main.java // potpie 출력
```
- 동작원리
  - ```Main.java```가 먼저 인수에 들어왔을 때, 자바는 ```Main.java```를 실행시키며, ```Utensil.NAME```을 만나고, ```Utensil.java``` 파일을 찾아서 클래스를 로드하려한다.  
    - 그래서 ```javac Main.java```의 경우 "pancake"가 출력된다.  
    - 그래서 ```javac Main.java Dessert.java```의 경우 클래스가 중복으로 선언되었다고 알린다.  
  - ```Dessert.java```가 먼저 인수에 들어왔을 때는, 자바는 Utensil 클래스와 Dessert 클래스의 정의를 불러와 놓는다.  
    - 그래서 ```javac Dessert.java Main.java```의 경우 "potpie"가 출력된다.
    - 
- 즉, **컴파일러에 어느 소스파일을 먼저 건네느냐에 따라서 정상 동작할수도 또는 컴파일 오류가 발생할 수도 있다.**  
- 파일을 나누면 위와 같은 복잡한 동작원리도 알 필요 없고, 잠재적 에러도 없으므로 ```Utensil.java```와 ```Dessert.java``` 각 파일별로 클래스는 1개씩만 선언하는 것이 좋다.

### 해결 방법
- 톱레벨 클래스(혹은 톱레벨 인터페이스)들을 서로 다른 소스 파일로 분리한다.  
- 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하자.
```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil{
        static final String NAME = "pan";
    }
    
    private static class Dessert{
        static final String NAME = "cake";
    }
}
```
 

