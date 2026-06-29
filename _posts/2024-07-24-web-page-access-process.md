---
title: "웹페이지 접속 과정"
date: 2024-07-24 06:59:38 +0900
categories: [네트워크]
tags: [네트워크, DNS, 로드밸런싱, Spring, CS]
description: "URL 입력부터 DNS 조회, TCP/IP 연결, HTTP 요청·응답까지 웹페이지 접속 과정을 심플·디테일 버전으로 정리하고, 로드밸런싱과 스프링 MVC 처리 흐름까지 짚었다."
---

### \[ 목차 ]
- 웹 페이지 접속 과정
  - 심플 버전
  - 디테일 버전
- 로드밸런서
- 웹 페이지 접속 과정 - 스프링

    
<br/>
<br/>

### ㅇ 웹 페이지 접속 과정 1 - 심플 버전
- 사용자가 URL(도메인)을 웹 브라우저에 입력
- 웹 브라우저는 입력한 URL을 바탕으로 DNS(Domain Name System)서버에 연결할 IP를 요청
- DNS서버는 IP주소를 웹 브라우저에게 응답으로 제공
- 웹 브라우저는 DNS서버에서 받은 IP를 통해 웹 서버와 TCP/IP 연결을 통해 세션 수립
- HTTP 요청을 보내고, 웹 서버는 받은 HTTP 요청에 응답. 응답은 웹 페이지와 필요한 리소스를 포함
- 웹 브라우저는 받은 응답을 바탕으로 사용자에게 웹 페이지를 보여줌
![](/assets/img/posts/web-page-access-process/01.jpeg)

<br/>
<br/>

### ㅇ 웹 페이지 접속 과정 2 - 디테일 버전
- 사용자가 URL(도메인)을 웹 브라우저에 입력
  - 브라우저의 URL 파싱 : 주소창에 입력된 문자열이 URL 형식을 띄고 있는지 확인 (아니라면 검색 기능 작동)
  - 브라우저 캐시 확인 : 브라우저 캐시에 저장된 데이터가 있다면, 서버에 접속하지 않고 캐시에서 데이터를 사용
  - HSTS 조회 : HTTP 요청은 HTTPS 요청으로 변환 (HTTP Strict Transport Security) 
- 웹 브라우저는 입력한 URL을 바탕으로 DNS(Domain Name System)서버에 연결할 IP를 요청
  - 로컬 DNS 캐시 확인 : DNS 정보가 로컬 DNS 캐시에 이미 존재하고, 캐시의 TTL ( Time To Live (DNS 레코드의 캐시 유효 시간)) 값이 유효하다면, DNS 서버에 요청을 보내지 않고, 로컬 DNS 캐시에서 해당 정보를 가져옴
  - 연쇄적인 DNS 서버 조회 : Root DNS 서버 -> TLD(Top-Level Domain) DNS 서버 -> Authoritative DNS 서버를 순차적으로 조회하여 IP 주소 반환
![](/assets/img/posts/web-page-access-process/02.gif)
- DNS서버는 IP주소를 웹 브라우저에게 응답으로 제공
- 웹 브라우저는 DNS서버에서 받은 IP를 통해 웹 서버와 TCP/IP 연결을 하고 세션 수립
  - IP 주소와 라우터를 이용한 웹 서버 추적 : 
    - 라우팅 테이블 점검 : DNS 서버에서 획득한 IP주소가 라우팅 테이블에 존재한다면 이를 사용하여 웹 서버를 추적, 존재하지 않는다면 라우팅 알고리즘을 통해 해당 서버의 게이트웨이까지 이동
    - ARP (Address Resolution Protocol)를 통한 MAC 주소 획득 : ARP 를 통해 목적지와 실질적으로 통신을 하기위해 ARP Table 내에서 IP 주소를 MAC 주소와 매칭
  - TCP/IP 연결 : -way handshake 과정(SYN, SYN-ACK, ACK)을 통해 연결을 수립
