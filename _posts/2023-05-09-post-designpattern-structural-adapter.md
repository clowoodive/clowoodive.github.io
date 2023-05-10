---
title: "[Design Pattern-구조] 어댑터(Adapter) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 구조 패턴
last_modified_at: 2023-05-09T00:00:00
---


## 정의

클라이언트가 사용하지 않는 인터페이스를 가진 서비스를 클라이언트가 사용하는 인터페이스를 가진 어댑터를 통해 래핑함으로써 사용 할수 있게 하는 패턴.

![design-pattern-adapter1]({{ '/assets/images/design-pattern-adapter1.png' | relative_url }}){: .align-center}

## 예제

```java
// 클라이언트에서 사용할 서비스의 공통 인터페이스
interface RoundThing {
    double getRadius();
}

// 비즈니스 로직을 포함하는 클라이언트 - 둥근 구멍
class RoundHole {

    private double radius;

    public RoundHole(double radius) {
        this.radius = radius;
    }

    public double getRadius() {
        return radius;
    }

    public boolean fits(RoundThing peg) {
        return this.getRadius() > peg.getRadius();
    }
}

// 클라이언트가 사용할 인터페이스를 따르는 서비스 - 둥근 못
class RoundPeg implements RoundThing {

    private double radius;

    public RoundPeg(double radius) {
        this.radius = radius;
    }

    @Override
    public double getRadius() {
        return radius;
    }
}

// 클라이언트가 사용 하려 하는 호환되지 않는 서비스 - 정사각형 못
class SquarePeg {

    private double width;

    public SquarePeg(double width) {
        this.width = width;
    }

    public double getWidth() {
        return width;
    }

    public double getSquare() {
        return Math.pow(this.width, 2);
    }
}

// 호환되지 않는 서비스를 래핑해서 공통 인터페이스를 제공하는 어댑터
class SquareAdaptor implements RoundThing {

    SquarePeg squarePeg;

    public SquareAdaptor(SquarePeg squarePeg) {
        this.squarePeg = squarePeg;
    }

    public void setSquarePeg(SquarePeg squarePeg) {
        this.squarePeg = squarePeg;
    }

    @Override
    public double getRadius() {
        // Math.sqrt(a^2 + b^2)에서 정사각형이므로 Math.sqrt(a^2 * 2)
        return Math.sqrt(Math.pow((squarePeg.getWidth() / 2), 2) * 2);
    }
}
```

테스트 코드로 확인.

```java
@Test
void adapter() {
    RoundHole roundHole = new RoundHole(10);
    System.out.println("roundHole radius : " + roundHole.getRadius());

    RoundThing fitRoundPeg = new RoundPeg(9);
    RoundThing bigRoundPeg = new RoundPeg(11);

    System.out.println(fitRoundPeg.getRadius() + " radius is fit to hole : " + roundHole.fits(fitRoundPeg));
    System.out.println(bigRoundPeg.getRadius() + " radius is fit to hole : " + roundHole.fits(bigRoundPeg));

    SquarePeg fitSquarePeg = new SquarePeg(14);
    SquarePeg bigSquarePeg = new SquarePeg(16);
    SquareAdaptor squareAdaptor = new SquareAdaptor(fitSquarePeg);

    System.out.println(squareAdaptor.getRadius() + " is fit to hole : " + roundHole.fits(squareAdaptor));
    squareAdaptor.setSquarePeg(bigSquarePeg);
    System.out.println(squareAdaptor.getRadius() + " is fit to hole : " + roundHole.fits(squareAdaptor));
}
```

출력 결과.

```powershell
roundHole radius : 10.0
9.0 radius is fit to hole : true
11.0 radius is fit to hole : false
9.899494936611665 is fit to hole : true
11.313708498984761 is fit to hole : false
```

## 적용 케이스

- 기존의 복잡한 레거시 코드 또는 외부 클래스를 손대지 않고 호환 가능하게 할 때
- 부모 클래스에 추가 할 수 없는 기능들을 자식 클래스에 적용하고 싶을 때
- java.util.Arrays.asList()
- java.util.Collections.list()

## 고려사항

- 클래스 구조의 복잡도 증가

### References

- [https://refactoring.guru/ko/design-patterns/adapter](https://refactoring.guru/ko/design-patterns/adapter)