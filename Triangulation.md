# Triangulation

원문 [Triangulation](http://tobeagile.com/2009/12/08/triangulation/)

> If I get stuck and I don’t know how a complex algorithm should work I’ll write a test for an error case. Then I’ll write a test for the simplest non-error case I can think of and return a hard coded value. Then I’ll write another test case and see if I can figure out the algorithm at that point. In doing so I gain some momentum and perhaps some insight in how the algorithm should behave on an edge case and a few normal cases.

- first write a test with one assertion

- the tested method returns a constant

- then write another assert for the same method

- → now you have to abstract the correct implementation of the method to make it pass

- time to remove duplication

- from TDD by Example, Kent Beck

## Triangulation

어떻게 복잡한 알고리즘을 구현할 지 모르는 상태에 빠지면(getting stuck되면)

- 에러 케이스에 대한 테스트 작성
- 정상 경우에 대한 가장 간단한 테스트 작성(하드 코딩된 값으로 성공시킬 수 있는)
- 또 다른 테스트 케이스를 작성하고, 이때의 알고리즘을 이해할 수 있는지 살핀다.
- 이렇게 함으로써 경계 케이스와 정상 케이스에 대해서 알고리즘이 어떻게 동작해야 하는지 인사이트를 얻을 수 있다.

이러한 기법은 삼각측량법(triangulation)이라고 한다. 이 방법은 천문 항법(celestial navigation)에 수천년 동안 사용되어 왔다.

움직임을 확인하기 위해서는 하나 보다는 2개 이상의 지평선(horizon)의 지점과 비교하는 것이 쉽다.

코딩도 마찬가지다 하나가 아니라 둘 이상의 테스트 케이스를 조사하는 것이 알고리즘의 동작 방식을 이해하기 쉽게 한다.

이 기법은 한가지 문제점은 잉여 테스트(redundant test)를 만들어 낸다는 것이다. 이러한 잉여 테스트는 리팩토링시 제거되어야 한다.

## kent beck이 말하는 상각 측량법의 이상한 점

- 어떻게 리팩터링할 지 전혀 감이 안 올 때만 사용
- 코드와 테스트 사이의 중복을 제거하고 일반적인 해법을 구할 방법이 보이면 그냥 그 방법대로 구현
  - 왜 한번에 끝낼 수 있는 일을 두고 또 다른 테스트를 만들어야 하나 ?
- 하지만 설계를 어떻게 할지 떠오르지 않을 때면, 삼각측량은 문제를 조금 다른 방향에서 생각해 볼 기회를 제공함
