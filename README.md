# 개발자용 작업실 꾸미기: Docker/Linux/Git 개발 워크스테이션 (WSL2)

## 1. 프로젝트 개요
터미널 조작, 파일 권한, Docker(이미지/컨테이너/포트/바인드 마운트/볼륨), Dockerfile 기반 웹 서버, Git/GitHub/VSCode 연동까지 직접 수행하고 그 과정을 검증 가능한 로그와 증거로 남긴 개발 워크스테이션 구축 미션이다.
이 README만 읽어도 전체 수행 내용을 파악할 수 있도록 명령, 출력, 증거 위치를 모두 기록했다.

## 2. 실행 환경
Windows에서 WSL2(Ubuntu)를 쓰고, Docker는 **Docker Desktop(WSL2 백엔드)** 으로 사용했다.

| 항목 | 내용 |
|---|---|
| OS | Windows (버전: `winver` 결과) + WSL2 (Ubuntu 26.04.1 LTS, 커널 6.18.54.1-microsoft-standard-WSL2) |
| 쉘 / 터미널 | zsh (Oh My Zsh) / Windows Terminal |
| Docker | Docker Desktop (WSL2 백엔드), Docker version 29.9.2, build a82e528 |
| Git | git version 2.53.0 |
| 에디터 | VS Code + WSL 확장 (1.143.1) |

```bash
wsl --list --verbose     # Windows 쪽 터미널
lsb_release -a
uname -r
echo $SHELL
docker --version
git --version
code --version
```
```text
(출력 붙여넣기)
```

## 3. 수행 항목 체크리스트
- [ ] 터미널 조작 (디렉토리 생성/이동/조회)
- [ ] 파일·디렉토리 권한 변경 (chmod)
- [ ] Docker 설치·점검
- [ ] Docker 운영 명령 (images / ps -a / logs / stats)
- [ ] hello-world 실행
- [ ] Ubuntu 컨테이너 진입 (exec)
- [ ] 웹 서버 소스코드 + Dockerfile + 빌드
- [ ] 포트 매핑 접속 (2회, 브라우저 주소창 포함 스크린샷)
- [ ] 바인드 마운트 (호스트 변경 전/후)
- [ ] Docker 볼륨 영속성 (컨테이너 삭제 전/후)
- [ ] Git 사용자 정보·기본 브랜치 설정
- [ ] VSCode GitHub 로그인·저장소 연동
- [ ] GitHub 저장소 푸시
- [ ] 트러블슈팅 2건 이상
- [ ] 민감정보(토큰, ID/PW, 이메일) 제거 확인

## 4. 검증 방법 및 결과 위치

