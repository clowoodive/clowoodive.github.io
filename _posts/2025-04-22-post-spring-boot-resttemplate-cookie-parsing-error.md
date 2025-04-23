---
title: "[Spring Boot] RestTemplate 응답 쿠키의 Expires 파라미터 파싱 에러 문제"
excerpt: ""

categories:
  - Spring
tags:
  - Spring
  - RestTemplate
  - Cookie
last_modified_at: 2025-04-22T00:00:00
---

{% capture notice-env %}
#### Environment
 - Java 11
 - Spring 5.3.18
 - Spring Boot 2.5.12
 - Gradle 6.9.2
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


# 1. 문제 발생

서버에서 외부 API 서버로 요청을 보내고 응답을 받았는데,  응답 쿠키 파싱을 하면서 경고가 발생하는것을 발견했다. 우리 서버가 관련해서 변경된 것은 없었고 요청받은 서버에서 변경사항이 있었던 것으로 보인다.

```
[main] WARN org.apache.http.client.protocol.ResponseProcessCookies - Invalid cookie header: "Set-Cookie: nsession=s%3ARyJSs5_NJCZ73zU81T1HnBIpeOIkKP6s.pVQw08yh0PUlPVXAhomTIcNKUFjxEC1osDz9h4W5LVo; Path=/; Expires=Thu, 06 Mar 2025 03:16:42 GMT; HttpOnly". Invalid 'expires' attribute: Thu, 06 Mar 2025 03:16:42 GMT
```

<br>

<br>

# 2. 원인

아래와 같이 따라 들어가면서 디버깅을 해보면, 쿠키 파싱을 하면서 마지막 BaseicExpiresHandler에서 경고가 발생하는데
processCookies() - ResponseProcessCookies.java

parse() - DefaultCookieSpec.java

parse() - CookieSpecBase.java

parse() - BasicExpiresHandler

아래와 같이 실제 쿠키의 expires 값과 datePatterns의 년/월/일 형식이 맞지 않아서 발생하고 있었다.

![spring-boot-resttemplate-cookie-parsing-error1]({{ '/assets/images/spring-boot-resttemplate-cookie-parsing-error1.png' | relative_url }}){: .align-center}

파싱 형식이 안맞는 이유를 조사해 보니 ClientHttpRequestFactory을 사용하여 내부에서 HttpClients.createSystem() 이 호출되어 HttpClient가 생성되면 JVM의 설정을 따르는데, JVM에도 명시적으로 설정되지 않아 default CookieSpec으로 세팅되어 있기 때문인 것으로 확인되었다.

default CookieSpec의 파싱 형식은 ‘-’ 가 포함된 RFC 1036 규격(위 그림에서 this.datePatterns)이며, 실제 수신된 쿠키 값(위 그림에서 value)은 RFC 1123 형식이어서 파싱이 안되는 것이다.

- RFC 1036, RFC 1123 : HTTP 헤더의 날짜 형식 규격이며 쿠키의 Expires 속성이 이를 따름.
- RFC 6265 : 쿠키 속성(Expires, Max-Age, Domain, Path  등) 규격. Expires의 경우 RFC 1123형식이 지정되어 있음.

<br>

<br>

# 3. 현재 설정 확인

HttpClient에 세팅된 CookieSpec 설정은 logging dubug가 활성화 되어 있으면 요청 시 아래와 같이 확인이 바로 가능하고

```
[main] DEBUG org.apache.http.client.protocol.RequestAddCookies - CookieSpec selected: default
```

설정들은 아래 코드로 확인 가능하다.

```java
// Java 시스템 프로퍼티 확인
Properties props = System.getProperties(); // http.cookie.policy
props.keySet().stream()
        .map(Object::toString)
        .filter(key -> key.toLowerCase().contains("cookie") || key.toLowerCase().contains("http"))
        .forEach(key -> logger.debug("cookie check, " + key + " = " + System.getProperty(key)));

// Apache HttpClient
CloseableHttpClient httpClient = HttpClients.createSystem(); // 시스템 설정 적용
RequestConfig config1 = RequestConfig.custom().build();
System.out.println("cookie check, Default CookieSpec: " + config1.getCookieSpec());
```

<br>

<br>

# 4. 해결 방안

CookieSpec을 default가 아닌 `standard`로 변경하면 Expires 속성에 RFC 1123 규격을 사용하는 RFC 6265 표준 모드로 쿠키 처리를 할 수 있게 되는데, 변경하는 방법은 아래와 같이 여러 곳에 적용 할 수 있다고 한다.

- Java 실행 시

```bash
java -Dhttp.cookie.policy=standard -jar myapp.jar
```

- Spring Boot의 application.properties 설정(Apache HttpClient는 이 설정을 읽지 않음)

```bash
system.http.cookie.policy=standard
```

- JVM 설정 변경

```java
System.setProperty("http.cookie.policy", "standard");
```

- 명시적으로 standard로 설정해서 HttpClient  생성

```java
CloseableHttpClient httpClient = HttpClients.custom()
                .setDefaultRequestConfig(RequestConfig.custom()
                        .setCookieSpec(CookieSpecs.STANDARD)
                        .build())
                .build();
                
HttpComponentsClientHttpRequestFactory clientHttpRequestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
restTemplate = new RestTemplate(getClientHttpRequestFactory());
```

<br>

<br>

# 5. 해결

코드 구조 상 마지막 방법인 명시적으로 STANDARD로 세팅한 HttpClinet를 사용하기로 했고, 이를 위해서 우선 의존성을 추가해야 한다.

```bash
# build.gradle
implementation 'org.apache.httpcomponents:httpclient' #4.5.13
```

그리고 httpClient 생성 코드를 적용 후 process() - RequestAddCookies.java  메서드의 아래 코드를 디버깅 해보면 RFC6265로 세팅하는 것을 확인할 수 있다.

```java
context.setAttribute(HttpClientContext.COOKIE_SPEC, cookieSpec); // rfc6265-lax
```

실제로 요청 응답 처리를 디버깅을 하면서 따라가 보면 기존과 달리 CookieSpecBase 대신 RFC6265CookieSpec을 사용하고, 파싱 핸들러는 BasicExpiresHandler 대신 LaxExpiresHandler를 사용하게 되어 문제없이 파싱되는 것을 확인할 수 있다.