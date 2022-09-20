# Docker 첫 설정  
보안 폴리시 준수를 위한 도커 설정 안내.  

## 설정 내용 요약  
* **sudo 쓰지 않도록** 설정함. (연구실 네트워크 구성 상, LDAP 상에 도커그룹을 설정하여 sudo 권한없어도 되도록 해두었음.)  
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
  * 실행 명령 차이 : ```docker exec``` 명령은 프로세스 새로 시작. ```docker attach``` 명령은 기존 작업 이어서 속행.  
  * 종료 명령 차이 : ```docker stop``` 명령으로 중지. 참고로 run 등 첫 실행 쉘에서 exit 치면 컨테이너도 중지됨(주의).  
  * 삭제 : 컨테이너 삭제는 ```docker rm [컨테이너이름]```. 이미지 삭제는 ```docker rmi [이미지태그이름]```  

----------------------------------------------------------------------

## 개요  
HW와 OS 계층을 가상화 하는(버츄얼) 기능은 현재 컴퓨팅 환경에서 필수적임.  

_**VM vs. Docker**_:  
* VM : 진정한 VM(버츄얼머신)과 sw레이어를 완전히 구현하는 작업은 복잡하고 육중한 일임.
  * 장점 - 모든 현존 컴퓨팅 환경의 구축. / 단점 - 사용자는 복잡한 설정과 느린 속도를 견뎌야 함.  
* Docker는 컴퓨팅 환경을 적정 수준에서만 실무적인 형태로 빠르고 편리하게 준-가상화하는 tool.  
  * 도커는 리눅스커널을 기본 가정하고, 하드웨어 밴더 간 구축된 표준에 따라 단순화된 독립 가상 환경을 구축.  

_**With or without Docker**_:  
* 도커가 없으면... 
  * 설치된 OS와 하드웨어 연결상태 및 드라이버, 라이브러리 등이 한 통 안에서 복잡하게 얽힘.  
  * 다양한 여러 사용자가 서로 다른 버젼 요구시 의존성이 꼬이거나 충돌하게 됨.  
* 도커가 있으면... 
  * 머신을 쓰는 각 사용자의 환경을 VM처럼 분리하여(=컨테이너) 각자 필요한 최소의 설치.  
  * 각자 분리된 스토리지 볼륨 안에 라이브러리 등 의존성 sw 버젼 설치로 상호 충돌 가능성 없어짐.  

**도커의 쾌적성과 충돌 방지 환경**  
도커로, 각자의 컨테이너 안에서 각자 구축한 환경에서 각자만의 프로세스를 돌리면서도, (속도저하 거의 없이) 
컨테이너 외부의 하드웨어 및 네트워크를 자유자재로 사용하게 됩니다.  
도커가 잘 구축된 서버에서 멀티유저가 동시에 멀티 프로세스를 수행하여도 상호 환경과 저장소는 배타적이며, 
본디 서버의 OS와 저장소는 그와 별개로 존재하며 돌아가는 이상적인 운영이 가능합니다.  

**도커의 이식성과 배포 편의성**  
  도커로 잘 구성한 이미지 및 컨테이너는 그대로 파일로 담아 다른 곳(다른 하드웨어, 다른 서버)에 가져가 실행할 수 
있습니다. 누군가 개발한 SW 및 서비스를 통채로 퍼담아 다른 곳에 가져가 실행하기 위한 좋은 도구가 되기에, 
OS & SW 및 라이브러리 환경을 직접 구축할 필요가 없는 사용자들에게 좋은 도구이며...  
또한 개인적인 사용에 있어서도, 본디의 OS를 변경하지 않으면서, 다른 설치 및 설정 구성을 잠시 시험 사용할 수 있는 
가상공간 도구의 역할로도 활용됩니다.  

이하 내용은 요약 문체로 서술되며, 명령어 구성은 실제 본 건물 환경에서 자주 사용되는 예제 구성으로 안내됩니다.  

