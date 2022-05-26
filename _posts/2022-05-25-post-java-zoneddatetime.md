---
title:  "[Java] ZonedDateTime.withZoneSameInstant()를 이용한 시간 변환(feat. 다양한 UTC 시간대의 국가들에서 유연한 서비스를 위한 timezone변환)"
excerpt: ""

categories:
  - Java
tags:
  - Java
last_modified_at: 2022-05-26T00:00:00
---

{% capture notice-env %}
#### Environment

- Spring Boot 2.5.13
- Spring 5.3.19
- Java 11
{% endcapture %}

<div class="notice--primary">{{ notice-env | markdownify }}</div>

## 개요

동일한 서버를 여러 국가에 서비스 하다보니 기획파트에서 특정 시간을 기준으로 동작해야 하는 기능들을 세팅하면서 실수 할 가능성이 높아 보였다. 글로벌 서비스이거나 처음부터 다수의 국가를 고려했다면 지금과는 다른 상태였겠지만 한 국가에서 서비스 하면서 타 국가로 확장되었기에 유연성이 떨어지는 상태였다. 한국 시간으로 모든 국가의 데이터를 세팅해야 한다면 매 국가마다 한국 시간으로 변환해야 하는데 시스템으로 지원하지 않는다면 휴먼에러가 발생할 가능성이 너무 높다. 기획의 실수는 곧 서버 파트에서 대응해야 하는 경우가 대부분이었기에 추후 대응 비용을 줄이기 위해 개선하기로 했다.

서버는 한국(UTC+9)에 있고 몰디브(UTC+5)에 서비스 한다고 가정해 보자. 사실 서버가 어디 있던지 서버 timezone은 변경 할 수 있지만 운영/관리 측면에서 서비스 국가의 timezone과 서버의 timezone을 맞추지 않는 것이 유리 할 수도 있기에 이 둘이 다를 수 있다.
해결을 위해 요구사항을 아래와 같이 정했다.

- 기획파트는 서비스 국가의 timezone을 세팅하고, 컨텐츠의 기준 시간도 해당 국가의 시간을 기준으로 세팅 함(한국 시간으로 할 경우 국가별 시간을 한국 시간으로 변환 하면서 발생 할 수 있는 실수 방지).
- 서버는 기획 데이터의 timezone을 이용해 서버의 timezone과 무관하게 동작하게 함.

먼저 짚고 넘어 갈 것은 운영체제의 timezone이 JVM의 default timezone이 되고, 따라서 클라이언트의 요청이 서버로 들어온 시간으로 사용하는 LocalDateTime.now()도 결국 운영체제의 timezone 기준이라는 것이다.

정리 해 보면 기획에서 정의한 컨텐츠 기준 시간을 서버 운영체제의 timezone에 맞게 변환하면 된다는 뜻이다.

## 구현

몰디브(UTC+5) 기준으로 2022년 5월 20일 0시에 컨텐츠가 시작된다고 하면, 한국(UTC+9)로 세팅된 서버에서 해당 시간은 4시간을 더한 2022년 5월 20일 4시가 되어야 한다. 이를 계산하기 위해 서비스 국가의 timezone 과 컨텐츠 기준시간을 파라미터로 받고, 서버 default timezone으로 변환된 DateTime을 얻어오는 메소드를 만들었다.

```java
public class ZonedDateTimeService {
    public LocalDateTime convertToServerTimeZonedDateTime(int serviceTimeZone, LocalDateTime serviceCheckTime) {
        var serviceZonedDateTime = ZonedDateTime.of(serviceCheckTime, ZoneOffset.ofHours(serviceTimeZone));
        var systemDefaultZoneId = ZoneId.systemDefault();

        var zonedDateTime = serviceZonedDateTime.withZoneSameInstant(systemDefaultZoneId);

        return zonedDateTime.toLocalDateTime();
    }
}
```

컨텐츠 시작 시간(`serviceCheckTime`)에 서비스 국가의 timezone을 반영한 `serviceZonedDateTime` 객체를 만들고, `withZoneSameInstant()`으로 서버의 default timezone을 적용해서 변환된 `LocalDateTime`을 반환한다.

기획 데이터인 `serviceTimeZone`과 `serviceCheckTime`은 편의상 로컬 변수로 정의했고, 호출부는 아래와 같다.

```java
public ApplicationRunner applicationRunner() {
    return args -> {
        int serviceTimeZone = 5;    // UTC+5 몰디브
        LocalDateTime serviceCheckTime = LocalDateTime.parse("2022-05-20T00:00:00");

        var serverLocalDateTime = zonedDateTimeService.convertToServerTimeZonedDateTime(serviceTimeZone, serviceCheckTime);

        System.out.println("service area time(UTC+" + serviceTimeZone + ") : " + serviceCheckTime);
        System.out.println("zoned time by server default timezone(UTC+" + ZoneId.systemDefault() + ") : " + serverLocalDateTime);
    };
}
```

실행 해 보면 아래와 같이 서버의 시간에 맞게 반환된다. 

```
service area time(UTC+5) : 2022-05-20T00:00
zoned time by server default timezone : 2022-05-20T04:00
```

이제 서비스 국가의 timezone과 해당 국가의 UTC 기준 시간만 기획적으로 잘 세팅하면 위 코드를 이용해서 서버가 어디에 있던지(운영체제 timezone이 무엇이던지) 변환해서 서비스 할 수 있다.

구현 자체는 간단한데 어디 시간을 기준으로 기획 데이터를 세팅할지, 어떻게 하면 휴먼에러를 최소화 할 수 있을까 하는 것들을 고민하는데 시간이 더 많이 든다.