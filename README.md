# Effective Java 3/E 스터디

[책] - [Effective Java 3/E](https://product.kyobobook.co.kr/detail/S000001033066)

> 해당 Repository는 스터디를 진행하면서 알게된 지식을 공유 및 정리하고,<br>
> 이해 안되는 부분을 질문하고 해결하는 공간입니다.

<br>

> 강의 내용에 대한 실습은 각자의 Repository에서 진행하며 관련 코드를<br>
> 공유하고 싶은 경우 markdown 문법을 활용 혹은 개인 Repository를 링크하는 식으로 진행합니다.

## 목표

- Java에 대해 더 깊이있게 이해한다.
- 각 아이템에 대해 이해하고, 실제로 사용해보며 익힌다.
- 서로의 지식을 공유하고, 함께 성장한다.

## 스터디 참가자

> 인원: 8명

<center>

|![다나](https://avatars.githubusercontent.com/u/85955988?v=4)|![럭키](https://avatars.githubusercontent.com/u/110045522?v=4)|![루카](https://avatars.githubusercontent.com/u/98688494?v=4)|![루키](https://avatars.githubusercontent.com/u/74547868?v=4)|![밀리](https://avatars.githubusercontent.com/u/87763333?v=4)|![피터](https://avatars.githubusercontent.com/u/97747863?v=4)|![워니](https://avatars.githubusercontent.com/u/116738827?v=4)|![캐슬](https://avatars.githubusercontent.com/u/62132755?v=4)|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|[다나](https://github.com/joowojr)|[럭키](https://github.com/Hyunstone)|[루카](https://github.com/luke0408)|[루키](https://github.com/destiny3912)|[밀리](https://github.com/hw130)|[피터](https://github.com/wcorn)|[워니](https://github.com/kiwijomn)|[캐슬](https://github.com/hosung-222)|

</center>

## 스터디 개요

- 기간: 2023.09.20 ~ 
- 장소: 디스코드
- 스터디 계획
  - 1주차: 09.20(수)
    - Item 01. 생성자 대신 정적 팩터리 메서드를 고려하라
    - Item 02. 생성자에 매개변수가 많다면 빌더를 고려하라
    - Item 03. private 생성자나 열거 타입으로 싱글턴임을 보증하라
    - Item 04. 인스턴스화를 막으려거든 private 생성자를 사용하라
    - Item 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
    - Item 06. 불필요한 객체 생성을 피하라
    - Item 07. 다 쓴 객체 참조를 해제하라
  - 2주차: 09.25(월)
    - Item 08. finalizer와 cleaner 사용을 피하라
    - Item 09. try-finally보다는 try-with-resources를 사용하라
    - Item 10. equals는 일반 규약을 지켜 재정의하라
    - Item 11. equals를 재정의하려거든 hasCode도 재정의하라
    - Item 12. toString을 항상 재정의하라
    - Item 13. clone 재정의는 주의해서 진행하라
    - Item 14. Comparable을 구현할지 고려하라
  - 3주차: 10.02(월)
    - Item 15. 클래스와 멤버의 접근 권한을 최소화하라
    - Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
    - Item 17. 변경 가능성을 최소화하라
    - Item 18. 상속보다는 컴포지션을 사용하라
    - Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
    - Item 20. 추상 클래스보다는 인터페이스를 우선하라
    - Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라
  - 4주차: 10.30(월)
    - Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라 
    - Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라 
    - Item 24. 멤버 클래스는 되도록 static으로 만들라 
    - Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라
    - Item 26. 로 타입은 사용하지 말라
    - Item 27. 비 검사 경고를 제거하라 
    - Item 28. 배열보다는 리스트를 사용하라 
  - 5주차: 11.06(월)
    - Item 29. 이왕이면 제네릭 타입으로 만들라
    - Item 30. 이왕이면 제네릭 메서드로 만들라
    - Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라
    - Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중해라
    - Item 33. 타입 안전 이종 컨테이너를 고려하라
    - Item 34. int 상수 대신 열거 타입을 사용하라
    - Item 35. ordinal 매서드 대신 인스턴스 필드를 사용하라
    - Item 36. 비트 필드 대신 EnumSet을 사용하라
  - 6주차: 11.13(월)
    - Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라
    - Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
    - Item 39. 명명 패턴보다 애너테이션을 사용하라
    - Item 40. @Override 애너테이션을 일관되게 사용하라
    - Item 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라
    - Item 42. 익명 클래스보다는 람다를 사용하라
    - Item 43. 람다보다는 메서드 참조를 사용하라
    - Item 44. 표준 함수형 인터페이스를 사용하라
  - 7주차: 11.20(월)
    - Item 45. 스트림은 주의해서 사용하라
    - Item 46. 스트림에서는 부작용 없는 함수를 사용하라
    - Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다
    - Item 48. 스트림 병렬화는 주의해서 적용하라
    - Item 49. 매개변수가 유효한지 검사하라
    - Item 50. 적시에 방어적 복사본을 만들라
    - Item 51. 메서드 시그니처를 신중히 설계하라
    - Item 52. 다중정의는 신중히 사용하라
  - 8주차: 11.27(월)
    - Item 53. 가변인수는 신중히 사용하라
    - Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라
    - Item 55. 옵셔널 반환은 신중히 하라
    - Item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라
    - Item 57. 지역변수의 범위를 최소화하라
    - Item 58. 전통적인 for 문보다는 for-each 문을 사용하라
    - Item 59. 라이브러리를 익히고 사용하라
    - Item 60. 정확한 답이 필요하다면 float와 double은 피하라

## 스터디 방식

> 기본적으로 자기 담당이 아니더라도 모든 강의를 듣고 스터디에 참여한 다는 것을 전제로 한다.

1. 스터디 방식은 매주 미팅을 통해 조금씩 개선해 나간다.
    - 매주 미팅때 아쉬운 부분 혹은 개인적인 이슈 등을 나눌 예정
2. 매주 각 챕터의 담당자를 정한다.
    - 담당자란 챕터의 내용을 정리해 발표하는 사람을 말한다.
    - 담당자는 Issue를 통해 들어온 질문을 해결하기 위해 최선을 다한다.
3. 매주 각 세션에 대한 Issue 생성 및 관리
    - Issue 생성 시, 순서에 맞게 생성한다.
    - Issue 생성 시, 탬플릿의 규칙을 지킨다.
    - 각 Issue는 해당 세션에 대한 소통을 하는 장소이다.
    - 팀원 들은 각 Issue를 통해 해당 세션에 대한 궁금증 및 질문을 공유한다.
    - 만약 담당자가 답변을 못하는 경우 미팅 시간에 해당 질문을 공유한다.
      - 너무 완벽하게 해결할 필요 [X]
      - 미팅을 통해 해결한 경우 Issue에 해결한 내용을 공유한다.
    - 담당자가 아니더라도 해당 질문에 답변할 수 있다.
4. 정리시에 외부 자료를 참고한 경우 참고 자료 명시를 확실하게 한다.

## 스터디 규칙 (필요시 작성)

- 깃 컨벤션
  - 커밋 메시지 규칙
  - 브랜치 규칙
  - 이슈 규칙
  - markdown 작성 규칙
- 스터디 수칙
  - 스터디 불참, 지각 등 참여도 규칙
