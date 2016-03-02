# Mac Finder, Spotligth 검색, Vim, Iterm2 유용한 기능/핫키들

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
- undo: ctrl + u
- redo: ctrl + r

- **Hello World !!가 100줄 들어가도록 하기**
	- escape 상태에서 "100iHello World !!" + enter(줄을 나누기 위해) + escape

- 단어의 끝으로 이동: e(<-> w)

- 서제스트
	- i(입력모드) + ctrl+n + ctrl_p 최근 사용한 단어들 세제스트
	- i(입력모드) + ctrl+w + p 최근 사용한 단어들 세제스트

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

## iterm2
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
