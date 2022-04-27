# Intellij Issue Case

## case 1.

### 현상

인텔리제이 클릭이 안됨

### 해결

한/영 전환 키 누르면 됨

### 영구해결

#### OS 전체 적용
defaults write -g ApplePressAndHoldEnabled -bool false

#### IntelliJ 만 적용
defaults write com.jetbrains.intellij ApplePressAndHoldEnabled -bool false

#### IntelliJ(community edition) 커뮤니티 버전
defaults write com.jetbrains.intellij.ce ApplePressAndHoldEnabled -bool false

### 원인

~~어떤 동작에서 해당 현상에 빠졌는지 확인 필요. (추후 확인 필요)~~

영문키 입력이 길게 눌러져 특부문자가 뜨는 경우



---

참고

1. https://velog.io/@d-h-k/Mac환경에서-IntelliJ-키보드-마우스-입력-씹힘-문제-해결
