VSCode 생산성 향상을 위한 팁들

# 유용한 Extension들

## Extension이 뭘까?
기본적으로 VSCode는 소스 코드 편집기이다. (IDE가 아니다. IDE라고 볼 수도 있긴 한데 엄밀히 따지면 코드 편집기에 가깝다고 한다.)
개발에 있어서 다른 앱과 연동하거나, 기존에 VSCode에서 실행 못하는 파일들을 실행할 수 있게 되거나, 문법을 자동완성 시켜주는 등 더 개발하기 편하게 만들어주는 것들이다.

MS나 GitHub같은 곳이 공식적으로 배포하거나, 개인이 배포하는 경우도 있다.
이런 도구들은 강력한 기능을 제공하기도 하니, 편의성을 위해 여러가지 찾아보는 것도 좋다.

## 개발에 도움이 되는 Extension들

1. indent-rainbow
   들여쓰기를 4가지 색상으로 번갈아 표시해준다. 주로 python에서 유용하다.
2. git graph
   리포지토리 브랜치를 가시화 시켜주는 확장이다.
   개인적으로 브랜치 관리할 때 편리했다.
3. Rainbow CSV
   CSV 파일의 각 컬럼을 색으로 강조표기한다.
   색상이 생겨서 가시성이 좋아진다.
4. Excel Viewer
   xlsx와 같은 파일들을 볼 수 있게 된다.
5. SQL Database Projects
   MS에서 공식으로 배포한 확장.
   SQL 기반 DB 프로젝트를 할 때 좋다.


referenced by
- [Visual Studio Code 확장 사용](https://learn.microsoft.com/ko-kr/power-pages/configure/vs-code-extension)
- [개발을 도와주는 vscode Extension](https://medium.com/@supersexy/vscode-extension-d00a8d2a0409)

# VSCode & GitHub & Git 연동하기
vscode는 좋다. github도 좋다!
이들을 합치면 어떻게 될까? 아주아주 좋다!
vscode는 git 사용을 위한 gui를 지원하기 때문에 매우 편하게 Git을 사용할 수 있다. 또한, GitHub로의 계정 연동을 지원하여 원격 리포지토리와의 연결도 놀랍도록 간편하다!
사실 대부분의 IDE는 GitHub 연동을 지원한다. 이 방법을 잘 배워 둔다면 다른 IDE에서도 활용 가능하다.

## VSCode - GitHub 계정 연동
### 계정 연동 전 체크
1. VSCode를 실행한다.
2. 좌측하단 사이드바의 환경설정 위 계정 버튼을 클릭한다.
3. VSCode가 이미 Github에 로그인 되어 있는지 확인한다.
	1. 로그인 되어있다면 사용하려는 Github ID가 맞는지 확인하고, 다른 아이디라면 로그아웃하자.

### VSCode Github Extension 설치
GitHub Pull Requests and Issues 는 VSCode 안에서 GitHub의 Pull Requests(이하 PR)와 Issues를 작성하고 관리할 수 있도록 제공하는 확장프로그램이다. GitHub에 로그인해야 사용할 수 있다.
[Github Pull Requests](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github)

로컬 저장소(repository) 준비하기
GitHub Pull Requests and Issues 확장프로그램을 사용하기 위해서는 VSCode에 Git 저장소가 열려있어야 한다. 기존 로컬 저장소를 열거나, 생성하도록 하자.
생성하는 과정은 생략한다!

### VSCode에서 GitHub 로그인하기
확장프로그램을 이용해 GitHub에 로그인
GitHub Pull Requests and Issues 확장프로그램을 이용해 GitHub에 로그인하자.
왼쪽 사이드바 탭을 유심히 살펴본다면, 전에 없던 아이콘이 생긴 것을 볼 수 있다!
로그인 버튼을 클릭하면 로그인할 수 있다.
![[how-to-login.png]]


referenced by
- [vscode - github 로그인하기](https://usingu.co.kr/frontend/vscode/vscode-github-%EB%A1%9C%EA%B7%B8%EC%9D%B8%ED%95%98%EA%B8%B0/)

# Windows에서 리눅스 환경 사용하기
## Windows에서 왜 굳이 리눅스 환경을 이용하는가?
Windows는 널리 알려진 OS이지만 개발을 하기 어려운 경우가 많다.
서버로 사용하기에는 보안이 영 좋지 못하기 때문에 실무에서는 리눅스를 사용하기 때문인데, 이 때문에 리눅스 환경에서의 개발이 좀 더 편하다. 커뮤니티도 더 잘 되어 있다.

또한 Windows에서는 사용하지 못하는 것들도 있다. 대표적으로 CUDA가 있고... python에서는 celery라는 패키지는 windows 운영체제를 아예 공식지원하지 않는 등 여러가지 불편한 점이 많다.
리눅스는 Windows와 다르게 오픈 소스라는 것도 한 몫한다. 상용 소프트웨어를 만들기 자유롭다는 점 때문인지 기업에서도 많이 쓰기 때문에...
그 외에도 다양한 이유들이 있으나 혼자 찾아보는 것도 재미있을 것이다!

## 어떻게 사용하는가?
WSL을 기본적으로 설치해야 한다.

### WSL?
WSL은 Windows Subsystem for Windows의 약자로, 기존 가상 머신의 무거움이나 귀찮은 설정없이 Windows에서 직접 Linux를 실행할 수 있게 해주는 유용한 도구이다. WSL을 이용하면 Ubuntu, OpenSUSE, Kali, Debian, Arch Linux 등의 Linux 배포판을 설치하고 사용할 수 있다.

## 어떻게 설치할까?

wsl은 다음 명령어를 통해 설치한다.

```
wsl --install
```

추가로 Microsoft Store에서 Ubuntu 18, 20, 22 등 다양한 운영체제를 가져오거나, 직접 [공식 페이지](https://releases.ubuntu.com/focal/) 를 통해 이미지를 가져와 설치하는 방법이 있다.
보통은 우분투 운영체제를 많이 사용한다.

운영체제 설치가 완료되었다면, 이것을 가상 머신으로써 동작시킬 수 있다.
이렇게 설치된 운영체제는 vscode에서도 인식이 가능하다. Remote Development 확장에서 해당 운영체제로 접속하여 로컬에서 우분투 운영체제를 통한 개발이 가능해진다.

단점으로는 우분투 운영체제 특성 상 파일이 많아지면 램을 많이 먹는 단점이 있다. 또한 가상 머신이기 때문에 사용하기 복잡하다. windows 기반 가상머신보다 그냥 linux os를 설치하는 것이 더 좋을 수도 있다.

본인의 상황에 따라 이런 방법을 사용해보는 것도 나쁘지 않다. 시도해보자!

reference by
- [Windows보다 Linux가 좋은 이유?](https://velog.io/@ring-h/Windows%EB%B3%B4%EB%8B%A4-Linux%EA%B0%80-%EC%A2%8B%EC%9D%80-%EC%9D%B4%EC%9C%A0)
- [Windows에서 Linux 사용하기 (with WSL2)](https://velog.io/@jskim/Windows%EC%97%90%EC%84%9C-Linux-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-with-WSL2)



