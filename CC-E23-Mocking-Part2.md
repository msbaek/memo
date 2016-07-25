# Episode 23. Mocking Part 2

이 글에서는 상태가 아니라 상호 협력(행위)을 검증해야 하는 경우에 대해서 알아보고,

상호 협력을 검증할 수 있는 3가지 방법(Test Specific Subclass, Self Shunt, Humble Object)에 대해서 알아본다.

## Behavior vs State

곱하기 연산이 제공되지 않고, 더하기 연산만 제공되는 경우 아래와 같이 곱하기를 구현할 수 있다.

```
public class BasicMathTest {
	private BasicMath basicMath;

	@Before
	public void setUp() throws Exception {
		basicMath = new BasicMath();
	}

	@Test
	public void small_number() {
		int a = 3;
		int b = 4;
		int result = basicMath.multiply(a, b);

		assertThat(result, is(12));
	}

	class BasicMath {
		private int multiply(int a, int b) {
			boolean posA = a >= 0;
			boolean posB = b >= 0;
			boolean negP = posA != posB;

			a = posA ? a : -a;
			b = posB ? b : -b;

			int p = 0;
			while (a-- > 0)
				p = add(p, b);

			return negP ? -p : p;
		}

		private int add(int p, int b) {
			return p + b;
		}
	}
}
```

이 코드가 모든 경우에 정확하게 동작하는지 어떻게 검증할 수 있을까 ?

좀 더 많은 테스트 케이스를 추가할 수 있다. 다양한 종류의 값이 나오는 테스트를 추가하고, 부호에 대한 테스트를 추가하고, 무작위 추출 테스트를 추가하여 검증할 수 있다.

무작위 추출 테스트로 충분할까 ? 만일 오동작하는 정수의 조합이 있다면 어떻게 하나 ? 모든 경우를 테스트해야 하나 ?
32비트 정수의 곱을 검증하기 위해서는 2^64 가지의 조합을 검증해야만 하나 ?
각 테스트가 0.5 microseconds가 소요된다면 285,000년이 소요된다.

무작위 추출 테스트로 충분하기 않고, 모든 경우를 테스트할 수 없다면 어떻게 테스트해야 하나 ?

이럴때는 다양한 경우의 모든 값(state)를 조사하는 대신 행위(behavior, 상호협력)를 검증할 수 있다.

### spy on the algorithm

3 x 4 = 12에 대해서 3+3+3+3=12로 계산되므로 add가 3을 인자로 4번 호출되었는지를 확인하면 된다.

