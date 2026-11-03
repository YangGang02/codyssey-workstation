# 개발 작업실 꾸미기

루키마리너 1기_양민기

이번 과제에서는 터미널로 파일과 폴더를 다루고 Docker로 웹 서버를 실행했다. 바인드 마운트와 볼륨 동작을 확인하고, 작업 파일과 수행 기록을 GitHub에 올렸다.

아래에는 사용한 명령어와 캡처에 나온 핵심 출력을 적었다. 긴 출력은 일부만 옮겼다. 실제 로그는 각 항목의 캡처에서 확인할 수 있다.

## 1. 실행 환경

- OS: macOS 26.6.2 / Apple Silicon arm64
- Shell / 터미널: zsh / 맥 기본 터미널
- Docker: 29.9.2 / Docker Desktop
- Git: 2.39.5 (Apple Git-154)

[환경 확인 명령과 출력](docs/logs/01-environment-readable.txt)

## 2. 수행 체크리스트

- [x] 터미널 기본 조작
- [x] 파일·폴더 권한 변경
- [x] Docker 설치·점검 및 운영 명령
- [x] hello-world, Ubuntu 실행 및 종료·유지 비교
- [x] Dockerfile 작성·빌드·웹 서버 실행
- [x] 포트 매핑과 브라우저 접속
- [x] 바인드 마운트 변경 반영
- [x] 볼륨의 컨테이너 삭제 전후 데이터 유지
- [x] Git 설정 및 GitHub·VSCode 연동

### 폴더 구성

~~~text
codyssey-workstation/
├── Dockerfile          # 웹 서버 이미지 빌드 설정
├── README.md           # 수행 내용과 검증 결과
├── .gitignore          # Git에서 제외할 파일 설정
├── practice/
│   └── memo.txt        # 파일 조작·권한 연습
├── site/
│   └── index.html      # 웹 서버에서 보여줄 페이지
└── docs/
    ├── images/         # 실습 화면 캡처
    └── logs/           # 명령어와 출력 기록
~~~

웹 페이지는 site, 연습 파일은 practice, 증빙 자료는 docs에 모았다.

## 3. 터미널 기본 조작

~~~text
$ pwd                                       # 현재 위치 확인
/Users/mingiyomi
$ ls -la                                      # 숨김 파일을 포함한 목록 확인
$ mkdir -p ~/Documents/codyssey-workstation    # 과제 폴더 생성
$ cd ~/Documents/codyssey-workstation          # 폴더 이동
$ pwd
/Users/mingiyomi/Documents/codyssey-workstation
$ mkdir practice                              # 연습 폴더 생성
$ touch practice/memo.txt                     # 빈 파일 생성
$ echo "My first Codyssey mission" > practice/memo.txt  # 내용 저장
$ cat practice/memo.txt                       # 파일 내용 확인
My first Codyssey mission
$ cp practice/memo.txt practice/memo-copy.txt  # 파일 복사
$ mv practice/memo-copy.txt practice/memo-backup.txt    # 이름 변경
$ rm practice/memo-backup.txt                  # 파일 삭제
$ ls -la practice
total 8
drwxr-xr-x  3 mingiyomi staff 96 11월 1 14:56 .
drwxr-xr-x@ 3 mingiyomi staff 96 11월 1 14:53 ..
-rw-r--r--  1 mingiyomi staff 26 11월 1 14:54 memo.txt
~~~

복사한 파일의 이름을 바꾸고 삭제한 뒤 memo.txt만 남은 것을 확인했다.

[위치·숨김 파일·폴더 생성](docs/images/21-terminal-location-and-hidden-files.png) · [파일 조작 결과](docs/images/22-terminal-file-operations.png)

## 4. 권한 변경

