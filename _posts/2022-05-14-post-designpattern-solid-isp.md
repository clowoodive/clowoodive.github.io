---
title:  "[Design Pattern-SOLID] 인터페이스 분리 원칙(Interface Segregation Principle)"
excerpt: "클래스는 자신이 사용하지 않는 메소드에 의존 관계를 맺으면 안된다"

categories:
  - Design Pattern
tags:
  - SOLID
last_modified_at: 2022-05-14T00:00:00
---


- ISP 정의

> 특정 클라이언트(=클래스)를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다.
또는 클래스는 자신이 사용하지 않는 메소드에 의존 관계를 맺으면 안된다.
> 

SOLID 원칙 중에 쉬운편인것 같은데 다른 것들이 어려워서 이것도 어려운 듯한 느낌마저 든다. 너무 깊게 생각하지 말고 간단하게 살펴보자.

![solid_isp1]({{ '/assets/images/solid_isp1.png' | relative_url }}){: .align-center}

자동차 인터페이스의 “기름을 넣는다” 메소드는 전기차에서는 사용하지 않으며 “배터리를 충전한다” 라는 메소드는 가솔린차, 경유차에서 사용하지 않는 메소드이다. 
이는 동력원에 따른 하위 자동차들이 하나의 인터페이스를 따르도록 범용적으로 설계해서 각 하위 차종들이 사용하지 않는 메소드에 의존하게 되는 예라고 볼 수 있다.

이 설계에 ISP를 적용해 보면,

![solid_isp2]({{ '/assets/images/solid_isp2.png' | relative_url }}){: .align-center}

각 하위 자동차들에 공통적인 메소드들은 그대로 두고 각각을 위한 메소드를 가지는 인터페이스를 분리해서 설계함으로써 사용하지 않는 메소드에 대한 의존성을 제거했다.
단일 책임 원칙(SRP)과 맥락이 비슷한 원칙이다.