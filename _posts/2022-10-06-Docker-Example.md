---
layout: post
title:  "Docker 실행 예제"
subtitle: "예제 컨테이너를 실행하면서 알아보는 Docker 명령어들을 학습해 봅니다."
date:   2022-10-06 14:05:53 +0900
categories: jekyll update

---

# Docker 실행 예제

예제 컨테이너를 실행하면서 알아보는 Docker 명령어들을 학습해 봅니다.

## MySql Container 실행

### Docker Container 환경변수 설정

Docker Hub에 가면 상세 사용법이 나와 있다. https://hub.docker.com/_/mysq

``` bash
sudo docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

``` -e```  환경 변수를 입력할때 옵션

환경 변수 값은 ```'```(Singl Quotation)을 사용하여 넣어준다. 아니면 명령어로 인식하여 오류가 날 수 있다.

### MySql Container Bash 실행

``` bash
sudo docker exec -it some-mysql bash
```

```printenv```를 통해 환경 변수를 확인해 보자..

```bash
printenv # 환경변수 전체를 보여준다.
printenv env_name #env_name이라는 환경 변수를 찾는다.
echo $env_name #env_name이라는 환경 변수를 찾는다.

```



### 생각해 볼 문제 -1

실습 도중 port를 잘못 기입하게 되었다. port binding 정보를 변경하려면 어떻게 해야 할까? 단순하게 포트 정보만 바꾸면 될 줄 알았는데 그런 기능은 없는것 같다. 이에 https://www.baeldung.com/linux/assign-port-docker-container 에서는 3가지 방법을 제시한다.

1. 동일 이미지로 새로운 Container launch
2. 이미지 Commit 후 새로운 Container launch
3. Docker 종료 후 Container 정보 파일 변경

 개인적 생각으로는 3.번의 경우 Docker 서비스를 중단 시켜야 하는 리스크가 있다고 본다.  



### 생각해 볼 문제 - 2 

DB 같이 영구적으로 관리되어야 할 자료를 Container로 관리하는게 맞을까? 저장해야 할 자료들은 Mount를 시켜 영속적으로 관리하되 환경 설정 정보들은 이미지로 관리되는게 맞는거 같다.





## Jupyter Notebook 설치하기

Jupyter Notebook 설정을 통해 마운트 설정방법을 알아보자.

https://hub.docker.com/r/jupyter/datascience-notebook

https://github.com/jupyter/docker-stacks



### Docker Conatiner 볼륨 마운트 설정

``` bash
docker run -v <host 경로>:<Container 경로>:<권한> 
```

`<host 경로>`에  이미 volume 이 있으면 `<volume 명>` 을 적으면 되며 이는 Volume 연결이 된다.

`<host 경로>`로 연결하는 것은 bindmount 가 된다.


``` bash
mkdir jupyternotebook
chmod 777 ./jupyternotebook
cd jupyternotebook

docker run --rm -p 8888:8888 -e JUPYTER_ENABLE_LAB=yes -v "$PWD":/home/jovyan/work:rw jupyter/datascience-notebook

```





## Docker Image 빌드하기

### application 작성

우선 빌드할 프로그램을 작성해 보자.


파이선으로 소켓을 이용해 입력한 데이터를 response해주는 프로그램이다.

``` python
  # test_server.py
  import socket
  with socket.socket() as s:
    s.bind(("0.0.0.0", 12345))
    s.listen()
    print("server is started")
    conn, addr = s.accept()
    # conn 클라이언트와 통신할 소켓
    # addr 클라이언트의 정보가 들어있음
    with conn:
      print("Connected by", addr)
      while True:
        data = conn.recv(1024)
        if not data: break
        conn.sendall(data)
```

  해당 프로그램을 테스트 해 보자...

  ``` bash
  # Terminal 1
  python3 test_server.py
  ---
  # Terminal 2
  nc 127.0.0.1 12345
  ```



### dockerfile 생성

docker file 작성

``` dockerfile
FROM python:3.7 # python 3.7 Image를 사용 하여 작성

RUN mkdir /echo 
COPY test_server.py /echo

CMD ["python", "/echo/test_server.py"] 
```

RUN 명령은 빌드시 실행하는 명령

CMD 는 컨테이너 실행시 실행하는 명령

이때 COPY 명령에서 현재 경로의 test_server.py를 복사하는 것이므로 dockerfile과 동일한 위치에 있어야 한다.



### 빌드후 테스트

``` bash
sudo docker build -t ehco_test . # sudo docker build -t <image 명> <dockerfile위치>
```

```-t``` 는 ``` --tag ``` 를 의미하며 저장소 이름, 이미지 이름 태그를 설정한다. ```<repository 명>/<image 명>:<tag 명>```

```bash
sudo docker images
sudo docker run -t -p 12345:12345 --name et --rm echo_test
```



``` bash
nc 127.0.0.1 12345
```





### Docker Image Push

``` bash
sudo docker login

sudo docker tag echo_test platonicojju/echo_test:v1 # docker tag <변경전 이미지명> <변경후 이미지명>

sudo docker images
sudo docker push platonicojju/echo_test:v1
```

Image를 push 하기 위해서 먼저 dockerhub에 login이 되어 있어야 한다.

기본적으로 이미지를 생성하면 ```:lastest```가 따라 붙는다. ``` docker tag``` 를 이용해서 Image 명 및 tag를 변경 할 수 있다. 



### Docker Private Repository

registry 라는 이미지를 통해 내부 repository를 구성할 수 있다.

``` bash
sudo docker run -d --name docker-registry -p 5000:5000 registry
```

실행 후 브라우저를 통해 ```http://127.0.0.1:5000/v2/``` 접속이 가능한지 확인한다.

``` bash
sudo docker tag platonicojju/echo_test:latest 127.0.0.1:5000/echo_test
sudo docker push 127.0.0.1:5000/echo_test
```

이미지를 private repository에 올려 본다.

```http://127.0.0.1:5000/v2/_catalog ```을 통해 resitry를 확인 할 수 있다.
