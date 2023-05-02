---
title: "[Design Pattern-생성] 빌더(Builder) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 생성 패턴
last_modified_at: 2023-04-28T00:00:00
---


## 정의

동일한 구성 코드를 사용해서 다른 타입이나 다른 스타일의 객체를 단계적으로 생성할수 있게 하는 패턴.

생성 단계는 선택적으로 적용 가능하며 일부 단계의 실행을 연기하거나, 재귀적으로 실행하게 구성 할수 있음.

자주 사용되는 생성 단계 순서를 담은 Director 클래스를 사용 하면 코드 재사용을 줄일 수 있고, 생성 과정 캡슐화의 이점도 얻을 수 있음.

![design-pattern-builder1]({{ '/assets/images/design-pattern-builder1.png' | relative_url }}){: .align-center}


## 예제

```java
interface Builder {

    public static final int SEATS_DEFAULT = 2;

    void setCarType(CarType type);

    void setSeats(int seats);

    void setEngine(Engine engine);

    void setNavigator(Navigator navigator);
}

class CarBuilder implements Builder {

    private CarType type;

    private int seats;

    private Engine engine;

    private Navigator navigator;

    @Override
    public void setCarType(CarType type) {
        this.type = type;
    }

    @Override
    public void setSeats(int seats) {
        this.seats = seats;
    }

    @Override
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    @Override
    public void setNavigator(Navigator navigator) {
        this.navigator = navigator;
    }

    public Car getResult() {
        return new Car(type, seats, engine, navigator);
    }
}

class CarManualBuilder implements Builder {

    private CarType type;

    private int seats;

    private Engine engine;

    private Navigator navigator;

    @Override
    public void setCarType(CarType type) {
        this.type = type;
    }

    @Override
    public void setSeats(int seats) {
        this.seats = seats;
    }

    @Override
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    @Override
    public void setNavigator(Navigator navigator) {
        this.navigator = navigator;
    }

    public CarManual getResult() {
        return new CarManual(type, seats, engine, navigator);
    }
}

class CarDirector {

    public void constructSedan(Builder builder) {
        builder.setCarType(CarType.Sedan);
        builder.setSeats(4);
        builder.setEngine(new Engine(2.0, 0));
        builder.setNavigator(new Navigator());
    }

    public void constructTruck(Builder builder) {
        builder.setCarType(CarType.Truck);
        builder.setSeats(2);
        builder.setEngine(new Engine(4.0, 0));
        builder.setNavigator(null);
    }
}

class Car {

    private final CarType carType;

    private final int seats;

    private final Engine engine;

    private final Navigator navigator;

    public Car(CarType carType, int seats, Engine engine, Navigator navigator) {
        this.carType = carType;
        this.seats = seats <= 0 ? Builder.SEATS_DEFAULT : seats;
        this.engine = engine;
        this.navigator = navigator;
    }

    public String info() {
        String info = "";
        info += "Type of car: " + carType + "\n";
        info += "Count of seats: " + seats + "\n";
        info += "Engine: volume - " + engine.getVolume() + "; mileage - " + engine.getMileage() + "\n";
        if (this.navigator != null) {
            info += "Navigator: Functional" + "\n";
        } else {
            info += "Navigator: N/A" + "\n";
        }
        return info;
    }
}

class CarManual {
    private final CarType carType;

    private final int seats;

    private final Engine engine;

    private final Navigator navigator;

    public CarManual(CarType carType, int seats, Engine engine, Navigator navigator) {
        this.carType = carType;
        this.seats = seats <= 0 ? Builder.SEATS_DEFAULT : seats;
        this.engine = engine;
        this.navigator = navigator;
    }

    public String print() {
        String info = "";
        info += "Type of car: " + carType + "\n";
        info += "Count of seats: " + seats + "\n";
        info += "Engine: volume - " + engine.getVolume() + "; mileage - " + engine.getMileage() + "\n";
        if (this.navigator != null) {
            info += "Navigator: Functional" + "\n";
        } else {
            info += "Navigator: N/A" + "\n";
        }
        return info;
    }
}

enum CarType {
    Sedan, Suv, Truck
}

class Engine {

    private final double volume;

    private double mileage;

    private boolean started;

    public Engine(double volume, double mileage) {
        this.volume = volume;
        this.mileage = mileage;
    }

    public void on() {
        this.started = true;
    }

    public void off() {
        this.started = false;
    }

    public boolean isStarted() {
        return started;
    }

    public void go(double mileage) {
        if (started) {
            this.mileage += mileage;
        } else {
            System.out.println("not started engine");
        }
    }

    public double getVolume() {
        return volume;
    }

    public double getMileage() {
        return mileage;
    }
}

class Navigator {

    private String route;

    public Navigator() {
        route = "default destination";
    }

    public Navigator(String route) {
        this.route = route;
    }
}
```

테스트 코드로 확인.

```java
@Test
void builder() {
    CarDirector carDirector = new CarDirector();

    // Director로 Sedan 생성
    CarBuilder sedanBuilder = new CarBuilder();
    carDirector.constructSedan(sedanBuilder);
    Car sedanCar = sedanBuilder.getResult();
    System.out.println("Car built :\n" + sedanCar.info());

    // 수동으로 Suv 생성
    CarBuilder suvBuilder = new CarBuilder();
    suvBuilder.setCarType(CarType.Suv);
    suvBuilder.setSeats(6);
    suvBuilder.setEngine(new Engine(3.0, 0));
    suvBuilder.setNavigator(new Navigator());
    Car SuvCar = suvBuilder.getResult();
    System.out.println("Car built :\n" + SuvCar.info());

    // Director로 Truck Manual 생성
    CarManualBuilder manualBuilder = new CarManualBuilder();
    carDirector.constructTruck(manualBuilder);
    CarManual truckManual = manualBuilder.getResult();
    System.out.println("CarManual built :\n" + truckManual.print());
}
```

출력 결과.

```powershell
Car built :
Type of car: Sedan
Count of seats: 4
Engine: volume - 2.0; mileage - 0.0
Navigator: Functional

Car built :
Type of car: Suv
Count of seats: 6
Engine: volume - 3.0; mileage - 0.0
Navigator: Functional

CarManual built :
Type of car: Truck
Count of seats: 2
Engine: volume - 4.0; mileage - 0.0
Navigator: N/A
```

## 적용 케이스

- 다양한 매개변수로 오버로드된 생성자가 많을 때
- 타입이나 세부사항이 다른 다양한 객체를 생성해야 할 때
- java.lang.StringBuilder.append()
- java.lang.StringBuffer.append()
- java.nio.IntBuffer.put()

## 고려사항

- 코드의 전반적인 복잡성이 증가함

### References

- [https://refactoring.guru/ko/design-patterns/builder](https://refactoring.guru/ko/design-patterns/builder)