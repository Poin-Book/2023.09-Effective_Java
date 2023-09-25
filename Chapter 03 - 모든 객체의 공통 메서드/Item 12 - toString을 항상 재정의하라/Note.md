# toString을 항상 재정의하라

> 작성자: 럭키

## 목차
- [toString을 재정의하지 않으면?](#tostring을-재정의하지-않으면)
- [toString을 재정의해야 하는 이유](#tostring을-재정의해야-하는-이유)
- [toString 자동완성 주의](#tostring-자동완성-주의)
- [정리](#정리)

## toString을 재정의하지 않으면?
toString을 오버라이딩 하지 않는 경우(Object의 toString을 사용할 경우), Object에서 기본적으로 제공하는 toString은 클래스이름 + @ + 16진수로 표현한 해시코드 값을 보여준다.

![image1](https://github.com/Poin-Book/2023.09-Effective_Java/assets/110045522/7614ad33-eca0-4c79-bed7-5ea5f6d6ae0b)

## toString을 재정의해야 하는 이유
toString은 ‘간결하면서 읽기 쉬운 형태의 유익한 정보’를 반환해야 한다. 

또한, 모든 하위 클래스에서 이 메서드를 재정의하라.

→ toString을 재정의하지 않을 경우, 쓸모 없는 메시지만 출력됨.

```java
public class PhoneNumber1 {
    private String number;

    public PhoneNumber1(String number) {
        this.number = number;
    }
}

public class PhoneNumber2 {
    public PhoneNumber2(String number) {
        this.number = number;
    }

    private String number;

    @Override
    public String toString() {
        return "PhoneNumber2{" +
                "number='" + number + '\'' +
                '}';
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(new PhoneNumber1("010-1234-1234"));
        System.out.println(new PhoneNumber2("010-1234-1234"));
    }
}
```

![image2](https://github.com/Poin-Book/2023.09-Effective_Java/assets/110045522/f3d6a88c-1212-4bae-95ad-f2ac646a92df)

**실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다**<br>
⇒ 논란의 소지가 있다. 주요 정보는 모두 반환하는 게 좋다는 게, 반은 맞고 반은 틀린 말이라 생각.<br>
책에서는 객체가 가진 모든 정보를 노출하는 게 좋다고 말은 하나, 실제로 외부에 노출시키면 안 되는 데이터가 많다.<br>
주문 내역으로 어떤 것을 주문했는지, 로그 정보가 탈취 당할 수 있다면 로깅도 조심해야 하는 찰나, **모든 데이터가 노출되는 건 좋지 않다**.

**toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자**<br>
⇒ toString으로 **공개한 정보는 Getter를 제공**해야 한다. toString을 구성하는 데이터를 따로따로 받을 수 있는 메소드를 제공하라는 의미이다.<br>
제공하지 않으면 toString으로 클라이언트 코드가 쪼개서 사용할 수는 있으나 클라이언트가 쪼개서 사용하는 것보단 각각 제공하는 게 낫다.

**VO(ValueObject)** 의 경우, 아래 예시와 같이 어떤 포맷으로 변환되는지 명시적으로 표기해주는 게 좋다.

```java
/**
 * 이 전화번호의 문자열 표현을 반환한다.
 * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
 * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
 * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
 *
 * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
 * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
 * 전화번호의 마지막 네 문자는 "0123"이 된다.
 */
```

## toString 자동완성 주의!
toString을 자동완성으로 만들어 주는 방법도 있지만, 이 방법이 적절하지 않은 이유는 **원하는 포맷이 존재하는 경우**가 있다면 문제가 된다.

그리고 toString을 자동완성(lombok, ide)으로 만들 경우 조심해야 하는 경우가 있다. 

바로 순환참조.

**순환참조**: JPA에서 양방향으로 연결된 엔티티를 JSON 형태로 직렬화하는 과정에서, 서로의 정보를 계속 순환하며 참조하여 StackOverflowError 를 발생시키는 현상

이 순환참조 문제를 해결하기 위해<br>
Lombok에 @Data 안에는 @ToString이 있으므로 getter, settter를 사용할거면 @Getter, @Setter을 사용.<br>
Entity를 반환하는 대신 DTO를 활용.<br>
@JsonIgnoreProperties({"user"}) 어노테이션을 이용.<br>
@JsonManagedReference와 @JsonBackReference 활용.<Br>
이런 해결방법을 활용하게 된다.

그런데 이는 toString 사용할 때 보다는 단순히 순환참조 문제를 해결하는 데 초점이 되어있는 방안들. 

=> toString의 경우 자동생성보단 **객체의 특성에 맞게** 직접 만들어주는 게 적절하다.

## 정리
⇒ **toString은 항상 재정의해서 사용하도록 하자. 상위 클래스에서 이미 알맞게 재정의 한 케이스는 빼고**