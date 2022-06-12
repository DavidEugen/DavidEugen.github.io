# 맥 적응기

## 세팅하기

맥 개발환경등은 [subicura 블로그][맥 안내서] 를 참고하여 진행하였다. *정말 정리 잘되어 있다 ㅠ갬동의 눙물*

특히 iterm2를 커스트마이징 [맥 개발환경 구축하기][맥 개발환경 구축하기]을 통해 개발환경을 세팅하는 부분이 참고할만 하다. *이런맛인가? 개발자들이 맥으로 개발하려는게..*

또한 맥은 brew로 패키지 관리를 한다. 홈브루는 루비 기반으로 만들어진 Mac OS X 용 패키지 관리자이다. [sukvvon 블로그][홈브루 명령어] 를 참조하면 좋을 것 같다. 홈브루에서 패키지는 formula라고 부르며, Formulae라는 패키지 브라우저를 통해 브라우저를 검색한다. 홈브루에서 cask 명령어들이 자주 나오는데 이는 홈브루의 확장으로 맥의 GUI 프로그램도 커맨드로 설치할 수 있게 도와주는 프로그램이다.

개발을 하다보면 jdk 버전을 따로 관리하는게 편할때가 있다. 이를 도와주는것이 jenv. [jenv 설치][jenv 설치]방법을 통해 jenv를 설치해 보자.

ISSUE

```jenv local 1.8.0.292``` 명령어로 자바 버전을 설정 했는데도 안바뀐다.

## 맥을 사용하면서...

### 장점!!

일주일 가량 사용해본 후일담을 풀어보자면, 일단 unix 기반의 CLI로 개발을 할 수 있어 좋다. brew등 패키지 매니저를 자유롭게 이용 할 수 있어 좋은것 같다. 위의 블로그를 통해 처음 세팅한 Iterm를 본다면 개발하는데 아기자기한 재미가 있다. 좋다. 

### 하지만 이제 단점. 

이건 아직 맥에 익숙해 져 있지 않기 때문에 발생하는 문제들이다. 단축키들이 모두 윈도우에 익숙해 져 있어 무의식중에 손가락들이 윈도우 단축키 포지션을 기억하고 움직이고 있다. 조금 지나면 나아지겠지만, 컨트롤과 윈도우 키를 사용했던 상황에서 command 키를 자주 사용하는 맥의 단축키를 새로 익히며 적응하기에 아직은 버벅임이 있다.

특히나 적응안되는것은 capslock 키를 이용한 한/영 전환. 다른 키로 매핑해서 쓸 수 있지만, 일단은 맥의 키로 적응해 보려 한다.



## 블로그

### 하고싶은 것

- 많은 시간 투자 하지 않고 ( 본질에 집중 )
- MD 파일로 TIL (Today I Learned) 작성 

### 현재까지 상황

github 블로그 작성 도구 : typora with Iterm2

### 추가로 고민해야 할 것들

- 계속해서 Github + Jekyll 로 진행 할 것인가?
- velog 나 medium 으로 쓰는게 나을것인가?
- *이미지*!!!!!!!! 



# 참고 

---
[맥 개발환경 구축하기]: https://subicura.com/2017/11/22/mac-os-development-environment-setup.html

[맥 안내서]: https://subicura.com/mac/guide/

[맥 기본 시스템 설정]: https://subicura.com/mac/setup/#dock-menu-bar

[홈브루 명령어]: https://sukvvon.tistory.com/7
[homebrew cask]: https://tagilog.tistory.com/576
[jenv 설치]: https://nesoy.github.io/articles/2019-07/jenv
[brew 로 java 설치]: https://llighter.github.io/install-java-on-mac/
