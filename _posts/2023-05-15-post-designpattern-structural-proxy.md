---
title: "[Design Pattern-구조] 프록시(Proxy) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 구조 패턴
last_modified_at: 2023-05-15T00:00:00
---


## 정의

클라이언트가 사용하고있는 서비스의 인터페이스를 구현하고 해당 서비스 객체의 참조를 보유한 프록시 객체가 원래 서비스를 대체함으로써, 요청을 원래 서비스 객체로 전달하는 구조 패턴.

실제 서비스를 감싸고 있기에 원래 객체에 대한 접근을 제어하고, 전후로 작업을 추가할 수 있음.

![design-pattern-proxy1]({{ '/assets/images/design-pattern-proxy1.png' | relative_url }}){: .align-center}

## 예제

```java
interface Service {
    int method1();

    int method2();
}

class RealService implements Service {

    private final int a = 1;

    private final int b = 10;

    private final int c = 100;

    @Override
    public int method1() {
        System.out.println("RealService method1 call.");
        return a + b + c;
    }

    @Override
    public int method2() {
        System.out.println("RealService method2 call.");
        return c - b - a;
    }
}

class ServiceProxy implements Service {

    private Service service;

    private Integer cacheMethod1;

    private Integer cacheMethod2;

    public ServiceProxy(Service service) {
        this.service = service;
    }

    @Override
    public int method1() {
        if (cacheMethod1 == null) {
            cacheMethod1 = service.method1();
            return cacheMethod1;
        }

        System.out.println("ServiceProxy method1 call - using cache");
        return cacheMethod1;
    }

    @Override
    public int method2() {
        if (cacheMethod2 == null) {
            cacheMethod2 = service.method2();
            return cacheMethod2;
        }

        System.out.println("ServiceProxy method2 call - using cache");
        return cacheMethod2;
    }
}
```

테스트 코드로 확인.

```java
@Test
void proxy() {
    Service service = new RealService();
    ServiceProxy proxy = new ServiceProxy(service);

    // 원래 객체 사용
    proxy.method1();
    proxy.method2();

    // 프록시의 캐시 사용
    proxy.method1();
    proxy.method2();
}
```

출력 결과.

```powershell
RealService method1 call.
RealService method2 call.
ServiceProxy method1 call - using cache
ServiceProxy method2 call - using cache
```

## 적용 케이스

- 로드하기 무거운 서비스 객체가 필요 시에 초기화 되게 하고 싶을 때(가상 프록시 - 지연 초기화)
- 특정 클라이언트만 원래 객체에 접근하게 하고 싶을 때(보호 프록시 - 접근 제어)
- 원격 서비스를 이용할 때 복잡한 로직들을 프록시에서 모두 처리하게 할 때(원격 프록시)
- 원래 서비스 객체에 대한 요청들을 로깅 하고 싶을 때(로깅 프록시)
- 원래 서비스로의 요청 결과를 캐시하고 캐시 수명을 관리 하고 싶을 때(캐싱 프록시)
- 원래 서비스 객체의 클라이언트들이 참조중인지 확인하여 자원을 해제하는 등의 관리가 필요할때
- java.lang.reflect.Proxy
- javax.inject.Inject

## 고려사항

- 서비스의 응답이 늦어질 수 있음

### References

- [https://refactoring.guru/ko/design-patterns/proxy](https://refactoring.guru/ko/design-patterns/proxy)