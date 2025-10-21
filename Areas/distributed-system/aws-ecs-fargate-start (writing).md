aws는 가장 잘 알려진 클라우드 서비스 제공업체이다.
대기업들은 보통 분산시스템으로 서비스를 운영하는데, 이를 구축하기란 쉽지 않다.

실제로 대용량 트래픽 같은 것들을 경험하려면 제대로된 분산시스템을 구축할 필요가 있다?
러닝 커브가 추가로 발생하므로 이런 것을 aws에 맞기는 서버리스 방식이 존재한다.
ecs fargate를 구축하게 되면 이런 것들을 고민하지 않아도 된다.

4가지를 살펴본다.

aws cli
> aws 접속 가능한 cli?

aws ecr
> elastic cloud registry.
> docker image와 유사하다. 배포를 위해 사용하는 스냅샷 내지는 이미지 개념이다.

aws ecs
> elastic cloud service
> aws에 컨테이너 확장, 로드 밸런싱, 네트워킹 등을 관리하도록 한다.
> 개발자는 더 편하게 클라우드 서비스 이용 가능.

aws cloud-formation
> ecs를 모니터링할 수 있다.

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html