![](https://api.monosnap.com/rpc/file/download?id=WSCDXhIKikq1W3pTTXfwAsRALDC472)

add 함수를 add가 호출되는 횟수를 세는 spy로 교체했다.

그런데 테스트가 알고리즘의 구현에 의존성을 갖게 되었다. 알고리즘을 개선하면 테스트가 실패할 것이다. 테스트가 구현에 의존성을 가지면 안된다.

알고리즘을 shift and add로 변경하면 상호작용 테스트는 실패하고, 무작위 추출 테스트들은 성공한다.

딜레마(TDD의 불확실성 원칙(uncertainty principle))

- 구현을 무시한채 결과를 검사하여 테스트를 하면 모든 경우에 정상 동작하는지 확신할 수 없고, 
- 결과를 무시한채 알고리즘을 spy하면 알고리즘을 리팩토링(개선)할 수 없다.

이 불확실성 원칙이 TDD의 2개의 학파로 이어진다. mockists(London School TDD), statiss(Traditional TDD).

어느것이 더 좋은 것인가 ?

- mock은 시스템의 의존성 역전 경계(dependency inversion boundary)를 교차하는 기능을 테스트할 때 유용
- 반면 경계를 교차하지 않는 기능이라면 전통적인 TDD가 더 유용할 가능성이 높음

따라서 적합한 방법을 택해야한다.

## Mocking Pattern

### Test Specific Subclass

![](https://api.monosnap.com/rpc/file/download?id=DdtWB9Qej1swhkhG5EYPu6g27WKoMh)

어떤 클래스의 함수를 그 클래스의 다른 함수를 변경해서 테스트하려고 할 때 사용하는 기법

![](https://api.monosnap.com/rpc/file/download?id=YnyYi19ssYwsqCeHoMXwoTU2PItJ3m)

자동 유축기에서 Milker#milk 함수는 젓소의 상태(유축이 가능한지)를 조사(checkSeal)하고 펌프를 동작시킨다. milk 함수를 젓소 없이 어떻게 테스트할 수 있을까 ?

- Milker 클래스를 상속해서 TestingMilker 클래스를 만든다.
- checkSeal, engagePump 함수를 override해서 아무 것도 안 하도록 만든다.

```
package miker;

public class Milker {
  private UdderSeal seal = new UdderSeal();
  private MilkPump pump = new MilkPump();
  private Gate gate = new CowGate();

  public void milk() {
    if (checkSeal())
      engagePump();
  }

  protected void engagePump() {
    pump.engage();
  }

  protected boolean checkSeal() {
    return seal.check();
  }
}
```

이 방법을 [Working Effectively with Legacy Code](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052)에서는 subclass & override로 설명한다.

```
package miker;

import org.junit.Test;

import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

public class MilkerTest {
  private boolean pumping;

  @Test
  public void milkLogic() throws Exception {
    Milker milker = new Milker() {
      protected void engagePump() {
        pumping = true;
      }

      protected boolean checkSeal() {
        return true;
      }
    };

    milker.milk();
    assertThat(pumping, is(true));
  }
}
```

milkLogic()에서 `new Milker() {`를 통해 subclassing & overriding

pumping은 sensing variable

### Self Shunt

![](https://api.monosnap.com/rpc/file/download?id=A84cOTxSyV4WkKPsZYuOXSkapiBuap)

테스트 클래스가 테스트 대상이 의조성을 갖는 인터페이스의 구현체가 되어 자신이 mock이 되고, SUT에 자신을 주입하는 방법

multiply 예제에 이 기법을 적용하면 아래와 같다.

![](https://api.monosnap.com/rpc/file/download?id=aXlA2usRg1QX4QDzTlivdjzgqslvQG)

milker가 젓소들을 가두고 있는 장소의 문을 제어한다고 가정하자. 그리고 milker가 유축을 끝내면 문을 연다고 가정하자. 이를 테스트하기 위해서는 Gate 인터페이스를 구현해야 하고, Gate의 open 메소드를 spy해야 한다. 별도의 분리된 spy 객체를 구현하는 대신 테스트 클래스가 spy가 될 수 있다.

```
package miker;

public class Milker {
  private UdderSeal seal = new UdderSeal();
  private MilkPump pump = new MilkPump();
  private Gate gate = new CowGate();

  public void milk() {
    if (checkSeal()) {
      engagePump();
      waitSeconds(60);
    }
    gate.open();
  }

  protected void waitSeconds(int seconds) {
    try {
      Thread.sleep(seconds*1000);
    } catch (InterruptedException e) {
    }
  }

  protected void engagePump() {
    pump.engage();
  }

  protected boolean checkSeal() {
    return seal.check();
  }

  public void setGate(Gate gate) {
    this.gate = gate;
  }
}
```

```
package miker;

import org.junit.Test;

import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

public class MilkerTest implements Gate {
  private boolean pumping;
  private boolean opened = false;

  @Test
  public void milkLogic() throws Exception {
    Milker milker = new Milker() {
      protected void waitSeconds(int seconds) {
      }

      protected void engagePump() {
        pumping = true;
      }

      protected boolean checkSeal() {
        return true;
      }
    };

    milker.setGate(this);
    milker.milk();
    assertThat(pumping, is(true));
    assertThat(opened, is(true));
  }

  public void open() {
    opened = true;
  }

  public void close() {
  }
}
```

self-shunt를 이용하면 SUT가 많은 의존성을 가지고 있어도 하나의 테스트가 그들 모두에 대한 spy/stub으로 동작할 수 있다. you’re centralizing all the spies.

### Humble Object

![](https://api.monosnap.com/rpc/file/download?id=SJVAJoFPT9uUya5AhCDDi6i6aznx4x)

Humble 오브젝트 패턴은 테스트 가능성을 높이기 위해 테스트하기 어려운 시스템의 경계에 적용된다. 이 패턴은 경계(HW, UI, DB 등)에 종속된 클래스(HumbleController)의 로직을 테스트가 필요 없을 정도로 최소화한다. 경계에 종속된 클래스에서 추출된 로직은 경계와 decouple되고, 테스트 가능한 클래스(ControllerLogic)로 이동된다.

이 말은 우리가 spring을 사용할 때 Controller, Dao, Repository 등의 구현체에서 경계(UI, DB)에 속한 로직들을 테스트할 필요가 없을 만큼 Humble하게 하여 Interface 뒤로 숨기고(Interface의 구현체가 되도록), 우리 코드는 이 Interface에 의존하고 테스트 가능하도록 해야 한다는 것을 의미한다. 테스트할 필요가 있는 코드가 있다면 테스트하기 어려운 코드와 Interface를 통해 분리하여 테스트 가능하도록 해야 한다.

유축기에서 게이트를 제어하는 pump가 컨트롤 보드에 의해 구동되는 경우를 가정해 보자. 컨트롤 보드와 협업하는 코드(HumbleController)는 테스트하기 어렵다. 그래서 우리가 해야 할 일은 컨트롤러를 제어하는 모든 로직에서 컨트롤러에서 decouple 가능한 코드를 추출하여 테스트 가능한 클래스(Controller Logic)로 이동시키는 것이다. 이 테스트 가능한 클래스는 컨트롤 보드에 대해서 알지 못한다.
이런 작업 후에는 컨트롤러와 직접 통신하는 아주 작은 코드만이 남고, 테스트할 필요가 없게된다.
HumbleController에 남은 코드는 테스트가 필요없는 직접 컨트롤러를 제어하는 최소한의 코드이다.

```
package miker;

public class MilkPump {
  private PumpRegister pumpRegister;

  public void engage() {
    for (int speed=1; speed<=6; speed++) {
      setPumpSpeed(speed);
      sleep(500);
    }
    setPumpSpeed(7);
  }

  protected void setPumpSpeed(int speed) {
    pumpRegister.setSpeed(speed);
  }

  protected void sleep(int milliseconds) {
    try {
      Thread.sleep(milliseconds);
    } catch (InterruptedException e) { }
  }

  protected void setPumpRegister(PumpRegister pumpRegister) {
    this.pumpRegister = pumpRegister;
  }
}
```

펌프가 가변 속도 모터에 의해 제어된다고 가정하자. 속도는 특별한 IO 레지스터에 3비트 숫자를 작성함으로써 지정된다. 0.5초 마다 모터의 속도를 0~7 사이로 증가시킴으로써 펌프를 제어(engage)한다.

PumpRegister는 테스트하기 어려운 코드이다. engage가 테스트하려는 알고리즘이고, setPumpSpeed, sleep는 override 가능하다.
PumpRegister가 controll board를 제어(테스트하기 어려운 IO 작업)한다.

**테스트하기 어려운 코드(PumpRegister)와 알고리즘(engage)을 다음과 같은 절차로 분리했다.**

1. 테스트하기 어려운 코드(IO를 하는 코드여서)를 인터페이스(PumpRegister) 뒤로 숨긴다(테스트하기 어려운, 할 필요가 없을만큼 Humble한 코드를 Humble Interface의 구현체가 되도록한다)
2. 모든 rampUp 알고리즘(engage() 메소드)에서 사용하는 함수들을 MilkPump 클래스 내의 헬퍼 메소드로 추출했다(setPumpSpeed, sleep 등을 protected로 선언해서 test에서 override할 수 있도록).

```
package miker;

import org.junit.Test;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.core.Is.is;

public class MilkPumpTest {
  String actions = "";

  @Test
  public void testPumpRampUp() throws Exception {
    MilkPump pump = new MilkPump() {
      protected void sleep(int milliseconds) {
        actions += "s";
      }

      protected void setPumpSpeed(int speed) {
        actions += speed;
      }
    };

    pump.setPumpRegister(new PumpRegisterDummy());
    pump.engage();
    assertThat(actions, is("1s2s3s4s5s6s7"));
  }
}
```

sensing varible action 변수를 통해서 모든 행위를 누적

**이 패턴은**

- testable code와 boundary를 넘어서 untestable한 code를 separate하고 decouple하여 DI를 통해 decoupling을 이룬다.
- 이를 통해 untestable code가 testable code에 의존하도록 DI를 적용하여 테스트 불가한 코드가 테스트 가능한 코드에 의존성을 갖도록한다. 
- 테스트하기 어려운 어떠한 boundary에도 이 방법을 적용할 수 있다. 
- ex. GUI Boundary
	- 모든 testable code를 모두 testable class로 분리하고, GUI와 통신하는 코드들을 humble code 뒤에 위치시키면 된다.

![](https://api.monosnap.com/rpc/file/download?id=13mAxWnIkTYzAm02ciuldf5wHrxwfh)

이 프로그램은 2가지 일을 한다. processing과 GUI.
milking cows process는 GUI랑 무관하다. GUI는 테스트하기 어려우니 GUI와 process를 분리하고자 한다. GUI를 Humble로 만들고자 한다. GUI를 테스트할 때는 fake process를 이용한다.
GUI를 테스트할 때는 fake process와 coupled하여 눈으로 확인한다. process와 관련된 것은 모두 stubbed out해서...

### GUI Example

MilkerControlPanel.java

```
package controlPanel;

import miker.MilkerProcess;
import miker.MilkerProcessImpl;

import javax.swing.*;
import javax.swing.border.Border;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class MilkerControlPanel extends JFrame {
  private Font font = new Font("Courrier New", Font.PLAIN, 18);
  private Border lineBorder = BorderFactory.createLineBorder(Color.black);
  private JPanel processMonitor = new JPanel();
  private JPanel waitingPanel = new JPanel(new GridLayout(2,1));
  private JPanel processingPanel = new JPanel();
  private JPanel milkedPanel = new JPanel(new GridLayout(2,1));
  private JPanel buttons = new JPanel();
  private JLabel cowsWaitingTitle = new JLabel();
  private JLabel cowsWaiting = new JLabel();
  private JLabel processing = new JLabel();
  private JLabel cowsMilkedTitle = new JLabel();
  private JLabel cowsMilked = new JLabel();
  private JButton startButton = new JButton();
  private JButton stopButton = new JButton();
  private MilkerControlPanelConnector connector;

  public MilkerControlPanel(MilkerControlPanelConnector connector) {
    this.connector = connector;
  }

  public static void main(String args[]) {
    MilkerProcess process = new MilkerProcessImpl();
    MilkerControlPanelConnector connector = new MilkerControlPanelConnector(process);
    new MilkerControlPanel(connector).start();
  }

  public void start() {
    setFonts();

    processMonitor.add(waitingPanel);
    processMonitor.add(processingPanel);
    processMonitor.add(milkedPanel);

    waitingPanel.setBorder(lineBorder);
    waitingPanel.add(cowsWaitingTitle);
    waitingPanel.add(cowsWaiting);

    processingPanel.add(processing);

    milkedPanel.setBorder(lineBorder);
    milkedPanel.add(cowsMilkedTitle);
    milkedPanel.add(cowsMilked);
    add(processMonitor);

    buttons.add(startButton);
    buttons.add(stopButton);
    add(buttons);

    startButton.addActionListener(new ActionListener() {
      public void actionPerformed(ActionEvent e) {
        connector.startButtonClicked();
      }
    });

    stopButton.addActionListener(new ActionListener() {
      public void actionPerformed(ActionEvent e) {
        connector.stopButton();
      }
    });

    setLayout(new FlowLayout());
    setSize(400, 150);
    setVisible(true);
    setDefaultCloseOperation(EXIT_ON_CLOSE);

    loadControlPanelValues();
    setDynamicLabels();
    setStaticLabels();

    while (true) {
      sleep();
      connector.updateDynamicValues();
      setDynamicLabels();
    }
  }

  private void setFonts() {
    cowsWaitingTitle.setFont(font);
    cowsWaiting.setFont(font);
    cowsMilkedTitle.setFont(font);
    cowsMilked.setFont(font);
    processing.setFont(font);
    startButton.setFont(font);
    stopButton.setFont(font);
  }

  private void loadControlPanelValues() {
    connector.loadStaticValues();
    connector.updateDynamicValues();
  }

  private void setDynamicLabels() {
    processing.setText(connector.processingFlag);
    cowsWaiting.setText(connector.cowsWaiting);
    cowsMilked.setText(connector.cowsMilked);
  }

  private void setStaticLabels() {
    setTitle(connector.controlPanelTitle);
    cowsWaitingTitle.setText(connector.cowsWaitingTitle);
    cowsMilkedTitle.setText(connector.cowsMilkedTitle);
    startButton.setText(connector.startButtonText);
    stopButton.setText(connector.stopButtonText);
  }

  private void sleep() {
    try {
      Thread.sleep(500);
    } catch (InterruptedException e) {
    }
  }
}
```

GUI 코드는 많은, 작은 지저분한 코드로 구성된다.  대부분의 GUI 코드는 static code이다. 다이얼로그 박스, 윈도우 등의 구조를 만드는 것과 같은 일만 한다. 실제로 뭔가를 하지 않는다. milking process에 관련된 코드는 없다. 모든 코드는 그저 단순한 프리젠테이션 코드이다.

startButton.addActionListener, stopButton.addActionListener만이 의미있는 객체의 메소드 호출을 한다(connector#startButtonClicked, connector#stopButton).
setStaticLabels도 connector를 사용. 이 GUI 코드는 title이 뭔지 모른다. 이런 기능을 GUI에 위치시키고 싶지 않다.

MilkerControlPanelConnector(HumbleInterface를 사용하는 GUI 코드)

이 예제에서는 MilkerControlPanel이 GUI이고 MilkerControlPanelConnector는 GUI와 Humble Interface를 연결하는 역할을 한다.

MVP 방식이였다면 MilkerControlPanel이 Humble Interface를 직접 사용했을 것 같다.

```
package controlPanel;

import miker.MilkerProcess;

public class MilkerControlPanelConnector {
  private MilkerProcess process;

  public MilkerControlPanelConnector(MilkerProcess process) {
    this.process = process;
  }

  //static values
  public String controlPanelTitle;
  public String cowsWaitingTitle;
  public String cowsMilkedTitle;
  public String startButtonText;
  public String stopButtonText;

  //dynamic values
  public String cowsWaiting;
  public String processingFlag;
  public String cowsMilked;

  public void startButtonClicked() {
    process.start();
  }

  public void stopButton() {
    process.stop();
  }

  public void loadStaticValues() {
    process.loadStaticValues(this);
  }

  public void updateDynamicValues() {
    process.updateDynamicValues(this);
  }
}
```

connector가 process를 갖는다. MilkerProcess는 Humble 인터페이스이다. 여기서 DI가 일어난다.

humble side는 process 뒤로 숨고, non humble side는 process 앞에 위치하여 process를 사용한다. startButton이 눌리면 process를 start하고 stopButton이 눌리면 process를 stop한다. 이게 버튼이 process에 참여하는 방법이다.

static, dynamic public variable들을 갖는다.

connector의 loadStaticVariables, updateDynamicValues 메소드들이 process에서 connector로 값을 가져오는 함수들이다.

MilkerProcess

```
package miker;

import controlPanel.MilkerControlPanelConnector;

public interface MilkerProcess {
  public void start();
  public void stop();
  void loadStaticValues(MilkerControlPanelConnector connector);
  void updateDynamicValues(MilkerControlPanelConnector connector);
}
```

MilkerProcess가 인터페이스이고 이를 통해 DI가 일어난다. non humble side는 인터페이스 앞에 있고, humble side는 interface 뒤에 있다.

MilkerProcessImpl

```
package miker;

import controlPanel.MilkerControlPanelConnector;

public class MilkerProcessImpl implements MilkerProcess {
  private boolean started = false;
  private int cowsMilked = 0;
  private int cowsWaiting = 100;

  public void start() {
    started = true;
  }

  public void stop() {
    started = false;
  }

  public void loadStaticValues(MilkerControlPanelConnector connector) {
    connector.controlPanelTitle = "Milker Control Panel";
    connector.cowsWaitingTitle = "Cows Waiting";
    connector.cowsMilkedTitle = "Cows Milked";
    connector.startButtonText = "Start";
    connector.stopButtonText = "Stop";
  }

  public void updateDynamicValues(MilkerControlPanelConnector connector) {
    connector.cowsWaiting = "" + (started ? cowsWaiting-- : cowsWaiting);
    connector.processingFlag = started ? "...processing..." : "STOPPED";
    connector.cowsMilked = "" + (started ? cowsMilked++ : cowsMilked);
  }
}
```

GUI를 테스트하기 위한 fake process 구현체
fake process를 통해 GUI를 테스트할 수 있다. GUI는 충분히 humble해서 눈으로 보면서 테스트할 수 있다. 실제로 GUI를 실행하고 눈으로 확인하면서 테스트.

이 프로그램에서 흥미있는 것은 milking process와 swing code의 isolation이다.


### Mocking Frameworks
mocking framework를 쓸때 mockito, easymock, jmock 등을 사용하지만 솔직히 mocking framework를 잘 사용하지 않는다.
TDD에서 mocking framework이 필수적인 것 처럼 이야기하지 않나 ? 안다. 대부분의 사람들이 사용하기를 좋아한다. 하지만 나는 대개 사용하지 않는다.

왜 ?

#### mock은 작성하기 쉽다.

![](https://api.monosnap.com/rpc/file/download?id=RSHzZReAW12C4xSq2SG2xYvYsafttM)

* IDE에서 implement interface를 선택. 그러면 Dummy는 바로 나온다.
* Stubbing 하려면 약간의 코드를 작성하면 된다.
* Spying하려면 함수가 호출되었는지를 저장할 수 있는 약간의 코드를 추가하면 된다.
* Mocking을 하는 것도 약간의 코드를 추가하면 된다.

Framework이 굳이 필요하지 않다. 거기다가 Mock을 수작업으로 작성하면 테스트의 가독성을 보다 증대할 수 있는 좋은 이름을 부여할 수도 있다. 그리고 다른 테스트에서 그 Mock을 재사용할 수도 있다. 그래서 Mock을 수작업으로 작성하는 것이 더 의미가 크다.

**setUp에서 Mock의 행위를 기술하는 것이 더 좋지 않은가 ?**

사실 그렇게 생각하지 않는다. 그게 별로 가독성이 좋지 않다고 생각한다.

![](https://api.monosnap.com/rpc/file/download?id=clVibj6wV3iToko2eqzrZQzkE20Mi6)

mock framework이 나쁘다는 것이 아니라 mock을 작성하는 것은 정말 쉬워서 가독성을 높이기 위해 직접 작성하는 것이 더 좋다는 것이다.

가끔은 사용한다고 했지 않은가 ? 언제 사용하나 ?

사용해야만 할 때 사용한다. mocking tool은 매우 강력하다. shielded 된 final interface를 override하거나, private variable/function을 access하거나... 이런 경우에는 mocking tool이 매우 유용하다.

하지만 잘 설계된 시스템에서는 그런 강력한 기능은 거의 필요치 않다. 하지만 쓰레기 더미 같은 레거시 시스템에서는 그러한 강력한 기능이 매우 유용하다.
