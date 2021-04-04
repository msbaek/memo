# [Clean Code Talks - Unit Testing](https://testing.googleblog.com/2008/11/clean-code-talks-unit-testing.html)

> “There is no secret to writing tests…… there are only secrets to writing testable code!”

- 테스트 작성에 대해서는 비법이 없다.
- 테스트 가능한 코드를 작성하는 비법만이 존재한다.
    - Good OO
    - Dependency Injection(Inversion)
    - Test Driven Development
    
## Progression of Testing

### Stage 1: Scenario Tests

> “Test the whole app as unit by pretending to be a user”

- 사용자처럼 대상 어플리케이션을 하나의 테스트 대상 단위로 처리하는 프레임워크를 만드는 방법 – “Scenario/Large Tests”
- 테스트가 느리다. 그래서 개발자들이 수행하지 않는다.
- 테스트가 Flaky하다.
	- Because you test the real app with real external dependencies sometimes things take a bit longer than expected and you get false negatives.

### Stage 2: Functional Tests

> “Functional/Medium Tests”

- 애플리케이션을 서브시스템으로 분해하고 외부 의존성을 simulator로 치환해서 각 서브시스템을 테스트
- 시나리오 테스트에서는 불가한 조건을 시뮬레이트할 수 있음
- 개발자들은 이 테스트를 코드를 제출(commit, PR, merge 등) 전에 실행해 볼 수 있음

### Stage 3: Unit Tests

> “Unit/Small Tests”

- 작은 영역의 코드를 테스트하는 것은 많은 혜택을 주므로 테스트 단위를 더 작은 단위로 나눌 수 있음
- 각 클래스를 격리하여 테스트
- 모든 오류 조건들을 시뮬레이트할 수 있음
- 개발자들은 파일을 수정할 때마다 단위 테스트를 실행할 수 있음
- 빨리 빠르고 테스트가 깨질 확률이 적음 


| Test Type	| Definition | Issues |
|-----------|------------|--------|
| Scenario / Large Tests | Test whole application by pretending to be a user | Slow / Flaky / Mostly happy paths |
| Functional / Medium Tests | External dependencies simulated | Test class interaction |
| Unit / Small Tests | Focus on application logic | Very Fast / No I/O / No need for a debugger |

## Different Kinds of Tests

![](https://api.monosnap.com/rpc/file/download?id=aNlffkPCHWKHrrOuUb337refU7Ll8O)

## Unit Testing a Class

![](https://api.monosnap.com/rpc/file/download?id=sjYYFLnihovlFwA3E38NKoqkDegU89)

![](https://api.monosnap.com/rpc/file/download?id=zmk1HzllmChUI5s6t7z3QuBF7tOTiu)

![](https://api.monosnap.com/rpc/file/download?id=5ORrns6Yp7mbPCUar0U8AGUjMxndBD)

- 테스트 코드 작성이 어려워진 이유
	- 객체 생성 코드와 비지니스 로직을 혼합해서…
	- 이로 인해 테스트는 운영환경과 다른 객체 그래프를 만들 수 없게 되고, 고립된(isolated) 상태에서 테스트 할 수 없게 되었다.
    
![](https://api.monosnap.com/rpc/file/download?id=t4Xu9A3UU2Bvl4xsyebOcMPItOJeeO)

![](https://api.monosnap.com/rpc/file/download?id=7Ipy2dEnucCZ3sNLgZLsvmWAWzJovc)

## Take Away

- Unit Tests are a preferred way of testing
- Unit Tests require separation of Instantiation of Object Graph from Business Logic under test