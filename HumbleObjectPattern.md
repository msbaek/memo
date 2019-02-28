# Humble Object Pattern

CC E23 Mocking Part2에 나옴

humble object pattern은 2가지 관점에서 바라 볼 수 있다.

1. H/W 등 구체적인 코드에 의존하는 어플리케이션 개발자
	- 내 로직에서 H/W 의존적인 부분을 Humble Interface를 통해 분리하고 내 로직은 Unit Test

2. 서버 로직에 의존하는 UI 개발자(App, Web Front)
	- GUI에서 Process Interface(Humble Interface)를 분리하고 Fake 구현체를 추가하고 
	- 내 로직(GUI)은 수동으로 테스트할 수 있는 만큼 Humble하게 유지


1의 경우는 H/W 등과 무관하게 자신의 코드를 테스트를 하면서 작성하고 싶을 것이고,

2의 경우는 서버 로직과 무관하게 자신의 코드를 테스트를 하면서 작성하고 싶을 것이다.

각각에 대해서 살펴보자.

## 1. H/W 등 구체적인 코드에 의존하는 어플리케이션 개발자

![Alt text](https://monosnap.com/image/pwbwVTo7qu7ub68oC4KnhqdnfHfHvs.png)

- Humble 오브젝트 패턴은 테스트 가능성을 높이기 위해 테스트하기 어려운 시스템의 경계에 적용
- 경계(HW, DB 등)에 종속된 클래스(Humble Object)의 로직을 테스트 할 필요 없을 정도로 최소화(humble하게 - Humble Controller)

![Alt text](https://monosnap.com/image/srfaTzu3I7zKXG9pXVafNQHougWciP.png)

- 경계에 종속된 클래스(Humble Controller)에서 추출된 중요한 비즈니스 로직(Controller Logic)은 경계와 decouple되고, 테스트 가능한 클래스(Business Object)로 이동
- Business Object는 Humbler Object를 Boundary Interface를 통해서 사용하고 Humble Object의 존재를 모르게 된다(**DIP**)
- Humble Object는 Boundary Interface를 구현한다.


> 이 말은 우리가 spring을 사용할 때 Controller, Dao, Repository 등의 구현체에서 경계(UI, DB)에 속한 로직들을 테스트할 필요가 없을 만큼 Humble하게 하여 Interface 뒤로 숨기고(Interface의 구현체가 되도록), 우리 코드는 이 Interface에 의존하고 테스트 가능하도록 해야 한다는 것을 의미한다. 테스트할 필요가 있는 코드가 있다면 테스트하기 어려운 코드와 Interface를 통해 분리하여 테스트 가능하도록 해야 한다.

### 1.1 Example

```java
package humbleobject.milker;

public class HumbleMilkPumpController {
    public void engage() {
        for (int speed=1; speed<=6; speed++) {
            setPumpSpeed(speed);
            sleep(500);
        }
        setPumpSpeed(7);
    }

    /**
     * hard to test code
     * because too much dependent on control board
     *
     * turn speed into binary digits(3 digits)
     * set speed in specific IO register
     */
    private void setPumpSpeed(int speed) {
        // set decimal to binary
        // get IO register from control board
        // set binary digits to IO register to control speed
    }

    private void sleep(int milliseconds) {
        try {
            Thread.sleep(milliseconds);
        } catch (InterruptedException e) { }
    }
}
```

- 유축기에서 게이트(젓소들이 유축장에 가두는)를 제어하는 pump가 컨트롤 보드에 의해 구동되는 경우 가정
- 컨트롤 보드와 협업하는 pump 관련 코드(HumbleMilkPumpController#setPumpSpeed)는 테스트하기 어렵다.
- HumbleMilkPumpController(컨트롤 보드와 협업하는 로직)에서 컨트롤 보드와 분리 가능한 코드를 추출하여 테스트 가능한 클래스(MilkPump)로 이동시키는 것
- 이 테스트 가능한 클래스(MilkPump)는 컨트롤 보드에 대해서 알지 못한다. 이런 작업 후에는 컨트롤러와 직접 통신하는 아주 작은 코드만이 남고, 테스트할 필요가 없게된다. HumbleController에 남은 코드는 테스트가 필요없는 직접 컨트롤러를 제어하는 최소한의 코드이다.

위 다이어그램에서 개발자에게 중요한 코드는 Controller Logic이다. 이 로직이 의존하고 있는 Humble Controller는 테스트, 개발에서 제외하고 싶다.

#### 1.1.1 Extract Boundary Interface

![Alt text](https://monosnap.com/image/aGLtADiCzNSWyjcZQrT0ivlv7ZZCOw.png)

- PumpRegister는 테스트하기 어려운 코드
- engage가 테스트하려는 알고리즘
- setPumpSpeed, sleep는 override 가능
- PumpRegister가 controll board를 제어(테스트하기 어려운 IO 작업)

이제 `HumbleMilkPumpController`는 더 이상 Humble하지 않으므로 `MilkPump`로 이름을 변경한다.

**테스트하기 어려운 코드(PumpRegister)와 알고리즘(engage)을 다음과 같은 절차로 분리했다.**

1. 테스트하기 어려운 코드(IO를 하는 코드여서)를 인터페이스(PumpRegister) 뒤로 숨긴다(테스트하기 어려운, 할 필요가 없을만큼 Humble한 코드를 Humble Interface의 구현체가 되도록한다)
2. 모든 rampUp 알고리즘(engage() 메소드)에서 사용하는 함수들을 MilkPump 클래스 내의 헬퍼 메소드로 추출했다(setPumpSpeed, sleep 등을 protected로 선언해서 test에서 override할 수 있도록).

```java
public class MilkPumpTest {
    private String actions = "";

    class TestingMilkPump extends MilkPump {
        @Override
        protected void setPumpSpeed(int speed) {
            actions += speed + "ss";
        }

        @Override
        protected void sleep(int milliseconds) {
            actions += milliseconds + "s";
        }
    }

    @Test
    public void engage() {
        TestingMilkPump milkPump = new TestingMilkPump();
        milkPump.engage();
        assertThat(actions, is("1ss500s2ss500s3ss500s4ss500s5ss500s6ss500s7ss"));
    }
}
```

이 패턴은

- testable code와 boundary를 넘어서 untestable한 code를 separate하고 decouple하여 DI를 통해 decoupling을 이룬다.
- 이를 통해 untestable code가 testable code에 의존하도록 DI를 적용하여 테스트 불가한 코드가 테스트 가능한 코드에 의존성을 갖도록한다.
- 테스트하기 어려운 어떠한 boundary에도 이 방법을 적용할 수 있다.

## 2. 서버 로직에 의존하는 UI 개발자(App, Web Front)

![Alt text](https://monosnap.com/image/ZbbHoGLMoMLIDzi8A2k9QGyZyMwWeW.png)

[LoginActivity.java](https://github.com/msbaek/androidmvp/blob/master/app/src/main/java/com/antonioleiva/mvpexample/app/Login/LoginActivity.java)

[LoginPresenterImpl.java](https://github.com/msbaek/androidmvp/blob/master/app/src/main/java/com/antonioleiva/mvpexample/app/Login/LoginPresenterImpl.java)

## 3. 결론

1. H/W 등 구체적인 코드에 의존하는 어플리케이션 개발자
	- 내 로직에서 H/W 의존적인 부분을 Humble Interface를 통해 분리하고 내 로직은 Unit Test

2. 서버 로직에 의존하는 UI 개발자(App, Web Front)
	- GUI에서 Process Interface(Humble Interface)를 분리하고 Fake 구현체를 추가하고 
	- 내 로직(GUI)은 수동으로 테스트할 수 있는 만큼 Humble하게 유지