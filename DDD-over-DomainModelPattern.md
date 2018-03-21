# Business Logic Organization Patterns

## 1. Transaction Script Pattern

객체지향을 좋아하더라도 개발할 비즈니스 로직이 너무 단순한 경우 객체지향 접근은  과할 수 있다. 이런 경우 **절차적인 코드(procedural code)**를 작성한다.

마틴파울러는 이를 [Transaction Script Pattern](http://https://martinfowler.com/eaaCatalog/transactionScript.html)이라고 불렀다.

이 방법에서는 객체지향 설계를 하는 것이 아니라, presentation tier의 요청을 처리하기 위해 transaction script(OrderService의 createOrder, reviseOrder, cancelOrder 등)라고 불리는 메소드를 작성한다.

**이 접근법의 중요한 특성은 아래 그림과 같이 행위를 구현하는 클래스(OrderService)가 상태를 저장하는 클래스(Order)와 분리된다는 것이다.**

![](https://api.monosnap.com/rpc/file/download?id=RpdvAN360qedlxqEa2xUnRYmDFHkvV)

스크립트는 보통 Service 클래스(OrderService)에 위치한다. Service는 하나의 요청과 해당 요청에 대한 시스템 행위에 대해서 하나의 메소드를 갖는다. 이 메소드가 해당 요청에 대한 비즈니스 로직을 구현한다. 데이터 객체(Order)는 행위가 없거나 거의 갖지 않는다.

이 접근법은 간단한 비즈니스 로직에는 잘 동작한다. 단점은 복잡한 비즈니스 로직을 구현하기에는 좋은 방법이 아니라는 것이다.

## 2. Domain Model Pattern

Transaction Script Pattern이 단순하여 매우 매력적이지만 **비즈니스 로직이 복잡해지면 유지보수가 악몽이 된다**.

> 오랜 기간 서비스를 운영해 본 개발자라면 변경 요구는 대부분 기능 자체가 아니라 데이터의 변경에 기인한다는 점에 공감할 것이다.
> Transaction Script Pattern에서는 데이터를 가지고 있는 클래스가 행위를 가지고 있지 않아서 데이터의 변경이 생기면 행위를 가지고 있는 클래스에 영향을 미친다. 심지어 행위를 가진 코드가 서비스 클래스 외에 여러곳에 존재한다면 문제는 더 심각해진다.
> 따라서 **운영과 유지보수를 생각한다면 데이터를 캡슐화하고 기능만 외부에 제공하는 방식이여야 데이터의 변경이 생겨도 외부의 영향을 최소화할 수 있다**.

Monolithic Application이 지속적으로 커지는 습성이 있는 것과 같이 Transaction Script도 동일한 문제를 갖는다.

결론적으로 극도로 단순한 어플리케이션을 작성하는 것이 아니라면 절차적 코드를 작성하는 Transaction Script의 유혹을 이겨내고 [Domain Model Pattern](https://martinfowler.com/eaaCatalog/domainModel.html)을 적용하여 객체지향 설계를 개발해야 한다.

객체지향설계에서 비즈니스 로직은 객체모델로 구성된다. 객체모델은 적은 수의 클래스들의 네트워크로 구성된다. 이런 설계 방식에서는 아래 그림과 같이 행위나 상태만 갖는 클래스들이 존재할 수는 있지만 대부분의 경우는 행위와 상태를 모두 갖는다.

![](https://api.monosnap.com/rpc/file/download?id=fRrRkwS4OVv0O3F0mWWPteHAW1ZzrS)

Transaction Script와 동일하게 OrderService는 각 요청과 해당 요청에 대한 시스템 행위에 대해 하나의 메소드를 갖는다. Domain Model Pattern에서 Service 메소드는 대개 단순한다. Service 메소드가 항상 도메인 객체에게 위임하기 때문이다. 도메인 객체는 비즈니스 로직의 대부분을 갖는다.

예를 들면 Service 메소드는 데이터베이스로부터 도메인 객체들을 로딩하고 도메인 객체들의 메소드를 호출한다. 

이 예의 경우 Order 클래스는 상태와 행위를 갖는다. 심지어 상태는 private이여서 메소드를 통해서만 간접적으로 접근 가능하다.

객체지향설계 방식의 장점

- 설계가 이해, 유지보수하기 쉽다.
	- 모든 일을 하는 하나의 큰 클래스 대신 적은 갯수의 책임을 갖는 다수의 작은 클래스들로 이뤄진다.
- 테스트하기 쉽다.
	- 각 클래스들은 독립적으로 테스트 가능하고 독립적으로 테스트되어야 한다.
- 확장하기 쉽다.
	- strategy pattern, template pattern 등과 같이 실제 코드를 수정하지 않고 컴포넌트를 확장할 수 있는 잘 알려진 디자인 패턴을 사용할 수 있다.

Domain Model Pattern은 잘 동작하지만 몇가지 문제를 갖는다. 특히 MSA에서...

이러한 문제를 언급하기 위해 DDD로 알려진 OOD의 정제(refinement)를 사용해야만 한다.

## 3. DDD

### 3.1 DDD 소개

DDD는 OOD의 정제로써 복잡한 어플리케이션의 비즈니스 로직을 개발하는 접근법이다. DDD의 Subdomain은 어플리케이션을 서비스들로 분해하는데 유용한 개념이다. DDD에서 각 서비스는 자신만의 도메인 모델을 갖는다. 이로 인해 어플리케이션 전반에 걸쳐 단일 도메인 모델을 가짐으로써 발생하는 문제를 제거한다.
Subdomain과 관련된 개념인 Bounded Context는 DDD의 2개의 Strategic Pattern이다.

DDD는 도메인 모델의 빌딩블록(building block)인 Tactical Pattern도 갖는다.

- Entity
- Value Object
- Factory
- Repository
- Service

등이 존재한다.

### 3.2 DDD Aggregate

food2go라는 음식 배달 어플리케이션을 예로 들면 아래와 같은 모델로 표현할 수 있다.

![](https://api.monosnap.com/rpc/file/download?id=vHz19DjiKX5VsjyoUCRY0wceBfgJi5)

일반적인 도메인 모델에서 각 업무 객체(business object)의 명시적 경계(explicit boundary)가 없고, 경계가 없는 것은 가끔 문제를 일으킨다. 특히 MSA에서...

#### 3.2.1 명확하지 않은 경계의 문제점

Order를 load하는 오퍼레이션을 구현하고자 한다. 이게 의미하는 것이 정확히 무엇일까 ? Order 객체를 load할 수도 있다. 하지만 현실은 Order를 load하는 것은 단순히 Order 자체만을 load하는 것으로는 부족하다. OrderLineItem, PaymentInfo, DeliveryInfo 등도 존재한다. 도메인 객체의 경계에 대한 정의가 개발자의 직감에 남겨졌다.
명시적 경계의 부재는 개념적 모호함 외에도 비즈니스 객체를 갱신할 때도 문제를 일으킨다. 일반적으로 비즈니스 객체는 항상 지켜야만 하는 업무 규칙인 불변률(invariants)을 갖는다. 예. 최저 주문액.