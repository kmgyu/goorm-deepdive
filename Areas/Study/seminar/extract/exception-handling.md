# Exception Handling

## 예외 처리 기본 개념

### 에러 vs Exception

#### 에러 (Error)
- 시스템 레벨에서 발생하는 심각한 문제
- 애플리케이션 코드에서 처리할 수 없는 오류
- OutOfMemoryError, StackOverflowError, VirtualMachineError 등
- catch 문으로 잡는 것은 권장되지 않음
- 대부분 JVM에서 발생
- 프로그램이 계속 실행될 수 없는 치명적인 상황
- 개발자가 직접 throw하거나 처리하는 경우는 거의 없음

#### Exception
- 애플리케이션 코드에서 대부분 처리 가능

### 예외 분류

#### Checked Exception
- 컴파일러가 예외 처리를 강제 (try-catch or throws)
- IOException, SQLException 등

#### Unchecked Exception (RuntimeException)
- 컴파일러가 예외 처리를 강제하지 않음

### 계층 구조
- 계층 구조를 가짐
- Throwable이라는 것을 Error와 Exception이 상속
- RuntimeException과 기타 CheckedException이 Exception을 상속함

## 예외 처리 방법

### try-catch-finally
- 하위 단계에서 상위 단계로 처리해줘야 함, 순서에 따라 터질 수가 있음
- **throw**: 예외를 직접 발생
- **throws**: 예외를 상위로 전달

### catch와 throws의 관계
- Checked Exception과 Runtime Exception 다 나뉨

#### Checked Exception

| 상황 | catch 설정 | throws 설정 | 결과 요약 |
|------|------------|-------------|-----------|
| 1 | X | X | **컴파일 에러 (throws 필요)** |
| 2 | X | O | 예외가 상위(호출한 메서드)로 전달됨 |
| 3 | O (예외처리) | X | catch에서 예외 처리 후, 이후 로직 정상 수행 |
| 4 | O (throw 처리) | X | 컴파일 에러 |
| 5 | O (예외처리) | O | 의미가 없음, throws는 제거하는게 좋음 |
| 6 | O (throw 처리) | O | throws 꼭 필요하며 결과는 상위로 호출 |

#### Runtime Exception

| 상황 | catch 설정 | throws 설정 | 결과 요약 |
|------|------------|-------------|-----------|
| 1 | X | X | 컴파일 에러없음 - 상위로 전달 |
| 2 | X | O | throws 설정하지 않아도 됨<br>예외가 상위(호출한 쪽) 자동 전달됨 |
| 3 | O (예외처리) | X | catch에서 예외 처리 후, 이후 로직 정상 수행 |
| 4 | O (throw 처리) | X | 상위로 전달 |
| 5 | O (예외처리) | O | 의미가 없음, throws는 제거하는게 좋음 |
| 6 | O (throw 처리) | O | throws설정 안해도 됨 |

## 커스텀 예외 처리

### 서블릿 기반 예외처리

### 스프링부트 예외처리

#### BasicErrorController
- 스프링 프레임워크에도 있는 오류 처리에 대한 기본 컨트롤러
- 기본적인 오류 응답을 자동 처리해줌
- HTTP 500이나 404같은 거 기본으로 보내주는 친구
- 뷰만 만들어주면 자동적으로 처리해주는 친구

#### 커스텀을 위해 사용되는 스프링 어노테이션
- ExceptionHandler, ControllerAdvice 등

## HandlerExceptionResolver

### 정의
- 예외 발생 시 어떻게 처리할지 결정하는 확장 포인트

### 구현체는 자동으로 등록되어 있음
- `ExceptionHandlerExceptionResolver` (`@ExceptionHandler` 지원)
- `ResponseStatusExceptionResolver` (`@ResponseStatus` 지원)
- `DefaultHandlerExceptionResolver` (기본 예외 처리)
- 이 순서대로 발동하는 함정카드 같은 거

### 동작 방식
- 핸들러에서 예외 발생하면 프론트 컨트롤러가 ExceptionResolver에 던짐
- 문제 해결 시도하는데, ModelAndView 렌더링도 완료해줌
- 404나 500 페이지 띄우는 거 때문에 그런듯

### 각 Resolver의 역할

