---
layout: post
title:  "Docker 기본 명령어"
date:   2022-10-02 11:43:53 +0900
categories: jekyll update

---

# Docker 기본 명령어



### Docker Image 다운로드

```bash
sudo docker pull [image] # 이미지 가져오기
```

### Docker Image 조회

``` bash
sudo docker images # 이미지 조회
```

### Docker Image 삭제

```bash
sudo docker rmi [image] # 이미지 삭제
```

### Filter를 사용한 Image 삭제

``` bash
sudo docker rmi $(docker images -q --filter "reference=ngi*")
sudo docker rmi $(docker images | grep "ngi*" | awk '{print $3}')
```

[필터를 사용한 Docker Image 삭제 방법](https://velog.io/@soonbee/docker-image%EB%A5%BC-%EC%82%AD%EC%A0%9C%ED%95%98%EB%8A%94-%EB%8B%A4%EC%96%91%ED%95%9C-%EB%B0%A9%EB%B2%95%EB%93%A4) 참고

```awk '{print $3}``` 에서 ```$3```은 Image Id 가 3번째 열이므로..

 

### Docker Container 생성

```bash
sudo docker create --name [container] [image]
```

### Docker Container 시작 과 재시작

``` bash
sudo docker start [container] # 컨테이너 시작
sudo docker restart [container] # 컨테이너 재시작

sudo docker run -d --name [container] [image] # create + start
```

``` docker create``` 와 ```docker start``` 를 포함한 명령어이다.

### Docker Container 조회

``` bash
sudo docker ps # 실행중인 컨테이너 조회
sudo docker ps -a # 모든 컨테이너 조회
```

### Docker Container 중지

```bash
sudo docker stop [container] # 컨테이너 중지
```

### Docker Container 삭제

```bash
sudo docker rm [container 명]
sudo docker rm [container ID]
```

실행중인 컨테이너는 삭제 불가하다. ```stop```을 컨테이너 중지 후 삭제.

Image도 컨테이너가 사용중이라면 삭제 불가능. ```-f``` 강제 삭제 옵션을 주거나 컨테이너 사용 종료 후 삭제 가능. 강제로 Image 삭제시 Container는 남아 있다.





