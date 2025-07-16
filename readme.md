Groom Deep Dive- 백엔드 5회차 과정 수업, 과제, 개인 공부 기록용 리포지토리

사용 도구
Obsidian, VSCode, Git

### 폴더 트리 구조

PARA method 기반
- **Projects**: 현재 진행 중인 작업 영역
- **Areas**: 지속적인 책임 영역
- **Resources**: 참고 자료
- **Archives**: 완료되거나 보류된 항목

### 파일명 규칙
[prefix]_[sufix].md

prefix
- 주를 기준으로 기록, ex) week[num]
- 주 단위는 1부터 시작, 일요일부터 토요일까지를 하나의 week으로 함.
- 레포트 등 과제의 경우 report_ 를 추가로 붙임

suffix
- 수업 내용은 
- 스페이스 바 사용 지양, 하이픈(-) 사용

강의 기록
- Resource에 기록함.

파일 유형

todo
- Areas에 기록함.
- 할 일이 생길 경우 기록, 완료된 항목은 ~~취소선~~ 처리
	- 기록 일자, 시작 일자, 종료 일자, 관련 파일 등의 정보를 항목 하단에 기록.
- Week 별로 구분하며, Week 기간을 넘어가도 Archives로 이전되지 않음.

report_list
- Projects에 기록함.
- 과제 및 레포트에 대한 것을 기록
- 완료된 항목은 취소선 처리
- Week 별로 구분하며, Week 기간을 넘어갈 경우 모든 과제가 완료된 경우에만 Archive로 이전됨.

프로젝트
프로젝트 명의 폴더에만 생성
프로젝트 종료시 Archives로 이전

### 커밋 규칙
커밋 간소화
head 
[날짜] TIL

body
add : 문서 추가
update : 문서 수정의 작은 변화
change : 폴더 트리 구조 변경 등의 변화

ex) add TIL, update sw develop process docs...