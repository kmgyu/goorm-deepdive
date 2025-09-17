# Hot-Reload?
핫 리로드, 또는 핫 모듈 replacement(HMR)은 개발 도중 애플리케이션 상태를 유지하면서 코드 변경 사항을 즉시 반영하는 기능이다.
즉 앱의 현재 상태를 유지하면서 변경사항을 반영하는 것이다.
그런데 Spring DevTools가 지원하는 기능은 애플리케이션 현재 상태(메모리의 Bean 상태, static 변수, 세션) 등은 유지가 되지 않는다.

사실 스프링이 지원하는 기능은 정확히는 [Automatic Restart](https://docs.spring.io/spring-boot/reference/using/devtools.html#using.devtools.restart)라는 명칭이 있다...!
그런데 핫-리로드로도 많이 쓰인다.

## 비슷한 용어

| 용어                               | 정의                                                | 특징                           | 대표 사용처                         |
| -------------------------------- | ------------------------------------------------- | ---------------------------- | ------------------------------ |
| **Hot Reload**                   | 코드 변경 시 앱을 **재시작 없이**, 현재 상태(메모리)를 유지하며 변경 사항을 적용 | 상태 유지, 즉시 반영                 | React, Flutter, React Native 등 |
| **Automatic Restart**            | 코드 변경 시 애플리케이션을 **자동으로 재시작**하여 반영                 | 상태 유지 ❌, 클래스패스 변경 감지         | Spring Boot DevTools           |
| **HotSwap**                      | JVM에서 실행 중인 클래스의 **메서드 본문만 바이트코드 교체**             | 제한적(메서드 본문만 교체 가능), 상태 유지 가능 | Java Debugger(JPDA)            |
| **Hot Deploy**                   | 단일 서버(WAS)에 애플리케이션을 **중단 없이 재배포**                 | 애플리케이션 컨테이너(WAS)가 담당         | Tomcat, JBoss, WebLogic 등      |
| **HMR (Hot Module Replacement)** | 모듈 단위로 변경된 JS 코드만 **즉시 교체**                       | 브라우저 리프레시 없음, 상태 유지          | Webpack, Vite, React, Vue 등    |
| **Live Reload**                  | 파일 변경 시 **자동으로 브라우저 새로고침**                        | 상태 유지 ❌, 주로 정적 리소스용          | Spring DevTools, BrowserSync   |

> Hot Deploy 설명 추가
> 톰캣이 유지되면서 어플리케이션 언로드 과정을 통해 app context와 관련 클래스 로더를 종료한다. 연결 풀 해제 등을 포함.
> 따라서 톰캣은 유지되면서 애플리케이션만 교체하는 방식.

## 장점

핫-리로드를 포함한 비슷한 위의 유형들은 모두 개발자 경험(DevX)를 위해, 즉 개발자의 편의성을 위해 만들어졌다.<br>
애플리케이션(또는 시스템)에서 아주 작은 수준의 코드를 추가/변경하고 테스트하는데 서버를 수동으로 재부팅한다고 생각해보자. 프로젝트가 커질 수록 끔찍한 일이다.

Spring boot DevTools의 AutoRestart를 기준으로, 다음과 같은 이점이 있다.<br>
Hot-Reload 등도 비슷하거나 동일한 이점을 제공한다.

- 개발 속도 향상<br>
  시간 절약으로 개발 속도가 빨라진다.
- 즉각적 피드백<br>
  변경 사항을 빠르게 확인하고 테스트할 수 있어 개발 과정에서 즉각적인 피드백을 얻을 수 있다.
- 리소스 절약<br>
  전체 애플리케이션 재시작에 비해 필요한 리소스가 적어 시스템 부담을 줄인다.


> 물론 Hot Deploy는 빠르게 배포하는 것이 목적이기에 조금 다르다. <br>
> 이쪽은 배포 속도 향상에 가깝다.


## Spring boot : Dev-Tools Automatic Restart
Spring boot DevTools가 지원하는 기능으로, 클래스 경로의 파일들이 변경될 때마다 자동으로 애플리케이션을 재시작 시키는 기능을 한다.

### 적용 방법
```gradle
developmentOnly 'org.springframework.boot:spring-boot-devtools'
```
build.gradle에 의존성을 추가해주기만 하면 자동으로 적용된다.

### Automatic Restart의 트리거 조건
어떤 조건에서 재시작이 발생하는가?<br>
DevTools는 컴파일된 `.class` 파일들이 위치한 클래스 경로(classpath)의 자원이 변경될 때를 감지한다. 즉, IDE가 컴파일해주어야 감지를 할 수 있다는 것이다. (파일 저장 시 자동으로 컴파일 되도록 설정되어 저장하는 순간 보통 재시작이 트리거된다.)

> 이클립스에서는 수정한 파일을 저장하는 것이 트리거가 된다.<br>
> 인텔리제이의 경우, build -> build project가 똑같은 효과를 가진다.<br>
> mvn compile, gradle build 또한 재시작 트리거가 된다.

> DevTools는 재시작 시 ApplicationContext의 종료 훅을 사용하여 애플리케이션을 종료한다.<br> 종료 훅을 비활성화한 경우(`SpringApplication.setRegisterShutdownHook(false)`) DevTools가 제대로 작동하지 않는다.

### 그래서 재시작인데 왜 빠른 거임?
Spring Boot에서 제공하는 재시작 기술은 두 개의 클래스 로더를 이용한다.
변하지 않는 정적 파일들(`.jar` 같은 서드파티 유형)은 `base` 클래스로더가 로딩하고, 현재 개발자가 개발하는 부분에서는 `restart` 클래스 로더가 로딩하게 된다.
이런 방식을 통해 `cold starts`(서버 완전 종료 후 재실행)보다 빠르게 실행할 수 있다.

**클래스 로더란?**
> 자바는 동적 로드, 즉 컴파일 타임이 아니라 런타임에 클래스를 로드하고 링크하는 특징이 있다.
> 이 동적 로드를 담당하는 부분이 JVM의 클래스 로더이다.

![[spring_hot-reload_classloader.png]]
[*자바 클래스 로더 종류*](https://steady-coding.tistory.com/593)
> System Class loader는 JVM의 표준적인 명칭이며, `CLASSPATH`에 있는 모든 클래스를 로드하는 역할
> 대부분의 문맥에서 Application Class loader와 동일하게 간주함
> 스프링 코어에서도 기본 클래스 로더(System Class Loader)를 사용한다.


![[spring_restartclassloader_inheritnace.png]]
> DevTools 사용 시 RestartClassLoader가 적용된다.

[restartClassLoader 소스코드](https://github.com/spring-projects/spring-boot/blob/main/module/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/restart/classloader/RestartClassLoader.java)

이 클래스 로더가 사용되는 것을 [Restarter](https://github.com/spring-projects/spring-boot/blob/main/module/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/restart/Restarter.java#L47)에서 볼 수 있다.
```java
	private Throwable doStart() throws Exception {
		Assert.state(this.mainClassName != null, "Unable to find the main class to restart");
		URL[] urls = this.urls.toArray(new URL[0]);
		ClassLoaderFiles updatedFiles = new ClassLoaderFiles(this.classLoaderFiles);
		ClassLoader classLoader = new RestartClassLoader(this.applicationClassLoader, urls, updatedFiles);
		if (this.logger.isDebugEnabled()) {
			this.logger.debug("Starting application " + this.mainClassName + " with URLs " + Arrays.asList(urls));
		}
		return relaunch(classLoader);
	}
```
*274-283line*
classLoader를 통해 클래스를 로드한다.


**동작 방식 알아보기**

![[jvm_how-to-work-classloader.png]]
[동작 방식](https://goodgid.github.io/Java-Class-Loader/)
클래스 로더는 로딩 - 링크 - 초기화 단계를 통해 클래스 로딩이 이루어진다.

> RestartClassLoader는 java.net.URLClassLoader를 상속받아 사용한다.<br>
> [*java SE11 docs*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URLClassLoader.html)<br>
> URLClassLoader는 JAR 파일이나 디렉토리 URL을 참조하고 주어진 URL에서 참조되는 다중 릴리즈 JAR 파일의 내용에서 클래스와 리소스 로드를 지원한다. 즉, 클래스 로드를 할 때 주어진 URL(클래스패스)의 자식 디렉토리에 있는 모든 것을 로드한다.<br>
> *스프링이 컴포넌트 스캔을 할 때 작동하는 방식을 생각해보자. 기본 클래스 패스(혹은 명시된 경로) 하위의 모든 패키지를 탐색하고, 해당 위치에 있는 클래스를 로드한다. 이 동작을 떠올려보면 restart class loader가 하는 일을 추론할 수 있다.*

# 무중단 배포와의 관계

우선, Hot-Reload를 포함한 기능들은 Production 빌드에서 절대 사용하면 안된다.
디버깅/계측 기반(debugging/instrumentation-based) 핫 코드 바꾸기를 사용하면 필드에 무언가를 할당한 후, 필드 할당이 다른 것으로 가정하는 메서드의 코드를 바꾸거나, 일부 코드가 여러 번 실행되는 결과를 초래할 수 있다. 이러한 불일치는 개발 환경에서 문제가 없으나 운영 환경에서는 위험하다.
이런 기능들은 개발을 위해서만 존재하는 기능이다.

Spring이 제공하는 기능은 더 견고할 수 있지만, 사용 중인 라이브러리가 올바르게 중지되지 않은 작업을 시작한 경우 또는 이와 유사한 경우 불일치가 발생할 수 있다.

## Hot-Deploy는 비슷하지만 다르다
Hot Deploy는 단일 서버 내에서 서버를 재시작하지 않고 애플리케이션을 갱신하는 방식으로, 무중단 배포에 가까운 개념이지만 로드밸런싱이나 다중 서버 기반의 무중단 배포(ZDD)와는 범위가 다르다.

무중단 배포의 경우 롤링 배포, 블루-그린 배포, 카나리 배포 등의 방법 등이 소개되는데, 이들 모두 2개 이상의 서버에 대한 방법이다. (실제 프로덕션에서도 서버를 여러 개 사용할 수 있으니 이쪽이 더 현실적인 방법이라 할 수 있다.)

+ HotDeploy는 IDE나 JVM의 기능이 아닌 애플리케이션 컨테이너의 기능이므로, HotSwap과는 전혀 다른 기능이다.





## Spring은 HotSwap도 지원한다.
Spring은 기능이 아주 많아서 Auto Restart 대신 HotSwap을 사용할 수도 있다.

### [HotSwap](https://docs.spring.io/spring-boot/how-to/hotswapping.html)이란?
 JPDA(Java Platform Debugger Architecture)의 기능이며 JDK 1.4부터 JPDA Enhancements를 통해 지원 되는 기능으로 디버거가 동일한 클래스 ID에 대해 빌드로 생성되는 클래스 바이트 코드를 제자리에서 업데이트할 수 있는 기능이다. 

즉, 모든 객체는 업데이트된 클래스를 참조하고 메소드가 호출될 때 변경된 코드를 실행할 수 있으므로 클래스(바이트 코드)가 변경될 때마다 IDE 컨테이너를 다시 로드할 필요가 없는 것이다.

JVM 수준에서 동작하기 때문에 매우 저수준에 있기도 하다.

### 바뀌는 범위
Hot Swapping은 바뀌는 범위가 매우 한정적이다.<br>
필드 추가, 메서드 추가, 클래스 명칭 및 계층 변경 등이 불가능하다.<br>
오직 정적 리소스 및 메서드의 내부의 수정만 적용된다.

#### automatic restart보다 좋은 점
스프링에서 추천하는 방법은 DevTools의 auto restart지만, 정적 리소스의 변경사항을 추적하려면 빌드가 필수적이라는 점이 존재한다.<br>
따라서 JVM이 재시작하게 되는데 이럴 필요없도록 Hot Swapping을 이용한다.

#### 사용법 in IntelliJ
대부분의 현대적인 IDE는 정적 리소스 및 자바 클래스 변경에 대한 hot-swapping을 지원한다.

인텔리제이에서는 다음과 같이 설정할 수 있다.

> DevTools Dependency를 적용해야 한다.
> 위로 올라가면 build.gradle에 적용하는 스크립트를 확인할 수 있다.


1. `Run/Debug Configurations` - `edit`<br>
![image1](Pasted%20image%2020250724163535.png)

2. `옵션 수정` 클릭<br>
![image2](Pasted%20image%2020250724163954.png)

3. `클래스를 핫스왑하고 실패할 경우 트리거 파일을 업데이트합니다.` 클릭<br>
![image3](Pasted%20image%2020250724164034.png)

이제 디버깅 시 다음과 같이 핫 스왑을 적용할 수 있다.
(저장 시 바로 적용은 되지 않고, 해당 버튼을 클릭해야 적용된다.)
![image4](Pasted%20image%2020250724170150.png)

> 주의: 설정 -  빌드, 실행, 배포 - 컴파일러에서 프로젝트 자동 빌드 옵션이 켜져있어야 한다.

## Live Reload는 또 무엇인가?
정적 리소스만 업로드한다.<br>
생각해보면 우리의 서버는 정적 리소스는 건드리지 않는 선에서 동적 소스만 바꾸는 방식을 택했다. 그럼 정적인 건 변경될 때 어떻게 관리하는 걸까?<br>
그리고 정적인 파일을 서버 종료도 안하고 어떻게 바로 로드하는 걸까? <br>
나이브한 방식으로 그때그때 정적파일을 URL을 통해 로드하는 것도 아닐 텐데 말이다.

### Live Reload란?
열려있는 브라우저에서 HTML, CSS와 같은 정적 리소스의 변경 사항을 즉시 반영하는 것이다.
HotSwap과 목표는 동일하지만 JVM 과는 관련이 없는 기능이다.

메이븐, 그래들 플러그인을 설정해서 소스에서 정적 파일을 바로 로드할 수 있다. 상위 수준 도구를 사용하여 해당 코드를 작성하는 경우 외부 CSS/JS 컴파일러 프로세스와 함께 사용할 수 있다. 

#### 사용법
속성값 설정

`application.properties`의 경우
```
spring.devtools.livereload.enabled=true  
spring.devtools.restart.enabled=true  
spring.thymeleaf.cache=false
```

`application.yml`의 경우
```
spring:
	devtools:
		livereload:
			enabled: true
		restart:
			enabled: true
	thymeleaf
		cache: false
```


추가 설정
> 설정 - 빌드, 실행, 배포 - 빌드 도구 - 빌드 스크립트 변경 후 프로젝트 동기화 : 모든 변경 내용<br>
> 설정 - 고급 설정 - 컴파일러 - 개발된 애플리케이션이 현재 실행 중인 경우에도 auto-make가 시작되도록 허용


# referenced
[Hot reload/deploy in Java and Spring Boot](https://stackoverflow.com/questions/78539280/hot-reload-deploy-in-java-and-spring-boot)<br>
[HMR 이해하기](https://gseok.github.io/tech-talk-2022/2022-01-24-what-is-HMR/)<br>
[# 스프링 부트 - HotSwap? HotDeploy? HotReload? 에 대해](https://devwithpug.github.io/spring/about-hot-swap-in-spring-boot/#hotswap)

