# Episode 23. Mocking Part 2

## Behavior vs State

![](https://api.monosnap.com/rpc/file/download?id=plbXJMsURCbK27vKiYOQZpeuUWsaLO)

32bit x 32bit에 대한 테스트를 어떻게 추가할 것인가 ? 64bit의 조합을 다 조사해야 하는가 ? 엄청난 시간이 소요될 것이다.

### spy on the algorithm

3 x 4 = 12에 대해서 3+3+3+3=12로 계산되므로 add가 3을 인자로 4번 호출되었는지를 확인하면 된다.

![](https://api.monosnap.com/rpc/file/download?id=2G1fImHDsIsApM1KNL1MRUa7l0yOOb)

![](https://api.monosnap.com/rpc/file/download?id=UXUgChGRn0oGuDzm3scsfyiWIpki2l)

![](https://api.monosnap.com/rpc/file/download?id=bsjCyJ2KbzOWcUgXbU2cI9q3znbrTV)

![](https://api.monosnap.com/rpc/file/download?id=OMo2BHp9cpng86Hvn6xdqml5YnkpAu)

mock 기반 테스트는 DIP 적용되었을 때 적합. 하지만 Test가 구현에 종속된다는 문제가 발생.

따라서 적합한 방법을 택해야한다.

## Mocking Pattern

### Test Specific Subclass

![](https://api.monosnap.com/rpc/file/download?id=xpt3CwmYHj3nnxHsZT5cNaN19oRkM3)

어떤 클래스의 함수를 그 클래스의 다른 함수를 변경하지 않고 테스트하려고 할 때 사용

![](https://api.monosnap.com/rpc/file/download?id=aIv1QJHabw8PnfVyQweBhjHxAmnonh)

automatic dairy-cow milker. That class has a function named ‘milk’ that checks the seal with the cow’s udder and then turns on the pump. Now let’s say you want to test that ‘milk’ function, but you don’t have a cow.
젖소가 없다면 어떻게 milk 함수를 테스트할 수 있을까 ?

- Milker 클래스를 상속해서 TestMilker 클래스를 만든다.
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

subclass & override와 유사한 개념인듯

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

pumping은 sensing variable

### Self Shunt

![](https://api.monosnap.com/rpc/file/download?id=TDxMB18VOuf1hX06EdMJQldntYe8xU)

테스트 클래스가 인터페이스의 구현체가 되어 자신이 mock이 되고, SUT에 자신을 주입하는 방법

Let’s say that the Milker controls the gate that holds the cow in place. Let’s also say that when the milker is done milking, it opens that gate. If we want to test that, then we can just have our test implement the Gate interface, and spy on the open method of the Gate. We don’t need no separate Spy object, our test can just do the spying itself.

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

![](https://api.monosnap.com/rpc/file/download?id=lxVNby1uOCWja25nAEeg0aGjm9m4DX)

The Humble Object Pattern is applied at the boundaries of the system that are hard to test; in order to increase the testability at those boundaries.
The pattern reduces the logic coupled to such a boundary to a bare minimum; humbling it to the point that it does not require testing. The extracted logic is decoupled from the boundary and moved into classes that are testable.

Let’s say that the pump at gate controls for that milking machne are driven by a controller board. Now it’s usually hard to test the software the talks to a controller board. So what we do is we take all the logic that controls that controller board, and we move it to a testable class that doesn’t know about the controller board. The only code left behind is the bare minimum code that talks directly to the controller, and doesn’t need any testing.

테스트하기 어려운 boundary에 대해 보다 테스트하기 용이하도록 하기 위해 적용한다.

![](https://api.monosnap.com/rpc/file/download?id=O4BA86Tr0mGBKBO2vGpUadc94O3aIX)

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

So let’s say our pump is controlled by a variable speed motor. The speed is specified by writing a three bit number to a special io register. You engage the pump by increasing the speed of that motor from 0 to 7 in half-second increments.

PumpRegister는 테스트하기 어려운 코드이다. engage가 테스트하려는 알고리즘이고, setPumpSpeed, sleep는 override 가능하다.
PumpRegister가 controll board를 제어(테스트하기 어려운 IO 작업)한다.

**테스트하기 어려운 코드(PumpRegister)와 알고리즘(engage)을 다음과 같은 절차로 분리했다.**
1. 테스트하기 어려운 코드(IO를 하는 코드여서)를 인터페이스(PumpRegister) 뒤로 숨기고… 이 인터페이스가 Humble(테스트할 자격 조차 없는)이다.
2. 모든 rampUp 알고리즘에서 사용하는 함수들을 MilkPump 클래스 내의 헬퍼 메소드로 추출했다(setPumpSpeed, sleep 등을 protected로 선언해서 test에서 override할 수 있도록).

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

**이 패턴은 testable code와 boundary를 넘어서 untestable한 code를 separate하고 decouple하여 DI를 통해 decoupling을 이룬다. 이를 통해 untestable code가 testable code에 의존하도록
DI를 적용하여 테스트 불가한 코드가 테스트 가능한 코드에 의존성을 갖도록한다. 테스트하기 어려운 어떠한 boundary에도 이 방법을 적용할 수 있다. ex. GUI Boundary**

모든 testable code를 모두 testable class로 분리하고, GUI와 통신하는 코드들을 humble code 뒤에 위치시키면 된다.

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
startButton.addActionListener, stopButton.addActionListener만이 의미있는 객체의 메소드 호출을 한다. connector#startButtonClicked, connector#stopButton
setStaticLabels도 connector를 사용. 이 GUI 코드는 title이 뭔지 모른다. 이런 기능을 GUI에 위치시키고 싶지 않다.

MilkerControlPanelConnector

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

connector가 process를 갖는다. MilkerProcess는 인터페이스이다. 여기서 DI가 일어난다. humble side는 process 이전에 위치하고, non humble side는 process 이후에 위치한다. startButton이 눌리면 process를 start하고 stopButton이 눌리면 process를 stop한다. 이게 버튼이 process에 참여하는 방법이다.

static, dynamic public variable들을 갖는다.

loadStaticVariables, updateDynamicValues가 process에서 connector로 값을 가져오는 함수들이다.

MilerProcess

```
package miker;

import controlPanel.MilkerControlPanelConnector;

public interface MilkerProcess {
  public void start();
  public void stop();
  void loadStaticValues(MilkerControlPanelConnector connector);
  void updateDynamicValues(MilkerControlPanelConnector connector);
}
MilkerProcess가 인터페이스이고 이를 통해 DI가 일어난다. non humble side는 인터페이스 뒤에 있고, humble side는 interface 앞에 있다.
MilkerProcessImpl
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