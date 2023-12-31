# Item 74 - 메서드가 던지는 모든 예외를 문서화하라

> 작성자: 밀리

## 목차

### ⚠️ 메서드가 던지는 예외는, 그 메서드를 올바로 사용하는 데 아주 중요한 정보이다. 따라서 각 메서드가 던지는 예외 하나하나를 문서화하는 것은 매우 중요하다 ⚠️

#### 🤔 왜 중요할까?
- 공통 상위 클래스 하나로 뭉뚱그려 선언하면, 메서드 사용자에게 각 예외에 대처할 수 있는 힌트를 주지 못할뿐더러, 같은 맥락에서 발생할 여지가 있는 다른 예외들까지 삼켜버릴 수 있어 API 사용성을 크게 떨어뜨린다. (예외: main 메서드. main은 오직 JVM만이 호출하므로 Exception을 던지도록 선언해도 ok)
- 자신이 일으킬 수 있는 오류들이 무엇인지 알려주면 프로그래머는 자연스럽게 해당 오류가 나지 않도록 코딩한다.  
- 잘 정비된 비검사 예외 문서는 사실상 그 메서드를 성공적으로 수행하기 위한 전제조건이 된다.  
- 발생 가능한 비검사 예외를 문서로 남기는 일은 인터페이스 메서드에서 특히 중요하다. 이 조건이 인터페이스의 일반 규약에 속하게 되어 그 인터페이스를 구현한 모든 구현체가 일관되게 동작하기 때문이다.

#### 🤩 그럼 어떻게 사용해야 될까?
1. 검사 예외는 항상 따로 선언하고, 각 예외가 발생하는 상황을 자바독의 ```@throws``` 태그를 사용하여 정확히 문서화하자.  
👉 유일한 예외는 main 메서드다. JVM에서만 호출하므로 Exception을 던지도록 선언해도 된다.  
2. 자바 언어가 요구하는 것은 아니지만 비검사 예외도 검사 예외처럼 정성껏 문서화해두면 좋다.  
👉 public 메서드라면 필요한 전제조건을 문서화해야 하며(아이템56), 그 수단으로 가장 좋은 것이 바로 비검사 예외들을 문서화하는 것이다.  
3. 메서드가 던질 수 있는 예외를 각각 ```@throws``` 태그로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.  
👉 검사냐 비검사냐에 따라 API 사용자가 해야 할 일이 달라지므로 이 둘을 확실히 구분해주는 게 좋다.  
4. 클래스를 수정하면서 새로운 비검사 예외를 던지게 되어도 소스 호환성과 바이너리 호환성이 그대로 유지되기 때문에 비검사 예외의 문서화가 불가능할 때도 있다.  
5. 한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면 그 예외를 (각각의 메서드가 아닌) 클래스 설명에 추가하는 방법도 있다.  
📌 ex) NullPointerException

### 결론
- 메서드가 던질 수 있는 모든 예외를 문서화하라. (검사 예외 / 비검사 예외 / 추상 메서드 / 구체 메서드)  
- 자바독 @throws 태그를 사용하라  
- 검사 예외는 메서드 선언의 throws 목록에 달고, 비검사 예외는 메서드 선언 쪽에는 달지 말자  
- 발생 가능한 예외를 문서로 남기지 않으면 클래스,인터페이스를 효과적으로 사용하기 어려울 수 있다
