---
layout: post
title:  "Mac Os에서 도커 설치하기"
date:   2022-10-01 10:43:53 +0900
categories: jekyll update

---

# Mac Os 에서 Docker 설치하기



## 개요

### 일단 맥에서 돌려보자

Window 상에서는 OS 환경이 맞지 않으므로 Docker Desktop을 사용하는 것은 알 고 있어다. Mac은 Unix 계열이니깐 Docker가 바로 실행되지 않을까 생각했던것은 오산이었다. Mac은 그냥 Apple 계열인것-_-;;

Mac 또한 Docker Desktop이 필요하다.

https://www.docker.com/products/docker-desktop/

https://docs.docker.com/desktop/install/mac-install/

위의 문서를 따라 설치하면 설치가 완료되고 기본적인 명령어 까지 실행해 볼 수 있다.

### 뭔가 찜찜하다..

그런데 가만히 생각해 보면 Docker Desktop 은 Mac Os의 리소스(커널?)을 공유하는 것이 아닌가? 설정이 어떻게 바뀌고 어떤 충돌을 야기할지...(~~얘기치 못한 상황들은 당황스럽지 않다.. 다만 시간이 겁나 들뿐...ㅠ~~). Mac Os 자체를 Server로 사용할 것이 아니라 학습하기 위한 용도이므로 SandBox 개념으로 가상화가 필요하다. 

그래서 짜잔~. 지금껏 윈도우에서 함께 해왔던 VitualBox!! 

... 는 개뿔.. Mac Silicon이다.. (~~실리콘 맥으로 넘어와서 느끼는 거지만 window와 세계관이 마이 달라~~).. 결국 Parallels를 찾고 있다...(~~지갑아 어딨니...ㅠ~~)

Parallels에서 Ubuntu를 설치하고 본격적인 Docker 놀이터로..

## Docker 설치

일단 관리자 계정으로 전환해 주자. 실제 운영이 아니므로 편하다.

``` shell
sudo -i
```

docker 를 설치해 주자.

``` bash
apt intall docker.io
```

설치가 완료되고 ``` docker ``` 라 입력하면 docker에서 쓸 수 있는 명령어를 조회 하며 정상적인 도커설치를 확인 할 수 있다.



## Docker 맛보기

일단 무작정 따라해 보자. 

이미지를 찾아보자.

```bash
docker search [이미지명]
```

검색을 통해 찾은 정확한 이미지 명을 복사해서 다음을 넣어준다.

``` bash
docker run -d -p <Host Port>:<Container Port> --name [내가 지정할 컨테이너 명] [찾은 이미지 명]

```

여기서 ```-d```는 detached 모드로 backround로 작업한다는 의미이다.

상세한 run 명령어는 https://docs.docker.com/engine/reference/run/ 에서 참고 할 수 있다.

현재 실행중인 프로세스를 확인해 보자.

``` bash
docker ps
```

tomcat이 정상적으로 떠 있는 것을 확인 했다면 브라우저에서 ``` localhost:8080```으로 tomcat이 떴는지 확인해 보자.



이제 Helloworld를 마쳤다.

## Docker의 기본 개념



> OS 같은 불변영역(Immutable infrastructure)와 application ,source code, system tool, library등 서버에 설치하는 서비스 운영 환경(서비스 인프라)을 분리하여 서비스 환경 부분을 배포한 뒤 가급적 변경하지 않고 사용하는 것을 의미. 서비스 환경의 업데이트는 시스템 환경 자체를 변경하는 것이 아닌 Image를 교체하는 방식으로 이루어 진다. 



## Docker의 기본 구성

<img src="https://user-images.githubusercontent.com/10527294/193436618-b8e7694e-4d95-4e5c-85f5-cd762710109b.png" alt="Docker Basic Taxonomy" style="zoom:50%;" />

### Docker Container

Docker Image를 실행한 상태로 캡슐화된 격리된 공간에서 프로세스를 동작시키는 기술이다.

### Docker Image

서비스 운영에 필요한 여러 프로그램, 소스코드 및 라이브러리, 실행 파일들을 묶은 형태. 설정 환경들이 패키징 되어 있다.

### Docker Registry

Docker Image들을 저장여 관리하는 장소.



## Docker의 라이프 사이클

<img src="https://user-images.githubusercontent.com/10527294/193436727-d87a228b-ea50-4b4f-88b3-7ee565844c31.png" alt="Docker의 LifeCyle" style="zoom:50%;" />

### Docker Image 관리

``` pull ``` 로 Registry 에서 Image 를 가져 올 수 있으며, Image를 ```push```를 통해 Registry에 저장 할 수 있다. 받은 Image는 ``` rmi``` 를 통해 삭제 할 수 있다.

### Docker Conatiner 관리

``` create``` 를 통해 conatiner를 생성 할 수 있으며 ``` commit```을 통해 변경된 layer를 저장 할 수 있다. ``` start ``` 로 Container를 실행하며  ```stop```으로 실행된 Container를 종료할 수 있다. ``` rm``` 으로 Conatiner를 삭제 할 수 있다. 

``` create``` 와 ```start``` 를 ```run``` 명령어를 통해 동시에 실행 할 수 있으며, 기존 Container가 존재시 ```start``` 뿐 아니라 ```create```도 함께 실행 되어 Conainer가 계속 생성되므로 ```create``` 가 필요한 상황에서 ```run```을 사용도록 주의해야 한다. 만약 Image가 존재하지 않는다면 ```pull``` 기능까지 함께 수행하게 된다.



