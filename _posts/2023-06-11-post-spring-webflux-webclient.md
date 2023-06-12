---
title: "[Spring-WebFlux] WebClient 사용 방법"
excerpt: ""

categories:
  - Spring
tags:
  - Spring
  - Spring-WebFlux
  - Spring Boot 
  - WebClient
  - RestTemplate
last_modified_at: 2023-06-11T00:00:00
---

{% capture notice-env %}
#### Environment
 - Java 11
 - Spring 5.3.18
 - Spring Boot 2.5.12
 - Gradle 7.5.1 
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


# 1. WebClient란?

Spring WebFlux에 포함된 HTTP client로 스레드나 동시성을 직접 다루지 않고 비동기 논리의 선언적 구성을 가능하게 하고 non-blocking 방식으로 스트리밍을 지원함. 

동기화 방식으로 사용하면 RestTemplate를 대체 할 수 있음.

아래와 같은 client library 들이 사용 가능하며, 후에 살펴볼  WebClient 생성 방식이 무엇이든 Spring Boot에서 default는 Reactor Netty의 `ReactorClientHttpConnector` 로 설정됨.

- [Reactor Netty](https://github.com/reactor/reactor-netty)
- [Jetty Reactive HttpClient](https://github.com/jetty-project/jetty-reactive-httpclient)
- [Apache HttpComponents](https://hc.apache.org/index.html)

  

<br>

<br>

# 2. 구성

## 2.1 WebClient 생성(Static Method)

```java
WebClient webClient = WebClient.create();
```

또는 아래와 같이 base url을 포함해서 생성 할 수 있음.

```java
WebClient webClient = WebClient.create("https://jsonplaceholder.typicode.com");
```

## 2.2 WebClient 생성(Builder)

```java
WebClient webClient = WebClient.builder()
                .baseUrl("https://jsonplaceholder.typicode.com")
                .build();
```

그리고 builder 인터페이스로 아래와 같은 기능을 제공함.

- `baseUrl`
- `uriBuilderFactory` : `baseUrl(String)`, `defaultUriVariables(Map)`와 같은 기능 설정/재정의를 위한 미리 구성된 인스턴스.
- `defaultUriVariables` : Map 형식으로 URI 템플릿을 확장할 때 사용할 기본 URL 매개변수 설정.
- `defaultHeader`
- `defaultCookie`
- `defaultRequest` : 모든 요청에 속성을 삽입할수 있음. 예를 들면 Spring MVC에서 Thread Local 데이터를 기반으로 요청 속석을 채우는 것.
- `filter` : 요청을 인터셉트하고 수정할 수 있음.
- `exchangeStrategies` : HTTP 메세지 reader/writer를 커스터마이징 할 수 있음.
- `clientConnector` : HTTP client 라이브러리 세팅. 디폴트는 앞서 설명한 것과 같이 `ReactorClientHttpConnector` .

builder 는 한번 선언되면 변경 불가능하지만,  복제 후 수정은 가능함.

따라서 아래의 코드에서 client2는 filterA, filterB, filterC, filterD가 모두 설정됨.

```java
WebClient client1 = WebClient.builder()
        .filter(filterA).filter(filterB)
        .build();

WebClient client2 = client1.mutate()
        .filter(filterC).filter(filterD)
        .build();
```

## 2.3 ****MaxInMemorySize -**** 버퍼링 데이터 제한

코덱의 버퍼링 데이터에 대한 메모리 제한이 있는데 이를 초과하면 에러가 발생함.

이 경우 제한 수치를 변경하는 방법은 아래와 같음.

```java
WebClient webClient = WebClient.builder()
        .codecs(configurer -> configurer.defaultCodecs().maxInMemorySize(2 * 1024 * 1024))
        .build();
```

---

# 3. Reactor Netty client로 구성

## 3.1 Reactor Netty client를 사용자 지정 할 경우 설정

```java
HttpClient httpClient = HttpClient.create().secure(sslSpec -> ...);

WebClient webClient = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

## 3.2 Resource

기본적으로 `HttpClient` 는 event loop threads 와 connection pool을 포함하는 Reactor Netty resources에 참여하게 되고, 이 글로벌 리소스는 프로세스 종료 전까지 활성화 된 상태로 유지됨.

프로세스와 서버가 생명주기를 같이 하지 않는 경우(e.g. WAR로 배포된 Spring MVC 애플리케이션) 스프링 ApplicationContext가 종료될 때 Reactor Netty 글로벌 리소스가 정리되도록 아래와 같이 `ReactorResourceFactory` 라는 Spring-managed bean을 사용 할 수 있음.

```java
@Bean
public ReactorResourceFactory reactorResourceFactory() {
    return new ReactorResourceFactory();
}
```

글로벌 리소스를 사용하지 않게 설정하는 경우는 [공식문서](https://docs.spring.io/spring-framework/docs/5.3.18/reference/html/web-reactive.html#webflux-client-builder) 참고.

## 3.3 Timeouts

connection timeout.

```java
import io.netty.channel.ChannelOption;

HttpClient httpClient = HttpClient.create()
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 10000);

WebClient webClient = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

read or write timeout.

```java
import io.netty.handler.timeout.ReadTimeoutHandler;
import io.netty.handler.timeout.WriteTimeoutHandler;

HttpClient httpClient = HttpClient.create()
        .doOnConnected(conn -> conn
                .addHandlerLast(new ReadTimeoutHandler(10))
                .addHandlerLast(new WriteTimeoutHandler(10)));

// Create WebClient...
```

response timeout(모든 요청).

```java
HttpClient httpClient = HttpClient.create()
        .responseTimeout(Duration.ofSeconds(2));

// Create WebClient...
```

response timeout(특정 요청).

```java
WebClient.create().get()
        .uri("https://example.org/path")
        .httpRequest(httpRequest -> {
            HttpClientRequest reactorRequest = httpRequest.getNativeRequest();
            reactorRequest.responseTimeout(Duration.ofSeconds(2));
        })
        .retrieve()
        .bodyToMono(String.class);
```

# 4. 용어 개념

## 4.1 Mono & Flux reactive type

- Mono는 0~1개의 응답을 받는 개념의 publisher
- Flux는 0~N개의 응답을 받는 개념의 publisher

## 4.2 sync & async

- 동기식은 `subscribe()` 등을 사용해서 요청/응답 콜백
- 비동기식은 `block()` 을 사용해서 요청/응답 처리

# 5. retrieve()

응답을 간단하게 추출하는 메서드.

## 5.1 toEntity()

ResponseEntity 타입으로 body를 추출.

```java
WebClient client = WebClient.create("https://example.org");

Mono<ResponseEntity<Person>> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .toEntity(Person.class);
```

## 5.2 bodyToMono()

body만 추출.

```java
WebClient client = WebClient.create("https://example.org");

Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .bodyToMono(Person.class);
```

## 5.3 bodyToFlux()

디코딩된 객체 스트림을 추출.

```java
Flux<Quote> result = client.get()
        .uri("/quotes").accept(MediaType.TEXT_EVENT_STREAM)
        .retrieve()
        .bodyToFlux(Quote.class);
```

## 5.4 에러 상태 코드에 따른 처리

```java
Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError, response -> response.bodyToMono(String.class).map(CustomServerErrorException::new))
        .onStatus(HttpStatus::is5xxServerError, response -> ...)
        .bodyToMono(Person.class);
```

# 6. Exchange

응답 상태코드에 따른 처리.

```java
Mono<Person> entityMono = client.get()
        .uri("/persons/1")
        .accept(MediaType.APPLICATION_JSON)
        .exchangeToMono(response -> {
            if (response.statusCode().equals(HttpStatus.OK)) {
                return response.bodyToMono(Person.class);
            }
            else {
                // Turn to error
                return response.createException().flatMap(Mono::error);
            }
        });
```

# 7. Request Body

아래 예제 방식 외에도 Mono/Flux 타입 객체를 body로 인코딩 할 수 있음.

```java
Person person = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue(person)
        .retrieve()
        .bodyToMono(Void.class);
```

# 8. Filters

요청을 가로채서 검사하거나 수정, 로깅 할 수 있음.

## 8.1 사용자 지정 filter 사용법

```java
ExchangeFilterFunction filterFunction = (clientRequest, nextFilter) -> {
    LOG.info("WebClient fitler executed");
    return nextFilter.exchange(clientRequest);
};

WebClient webClient = WebClient.builder()
  .filter(filterFunction)
  .build();
```

## 8.2 표준 filter

인증 헤더를 요청에 추가해주는 `basicAuthentication()` 제공.

```java
import static org.springframework.web.reactive.function.client.ExchangeFilterFunctions.basicAuthentication;

WebClient client = WebClient.builder()
        .filter(basicAuthentication("user", "password"))
        .build();
```

# 9. Attributes

WebClient.Builder 레벨에서 전역 콜백을 구성해서 모든 요청에 속성을 삽입 할 수 있음(i.e. Spring MVC 애플리케이션에서 ThreadLocal 데이터를 기반으로 요청 속성을 채우기).

아래는 속성을 삽입해서 필터 동작에 영향을 주려는 예제.

```java
WebClient client = WebClient.builder()
        .filter((request, next) -> {
            Optional<Object> usr = request.attribute("myAttribute");
            // ...
        })
        .build();

client.get().uri("https://example.org/")
        .attribute("myAttribute", "...")
        .retrieve()
        .bodyToMono(Void.class);

    }
```

# 10. 동기식 사용(Synchronous Use)

## 10.1 단일 요청

```java
Person person = client.get().uri("/person/{id}", i).retrieve()
    .bodyToMono(Person.class)
    .block();

List<Person> persons = client.get().uri("/persons").retrieve()
    .bodyToFlux(Person.class)
    .collectList()
    .block();
```

## 10.2 다중 요청

여러 요청을 각각 block하는 것은 비효율 적이므로, 묶어서 요청하고 block 하는 방식도 가능.

```java
Mono<Person> personMono = client.get().uri("/person/{id}", personId)
        .retrieve().bodyToMono(Person.class);

Mono<List<Hobby>> hobbiesMono = client.get().uri("/person/{id}/hobbies", personId)
        .retrieve().bodyToFlux(Hobby.class).collectList();

Map<String, Object> data = Mono.zip(personMono, hobbiesMono, (person, hobbies) -> {
            Map<String, String> map = new LinkedHashMap<>();
            map.put("person", person);
            map.put("hobbies", hobbies);
            return map;
        })
        .block();
```

# 11. Spring MVC controller 에서의 Mono/Flux

Spring MVC나 Spring WebFlux controller에서 클라이언트 호출 결과를 반환해야 할 경우에는 block을 사용하지 않고 reactive type(Mono/Flux)를  return 하면 됨.

<!--

예제 코드가 적용된 프로젝트는 [여기](https://github.com/clowoodive/toy/tree/main/investing).

-->

### References

- [https://docs.spring.io/spring-framework/docs/5.3.18/reference/html/web-reactive.html#webflux-client-body](https://docs.spring.io/spring-framework/docs/5.3.18/reference/html/web-reactive.html#webflux-client)