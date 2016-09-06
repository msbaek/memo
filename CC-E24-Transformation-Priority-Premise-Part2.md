# Episode 24. Transformation Priority Premise Part 2

![](https://api.monosnap.com/rpc/file/download?id=a7p04ygLmVrJwfTRsHvekhWNxrdVxK)

# Sort

TDD와 Transformation을 이용해서 정수를 소트하는 알고리즘을 작성해 보자.

## 0. 시작

아래와 같은 실행 가능한 환경으로 시작한다. 모든 설정이 제대로 되었는지 확인할 수 있도록...

```
package sort;

public class SortTest {
    @Test
    public void nothing() {
    }
}
```

## 1. 생각해 낼 수 있는 가장 단순한(trivial) 테스트로 시작

empty list를 소트하는 것.

### 1.1 add failing test

```
@Test
public void sortings() {
	assertThat(sort(Arrays.asList()), is(Arrays.asList()));
}
```

### 1.2 transform - null

![](https://api.monosnap.com/rpc/file/download?id=9NFL6nhrKJXroz0odJIe45b0AYDwlC)

### 1.3 transform - null to constant

![](https://api.monosnap.com/rpc/file/download?id=jXBsZBn51yTn1gofrdNEQ6JvoUOasU)

### 1.4 refactoring

remove duplication

![](https://api.monosnap.com/rpc/file/download?id=oMY1viX0kbVkZSTUnSvnRirwXv3tDI)

## 2. next trivial test.

원소가 하나인 경우

### 2.1 add failing test
```
assertThat(sort(intList(1)), is(intList(1)));
```

![](https://api.monosnap.com/rpc/file/download?id=evzK07aikWhgEDVxBsiQQMV9mVrLam)

### 2.2 transform - constant to variable

![](https://api.monosnap.com/rpc/file/download?id=H8jZ96Uz6feUz4wYPMMRPorH0zjSG3)

## 3. next trivial test.

원소가 두개인데 정렬이 안 된 경우

### 3.1 add failing test

```
assertThat(sort(intList(2, 1)), is(intList(1, 2)));
```

### 3.2 transform - split flow, assign

2개의 원소의 순서가 틀렸다면 swap하면 된다.

![](https://api.monosnap.com/rpc/file/download?id=0NANBEiSnrVtiZYlXBVWcdTizv84mf)

### 3.3 refactoring - prefare extract method

![](https://api.monosnap.com/rpc/file/download?id=3sKeku5qrYJbzpTZevh4jyoZqvG8c1)

### 3.4 refactoring - extract method

![](https://api.monosnap.com/rpc/file/download?id=HzZTfFNYBbyOYFuXP1q2NTDY6gHpbz)

### 3.5 refactoring - inline extracted variables

![](https://api.monosnap.com/rpc/file/download?id=Q4O998YPBsknbIQhWxEqIsA4N31q6B)

## 4. next trivial test.

원소개 3개인 경우의 수는 3! = 6가지이다. 이 중 2가지는 이미 성공했다. 

- 모든 원소가 정렬된 경우
- 처음 2 원소가 정렬되지 않은 경우

그래서 원소가 세개인데 2번째와 3번째 요소가 정렬이 안된 경우를 추가

### 4.1 add failing test

```
assertSorted(intList(1, 3, 2), intList(1, 2, 3));
```

### 4.1 transform - split flow, assign

첫번째와 두번째 원소를 비교하여 swap한 것처럼, 두번째와 세번째 원소도 동일한 방법으로 처리

![](https://api.monosnap.com/rpc/file/download?id=5dL6uqvpLGijS8nmIukqxzPgTmTaif)

### 4.2 refactoring duplication(too specific)

extract variable, if to while

![](https://api.monosnap.com/rpc/file/download?id=n4B9p2ASOLI43PZtGGupCghAZJKdzV)

### 4.3 refactoring - extract methods, while to for

![](https://api.monosnap.com/rpc/file/download?id=XRRyGvFWVbBSqNAj5pyKBIU605nORo)

## 5. next test

가장 작은 값이 제일 나중에 있는 경우 실패한다.

### 5.1 add failing test

```
assertSorted(intList(3, 2, 1), intList(1, 2, 3));
```

### 5.2 transform - iterate transformation

이 알고리즘의 문제는 큰 수는 뒤로 보내지만, 작은 수는 앞으로 한칸만 올 수 있다는 것이다. 그래서 제일 작은 값이 맨 앞으로 올 수 있을 만큼 반복해야 한다.

list.size()를 size라는 변수로 transform

![](https://api.monosnap.com/rpc/file/download?id=3YFicfmuIacJfjCVSd9FXXj1BuMss2)

## 6. next trivial test

### 6.1 add failing test

이렇게 만들어진 버블 소트는 원소의 갯수가 많아지면 느려지는 문제를 갖는다.

- 1k - 26ms
- 5k - 217ms
- 10k - 952ms

N^2 알고리즘이기에...

## 7. ???

이렇게 된 것이 TDD의 문제가 아니라 transformation을 적용할 때 우선순위를 따르지 않았기 때문이라고 한다.
이제부터는 우선순위를 따르면서 다시 sort를 구현해 본다. 이전의 테스트로 돌아가서 그 테스트를 성공시킬 수 있는 다른 방법은 없었는지 고민해 보자.

3번째 테스트 케이스(2개의 원소를 갖는)로 돌아가자...

## 8. 2 elements

### 8.1 failing test

```
assertSorted(intList(2, 1), intList(1, 2));
```

### 8.2 transform - split flow, add computation

11번째 우선순위를 갖는 assign을 적용하기 전에 다른 기법을 적용할 수 있다.

인자로 전달받은 list를 변경(assign)하기 보다는 새로운 리스트를 만드는 것이 좋다.

![](https://api.monosnap.com/rpc/file/download?id=CKy0LQD5o6VzB2PfE7J3tYZrS44Zcc)

## 9. 3 elements

### 9.1 add failing test

3개의 원소를 갖고, 처음 2개의 순서가 틀린 테스트로 시작

`assertSorted(intList(2, 1, 3), intList(1, 2, 3));`

그 전에는 

`assertSorted(intList(1, 3, 2), intList(1, 2, 3));`

였다.

### 9.2 transform - split flow, add computation

3개의 원소가 있고, 처음 2개의 원소만 순서가 틀렸으므로 첫 원소는 중간값이다(마지막 원소는 순서가 맞고, 처음 2개만 틀린 것이니).

그럼 첫번째 원소 외의 2개의 원소의 값을 비교해서 적절한 순서로 리스트에 추가하면 된다.

![](https://api.monosnap.com/rpc/file/download?id=vlDr1WabC08o3Y30PQcPWKaqKMEMy7)

### 9.3 add another test

`assertSorted(intList(2, 3, 1), intList(1, 2, 3));`

### 9.4 prefare refactoring

![](https://api.monosnap.com/rpc/file/download?id=j7KGwfUltEiOHlehSlkzwBErtCIE1m)

code duplication이 존재하고 code가 specific하다... refactoring을 해야 한다.

3개의 조건문이 비슷한 패턴을 보인다. 어떻게 일반화할 수 있을까 ?

다른 점은 리스트에 추가되는 원소의 갯수이다. 이것은 loop를 의미한다.

#### 9.4.1 원소가 3개인 경우

2개의 조건문에서는 최저값, 중간값, 최고값을 리스트에 추가하는 일을 한다.

![](https://api.monosnap.com/rpc/file/download?id=yDqcKgOkCpiKdjQGPWAIZEWMK89J1D)

if -> for

![](https://api.monosnap.com/rpc/file/download?id=E67lhTiPesj4tBtSHj5h5MrybIcf1W)

more generalize

![](https://api.monosnap.com/rpc/file/download?id=3TkLZfnEA5hPbrCT9nK0thDGxVkcC6)

make it works

![](https://api.monosnap.com/rpc/file/download?id=nMAjERe8tcERZ52ArXlpww0EdOIGIm)

more refactoring

![](https://api.monosnap.com/rpc/file/download?id=P90h5z6gf6mv3VFltccE723qrxVO1Z)

## 10. 2가지 실패하는 경우

### 10.1 add failing test

1. 첫번째 원소가 가장 큰 경우 `assertSorted(intList(3, 1, 2), intList(1, 2, 3));`
2. 첫번째 원소가 가장 작은 경우 `assertSorted(intList(1, 3, 2), intList(1, 2, 3));`

1번의 경우는 l이 2개가 되고, 2번의 경우는 h가 2개가 되어 테스트가 실패한다.

### 10.2 make it works

![](https://api.monosnap.com/rpc/file/download?id=06soDtVpugeA6qP9nrvdJThjlCzDAx)

## 11. 1개 이상의 중간값이 존재하는 경우

### 11.1 add failing test

`assertSorted(intList(3, 1, 2, 2), intList(1, 2, 2, 3));`

### 11.2 make it works

![](https://api.monosnap.com/rpc/file/download?id=Ysa2toBtg3RhMOrEC6gKZQBSJycoou)

이 quick sort는 원소의 갯수가 많아도 속도 크게 저하되지 않는다.

## Conclusion

- 테스트를 성공시킬 2가지 이상의 transformation이 있다면 우선순위가 높은 방법을 적용하라.
- 이게 모든 경우에 적용되는지 확신할 수 없어서 Law, Theory가 아니라 Premise라고 한다.

### getting stuck

현재 테스트를 성공시키기 위해서 너무 많은 코드를 작성해야만 하는 경우

이럴때는 테스트를 성공시키기 위해서 보다 높은 우선순위의 transformation을 적용할 수 없었는지 살펴라. 또 테스트를 값자기 복잡한 것을 추가하지 않았는지 살펴라.

**테스트를 추가하는 순서와 성공시키기 위해 적용하는 Transformation의 순서가 중요하다.**

한번에 문제의 본질에 바로 접근하지 말아야 한다. 충분히 trivial 테스트를 추가하고, 최대한 단순한 방법으로 해결하다 보면 견고한 알고리즘이 얻어진다.