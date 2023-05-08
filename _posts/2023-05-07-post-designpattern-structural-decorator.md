---
title: "[Design Pattern-구조] 데코레이터(Decorator) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 구조 패턴
last_modified_at: 2023-05-07T00:00:00
---


## 정의

객체의 동작을 확장하기 위해 상속이 아닌 집약(aggregation)을 사용하는 방식.

런타임에 자유롭게 행동의 확장/조합이 가능하고, 다양한 방식의 행동을 제공하는 모놀리식 클래스를 주 클래스와 데코레이터 클래스들로 나눌 수 있음.

![design-pattern-decorator1]({{ '/assets/images/design-pattern-decorator1.png' | relative_url }}){: .align-center}

## 예제

```java
class CoffeeMaker implements DrinkMaker {

    private int size;

    private int shot;

    private boolean isCold;

    @Override
    public void makeDrink(int size, int shot, boolean isCold) {
        this.size = size;
        this.shot = shot;
        this.isCold = isCold;
    }

    @Override
    public void showDetail() {
        System.out.println("--- detail ---");
        System.out.println("Size : " + size);
        System.out.println("Shot : " + shot);
        System.out.println("Cold : " + isCold);
    }
}

class DrinkMakerDecorator implements DrinkMaker {

    private DrinkMaker maker;

    public DrinkMakerDecorator(DrinkMaker maker) {
        this.maker = maker;
    }

    @Override
    public void makeDrink(int size, int shot, boolean isCold) {
        maker.makeDrink(size, shot, isCold);
    }

    @Override
    public void showDetail() {
        maker.showDetail();
    }
}

class SizeDecorator extends DrinkMakerDecorator {

    public SizeDecorator(DrinkMaker maker) {
        super(maker);
    }

    @Override
    public void makeDrink(int size, int shot, boolean isCold) {
        super.makeDrink(addSize(size), shot, isCold);
    }

    private int addSize(int size) {
        System.out.println("size up.");
        return size + 1;
    }

    @Override
    public void showDetail() {
        super.showDetail();
    }
}

class ShotDecorator extends DrinkMakerDecorator {

    public ShotDecorator(DrinkMaker maker) {
        super(maker);
    }

    @Override
    public void makeDrink(int size, int shot, boolean isCold) {
        super.makeDrink(size, addShot(shot), isCold);
    }

    private int addShot(int shot) {
        System.out.println("add shot.");
        return shot + 1;
    }

    @Override
    public void showDetail() {
        super.showDetail();
    }
}

class ColdDecorator extends DrinkMakerDecorator {

    public ColdDecorator(DrinkMaker maker) {
        super(maker);
    }

    @Override
    public void makeDrink(int size, int shot, boolean isCold) {
        System.out.println("change cold.");
        super.makeDrink(size, shot, true);
    }

    @Override
    public void showDetail() {
        super.showDetail();
    }
}
```

테스트 코드로 확인.

```java
@Test
void decorator() {
    DrinkMakerDecorator decorator = new ColdDecorator(
            new ShotDecorator(
                    new SizeDecorator(
                            new CoffeeMaker()
                    )
            )
    );

    decorator.makeDrink(2, 1, false);
    decorator.showDetail();
}
```

출력 결과.

```powershell
change cold.
add shot.
size up.
--- detail ---
Size : 3
Shot : 2
Cold : true
```

## 적용 케이스

- 런타임에 다양한 행동들을 추가하고 싶을 때
- 상속으로 행동을 확장하기 어색하거나 불가능할 때(e.g. final이 선언된 클래스)
- java.io.InputStream의 자식 클래스들

## 고려사항

- 데코레이터들의 순서에 따라 행동들의 의존성이 생김
- 데코레이터 스택에서 특정 데코레이터 제거가 어려움

### References

- [https://refactoring.guru/ko/design-patterns/decorator](https://refactoring.guru/ko/design-patterns/decorator)