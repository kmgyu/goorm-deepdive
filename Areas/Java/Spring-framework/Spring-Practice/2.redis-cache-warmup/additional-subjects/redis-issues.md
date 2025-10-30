# 복붙하는 바람에 네트워크 상에서 node를 인식을 못한다.
cluster-announce-ip 설정 문제
해결됨.



# redis insight ERR_EMPTY_RESPONSE

8001로 진입했는데 안나온다.

살펴보니, redislab의 redis-insight는 최신버전에서 5540으로 바뀌었다.
이걸 공식 이미지에서 안알려준다.
세상에나.
안에 들어가면 등록 할 때는 도커 네트워크의 ip이름을 사용해주면 된다.
redis-node-1:6379와 같은 형식으로 작성해주었다.



# 캐싱이 안됨
그런데 이번엔 캐싱이 안된다.
cache manager 조차도 null이 된다.

의존성 문제.
cache와 redis 의존성 추가해줌.
해결.

# 이번엔 연결이 안된다.
https://msna.tistory.com/12
유사해보이는 문제. 도움은 안됌.

해결하려고 host.docker.internal 써보고... ip 가 문제인가 conf가 문제인가 열심히 테스트해보다가 해결 못했다.


# Got no slots in CLUSTER SLOTS
이번에 정상이던 세팅으로 복구했더니 다시 생겨났다.
기존에 레디스 인사이트 복구 전 생겼던 문제. 사라졌다 재발함.
원인을 모르겠다. 조건이 같은데...