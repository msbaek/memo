# Sort

## 1. empty list

- null transform
- null to constant

## 2. (1) - one element

- constant to variable: `return list;`

## 3. (2, 1) - two out of order elements

- split flow: `if(list.size() > 1)`, `if(list.get(0) > list.get(1))`
- assign: 0, 1번쩨 값을 스왑
- refactor test: extract method - assertSorted, intList

## 4. (1, 3, 2)

원소개 3개인 경우의 수는 3! = 6가지이다. 이 중 2가지는 이미 성공했다.

- 모든 원소가 정렬된 경우
- 처음 2 원소가 정렬되지 않은 경우

그래서 원소가 세개인데 2번째와 3번째 요소가 정렬이 안된 경우를 추가

- split flow: `if(list.size() > 2)`
- assign: 1,2번째 값을 스왑
- refactor to remove dup.
	- introduce index
- if to while: `if(list.size() > 1)` --> `while(index < list.size() - 1)`
- refactor: while to for
- refactor: extract method `outOfOrder`, `swap`

## 5. (3, 2, 1)

가장 작은 값이 제일 나중에 있는 경우 실패한다.

큰 수는 뒤로 보내지만, 작은 수는 앞으로 한칸만 오는 문제.

제일 작은 값이 맨 앞으로 올 수 있을 만큼 반복해야 한다.

- iterate transformation
	- extract `size` variable
	- `for (int size = list.size(); size > 0; size--)` 추가
	
transformation priority premise를 무시해서 성능이 않 좋은 알고리즘(버블소트)이 나왔다.

## (2, 1)로 되돌아가자.

- split flow: `if(list.size() <= 1)`, `if(list.get(0) > list.get(1))`
- add computation: `sorted.add(list.get(1));` ...

## (2, 1, 3)

처음 두 원소의 순서가 틀린 경우

`3개의 원소가 있고, 처음 2개의 원소만 순서가 틀렸으므로 첫 원소는 중간값`

- split flow: `if(list.size() == 2)`, `else if(list.size() == 3)`
- add computation: `sorted.add(2);`

## (2, 3, 1)

- refactoring: dup이 있고, code가 specific
	- low, mid, high 도입: `sorted.add(low)...`
- if --> for: `for(Integer i : list)`
- more generalize for 2 elements: 2개인 경우에 대한 처리 제거
- more refactoring: `if (list.size() <= 1)` --> `if (list.size() == 0)`

## 첫번째 원소가 제일 크거나/작은 경우 - (3, 1, 2), (1, 3, 2)

low나 high가 2개가 되어 실패

- variable to array(?)
- recurse

## 2개 이상의 중간값이 존재하는 경우 - (3, 1, 2, 2)

- 범위 조정: `for (Integer i : list.subList(1, list.size()))`
- 조건 변경: `if(i > high)` --> `else`