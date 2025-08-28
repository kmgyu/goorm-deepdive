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

## 0714 TIL
Hello GitHub Actions
.github 디렉토리에서 workflow의 yml을 관리한다.
yml 문법을 알아야 어떻게 할 것 같다. 으아각

## 0728 TIL
Test with Actions
copilot이랑 대충 ai쪽 섹션이 있었는 데 사라졌다. 아니 뭐지;
CI에 대해서 배운다.
협업하는 방식과, 피드백, 테스트, 텥스트보고서 등...?
브랜치 보호랑 PR이랑...

1. 테스트 워크플로
워크플로 생성법에 대해서 배운다.

용어
워크플로 : 자동화의 단위. 트랜잭션처럼 시작하고 종료하는 순간에 대한 최소 단위라고 보면 될듯. 비유가 정확할까 싶겠지만.. 유사하다!
잡 : 워크플로의 섹션. 한 개 이상의 스텝으로 이루어졌다.
스텝 : 자동화의 효과를 나타낸다. 깃헙 액션이나 콘솔을 통해 무언가를 출력하는 것으로 정의될 수도 있다?
액션 : 워크플로와 호환되는 방식으로 작성된 자동화의 일부라는데, 실제 워크플로와 호환되도록 작성된 자동화... 대충 그런의미인듯?
GitHub, 오픈 소스 커뮤니티, 직접 작성 등의 방법이 있다.

> 참고 : [문서](https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions)를 통해 문법을 확인 가능하다.

브랜치 PR하다가 에러가 나서 수정했더니 그 커밋도 PR에 적용이 된다. 신기하네.

아티팩트 스토리지를 이용해서 각 액션의 결과물을 다음 액션으로 넘길 수 있다고 한다. 이게 맞겠지? 
브랜치 프로텍션을 해줘야 한다. 하지 않으면 봇이 멋대로 메인에 머지할 수도 있다고... 컨트리뷰터가 아니더라도 악용할 수 있기 때문에 이런 조치를 취해줘야 하는 듯.

## 0730 TIL
Publish packages
깃헙 액션을 이용해 도커 이미지로 퍼블리싱하는 방법을 배운다!

> **Continuous integration** (CI) is a practice where developers integrate tested code into a shared branch several times per day. **Continuous delivery** (CD) is the next phase of **continuous integration** (CI) where we also make sure to package the code in a _release_ and store it somewhere - preferably, in an artifact repository. Lastly, **Continuous deployment** (CD) takes **continuous delivery** (CD) to the next level by directly deploying our releases to the world.

도커파일 넣었는데 패키지 등록이 안된다. 어쩌라는 거지?
잘 찾아보니 publish.yml에서 yourname을 내 계정으로 바꿔주지 않았었다. 이런 젠장

도커 이미지 잘 뜬다.
이거 하려고 도커랑 wsl2 세팅햇다.
근데 나쁘지 않은데?
yml 생쇼한것만 빼면... 괜찮다....

## 0731 TIL
Deploy to Azure
아주르가 뭔데 십덕아

라벨에 따라서 다른 액션을 줄 수 있다.
계정을 만들어야 한다.
계정 아직 안만들었다. 다음에 한다. 으엥

## 0801 TIL
구독서비스 신청해야하는데 학생 요금제가 있어서 살았다. 끼얏호우

```
winget install --exact --id Microsoft.AzureCLI
```

azure cli 깔아준다. windows 10 이상부터는 winget이라는 패키지 관리자를 쓴다.

``` bash
az ad sp create-for-rbac --name "GitHub-Actions" --role contributor \
 --scopes /subscriptions/{subscription-id} \
 --sdk-auth

    # Replace {subscription-id} with the same id stored in AZURE_SUBSCRIPTION_ID.
```
이거 추가하고한다.

시크릿이랑 env랑 다른 게 뭔지 모르겠다.
리포지토리에서 로드하는 건 액션이 알아서 하나?

엄.....
스테이징은 도커 컨테이너 생성?
azure config는 뭐하는 거지. 아주르에 배포하는 액션인 듯?

This new workflow has two jobs:

1. **Set up Azure resources** will run if the pull request contains a label with the name "spin up environment".
2. **Destroy Azure resources** will run if the pull request contains a label with the name "destroy environment".

**Setting up Azure resources**

The first job sets up the Azure resources as follows:

