---
title: "[Design Pattern-생성] 싱글턴(Singleton) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 생성 패턴
last_modified_at: 2023-05-01T00:00:00
---


## 정의

클래스의 인스턴스를 하나만 가지도록 하고 전역적인 접근을 지원하는 패턴.

멀티 스레드 환경에서 다양한 문제점이 발생하므로 이를 대처 할 수 있는 구현을 고려해야 함.

![design-pattern-singleton1]({{ '/assets/images/design-pattern-singleton1.png' | relative_url }}){: .align-center}

## 예제

```java
// 멀티 스레드 환경에서 안전한 예제
class Singleton {

    public int data = 0;

    private Singleton() {
    }

    public static Singleton getInstance() {
//        try {
//            Thread.sleep(1000);
//        } catch (InterruptedException ex) {
//            ex.printStackTrace();
//        }
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        public static final Singleton INSTANCE = new Singleton();
    }
}

// 멀티 스레드 환경에서 안전하지 못한 예제
class Singleton {
    private static Singleton instance;

    public int data = 0;

    private Singleton() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

멀티스레드에 취약한 코드는 sleep을 삽입해서 쉽게 문제점을 확인 할 수 있고, 클래스 로더에 의해 static Instance 초기화가 한번만 호출되게 한 경우는 사실상 sleep으로 확인 할수 없고 의도한 대로 같은 인스턴스를 리턴함을 알 수 있음.

테스트 코드로 확인.

```java
@Test
void singleton() {
    List<Thread> threads = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        threads.add(new Thread(new SingletonThread()));
    }

    for (int i = 0; i < 10; i++) {
        threads.get(i).start();
    }

    // System.out 대기
    try {
        Thread.sleep(2000);
    } catch (InterruptedException ex) {
        ex.printStackTrace();
    }
}

class SingletonThread implements Runnable {
    @Override
    public void run() {
        Singleton singleton = Singleton.getInstance();
        System.out.println(singleton.toString());
    }
}
```

출력 결과.

```powershell
# 멀티 스레드 환경에서 안전한 예제
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6
clowoodive.example.designpattern.creation.Singleton@5d18f1e6

# 멀티 스레드 환경에서 안전하지 못한 예제
clowoodive.example.designpattern.creation.Singleton@2b311f5a
clowoodive.example.designpattern.creation.Singleton@49083233
clowoodive.example.designpattern.creation.Singleton@31bcca3f
clowoodive.example.designpattern.creation.Singleton@6099fa61
clowoodive.example.designpattern.creation.Singleton@dafcff4
clowoodive.example.designpattern.creation.Singleton@6099fa61
clowoodive.example.designpattern.creation.Singleton@1b60e4b5
clowoodive.example.designpattern.creation.Singleton@2193ddb7
clowoodive.example.designpattern.creation.Singleton@49b64275
clowoodive.example.designpattern.creation.Singleton@5fa343d0
```

## 적용 케이스

- 모든 프로그램 내의 클라이언트가 사용할 수 있는 단일 인스턴스가 필요한 경우
- 전역 변수들을 강하게 제어하고 싶을 때
- java.lang.Runtime.getRuntime()

## 고려사항

- 테스트가 힘듬
- 다중 스레드 환경에서 싱글턴 객체가 여러개 생성되지 않도록 주의
- “클래스의 인스턴스를 하나만 생성하게 한다” 와 “인스턴스의 전역 접근을 가능하게 한다” 라는 두가지 책임을 가져 단일 책임 춴칙을 위배함

### References

- [https://refactoring.guru/ko/design-patterns/singleton](https://refactoring.guru/ko/design-patterns/singleton)