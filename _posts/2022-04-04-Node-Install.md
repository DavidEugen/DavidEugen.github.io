# Node 설치 하기

node.js 도 다양한 버전이 존재.
**NVM(Node Version Manager)**를 통해 node.js의 버전을 쉽게 관리 할 수 있다.

node.js 홈페이지에 들어가서 각 버전을 다운로드받을 수 있지만 이 글에서는 **NVM**을 먼저 설치하고 그것을 통해 node.js 를 설치해 사용하는 법을 알아볼 것이다.

## 1. NVM (Node Version Manager) 설치

- macOS 기준
``` shell 
brew update
brew install nvm
```

설치시 다음과 같은 메세지 출력

```shell
Please note that upstream has asked us to make explicit managing
nvm via Homebrew is unsupported by them and you should check any
problems against the standard nvm install method prior to reporting.

You should create NVM's working directory if it doesn't exist:

  mkdir ~/.nvm

Add the following to ~/.zshrc or your desired shell
configuration file:

  export NVM_DIR="$HOME/.nvm"
  [ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"  # This loads nvm
  [ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion

You can set $NVM_DIR to any location, but leaving it unchanged from
/opt/homebrew/opt/nvm will destroy any nvm-installed Node installations
upon upgrade/reinstall.
```

위의 안내대로 ```mkdir ~/.nvm``` 로 생성 후 ```vi ~/.zshrc``` 을 통해 상단 내용 작성후
```source ~/.zshrc``` 로 적용


다시 ```nvm --version``` 을 통해 확인해 본다.

## 2. node 설치

nvm 리스트를 싱행해 본다.

```shell

nvm ls                                                                               				N/A
iojs -> N/A (default)
node -> stable (-> N/A) (default)
unstable -> N/A (default)


```

lts 버전을 설치 할 수 있다

``` shell
nvm install --lts
nvm ls
```
사용을 원하는 node 버전을 ``` nvm install <버전>``` 을 통해 설치 할 수 있다.

## 3. NVM 명령어들
``` shell
# 설치된 node.js 목록 확인하기
nvm ls

# node.js 버전별 설치
nvm install v14 # 버전 명시. version-like표시들 모두 가능
nvm install node # 최신 버전 설치
nvm install --lts # lts 버전 설치

# 특정 버전 node 사용하기
nvm use <version>

# 현재 사용중인 버전 확인
nvm current
 
```


----

참고.

1. nvm 설치 : https://velog.io/@mayinjanuary/NVM-%EC%9D%B4%EB%9E%80-%EB%85%B8%EB%93%9CNode.js-%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC%ED%95%98%EB%8A%94-%EB%B2%95
