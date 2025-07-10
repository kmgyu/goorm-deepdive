# 태그

다른 VCS처럼 Git도 태그를 지원한다. 사람들은 보통 릴리즈할 때 사용한다.
Git의 태그는 _Lightweight_ 태그와 _Annotated_ 태그로 두 종류가 있다.

### Lightweight 태그

특징
- 브랜치와 유사
- 가리키는 지점을 최신 커밋으로 이동시키지 않음.
  특정 커밋에 대한 포인터라고 볼 수 있음.

### Annotated 태그

특징
- Git 데이터베이스에 태그를 만든 사람의 이름, 이메일과 태그를 만든 날짜, 그리고 태그 메시지도 저장한다.
 - GPG(GNU Privacy Guard)로 서명할 수 있다.
 
 일반적으로 Annotated 태그를 만들어 이 모든 정보를 사용할 수 있도록 하는 것이 좋다고 권장하고 있다.
 하지만 임시로 생성하는 태그거나 이러한 정보를 유지할 필요가 없는 경우에는 Lightweight 태그를 사용할 수도 있다.

``reference and related``
[2.6 Git의 기초 - 태그](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%ED%83%9C%EA%B7%B8)
[[GIT] ⚡️ 태그 기능 및 사용법 (git tag)](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-%ED%83%9C%EA%B7%B8-%EA%B8%B0%EB%8A%A5-%EB%B0%8F-%EC%82%AC%EC%9A%A9%EB%B2%95-tag#annotated_%ED%83%9C%EA%B7%B8_%EC%83%9D%EC%84%B1)
