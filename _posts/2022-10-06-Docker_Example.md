# Docker 실행 예제





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

<host 경로>에  이미 volume 이 있으면 <volume 명> 을 적으면 되며 이는 Volume 연결이 된다.

<host 경로>로 연결하는 것은 bindmount 가 된다.



``` bash
mkdir jupyternotebook
chmod 777 ./jupyternotebook
cd jupyternotebook

docker run --rm -p 8888:8888 -e JUPYTER_ENABLE_LAB=yes -v "$PWD":/home/jovyan/work:rw jupyter/datascience-notebook

```





