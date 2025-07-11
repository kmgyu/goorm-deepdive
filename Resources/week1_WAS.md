WAS의 내부적인 것을 깊게 파려고 하다보니 네트워킹 지식도 연계되었다.
WAS를 중심으로 살펴보는 자료이다.

# WAS

## 정의
**웹 애플리케이션 서버**(Web Application Server, 약자 **WAS**)는 웹 애플리케이션과 서버  환경을 만들어 동작시키는 기능을 제공하는 소프트웨어 프레임워크이다. 인터넷 상에서 HTTP를 통해 사용자 컴퓨터나 장치에 애플리케이션을 수행해 주는 미들웨어(소프트웨어 엔진)로 볼 수 있다. 웹 애플리케이션 서버는 동적 서버 콘텐츠를 수행하는 것으로 일반적인 웹 서버와 구별이 되며, 주로 데이터베이스 서버와 같이 수행이 된다. 한국에서는 일반적으로 "WAS" 또는 "WAS S/W"로 통칭하고 있으며 공공기관에서는 "웹 응용 서버"로 사용되고, 영어권에서는 "Application Server" (약자 AS)로 불린다.

대부분이 자바 기반으로, 주로 자바 EE 표준을 수용하고 있으나 자바 기반이지만 자바 EE 표준을 따르지 않는 제품과 .NET이나 Citrix 기반인 비 자바 계열도 존재한다.

## 기본 기능
웹 애플리케이션 서버의 기본 기능은 3가지이다.

- 프로그램 실행 환경과 데이터베이스 접속 기능을 제공한다.
- 여러 개의 트랜잭션을 관리한다.
- 업무를 처리하는 비즈니스 로직을 수행한다.

다만, 웹 애플리케이션의 정확한 정의는 존재하지 않아서 일부 기능을 제공하지 않는 웹 애플리케이션 서버도 존재한다. 업체들은 이러한 3가지 기능 말고도 여러 기능을 추가하고 강화하고 있다.

## 등장 배경
1990년대 초반 클라이언트/서버 시스템에서는 클라이언트에 화면을 구성하는 각종 기능을 제공하는 Thick 클라이언트가 대세였음. 이는 RDBMS를 포함한 서버 비용이 높았기 때문이고, 화면 프로그램을 교체하는 경우가 있었으나 이 경우 사용자가 주로 인트라넷이었기 때문에 큰 어려움이 없었음.

그러나 1990년대 후반 인터넷 보급과 전자 상거래 요구가 생겨나며 사용자가 불특정 다수로 늘어나고, 시스템이 변경될 때마다 클라이언트를 교체하는 건 사실상 불가능해지는 상황이 발생함.
또한 인터넷 기술 발전, 서버 고성능화 등의 요인으로 인해 어플리케이션의 위치가 클라이언트에서 서버로 바뀌게 됨.

## 분류
크게 자바 EE 표준준수(J2EE, 현재 명칭은 jakarta EE) WAS와, 자바 기반이나, 자바 EE를 비준수하는 WAS, 기타로 나뉜다.

**J2EE?**
> 자바의 기본적인 기능을 정의한 자바 SE에 웹 서버 역할을 추가한 것으로 자바 애플리케이션을 동작시킬 수 있는 컨테이너 등을 표준화한 스펙이다.
> Jakarta EE 인증 WAS 또는 표준준수 WAS는 Jakarta EE 스펙을 수용하는 WAS이다.

특징:
- Java로 구현된 기술이기에, Java가 갖는 특징인 플랫폼 독립성을 갖춤.
- 매우 방대한 범위를 다루는 스펙 집합. 이 자체는 WAS가 아니다.
  대표적으로 서블릿, JSP, EJB, RMI, JNDI, JDBC, JCA, JMS 등이 있음.

### J2EE 인증 WAS
- WebLogic Server:
    오라클에서 제공하는 상용 WAS로, 강력한 기능과 안정성 제공. 
- WebSphere Application Server:
    IBM에서 제공하는 상용 WAS로, 다양한 플랫폼과 환경에서 사용 가능. 
- JEUS:
    티맥스소프트에서 개발한 WAS로, 국내 환경에서 널리 사용됨.
- GlassFish:
    오라클에서 개발하고 현재는 Eclipse 재단에서 관리하는 오픈 소스 WAS. 
- TomEE:
    Apache Tomcat을 기반으로 하여 Jakarta EE 기능을 추가한 오픈 소스 WAS. 
- WildFly (JBoss Application Server):
    레드햇에서 제공하는 오픈 소스 WAS로, 엔터프라이즈 환경에서 사용됨. 

> J2EE는 방대한 스펙 탓에 기능이 복잡하고 어려워 POJO(Plain Old Java Object) 라는 용어가 생기기도 하였다. 이는 스프링 프레임워크 탄생으로 이어진다.
> 현재는 Oracle로 라이센스가 넘어가면서 Jakarta EE로 바뀌고, 지금도 계속해서 갱신중이다.

### J2EE 비준수 자바 기반 WAS

- Jetty
  경량화, 웹소켓 지원, 모듈성 등의 특징이 있음.
- Undertow
  경량화, 고성능, 유연성 등의 특징이 있음.
