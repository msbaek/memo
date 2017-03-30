# Composition over Inheritance

[원문-Using Composition in Object Modeling](https://dzone.com/articles/using-composition-object)

```java
public class Person {
  private Long id;
  private String firstName;
  private String lastName;
}
 
public class User extends Person {
  private String userName;
  private String password;
 
}
 
public class Customer extends Person {
  private int creditLimit;
}
 
public class Employee extends Person {
  private Date beginDate;
  private Date endDate;
}
```

## 전형적인 문제

- Customer가 항상 Person은 아니다.
- 어떤 Person은 Customer이면서 Vendor이면서 Employee이면서 또 User일 수도 있다.
- 상속을 사용하는 것은 하나의 클래스로부터 상속받는 하나의 기회를 박탈함으로써 과도하게 제약을 가한다
- 어떤 Person이 회사를 그만두거나 재입사하면 어떻게 해야 하나 ?
    - 이럴 경우 시스템에서Person에 대한 2개의 인스턴스를 유지해야 하나 ?
    - Person이 Customer이면서 User이고 Employee라면 대개의 Person 인스턴스는 중복이 될 것이다.
    - 하나의 인스턴스에서 이름이 변경되었는데 다른 인스턴스들은 갱신되지 않는 문제가 발생할 수도 있다.

## Role your own

- 일반적인 해법은 Person이 역할 집합을 가지게 하여 동시에 user, employy, customer, vendor 등의 역할을 수행하게 하는 것이다.
- 이 방법은 상속 계층에 대한 고민을 해결한다. 또한 user가 employee이든지 혹은 반대이든지에 대한 문제도 핵결한다. 그리고 한번에 Person이 모든 역할을 수행할 수도 있게 한다.
- 하지만 이것은 user, customer, vendor, employee가 모두 person일 때만 동작한다.
- 또한 이러한 엔터티들을 참조하는 것이 명확하지 않다. customer를 참조하는 order가 있을 때 person을 참조해야 하나 ? 혹은 customer의 role을 참조해야 하나 ?

## Compose Yourself

- 관계를 모델링하기 위해 상속 대신 컴포지션을 사용할 수 있다.
- Person, Customer, Employee를 별도의 클래스로 정의
- 해당 역할에 특화된 정보만 encapsulate하고 관련된 person에 대한 참조를 갖도록
- 이 방법의 장점은 서로 다른 클래스들이 Person 클래스를 상속할 필요가 없다는 것이다. 원하는 클래스를 상속할 수 있도록 한다.

```java
public class User {
  private Long id;
  private String userName;
  private String password;
  private Person person;
}
 
public class Customer {
  private Long id;
  private Person person;
  private int creditLimit;
}
 
public class Employee {
  private Long id;
  private Person person;
  private Date beginDate;
  private Date endDate;
}
```

- Employee가 User를 상속받아야 하는지 혹은 반대의 경우인지 고민할 필요가 없다.
- 훨씬 유연한 설계가 된다.
- CompanyCustomer가 추가되어야 하는 경우를 살펴보자.
    - Customer가 Person을 상속하는 구조라면 이러한 변경 요청은 악목이 될 것이다.
    - 컴포지션을 사용한 경우라면 아래와 같이 해결 할 수 있다.

```java
public abstract class AbstractCustomer {
  private Long id;
  private int creditLimit;
  public abstract String getName();
}
 
public class PersonCustomer extends AbstractCustomer {
  private Person person;
 
  public abstract String getName() {
    return person.getName();
  }
}
 
public class CompanyCustomer extends AbstractCustomer {
  private Company company;
 
  public abstract String getName() {
    return company.getName();
  }
}
```

- 위의 예에서 모든 타입에 적용되는 공통 속성은 베이스 클래스에 위치시킨다. 그러면 특정 서브클래스 타입이 아닌 베이스 클래스 타입에 대한 참조로 이러한 기능을 사용할 수 있게 된다.

`String msg = "Hi There "+customer.getName();`

- 위 코드가 아래 코드 보다 훨 깔끔하다.

```java
String msg = "Hi There ";
 
if (customer instanceof PersonCustomer) {
  msg = msg+((PersonCustomer)customer).getPerson.getPersonName();
}
 
if (customer instanceof CompanyCustomer) {
  msg = msg+((CompanyCustomer)customer).getCompany.getCompanyName();
}
```

## Unique People, non-unique references

- 회사를 퇴사하고 재입사하는 경우, 쉽게 해결할 수 있다.
- 퇴사시 Employee 인스턴스에 endDate를 설정하고, 재입사사 동일한 Person 인스턴스를 참조하는 새로운 Emploee 인스턴스를 생성하면 된다.
- 퇴사, 재입사를 해도 여전히 하나의 Person 인스턴스를 사용한다. 이 인스턴스는 2개의 Employee 인스턴스에 의해 참조되고, 중복은 없어진다.