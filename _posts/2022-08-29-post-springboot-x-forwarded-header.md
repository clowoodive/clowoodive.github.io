---
title: "[Spring Boot] X-Forwarded-* header(헤더에서 client IP 추출)"
excerpt: ""

categories:
  - Spring Boot
tags:
  - Configuration
last_modified_at: 2022-08-29T00:00:00
---

{% capture notice-env %}
#### Environment
- Java 11
- Spring Boot 2.5.14
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


## X-Forwarded-* header

Front-end Proxy(proxy, load-balancer, cloud 등)가 그  뒤에서 수행되는 애플리케이션에게 원래의 정보(host, port, scheme 등)을 전달하기 위한 헤더들.

현재 서비스에서는 내부자만 허용되는 API의 interceptor에서 source IP 를 검사하기 위해 사용.

## 적용 방법

proxy가 `X-Forwarded-For` 나 `X-Forwarded-Proto` 와 같은 보편적인 헤더만 추가한다면 아래 속성 만으로 충분함.

```yaml
server.forward-headers-strategy = NATIVE
```

- NONE : X-Forwarded-* 헤더 무시.
- NATIVE : 컨테이너(tomcat, jetty, netty 등)의 지원 사용하는 것으로 X-Forwarded-[Host/Port/Proto/Ssl] 등 X-Forwarded-Prefix를 제외한 헤더 핸들링.
    - Tomcat에서는 아래의 server.tomcat.remoteip.* 속성으로  `RemoteIpValve` 를 구성
- FRAMEWORK : Spring의 지원을 사용하는 것으로 Spring Boot가 `ForwardedHeaderFilter`
bean을 자동 생성.

Tomcat에서는 추가적으로 헤더이름을 아래처럼 커스터마이징 할 수 있음.

```yaml
server.tomcat.remoteip.remote-ip-header = x-your-remote-ip-header
server.tomcat.remoteip.protocol-header = x-your-protocol-header
```

`server.tomcat.remoteip.remote-ip-header` 에 설정된 헤더에 클라이언트 IP부터 중간에 거치게 되는 proxy들의 IP가 모두 포함되는데 internal-proxies 속성에 등록된  IP들을 필터링 함으로써 클라이언트 IP 추출(`request.getRemoteAddr()`)이 가능함. 비워 둘 경우 신뢰 할 수 있는 기본 IP(127/8, 192.168/16 등) 이 적용되는 것으로 알고 있음.

```yaml
# 빈 값으로 두면 모든 proxy를 신뢰하며, 프로덕션에서는 조심해야 함.
server.tomcat.remoteip.internal-proxies = 192\\.168\\.\\d{1,3}\\.\\d{1,3}
```

그리고 valve 인스턴스 재정의를 통해 Tomcat의 `RemoteIpValve` 를 완전히 컨트롤 할 수 있음.

