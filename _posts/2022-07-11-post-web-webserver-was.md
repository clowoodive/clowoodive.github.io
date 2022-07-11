---
title: "[Web] Web Server와 WAS"
excerpt: ""

categories:
  - Web
  - WAS
tags:
  - Web
  - WAS
last_modified_at: 2022-07-11T00:00:00
---


하단 References에 링크된 글을 보며 이해한 내용을 정리하는 글이므로 자세한 내용은 링크글 참조 하세요.

### Static Pages

클라이언트(웹 브라우저 또는 웹 크롤러)의 파일 경로 요청에 대한 응답으로 저장된 file contents(image, html, css, javascrip 등)를 반환하는 것으로, 동일한 경로 요청에 대해 항상 동일한 페이지를 반환함.

### Dynamic Pages

클라이언트의 가변 인자 요청에 대한 응답을 웹 서버에 의해 실행되는 프로그램(Servlet)을 통해 동적인 contents를 반환함.

### Web Server

- Static Pages를 제공.
- 클라이언트 요청을 Dynamic Pages 를 제공하는 WAS에 전달하고 결과를 받아 클라이언트에 응답.
- ex) Apache Server, Nginx, IIS 등

### WAS(Web Application Server)

- DB 사용 및 다양한 로직 처리를 통해 Dynamic Pages 제공.
- Web Container, Servlet Container로 불림.
- JSP, Servlet 구동 환경을 제공.
- Web Server의 기능도 제공하며 현재는 성능상 큰 차이가 없음. 즉 WAS = Web Server + Web Container.
- ex) Tomcat, JBoss, Jeus, Web Sphere 등

### Web Server와 WAS를 분리해서 구성하는 이유

- Static/Dynamic 컨텐츠를 분리해서 제공함으로써 반응성 향상.
- Web Server를 통해 SSL에 대한 암호화/복호화 처리.
- fail over와 fail back에 유리하고 무중단 운영에 유리함.
- 동적 처리를 담당하는 WAS를 로드밸런싱을 통해 여러대 구축 가능.
- 자원의 효율성, 장애 극복, 배포 및 유지보수 편의성을 위해.

### References

- [https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html](https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html)

<!--

아래 링크의 하단 관련된Post 참고할것

[https://gmlwjd9405.github.io/2018/10/29/web-application-structure.html](https://gmlwjd9405.github.io/2018/10/29/web-application-structure.html)

[https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html](https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html)

[https://webos-supporters.tistory.com/13](https://webos-supporters.tistory.com/13)

-->