## 기본 설치  
내용을 다루기 전에 기초적으로 설치해야 할 것들.  

### WSL (WSL2)  
리눅스 계열 OS를 사용하고 있다면 WSL을 설치할 필요가 없이, 해당 OS의 패키지 설치 매니져(apt, yum 등등)를 통해 여타 설치 작업을 진행함.  
MS윈도우즈를 사용하고 있다면 (Win10 이상 가정) WSL 기능으로 리눅스 서브시스템을 설치한 뒤 리눅스 프롬프트에서 진행하게 됨.  
아래 내용은 Win10 파워쉘(powershell)을 관리자모드로 실행한 창에서 진행된 내용의 캡쳐임 (명령어 및 화면 출력 모두 포함).  
입력 (wsl 명령어에 --install 옵션커맨드로 Ubuntu 배포판을 설치함)
```
PS C:\WINDOWS\system32> wsl --install -d Ubuntu
```
출력
```
다운로드 중: Ubuntu
설치 중: Ubuntu
[=======    ]
Ubuntu이(가) 설치되었습니다.
Ubuntu 실행 중...
```
명령어 실행 직후 다운로드에 따라 시간이 걸리고, 설치 과정에 별도의 시간이 걸린 뒤...  
wsl 하 ubuntu 시스템 상 bash 프롬프트가 뜨면, 최초 사용자 아이디(아래 예시에서는 myid_yo) 등을 설정하게 됨.  
진행과정 (입력 출력 포함)
```
Installing, this may take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: myid_yo
New password: **********
Retype new password: **********
passwd: password updated successfully
Installation successful!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.10.16.3-microsoft-standard-WSL2 x86_64)
 * Documentation: https://help.ubuntu.com    * Management: https://landscape.canonical.com    * Support: https://ubuntu.com/advantage
  System information as of Wed Sep 14 15:09:03 KST 2022
  System load:  0.0                Processes:             8    Usage of /:   0.4% of 250.98GB   Users logged in:       0
  Memory usage: 0%                 Swap usage:   0%            IPv4 address for eth0: 172.30.140.147
0 updates can be installed immediately. 0 of these updates are security updates.
The list of available updates is more than a week old.
To check for new updates run: sudo apt update
This message is shown once once a day. To disable it please create the
/home/myid_yo/.hushlogin file.
myid_yo@DESKTOP-D009:~$ _
```
(참고로, 가장 아랫줄에서 cli 커서 직전의 DESKTOP-D009 등은 해당 윈도우즈 PC이름에 따라 달라짐)  
위와 같이 완성 후 wsl 내에 설치된 우분투 os 상에서 동작하는 bash 쉘 프롬프트 커서_가 떠있게 됨.  

### apt update
apt를 통한 설치를 진행하기 전에, apt를 최신 상태로 업데이트 해야함.  
과거 버젼에서 upgrade 및 update를 여러 옵션을 줘야 했으나, 방금 설치한 최신 우분투라는 가정에서, 간단히 아래 명령어로 진행. 
입력 (관리자 권한인 sudo 명령어이므로, 엔터 직후에 관리자 비밀번호를 입력한 뒤 동작함)
```
sudo apt update
```
출력
```
[sudo] password for myid-yo:
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
...(중략)...
Get:45 http://archive.ubuntu.com/ubuntu focal-backports/multiverse amd64 c-n-f Metadata [116 B]
Fetched 25.9 MB in 33s (786 kB/s)
Reading package lists... Done
myid-yo@DESKTOP-D009:~$ _
```
참고로, 기존에 이미 설치되고 동작되던 리눅스 상에서 신규 설치작업을 하는 경우, 
설치하려는 패키지 혹은 흔적 및 쓰레기가 남아있는 것을 소거한 뒤 신규 설치를 진행하기도 함. 
입력 (소거 명령어 예시)
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### Docker 설치
Ubuntu 등 리눅스에서는 cli에서 여러 명령어를 입력하여 설치를 진행. (시기와 버젼업 상황에 따라 다르니 웹검색)  

MS-Windows 에서는 (윈10 이후 WSL 탑재 가정) 윈도우용 Docker 인스톨러로 설치하는 것을 권장함.  
검색을 통하여 MS 공식 사이트 (docs.microsoft.com) 내 docker 안내 및 공식 사이트 (docs.docker.com)에서 다운로드 가능.  
설치에서 wsl 계층에 설치 사용이 가능하고, 설치하고 나면 cli는 물론 gui에서 설정 가능한 도커앱을 사용할 수 있음.  
[그림] MS윈도우즈용 공식 도커앱 인스톨러 설치 완료 화면
![image](https://user-images.githubusercontent.com/49431924/190540258-06d0f89c-fb11-40f9-aae2-c6a6bec18eb6.png)

도커 설치를 완료한 뒤, 제대로 설치 되었는지 확인은 아래와 같음. 
```
입력
출력
```
.