#### ExceptionHandlerExceptionResolver
- @ExceptionHandler가 붙은 메서드를 가장 먼저 처리

#### ResponseStatusExceptionResolver
- @ResponseStatus 붙었거나 ResponseStatusException일 경우 설정된 상태코드, 메시지로 응답

#### DefaultHandlerExceptionResolver
- 처리하지 못한 오류들을 스프링 표준으로 응답해줌
- HttpRequestMethodNotSupportedException은 405
- MissingServletRequestParameterException은 400 등
- 예외 종류에 따라 적절한 HTTP 상태코드로 변환해서 응답

## ExceptionHandler

### 정의
- @ExceptionHandler는 컨트롤러 단위로 세밀하고 유연한 예외 처리를 지원
- REST API 시대에 다양한 응답 포맷을 쉽게 반환하도록 하기 위해 도입됨

## ControllerAdvice

### 정의
- 다른 컨트롤러에서 발생한 예외도 처리가 가능
- 패키지도 경로로 지정 가능
- **모든 컨트롤러(@Controller, @RestController)에 공통적으로 적용**되는 기능을 정의하는 클래스에 붙이는 어노테이션
- 주로 **전역 예외 처리**(Global Exception Handling), **전역 바인딩 설정**, **전역 모델 속성 추가** 등에 사용됨

## 예외 처리 플로우

```
클라이언트
  ↓
WAS
  ↓
필터
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
HandlerAdapter
  ↓
인터셉터 preHandle
  ↓
AOP (진입)
  ↓
핸들러(Controller)  ←★예외 발생!
  ↓
AOP (예외 발생 시 처리)
  ↓
인터셉터 postHandle  ←★예외 발생시 실행되지 않음! (이유는 핸들러에서 리턴이 있을때 실행되기때문)
  ↓
인터셉터 afterCompletion
  ↓
DispatcherServlet (예외 감지)
  ↓
ExceptionResolver (@ExceptionHandler, @ControllerAdvice 등)
  ↓
(필요시) 뷰랜더링
  ↓
DispatcherServlet
  ↓
WAS
  ↓
클라이언트
```

## API 처리할 때 HandlerExceptionResolver를 사용해야 하는 이유

### 필요성
- 다양한 방식의 응답이 필요해짐
- 예외 상황에 따른 유연한 처리 및 일관된 에러 응답 포맷이 필요했음

### 중앙에서 관리하는 것의 장단점
- 정말로 중앙에서 오류를 처리해주는 게 좋은 방법일까? 중앙에서 관리하기 때문에 의존성이 발생하게 됨
- 의존성을 가져가게 되면 영향을 받는 것들이 많아지니 저울질을 해서 선택해줘야 함

### MVC vs REST API
- exception 될 경우 MVC는 model and view 적용해야 하는데 REST API는 그게 없으니까... handler exception resolver?

## 필터와 예외 처리

### 필터 거칠까?
- 거침

### ExcludePath 패턴
- ExcludePath 패턴이라는 걸 사용해서 생략할 수도 있음

## 비동기 처리에서의 예외 처리

### 문제점
- 컨트롤러에서 비동기 처리할 때, 동작 중 오류가 발생함
- 그렇다면 비동기 단계에서 발생했을 때 자바는 어느 단계에서 에러 처리를 할까?

### 해결 방법
- Context가 달라지니까 에러 핸들러가 확인할 수 있는 길이 없음
- @Async 같은 거 선언해놓으면 잡을 수가 없어서 비동기 안쪽에 future로 선언을 해놓거나... future handler error exception이라는 걸 붙여서 에러 처리 어떻게 할 지 미리 등록을 해둬야 됨

## 에러처리 및 핸들링

### 배칭
- 에러를 배치로 처리

### 로깅
- 에러 로깅

### 개발 리소스
- 이거 하나하나 만드는 데에 리소스가 많이 들어간다고 함
- 개념 자체를 분리하고 공부하는 것이 좋음

## #todo
- 예외 처리의 성능 최적화
- 예외 처리의 로깅 전략
- 예외 처리의 모니터링 및 알림
- 예외 처리의 국제화 (i18n)
- 예외 처리의 보안 고려사항
- 예외 처리의 테스트 전략
- 예외 처리의 메트릭 수집
- 예외 처리의 복구 전략