---
## Docker Build & Run 순서  
도커 사용은 크게 다음의 과정으로 진행됨.  
|순서 |--설명--------------|--명령어 예시------------------------------|
|---|---|---|
|1. **Pull**    |Base가 될 도커 이미지를 가져옴 |```docker pull tensorflow/tensorflow```|  
|2. **Prepare** |폴더 경로와 텍스트 파일을 준비함|```id``` / ```mkdir docker``` / ```cd docker``` / ```vi dockerfile```|  
|3. **Build**   |사용할 도커 이미지를 빌드함    |```docker build -t myid-yo/test . -f dockerfile```|  
|4. **Run**     |컨테이너를 실행하여 들어감     |```docker run -ti --gpus all --name=... -v ... -p ... myid-yo/test```|  

---
### 1. Pull (get base docker image)  
DockerHub 등에서 사용할 docker image를 찾고, 이미지 이름 문자열을 알아 둠.  
(예를 들어 텐서플로우 공식 도커 이미지를 찾아보면, ```"tensorflow/tensorflow"``` 임을 알 수 있음)  

이미지를 다운로드하는 명령어 형식은 ~~sudo~~ ```docker pull "docker image name"```과 같음.  
(예로 위 텐서플로우의 경우 실제 명령어는 ```docker pull tensorflow/tensorflow```가 됨. 이 경우 도커 이미지 이름에 공백문자가 없으므로 따옴표를 생략할 수 있음.)  

* _**sudo에 관하여: 예전에 개인적 사용 시 sudo 명령어를 많이 사용했으나, 연구실 보안 환경에서는 되도록 sudo 사용을 자제해야 하므로, sudo 없이 도커를 사용할 수 있는 설정을 잘 진행해야 함.**_  

**예시(입력 명령어)**:: 텐서플로우 도커 pull 명령 실행. (예시와 같이 정확한 버젼까지 지정할 수도 있고, 생략하여 "tensorflow/tensorflow"만 입력해도 알아서 버젼을 pull함)  
```
docker pull "tenserflow/tensorflow:1.15.0-gpu-py3"
```
**예시**:: pull시 입출력 예시 화면  
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
혹은 그 이미지를 근간으로 시작하여 내가 만드는 용도의 환경을 추가한 수정 이미지를 구워서 배포할 수 있음.  
  
* 개인적인 docker hub repository에 올려둔 image가 있는 경우, 같은 방식으로 image pull 가능함. 명령어 예시 : ~~sudo~~ ```docker pull myimage/test:latest``` (콜론 뒤에 최신버젼을 지정)  
* 개인 이미지 혹은 타인이 완성해 놓은 이미지를 그대로 이용하는 경우, 아래의 준비 및 빌드 과정이 필요 없으므로, **Run 챕터로 스킵 가능**.  

---
### 2. Prepare (folder and file)  
내 환경의 도커 이미지를 만들기 위해 사전 준비로 uid를 알아내고 컨테이너 생성용 스크립트를 작성한 뒤 docker build 명령어를 실행하게 됨.  

**UID 알아내기**  
리눅스 쉘에서 간단히 명령어 ```id```를 입력하면, 사용중인 id에 대해 아래와 같이 여러 정보가 출력됨.  
(예시에서의 리눅스 사용자 아이디는 "myid-yo"로 진행중)  
```
myid-yo@DESKTOP:~$ id

uid=1000(myid-yo) gid=1000(myid-yo) groups=1000(myid-yo), 4(adm), 20(dialout), 24(cdrom), 25(floppy), 
27(sudo), 29(audio), 30(dip), 44(video), 46(plugdev), 117(netdev), 999(docker)
```
위 예시에서 "1000"으로 출력된 uid값을 나중에 사용하게 되므로 적든지 기억해둬야함. (위 명령어로 그룹아이디 상태 및 docker가 설치 설정된 상태도 알 수 있음)  
  
