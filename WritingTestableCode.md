# Guide: Writing Testable Code

[원문](http://misko.hevery.com/code-reviewers-guide/)

## Flaw #1: Constructor does Real Work

[원문](http://misko.hevery.com/code-reviewers-guide/flaw-constructor-does-real-work)

- constructor에서 아래와 같은 작업은 테스트에 필요한 seam을 제거하고, 서브블래스와 mock이 원치 않는 행위를 상속받게 만든다.
    - collaborator를 생성/초기화
    - 다른 service와 상호작용
    - 자기 자신의 상태 설정
- 다시 말해서 constructor에서 많은 일을 하면 테스트에서 생성, collaborator 변경을 어렵게 만든다.

#### Warning Signs:

- constructor나 필드 정의에서 new 키워드가 사용된 경우
- constructor나 필드 정의에서 static method를 호출한 경우
- constructor에서 필드 할당문 이외의 다른 문장이 사용된 경우
- constructor 종료 후에 객체의 완전히 초기화 되지 않은 경우(initialize 메소드 주의)
- constructor에 조건문/루프문 같은 제어 로직이 있는 경우
- factory나 builder 사용 없이 constructor에 복잡한 객체 그래프 생성 로직이 있는 경우
- initialization 블록을 추가하거나 사용하는 경우

#### Why this is a Flaw

- constructor에서 collaborator를 생성/초기화하면 유연성이 없어지고 완성도가 떨어진 겹합도(coupling)이 높은 설계가 된다.
- 이런 경우 테스트시 테스트 collaborator 주입이 불가능하다.
	- SRP(Single Responsibility Principle) 위반
		- 객체 그래프 생성은 엄연히 독립된 책임이고 이 책임은 constructor에서 수행하는 것은 SRP 위반이다.
	- 직접 테스트하는 것이 어려워짐
		- 치환 불가한 collaborator의 미묘한 변경은 constructor에 반영되어야 하는 경우가 발생할 수 있다. 이런 경우 테스트가 어려워진다.
	- 테스트를 위한 서브클래싱/Overriding이 여전히 결함으로 존재한다.
	- 테스트를 위한 Collaborator로 치환이 불가하다.
	- “Seam”을 없앤다.
	- Multiple Constructor를 갖는다 해도 여전히 문제
	- Bottom Line
        - 고립된 상태(isolation)나 테스트 더블 collaborator로 얼마나 쉽게 클래스를 생성할 수 있는가가 관건
        - 생성하기 어렵다면 constructor에서 너무 많은 일을 하는 것임
        - 테스트에서 생성하기 어렵다면 해당 클래스를 사용하는 다른 코드에서도 사용하기 어렵다.

#### Recognizing the Flaw

- 아래와 같은 증상을 살펴보라.
	- 테스트에서 테스트 더블로 치환하고 싶은 객체를 new 키워드로 생성하고 있지 않는가 ?
	- mocking, injection이 불가한 static method 호출이 있는가 ?
	- conditional/loop logic이 존재하는가 ?

#### Fixing the Flaw

- “Do not create collaborators in your contructor, but pass them in”
- “Don't look for things !! Ask for things !!
- 객체 그래프 생성/초기화 책임을 다른 객체로 이동시켜라(builder, factory 등을 추출하고 이런 collaborator들을 constructor에 전달하라).

#### Examples

###### Service Object Digging Around in Value Object

- Before: Hard to Test
	- SUT
    
```java
// Basic new operators called directly in
//   the class' constructor. (Forever
//   preventing a seam to create different
//   kitchen and bedroom collaborators).
class House {
    Kitchen kitchen = new Kitchen();
    Bedroom bedroom;
 
    House() {
        bedroom = new Bedroom();
    }
 
    // ...
}
```

	- Test
    
```java    
// An attempted test that becomes pretty hard
 
class HouseTest extends TestCase {
    public void testThisIsReallyHard() {
        House house = new House();
        // Darn! I'm stuck with those Kitchen and
        //   Bedroom objects created in the
        //   constructor. 
 
        // ...
    }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java    
class House {
    Kitchen kitchen;
    Bedroom bedroom;
 
    // Have Guice create the objects
    //   and pass them in
    @Inject
    House(Kitchen k, Bedroom b) {
        kitchen = k;
        bedroom = b;
    }
    // ...
}
```

	- Test
    
```java    
// New and Improved is trivially testable, with any
//   test-double objects as collaborators.
 
class HouseTest extends TestCase {
    public void testThisIsEasyAndFlexible() {
        Kitchen dummyKitchen = new DummyKitchen();
        Bedroom dummyBedroom = new DummyBedroom();
 
        House house = new House(dummyKitchen, dummyBedroom);
 
        // Awesome, I can use test doubles that
        //   are lighter weight.
 
        // ...
    }
}
```

- 위 예제는 객체 그래프 생성과 로직이 섞여있다. 테스트를 작성할 때 운영환경과 다른 객체 그래프(대개는 좀 작은 객체 그래프로, 어떤 객체들은 테스트 더블로 치환된)를 만들고자 할 때가 많다.
- new 키워드를 constructor에 유지하고는 테스트를 위한 객체 그래프를 만들수 없다.
- Flaws:
    - 필드 정의에 new 키워드를 사용했다.
    - 만일 Kitchen이 파일/데이터베이스와 같이 생성 비용이 많이 드는 경우라면 House 객체를 만들기 어려워진다.
    - kitchen이나 bedroom의 행위를 polymorphical하게 변경할 수 없기 때문에 설계가 깨지기 쉽다.
- Kitchen이 value object(LinkdList, Map, User, Email Address 등과 같이)라면 value object가 service 객체를 참조하지 않기 때문에 inline으로 생성할 수 있다. Service 객체는 테스트 더블로 치환될 필요가 있는 타입이다. 그래서 static method 호출로 직접 생성해서는 안된다.

###### Constructor takes a partially initialized object and has to set it up

- Before: Hard to Test
	- SUT
    
```java
// SUT initializes collaborators. This prevents
//   tests and users of Garden from altering them.
 
class Garden {
    Garden(Gardener joe) {
        joe.setWorkday(new TwelveHourWorkday());
        joe.setBoots(new BootsWithMassiveStaticInitBlock());
        this.joe = joe;
    }
    // ...
}
```

	- Test
    
```java    
// A test that is very slow, and forced
//   to run the static init block multiple times.
 
class GardenTest extends TestCase {
    public void testMustUseFullFledgedGardener() {
        Gardener gardener = new Gardener();
        Garden garden = new Garden(gardener);
        new AphidPlague(garden).infect();
        garden.notifyGardenerSickShrubbery();
        assertTrue(gardener.isWorking());
    }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java
// Let Guice create the gardener, and have a
//   provider configure it.
 
class Garden {
    Gardener joe;
 
    @Inject
    Garden(Gardener joe) {
        this.joe = joe;
    }
 
    // ...
}
 
// In the Module configuring Guice.
 
@Provides
Gardener getGardenerJoe(Workday workday, BootsWithMassiveStaticInitBlock badBoots) {
    Gardener joe = new Gardener();
    joe.setWorkday(workday);
 
    // Ideally, you'll refactor the static init.
    joe.setBoots(badBoots);
    return joe;
}
```    

	- Test
    
```java    
// The new tests run quickly and are not
//   dependent on the slow
//   BootsWithMassiveStaticInitBlock
 
class GardenTest extends TestCase {
 
    public void testUsesGardenerWithDummies() {
        Gardener gardener = new Gardener();
        gardener.setWorkday(new OneMinuteWorkday());
        // Okay to pass in null, b/c not relevant
        //   in this test.
        gardener.setBoots(null);
 
        Garden garden = new Garden(gardener);
 
        new AphidPlague(garden).infect();
        garden.notifyGardenerSickShrubbery();
 
        assertTrue(gardener.isWorking());
    }
}
```

- 객체 그래프 생성(Garden의 collaborator Gardener를 생성하고 설정하는 것)은 Garden이 수행해야 하는 책임과 다른 책임이다.
- 이 처럼 constructor에 설정과 생성이 섞여있으면, 객체는 깨지기 쉽고 구체적 객체 그래프 구조에 얽메이게 된다. 이로 인해 코드를 변경하기 어렵고, 테스트하기 거의 불가능해 진다.
- Flaws:
    - Garden은 Gardener를 필요로 하지만 Gardener를 설정하는 것은 Garden의 책임이 아니다.
    - Garden의 유닛 테스트에서 warkday는 constructor에서 설정된다. 이로 인해 Joe가 하루에 12시간 일하게 된다. 이러한 의존성 걸정은 테스트가 느리게 동작하게 만든다. 유닛 테스트에서는 짧은 시간만 일하도록 설정하기를 원할 것이다.
    - boots를 변경할 수 없다. BootsWithMassiveStaticInitBlock을 사용하고 로딩하는 문제를 회피하기 위해 boots에 대해 테스트 더블을 사용하고 싶을 것이다(Static initialization block은 위험하고 문제를 야기할 소지가 많다. 특히 전역상태와 상호작용할 경우는 더 위험하다).
- 초기화되어야 하는 collaborator가 필요한 경우 2개의 객체를 갖도록 하라. 2개의 객체를 초기화하고 완전히 초기화된 상태로 클래스의 constructor에 전달하라.

###### Violating the Law of Demeter in Constructor

- Before: Hard to Test
	- SUT

```java
// Violates the Law of Demeter
// Brittle because of excessive dependencies
// Mixes object lookup with assignment
 
class AccountView {
    User user;
    AccountView() {
        user = RPCClient.getInstance().getUser();
    }
}
```

	- Test
    
```java    
// Hard to test because needs real RPCClient
class ACcountViewTest extends TestCase {
 
    public void testUnfortunatelyWithRealRPC() {
        AccountView view = new AccountView();
        // Shucks! We just had to connect to a real
        //   RPCClient. This test is now slow.
 
        // ...
    }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java
class AccountView {
    User user;
 
    @Inject
    AccountView(User user) {
        this.user = user;
    }
}
 
// The User is provided by a GUICE provider
@Provides
User getUser(RPCClient rpcClient) {
    return rpcClient.getUser();
}
 
// RPCClient is also provided, and it is no longer
//   a JVM Singleton.
@Provides @Singleton
RPCClient getRPCClient() {
    // we removed the JVM Singleton
    //   and have GUICE manage the scope
    return new RPCClient();
}
```

	- Test

```java
// Easy to test with Dependency Injection
class AccountViewTest extends TestCase {
 
    public void testLightweightAndFlexible() {
        User user = new DummyUser();
        AccountView view = new AccountView(user);
        // Easy to test and fast with test-double
        //   user.
 
        // ...
    }
}
```

- 어플리케이션의 전역 상태에 접근하고 RPCClient singleton의 holder를 얻었다. singleton은 필요 없는 것이고 User만이 필요한 것인데 말이다. 첫번째 잘못은 seam을 제공하지 않는 static method를 사용한 것이고, 두번째 잘못은 “Law of Demeter”를 위배한 것이다.
- Flaws:
    - mock 객체를 사용하기 위해 RPCClient.getInstance() 메소드를 가로챌 수 없다(static method는 non-interceptable & non-mockable).
    - SUT가 RPCClient를 필요로 하지 않는데 왜 RPCClient를 mock으로 치환해야 하는가?(AccountView는 rpc instance를 필드에 저장하지 않는다). User만 저장/접근할 수 있으면 된다.
    - AccountView를 생성하려는 모든 테스트는 위의 문제를 갖는다. 하나의 테스트에서 문제를 해결했다고 하더라도 다른 테스트에서는 문제가 해결된 것이 아니다.
- 개선된 코드에서는 직접적으로 필요한 객체만 전달되었다: User collaborator. 테스트시 생성해야 하는 것은 (real or test double) User 객체뿐이다. 이로 인해 설계가 보다 유연해지고 테스트 가능성이 보다 높아진다.

###### Creating Unneeded Third Party Objects in Constructor

- Before: Hard to Test
	- SUT
    
```java    
// Creating unneeded third party objects,
//   Mixing object construction with logic, &
//   "new" keyword removes a seam for other
//   EngineFactory's to be used in tests.
//   Also ties you to the (slow) file system.
 
class Car {
    Engine engine;
    Car(File file) {
        String model = readEngineModel(file);
        engine = new EngineFactory().create(model);
    }
 
    // ...
}
```

	- Test
    
```java    
// The test exposes the brittleness of the Car
class AccountViewTest extends TestCase {
 
    public void testNoSeamForFakeEngine() {
        // Aggh! I hate using files in unit tests
        File file = new File("engine.config");
        Car car = new Car(file);
 
        // I want to test with a fake engine
        //   but I can't since the EngineFactory
        //   only knows how to make real engines.
    }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java    
// Asks for precisely what it needs
 
class Car {
    Engine engine;
 
    @Inject
    Car(Engine engine) {
        this.engine = engine;
    }
 
    // ...
}
 
// Have a provider in the Module
//   to give you the Engine
@Provides
Engine getEngine(EngineFactory engineFactory, @EngineModel String model) {
    //
    return engineFactory.create(model);
}
 
// Elsewhere there is a provider to
//   get the factory and model
```

	- Test
    
```java
// Now we can see a flexible, injectible design
class AccountViewTest extends TestCase {
 
    public void testShowsWeHaveCleanDesign() {
        Engine fakeEngine = new FakeEngine();
        Car car = new Car(fakeEngine);
 
        // Now testing is easy, with the car taking
        //   exactly what it needs.
    }
}
```

- Car가 자신의 엔진을 만들기 위해 EngineFactory를 필요로 하는 것은 의미에 맞지 않는다. Car는 엔진을 어떻게 만들것인가를 상관하지 말고 이미 만들어진 엔진을 공급 받아야 한다. 주행하는 것이 목적인 Car는 공장에 대한 레퍼런스를 갖지 말아야 한다. 같은 맥락으로 constructor에서는 직접적으로 필요하지 않은 3rd party 객체가 아닉라 그 객체가 생성하는 객체만 사용해야 한다.
- Flaws:
    - 실제로 필요한 것은 Engine인데 File을 넘기고 있다.
    - 3rd party 객체(EngineFactory)를 생성하고 있다. 3rd party 객체 생성은 inject/override 불가하므로 불필요한 작업이다.
    - Car가 어떻게 EngineFactory를 만드는지 또 어떻게 엔진을 만드는지 아는 것은 어리석인 것이다.
    - 이 테스트의 문제를 해소한다고 해도 AccountView를 생성해야 하는 모든 테스트는 위와 같은 불합리한 작업을 수행해야 한다.
    - Car constructor가 호출되는 모든 테스트는 file을 접근해야 한다. 이 작업은 매우 느리고, 테스트가 진정한 유닛 테스트가 될 수 없게 한다.
- 이러한 3rd party 객체를 제거하고 constructor에서의 작업을 단순한 변수 할당문으로 치환하라. 사전에 설정된 변수들을 constructor의 필드로 할당하라. 다른 객체(factory, builder, DI container)가 constructor의 parameter를 생성하는 작업을 담당하도록 하라. 객체의 주요 책임과 객체 그래프 생성을 분리하여 보다 유연하고 유지보수 가능한 설계를 유지하라.

###### Directly Reading Flag Values in Constructor

- Before: Hard to Test
	- SUT
    
```java    
// Reading flag values to create collaborators
 
class PingServer {
    Socket socket;
    PingServer() {
        socket = new Socket(FLAG_PORT.get());
    }
 
    // ...
}
```

	- Test
    
```java    
// The test is brittle and tied directly to a
//   Flag's static method (global state).
 
class PingServerTest extends TestCase {
    public void testWithDefaultPort() {
        PingServer server = new PingServer();
        // This looks innocent enough, but really
        //   it forces you to mutate global state
        //   (the flag) to run on another port.
    }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java    
// Best solution (although you also could pass
//   in an int of the Socket's port to use)
 
class PingServer {
    Socket socket;
 
    @Inject
    PingServer(Socket socket) {
        this.socket = socket;
    }
}
 
// This uses the FlagBinder to bind Flags to
// the @Named annotation values. Somewhere in
// a Module's configure method:
new FlagBinder(
    binder().bind(FlagsClassX.class));
 
// And the method provider for the Socket
@Provides
Socket getSocket(@Named("port") int port) {
    // The responsibility of this provider is
    //   to give a fully configured Socket
    //   which may involve more than just "new"
    return new Socket(port);
}
```

	- Test
    
```java    
// The revised code is flexible, and easily
//   tested (without any global state). 
 
class PingServerTest extends TestCase {
  public void testWithNewPort() {
    int customPort = 1234;
    Socket socket = new Socket(customPort);
    PingServer server = new PingServer(socket);
 
    // ...
  }
}
```

- 인자를 갖지 않는 constructor가 많은 의존성을 가지고 있다.API가 거짓을 말하고 있는 것이다. API는 인자가 없으므로 쉽게 만들수 있다고 말고하고 있지만 PingServer는 불안정하고 전역 상태에 의존하고 있다.
- Flaws:
    - 테스트는 객체를 생성하기 위해 전역 변수 FLAG_PORT에 의존하고 있다. 테스트 순서에 의해 테스트가 영향을 받게 된다.
    - statitic하게 접근 가능한 전역 변수 플래그로 인해 병렬로 테스트 수행이 불가해 진다.
    - 객체 생성이 잘못된 곳에서 수행되고 있어서 Socket에 추가적인 설정(setSoTimeout 호출 등)이 불가하다.
- PingServer는 port 번호가 아니라 소켓을 필요로 한다. port 번호를 전달함으로써 테스트시 실제 소켓/쓰레드를 사용해야만 한다. 포트 번호가 아니라 소켓을 전달하도록 수정하면 테스트시 mock 소켓을 사용할 수 있다.
- 명시적으로 port 번호를 전달함으로써 전역 상태에 대한 의존성을 제거하여 테스트를 단순화할 수 있다. 궁극의 해결책은 진짜로 필요한 소켓을 전달하는 것이다.

###### Directly Reading Flags and Creating Objects in Constructor

- Before: Hard to Test
	- SUT
    
```java    
// Branching on flag values to determine state.
 
class CurlingTeamMember {
  Jersey jersey;
 
  CurlingTeamMember() {
    if (FLAG_isSuedeJersey.get()) {
      jersey = new SuedeJersey();
    } else {
      jersey = new NylonJersey();
    }
  }
}
```

	- Test
    
```java    
// Testing the CurlingTeamMember is difficult.
//   In fact you can't use any Jersey other
//   than the SuedeJersey or NylonJersey.
 
class CurlingTeamMemberTest extends TestCase {
  public void testImpossibleToChangeJersey() {
    //  You are forced to use global state.
    // ... Set the flag how you want it
    CurlingTeamMember russ =
        new CurlingTeamMember();
 
    // Tests are locked in to using one
    //   of the two jerseys above.
  }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java    
// We moved the responsibility of the selection
//   of Jerseys into a provider.
 
class CurlingTeamMember {
  Jersey jersey;
 
  // Recommended, because responsibilities of
  // Construction/Initialization and whatever
  // this object does outside it's constructor
  // have been separated.
  @Inject
  CurlingTeamMember(Jersey jersey) {
    this.jersey = jersey;
  }
}
 
// Then use the FlagBinder to bind flags to
//   injectable values. (Inside your Module's
//   configure method)
  new FlagBinder(
      binder().bind(FlagsClassX.class));
 
// By asking for Provider<SuedeJersey>
//   instead of calling new SuedeJersey
//   you leave the SuedeJersey to be free
//   to ask for its dependencies.
@Provides
Jersey getJersey(
     Provider<SuedeJersey> suedeJerseyProvider,
     Provider<NylonJersey> nylonJerseyProvider,
     @Named('isSuedeJersey') suede) {
  if (sued) {
    return suedeJerseyProvider.get();
  } else {
    return nylonJerseyProvider.get();
  }
}
```

	- Test
    
```java    
// We moved the responsibility of the selection
//   of Jerseys into a provider.
 
class CurlingTeamMember {
  Jersey jersey;
 
  // Recommended, because responsibilities of
  // Construction/Initialization and whatever
  // this object does outside it's constructor
  // have been separated.
  @Inject
  CurlingTeamMember(Jersey jersey) {
    this.jersey = jersey;
  }
}
 
// Then use the FlagBinder to bind flags to
//   injectable values. (Inside your Module's
//   configure method)
  new FlagBinder(
      binder().bind(FlagsClassX.class));
 
// By asking for Provider<SuedeJersey>
//   instead of calling new SuedeJersey
//   you leave the SuedeJersey to be free
//   to ask for its dependencies.
@Provides
Jersey getJersey(
     Provider<SuedeJersey> suedeJerseyProvider,
     Provider<NylonJersey> nylonJerseyProvider,
     @Named('isSuedeJersey') suede) {
  if (sued) {
    return suedeJerseyProvider.get();
  } else {
    return nylonJerseyProvider.get();
  }
}
```

- Flaws:
    - 플래그를 직접 읽는 것은 값을 얻기 위해 전역 상태를 사용하는 것이다. 전역 상태가 분리(isolate)되지 않아서 악영향이 생긴다. 이전 테스트나 동시에 수행되는 다른 쓰레드가 예상하지 않은 상태로 설정할 수 있기 때문이다.
    - 플래그 값에 따라 다른 타입의 Jersey를 직접 생성하고 있다. CurlingTeamMember를 생성하는 테스트는 다른 Jersey collaborator를 주입할 seam을 갖지 못한다.
    - CurlingTeamMember의 책임이 광범위하다.
    
###### Moving the Constructor's "work" into an Initialize Method

- Before: Hard to Test
	- SUT
    
```java    
// With statics, singletons, and a tricky
//   initialize method this class is brittle.
 
class VisualVoicemail {
  User user;
  List<Call> calls;
 
  @Inject
  VisualVoicemail(User user) {
    // Look at me, aren't you proud? I've got
    // an easy constructor, and I use Guice
    this.user = user;
  }
 
  initialize() {
     Server.readConfigFromFile();
     Server server = Server.getSingleton();
     calls = server.getCallsFor(user);
  }
 
  // This was tricky, but I think I figured
  // out how to make this testable!
  @VisibleForTesting
  void setCalls(List<Call> calls) {
    this.calls = calls;
  }
 
  // ...
}
```

	- Test
    
```java    
// Brittle code exposed through the test
 
class VisualVoicemailTest extends TestCase {
 
  public void testExposesBrittleDesign() {
    User dummyUser = new DummyUser();
    VisualVoicemail voicemail =
        new VisualVoicemail(dummyUser);
    voicemail.setCalls(buildListOfTestCalls());
 
    // Technically this can be tested, as long
    //   as you don't need the Server to have
    //   read the config file. But testing
    //   without testing the initialize()
    //   excludes important behavior.
 
    // Also, the code is brittle and hard to
    //   later on add new functionalities.
  }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java    
// Using DI and Guice, this is a
//   superior design.
 
class VisualVoicemail {
  List<Call> calls;
 
  VisualVoicemail(List<Call> calls) {
    this.calls = calls;
  }
}
 
// You'll need a provider to get the calls
@Provides
List<Call> getCalls(Server server,
    @RequestScoped User user) {
  return server.getCallsFor(user);
}
 
// And a provider for the Server. Guice will
//  let you get rid of the JVM Singleton too.
@Provides @Singleton
Server getServer(ServerConfig config) {
  return new Server(config);
}
 
@Provides @Singleton
ServerConfig getServerConfig(
    @Named("serverConfigPath") path) {
  return new ServerConfig(new File(path));
}
 
// Somewhere, in your Module's configure()
//   use the FlagBinder.
  new FlagBinder(binder().bind(
      FlagClassX.class))
```      

	- Test

```java
// Dependency Injection exposes your
//   dependencies and allows for seams to
//   inject different collaborators.
 
class VisualVoicemailTest extends TestCase {
 
  VisualVoicemail voicemail =
      new VisualVoicemail(
          buildListOfTestCalls());
 
  // ... now you can test this however you want.
}
```

- “work”를 initialize 메소드로 이동시키는 것은 해결책이 아니다. 객체가 하나의 책임만 갖도록 decouple해야 한다(이때 하나의 책임은 완전하게 설정된 객체 그래프를 제공하는 것이다).
- Flaws:
    - 코드의 안정성이 없고 몇개의 statitic initialization 호출에 결부되어 있다.
    - initialization 메소드는 객체가 너무 많은 책임을 갖는다는 것을 나타내는 현격한 증거이다: 의존성 initialization은 다른 클래스에서 수행되어야 하고, 바로 사용할 수 있는 객체들이 constructor에 전달되어야 한다.
    - 테스트시 initialize 메소드를 호출하고자 한다면 Server.readConfigFromFile 메소드는 intercept 불가하다.
    - 테스트시 Server는 initialize 불가하다. Server를 사용하고자 한다면 전역 singleton 상태에서 얻어와야 한다. 2개의 테스트가 동시에 수행되거나 이전 테스트가 Server를 예상하지 않은 상태로 초기화했다면 전역 상태로 인해 테스트가 실패한다.
    
###### Having Multiple Constructors, where one is Just for Testing

- Before: Hard to Test
	- SUT
    
```java    
// Half way easy to construct. The other half
//   expensive to construct. And for collaborators
//   that use the expensive constructor - they
//   become expensive as well.
 
class VideoPlaylistIndex {
  VideoRepository repo;
 
  @VisibleForTesting
  VideoPlaylistIndex(
      VideoRepository repo) {
    // Look at me, aren't you proud?
    // An easy constructor for testing!
    this.repo = repo;
  }
 
  VideoPlaylistIndex() {
    this.repo = new FullLibraryIndex();
  }
 
  // ...
 
}
 
// And a collaborator, that is expensive to build
//   because the hard coded index construction.
 
class PlaylistGenerator {
 
  VideoPlaylistIndex index =
      new VideoPlaylistIndex();
 
  Playlist buildPlaylist(Query q) {
    return index.search(q);
  }
}
```

	- Test
    
```java    
// Testing the VideoPlaylistIndex is easy,
//  but testing the PlaylistGenerator is not!
 
class PlaylistGeneratorTest extends TestCase {
 
  public void testBadDesignHasNoSeams() {
    PlaylistGenerator generator =
        new PlaylistGenerator();
    // Doh! Now we're tied to the
    //   VideoPlaylistIndex with the bulky
    //   FullLibraryIndex that will make slow
    //   tests.
  }
}
```

- After: Testable and Flexible Design
	- SUT
    
```java    
// Easy to construct, and no other objects are
//   harmed by using an expensive constructor.
 
class VideoPlaylistIndex {
  VideoRepository repo;
 
  VideoPlaylistIndex(
      VideoRepository repo) {
    // One constructor to rule them all
    this.repo = repo;
  }
}
 
// And a collaborator, that is now easy to
//   build.
 
class PlaylistGenerator {
  VideoPlaylistIndex index;
 
  // pass in with manual DI
  PlaylistGenerator(
      VideoPlaylistIndex index) {
    this.index = index;
  } 
 
  Playlist buildPlaylist(Query q) {
    return index.search(q);
  }
}
```

	- Test
    
```java    
// Easy to test when Dependency Injection
//   is used everywhere. 
 
class PlaylistGeneratorTest extends TestCase {
 
  public void testFlexibleDesignWithDI() {
    VideoPlaylistIndex fakeIndex =
        new InMemoryVideoPlaylistIndex()
    PlaylistGenerator generator =
        new PlaylistGenerator(fakeIndex);
 
    // Success! The generator does not care
    //   about the index used during testing
    //   so a fakeIndex is passed in.
  }
}
```

- 다중 constructor(일부는 테스트에서만 사용되는)는 코드의 일부가 여전히 테스트하기 어렵다는 것을 나타낸다. VideoRepository를 인자로 갖는 Constructor로 인해 VideoPlaylistIndex는 테스트하기 쉽다. 하지만 인자가 없는 constructor로 인해 테스트가 어려워진다.
- Flaws:
    - PlaylistGenerator는 테스트하기 어렵다. VideoPlaylistIndex의 디폴트 constructor를 사용하도록 하드코딩되어 있기 때문이다(FullLibraryIndex가 사용됨). PlaylistGenerator를 테스트할 때 FullLibraryIndex에 대한 테스트는 배제하고 싶어도 그럴 수 없다.
    - 이상적으로는 PlaylistGenerator의 constructor가 VideoPlaylistIndex를 직접 생성하는 대신 VideoPlaylistIndex에 요청해야 한다. PlaylistGenerator에서 VideoPlaylistIndex의 디폴트 constructor 사용을 없애면 VideoPlaylistIndex의 디폴트 constructor를 제거할 수 있다. 대개의 경우 다중 constructor는 불필요하다.

##  Flaw #2: Digging into Collaborators

[원문](http://misko.hevery.com/code-reviewers-guide/flaw-digging-into-collaborators)

- “holder”, “context”, “kitchen sink” 같은 객체들(모두 메소드에서 직접적으로 필요한 Specific Object에 대한 접근을 제공하는 grab bag 역할)의 사용을 피하라.
- method에서 직접적으로 사용할 필요가 있는 객체는 method나 constructor의 parameter로 전달하라.
- User 객체(Holder)가 있고, 메소드에서 Address 객체(Specific Object)가 직접적으로 필요할 때
	- Holder(User)를 전달하여 Specific Object를 얻지 말고(User#getAddress() 호출을 통해), 직접적으로 필요한 Specific Object를 method나 constructor의 parameter로 전달하라.

#### Warning Signs

- “Train Wreck” or a “Law of Demeter” violation
- 전달된 객체가 직접적으로 사용되지 않은 경우(다른 객체를 얻기 위해서만 사용되는 경우) – Holder
- 테스트에서 mock을 반환하는 mock을 생성해야 하는 경우
- Law of Demeter violation: 하나 이상의 도트를 이용하여 객체 그래프를 탐색하여 메소드 호출 체인이 발생하는 경우
- 의심스러운 이름들이 사용되는 경우. eg. context, environment, principal, container, or manager 등
- fixture setup이 너무 복잡해서 테스트를 작성하기 어려운 경우

#### Why this is a Flaw

- Deceitful API
    - 카드를 통한 결재의 경우를 예로 들어보자.
    - 메소드 시그니처에서는 단순한 문자열이 카드번호가 필요하다고 되어 있다.
    - 메소드 바디에서는 CardProcessor나 PaymentGateway 등 실제로 사용할 객체를 구해서 카드번호를 이용한 결재를 해야 한다.
    - 메소드 시그니처에 실제 의존성(String이 아니라 CardProcessor나 PaymentGateway)의 표현되지 않는다.
- Makes for Brittle Code
    - Holder를 사용하는 경우 변경이 요구될 때 새로운 상호작용 처리를 위해 모든 Holder를 수정해야 한다.
    - 또한 코드가 점진적으로 Holder와 같은 Intermediary에 종속적으로 되어 보다 복잡해 진다.
    - Specific Object를 사용한다면 코드의 안정성을 높일 수 있다.
    - 이 경우도 하나 이상의 책임을 갖는 class를 분리하는 경우는 발생할 수 있다.
    - 하지만 걱정하지 말고 반드시 SRP(Single Reponsibility Principle) 고수하라.
- Bloats your code and complicates what's really happening
    - 실제로 사용할 객체를 Holder와 같은 intermediary를 통해 얻는 과정을 불필요하게 코드에 유지함으로써 코드의 길이/혼란이 증가한다.
- Hard for Testing
    - Holder를 사용하는 메소드를 테스트할 때 해당 메소드가 Holder에서 무엇을 요구할 지 추측하기 어렵다(무엇은 상관없을지도 추측이 어렵다).
    - 다시 말해 empty holder를 넘긴 후 NPE(NullPointerException)를 발생시키고, NPE가 발생하지 않도록 Holder의 상태를 처리하고 다시 시도하는 지루한 반복 방식이 된다.

#### Recognizing the Flaw

- 이 문제는 “Train Wreck”, “Law of Demeter” 위반으로도 알려져있다.
- 아래와 같은 증상으로 식별할 수 있다.
    - 테스트 작성시 mock을 반환하는 mock을 생성해야 하는 경우
    - “context”라는 이름을 가진 객체가 있는 경우
    - 둘 이상의 도트가 method chaining에 발생하고 해당 method가 getters인 경우
    - 복잡한 fixture setup 때문에 테스트를 작성하기 어려운 경우

#### Fixing the Flaw

- 필요한 객체를 얻으려 하지 말고 실제로 필요한 객체를 method나 constructor의 parameter로 전달하라.
- 아래 원칙을 고수하라:
    - 반드시 1촌 친구(immediate friends)와만 상호작용하라.
    - 진짜로 필요한 객체만 constructor에 inject하거나 method에 paramter로 전달하라.
    - 객체를 찾거나 설정하는 책임은 factory나 DI Container(spring)에 위임하라.

#### Examples by Diagrams

###### 잘못된 예

![](https://api.monosnap.com/rpc/file/download?id=U8eGba7yGhdyBhhgYmOdK87btGMyHE)

###### 잘된 예

![](https://api.monosnap.com/rpc/file/download?id=nhwqm6ePSnBqVSySUZe6TKlEo3RPMY)


#### Examples by Codes

###### Service Object Digging Around in Value Object

- Before: Hard to Test
	- SUT

```java
// This is a service object that works with a value
// object (the User and amount). 
 
class SalesTaxCalculator {
    TaxTable taxTable;
 
    SalesTaxCalculator(TaxTable taxTable) {
        this.taxTable = taxTable;
    }
 
    float computeSalesTax(User user, Invoice invoice) {
        // note that "user" is never used directly
        Address address = user.getAddress(); // holder
        float amount = invoice.getSubTotal(); // holder
        return amount * taxTable.getTaxRate(address);
    }
}
```

	- Test
    
```java    
// Testing exposes the problem by the amount of work
//   necessary to build the object graph, and test the
//   small behavior you are interested in.
 
class SalesTaxCalculatorTest extends TestCase {
    SalesTaxCalculator calc = new SalesTaxCalculator(new TaxTable());
    // So much work wiring together all the objects needed
    Address address = new Address("1600 Amphitheatre Parkway...");
    User user = new User(address);
    Invoice invoice = new Invoice(1, new ProductX(95.00));
    // ...
    assertEquals(0.09, calc.computeSalesTax(user, invoice), 0.05);
}
```

- After: Testable and Flexible Design
	- SUT
    
```java    
// Reworked, it only asks for the specific objects
// that it needs to collaborate with.
 
class SalesTaxCalculator {
    TaxTable taxTable;
    SalesTaxCalculator(TaxTable taxTable) {
        this.taxTable = taxTable;
    }
 
    // Note that we no longer use User, nor do we dig inside
    // the address. (Note: We would use a Money, BigDecimal,
    // etc. in reality).
    float computeSalesTax(Address address, float amount) {
        return amount * taxTable.getTaxRate(address);
    }
}
```

	- Test
    
```java    
// The new API is clearer in what collaborators it needs.
 
class SalesTaxCalculatorTest extends TestCase {
    SalesTaxCalculator calc = new SalesTaxCalculator(new TaxTable());
    // Only wire together the objects that are needed
    Address address = new Address("1600 Amphitheatre Parkway...");
    // ...
    assertEquals(0.09, calc.computeSalesTax(address, 95.00), 0.05);
    }
}
```

- 위 예제는 calcuation(business logic)과 object lookup이 혼합되어 있다. 실제 클래스의 역할/책임은 세금을 계산하는 것이다.
- Flaws:
    - 테스트시 User, Invoice 객체 생성이 불필요하게 요구된다.
    - 메소드 사용자의 입장에서 메소드 시그니처는 거짓말(deceitful)을 하고 있다. 실제 필요한 것은 주소와 금액인데, API는 User와 Invoice라고 알리고 있다.
    - 재사용을 하게 된다면 Invoice, User와 같이 불필요한 클래스가 신규 소스에 필요하게 된다(의존성(Dependency)이 증가하게 된다).
    
###### Service Object Directly Violating "Law of Demeter"

- Before: Hard to Test
	- SUT
    
```java    
// This is a service object which violates the
//   Law of Demeter. 
 
class LoginPage {
    RPCClient client;
    HttpRequest request;
 
    LoginPage(RPCClient client, HttpServletRequest request) {
        this.client = client;
        this.request = request;
    }
 
    boolean login() {
        String cookie = request.getCookie();
        return client.getAuthenticator().authenticate(cookie);
    }
}
```

	- Test
    
```java    
// The extensive and complicated easy mock usage is
//   a clue that the design is brittle.
 
class LoginPageTest extends TestCase {
    public void testTooComplicatedThanItNeedsToBe() {
        Authenticator authenticator = new FakeAuthenticator();
        IMocksControl control = EasyMock.createControl();
        RPCClient client = control.createMock(RPCClient.class);
        EasyMock.expect(client.getAuthenticator()).andReturn(authenticator);
        HttpServletRequest request = control.createMock(HttpServletRequest.class);
        Cookie[] cookies = new Cookie[]{new Cookie("g", "xyz123")};
        EasyMock.expect(request.getCookies()).andReturn(cookies);
        control.replay();
 
        LoginPage page = new LoginPage(client, request);
        // ...
        assertTrue(page.login());
        control.verify();
}
```

- Ater: Testable and Flexible Design
	- SUT
    
```java    
// The specific object we need is passed in
//   directly.
 
class LoginPage {
    LoginPage(@Cookie String cookie, Authenticator authenticator) {
        this.cookie = cookie;
        this.authenticator = authenticator;
    }
 
    boolean login() {
        return authenticator.authenticate(cookie);
    }
}
```

	- Test
    
```java    
// Things now have a looser coupling, and are more
//   maintainable, flexible, and testable.
 
class LoginPageTest extends TestCase {
    public void testMuchEasier() {
        Cookie cookie = new Cookie("g", "xyz123");
        Authenticator authenticator = new FakeAuthenticator();
        LoginPage page = new LoginPage(cookie, authenticator);
        // ...
        assertTrue(page.login());
    }
}
```

- Flaws:
    - RPCClient는 직접 사용되지 않는다. 왜 전달되었는가 ?
    - HttpRequest는 직접 사용되지 않는다. 왜 전달되었는가 ?
    - Cookie가 직접 필요한 객체인데, HttpRequest로 부터 얻어야 한다. HttpRequest를 테스트에서 설정하는 것은 성가신 일이다.
    - Authenticator가 직접 필요한 객체인데 RPCClient로 부터 얻어오고 있다.
    
###### Law of Demeter Violated to Inappropriately make a Service Locator

- Before: Hard to Test
	- SUT

```java
// Database has an single responsibility identity
//   crisis.
 
class UpdateBug {
    Database db;
 
    UpdateBug(Database db) {
        this.db = db;
    }
 
    void execute(Bug bug) {
        // Digging around violating Law of Demeter
        db.getLock().acquire();
        try {
            db.save(bug);
        } finally {
            db.getLock().release();
        }
    }
}
```

	- Test
    
```java
// Testing even the happy path is complicated with all
// the mock objects that are needed. Especially
// mocks that take mocks (very bad).
 
class UpdateBugTest extends TestCase {
 
    public void testThisIsRidiculousHappyPath() {
        Bug bug = new Bug("description");
 
        // This both violates Law of Demeter and abuses
        //   mocks, where mocks aren't entirely needed.
        IMocksControl control = EasyMock.createControl();
        Database db = control.createMock(Database.class);
        Lock lock = control.createMock(Lock.class);
 
        // Yikes, this mock (db) returns another mock.
        EasyMock.expect(db.getLock()).andReturn(lock);
        lock.acquire();
        db.save(bug);
        EasyMock.expect(db.getLock()).andReturn(lock);
        lock.release();
        control.replay();
        // Now we're done setting up mocks, finally!
 
        UpdateBug updateBug = new UpdateBug(db);
        updateBug.execute(bug);
        // Verify it happened as expected
        control.verify();
        // Note: another test with multiple execute
        //   attempts would need to assert the specific
        //   locking behavior is as we expect.
    }
}
```

- Ater: Testable and Flexible Design
	- SUT
    
```java    
// The revised Database has a Single Responsibility.
 
class UpdateBug {
    Database db;
    Lock lock;
 
    UpdateBug(Database db, Lock lock) {
        this.db = db;
    }
 
    void execute(Bug bug) {
        // the db no longer has a getLock method
        lock.acquire();
        try {
            db.save(bug);
        } finally {
            lock.release();
        }
    }
}
// Note: In Database, the getLock() method was removed
```

	- Test
    
```java    
// Two improved solutions: State Based Testing
//   and Behavior Based (Mockist) Testing.
 
// First Sol'n, as State Based Testing.
class UpdateBugStateBasedTest extends TestCase {
    public void testThisIsMoreElegantStateBased() {
        Bug bug = new Bug("description");
 
        // Use our in memory version instead of a mock
        InMemoryDatabase db = new InMemoryDatabase();
        Lock lock = new Lock();
        UpdateBug updateBug = new UpdateBug(db, lock);
 
        // Utilize State testing on the in memory db.
        assertEquals(bug, db.getLastSaved());
    }
}
 
// Second Sol'n, as Behavior Based Testing.
//   (using mocks).
class UpdateBugMockistTest extends TestCase {
 
    public void testBehaviorBasedTestingMockStyle() {
        Bug bug = new Bug("description");
 
        IMocksControl control = EasyMock.createControl();
        Database db = control.createMock(Database.class);
        Lock lock = control.createMock(Lock.class);
        lock.acquire();
        db.save(bug);
        lock.release();
        control.replay();
        // Two lines less for setting up mocks.
 
        UpdateBug updateBug = new UpdateBug(db, lock);
        updateBug.execute(bug);
        // Verify it happened as expected
        control.verify();
    }
}
```

- 클래스는 다른 객체에 대한 ServiceLocator 역할을 하지 말고, 하나의 책임을 가져야 한다.
- Flaws:
    - db.getLock()은 Database 클래스의 역할이 아니다. 또 db.getLock().acquire(), db.getLock().release()는 “Law of Demeter”를 위반하고 있다.
    - UpdateBag 클래스를 테스트할 때 Database#getLock 메소들를 mock으로 처리해야 한다.
	- Database 클래스는 database로도 동작하고, service locator(lock을 제공)로도 동작한다. “Law of Demeter”도 위반하고, service locator로도 동작하고 있어서 심각한 문제를 가지고 있다. Database 클래스의 역할은 entity들을 database에 저장하는 것이다.
- Database 클래스의 getLock 메소드는 제거되어야 한다.
    - Database가 lock에 대한 참조가 필요하더라도 제거되어야 한다.
    - You should never have to mock out a setter or getter
    
###### Object Called "Context" is a Great Big Hint to look for a Violation

- Before: Hard to Test
	- SUT
    
```java    
// Context objects can be a java.util.Map or some
//   custom grab bag of stuff.
 
class MembershipPlan {
 
    void processOrder(UserContext userContext) {
        User user = userContext.getUser();
        PlanLevel level = userContext.getLevel();
        Order order = userContext.getOrder();
 
        // ... process
    }
}
```

	- Test
    
```java    
// An example test method working against a
//   wretched context object.
 
    public void testWithContextMakesMeVomit() {
        MembershipPlan plan = new MembershipPlan();
        UserContext userContext = new UserContext();
        userContext.setUser(new User("Kim"));
        PlanLevel level = new PlanLevel(143, "yearly");
        userContext.setLevel(level);
        Order order = new Order("SuperDeluxe", 100, true);
        userContext.setOrder(order);
 
        plan.processOrder(userContext);
 
        // Then make assertions against the user, etc ...
}
```

- Ater: Testable and Flexible Design
	- SUT
    
```java    
// Replace context with the specific parameters that
//   are needed within the method.
 
class MembershipPlan {
 
    void processOrder(User user, PlanLevel level, Order order) { 
        // ... process
    }
}
```

	- Test

```java
// The new design is simpler and will easily evolve.
 
    public void testWithHonestApiDeclaringWhatItNeeds() {
        MembershipPlan plan = new MembershipPlan();
        User user = new User("Kim");
        PlanLevel level = new PlanLevel(143, "yearly");
        Order order = new Order("SuperDeluxe", 100, true);
 
        plan.processOrder(user, level, order);
 
        // Then make assertions against the user, etc ...
    }
}
```

- context 객체는 이론상 괜찮아 보인다(context 객체에 어떤 속성이 추가되어도 context 객체를 사용하는 클래스의 시그니처가 변경될 필요가 없다).
- 하지만 context 객체는 테스트하기 매우 어렵다.
- search에 사용된 map이 이 경우일 듯…
- Flaws:
    - API는 테스트를 위해 필요한 것인 userContext가 전부라고 표현한다. 하지만 테스트 작성자는 실제 userContext에 어떤 값들이 들어 있어야 하는지 알 수 없다. 이런 경우 필요할 것 같은 값을 채워가며 정상적인 결과가 나올 때까지 반복 작업을 해야 한다.
    - API가 flexible(메소드 시그니처 변경 없이 파라미터를 추가할 수 있다)하다고 할 수도 있다. 하지만 refactoring tool을 사용할 수 없이 코드가 깨지기 쉽고, 사용자가 어떤 파라미터가 필요한지 알수 없다는 문제를 갖는다. API만 보고는 어떤 Collaborator가 필요한지 알 수 없다. 이러한 API는 프로젝트의 신규 멤버가 클래스의 목적/역할/행위를 이해하기 어렵게 만든다. 이럴 경우 API가 의존성에 대해 거짓말하고 있다고 한다.

###### When this is not a Flaw

- fluent style을 사용하여 DSL(Domain Specific Language)에서 설정을 하는 경우
- 이 경우 value object(항상 새로 만들어지는)를 생성하는 것이기 때문에 문제가 안된다.

```java
// A DSL may be an acceptable violation.
//   i.e. in a GUICE Module's configure method
    bind(Some.class)
        .annotatedWith(Annotation.class)
        .to(SomeImplementaion.class)
        .in(SomeScope.class);
```

## Flaw #3: Brittle Global State & Singletons

[원문](http://misko.hevery.com/code-reviewers-guide/flaw-brittle-global-state-singletons)

#### Warning Signs

- singleton을 사용하거나 추가하는 경우
- static field나 static method를 사용하거나 추가하는 경우
- static initialization 블록을 사용하거나 추가하는 경우
- registry를 사용하거나 추가하는 경우
- service locator를 사용하거나 추가하는 경우

## Flaw #4: Class Does Too Much

[원문](http://misko.hevery.com/code-reviewers-guide/flaw-class-does-too-much)

#### Warning Signs

- 클래스가 무엇을 하는지를 종합해서 표현할 때 “and”가 포함되는 경우
- 새로운 멤버가 클래스를 읽고, 이해하는 것이 도전적인(어려운) 일인 경우
- 클래스가 몇몇 메소드에 의해서만 사용되는 필드를 갖는 경우
- 클래스에 파라미터만 사용하는 static method가 있는 경우