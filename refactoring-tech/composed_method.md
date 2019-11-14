# Composed Method

Composed Method는 Kent Bech의 "Smalltalk Best Practice Patterns"에 나오는 기법

- Keep all of the operations in a method at the same level of abstraction.
- Divide your program into methods that perform one identiﬁable task.
- This will naturally result in programs with many small methods, each a few lines long.

## Clean Code: Fundamentals, Episode 3 Functions
https://cleancoders.com/video-details/clean-code-episode-3

### One Level of Abstraction per Function

- 함수가 하나의 일을 하려면 함수 내의 모든 문장은 같은 레벨로 추상화되어야함
	- getHtml() : 매우 높은 수준의 추상화
	- String pathName = PathParser.render(pagePath); : 중간 수준의 추상화
	- .append(“\n”); : 매우 낮은 수준의 추상화
- 이렇게 추상화 수준이 다른 라인들이 함수 내에 존재하면 안됨
- 추상화 수준이 섞여 있는 함수는 항상 혼란스럽다 ^^ 독자는 특정 표현식이 필수 개념인지 구체적인 부분인지 구별 할 수 없을 것이다.
- “깨진 유리창의 법칙”과 마찬가지로 한번 필수 개념(고수준의 추상화)이 구체적 구현(저수준의 추상화)과 섞이기 시작하면 점점 더 많은 구체적 구현이 함수에 증가하게 된다.

### Reading Code from Top to Bottom: The Stepdown Rule
- 독자는 top-down 방식으로 이야기 하듯이 코드를 읽기를 원한다.
- 함수는 바로 하위 수준의 추상화를 갖는 함수 호출로 구성되어, 프로그램을 읽을 때, 함수의 목록을 마치 추상화 단계를 내려가면서 읽듯이 보고 싶어함
- 이러한 방식을 Stepdown Rule이라고 함
- 다시 말해서 프로그램을 마치 일련의 TO 단락인 것 처럼 읽을 수 있어야 한다. 각 단락은 현재의 추상화 수준을 기술하고, 이어지는 다음 추상화 수준을 갖는 TO 단락을 참조한다.

예.
```language
To include the setups and teardowns, we include setups, then we include the test page content, and then we include the teardowns.
	To include the setups, we include the suite setup if this is a suite, then we include the regular setup.
	To include the suite setup, we search the parent hierarchy for the “SuiteSetUp” page and add an include statement with the path of that page.
	To search the parent …
```
- 코드를 일련의 top-down TO 단락으로 읽을 수 있도록 만드는 것이 추상화 수준의 일관성을 유지하는 효과적인 기법이다.

## 참고자료
http://c2.com/ppr/wiki/WikiPagesAboutRefactoring/ComposedMethod.html
http://www.javajigi.net/display/SWD/Compose+Method
https://www.safaribooksonline.com/library/view/refactoring-to-patterns/0321213351/ch07.html#ch07lev1sec1
http://www.informit.com/articles/article.aspx?p=1398607