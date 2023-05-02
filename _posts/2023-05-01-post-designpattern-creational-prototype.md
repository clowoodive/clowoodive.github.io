---
title: "[Design Pattern-생성] 프로토타입(Prototype) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 생성 패턴
last_modified_at: 2023-05-01T00:00:00
---


## 정의

객체 복제 시 복제를 지원하는 공통 인터페이스를 통해 복제 함으로써 해당 객체의 클래스와 의존성을 낮추고, private 필드도 복사 할 수 있음.

클래스는 같지만 설정이 다른 객체 생성을 위해 다수의 서브 클래스를 사용하는 경우, 미리 자주 사용되는 프로토타입을 생성해서 관리하고 복제해 주는 Prototype Registry 방식 적용 가능.

![design-pattern-prototype1]({{ '/assets/images/design-pattern-prototype1.png' | relative_url }}){: .align-center}

![design-pattern-prototype2]({{ '/assets/images/design-pattern-prototype2.png' | relative_url }}){: .align-center}

## 예제

```java
// Prototype
abstract class Shape {

    public String color;
    private int x; // 비공개 필드
    private int y; // 비공개 필드

    public Shape() {
    }

    public Shape(Shape origin) {
        if (origin != null) {
            this.x = origin.x; // 같은 클래스의 오브젝트는 비공개 필드 접근 가능
            this.y = origin.y;
            this.color = origin.color;
        }
    }

    public void setX(int x) {
        this.x = x;
    }

    public void setY(int y) {
        this.y = y;
    }

    public abstract Shape clone();

    public boolean equals(Object object2) {
        if (!(object2 instanceof Shape))
            return false;
        Shape shape2 = (Shape) object2;
        return shape2.x == x && shape2.y == y && shape2.color.equals(color);
    }
}

class Circle extends Shape {

    public int radius;

    public Circle() {
    }

    public Circle(Circle origin) {
        super(origin);
        if (origin != null) {
            this.radius = origin.radius;
        }
    }

    @Override
    public Shape clone() {
        return new Circle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Circle) || !super.equals(object2))
            return false;
        Circle shape2 = (Circle) object2;
        return shape2.radius == radius;
    }
}

class Rectangle extends Shape {

    public int width;

    public int height;

    public Rectangle() {
    }

    public Rectangle(Rectangle origin) {
        super(origin);
        if (origin != null) {
            this.width = origin.width;
            this.height = origin.height;
        }
    }

    @Override
    public Shape clone() {
        return new Rectangle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Rectangle) || !super.equals(object2))
            return false;
        Rectangle shape2 = (Rectangle) object2;
        return shape2.width == width && shape2.height == height;
    }
}
```

테스트 코드로 확인.

```java
@Test
void prototype() {
    Circle circle = new Circle();
    circle.setX(5);
    circle.setY(10);
    circle.color = "red";
    circle.radius = 15;

    Shape cloneCircle = circle.clone();
    if (circle == cloneCircle)
        System.out.println("circle and cloneCircle is same instance");
    else
        System.out.println("circle and cloneCircle is different instance");

    circle.radius = 20;
    if (circle.equals(cloneCircle))
        System.out.println("circle and cloneCircle has same value");
    else
        System.out.println("circle and cloneCircle has different value");

    if (cloneCircle.equals(circle))
        System.out.println("cloneCircle and circle has same value");
    else
        System.out.println("cloneCircle and circle has different value");
}
```

출력 결과.

```powershell
circle and cloneCircle is different instance
circle and cloneCircle has different value
cloneCircle and circle has different value
```

## 적용 케이스

- 인터페이스를 통해 사용되는 객체를 복제해야 할 경우, 하위 구상클래스들에 의존하지 않고 결합도를 유지한 상태로 복제하고 싶을 때
- 객체를 초기화 하는 방식만 다른 서브 클래스의 수가 많을 경우 이 서브클래스를 줄이고 싶을 때
- java.lang.Ojbect.clone() - java.lang.Cloneable 인터페이스를 구현해야 함

## 고려사항

- 순환 참조가 있는 복잡한 객체들을 복제하기는 까다로움

### References

- [https://refactoring.guru/ko/design-patterns/prototype](https://refactoring.guru/ko/design-patterns/prototype)