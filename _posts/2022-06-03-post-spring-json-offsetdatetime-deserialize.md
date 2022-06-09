---
title:  "[Spring] RestTemplate의 json 응답을 OffsetDateTime으로 deserialize 하기"
excerpt: ""

categories:
  - Spring
tags:
  - Json
  - Deserialize
last_modified_at: 2022-06-03T00:00:00
---

{% capture notice-env %}
#### Environment
- Java 11
- Spring Boot 2.6.8
- Spring Cloud Dependencies 2021.0.3
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


RestTemplete으로 받은 json 응답 중 "expire_at":"2022-06-10T23:59:59+05:00”와 같은 값을 OffsetDateTime 타입으로 deserialize하면서 offset이 ‘Z’로 변경되는 문제를 해결하기 위해 시도한 내용들을 기록한다.

### RestTemplate 호출

아래와 같이 호출해서 받은 json 응답을 `RestTemplateProto.Res` 객체에 매핑하면서 deserialize 한다.

```java
var restTemplate = new RestTemplate();
            var res = restTemplate.exchange(domain, HttpMethod.POST, new HttpEntity<String>(body, headers), RestTemplateProto.Res.class);
            System.out.println("expireAt: "res.getBody().getExpireAt());
```

매핑될 클래스는 아래와 같다.

```java
public abstract class RestTemplateProto {
	public static class Res {
		... 생략
		private OffsetDateTime expireAt;
		... 생략
	}
}
```

응답으로 받은 json 중 다른 필드들은 문제 없이 매핑되었으나 유독 아래 필드만 OffsetDateTime으로 매핑이 실패했다.

```json
"expireAt":"2022-06-09T11:22:33+09:00"
```

매핑된 클래스 값을 출력하면 아래와 같이 offset값이 Z로 바뀌어 버리는 것이다.

```json
expireAt: 2022-06-09T02:22:33Z
```

### Jackson ObjectMapper 재정의 하면서 해결 시도

Spring Boot에서 json을 serialize/deserialize 할 때에는 ObjectMapper를 사용하는데, 이 ObjectMapper을 세팅하는 다양한 방법 중 `Jackson2ObjectMapperBuilder` 로 재정의하는 방법을 사용했다. 참고로 이 빌더는 class path에서 일부 잘 알려진 module들을 감지하면 알아서 등록 해 준다.

`JavaTimeModule` 에 deserializer를 재정의 하는 방법을 시도했으나, 재정의한 deserializer가 사용되지 않았다. Spring 소스를 한참 디버깅하면서 알아봤으나 원인을 찾지 못했고, 검색으로도 해결책을 찾지 못했다. 기본적인 날짜 타입들을 내부적으로 deserialize하고 있으나 OffsetDateTime 처리 과정에서 Z로 변경하는데 이를 방지하는 방법을 못찾았다. 검색 중 발견한 해결 방법 중 String으로 받아서 파싱해서 LocalDateTime으로 변환하는 방법도 있었다. 하지만 프레임워크에서 제공하는 방법으로 해결하고 싶어서 패스.

```java
@Bean
    public Jackson2ObjectMapperBuilder objectMapperBuilder() {
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addDeserializer(OffsetDateTime.class, new JsonDeserializer<OffsetDateTime>() {
            @Override
            public OffsetDateTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
                return OffsetDateTime.parse(
                        jsonParser.getText()
//                    DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssxxx")
                );
            }
        });

        return new Jackson2ObjectMapperBuilder()
                .defaultViewInclusion(false)
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                .featuresToDisable(SerializationFeature.WRITE_DURATIONS_AS_TIMESTAMPS)
                .failOnUnknownProperties(false)
                .failOnEmptyBeans(true)
                .modules(javaTimeModule)
                ;
    }
```

그 외에 해당 필드에 애너테이션으로 아래와 같은 시도들을 무수히 했지만 성공하지 못했다.

`@JsonFormat`과 `@DateTimeFormat` 애터네이션이 적용되는 방식은 좀더 알아볼 필요가 있다.

참고로 “+09:00” 와 같은 형식의 offset은 xxx(또는 XXX - 0일때 Z로 표현)로 표현한다.

```java
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssxxx")
@DateTimeFormat(pattern = "yyyy-MM-dd'T'HH:mm:ssxxx")
private OffsetDateTime expireAt;
```

마지막 시도로 *`@JsonDeserialize` 애너테이션을 사용해 봤다.*

위에서 module에 deserializer를 재정의 한 것과 같은 코드인데 따로 클래스로 정의하고

```java
public static class CustomOffsetDateTimeDeserializer extends JsonDeserializer<OffsetDateTime> {
        @Override
        public OffsetDateTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext)
                throws IOException {
            return OffsetDateTime.parse(
                    jsonParser.getText()
//                    DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssxxx")
            );
        }
    }
```

해당 필드에 애너테이션으로 적용했다.

```java
@JsonDeserialize(using = JacksonConfig.CustomOffsetDateTimeDeserializer.class)
private OffsetDateTime expireAt;
```

결과는 아래와 같이 제대로 출력되었다.

```json
expireAt: 2022-06-09T02:22:33+09:00
```

이 방법은 애너테이션을 적용한 필드만 적용되기에 프로젝트 전체에서 처리가 필요하다면 module에 세팅하는 첫번째 시도가 성공할 수 있는 방법을 찾아야 한다. 하지만 현재 프로젝트에서는 딱 이 필드만 OffsetDateTime을 사용하므로 우선은 이렇게 적용하고 추후에 방법을 찾아봐야 겠다.

<!--

[https://d2.naver.com/helloworld/0473330](https://d2.naver.com/helloworld/0473330)

[https://www.baeldung.com/spring-boot-customize-jackson-objectmapper](https://www.baeldung.com/spring-boot-customize-jackson-objectmapper)

[https://akageun.github.io/2020/01/02/java-jackson-custom-serialize.html](https://akageun.github.io/2020/01/02/java-jackson-custom-serialize.html)

[https://stackoverflow.com/questions/53232600/java-8-exception-com-fasterxml-jackson-datatype-jsr310-deser-instantdeserialize](https://stackoverflow.com/questions/53232600/java-8-exception-com-fasterxml-jackson-datatype-jsr310-deser-instantdeserialize)

[https://stackoverflow.com/questions/54536198/java-8-datetimeformatter-rejects-correct-iso-8601-date-time-with-offset](https://stackoverflow.com/questions/54536198/java-8-datetimeformatter-rejects-correct-iso-8601-date-time-with-offset)

[https://stackoverflow.com/questions/54010217/offsetdatetime-tostring-return-different-format-date-string](https://stackoverflow.com/questions/54010217/offsetdatetime-tostring-return-different-format-date-string)

[https://stackoverflow.com/questions/46263773/jackson-parse-custom-offset-date-time](https://stackoverflow.com/questions/46263773/jackson-parse-custom-offset-date-time)

[https://howtodoinjava.com/java/date-time/zoneddatetime-parse/](https://howtodoinjava.com/java/date-time/zoneddatetime-parse/)

[https://stackoverflow.com/questions/59838777/parsing-iso-date-string-into-zonedatetime-with-resttemplate-in-spring](https://stackoverflow.com/questions/59838777/parsing-iso-date-string-into-zonedatetime-with-resttemplate-in-spring)

[https://juneyr.dev/2018-12-27/java8-datetime](https://juneyr.dev/2018-12-27/java8-datetime)

-->