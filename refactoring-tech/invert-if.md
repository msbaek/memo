## Invert-If: Guard Clause, Early Return

![invert-if.png](images%2Finvert-if.png)

- 조건문 안에 중요 로직 있음
    - 들여쓰기가 늘어서 복잡도 증가
    - 제일 중요한 로직이 제일 처음
        - 예외적인 경우를 추가하기 어려워짐
- invert if
    - earyly return, guard clause 효과
    - 중요 로직에 대한 들여쓰기가 줄어듦
    - 이렇게 예외적인, 사소로운 로직을 먼저 처리하면 중요 로직이 더 단순해짐
