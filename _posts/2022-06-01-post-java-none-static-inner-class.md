---
title:  "[Java] non-static inner class 인스턴스화"
excerpt: ""

categories:
  - Java
tags:
  - Java
last_modified_at: 2022-06-01T00:00:00
---

{% capture notice-env %}
#### Environment
- Spring Boot 2.5.13
- Spring 5.3.19
- Java 11
{% endcapture %}

<div class="notice--primary">{{ notice-env | markdownify }}</div>

```java
var res = restTemplate.exchange(domain, HttpMethod.POST, new HttpEntity<String>(body, headers), CouponProto.AcquireRes.class);
```

위와 같이 `RestTemplete` 으로 POST 요청을 보내고 받은 응답을 객체에 담는 과정에서 아래와 같은 에러가 발생했다.

```bash
Caused by: com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.XXX.CouponProto$AcquireRes`: non-static inner classes like this can only by instantiated using default, no-argument constructor
 at [Source: (org.springframework.util.StreamUtils$NonClosingInputStream); line: 1, column: 2]
```

원인은 ***“non-static inner classes like this can only by instantiated using default, no-argument constructor”*** 이 부분인데, 테스트 하느라 이것 저것 수정하다가 `AcquireRes` inner class의 static이 누락되어 발생한 문제였다.

non-static inner class는 인수가 없는 기본 생성자로만 인스턴스화 할 수 있기에, 받아 온 응답의  값들을 인수로 전달해서 인스턴스화 하려다 실패한 것이다.