~~~text
$ ls -l practice/memo.txt          # 파일 권한 확인
-rw-r--r-- 1 mingiyomi staff 26 11월 1 14:54 practice/memo.txt
$ chmod 600 practice/memo.txt      # 소유자만 읽기·쓰기
$ ls -l practice/memo.txt
-rw------- 1 mingiyomi staff 26 11월 1 14:54 practice/memo.txt
$ chmod 644 practice/memo.txt      # 소유자 읽기·쓰기, 나머지 읽기
$ ls -l practice/memo.txt
-rw-r--r-- 1 mingiyomi staff 26 11월 1 14:54 practice/memo.txt
$ ls -ld practice                 # 폴더 자체의 권한 확인
drwxr-xr-x 3 mingiyomi staff 96 11월 1 14:56 practice
$ chmod 700 practice              # 소유자만 폴더 사용
$ ls -ld practice
drwx------ 3 mingiyomi staff 96 11월 1 14:56 practice
$ chmod 755 practice              # 소유자 전체 권한, 나머지 읽기·진입
$ ls -ld practice
drwxr-xr-x 3 mingiyomi staff 96 11월 1 14:56 practice
~~~

각 변경 뒤 ls로 확인했다. 파일은 644→600→644, 폴더는 755→700→755로 바뀌었다. r=읽기(4), w=쓰기(2), x=실행·폴더 진입(1)을 더해 소유자·그룹·그 외 사용자 순서로 적는다.

[권한 변경 전후 로그](docs/images/14-file-directory-permissions.png)

## 5. Docker 설치·운영과 Ubuntu

Docker Desktop을 설치하고 켜둔 상태에서 명령어를 실행했다.

이미지는 프로그램과 파일이 담긴 읽기 전용 실행 틀이고, 컨테이너는 그 이미지로 만든 실행 환경이다. docker build는 이미지를 만들고 docker run은 컨테이너를 생성해 실행한다. 컨테이너 안에서 파일을 바꿔도 원본 이미지는 바뀌지 않는다. 컨테이너는 중지 후 다시 시작할 수 있지만, 삭제하면 그 컨테이너의 쓰기 계층도 사라진다. 볼륨이나 바인드 마운트에 저장한 데이터는 별도로 남는다. ([Docker 이미지 설명](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/))

~~~text
$ docker --version                         # Docker 버전 확인
Docker version 29.9.2, build a82e528
$ docker info                              # 엔진 동작과 환경 확인
Server Version: 29.9.2
Operating System: Docker Desktop
OSType: linux
Architecture: aarch64
$ docker run --name codyssey-hello hello-world  # 이미지 다운로드 및 컨테이너 실행
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
$ docker images                            # 이미지 목록
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   c567b231f656   22.6kB       10.3kB         U
$ docker ps                                # 실행 중인 컨테이너
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
$ docker ps -a                             # 종료된 컨테이너까지 확인
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                          PORTS   NAMES
15b2cf3ac944   hello-world   "/hello"   About a minute ago   Exited (0) About a minute ago           codyssey-hello
$ docker logs codyssey-hello               # 컨테이너 출력 확인
Hello from Docker!
~~~

hello-world에서 “Hello from Docker!”가 나왔다. 실행 후 종료되어 ps에는 없고 ps -a에는 Exited (0)으로 남았다.

[설치](docs/images/19-docker-install.png) · [엔진 점검 앞부분](docs/images/16-docker-info-client.png) · [뒷부분](docs/images/15-docker-info-server.png) · [hello-world](docs/images/08-hello-world.png) · [images·ps·logs](docs/images/09-images-ps-logs.png)

Ubuntu에서 사용한 명령어다.

