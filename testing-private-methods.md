# private method를 어떻게 테스트 하나 ?

> private method를 어떻게 테스트하면 좋을까 ?

종종 듣는 질문이다.

private method를 테스트하는 4가지 방법에 대해서 간략하게 알아보자.

## 1. protected나 package level로 scope을 변경하여

private을 protected나 package level(scope 제한자가 없는)로 변경하면 test는 test directory에 production code와 같은 패키지에 있기 때문에 호출이 가능하다.

protected는 sub class에서도 호출이 가능하니 packag level이 더 적합해 보인다.

이게 제일 일반적인 방법이라고 생각한다.

## 2. reflection을 이용해서

reflection(spring [ReflectionUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/ReflectionUtils.html) 등)을 이용해서 private method를 테스트에서 호출하기 전에 public 등으로 변경하여 테스트하는 방법도 있다.

이게 제일 안 좋은(?) 방법이라고 생각한다.

## 3. public으로 변경해서

테스트할 가치가 있는 메소드라면 private method는 public으로 변경할 수 있을만큼 중요한 메소드일 수 있다.

private method가 복잡해서 테스트를 꼭 해야 할 수준이라면 public으로 변경하고 test를 하면서 refactoring을 하는 것이 좋을 것이다.


## 4. 테스트 하지 말라

private method는 public method에 비해 복잡하지 않고, 덜 중요한 method여야 할 것이다.

진짜 복잡하지 않은 private method라면 그 메소드를 호출하는 public method를 테스트할 때 함께 테스트되는 것이 바랍직하다.

그러니 별도로 테스트하지 않는 것도 좋은 전략이라고 생각한다.

## 결론

개인적으로는 3,4번을 먼저 고려하고 굳이 private method를 별도로 테스트해야 한다면 1번(package level)을 고혀하는 것이 맞다고 생각한다.