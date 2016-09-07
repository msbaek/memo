# PrimeFactor

## 1

```
	@Test
	public void factors() {
		assertThat(primeFactorsOf(1), is(Collections.emptyList()));
	}
```

- null transfromation
- null to constant transfromation

## 2

- constant to variable
- split flow
- refactor to more general(`if n > 1`)

## 3

- constant to variable

## 4

- split flow: `if n % 2 == 0`
- split flow: `if n > 1`
- refactor: move if to outer
- refactor: remove dup. in test. `isListOf(2, 2)`

## 5, 6, 7

## 8

- if to while
	- `while은 단지 if의 일반화된 형식이고, if는 단지 while의 특별한 형식`
- refactor: while to for

## 9

- make it pass by dup.
	- 이 중복은 transform이 아님. 어느 것도 일반화하지 않았음.
- 일반화된 루프를 적용하여 중복을 제거
- divisor 추출
- refactor: while to for