~~~text
$ docker run -it --name codyssey-ubuntu ubuntu:24.04 bash  # Ubuntu 내부 진입
# 컨테이너 안에서 실행
root@3d676117715a:/# pwd
/
root@3d676117715a:/# ls
bin boot dev etc home lib media mnt opt proc root run sbin srv sys tmp usr var
root@3d676117715a:/# echo "Hello from Ubuntu container"
Hello from Ubuntu container
root@3d676117715a:/# cat /etc/os-release  # Ubuntu 버전 확인
PRETTY_NAME="Ubuntu 24.04.5 LTS"
VERSION_ID="24.04"
root@3d676117715a:/# exit                  # 주 프로그램 bash 종료
exit
# 맥 터미널에서 실행
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND   CREATED         STATUS                      PORTS   NAMES
3d676117715a   ubuntu:24.04   "bash"    2 minutes ago   Exited (0) 8 seconds ago            codyssey-ubuntu
$ docker start codyssey-ubuntu            # 기존 컨테이너 재시작
codyssey-ubuntu
$ docker exec -it codyssey-ubuntu bash       # 실행 중인 컨테이너에 추가 bash 실행
# 컨테이너 안에서 실행
root@3d676117715a:/# echo "Connected using docker exec"
Connected using docker exec
root@3d676117715a:/# exit                  # 추가 bash에서 나오기
exit
# 맥 터미널에서 실행
$ docker ps                              # 계속 실행 중인지 확인
CONTAINER ID   IMAGE          COMMAND   CREATED         STATUS          PORTS   NAMES
3d676117715a   ubuntu:24.04   "bash"    4 minutes ago   Up 26 seconds           codyssey-ubuntu
$ docker stats --no-stream codyssey-ubuntu  # CPU·메모리 사용량 확인
CONTAINER ID   NAME              CPU %   MEM USAGE / LIMIT   MEM %   NET I/O       BLOCK I/O   PIDS
3d676117715a   codyssey-ubuntu   0.00%   1.992MiB / 7.748GiB  0.03%   1.17kB / 126B 0B / 0B     1
$ docker stop codyssey-ubuntu             # 컨테이너 중지
codyssey-ubuntu
$ docker ps -a                           # 중지 상태 확인
CONTAINER ID   IMAGE          COMMAND   CREATED          STATUS                       PORTS   NAMES
3d676117715a   ubuntu:24.04   "bash"    13 minutes ago   Exited (137) 5 seconds ago           codyssey-ubuntu
~~~

처음 bash를 exit했을 때는 컨테이너가 종료됐다. exec로 들어갔다 나왔을 때는 컨테이너가 계속 실행됐다. CPU와 메모리 사용량을 확인하고 컨테이너를 중지했다.

[Ubuntu 내부 명령·종료·유지 비교](docs/images/20-ubuntu-exit-exec.png) · [stats](docs/images/03-container-stats.png) · [stop](docs/images/04-container-stop.png)

## 6. Dockerfile과 웹 서버

nginx:alpine을 기반으로 기본 페이지를 작성한 HTML로 교체했다. EXPOSE는 사용할 포트를 표시한다.

~~~dockerfile
FROM nginx:alpine
COPY site/ /usr/share/nginx/html/
EXPOSE 80
~~~

~~~text
$ docker build -t codyssey-web:1.0 .      # 현재 폴더의 Dockerfile로 이미지 빌드
[+] Building 4.9s (8/8) FINISHED
=> [2/2] COPY site/ /usr/share/nginx/html/
=> => naming to docker.io/library/codyssey-web:1.0
$ docker run -d --name codyssey-web -p 8080:80 codyssey-web:1.0  # 백그라운드 실행
5a27e8f4c659a4142e45cd5a0883ae38d7100421446f0f6a5957b6672b8aeb4f
$ curl -i http://localhost:8080           # HTTP 응답 확인
HTTP/1.1 200 OK
Server: nginx/1.31.8
Content-Type: text/html
...
<h1>대전 코디세이 1기 개발 작업실</h1>
<p>이번 목표: 직접 실행하고, 확인하고, 기록하기!</p>
~~~

이미지 빌드와 컨테이너 실행이 완료됐고 200 OK 응답이 나왔다. 8080:80은 맥의 8080번 포트를 컨테이너의 80번 포트에 연결한다.

