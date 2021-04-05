# [London vs Chicago](https://devlead.io/DevTips/LondonVsChicago)

## 선택이 아닌 통합
- TDD의 2가지 학파
    - London School
      - Outside-in, behavior-based approach
      - CQS를 촉진
      - Test Double에 크게 의존
      - 다소 테스트를 깨지기 쉽게 함
    - Chicago School 
      - Inside-out, state-based approach 
      - 높은 응집도를 촉진
      - design patterns에 중점을 둠
      - YAGNI는 위험
- 하나를 선택하는 것이 아님
    - 품질 모델을 이해하고 품질 최적화를 추진해야 함
    - 2가지 방법은 각각 장단점이 있음
    - TDD의 최선책은 이 2가지를 통합적으로 채택하는 것임
    
## London School

- 애플리케이션 외부(일반적으로 API나 컨트롤러)에서 시작하여 도메인 모델과 영속 계층 등의 하위 계층으로 향하며 작업하는 행위 기반 Formal TDD 접근법

### 장점

- 행위 중심
    - 시스템의 진입점에서 시작하여 하위 계층으로 작업함으로써 사용자가 애플리케이션을 탐색하는 방법을 정리하는데 도움이 됨
    - 이를 위해 과도한 Test Double이 요구됨
    - 이를 통해 코드베이스를 작게 유지하고, 사용되지 않는 코드(dead code)를 작성하지 않도록하는데 도움이 되지만 깨지기 쉬운 테스트를 유발하는 경향이 있음
        - 이로 인해 리팩토링이 더 어려워져 지속적인 리팩토링을 방해함
- CQS 촉진    
    - 행위에 초점을 맞추면 순수 함수를 촉진하여 side effect을 관리하는 데 도움이 됨
        - 함수형 프로그래밍으로의 관문을 제공

### 단점

- 깨지기 쉬운 테스트(Fragile Tests)
    - 테스트 주도로 행위를 도출함으로써 코드베이스를 작게 유지하고, 사용되지 않는 코드를 작성하지 않도록 하는데 도움이 되지만 깨지기 쉬운 테스트를 만드는 경향이 있음
- 어려운 리팩토링
    - 프로덕션 코드와 강하게 결합된(tightly coupled) 테스트를 사용하면 지속적인 리팩토링이 어렵고 많은 시간이 소요됨
    - 이것은 top-down TDD의 가장 큰 단점 중 하나로써 코드가 변경될 때마다 깨진 테스트를 처리해야 함
    
## Chicago School

- Chicago School(a.k.a Detroit School)은 애플리케이션의 내부(도메인 모델)에서 시작하여 외부(API)로 향하는 탐색적이고 상태에 기반한 informal 접근법

### 장점

- 강력한 안전망(Strong Safety Net)
    - 아키텍처의 저수준에서 시작했기 때문에 이전 테스트들을 기반으로 지속적으로 구축함(continuously building on prior tests)
    - 이로 인해 구현과 분리(decoupled. 객체간의 관계에 대한 지식이 test에 없는)된 테스트를 생성하는 경향이 있음
        - 기존 테스트를 깨지 않고 구현체 대한 공격적인 리팩토링을 가능케 함
    - 또한 지속적인 리팩토링에 대한 안정망을 제공하는 고도로 중복된 회귀 테스트를 제공함
- 높은 응집도
    - 테스트가 점점 구체화(generic ⇢ specific)되면서 결과로 생성되는 프로덕션 코드는 매우 높은 응집도를 갖게됨
    - As the tests get more specific, the code gets more generic
    - 이것이 높은 응집도를 촉진함
    - 높은 응집도는 느슨한 결합도를 제공함
    - 그 결과 유지보수성, 테스트 용이성, 확장성(extensibility) 등을 포함한 높은 코드 품질을 촉진함
- 테스트 대역(Test Double) 최소화
    - inside out 방식의 개발은 바로 전에 작성된 테스트 위에 개발을 하기 때문에 훨씬 적은 테스트 대역을 필요로함
    - 대부분의 경우 이전(저 수준) 테스트의 결과물을 이용하여 구현하기 때문에 mocking/stubbing할 이유가 거의 없음
    - 이를 통해 보다 안정적이고 덜 깨지는 테스트를 개발하는데 도움이 됨
    
### 단점

- YAGNI(You Are Not Gonna Need It)
    - 내부(저수준)부터 개발할 때 종종 오버엔지니어링이 촉진됨
        - 나중에 필요치 않은 코드를 작성하게 되는 경우가 종종 있음