# 개발 작업실 꾸미기

루키마리너 1기_양민기

이번 과제에서는 터미널로 파일과 폴더를 다루고 Docker로 웹 서버를 실행했다. 바인드 마운트와 볼륨 동작을 확인하고, 작업 파일과 수행 기록을 GitHub에 올렸다.

아래에는 사용한 명령어와 기능을 적었다. 실제 실행 로그는 각 항목의 캡처에서 확인할 수 있다.

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

## 3. 터미널 기본 조작

~~~bash
pwd                                         # 현재 위치 확인
ls -la                                      # 숨김 파일을 포함한 목록 확인
mkdir -p ~/Documents/codyssey-workstation    # 과제 폴더 생성
cd ~/Documents/codyssey-workstation          # 폴더 이동
mkdir practice                              # 연습 폴더 생성
touch practice/memo.txt                     # 빈 파일 생성
echo "My first Codyssey mission" > practice/memo.txt  # 내용 저장
cat practice/memo.txt                       # 파일 내용 확인
cp practice/memo.txt practice/memo-copy.txt  # 파일 복사
mv practice/memo-copy.txt practice/memo-backup.txt    # 이름 변경
rm practice/memo-backup.txt                  # 파일 삭제
~~~

복사한 파일의 이름을 바꾸고 삭제한 뒤 memo.txt만 남은 것을 확인했다.

[위치·숨김 파일·폴더 생성](docs/images/21-terminal-location-and-hidden-files.png) · [파일 조작 결과](docs/images/22-terminal-file-operations.png)

## 4. 권한 변경

~~~bash
ls -l practice/memo.txt          # 파일 권한 확인
chmod 600 practice/memo.txt      # 소유자만 읽기·쓰기
chmod 644 practice/memo.txt      # 소유자 읽기·쓰기, 나머지 읽기
ls -ld practice                 # 폴더 자체의 권한 확인
chmod 700 practice              # 소유자만 폴더 사용
chmod 755 practice              # 소유자 전체 권한, 나머지 읽기·진입
~~~

각 변경 뒤 ls로 확인했다. 파일은 644→600→644, 폴더는 755→700→755로 바뀌었다. r=읽기(4), w=쓰기(2), x=실행·폴더 진입(1)을 더해 소유자·그룹·그 외 사용자 순서로 적는다.

[권한 변경 전후 로그](docs/images/14-file-directory-permissions.png)

## 5. Docker 설치·운영과 Ubuntu

Docker Desktop을 설치하고 켜둔 상태에서 명령어를 실행했다.

~~~bash
docker --version                         # Docker 버전 확인
docker info                              # 엔진 동작과 환경 확인
docker run --name codyssey-hello hello-world  # 이미지 다운로드 및 컨테이너 실행
docker images                            # 이미지 목록
docker ps                                # 실행 중인 컨테이너
docker ps -a                             # 종료된 컨테이너까지 확인
docker logs codyssey-hello               # 컨테이너 출력 확인
~~~

hello-world에서 “Hello from Docker!”가 나왔다. 실행 후 종료되어 ps에는 없고 ps -a에는 Exited (0)으로 남았다.

[설치](docs/images/19-docker-install.png) · [엔진 점검 앞부분](docs/images/16-docker-info-client.png) · [뒷부분](docs/images/15-docker-info-server.png) · [hello-world](docs/images/08-hello-world.png) · [images·ps·logs](docs/images/09-images-ps-logs.png)

Ubuntu에서 사용한 명령어다.

~~~bash
docker run -it --name codyssey-ubuntu ubuntu:24.04 bash  # Ubuntu 내부 진입
# 컨테이너 안에서 실행
pwd
ls
echo "Hello from Ubuntu container"
cat /etc/os-release                      # Ubuntu 버전 확인
exit                                     # 주 프로그램 bash 종료
# 맥 터미널에서 실행
docker start codyssey-ubuntu              # 기존 컨테이너 재시작
docker exec -it codyssey-ubuntu bash       # 실행 중인 컨테이너에 추가 bash 실행
# 컨테이너 안에서 실행
echo "Connected using docker exec"
exit                                     # 추가 bash에서 나오기
# 맥 터미널에서 실행
docker ps                                # 계속 실행 중인지 확인
docker stats --no-stream codyssey-ubuntu  # CPU·메모리 사용량 확인
docker stop codyssey-ubuntu               # 컨테이너 중지
docker ps -a                             # 중지 상태 확인
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

