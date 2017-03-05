# The Clean Code Talks -- Inheritance, Polymorphism, & Testing

[원본 영상](https://www.youtube.com/watch?v=4F72VULWFvc)

객체 지향에서는 if, switch를 사용하지 말라고 한다. 가장 큰 이유는 문어발식 의존성(if, case에서 사용하는 코드에 대한 의존성) 때문이라고 생각한다. 이러한 문어발식 의존성 때문에 너무나 깨지기 쉬운 코드가 되므로…

이 동영상에서 [Miško Hevery](http://misko.hevery.com/about/)는 또 다른 이유에 대해서 언급하고, 이를 해소하기 위한 방법에 대해서 언급한다.

#### 전제

대부분의 if 문장은 다형성(polymorphism)으로 대체될 수 있다.

#### if를 다형성으로 대체해야 하는 이유

- if가 없는 함수는 읽기 쉽다.
- if가 없는 함수는 테스트하기 쉽다.
- 다형성이 적용된 시스템은 유지보수하기 용이하다.

#### 언제 다형성을 사용하나

- 객체가 자신의 상태에 따라 다르게 동작해야 할 때
- 코드의 2곳 이상에서 동일한 조건을 조사해야 할 때
	- if가 하나만 존재한다면 놔둘수도 있다

#### if로부터 안전하기

- 절대 null을 리턴하지 말라. 대신 Null Object(e.g. empty list)을 리턴하라.
- 에러 코드를 리턴하지 말라. 대신 exception(RuntimeException)을 발생시켜라.

#### 만연한(rampant) 서브클래싱

다형성은 서브 클래싱을 사용한다. 무분별하게 상속을 사용하지 않도록 주의하라.

상속은 정적 의존성으로서 가장 강력한 의존성의 하나이다. 대부분의 레거시 코드 개선은 의존성을 깨는 것에 있다. 그러므로 무분별한 사용을 삼가해라.

#### 조건문을 다형성으로 치환하기

- 객체의 타입에 따라 다른 행위를 선택하는 조건문이 있을 때
- 본래의 메소드는 추상 타입으로 변경하고, 조건에 따라 다르게 수행되는 문장들을 서브 클래스에서 오버라이드하도록 옮긴다.

## 예제. 계산기

`1 + 2 * 3`

을 계산하는 프로그램을 생각해 보자. 아래 그림과 같은 infix notation을 사용하여 해결할 수 있을 것이다.

![](https://api.monosnap.com/rpc/file/download?id=B34VNO2BkdDpHXH4Y789qdYlwrN2ck)

위 구조를 Node라는 클래스로 표현하면 아래와 같다.

```java
    @Test
    public void one_plus_two_multiply_3_should_7() {
        Node one = new Node(1);
        Node two = new Node(2);
        Node three = new Node(3);

        Node multiply = new Node(two, three, Node.Operator.MULTIPLY);
        Node plus = new Node(one, multiply, Node.Operator.ADD);

        assertThat(plus.evaluate(), equalTo(7));
    }
```

이 테스트를 한번에 성공시킬 수 없으니. Node one = new Node(1);를 제외한 모든 라인을 커멘트 처리하고, 한 라인씩 동작하도록 만든다.

```java
    @Test
    public void one_plus_two_multiply_3_should_7() {
        Node one = new Node(1);

        assertThat(one.evaluate(), equalTo(1));
        // Node two = new Node(2);
        // Node three = new Node(3);
        // Node multiply = new Node(two, three, Node.Operator.MULTIPLY);
        // Node plus = new Node(one, multiply, Node.Operator.ADD);
        // assertThat(plus.evaluate(), equalTo(7));
    }
```    

이런 식으로 한 라인씩 처리하면 아래와 같은 결과를 얻는다.

```java
    @Test
    public void one_plus_two_multiply_3_should_7() {
        Node one = new Node(1);
        Node two = new Node(2);
        Node three = new Node(3);

        Node multiply = new Node(two, three, Node.Operator.MULTIPLY);
        Node plus = new Node(one, multiply, Node.Operator.ADD);

        assertThat(plus.evaluate(), equalTo(7));
    }

public class Node {
    static enum Operator {MULTIPLY, ADD, VALUE};
    private Operator operator = Operator.VALUE;

    private int value;
    private Node left;
    private Node right;

    public Node(int value) {
        this.value = value;
    }

    public Node(Node left, Node right, Operator operator) {
        this.left = left;
        this.right = right;
        this.operator = operator;
    }

    public int evaluate() {
        switch (operator) {
            case MULTIPLY:
                return left.evaluate() * right.evaluate();
            case ADD:
                return left.evaluate() + right.evaluate();
            case VALUE:
                return value;
        }
        throw new RuntimeException(“unsupported operator”);
    }
}
```

![](https://api.monosnap.com/rpc/file/download?id=TQ7soRxiBz7uAS1MZSdPUjCHvX2cCx)

위 코드의 문제점

- evalute가 연산자가 추가될 때마다 변경되어야 하고(OCP 위반)
- value node인 경우는 left, right가 항상 null이고
- operator node의 경우는 value가 항상 의미가 없고,

## Refactoring

subclassing을 적용하자.

### 1. test 수정

테스트를 먼저 아래와 같이 수정한다.

```java
    @Test
    public void one_plus_two_multiply_3_should_7() {
        Node one = new ValueNode(1);
        Node two = new ValueNode(2);
        Node three = new ValueNode(3);

        Node multiply = new Node(two, three, Node.Operator.MULTIPLY);
        Node plus = new Node(one, multiply, Node.Operator.ADD);

        assertThat(plus.evaluate(), equalTo(7));
    }
```  

이 테스트가 동작하도록 수정한다. 이때 ide가 제공하는 quick fix를 최대한 활용.

아래와 같은 절차를 따른다.

- 테스트를 위와 같이 polymorphic하게 수정
- quick fix를 이용해서 2개의 서브클래스 생성
- push down members를 통해 evaluate 메소드를 서브 클래스로 이동. 이때 수퍼 클래스에는 abstract로 남도록
- 서브 클래스의 evaluate 메소드에서 불필요한 코드 제거

![](https://api.monosnap.com/rpc/file/download?id=0sO5brQqZgFevAXSQ1xdvXV62TdJYx)

OperatorNode를 AddNode, MultiplyNode로 서브클래싱하기 위해 아래와 같이 테스트를 수정

```java
    @Test
    public void one_plus_two_multiply_3_should_7() {
        Node one = new ValueNode(1);
        Node two = new ValueNode(2);
        Node three = new ValueNode(3);

        OperatorNode multiply = new MultiplyNode(two, three, Node.Operator.MULTIPLY);
        OperatorNode plus = new AddNode(one, multiply, Node.Operator.ADD);

        assertThat(plus.evaluate(), equalTo(7));
    }
```

**quick fix**

- push members down: OperationNode::evaluate를 AddNode, MultiplyNode로 이동.

**불필요한 코드(AddNode, MultiplyNode의 evaluate 메소드에서) 제거.**

![](https://api.monosnap.com/rpc/file/download?id=BqpvMlQ6xB1K9c6Xe0IcD4cunmWOto)

**불필요해진 operator 필드, enum 제거**

![](https://api.monosnap.com/rpc/file/download?id=l8JZxO2oUruw5AOX78cr15ZVibk9wm)

이제 불필요한 정보(ValueNode에서 righ, left / OperatorNode에서 value / AddNode, MutiplyNode에서 operator)가 제거되었다.

**다형성 해결책이 좋은 이유**

- 새로운 행위(이 예제에서는 빼기, 나눗셈과 같은 연산)가 기존 코드 변경 없이 추가될 수 있다. **OCP 준수**
- 각 행위(operation, concern)이 별도의 파일로 분리되어 테스트와 이해가 용이하다.

**다형성을 선호해야 하는 경우**

- switch 문장은 거의 대부분 다형성을 적용해야 하는 대상이다.
- 2곳 이상에서 동일한 조건을 검사하는 if 문장.

**if는 없어지는 것인가 ?**

실제로 없어지지는 않고 객체를 생성하는 시점이 if의 역할을 대행한다. 위의 예제에서는 test 코드에서 if에 해당하는 역할을 한다.

보통의 경우 비즈니스 규칙을 구현하는 객체와 조건에 따라 적합한 객체 그래프를 생성하는 Factory로 구분되고, Factory에 if가 존재한다. 비즈니스 객체들은 Factory에 대한 인터페이스에만 의존하고 구현에 의존하지 않음으로써 if로 더렵혀진 Factory의 구현체에 의존하지 않게 된다. DI F/W이 이 Factory 구현체에 대한 역할을 제공한다.

**이 방법의 잇점**

- 조건문이 한곳에 localize된다.
- 중복이 없어진다.
- 책임과 전역 상태가 분리된다.
- 공통적인 코드가 한곳에 모여진다.
- 테스트를 독립적으로, 쉽게, 병렬로 진행할 수 있다.
- 서브 클래스(오버라이드하는 메소드)를 조사하면 어떤 차이가 있는지 쉽게 알 수 있다.