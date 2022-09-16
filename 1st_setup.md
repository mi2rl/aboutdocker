# Docker 첫 설정  
보안 폴리시 준수를 위한 도커 설정 안내.  

## 개요  
개요 TBU  

## 요약  
* **sudo 쓰지 않도록** 설정하자. (연구실 네트워크 구성 상, LDAP 상에 도커그룹을 설정하여 sudo 권한없어도 되도록 해두었음.)
* **pull -> prepare -> build -> run**
  * pull : (이하, 예시 명령어) ```docker pull tensorflow/tensorflow```
  * prepare : id 명령어로 uid 번호를 알아내고, 홈폴더 (~) 아래 docker 폴더 만들어서 텍스트파일 "dockerfile"을 작성함.
    *  ```FROM tensorflow/tensorflow \ RUN export HTTP_PROXY=http://192.168....:3128 \ ... \ RUN apt update \ RUN apt install –y sudo vim \ RUN addgroup ... \ RUN adduser ... \ USER myid-yo```
  * build : ```docker build -t myid-yo/test . -f dockerfile```
  * run : ```docker run -ti --gpus all --name=my_test_c -v /mnt/nas100/myid_yo:/workspace -p 8080:8080 myid-yo/test```
* **비번 변경** :  
  * ```docker start my_test_c``` 실행 후 ```docker exec -u 0 -ti my_test_c bash``` 실행 (root 접속됨).  
  * ```passwd myid-yo``` 실행하여 비밀번호 새로 잘 설정한 뒤 사용할 것. (설정 마친 후 잊지말고 exit 실행)  
* 그밖에...  
  * 실행 : ```docker exec``` 명령 vs. ```docker attach``` 명령
  * 종료 : ```docker stop```
  * 삭제 : 컨테이너 삭제는 ```docker rm [컨테이너]```. 이미지 삭제는 ```docker rmi [이미지태그]```

## Docker Build & Run 순서  
도커 사용은 크게 다음의 과정으로 진행됨.  

|도커 빌드 및 사용 대략적인 과정|------------------------------------------------|
|---|---|
|1. **Pull** (get base docker image) |Base가 될 도커 이미지를 가져옴|  
|2. **Prepare** (folder and file)    |폴더 경로와 텍스트 파일을 준비함|  
|3. **Build** (my image)             |사용할 도커 이미지를 빌드함|  
|4. **Run** (docker container)       |컨테이너를 실행하여 들어감|  

### 1. Pull (get base docker image)  
DockerHub 등에서 사용할 docker image를 찾고, 이미지 이름 문자열을 확보함.  
(예를 들어 텐서플로우 공식 도커 이미지를 찾아보면, ```"tensorflow/tensorflow"``` 임을 알 수 있음)  

명령어를 ~~sudo~~ ```docker pull "docker image name"```과 같이 입력하여 다운로드.  
(예로 위 텐서플로우의 경우 ```docker pull tensorflow/tensorflow``` 라는 명령어. 도커 이미지 이름에 공백문자가 없으므로 따옴표를 생략할 수 있음.)  

_* sudo에 관하여: 예전에 개인적 사용 시 sudo 명령어를 많이 사용했으나, 연구실 보안 환경에서는 되도록 sudo 사용을 자제해야 하므로, sudo 없이 도커를 사용할 수 있는 설정 및 설치를 첫단추부터 잘 진행하게 됨._  

