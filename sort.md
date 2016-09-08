# Sort

## 1. empty list

- null transform
- null to constant

## 2. (1) - one element

- constant to variable: `return list;`


## (2, 1)로 되돌아가자.

인자로 전달받은 list를 변경(assign)하기 보다는 새로운 리스트(sorted)를 만드는 것이 좋다.

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