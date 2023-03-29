---
title: "[Spring MVC] Spring Boot에서 @RestControllerAdvice 이용한 Error Handling 하기"
excerpt: ""

categories:
  - Spring
tags:
  - Spring Boot
  - MVC
  - Error
  - @ControllerAdvice
last_modified_at: 2023-03-28T00:00:00
---

{% capture notice-env %}
#### Environment
 - Java 11
 - Spring Boot 2.5.12
 - Gradle 7.5.1 
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


Spring Boot는 servlet container에 `/error` 매핑으로 글로벌 오류 페이지를 등록 해 준다. 

이 기능을 통해 머신 클라이언트(@ResponseBody 에 의해 JSON과 같은 응답을 받는 클라이언트)는 HTTP 상태코드, 예외 메세지 등이 포함된 JSON 응답을 받게되고, 브라우저 클라이언트는 같은 정보를 HTML 형식의 `whitelabel` 오류 페이지로 받게 된다.

기본 오류 동작은 `server.error`로 시작하는 [속성](https://docs.spring.io/spring-boot/docs/2.5.14/reference/htmlsingle/#appendix.application-properties.server)들로 조정 할 수 있으며, 커스텀 error page를 등록하는 방법은 여기에서 다루지 않으니 하단 링크를 참고하길.

오류의 커스텀 JSON 응답은 `@ControllerAdvice` 와 `@ResponseBody` 메타 애노테이션을 포함하는 `@RestControllerAdvice` 를 클래스에 선언해 적용 할 수 있다.  이 클래스 멤버 메소드 중 `@ExceptionHandler` 가 선언되어 있는 메소드는 자동 빈 스캔을 통해 등록된다.

```java
@RestControllerAdvice
public class InvestingExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(InvestingException.class)
    protected ResponseEntity<InvestingException.ResultBody> handleInvestingException(InvestingException iex) {

        System.out.println("ResultCode : " + iex.resultCode + ", ResultMessage : " + iex.resultMessage);

        InvestingException.ResultBody resultBody = new InvestingException.ResultBody(iex.resultCode.getCode(), iex.resultMessage);

        return new ResponseEntity<>(resultBody, HttpStatus.FORBIDDEN);
    }

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<InvestingException.ResultBody> handleRuntimeException(Exception ex) {

        System.out.println("class : " + ex.getClass().getName() + ", message : " + ex.getMessage());

        InvestingException.ResultBody resultBody = new InvestingException.ResultBody(ResultCode.InternalServerError.getCode(), null);

        return new ResponseEntity<>(resultBody, HttpStatus.FORBIDDEN);
    }
}
```

```java
public class InvestingException extends RuntimeException {
    public final ResultCode resultCode;
    public final String resultMessage;

    public InvestingException(ResultCode resultCode) {
        this.resultCode = resultCode;
        this.resultMessage = null;
    }

    public InvestingException(ResultCode resultCode, String message) {
        this.resultCode = resultCode;
        this.resultMessage = message;
    }

    public static class ResultBody {
        @JsonProperty("result_code")
        public final int resultCode;
        @JsonProperty("result_message")
        public final String resultMessage;

        public ResultBody(int resultCode, String message) {
            this.resultCode = resultCode;
            this.resultMessage = message;
        }
    }
}
```

위 예제는 비즈니스 로직에서 예상되는 예외를 담은 `InvestingException.class`와 `RuntimeException.class` 예외를 잡아서 HTTP 상태 코드는 항상 403을 응답하게 하고, 실제 오류코드나 오류 메시지는 가공해서 body에 담아 전달하고 있다. 

서버에서 대응을 위해 `System.out.println` 코드 위치에 실제로 사용중인 로깅 프레임워크를 통해 로그를 남기게 구현해서 실 서비스에서도 사용중이다.

`@ExceptionHandler` 메소드를 컨트롤러 내에도 정의하거나, `@ControllerAdvice` 가 다중 선언 될 경우 우선순위 고려, ResponseBody의 오류 정보로 전역 예외 처리를 하게 도와주는 `@ResponseEntityExceptionHandler` 등의 추가적인 내용은 [스프링 문서](https://docs.spring.io/spring-framework/docs/5.3.18/reference/html/web.html#mvc-ann-exceptionhandler)를 참고 하길.

해당 스프링 문서 하단에는 `@ExceptionHandler` 가 선언된 메소드에서 사용 가능한 인자와 리턴 타입의 설명도 있으니 매우 유용하다.

예제 코드가 적용된 프로젝트는 [여기](https://github.com/clowoodive/toy/tree/main/investing).

### References

- [https://docs.spring.io/spring-framework/docs/5.3.18/reference/html/web.html#mvc-ann-exceptionhandler](https://docs.spring.io/spring-framework/docs/5.3.18/reference/html/web.html#mvc-ann-exceptionhandler)
- [https://docs.spring.io/spring-boot/docs/2.5.12/reference/htmlsingle/#features.developing-web-applications.spring-mvc.error-handling](https://docs.spring.io/spring-boot/docs/2.5.12/reference/htmlsingle/#features.developing-web-applications.spring-mvc.error-handling)