# CC E24 Transformation Priority Premise Part 1

# Welcome

transformation은 refactoring와 대응 관계(counterpart)에 있다.

- refactoring이 행위의 변경 없이 코드의 구조를 변경하기 위해 소스 코드에 작은 변경을 한다면,
- transformation은 반대로 구조의 변경 없이 행위를 변경하기 위해 소스 코드에 작은 변경을 가하는 것이다.

transformation의 우선 순위를 잘 적용하면 보다 나은 알고리즘을 얻을 수 있다는 것을 보여 줄 것이다.

# Transformation

마틴 파울러의 책에 의한 리팩토링의 정의

> Refactoring, is: A change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior.

실제 rename 같은 것은 행위에 아무런 변경도 유발하지 않겠지만, extract method 같은 것은 실행 속도를 다소 느리게 할 것이다(수 nano sec.). 그러므로 행위의 변경 없이가 의미하는 것은 주목할 만한 변경 없이라고 이해해야 한다.

반대로 구조의 변경 없이 행위를 변경할 수 있나 ? 불가능하다. 만일 addition 연산을 substraction 연산으로 단순히 변경했다면 이게 구조의 변경을 유발했을까 ? 이런 경우는 구조의 변경이 없을 것이다.

또 하나의 문장을 while loop으로 감싸는 경우도 구조를 주목할만큼 변경하지 않는다. 하지만 행위는 변경된다.

이런 경우들에서 주목할 만큼이 얼마만큼을 의미하는지에 따라 구조적 변경이 있는지 없는지가 달라질 것이다.

**transformation이 TDD의 RGB 중 어디에 적합할까 ?**

실패하는 테스트를 성공시키기 위해 사용될 수 있다. 즉 Red, Transform to Green, Refactor...

- 테스트가 실패하면 구조는 유지한채 성공시키기 위해 행위를 변경하다. - transformation
- 테스트가 성공하면 행위를 유지한채 구조를 개선한다. - refactoring

**실패하는 테스트를 성공시키는데 적용되는 규칙: As the tests get more specific, the code gets more generic.**

그럼 transformation은 어떤 방향으로 코드를 이동시키나 ?(What direction must these transformations move the code in ?)

**transformation은 테스트를 성공시키는데 사용된다. transformation는 코드를 보다 general하게 한다. transformation은 generalizer이다.**

