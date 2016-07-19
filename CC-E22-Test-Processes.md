# Test Process: Simple Techniques

이 문서에서는 [Clean Coders Episode 22](https://cleancoders.com/episode/clean-code-episode-22/show)의 내용을 정리한 것으로 다음과 같은 TDD의 테크닉들을 소개한다.

1. Fake it till you make it
2. Stairstep Tests
3. Assert First
4. Triangulation
5. One To Many
6. Refactoring Tests

## 1. Fake it till you make it
```
public class StackTest {
	@Test
	public void uponCreation_stackWillBeEmpty() {
		Stack stack = new Stack();
		assertThat(stack.isEmpty(), is(true));
	}
}

public class Stack {
	public boolean isEmpty() {
		return false;
# 	}
}
```

에서 isEmpty를 호출하면 return false;여서 테스트가 실패한다. 이때 return true;로 변경해서 동작하도록 한다.

![](https://api.monosnap.com/rpc/file/download?id=evnTXBhrRYTPg1FpGGqH5oQ9GzSlno)

테스트를 성공시키기 위해 fake했다. 실제로 로직을 구현한 것이 아니라 상수를 반환하여 테스트가 성공하도록 했다는 의미이다.

fake it은 null, 0, 1, true, false, empty list 등을 반환하는 방법으로 테스트를 성공시키는 기법이다.

이러한 기법을 **fake it till you make it**이라고 한다. **반드시 구현을 해야만 하는 상황이 발생할 때까지는 최대한 fake함으로써 테스트를 성공시키라는 의미**이다.

**예측되면 하지마라. 반드시 할 필요가 있을때 그때 해라... <-- 애자일**

### 1.1 Incrementalism

faking의 목적은 incrementalism이다. 테스트 케이스를 통과시키기 위해 fake(가능한 가장 단순한 방법으로 테스트가 성공하도록)한다. 그 다음 다른 테스트 케이스를 통과시키기 위해 조금 덜 fake한다. 계속 테스트 케이스를 조금 덜 fake하면서 추가하다가 전혀 fake하지 않고 테스트 케이스를 추가할때까지 반복한다.

목적을 이루기 위해 점진적으로 진행하는 것은 시간 낭비가 아니라 **시간을 절약**하는 것이다. 더 빨리 개발하는 것이다. 조금씩 진행하는 것이 한번에 크게 진행하는 것보다 항상 빠른 방법이다.

### 1.2 Getting Stuck

어떤 경우에 fake it은 동작하지 않는다. 

[wordwrap 예제](https://github.com/msbaek/wordwrap)에서 getting stuck된 상황을 봐라. 어떤 경우에는 테스트를 작성했는데 fake로 성공시킬 수 없고, 테스트를 성공시키기 위해서는 그 즉시 모든 알고리즘을 구현해야 하는 경우가 있다. 이런 경우 getting stucking(더 이상 나아갈 수 없는 상태)된 것이다. 이때의 해결책은 **"write a simpler test"**이다.

### 1.3 다음 수를 고려한 플레이

테스트를 작성하는 것은 당구를 치는 것과 같다. 좋은 당구 선수는 매 큐를 다음 큐를 생각하며 경기를 한다. 당신이 작성하는 **모든 테스트는 fake될 수 있는 여유를 남겨야 한다**. 더 이상 fake할 수 없을때 까지...테스트를 잘 작성하는 사람은 한번에 모든 것을 구현하려고 하지 않고 다음 테스트 케이스에서 fake하기 위한 여유 공간을 남겨두려고 한다.

이것은 **테스트를 작고, 간단한 설계를 갖도록 유지하는 것**을 의미한다. **당신은 생각할 수 있는 가장 복잡한 테스트를 작성함으로써 당신이 똑똑하다는 것을 세상에 알리려 하면 안된다**. 그 대신, 더 이상 테스트할 것이 없을 때까지 복잡한 테스트 작성을 의도적으로 금해야 한다. "engaing as few brain cells as possible at any given moment"라고 표현한다.

다르게 말하면 어떤 함수를 테스트할 때 **outside에서 inside로 접근**해야 한다는 것이다. 이 말은 간단하고 적절한(simpler proper) 테스트 케이스를 먼저 추가하라는 것이다(ex. validating argument, simple queries). 이런 simpler proper 이슈들이 모두 해소되면 보다 복잡한 inner working을 언급할 수 있다.

이렇게 테스트를 추가해 나가면 **당신이 정말로 중요한 inner working을 할 때 그 deep inner working이 이미 존재**하는 것을 발견하는 경우가 종종 있을 것이다. 복잡한 테스트 케이스를 추가했는데 저절로 테스트가 성공하는...(bowling game, wordwrap의 마지막 테스트와 같은)

## 2. Stairstep Tests

### 2.1 볼링 게임 예제

 - 제일 처음에 Game 클래스를 생성하기 위한 canCreateGame 테스트를 추가하다.
 	- Game 클래스를 생성하는 대신... 테스트가 있어야만 코드를 작성할 수 있다.
 
    ```
    @Test
    public void canCreateGame() {
      Game g = new Game();
    }
    ```

 - 그 다음 roll 메소드를 추가하기 위한 canRoll 테스트를 추가한다.
 
    ```
    @Test
    public void canRoll() {
        Game g = new Game();
        g.roll(0);
    }
    ```
    
 - 여기까지 구현하면 `Game game = new Game();`이 2개의 테스트 케이스에서 **중복**된다. game을 필드로 추출하고 setup 메소드에서 초기화하도록 rafctoring한다.
 
    ```
    Game g;

	@Before
    public void setup() {
        g = new Game();
    }
    
    @Test
    public void canCreateGame() {
    }

    @Test
    public void canRoll() {
        g.roll(0);
    }
    ```
    
 - 그러면 canCreateGame은 empty body가 되어 불필요해지므로 삭제한다.
 - gutterGame 테스트 케이스를 추가한다. score 구현시 디폴트 0대신 -1을 반환하여 실패하는지 확인하고, 후에 0으로 변경하여 성공시킨다(faking it. 성공만 확인하는 것이 아니라 실패도 확인).
 
    ```
    @Test
    public void gutterGame() {
        for(int i = 0; i < 20; i++)
            game.roll(0);
        assertThat(game.score(), is(0));
    } 
    ```

 - refactoring: canRoll이 불필요해짐. 왜냐면 gutterGame에서 roll을 호출하니... 이런게 stairstep test이다.

### 2.2 stairstep test

**테스트의 유일한 존재 목적이 다음 테스트를 순차적으로 구현하기 위함인 경우**(다음 테스트를 구현한 후에는 stairstep test는 삭제된다).

stairstep test는 임시적인 중간 단계 코드로 의미를 갖는다.

## 3 Assert First

테스트 작성시 assert부터 즉 거꾸로 작성하는 것이다.

### 3.1. assert

```
public class MailTest {
	@Test public void newMailBox_isEmpty() {
		assertThat(mailbox.messageCount(), is(0));
	}
}
```

### 3.2. mailbox를 local variable로 선언(quick fix를 이용)

```
public class MailTest {
	@Test public void newMailBox_isEmpty() {
		MailBox mailbox = null;
		assertThat(mailbox.messageCount(), is(0));
	}
}
```

### 3.3. MailBox 클래스 생성

```
public class MainBox {
}
```

### 3.4. new MailBox()를 호출하여 uninitialized 이슈 제거
```
public class MailTest {
	@Test public void newMailBox_isEmpty() {
		MailBox mailbox = new MailBox();
		assertThat(mailbox.messageCount(), is(0));
	}
}
```

### 3.5. messageCount 메소드 선언
```
public class MainBox {
	public int messageCount() {
    	return 0;
    }
}
```

assert가 compile error를 유발하고, 또 execution error를 유발함으로써 코딩이 일어나도록 하는 것에 주의하라. 이렇게 함으로써 항상 test에서 시작할 수 있다.

**Test Drives** : test가 나를 운전하도록 !!!

**production 코드에 뭔가를 넣고 싶다면 먼저 그렇게 할 수 밖에 없도록 만드는 테스트 코드를 추가하라.** 이게 **test drive**이고, **agile, lean development**이다. 또 **Needs Driven Development**이다.

## 4. Triangulation

[Triangulation](http://msbaek.tumblr.com/post/74259407794/triangulation)

토목/수학에서 말하는 삼각법(물체간의 거리를 측정하는)이 아니라 TDD에서 삼각법은 **generalization을 만드는 방법**의 하나를 의미한다.

> **"As the tests get more specific, the code gets more GENERIC"**

하나의 테스트가 아니라 여러개의 테스트를 추가함으로써 문제와 해결책을 좀 더 명확히 하는 기법이다. 삼각법이 2개 이상의 지점의 위치를 이용하여 현 위치를 측정하는 것 처럼...

**하나의 테스트만 존재할 때는 fake할 수 있다**(상수를 반환함으로써). 하지만 상수로 처리 불가한 테스트를 추가하면(삼각법에서 2개 이상의 지점을 사용하는 것처럼) fake할 수 없게 된다.

서로 다른 계정을 다루는 뱅킹 어플리케이션을 작성하는 것을 가정해 보자. 일단은 **savings account(보통 예금)**, **money market account(금융시장 계정)**를 다루는...

이 요구사항은 **계층적**이다.

[junit-hierarchicalcontextrunner](https://github.com/bechte/junit-hierarchicalcontextrunner)를 이용해서 계층적 단위 테스트를 작성할 수 있다.

`testCompile group: 'de.bechte.junit', name: 'junit-hierarchicalcontextrunner', version: '4.12.1'`

### 4.1. assert 작성

![](https://api.monosnap.com/rpc/file/download?id=mYZ52pajTfHok5OwiJ250dwQ4P9X6K)

### 4.2. add field variable and class type

![](https://api.monosnap.com/rpc/file/download?id=xGRsWd7lzTNCCnEjwa2haMidtigQv8)

### 4.3. make it run
컴파일 오류만 없애서 실행해 보고, 실행 오류가 테스트 코드 작성을 유도하도록 한다.

![](https://api.monosnap.com/rpc/file/download?id=gTV1mIyPEdfLDCSLxN9Vj4ipsvneLT)

getInterestEarned가 3.0을 반환하도록 하여 테스트를 성공시킨다(fake).

![](https://api.monosnap.com/rpc/file/download?id=CPmcVawBZbsNSJIdF7kYS9kBDo7wXC)

지금까지는 매우 특별한 경우에 대한 것이였다. 구현이 매우 특별한 경우(interest가 3인 경우)에만 성공한다.
보다 제너릭하도록 triangulate해야 한다.

### 4.4. add other test case
accountWithBalanceOf100_earns3InInterest를 복사해서 새로운 테스트 케이스를 추가한다. 밸런스를 200이 되게하는...그래서 이자가 6.0이 되도록하는...

![](https://api.monosnap.com/rpc/file/download?id=0fV2MYxtCK0xsNXZm0vLrkbFrQjn3W)

이 테스트는 구현이 trivial(specific - 특별한 경우에만 동작)해서 실패한다. 이 테스트를 성공시키기 위해 untrivialize(generalizer)한다.

### 4.5. make it general
getInterestEarned()가 3.0이 아니라 balance * 0.03을 반환하도록 수정. 이게 구현을 generalize하고 테스트를 성공시킨다.

![](https://api.monosnap.com/rpc/file/download?id=vEf00GF8GSX7PlsLgg04juUTIN6X8p)

### 4.6. more triangulate
SavingsAccount외에 MoneyMarketAccount도 있다. 보다 triangulate해야 한다.

![](https://api.monosnap.com/rpc/file/download?id=1rb4l5pxWRG5gaiXkgMmRyiB0mVJqH)

### 4.7. make it work
MoenyMarketAccount#getInterestEarned가 return balance * 0.04로 변환

![](https://api.monosnap.com/rpc/file/download?id=zdJdX8KCDcyosTr6gbX4WuNacoBT0b)

### 4.8. refactoring
2개의 Account가 매우 똑같다.
intelliJ에서 Extract Superclass를 실행한다.

![](https://api.monosnap.com/rpc/file/download?id=jyUAfzTnbqF5ef0K2OiOUpKRwN5xIH)

balance를 선택하고 `Extract Method`를 하여 `getBalance` 메소드로 추출한다.

![](https://api.monosnap.com/rpc/file/download?id=lkyL0joYEiKi3s4zZDamuQKUlXZofJ)

`getInterestEarned`를 선택하고 `Push Members Down`을 실행한다. 이때 `Keep abstract`를 체크한다.

![](https://api.monosnap.com/rpc/file/download?id=fz2vHTYM4bBKJPQokyibYnZNNSv4V4)

`MoneyMarketAccount#getInterestEarned`에서 컴파일 오류가 발생하는 `getBalance()`를 선택하고, OPT+Enter를 클릭한 후 `Make Protected`를 선택하여 컴파일 오류를 해소하고, 테스트가 동작하도록 한다.

![](https://api.monosnap.com/rpc/file/download?id=8NXdfbthoumVW9vGvZtBRHjpr91RBf)

### 4.9. refactor SavingsAccount
SavingsAccount에 대해서는 extends Account를 추가하고 getInterestEarned를 제외한 나머지를 지운다. balance는 getBalance로 변경하고...

![](https://api.monosnap.com/rpc/file/download?id=jpkhIGb2YhfnYO7zhBT7l8oLLbXH1S)

### 4.10. refactor getInterestEarned()
Account#getInterestEarned를 abstract를 제거하고 아래와 같이 구현한다.

```
    public double getInterestEarned() {
        balance * getInterestRate();
    }

    protected abstract double getInterestRate();
```

IDE의 도움을 얻어 하위 클래스에서 구현한다(abstract method에서 opt+enter나 백열등 표시를 클릭).

![](https://api.monosnap.com/rpc/file/download?id=3zOFhiVRCXASO5V63TwRe8os1r88X5)

### 4.11. refactor test
테스트 코드에서 XXXContext에 있는 private SavingsAccount myAccount; private MoneyMarketAccount myAccount; 를 제거하고
BankAccountTest에 private Account myAccount;를 추가

![](https://api.monosnap.com/rpc/file/download?id=y5FQ2l4IBOQcFWGcq59H5CkVLeySWd)

이미 뭘 구현할지 알고 있는 경우에나 가능한 것 아니냐고 묻는다. 무슨 코드를 작성하게 만드는 코드를 작성했다. 맞다 어떤 경우에는 어떤 일반화를 할지 이미 알고 있으니 그것이 필요하도록 테스트를 작성할 수 있다.

## 5. One To Many

one to many practice는 리스트에 있는 많은 아이템을 다뤄야 하는 것을 알고 있더라도 하나의 아이템을 가지고 시작하는 것이 최상이라는 것이다. 

Kent Beck의 TDD by Example을 인용한다면, **"어떻게 객체의 컬렉션에 동작하는 오퍼레이션을 구현하겠는가 ? 컬렉션 없이 먼저 구현하고, 컬렉션에 대해서도 동작하도록 만들어라”**이라는 말로 설명할 수 있다.

스택을 가지고 설명해 알아보자.

스택은 push한 것을 pop해야 한다. 테스트는 아래와 같을 것이다.

![](https://api.monosnap.com/rpc/file/download?id=hbgT57Q9rG8O3uC0QNo8wuKVL2wsSA)

이 테스트를 성공시키는 가장 쉬운 방법은 pop이 99를 반환하는 것이다.

![](https://api.monosnap.com/rpc/file/download?id=9gcznaJIn65RsOFHmYj5WC3OkhvNnL)

> fake

이 해결책은 너무 specific하므로, 66에 대한 절차를 반복함으로써 삼각법을 적용한다.

![](https://api.monosnap.com/rpc/file/download?id=YjNHfRjoSIB0qo8ptXNUMOPkfPejR1)

이제 push된 것을 저장해야만 하는 상황이 되었다. 우리는 stack이 하나 이상의 값을 저장할 수 있다는 것을 안다. 따라서 바로 배열 등을 이용하고 싶을 것이다.  one-to-many 기법은 stack이 하나의 원소에 대해서만 잘 동작하도록 만들라고 조언한다.

![](https://api.monosnap.com/rpc/file/download?id=CtxEmRqKHcQWn3MxsMMyVZaq1YudNM)

이와 같이 하나의 원소를 저장하여 테스트를 성공시킨다.

> singular case에 대해서는 동작한다. 이게 One To Many의 첫번째 부분

테스트는 여전히 specific하다. 99, 66을 연속해서 push하고 역순으로 pop되도록 삼각법을 다시 적용한다.

> singular case를 먼저 성공시키고, plural case를 성공시키는 것이 One to Many를 실천하는 방법이다.

![](https://api.monosnap.com/rpc/file/download?id=mrH49VBIvCwEc2O7ctPyQo2haJiEKw)

아래와 같이 int를 int[] 변환하여 테스트를 성공시킨다.

![](https://api.monosnap.com/rpc/file/download?id=WuXMfm34a2ujJNyIbNlcZ7WBdBYpDd)

## 6. Refactoring Tests

> production 코드만 리팩토링하고 test 코드는 리팩토링하지 않으면 재앙에 빠진다.

처음엔 문제가 없지만 테스트 코드를 리팩토링하지 않고 방치하면 테스트 코드는 fragile, brittle해진다. production 코드에 간단한 변경만 가해도 많은 테스트가 깨지게되고, 테스트 코드는 점점 더 유지보수하기 어려워진다. 이렇게 **유지보수가 어려워지면 테스트 코드를 다 버리게된다**. 실제로 이런 경우를 겪었다고 한다. **테스트 코드를 버리게되면, 테스트 케이스들을 신뢰할 수 없게되고, 그럼 Production 코드를 리팩토링할 수 없게 된다. 그럼 코드가 섞어들어간다.**

> 그러니 "Refactor your tests"

테스트도 시스템의 일부다. 처분하거나, 버리거나 할 수 있는 것이 아니다. Production 코드와 동일한 수준으로 다뤄져야 한다. **심지어 테스트 코드가 리팩토링이 가능케하는 시발점이므로 Production 코드보다 더 소중하게 다뤄야한다.**

### 6.1 The Two Disks

예전에 애자일 관련 컨퍼런스에 논쟁이 되었던 것 중에 TDD에 관한 것이 있었다. 만일 테스트와 production이 서로 독립된 디스크에 저장되었는데 디스크가 깨졌다면 어떤 디스크가 안깨졌기를 원하는가 ?

Production 코드가 살아 남아 있기를 기대한다. Production 코드가 없어지면 큰 일 아닌가 ? 테스트 코드는 다시 작성하면 되는 것 아닌가 ? 소스 코드에서 테스트 코그를 만들어내는 것은 쉬운 일이 아니다. 이론적으로는 가능하나 실제적으로는 거의 불가하다.

> 테스트 코드로부터 소스 코드가 생성되었으므로 테스트 코드만 있다면 소스 코드는 다시 만들어 낼 수 있다.

"[트랩 도어 펑션](https://ko.wikipedia.org/wiki/%ED%8A%B8%EB%9E%A9%EB%8F%84%EC%96%B4_%ED%95%A8%EC%88%98)"이라고 한다. 한방향으로는 쉽게 evaluate할 수 있지만 다른 방향으로는 평가하기 매우 어려운 함수를 일컫는 말이다.

그래서 이 논쟁의 패널들은 소스 코드로 부터 테스트 코드를 재생해내는 것은 비현실적이라고 결정했다.

> 소스 코드가 저장된 디스크가 깨졌다면 많은 일을 해야 할 것이다. 하지만 현실적으로 가능한 일이다.

전체 어플리케이션을 다시 만들어야 하는데... 맞다. 논리적으로 테스트는 요구사항 명세이다. 그러므로 전체 어플리케이션을 다시 만드는 것은 점진적으로 한번에 한 테스트가 동작하도록 구현하면 된다. 게다가 동일 시스템을 두번 만들게 되어 얻는 잇점이 있다. **대부분의 경우 두번째 만든 시스템의 설계가 첫번째 시스템의 설계보다 우수하다**.

다시 말해서 프로덕션 코드를 잃으면 테스트 코드에 기반하여 다시 만들면 된다. 게다가 다시 만들면 대부분 더 나은 설계를 갖는다. 하지만 테스트 코드를 잃으면 프로덕션 코드로부터 테스트 코드를 만들어 낼 수가 없다.

테스트 코드가 없는 프로덕션 코드는 상한 음식처럼 썩어 들어간다.

테스트 코드는 프로덕션 코드보다 더 깨끗하게 유지해야 한다.

> 잘 작성된 테스트 코드는 잘 쓰여진 명세서처럼 읽혀져야 한다.

### 6.2 Tests are Specifications

기능 요구사항 등 다른 요구사항 명세는 쉽게 irrelevant해 진다.

시스템이 배포될 준비가 되었다는 것을 어떻게 알 수 있는가 ? 누가 서명하나 ? 배포담당자, PM... 그럼 PM은 어떻게 배포될 준비가 되었는지 알 수 있는가 ? QA가 모든 테스트 케이스를 수행하여 아무런 문제가 없을 때 PM이 서명한다. 그럼 QA는 어떻게 시스템이 배포 가능한지 알수 있는가 ? QA가 테스트를 수행하고 문제가 없으면 배포 가능하다고 한다. 그럼 테스트가 모두 성공하면 배포한다는 것인가 ? 그러니까 테스트가 요구사항이라는 것이다. QA가 검사해야 할 다른 문서가 있는가 ? 아니면 테스트 결과를 믿는가 ? QA는 테스트 결과를 믿어야 한다. 그러므로 테스트가 요구사항이고 명세서이다. 테스트가 성공하면 배포하고 실패하면 배포하지 않는다.

이런 결론에 도달할 수 있다. 심지어 TDD의 단위 테스트를 작성하는 것 조차도 실제로는 요구사항을 작성하는 것이라는 것이다. 요구사항을 작성하는 것이라면 당연히 요구사항으로 읽혀야 한다.

**Write the tests that you'd want to read**. - "Growing Object-Oriented Software, Guided by Tests" 이게 테스트를 작성하는 것데 대한 golden rule이다.

![](https://api.monosnap.com/rpc/file/download?id=LGXV3NPklJEv8bPFJeWy6BHyPmnC4c)

이런 코드를 읽고 싶은가. accidental complexity, details. 한번도 리팩토링 안 한 코드 같다. 아래와 같은 Extract Method를 3번만 적용하면.

![](https://api.monosnap.com/rpc/file/download?id=HZHucMrifjwQZrDHzNpR0Or2R2qMvB)

아래와 같이 읽을 수 있도록 깨끗해진다.

```
public void testDefaultJsonResponseForProperties() {
  createTestPage();
    
  SimpleResponse jsonResponse = requestPropertiesInJson();
    
  assertJsonPropertiesAreAllDefaults(jsonResponse);
}
```

Kent Beck은 "**Write to an Audience**"라고 말했다. 명확한 명세서를 읽고 싶은 독자를 대상으로...

스티브 베넷, *"Test Behavior Not APIs" API에 fit되도록 테스트를 작성하지 말라. API와 무관하게 시스템에 기대하는 것을 표현하도록 테스트를 작성하라. 사실은 테스트를 먼저 작성하므로 테스트가 기대하는 행위를 기반으로 API를 설계하게된다.

이 모든 것은 **커뮤니케이션**에 대한 것이다. SW Engineer의 가장 중요한 업무, 코드를 작성한 가장 중요한 목적은 커뮤니케이트하기 위한 것이다.

### 6.3 Test Come First

론 제프리의 Simple Design에 대한 4가지 규칙

- runs all the tests
- contains no duplicate code
- express all ideas the author wants to express
- minimized the number of classes & methods

Kent Beck은 이 규칙을 다르게 말했다.

- First, make it work
- Then, make it right
- Then, make it small and fast

이 규칙의 공통점은 **일단 돌아가게 만들고, 이쁘게(refactor. focus on: internal structure, illiminate duplication, Apply SOLID & Design Principles) 만드는 것**이다. 왜 한번에 안하고 ? 하지만 **한번에 2가지를 함께하는 것은 힘들다**.

Kent Beck이 오래 전에 언급에 가정: 모든 설계 원칙은 중복 제거에 근원한다. 중복 제거에 철저히 집중하면 자동적으로 모든 설계 원칙을 따르게 된다.

### 6.4 Heresy !

론 제프리의 Simple Design에 대한 4가지 규칙의 순서를 바꾸기를 원한다. 3번을 1번으로 옮겼다.

- express all ideas the author wants to express
- runs all the tests
- contains no duplicate code
- minimized the number of classes & methods

이게 **assert first**(writing the test backward)를 지키는 것이다.

> assert를 먼저 작성하는 것은 테스트의 결과로 시작하는 것을 의미한다. 그러면 읽기 쉽게 테스트를 작성하기 위한 모든 자유를 갖는 것을 의미한다.

Test First는 단지 테스트를 먼저 작성하는 것을 의미하는 것만은 아니다. **Test Come First**를 의미한다.

- 코드를 작성할 때 제일 먼저 오고
- 리팩토링할 때 제일 먼저 오고
- 코드를 깨끗하게 할 때 제일 먼저 오고
- 유지보수할 때 제일 먼저 오고
- 테스트는 모든 경우에 제일 먼저 온다.

이게 "Test First"의 의미이다. 모든 경우에 테스트가 가장 먼저 온다.