- Tomcat
  아파치 소프트웨어 재단에서 개발한 서블릿 컨테이너만 있는 WAS

이 외에도 Grizlly, Netty 등이 있다.
> https://okky.kr/questions/237951
> 이전에는 J2EE 스펙을 모두 준수해야 WAS라고 통칭하며, 그 일부만 지킬 경우는 따로 분류하는 모습이 있었다.
> 현재는 Tomcat도 WAS라고 통칭하는 데, 이를 통해 세대차이(...)를 느낄 수 있다.

### 자바 기반이 아닌 WAS들

- Gunicorn
  django, flask등 다양한 python 기반 프레임 워크와 호환됨.
  루비의 유니콘 프로젝트에서 영감을 받음.
- Node.js
  서버 사이드에서 js를 사용할 수 있게 해줌.
- puma
  ruby를 위한 웹 서버
  속도와 병렬성을 위해 만들어짐

> php는 WAS가 따로 없고 모듈을 통해 동작한다. mod_php, php-fpm 등을 이용한다.

## 통신 방식

### CGI (Common Gateway Interface)
웹 서버와 WAS 간의 데이터 통신 방식, 인터페이스이다.
1993년 등장하였으며 

장점
- 스펙만 준수하면 되기에 언어, 플랫폼 독립적이다.
- 매우 단순하고 다른 server-side 프로그래밍 언어에 비해 advanced task를 훨씬 쉽게 수행할 수 있다.  
- 재사용할 수 있는 CGI 코드 라이브러리가 풍부하다.  
- CGI가 웹서버에서 실행될 때 안전하다.  
- CGI 코드를 수행하는데 특정 라이브러리가 필요하지 않기 때문에 매우 가볍다.

단점
- 요청이 올 때마다 프로세스가 하나씩 실행되기에 느리다.
  스크립트 언어(인터프리터 사용)의 경우 매 코드 실행 시 마다 매번 해석해야 하기에 특히 치명적임.
- HTTP 요청마다 새로운 프로세스를 만들기 때문에 서버 메모리를 많이 소모한다.
- 페이지 로드 사이에 데이터가 메모리에 캐시될 수 없다.

> 해당 단점을 극복하기 위해 자바에서는 Servlet, 파이썬에서는 WSGI, ruby에서는 rack이 등장하게 되었다.


``Referenced and Related``

[WAS, J2EE](https://ko.wikipedia.org/wiki/%EC%9B%B9_%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98_%EC%84%9C%EB%B2%84)

[netty vs jetty vs undertow](https://velog.io/@tjddyd1565/Netty-Jetty-Undertow-%EB%B9%84%EA%B5%90)

[J2EE](https://vaert.tistory.com/182)

[서블릿 컨테이너](https://velog.io/@han_been/%EC%84%9C%EB%B8%94%EB%A6%BF-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88Servlet-Container-%EB%9E%80)

[스프링 탄생](https://suzuworld.tistory.com/87)

[j2ee와 스프링 차이점](https://choichumji.tistory.com/133)

[cgi, 서블릿 컨테이너](https://jinbroing.tistory.com/205)

[CGI란](https://live-everyday.tistory.com/197)

[CGI와 ASGI, WSGI](https://velog.io/@jeong-god/WSGI-CGI-ASGI%EB%9E%80)

[구니콘?](https://velog.io/@yoojinjangjang/%EA%B5%AC%EB%8B%88%EC%BD%98Gunicorn%EC%9D%B4%EB%9E%80)

[WAS?](https://blog.naver.com/hhm731/221271943043)

[java 기반 웹 시스템의 이해](https://docs.openmaru.io/docs/jboss-eap/Java_Understanding_of_WebSystem/#:~:text=%EC%9D%B4%EB%B2%88%20%EC%9E%A5%EC%97%90%EC%84%9C%EB%8A%94%20Java%20%EA%B8%B0%EB%B0%98%EC%9D%98%20%EC%9B%B9%20%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%20%EC%84%9C%EB%B2%84%EC%97%90,EE%EC%9D%98%20%EC%A3%BC%EC%9A%94%20%EA%B8%B0%EC%88%A0%EB%93%A4%EA%B3%BC%20Java%20%EA%B8%B0%EB%B0%98%EC%9D%98%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%ED%86%B5%ED%95%A9)

[java EE, java SE, Servlet, Servlet Container](https://rainbow97.tistory.com/entry/JAVA-Java-EE#:~:text=WAS%EC%99%80%20%EC%84%9C%EB%B8%94%EB%A6%BF%20%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%20%C2%B7%20WAS%EB%9E%80%20%EC%9B%B9%20%EA%B8%B0%EB%B0%98,Java%20EE%20%EA%B8%B0%EC%88%A0%20%EC%82%AC%EC%96%91%EC%9D%84%20%EC%A4%80%EC%88%98%ED%95%B4%EC%84%9C%20%EB%A7%8C%EB%93%A0%20%EC%84%9C%EB%B2%84%EC%9D%B4%EB%8B%A4.)

[웹 서버와 WAS의 차이, 그리고 아파치와 NGINX 알아보기](https://bombo96.tistory.com/65#Apache%20VS%20NGINX%C2%A0-1)


