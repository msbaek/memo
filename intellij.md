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

## 참고자료
[인텔리j 활용 꿀팁 42가지 정리](http://www.popit.kr/인텔리j-활용-꿀팁-42가지-정리/)