자세한 내용은 [Running Behind a Front-end Proxy Server](https://docs.spring.io/spring-boot/docs/2.5.12/reference/htmlsingle/#howto.webserver.use-behind-a-proxy-server) 참고.

## 이슈

Spring Boot 버전 업그레이드를 진행 하면서 어느 시점부터 `server.tomcat.internal-proxies` 필터가 적용되지 않고 마지막으로 통과한 LB IP가 코드(`request.getRemoteAddr()`)에서 반환됨.

- 발생 환경
    - `server.forward-headers-strategy` 속성을 사용하지 않던 상태.(default 값 적용)
    - `server.tomcat.internal-proxies` 에 신뢰 할 수 있는 LB 및 Cloud단의 IP들을 등록함으로써 `X-Forwarded-For` 에 담긴 IP들 중에서 실제 클라이언트 IP가 `request.getRemoteAddr()` 의 반환값이 되게 함.
    - 실제 클라이언트 IP를 인터셉터에서 판단해서 허용되지 않은 IP면 요청 거절.
- 구성 속성 확인
    - Spring Boot 2.1까지 `server.use-forward-headers` 속성이 사용되었고, Spring Boot 2.2부터는 `server.forward-headers-strategy` 속성 사용.
    - Spring Boot 2.2까지 `server.tomcat.internal-proxies` 속성이 사용되었고, Spring Boot 2.3부터는 `server.tomcat.remoteip.internal-proxies` 속성 사용.
- 해결
    - `server.tomcat.remoteip.internal-proxies` 로 속성 변경.
    - `server.forward-headers-strategy = NATIVE` 속성으로 정상 동작하는지 확인 및 적용.
    - deprecated된 속성들이 섞여 있어서 NONE/NATIVE/FRAMEWORK 설정에 따른 동작 확인에 어려움을 겪음.
    

## 테스트

server.forward-headers-strategy 속성에 따른 request.getRemoteAddr() 반환값 테스트를 위해 X-Forwarded-For 헤더에 임의로 IP를 추가하여 호출.

- 호출
    - 로컬 환경에서 서버를 구동하고, RestMan으로 호출
    - 헤더 세팅
        - X-Forwarded-For : 1.1.1.1, 127.0.0.1, 2.2.2.2
    - 실제 호출 source IP : 127.0.0.1
- [application.properties](http://application.properties) 세팅

```yaml
logging.level.org.apache.catalina.valves = DEBUG  # RemoteIpValve.class 로그 출력
server.forward-headers-strategy = NONE            # NONE or NATIVE or FRAMEWORK
server.tomcat.remoteip.internal-proxies = 1\\.1\\.1\\.1|127\\.0\\.0\\.1|2\\.2\\.2\\.2
```

### server.forward-headers-strategy = NONE

- internal-proxies 주석처리
    - 마지막 IP(2.2.2.2) 반환
- 모든 ip 필터링(1.1.1.1 | 127.0.0.1 | 2.2.2.2)
    - 첫번째 IP(1.1.1.1) 반환
- 127.0.0.1 제외 필터링(1.1.1.1 | 2.2.2.2)
    - 127.0.0.1 반환 - 정상 동작
- 첫번째 IP(1.1.1.1) 제외 필터링(127.0.0.1 | 2.2.2.2)
    - 첫번째 IP(1.1.1.1) 반환
- 마지막 IP(2.2.2.2) 제외 필터링(1.1.1.1 | 127.0.0.1)
    - 마지막 IP(2.2.2.2) 반환

### server.forward-headers-strategy = NATIVE

- internal-proxies 주석처리
    - 마지막 IP(2.2.2.2) 반환
- 모든 ip 필터링(1.1.1.1 | 127.0.0.1 | 2.2.2.2)
    - 첫번째 IP(1.1.1.1) 반환
- 127.0.0.1 제외 필터링(1.1.1.1 | 2.2.2.2)
    - 127.0.0.1 반환 - 정상 동작
- 첫번째 IP(1.1.1.1) 제외 필터링(127.0.0.1 | 2.2.2.2)
    - 첫번째 IP(1.1.1.1) 반환
- 마지막 IP(2.2.2.2) 제외 필터링(1.1.1.1 | 127.0.0.1)
    - 마지막 IP(2.2.2.2) 반환

### server.forward-headers-strategy = FRAMEWORK

- internal-proxies 주석처리
    - 첫번째 IP(1.1.1.1) 반환
- 모든 ip 필터링(1.1.1.1 | 127.0.0.1 | 2.2.2.2)
    - 첫번째 IP(1.1.1.1) 반환
- 127.0.0.1 제외 필터링(1.1.1.1 | 2.2.2.2)
    - 첫번째 IP(1.1.1.1) 반환 - 비정상
    - Skip RemoteIpValve for request /actuator with originalRemoteAddr '127.0.0.1’
- 첫번째 IP(1.1.1.1) 제외 필터링(127.0.0.1 | 2.2.2.2)
    - 첫번째 IP(1.1.1.1) 반환
- 마지막 IP(2.2.2.2) 제외 필터링(1.1.1.1 | 127.0.0.1)
    - 첫번째 IP(1.1.1.1) 반환

## 테스트 결론

Spring Boot의 Emebedded Tomcat 환경에서  `X-Forwarded-For` 나 `X-Forwarded-Proto` 헤더만 사용하는 수준에서는 FRAMEWORK 옵션이 필요하지 않고 원하는 대로 동작하지 않으므로 NATIVE 사용.

`server.forward-headers-strategy` 속성의 default 값은 Tomcat의 경우 NONE이며 NATIVE와 같게 동작하지만 명시적으로 NATIVE 권장.

Tomcat Access 로그에 `X-Forwarded-For` 를 출력하면 internal-proxies 에 해당하는 IP 들은 제거되고  출력되니 참고.