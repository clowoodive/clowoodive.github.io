---
title:  "[디자인 패턴] SOLID - 개방 폐쇄 원칙(Open/closed Principle)"
excerpt: "소프트웨어 개체(클래스, 모듈 함수 등등)는 확장에 대해 열려있어야 하고, 변경에 대해서는 닫혀 있어야 한다."

categories:
  - 디자인 패턴
  - OOP
tags:
  - 디자인 패턴
  - OOP
last_modified_at: 2022-04-23T00:00:00
---


- OCP 정의

> 소프트웨어 개체(클래스, 모듈 함수 등등)는 확장에 대해 열려있어야 하고, 변경에 대해서는 닫혀 있어야 한다.
> 

- “확장에 대해 열려 있다”
    - 요구 사항이 달라짐에 따라 모듈의 동작을 변경하거나 추가해 확장 할수 있다.
- “수정에 대해 닫혀 있다”
    - 모듈의 변경이나 확장 시 그 모듈의 소스코드나 바이너리를 수정하지 않아야 한다.

간단하게 요약하면 변경이나 확장을 하면서 기존 코드를 수정하지 않게 설계하라는 말이다. 이는 추상클래스나 인터페이스를 통해서 구현 될 수 있는데 예제 코드를 통해 살펴보자.

투자 상품에 투자를 하는 api를 제공하는 간단한 controller와 service이다.

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

최초에 `GoldInvestingService`만 제공 하다가 Fund 투자 기능을 추가 했다고 본다면, `FundInvestingService` 를 추가해서  확장에는 열려 있지만 호출부인 컨트롤러에도 `investingFund` 라는 메소드 명으로 요청 처리 코드가 추가되었기에 변경에 닫혀 있지 못하다.

이를 변경에 닫혀 있도록 수정해 보면,

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

`InvestingService` 인터페이스의 investing 메소드를 구현하는 서비스로 변경하고

```java
public class InvestingController {
		@PostMapping(value = "/solid/ocp/investing")
    public void investing(@RequestBody InvestingReq req) {
        InvestingService investingService = investingServiceFactory.getByType(req.productType);
        investingService.investing(req);
    }
}
```

`InvestingReq` 요청으로 productType을 받아서 `InvestingServiceFactory` 에서 해당 타입의 서비스를 꺼내오게 함으로써 하나의 요청으로 처리하게 변경했다.

만약 이 상태에서 다른 투자상품을 추가 하려면 `InvestingService` 인터페이스를 구현하는 service를 구현하고  `InvestingServiceFactory` 내부만 조금 수정 해 주면 된다. 이렇게 하면 호출부(컨트롤러)의 수정이 없어도 되므로 변경에 닫혀 있다고 볼 수 있다. 

추가로 예를 들면 메모리 DB를 사용하다가 H2 DB를 사용하도록 변경할 때 H2 DataSource를 사용하고 Repository 인터페이스 구현하는 H2 DB용 구현체를 만들고, connection 설정 및 DI를 통한 구현체만 대체 해 주면 된다. 다른 DB로의 변경에 대한 확장에 열려 있고, 설정 및 구현체 주입 이외의 로직의 변경에는 닫혀 있다.

DI 를 통한 구현체 대체를 본문 예제에서 찾아보자면 `investingServiceFactory` 의 내부 로직이라고 볼 수 있다.

추가 [예시](https://github.com/clowoodive/example/tree/main/example-lecture-spring-management-member)는 이 코드의 커밋(32105f7178c6c3ea89511e4aba30aa291e5054ff) 참조.

<!--

[https://github.com/cheese10yun/spring-SOLID/blob/master/docs/OCP.md](https://github.com/cheese10yun/spring-SOLID/blob/master/docs/OCP.md)

[https://whitepro.tistory.com/305?category=985210](https://whitepro.tistory.com/305?category=985210)

-->