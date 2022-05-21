# Git에서 자주 일어나는 이슈 정리

## [ ] conflict  case 확인 필요

================================

## case1.

### 현상

remote repository 삭제 이후 동일 이름의 repository 바로 생성하고 기존 project 연결시

기존 git history가 그대로 남아 있음.

### 도출 과정

초기 
```
git init
git add .
git commit -m ""
git remote add origin [git 주소]
git branch -M main
git push -u origin main
```

입력시 기존 remote repository가 존재하는 것으로 나옴

```
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/DavidEugen/study-spring-core.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```




## Git 자주 쓰는 명령어들

### Clone

```shell
$ git clone
```

원격 repository에서 가져온다.



### 원격 저장소 지정

```shell
$ git remote add origin <remote-url>
```

해당 프로젝트를 외부 원격 저장소인 remote-url 을 지정. 이때 origin은 일종의 Alias 이며중심이 되는 프로젝트로 암묵적으로 origin으로 합의.


### 원격 저장소 확인

```shell
$ git remote -v
```

등록된 원격 저장소들을 확인 할 수 있다.

### Push

```shell
$ git push -u origin <branch name>
```

해당 프로젝트에서 커밋한 사항들을 origin으로 push한다. local repository의 main 브랜치가 remote repository의 main 브랜치를 바라보게 하는 명령어. -u 는 --set-upstream으로 최초 명령어 실행 이후 이를 기억해 git push, git pull만 해도 된다.

### 브랜치 이동

```shell
$ git checkout <branch name>
```

다른 branch로 head를 이동 이시킨다.







---

참고

https://parksb.github.io/article/28.html