> Transformation은 구조에 영향을 미치지 않은채 행위를 일반화(generalize)하는 코드에 대한 작은 변경이다.
transformation은 구조 변경없이 행위를 일반화한다.```

# PrimeFactor 예제

어떤 것이 Transformation인지 알아본다.

## 1. add failing test(most simple)

![](https://api.monosnap.com/rpc/file/download?id=KpJb2jCO23LnImnW4xlzmBakuh65xV)

### 1.1 make it compile

![](https://api.monosnap.com/rpc/file/download?id=NHDxb4hiRDj4xCfzQTJuqIyu4GIU5N)

가장 쉽게 성공(지금은 컴파일)시키는 방법은 null을 반환하는 것이다. 이것이 첫번째 transformation인지 **null transformation**이다.

### 1.2 transform to pass
primeFactorsOf가 empty list가 아니라 null을 반환하기 때문에 이 테스트는 실패한다. 이 테스트를 성공하도록 하기 위해 null을 empty list로 transform한다.

![](https://api.monosnap.com/rpc/file/download?id=qqpqrwTvID2AqPgTn1UxPjnAnffsCg)

우리의 두번째 **transform은 null -> constant**이다.

null도 constant가 아닌가 ? **null도 constant이지만 null은 매우 특별한 constant**이다. 정의할 수 있는 타입도 없고, 값도 없는 constant이다. 따라서 type이 있고 값을 가질 수 있는 constant와는 다르다.

> null -> constant로 수행한 transform은 코드를 보다 일반적(general)으로 만든다. null은 매우 특별한 경우이고, immutable이지만 empty list는 가능성을 가지고 있다.
일반적으로 된다는 것은 보다 다양한 경우를 처리할 수 있다는 것을 의미한다.

## 2. add failing test for 2
2번째 테스트로 `assertThat(primeFactorsOf(1), is(Arrays.asList(2)));`를 추가한다. 이 테스트는 실패한다.

![](https://api.monosnap.com/rpc/file/download?id=bcRnZorqHw9q5Rw9B04ywSREFK8auY)

### 2.1 transform - 'constant to variable'

![](https://api.monosnap.com/rpc/file/download?id=AxIiKwVMHHcd4y5wGTpJjvKWHafkNZ)

이 테스트를 성공시키기 위해 new ArrayList<Integer>() constant를 일반화(generalize)한다.

이를 위해 constant를 factors라는 이름의 변수로 transform한다. 3번째 transform은 **constant -> variable**이다.

> 이건 refactoring(extract variable)이고, 행위를 변경하지 않았다...
부분적으로는 맞는 말이다. constant를 variable로 변경하는 것 만으로 행위를 변경하지는 않는다. 하지만 행위 변경이 가능하도록 한다. 다시 말해 constant를 variable로 변경하는 것이 독립적으로 transformation은 아니지만 transformation의 필요한 부분이다.

### 2.2 transform - ‘split flow'

![](https://api.monosnap.com/rpc/file/download?id=k2Ufj3I4c6YQ3RmsiYB21Q1RVlF5RS)

constant를 variable로 변경한 transformation은 **split flow**라는 4번째 transformation을 가능하게 한다. split flow transformation에서 if 문장을 사용해서 흐름을 분리한다.
이 transformation은 제어의 흐름을 분리한다. 이 transformation은 constant -> variable transformation에 의해 가능해졌다.
	if 문장이 코드를 더 구체적으로 만들었다. 행위가 더 일반화되지 않았다.

### 2.3 refactoring to general
**if(n == 2)**였은 매우 specific(테스트의 의도를 그대로 나타낸 것)한 경우이다. 보다 general한 경우를 처리할 수 있도록 리팩토링한다.

![](https://api.monosnap.com/rpc/file/download?id=X43TzM9eH2KLOer9o4ljqLdE8SVbHm)

**if(n > 1)**로 하면 가능성을 열어두게 되기 때문에 훨씬 더 일반적이다.

## 3. add failing test for 3

![](https://api.monosnap.com/rpc/file/download?id=eqY27run6fJfbgcIhOKhlUt8mEtsYA)

세번째 테스트로 assertThat(primeFactorsOf(3), isListOf(3));를 추가하면 테스트가 실패한다.

### 3.1 transform - ‘constant to variable'

![](https://api.monosnap.com/rpc/file/download?id=UxfYde9xtNIFldCcuu5hKa8aoB3JE4)

이 테스트를 성공시키기 위해 **constant -> varaible** transformation을 적용하여 **factors.add(2)를factors.add(n)**으로 변경한다.

## 4. add failing test for 4

![](https://api.monosnap.com/rpc/file/download?id=H0PMVQbYzUfhQWrV9TLT0tlsjukdsZ)

네번째 테스트로 assertThat(primeFactorsOf(4), isListOf(2,2));를 추가한다.

### 4.1 transform - split flow
이 테스트를 성공시키기 위해선 다시 n이 2로 나눠지는지 여부에 따라 split을 해야 한다.

![](https://api.monosnap.com/rpc/file/download?id=xEFmv9VyGsxna7xuzfaQICqa7nGF22)

### 4.2 more transform - split flow
하지만 이렇게 고쳐도 테스트는 실패한다. 4의 경우는 성공하지만 2의 경우는 2만이 아니라 1도 포함되기 때문이다. 성공시키기 위해서 한번 더 split한다.

![](https://api.monosnap.com/rpc/file/download?id=8YxsHveprixcnlhwzU7H2XJfvrbIfa)

이렇게 하면 성공된다.

### 4.3 refactor

![](https://api.monosnap.com/rpc/file/download?id=XktdknU22zRI57Zf9XMYSqUjlNdkCv)

n > 1인 경우에 대해서는 2번이나 split을 했다. 걱정하지 말라. 그런 split은 곧 사라진다. 2번째 n > 1부분은 조건절 외부로 이동시켜도 아무런 이슈가 없다.

### 4.4 refacotr

test code의 중복 제거

![image](https://api.monosnap.com/rpc/file/download?id=327QST6STNSl8SzfDsPq8NtE97Jn5O)

## 5. add more tests
5,6,7의 경우는 테스트를 추가해도 패스한다.

## 6. add failing test for 8
`assertThat(primeFactorsOf(8), isListOf(2, 2, 2));`는 실패한다.

### 6.1 transform - if to while

![](https://api.monosnap.com/rpc/file/download?id=Q5JsKRlIAaIYT3wYn9j5NurT0xWTxD)

**if -> while** transformation을 적용하여 테스트를 성공시킬 수 있다.

**while은 단지 if의 일반화된 형식이고, if는 단지 while의 특별한 형식이다**

### 6.2 refactor
while loop를 아래와 같이 for loop로 refactoring하여 가독성을 높인다.

![](https://api.monosnap.com/rpc/file/download?id=y0OdlMDVbGyz3JQgMkLEZ03yCMEm2x)

## 7. add failing test for 9
`assertThat(primeFactorsOf(9), isListOf(3, 3));` 테스트를 추가한다.

### 7.1 make it pass by duplicating
아래와 같이 3으로 나눠지는 경우를 추가하여 테스트를 성공시킨다.

![](https://api.monosnap.com/rpc/file/download?id=3KHvpgmj2bTdYifDrSPjph55P5H0oo)

### 7.2 transform to remove duplication - add more generalized loop

**duplication**이 발생했다.
중복 자체는 있을 수도 있지만 중복된 코드가 있는 상태에서 소스 리파지토리에 체크인되어서는 안된다.
**이 중복은 transformation이 아니다. 왜냐하면 어떤 것도 일반화하지 않았기 때문이다.** 이제 할일은 **중복을 제거하기 위해 보다 일반화된 루프를 적용**하는 것이다. **중복된 코드는 언제나 일반적이지 않고 특수하다**

![](https://api.monosnap.com/rpc/file/download?id=HrLEvN0FEeGmO4RJV8NUTUjq38G9ww)

### 7.3 refactor
while loop를 for loop로 refactoring하여 가독성 증대


이 예제에서 아래와 같은 4가지 Transformation이 사용되었다.

- null to constant
- constant to variable
- split flow
- if to while

# The Transformation List

> 1. Null
> 2. Null to Constant
> 3. Constant to Variable
> 4. Add Computation
> 5. Split Flow
> 6. Variable to Array
> 7. Array to Container
> 8. If to While
> 9. Recurse
> 10. Iterate
> 11. Assign
> 12. Add Case

## 1. Null

```
@Test
public void factors() {
	assertThat(primeFactorsOf(1), isListOf());
}
```

이 상태에서 IDE의 hot fix 기능을 이용해서 primeFactorsOf를 구현할 수 있다. 이때 IDE는 null을 반환하도록 구현을 제공한다. 이게 Null Transformation이다.

## 2. Null to Constant

![image](https://api.monosnap.com/rpc/file/download?id=flvtaMMdZgcI0KusVgoEEWFSXDlTtF)

위에서 **null** 대신 **new ArrayList<Integer>()**로 변경하는 것이 Null to Constant Transformation이다.

## 3. Constant to Variable

constant를 variable이나 function의 argument로 변환한다.

ex1). `return new ArrayList<Integer>();`를 변수로 변경하는 것

![image](https://api.monosnap.com/rpc/file/download?id=XkRFm0Uh4aEY3ch5nBKyLYi9ATqucP)

ex2). `factors.add(2)`를 `factors.add(n)`으로 변경하는 것

![image](https://api.monosnap.com/rpc/file/download?id=d60blRT5CKhVyF4aKm0sC2YVbSQ58O)

ex3). `n % 2`의 2를 아래와 같이 변수(divisor)로 변환하는 것.

![image](https://api.monosnap.com/rpc/file/download?id=5KYM7xTfutQirLWAnF9OKjp87IohmM)

## 4. Add Computation

하나 혹은 둘의 계산식을 추가하는 변환. 심지어 변수를 추가하기도 한다. 하지만 절대 이미 값을 가진 변수에 값을 할당하지는 않는다.

이 변환은 수학 계산 등을 할 수 있고, 다른 함수를 호출할 수도 있다. 하지만 이미 존재하는 변수의 상태를 변경할 수는 없다.

![image](https://api.monosnap.com/rpc/file/download?id=h7SVVn0pOtySd6myarWnQ03WRPSC5N)

위 예에서 `if(n > 1), if(n % 2 == 0), n /= 2` 가 transformation에 해당한다.

## 5. Split Flow

이 transformation은 대개 if 문장 등과 관계가 있다. 이 transformation은 실행 흐름을 2개(반드시 2개)의 흐름으로 분리한다.

예.

- 2에 대한 테스트를 성공시키기 위해 도입된 `if(n == 2)`

![image](https://api.monosnap.com/rpc/file/download?id=NpWNrEGGzPT9tTTaGNu8ICjX4Ion6w)

- 4에 대한 테스트를 성공시키기 위해 도입된 `if(n % 2 == 0)`

![image](https://api.monosnap.com/rpc/file/download?id=oMtlLjM4h39wEXZ6ufGi9tRQU3fmHu)


## 6. Variable to Array

아래 stack의 예제에서와 같이 **one-to-many**에 대처하는 기법이다. 

![image](https://api.monosnap.com/rpc/file/download?id=8FUU7jIpIg094mOhbxt3CMpxRu6UX6)


![image](https://api.monosnap.com/rpc/file/download?id=Lim3FsdKjO2r8XmriRYCMhGNM3Zx8I)

이 transformation은 딱 하나의 값에 대해서 동작하는 코드를 둘 이상의 값에 대해서 동작하도록 만들때 필요한 기법이다.

## 7. Array to Container

ex.
stack에서 `int [] elements = new int[2]`를 `List<Integer> elements = new ArrayList<Integer>()`로 변환하는 것.

## 8. If to While

**분리된 플로우가 반복도 되어야 하는 경우에 적용하는 transformation**

특히 variable to array transformation이 적용되고 나면 variable일때는 잘 동작하던 if가 array인 경우에는 while로 변경되어야 하는 경우가 많다.

예. 8에 대한 테스트를 성공시키기 위해서

![image](https://api.monosnap.com/rpc/file/download?id=HKogEkG5nwwJVUtw349ysYFagXhzq9)

`if (n % 2 == 0) {`를 `while (n % 2 == 0) {`로 변경

예. 9에 대한 테스트를 성공시키기 위해.

![image](https://api.monosnap.com/rpc/file/download?id=3MeX6Pcsg4JTmNsOEtDctjOjKKj5zj)

에서 `if(n > 1) {`를 `while(n > 1) {`로 변환하고 `factors.add(divisor);` 다음에 `divisor++;`를 추가

## 9. Recurse

함수에 포함된 계산 로직을 반복해야 하는 경우, 간단히 함수가 자신을 호출하도록 한다.

Recurse transformation은 방치된(neglected)/잊혀진(forgotten) transformation이다. 대부분의 Java, C, C++, C# 프로그래머들은 recursion이 iteration보다 대개의 경우 간단하다는 것을 잊고 있다. 심지어 많은 프로그래머들은 recursion을 생각조차 하지 않고 for loop을 사용하려고 한다.

예. e19 wordWrap

![image](https://api.monosnap.com/rpc/file/download?id=jiKyPjxUXUsk3dmaoxUxg3yiu7TAeB)

이전 코드에서 마지막 줄에서 loop을 사용하지 않기 때문에 마지막 테스트(xxx)는 실패한다. 새로운 코드와 같이 마지막 라인에 recursion transformation을 적용하면 간단히 해결된다.

## 10. Iterate

이 transformation은 반복되어야 하는 계산 로직이 있는데 어떠한 이유로 recursion은 사용하지 않기를 원할 때 사용한다. 대개의 경우 for loop를 사용한다.

![image](https://api.monosnap.com/rpc/file/download?id=jCq0xtWw0tQwRPRF2WmsVV4cR9FRJu)

## 11. Assign

이 transformation 이미 존재하는 변수의 값을 변경하고자 할 때 사용한다.

이 transformation은 이미 존재하는 변수의 값을 변경하고자 할때만 사용된다. 변수를 초기화할 때는 assign transformation을 사용하지 않는다.

- `factors.add(2)`
- `n /= 2;`
- `for(....divisor++)`

이렇게 3번 사용되었다.

## 12. Add Case

이미 분리된 흐림(split flow)가 있는데 더 분리하고자 할 때 이 transformation을 사용한다. 기 존재하는 if 문장에 else-if를 추가하거나, switch 문장에 새로운 case 절을 추가하는 것 처럼...

![image](https://api.monosnap.com/rpc/file/download?id=eLjNrnPZvDFXC1775DWKECLQlbcTGd)

score 함수의 for loop 안의 3가지 if 절.