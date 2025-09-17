## 이슈 트래킹 시스템이란 무엇일까?
프로젝트에서의 이슈는 개발할 새 기능, 수정해야 하는 결합, 문제가 된 이슈 등을 통틀어서 말합니다. 이것을 티켓, 이슈, 테스크, 케이스라고 부릅니다. **이슈 트래킹 시스템은 이런 이슈를 생성하고 담당자를 지정하여 해결하도록 하는 시스템**을 말합니다.

``referenced``
[[Jira] 이슈 트래킹 시스템을 활용한 프로젝트 만들기](https://velog.io/@jihyeong00/%EC%9D%B4%EC%8A%88%ED%8A%B8%EB%9E%98%ED%82%B9%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%9C-%ED%98%91%EC%97%85%EB%B0%A9%EC%8B%9D)

### 그럼 왜 이슈 트래킹 시스템을 사용할까?
- **성공적인 프로젝트 관리** 
  이슈 트래킹 툴을 사용하면 어떤 일이 발생했고, 어떤 일이 아직 해결되지 않았는지 항상 알 수 있습니다. 이슈를 관리하는 체계화된 방식을 제공하여 팀원 모두가 문제를 효율적으로 해결할 수 있게 돕습니다.
- **프로젝트 진행 확인과 협업**
  모든 팀원은 자신이 완료해야 하는 일이 무엇인지, 이 일이 완료되면 다른 팀원이 무엇을 해야 하는지 자신과 팀 구성원의 역할을 정확히 알 수 있습니다. 또 프로젝트의 중앙 허브가 되어 팀원 간의 정보 격차를 줄여줍니다.
- **이슈 DB화**
  나중에 똑같은 문제가 발생하거나 비슷한 문제가 발생 할 때 예전 자료를 참고하여 더욱 쉽게 쉽게 처리 할 수 있습니다.

``referenced``
[이슈 트래킹 시스템 탈탈 털기 — ITS 정의부터 Jira vs Asana vs Linear 비교까지](https://medium.com/saas-design-archive/%EC%9D%B4%EC%8A%88-%ED%8A%B8%EB%9E%98%ED%82%B9-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%83%88%ED%83%88-%ED%84%B8%EA%B8%B0-its-%EC%A0%95%EC%9D%98%EB%B6%80%ED%84%B0-jira-vs-asana-vs-linear-%EB%B9%84%EA%B5%90%EA%B9%8C%EC%A7%80-4407eaee9199)
유지보수에 뿐만 아니라, 새로운 기능을 만들거나 프로젝트 관리에 있어서 가장 적합한 시스템이기에 서비스 기업에서 사용하는 이유가 아닌가 싶다.
또한, 요즘 많이 쓰이는 애자일 기반 방법론처럼 변화에 유연하게 대응하기 위해 제안된 시스템이라고도 볼 수 있을 것 같다.


## 이슈 트래킹 시스템에는 어떤 것이 있는가?
이슈 트래킹 시스템은 Jira, phabricator, yona 등 다양한 것이 있다. 여기서 자세히 다루지는 않을 것이다.

Jetbrains가 2019년을 기준으로 다룬 [이슈트래킹 리뷰](https://www.jetbrains.com/ko-kr/lp/tracking-tools-review-2019/)는 다음과 같다.

> 조사 대상 도구 중 클라우드 버전만을 제공하는 제품은 Asana, Monday.com, Trello 및 Wrike였으며 독립실행형 버전만을 제공하는 제품은 Bugzilla 입니다. 한편 클라우드 및 독립실행형 버전을 모두 제공하는 솔루션은 Azure DevOps Server(구 Team Foundation Server), GitHub Issus, GitLab Issues, Jira, Pivotal(현 VMware Tanzu), Redmine 및 YouTrack 입니다.

### 가장 잘 알려진 도구
- Jira - 72%
- Trello - 67%
- GitHub Issues - 63%
- GitLab Issues - 42%
- Redmine - 30%
- YouTrack - 19%

### 가장 많이 추천 받은 도구
- YouTrack - 66%
- GitHub Issues - 65%
- Asana - 60%
- GitLab Issues - 59%

### 가장 높은 만족도
- Jira - 68%
- GitHub Issues - 67%
- GitLab Issues - 61%
- YouTrack - 59%

### 기능 면에서 최고의 도구
- YouTrack - 3.9/5.0
- GitHub Issues - 3.7/5.0
- Jira - 3.7/5.0

### 특성면에서 최고의 도구
- GitHub Issues - 4.1/5.0
- Jira - 4.1/5.0
- YouTrack - 3.9/5.0

### IT 이외 분야의 기업에서 선호됨
- Trello - 33%
- YouTrack - 32%

> 클라우드 또는 독립실행형
> 온프레미스 및 클라우드 솔루션은 거의 동등한 비중을 차지했습니다: 응답자의 51%는 클라우드 도구를, 49%는 독립실행형 솔루션을 사용합니다.
> 사용자들이 독립실행형 솔루션을 계속 사용하는 주된 이유는 기업 정책인 반면 클라우드 솔루션을 선택한 이유는 더욱 간편한 유지보수 및 관리였습니다.

점수의 기준 및 추가적인 내용은 원문을 찾아 읽어보도록 하자.
주로 Jira, GitHub Issues, GitLab Issues, YouTrack, Trello 등이 사용됨을 알 수 있다. (현재 2025년 기준으로는 어떤 것이 사용될지는 모르겠지만 Jira는 20년 이후 포스트에서도 자주 나오는 것으로 봐서 많이 쓰이는 것 같다.)

``related``
[이슈 트래킹 시스템 비교 (Issue Tracking system/Bug Tracking System)](https://blog.gaerae.com/2014/05/issue-tracking-system-bug-tracking-system.html) - 2019.03.10 작성됨

## 주요 용어

### 이슈 유형

Jira를 기준으로 작성되었다.
프로젝트별 컨벤션 문서, 또는 툴에 따라 **상이**할 수 있다.

- Epic (큰틀) : 여러 스프린트에 걸쳐서 끝나지 않고, 여러 스토리들의 집합입니다.
- Story : “{사용자} 로써 {무엇}을 하고싶다” 에 대한 액터의 유즈케이스
- Chore : 사용자와는 직접적으로 관계되지 않는 개발 (DB 세팅, 분리 등)
- Task : 구현에는 직접적으로 관련이 없는 업무 (문서작성 등)
- Issue : 이슈 사항 (서버 다운, 클라우드 계약 등)
- Bug : 테스트 엔지니어로부터 버그로 리포팅된 타입
- Sub Task : 스토리 혹은 초어들을 개발하기 위해 진행되는 실제 세부 개발사항들

워크플로우 (Workflow): 이슈의 진행 단계를 정의하며, 이슈가 생성부터 완료까지 거치는 과정을 시각적으로 보여줍니다.

칸반 보드 (Kanban board): 이슈의 진행 단계를 시각적으로 표현하여 우선순위를 관리하고 효율적인 작업을 지원하는 도구입니다.

필드 (Field): 이슈를 구성하는 필요한 정보 항목입니다. (예: 제목, 설명, 담당자, 우선순위, 상태, 마감일 등)

스크린 (Screen): 이슈를 생성하거나 편집할 때 보이는 필드의 집합입니다.

``referenced``
[Jira를 통해 스크럼 관리하기](https://taes-k.github.io/2019/12/07/sw-jira-scrum/)
[Jira 사용 전 알아야 할 기초 용어 총정리](https://dmove.tistory.com/entry/atlassian-jira-word-dictionary)


