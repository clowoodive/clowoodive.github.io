---
title: "[Design Pattern-구조] 플라이웨이트(Flyweight) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 구조 패턴
last_modified_at: 2023-05-22T00:00:00
---


## 정의

여러 객체에서 공통적으로 사용되는 부분을 다른 객체(Flyweight)로 분리해서 공유함으로써 RAM 사용량을 줄일 수 있는 패턴.

공통적으로 사용되는 객체를 캐싱하고 이를 반환하는 메서드가 있음.(i.e FlyWeightFactory)

플라이웨이트 객체는 다른 컨텍스트에서 사용 될 수 있으므로 최초 초기화 후 변경되지 않도록 해야함.

![design-pattern-flyweight1]({{ '/assets/images/design-pattern-flyweight1.png' | relative_url }}){: .align-center}

## 예제

```java
class Circle {

    private int x;

    private int y;

    private int radius;

    private ShapeType shapeType;

    public Circle(int x, int y, int radius, String shape, String color, String longData) {
        this.x = x;
        this.y = y;
        this.radius = radius;
        this.shapeType = ShapeTypeFactory.getShapeType(shape, color, longData);
    }

    public ShapeType getShapeType() {
        return shapeType;
    }
}

class Ellipse {

    private int x;

    private int y;

    private int shortRadius;

    private int longRadius;

    private ShapeType shapeType;

    public Ellipse(int x, int y, int shortRadius, int longRadius, String shape, String color, String longData) {
        this.x = x;
        this.y = y;
        this.shortRadius = shortRadius;
        this.longRadius = longRadius;
        this.shapeType = ShapeTypeFactory.getShapeType(shape, color, longData);
    }

    public ShapeType getShapeType() {
        return shapeType;
    }
}

// Flyweight
class ShapeType {

    private String shape;

    private String color;

    private String longData;

    public ShapeType(String shape, String color, String longData) {
        this.shape = shape;
        this.color = color;
        this.longData = longData;
    }
}

class ShapeTypeFactory {

    private static Map<String, ShapeType> shapeTypes = new HashMap<>();

    public static ShapeType getShapeType(String shape, String color, String longData) {
        ShapeType shapeType = shapeTypes.get(shape);
        if (shapeType == null) {
            shapeType = new ShapeType(shape, color, longData);
            shapeTypes.put(shape, shapeType);
            System.out.println("first generate shapeType :" + shape);
        }
        return shapeType;
    }
}
```

테스트 코드로 확인.

```java
@Test
void flyweight() {
    Circle circle1 = new Circle(5, 10, 8, "circle", "red", "circle data");
    Circle circle2 = new Circle(7, 9, 5, "circle", "red", "circle data");

    System.out.println(circle1.getShapeType());
    System.out.println(circle2.getShapeType());
}
```

출력 결과.

```powershell
first generate shapeType :circle
clowoodive.example.designpattern.structure.ShapeType@77128536
clowoodive.example.designpattern.structure.ShapeType@77128536
```

## 적용 케이스

- 유사한 객체(공통 데이터가 많은)가 많아서 RAM을 많이 사용 할 때
- java.lang.Integer.valueOf(int)

## 고려사항

- 코드가 복잡해짐
- RAM을 절약한 대신 CPU를 더 쓰게될 수 있음

### References

- [https://refactoring.guru/ko/design-patterns/flyweight](https://refactoring.guru/ko/design-patterns/flyweight)