~~~bash
docker build -t codyssey-web:1.0 .        # 현재 폴더의 Dockerfile로 이미지 빌드
docker run -d --name codyssey-web -p 8080:80 codyssey-web:1.0  # 백그라운드 실행
curl -i http://localhost:8080             # HTTP 응답 확인
~~~

이미지 빌드와 컨테이너 실행이 완료됐고 200 OK 응답이 나왔다. 8080:80은 맥의 8080번 포트를 컨테이너의 80번 포트에 연결한다.

[Dockerfile](Dockerfile) · [HTML 소스](site/index.html) · [빌드 결과](docs/images/18-web-source-and-build.png) · [실행·포트·응답 결과](docs/images/06-web-run-and-curl.png)

![8080 브라우저 접속](docs/images/11-browser-8080-menu.png)

## 7. 바인드 마운트

맥의 site 폴더를 컨테이너에 연결하고 HTML을 수정했다.

~~~bash
docker run -d --name codyssey-web-bind -p 8081:80 -v "$(pwd)/site:/usr/share/nginx/html:ro" codyssey-web:1.0
curl -i http://localhost:8081    # 수정 전 응답 확인
nano site/index.html            # 맥의 HTML 수정
curl -i http://localhost:8081    # 수정 후 응답 확인
curl -i http://localhost:8080    # 이미지에 복사한 원래 파일과 비교
~~~

재빌드나 재시작 없이 8081의 문장이 바뀌었고 8080은 원래 문장이 유지됐다.

[실행 명령·수정 전후 로그](docs/images/01-bind-mount-before-after.png) · [8080 비교 결과](docs/images/02-image-copy-original.png)

![8081 브라우저 접속](docs/images/12-browser-8081-menu.png)

## 8. 볼륨

~~~bash
docker volume create codyssey-data       # 볼륨 생성
docker run -d --name codyssey-volume-before -v codyssey-data:/data ubuntu:24.04 sleep infinity  # 볼륨 연결
docker exec codyssey-volume-before bash -c 'echo "Codyssey volume persistence test" > /data/note.txt'  # 파일 저장
docker exec codyssey-volume-before cat /data/note.txt  # 삭제 전 내용 확인
docker rm -f codyssey-volume-before       # 컨테이너 삭제
docker ps -a                             # 이전 컨테이너가 사라졌는지 확인
docker volume ls                         # 볼륨이 남아 있는지 확인
docker run -d --name codyssey-volume-after -v codyssey-data:/data ubuntu:24.04 sleep infinity  # 새 컨테이너에 같은 볼륨 연결
docker exec codyssey-volume-after cat /data/note.txt   # 삭제 후 내용 확인
~~~

삭제 전후 모두 “Codyssey volume persistence test”가 나왔다. 컨테이너를 지워도 볼륨에 파일이 남았다.

[볼륨 생성·연결·컨테이너 삭제 전후 로그](docs/images/05-volume-persistence.png)

## 9. Git 설정과 GitHub·VSCode 연동

~~~bash
git config --global init.defaultBranch main  # 기본 브랜치 설정
git config --global --get init.defaultBranch # 설정값 확인
git init                                    # 로컬 저장소 생성
git config user.name "양민기"                 # 작성자 이름 설정
git config user.email "didthq100@gmail.com"   # 작성자 이메일 설정
git config --local --list                    # 저장소 설정 확인
git add Dockerfile site practice .gitignore # 커밋에 넣을 파일 선택
git commit -m "터미널 실습과 Docker 웹 서버 추가"  # 변경 이력 저장
git remote add origin https://github.com/didthq100-arch/codyssey-workstation.git # 원격 저장소 연결
git remote -v                               # 연결 주소 확인
git push -u origin main                     # GitHub에 업로드
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

[전체 캡처 폴더](docs/images/)
