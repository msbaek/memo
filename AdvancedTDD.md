# Advanced TDD

TDD에서 테스트를 추가하는 순서와 이를 성공시키기 위해 적용하는 기법의 순서는 매우 중요하다.

한번에 바로 문제의 본질에 접근하고자 하다 보면 다른 테스트 케이스를 추가할 때 **Getting Stuck**되는 경우가 발생한다. Getting Stuck은 더 이상 진전하기 어려운 경우를 말한다. 이때는 이전 테스트 케이스 중에 잘못된 순서로 급하게 추가된 경우나 테스트를 성공시키기 위해 잘못된 순서로 해결책을 적용하지 않았는지 살펴야하고, 뒤로 돌아가서 다시 구현해야 한다.

이상의 내용을 clean coders video의 몇몇 에피소드의 예제를 통해서 알아본다.

## CC 19 Advanced TDD1
- 테스트를 추가하는 순서
- 리팩토링을 통해 test가 more specific해짐에 따라 production code가 more generic해지는 과정 설명
- [name inverter 예제](https://github.com/msbaek/code-samples/commits/cc-19-name-inverter?page=2)

## CC 19 Advanced TDD2
- [prime number generator 예제](https://github.com/msbaek/primefactors)

## CC E22 Test Process
https://github.com/msbaek/memo/blob/master/CC-E22-Test-Processes.md

## CC E23 Mocking Part 1
https://github.com/msbaek/memo/blob/master/CC-E23-Mocking-Part1.md

## CC E23 Mocking Part 2
https://github.com/msbaek/memo/blob/master/CC-E23-Mocking-Part2.md

## CC E24 Transformation Priority Premise Part 1
https://github.com/msbaek/memo/blob/master/CC-E24-Transformation-Priority-Premise-Part1.md

## CC E24 Transformation Priority Premise Part 2
https://github.com/msbaek/memo/blob/master/CC-E24-Transformation-Priority-Premise-Part2.md
