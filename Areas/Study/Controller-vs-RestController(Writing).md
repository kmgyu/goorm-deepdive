Spring Boot는 MVC 아키텍처를 위해 Spring-Boot-MVC라는 모듈을 사용한다.
(해당 모듈은 Spring Boot Starter Web에 종속성으로 연결됨. Spring Initializr에서 확인할 수 있다.)

MVC는 Model, View, Controller로 이루어져있고, 스프링은 해당 구조에서 `컨트롤러`를 크게는 `@Controller`와, `@Service` 형태로 분리해서 사용한다. 이 외에도 생략된 부분이 많지만 라우팅하는 것과, 비즈니스 로직만 생각했을 때 다음과 같이 분리할 수 있겠다.

백엔드 개발을 하면서 `@Controller`보다는 `@RestController`를 사용한 경험이 많을 것이다. 그렇다면 그 둘에는 어떤 것이 차이가 있는가? `@RestController`를 많이 쓰니 확실하게 개선된 것인가? 에 대해서, 알아보도록 하자.


!!! alert !!!
번역체 수정 필요


# @Controller

클래식 컨트롤러를 사용하기 위해 사용하는 어노테이션이다.
자동으로 구현체를 탐지해주는 `@Component` 어노테이션의 특화된 클래스라고 할 수 있다.
보통 `@RequestMapping` 어노테이션과 조합하여 리퀘스트 핸들링 메서드를 만든다.

원문
> We can annotate classic controllers with the _@Controller_ annotation. This is simply a specialization of the _@Component_ class, which allows us to auto-detect implementation classes through the classpath scanning.
> We typically use _@Controller_ in combination with a _@RequestMapping_ annotation for request handling methods.

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

> 가장 간단하게는 JSON으로 반환해준다고 생각하면 된다...

# @RestController
`@RestController`는 컨트롤러가 더 특화된 방향이다.
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

`@RestController`를 이용 시, 모든 리퀘스트 핸들링 메서드에서는 HttpReponse안에 자동으로 직렬화된 객체를 반환하기 때문에 `@ResponseBody` 어노테이션을 요구하지 않는다. 즉, 코드가 더 간결해졌다.

# 심화

## Controller로 View 반환하기
## Controller로 Data 반환하기



# 더 생각해볼 수 있는 것들

- `@ResponseBody`는 JSON으로만 직렬화 가능한가?
	- SOAP를 고려할 수도 있지 않을까? XML 직렬화는 안되는가?
- @RequestMapping이라는 어노테이션도 존재한다.
	- RequestBody에서 가져오는 역직렬화도 존재하는데, 어떤 방식으로 동작할까?
- @ResponseBody 비명시시 String으로 뷰에서 리소스를 찾는 것이 일반적인 작동법임. 이때 작동법도 이해시킬 필요가 있다...!



The Spring @Controller and @RestController Annotations
https://www.baeldung.com/spring-controller-vs-restcontroller

[Spring] @Controller와 @RestController 차이
- 그림자료 있음. 내부 구조를 이해하기 좋으므로 해당 자료 인용 필요
https://mangkyu.tistory.com/49

@Controller와 @RestController의 차이점
https://dncjf64.tistory.com/288