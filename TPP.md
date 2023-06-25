# Transformation

- transformation은 refactoring과 대응 관계(counterpart)에 있다.
  - refactoring이 행위의 변경 없이 코드의 구조를 변경하기 위해 소스 코드에 작은 변경을 한다면,
  - transformation은 반대로 구조의 변경 없이 행위를 변경하기 위해 소스 코드에 작은 변경을 가하는 것이다.
- 좋은 알고리즘을 얻기 위한 우선 순위 존재
  - transformation의 우선 순위를 잘 적용하면 보다 나은 알고리즘을 얻을 수 있다는 것을 보여 줌
  - 중요한 구현을 미루는 것이 좋은 알고리즘으로 안내함(ex. guard clause)

## **transformation이 TDD의 RGB 중 어디에 적합할까 ?**

- 실패하는 테스트를 성공시키기 위해 사용될 수 있다. 즉 Red, Transform to Green, Refactor...
- 테스트가 실패하면 구조는 유지한채 성공시키기 위해 행위를 변경하다. - transformation
- 테스트가 성공하면 행위를 유지한채 구조를 개선한다. - refactoring

- **실패하는 테스트를 성공시키는데 적용되는 규칙**

  > As the tests get more specific, the code gets more generic

- 그럼 transformation은 어떤 방향으로 코드를 이동시키나 ?
  - transformation은 테스트를 성공시키는데 사용됨
  - transformation는 코드를 보다 general하게 함
  - transformation은 generalizer임
    > Transformation은 구조에 영향을 미치지 않은채 행위를 일반화(generalize)하는 코드에 대한 작은 변경이다.
    > transformation은 구조 변경없이 행위를 일반화한다.```

# Rule

- `{} → Nil`
  - 아무 코드가 없는 상태에서 시작함
    - 가장 퇴화한 테스트를 작성
    - 컴파일이 되지만 실패해야 함
    - SUT에서 null을 반환
- `Nil → Constant`
  - 현재 테스트를 통과시키도록 상수를 반환
- `Constant → Variable`
  - Triangulate를 하면 Constant로 테스트를 통과시킬 수 없게 됨
  - 이러한 변환들이 계속 매우 구체적인 상태에서 살짝 더 일반적인 상태로 옮김
  - 이런 변환은 모두 일반화, 즉 코드가 이전보다 더 다양한 제약 조건을 처리할 수 있도록 하는 방법임
- `Unconditional → Selection`
  - 조건문(if) 추가
- `Value → List`
  - one to many
- `Selection → Iteration`
  - if → while
- `Statement → Recursion`
  - 반복문 대신 명령문을 재귀 명령문으로 변경
  - 재귀 외에는 별도 반복 기능이 없는 경우에 특히 자주 볼 수 있음
- `Value → Mutated Value`
  - 변수의 값을 변경
    - 반복문에서 부분값들을 모으거나 점진적인 계산을 위해 사용됨

# 비고

- 둘 이상의 변환을 조합해서 테스트를 통과시키고 싶은 생각이 든다면 ?
  - 하나 이상의 테스트를 빠트린 것을 수 있음
  - 하나의 변환만 사용해서 통과시킬 수 있는 테스트를 찾아보라
  - 선택을 해야 한다면 우선순위가 높은 변환을 취하라
- 순서대로 변환을 적용하다 보면 함수형 프로그래밍 스타일을 사용하는 해법을 구현하게 됨
