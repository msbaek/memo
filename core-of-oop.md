## 객체지향의 정수/핵심 ?

객체지향이 뭐냐고 말하면 사람들이 많이 말하것으로

- 재사용
- 실세계에 대한 객체 모델링
- Encapsulation
- Inheritance
- Abstraction
- Polymorphism

등을 이야기한다. 근데 이런 것들은 그저 **객체지향의 Mechanism일 뿐 정수/핵심은 아니다**.

객체지향의 정수/핵심은 **“의존성 관리"**이다. 이를 통해

- 변경 영향 최소화
- 독립적인 배포(전체가 아닌 변경 내역만)
- 독립적인 개발(상호 영향을 없애서 각자 동시 개발할 수 있는)

이렇게 보면 객체지향에서 가장 중요한 것은 DIP(Dependency Inversion Principle)를 통한 High Level Policy와 Low Level Details의 분리인 것이다.
