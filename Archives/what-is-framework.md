# 프레임워크

## 정의
소기의 목적을 달성하거나 복잡한 문제를 해결하고 서술하는 데 사용되는 기본 개념 구조

*프레임워크라고 하면 가장 먼저 웹 프레임워크인 스프링이 떠오른다.*  
*그러나 닷넷, 하둡과 같은 플랫폼 기반 프레임워크부터, 소프트웨어 개발 방법론에서 사용되는 애자일 프레임워크까지 다양한 맥락에서 이 용어가 사용된다.*

## 프레임워크와 라이브러리의 차이?
프레임워크와 라이브러리는 개발자가 반복적인 일을 하지 않게 만들어준다는 점에서 유사하다. 어떤 차이가 있을까? 이미 학습한 내용일 수 있지만, 프레임워크를 이해하기 위해 핵심적인 차이점을 다시 정리해보자.

정보처리기사에서 제시되는 프레임워크의 특징은 다음과 같다.
- 모듈화
  인터페이스에 의한 캡슐화를 통해 모듈화를 강화하고 설계와 구현의 변경에 따르는 영향을 극소화한다.
- 재사용성
  반복적으로 사용할 수 있는 컴포넌트를 정의할 수 있게 하여 재사용성을 높여준다.
- 확장성
  다형성을 통해 애플리케이션이 프레임워크의 인터페이스를 넓게 사용할 수 있도록 한다. 프레임워크는 애플리케이션 서비스나 특성이 변하더라도 그 영향에서 독립적으로 유지되며 이로 인해 프레임워크는 다양한 상황에서 재사용될 수 있는 장점을 가진다.
- **제어의 역전 (IoC, Inversion of Control)**

![프레임워크 - 개발자 코드 - 라이브러리](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FMvJNW%2FbtsbPW6SKz0%2FAAAAAAAAAAAAAAAAAAAAAGEt_h8BK5ku2T6tUy73UyLoja_q9gOev4jpUyOgKpCX%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DeZLH26FetINyOhDWeSy0T4Yvd8U%253D)
> 제어 흐름의 주체를 시각적으로 나타내는 사진

프레임워크와 라이브러리와의 차이점은 바로 제어의 역전에 있다.
애플리케이션의 흐름에 대한 주도권이 어디에 있는지가 차이점이라는 것이다.
예를 들면 프레임워크는 설정파일의 태그설정이나, DB 연동 방법 등에 대한 규칙을 갖고 있고, 개발자는 이를 따라야 한다.
반대로 라이브러리는 개발자가 직접 호출하기 때문에 개발자에게 주도권이 있다.

> 예를 들어, React는 사용자 인터페이스(UI)를 구성하는 기능만을 제공하므로 일반적으로 라이브러리로 분류된다. (코드 컨벤션, 파일 구조 등을 강제하지 않음.) [MDN-react](https://developer.mozilla.org/ko/docs/Learn_web_development/Core/Frameworks_libraries/React_getting_started)

[프레임워크 vs 라이브러리](https://sharonprogress.tistory.com/169)
https://hypers84.tistory.com/2461

### 다양한 프레임워크
- JWT
- Spring
- .Net Framework
- Qt
- Android
- Bootstrap

