

### mac

##### 1. 설치
```
$ brew install jenkins
```

##### 2. 설정변경
```
$ vi /usr/local/Cellar/jenkins/2.159/homebrew.mxcl.jenkins.plist


<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>homebrew.mxcl.jenkins</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/libexec/java_home</string>
      <string>-v</string>
      <string>1.8</string>
      <string>--exec</string>
      <string>java</string>
      <string>-Dmail.smtp.starttls.enable=true</string>
      <string>-jar</string>
      <string>/usr/local/opt/jenkins/libexec/jenkins.war</string>
      <string>--httpListenAddress=127.0.0.1</string>
      <string>--httpPort=8888</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
  </dict>
</plist>
```
Port 를 8888 로 변경 했다.

##### 3. 실행
```
$ brew services start jenkins
==> Successfully started `jenkins` (label: homebrew.mxcl.jenkins)
```

##### 4. admin 페이지 접속

어드민 페이지가 열리고, admin passwd 를 입력하라고 한다
```
$ cat /Users/user/.jenkins/secrets/initialAdminPassword
2b19a4b5754448279327413524446154
```

passwd 를 입력하면 다음 화면으로

##### 5. 플러그인설치
추천 / 선택 두가지 버튼이 나오는데, 선택을 누르니, 기본적인 플러그인이 이미 선택되어 있고, 추가로 github 플러그인만 더 체크 해서 다음으로 진행 했다.

##### 6. 어드민정보를 입력 하고 나면 완료
계정명, 암호, 암호확인, 이름, 이메일
