# Refactoring
<!-- TOC -->
* [IntelliJ 리팩터링 툴을 활용한 리팩터링](#intellij-리팩터링-툴을-활용한-리팩터링)
* [예제들](#예제들)
* [기법들](#기법들)
  * [Composed Method, SLAP, Step Down Rule](#composed-method-slap-step-down-rule)
  * [Invert If: Guard Clause, Early Return](#invert-if--guard-clause-early-return)
<!-- TOC -->

## IntelliJ 리팩터링 툴을 활용한 리팩터링
- https://github.com/msbaek/refactoring-tools
- Intellij의 Refactoring 메뉴에 있는 모든 Refactoring 기능들을 jetbrains 사이트의 예제 및 일부 보완한 예제를 통해 리팩토링시 주의 사항이나 전후를 비교
- com.example.refactorings.RefactoringTest 의 각 테스트를 따라서 보면서 화면과 같이 변화가 일어나도록 진행하면 됨

## 예제들
- fitness: https://github.com/msbaek/fitness-example
- print-rime: https://github.com/msbaek/print-prime 이 예제는 복잡한 함수를 2개의 클래스로 Extract Method Object하는 과정
- expenses: https://github.com/msbaek/expense
- videostore: https://github.com/msbaek/videostore
- extract till you drop: https://github.com/msbaek/refactoring-cases 이 예제는 extract till you drop(extract method를 수행할 수 없을 때까지 수행)을 한 후에 남게되는 많은 작은 private method들을 역할에 맞게 다른 클래스로 위치시키는 것을 보여줌
- OCP: https://github.com/msbaek/practical-refactoring-cases 기존 로직과 유사한 로직이 추가되어야 할 때 기존 코드를 수정하지 않고 새로운 코드를 추가하는 방법을 취하여 안정적으로 운영할 수 있는 기법에 대해서 알아본다.
- [kt4u_examples.md](kt4u_examples.md)
---
## 기법들
### [Composed Method, SLAP, Step Down Rule](composed_method.md)
### [Invert If: Guard Clause, Early Return](invert-if.md)
### [Functional Core & Imperative Shell]()
- application(controller, service 등)에는 비즈니스 로직이 없어야 함
- 비즈니스 로직은 domain에 있어야 함
  - entity, value object, aggregate
  - 아니면 domain service
- application은
  - 빵: 비즈니스 로직 수행을 위해 필요한 작업(데이터 읽기 등)
  - 속: 비즈니스 로직 수행
  - 빵: 부가작용(side effect) 수행
  - application에서 큰 메소드를 작은 메소드로 추출
    - 이 작은 메소드들이 BL이라면 Domain으로 이동
### [Naming](naming.md)
### Monster Method 공력하기
- Prepare `Extract Method` → `Extract Method` → `Move Instance Method`
### Prepare Extract Method
#### [Replace Temp With Query]
#### Move Return closer to computation
### [Valuable Value Object](valuable_value_object.md)