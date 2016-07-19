# Episode 23. Mocking Part 1

``` 
요약
Dummy
- interface의 모든 메소드들이 `return null;`로 구현된 테스트 더블

Stub
- dummy의 일종. 0이나 null 대신 테스트가 필요로 하는 특정 값을 반환

Spy
- stub의 일종. 자신이 호출된 fact를 기억하고 후에 테스트에 이러한 fact를 보고
  - 어떤 함수가, 언제, 몇번, 어떤 인자로 호출되었는지 등

Mock
- spy의 일종. 어떤 일이 일어나야 하는지를 아는 spy

Fake
- simulator
- 실세계 객체가 하는 것 처럼 입력에 따라 다른 응답을 함.
- 복잡도와 유지보수를 위해서 가급적 피애햐 함
```

mocking library를 안 사용하는 것이 더 좋은 것 같다.

## Boundaries and Mocks

![](https://api.monosnap.com/rpc/file/download?id=CU3JCIfaywczeVNhMpgXwrOW5KQqUk)

* user가 login하면 controller가 user name과 password를 간단한 DS(LoginRequest)에 담아서 LoginInteractor에 전달한다.
* LoginInteractor는 user name과 password를 가지고 authorizer를 통해 확인한다.
* Authorizer가 user를 accept하면 Authorizer는 valid user id를 반환한다.
* LoginInteractor는 UserGateway를 사용해서 User 비즈니즈 객체를 fetch하고, 마지막 로그인 시간, 로그인 횟수 등을 얻는다.
* LoginInteractor 는 이러한 데이터를 또 다른 단순한 DS(LoginResponse)를 통해 LoginPresenter에 전달한다.
* LoginPresenter는 화면에 보일 데이터를 준비한다.

### LoginInteractor에 대한 자동화 테스트를 어떻게 만들 것인가 ?

다음과 같이 쉽게 자동화된 테스트를 만들 수 있다고 한다.

* setup에서 User를 생성. username, password를 부여
* user record를 수정하여 last login time과 login 횟수를 갖도록 한다.

이와 같이 DB를 사용하면 테스트가 느려진다. 그리고 웹의 경우는 HTML을 긁어서 결과를 확인해야 할 수도 있다. UI를 변경할 때마다 테스트가 깨지게 된다. 또 테스트를 웹서버에서 수행해야만 한다. 이건 정말 느리다. 차라리 수동 테스트를 하는게 낫다.

Mocking을 이용하면 해소된다. DB에 연결할 필요도 없고, 웹서버를 사용할 필요도 없다. 또 HTML을 긁어서 확인할 필요도 없다. mocking이 수월하도록 nice clean decoupled 아키텍쳐를 만들어야 한다.

위 그림에서 빨간선 내부에 Interactor와 Interactor가 자신의 일을 수행하기 위해 호출해야 하는 모든 인터페이스가 존재한다. 이 인터페이스의 구현체들은 authorizing, DB access, presentation 등의 기능을 수행한다. 테스트할 때는 이러한 구현체를 사용할 필요가 없다. 다른 구현체를 사용할 수 있다. 테스트를 돕는 구현체... 이것이 Mocking이다.

테스트에서 개별적으로 원하는 결과를 반환하도록 하기 위해 stub을 생성한다.

* UserId(1)을 반환하는 StubAuthorizer
* InvalidUserID를 반환하는 RejectingAuthorizerStub

등을 만들어 놓고 테스트 케이스마다 적절한 구현체를 사용하면 된다.

LoginInteractorImpl에 setAuthorizer, setUserGateway는 있고, 이들에 대한 stub이 필요하지만 presenter는 다르다고 한다. presenter에 대해서는 적절한 시기에 적절한 메소드가 호출되었는지만 확인하면 된다. - interactor의 presenter에 대한 relationship을 spy하기 원한다.

* stub은 원하는 결과를 반환하도록 설정 가능
* spy는 추가적으로 어떤 인자를 전달받았는지를 저장했다가 전달받은 인자를 assert할 수 있음.

## Test Doubles

![](https://api.monosnap.com/rpc/file/download?id=Twpm81WPaGXqhywpwPGvBisNluaKJt)

### The Dummy

가장 단순한 테스트 더블. interface에 있는 모든 메소드를 do nothing으로 구현하는 객체가 Dummy. 리턴하는 값이 필요하다면 대개 null이나 0을 반환

```
public class HourlyReportInteractor {
  public void generateReport(Date reportDate, Session session) {
    if (!isValidReportDate(reportDate))
      throw new InvalidDate();
    //...
  }
```  

위 코드를 테스트하려고 한다.

```
  @Test(expected = HourlyReportInteractor.InvalidDate.class)
  public void testInvalidDate() throws Exception {
    Date reportDate = null;
    Session session = new DummySession();
    interactor.generateReport(reportDate, session);
  }
```

DummySession - Session은 생성하기 어려운 객체이다. - reportDate가 invalid한 경우 session은 사용되지도 않는다. - DummySession을 이용한다(모든 method에서 return null) - 이런 경우 Dummy가 효율적이다.

테스트하려는 메소드가 인자를 갖는데 테스트나 테스트되는 함수가 인자와 무관할 때 사용한다.

Dummy는 테스트 더블의 일종으로 어떤 액션도 취하지 않는 함수들로 이뤄진다.

### The Stub

stub은 dummy의 일종이나 null, 0을 반환하는 대신 특정 테스트가 요구하는 값을 반환한다.

stub은 dummy이다. 함수에서 어떤 일도 하지 않는다. 하지만 0이나 null 대신 어떤 요구되는 고정된 특별한 값을 반환한다. 아무것도 하지 않지만 당신이 원하는 값을 반환한다.

stub은 특정 경로로 테스트가 수행되도록 할 때 사용된다.

login interactor의 경우 사용자가 잘못된 패스워드를 입력했을 때의 행위를 어떻게 테스트할 것인가 ? authenticator를 stub해서 false를 반환하도록 하여 interactor가 올바른 행위를 하는지를 확인할 수 있다.

테스트와 stub 간에는 대응 관계가 존재한다. 테스트는 적절한 경로로 수행되도록 하는 stub를 선택하도록 작성된다. 올바른 패스워드로 login이 성공하는지 조사하는 테스트는 true를 반환하는 stub을 사용하고, 잘못된 패스워드를 처리하는 login을 조사하는 테스트는 false를 반환하는 stub을 사용한다.

### The Spy

spy는 stub이다. spy는 자신이 호출된 특별한 fact를 기억하고, 후에 테스트에 이러한 fact를 보고한다.

* 어떤 함수가 호출되었는지
* 그 함수가 언제 호출되었는지
* 몇번이나 호출되었는지
* 어떤 인자가 전달되었는지

를 알려줄 수 있다.

production 코드가 외부 서비스를 적절하게 호출하였는지 테스트하고자 할 때 유용한 기법이다.

### The Mock

Mock은 Spy이고 추가적으로

* 테스트에 유용한 값을 반환하고
* 호출된 fact들에 대해서 기억한다.
* 어떤 일이 일어나야 하는지를 아는 Spy이다.

### The Fake

마지막 테스트 더블은 Fake이다. Fake는 dummy도 stub도 spy도 mock도 아닌 Simulator이다.

실세계 객체가 하는 것 처럼 입력에 따라 다른 응답을 한다.

fake가 많아지면 복잡해지고 유지보수하기 어려워진다. Test에서 필요한 fake만 작성하도록 한다.

**avoid fakes when you can**

단위 테스트에서는 fake까지 필요한 경우는 드물다. 하지만 integration test에서는 fake가 유용한 경우가 많다.