**docker text 파일 만들기**  
뒤에 실행하게 될 docker build 명령어에서 빌드텍스트 파일을 지정하게 되므로, 그 텍스트 파일을 만들어야 함.  
예시와 같이, 작업을 할 폴더를 정하여 들어가서 텍스트 파일을 작성.  
**예시**:: 사용자 홈폴더에 돌아가서(```cd ~```), 그 아래에 docker라는 폴더를 만들고(```mkdir docker```) 들어가서(```cd docker```) dockerfile이라는 이름의 텍스트파일을 생성 편집 시작(```vi dockerfile```).  
~~~
myid-yo@DESKTOP:/usr$ cd ~
myid-yo@DESKTOP:~$ mkdir docker
myid-yo@DESKTOP:~$ cd docker
myid-yo@DESKTOP:~/docker$ vi dockerfile
~~~
마지막 명령어로 화면에 띄운 ```vi``` 편집기에서 (시작 직후에 a키를 눌러서) "입력모드"에 들어간 뒤,  
"FROM ..."등의 도커명령어들을 아래 예시와 같이 여러줄 입력하고,  
편집모드를 빠져나와(즉, esc누름) 저장하고 종료함(즉, ":wq" 타이핑 후 엔터).  
**예시1**:: dockerfile이라는 텍스트 파일 안에 저장할 내용. (gid 및 uid에 지정된 숫자는 위에서 기억해 둔 값)  
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

**예시2**:: 조금 더 내용이 많은 다른 dockerfile 내용. (라인 끝 3128 앞에 넣을 내용은 시스템어드민에게 문의)  
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
(위에서 설명했듯) vi에서 a(혹은 i)로 입력모드 들어가 입력할 텍스트를 다 넣은 뒤, esc를 누르고 콜론(:)을 누른 뒤, 저장종료 명령어인 wq를 입력하고 enter를 누르면 파일이 생성 저장된 후 다시 쉘 프롬프트로 탈출하게 됨.  
텍스트 파일 편집 및 저장을 마쳤으면 ```ls``` 혹은 ```cat``` 명령어로 제대로 만들어졌는지 확인 가능.  

---
### 3. Build (my image)
작성한 파일이 있는 폴더에서 아래 명령어를 입력하여 내가 사용할 이미지를 빌드함.  
명령어 형식은 ``` docker build -t [이미지태그이름짓기] . -f [위에준비한텍스트파일]```과 같음.  

**예시**:: 도커 빌드 명령어에 이미지 이름(태그이름, -t)을 "myid-yo/test"로 지어주고, 현재 경로를 작업 경로로(.), 빌드 내용 파일(-f)은 위에서 작성한 "dockerfile"로 실행.  
```
docker build -t myid-yo/test . -f dockerfile
```
**예시**:: Build진행 입출력 화면  
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

---
### 4. Run (docker container)  
이미지를 지정하여 도커를 실행하면 컨테이너가 생성되어 그 환경 내부 쉘로 들어가게 됨.  
도커 실행 명령어 형식은 ```docker run -ti --gpus all --name=[컨테이너이름짓기] -v [밖경로:안경로] -p [밖포트:안포트] [도커이미지이름]```  

실제 (외부) 시스템 상 스토리지와 컨테이너 환경 안에서의 스토리지는 (가상화 원칙과 같이) "분리"되지만,  
"-v" 옵션을 사용하여 실제 시스템의 특정 경로(폴더)를 컨테이너 환경 안에서의 지정 경로로 연결 사용 가능.  
"-v" 옵션은 여러번 반복할 수 있음. (여러 다른 폴더를 컨테이너 안에 여러 다른 경로로 연결 사용).  

**예시**:: ```docker run``` 명령에 기본옵션을 주고, 지금 실행되는 생성 컨테이너 이름을 "my_test_c"로 짓고, 실제 시스템 상 "/mnt/c/user" 폴더를 도커 안에서도 접근 가능한 내부 경로 "/workspace"로 연결하며, 포트는 내외부 8080 그대로, 실행될 도커 이미지는 위에서 만든 "myid-yo/test" 태그 이름을 지정함.  
```
docker run -ti --gpus all --name=my_test_c -v /mnt/c/user:/workspace -p 8080:8080 myid-yo/test
```
에러가 없다면 도커 컨테이너가 실행되고, 그 환경 안쪽 쉘(bash 등) 프롬프트인 cli가 펼쳐지게 됨.  

