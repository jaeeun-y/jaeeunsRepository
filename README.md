## jaeeunsRepository

### 프로젝트 개요  

1. 커스텀 웹 서버 이미지 생성  

2. 컨테이너와 포트 매핑 실습
   ㄴ 어느 컴퓨터에서든 호스트 포트와 도커 내부 포트의 매핑을 통해 재현성 구현

3. 볼륨을 이용한 데이터 영속성 확인  
   ㄴ 도커 볼륨을 사용해 데이터를 영속화
 
4. 버전 관리와 깃허브 연동  

+ 트러블슈팅
----  
## 0. 실행 환경  
OS: macOS  
Docker version: 28.5.2  
Z쉘 zsh (macOS 기본 쉘)  
Git version: 2.53.0

## 1. 터미널 조작 로그 기록

### 1-1 현재 위치 및 목록 확인 (숨김 파일 포함)
```
#현재 작업 디렉토리 확인
student@c6r1s8 ~ % pwd
/Users/student
```
```

#파일 목록 확인
student@c6r1s8 ~ % ls -al
total 8
drwxr-x---+ 18 student student   576  4  2 17:02 .
drwxr-xr-x   9 root    admin     288  4  2 16:45 ..
-r--------   1 student student     8  4  2 16:52 .CFUserTextEncoding
drwxr-xr-x   5 student student   160  4  2 16:52 .docker
drwxr-xr-x  10 student student   320  4  2 16:53 .orbstack
drwxr-xr-x   3 student student    96  4  2 16:52 .ssh
drwx------+  2 student student    64  4  2 16:51 .Trash
drwxr-xr-x   3 student student    96  4  2 16:52 .vscode
drwx------   3 student student    96  4  2 17:02 .zsh_sessions 
drwx------+  3 student student    96  4  2 16:45 Desktop
drwx------+  3 student student    96  4  2 16:45 Documents
drwx------+  4 student student   128  4  2 17:00 Downloads
...
```

### 1-2 파일 생명주기 관리 (생성, 복사, 삭제)
```
student@c6r1s8 ~ % touch newfile1
student@c6r1s8 ~ % ls
Desktop		Downloads	Movies		newfile1	Pictures //새 파일이 생성됨
Documents	Library		Music		OrbStack	Public
```

```
student@c6r1s8 ~ % cat newfile1
hello jaeeun
student@c6r2s8 ~ % cat newfile2
cat: newfile2: No such file or directory
student@c6r1s8 ~ % cp newfile1 newfile2 //newfile1 파일 복사
student@c6r1s8 ~ % cat newfile2
hello jaeeun
```

### 1-3 이동 및 이름변경
```
#파일의 이름 변경
student@c6r1s8 ~ % mv newfile3  newfile4 
student@c6r1s8 ~ touch newfile3
student@c6r2s8 ~ % ls
Desktop		Documents	Library		Music		Pictures	myweb
Dockerfile	Downloads	Movies		OrbStack	Public newfile3
student@c6r1s8 ~ % mv newfile3  newfile4
student@c6r1s8 ~ % ls
Desktop		Downloads	Movies		newfile4	Pictures
Documents	Library		Music		OrbStack	Public
```
```
#파일의 이동
student@c6r1s8 ~ % mv newfile4 newdir1
student@c6r1s8 ~ % cd newdir1 //change directory로 newdir1로 이동
student@c6r1s8 newdir1 % ls
newfile4
```
### 1-4 파일 내용 확인
```
student@c6r1s8 ~ % echo "hello jaeeun" > newfile1
student@c6r1s8 ~ % cat newfile1 //파일 내용 확인
hello jaeeun
```
----
## 2. 권한 실습
### 2-1 권한 확인 및 권한 변경

r 읽기 권한 (4)
w 쓰기 권한 (2)
x 실행 권한 (1)

d 디렉터리
- 파일