- HTTP 요청을 보내고, 웹 서버는 받은 HTTP 요청에 응답
  - 응답은 웹 페이지와 필요한 리소스를 포함
  - 웹 서버에서 컨텐츠를 전송할 때는 로드 밸런싱(Elastic Load Balancing, ELB) 기능을 이용해 서버를 분산해 부하를 줄여줌
- 웹 브라우저는 받은 응답을 바탕으로 사용자에게 웹 페이지를 표시
  - 콘텐츠 렌더링 : 응답을 받은 웹 브라우저는 HTML, CSS, JavaScript 등의 웹 자원을 처리하여 웹 페이지를 렌더링하고 사용자에게 표시 (CSR)

<br/>
<br/>


### ㅇ Load Balancing
- 둘 이상의 중앙처리장치/저장장치와 같은 컴퓨터 자원들에게 작업(Work), 즉, 부하(Load)를 나누는 것
  - 자원 사용 최적화
  - 처리량 극대화
  - 응답 시간 줄이기
  - 고장 허용성 보장
- Scale-out
  - 하나의 Server 보다는 여러 대의 Server가 나눠서 일을 하는 방법
  - (vs) Scale-up : Server가 더 빠르게 동작하기 위해 하드웨어 성능을 올리는 방법
  - 하드웨어 향상하는 비용보다 서버 한대 추가 비용이 더 적음
  - 여러 대의 Server 덕분에 무중단 서비스를 제공 가능
  - **여러 대의 Server에게 균등하게 Traffic을 분산시켜주는 역할을 하는 것이 Load Balancer**
- 주요 기능
  - 트래픽 분산 : 트래픽을 분산시켜 각 서버의 부하를 균일하게 관리
  - 오토 스케일링 : 조건에 맞춰 필요시 server에 컴퓨터 수를 늘리거나 줄여서 부하 관리 가능 (Scale out, Scale in)
  - Health Check : 해당 포트에 트래픽을 보내 애플리케이션이 올바르게 작동하는지 여부를 판별
  - 보안 서비스: 웹 애플리케이션 방화벽(WAF)과 네트워크 주소 변환(NAT) 같은 보안 기능을 통해 외부 공격으로부터 내부 네트워크를 보호
- 종류
  - L2 : Mac주소를 바탕으로 Load Balancing
  - L3
    - 네트워크 계층 (OSI 7계층의 제 3계층)
    - IP 주소를 바탕으로 Load Balancing
    - 이 과정에서 NAT(Network Address Translation)가 사용될 수 있음
    - 네트워크 계층의 로드 밸런싱은 라우팅 결정을 기반으로 작동하며, 패킷의 전달 경로를 조정하는 역할을 함
  - L4
    - 전송 계층 (OSI 7계층의 제 4계층)
    - Transport Layer(IP와 TCP/UDP Port) Level에서 Load Balancing 
    - TCP 연결 또는 UDP 흐름을 감지하고, 이를 기반으로 서버들 사이에 트래픽을 분산
    - 전송 계층의 로드 밸런싱은 세션 기반의 결정을 포함하며, 클라이언트와 서버 사이의 연결을 유지하는 데 중요한 역할을 함
  - L7 : Application Layer(사용자의 Request) Level에서 Load Balancing (HTTP, HTTPS, FTP)
  - 트래픽 분산이 목적이라면 네트워크 계층에서의 로드 밸런싱이 효과적일 수 있으며, 세션 유지가 중요한 애플리케이션에서는 전송 계층에서의 로드 밸런싱이 더 적합할 수 있음
- 로드밸런싱 알고리즘
  - 라운드 로빈 (Round Robin) : 각 서버에 순차적으로 요청을 분배하는 가장 단순한 형태의 로드 밸런싱 방법
  - 최소 연결 (Least Connections) : 연결 개수가 가장 적은 서버를 선택. 트래픽으로 인해 세션이 길어지는 경우 효과적
  -소스 (Source) : 사용자의 IP를 Hashing하여 분배. 사용자가 항상 같은 서버로 연결되는 것을 보장

<br/>
<br/>

