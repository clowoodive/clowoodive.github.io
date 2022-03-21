---
title: "GCP에서 글로벌 서비스를 위한 리전 선택"
excerpt: "빠른 글로벌 서비스를 위한 Region 선택"

categories:
  - GCP
tags:
last_modified_at: 2022-01-04T08:06:00-05:00
---


글로벌 서비스를 시작하려고 준비 하면서 첫번째 고민이 되는 것은 "GCP의 어떤 리전(Region)에 서비스들을 위치시켜야 할까?" 이다.  
일본 도쿄 region만 해도 반응이 느린것이 체감 되는 액션도 있는데...  
검색을 해봤지만 특정 국가를 타게팅 하지 못하는 이상 가이드 문서 대로 미국이 최선의 선택인것 같고, 그나마 유저가 많을 것으로 예상되는 아시아와 가까운 미국 서부의 리전 중 하나를 선택하기로 우선은 정했다.  

웹서버는 GCP의 Premium Tier 네트워크 서비스를 이용하면 Google Network를 사용하므로 리전에서 먼 나라의 클라이언트 응답속도를 줄여줄 수 있을것 같은데, TCP는 잘 알려진 포트만 가능한 것으로 봐서 불가능 할 것 같다.  
FPS 게임처럼 고반응성을 요구하는 것은 아니니 우선은 이렇게 서비스 해보기로!

* __참고__
    - https://cloud.google.com/network-tiers/docs/overview
    + https://cloud.google.com/solutions/best-practices-compute-engine-region-selection
