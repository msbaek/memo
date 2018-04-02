# IntelliJ 설치 & 설정

IntelliJ Ultimate Edition를 사용 중인데 아래 설명하는 부분을 추가적으로 설치, 설정하여 사용중이다.

## Key Map

"mac osx 10.5+"를 사용

![](https://api.monosnap.com/rpc/file/download?id=7sak069xsiMSgJ9JfPzEat9sIiATYf)

## 	Theme

Darcular Theme을 사용한다.

![](https://api.monosnap.com/rpc/file/download?id=DmylUbMuxXoCvslvbdpcayJWzfVmem)

## Plugin

Lombok
- intellij 내에서 lombok을 사용하기 위한 플러그인

ideaVim
- intellij에서 vim keymapping을 이용하기 위한 플러그인

[Live Edit](http://blog.jetbrains.com/webide/2012/08/liveedit-plugin-features-in-detail/)
- Live Edit plugin을 설치하고, settings에서 live edit를 enable하면 브라우저에서 현재 html을 바로 볼 수 있음.
- 브라우저가 일단 열린 후엔 ctrl+r로 갱신(빌드가 되어야 반영됨)

## 주요 에디터 기능

- Incremental selection: Cmd(Ctrl) + W, opt+up arrow
- Reformat code: Option + Cmd + L
- Find members in current file: Cmd + F12
- Show recently changed files: Cmd + Shift + E
    
## Java8 Migration Feature 활용하기

![](https://api.monosnap.com/rpc/file/download?id=NpP7RcK6c8N6cuNSpqQMUqdLG5jMNv)

위와 같이 설정에서 "Editor / Inspections" 를 선택하고 Java8 하위의 

- Lambda can be replaced with method call
- Loop can be collapsed with Stream API
- Loop can be collapsed with Arrays.setAll()

등을 선택하고 Severity를 "Week Warning"으로 변경하면 Editor에서 해당 기능을 사용할 수 있을 때 지정된 색으로 보여준다. 또 F2를 눌러서 next warning/error로 이동시 선택되고, show intention actions(opt+space)를 누르면 lambda, stream, method reference 등으로 변환해 준다.

## overriden hot key

- select in project view: Ctrl + Opt + L

## H2 File DB 사용하기

![](https://api.monosnap.com/rpc/file/download?id=46dm5qkBEs4qUbGXUF3XWeSKH53WEk)

![](https://api.monosnap.com/rpc/file/download?id=7NlNzS0BIoES7XJX1pmPUxdWAmZACR)

## [IntelliJ를 JIRA와 연동해서 사용하기](http://jojoldu.tistory.com/260)

## [git flow integration](http://jojoldu.tistory.com/268)

## [IntelliJ에서 Json 작업 쉽게 하기](http://jojoldu.tistory.com/273)

### json view plugin

### DTO generator plugin

- DTO from json

### POJO to JSON plugin

- make json(class명에서 우클릭)

### 문자열에서 바로 Json String 생성하기

- 문자열에서 opt+enter - inject language or reference / json 선택

![](https://api.monosnap.com/rpc/file/download?id=ABrJBIUW9wX12FOzFuNHYLMGiVa8o0)

- opt + enter / edit json fragment

![](https://api.monosnap.com/rpc/file/download?id=AsQXEliOlkta1YdK55kfnWKbDqr6EH)

![](https://api.monosnap.com/rpc/file/download?id=8Vc8RK0LswqJxaPR9blJ8n8BJQWFc0)

그럼 JSON 코드 사용모드가 열림.

아래 하단 영역에서 JSON 입력시 자동으로 커서가 있는 곳에 \추가한 JSON 형태의 문자열이 코드 작성시 자동으로 추가됨

![](https://api.monosnap.com/rpc/file/download?id=QTmggkYmx2ginznrppxvJFwJPyQAU0)

## 한영변환 플러그인

jojoldu로 검색하여 Translator plugin 설치

* 한영변환. 주문취소 선택 후 opt+1하면 cancelOrder 를 보여주고
* opt+2하면 cancelOrder로 치환해줌
 
## 참고자료
[인텔리j 활용 꿀팁 42가지 정리](http://www.popit.kr/인텔리j-활용-꿀팁-42가지-정리/)
[Collaboration within IntelliJ IDEA](https://www.youtube.com/watch?v=wBXSUdT1jX0)