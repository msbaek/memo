# DAO와 REPOSITORY 논쟁

[원문](http://aeternum.egloos.com/1160846)

## J2EE 권장 기본 아키텍처

- 도메인 레이어: EJB Session Bean
- 퍼시스턴스 레이어: EJB Entity Bean
    - Entity Bean
        - CMP(Container-Managed Persistence): 매핑 제약
        - BMP(Bean-Managed Persistence): 비유용성, 오버헤드, 빈약한 EJB QL 지원, 침투적인 인프라스트럭쳐 코드, 복잡도 등
- 의 단점으로 인해 개발자들로부터 외면을 받고 말았다.
- Entity Bean이 사라질 무렵 선언적으로 트랜잭션을 정의할 수 있는 CMT(Container-Managed Transaction) 기반의 Session Bean은 다양한 어플리케이션에서 폭 넓게 사용되었다.

## Entity Bean 없이 Session Bean만을 사용하면서 문제 발생

- 비즈니스 로직만을 담아야할 Session Bean 내부로 퍼시스턴스 로직이 침투
- Session Bean과 Entity Bean에 의해 자연스럽게 구분되던 도메인 레이어와 퍼시스턴스 레이어 간의 경계가 모호해지면서 Session Bean 내에 비즈니스 로직과 퍼시스턴스 로직이 섞여 버리는 경우가 많았다.
- 이를 해결하기 위해 퍼시스턴스 저장소로 접근하는 모든 로직을 캡슐화하는 별도의 객체를 추가하여 도메인 레이어와 퍼시스턴스 레이어를 명확하게 분리하는 방법이 제시
- Entity Bean의 퍼시스턴스 지원 기능을 대체하는 동시에 비즈니스 로직과 퍼시스턴스 로직의 명확한 분리를 위해 퍼시스턴스 로직을 캡슐화하고 도메인 레이어에 객체 지향적인 인터페이스를 제공하는 객체를 DAO(Data Access Object) 라고 한다.

## REPOSITORY

- Domain-Driven Design 의 기본 빌딩 블록 중 하나.
- 도메인 레이어에 객체 지향적인 컬렉션 관리 인터페이스를 제공하기 위한 PURE FABRICATION
- 변경에 대한 불변식을 유지하기 위해 하나의 단위(aggregate)로 취급.
    - AGGREGATE: 변경의 빈도가 비슷하고, 동시 접근에 대한 락의 단위가 되는 객체의 집합, AGGREGATE 별로 하나의 REPOSITORY를 사용
- 특정 객체 집합의 가상적인 메모리 컬렉션을 관리
    - 시스템에 1억건의 주문 정보가 존재한다면 OrderRepository는 1억건의 Order 인스턴스와 그와 관려된 OrderLineItem 인스턴스가 메모리 내에 로드되어 있다고 가정한다.
- REPOSITORY 자체는 퍼시스턴스 메커니즘에 대한 어떤 가정도 하지 않는다
- REPOSITORY를 사용할 때는 이미 모든 Order 객체와 OrderLineItem이 메모리에 로드되어 있다고 가정하고 메모리에 로드된 특정 객체, 객체 집합, 전체 객체에 접근하기 위해 REPOSITORY를 사용한다.
- REPOSITORY에 퍼시스턴스 로직이 포함될 경우 REPOSITORY를 직접 사용하는 도메인 객체 역시 특정 퍼시스턴스 메커니즘에 의존하게 되는 부작용이 발생
- REPOSITORY를 인터페이스와 구현부로 분리한 후 인터페이스는 도메인 레이어에 속하도록 하고, 구현부는 퍼시스턴스 레이어에 속하도록 하는 것
- DIP(Dependency Inversion Principle)에 기반하여 인터페이스와 구현부 사이의 계층을 분리하는 패턴을 SEPARATED INTERFACE이라고 함

## DAO와 Repository의 차이점

- Entity Bean을 대체하기 위한 DAO의 인터페이스는 데이터베이스의 CRUD 쿼리와 1:1 매칭되는 세밀한 단위의 오퍼레이션을 제공.
- REPOSITORY는 메모리에 로드된 객체 컬렉션에 대한 집합 처리를 위한 인터페이스를 제공
- DAO가 제공하는 오퍼레이션이 REPOSITORY 가 제공하는 오퍼레이션보다 더 세밀. REPOSITORY에서 제공하는 하나의 오퍼레이션이 DAO의 여러 오퍼레이션에 매핑되는 것이 일반적.
- 하나의 REPOSITORY 내부에서 다수의 DAO를 호출하는 방식으로 REPOSITORY를 구현할 수 있다.
- DAO는 데이터베이스 뿐만 아니라 B2B, LDAP, 메인 프레임, 레거시 시스템과 같은 다양한 종류의 외부 시스템과의 상호작용을 캡슐화하기 위해 사용될 수 있다. - 외부 시스템에 대한 GATEWAY 역할
- REPOSITORY는 객체 컬렉션 처리에 관한 책임만을 가지고 있다. DDD에서 외부 시스템과의 상호작용은 별도의 SERVICE가 담당.
- DAO는 퍼시스턴스 레이어에 속한다. 반면 REPOSITORY의 인터페이스는 도메인 레이어에 속한다.
- 테이블 별로 하나의 DAO를 만든다(TABLE DATA GATEWAY 패턴). Entity Bean을 DAO로 대체하면서 테이블 당 하나의 Entity Bean을 만들어야 했던 EJB의 제약을 그대로 답습한 것이 아닌가 추측
- DAO가 인터페이스 차원에서는 데이터베이스에 대한 의존도를 캡슐화할 수 있다고 하더라도 테이블 변경에 대해서는 투명하지 않다는 것을 의미
- REPOSITORY의 인터페이스는 AGGREGATE ROOT(또는 Entry Point)와 관련된 오퍼레이션만을 포함하며 따라서 테이블 변경에 대해서 투명하다.
- DAO는 TRANSACTION SCRIPT 패턴과 함께 사용된다. 반면 REPOSITORY는 DOMAIN MDOEL 패턴과 함께 사용된다.
- 두 빌딩 블록을 식별하는 과정에서 나타난다. REPOSITORY는 도메인 모델링 단계에서 하나의 개념 단위로 묶인 AGGREGATE를 도출하는 과정에서 자동으로 식별된다. 앞에서 설명한 바와 같이 DAO는 TABLE DATA GATEWAY 패턴에 따라 데이터베이스 테이블 별로 생성되는 것이 일반적이며 퍼시스턴스 레이어에 대한 FACADE 역할을 수행한다. 따라서 DAO 식별 과정은 도메인 규칙 보다는 데이터베이스 테이블의 단위에 영향을 받는 경향이 강하다.