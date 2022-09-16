# Docker 첫 설정  
보안 폴리시 준수를 위한 도커 설정 안내.  

## 개요  
개요 TBU  

## Docker Build 순서  
1. Pull (get docker image)  
2. Container (image build)  
3. Run (docker container)  

### 1. Pull (get docker image)  
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

### 2. Build (container)  
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
텍스트 파일 편집 및 저장을 마쳤으면 ```ls``` 혹은 ```cat``` 명령어로 제대로 되었는지 확인 가능.  

### 3. Run 

