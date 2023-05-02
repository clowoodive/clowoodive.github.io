---
title: "[Design Pattern-행위] 방문자(Visitor) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
last_modified_at: 2023-04-24T00:00:00
---


## 정의

더블 디스패치를 통해서 대상 클래스 내의 알고리즘들을 방문자(Visitor)로 분리해서 처리하는 패턴. 

대상 클래스가 다른 유사한 알고리즘을 같은 클래스에서 관리 할 수 있고, 대상 클래스를 변경하지 않고 새로운 행동 도입 가능.

- 더블 디스패치 : 런타임에 바인딩이 결정되는 동적 바인딩을 두번 사용한 디스패치
    - element.accept(visitor)를 통해 대상 element가 결정됨
    - v.visit(this)를 통해 호출할 visitor메소드 결정됨

![design-pattern-visitor1]({{ '/assets/images/design-pattern-visitor1.png' | relative_url }}){: .align-center}

## 예제

```java
interface Shape {

    void draw();

    void accept(Visitor visitor);
}

interface Visitor {

    void visit(Circle circle);

    void visit(Triangle triangle);
}

class Circle implements Shape {

    @Override
    public void draw() {
        System.out.println("circle drawing");
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

class Triangle implements Shape {

    @Override
    public void draw() {
        System.out.println("triangle drawing");
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

public class VisitorConcrete implements Visitor {

    @Override
    public void visit(Circle circle) {
        System.out.println("circle 추가 작업");
        circle.draw();
    }

    @Override
    public void visit(Triangle triangle) {
        System.out.println("triangle 추가 작업");
        triangle.draw();
    }
}
```

테스트 코드로 확인.

```java
@Test
void visitor() {
    List<Shape> shapes = new ArrayList<>();
    Shape circle = new Circle();
    Shape triangle = new Triangle();

    shapes.add(circle);
    shapes.add(triangle);

    VisitorConcrete visitor = new VisitorConcrete();

    for (Shape shape : shapes) {
        shape.accept(visitor);
    }
}
```

출력 결과.

```powershell
circle 추가 작업
circle drawing
triangle 추가 작업
triangle drawing
```

## 적용 케이스

- 기존 코드를 변경하지 않고 기존 클래스 계층 구조에 새로운 행동을 추가하고 싶을 때
- 클래스의 주 알고리즘을 제외한 부가적인 행동을 분리해서 주 알고리즘에 집중하게 하고싶을 때
- 일부 행동이 계층구조의 일부 클래스에만 필요할 경우
- javax.lang.model.element.AnnotationValue / AnnotationValueVisitor
- javax.lang.model.element.Element / ElementVisitor
- javax.lang.model.type.TypeMirror / TypeVisitor

## 고려사항

- 방문자가 대상 클래스의 속성들을 사용하기 위한 적절한 접근자가 설정되어 있어야 함
- 대상 클래스의 계층 구조가 변경되면 모든 방문자를 수정해야 할 수 있음

### References

- [https://refactoring.guru/ko/design-patterns/visitor](https://refactoring.guru/ko/design-patterns/visitor)