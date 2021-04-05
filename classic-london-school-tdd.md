# Classic TDD or "London School"? - Software People Inspiring

원문 [Classic TDD or "London School"? - Software People Inspiring](http://codemanship.co.uk/parlezuml/blog/?postid=987)

## Classic TDD or “London School”?

- classic TDD를 써야 하는가 ? “London School” TDD를 써야 하는가 ?
- classic TDD 기법을 구분되게 만드는 특징은 [Triangulation](https://github.com/msbaek/memo/blob/master/Triangulation.md) 기법을 강조한다는 점임
  - Triangulation에 대해서 [Clean Coders Episode 22에서 설명된 내용](https://github.com/msbaek/memo/blob/HEAD/CC-E22-Test-Processes.md#4-triangulation) 은 아래와 같다
    - 토목/수학에서 말하는 삼각법(물체간의 거리를 측정하는)이 아니라 TDD에서 삼각법은 generalization을 만드는 방법의 하나를 의미
    - "As the tests get more specific, the code gets more GENERIC”
    - 하나의 테스트가 아니라 여러개의 테스트를 추가함으로써 문제와 해결책을 좀 더 명확히 하는 기법 
    - 삼각법이 2개 이상의 지점의 위치를 이용하여 현 위치를 측정하는 것 처럼... 
    - 하나의 테스트만 존재할 때는 fake할 수 있음(상수를 반환함으로써)
    - 하지만 상수로 처리 불가한 테스트를 추가하면(삼각법에서 2개 이상의 지점을 사용하는 것처럼) fake할 수 없게 되어 code가 generic해 짐
- 기본적으로 Classic TDD는 테스트를 추가하면서 점진적으로 알고리즘이 구체화되고, London School TDD는 점진적으로 collaborator를 찾게 된다.

## Classic TDD(Inside out)

- Kent Beck의 [“Test-driven Development By Example"](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530) 를 봤다면 classic TDD에 익숙할 것 
- 이 기법에서는 test가 구체화됨에 따라 알고리즘이 generic(특정한 경우만 아니라 다양한 경우를 모두 수용함을 의미)해 지도록 TDD를 진행
- 다시 말해 사소한 테스트부터 점진적으로 복잡한 테스트를 추가함에 따라 프로덕션 코드(알로리즘)는 모든 경우를 수용할 수 있도록 generic해 진다는 말임

### 예). 아라비아 숫자를 로만 숫자로 변경하는 프로그램
- 가장 단순한 테스트로 시작. "roman numeral for 1 is I”. 이에 대한 가장 단순한 해결책은 하드 코딩된 “I"를 반환하는 것
- 그 다음으로 생각할 수 있는 가장 단순한 실패하는 테스트로 전개

`Test #1: 1 = I -> return "I”`

`Test #2: 2 = II -> if integer = 1 return “I” else return “II”`

`Test #3: 3 = III -> while(integer > 0) concatenate “I” to result and decrement integer`

- 이 기법의 핵심은 한번에 하나의 실패하는 테스트에 대한 해결책을 일반화하여 결과적으로 해당 시점까지의 모든 테스트를 만족시킬 수 있는 간단하고, 우아한 해결책을 얻는 것임
- 이 기법은 **알고리즘**을 강조한다.
- 가장 간단하고, 명확한 문제에 대해 일련의 테스트 예제를 통해 triangulate하고 해결책을 일반화(generalize)하여 해당 시점까지의 모든 테스트 케이스를 만족하는 일반적인 해결책을 만들어냄
- 객체지향 설계는 하나의 단순한 시작점, 클래스 ,책임, 상호작용에서 시작, 성장, 리팩토링을 통해 구조적(organically)으로 최종적인 설계로 성장함

## London School of TDD(Outside in)

> Interaction-style TDD is also called mockist-style, or London-style after London's Extreme Tuesday club where it became popular. It is usually contrasted with Detroit-style or classic TDD which is more state-based([what-are-the-london-and-chicago-schools-of-tdd](http://softwareengineering.stackexchange.com/questions/123627/what-are-the-london-and-chicago-schools-of-tdd))

> It's born out of the furnace of component-based, distributed and service-oriented applications that are especially prevelant in the City of London (if you've got bail-out money to burn, why keep it simple, right?). The design emphasis of "enterprise applications" tends to be message passing as opposed to raw algorithmic computation([codemanship](http://codemanship.co.uk/parlezuml/blog/?postid=987))

- 이 방법은 알고리즘이 아니라 **역할, 책임, 상호작용**을 강조
- 이 기법은 런던시에 만연하던 CBD, 분산/서비스 지향 등에서 나왔음 
- 이 방법에서 설계는 “엔터프라이즈 어플리케이션"은 순수한 알고리즘 계산이 아니라 메시지 패싱하는 경향이 있다는 것을 강조
- 이게 객체지향 설계가 강조하는 점이기도 함
- 이 기법에 대한 정의문은 [Growing Object Oriented Software Guided By Tests](https://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627) 에서 찾아 볼 수 있음
- TDD에 대한 아주 간략화된 이 방식의 서술은 아래와 같음
  - 역할(Role), 책임(Responsibility)을 식별하고
  - 시스템 레벨의 시나리오나 acceptance 테스트를 만족시키기 위한 해결책에 대한 end-to-end 구현에 존재하는 책임들 간의 주요한 상호작용(interaction/collaboration)을 식별
  - 한번에 하나씩 각 협업객체(collaborator)에 요구되는 코드를 구현
    - 이때 이 객체의 직접적인 협업객체들은 fake처리
    - front-end 테스트를 통과하는 동작하는 end-to-end 구현체를 얻을때까지 상호작용하는 객체들을 점진적으로 개발
  
### 예. userName, password, email을 입력하여 가입하는 웹 어플리케이션
- email 주소가 잘 못 되었을 때를 다루는 acceptance 테스트는 아래와 같음

`User Sign Up Page -> Sign-up Controller.openUserAccount(userName, password, email) -> Email Validator.validateEmail(email)`

- 제대로 동작하지 않는 email 주소를 인자로 openUserAccount()를 호출할 것이라고 기대되는 mock Sign-up Controller를 가지고 User Sign-up Page에 대한 테스트를 만듦으로써 시작
- 이때 controller는 결과로 User Sign-Up Page에 에러 메시지를 출력하도록 기술됨
- 위 테스트를 통과하면 email 주소가 openUserAccount에 전달되면 validateEmail() 메소드 호출되는 것을 기대하는 mock Email Validator를 가지고 Sign-up Controller.openUserAccount()에 대한 테스트를 작성
  - 잘못된 email 주소가 전달되면 Email Validator는 false를 리턴하고 controller는 false를 반환하게 됨
- 그리고 나서, 잘못된 email 주소를 가지고 validateEmail() 메소드를 호출하면 false를 반환하는 Email Validator에 대한 테스트를 작성

- 분산 엔터프라이즈 어플리케이션을 작성하는 사람들은 종종 London School은 그들의 업무에 가장 적합하다고 함
- 그런데 위 예제에서 Email Validator는 아래와 같은 duff의 이메일 주소에 대한 테스트 케이스에 대해 간단하고, 명확한 구현을 갖는 하나의 테스트인가 ?
  - 이경우 종종 TDD가 혼란스러워짐. 너무 자주 개발자들은 하나의 테스트 케이스로 크게 나아가려 한다.
- 아래와 같은 duff의 email 주소를 고려해 보자:

```java
duffrubbish.com
duff@rubbish
@duffrubbish.com
?duff@rubbish.com
```

- 모두 잘못된 email 주소들이다. 하지만 서로 다른 잘못된 원인을 가짐. 잘못된 이라는 말은 다수의 원인을 커버함
- 정규식을 이용해서 하나의 테스트로 이 모든 경우를 해소하려는 유혹이 생김
  - 이럴 경우 우리의 테스트는 하나 이상의 일을 하는 것이고 절대 그러면 안됨
- 또한 각 규칙에 대해 항상 명확한 구현이 존재하는 것은 아님 
- 가능한 최대한 작은 작업으로 문제를 해결하는 것이 통하는 방법. 다시 말해 한번에 하나의 규칙씩 작업하는 것, 덜 명확한 규칙을 위해서는 해결책에 triangulating하는 것을 의미
- 경험상으로 보면 
  - 상호작용에만 집중하면 알고리즘상으로는 신뢰하기 어려운 코드가 나옴(버그가 포함된)
  - 알고리즘에만 집중하면 rigid/brittle한 end-to-end 설계가 되고, 가끔은 시스템 레벨 요구사항을 만족시키지 못함
-  우리의 목표는 신뢰할 만한 코드(버그가 없는)면서 내부적으로 좋은 설계를 갖는 코드를 만드는 것
   - 이를 위해서는 적절한 시기에 2가지 TDD 기법이 모두 필요함
   - 알고리즘에 집중할 때는 Classic School이, 상호작용에 집중할 때는 London School
- 상당한 복잡성을 가진 소프트웨어를 구현하는 로직은 알고리즘만으로도 안되고, 상호작용만으로도 안됨
- 따라서 Classic TDD와 London School TDD은 동일하게 중요함