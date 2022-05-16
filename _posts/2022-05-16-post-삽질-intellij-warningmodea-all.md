---
title:  "[디자인 패턴] SOLID - 의존관계 역전 원칙(Dependency Inversion Principle)"
excerpt: "전통적인 의존관계를 반전시킴으로써 상위 계층을 하위 계층의 구현으로부터 분리하라"

categories:
  - 디자인 패턴
  - OOP
tags:
  - 디자인 패턴
  - OOP
last_modified_at: 2022-05-15T00:00:00
---


- DIP 정의

> 상위 모듈은 하위 모듈의 구현에 의존해서는 안되고, 상위/하위 모듈 모두 추상화에 의존해야 한다.
또는 추상화에 의존해야지, 구체화에 의존하면 안된다.
> 

상위 계층(정책 결정)이 하위 계층(세부 사항)에 의존하는 전통적인 의존관계를 반전시킴으로써 상위 계층을 하위 계층의 구현으로부터 분리하라는 것이다.

OCP에서 사용했던 예제코드로 간단하게 살펴보자.

```java
@Service
public class GoldInvestingService {
    public void investing(InvestingReq req) {
        System.out.println("Gold Investing");
    }
}
```

```java
@Service
public class FundInvestingService {
    public void investing(InvestingReq req) {
        System.out.println("Fund Investing");
    }
}
```

```java
public class InvestingController {
		@PostMapping(value = "/solid/ocp/investing/gold")
    public void investingGold(@RequestBody InvestingReq req) {
        goldInvestingService.investing(req);
    }

    @PostMapping(value = "/solid/ocp/investing/fund")
    public void investingFund(@RequestBody InvestingReq req) {
        fundInvestingService.investing(req);
    }
}
```

컨트롤러(상위 계층)가 제공하는 두 API는 goldInvestingService, fundInvestingService(하위 계층)에 의존하고 있다. 따라서 새로운 silver상품이 추가되면 컨트롤러의 코드도 수정되어야 하는데, 추상화를 통해 상위 계층과 하위 계층을 분리함으로써 이를 방지 해보자.

```java
public interface InvestingService {
		void investing(InvestingReq req);
}
```

```java
@Service
public class GoldInvestingService implements InvestingService {
    @Override
    public void investing(InvestingReq req) {
        System.out.println("Gold Investing");
    }
}
```

```java
@Service
public class FundInvestingService implements InvestingService {
    @Override
    public void investing(InvestingReq req) {
        System.out.println("Fund Investing");
    }
}
```

서비스들을 추상화 한 `InvestingService` 인터페이스를 정의하고 메소드를 구현하게한다.

```java
public class InvestingController {
		@PostMapping(value = "/solid/ocp/investing")
    public void investing(@RequestBody InvestingReq req) {
        InvestingService investingService = investingServiceFactory.getByType(req.productType);
        investingService.investing(req);
    }
}
```

그리고 상위 계층에서는 이 InvestingService 인터페이스에만 의존하게 함으로써 SilverInvestingService와 같은 새로운 서비스를 추가하게 되더라도 상위 계층은 변경이 발생하지 않는다.

처음 SRP를 살펴보면서 SOLID원칙은 서로 연관이 많다고 얘기 했었는데, DIP의 경우도 확장에는 열려있고 변경에는 닫혀있어야 한다는 Open/closed Principle 원칙이 떠오른다. OCP를 준수하는 방법중 하나가 DIP가 될 수 있고 DIP를 준수 하는 방법중 하나가 DI 아닐까 하는 생각이 든다.