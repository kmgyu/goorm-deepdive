## 0709 TIL
### Introduction to GitHub
실습을 위한 OT
가장 기초적인 걸 배운다.
1. 브랜치 만들기
2. 파일 커밋
3. PR 오픈
4. PR 병합

해본 내용이기에 실습은 생략하였음.

#### 느낀 점
개인적으로는 workflows가 흥미로웠음.

GitHub Actions를 활용해 리포지토리에서 단계별 학습을 진행할 때마다 자동화시킨 워크플로우였음.
특히 튜토리얼용 이슈를 생성한다는 게 인상적. action 도중 이슈 자동 생성 등에 활용하는 것을 목적으로 한 것인지 유용해보였다.
yml이나 yaml은 잘 다루지 못하지만 적어도 맥락을 읽어볼 수는 있어야 한다는 걸 알았음.

### Communicate using Markdown
마크다운 파일을 이용해 문서 작성하는 법을 배우는 파트
1. 헤더 생성
2. 이미지 삽입
3. 코드 스니펫 예제
4. 작업 리스트 만들어보기
5. PR 머지
다섯 가지를 실습한다.

위와 동일하게... 모두 해본 내용이기에 실습은 생략하였음...
#### 느낀 점
깃헙 이슈, PR, discussions에서도 md 문법이 적용된다는 게 흥미로웠음.
공식 문서니까 당연하지만... 레포트로 활용할 수 있을 정도로 양질의 정보였다.
```
# 코드스니펫에서는 md 문법이 제한된다.
# 야매로 md 문법을 배웠는데, 처음 안 사실이어서 흥미로웠다.
```

#### 추가 공부
코드스니펫에서 언어별로 하이라이트를 주는 건 lsp라도 적용하는 건지... 어떻게 하는 건지 모르겠다.

### GitHub Pages

Settings - Pages
Build and deployment section (Code and automation section 이라고 하는데, 현재 변경된 것 같음)
Source가 Deploy from a branch로 되어있는지 확인하기
Branch의 경우, 추가적으로 조사가 필요해보인다.
디렉토리 위치가 root인 경우와 /docs인 경우 결과가 달랐는데, 문서 작성 위치를 기준으로 하는 것으로 보인다.

배포 프로세스도 중요한 것 같음.

#### 느낀점
깃헙 페이지다! 다른 사람이 활용하던 것만 보고 언제 만들지 생각하고 있었는데, 드디어 배울 기회가 생겼다.
사실 나도 이미 가지고 있다...
https://kmgyu.github.io/

리포지토리 이름도 github pages와 같은 형식으로 작성해야 하는 줄 알았는데, 전혀 상관없었다.

블로그로 운영하는 케이스와 아예 웹서비스를 깃헙 페이지에 탑재해서 운영하는 케이스도 있었는데, 배포할 경우 깃헙 페이지를 사용하는 방안도 고려해볼 법 하다. (이 경우 추가조사가 필요할 듯)

블로그의 경우 더 예쁜 본인의 블로그를 위해 오픈소스를 사용하는 경우가 있었다... 몇 개 스타해두긴 했는데 추가적으로 알아볼 필요가 있다.

## 0710 TIL

review PR부터는 직접 실습하며 진행하였음.
### review PR

PR에도 다양한 기능이 있다는 걸 알게 되었다.
단순히 PR을 코드 리뷰 하는 것 뿐만 아니라 깃허브 내에서 suggestion 기능을 통해 제안하고, 커밋까지 하여 에디터 역할을 할 수 있는 것, Assigner 등... 새로운 기능을 알게 되었다.
교육용으로 굉장히 좋은 것 같다. 지인들에게 추천해야 겠다.
### resolve merge conflicts

캡스톤 진행하면서 질리도록 경험해 본 내용이라 익숙한 내용이었다.
추가로 프로젝트 크기가 커질 경우 merge conflict를 놓치게 되면 그 상태 그대로 커밋 되버려서 끔찍했던 기억이 있다... 꼼꼼히 확인하는게 좋다.
vscode를 이용해 merge conflict를 확인할 수 있다. 여기도 좋은 도구를 제공한다.

### release-based workflow

릴리즈 기반 워크플로를 배우는 실습이다.
베타 릴리즈를 만들고, 피쳐 릴리즈를 만들고...
아마 태깅과 브랜치 만들기, 커밋 등을 종합적으로 하는 실습으로 보인다.

깃헙 플로우에 대한 설명을 먼저하고 시작한다. 깃 플로우보다 경량화되고 간단하다고는 들었는데 이것도 한번 사용해봐야겠다.

태깅과 릴리즈를 사용해보았는데, 릴리즈 기능은 처음 사용해봐서 생소했다.
릴리즈 노트 자동화와 PR 내용 템플릿이 굉장히 신기했는데, 알아두면 좋을 것 같다. 물론 DevOps가 하는 내용이지만... 알아둬서 나쁠 건 없다고 생각한다.

순서가 꼬인건지, 릴리즈 finalize를 그대로 넘어갔다. 뎃?? 실수로 누른 건 아닌 것 같은데.

PR 타이틀이나 description에 대한 내용도 참고할 것이 있다!

## 0711 TIL
여기부터는 리포지토리를 좀 남겨둘 필요를 느꼈다.
깃헙 액션 신기행
### connect the dots
_Useful tips when navigating through your repository._

