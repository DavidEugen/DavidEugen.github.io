

# Intellij Tip

|내용|인텔리제이 Mac| 이클립스 Win ||
| :--- | :--- | :--- | ---- |
|문장 자동 완성|cmd + shift + enter|||
|지역 변수 할당|cmd + opt + V| ||
|파일 패키지 등..새로 생성|cmd + N|ctrl + n|파일 탐색창에서|
|contructor, getter,setter..새로 생성|cmd + N, ctrl + enter|alt + insert|Editor 창에서|
|코드 어시스턴트 ( Show Context Actions)|opt + enter|ctrl + 1||
|소스정리 (Reformat Code)|cmd + opt + L|ctrl + shift + F||
|자동 임포트 정리(Optimize Imports)|ctrl + opt + O|ctrl + shift + O||
|테스트 케이스 생성,이동|cmd + shift + T|||
|코드 줄 복사 |cmd + D|ctrl + shift + 위 아래 방향키||
|코드 줄 이동 |cmd + shift + 위 아래 방향 키<br />opt + shift + 위 아래 방향키| atl + 위 아래 방향키|cmd + shift + 위 아래 방향 키 : 문법 허용 내에서 이동 (메서드 안에서 이동)<br />opt + shift + 위 아래 방향키 : 문법 상관 없이 이동|
|커서 이전 위치 |opt + shift + 좌 우 방향키| alt + 좌 우 방향키 ||
|과거 히스토리 |cmd + E| |주의 : backsapce 누르면 해당 창 닫힘|
|리팩터링 |cmd + opt + M| alt + shift + M ||
|실행 |ctrl + shift + R|  ||

* 이클립스 윈도우 단축키는 생각나는 대로 적는것, 비교, 참고용



## Terminal 에서 바로 Intellij 실행 하기

1. 인텔리제이 실행

2. Tools - Create Command-Line Launcehr... 클릭

3. [내용 보강 필요]

   - 터미널에서 실행시킬 명령어로 스크립트를 만든다.

     > You can create a launcher script to enable opening files and projects in IntelliJ IDEA from the command line. Specify the name of the script and the path where it should be created: 

   - 기본적으로 /usr/local/bin/idea 세팅된다.

4. 터미널에서 ``` idea . ``` 으로 실행 할 수 있다. 



----

참고.

1. https://velog.io/@martinalee94/Intellij-%ED%84%B0%EB%AF%B8%EB%84%90%EC%97%90%EC%84%9C-%EC%9D%B8%ED%85%94%EB%A6%AC%EC%A0%9C%EC%9D%B4-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0mac
