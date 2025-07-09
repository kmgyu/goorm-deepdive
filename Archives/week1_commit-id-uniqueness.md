커밋 id의 앞자리 7개 문자만으로도 유일성 보장이 가능하다기에 궁금하여 찾게 되었다.

우선 각 리포지토리에 대해서는 유일하지만, Github 전체를 놓고 본다면 극히 낮은 확률로 일치한다.
만약 앞자리 N개로 제한할 경우 짧게 제한할 수록 일치할 확률은 높아질 것으로 보인다.
깃은 sha-256 알고리즘을 이용해 메시지 다이제스트* (메시지 압축 함수, 해싱 알고리즘을 이와 같이 표현하는 것으로 보임. 새로 알았다!)를 표현하는데 이에 따라서 앞자리 N개로 제한하더라도 7개 정도라면 리포지토리 내에서는 일치할 확률이 없을 것으로 보인다.

referenced
[커밋 아이디(해쉬)의 고유성은 각 레포지토리 안에서만 보장되나요?](https://www.codeit.kr/community/questions/UXVlc3Rpb246NjFhOTk4OThmZTEyMjg1ZWI2Zjc0YTZh)
[각 git 커밋 ID는 고유한가요, 아니면 해당 저장소에 대해서만 고유함이 보장되나요?](https://www.quora.com/Is-each-git-commit-ID-unique-or-guaranteed-to-be-unique-only-for-that-repo)
[Git에서 커밋 ID를 Hash로 관리하는 이유](https://velog.io/@buna1592/Git%EC%97%90%EC%84%9C-%EC%BB%A4%EB%B0%8B-ID%EB%A5%BC-Hash%EB%A1%9C-%EA%B4%80%EB%A6%AC%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)