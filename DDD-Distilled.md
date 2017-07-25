# Vaughn Vernon and Implementing Domain Driven Design

# 1. Introduction
## scrum
todo - doing - done으로 타스크를 옮기는 것이 전부가 아니다(Task-Board Shuffle).
무슨 일을 하기 전에 먼저 SW 설계에 대해서 논의해야 한다.
scrum은 지식을 습득하는 수단이 되어야 한다. 이런게 설계다.

Task-Board Shuffle & No Design lead to developing a Big Ball of Mud

## intetional design
구불구불하고 오르막과 내리막이 많이 있는 길을 볼 수 있다. 이 길이 어떻게 생겼을까 ? 이 길은 최초에는 작은 손수레들이 지나다니던 길이였을 것이다. 그러다 시간이 지나면서 조금씩 넓어지고, 도로 포장이 되면서 자동차들이 다니게 되었을 것이다.
이런 길로 자동차를 빠르게 운전하는 것은 불편하고 위험하기까지 하다.
아주 빠른 속도로 다닐 수 있는 수퍼하이웨이를 생각해 보자. 이 경우 누구도 구불구불하거나 오르막과 내리막이 반복되게 만들진 않을 것이다.
의도를 가지고 접근했나가 이러한 차이를 만든다. SW 설계도 마찬가지다 의도를 가지고 설계를 해야 한다.

## 1/2. Strategic Design Tools
DDD에서 가장 중요한 것. BC. Ubiquitous Language, Context Mapping, sub-domains,

## 2/2. Tactical Design Tools
aggregate
	- help you to control txn
	- maintain business invariants with data consistency with your model
    
domain events
	- domain model에서 중요한 어떤 일이 있어 났음을 모델링하는데 도움이 됨
    
# 2. Strategic Design with Bounded Contexts and the Ubiquitous Language
## BC, UL
독일인이 독일어를 사용하고, 프랑스인이 불어를 사용하듯 BC에 따라 UL이 달라진다.
심지어 스페인과 콜롬비아에서 사용되는 스페인어는 서로 다를 수 있다. 유사한 UL도 BC에 따라 의미가 달라질 수 있다.

![](https://api.monosnap.com/rpc/file/download?id=qXxfTkfWOmANCjFcXInI9JT5AKaEa1)

BC내에서는 UL로 의미로 갖지만 BC를 벗어나면 그 의미를 갖지 않게된다.

BC가 없다면 모델이 점점 커졌을 때 Big Ball of Mud에서 허덕이여야 할 것이다.

## Domain Experts and Business Drivers

![](https://api.monosnap.com/rpc/file/download?id=KaZFKOES7kwxHq6YFznZkWtRPLyccE)

underwriting, claims, inspections 등의 도메인 영역에 Policy라는 개념이 존재한다. 이를 하나의 개념으로 표현한다면 각 영역의 도메인 전문가들이 자신의 영역에 유리하도록 정의하려고 할 것이다. 어떻게 모델링해야 할까 ?

![](https://api.monosnap.com/rpc/file/download?id=AL0Ja9JTz6KcMJQMKXRYDZ50ZMVRez)

DDD에서는 각 영역에 맞도록 각각의 모델을 갖도록 한다.

업무 영역간의 차이를 수용하고, 각 영역에 맞도록 명식적으로 모델링한다.

## Case Study - Scrum Project Mgt. Tool

초기 모델 

![](https://api.monosnap.com/rpc/file/download?id=htYBgFt5jUTcWvpGnUPH27CP6DVrmm)

구독자가 추가된 모델

![](https://api.monosnap.com/rpc/file/download?id=IN9WAXvmmVV4RXInzRvEq6TTyffVhl)

forum 논의가 추가된 모델

![](https://api.monosnap.com/rpc/file/download?id=hJaFNWGc32ro5u61CdJpQQFbbqlCPx)

tenent의 account가 추가된 모델

![](https://api.monosnap.com/rpc/file/download?id=HXIvjlBR46GvMYuPYPRXvrDWHRa0Gq)

team, po가 추가된 모델

![](https://api.monosnap.com/rpc/file/download?id=HnvI51uzTOqqjfwT1DSnNTfiGpWf82)

지속적으로 모델이 추가되면서 Big Ball of Mud가 되었다.

![](https://api.monosnap.com/rpc/file/download?id=kbYFIyWCh2Ip6oyIqC5NPHoUaob56t)

## Fundamental Strategic Design Needs

big ball of mud를 피하기 위해 어떤 툴을 써야 하나 ? BC & UL

BC
helps us to model what is in context
semantically what belongs in our bc according to our UL
and what is out of context ? What should not be in our model ?
우리 모델에 무엇이 있나와 무엇이 없나는 동일하게 중요하다.

도메인 전문과와 개발자가 cohesive하게 함께 작업해야 한다.

tenent, user, permission이 이 프로젝트를 위해서는 필요하지만 scrum이라는 업무 영역에는 안 맞다.
out시킨다.

![](https://api.monosnap.com/rpc/file/download?id=nBhtlc9c4Glse0UybMhGGP1S5RfU1e)

account 관련 모델들도 마찬가지다.

![](https://api.monosnap.com/rpc/file/download?id=c46yGMNnED1VZaBdWPuG4ssQytzkeQ)

time consuming resources 관련 모델들은 scrum team의 전체적인 오퍼레이션에는 매우 중요하지만 scrum 모델에 포함되지는 않는다. out of context이다.

![](https://api.monosnap.com/rpc/file/download?id=Gw8PducvLjATETTUs24560Qux2UUsW)

bc간을 연결해서 사용

![](https://api.monosnap.com/rpc/file/download?id=oKpS3FvcMOSppqKYfMwfT4WnKOWeNs)

core domain은 꽤 작다.

![](https://api.monosnap.com/rpc/file/download?id=CMyEe5T7WeF9fi6Jcv28IokwTDp6Ba)

bc에서 뺀 모델들은 각기 자신의 BC를 갖느다.

![](https://api.monosnap.com/rpc/file/download?id=EW80A2GmcZitV49uiuRtZ4jRiUjBYg)

DDD 프로젝트에는 항시 다수의 BC가 존재한다.

VIDEO 16 OF 66 FROM 

