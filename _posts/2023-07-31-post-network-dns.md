---
title: "[Network] DNS(Domain Name System)란?"
excerpt: ""

categories:
  - Network
tags:
  - DNS
  - Domain
last_modified_at: 2023-07-31T00:00:00
---


# 1. DNS 정의

인터넷 상의 컴퓨터가 통신하기 위한 IP주소를 숫자로 기억할 필요 없이,  www.google.com과 같은 도메인을 입력하면 IP주소로 변환 해 주는 시스템.

클라우드 서비스 중 AWS에서는 Route 53, GCP와 Alibaba에서는 Cloud DNS라는 이름으로 관련 서비스 제공.

### FQDN(Fully Qualified Domain Name) 이란?

www.google.com 주소를 예를 들면, 아래와 같이 도메인(google.com)과 호스트(www)로 구분 할 수있는데, 이 두가지를 모두 표기하는 방식을 말함.

<br>

<br>

# 2. DNS Resolving(재귀적 질의)

## 2.1. DNS Name server 종류

- Root DNS Server
    - ICANN이라는 비영리 단체의 최상위 네임서버.
    - root-server.net이라는 도메인에 a~m까지의 호스트를 가지는 네임서버가 세계에 흩어져 있고 각각 수십개의 사이트를 가짐.
    - TLD(Top-Level Domain) 주소를 저장/반환 함.
- Top-Level Domain(TLD) DNS Server
    - 도메인 등록기관이 관리하는 서버.
    - .kr/.us 같은 국가코드 최상위 도메인,  .com/.net/.org 와 같은 일반 최상위 도메인들을 저장/반환 함.
- Second-Level Domain(SLD) DNS Server
    - 도메인 이름을 등록한 도메인/호스팅 업체의 네임서버.(e.g. AWS Route 53 등)
    - 권한 있는(**Authoritative)** DNS 서버 라고도 함.
    - 실제 도메인과 IP주소의 관계가 정의됨.

## 2.2. 질의 절차

![network_dns1]({{ '/assets/images/network_dns1.png' | relative_url }}){: .align-center}

                                        출처 : AWS Route 53 document

각 단계의 캐시가 없다고 가정하고,

- client는 DNS Resolver에게 재귀적 질의.
- DNS Resolver가 위임받아 순차적(Iterative)으로 계층화된 Name Server들에게 재귀적 질의.
    - Root 네임서버에게 TLD 네임서버 주소를 질의.
    - 응답 받은 TLD 네임서버에게 최종 권한있는 네임서버(=SLD그림에서 Route 53) 주소 질의.
    - SLD 네임서버로부터 최종적으로 수신한 IP 주소 반환.
- IP 주소로 client에서 요청 시작.

<br>

<br>

# 3. DNS Record Type

- SOA(권한의 시작점) : 도메인 네임서버, 관리자 메일, 시리얼 번호, 타이머 등의 도메인에 관한 정보 정의.
- NM(네임서버 레코드) : 권한 있는 이름 서버들을 지정하고 DNS영역을 위임.
- A(주소 레코드) : FQDN을 IP에 매핑.
- CNAME(별칭 레코드) : 다른 도메인의 별칭을 등록.

### References

- [https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns/](https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns/)
- [https://aws.amazon.com/ko/route53/what-is-dns/](https://aws.amazon.com/ko/route53/what-is-dns/)
- [https://velog.io/@minj9_6/DNS와-작동원리](https://velog.io/@minj9_6/DNS%EC%99%80-%EC%9E%91%EB%8F%99%EC%9B%90%EB%A6%AC)
- [https://hanamon.kr/dns란-도메인-네임-시스템-개념부터-작동-방식까지/](https://hanamon.kr/dns%EB%9E%80-%EB%8F%84%EB%A9%94%EC%9D%B8-%EB%84%A4%EC%9E%84-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B0%9C%EB%85%90%EB%B6%80%ED%84%B0-%EC%9E%91%EB%8F%99-%EB%B0%A9%EC%8B%9D%EA%B9%8C%EC%A7%80/)