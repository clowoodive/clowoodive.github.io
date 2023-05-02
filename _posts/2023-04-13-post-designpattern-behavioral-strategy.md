---
title: "[Design Pattern-행위] 전략(Strategy) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
  - 합성
last_modified_at: 2023-04-13T00:00:00
---


## 정의

다양한 방식으로 수행되는 클래스(Context)의 해당 알고리즘을 전략(Strategy)이라는 클래스로 추출하고, 이 전략들의 인터페이스 참조를 Context가 가지게 함. 클라이언트가 원하는 전략을 결정해서 Context에 주입하고 Context는 그 전략에 작업을 위임함. 

![design-pattern-strategy1]({{ '/assets/images/design-pattern-strategy1.png' | relative_url }}){: .align-center}

## 예제

```java
public class StrategyContext {

    private Strategy strategy;

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public void executeStrategy(int weight) {
        this.strategy.calcExpense(weight);
    }
}

interface Strategy {
    void calcExpense(int weight);
}

class ConcreteStrategy1 implements Strategy {

    @Override
    public void calcExpense(int weight) {
        System.out.println("strategy1 calculate weight : " + weight);
    }
}

class ConcreteStrategy2 implements Strategy {

    @Override
    public void calcExpense(int weight) {
        System.out.println("strategy2 calculate weight : " + weight);
    }
}
```

테스트 코드로 확인.

```java
@Test
void strategy() {
    StrategyContext strategyContext = new StrategyContext();

    System.out.println("----- strategy1 -----");
    ConcreteStrategy1 concreteStrategy1 = new ConcreteStrategy1();
    strategyContext.setStrategy(concreteStrategy1);
    strategyContext.executeStrategy(1);

    System.out.println("----- strategy2 -----");
    ConcreteStrategy2 concreteStrategy2 = new ConcreteStrategy2();
    strategyContext.setStrategy(concreteStrategy2);
    strategyContext.executeStrategy(2);
}
```

출력 결과.

```powershell
----- strategy1 -----
strategy1 calculate weight : 1
----- strategy2 -----
strategy2 calculate weight : 2
```

## 적용 케이스

- 객체 내에서 한 알고리즘의 다양한 변형을 사용하고 싶을 때
- 거대한 조건문으로 구성된 알고리즘을 분리하고 싶을 때
- 런타임 중에 알고리즘을 바꾸고 싶을 때
- 일부 알고리즘만 차이가 있는 유사한 여러 클래스가 있을 때
- Collections.sort() - java.util.Comparator.compare()

## 고려사항

- 변화가 적은 알고리즘에 적용하면 오히려 복잡한 구성이 될수 있음
- 클라이언트가 전략을 알고있어야 선택 할 수 있음
- 익명 함수(Java의 경우 Functional Interface)를 통해 좀더 간편하게 구현 가능

### References

- [https://refactoring.guru/ko/design-patterns/strategy](https://refactoring.guru/ko/design-patterns/strategy)