| 항목 | 검증 명령 | 확인 포인트 | 결과 위치 |
|---|---|---|---|
| 환경 | `docker --version`, `git --version` | 버전 출력 | [§2](#2-실행-환경) |
| 터미널 | `pwd`, `ls -la` | 작업 디렉토리 생성 | [§5-1](#5-1-터미널-조작) |
| 권한 | `ls -l test.sh` | `-rw-` → `-rwx` 변화 | [§5-2](#5-2-권한-변경) |
| Docker 점검 | `docker info` | Server 정보 출력 | [§6-1](#6-1-설치점검) |
| 운영 명령 | `docker images/ps -a/logs/stats` | 이미지·컨테이너·로그·리소스 | [§6-2](#6-2-운영-명령) |
| 웹 서버 | `docker build`, `docker run` | 빌드 성공, 컨테이너 Up | [§7](#7-dockerfile-기반-웹-서버) |
| 포트 매핑 | 브라우저 `localhost:8000/8001` | 페이지 표시 | [§8](#8-포트-매핑-증거) |
| 바인드 마운트 | 호스트 수정 후 `docker exec cat` | 변경 즉시 반영 | [§9](#9-바인드-마운트) |
| 볼륨 | 컨테이너 삭제 후 재연결 `cat` | 데이터 유지 | [§10](#10-docker-볼륨-영속성) |
| Git/GitHub | `git config --global --list`, `git log` | 설정·커밋 이력 | [§11](#11-git--github--vscode) |

## 5. 터미널 조작 로그
### 5-1. 터미널 조작
```bash
mkdir -p ~/docker-workspace && cd ~/docker-workspace
pwd
ls -la
```
```text
(출력 붙여넣기)
```
### 5-2. 권한 변경
```bash
touch test.sh
ls -l test.sh        # 변경 전
chmod +x test.sh
ls -l test.sh        # 변경 후
```
```text
(출력 붙여넣기)
```
> 권한 숫자: r=4, w=2, x=1 → `chmod 755` = 소유자 rwx, 그룹 r-x, 기타 r-x

## 6. Docker 운영/검증 로그
### 6-1. 설치·점검
```bash
docker --version
docker info
docker run hello-world
```
```text
(출력 붙여넣기)
```
### 6-2. 운영 명령
```bash
docker images
docker ps -a
docker logs <컨테이너명>
docker stats --no-stream
```
```text
(출력 붙여넣기)
```
### 6-3. Ubuntu 컨테이너 진입
```bash
docker run -d --name ubuntu-test ubuntu sleep 1000
docker exec ubuntu-test ls -la
docker exec -it ubuntu-test bash   # 내부에서 exit
docker stop ubuntu-test && docker rm ubuntu-test
```
```text
(출력 붙여넣기)
```

## 7. Dockerfile 기반 웹 서버
구조
```text
docker-workspace/
├── app/app.py
├── data/
├── Dockerfile
└── README.md
```
`app/app.py`
```python
from http.server import HTTPServer, BaseHTTPRequestHandler

class MyHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(b'<h1>Hello from Docker!</h1>')

if __name__ == '__main__':
    HTTPServer(('0.0.0.0', 8000), MyHandler).serve_forever()
```
`Dockerfile`
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY app/app.py .
EXPOSE 8000
CMD ["python", "app.py"]
```
빌드·실행
```bash
docker build -t my-web-server:1.0 .
docker images | grep my-web-server
```
```text
(빌드 로그 붙여넣기)
```

## 8. 포트 매핑 증거
```bash
docker run -d -p 8000:8000 --name web-1 my-web-server:1.0
docker ps
```
![8000 접속](docs/screenshots/port-8000.png)

```bash
docker run -d -p 8001:8000 --name web-2 my-web-server:1.0
docker ps
```
![8001 접속](docs/screenshots/port-8001.png)

> 스크린샷에 브라우저 주소창이 보여야 한다.

## 9. 바인드 마운트
```bash
mkdir -p data && echo "before" > data/test.txt
docker run -d -v "$(pwd)/data:/data" --name bind-test ubuntu sleep 1000
docker exec bind-test cat /data/test.txt     # 변경 전
echo "after (host에서 수정)" > data/test.txt
docker exec bind-test cat /data/test.txt     # 변경 후
```
```text
(출력 붙여넣기: 전/후 비교)
```

## 10. Docker 볼륨 영속성
```bash
docker volume create my-volume
docker run -d -v my-volume:/data --name vol-1 ubuntu sleep 1000
docker exec vol-1 bash -c "echo persistent > /data/p.txt"
docker exec vol-1 cat /data/p.txt                 # 삭제 전
docker rm -f vol-1                                # 컨테이너 삭제
docker run -d -v my-volume:/data --name vol-2 ubuntu sleep 1000
docker exec vol-2 cat /data/p.txt                 # 삭제 후(유지 확인)
docker volume inspect my-volume
```
```text
(출력 붙여넣기: 전/후 비교)
```

## 11. Git · GitHub · VSCode
```bash
git config --global user.name "이름"
git config --global init.defaultBranch main
git config --global --list      # 이메일 등 개인정보는 가리고 첨부
git init && git add . && git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<계정>/<저장소>.git
git push -u origin main
git log --oneline
```
```text
(출력 붙여넣기)
```
![VSCode GitHub 로그인·저장소 연동](docs/screenshots/vscode-github.png)

## 12. 트러블슈팅
> 실제로 겪은 문제만 기록한다. (아래는 작성 형식)

### 문제 1: (제목)
- **문제**: 
- **원인 가설**: 
- **확인**: (사용한 명령과 출력)
- **해결/대안**: 

### 문제 2: (제목)
- **문제**: 
- **원인 가설**: 
- **확인**: 
- **해결/대안**: 

## 13. 보안 체크
- [ ] 토큰, ID/PW, 이메일 등 민감정보가 로그·스크린샷에 없음