예시(입력):: 텐서플로우의 정확한 버젼까지 지정하여 pull 명령 실행. ("tensorflow/tensorflow만 입력하면 알아서 버젼을 pull함)  
```
docker pull "tenserflow/tensorflow:1.15.0-gpu-py3"
```
입출력 예시 화면::  
```
myid-yo@DESKTOP:~$ docker pull tensorflow/tensorflow

Using default tag: latest
latest: Pulling from tensorflow/tensorflow
675920708c8b: Pull complete
...(중략)...
b14946c4f6e6: Pull complete
Digest: sha256:7f9f23ce2473eb52d17fe1b465c79c3a3604047343e23acc036296f512071bc9
Status: Downloaded newer image for tensorflow/tensorflow:latest
docker.io/tensorflow/tensorflow:latest

myid-yo@DESKTOP:~$
```
위와 같이 완료되면 로컬에 도커 이미지가 준비되었으므로, 그것에서 도커환경을 실행하여 들어갈 수 있고,  
혹은 저 이미지를 근간으로 시작하여 내가 만드는 용도의 이미지 환경을 추가하거나 이미지를 구워서 배포할 수 있음.  
  
* 개인적인 docker hub repository에 올려둔 image가 있는 경우, 같은 방식으로 image pull 가능함. 명령어 : ~~sudo~~ ```docker pull myimage/test:latest```  
* 개인 이미지 혹은 타인이 완성해 놓은 이미지를 이용하는 경우, 아래의 준비 및 빌드 과정이 필요 없으므로, Run 챕터로 스킵 가능.  

### 2. Prepare (folder and file)  
내 환경의 도커 이미지를 만들기 위해 사전 준비로 uid를 알아내고 컨테이너 생성용 스크립트를 작성한 뒤 docker build 명령어를 실행하게 됨.  

**UID 알아내기**  
리눅스 쉘 명령으로 ```id```를 입력하면, 사용중인 id에 대해 아래와 같이 여러 정보가 출력됨.  
(예시에서의 리눅스 사용자 아이디는 "myid-yo"로 진행중임)  
```
myid-yo@DESKTOP:~$ id

uid=1000(myid-yo) gid=1000(myid-yo) groups=1000(myid-yo), 4(adm), 20(dialout), 24(cdrom), 25(floppy), 
27(sudo), 29(audio), 30(dip), 44(video), 46(plugdev), 117(netdev), 999(docker)
```
위 예시에서 "1000"으로 출력된 uid값을 나중에 사용하게 되므로 적든지 기억해둬야함. (위 명령어로 그룹아이디 상태 및 docker가 설치 설정된 상태도 알 수 있음)  
  
**docker text 파일 만들기**  
뒤에 실행하게 될 docker build 명령어에서 텍스트 파일을 지정하게 되므로, 그 텍스트 파일을 만들어야 함.  
예시와 같이, 작업을 할 폴더를 정하여 들어가서 텍스트 파일을 작성.  
예시:: 사용자 홈폴더에 돌아가서(```cd ~```), 그 아래에 docker라는 폴더를 만들고(```mkdir docker```) 들어가서(```cd docker```) dockerfile이라는 이름의 텍스트파일을 생성 편집 시작(```vi dockerfile```).  
~~~
myid-yo@DESKTOP:/usr$ cd ~
myid-yo@DESKTOP:~$ mkdir docker
myid-yo@DESKTOP:~$ cd docker
myid-yo@DESKTOP:~/docker$ vi dockerfile
~~~
마지막 명령어로 뜨는 vi 편집기 화면에서 새로운 내용 입력모드(a키를 누름)에 들어간 뒤,  
"FROM ..."등의 도커명령어들을 아래 예시와 같이 여러줄 입력하고,  
편집모드를 빠져나와(esc누름) 저장하고 종료함(":wq" 타이핑 후 엔터).  
예시1::dockerfile이라는 텍스트 파일 안에 저장할 내용. (gid 및 uid에 지정된 숫자는 위에서 기억해 둔 값임)  
~~~
FROM tensorflow/tensorflow
RUN apt update
RUN apt install –y sudo vim
RUN addgroup --gid 1000 gusers
RUN adduser  --uid 1000 --gid 1000 --disabled-password -gecos '' myid-yo
RUN adduser myid-yo sudo
USER myid-yo
~~~
* 참고로, 마지막 줄 이전에 원하는 리눅스 패키지의 설치 (apt install) 및 파이썬 패키지 설치 (pip install) 등을 추가할 수 있음.  
* 또한, 첫줄 직후 부분에 외부차단 인트라넷 보안 프록시 관련 내용이 추가될 수도 있음.  
예시2::조금 더 내용이 많은 다른 dockerfile 내용. (3128 앞쪽 내용은 시스템어드민에게 문의)  
~~~
FROM tensorflow/tensorflow
RUN export HTTP_PROXY=http://192.168....:3128
RUN export HTTPS_PROXY=http://192.168...:3128
RUN export PIP_PROXY=http://192.168.....:3128
RUN apt update
RUN apt install –y sudo vim
RUN addgroup --gid 1000 gusers
RUN adduser  --uid 1000 --gid 1000 --disabled-password -gecos '' myid-yo
RUN adduser myid-yo sudo
RUN apt install -y my_preferred_util
RUN pip install my_favo_py_lib
USER myid-yo
~~~
vi에서 a(혹은 i)를 눌러 텍스트를 다 입력한 뒤에 (위에 설명했 듯) esc를 누르고 콜론(:)을 누른 뒤, 명령어 wq를 입력하고 enter를 누르면 파일이 생성 저장된 후 다시 쉘 프롬프트로 탈출하게 됨.  
텍스트 파일 편집 및 저장을 마쳤으면 ```ls``` 혹은 ```cat``` 명령어로 제대로 되었는지 확인 가능.  

### 3. Build (my image)
작성한 파일이 있는 폴더에서 아래 명령어를 입력하여 내가 사용할 이미지를 빌드함.  
명령어 형식은 ``` docker build -t [이미지태그이름짓기] . -f [위에준비한텍스트파일]```과 같음.  

예시::도커 빌드 명령어에 이미지 이름(태그, -t)를 "myid-yo/test"로 지어주고, 현재 경로를 작업 경로로(.), 빌드 내용 파일(-f)은 위에서 작성한 "dockerfile"로 실행.  
```
docker build -t myid-yo/test . -f dockerfile
```
예시::Build진행 입출력 화면
```
myid-yo@DESKTOP:~/docker$ docker build -t myid-yo/test . -f dockerfile

[+] Building 28.9s (5/9)
 => [internal] load build definition from dockerfile                                                               0.0s
 => => transferring dockerfile: 253B                                                                               0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load metadata for docker.io/tensorflow/tensorflow:latest                                            0.0s
 => [1/6] FROM docker.io/tensorflow/tensorflow                                                                     0.4s
 => [2/6] RUN apt update                                                                                          21.6s
 => => # Get:3 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 vim-common all 2:8.1.2269-1ubuntu5.8 [85.2 kB]
 => => # Get:4 ...(중간생략)...
 => [6/6] RUN adduser myid-yo sudo                                                                                   0.6s
 => exporting to image                                                                                             0.4s
 => => exporting layers                                                                                            0.4s
 => => writing image sha256:8e9f2f7370a0083ce7e91604ced91d704831115c81a09b3b261c60801c6a68d8                       0.0s
 => => naming to docker.io/myid-yo/test                                                                            0.0s

myid-yo@DESKTOP:~/docker$
```
에러가 없다면 위와 같이 완결되어 새로운 도커 이미지 파일이 생성됨.  
* 사용할 이미지가 잘 빌드된 것을 확인한 뒤에는 base 용도로 처음에 받았던 이미지를 삭제할 수 있음.  
  * 삭제 명령어 예시 : ```docker rmi tensorflow/tensorflow```

### 4. Run (docker container)  
이미지를 지정하여 도커를 실행하면 컨테이너가 생성되어 그 환경 내부 쉘로 들어가게 됨.  

실제 시스템 상 스토리지에 있는 어느 경로를 컨테이너 안에서도 특정 경로로 연결 접근하기 위해 "-v" 옵션을 지정할 수 있음.  
"-v" 옵션은 여러번 반복하여 외부(실제 시스템)의 여러 다른 폴더를 컨테이너 환경 내부의 여러 다른 경로로 사용 가능함.  

도커 실행 명령어 형식은 ```docker run -ti --gpus all --name=[컨테이너이름짓기] -v [밖경로:안경로] -p [밖포트:안포트] [도커이미지이름]```  
예시::docker run 명령에 기본옵션을 주고, 지금 실행되는 생성 컨테이너 이름을 "my_test_c"로 짓고, 실제 시스템 상 "/mnt/c/user" 폴더를 도커 안에서도 접근 가능한 내부 경로 "/workspace"로 연결하며, 포트는 내외부 8080 그대로, 실행될 도커 이미지는 위에서 만든 "myid-yo/test" 태그 이름을 지정함.  
```
docker run -ti --gpus all --name=my_test_c -v /mnt/c/user:/workspace -p 8080:8080 myid-yo/test
```
에러가 없다면 도커 컨테이너가 실행되고, 그 환경 안쪽 쉘(bash 등) 프롬프트인 cli가 펼쳐지게 됨.  

이후, 원하는 작업을 진행하면 됨.  

### 그밖에...  
#### [비밀번호 설정]
당연히 보안을 위하여 처음에 아이디 비밀번호를 잘 설정하고 시작해야 함.  

비밀번호 설정 작업을 위해 ```docker start [컨테이너이름]``` 실행 후 ```docker exec -u 0 -ti [컨테이너이름] bash``` 라는 명령어로 0번째(uid) 아이디인 root로 로그인 한 뒤 아래 명령어들로 비번 설정을 진행.  
예시::작업 진행 입출력 화면
```
myid-yo@DESKTOP:~$ docker start my_test_c

myid-yo@DESKTOP:~$ docker exec -u 0 -ti my_test_c bash

root@8d44c1b:/# passwd myid-yo

Enter new UNIX password: **********
Retype new UNIX password: **********
passwd: password updated successfully

root@8d44c1b:/# exit
```

#### [attach와 exec의 차이]
TBU

#### [종료, 소거, 삭제 ...]
TBU