제목부터 점들을 연결하기이다. 진행하면서 의미를 좀 더 알 수 있게 되었는데, 커밋이나 이슈, PR 등의 연결성을 더 강하게 만드는 방법에 대해 소개하는 튜토리얼이라고 보면 될 것이다.
오픈소스 기여나 프로젝트에서 활용할 수 있을 것이다.

step1
깃헙 이슈 해결에 대해 배운다.
해시태그``#[number]`` 를 통해 중복 이슈에 대해 언급할 수 있다.
깃헙 이슈 볼때마다 관련 이슈 나오는 게 있었는데 이런식으로 넣는 거였다!
지식이 늘었다.

이슈 형식은 이렇게 썼다. 이슈 기초 가이드로도 쓸 수 있겠다.
 
GIVEN:
- User opens _sidebar.md file

WHEN:
- User navigates by clicking on [Documentation references] link

THEN (EXPECTED):
- [Documentation references] page must be open successfully

OBSERVED:
- File not found error OR GitHub reports HTTP 404 error (file not found)


step2
git blame을 사용해 깃 히스토리에서 커밋을 찾기?
이 커밋과 관련된 PR은 무엇인지, 누가 승인했는지, 병합 전 어떻게 테스트했는지 등을 찾기 위해 만들어진 도구로 보인다.
이름이 참 재밌다. git blame...

git blame에 대해서는 튜토리얼에서 설명하고 있다. 찾아보자.
Git lens에서 보여주는 것처럼 어떤 코드가 어느 커밋에서 등장했는지 같은.. 정보를 보여주는 것으로 보인다. Git lens가 git blame 기반일 수도 있다.

sha에 대해서도 설명한다. 커밋 id에 사용되기 때문에 부가적으로 설명한다.

커밋 id 앞자리 7글자를 작성하면, 놀랍게도 해당 커밋으로 이동이 된다! 신기해서 전부 입력했더니 결과는 동일하다.

step3
이제 이슈와 관련된 커밋을 발견했다. 연결이 되었다!
커밋이 병합된 PR을 찾고, 수정한다.
수정 후, 댓글에 이슈에 대해 언급 가능하다.
이렇게 하면 이슈에 대해 수정한 PR을 만들 수 있다.

## code with codespaces
_Develop code using GitHub Codespaces and Visual Studio Code!_

깃헙 코드스페이스를 통해 개발하는 방법에 대한 실습이다.
깃헙에서 호스팅하는 클라우드 개발 환경으로 보인다.

https://vscode.dev/?vscode-lang=ko
이걸 이용해 vsc를 원격으로 접속할 수 있다.
개쩔어 뭐야 이거

코드스페이스 들여다보기는 가능하고, 클라우드 서비스를 제대로 이용하려면 MS 계정 또는 깃헙 인증을 하고 리소스를 할당받아야 한다. 상당히... 귀찮다...
근데도 쩐당

dev container라고 해서 가상 환경에서 관리하는 형식이라고 설명함.
도커를 이용하며, json 파일로 가상환경 세팅도 할 수 있다.

dotfile이라는 것도 있다...
When using any development environment, customizing the settings and tools to your preferences and workflows is an important step. GitHub Codespaces offers two main ways of personalizing your codespace: `Settings Sync` with VS Code and `dotfiles`.

`Dotfiles` will be the focus of this activity.

**What are `dotfiles`?** Dotfiles are files and folders on Unix-like systems starting with . that control the configuration of applications and shells on your system. You can store and manage your dotfiles in a repository on GitHub.

시스템의 애플리케이션과 셸 구성을 제어하며, 깃헙 리포지토리에서 저장 관리가 가능한 파일이라고 한다.

sl 실행 안되서 수동으로 설치하고 다시 재생시켰다.
이거 어거지로 하는 건데 맞나...?
이러면 잘못된 건데 엄
아무튼 작동은 한다.
sl 누르면 기차가 칙칙폭폭 나오는게 정상이다.

### introduction to repository management
_Learn the basics of several GitHub features that can help support a collaborative, friendly, and healthy project._

리포지토리 관리에 대한 실습이다.
협업자들이 많아져도 리포지토리 내용을 안전하게 유지하는 내용을 학습할 것으로 보인다.

룰셋 생성하기, 커뮤니케이팅 절차로 협업자들 가이드하기 등...

1. Add a simple rulesets and configuration to restrict repository content.
2. Communicate procedures to help guide collaborators.
3. Assign responsibility of parts of the code to particular collaborators.
4. Learn the difference between collaboration in a personal repository and organization repository.
5. Establish ground rules to promote a health collaboration environment.
6. Establish a process for managing security updates.

위 과정을 거친다!

디테일한 설명은 넣지 않았다고 하니 이는 다른 오픈소스들을 예제로 참고하면 될 것 같음.

웬 오징어 선생님이 나와서 이슈 탭에서 설명해주는데 짱신기했다.

컨트리뷰선 가이드 페이지와 코드 오너 페이지를 만든다.
행동 강령(code of conducts)에 대한 문서도 작성하게 된다.

버그나 기능 요청 같은 것에 대한 템플릿도 작성한다.
md 형식인데, 추가하는 형식이 독특했다.
settings의 general 탭, features 섹션의 issuses 로 들어가서 직접 수정해야 한다! 너무 외진 곳에 있는 것 같다.

코드 보안에 대한 처리도 해본다.
dependabot, 코드 스캔, security policy 등의 세팅들...

PR이나 이슈 같은 쪽은 조금 허술한 면이 있었는데 이번에 한번 보면서 개선할 수 있게 되었다.