---
title: "[Design Pattern-행위] 관찰자(Observer) 패턴"
excerpt: ""

categories:
  - Design Pattern
tags:
  - 행위 패턴
last_modified_at: 2023-04-18T00:00:00
---


## 정의

변경 될 수 있는 객체(subject)에 대한 변경 이벤트를 자신(observer, 관찰자)을 구독한 객체들에게 알리는 매커니즘을 정의하는 패턴. 구독 객체들이 구독/구독취소를 동적으로 할 수 있음.

- 커맨드 패턴 - 발신자와 수신자 간의 단방향 연결
- 중재자 패턴 - 객체들간의 직접 연결을 제거하고 중재자가 간접적으로 전달
- 책임연쇄 패턴 - 수신자 중 하나에 의해 요청이 처리 될 때까지 요청을 순차적으로 전달

![design-pattern-observer1]({{ '/assets/images/design-pattern-observer1.png' | relative_url }}){: .align-center}

## 예제

```java
interface Subscriber {
    void update(String eventType, String eventMessage);
}

class EventManager {
    private Map<String, List<Subscriber>> subscribers = new HashMap<>();

    public void subscribe(String eventType, Subscriber subscriber) {
        List<Subscriber> subList = this.subscribers.get(eventType);
        if (subList == null) {
            subList = new ArrayList<>();
            subscribers.put(eventType, subList);
        }
        subList.add(subscriber);
    }

    public void unsubscribe(String eventType, Subscriber subscriber) {
        List<Subscriber> subList = this.subscribers.get(eventType);
        if (subList != null) {
            subList.remove(subscriber);
        }
    }

    protected void notify(String eventType, String eventMessage) {
        List<Subscriber> subList = this.subscribers.get(eventType);
        if (subList != null) {
            subList.forEach(subscriber -> subscriber.update(eventType, eventMessage));
        }
    }
}

public class ObserverSubject {

    private static final int UPPER_BOUND_GAUGE = 80;

    private static final int LOWER_BOUND_GAUGE = 20;

    public EventManager eventManager = new EventManager();

    private int gauge;

    public void setGauge(int gauge) {
        this.gauge = gauge;

        if (this.gauge >= UPPER_BOUND_GAUGE) {
            eventManager.notify("upper", "gauge is over " + UPPER_BOUND_GAUGE + ", now " + this.gauge);
        } else if (this.gauge <= LOWER_BOUND_GAUGE) {
            eventManager.notify("lower", "gauge is under " + LOWER_BOUND_GAUGE + ", now " + this.gauge);
        }
    }
}

class AlarmSubscriber implements Subscriber {
    @Override
    public void update(String eventType, String eventMessage) {
        System.out.println("AlarmSubscriber receive Type(" + eventType + ") Message(" + eventMessage + ")");
    }
}

class UpperAlarmSubscriber implements Subscriber {
    @Override
    public void update(String eventType, String eventMessage) {
        System.out.println("UpperAlarmSubscriber receive Type(" + eventType + ") Message(" + eventMessage + ")");
    }
}
```

테스트 코드로 확인.

```java
@Test
void observer() {
    ObserverSubject subject = new ObserverSubject();

    Subscriber alarmSubscriber = new AlarmSubscriber();
    subject.eventManager.subscribe("upper", alarmSubscriber);
    subject.eventManager.subscribe("lower", alarmSubscriber);

    Subscriber upperAlarmSubscriber = new UpperAlarmSubscriber();
    subject.eventManager.subscribe("upper", upperAlarmSubscriber);

    subject.setGauge(50);
    subject.setGauge(80);

    subject.eventManager.unsubscribe("upper", alarmSubscriber);
    subject.setGauge(90);
    subject.setGauge(19);
}
```

출력 결과.

```powershell
AlarmSubscriber receive Type(upper) Message(gauge is over 80, now 80)
UpperAlarmSubscriber receive Type(upper) Message(gauge is over 80, now 80)
# alarmSubscriber unsubscribe "upper"
UpperAlarmSubscriber receive Type(upper) Message(gauge is over 80, now 90)
AlarmSubscriber receive Type(lower) Message(gauge is under 20, now 19)
```

## 적용 케이스

- 한 객체의 변경상태를 알아야 할 다른 객체들을 미리 알수 없거나 동적으로 변할 때
- 한 객체의 변경상태를 일정 시간 또는 특정 경우에만 다른 객체들이 알아야 할 때
- java.util.EventListener
- javax.servlet.http.HttpSessionAttributeListener

## 고려사항

- 기존 클래스 계층 구조에 관찰자 패턴 적용시 합성 방식(구독/구독취소/알림을 담당하는 별도의 manager 객체를 subject가 의존)을 고려하고, 아닌 경우에는 subject가 구독 매커니즘 로직을 가져도 됨
- 중재자 패턴은 객체들간의 상호 의존성을 제거하면서 중재자를 통한 양방향 통신이 가능하고, 관찰자 패턴은 단방향 연결이지만 구독/구독 취소를 동적으로 할 수 있는 차이점이 있음

### References

- [https://refactoring.guru/ko/design-patterns/observer](https://refactoring.guru/ko/design-patterns/observer)