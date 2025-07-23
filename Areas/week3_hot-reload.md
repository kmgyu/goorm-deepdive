# Hot-Reload?
핫 리로드, 또는 핫 모듈 replacement(HMR)은 개발 도중 애플리케이션 상태를 유지하면서 코드 변경 사항을 즉시 반영하는 기능이다.
즉 앱의 현재 상태를 유지하면서 변경사항을 반영하는 것이다.
그런데 Spring DevTools가 지원하는 기능은 애플리케이션 현재 상태(메모리의 Bean 상태, static 변수, 세션) 등은 유지가 되지 않는다.

그렇다. 사실 스프링이 지원하는 기능은 정확히는 [Automatic Restart](https://docs.spring.io/spring-boot/reference/using/devtools.html#using.devtools.restart)라는 기능인 것이다!!


## 용어 정리
핫-리로드 라는 단어 자체가 굉장히 애매했었다. 따라서 용어정리를 위해 한번 정리를 해주었다...

| 용어                               | 정의                                                | 특징                           | 대표 사용처                         |
| -------------------------------- | ------------------------------------------------- | ---------------------------- | ------------------------------ |
| **Hot Reload**                   | 코드 변경 시 앱을 **재시작 없이**, 현재 상태(메모리)를 유지하며 변경 사항을 적용 | 상태 유지, 즉시 반영                 | React, Flutter, React Native 등 |
| **Automatic Restart**            | 코드 변경 시 애플리케이션을 **자동으로 재시작**하여 반영                 | 상태 유지 ❌, 클래스패스 변경 감지         | Spring Boot DevTools           |
| **HotSwap**                      | JVM에서 실행 중인 클래스의 **메서드 본문만 바이트코드 교체**             | 제한적(메서드 본문만 교체 가능), 상태 유지 가능 | Java Debugger(JPDA)            |
| **Hot Deploy**                   | 단일 서버(WAS)에 애플리케이션을 **중단 없이 재배포**                 | 애플리케이션 컨테이너(WAS)가 담당         | Tomcat, JBoss, WebLogic 등      |
| **HMR (Hot Module Replacement)** | 모듈 단위로 변경된 JS 코드만 **즉시 교체**                       | 브라우저 리프레시 없음, 상태 유지          | Webpack, Vite, React, Vue 등    |
| **Live Reload**                  | 파일 변경 시 **자동으로 브라우저 새로고침**                        | 상태 유지 ❌, 주로 정적 리소스용          | Spring DevTools, BrowserSync   |

## 장점

핫-리로드를 포함한 비슷한 위의 유형들은 모두 개발자 경험(DevX)를 위해, 즉 개발자의 편의성을 위해 만들어졌다.
애플리케이션(또는 시스템)에서 아주 작은 수준의 코드를 추가/변경하고 테스트하는데 서버를 수동으로 재부팅한다고 생각해보자. 프로젝트가 커질 수록 끔찍한 일이다.

Spring boot DevTools의 AutoRestart를 기준으로, 다음과 같은 이점이 있다.
Hot-Reload 등도 비슷하거나 동일한 이점을 제공한다.

- 개발 속도 향상
  시간 절약으로 개발 속도가 빨라진다.
- 즉각적 피드백
  변경 사항을 빠르게 확인하고 테스트할 수 있어 개발 과정에서 즉각적인 피드백을 얻을 수 있다.
- 리소스 절약
  전체 애플리케이션 재시작에 비해 필요한 리소스가 적어 시스템 부담을 줄인다.


> 물론 Hot Deploy는 빠르게 배포하는 것이 목적이기에 조금 다르다. 
> 이쪽은 배포 속도 향상에 가깝다.


## Spring boot : Dev-Tools Automatic Restart
Spring boot DevTools가 지원하는 기능으로, 클래스 경로의 파일들이 변경될 때마다 자동으로 애플리케이션을 재시작 시키는 기능을 한다.

### Automatic Restart의 트리거 조건
어떤 조건에서 재시작이 발생하는가?
DevTools는 컴파일된 `.class` 파일들이 위치한 클래스 경로의 자원이 변경될 때를 감지한다. 즉, IDE가 컴파일해주어야 감지를 할 수 있다는 것이다.

> 이클립스에서는 수정한 파일을 저장하는 것이 트리거가 된다.
> 인텔리제이의 경우, build -> build project가 똑같은 효과를 가진다.
> mvn compile, gradle build 또한 재시작 트리거가 된다.

> DevTools는 재시작 시 ApplicationContext의 종료 훅을 사용하여(!!!) 애플리케이션을 종료한다. 종료 훅을 비활성화한 경우(`SpringApplication.setRegisterShutdownHook(false)`) DevTools가 제대로 작동하지 않는다.

### 그래서 재시작인데 왜 빠른 거임?
Spring Boot에서 제공하는 재시작 기술은 두 개의 클래스 로더를 이용한다.
변하지 않는 정적 파일들(`.jar` 같은 서드파티 유형)은 `base` 클래스로더가 로딩하고, 현재 개발자가 개발하는 부분에서는 `restart` 클래스 로더가 로딩하게 된다.
이런 방식을 통해 `cold starts`(서버 완전 종료 후 재실행)보다 빠르게 실행할 수 있다.

 재 릴리즈된 [restartClassLoader](https://github.com/spring-projects/spring-boot/blob/main/module/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/restart/classloader/RestartClassLoader.java)의 소스코드이다.

이 클래스 로더는 [Restarter](https://github.com/spring-projects/spring-boot/blob/main/module/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/restart/Restarter.java#L47)에서 실제로 사용된다.
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

**클래스 로더란?**
> 자바는 동적 로드, 즉 컴파일 타임이 아니라 런타임에 클래스를 로드하고 링크하는 특징이 있다.
> 이 동적 로드를 담당하는 부분이 JVM의 클래스 로더이다.

![Java JVM의 클래스 로더란? - 클래스 로더의 종류](https://blog.kakaocdn.net/dna/AbVQB/btrsH92x5AR/AAAAAAAAAAAAAAAAAAAAAI6VIkjuy7S5aNuukHSg6OLY9WQSuyQHcX48rX0tdxWO/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1753973999&allow_ip=&allow_referer=&signature=v0X8XXSeiNHGKC7eTwBHQrjIHig%3D "[Java] JVM의 클래스 로더란? - 클래스 로더의 종류")
[*자바 클래스 로더 종류*](https://steady-coding.tistory.com/593)

> RestartClassLoader는 java.net.URLClassLoader를 상속받아 사용한다.
> [*java SE11 docs*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URLClassLoader.html)
> URLClassLoader는 JAR 파일이나 디렉토리 URL을 참조하고 주어진 URL에서 참조되는 다중 릴리즈 JAR 파일의 내용에서 클래스와 리소스 로드를 지원한다. 즉, 클래스 로드를 할 때 주어진 URL(클래스패스)의 자식 디렉토리에 있는 모든 것을 로드한다.
> *우리는 스프링이 컴포넌트 스캔을 할 때 유사한 방식을 사용하는 것을 본 적이 있다... 그 내부에선 이런 일이 벌어진다!* 

# 무중단 배포와의 관계

우선, Hot-Reload를 포함한 기능들은 Production 빌드에서 절대 사용하면 안된다.
디버깅/계측 기반(debugging/instrumentation-based) 핫 코드 바꾸기를 사용하면 필드에 무언가를 할당한 후, 필드 할당이 다른 것으로 가정하는 메서드의 코드를 바꾸거나, 일부 코드가 여러 번 실행되는 결과를 초래할 수 있다. 이러한 불일치는 개발 환경에서 문제가 없으나 운영 환경에서는 위험하다.
이런 기능들은 개발을 위해서만 존재하는 기능이다.

Spring이 제공하는 기능은 더 견고할 수 있지만, 사용 중인 라이브러리가 올바르게 중지되지 않은 작업을 시작한 경우 또는 이와 유사한 경우 불일치가 발생할 수 있다.

## Hot-Deploy는 비슷하지만 다르다
Hot Deploy는 단일 서버 내에서 서버를 재시작하지 않고 애플리케이션을 갱신하는 방식으로, 무중단 배포에 가까운 개념이지만 로드밸런싱이나 다중 서버 기반의 무중단 배포(ZDD)와는 범위가 다르다.

무중단 배포의 경우 롤링 배포, 블루-그린 배포, 카나리 배포 등의 방법 등이 소개되는데, 이들 모두 2개 이상의 서버에 대한 방법이다. (실제 프로덕션에서도 서버를 여러 개 사용할 수 있으니 이쪽이 더 현실적인 방법이라 할 수 있다.)



https://stackoverflow.com/questions/78539280/hot-reload-deploy-in-java-and-spring-boot/78540111#78540111
https://gseok.github.io/tech-talk-2022/2022-01-24-what-is-HMR/
https://devwithpug.github.io/spring/about-hot-swap-in-spring-boot/#hotswap