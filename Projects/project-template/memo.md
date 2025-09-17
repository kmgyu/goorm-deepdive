프로젝트 시작을 더 빠르게 하기 위한 스프링-부트 템플릿 프로젝트

공부도 같이 한다.


일단 오픈소스부터 뒤져본다.
2018년 아카이브된 리포지토리
https://github.com/making/spring-webapp-template

가장 최근 커밋이 2023년인 리포지토리
https://github.com/AlanBinu007/Spring-Boot-Advanced-Projects

티스토리에서 진행한 스프링 프로젝트 템플릿
글 참고하면 좋을 듯.
https://9401ndk.tistory.com/82

계획

1. 보안
OAuth, Jwt, Session 템플릿 별로 생성할지?
일단 OAuth와 Session은 필요하다.
HTTPS 같은 기본적인 보안 스펙도 지켜져야 한다.

2. 기초 CRUD
CRUD 및 welcome url을 rest api로 작성
컨트롤러부터 리포지토리까지 기본적으로 만들어져 있어야 한다...

3. DB
mysql이 기본 빌드, 다른 드라이버는 주석처리

4. 배포 세팅
.bat, Dockerfile, DockerCompose 등 기본적인 배포 세팅
추가적으로 k8s나 k3s 같은 MSA 환경을 위한 세팅으로도 확장 가능...

5. 로깅
있어야 함.


# 프로젝트 목적

보일러 플레이트를 통한 초기 세팅 간소화

어디까지 세팅해주어야 하나?
몰루...

