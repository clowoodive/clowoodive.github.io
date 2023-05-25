---
title: "[Design Pattern-구조] 브리지(Bridge) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 구조 패턴
last_modified_at: 2023-05-24T00:00:00
---


## 정의

큰 클래스나 밀접하게 관련된 여러 클래스들의 집합을 두개의 계층구조(추상화/구현)로 분리해서 독립적으로 개발 할수 있도록 하는 패턴. 

추상화(=인터페이스. i.e. 리모컨 )는 Java의 인터페이스를 말하는 것이 아니라 구현(=플랫폼. i.e. 디바이스)에 작업들을 위임하는 상위 수준의 제어 레이어를 뜻함.

분리된 새로운 계층(구현)의 객체는 상속이 아닌 합성으로서 원래 클래스에서 참조하게 되는데. 이 참조가 추상화 계층과 구현 계층의 다리(브리지) 역할을 함.

## 구조

![design-pattern-bridge1]({{ '/assets/images/design-pattern-bridge1.png' | relative_url }}){: .align-center}

## 예제

```java
interface Device {
    boolean isEnabled();

    void enable();

    void disable();

    int getVolume();

    void setVolume(int vol);

    void printStatus();
}

class Tv implements Device {
    private boolean on;

    private int volume = 30;

    @Override
    public boolean isEnabled() {
        return on;
    }

    @Override
    public void enable() {
        on = true;
    }

    @Override
    public void disable() {
        on = false;
    }

    @Override
    public int getVolume() {
        return volume;
    }

    @Override
    public void setVolume(int vol) {
        if (vol > 100) {
            this.volume = 100;
        } else if (vol < 0) {
            this.volume = 0;
        } else {
            this.volume = vol;
        }
    }

    @Override
    public void printStatus() {
        System.out.println("---------------- TV ----------------");
        System.out.println("| Power is " + (on ? "enabled" : "disabled"));
        System.out.println("| Volume is " + volume + "%");
        System.out.println("------------------------------------\n");
    }
}

class Radio implements Device {
    private boolean on;

    private int volume = 30;

    @Override
    public boolean isEnabled() {
        return on;
    }

    @Override
    public void enable() {
        on = true;
    }

    @Override
    public void disable() {
        on = false;
    }

    @Override
    public int getVolume() {
        return volume;
    }

    @Override
    public void setVolume(int vol) {
        if (vol > 100) {
            this.volume = 100;
        } else if (vol < 0) {
            this.volume = 0;
        } else {
            this.volume = vol;
        }
    }

    @Override
    public void printStatus() {
        System.out.println("--------------- Radio ---------------");
        System.out.println("| Power is " + (on ? "enabled" : "disabled"));
        System.out.println("| Volume is " + volume + "%");
        System.out.println("------------------------------------\n");
    }
}

interface Remote {
    void power();

    void volumeUp();

    void volumeDown();
}

class SimpleRemote implements Remote {
    protected Device device;

    public SimpleRemote(Device device) {
        this.device = device;
    }

    @Override
    public void power() {
        System.out.println("Remote: power toggle");
        if (device.isEnabled()) {
            device.disable();
        } else {
            device.enable();
        }
    }

    @Override
    public void volumeUp() {
        System.out.println("Remote: volume down");
        device.setVolume(device.getVolume() + 10);
    }

    @Override
    public void volumeDown() {
        System.out.println("Remote: volume up");
        device.setVolume(device.getVolume() - 10);
    }
}

class PremiumRemote extends SimpleRemote {

    public PremiumRemote(Device device) {
        super(device);
    }

    public void mute() {
        System.out.println("Remote: mute");
        device.setVolume(0);
    }
}
```

테스트 코드로 확인.

```java
@Test
void bridge() {
    testDevice(new Tv());
    testDevice(new Radio());
}

private void testDevice(Device device) {
    System.out.println("Tests with simple remote.");
    SimpleRemote simpleRemote = new SimpleRemote(device);
    simpleRemote.power();
    device.printStatus();

    System.out.println("Tests with premium remote.");
    PremiumRemote premiumRemote = new PremiumRemote(device);
    premiumRemote.power();
    premiumRemote.mute();
    device.printStatus();
}
```

출력 결과.

```powershell
Tests with simple remote.
Remote: power toggle
---------------- TV ----------------
| Power is enabled
| Volume is 30%
------------------------------------

Tests with premium remote.
Remote: power toggle
Remote: mute
---------------- TV ----------------
| Power is disabled
| Volume is 0%
------------------------------------

Tests with simple remote.
Remote: power toggle
--------------- Radio ---------------
| Power is enabled
| Volume is 30%
------------------------------------

Tests with premium remote.
Remote: power toggle
Remote: mute
--------------- Radio ---------------
| Power is disabled
| Volume is 0%
------------------------------------
```

## 적용 케이스

- 여러 변형을 가진 모놀리식 클래스를 나누고 정돈하고 싶을 때
- 런타임에 구현을 전환하고 싶을 때
- 크로스 플랫폼 앱 처리, 여러 데이터베이스 처리, 같은 종류의 여러 API 처리 시 사용

## 고려사항

- 전략 패턴과 혼동 될 수 있음
- 결합도가 높은 클래스에 적용하면 코드가 더 복잡해질 수 있음

### References

- [https://refactoring.guru/ko/design-patterns/bridge](https://refactoring.guru/ko/design-patterns/bridge)