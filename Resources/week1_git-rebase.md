# Rebase

Git에서 한 브랜치에서 다른 브랜치로 합치는 방법으로는 두 가지가 있다. 하나는 Merge 이고 다른 하나는 Rebase 다.
Rebase가 무엇인지, 어떻게 사용하는지, 좋은 점은 뭐고, 어떤 상황에서 사용하고 어떤 상황에서 사용하지 말아야 하는지 알아보도록 하자.

## rebase 기초


![[rebase-case01.png]]
*두 개의 브랜치로 나누어진 커밋 히스토리*

이 두 브랜치를 합치는 가장 쉬운 방법은 merge를 사용하는 것이다.
두 브랜치의 마지막 커밋 두 개 ``C3``, ``C4``와 공통조상 ``C2``를 사용하는 3-way Merge로 새로운 커밋을 만들어 낸다.
***화살표 방향에 주의하자! C0가 루트 커밋이고, 각 커밋간 엣지는 child to parent를 나타낸다.***

![[rebase-case02.png]]

*나뉜 브랜치를 Merge 하기*

> 3-way Merge에 관한 글은 [이 글](https://git-scm.com/book/ko/v2/Git-%eb%b8%8c%eb%9e%9c%ec%b9%98-%eb%b8%8c%eb%9e%9c%ec%b9%98%ec%99%80-Merge-%ec%9d%98-%ea%b8%b0%ec%b4%88)을 참고하자.


Merge와 비슷한 결과를 만드는 다른 방식으로, `C4` 에서 변경된 사항을 Patch로 만들고 이를 다시 `C3` 에 적용시키는 방법이 있다. Git에서는 이런 방식을 **_Rebase_** 라고 한다.
이를 통해 한 브랜치에서 변경된 사항을 다른 브랜치에 적용할 수 있다.

위의 예제는 아래와 같은 명령으로 Rebase 한다.

```console
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

실제로 일어나는 일을 설명하자면 일단 두 브랜치가 나뉘기 전인 공통 커밋으로 이동하고 나서 그 커밋부터 지금 Checkout 한 브랜치가 가리키는 커밋까지 diff를 차례로 만들어 어딘가에 임시로 저장해 놓는다. Rebase 할 브랜치(experiment)가 합칠 브랜치(master)가 가리키는 커밋을 가리키게 하고 아까 저장해 놓았던 변경사항을 차례대로 적용한다.

![[rebase-case03.png]]

*`C4` 의 변경사항을 `C3` 에 적용하는 Rebase 과정*
그리고 나서 `master` 브랜치를 Fast-forward 시킨다.
```console
$ git checkout master
$ git merge experiment
```

![[rebase-case04.png]]

*master 브랜치를 Fast-forward시키기*

`C4'` 로 표시된 커밋에서의 내용은 Merge 예제에서 살펴본 `C5` 커밋에서의 내용과 같을 것이다. Merge 이든 Rebase 든 둘 다 합치는 관점에서는 서로 다를 게 없다. 하지만, Rebase가 **좀 더 깨끗한 히스토리**를 만든다. Rebase 한 브랜치의 Log를 살펴보면 히스토리가 선형이다. 일을 병렬로 동시에 진행해도 Rebase 하고 나면 모든 작업이 차례대로 수행된 것처럼 보인다.

### rebase의 용도?
**Rebase는 보통 리모트 브랜치에 커밋을 깔끔하게 적용하고 싶을 때 사용한다.**
아마 이렇게 Rebase 하는 리모트 브랜치는 직접 관리하는 것이 아니라 contribute 형식으로 참여하는 브랜치일 것이다.
이때 메인 프로젝트에 Patch를 보낼 준비가 되면 하는 것이 Rebase 니까 브랜치에서 하던 일을 완전히 마치고 `origin/master` 로 Rebase 한다. 이렇게 Rebase 하고 나면 프로젝트 관리자는 어떠한 통합작업도 필요 없다. 그냥 master 브랜치를 Fast-forward 시키면 된다.

Rebase를 하든지, Merge를 하든지 최종 결과물은 같고 커밋 히스토리만 다르다는 것이 중요하다. Rebase 의 경우는 브랜치의 변경사항을 순서대로 다른 브랜치에 적용하면서 합치고 Merge 의 경우는 두 브랜치의 최종 결과만을 가지고 합친다.

## rebase 활용

![[rebase-case05.png]]
*다른 토픽 브랜치에서 갈라져 나온 토픽 브랜치*

이제 좀 더 복잡한 케이스로 들어가보자.
현재 상황은 테스트가 덜 된  server 브랜치는 그대로 두고 client 브랜치만 master 로 합치려고 한다.
server 와는 아무 관련이 없는 client 커밋은 C8, C9 이다. 이 두 커밋을 master 브랜치에 적용하기 위해서 --onto 옵션을 사용하여 아래와 같은 명령을 실행한다:

```console
$ git rebase --onto master server client
```

이 명령은 `master` 브랜치부터 `server` 브랜치와 `client` 브랜치의 공통 조상까지의 커밋을 `client` 브랜치에서 없애고 싶을 때 사용한다. `client` 브랜치에서만 변경된 패치를 만들어 `master` 브랜치에서 `client` 브랜치를 기반으로 새로 만들어 적용한다.

![[rebase-case06.png]]

*다른 토픽 브랜치에서 갈라져 나온 토픽 브랜치를 Rebase 하기*

이제 master 브랜치로 돌아가서 Fast-forward 시킬 수 있다.
그 결과는 아래 그림과 같다.

```console
$ git checkout master
$ git merge client
```


![[rebase-case07.png]]


*master 브랜치를 client 브랜치 위치로 진행 시키기*


`server` 브랜치의 일이 다 끝나면 `git rebase <basebranch> <topicbranch>` 라는 명령으로 Checkout 하지 않고 바로 `server` 브랜치를 `master` 브랜치로 Rebase 할 수 있다. 이 명령은 토픽(server) 브랜치를 Checkout 하고 베이스(master) 브랜치에 Rebase 한다.

```console
$ git rebase master server
```

server 브랜치의 수정사항을 master 브랜치에 적용했다. 그 결과는 아래와 같다.

![[rebase-case08.png]]

 *master 브랜치에 server 브랜치의 수정 사항을 적용*

그리고 나서 `master` 브랜치를 Fast-forward 시킨다.

```console
$ git checkout master
$ git merge server
```

모든 것이 `master` 브랜치에 통합됐기 때문에 더 필요하지 않다면 `client` 나 `server` 브랜치는 삭제해도 된다. 브랜치를 삭제해도 커밋 히스토리는 다음 그림과 같이 여전히 남아 있다.

```console
$ git branch -d client
$ git branch -d server
```

![[rebase-case09.png]]
*최종 커밋 히스토리*

## Rebase 의 위험성

Rebase에 대한 주의사항은 아래 한 문장으로 표현된다.

> **이미 공개 저장소에 Push 한 커밋을 Rebase 하지 마라**

이 지침만 지키면 Rebase를 하는 데 문제 될 게 없다. 하지만, 이 주의사항을 지키지 않으면 공개적인 비난을 받게 될 것이다.

Rebase는 기존의 커밋을 그대로 사용하는 것이 아니라 **내용은 같지만 다른 커밋을 새로 만든다**. 새 커밋을 서버에 Push 하고 동료 중 누군가가 그 커밋을 Pull 해서 작업을 한다고 하자. 그런데 그 커밋을 `git rebase` 로 바꿔서 Push 해버리면 동료가 다시 Push 했을 때 동료는 다시 Merge 해야 한다. 그리고 동료가 다시 Merge 한 내용을 Pull 하면 내 코드는 정말 엉망이 된다!!

이미 공개 저장소에 Push 한 커밋을 Rebase 하면 어떤 결과가 초래되는지 예제를 통해 알아보자. 중앙 저장소에서 Clone 하고 일부 수정을 하면 커밋 히스토리는 아래와 같다.

![[rebase-case10.png]]
*저장소를 Clone 하고 일부 수정함*

이제 팀원 중 누군가 커밋, Merge 하고 나서 서버에 Push 한다. 이 리모트 브랜치를 Fetch, Merge 하면 히스토리는 아래와 같이 된다.

![[rebase-case11.png]]###### *Fetch 한 후 Merge 함*

그런데 Push 했던 팀원은 Merge 한 일을 되돌리고 다시 Rebase 한다. 서버의 히스토리를 새로 덮어씌우려면 `git push --force` 명령을 사용해야 한다. 이후에 저장소에서 Fetch 하고 나면 아래 그림과 같은 상태가 된다.


![[rebase-case12.png]]
###### *한 팀원이 다른 팀원이 의존하는 커밋을 없애고 Rebase 한 커밋을 다시 Push 함*

자 이렇게 되면 짬뽕이 된다. `git pull` 로 서버의 내용을 가져와서 Merge 하면 같은 내용의 수정사항을 포함한 Merge 커밋이 아래와 같이 만들어진다.

![[rebase-case13.png]]
###### *같은 Merge를 다시 한다*

`git log` 로 히스토리를 확인해보면 저자, 커밋 날짜, 메시지가 같은 커밋이 두 개 있다(``C4``, ``C4'``). 이렇게 되면 혼란스럽다. 게다가 이 히스토리를 서버에 Push 하면 같은 커밋이 두 개 있기 때문에 다른 사람들도 혼란스러워한다. `C4` 와 `C6` 는 포함되지 말았어야 할 커밋이다. 애초에 서버로 데이터를 보내기 전에 Rebase로 커밋을 정리했어야 했다.

### Rebase 한 것을 다시 Rebase 하기

만약 이런 상황에 빠질 때 유용한 Git 기능이 하나 있다. 어떤 팀원이 강제로 내가 한일을 덮어썼다고 하자. 그러면 내가 했던 일이 무엇이고 덮어쓴 내용이 무엇인지 알아내야 한다.

커밋 SHA 체크섬 외에도 Git은 커밋에 Patch 할 내용으로 SHA-1 체크섬을 한번 더 구한다. 이 값은 “patch-id” 라고 한다.

덮어쓴 커밋을 받아서 그 커밋을 기준으로 Rebase 할 때 Git은 원래 누가 작성한 코드인지 잘 찾아 낸다. 그래서 Patch가 원래대로 잘 적용된다.

예를 들어 앞서 살펴본 예제를 보면 [한 팀원이 다른 팀원이 의존하는 커밋을 없애고 Rebase 한 커밋을 다시 Push 함](#한-팀원이-다른-팀원이-의존하는-커밋을-없애고-rebase-한-커밋을-다시-push-함) 상황에서 Merge 하는 대신 `git rebase teamone/master` 명령을 실행하면 Git은 아래와 같은 작업을 한다.

- 현재 브랜치에만 포함된 커밋을 결정한다. (C2, C3, C4, C6, C7)
    
- Merge 커밋이 아닌 것을 결정한다. (C2, C3, C4)
    
- 이 중 merge할 브랜치에 덮어쓰이지 않은 커밋을 결정한다. (C2, C3. C4는 C4’와 동일한 Patch다)
    
- 결정한 커밋을 `teamone/master` 브랜치에 적용한다.
    

결과를 확인해보면 [같은 Merge를 다시 한다](#같은-merge를-다시-한다) 같은 결과 대신 제대로 정리된 [강제로 덮어쓴 브랜치에 Rebase 하기](#강제로-덮어쓴-브랜치에-rebase-하기) 같은 결과를 얻을 수 있다.


![[rebase-case14.png]]
###### *강제로 덮어쓴 브랜치에 Rebase 하기*

동료가 생성했던 C4와 C4' 커밋 내용이 완전히 같을 때만 이렇게 동작된다. 커밋 내용이 아예 다르거나 비슷하다면 커밋이 두 개 생긴다(같은 내용이 두 번 커밋될 수 있기 때문에 깔끔하지 않다).

`git pull` 명령을 실행할 때 옵션을 붙여서 `git pull --rebase` 로 Rebase 할 수도 있다. 물론 `git fetch` 와 `git rebase teamone/master` 이 두 명령을 직접 순서대로 실행해도 된다.

`git pull` 명령을 실행할 때 기본적으로 `--rebase` 옵션이 적용되도록 `pull.rebase` 설정을 추가할 수 있다. `git config --global pull.rebase true` 명령으로 추가한다.

Push 하기 전에 정리하려고 Rebase 하는 것은 괜찮다. 또 절대 공개하지 않고 혼자 Rebase 하는 경우도 괜찮다. 하지만, 이미 공개하여 사람들이 사용하는 커밋을 Rebase 하면 틀림없이 문제가 생긴다.

나중에 후회하지 말고 `git pull --rebase` 로 문제를 미리 방지할 수 있다는 것을 같이 작업하는 동료와 모두 함께 공유하기 바란다.

### Rebase vs. Merge

지금까지 Rebase에 대해 알아보았다. 그렇다면 Rebase와 Merge중 무엇이 더 좋을까?
지저분한 Merge 커밋이 커밋 히스토리에 남아있다면 문제가 없을까? Rebase를 사용하게 된다면, 그 기록이 훼손되는 것은 아닌가? 프로젝트 진행 흐름을 다른 사람에게 설명하기 쉬울까? 등 모든 팀과 모든 이가 처한 상황은 모두 다르기 때문에 각자의 상황과 각자의 판단에 달렸다.

일반적인 해답의 경우 앞서 언급했듯이, 로컬 브랜치에서 작업할 때는 히스토리를 정리하기 위해서 Rebase 할 수도 있지만, 리모트 등 어딘가에 Push로 내보낸 커밋에 대해서는 절대 Rebase 하지 말아야 한다.


## ``referenced``
- [3.6 Git 브랜치 - Rebase 하기](https://git-scm.com/book/ko/v2/Git-%eb%b8%8c%eb%9e%9c%ec%b9%98-Rebase-%ed%95%98%ea%b8%b0)

## ``related``
- 3.2 Git 브랜치 - 브랜치와 Merge 의 기초
- 7.8 Git 도구 - 고급 Merge 