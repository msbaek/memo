# Mac에서 사용하는 툴들

[toc]

## Finder
- 'Command-['	or 'Command-]'	이전/다음에 확인한 폴더로
- '**Command-Shift-G**':  해당 폴더의 이름 입력으로 바로 이동. 폴더 명의 일부를 입력하고 Tab 키를 눌러 자동 완성 가능

## Spotlight 검색
- moby dick: moby나 dick 중 한 단어라도 포함된 문서
- “moby dick”: 두 단어가 모두 들어간 문서
- moby -Melville: 저자가 Melville이고 moby라는 단어가 들어간 문서
- name:”moby dick": 제목이 "moby dick"인 문서
- **kind: pdf “moby dick”**: pdf 중에 두 단어가 모두 들어간 문서

## vim
- open previous file: :e#
- undo: u
- redo: ctrl + r
- **Hello World !!가 100줄 들어가도록 하기**
	- escape 상태에서 "100iHello World !!" + enter(줄을 나누기 위해) + escape
- 단어의 끝으로 이동: e(<-> w)
- 서제스트
	- i(입력모드) + ctrl+n + ctrl_p 최근 사용한 단어들 세제스트
- **모든 행 죠인**
	- escape 상태 -> ^v -> 화살표로 행 선택 -> J(shift+j)
- **여러 라인에 대해 작업**
	- ^v -> 화살표로 행(필요하면 열) 선택 ->
		- I(shift+i) -> "xxxx" + escape(선택한 열의 행들에 xxxx가 삽입됨)
		- c -> 변경하려는 텍스트 입력
		- x -> 선택한 영역 삭제
- macro
	- 생성: q + macro key(ex. a) + 원하는 편집 행위 + escape + q
	- 실행: @ + macro key
    - n회 반복: n(ex. 100) + @ + macro key
- to change using regular expression
- 중복 제거 정렬: sort u
```
    bau
    ceu
    diu
    fou
    gau
```

    to

```
    byau
    cyeu
    dyiu
    fyou
    gyau
```

`:%s/\v(\w)(\w\w)/\1y\2/g`
'\v'는 [very magic](http://vim.wikia.com/wiki/Simplifying_regular_expressions_using_magic_and_no-magic)이라고 하며 a-zA-Z0-9, _ 를 제외한 이후 문자가 특별한 의미를 갖도록 한다.

### more powerful vim

[dotfiles](https://github.com/skwp/dotfiles) 사용하면 몇가지 좀 더 파워풀한 기능 사용 가능

- show nerd tree: cmd+shift+n
- Show current file in NERDTree: Ctrl-\ - 
- fuzzy file selector: ,t - CtrlP 

## [iTerm2](https://www.iterm2.com)

- cmd+d: 수평분활
- shift+cmd+d: 수직분할
- cmd+opt+화살표: 윈도우간 이동
- cmd+;: auto-completion. 터미널에 나온 단어로 auto completion해 줌.
- opt+enter: paste current selection
- cmd+shift+h: paste history
- tripple click: select entire line
- Quadruple-clicking: performs a "smart select", matching recognized strings such as URLs and email addresses.
- cmd+clicking: click할 url/file을 오픈
- cmd+opt: select rectangle
- Menu
	- Shell > Broadcast Input > Send Input to All Sessions -- cmd + opt + i
		- keyboard input will be sent to all tabs in the current window.
	- Shell > Log > Start/Stop
		- Logging saves all input received in a session to a file on disk.
	- View > Step Back/Forward in Time
		- Stepping through time allows you to see what was on the screen at a previous time.
        
## [Monosnap](https://monosnap.com/welcome)

화면 캡쳐 프로그램.

캡쳐된 이미지에 글씨를 넣거나 등의 수정이 가능

캡쳐 후 파일로 저장, 웹사이트에 업로드(후에 링크 사용), 클립보드 복사 등이 가능

## Things
iphone과 mac에 cloud로 연동되는 할 일 관리 툴 [Things](https://culturedcode.com/things/)

Things는 [GTD(Getting Things Done)](http://bit.ly/2mogbdF) 툴로써 메일이나 웹사이트를 보고 있을 때 핫키로 타스크를 등록하면 메일/사이트로 바로 갈 수 있는 링크가 타스크에 포함되어 쉽게 할일을 정리할 수 있게 해준다. 상용이지만 정말 유용하게 사용하는 툴이다.

![](https://api.monosnap.com/rpc/file/download?id=fYbKpeF2Pu7eLkv9aS97qEJOTa25Te)

* 위 스크린 캡쳐와 편집은 Monosnap을 이용

## Fantastical

[fantastical](https://flexibits.com/fantastical)은 핫키로 쉽게 캘린더에 일정을 등록할 수 있는 툴이다. mac os에 기본 탑재됐어야 할 툴이라는 말을 많이 듣는다.

![](https://api.monosnap.com/rpc/file/download?id=0ZHKWiZp85MjRAMdtKE3fvlgQISSGI)

위와 같이 핫키로 일정을 쉽게 등록하고, 오늘부터 몇일 후의 일정을 쉽게 확인할 수 있다.

## [Alfred](https://www.alfredapp.com)

alfred는 mac 사용자라면 반드시 사용해야 할 launcher 프로그램이다. 마우스로 메뉴를 클릭하지 않고 핫키로 런쳐를 호출하고 원하는 기능을 수행할 수 있어서 생산성을 높여준다.

많은 기능 중에 많이 사용하는 기능은 아래와 같다.

- 사내 wiki를 검색 사이트로 등록하고 검색
- iTunes Mini Player를 핫키로 사용.
- 휴지통 비우기
- 클립보드 리스트 관리
- 자주 타이핑하는 단어 snippnets으로 등록 사용
- spelling check, 사전 검색
- 북마트 사이트 이동(chrome bookmarks workflows)
- short url workflows
- lock, shutdown, restart, sleep 등 시스템 명령을 쉽게 실행

## [Theine](https://itunes.apple.com/kr/app/theine/id955848755?mt=12)

회의 시간에 잠자기 금지로 설정할 때 사용하는 앱

## [AutoKeyboard](http://macnews.tistory.com/2606)

cmd+tab으로 앱 전환시 설정된 언어로 맞춰주는 앱

## [DEVONthink Pro](http://www.devontechnologies.com/products/devonthink/overview.html)

모든 문서(web문서, pdf, keynote ..., txt)를 저장하고 검색할 수 있는 어플리케이션

- 웹문서를 보면서 형광펜으로 줄을 그으면서 읽을 때 
- 각종 일지를 저장/작성/검색

할 때 유용하게 사용하고 있음.

## [Haroopad](http://pad.haroopress.com)

vim 키모드를 지원하는 마크다운 에디터

## [XMind](http://www.xmind.net)

free mindmap 어플리케이션

어떤 생각을 정리하거나 내 생각을 가볍게 다른 이에게 전달할 때 마인드맵은 매우 강력하다. 

presentation(ppt, keynote 등)보다 효과적으로 내 생각을 전달할 수 있다고 생각한다.

## [Softorino YouTube Converter](https://softorino.com/youtube-converter)

youtube url을 로컬에 다운로드해 주는 프로그램

## [Spectacle.app](https://www.spectacleapp.com)

화면에 윈도우를 배치하는데 용이한 프로그램