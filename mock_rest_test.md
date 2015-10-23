# Mock REST 테스트


## 1. [Deployd](http://www.deployd.com/)
이 글은 [http://www.youtube.com/watch?v=0V8fQoqQLLA](http://www.youtube.com/watch?v=0V8fQoqQLLA) 영상을 보다가 작성했음.

### 설치

`npm install deployd -g`

### Hello World

[Your first Deployd API](http://docs.deployd.com/docs/getting-started/your-first-api.md)

#### deployd 프로젝트 생성 및 기동

```
dpd create hello-world
cd hello-world
dpd -d
```

#### collection 생성

deployd를 이용하면 mongodb를 이용해서 데이터를 관리하게 된다.

##### 1. collection 생성

브라우저에서 Resources 옆의 '+' 버튼을 클릭하고 colelction 이름을 '/things'로 입력

##### 2. 프로퍼티 생성

name 프로퍼티를 생성

##### 3. 데이터 입력

Resources 메뉴 밑의 DATA를 클릭하고 데이터를 입력

##### 4. 데이터 확인

[http://localhost:2403/things/](http://localhost:2403/things/) 에서 데이터를 확인

### 맺음말

요즘 REST-JSON으로 프론트와 서버를 구성하는 것에 관심을 많이 가지고 있습니다. deployd 같은 것을 이용한다면 프론트 개발자가 먼저 프로토타이핑을 진행하고 백엔드 개발자가 해당 스펙을 구현하는 아니면 REST-JSON만 정의하면서 동시에 작업이 가능할 것 같습니다. 전에 [easymock](https://github.com/CyberAgent/node-easymock)이라는 것도 봤었는데 Mongodb를 이용하는 deployd가 보다 나은 느낌입니다.

## 2. EasyMock
[http://blog.outsider.ne.kr/991](http://blog.outsider.ne.kr/991)

아래와 같이 설치

`npm install -g easymock`

아래와 같이 실행

`$ easymock`

### REST + JSON을 위한 Mock Server
디렉토리에 _get.json, _post.json, _put.json, _delete.json 등의 파일을 만들어 놓고 요청을 보내면 적합한 json 파일이 반환된다.

`curl -X GET -i http://localhost:3000 등으로`

문서화를 위한 커스텀 헤더 및 주석

```
     # 성공
     > Parameters:
     > - id
     < @header X-GitHub-Media-Type: github.beta
     {
       "success": true,
       "id": 1
     } 
```

### 자동문서화

[http://localhost:3000/_documentation/](http://localhost:3000/_documentation/)

## 3. WireMock
http://wiremock.org/getting-started.html
http://javacan.tistory.com/entry/useful-test-tool-WireMock
