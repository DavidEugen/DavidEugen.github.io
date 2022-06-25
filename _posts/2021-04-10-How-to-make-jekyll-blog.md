---
layout: post
title:  "Jekyll Install"
date:   2021-04-10 03:05:53 +0900
categories: jekyll update
---

# Jekyll Install
* github page 는 일종의 정적 웹 서비스

## 1. 루비 인스톨러 for windows
download 위치 : https://rubyinstaller.org/downloads/   
dev kit 다운로드, 관리자 권한 설치.   
C:\ 에 설치

## 2. git pages 설치
https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll

### 2.1. git remote repository 생성

### 2.2. 로컬 git 디렉터리 생성 
```
cd PARENT-FOLDER
git init REPOSITORY-NAME
cd REPOSITORY-NAME

git remote add origin https://github.com/DavidEugen/DavidEugen.github.io.git
git pull origin main
```

### 2.3. jekyll 생성
```
jekyll new .
```

### 2.4. gemfile 수정
#### 2.4.1. jekyll 주석처리
``` gem "jekyll" ``` -> ```#gem "jekyll" ```

#### 2.4.2. github-pages gem 등록  GITHUB-PAGES-VERSION => 해당 버전
``` gem "github-pages", "~> GITHUB-PAGES-VERSION", group: :jekyll_plugins ```

### 2.5 번들 update
```bundle update```

### 2.6 로컬 테스트
```
bundle exec jekyll serve
```

### 2.7. git 반영 
```
git remote add origin https://github.com/USER/REPOSITORY.git

git push -u origin BRANCH
```



### GitHub과 로컬의 환경 차이 이슈



참고

---

[1]: https://fuzzysound.github.io/github-and-jekyll	"깃헙과 로컬의 환경 차이 이슈"
