---
title:  "의존성 주입(DI)"
excerpt: "Dependency Injection의 장점과 방법"

categories:
  - MySQL
tags:
  - Database
  - MySQL
last_modified_at: 2021-11-29T15:06:00-05:00
---

MySQL SELECT 쿼리 최적화에는 여러가지 방법들이 있지만  
인덱스의 적용 여부가 체감상 가장 크지 않았나 싶다.  
이 글에 정리한 간단한 인덱스 적용보다 더 고차원의 방안이 필요하다면  
공홈의 Optimization 문서 전체를 꼼꼼히 뒤져 볼 필요가 있다.  
주의 할 점은 같은 인덱스 세팅이어도 테스트 데이터 구성 및 환경 세팅에 따라  
Optimizer가 실행계획을 달리 하므로 실제와 유사하게 잘 세팅해야 한다.  
(MySQL 5.7 버전 innodb 기준임을 참고).