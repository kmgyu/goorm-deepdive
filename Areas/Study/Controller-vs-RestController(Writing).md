Spring Boot는 MVC 아키텍처를 위해 Spring-Boot-MVC라는 모듈을 사용한다.
(해당 모듈은 Spring Boot Starter Web에 종속성으로 연결됨. Spring Initializr에서 확인할 수 있다.)

MVC는 Model, View, Controller로 이루어져있고, 스프링은 어노테이션 기반 MVC를 통해 `@Controller`로 컨트롤러를 정의한다. 이때 `@Controller`보다는 `@RestController`를 사용한 경험이 많을 것이다. 그렇다면 그 둘에는 어떤 것이 차이가 있는가에 대해서 알아보도록 하자.

> ? : MVC 모델에서 비즈니스 로직은 어디에 위치하는가?
> 갑작스러운 궁금증으로 찾던 도중 [적합한 자료(구글 그룹 대화)](https://groups.google.com/g/ksug/c/RAMmQAV_P9I)를 발견함.
> 원칙적으로는 MVC 기술 유형이 비즈니스 로직 구성 방식에 전혀 영향을 미치면 안된다.
> 단순한 애플리케이션일 경우 컨트롤러에 바로 비즈니스 로직을 구성할 수 있으나, 서비스나 도메인 객체 또는, 비즈니스 객체로 로직을 옮기는 것이 좋다.

# @Controller

전통적인 Spring MVC 컨트롤러를 사용하기 위해 사용하는 어노테이션이다.
자동으로 구현체를 탐지해주는 `@Component` 어노테이션 중 특화된 클래스라고 할 수 있다.
보통 `@RequestMapping` 어노테이션과 조합하여 리퀘스트 핸들링 메서드를 만들 때 사용한다.

간단한 예제를 확인하자.

```java
@Controller
@RequestMapping("books")
public class SimpleBookController {

    @GetMapping("/{id}", produces = "application/json")
    public @ResponseBody Book getBook(@PathVariable int id) {
        return findBookById(id);
    }

    private Book findBookById(int id) {
        // ...
    }
}
```

`@ResponseBody`를 이용해 리퀘스트 핸들링 메서드를 정의하였는데, 이 어노테이션은 자동으로 반환 객체의 HttpResponse에 대한 직렬화를 제공한다.

> @ResponseBody를 사용하는 경우 일반적으로는 JSON이 많이 쓰이나, XML도 가능하다.
> BSON (Binary JSON)은 추가적인 세팅이 필요하다.
> [stack overflow - how send bson via spring @responsebody](https://stackoverflow.com/questions/18525095/how-send-bson-via-spring-response-body)
> [BSON? - velog](https://velog.io/@chayezo/MongoDB-JSON-vs.-BSON)

**XML로 변환하기 예제**
[참고 소스코드](https://github.com/kmgyu/MokJoon/tree/test/example-xml)

XML로 반환하려면 직렬화하는 몇가지 추가적인 코드가 필요하다.

build.gradle
```gradle
implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml'
```

DTO
```java
@Builder  
@Getter  
@AllArgsConstructor  
@NoArgsConstructor  
// localName으로 설정한 값을 루트 요소로 지정한다.  
// 미지정시 클래스명이 지정됨.  
@JacksonXmlRootElement(localName = "BaekJoonUser")  
public class XMLUser {  
//  루트 요소 안에 들어갈 하위 요소를 지정함.  
//    isAtrribute = true 지정시, 루트 태그의 속성으로 들어감.  
//    localName 속성 지정시, 지정한 값으로 요소가 표기됨.  
    @JacksonXmlProperty(isAttribute = true) private long id;  
    @JacksonXmlProperty(localName = "userId") private String username;  
}
```

Controller
```java
@RequestMapping("/scrap")  
@RequiredArgsConstructor  
@RestController  
public class ScrapController {  
	...
  
    @GetMapping(value = "/xml", produces = MediaType.APPLICATION_XML_VALUE)  
    public XMLUser getUser() {  
        return XMLUser.builder()  
                .id(1L)  
                .username("hong-kill-dong")  
                .build();  
    }  
  
}
```

반환 결과 (localhost:8080/scrap/xml)
![](controllerVSrestcontroller.png)
[추가 참고 자료 - 티스토리](https://binit.tistory.com/28)


## 작동 흐름 이해하기
---

### Case1 : `@Controller`로 View 반환하기

`@Contorller` 자체는 주로 View를 반환하기 위해 사용한다.
메서드의 반환 타입이 String이라면, 프로젝트의 지정된 경로 내에서 일치하는 파일 이름을 찾아 렌더링하고, 사용자에게 반환하는 과정을 거친다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FwqQav%2FbtstffuvCYG%2FAAAAAAAAAAAAAAAAAAAAABs5fzNgHti-sw1XfqtMXKn8MH6qeZNsfHyq3_lcIDLN%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3DPJ5zkecQKa6myvBH4PfPLE0YqPA%253D)
1. Client는 URI 형식으로 웹 서비스에 요청을 보낸다.
2. DispatcherServlet이 요청을 처리할 대상을 찾는다.
	1. HandlerMapping을 통해 요청을 처리할 Controller를 찾는다.
3. HandlerAdapter을 통해 요청을 Controller로 위임한다.
4. Controller는 요청을 처리한 후에 ViewName을 반환한다.
5. DispatcherServlet은 ViewResolver를 통해 ViewName에 해당하는 View를 찾아 사용자에게 반환한다.

### Case2 : Controller로 Data 반환하기

`@ResponseBody`를 활용하는 케이스이다. 앞의 예제에서 보여진 바 있다.

![](https://blog.kakaocdn.net/dna/b2WNwu/btstaBd0DBg/AAAAAAAAAAAAAAAAAAAAALbFTn3n8QQ8DYaiozriHcbNiYHzyQUQ9VKTVJaEbsGx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=gP30rfHSo5bqn6fbx%2FotIgRt7yI%3D)

1. Client는 URI 형식으로 웹 서비스에 요청을 보낸다.
2. DispatcherServlet이 요청을 처리할 대상을 찾는다.
3. HandlerAdapter을 통해 요청을 Controller로 위임한다.
4. Controller는 요청을 처리한 후에 객체를 반환한다.
5. 반환되는 객체는 Json으로 Serialize되어 사용자에게 반환된다.

컨트롤러를 통해 객체를 반환할 시에는 일반적으로 ResponseEntity로 감싸서 반환한다. 그리고 객체를 반환하기 위해 `viewResolver` 대신 `HttpMessageConverter`가 동작한다.
`HttpMessageConverter`에는 여러 `Converter`가 등록되어 있고, 반환해야 하는 데이터에 따라 사용되는 Converter가 달라진다.
단순 문자열인 경우, `StringHttpMessageConverter`가 사용되고, 객체인 경우 `MappingJackson2HttpMessageConverter`가 사용되며, 데이터 종류에 따라 서로 다른 `MessageConverter`가 작동하게 된다.
`Spring`은 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해 적합한 `HttpMessageConverter`를 선택하여 이를 처리한다. MessageConverter가 동작하는 시점은 HandlerAdapter와 Controller가 요청을 주고받는 시점이다.

![](controllerVSrestcontroller2.png)

> 참고
> 문자열을 반환할 시, 위와 같이 html 형태로 반환한다.
> Jackson은 java의 json 직렬화 라이브러리이다.

> 참고하기 좋은 자료
> [`@ResponseBody` - Spring 공식 문서](https://docs.spring.io/spring-framework/reference/web/webflux/controller/ann-methods/responsebody.html)

Jackson JSON&XML 프로퍼티 커스터마이징도 가능하다고 한다.
> [Jackson2ObjectMapperBuilder spring-javadoc](https://docs.spring.io/spring-framework/docs/6.2.9/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html)


# @RestController
`@RestController`는 `@ResponseBody`조차 표기하지 않기 위해 만들어졌다.
`@Controller`와 `@ResponseBody` 어노테이션을 포함하며, 그 결과로 컨트롤러 구현이 더 간단해진다.

```java
@RestController
@RequestMapping("books-rest")
public class SimpleBookRestController {
    
    @GetMapping("/{id}", produces = "application/json")
    public Book getBook(@PathVariable int id) {
        return findBookById(id);
    }

    private Book findBookById(int id) {
        // ...
    }
}
```

`@RestController`를 이용 시, 모든 리퀘스트 핸들링 메서드에서는 HttpReponse안에 자동으로 직렬화된 객체를 반환하기 때문에 `@ResponseBody` 어노테이션을 요구하지 않는다. 즉, 코드가 더 간결해진다.


## 내부구조

![](https://blog.kakaocdn.net/dna/bozaJn/btsteSlYKCB/AAAAAAAAAAAAAAAAAAAAAFZoxtMKfCJerqO7Gfrl26jeuwuY7bACuEfRad1yfUPl/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=m30hiatg%2FvFpZH5Zpcl8jPxA4fQ%3D)

1. Client는 URI 형식으로 웹 서비스에 요청을 보낸다.
2. DispatcherServlet이 요청을 처리할 대상을 찾는다.
3. HandlerAdapter을 통해 요청을 Controller로 위임한다.
4. Controller는 요청을 처리한 후에 객체를 반환한다.
5. 반환되는 객체는 Json으로 Serialize되어 사용자에게 반환된다.

`@Controller`가 `@ResponseBody`어노테이션을 사용했을 때와 차이가 없다.


# Dispatcher Servlet의 내부 동작 살펴보기

Method Dispatch란 어떤 메소드를 호출할 것인가를 결정하고 실행하는 과정을 뜻한다. 컴파일 시점, 런타임 시점인지에 따라 정적/동적으로 나뉘는데, 여기서 말하는 dispatch는 동적 dispatch이다. (더블 dispatch도 존재한다고 함.)

Dispatcher Servlet의 dispatch, 즉 요청을 받고 반환하는 과정은 doDispatch 메서드를 통해 이루어진다.

> 주석을 통해 #1, #2 등으로 단계를 나누어주었음.
> 그림에 나오지 않은 코드의 경우에도  이해를 위해해 주석으로 중간중간 표기해 주었음.

```java
  
/**  
 * Process the actual dispatching to the handler. * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.  
 * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters * to find the first that supports the handler class. * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers  
 * themselves to decide which methods are acceptable. * @param request current HTTP request  
 * @param response current HTTP response  
 * @throws Exception in case of any kind of processing failure  
 */@SuppressWarnings("deprecation")  
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {  
    HttpServletRequest processedRequest = request;  
    HandlerExecutionChain mappedHandler = null;  
    boolean multipartRequestParsed = false;  
  
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);  
  
    try {  
       ModelAndView mv = null;  
       Exception dispatchException = null;  
  
       try {  
          processedRequest = checkMultipart(request);  
          multipartRequestParsed = (processedRequest != request);  

		// #1 핸들러 매핑
          // Determine handler for the current request.  
          mappedHandler = getHandler(processedRequest);  
          if (mappedHandler == null) {  
             noHandlerFound(processedRequest, response);  
             return;  
          }  

		// #2 매핑된 핸들러와 어댑터 연결
          // Determine handler adapter for the current request.  
          HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());  

		// Last-Modified 헤더를 기반으로 조건부 GET에 대한 캐시처리 수행. 새로고침이나 재요청 시 리소스가 변경되지 않은 경우, 304 Not Modified 응답을 내보낸다.
		// ServletWebRequest를 새로 생성할 시, checkNotModified에서 Response에 대해 setStatus까지 시켜주기 때문에 출력 스트림 종료 조건까지 처리된다고 함... 따라서 return;를 하더라도 상관이 없다.
          // Process last-modified header, if supported by the handler.  
          String method = request.getMethod();  
          boolean isGet = HttpMethod.GET.matches(method);  
          if (isGet || HttpMethod.HEAD.matches(method)) {  
             long lastModified = ha.getLastModified(request, mappedHandler.getHandler());  
             if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {  
                return;  
             }  
          }  

		// 컨트롤러의 핸들러 메소드 실행 전에 Interceptor를 호출해 요청을 가로챌 수 있도록 호출한다. 접근 거절 시, 컨트롤러 호출을 멈추고 적용된다.
		// 예시 : 로그인된 사용자 유무에 따른 접근 허용 여부
		// 인터셉터 예시 코드 및 자료
		// https://innovation123.tistory.com/52,
		// https://velog.io/@timssuh/Spring-Boot%EC%97%90%EC%84%9C-%EC%BB%A4%EC%8A%A4%ED%85%80-Interceptor-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0#:~:text=preHandler()%20:%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%20%EB%A9%94%EC%84%9C%EB%93%9C%EA%B0%80%20%EC%8B%A4%ED%96%89%EB%90%98%EA%B8%B0%20%EC%A0%84.%20false%EC%9D%B4%EB%A9%B4,%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%20%EB%A9%94%EC%8B%9C%EC%A7%80%20%EC%8B%A4%ED%96%89%EC%A7%81%ED%9B%84%20View%20%ED%8E%98%EC%9D%B4%EC%A7%80%20%EB%A0%8C%EB%8D%94%EB%A7%81%20%EC%A0%84.
          if (!mappedHandler.applyPreHandle(processedRequest, response)) {  
             return;  
          }  

		// 실제로 핸들러를 호출하는 파트
          // Actually invoke the handler.  
          mv = ha.handle(processedRequest, response, mappedHandler.getHandler());  

		// 비동기 처리가 시작된 경우, 중복 처리를 방지한다.
          if (asyncManager.isConcurrentHandlingStarted()) {  
             return;  
          }  

		// 핸들러 실행 이후, 뷰 렌더링
          applyDefaultViewName(processedRequest, mv);  
          mappedHandler.applyPostHandle(processedRequest, response, mv);  
       }  
       catch (Exception ex) {  
          dispatchException = ex;  
       }  
       catch (Throwable err) {  
       // 실패한 경우에 대한 처리들
          // As of 4.3, we're processing Errors thrown from handler methods as well,  
          // making them available for @ExceptionHandler methods and other scenarios.          dispatchException = new ServletException("Handler dispatch failed: " + err, err);  
       }  
       processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);  
    }  
    catch (Exception ex) {  
       triggerAfterCompletion(processedRequest, response, mappedHandler, ex);  
    }  
    catch (Throwable err) {  
       triggerAfterCompletion(processedRequest, response, mappedHandler,  
             new ServletException("Handler processing failed: " + err, err));  
    }  
    finally {  
       if (asyncManager.isConcurrentHandlingStarted()) {  
          // Instead of postHandle and afterCompletion  
          if (mappedHandler != null) {  
             mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);  
          }  
          asyncManager.setMultipartRequestParsed(multipartRequestParsed);  
       }  
       else {  
          // Clean up any resources used by a multipart request.  
          if (multipartRequestParsed || asyncManager.isMultipartRequestParsed()) {  
             cleanupMultipart(processedRequest);  
          }  
       }  
    }  
}
```



# 추가적인 구상거리

### @ResponseBody? @RequestBody?
`@ResponseBody`는 Handler 매핑을 통해 자동으로 직렬화시켜주는 전략을 선택했다.



The Spring @Controller and @RestController Annotations
https://www.baeldung.com/spring-controller-vs-restcontroller

[Spring] @Controller와 @RestController 차이
- 그림자료 있음. 내부 구조를 이해하기 좋으므로 해당 자료 인용 필요
https://mangkyu.tistory.com/49

@Controller와 @RestController의 차이점
https://dncjf64.tistory.com/288