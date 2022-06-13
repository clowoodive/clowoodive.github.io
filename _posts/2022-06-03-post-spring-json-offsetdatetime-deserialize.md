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

### 추가 시도 1

이 [링크](https://d2.naver.com/helloworld/0473330)에 Module을 사용한 방법과 괜찮은 개념들이 있어서 시도 했으나, RestTemplate에서는 잘 되지 않았다. 추후에 다른 케이스에서 한번 해볼 만한 좋은 내용인 것 같다.

### 추가 시도 2

결국 MessageConverter를 변경하는 방법으로 적용하기로 했다. 

우선 해당 RestTemplate 요청에만 사용할 ObjectMapper를 선언하고, 여기에 CustomOffsetDateTimeDeserializer를 추가한 JavaTimeModule을 등록한다.

```java
public static ObjectMapper objectMapper() {
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addDeserializer(OffsetDateTime.class, new CustomOffsetDateTimeDeserializer());
        return new ObjectMapper()
                .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
                .configure(SerializationFeature.WRITE_DURATIONS_AS_TIMESTAMPS, false)
                .configure(DeserializationFeature.ADJUST_DATES_TO_CONTEXT_TIME_ZONE, false)
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                .configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, true)
                .configure(MapperFeature.DEFAULT_VIEW_INCLUSION, false)
                .registerModules(javaTimeModule);
    }
```

그리고 위 objectMapper를 세팅한 MappingJackson2HttpMessageConverter 를 RestTemplate의 0번째 인덱스에 message converter로 등록한다. 이렇게 할 경우에는 RestTemplate이 가진 converter중에 MappingJackson2HttpMessageConverter 타입이 default까지 포함해서 2개가 되지만 인덱스가 낮은 것을 먼저 사용하기에 제대로 동작한다.

```java
var restTemplate = new RestTemplate();
MappingJackson2HttpMessageConverter messageConverter = new MappingJackson2HttpMessageConverter();
messageConverter.setObjectMapper(JacksonConfig.objectMapper());
restTemplate.getMessageConverters().add(0, messageConverter);

var res = restTemplate.exchange(domain, HttpMethod.POST, new HttpEntity<String>(body, headers), RestTemplateProto.Res.class);
```

하지만 다른 필드들을 컨버팅 하면서 문제가 생기기도 했다. 아래와 같이 이미 String으로 serialize한 것인데 message converter가 다시 변환 해버리는 것이다.

```json
string body : {"req_id":"custom call"}
message converted body : "{\"req_id\":\"custom call\"}"
```

default로 등록된 message converter들의 순서에 영향을 받고 있었고, 내부적으로 등록되는 순서도 중요한 것 같아서 아래와 같이 message converter를 해당 index에 교체하는 방식으로 변경했다.

```java
for(inti = 0; i < restTemplate.getMessageConverters().size(); i++) {
finalHttpMessageConverter<?> httpMessageConverter = restTemplate.getMessageConverters().get(i);
if(httpMessageConverterinstanceofMappingJackson2HttpMessageConverter){
        restTemplate.getMessageConverters().set(i, messageConverter);
    }
}
```

커스터마이징 하지 않은 RestTemplate은 여전히 아래와 같이 응답을 deserialize하면서 Z로 변경되고

```json
body : {"res_id":"default call","res_at":"2022-05-22T11:22:33+08:00"}

resId: default call
resAt: 2022-05-22T03:22:33Z
```

message converter를 교체한 후에는 offsettime값이 제대로 출력되는 것을 확인 할 수 있다.

```json
body : {"res_id":"custom call","res_at":"2022-05-22T11:22:33+08:00"}

resId: custom call
resAt: 2022-05-22T11:22:33+08:00
```

<!--

[https://stackoverflow.com/questions/9381665/how-can-we-configure-the-internal-jackson-mapper-when-using-resttemplate](https://stackoverflow.com/questions/9381665/how-can-we-configure-the-internal-jackson-mapper-when-using-resttemplate)

[https://d2.naver.com/helloworld/0473330](https://d2.naver.com/helloworld/0473330)

[https://www.baeldung.com/spring-boot-customize-jackson-objectmapper](https://www.baeldung.com/spring-boot-customize-jackson-objectmapper)

[https://kwonnam.pe.kr/wiki/springframework/springboot/json](https://kwonnam.pe.kr/wiki/springframework/springboot/json)

[https://akageun.github.io/2020/01/02/java-jackson-custom-serialize.html](https://akageun.github.io/2020/01/02/java-jackson-custom-serialize.html)

[https://stackoverflow.com/questions/53232600/java-8-exception-com-fasterxml-jackson-datatype-jsr310-deser-instantdeserialize](https://stackoverflow.com/questions/53232600/java-8-exception-com-fasterxml-jackson-datatype-jsr310-deser-instantdeserialize)

[https://stackoverflow.com/questions/54536198/java-8-datetimeformatter-rejects-correct-iso-8601-date-time-with-offset](https://stackoverflow.com/questions/54536198/java-8-datetimeformatter-rejects-correct-iso-8601-date-time-with-offset)

[https://stackoverflow.com/questions/54010217/offsetdatetime-tostring-return-different-format-date-string](https://stackoverflow.com/questions/54010217/offsetdatetime-tostring-return-different-format-date-string)

[https://stackoverflow.com/questions/46263773/jackson-parse-custom-offset-date-time](https://stackoverflow.com/questions/46263773/jackson-parse-custom-offset-date-time)

[https://howtodoinjava.com/java/date-time/zoneddatetime-parse/](https://howtodoinjava.com/java/date-time/zoneddatetime-parse/)

[https://stackoverflow.com/questions/59838777/parsing-iso-date-string-into-zonedatetime-with-resttemplate-in-spring](https://stackoverflow.com/questions/59838777/parsing-iso-date-string-into-zonedatetime-with-resttemplate-in-spring)

[https://juneyr.dev/2018-12-27/java8-datetime](https://juneyr.dev/2018-12-27/java8-datetime)

-->