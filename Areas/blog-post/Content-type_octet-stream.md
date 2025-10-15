
Http 헤더에서 Content type은 해당 요청의 데이터 타입을 나타낸다.

Content type은 다양한 타입을 나타내는데, 텍스트, 응용 데이터, 멀티파트, 이미지, 오디오/비디오 등 다양한 데이터 형식을 포함한다.

이중 우리는 octet-stream이라는 application 유형의 타입을 볼 수 있다.
바이너리 데이터를 전송할 때 사용하는 것인데, 왜 바이너리임에도 하필 octet이라는 표기를 했을까?

이쯤에서 바이트의 단위를 생각해볼 필요가 있다.
바이트는 8비트로 이루어진다는 사실을 모두 기억할 것이다.
그렇다면 바이트인데 왜 octet이라는 불필요한 표기를 한 것인가?

초기 컴퓨터 구조에서 바이트는 그 단위가 서로 달랐기 때문이다.
+워드 길이도 달랐다. 요즘은 32, 64 단위로 고정된다!

CDC 6600 : 12비트 단위 지칭에 사용
PDP 10 : 가변 길이 바이트 (1~36비트)
IBM 7030 stretch : 가변 길이 바이트 (1~8비트)

그래서 바이트 스트림임에도 불구하고 8비트 단위를 고정하여 사용하는 것이다...

Reference
https://en.wikipedia.org/wiki/CDC_6600

https://en.wikipedia.org/wiki/IBM_7030_Stretch

https://en.wikipedia.org/wiki/PDP-10

https://pjs-world.tistory.com/entry/HTTP-Content-Type-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%A2%85%EB%A5%98

https://www.quora.com/Why-do-the-various-RFCs-use-the-term-octet-instead-of-byte