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
sudo docker stop `sudo docker ps -a -q` # sudo docker stop $(sudo docker ps -a -q) 동일
sudo docker rm `sudo docker ps -a -q` # sudo docker rm $(sudo docker ps -a -q) 동일
```

```-q``` 는 ID 만 조회 한다.2



### 임시 Docker Container 생성

``` bash
sudo docker run -d -p 80:8080 --rm --name [container 명] [이미지명] 
```

```--rm``` 을 통해 stop시 컨테이너가 삭제된다.



### Docker Container Log 확인

```bash
sudo docker logs [container 명] # stdout, stderr
```







  
