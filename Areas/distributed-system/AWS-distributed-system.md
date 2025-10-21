
분산 환경에서의 문제점이나 성능 테스트를 실험하려면 클라우드를 이용해야 한다.

AWS, Azure, Vultr 등을 사용할 수 있다.

환경 구축하기 쉬운... AWS를 사용했다.
Azure는 학생 버전이 존재하지만 학습 곡선이 높다. 빠르게 해야 한다.

분산 환경 구축에 앞서, 깃헙에서 직접 소스코드를 풀해와서 빌드하고 관리하는 것은 끔찍한 일이다. Docker를 이용해 이미지화해서 컨테이너로 관리해주도록 하자.

CI/CD 파이프라인을 구축해볼 것이다.
1. Github Actions를 통해 이미지를 빌드하고, Docker hub에 푸시한다.
2. EC2 ssh access
3. 기존 컨테이너 닫고 pull, 도커 이미지 실행
Github Actions를 이용하면 이런 식으로 된다.
근데 분산환경이면 ec2 다해줘야 하지 않나?

---

분산 시스템 구축 자체가 처음이기에 구글링 해가면서 조사했다.

[[aws-summit-2017-aws-ecs]]

https://heekng.tistory.com/242 - aws ecs + fargate 클라우드 배포


aws에 구축하는 클라우드 디자인 패턴 시리즈 1~5부를 참고했다.
[AWS 에 구축하는 클라우드 디자인 패턴 시리즈 1부: 안정성](https://aws.amazon.com/ko/blogs/tech/cloud-design-patterns-on-aws-1/)  
[AWS 에 구축하는 클라우드 디자인 패턴 시리즈 2부: 연결성 및 조합](https://aws.amazon.com/ko/blogs/tech/cloud-design-patterns-on-aws-2/)  
[AWS 에 구축하는 클라우드 디자인 패턴 시리즈 3부: 마이그레이션](https://aws.amazon.com/ko/blogs/tech/cloud-design-patterns-on-aws-3/)  
[AWS 에 구축하는 클라우드 디자인 패턴 시리즈 4부: API 관리](https://aws.amazon.com/ko/blogs/tech/cloud-design-patterns-on-aws-4/)  
[AWS 에 구축하는 클라우드 디자인 패턴 시리즈 5부: 데이터 관리](https://aws.amazon.com/ko/blogs/tech/cloud-design-patterns-on-aws-5/)

초기에는 kafka로 cheorography 패턴을 구성해야 하지 않나 싶었는데 트래픽 분산을 해야하기에 로드 밸런서가 구조에 필요했다.

이 경우 로드밸런서로 nginx 또는 aws alb, k8s 등이 있다.

EC2는 2~3대로 테스트 목적으로만 사용할 것이기에 당연히 빠른 구현이 가능한 nginx를 선택한다.


251021 현재 구조는 다음과 같다.

인스턴스
LB (load balancer)
api-1
api-2
monitoring

DB는 제외되었으나, 로컬에서라도 배포해야하나 생각중

CI/CD도 겸해서 해볼것이다.

Jenkins를 사용해야하나 github actions를 사용해야하나 고민중

로드 밸런싱 사용하기 때문에 api 쪽은 8080만 뚫어준다.
나머지는 private ip 사용하기 때문에 아웃바운드 뚫어주지 않는다.
(정확히는 동일한 VPC를 사용하기 때문에 가능하다.)


추가 참고 자료 모음
https://ksh-coding.tistory.com/134 - aws ECS
https://kdh0518.tistory.com/25 - docker hub ci/cd
