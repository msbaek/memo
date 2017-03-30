# Law Of Demeter

[원문-Clean Code Talks - Dependency Injection](https://testing.googleblog.com/2008/11/clean-code-talks-dependency-injection.html)

## 위키피디아 정의

- http://en.wikipedia.org/wiki/Law_of_Demeter
- “Only talk to your immediate friends.”
- 객체 A가 객체 B에게 서비스를 요청(메소드 호출을 통해)하는 것은 적합
- 객체 A가 객체 B를 통해 또 다른 객체 C에 접근하여 서비스를 요청하는 것은 위배
    - 묵시적으로 A가 B의 내부 구조에 대해 많은 지식(종속성)을 갖게 된다.
    - 필요하다면 B를 수정하여 A가 B에게 직접 요청을 하고(simple), B가 요청을 적절한 다른 객체(C)에게 전파하는 것이 해결책(A는 더이상 B의 내부 구조에 대해 알 필요가 없다).
    - 혹은 A가 C에 대한 레퍼런스를 직접 가지고 직접 호출하는 방법도 있다.
- 형식적 정의
    - 객체 O의 메소드 M은 아래와 같은 객체들의 메소드들만 호출할 수 있다
        - O itself
        - M's parameters
        - any objects created/instantiated within M
        - O's direct component objects
- 다시 말해 객체는 어떤 메소드가 반환한 결과 객체의 메소드를 호출하지 말아야 한다. – “use only one dot”.
    - `a.b.Method()` - breaks the law
    - `a.Method()` - OK

## 예제로 살펴보기

- 상점에서 3000원 짜리 물품을 구매하는 경우
    - 점원에게 3000원을 지불할 것인가 ?
    - 점원에게 지갑을 주고 3000월을 빼가라고 할 것인가 ?

### 1. Violated

- SUT(System Under Test)

```java
class Goods {
  AccountsReceivable ar;
  void purchase(Customer c) {
   Money m = c.getWallet().getMoney();
   ar.recordSale(this, m);
  }
}
```

- Test
	- To test this we need to create a valid Customer with a valid Wallet which contains the real item of interest.(Money)
    
```java    
class GoodsTest {
  void testPurchaseIsHorribleBreaksLoD() {
    AccountsReceivable ar = new FakeAR();
    Goods g = new Goods(ar);
    Money m = new Money(25, USD);
    Wallet w = new Wallet(m);
    Customer c = new Customer(w);
    g.purchase(c);
    assertEquals(25, ar.getSales());
  }
}
```

### 2. Obeyed

- SUT

```java
class Goods {
  AccountsReceivable ar;
  void purchase(Money m) {
    ar.recordSale(this, m);
  }
}
```

- Test

```java
class GoodsTest {
  void testPurchaseTheRightWay() {
    AccountsReceivable ar = new MockAR();
    Goods g = new Goods(ar);
    g.purchase(new Money(25, USD));
    assertEquals(25, ar.getSales());
  }
}
```