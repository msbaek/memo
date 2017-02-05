# 타입 의존적 코드 리팩토링 - 서브클래싱

먼저 [토비의 봄 TV](https://www.youtube.com/channel/UCcqH2RV1-9ebRBhmN_uaSNg)를 통해 reactive에 대한 이해를 높여 주신 [이일민님](https://www.facebook.com/tobyilee?fref=ts)에게 감사를 표합니다 ^^

[토비의 봄 TV 10회 스프링 리액티브 프로그래밍 (6) AsyncRestTemplate의 콜백 헬과 중복 작업 문제](https://www.youtube.com/watch?v=Tb43EyWTSlQ) 의 37:30초 부터의 리팩토링 부분을 아래와 같은 순서로 좀 더 IDE의 기능을 이용해서 안정하게 진행해 보았다.

## 0. 문제 영역
```java
private void run(ResponseEntity<String> value) {
	if(con != null)
		con.accept(value);
	else if(fn != null) {
		ListenableFuture<ResponseEntity<String>> lf = fn.apply(value);
		lf.addCallback(s -> complete(s), e -> error(e));
	}
}
```

위 코드에서 Completion이 Consumer를 갖는지 ? Function을 갖는지 ? 에 따라 분기가 일어나고 있습니다. 이렇게 타입(Consumer or Function)에 의존적인 코드는 subclassing으로 OCP를 준수하도록 개선할 수 있다.

## 1. 객체 생성 부분 수정

먼저 서브클래스 생성이 필요하도록 객체 생성 부분을 수정한다.

![](https://api.monosnap.com/rpc/file/download?id=63PgfU88MulgyCm0KO6FAgZc0ELuEK)

## 2. IDE의 hotfix 기능을 이용해서 클래스 생성

intellij의 show intention actions(opt + enter) 기능을 이용해서 IDE를 통해 서브클래스를 생성한다.

```java
public static class ApplyCompletion extends Completion {
	public ApplyCompletion(Function<ResponseEntity<String>, ListenableFuture<ResponseEntity<String>>> fn) {
	}
}

public static class AcceptCompletion extends Completion {
	public AcceptCompletion(Consumer<ResponseEntity<String>> con) {
	}
}
```

그리고 intelliJ에서 fn, con 등에 커서를 위치시키고 option+enter를 수행하고, "Create field for parameters"를 실행한다.

![](https://api.monosnap.com/rpc/file/download?id=skChm53IdIQUFzaaduaQM1movTqOss)

그리고 사용되지 않는 생성자 `Completion(Consumer<ResponseEntity<String>> con)`,  `Completion(Function<ResponseEntity<String>, ListenableFuture<ResponseEntity<String>>> fn)`를 삭제한다.

## 3. Push members down

이제 서브클래스로 이동할 `run` 메소드에 커서를 위치키시고, `push members down` 리팩토링을 실행한다(이때 run method를 abstract로 선택한다).

![](https://api.monosnap.com/rpc/file/download?id=NGLnUqlj08PYHbHrm3RHPanOYK6c4e)

Completion 클래스 선언부와 run 메소드의 abstract 키워드를 제거하고, run 메소드는 아무 일도 하지 않도록 수정한다.

![](https://api.monosnap.com/rpc/file/download?id=TS39091kvFP0ejWwHb8pz6nq72hJWi)

## 4. Implement Subclass Methods

ApplyCompletion, AcceptCompletion의 run 메소드에서 사용되지 않을 코드를 제거하여 올바로 동작하도록 한다.
타입에 대한 의존도를 별도의 서브클래스로 추출했으므로 run 메소드의 조건분은 이제 필요 없다.
이때 Completion#complete, error 메소드를 private에서 protected로 변경하여 서브클래스에서 호출 가능하도록 변경한다.

![](https://api.monosnap.com/rpc/file/download?id=gil8ChNT74SDuVhsvJjcdbdizEBEBs)

마지막으로 서브클래스로 이동되어 사용되지 않는 변수들을 정리한다.

![](https://api.monosnap.com/rpc/file/download?id=r8738bka6eHg5MJCkn9PcFfQyNR3w9)

## 참고

[Switch 문장을 Polymorphic Dispatcher로 치환하기](https://github.com/msbaek/videostore)
[Expense 예제](https://github.com/msbaek/expense) - `Replace Type Code with Subclasses` 부분