### ㅇ 스프링 관점에서의 웹 페이지 접속 과정
(스프링부트, 스프링MVC 사용)
- 사용자가 URL(도메인)을 웹 브라우저에 입력
- 브라우저의 요청
  - 브라우저는 입력한 URL을 DNS를 통해 IP 주소로 변환하고, 웹 서버와 TCP/IP 연결을 수립합
  - HTTP 요청을 서버에 보냄
- **스프링 서버의 요청 처리**
  - 요청 수신
    - 스프링 부트 애플리케이션이 HTTP 요청을 수신합니다. 스프링 부트는 내장 서버(예: Tomcat, Jetty, Undertow)를 사용하여 요청을 처리합니다.
  - 디스패처 서블릿(DispatcherServlet)
    - 스프링 부트는 DispatcherServlet이라는 중앙 서블릿을 통해 모든 HTTP 요청을 처리합니다. 이 서블릿은 요청을 적절한 컨트롤러로 전달합니다.
  - 컨트롤러 매핑
    - DispatcherServlet은 요청의 URL 패턴에 맞는 컨트롤러를 찾기 위해 요청 매핑(Handler Mapping)을 사용합니다. 요청 URL과 일치하는 컨트롤러의 메서드를 찾습니다.
  - 컨트롤러 메서드 호출
    - 요청이 일치하는 컨트롤러의 메서드가 호출됩니다. 이 메서드는 비즈니스 로직을 처리하고, 필요한 데이터를 모델에 담아 뷰에 전달합니다.
  - 모델과 뷰 반환
    - 컨트롤러는 데이터(모델)와 뷰 이름을 반환합니다. 뷰 이름은 보통 HTML 템플릿의 파일 이름을 의미합니다.
  - 뷰 리졸버(View Resolver)
    - DispatcherServlet은 뷰 리졸버를 통해 컨트롤러가 반환한 뷰 이름을 실제 뷰(예: Thymeleaf, JSP 파일)로 변환합니다. 이 뷰는 최종 HTML 페이지를 생성하는 역할을 합니다.
  - 뷰 렌더링
    - 선택된 뷰 템플릿 엔진(예: Thymeleaf, JSP)이 모델 데이터를 기반으로 최종 HTML 페이지를 렌더링합니다. 렌더링된 HTML 페이지는 HTTP 응답으로 반환됩니다.
  - 반호
- HTTP 응답 전송
  - 렌더링된 HTML 페이지가 HTTP 응답으로 웹 브라우저에 전송됩니다.
- 웹 브라우저에서 페이지 표시
  - 웹 브라우저는 HTTP 응답으로 받은 HTML 페이지를 화면에 표시합니다. 브라우저는 추가적인 리소스(예: CSS, JavaScript, 이미지)를 요청할 수 있으며, 이 리소스들도 동일한 방식으로 서버에서 처리됩니다.
- REST API 사용 시 뷰 반환 처리:
  - 컨트롤러는 보통 뷰 대신 데이터만을 포함한 응답을 반환합니다. 이 경우, 뷰 이름이 비어 있으며(null), 반환되는 데이터 객체는 HTTP 응답 본문으로 직접 변환됩니다.
  - 응답 변환 처리: 데이터는 @ResponseBody 어노테이션이 붙은 컨트롤러 메서드에서 반환될 수 있으며, 스프링의 메시지 컨버터(HttpMessageConverter)를 통해 클라이언트가 요청한 형식(JSON, XML 등)으로 자동 변환되어 응답됩니다.
  - @RestController의 역할: @RestController 어노테이션은 클래스 레벨에서 사용되며, 이는 해당 컨트롤러의 모든 메서드가 @ResponseBody를 암묵적으로 가진다는 것을 의미합니다. 따라서, REST API를 제공하는 컨트롤러에서는 별도의 뷰를 관리하지 않고, 데이터를 직접 HTTP 응답으로 매핑합니다.

<br/>
<br/>

---

#### 출처
- https://medium.com/@2kunhee94/브라우저의-주소창에-url을-입력하면-일어나는-일-97a0837b8bf8
- DNS : https://opentutorials.org/course/3276
- 로드밸런싱 : https://nesoy.github.io/articles/2018-06/Load-Balancer
- 스프링 관점에서의 웹 페이지 접속 과정 : ChatGPT

<br/>

