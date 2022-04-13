# GIT Ignore 파일 사후 적용

Git Ignore에 추가된 부분 차후 적용시 Git에서 삭제하는 방법



 ```git rm <fileName>```
 작업 디렉토리와 스테이징 영역에서 파일을 삭제
 
 ```git rm --cached <fileName>```
 스테이징 영역에서만 파일이 삭제. 작업 디렉토리나 저장소에서는 삭제되지 않음

```linux
$ git rm --cached -r .
//현재 디렉토리 기준 스테이징 된 파일을 모두 삭제 (-r 안붙이면 에러 발생 )

$ git add .
//현재 디렉토리 (.gitignore가 적용된) 작업트리를 새로 스테이징

```