1. Logs into your Azure account with the [`azure/login`](https://github.com/Azure/login) action. The `AZURE_CREDENTIALS` secret you created earlier is used for authentication.
2. Creates an [Azure resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups) by running [`az group create`](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-create) on the Azure CLI, which is [pre-installed on the GitHub-hosted runner](https://help.github.com/en/actions/reference/software-installed-on-github-hosted-runners).
3. Creates an [App Service plan](https://docs.microsoft.com/en-us/azure/app-service/overview-hosting-plans) by running [`az appservice plan create`](https://docs.microsoft.com/en-us/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) on the Azure CLI.
4. Creates a [web app](https://docs.microsoft.com/en-us/azure/app-service/overview) by running [`az webapp create`](https://docs.microsoft.com/en-us/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) on the Azure CLI.
5. Configures the newly created web app to use [GitHub Packages](https://help.github.com/en/packages/publishing-and-managing-packages/about-github-packages) by using [`az webapp config`](https://docs.microsoft.com/en-us/cli/azure/webapp/config?view=azure-cli-latest) on the Azure CLI. Azure can be configured to use its own [Azure Container Registry](https://docs.microsoft.com/en-us/azure/container-registry/), [DockerHub](https://docs.docker.com/docker-hub/), or a custom (private) registry. In this case, we'll configure GitHub Packages as a custom registry.

**Destroying Azure resources**

The second job destroys Azure resources so that you do not use your free minutes or incur billing. The job works as follows:

1. Logs into your Azure account with the [`azure/login`](https://github.com/Azure/login) action. The `AZURE_CREDENTIALS` secret you created earlier is used for authentication.
2. Deletes the resource group we created earlier using [`az group delete`](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-delete) on the Azure CLI.

두뇌 뜌땨당해버린 와타시는 흠 그렇군 하고 말아버리는 것이에오...
액션에 if랑 with도 들어간다. creds는 credentials인가? 파일채로 끌어오는건가

라벨 붙이니까 액션 돌아간다. PR까지 자동으로 해준다!! 딸깍 한번에 전부 해준다니 이렇게 편할 수가 없다

deploy가 안된다. 아이에에에에!! 에러??!! 에러 왜???????

음
원인을 알아냈다! 시크릿에 구독 ID를 등록안했다.
듀...

아니 키 넣어줬더니 이번에는 등록안되있다고 뭐라고하네
[에러모음](https://learn.microsoft.com/en-us/azure/azure-resource-manager/troubleshooting/common-deployment-errors#noregisteredproviderfound)

[해결법](https://learn.microsoft.com/en-us/azure/azure-resource-manager/troubleshooting/error-register-resource-provider?tabs=azure-cli)


```
az provider register --namespace Microsoft.Web
```
오류 코드 쪽에서 어떤 오류인지에 대한 설명과, 해결하는 곳으로 도달하는 하이퍼링크가 깔쌈하게 되있다. 역시 국가권력급 대기업이다. 문서화가 잘되있다.

Re-run 버튼 예전에 써보고 어디서 찾는지 까먹었다가 알게된건데 workflow랑 job마다 구분이 되어있다! 딱히 쓸모있는 지식은 아니지만 지식이 늘?었다.

yml 관련해서 on, job, workflow 단위 다 봤던 거 같은데 기록이 안되어있다. 이건 좀 사고인데... 여기 써진 기록 정리하면서 추가하면 될 것 같음

아무튼 스테이징하니 리소스 모니터링하면서 VM이랑 웹앱 추가된걸 확인했다.
이제 배포쪽 액션이 실행되면?

틱퉥퉈!!!!가 나온다

머리를 좀 더 굴려보니까 도커파일을 리포지토리에 미리 넣어놓고 VM에서 컨테이너형식으로 빌드하는 것 같ㄷㅏ.
스테이징이 이미지 만들어서 클라우드(아주르) 쪽에 이미지 빌드하고 뚝딱뚝딱쓰하는 거인듯.

이번엔 CI/CD에서 CD단계다.
production-deployment-workflow 브랜치만든다.

> **Continuous delivery** (CD) is a concept that contains many behaviors and other, more specific concepts. One of those concepts is **test in production**. That can mean different things to different projects and different companies, and isn't a strict rule that says you are or aren't "doing CD".

라는데, 그럼 뭐 재즈 스윙 같은건가.
넌 전혀 CD하고 있지 않아. 넌 매우 CD하고 있어 이런거임?

아무턴 이 단계까지 하면 프로덕션 배포까지 끝난다고 한다. 메다닥 끝나서 좀 더 공부가 필요할 것 같은 레후.

이제 실습 끝났으니 리소스 다 지우라고 한다. 야호!

음. 수동으로 지웠는데 destory environment가 따로 있었다. 이제봤네
닫았던 PR을 다시 열어서 라벨을 새로 적용하는 방법도 있나보다. 
근데 실제로 그렇게 할 케이스가 있나..?
으에...

작업 Recap
- Trigger a job based on labels
- Set up the Azure environment
- Spin up environment based on labels
- Deploy to a staging environment based on labels
- Deploy to a production environment based on labels
- Destroy environment based on labels



## 0818 TIL
Write JS actions
https://github.com/skills/write-javascript-actions

깃헙 JS 액션을 작성하고 자동화 커스텀된 특별한 태스크를 워크플로에 적용한다.

step 1. js 플젝 초기화
액션은 기본으로 활성화되있지만 리포지토리가 그걸 쓰게 해줘야 함.
일단 워크플로 파일부터 만들 거임

워크플로 파일은 태스크 자동화하는 레시피라고 보면 된다.
특정 트리거에 따라 발생해야할 job과 step에 대한 시작과 종료 지침을 하우징함.

리포지토리는 다양한 범위의 태스크 유형을 가진 복수의 워크플로 파일을 보유할 수 있음. 따라서 워크플로에 네이밍하는 건 매우 중요하다. 선택된 이름은 수행하는 작업을 반영해야 한다.

JS 작업은 깃헙 개발을 위해 깃헙 툴킷을 이용한다.

npm을 이용해 설치할 확장 라이브러리임. 따라서 node.js가 필요함.

로컬환경에서 워크플로 작성은 리포지토리에서 바로 처리하는 것보다 훨씬 쉽다. 로컬 환경에서 이런 단계 수행 시 원하는 편집기를 사용가능하므로, 코드 작성 시 익숙한 모든 확장 기능 및 스니펫을 사용 가능.

스텝 2. 액션 설정하기
액션 추가한다.

스텝 3. 메타데이터 파일 만들기

액션 메타데이터
우리가 작성하는 모든 깃헙 액션에는 메타데이터 파일이 필요함.
몇 가지 규칙이 있다.
- 파일 이름은 반드시 action.yml
- 도커 컨테이너와 JS 액션이 요구됨.
- YAML 문법으로 작성됨

액션에 대한 다음 정보를 정의한다.

|Parameter|Description|Required|
|---|---|:-:|
|Name|The name of your action. Helps visually identify the actions in a job.|✅|
|Description|A summary of what your action does.|✅|
|Inputs|Input parameters allow you to specify data that the action expects to use during runtime. These parameters become environment variables in the runner.|❌|
|Outputs|Specifies the data that subsequent actions can use later in the workflow after the action that defines these outputs has run.|❌|
|Runs|The command to run when the action executes.|✅|
|Branding|You can use a color and Feather icon to create a badge to personalize and distinguish your action in GitHub Marketplace.|❌|

메타데이터 파일을 추가한다.

스텝 4. 액션을 위한 JS 파일 만들기

> 아시다시피, JavaScript를 비롯한 다른 프로그래밍 언어에서는 코드를 모듈로 나누어 가독성과 유지 관리를 용이하게 하는 것이 일반적입니다. JavaScript 액션은 특정 트리거에 따라 실행되는 JavaScript로 작성된 프로그램일 뿐이므로, 액션 코드도 모듈화할 수 있습니다.

두 개 파일을 만든다.
하나는 외부 API와 joke를 가져오는 로직
다른 건 그 모듈을 불러서 joke를 액션 콘솔에 출력시킬 것
제대로 해석한 건가?
joke.js로 외부에서 가져오고, main.js를 통해 실행하는 듯

스텝 5. 액션을 워크 플로 파일에 추가하기
```
      - name: ha-ha
        uses: ./.github/actions/joke-action
```
해당 코드를 이미 존재하는 yml에 추가한다.

스텝 6. joke 액션 트리거
first joke, second joke 라벨로 액션을 트리거한다.

joke를 액션 콘솔에서 확인할 수 있다.
```
Two peanuts were walking down the street. One was a salted
```
second joke는 작동을 안한다. 왜지?
### 0828 TIL
reusable workflows
https://github.com/skills/reusable-workflows

재사용 가능한 워크 플로 전략

깃헙 워크 플로 복붙하는 사람이 많을 것이다.
이런 귀찮은 복붙을 없애기 위해 재사용 워크 플로를 소개한다!
그 예시로, 노드 앱이 모두 같은 방식으로 빌드하는 경우 그냥 같은 워크플로를 사용할 수 있음.

재사용 가능하게 만드려면 어떻게 해야 하는가?
재사용가능한 워크플로는  푸시, 이슈, 워크플로 디스패치 등과 유사한 이벤트 트리거인 workflow_call 트리거를 포함한다.

스텝3
매트릭스 전략을 추가하여 범용으로 만든다!

**What is a matrix strategy**: A matrix strategy lets you use variables in a single job definition to automatically create multiple job runs that are based on the combinations of the variables. For example, you can use a matrix strategy to test your code in multiple versions of a language or on multiple operating systems. Below is an example:

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
        os: [ubuntu-latest, windows-latest]
```

example matrix는 os와 nodejs 버전을 조합해 가능한 것으로 실행한다.
총 6개 잡을 각각 돌려서 가능한 것을 실행시킴.

Actions 탭에서 내 워크플로들이 뜬다. 거기서 선택해서 직접 브랜치에 트리거 시킬 수 있다. 브랜치도 선택해야 함.
run workflow 돌리면 된다.