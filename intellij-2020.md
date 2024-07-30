# IntelliJ

<!-- TOC -->
* [IntelliJ](#intellij)
  * [설치](#설치)
      * [Create Launcher Script](#create-launcher-script)
  * [유용한 설정 / 기능](#유용한-설정--기능)
      * [IntelliJ에서 Json 작업 쉽게 하기](#intellij에서-json-작업-쉽게-하기)
      * [IntelliJ를 JIRA와 연동해서 사용하기](#intellij를-jira와-연동해서-사용하기)
      * [git flow integration](#git-flow-integration)
      * [method body template 설정](#method-body-template-설정)
      * [f2 눌러서 다음 에러로 이동할 때 warning은 무시하기](#f2-눌러서-다음-에러로-이동할-때-warning은-무시하기)
      * [show whitespaces](#show-whitespaces)
      * [align when multiline](#align-when-multiline)
      * [Wrap Chained Method Calls](#wrap-chained-method-calls)
      * [Do not wrap after single annotation](#do-not-wrap-after-single-annotation)
      * [Inspections(힌트를 warning으로 보여주기)](#inspections힌트를-warning으로-보여주기)
      * [메소드 내의 여러 곳의 return 문장을 이동하기](#메소드-내의-여러-곳의-return-문장을-이동하기)
      * [동적으로 Pattern 객체가 생성되는 코드들을 IntelliJ에서 inspection으로 찾아내기](#동적으로-pattern-객체가-생성되는-코드들을-intellij에서-inspection으로-찾아내기)
  * [Plugins](#plugins)
      * [한영 번역](#한영-번역)
      * [grepConsole](#grepconsole)
      * [checkStyle-idea, pmd plugin, findbugs-idea](#checkstyle-idea-pmd-plugin-findbugs-idea)
      * [gradle dependency helper](#gradle-dependency-helper)
      * [IdeaVim](#ideavim)
      * [Live Edit Tool](#live-edit-tool)
      * [Lombok](#lombok)
      * [Nyan Progress bar](#nyan-progress-bar)
      * [Nyan Tray](#nyan-tray)
      * [http plugin](#http-plugin)
      * [code smells](#code-smells)
      * [Grazie](#grazie)
  * [Trouble Shootings](#trouble-shootings)
      * [VI에서 방향이동 누르고 있어도 한번만 이동되는 문제 해결책](#vi에서-방향이동-누르고-있어도-한번만-이동되는-문제-해결책)
      * [Lombok이 동작하지 않는 문제](#lombok이-동작하지-않는-문제)
      * ["No .git director found!” 오류](#no-git-director-found-오류)
      * [compile heap size 설정하기](#compile-heap-size-설정하기)
      * [빌드 시 gc overhead limit exceeded 에러 떨어지는 경우 발생](#빌드-시-gc-overhead-limit-exceeded-에러-떨어지는-경우-발생)
  * [Shortcuts](#shortcuts)
  * [참고문서](#참고문서)
<!-- TOC -->

## 설치

#### Create Launcher Script

![Alt text](https://monosnap.com/image/lh428wVlVk3kF6bhh9uNAA6Lyzs9uk)


## 유용한 설정 / 기능

#### IntelliJ에서 Json 작업 쉽게 하기

- https://jojoldu.tistory.com/273

#### IntelliJ를 JIRA와 연동해서 사용하기
- http://jojoldu.tistory.com/260
 
 #### git flow integration
- http://jojoldu.tistory.com/268

#### method body template 설정
![Alt text](https://monosnap.com/image/XJ18CcxOylvWb9am0pMdqQC4BjziMd)

#### f2 눌러서 다음 에러로 이동할 때 warning은 무시하기
![Alt text](https://monosnap.com/image/UvpucNxkpSwrzr1qzMaOjSEl0C255T)

#### show whitespaces
![Alt text](https://monosnap.com/image/PP3RGQL8mFjwrx4RxkepHDTJnM5A7N)

#### align when multiline
![Alt text](https://monosnap.com/image/A3Zrdq5RS4N3B37e8qupPFcPk2pX79)

#### Wrap Chained Method Calls
![Alt text](https://monosnap.com/image/IXjoJhODeqwaNJlvE1yOt4BksqRdos)

#### Do not wrap after single annotation
![Alt text](https://monosnap.com/image/Dap2EbnrsKMdeIHeb1Jq40DfGgjkjt)
https://www.facebook.com/tobyilee/posts/10209064690669000

![Alt text](https://monosnap.com/image/ExKfgf7Bbb466duZz4H4BBTm9JpnqC)

#### Inspections(힌트를 warning으로 보여주기)
- Settings / Editor / Inspections 에서 "no highlighting ..."으로 지정된 항목을 warning 등으로 변경하면 에디터에서 해당 경우들이 warning으로 표현됨
- 그럼 "Show Context Actions"를 실행해서 빠르게 개선/수정 가능
- ![Alt text](https://monosnap.com/image/BUmf0PpmgeJYjZcB27oR1i8nUR7ZcS)
- ![Alt text](https://monosnap.com/image/wEQK9R84qDeHbq2KFiP1gTlf6AYFTR)

- foreach loop를 stream api로 변환하도록 warning 보여주기 설정하기
- 설정하기
- ![Alt text](https://monosnap.com/image/z3nUbduy28e5VO2o4oI8sNT3vxvm9H)
- serverity를 warning으로 F2로 이동이 가능하고, opt+enter(show intention actions)를 하면
- ![Alt text](https://monosnap.com/image/QsAG93lrmnFnFXzh2nInV7Khl6jG5m)
- 이렇게 "Replace with collect"가 먼저 나옴.
- 이걸 default처럼 "no highlighting, only fix"로 놓으면 F2로 이동이 안되고, 
- ![Alt text](https://monosnap.com/image/hxAIwAqIdFNg1UyPPBEeAdiTzCb4BR)
- "Replace with collect"가 초록색으로 나중에 나옴...

#### 메소드 내의 여러 곳의 return 문장을 이동하기
- return 에 커서를 두고 CMD + SHIFT + F7 로 모든 return을 highlighting 
- CMD + G 로 탐색

#### 동적으로 Pattern 객체가 생성되는 코드들을 IntelliJ에서 inspection으로 찾아내기
- cmd + shift + A
- Run inspection by name 실행
- Dynamic regular expression could be replaced by compiled Pattern 실행(Inspection에서 별도로 체크하면, 코딩시에 warning으로 뜨는 부분을 쉽게 확인할 수 있음)
	- Settings / Editor / Inspection / Java / Performance / Dynamic regular expression could be replaced by compiled Pattern

## Plugins

#### 한영 번역
- plugins에서 jojoldu로 검색하여 Translator plugin 설치
- 한영변환. 주문취소 선택 후 opt+1하면 cancelOrder 를 보여주고
- opt+2하면 cancelOrder로 치환해줌

#### grepConsole

- https://nesoy.github.io/articles/2018-04/Intellij-grepcode

#### checkStyle-idea, pmd plugin, findbugs-idea

#### gradle dependency helper

- https://plugins.jetbrains.com/plugin/7299-gradle-dependencies-helper
- library is searched in Smart Code Completion by Maven repository

#### IdeaVim

- https://github.com/JetBrains/ideavim
- Vim emulation plug-in for IDEs based on the IntelliJ platform

#### Live Edit Tool
- Live Edit plugin을 설치하고, settings에서 live edit를 enable하면 브라우저에서 현재 html을 바로 볼 수 있음.
- 브라우저가 일단 열린 후엔 ctrl+r로 갱신.

#### Lombok

#### Nyan Progress bar

- https://mingggu.tistory.com/114

#### Nyan Tray

#### http plugin
https://jojoldu.tistory.com/266

#### code smells
https://plugins.jetbrains.com/plugin/14016-intellijdeodorant

#### Grazie
- https://blog.jetbrains.com/idea/2019/11/meet-grazie-the-ultimate-spelling-grammar-and-style-checker-for-intellij-idea/
- the ultimate spelling, grammar, and style checker for IntelliJ IDEA

## Trouble Shootings

#### VI에서 방향이동 누르고 있어도 한번만 이동되는 문제 해결책
- console에서 defaults write -g ApplePressAndHoldEnabled -bool false
- 인텔리제이 재시작

#### Lombok이 동작하지 않는 문제
- 먼저 lombok 플러그인 설치
- 아래 그림과 같이 “Enable annotation processing”을 체크
- ![Alt text](https://monosnap.com/image/267bsPOZHzyKoyJ2LpWog9fxHyf6tv)
- 아래 그림처럼 “Configuration of intellij seams to be ok”를 확인
- ![Alt text](https://monosnap.com/image/HEjx6ZAITcb494lu41MQb7GU1XKWLy)

####  "No .git director found!” 오류
- build.gradle 파일에
	- `git = org.ajoberstar.grgit.Grgit.open()`
- 이런 라인이 있을 때 intellij에서 import하면 
- 아래와 같은 오류가 발생함
- ![Alt text](https://monosnap.com/image/2FEU8qXptDhaWAgUlaPQBooT8GjwZW)
- 이때 위 라인을 아래와 같이 변경하면 문제가 해결됨
	- `git = org.ajoberstar.grgit.Grgit.open(currentDir: file('.'))`
- 참고: https://github.com/ajoberstar/grgit/issues/140

#### compile heap size 설정하기
- ![Alt text](https://monosnap.com/image/StzUeEC4vfk45Cy1zxt9yMHcmGJgVm)

#### 빌드 시 gc overhead limit exceeded 에러 떨어지는 경우 발생
- Preferences / Build, Executetion, Deployment / Compiler / User-local build process VM options (overrides Shared options)
- -Xmx4096m  지정

## Shortcuts

| Menu | Shortcuts |
| :--- | --------- |
| Argument documentation for method calls | Cmd + P | 
| (Un)Comment line | Cmd + L | 
| (Un)Comment selection | Shift + Control + / | 
| Collapse all | cmd+shift+(-/+) |
| Collapse/expand all | shift + cmd + -/+ |
| Collapse/expand | cmd + -/+ |
| Collapse/uncollapse | cmd+(-/+) |
| Cycle through the history of most recent changes | Cmd + Shift + Backspace | 
| Delete line | Cmd + Y | 
| Documentation comment block | /** + Enter | 
| Find action | cmd + shitt + a |
| Find members in current file | Cmd + F12 | 
| Find members in current project | Cmd + Alt + Shift + N | 
| Generate | ctrl + n |
| Incremental selection | Cmd(Ctrl) + W | 
| Move code up and downs | opt+shift+(up/down) |
| Paste from five previous copies | Cmd + Shift + V | 
| Reformat code | Option + Cmd + L | 
| Select word at caret | cmd + w |
| Show recently changed files | Cmd + Shift + E | 
| Structure | cmd + 7 |
| Switcher | ctrl + tab |
| Toggle between test & production | cmd + shift + t |


## 참고문서

- [Collaboration within IntelliJ IDEA](https://bit.ly/2PreAS4)