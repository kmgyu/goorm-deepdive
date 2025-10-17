
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


참고자료모음
https://ksh-coding.tistory.com/134 - aws ECS
https://kdh0518.tistory.com/25 - docker hub ci/cd
