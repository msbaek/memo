# Classic TDD or "London School"? - Software People Inspiring

원문 [Classic TDD or "London School"? - Software People Inspiring](http://codemanship.co.uk/parlezuml/blog/?postid=987)

## Classic TDD or “London School”?

classic TDD를 써야 하는가 ? “London School” TDD를 써야 하는가 ?

먼저 이 둘이 무엇을 의미하는지 살펴보자. 이 둘의 차이는 각 기법이 강조하는 [Triangulation](https://github.com/msbaek/memo/blob/master/Triangulation.md) 기법이 다르다는 것이다. 기본적으로 Classic TDD는 테스트를 추가하면서 점진적으로 알고리즘이 구체화되고, London School TDD는 점진적으로 collaborator를 찾게 된다.

## Classic TDD

Kent Beck의 [“Test-driven Development By Example"](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530)를 봤다면 classic TDD에 익숙할 것이다. 이 기법에서는 test 구체화됨에 따라 알고리즘이 generic(특정한 경우만 아니라 다양한 경우를 모두 수용함을 의미)해 지도록 TDD를 진행한다. 다시 말해 사소한 테스트부터 점진적으로 복잡한 테스트를 추가함에 따라 프로덕션 코드(알로리즘)는 모든 경우를 수용할 수 있도록 generic해 진다는 말이다.

### 예). 아라비아 숫자를 로만 숫자로 변경하는 프로그램

1. 가장 단순한 테스트로 시작. "roman numeral for 1 is I”. 이에 대한 가장 단순한 해결책은 하드 코딩된 “I"를 반환하는 것이다.
2. 그리고 생각할 수 있는 가장 단순한 실패하는 테스트로 전개한다.

`Test #1: 1 = I -> return "I”`

`Test #2: 2 = II -> if integer = 1 return “I” else return “II”`

`Test #3: 3 = III -> while(integer > 0) concatenate “I” to result and decrement integer`

이 기법의 핵심은 한번에 하나의 실패하는 테스트에 대한 해결책을 일반화하여 결과적으로 해당 시점까지의 모든 테스트를 만족시킬 수 있는 간단하고, 우아한 해결책을 얻는 것이다.

이 기법은 **알고리즘**을 강조한다.

가장 간단하고, 명확한 문제에 대해 일련의 테스트 예제를 통해 triangulate하고 해결책을 일반화(generalize)하여 해당 시점까지의 모든 테스트 케이스를 만족하는 일반적인 해결책을 만들어낸다. 객체지향 설계는 하나의 단순한 시작점, 클래스 ,책임, 상호작용에서 시작해서 성정하여 리팩토링 프로세스를 통해 구조적(organically)으로 최종적인 설계로 설정한다.

## London School of TDD

> Interaction-style TDD is also called mockist-style, or London-style after London's Extreme Tuesday club where it became popular. It is usually contrasted with Detroit-style or classic TDD which is more state-based([what-are-the-london-and-chicago-schools-of-tdd](http://softwareengineering.stackexchange.com/questions/123627/what-are-the-london-and-chicago-schools-of-tdd))

> It's born out of the furnace of component-based, distributed and service-oriented applications that are especially prevelant in the City of London (if you've got bail-out money to burn, why keep it simple, right?). The design emphasis of "enterprise applications" tends to be message passing as opposed to raw algorithmic computation([codemanship](http://codemanship.co.uk/parlezuml/blog/?postid=987))

이 방법은 알고리즘이 아니라 **역할, 책임, 상호작용**을 강조한다. 이 기법은 런던시에 만연하던 CBD, 분산/서비스 지향 등에서 나왔다. 이 방법에서 설계는 “엔터프라이즈 어플리케이션"은 순수한 알고리즘 계산이 아니라 메시지 패싱하는 경향이 있다는 것을 강조한다. 이게 객체지향 설계가 강조하는 점이기도 하다.

이 기법에 대한 정의문은 ”Growing Object Oriented Software Guided By Tests“에서 찾아 볼 수 있다. TDD에 대한 아주 간략화된 이 방식의 서술은 아래와 같다.

- 역할(Role), 책임(Responsibility)을 식별하고
- 시스템 레벨의 시나리오나 acceptance 테스트를 만족시키기 위한 해결책에 대한 end-to-end 구현에 존재하는 책임들 간의 주요한 상호작용(interaction/collaboration)을 식별한다.
- 한번에 하나씩 각 협업객체(collaborator)에 요구되는 코드를 구현한다. 이때 이 객체의 직접적인 협업객체들은 fake처리한다. front-end 테스트를 통과하는 동작하는 end-to-end 구현체를 얻을때까지 상호작용하는 객체들을 점진적으로 개발한다.

예. userName, password, email을 입력하여 가입하는 웹 어플리케이션. email 주소가 잘 못 되었을 때를 다루는 acceptance 테스트는 아래와 같다.

`User Sign Up Page -> Sign-up Controller.openUserAccount(userName, password, email) -> Email Validator.validateEmail(email)`

- 제대로 동작하지 않는 email 주소를 인자로 openUserAccount()를 호출할 것이라고 기대되는 mock Sign-up Controller를 가지고 User Sign-up Page에 대한 테스트를 만듦으로써 시작한다. 이때 controller는 결과로 User Sign-Up Page에 에러 메시지를 출력하도록 기술된다.
- 위 테스트를 통과하면 email 주소가 openUserAccount에 전달되면 validateEmail() 메소드 호출되는 것을 기대하는 mock Email Validator를 가지고 Sign-up Controller.openUserAccount()에 대한 테스트를 작성한다. 잘못된 email 주소가 전달되면 Email Validator는 false를 리턴하고 controller는 false를 반환하게 된다.
- 그리고 나서, 잘못된 email 주소를 가지고 validateEmail() 메소드를 호출하면 false를 반환하는 Email Validator에 대한 테스트를 작성할 수 있다.

분산 엔터프라이즈 어플리케이션을 작성하는 사람들은 종종 London School은 그들의 업무에 가장 적합하다고 한다.

그런데 위 예제에서 잘못된 email 주소를 갖는 Email Validator에 대한 테스트에 도달했을 때 정말로 간단하고 명확한 구현을 갖는 하나의 테스트만 존재하는가 ?

이경우 종종 TDD가 혼란스러워진다. 너무 자주 개발자들은 하나의 테스트 케이스로 크게 나아가려 한다.

아래와 같은 잘못된 email 주소를 고려해 보자:

```java
duffrubbish.com

duff@rubbish

@duffrubbish.com

?duff@rubbish.com
```

모두 잘못된 email 주소들이다. 하지만 서로 다른 잘못된 원인을 갖는다. 잘못된 이라는 말은 다수의 원인을 커버한다.

정규식을 이용해서 하나의 테스트로 이 모든 경우를 해소하려는 유혹이 생긴다. 이럴 경우 우리의 테스트는 하나 이상의 일을 하는 것이고 절대 그러면 안된다.

또한 각 규칙에 대해 항상 명확한 구현이 존재하는 것은 아니다. 가능한 최대한 작업 작업으로 문제를 해결하는 것이 통하는 방법이다. 다시 말해 한번에 하나의 규칙씩 작업하는 것, 덜 명확한 규칙을 위해서는 해결책에 triangulating하는 것을 의미한다.

경험상으로 보면 상호작용에만 집중하면 알고리즘상으로는 신뢰하기 어려운 코드가 나온다(버그가 포함된). 또 알고리즘에만 집중하면 rigid/brittle한 end-to-end 설계가 되고, 가끔은 시스템 레벨 요구사항을 만족시키지 못한다.

우리의 목표는 신뢰할 만한 코드(버그가 없는)이면서 내부적으로 좋은 설계를 갖는 코드를 만드는 것이고, 이를 위해서는 적절한 시기에 2가지 TDD 기법이 모두 필요하다. 알고리즘에 집중할 때는 Classic School이, 상호작용에 집중할 때는 London School이…

상당한 복잡성을 가진 소프트웨어를 구현하는 로직은 알고리즘만으로도 안되고, 상호작용만으로도 안된다.

따라서 Classic TDD와 London School TDD은 동일하게 중요하다.