### -변경 전  
```
#디렉터리 권한 확인
student@c6r1s8 ~ % ls -l newdir1
total 8
-rw-r--r--  1 student student  3  4  2 18:12 newfile4
#파일 권한 확인
student@c6r1s8 ~ % ls -l newdir1/newfile4 
-rw-r--r--  1 student student  3  4  2 18:12 newdir1/newfile4
```
### -변경 후
```
% chmod -R 777 newdir1
% ls -ld newdir1
drwxrwxrwx  3 student student  96  4  2 18:17 newdir1
% ls -l newdir1        
total 8
-rwxrwxrwx  1 student student  3  4  2 18:12 newfile4
```
----
## 3. Docker 설치 및 기본 점검
### 3-1 Docker 버전 확인 결과
```
% docker --version
Docker version 28.5.2, build ecc6942
```
### 3-2 Docker 데몬 동작 여부 확인 결과
```
% docker info
Client: // 도커에게 사용자의 명령을 받아서 전달하는 도구
Version:    28.5.2
 Context:    orbstack
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.29.1
    Path:     /Users/student/.docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.40.3
    Path:     /Users/student/.docker/cli-plugins/docker-compose
Server: // 실제로 일을 수행하는 도커 엔진
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 28.5.2
 Storage Driver: overlay2
  Backing Filesystem: btrfs
...(중략)..
```
----
## 4. Docker 기본 운영 명령 수행
### 이미지: 다운로드/목록 확인
```
#이미지 hello-world 다운로드
% docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
Digest: sha256:452a468a4bf985040037cb6d5392410206e47db9bf5b7278d281f94d1c2d0931
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
% docker images
REPOSITORY    TAG       IMAGE ID       CREATED      SIZE
hello-world   latest    e2ac70e7319a   9 days ago   10.1kB
//docker pull 명령어를 활용하여 docker hub에서 hello-world 이미지를 다운로드 받아
정상적으로 images 목록에 추가된 것을 확인함.
```
### 컨테이너: 실행/중지/목록 확인
```
#이미지를 백그라운드에서 실행
% docker run -d --name myweb nginx
#실행 중인 컨테이너
% docker ps                       
CONTAINER ID   IMAGE     COMMAND                   CREATED              STATUS              PORTS     NAMES
7351add7fbde   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    myweb
//status가 up 이므로 실행 중임을 확인할 수 있음
#중지
% docker stop myweb
myweb
#(죽어있는 컨테이너까지 포함한) 목록 확인
% docker ps -a
CONTAINER ID   IMAGE         COMMAND                   CREATED          STATUS                          PORTS     NAMES
7351add7fbde   nginx         "/docker-entrypoint.…"   4 minutes ago    Exited (0) About a minute ago             myweb
8ae248d8fdd2   hello-world   "/hello"                  15 minutes ago   Exited (0) 15 minutes ago                 clever_haibt
//두 컨테이너 다 status가 Exited인 것을 확인할 수 있음
```
### 운영: 로그 확인, 리소스 확인
```
#로그 확인
% docker logs myweb
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/04/02 11:38:49 [notice] 1#1: using the "epoll" event method
2026/04/02 11:38:49 [notice] 1#1: nginx/1.29.7
2026/04/02 11:38:49 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19) 
2026/04/02 11:38:49 [notice] 1#1: OS: Linux 6.17.8-orbstack-00308-g8f9c941121b1
...
```
```
#리소스 확인 // 컨테이너의 CPU나 메모리 점유율 확인
% docker logs myweb
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O     PIDS 
7351add7fbde   myweb     0.00%     20.43MiB / 15.67GiB   0.13%     1.13kB / 126B   16.1MB / 0B   7 
```
----
## 5. 컨테이너 실행 실습
### 5-1 hello-worlld 실행
```
% docker run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
```
### 5-2 ubuntu 컨테이너 실행 & 내부 진입 후 간단 명령 수행 결과
```
#ubuntu 컨테이너 실행
% docker run -it  ubuntu bash
```
```
#파일 목록 확인
root@b6ab414dfabc:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
#새 디렉토리 생성 후 내용 출력
root@b6ab414dfabc:/# echo "so tired" > newdir1
root@b6ab414dfabc:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  newdir1  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
### 5-3 컨테이너 종료/유지의 차이
- 실험용 컨테이너 준비
% docker run -dit --name testcontainer  ubuntu bash // -d: 백그라운드에서, -i: 컨테이너 오픈 상태 유지, -t:가상터미널 할당
```
#attach
#새 프로세스로 접속
% docker attach  testcontainer    
#프로세스 확인
root@cb76d25f228d:/# ps aux    
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4596  2608 pts/0    Ss   12:45   0:00 bash
root@cb76d25f228d:/# exit
#컨테이너 목록 확인
% docker ps -a
CONTAINER ID   IMAGE         COMMAND                   CREATED             STATUS                      PORTS     NAMES
cb76d25f228d   ubuntu        "bash"                    16 minutes ago      Exited (0) 12 seconds ago             testcontainer
```
```
#exec
#새 프로세스로 접속
% docker exec -it testcontainer bash
#프로세스 확인 후 종료
root@cb76d25f228d:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4596  3884 pts/0    Ss+  12:45   0:00 bash
root           9  0.0  0.0   4596  4016 pts/1    Ss   12:45   0:00 bash // 새로 만든 bash셸
root@cb76d25f228d:/# exit
#컨테이너 목록 확인
% docker ps   
CONTAINER ID   IMAGE     COMMAND                   CREATED              STATUS              PORTS     NAMES
cb76d25f228d   ubuntu    "bash"                    About a minute ago   Up About a minute             testcontainer
//컨테이너가 종료되지 않고 남아있음을 확인 가능
```
----
## 6. 기존 Dockerfile 기반 커스텀 이미지 제작
(A) 웹 서버 베이스 이미지 활용 + 정적 콘텐츠/설정 교체  

이미지 = 정적 틀
컨테이너 = 실행되는 동적 프로세스

### 6-1 커스텀 이미지 빌드 성공 및 컨테이너 실행 성공

myweb
 ㄴ src
   ㄴ index.html
 ㄴ Dockerfile


```
# 디렉터리 생성
% mkdir -p myweb/src // myweb 폴더와 그 안에 소스코드를 담을 src폴더를 생성
% cd myweb
#src폴더 속 html 웹 페이지 파일 생성
myweb % echo "<h1>Hello, Docker World! (Version 1)</h1>" > src/index.html
#Dockerfile 작성하기
myweb % cat <<EOF > Dockerfile
FROM nginx:latest //가장 최근 nginx이미디를 뼈대로 가져옴
COPY src/ /usr/share/nginx/html/ //내 컴퓨터의 src 폴더 안에 있는 파일들을 컨테이너 내부의 /usr/share/nginx/html/ 폴더로 복사해라
EOF
#Dockerfile build
myweb % docker build -t myweb-app . //-t: 새로 만들어지는 이미지에 이름 붙이기
[+] Building 0.5s (7/7) FINISHED                                                                                                                           docker:orbstack
 => [internal] load build definition from Dockerfile                                                                                                                  0.1s
 => => transferring dockerfile: 88B                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                                                                                       0.0s
 => [internal] load .dockerignore                                                                                                                                     0.1s
 => => transferring context: 2B                                                                                                                                       0.0s
 => [internal] load build context                                                                                                                                     0.1s
 => => transferring context: 58B                                                                                                                                      0.0s
 => [1/2] FROM docker.io/library/nginx:latest                                                                                                                         0.0s
 => CACHED [2/2] COPY src/ /usr/share/nginx/html/                                                                                                                     0.0s
 => exporting to image                                                                                                                                                0.0s
 => => exporting layers                                                                                                                                               0.0s
 => => writing image sha256:851e940c91be86bc64f520b0b61c723a6daaacffa582944bed48207353a0ae44                                                                          0.0s
 => => naming to docker.io/library/myweb-app          