이 실습의 기본 bridge 네트워크에서는 컨테이너마다 네트워크 네임스페이스가 분리되어 IP·포트 공간을 따로 사용한다. 그래서 컨테이너 안의 localhost:80과 맥의 localhost:80은 다르다. 네임스페이스가 네트워크 공간을 나누는 기능이라면, -p 8080:80은 나뉜 공간 사이에 접속 경로를 만드는 포트 매핑이다. 맥에서는 Docker Desktop이 맥의 8080번으로 받은 요청을 Linux 가상 머신 안의 컨테이너 80번으로 전달한다. ([Docker 네트워크 설명](https://docs.docker.com/engine/network/), [Docker Desktop 포트 연결](https://docs.docker.com/desktop/features/networking/#how-exposed-ports-work))

[Dockerfile](Dockerfile) · [HTML 소스](site/index.html) · [빌드 결과](docs/images/18-web-source-and-build.png) · [실행·포트·응답 결과](docs/images/06-web-run-and-curl.png)

![8080 브라우저 접속](docs/images/11-browser-8080-menu.png)

## 7. 바인드 마운트

맥의 site 폴더를 컨테이너에 연결하고 HTML을 수정했다.

~~~text
$ docker run -d --name codyssey-web-bind -p 8081:80 -v "$(pwd)/site:/usr/share/nginx/html:ro" codyssey-web:1.0
abf2aaa7f73402900136b759ea50a889486e12e420a02b18c5b77cd1747cbe0f
$ curl -i http://localhost:8081  # 수정 전 응답 확인
HTTP/1.1 200 OK
...
<p>이번 목표: 직접 실행하고, 확인하고, 기록하기!</p>
$ nano site/index.html          # 맥의 HTML 수정
$ curl -i http://localhost:8081  # 수정 후 응답 확인
HTTP/1.1 200 OK
...
<p>바인드 마운트 실습: 파일 수정이 바로 반영됩니다.</p>
$ curl -i http://localhost:8080  # 이미지에 복사한 원래 파일과 비교
HTTP/1.1 200 OK
...
<p>이번 목표: 직접 실행하고, 확인하고, 기록하기!</p>
~~~

재빌드나 재시작 없이 8081의 문장이 바뀌었고 8080은 원래 문장이 유지됐다.

[실행 명령·수정 전후 로그](docs/images/01-bind-mount-before-after.png) · [8080 비교 결과](docs/images/02-image-copy-original.png)

![8081 브라우저 접속](docs/images/12-browser-8081-menu.png)

## 8. 볼륨

~~~text
$ docker volume create codyssey-data     # 볼륨 생성
codyssey-data
$ docker run -d --name codyssey-volume-before -v codyssey-data:/data ubuntu:24.04 sleep infinity  # 볼륨 연결
5963ddc061ba972801304c2db6d49192b4f228feb2424fcabdd28b766cb3d4be
$ docker exec codyssey-volume-before bash -c 'echo "Codyssey volume persistence test" > /data/note.txt'  # 파일 저장
$ docker exec codyssey-volume-before cat /data/note.txt  # 삭제 전 내용 확인
Codyssey volume persistence test
$ docker rm -f codyssey-volume-before     # 컨테이너 삭제
codyssey-volume-before
$ docker ps -a                           # 이전 컨테이너가 사라졌는지 확인
...
$ docker volume ls                       # 볼륨이 남아 있는지 확인
DRIVER   VOLUME NAME
local    codyssey-data
$ docker run -d --name codyssey-volume-after -v codyssey-data:/data ubuntu:24.04 sleep infinity  # 새 컨테이너에 같은 볼륨 연결
56313dd19f83cb98b2afd798eb9ff3582e92ccc5220dd0d6efba2ff89538c42d
$ docker exec codyssey-volume-after cat /data/note.txt  # 삭제 후 내용 확인
Codyssey volume persistence test
~~~

삭제 전후 모두 “Codyssey volume persistence test”가 나왔다. 컨테이너를 지워도 볼륨에 파일이 남았다.

[볼륨 생성·연결·컨테이너 삭제 전후 로그](docs/images/05-volume-persistence.png)

## 9. Git 설정과 GitHub·VSCode 연동

~~~text
$ git config --global init.defaultBranch main  # 기본 브랜치 설정
$ git config --global --get init.defaultBranch # 설정값 확인
main
$ git init                                    # 로컬 저장소 생성
$ git config user.name "양민기"                 # 작성자 이름 설정
$ git config user.email "didthq100@gmail.com"   # 작성자 이메일 설정
$ git config --local --list                    # 저장소 설정 확인
user.name=양민기
user.email=didthq100@gmail.com
$ git add Dockerfile site practice .gitignore # 커밋에 넣을 파일 선택
$ git commit -m "터미널 실습과 Docker 웹 서버 추가"  # 변경 이력 저장
[main (root-commit) 9932f75] 터미널 실습과 Docker 웹 서버 추가
4 files changed, 18 insertions(+)
$ git remote add origin https://github.com/didthq100-arch/codyssey-workstation.git # 원격 저장소 연결
$ git remote -v                               # 연결 주소 확인
$ git push -u origin main                     # GitHub에 업로드
To https://github.com/didthq100-arch/codyssey-workstation.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
~~~

Git으로 내 컴퓨터에 변경 이력을 저장하고, VSCode 터미널에서 GitHub에 업로드했다.

[Git 설정](docs/images/13-git-config-and-stage.png) · [첫 커밋](docs/images/17-git-first-commit.png) · [VSCode 연결·업로드](docs/images/23-vscode-github-push.png) · [업로드 로그](docs/logs/02-github-readable.txt)

## 10. 하면서 막혔던 부분

1. **Docker 명령어를 찾지 못함**
   - 문제: 설치 후 command not found가 나왔다.
   - 원인 가설: 기존 터미널에 새 경로 설정이 반영되지 않았을 수 있었다.
   - 확인: exec zsh -l 실행 후 docker --version과 command -v docker를 입력했다.
   - 해결: 버전과 실행 파일 경로가 출력됐고 명령어가 동작했다.
   - [실제 로그](docs/images/07-docker-path-fix.png)

2. **HTML 파일을 찾지 못함**
   - 문제: cat /index.html에서 파일이 없다고 나왔다.
   - 원인 가설: 파일 경로를 잘못 입력했다.
   - 확인: 작성할 때 사용한 상대 경로인 cat site/index.html로 읽었다.
   - 해결: HTML 내용이 출력됐다. /로 시작하는 절대 경로와 현재 폴더 기준 상대 경로의 차이를 확인했다.
   - [실제 로그](docs/images/18-web-source-and-build.png)

## 11. 다시 실행할 때

Docker Desktop을 켜고 내려받은 저장소 폴더에서 6~8절의 명령을 실행한다. 같은 컨테이너 이름이나 포트가 있으면 바꾸고, 개인 PC 경로는 자신의 저장소 경로로 바꾼다.

바인드 마운트는 이미지를 빌드한 다음 HTML 문장을 수정해서 비교한다.

### 포트가 이미 사용 중일 때

다음은 충돌이 생겼을 때 사용할 확인·해제 절차다.

1. lsof로 맥의 8080번 포트를 사용하는 프로그램과 PID를 확인한다. docker ps에서는 컨테이너 이름과 포트 매핑을 확인한다.

~~~bash
lsof -nP -iTCP:8080 -sTCP:LISTEN
docker ps --format 'table {{.Names}}\t{{.Ports}}'
~~~

2. 점유한 것이 내 실습 컨테이너 codyssey-web이라면 아래처럼 중지한다. 다른 프로그램이라면 해당 앱에서 서버를 정상 종료한다. Docker Desktop 프로세스로 표시되면 Docker Desktop 자체를 종료하기보다 해당 컨테이너를 찾는다.

~~~bash
docker stop codyssey-web
lsof -nP -iTCP:8080 -sTCP:LISTEN
~~~

3. 다시 조회했을 때 점유 프로세스가 없으면 포트가 해제된 것이다. 다른 프로그램을 계속 써야 한다면 새 컨테이너는 -p 8082:80처럼 빈 호스트 포트로 실행한다.

### 데이터 백업

다음은 필요할 때 사용할 백업 절차다. 파일을 쓰는 프로그램은 먼저 중지하고 백업한다.

~~~bash
# 과제 저장소 폴더에서 실행: 맥의 site 폴더 백업
mkdir -p "$HOME/Documents/codyssey-backup"
tar -czf "$HOME/Documents/codyssey-backup/site.tar.gz" site

# codyssey-data 볼륨이 있는지 먼저 확인한 뒤 백업
docker volume inspect codyssey-data
docker run --rm -v codyssey-data:/data:ro \
  -v "$HOME/Documents/codyssey-backup:/backup" \
  ubuntu:24.04 tar -czf /backup/codyssey-data.tar.gz -C /data .

# 압축 파일의 목록을 읽어 백업 내용 확인
tar -tzf "$HOME/Documents/codyssey-backup/site.tar.gz"
tar -tzf "$HOME/Documents/codyssey-backup/codyssey-data.tar.gz"
~~~

볼륨은 읽기 전용으로 연결하고, 압축 파일은 저장소 밖의 codyssey-backup 폴더에 보관한다. 중요한 백업은 외장 저장 장치 등에도 복사한다. ([Docker 볼륨 백업 안내](https://docs.docker.com/engine/storage/volumes/#back-up-restore-or-migrate-data-volumes))

[전체 캡처 폴더](docs/images/)
