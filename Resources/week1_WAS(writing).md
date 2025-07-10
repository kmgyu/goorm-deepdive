Web Server, Web Application Server 등의 차이점을 설명하라고 하면 정적이냐, 동적이냐 등의 특징을 설명할 수 있지만 WAS의 유형과 네트워크 관련 개념을 엮어 설명하려고 하면 잘 안되는 경우가 많다.

따라서 이 글에서는 WAS가 무엇인지 좀 더 자세히 살펴볼 것이다. CGI나 WSGI, ASGI, 서블렛 등도 같이 넣고 싶은데... 그렇게 되면 시리즈가 될 것 같다.

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


~~이후 내용부터는 제대로 정리되지 않았음
~~
https://ko.wikipedia.org/wiki/%EC%9B%B9_%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98_%EC%84%9C%EB%B2%84

https://velog.io/@yoojinjangjang/%EA%B5%AC%EB%8B%88%EC%BD%98Gunicorn%EC%9D%B4%EB%9E%80

https://linuxism.ustd.ip.or.kr/1006#:~:text=Java%20EE%20%ED%91%9C%EC%A4%80%EC%A4%80%EC%88%98%20%EC%9B%B9%20%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%20%EC%84%9C%EB%B2%84%EB%8A%94%20Java,%EC%9B%B9%20%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%20%EC%84%9C%EB%B2%84%EC%9D%B4%EB%8B%A4.%5B%ED%8E%B8%EC%A7%91%5D%EA%B5%AC%EC%84%B1%EC%9A%94%EC%86%8CJava%20EE%20%ED%91%9C%EC%A4%80%EA%B8%B0%EB%B0%98%20%EC%9B%B9%20%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%97%90%EC%84%9C

https://blog.naver.com/hhm731/221271943043

https://docs.openmaru.io/docs/jboss-eap/Java_Understanding_of_WebSystem/#:~:text=%EC%9D%B4%EB%B2%88%20%EC%9E%A5%EC%97%90%EC%84%9C%EB%8A%94%20Java%20%EA%B8%B0%EB%B0%98%EC%9D%98%20%EC%9B%B9%20%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%20%EC%84%9C%EB%B2%84%EC%97%90,EE%EC%9D%98%20%EC%A3%BC%EC%9A%94%20%EA%B8%B0%EC%88%A0%EB%93%A4%EA%B3%BC%20Java%20%EA%B8%B0%EB%B0%98%EC%9D%98%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%ED%86%B5%ED%95%A9

https://rainbow97.tistory.com/entry/JAVA-Java-EE#:~:text=WAS%EC%99%80%20%EC%84%9C%EB%B8%94%EB%A6%BF%20%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%20%C2%B7%20WAS%EB%9E%80%20%EC%9B%B9%20%EA%B8%B0%EB%B0%98,Java%20EE%20%EA%B8%B0%EC%88%A0%20%EC%82%AC%EC%96%91%EC%9D%84%20%EC%A4%80%EC%88%98%ED%95%B4%EC%84%9C%20%EB%A7%8C%EB%93%A0%20%EC%84%9C%EB%B2%84%EC%9D%B4%EB%8B%A4.

https://velog.io/@yoojinjangjang/%EA%B5%AC%EB%8B%88%EC%BD%98Gunicorn%EC%9D%B4%EB%9E%80

https://bombo96.tistory.com/65#Apache%20VS%20NGINX%C2%A0-1
## 분류
크게 자바 EE 표준준수 WAS와, 자바 기반이나, 자바 EE를 비준수하는 WAS, 기타로 나뉜다.

- **WebLogic Server:**
    오라클에서 제공하는 상용 WAS로, 강력한 기능과 안정성을 제공합니다. 
    
- **WebSphere Application Server:**
    IBM에서 제공하는 상용 WAS로, 다양한 플랫폼과 환경에서 사용 가능합니다. 
    
- **JEUS:
    티맥스소프트에서 개발한 WAS로, 국내 환경에서 널리 사용됩니다. [티스토리](https://uyfuyfuy-042.tistory.com/entry/WEBWAS-%EB%9E%80) 
    
- **GlassFish:
    오라클에서 개발하고 현재는 Eclipse 재단에서 관리하는 오픈 소스 WAS입니다. 
    
- **TomEE:**
    Apache Tomcat을 기반으로 하여 Jakarta EE 기능을 추가한 오픈 소스 WAS입니다. 
    
- **WildFly (JBoss EAP):
    레드햇에서 제공하는 오픈 소스 WAS로, 엔터프라이즈 환경에서 사용됩니다. 
    

Java EE 표준 준수 서버의 특징:

- **Java EE 명세 준수:
    서블릿, JSP, EJB, JPA, JMS 등 Java EE 명세에서 정의된 기술들을 지원합니다. 
    
- **엔터프라이즈급 기능:
    트랜잭션 관리, 보안, 분산 처리, 클러스터링 등 대규모 시스템 구축에 필요한 기능을 제공합니다. 
    
- **다양한 플랫폼 지원:
    윈도우, 리눅스, 유닉스 등 다양한 운영체제에서 동작합니다. 
    
- **확장성 및 유연성:
    필요에 따라 기능을 추가하거나 확장할 수 있습니다. 
    
- **상용 및 오픈 소스 선택 가능:
    다양한 요구사항에 맞춰 상용 또는 오픈 소스 서버를 선택할 수 있습니다. 
    

참고:
- Java EE는 현재 Jakarta EE로 명칭이 변경되었습니다. 
- WAS는 웹 애플리케이션 서버의 약자로, 웹 기반 애플리케이션을 실행하고 관리하는 역할을 합니다