#컨테이너 실행 및 포트 매핑
myweb % docker run -d -p 8080:80 --name custom-web myweb-app 
```
<img width="1658" height="482" alt="image" src="https://github.com/user-attachments/assets/25f0fd80-9293-40e0-ae21-f8366724c1dd" />
----

## 7. 포트 매핑 및 접속 증거

컨테이너는 도커 내부의 가상IP를 가지므로 내 컴퓨터(호스트) 외부에서는 직접 접근할 수 없다.
따라서 호스트의 8080포트와 컨테이너의 80포트를 매핑해 브라우저 접속이 가능하게 한다.

```
myweb % curl http://localhost:8080  
<h1>Hello, Docker World! (Version 1)</h1>
```

----

## 8. Docker 볼륨 영속성 검증

### 8-1 Docker 볼륨을 생성하고 컨테이너에 연결
```
#볼륨 생성
% docker volume create myvolume1
myvolume1
#컨테이너에 연결
% docker run -d --name voltest1 -v myvolume1:/data ubuntu bash -c "echo 'i want to go home.' > /data/result.txt && sleep 3600" 
d7c70c67897f230e889d59dec1c6e5a179dc91bd5d211c8085e303f7a26551b9
```

### 8-2 컨테이너 삭제 전/후로 데이터가 유지됨을 증명
```
#컨테이너 강제 삭제
% docker rm -f voltest1
voltest1
#새 컨테이너의 /data 경로와 마운트
% docker run --rm -v myvolume1:/data ubuntu cat /data/result.txt
i want to go home.
```

----

## 9. Git 설정 및 GitHub 연동

```
#환경 설정 확인
% git config --list 
credential.helper=osxkeychain
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=https://github.com/jaeeun-y/jaeeunsRepository.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
```
---

# 트래블슈팅
1. chmod 명령어로 디렉터리의 권한을 변경할 때 권한이 변경되지 않는 문제가 발생함.
[가설] 디렉터리 내부의 하위 파일들 때문에 권한이 변경되지 않을 것이다.  
 
 -> -R 옵션을 사용했더니 내부의 모든 파일과 디렉터리의 권한이 바뀌었다.  

2. docker status로 리소스를 확인하기 위해 stop으로 멈춘 컨테이너를 계속 run명령으로 살리려 시도함.  
 [가설] 이미 죽은 컨테이너라서 run명령이 먹지 않을 것이다.
   
-> run이 아닌 start명령으로 사용했더니 해결되었다.
     
3. status가 Exited인 hello-world가 rm 명령으로 지워지지 않는 문제가 발생함.  
 [가설] rm명령은 컨테이너를 삭제하는 명령어라 지워지지 않았을 것이다.

-> docker ps -a 를 통해 (멈춘 컨테이너 포함) 컨테이너 목록을 확인했더니, hello-world 이미지를 사용한 컨테이너의 고유 이름이 도커가 임의로 부여한 clever_habit 이었음을 확인함.

-> docker rm clever_habit 명령어로 해결되었다.
