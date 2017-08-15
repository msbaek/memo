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

## Introduction

## Why Used

## Aggregate Rules of Thumb

## Modeling Aggregates

## Choose Your Abstractions Carefully

## Right-Sizing Aggregates

## Testable Units

## Summary