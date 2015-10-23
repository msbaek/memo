안드로이드 애플리케이션을 이루는 주요 구성 요소

- Activity : 가장 메인이 되는 컴포넌트로, 모바일 앱의 특성상, 모바일앱은 하나의 UI가 떠서 사용자로 부터 입력을 받고, 출력을 담당한다. 즉 하나의 화면 인터페이스에 해당한다고 보면된다.
- Service : 백그라운드에서 도는 컴포넌트로 UI가 없이 동작한다. 가장 쉬운 예로 음악 플레이 처럼 화면이 없는 상태에서 백그라운드로 도는 케이스가 가장 대표적인 예이다.
- BroadCastReceiver : 이벤트를 처리하는 컴포넌트로, 안드로이드의 Intent를 받아서 처리한다. 이 Intent는 Pub/Sub형태로 바인딩되며, 특정 intent가 발생하면, 이를 subscribe하는 BroadCastReceiver가 이를 받아서 처린한다.
- ContentsProvider : ContentsProvider는 일종의 Database를 추상화 해놓은 개념으로 , 단순히 데이타를 저장하는 것 뿐만 아니라, 저장된 데이타를 다른 앱간에 공유하는 기능도 지원한다.

![image](http://developer.android.com/images/activity_lifecycle.png)