이후, 원하는 작업을 진행하면 됨.  

---
### 그밖에...  
#### [비밀번호 설정]
당연히 보안을 위하여 처음에 아이디 비밀번호를 잘 설정하고 시작해야 함.  

비밀번호 설정 작업을 위해 ```docker start [컨테이너이름]``` 실행 후 ```docker exec -u 0 -ti [컨테이너이름] bash``` 라는 명령어로 로그인 한 뒤 ("-u 0"로 영번째 uid인 root로 접속하게 됨) 아래 명령어들로 비번 설정을 진행.  
**예시**:: pw작업 진행 입출력 화면
```
myid-yo@DESKTOP:~$ docker start my_test_c

myid-yo@DESKTOP:~$ docker exec -u 0 -ti my_test_c bash

root@8d44c1b:/# passwd myid-yo

Enter new UNIX password: **********
Retype new UNIX password: **********
passwd: password updated successfully

root@8d44c1b:/# exit
```

---
#### [실행 명령들 차이점]
도커를 사용한다는 것은, 이미지를 사용하는 것보다 "컨테이너"를 실행하여 해당 환경 안에 들어가서 사용하는 것을 뜻함.  

컨테이너를 시작하는 과정은 여러가지일 수 있으나, 대표적인 **시작 명령어 run과 start의 차이점**은 다음과 같음.  
* **run** : 지정된 이미지로부터 최초로 컨테이너를 생성하여 리눅스 시스템을 부팅하고, 준비되면 사용자에게 기능을 제공함. 
  * 부팅이 완료되면 도커 런 명령의 옵션으로 지정된 첫 프로세스를 실행해주는데, -t 옵션(tty라는 의미)과 함께 bash를 지정하면 사용자가 cli로 진입됨. 
  * 런명령에 --rm 옵션을 주는 경우, 부팅 첫 프로세스가 종료되면 컨테이너도 종료 및 제거되므로 매우 주의해야 함. 예를 들어 bash인 경우 해당 bash에서 exit 명령하는 순간 컨테이너까지 종료 제거됨. (반드시 의도하는게 아니라면 run 명령어에 --rm 옵션을 주지 말것.)  
* **start** : 이미 부팅되어 있는 도커 프로세스를 시작하여 작업에 돌입하기 위한 명령어 (빠름). 

한 개의 리눅스 OS로서 활성화 되어 있는 컨테이너에, 명령어나 프로세스를 **실행 진입하는 명령어 exec와 attach 차이점**은 다음과 같음.  
리눅스는 멀티유저 멀티프로세스 OS이어서 아무 타이밍에나 서로 다른 id가 각기 로그인하여 동시다발적인 프로세스를 수행함.  
심지어 같은 사용자의 아이디마저 새로 bash를 띄워 로그인하면 별개의 프로세스로 로그인하여 분리된 프로세스를 실행 가능함. 
* **exec** : 새로운 프로세스를 실행함. (bash를 exec하면 새로운 프로세스로써 bash가 뜨게 됨.) 
  * 주요 작업용 프로세스를 하던 중에 시스템에 별개 아이디로 로그인하여 사이드웍을 수행할 수 있으며, 리눅스 시스템이기 때문에 그 결과 및 여파가 다른 아이디(주작업 프로세스 측)에도 영향을 끼침. 
  * exec로 실행했던 내용 중 ram에서만 동작하던 내용들은 exit와 함께 사라질 것임. (스토리지에 저장하는 작업들은 영원히 저장됨) 
* **attach** : 기존에 있던 프로세스에 재돌입 하여 앞으로 띄움. 
  * 예전에 돌리던 프로세스가 백그라운드로 묻혔거나 혹은 suspend된 상황에서 attach 명령을 쓰면 다시 활성화하여 작업을 속행함. 

---
#### [종료 명령어들 차이점]
TBU
