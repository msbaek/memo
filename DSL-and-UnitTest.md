# 요약
- 테스트 유틸리티 메서드(TEST UTILITY METHOD)
    - 픽스처 생성 코드를 메서드로 추출(EXTRACT METHOD 리팩토링)한 별도의 메소드
      1. extract creation method 
      2. parameterized creation method로 전환
        - 테스트와 무관한 정보는 기본값으로 설정

- 슈퍼 클래스를 이용한 생성 메서드의 중복 제거(TESTCASE SUPERCLASS)
    - 문제: 위 두 테스트 케이스 이외의 다양한 테스트 케이스에서도 이 2가지 버전의 createPlanet()을 사용할 수 있어야 한다는 것
    - EXTRACT SUPER CLASS 리팩토링을 통해 두 테스트 케이스 클래스의 공통 부모 클래스를 추출
    - PULL UP METHOD 리팩토링을 통해 생성 메서드를 공통의 부모 클래스로 옮기는 것

- FACTORY를 이용한 생성 메서드(CREATION METHOD) 중복 제거(TEST HELPER, Object Mother)
    - Test Case Super Class의 문제점
        - Java이 경우 단일상속 제약 존재
        - 다양한 픽스처를 생성하기 위해 많은 수의 생성 메서드를 슈퍼 클래스에 포함시킬 경우 관리와 유지보수에 많은 어려움
    - 슈퍼 클래스가 아닌 별도의 독립 클래스로 생성 메서드(CREATION METHOD)를 옮기는 것
    - 테스트 케이스 클래스 간의 중복 코드를 제거하기 위해 재사용 가능한 테스트 유틸리티 메서드를 제공하는 독립적인 클래스를 테스트 도우미(TEST HELPER)라고 함
	- Extract Delegate를 수행해서 PlanetFactory를 생성

- Test Data Builder

# [도메인 특화 언어와 단위 테스트 - 1부(上)](http://aeternum.egloos.com/2954659)

- XP를 위시로 한 애자일 진영이 소프트웨어 커뮤니티에 미친 가장 큰 영향
    - 소프트웨어의 품질을 좌우하는 핵심적인 설계 기법으로서 단위 테스트(Unit Test)의 지위가 격상
- 실패하는 단위 테스트 없이 코드를 작성하지 말라는 테스트 주도 개발(Test-Driven Development)의 기본 원칙
    - 리팩토링(Refactoring), 지속적인 통합(Continuous Integration)과 같은 피드백 기반의 실행지침과 결합됨으로써 
    - 안정적이고 신뢰도 높은 소프트웨어 개발을 가능하게 한다.
- 단위 테스트의 장점에도 불구하고 테스트를 아무렇게나 작성할 경우 프로젝트의 진행을 방해하고 소프트웨어의 품질을 저하시키는 걸림돌로 전락할 수 있다

## 예제 어플리케이션 - 지구 검증 소프트웨어

- 대기는 80%의 질소와 20%의 산소로 구성되어야 한다.
- 새로운 지구와 동일한 개수의 대륙과 대양을 포함해야 한다.
- 대기, 대륙, 대양 각각에는 제조가격이 책정되며 행성의 가격은 이 3가지 요소의 총합과 같다.
- 예산을 초과해서는 안 된다.

- 행성 검증이라는 도메인을 구성하는 대기, 대륙, 대양, 행성이라는 4가지 핵심적인 도메인 개념을 코드로 표현해 보기로 하자. 

### 대기(Atmosphere)
- 대기는 산소와 질소의 두 가지 원소로 구성. 8:2의 비율로
- 대기를 구성하는 원소(Element)를 구현해야 
- 원소는 이름(name)과 비율(ratio)의 두 가지 속성을 가짐

```java
public class Element {
    private String name;
    private Ratio ratio;

    public static Element element(String name, Ratio ratio) {
        return new Element(name, ratio);
    }

    private Element(String name, Ratio ratio) {
        this.name = name;
        this.ratio = ratio;
    }

    public Ratio getRatio() {
        return ratio; 
    }
} 
```

- 대기는 구성 원소의 목록과 가격 정보를 포함
- 제약 조건: 자신을 구성하는 원소의 개수가 2개 이상이어야 하고 모든 원소의 비율을 합한 값이 반드시 1이 되어야 함
- 이 제약 조건은 대기의 **불변식(invariant)의 일부이므로 생성시에 제약 조건을 만족하는 지 여부를 확인**해야 한다

```java
public class Atmosphere {
    private Money price;
    private List<Element> elements = new ArrayList<Element>();

    public Money getPrice() {
        return price;
    }     

    public Atmosphere(Money price, Element ... elements) {
        this.price = price;
        this.elements = Arrays.asList(elements);
        assertElements();
    }

    private void assertElements() {
        checkElementSize();        
        checkElementRatio();
    }

    private void checkElementSize() {
        if (elements.size() < 2) {
             throw new IllegalArgumentException("대기는 적어도 2개 이상의 원소로 구성되어야 합니다.");
        }
    }

    private void checkElementRatio() {
        if (!accumulateRatio().isOne()) {
            throw new IllegalArgumentException("전체 원소의 비율 총합은 1이어야 합니다.");
        }
    }

    private Ratio accumulateRatio() {
        Ratio result = Ratio.of(0);
        for(Element element : elements) {
            result = result.plus(element.getRatio());
        }
        return result;
    }
}
```

## 대륙(Continent)
- 지구의 육지는 하나 이상의 대륙으로 구성
- 대륙에는 국가(Nation)가 존재할 수 있다.
- 국가는 이름과 제조 가격을 속성으로 가지는 간단한 클래스로 추상화할 수 있다.

```java
public class Nation  {
    private String name;
    private Money price;

    public Money getPirce() {
        return price;
    }

    public Nation(String name, Money price) {
        this.name = name;
        this.price = price;
    }
}
```

- 대륙은 이름과 국가의 목록을 속성으로 포함
- 남극 대륙과 같이 국가가 존재하지 않는 대륙도 존재할 수 있으므로 nations 는 크기가 0인 빈 컬렉션일 수도 있다. 대륙의 제조 가격은 대륙 안에 존재하는 국가(Nation) 전체의 제조 가격을 더한 금액과 동일하다.

```java
public class Continent {
    private String name;
    private List<Nation> nations = new ArrayList<Nation>();

    public Money getPirce() {
        Money result = Money.wons(0);
        for(Nation each : nations) {
            result = result.plus(each.getPirce());
        }
        return result;
    }

    public Continent(String name) {
        this.name = name;
    }
    
    public Continent(String name, Nation ... nations) {
        this.name = name;
        this.nations = Arrays.asList(nations);
    }
}
```

## 대양(Ocean)
- 바다는 대양들의 집합으로 추상화할 수 있다.
- 대양은 이름과 제조 가격을 속성으로 가지는 단순한 객체다.

```java
public class Ocean {
    private String name;
    private Money price;

    public Money getPirce() {
        return price;
    }

    public Ocean(String name, Money price) {
        this.name = name;
        this.price = price;
    }
}
```

## 행성(Planet)
- 행성은 개발 중인 소프트웨어의 가장 핵심적인 도메인 개념
- 행성은 대기, 대륙의 집합, 대양의 집합으로 구성
- 3가지 속성은 반드시 존재해야 하는 필수 속성이며 행성은 대륙과 대양을 각각 최소 1개 이상 포함해야 한다.

```java
public class Planet {
    private Atmosphere atmosphere;
    private List<Continent> continents;
    private List<Ocean> oceans;
    
    public Planet(Atmosphere atmosphere, List<Continent> continents, List<Ocean> oceans) {
        this.atmosphere = atmosphere;
        this.continents = continents;
        this.oceans = oceans;
        assertValid();
    }

    private void assertValid() {
        if (continents.size() < 1) {
            throw new IllegalArgumentException("대륙은 1개 이상 존재해야 합니다.");        
        }
        
        if (oceans.size() < 1) {
            throw new IllegalArgumentException("대양은 1개 이상 존재해야 합니다.");    
        }
    }

    public Collection<Continent> getContinents() {
        return Collections.unmodifiableCollection(continents);
    }

    public Collection<Ocean> getOceans() {
        return Collections.unmodifiableCollection(oceans);
    }
}
```

- 행성의 제조 가격은 대기, 대륙, 대양의 제조 가격을 모두 더한 값이다.

```java
public class Planet {
    public Money getPrice() {
        return atmosphere.getPrice().plus(getContinentsPrice()).plus(getOceansPrice());
    }

    private Money getOceansPrice() {
        Money result = Money.wons(0);
        for(Continent each : continents) {
            result = result.plus(each.getPirce());
        }
        return result;
    }

    private Money getContinentsPrice() {
        Money result = Money.wons(0);
        for(Ocean each : oceans) {
            result = result.plus(each.getPirce());
        }
        return result;
    }
}
```

