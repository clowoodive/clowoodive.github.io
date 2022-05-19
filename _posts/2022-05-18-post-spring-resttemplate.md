---
title:  "[Spring] RestTemplate 살펴보기"
excerpt: "클라이언트측 HTTP 엑세스를 위한 클래스"

categories:
  - Spring
tags:
  - Spring
  - HTTP
  - 동기식
last_modified_at: 2022-05-18T00:00:00
---


## RestTemplete?

Spring 3에서 도입된 클라이언트측 HTTP 엑세스를 위한 클래스이다. Blocking I/O 기반의 동기식으로 처리되며 thread-safe하고 callback을 사용 할수 있다. 
아래와 같은 주요 HTTP 메소드 호출을 위한 [메소드를 제공](https://spring.io/blog/2009/03/27/rest-in-spring-3-resttemplate)하며, Spring 5대에 와서는 동기식/비동기식을 모두 지원하는 새로운 HTTP client로 WebClient를 권장하고 있고 RestTemplate는 maintenance/deprecated 로 가닥이 잡히고 있다.

<!-- ![spring_resttemplate1]({{ '/assets/images/spring_resttemplate1.png' | relative_url }}){: .align-center} -->

| HTTP | RestTemplate |
| --- | --- |
| DELETE | delete(String, String...) |
| GET | getForObject(String, Class, String...) |
| HEAD | headForHeaders(String, String...) |
| OPTIONS | optionsForAllow(String, String...) |
| POST | postForLocation(String, Object, String...) |
| PUT | put(String, Object, String...) |

## 내부 동작 원리

![spring_resttemplate2]({{ '/assets/images/spring_resttemplate2.png' | relative_url }}){: .align-center}

바로 그 이미지다. 10개의 블로그를 살펴보면 7개쯤 이 이미지가 보이는 것 같다. 출처가 없어서 안쓰고 싶었지만 앞으로 보면서 이해하기에 좋을것 같아서 가져왔다.

간략하게 살펴보면 `HttpMessageConverter`는 송신한 객체를 HTTP 요청(json 등) 으로 변환하거나 수신한 응답(json 등)을 객체로 변환하고, `ClientHttpRequestInterceptor`로 가로채서 요청/응답을 로깅  할수도 있다. 그리고 `ClientHttpRequestFactory`를 통해 요청이나 응답 처리를 하고 에러 핸들링도 할 수 있다.

## 예제

RestTemplete는 디폴트 생성자를 사용하거나 `ClientHttpRequestFactory` 를 매개변수로 받는 경우 등 여러가지가 있지만 여기서는 `RestTemplateBuilder` 를 통한 방법으로 한다.

RestTemplate 생성 방식에 따라 connection pooling이 달라지니 실서비스 적용 시 주의깊게 살펴야 한다.
{: .notice--warning}

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

dependency는 starter-web이 필요하고

```java
@Autowired
RestTemplateBuilder restTemplateBuilder;

...

RestTemplate restTemplate = restTemplateBuilder.setConnectTimeout(Duration.ofSeconds(5)).setReadTimeout(Duration.ofSeconds(5)).build();
String result = restTemplate.getForObject("https://api.github.com/users/clowoodive/repos", String.class);
System.out.println(result);
```

예제는 아주 간략게 구성했다. `getForObject` 를 통해 github에서 제공하는 api로 나의 repository정보를 가져온다. 
이번에 `RestTemplateBuilder`를 처음 사용하면서 보니 builder에서도 connectTime과 readTime을 세팅할 수 있었다. 하지만 더 많은 세팅을 위해서는 기존처럼 `HttpComponentsClientHttpRequestFactory`, `RequestConfig` 를 커스터마이징 해서 RestTemplate을 생성하는 방법으로 해야한다.

- timeout 참고
    - connectTimeout : 소켓 연결을 맺기 까지(handshake)의 타임아웃.
    - connectionRequestTimeout : connection manager의 pool에서 connection을 꺼내 오는 타임아웃. manager의 최대 connection 수 등에 의해 영향받음.
    - readTimeout : 요청과 응답 사이의 타임아웃.(=socketTimeout)

<!--

[https://spring.io/blog/2009/03/27/rest-in-spring-3-resttemplate](https://spring.io/blog/2009/03/27/rest-in-spring-3-resttemplate)

[https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/boot-features-resttemplate.html](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/boot-features-resttemplate.html)

[https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch18s03.html](https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch18s03.html)

[https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#io.rest-client](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#io.rest-client)

[https://gunju-ko.github.io/spring/2018/08/24/RestTemplate.html](https://gunju-ko.github.io/spring/2018/08/24/RestTemplate.html)

[https://velog.io/@blxckdog7702/Connection-timeout-vs-Connection-request-timeout-vs-Socket-timeout](https://velog.io/@blxckdog7702/Connection-timeout-vs-Connection-request-timeout-vs-Socket-timeout)

-->