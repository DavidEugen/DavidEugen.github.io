---
layout: post
title:  "Docker 알아두면 좋은 명령어"
date:   2022-10-03 11:43:53 +0900
categories: jekyll update

---

# Docker 알아두면 좋은 명령어



### Docker Image 정보 확인

```bash
sudo docker inspect [이미지]
```

컨테이너에 필요한 포트 정보등을 알 수 있다.

### Docker 정보 확인

``` bash
sudo docker info 
```

Docker 파일 시스템 위치등을 알 수 있다.



### Docker Container 내부 셸 실행

```bash
sudo docker exec -it [컨테이너명] /bin/bash
```

```-i --interactive ``` Keep STDIN open even if not attached

 ```-t --tty``` Allocate a pseudo-TTY

### Host와 Docker Container 간 파일 복사

```bash
sudo docker cp <path> <to container>:<path>
sudo docker cp <from container>:<path> <path>
```



### Docker Container 모두 삭제

``` bash
sudo docker stop `sudo docker ps -a -q`
sudo docker rm `sudo docker ps -a -q`
```

`` 을 이용해 결과를 변수로 사용 할 수 있다.

```-q``` 는 ID 만 조회 한다.



### 임시 Docker Container 생성

``` bash
sudo docker run -d -p 80:8080 --rm --name [container 명] [이미지명] 
```

```--rm``` 을 통해 stop시 컨테이너가 삭제된다.



### Docker Container Log 확인

```bash
sudo docker logs [container 명] # stdout, stderr
```





## Docker 실행 예제

### Docker Container 환경변수 설정 - MySql Container 실행

Docker Hub에 가면 상세 사용법이 나와 있다. https://hub.docker.com/_/mysq

``` bash
sudo docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

``` -e```  환경 변수를 입력할때 옵션

환경 변수 값은 ```'```(Singl Quotation)을 사용하여 넣어준다. 아니면 명령어로 인식하여 오류가 날 수 있다.

### MySql Container Bash 실행

``` bash
sudo docker exec -it mysql bash
```

```printenv```를 통해 환경 변수를 확인해 보자..

```bash
printenv # 환경변수 전체를 보여준다.
printenv env_name #env_name이라는 환경 변수를 찾는다.
echo $env_name #env_name이라는 환경 변수를 찾는다.

```



### Docker Conatiner 볼륨 마운트 설정

``` bash
docker run -v <host 경로>:<Container 경로>:<권한> 
```









  