- 지금까지 구현한 지구 검증 소프트웨어의 최종적인 도메인 레이어 클래스들을 다이어그램

![Alt text](https://monosnap.com/image/FGPtsMrEhbu6LiP7aFQ8VtkatRfw54)

# [도메인 특화 언어와 단위 테스트 - 1부(下)](http://aeternum.egloos.com/2955485)

## 데이터와 데이터를 이용한 행위를 함께 둘지에 대한 기준 - 응집도, 결합도
- 기본 원칙은 데이터와 데이터를 이용하는 행위를 함께 두는 것
- 기준은 응집도와 결합도
- 영속성 로직
    - 엔티티 내에 데이터베이스 저장과 같은 영속성 로직을 위치시키는 ACTIVE RECORD 패턴보다는 별도의 독립적인 클래스로 영속성 로직을 분리하는 DATA MAPPER 패턴을 선호
    - 영속성 로직과 도메인 로직을 엔티티에 함께 섞는 것은 전체적으로 객체들의 응집도를 낮추고 결합도를 높이는 부정적인 결과를 낳는다.
    - 따라서 영속성 로직이 엔티티의 데이터를 사용하더라도 시스템 전체적인 관점에서 응집도를 높이고 결합도를 낮추기 위해서는 영속성 로직을 별도의 클래스로 분리하는 것이 더 좋은 선택
- 정합성 검증하는 조건 체크 로직
    - 응집도를 낮추고 결합도를 높인다면 독립적인 클래스로 정합성 검증 로직을 분리하는 것이 더 좋다
    - 특정 엔티티에 대한 정합성을 체크하는 로직을 엔티티 자체가 아니라 독립적인 객체에 두는 것을 SPECIFICATION 패턴이라고 함

![Alt text](https://monosnap.com/image/fe0sZBv2eBcrL0PPQuf8mS4gAaKIhg)

행성이 가지는 대륙의 개수가 2이고 제조 가격이 20,000원 이하인 지를 체크하는 경우 
```java
Specification specification = new ContinentSpecification(2)
    .and(new PriceSpecification(Money.wons(9000)));
specification.isSatisfied(planet);
```

## 단위 테스트의 늪

```java
class ContinentSpecificationShould {
    @Test
    public void specified_with_a_planet() {
        Planet planet = new Planet(
                new Atmosphere(Money.wons(5000),
                        element("N", Ratio.of(0.8)),
                        element("O", Ratio.of(0.2))),
                Arrays.asList(
                        new Continent("아시아"),
                        new Continent("유럽")),
                Arrays.asList(
                        new Ocean("태평양", Money.wons(1000)),
                        new Ocean("대서양", Money.wons(1000))));

        ContinentSpecification specification = new ContinentSpecification(2);

        assertTrue(specification.test(planet));
    }
}
```

픽스처 생성과 관련해서 다음과 같은 몇 가지 문제점
- 애매한 테스트(OBSCURE TEST)
    - 테스트에서 픽스처와 관련 없는 정보를 너무 상세하게 보여주기 때문에 테스트 중인 시스템의 동작에 실제로 영향을 미치는 것이 어떤 것인지를 파악하기 어렵다
    - 위 테스트 케이스에서 Planet의 인스턴스를 생성하기 위해 생성자에 전달된 세 가지 인자 중 대륙의 수를 검증하는 테스트에 직접적으로 관련이 있는 것은 Continent뿐
    - Atmosphere와 Ocean은 테스트 결과와 아무런 상관이 없다
    - 그럼에도 불구하고 Atmosphere와 Ocean은 Planet을 생성하기 위해 컴파일러가 요구하는 필수 파라미터이기 때문에 테스트와의 관련성과 무관하게 생성자에 전달해야 한다
    - 이처럼 테스트하려는 행위와 관련이 없는 불필요한 정보는 테스트의 목적을 흐리고 테스트 케이스를 이해하는데 불필요한 정신적 과부하를 초래
- 깨지기 쉬운 테스트(FRAGILE TEST)
    – 위 테스트 케이스는 Planet, Atmosphere, Continent, Ocean의 생성자를 직접 호출하기 때문에 생성자의 시그니처가 변경될 경우 객체들을 사용하는 모든 테스트 케이스를 수정해야 함
    - 이처럼 테스트 케이스에서 참조하는 객체의 API가 변경될 경우 테스트 케이스에서 컴파일 에러가 발생하는 것을 인터페이스에 민감함(INTERFACE SENSITIVITY) 문제라고 함
    - 도메인 객체들을 사용하는 테스트 케이스가 늘어날수록 테스트 케이스를 수정하고 관리하는 것이 점점 더 어려워짐
- Planet의 생성자를 하나의 테스트 케이스 클래스에서만 호출하고 있다면 큰 문제가 되지는 않는다
- 그러나 객체들의 풍부한 협력 관계를 기반으로 하는 대부분의 객체 지향 시스템은 동일한 클래스의 인스턴스를 여러 테스트 클래스의 픽스처로 사용하는 것이 일반적

```java
class PriceSpecificationShould {
    @Test
    public void specify_with_planet() {
        Planet planet = new Planet(
                new Atmosphere(Money.wons(5000),
                        element("N", Ratio.of(0.8)),
                        element("O", Ratio.of(0.2))),
                Arrays.asList(
                        new Continent("아시아",
                                new Nation("대한민국", Money.wons(1000))),
                        new Continent("유럽",
                                new Nation("영국", Money.wons(1000)))),
                Arrays.asList(
                        new Ocean("태평양", Money.wons(1000)),
                        new Ocean("대서양", Money.wons(1000))));

        Predicate specification = new PriceSpecification(Money.wons(9000));

        assertTrue(specification.test(planet));
    }
}
```

두 테스트 케이스 간에 발생하는 문제점은 명확
- 픽스처를 생성하는 부분에 중복 코드가 존재
- 테스트 코드 중복
    – 유사한 주제에 대해서 약간씩 다른 시나리오로 테스트를 해야 하는 경우 유사한 픽스처 로직이 여러 테스트 케이스에 중복되어 나타남
    - 테스트 케이스 간에 유사한 코드가 나타나야 하는 경우 이를 해결하는 가장 간단한 방법은 코드를 잘라 붙여넣기(CUT-AND-PASTE) 한 후 필요한 부분을 수정하는 것
    - 픽스처 생성과 관련된 테스트 코드 중복 문제는 특히 깨지기 쉬운 테스트의 원인이 됨
    - 여러 테스트 케이스에 생성자를 호출하는 코드가 중복되어 나타나기 때문에 생성자의 시그니처가 변경되면 중복 코드가 나타나는 모든 테스트 케이스를 수정해야 함

애매한 테스트, 깨지기 쉬운 테스트, 테스트 코드 중복 문제로 속을 썩이는 테스트 케이스는 다음과 같은 유지 보수 문제를 일으킴
- 높은 테스트 유지 비용
    – 애매하고, 깨지기 쉬우며, 중복 코드에 시달리는 테스트 케이스는 유지보수하기 어려움
    - 프로덕션 코드에 이런 문제가 발생한다면 최소한 리팩토링을 해보려는 시도라도 하겠지만 테스트 코드에 이런 문제가 발생하는 경우 시간에 쫓기는 대부분의 사람들은 @Ignore를 붙이거나 테스트 케이스를 삭제함
- 테스트를 작성하지 않는 개발자
    – 높은 테스트 유지 비용으로 인해 테스트가 개발 과정의 병목이라는 인식이 퍼질수록 개발자들은 테스트를 작성하지 않으려고 함
    
‘깨진 창문 이론’을 기억하라. 
테스트가 방치되기 시작하면 빠른 속도로 복잡하고 이해하기 어려운 테스트들이 단순하고 깔끔한 테스트의 비율을 잠식해 나갈 것
더 많은 창문이 깨지기 전에 테스트 케이스를 보수해야 한다.

# [도메인 특화 언어와 단위 테스트 - 2부(上)](http://aeternum.egloos.com/2957124)
## 테스트 코드 리팩토링
테스트와 관련 없는 정보를 너무 상세하게 노출시켜 애매한 테스트가 되어버리는 문제를 해결할 수 있는 한 가지 방법 - 안정적인 인터페이스를 이용해 픽스처를 생성하는 코드를 캡슐화시키는 것
- 가장 간단한 방법은 픽스처 생성 코드를 메서드로 추출(EXTRACT METHOD 리팩토링)해서 별도의 테스트 유틸리티 메서드(TEST UTILITY METHOD)로 분리하는 것

### 1. 생성 메서드(Creation Method)

![Alt text](https://monosnap.com/image/O0lMcjLMXlgG62wtEIK2AEpOY6vdko)

생성 메서드 내부로 픽스처 생성 로직을 캡슐화하는 장점
- 인터페이스에 민감함(INTERFACE SENSITIVITY) 문제로 인한 깨지기 쉬운 테스트의 위험성을 낮출 수 있음
    - 픽스처 생성 로직을 생성 메서드 내부로 캡슐화하고 직접 생성자를 호출하는 대신 생성 메서드를 호출하도록 변경하면 생성자의 시그니처 변경으로 인한 파급 효과의 범위를 제한
- 생성 메서드는 중복되는 코드를 별도의 메서드로 추출할 수 있기 때문에 제한적이나마 테스트 코드 중복 문제를 해결

리팩토링한 테스트 코드에서는 여전히 애매한 테스트 문제가 존재
- 리팩토링 전의 코드가 관련 없는 정보를 과도하게 노출시킴으로써 애매한 테스트 문제를 초래했다면 
- 리팩토링 후의 코드는 테스트 결과와 관련된 중요한 정보를 은폐하고 있기 때문에 애매한 테스트 문제를 초래
- createPlanet()이라는 메서드의 시그니처 어디에서도 대륙의 개수를 검증하기 위해 Planet을 생성한다는 정보를 알 수가 없다
- 이와 같은 경우를 애매한 테스트 문제의 또 다른 형태로 미스터리한 손님(Mysterious Guest)이라고 함
- 미스터리한 손님 문제를 해결하는 한 가지 방법은 
    - 테스트와 무관한 정보는 기본값을 설정하도록 생성 메서드 내에 캡슐화
    - 테스트와 관련된 정보만 생성 메서드의 파라미터로 전달함으로써 테스트 검증에 중요한 정보가 무엇인지를 한 눈에 파악할 수 있도록 하는 것
    - 테스트와 관련된 중요한 정보를 파라미터로 전달받는 생성 메서드의 특별한 형태를 매개변수화된 생성 메서드(PARAMETERIZED CREATION METHOD)라고 함

#### 1.1 매개 변수화된 생성 메소드(Parameterized Creation Method)
![Alt text](https://monosnap.com/image/56PCmEk2FzX5yeT2G5HqTMTLtpJ1mE)

![Alt text](https://monosnap.com/image/ccbfAwsI9XvsPLWXQhUkOPdzJAbXfJ)

#### 1.2 슈퍼 클래스를 이용한 생성 메서드의 중복 제거

새로운 문제는 위 두 테스트 케이스 이외의 다양한 테스트 케이스에서도 이 2가지 버전의 createPlanet()을 사용할 수 있어야 한다는 것
- 현재의 구조 상으로는 생성 메서드가 특정한 테스트 케이스 클래스와 강하게 결합되어 있기 때문에 다른 테스트 케이스 클래스에서 원하는 메서드를 재사용하기가 쉽지 않다.
    - 한 가지 방법은 각 테스트 케이스로 적절한 createPlanet() 메서드를 복사하는 것
    - 그러나 메서드를 복사하는 것은 테스트 코드 중복 문제로의 회귀를 의미
- 더 좋은 방법은 EXTRACT SUPER CLASS 리팩토링을 통해 두 테스트 케이스 클래스의 공통 부모 클래스를 추출한 후 PULL UP METHOD 리팩토링을 통해 생성 메서드를 공통의 부모 클래스로 옮기는 것
- Planet을 생성할 필요가 있는 테스트 케이스에서는 추출한 부모 클래스를 상속 받는 것으로 간단하게 생성 메서드(CREATION METHOD)를 재사용할 수 있음
- 이처럼 테스트 케이스 클래스 간의 중복 코드를 제거하기 위해 공통 코드를 포함하도록 생성되는 부모 클래스를 테스트 케이스 슈퍼클래스(TESTCASE SUPERCLASS)라고 함
- 생성 메서드는 서브 클래스에서만 접근 가능하도록 제한하는 것이 좋으므로 가시성을 protected로 변경한
- 생성 메서드를 부모 클래스로 이동 시킴으로써 테스트의 목적과는 무관한 생성 코드를 테스트 케이스 클래스로부터 제거

```java
public class AbstractPlanetTest {
    protected Planet createPlanet(Continent... continents) {
        return new Planet(
                new Atmosphere(Money.wons(5000),
                        element("N", Ratio.of(0.8)),
                        element("O", Ratio.of(0.2))),
                Arrays.asList(continents),
                Arrays.asList(
                        new Ocean("태평양", Money.wons(1000)),
                        new Ocean("대서양", Money.wons(1000))));
    }

    protected Planet createPlanet(Money wons, List<Money> nationPrice, List<Money> oceanPrice) {
        return new Planet(
                createAtmosphere(wons),
                createContinents(nationPrice),
                createOceans(oceanPrice));
    }

    private Atmosphere createAtmosphere(Money wons) {
        return new Atmosphere(wons,
                element("N", Ratio.of(0.8)),
                element("O", Ratio.of(0.2)));
    }

    private List<Continent> createContinents(List<Money> wons) {
        return wons.stream()
                   .map(w -> new Continent("대륙", new Nation("국가", w)))
                   .collect(Collectors.toList());
    }

    private List<Ocean> createOceans(List<Money> wons) {
        return wons.stream()
                   .map(w -> new Ocean("대양", w))
                   .collect(Collectors.toList());
    }
}
```

# [도메인 특화 언어와 단위 테스트 - 2부(下)(http://aeternum.egloos.com/2957825)

### 2. FACTORY를 이용한 생성 메서드(CREATION METHOD) 중복 제거
- 슈퍼 클래스가 아닌 별도의 독립 클래스로 생성 메서드(CREATION METHOD)를 옮기는 것
- 테스트 케이스 클래스 간의 중복 코드를 제거하기 위해 재사용 가능한 테스트 유틸리티 메서드를 제공하는 독립적인 클래스를 테스트 도우미(TEST HELPER)라고 함

Test Case Super Class의 문제점
- Java이 경우 단일상속 제약 존재
- 다양한 픽스처를 생성하기 위해 많은 수의 생성 메서드를 슈퍼 클래스에 포함시킬 경우 관리와 유지보수에 많은 어려움

1. Extract Delegate를 수행해서 PlanetFactory를 생성
![Alt text](https://monosnap.com/image/LLKAniShfWmDxt7bp3tvoitoXSpKb4)

Tese Case Super Class는 PlanetFactory를 사용하도록 변경되었음
![Alt text](https://monosnap.com/image/YOLK90SSJAcLChupuLaNMGK0cyGrc2)

테스트 케이스도 PlanetFactory를 사용하도록 변경되었음
![Alt text](https://monosnap.com/image/GUdikYhmh9Rvze0gk02pOmnk9W86Zx)

2. compile error 제거
정적 메소드 호출로 변경
![Alt text](https://monosnap.com/image/kkoRhJ1u7nKRiAW21CS8ksutTvtpKe)

이후 F2로 컴파일 에러 찾아서 'Show Context Actions'를 수행해서 에러 제거
![Alt text](https://monosnap.com/image/fEkWfYS541iDq6fzF5cmRkVIzosaRY)

### Object Mother
- 테스트에 필요한 픽스처의 생성, 수정, 삭제와 관련된 책임을 담당하는 독립적인 테스트 도우미
- Peter Schuh과 Stephanie Punke는 "ObjectMother - Easing Test Object Creation in XP"라는 논문에서 OBJECT MOTHER를 다음과 같은 기능을 제공하는 테스트 도우미로 정의하고 있다.
    - 필수적인 속성들을 포함하는 완벽한 상태를 가진 비즈니스 객체를 제공한다.
    - 생명 주기 내의 임의 시점에 요청된 객체를 반환한다.
    - 원하는 상태로 객체를 생성할 수 있는 편리한 방법을 제공한다.
    - 테스트 프로세스 중에 객체의 상태를 변경할 수 있도록 한다.
    - 필요한 경우 테스트 프로세스가 종료될 때 객체 및 관련된 모든 다른 객체들을 제거한다.
- 클래스 계층 구조와 무관하게 테스트 케이스 클래스로부터 픽스처 생성과 관련된 부담을 제거할 수 있음
- 그러나 FACTORY 기반의 OBJECT MOTHER는 구축된 시스템의 규모가 증가하면 증가할 수록 유지보수성의 한계에 봉착하게됨
    - 다양한 테스트 케이스로부터 픽스처 생성 로직을 PlanetFactory로 옮겨감에 따라 PlanetFactory에는 시그니처는 다르지만 구현 상으로는 유사한 매개변수화된 생성 메서드가 기하급수적으로 늘어나게 됨
    - 새로운 테스트 케이스를 추가할 때마다 요구되는 픽스처의 상태가 다양하게 변하기 때문에 시간이 지날수록 PlanetFactory에 추가되는 생성 메서드(매개변수화된)의 수는 폭발적으로 증가됨
    - 상태 조합이 복잡해 질수록 PlanetFactory에 포함된 생성 메서드 간의 중복 코드 역시 함께 증가하게 딤
 
### 하나의 잘못된 속성(ONE BAD ATTRIBUTE) 패턴
- 생성 메서드의 증가에 따른 코드 중복을 해결하는 한 가지 방법
- 유효한 상태를 가진 기본 픽스처를 생성한 후 일부 속성만 원하는 상태로 수정함으로써 코드 중복을 제거하는 방법
- 우선 픽스처의 모든 속성을 기본값으로 할당하는 createDefault()와 같은 이름을 가진 기본 생성 메서드를 추가
- 다른 생성 메서드들은 기본 생성 메서드를 호출한 후 반환되는 기본 픽스처의 일부 속성을 원하는 값으로 수정
```java
public abstract class PlanetFactory {
    public static Planet createDefault() {
        return new Planet(
                       new Atmosphere(Money.wons(5000), 
                           element("N", Ratio.of(0.8)), 
                           element("O", Ratio.of(0.2))),
                       Arrays.asList(
                           new Continent("아시아"),
                           new Continent("유럽")),
                       Arrays.asList(
                           new Ocean("태평양", Money.wons(1000)),
                           new Ocean("대서양", Money.wons(1000))));
    }

    public static Planet createPlanet(Continent ... continents) {
        Planet result = createDefault();
        result.setContinents(Arrays.asList(continents));
        return result;
    }
    
    public static Planet createPlanet(Ocean ... oceans) {
        Planet result = createDefault();
        result.setOceans(Arrays.asList(oceans));
        return result;
    }
    
    public static Planet createPlanet(List<Continent> continents, List<Ocean> oceans) {
         Planet result = createDefault();
         result.setContinents(continents);
         result.setOceans(oceans);
         return result;
     }
     ....
}
```

그러나 FACTORY 기반의 테스트 도우미에서 하나의 잘못된 속성 패턴을 적용하는 방법에는 다음과 같은 단점이 존재
- 불변 객체 생성 불가능
    - 하나의 잘못된 속성 패턴을 적용하기 위해서는 기본 생성 메서드가 반환하는 객체의 상태를 변경할 수 있어야 함
    - 따라서 객체의 상태를 변경할 수 없는 Value Object에 대해서는 하나의 잘못된 속성 패턴을 적용할 수 없음
- 캡슐화 저해 
    - 비록 상태 변경이 가능한 ENTITY라고 하더라도 클래스에 해당 속성을 변경할 수 있는 SETTER가 반드시 존재해야 함
    - 비록 테스트를 위해 가시성을 패키지 수준으로 낮춘다고 하더라도 속성 별로 SETTER를 무분별하게 제공하는 것은 객체의 캡슐화를 저해함
    
하나의 잘못된 속성 패턴을 사용해 코드 중복을 제거하더라도 FACTORY 기반의 OBJECT MOTHER 패턴은 근본적으로 다음과 같은 문제를 가지고 있음
- 생성 메서드의 폭발적 증가 
    – 비록 픽스처 생성 코드에 대한 중복은 제거할 수 있을지 몰라도 테스트에 필요한 픽스처의 속성 조합에 따른 생성 메서드의 폭발적 증가를 막을 수 있는 방법은 없음
    - 테스트의 수가 많아질수록 어떤 생성 메서드를 사용해야 하는 지조차 파악하기 힘들게 됨
- 클래스 인터페이스 변경으로 인한 파급 효과
    – 객체를 생성하는 코드가 FACTORY 내부로 캡슐화되기 때문에 생성자의 시그니처 변경으로 인한 테스트 케이스의 수정은 막을 수 있음
    - 그러나 속성의 조합을 기반으로 생성 메서드(CREATION METHOD)가 추가되기 때문에 이미 존재하는 속성을 제거하거나 새로운 속성을 추가할 경우 생성 메서드 뿐만 아니라 생성 메서드를 호출하는 모든 테스트 케이스의 코드를 수정해야 함

OBJECT MOTHER와 같은 FACTORY 기반의 테스트 도우미를 사용할 경우 
- 여러 테스트 케이스에서 픽스처 생성 코드가 중복되는 문제는 해결할 수 있지만 
- 테스트를 통해 검증해야 하는 픽스처의 상태 변이가 다양해질수록 테스트 코드의 유지보수를 어렵게 만드는 장애물로 변해버림

Steve Freeman과 Net Pryce는 “Growing Object-Oriented Software, Guided by Tests”에서 OBJECT MOTHER의 단점을 해결할 수 있는 TEST DATA BUILDER라는 패턴을 제시
- FACTORY 패턴이 아닌 BUILDER 패턴을 기반으로 픽스처 생성 코드를 설계
- TEST DATA BUILDER의 진정한 핵심은 테스트를 위한 전용 언어인 '도메인 특화 언어(Domain-Specific Language, DSL)'를 구축하는 것

# [도메인 특화 언어와 단위 테스트 - 3부(上)](http://aeternum.egloos.com/2962600)
## 소프트웨어의 본질적인 복잡성
- 소프트웨어 개발의 본질적인 문제 중 하나는 복잡성(complexity)    
- 소프트웨어의 복잡성은 소프트웨어가 가진 본질적인 어려움이므로 제거가 불가능
- 그러나 우리가 가진 인지능력의 한계 내에서 복잡성을 좀 더 다루기 쉬운 형태로 재단하는 것은 가능
- 프로그래밍 언어의 역사는 프로시저 추상화와 데이터 추상화와 같은 소프트웨어의 비본질적인 복잡도를 낮추기 위한 다양한 시도의 연속

## 도메인 특화 언어(Domain-Specific Language)
- 소프트웨어의 본질적인 복잡도를 해결할 수 있을 것으로 기대되는 한 가지 설계 방식
- UNIX 커뮤니티에는 특정한 응용 영역의 문제를 해결하기 위해 그 영역에만 적용할 수 있는 특수한 언어를 만들어 문제를 해결하는 오랜 전통이 존재
    - 이런 언어를 UNIX 커뮤니티에서는 “작은 언어(little language)”, 또는 “미니 언어(mini language)”라고함
    - UNIX의 미니 언어 전통 안에서 개발자는 도메인에서 사용되는 어휘, 문법, 의미론을 사용함으로써 도메인에 특화된 새로운 언어를 창조하고 이 언어를 이용해 프로그램을 작성
    - 이런 방식으로 작성된 애플리케이션은 프로그래밍 언어의 계층 위에 특정 도메인을 위한 특수 목적 언어 계층이 위치하는 계층형 아키텍처(Layered Architecture)의 특수한 형태를 띠게 됨
    - 즉, 하나의 프로그램 안에 다수의 언어 계층이 존재
    - 프로그래밍 언어로 작성된 추상화 계층 위에 도메인 언어로 작성된 추상화를 쌓아 올리는 것은 복잡성을 극복하기 위해 소프트웨어를 계층적인 형태로 분할하는 전통적인 설계 방식을 확장한 것
    - 미니 언어를 이용하면 도메인을 구성하는 추상 개념을 통해 사고하고 프로그래밍할 수 있음
    - 따라서 개발자는 부차적인 복잡성의 무게에 짓눌리지 않고도 본질적인 복잡성의 해결에 집중할 수 있다.
    - 개발자가 도메인의 용어로 프로그램을 개발하면 컴파일러나 도구가 도메인의 용어를 컴퓨터가 이해할 수 있는 명령어로 변환
    - 즉, 명세가 실행 가능한 코드가 되는 것

도메인 특화 언어(DSL, Domain-Specific Language)
- UNIX의 미니 언어와 같이 “특정한 도메인에 초점을 맞춘 제한적인 표현력을 가진 컴퓨터 프로그래밍 언어”
- 용어를 구성하고 있는 각 단어를 분석함으로써 DSL의 특성을 이해할 수 있다.
    - 도메인
        - “지식이나 영향력, 또는 활동의 영역”을 의미
        - DSL은 특정한 도메인 내에서 사용될 것을 가정하는 언어
        - 특정 도메인에 초점을 맞춤으로써 애플리케이션을 자연스럽게 성장시킬 수 있는 “문맥(context)”을 제공
    - 특화(Specific)
        - “제한된 문맥(bounded context)”과 “표현력의 제약(limited expressiveness)”을 강조
        - “제한된 문맥”이란 언어가 적용될 수 있는 영역의 범위가 제한적이라는 것
        - DSL은 제한된 영역에 대해 사고하고 이해하고 표현하기 위한 언어
    - 언어(Language)
        - DSL의 언어적인 특성은 “프로그래밍 언어”와 “유창함(fluency)”이라는 두 가지 측면을 포괄
        - DSL은 일반적인 프로그램 언어보다 특정 문맥(context)에서의 유창함(fluency)을 더욱 더 강조
- DSL이 가져야 하는 기본적인 3가지 특징
    - 단순함(Simplicity)
        – DSL을 사용하는 이유가 도메인과 관련된 복잡성을 낮추는 것
        - 도메인에 익숙한 개발자나 클라이언트는 DSL로 작성된 스크립트를 보고 그 의미를 쉽게 파악할 수 있어야 함
        - 도메인에서 사용되는 어휘를 사용하고 언어를 익히기 위해 필요한 문법을 제한함으로써 이해하기 쉬운 단순한 언어를 창조할 수 있음
    - 간결함(Conciseness)
        – DSL은 간단하면서도 표현력이 있는 문법을 선택해야
        - 문제와 관련되지 않은 군더더기 없는 표현을 최대한 제거함으로써 코드를 간결하게 만들 수 있음
    - 유창함(Fluency)
        - 언어의 유창함이란 “얼마나 빠르고 쉽게 사용자들이 정보를 처리할 수 있는가”로 정의할 수 있음
        - DSL의 유창함을 지원하는 두 가지 요소는 “전문 용어(jargon)”와 “문맥(context)”
        - 경험이 많은 프로그래머들은 특별한 설명이 없이도 “STRATEGY 패턴”이라는 전문 용어에 함축되어 있는 수많은 의미를 빠르게 이해
## 도메인 특화 언어의 구조
- 기본적인 인터페이스 설계 원칙 중 하나는 “커맨드-쿼리 분리(Command-Query Separation, CQS)”
    - 메서드는 반드시 부수 효과를 발생시키는 커맨드이거나 부수 효과를 발생시키지 않는 쿼리 둘 중 하나여야하고 두 가지 특성 모두를 가져서는 안 된다
    - 객체들을 독립적인 기계로 보는 객체지향의 오랜 전통에 기인
    - 객체는 블랙박스이며 객체의 인터페이스는 객체의 관찰 가능한 상태를 보기 위한 일련의 디스플레이와 객체의 상태를 변경하기 위해 누를 수 있는 버튼의 집합
    - 이런 스타일의 인터페이스를 사용함으로써 객체의 캡슐화와 다양한 문맥에서의 재사용을 보장할 수 있다
![Alt text](https://monosnap.com/image/SNN1t8LoLyCWejmHZtyNDwdiULXJGc)
    - 각 오퍼레이션들은 독립적인 상황에서 사용될 수 있도록 이름 지어진다.
        - <그림 1>에서 insert() 오퍼레이션은 empty() 오퍼레이션이나 delete() 오퍼레이션에 독립적으로 호출 가능
        - 인터페이스를 구성하는 오퍼레이션 간에 의존성이 적으면 적을수록 다양한 문맥에서 사용 가능하고 자유롭게 조합될 수 있는 객체를 만들 수 있다.
        - 커맨드-쿼리 인터페이스는 사용 문맥과의 결합도를 최소화하도록 설계된다.
- DSL
    - 이와 대조적으로 DSL의 인터페이스는 사용되는 문맥에 기반한 유창함을 강조
    - Martin Fowler와 Eric Evans는 DSL에 사용되는 인터페이스 형식을 전통적인 커맨드-쿼리 인터페이스 형식과 구분하기 위해 “유창한 인터페이스(Fluent Interface)”라고 부른다.
- 커맨드-쿼리 인터페이스와 유창한 인터페이스의 가장 두드러지는 차이점: 오퍼레이션의 이름을 짓는 방식    
    - 커맨드-쿼리 인터페이스: 문맥과 무관하게 독립적으로 사용될 것을 가정하기 때문에 설계자는 오퍼레이션의 이름 안에 최대한 많은 정보를 담으려고 노력
    - 유창한 인터페이스: 특정한 사용 문맥을 가정하기 때문에 독립적으로는 의미가 없고 연결되는 문맥 상에서 충분한 의미를 가질 수 있는 간결한 이름을 사용하려고 노력
    - 커맨드-쿼리 인터페이스는 훌륭한 객체 인터페이스, 유창한 인터페이스는 훌륭한 DSL 인터페이스를 낳는다
- 하나의 객체에 두 가지 방식의 인터페이스를 조합할 경우 이해하기 어려워 진
    - 따라서 DSL 설계와 관련해서 주의해야 할 기본 원칙은 커맨드-쿼리 인터페이스와 유창한 인터페이스를 서로 섞지 않는 것
- DSL을 구조화하는 일반적인 방식: 커맨드-쿼리 인터페이스를 제공하는 객체 모델 위에 유창한 인터페이스 방식의 독립적인 DSL 레이어를 구축하는 것
    - 커맨드-쿼리 인터페이스를 제공하는 하부 모델 위에서 유창한 인터페이스를 제공하는 독립적인 언어 계층을 표현식 빌더(EXPRESSION BUILDER)라고 함
    - DSL은 커맨드-쿼리 인터페이스 기반의 객체 모델을 제어할 수 있는 유창한 인터페이스를 제공하는 일종의 FAÇADE로 볼 수 있다.
    - Martin Fowler는 DSL에 의미론을 제공하는 커맨드-쿼리 인터페이스 기반의 객체 모델을 의미 모델(SEMANTIC MODEL)이라고 부른다.
    - 패러다임이 객체지향일 경우 일반적으로 의미 모델은 DOMAIN MODEL 패턴의 형태를 따른다.
![Alt text](https://monosnap.com/image/S6u8PsfTRP1qAG1tGFTVK7Eo9zt8K6)

## 도메인 특화 언어의 구분
- DSL을 구축하는 방법은 다음과 같은 3가지 범주
    - 외부 DSL(external DSL)
        – 애플리케이션 작성에 사용되는 주 언어(호스트 언어라고 부른다)가 아닌 별도의 독립적인 언어를 이용해서 DSL을 작성
        - ex. 정규 표현식, SQL, Awk, Spring이나 Hibernate에서 사용되는 XML 설정 파일
    - 내부 DSL(internal DSL)
        – 외부 DSL과 달리 독립적인 언어가 아닌 범용 프로그래밍 언어를 사용
    - 언어 워크벤치(language workbench)
        – 언어 워크벤치는 DSL을 정의하고 개발하기 위한 용도로 개발된 IDE
- DSL에서는 생성 중인 의미 모델의 중간 상태를 저장하기 위한 변수 필요
    - 내부 DSL에서 의미 모델의 중간 상태를 저장하기 위해 내부적으로 관리하는 변수를 컨텍스트 변수(CONTEXT VARIABLE)
    - 컨텍스트 변수(CONTEXT VARIABLE)의 역할은 대화에서 문맥이 제공하는 역할과 동일
    - DSL은 오퍼레이션의 실행 결과를  컨텍스트 변수에 누적시킨다
    - 컨텍스트 변수를 이용함으로써 오퍼레이션 실행 간에 명시적으로 전달해야 하는 정보를 암시적으로 만듦으로써 신호 대비 잡음 비율을 줄인다. 
    - 따라서 언어를 유창하게 만든다

# [도메인 특화 언어와 단위 테스트 - 3부(下)](http://aeternum.egloos.com/2968449)
## 내부 DSL(Internal DSL)
- 내부 DSL을 구축할 때 사용하는 가장 중요한 추상화 메커니즘은 함수(또는 프로시저)
- 전통적인 커맨드-쿼리 인터페이스의 설계가 함수를 기반으로 하는 것처럼 유창한 인터페이스의 설계 역시 함수를 기반으로 함
- 차이점은 함수가 조합되는 방식
    - 커맨드-쿼리 인터페이스의 설계자는 각 함수가 독립적으로 사용될 것이라고 가정
    - 유창한 인터페이스의 설계자는 관련된 함수들이 더 커다란 문맥 안에서 조합되어 사용된다고 가정
    - 이런 가정은 오퍼레이션의 이름에도 영향을 미침
        - 커맨드-쿼리 인터페이스의 경우 개별 오퍼레이션이 독립적으로 사용되더라도 그 의도를 명확하게 전달할 수 있는 이름을 선택
        - 유창한 인터페이스를 구성하는 각각의 오퍼레이션 이름은 홀로 놓고 보면 그 의도를 명확하게 이해하기가 어렵다. 
        - 특정한 문맥 내에서 다른 오퍼레이션들과 함께 사용될 경우에만 의도와 의미가 명확해짐
1. 메서드 체이닝(METHOD CHAINING)
- 메서드 체이닝은 유창한 인터페이스 구현 방식 중에서 가장 대중적이고 널리 사용되는 방식
- 상태를 변경하는 커맨드(COMMAND) 메서드의 반환 값으로 호스트 객체를 반환함으로써 호스트 객체의 메서드를 연쇄적으로 호출할 수 있도록 한다
```java
planet()
        .atmosphere()
                .element("N", 0.8)
                .element("O", 0.2)
                .price(Money.wons(5000))
        .continent()
                .name("아시아")
                .nation()
                        .name("대한민국")
                        .price(1000)
                .nation()
                        .name("일본")
                        .price(1000)
        .continent()
                .name("유럽")
                .nation()
                        .name("영국")
                        .price(1000)
                .nation()
                        .name("프랑스")
                        .price(1000)
        .ocean()
                .name("태평양")
                .price(Money.wons(1000))
        .ocean()
                .name("대서양")
                .price(Money.wons(1000))
        .end();
```
- 메서드들이 반환하는 객체가 의미 모델의 객체가 아니라는 점(즉, 도메인 모델의 객체가 아니라는 점)에 주의
    - 커맨드-쿼리 인터페이스로 작성된 의미 모델에 유창한 인터페이스를 혼합할 경우 인터페이스의 일관성이 무너지기 때문에 의미 모델 상의 객체를 메서드 체이닝의 대상으로 사용해서는 안됨
    - 따라서 커맨드 메서드들이 반환하는 객체는 유창한 인터페이스 방식으로 구현되는 독립적인 표현식 빌더
    - 각 표현식 빌더의 커맨드 메서드는 다음의 호출에 사용될 수 있는 표현식 빌더를 반환
    - planet(): PlanetBuilder, PlanetBuilder.atmosphere(): AtmosphereBuilder, element(): AtmosphreBuilder 인스턴스를 반환
- 메서드 체이닝은 커맨드-쿼리 분리(Command-Query Separation) 원칙, 데메테르 법칙(Law of Demeter)을 위반
    - 열차 충돌(train wreck)’의 형태를 나타남
    - 일반적인 커맨드-쿼리 인터페이스에서는 데메테르 법칙의 위반이 인터페이스 설계에 결함이 있을 수도 있음을 암시하지 DSL이라는 문맥에서는 인터페이스의 유창함을 만드는 핵심적인 구조를 제공
- 메서드 체이닝 단점
    - 컨텍스트 변수를 관리하기가 어렵다는 점
    - 다양한 메서드가 다양한 위치에서 호출될 수 있다는 점
        - Continent를 생성하기 시작하는 continent() 메서드는 Atmosphere를 생성하는 도중에서도, 다른 Continent를 생성하는 도중에도, Nation을 생성하는 도중에도 호출될 수 있음
        - 따라서 continent() 메서드는 AtmosphereBuilder에도, ContinentBuilder에도, NationBuilder에도 추가되어야 함
        - 이것은 각 표현식 빌더의 인터페이스를 오염시키고 표현식 빌더 간의 코드 중복과 의존성을 관리하기 어렵게 만듦
    - 의미 모델의 생성을 언제 중단할 것인가를 결정하기가 어렵다

2. 함수 시퀀스(FUNCTION SEQUENCE)
- 객체를 이용해 메서드를 연결하는 메서드 체이닝과 달리 함수 시퀀스 방식은 상호 독립적인 함수의 나열을 통해 내부 DSL을 구현
```java
planet();
        atmosphere();
                element("N", 0.8);
                element("O", 0.2);
                price(5000);
        continent();
                name("아시아");
                nation();
                        name("대한민국");
                        price(1000);
                nation();
                        name("일본");
                        price(1000);
        continent();
                name("유럽");
                nation();
                        name("영국");
                        price(1000);
                nation();
                        name("프랑스");
                        price(1000);
        ocean();
                name("태평양");
                price(1000);
        ocean();
                name("대서양");
                price(1000);
```
- 함수 간에 호출 순서 이외의 직접적인 연관성이 없음
- 함수 호출 사이의 관계는 내부적인 파싱 데이터를 이용해 암묵적으로 관리되므로 함수 시퀀스는 다른 방식보다 상대적으로 많은 양의 컨텍스트 변수를 관리해야 함
- 함수 시퀀스는 불필요한 표현상의 잡음을 줄이기 위해 전역 함수의 형태를 취함
- 전통적으로 전역 함수는 두 가지 문제점을 가짐
    - 전역 함수는 전역 이름 공간(global namespace)에 위치하므로 모든 곳에서 호출될 수 있음
    - 전역 함수는 정적 파싱 데이터(static parsing data)를 필요로 함
    - 두 가지 방식 모두 예측 불가능한 부수 효과(side effect)로 인해 프로그램을 불안정한 상태로 내몰 수 있다.
- 함수 시퀀스의 문제점을 해결할 수 있는 가장 좋은 방법은 객체를 이용해서 전역 함수와 전역 데이터의 범위를 객체 내부로 제한하는 것

3. 내포 함수(NESTED FUNCTION)
- 내포 함수는 다른 함수 호출 결과를 함수의 인자로 취함으로써 다수의 함수들을 조합하는 방식
```java
planet(
        atmosphere(
                price(5000),
                element("N", 0.8),
                element("O", 0.2)
        ),
        continents(
                continent(
                        name("아시아"), 
                        nation(
                                name("대한민국"), 
                                price(1000)
                        ), 
                       nation(
                                name("일본"), 
                                price(1000)
                        )
                ),
                continent(
                        name("유럽"),
                        nation(
                                name("영국"), 
                                price(1000)
                        ),
                        nation(
                                name("프랑스"), 
                                price(1000)
                       )
                )
        ),
        oceans(
                ocean(
                        name("태평양"),
                        price(1000)
                ),
                ocean(
                        name("대서양"),
                        price(1000)
                )
        )
);
```
- 메서드 체이닝과 함수 시퀀스는 컨텍스트 변수를 이용해 파싱 데이터를 관리
- 이에 비해 내포 함수는 함수 호출 결과를 함수의 인자로 취하기 때문에 중간 결과를 저장하기 위한 컨텍스트 변수가 필요 없다.
    - 그러나 포함 관계에 따라 왼쪽에서 오른쪽으로 자연스럽게 읽히는 메서드 체이닝과 함수 시퀀스 방식과 달리 
    - 내포 함수는 생성 순서에 따라 안에서 밖으로 읽어야 하기 때문에 코드의 가독성이나 이해도가 상대적으로 떨어진다. 
- 또 다른 단점은 함수 인자의 기본값을 지정할 수 없는 호스트 언어에서는 모든 함수 인자를 반드시 전달해야 한다는 것
- 전역 함수의 형태를 취하므로 함수 시퀀스와 유사한 전역 가시성의 문제에 직면

4. 객체 범위(OBJECT SCOPING)
- 객체 범위 기법은 함수 시퀀스와 내포 함수의 전역 가시성 문제를 해결하기 위해 사용
- 객체 범위는 전역 함수의 이름 공간으로 사용할 클래스를 추가하고 전역 데이터의 범위를 객체 내부로 제한함으로써 전역 가시성의 문제를 해결
- 객체 범위 기법은 클래스 상속에 기반
- 절차
    - 새로운 추상 클래스를 추가하고 전역 함수와 전역 데이터를 클래스의 멤버로 선언
    - 앞에서 함수 시퀀스 또는 내포 함수 방식으로 작성된 클래스들을 추상 클래스의 서브 클래스로 만듦
    - 이제 DSL 스크립트에서 사용하는 함수와 데이터는 전역 범위가 아니라 슈퍼 클래스의 인스턴스 범위로 제한
```java
public abstract class PlanetBuilder {
        protected Money price(long price) {
                return Money.wons(price);
        }
    
        protected Element element(String name, double ratio) {
                return Element.element(name, Ratio.of(ratio));
        }

        protected Atmosphere atmosphere(Money price, Element ... elements) {
                return new Atmosphere(price, elements);
        }

        protected String name(String name) {
                return name;
        }

        protected Nation nation(String name, Money price) {
                return new Nation(name, price);
        }

        protected List<Ocean> oceans(Ocean ... oceans) {
                return Arrays.asList(oceans);
        }

        protected Ocean ocean(String name, Money price) {
                return new Ocean(name, price);
        }

        protected List<Continent> continents(Continent ... continents) {
                return Arrays.asList(continents);
        }

        protected Continent continent(String name, Nation ... nations) {
                return new Continent(name, nations);
        }

        protected Planet planet(Atmosphere atmosphere, 
                List<Continent> continents, List<Ocean> oceans) {
                return new Planet(atmosphere, continents, oceans);
        }
}
```
- DSL 스크립트를 포함하는 클래스를 PlanetBuilder의 서브 클래스로 선언
    - 이제 슈퍼 클래스에 정의된 메소드를 사용할 수 있으므로 전역 이름 공간의 눈치를 볼 필요가 없음
```java
public class EarthBuilder extends PlanetBuilder {
        public Planet build() {
                return planet(
                                    atmosphere(
                                            price(5000),
                                            element("N", 0.8),
                                            element("O", 0.2)
                                    ),
                                    continents(
                                            continent(
                                                    name("아시아"), 
                                                    nation(
                                                            name("대한민국"), 
                                                            price(1000)
                                                    ), 
                                                    nation(
                                                            name("일본"), 
                                                            price(1000)
                                                    )
                                            ),
                                           continent(
                                                    name("유럽"),
                                                    nation(
                                                            name("영국"), 
                                                            price(1000)
                                                    ),
                                                    nation(
                                                            name("프랑스"), 
                                                            price(1000)
                                                    )
                                           )
                                     ),
                                    oceans(
                                            ocean(
                                                    name("태평양"),
                                                    price(1000)
                                            ),
                                            ocean(
                                                    name("대서양"),
                                                    price(1000)
                                            )
                                    )
                           );
        }
}
```

5. 내부 DSL 방식 조합하기
- <리스트 6>은 4가지 내부 DSL패턴을 모두 사용해서 두 개의 행성을 생성하는 DSL 코드
    - 두 개의 독립적인 planet() 함수 호출은 함수 시퀀스의 형태
    - planet() 함수는 필수 값들을 인자로 전달하는 내포 함수
    - 내포 함수에 전달할 인자들을 생성하기 위해서는 메서드 체이닝 기법
    - 여러 개의 대륙(continent()의 반환 값)과 대양(ocean()의 반환 값)을 목록으로 변환하는 continents()와 oceans()는 슈퍼 클래스인 PlanetBuilder으로부터 상속
    - 전역 범위의 함수와 데이터를 내부적으로 은폐하기 위해 객체 범위 방식
```java
public class BuilderCombination extends PlanetBuilder {
        public void build() {
                planet(
                        atmosphere()
                                .elements(element("N", 0.8), element("O", 0.2))
                                .price(5000),
                        continents(
                                continent().name("아프리카")
                                        .nations(
                                                nation().name("이집트").price(1000),
                                                nation().name("콩고").price(1000)),
                                continent().name("북아메리카")
                                        .nations(
                                                nation().name("캐나다").price(1000),
                                                nation().name("미국").price(1000))),
                        oceans(
                                ocean().name("남극해").price(1000),
                                ocean().name("북극해").price(1000)));
        
                planet(
                        atmosphere()
                                .elements(element("N", 0.8), element("O", 0.2))
                                .price(5000),
                        continents(
                                continent().name("아시아")
                                       .nations(
                                               nation().name("대한민국").price(1000),
                                               nation().name("일본").price(1000)),
                                continent().name("유럽")
                                       .nations(
                                               nation().name("영국").price(1000),
                                               nation().name("프랑스").price(1000))),
                                oceans(
                                        ocean().name("태평양").price(1000),
                                        ocean().name("대서양").price(1000)));
                ......
        }
}
```
- 여러 방식을 조합할 경우의 문제점: DSL의 사용 방법이 복잡해질 수 있다
    - 어떤 부분에서 어떤 기법이 사용되는 지를 예측하기 어려울 경우 전체적인 DSL의 유용성이 저하됨
    - DSL 방식을 조합할 경우 일정한 규칙에 따라 사용 방식을 쉽게 예측 가능하도록 언어를 설계해야
    - 빠른 속도로 정보를 처리할 수 있는 DSL의 유창함은 DSL로 작성된 코드를 읽는 사람들뿐만 아니라 DSL을 이용해 코드를 작성하는 사람에게도 중요
- 이제 앞에서 살펴본 픽스처 생성 문제를 해결하기 위해 DSL을 적용하는 문제로 돌아가 OBJECT MOTHER 패턴을 대체할 수 있는 TEST DATA BUILDER 패턴에 관해 살펴보자
    - TEST DATA BUILDER 패턴의 핵심은 테스트 픽스처 생성이라는 제한된 도메인에 특화된 언어를 내부 DSL의 형태로 구현하는 것

# [도메인 특화 언어와 단위 테스트 - 4부(上)](http://aeternum.egloos.com/2989078)
## 테스트 도메인에 특화된 언어
1. OBJECT MOTHER 패턴
- 픽스처 생성과 관련된 문제를 해결하기 위해 사용할 수 있는 한 가지 패턴
- 픽스처 생성을 위한 오퍼레이션을 제공하는 일종의 FACTORY
- 도메인 객체의 생성자 호출을 FACTORY 내부로 캡슐화함으로써 생성자 변경에 의한 파급효과를 OBJECT MOTHER 내부로 제한
- 단점: 테스트에 필요한 픽스처의 상태 조합에 따라 오퍼레이션 수가 폭발적으로 증가하기 때문에 중복 코드를 양산, 구현과 유지보수가 복잡해지며, 변경에 취약

2. TEST DATA BUILDER 패턴
- BUILDER 패턴을 기반
- 기본적인 접근 방법은 커맨드-쿼리 인터페이스를 제공하는 도메인 계층 위에 픽스처 생성이라는 제한된 범위를 지원하기 위해 유창한 인터페이스의 언어 계층을 얹는 것
- 즉, 핵심은 테스트를 위한 도메인-특화 언어를 창조하는 것
- 이 패턴의 중심에는 연쇄적인 메서드 호출을 통해 유창한 인터페이스를 제공하는 메서드 체이닝이 위치
- 커맨드-쿼리 인터페이스를 기반으로 하는 GOF의 BUILDER 패턴과 달리 TEST DATA BUILDER 패턴은 오퍼레이션의 결과로 반환되는 객체(호스트 객체라고 부른다)에 대한 연쇄적인 메서드 호출을 제공함으로써 픽스처 생성 코드를 유창하고 간결하게 만든다. 
- TEST DATA BUILDER는 도메인 객체를 의미 모델(semantic model, domain model)로 사용하고 오퍼레이션 호출을 결합해 픽스처로 생성할 도메인 객체를 생성하는 표현식 빌더(EXPRESSION BUILDER)다
- TEST DATA BUILDER 구현 절차
    1. 빌더 클래스 추가
        - 빌더 클래스의 이름은 “애플리케이션 클래스 이름 + Builder”의 형식으로 명명(Nation --> NationBuilder)
    2. 빌더 자체의 인스턴스를 생성해서 반환하는 정적 메서드를 추가
        - 정적 메서드의 이름은 “a + 애플리케이션 클래스 이름” 또는 “an + 애플리케이션 클래스 이름”의 형식을 따름(NationBuilder.aNation())
    3. 애플리케이션 클래스의 모든 속성을 빌더 클래스에 추가
    4. 속성 선언 시에 반드시 기본값을 할당
        - 기본값은 테스트 케이스에서 픽스처를 생성할 때 테스트와 관련 없는 정보를 표시하지 않아도 무방하도록 만들기 위해 사용
    5. 속성을 설정할 수 있는 커맨드(Command) 메서드 추가
        - 커맨드 메서드의 이름은 set으로 시작하는 일반적인 JavaBeans 관례를 따르지 않고 with로 시작하는 관례를 따름
        - 또한 반환값이 없는 전통적인 커맨드 메서드와 달리 빌더의 커맨드는 메서드 체이닝을 지원하기 위해 빌더 자신을 반환
        - 따라서 각 커맨드 메서드의 마지막 문장은 return this로 끝나야 함
    6. build() 메서드 추가
        - build() 메서드의 용도는 픽스처로 사용할 애플리케이션 객체를 생성한 후 반환
        - 메서드 체이닝이 완결되었다는 것을 명시적으로 표현
```java
public class NationBuilder {
  private String name = "대한민국";
  private Money price = Money.wons(1000);

  public NationBuilder withName(String name) {
    this.name = name;
    return this;
  }
    
  public NationBuilder withPrice(Money price) {
    this.price = price ;
    return this;
  }
    
  public Nation build() {
    return new Nation(name, price);
  }
}
```        
- 이 빌더의 특징(DSL과 관련된 TEST DATA BUILDER 패턴의 특징)
    - 인터페이스는 일반적인 커맨드-쿼리 인터페이스 스타일이 아닌 메서드 체이닝 기반의 유창한 인터페이스 스타일
        - 빌더의 생성자를 직접 호출하는 코드
          ```java
          Nation nation = new NationBuilder()
                            .withName("대한민국")
                            .withPrice(Money.wons(1000))
                            .build();
          ```
    - set으로 시작하는 전통적인 JavaBeans 컨벤션과 다르게 with로 시작
        - with로 시작하는 이유는 메서드의 이름이 독립적으로 사용되는 것이 아니라 다른 메서드와 연결도어 함께 사용되는 문맥을 강조하기 위해서
- DSL의 간결함과 유창함이라는 측면에서 좀 더 개선할 여지가 있다
    - new NationBuilder()와 같이 직접 빌더의 생성자를 호출해야 함
    - 클래스의 인스턴스를 생성하기 위해 new라는 키워드를 사용하는 것은 자연스러운 문장의 흐름을 방해
    - 빌더의 생성자를 호출하는 부분을 aNation()이라는 이름을 가진 정적 메서드로 대체함으로써 DSL의 흐름을 깔끔하게 만들 수 있다.
    ```java
    public class NationBuilder {
      public static NationBuilder aNation() {
        return new NationBuilder();
      }
      ...
    }
    ```
    - <리스트 5> 정적 임포트를 이용한 메서드 흐름의 개선
    ```java
      Nation nation = aNation()
                        .withName("대한민국")
                        .withPrice(Money.wons(1000))
                        .build(); 
    ```
- NationBuilder의 이름(name)과 가격(price) 속성을 선언할 때 기본 값으로 “대한민국”과 Money.wons(1000)을 할당하는 것에 주목
    - 빌더의 속성에 기본 값을 할당해 놓으면 테스트 케이스 작성 시 테스트 결과와 관련이 없는 불필요한 정보를 생략해도 NullPointerException이 발생하지 않도록 할 수 있다.  
    ```java
    // 기본 값을 가지도록 Nation 인스턴스 생성
    Nation nation = aNation().build(); 
  
    // 제조 가격만을 강조하고 싶은 경우
    Nation nation = aNation()
                      .withPrice(Money.wons(1000))
                      .build();
    ```   
- TEST DATA BUILDER의 속성은 DSL 관점에서 문맥 변수(Context Variable)에 해당
    - 문맥 변수에 기본 값을 할당함으로써 공유 문맥에 의존하는 여러 오퍼레이션들 호출 간에 전달해야 하는 불필요한 정보의 양을 줄일 수 있음
    - 또한 문맥 변수의 현재 값을 기반으로 다양하게 오퍼레이션 호출을 조합할 수 있기 때문에 API 사용성의 측면에서 유연성이 높아짐
    - TEST DATA BUILDER가 제공하는 이러한 유연성은 새로운 상태를 가진 픽스처가 필요한 경우 매번 새로운 오퍼레이션을 추가해야 하는 OBJECT MOTHER 패턴과는 대조적  

## TEST DATA BUILDER 조합하기
```java
public class ContinentBuilder {
  private String name = "아시아";
  private List<Nation> nations = Arrays.asList(
        aNation().with("대한민국").with(wons(1000)).build(),
        aNation().with("일본").with(wons(1000)).build());
    
  public static ContinentBuilder aContinent() {
    return new ContinentBuilder();
  }
    
  public ContinentBuilder withName(String name) {
    this.name = name;
    return this;
  }
    
  public ContinentBuilder withNations(Nation ... nations) {
    this.nations = Arrays.asList(nations);
    return this;
  }
    
  public Continent build() {
    return new Continent(name , nations.toArray(new Nation[0]));
  }
}
```
- 국가 정보만 중요한 경우의 Continent 생성 코드
    ```java
  Continent continent = aContinent()
                          .withNations(aNation().withName("캐나나").build(), 
                                       aNation().withName("미국").build())
                          .build();
    ```
- 위 모드의 문제점
    - withNations() 메소드 안에 aNation() 메서드를 호출하는 코드가 존재
    - 두 메서드(withNations, aNation)에 “Nation”이라는 단어가 중복
    - 두 메서드 이름 모두에 “Nation”이라는 단어가 포함되는 것은 불필요한 중복이며 결과적으로 DSL의 흐름을 방해
- 간결함과 유창함을 향상시키는 한 가지 방법은 
    - withNations() 메서드를 간단히 with()로 변경하고 
    - 뒤이어 호출되는 aNation() 메서드의 반환 타입을 이용해 컴파일러가 어떤 타입의 파라미터를 받는 with() 메서드 인지를 판단하도록 하는 것
- withName()과 withNations()를 with()로 수정
    ```java
      public class ContinentBuilder {
        public ContinentBuilder with(name) {
          this.name = name;
          return this;
        }
          
        public ContinentBuilder with(Nation ... nations) {
          this.nations = Arrays.asList(nations);
          return this;
        }
      }
    ```
- with 메서드의 이름을 수정한 NationBuilder
    ```java
  public class NationBuilder {
    public NationBuilder with(String name) {
      this.name = name;
      return this;
    }
      
    public NationBuilder with(Money price) {
      this.price = price;
      return this;
    }
    ...
  }  
    ```
- 국가 정보만 중요한 경우의 Continent 생성 코드
    ```java
  Continent continent = aContinent()
                          .with(aNation().with("캐나나").build(), 
                                aNation().with("미국").build())
                          .build();
    ```
- 한 가지 눈에 거슬리는 부분은 Nation을 생성할 때마다 매번 NationBuilder의 build() 메서드를 호출하는 부분
- 만약 생략해도 의미 전달에 지장이 없다면 정보를 제거하는 것이 DSL의 간결함과 유창함을 향상시키는 또 다른 방법
- 외부에서 직접 build()를 호출하지 않아도 되도록 Nation이 아닌 NationBuilder를 인자로 전달 받도록 ContinentBuilder의 with 메서드를 수정  
```java
public class ContinentBuilder {
  public ContinentBuilder with(NationBuilder ... nations) {
    this.nations = new ArrayList<Nation>();
        
    for(NationBuilder each : nations) {
        this.nations.add(each.build());
    }

    return this;
  }
  ...
}
```
-  build() 메서드 호출을 제거한 Continent 생성 코드
```java
  Continent continent = aContinent()
                          .with(aNation().with("캐나나"), 
                                aNation().with("미국"))
                          .build();
```

# [도메인 특화 언어와 단위 테스트 - 4부(下)[完]](http://aeternum.egloos.com/2989908)
## 테스트 케이스 리팩토링
1. 행성에 포함된 대륙의 개수를 체크하는 테스트 케이스
```java
  public class ContinentSpecificationTest {
    @Test
    public void continentSize() {
      Planet planet = new Planet(
                       new Atmosphere(Money.wons(5000), 
                         element("N", Ratio.of(0.8)), 
                         element("O", Ratio.of(0.2))),
                       Arrays.asList(
                         new Continent("아시아"),
                         new Continent("유럽")),
                       Arrays.asList(
                         new Ocean("태평양", Money.wons(1000)),
                         new Ocean("대서양", Money.wons(1000))));
      
      ContinentSpecification specification = new ContinentSpecification(2);
          
      assertTrue(specification.isSatisfied(planet));
    }
  }
```
- TEST DATA BUILDER 패턴을 이용해 리팩토링
    - Continent 이외의 테스트 결과와 관련이 없는 불필요한 정보를 제거할 수 있음
    - 테스트와 상관 없는 Atmosphere와 Ocean은 테스트 케이스의 초점을 흐르지 않도록 빌더 안으로 숨길 수 있음
    - 따라서 픽스처를 생성하는 코드가 간략해지고 테스트 결과와 직접적으로 관련 있는 정보만 코드 상에 보여지기 때문에 테스트의 의도를 파악하기가 쉬워짐
    - 관련된 정보만을 표현하는 TEST DATA BUILDER
    ```java
  public class ContinentSpecificationTest {
    @Test
    public void continentSize() {
      Planet planet = aPlanet().with(aContinent(), aContinent()).build();
   
      ContinentSpecification specification = new ContinentSpecification(2);
          
      assertTrue(specification.isSatisfied(planet));
    }
  }
    ```

2. 행성의 전체 가격이 20,000원보다 낮거나 같은 지 여부를 테스트
```java
   public class PriceSpecificationTest  {
     @Test
     public void price() {
       Planet planet = new Planet(
                        new Atmosphere(Money.wons(5000), 
                          element("N", Ratio.of(0.8)), 
                          element("O", Ratio.of(0.2))),
                        Arrays.asList(
                          new Continent("아시아", 
                            new Nation("대한민국", Money.wons(1000))),
                          new Continent("유럽", 
                           new Nation("영국", Money.wons(1000)))),
                        Arrays.asList(
                          new Ocean("태평양", Money.wons(1000)),
                          new Ocean("대서양", Money.wons(1000))));
       
       Specification specification = new PriceSpecification(Money.wons(9000));
       
       assertTrue(specification.isSatisfied(planet));
     }
   }
```
- TEST DATA BUILDER를 이용해서 리팩토링
    - 제조 가격 이외의 불필요한 정보를 빌더 내부로 숨기고 있음
    - 따라서 테스트 케이스가 좀 더 간결해지고 이해하기 쉬워졌음을 알 수 있다.
    ```java
    public class PriceSpecificationTest  {
      @Test
      public void price() {
        Planet planet = aPlanet()
                          .with(anAtmosphere().with(wons(5000)))
                          .with(aContinent().with(aNation().with(wons(1000))),
                                aContinent().with(aNation().with(wons(1000))))
                          .with(anOcean().with(wons(1000)), 
                                anOcean().with(wons(1000)))                                
                          .build();

        Specification specification = new PriceSpecification(Money.wons(9000));
        
        assertTrue(specification.isSatisfied(planet));
      }
    }
    ```
- **TEST DATA BUILDER는 다양한 픽스처의 상태를 조합해야 하는 상황에서도 유연하게 대처가 가능**