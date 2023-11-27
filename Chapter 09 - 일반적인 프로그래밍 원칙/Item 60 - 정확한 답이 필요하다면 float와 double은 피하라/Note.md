# 정확한 답이 필요하다면 float와 double은 피하라

> 작성자: 캐슬

## 목차
1. float와 double 사용할 때 문제점
2. BigDecimal 사용
    1. BigDecimal 의 단점
3. Int / long 타입 사용
4. 핵심 정리

## float와 double 사용할 때 문제점

float와 double 타입은 금융관련 계산과는 맞지 않는다.

- 0.1혹은 10의 음의 거듭 제곱 수를 표현할 수 없기 때문이다.

ex) 1.03달러중 42센트를 사용했을 때

```python
System.out.println(1.03-0.42);
// 출력값 : 0.6100000000000001
```

출력을 하기 전 반올림을 하여도 틀린 답이 나올 수 있다.

코드 60-1 오류발생! 금융 계산에 부동 소수 타입을 사용

```python
public static void main(String[] args){
  double funds = 1.00;
  int itemsBought = 0;
  for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price; 
    itemsBought++;
  }
  System.out.println(itemsBought); //  itemsBought -> 3 
  System.out.println("잔돈" + funds);  // -> 0.39999999999999
}
```

- 잔돈 출력 값은 0.399999… 달러가 된다. ⇒ 잘못된 결과이다.

> 금융 계산에는 BigDecimal, int 혹은 long을 사용해야 한다.
>

## 1. BigDecimal 사용

*코드60-2 BigDecimal을 사용한 해법. 속도가 느리고 쓰기 불편하다.*

```python
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS;
        funds.compareTo(price) >= 0;
        price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입"); // -> 사탕 4개 
    System.out.println("잔돈(달러): " + funds); // -> 잔돈 0달러 
}
```

- 잔돈 출력 값은 0달러가 된다 ⇒ 정확한 결과이다.

### BigDecimal 의 단점

- 기본 타입보다 쓰기가 불편하고 느리다.
    - 단발성 계산이면 문제는 무시할 수 있지만 쓰기에 불편하다.

## 2. Int / long 타입 사용

- int/long 타입 사용시 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.
- 위 예제에서는 모든 계산을 달러 대신 센트로 수행하면 이문제를 해결할 수 있다.

*코드 60-3 정수 타입을 사용한 해법*

```python
public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(센트): " + funds);
}
```

## 핵심 정리

정확한 답이 필요한 계산에는 float나 double을 피하라. 소수점 추정은 시스템에 맡기고, 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라.

BigDecimal이 제공하는 여덞 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다.

법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서 아주 편리한 기능이다.

반면, 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라.

숫자를 아홉 자리 십진수로 표현할 수 있다면 int를 사용하고,

열여덞 자리 십진수로 표현할 수 있다면 long을 사용하라. 열여덞 자리를 넘어가면 BigDeciamal을 사용해야 한다.