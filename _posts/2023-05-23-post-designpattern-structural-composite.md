---
title: "[Design Pattern-구조] 복합체(Composite) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 구조 패턴
last_modified_at: 2023-05-23T00:00:00
---


## 정의

객체들의 관계를 트리 구조로 구성하여 부분-전체 계층을 표현하는 패턴.

단일 객체(Leaf)와 단일 객체들을 포함하는 복합 객체(Composite)로 구성되며, 이들을 공통 인터페이스를 통해 재귀적으로 호출해서 하나의 객체 처럼 다룰 수 있음.

![design-pattern-composite1]({{ '/assets/images/design-pattern-composite1.png' | relative_url }}){: .align-center}

## 예제

```java
interface RentalProduct {
    int calcCost(int days);
}

class SofaLeaf implements RentalProduct {
    private int cost;

    public SofaLeaf(int cost) {
        this.cost = cost;
    }

    @Override
    public int calcCost(int days) {
        return cost * days;
    }
}

class TableLeaf implements RentalProduct {

    private int cost;

    public TableLeaf(int cost) {
        this.cost = cost;
    }

    @Override
    public int calcCost(int days) {
        return cost * days;
    }
}

class AirConditionerLeaf implements RentalProduct {

    private int cost;

    public AirConditionerLeaf(int cost) {
        this.cost = cost;
    }

    @Override
    public int calcCost(int days) {
        return cost * days;
    }
}

class RoomLeaf implements RentalProduct {
    private int cost;

    public RoomLeaf(int cost) {
        this.cost = cost;
    }

    @Override
    public int calcCost(int days) {
        return cost * days;
    }
}

class RentalComposite implements RentalProduct {

    private List<RentalProduct> rentalProducts = new ArrayList<>();

    public RentalComposite(RentalProduct... products) {
        rentalProducts.addAll(Arrays.asList(products));
    }

    @Override
    public int calcCost(int days) {
        return rentalProducts.stream().mapToInt(product -> product.calcCost(days)).sum();
    }
}
```

테스트 코드로 확인.

```java
@Test
void composite() {
    RentalProduct sofa = new SofaLeaf(1);
    RentalProduct smallRoom = new RoomLeaf(10);

    RentalProduct table = new TableLeaf(100);
    RentalProduct airConditioner = new AirConditionerLeaf(1000);
    RentalProduct largeRoom = new RoomLeaf(10000);
    RentalProduct roomComposite = new RentalComposite(largeRoom, airConditioner, table);

    RentalProduct rentalCompositeAll = new RentalComposite(sofa, smallRoom, roomComposite);

    int totalCost = rentalCompositeAll.calcCost(3);

    System.out.println("total cost is " + totalCost);
}
```

출력 결과.

```powershell
total cost is 33333
```

## 적용 케이스

- 트리와 같은 계층 구조를 구성해야 할 때
- 클라이언트가 단순 요소(Leaf)와 복합 요소(Composite)를 모두 균일하게 처리하게 하고 싶을 때
- java.awt.Container.add()
- javax.faces.component.UIComponent.getChildren()

## 고려사항

- 기능이 많이 다른 클래스에 대해 공통 인터페이스를 제공하기 어려울 수 있고, 이를 해결하기 위해 인터페이스를 지나치게 일반화 하면 이해하기 어려울 수 있음

### References

- [https://refactoring.guru/design-patterns/composite](https://refactoring.guru/design-patterns/composite)