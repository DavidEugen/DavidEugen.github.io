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


---

참고

https://parksb.github.io/article/28.html
