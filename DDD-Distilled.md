# DDD-Distilled

https://www.safaribooksonline.com/library/view/domain-driven-design-distilled/9780134593449/

[TOC]

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

## Delivering Ubiquotous Language

![](https://api.monosnap.com/rpc/file/download?id=Bt0A9m1ik77kitxVmKhRYcYxKL1jvh)

시나리오를 완성시키기 위해 다음과 같은 질문을 해 본다.

1. 누가 이 행위를 하는가 ?
2. 누가 이 행위에 관심이 있는가 ?

![](https://api.monosnap.com/rpc/file/download?id=UHSGgIJf267G4ayCbueEdYootqlCrL)

![](https://api.monosnap.com/rpc/file/download?id=Lidpvg7Vh9JdzMpcJ3aNIwJ3JRNW4H)

![](https://api.monosnap.com/rpc/file/download?id=VlufNPyjzDOWNr5CbYpIfHZeiExuP2)

![](https://api.monosnap.com/rpc/file/download?id=eyB5DFZ9dgv66QgDNKWRKPqzOywWAf)

![](https://api.monosnap.com/rpc/file/download?id=tNHffvnJazBzCObPQEzWTv8PT6k2Or)

이런 수준의 잘 작성된 시니라오가 있으면 테스트를 작성할 수 있다.

![](https://api.monosnap.com/rpc/file/download?id=PeLti7190je3eEjsyMbgB6HR1pkBN9)

![](https://api.monosnap.com/rpc/file/download?id=vaV1NiUjFYg6Imnf3802ov9AsE7cH5)

## Architecture

![](https://api.monosnap.com/rpc/file/download?id=HLvvLqkeg7xTFFm9nUoXOWBNKHw1yz)

![](https://api.monosnap.com/rpc/file/download?id=UYyiUcBl0PhEzpGOAr0If8R4gq0lwH)

이외에 

- EDA: event sourcing
- CQRS 
- Reactive and Actor Model
- REST
- MSA

등이 존재

# Strategic Design with Subdomains

BC : SD = 1 : 1

## What Is a Subdomain?

![](https://api.monosnap.com/rpc/file/download?id=X4Ntf181J9EMJdgMbGAR5QvXnDBxbS)

![](https://api.monosnap.com/rpc/file/download?id=qthjvfowZZ4dRFxTbsaXjX7YDYdVOZ)

![](https://api.monosnap.com/rpc/file/download?id=6NwIprFSb522Gyg4ZrtzrXldsNbcuZ)

## Types of Subdomains

![](https://api.monosnap.com/rpc/file/download?id=3dDtRrbixVUnslZsesULPMzy7hRChx)

![](https://api.monosnap.com/rpc/file/download?id=Q9uYfNgVtc0JOaRBzZr2Ftln1MURCX)

![](https://api.monosnap.com/rpc/file/download?id=Z9yn6vz7nQdXvnVuN3TZARnZnlU2s6)

## Dealing with Complexity

![](https://api.monosnap.com/rpc/file/download?id=Y8Tj01nQKzm473Nx7Ak10VA0b6dcai)

![](https://api.monosnap.com/rpc/file/download?id=XdP3ozgckwuc5XbjK5rkbSiv3z1Jb4)

# Strategic Design with Context Mapping

![](https://api.monosnap.com/rpc/file/download?id=xi1qkOKzv5oru2musFDmEVo39GZ01C)

## kinds of Mappings

![](https://api.monosnap.com/rpc/file/download?id=Skf3B6FWyo17wxOzhZlXNiJ09yTuXT)

## Partnership

![](https://api.monosnap.com/rpc/file/download?id=JWUsYi4eMvlfnEvH0vk9WCQUHLu5JU)

두개의 팀이 두개의 BC에서 일하는데 공통의 목적. 상호 지원. 
강력한 의존성(그래서 두꺼운 선으로 표현)

## Shared Kernel

![](https://api.monosnap.com/rpc/file/download?id=xzTRT6nQcLu5FuMoR00N3vLBgu1zmH)

## Customer-Supplier

![](https://api.monosnap.com/rpc/file/download?id=Ms2Ag9lnnLGKN39lVf85r2tf78FC3f)

upstream downstream 관계가 두 팀간에 존재하는 경우

## Conformist

![](https://api.monosnap.com/rpc/file/download?id=HR9bllZ5dhdfH71Qbqo2smqCDA6gHT)

## Anti-corruption Layer

![](https://api.monosnap.com/rpc/file/download?id=rlmCMz7YEGQAWlmtKd0sQtEOd4YExf)

ACL 때문에 team2의 모델은 team1의 모델과 다를 수도 있다(위 그림처럼)

## Open Host Service

![](https://api.monosnap.com/rpc/file/download?id=hlbuZoYOz6IkHxp4xs2Pnf6XI0aAY8)

## Published Language

![](https://api.monosnap.com/rpc/file/download?id=jUCO5gDUArTHyqEikobwr8K2346KiE)

## Separate Ways

![](https://api.monosnap.com/rpc/file/download?id=AJA6c0pEPocscUoPIFB7EwzznZkISs)

어떤 이유에서 team2가 team1에서 받은 모델을 변경하여 독자적인 방식으로 사용하는 경우

## Big Ball of Mud

![](https://api.monosnap.com/rpc/file/download?id=4NYmKdBmJ7LrYIgPMP14PNtpM4J7xe)

## Making Good Use of Context Mapping

![](https://api.monosnap.com/rpc/file/download?id=wO5qpEv9L4b6yk2aed3rpVU9DqmuzD)

## RPC with SOAP

![](https://api.monosnap.com/rpc/file/download?id=IA3cwQKrr0zTfP2sNlHytIgTPhq36i)

![](https://api.monosnap.com/rpc/file/download?id=E9SQF7FRv1KDfxdYiyjInjroaTOA09)

![](https://api.monosnap.com/rpc/file/download?id=ASUcn1w7OMVf4YKWY064yVk4FSXtD8)

## RESTful HTTP

![](https://api.monosnap.com/rpc/file/download?id=vmeAcUPXXQ3fE0vivPsXMnlkc0lU13)

![](https://api.monosnap.com/rpc/file/download?id=O8dM7Wyw14hEIXlXT6gTScMmavokTM)

## Messaging

![](https://api.monosnap.com/rpc/file/download?id=hcJf6pfJ66BxV1qNz56L9v0GMReFS6)

![](https://api.monosnap.com/rpc/file/download?id=Gs8d0iemjU9JwFftadI1THWd38mCGj)

![](https://api.monosnap.com/rpc/file/download?id=CyxSQNGAuFJXSEBbmR7imidSvVG016)

![](https://api.monosnap.com/rpc/file/download?id=QdGzj8rPqtAV99gL9u9DQRiS2r5r4S)

## An Example in Context Mapping

insurance company를 예제로

![](https://api.monosnap.com/rpc/file/download?id=EGMdzyQXkrDDs670BqXHPjDjpxRKvd)

![](https://api.monosnap.com/rpc/file/download?id=ySo7jk4MwjARKYw1pkvX6kY5afiqTs)

![](https://api.monosnap.com/rpc/file/download?id=H7I3uo0InVAQTLBlR9HyxP3e6e2BvP)

![](https://api.monosnap.com/rpc/file/download?id=jDrM2Thc7MFgDpa3DYH8U9PUP0OXyi)

![](https://api.monosnap.com/rpc/file/download?id=lEHdJBszXhSByYcQZO4oXFzEjXly2n)

![](https://api.monosnap.com/rpc/file/download?id=Egw7weAjN5wipRQ9WD0eIATzCm0xqD)

## Summary

![](https://api.monosnap.com/rpc/file/download?id=cH4uuo3LKFpTmQY8LFrwOTtQ72b78c)

# Tactical Design with Aggregates

aggregate은 관련된 클래스들의 집합에 대해서 txn을 이용해서 consistency를 유지하도록 설계하는 tactical design 전략이다.

## Introduction

![](https://api.monosnap.com/rpc/file/download?id=pPjbvemn3d2MF1BTbjazJueQNvecz7)

## Why Used

![](https://api.monosnap.com/rpc/file/download?id=9pRXaCvuBx32nnAqM58aV6woEbAeoR)

aggregate의 state를 제어하는 txn들. 하나의 txn은 하나의 aggregate의 상태만 담당

## Aggregate Rules of Thumb

![](https://api.monosnap.com/rpc/file/download?id=56pAhSc6ZEPl1xsJGPt7C8C2Dfy1O0)

### Rule 1: Protect Business Invariants

![](https://api.monosnap.com/rpc/file/download?id=P2cYt0kErjsSLIm3DMUgGb9bbDlcMO)

각 aggregate은 별도의 txn에 의해 처리된다.

![](https://api.monosnap.com/rpc/file/download?id=WzBqIuV49t4kfMKNvcFJEvVjtegB3q)

protecting business invariant(constraint)

### 2. Rule 2: Design Small Aggregates

큰 aggregate의 문제점

![](https://api.monosnap.com/rpc/file/download?id=k65d7oKkuYydRjUkYKNaVJyhr2ofpG)

1. transactional failure

- 서로 다른 사용자들이 서로 다른 목적으로 같은 aggregate을 각각의 txn에서 변경하고자 하기 때문.

2. memory

- aggregate이 크면 메모리를 많이 차지한다. 부하/GC를 유발한다.

product aggregate을 4개의 aggregate로 나눴다.

![](https://api.monosnap.com/rpc/file/download?id=Hrtq0nz3eVehDQnIa2RPPmKzTd4gc5)

prevents
- concurrency failure
- large memory footprints(gc)

### Rule 3: Reference by Indentity Only

![](https://api.monosnap.com/rpc/file/download?id=0Wq1ESTcqnlFQ7VTojLfMx76RdLb8x)

각각의 child aggregate은 parent를 알아야 한다.

어떻게 parent(product)를 참조할까 ? 직접 참조는 없다. 단지 productId를 통해서 parent를 참조한다.

이 구조는 몇가지 장점을 갖는다.

1. smaller memory footprint: child aggregate을 로딩할 때
2. 동시에 parent와 child를 수정해도 변경에 대해서 접근(reach out)할 수 있다.
8:10
어떤 하나의 child aggregate을 수정하면서 동시에 parent aggregate에 직접 접근하여 변경할 수 없다.
하나의 aggregate에 제약 조건을 부여하고, 하나의 txn을 관리하여 1번째 규칙을 지킨다.

### Rule 4: Update Other Aggregates Eventually - Eventual Consistency

![](https://api.monosnap.com/rpc/file/download?id=7jQVkZ20RUWQHXGQhQBxEHDzqHGX1I)

![](https://api.monosnap.com/rpc/file/download?id=ZH9BrEQSlBSTgNmkzxWxldhzENXYWF)

BacklogItem이 commit되면 BacklogItemCommitted domain event를 발생시킴. 그러면 별도의 txn에서 sprint가 처리된다.

![](https://api.monosnap.com/rpc/file/download?id=De0UYpG9rS07NhsuyQuXSDZk9eAozh)

message publish/subscribe에 의해 처리됨.

![](https://api.monosnap.com/rpc/file/download?id=4a6okQ6Jjnv620dJ1se987dLxJFrw0)

## Modeling Aggregates

### watch out the hook

anemic domain model(behavior가 없는)는 지양해야 한다.

![](https://api.monosnap.com/rpc/file/download?id=ZEiqrqqOapSKgVvLwEdwhacngkzwK4)

aggregate은 transactional consistency boundary이다.

aggregate root는 aggregate을 나타낼 수 있는 이름을 갖도록 한다.

![](https://api.monosnap.com/rpc/file/download?id=2kcvbyxNlgr6cPAVS0CaWel89LKrKG)

![](https://api.monosnap.com/rpc/file/download?id=eWIDIx7h7sS3ybt6dPNtAdpG4tFPfO)

product은 2개의 value object(ProductId, TenantId)를 갖는다.

![](https://api.monosnap.com/rpc/file/download?id=r1fWzwsAb7kQPwuRylolySrGdc8XSA)

추가적인 필드(description, name)을 갖는다.

setter는 private, getter는 public으로. 왜 ?

주의하지 않으면 anemic domain model로 빠지기 때문이다. anemic domain model은 대개 public getter/setter를 갖는다. root entity나 다른 entity의 데이터를 어떤 행위도 거치지 않고 변경할 수 있다.

![](https://api.monosnap.com/rpc/file/download?id=BhLhTRTq7nUz4h9LBHJbMvl5301KAw)

## Choose Your Abstractions Carefully

![](https://api.monosnap.com/rpc/file/download?id=jkT8rGTSAsGBGNZoX3Ys1tjoHd3VUa)

scrum domain에서 naturally 언급하는 것 이상 추상화를 하면 아래와 같은 결과를 얻을 수 있다.

![](https://api.monosnap.com/rpc/file/download?id=U679GCn0Pwhmeqcf3O3IEpSkvwwLdU)

이 경우 어떤 문제가 있을까 ?

![](https://api.monosnap.com/rpc/file/download?id=nUHyeTadC2b4qNfj15ay7CazHMRw1W)

ubiquotous lang.별로 모델링하라.

![](https://api.monosnap.com/rpc/file/download?id=Rr80EZfU3jrRfzCOBksz9p6FjU967P)

## Right-Sizing Aggregates

![](https://api.monosnap.com/rpc/file/download?id=s0U7WWWB3vodb8mxMgW5KI6pyeaouO)

어떻게 적절한 사이즈인지 알 수 있을까 ?

![](https://api.monosnap.com/rpc/file/download?id=O2mDdtnf95Kyp1ObBrPkt2xmmDkl8U)

1. rule 2: 처음엔 1개의 entity를 갖는 aggregate로 시작
점차 어떤 entity들이 필요한지 알게 된다.
2. rule 1
4. 

![](https://api.monosnap.com/rpc/file/download?id=y9Kv4dGNChGhdh3RrsqXaAV6Wb3aYd)

위와 같이 A1이 변경되었을 때 A2가 immediate하게 변경되어야 한다면 A1, A2는 같은 aggregate(Aggregate A 1 2)에 있어야 한다.

A1과 C14는 eventual consistency가 허영된다. 이럴 경우 domain event를 발생시켜 갱신한다.

## Testable Units

aggregate design이 unit testable하길 원한다.

![](https://api.monosnap.com/rpc/file/download?id=Mc8RvT74Xa0FcGkssmQAEFzU58ka7x)

## Summary

![](https://api.monosnap.com/rpc/file/download?id=lYBfcxMOnya8q1ZSVI1Q88FmCnDPjJ)

# 6 Tactical Design with Domain Events

## Introduction

![](https://api.monosnap.com/rpc/file/download?id=SbAiYPFF09i9YaV56C7UUYrbsqXdsy)

casual consistency

![](https://api.monosnap.com/rpc/file/download?id=rqKYQBdWRfDuzeQVQTMWxg2RPbCt5U)

cause가 effect/response 전에 보여야 한다. domain event가 발생한 순서대로 처리되어야 하는 것이 casual consistency이다. 

## Designing, Implementing, and Using Domain Events

![](https://api.monosnap.com/rpc/file/download?id=pELQQGuLJrI0jRfPIMUh5umtIgrVq1)

모든 이벤트를 위한 인터페이스를 정의하라

![](https://api.monosnap.com/rpc/file/download?id=hfup9lazOs21oPpI3XajoSgcmj8I4f)

이벤트 naming에 유의하라. noun + past tense

domain event의 cause는 무엇일까 ?

![](https://api.monosnap.com/rpc/file/download?id=vzxHespNOzkMwG7SPPPO3lhfVT4ko7)

상품 생성과 관련된 커멘드(Create Product)이다.

![](https://api.monosnap.com/rpc/file/download?id=DCYnOa7EZC4SonmhWzGzdqZsPMqEKB)

커멘드는 원하는 기능에 필요한 데이터를 갖는다.

![](https://api.monosnap.com/rpc/file/download?id=vF6Tuxnre2X1JiPkVss1CLZwaRgJ7V)

이벤트는 커멘드와 같은 데이터를 갖는다.

![](https://api.monosnap.com/rpc/file/download?id=9uUV4nEE5ObkzOGLfcNpFf71ru648U)

앞에서 보여준 event들은 위와 같이 속성을 갖는다.

![](https://api.monosnap.com/rpc/file/download?id=4MAa18gunVLSdV0MQ6QinQhsj7mZk7)

CommitBacklogItemToSprint command는 BacklogItem Aggregate에 영향을 미친다.
BacklogItem Aggregate은 BacklogItemCommited domain event를 발생시킨다.

![](https://api.monosnap.com/rpc/file/download?id=U0aRmjKaoqXelHgvEcMIZJVqkVYdWS)

domain event은 event table에 저장된다. 하나의 트랜잭션에서 BacklogItem도 DB에 commited되고, BacklogItemCommited domain event도 commited된다. save도 함께 되고, rollback도 함께 된다.

![](https://api.monosnap.com/rpc/file/download?id=8Mtps9SjjvXpsoXZJuJhnpCnoYTCxw)

domain event가 event table에 저장된 후에 messaging mechanism에 의해 subscribing bc에 전달된다.

![](https://api.monosnap.com/rpc/file/download?id=t5Q1AcgQZpNoNYTC5s30XNthmcsHOo)

## Event Sourcing

![](https://api.monosnap.com/rpc/file/download?id=5tQ80U3Us3peGj8gJeKYjUjc2j9y3s)

event가 발생하면 event stream으로 DB의 event log에 저장된다.

이 상태를 다시 만들려면 domain event를 발생한 순서로 읽어서 순서대로 적용하면 됨.

1. CommitBacklogItemToSprint 커맨드 실행.
2. BacklogItem Aggregate은 BackLogItemCommited 이벤트를 발생키시고
3. 이 이벤트는 event table에 저장됨
4. Planned, Defined, Commited 등 생성된 순으로 읽어서 BacklogItem에 reapply하면 최종 상태가 다시 생성됨

![](https://api.monosnap.com/rpc/file/download?id=Cp8v4vjFHxaf7H2bnbNXPVf28Gvavn)

## Summary

![](https://api.monosnap.com/rpc/file/download?id=9CgeDiTeT1hxLpdmJ81ou93dV16Pki)

# 7 Acceleration and Management Tools

## Introduction

model effectively vs time matters

acceleration / mgt. tool

no estimates: 하루 이상은 정확치 않더라. estimate도 시간이 든다. 정확치도 않고.

![](https://api.monosnap.com/rpc/file/download?id=udOaTeiica8p3KEQQkwZhPoj0jGMZo)

![](https://api.monosnap.com/rpc/file/download?id=reTPa8SzyzmTai9rqI4p16my6WpLkZ)

## Event Storming

accelerated modeling & knowledge crunching technique

![](https://api.monosnap.com/rpc/file/download?id=fP51hFWVt6uTD80bMse9yenIUxzsgu)

event storming을 통해서 비지니스 프로세스에 집중하고 Data, DB, Data Model 등에 집중하지 않는다.

먼저 비지니스 프로세스에서 일어나는 오퍼레이션에 집중한다. 그리고 이를 도메인 이벤트로 모델링한다. 또 도메인 이벤트의 cause도 모델링(command)하고, command가 동작하는 aggregate, entity 등을 모델링한다. aggregate 등은 도메인 이벤트를 발생시킬 수 있다.

이벤트 스토밍을 할 때는 팀의 모든 멤버(도메인 전문가, 개발자 ...)가 많은 것을 함께 열심히 공부해야 한다. 도메인 스토밍은 항상 동작하는 래피드한 모델링 기법이다.

1. right people

적어도 하나 이상의 도메인 전문가와 개발자들이 참여해야 한다. 연습에 도메인 전문가가 빠지면 안된다. 이벤트 스토밍은 기본적으로 많은 Right 질문을 하는 것이다. right 질문은 right people만 할 수 있고, 답변도 그들만 할 수 있다.

2. Modeling Supplies

![](https://api.monosnap.com/rpc/file/download?id=iyc2vbnAOyxou9okOTYDWRwsDJRPf9)

event: orange sticky
command: blue sticky
aggregate, entity: light yellow sticky
fine tip black marker
넓은 화이트 보드(?)

3. Modeling Elements
![](https://api.monosnap.com/rpc/file/download?id=hoa9ew0KzShrW0g2swfW65v2a26VVw)

### Step 1

![](https://api.monosnap.com/rpc/file/download?id=JSBorSsATF4NNBDaQEtV2xwXLuctUc)

첫번째 발생 가능한 이벤트로 시작하지 않아도 됨.

이벤트를 발생하는 순서대로 배치. time series로

### Step 2

![](https://api.monosnap.com/rpc/file/download?id=uZ1kGeNLECs0q4DWt9Wqmijq4Mlv7x)

어떤 커맨드는 다수의 이벤트를 발생시킬 수도 있고, 병렬적으로 이벤트를 발생시킬 수도 있다.

### Step 3

![](https://api.monosnap.com/rpc/file/download?id=I2eQ3o9M6ZNDgelKz3frU6YU8zdidY)

aggregate이 여러번 필요하면 중복된 sticky를 만들어서 사용. 시간 순으로 배열하는 것이 중요.

### Step 4

![](https://api.monosnap.com/rpc/file/download?id=4LNfvqnfW6wJrPWEoR6pc1ebbXibxx)

어떤게 core domain이고, 어떤게 조직에 더 중요한지 식별하는 것이 중요.

### Step 5

![](https://api.monosnap.com/rpc/file/download?id=aPGS71vCjhHzZkppJ0TFlkoHtdfid5)

## Managing DDD on an Agile Project

![](https://api.monosnap.com/rpc/file/download?id=upe9Ibr6IrdhhqD3UsfMpOY78R0Tbe)

scrum, kanban 혹은 다른 애자일 프로세스 기법을 쓰는지는 중요치 않다.

## First Things First

![](https://api.monosnap.com/rpc/file/download?id=5mpEg1jKvJWjdKsNuodZwMphiE630N)

![](https://api.monosnap.com/rpc/file/download?id=JGf1efClvy3Cz9ncFEBC6kDDxPaZul)

## Modeling Spikes and Modeling Debt

inception 단계에서 modeling spikes

![](https://api.monosnap.com/rpc/file/download?id=qf1aK472hmYjSeYPTJA3ksDLAVdidB)

![](https://api.monosnap.com/rpc/file/download?id=AxJtFMEoYmYFUzNnF7OR2enQW6zlpo)

## Identifying Tasks and Estimating Effort

![](https://api.monosnap.com/rpc/file/download?id=NSamVrFoILTXTsNs3zgbKqfb74QQc5)

![](https://api.monosnap.com/rpc/file/download?id=LMiuzvkcLAJ2NaGBkbr26ipUZKuHgq)

estimation unit을 hours를 적용해서 추정

table of estimation units는 unit test를 포함할 수 있다.

## Timeboxed Modeling

![](https://api.monosnap.com/rpc/file/download?id=VWxB1k938Q0WFX1Xl3M96l8np5XksO)

시간 단위 time boxing. 중요한 것은 시간을 지키려는 노력.

완료 후에 투입된 시간을 적고, 추정된 시간과 다른 경우 이유를 알아야 한다. 그래야만 추정을 잘 할 수 있게 된다.

이런 time boxing modeling을 위해 estimation unit을 사용하고, 트래킹하라.

![](https://api.monosnap.com/rpc/file/download?id=ZqUu9RLl4kTwpFcJMzWq9aw8I48vIe)

![](https://api.monosnap.com/rpc/file/download?id=luZlydCOuyH1i5JiZdK0cjovfo6f5m)

## Summary

![](https://api.monosnap.com/rpc/file/download?id=QDCQwacsvNvGwQFMqCWBGt0WN60335)

# Summary

![](https://api.monosnap.com/rpc/file/download?id=FzRvnQGE6HpsVmidZenos34juIeKNV)

![](https://api.monosnap.com/rpc/file/download?id=AIlr3ooeyjbKPJWOOFxrkJVVvSCMQ6)
