# 개발 워크스테이션 구축 (터미널 · Docker · Git/GitHub)

## 0. 프로젝트 개요

- **미션명**: 내 컴퓨터에 개발자용 '작업실' 꾸미기
- **목표**: 터미널(리눅스 CLI), Docker, Git/GitHub를 직접 세팅하고, 실행 결과(로그 / 접속 / 데이터 유지)로 핵심 흐름을 검증한다. 같은 서비스를 여러 번 실행해도 재현되는 환경을 만드는 사고방식을 익힌다.
- **수행 범위**
  - 개발 환경 준비 (WSL2, Git/GitHub, Python, Docker Desktop, VS Code)
  - 터미널 조작과 파일 권한 실습
  - Docker 설치·점검, 컨테이너 실행/관리
  - Dockerfile 기반 커스텀 웹 서버 이미지, 포트 매핑
  - 바인드 마운트(변경 반영), 볼륨(데이터 영속성)
  - Git 설정, VS Code GitHub 연동, 저장소 제출

---

## 1. 실행 환경

| 항목 | 내용 |
|------|------|
| OS | {{ Windows {{ 10 또는 11 }} + WSL2 (Ubuntu 26.04.1 LTS, 커널 6.18.54.1-microsoft-standard-WSL2)}} |
| Shell / 터미널 | zsh (Oh My Zsh) / {{ Windows Terminal }} |
| Docker | Docker Desktop (WSL2 백엔드), Docker version 29.9.2, build a82e528 |
| Git | {{ git version 2.53.0 }} |
| 에디터 | VS Code + WSL 확장 |

```bash
$ cat /etc/os-release | head -n 3
$ echo $SHELL
$ docker --version
$ git --version
```

<img width="649" height="360" alt="image" src="https://github.com/user-attachments/assets/d794602b-1b9e-491b-9399-b019d9275871" />


> **환경 구조**: Windows에는 Docker Desktop 앱과 VS Code 창만 두고, 실제 프로젝트 파일·Docker 엔진·터미널 작업은 모두 WSL2(Ubuntu) 안에서 수행한다. `docker` 명령은 Ubuntu 터미널에서 입력한다.

---

## 2. 수행 체크리스트

- [ ] 개발 환경 준비 (WSL2 / 기본 도구 / Git·gh / Python·venv / Docker Desktop / VS Code)
- [ ] 터미널 기본 조작 (위치·목록·이동·생성·복사·이동/이름변경·삭제·내용 확인·빈 파일)
- [ ] 권한 실습 (파일 1개 + 디렉토리 1개, 변경 전/후)
- [ ] Docker 설치 점검 (`docker --version`, `docker info`)
- [ ] Docker 운영 명령 (이미지 / 컨테이너 / 로그 / 리소스)
- [ ] hello-world 실행
- [ ] ubuntu 컨테이너 진입 및 명령 수행, attach/exec 차이 정리
- [ ] 웹 서버 소스 + Dockerfile 직접 작성, 커스텀 이미지 빌드/실행
- [ ] 포트 매핑 접속 (2회, 브라우저 주소창 포함)
- [ ] 바인드 마운트 반영 (변경 전/후)
- [ ] Docker 볼륨 영속성 (컨테이너 삭제 전/후)
- [ ] Git 사용자 정보 / 기본 브랜치 설정 (`git config --list`)
- [ ] VS Code GitHub 로그인 및 저장소 연동
- [ ] 트러블슈팅 2건 이상
- [ ] 개념 정리 (경로 / 권한 / 포트 매핑 / 볼륨 / Git vs GitHub)
- [ ] 민감정보 마스킹 확인
- [ ] (보너스) Compose / 환경 변수 / SSH 키

---

## 3. 저장소 구조

```text
.
├── README.md
├── Dockerfile
├── site/                 # 웹 서버 콘텐츠(소스)
│   └── index.html
├── .gitignore            # .venv/ 제외
└── docs/
    └── images/           # 스크린샷 모음
```

---

## 4. 수행 로그

### 4-0. 개발 환경 준비

#### (1) WSL 설치 — Windows PowerShell(관리자)

```powershell
> wsl --install
> wsl -l -v          # Ubuntu가 VERSION 2 인지 확인
> wsl --update       # WSL 커널 최신화
```
<img width="733" height="406" alt="image" src="https://github.com/user-attachments/assets/eb47e661-3eaf-47bb-8213-13bf0f9af867" />

재부팅 후 Ubuntu 창에서 Linux 사용자 이름/비밀번호를 생성했다.

![WSL 설치 및 확인](docs/images/01-wsl.png)

#### (2) 기본 준비 — Ubuntu

```bash
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y build-essential curl wget git unzip
$ mkdir -p ~/projects && cd ~/projects && mkdir myapp && cd myapp
$ pwd
$ explorer.exe .       # Windows 탐색기로 열기
```

![기본 준비](docs/images/01-base.png)

#### (3) Git / GitHub 설정 — Ubuntu

```bash
$ git config --global user.name "{{ 이름 }}"
$ git config --global user.email "{{ GitHub 이메일 }}"
$ git config --global init.defaultBranch main
$ sudo apt install -y gh
$ gh auth login        # GitHub.com → HTTPS → 브라우저 인증
$ gh auth status
```

![Git 설정 / gh 로그인](docs/images/02-git-gh.png)

> ⚠️ 일회용 인증 코드는 캡처에서 가릴 것

#### (4) Python 개발 도구 — Ubuntu

```bash
$ sudo apt install -y python3-pip python3-venv
$ python3 --version && pip3 --version
$ cd ~/projects/myapp
$ python3 -m venv .venv && source .venv/bin/activate
(.venv) $ pip install --upgrade pip
(.venv) $ pip install ruff
(.venv) $ ruff check . && ruff format .
```

![Python / venv / ruff](docs/images/03-python.png)

#### (5) Docker Desktop — Windows 설치, Ubuntu에서 사용

```powershell
> winget install -e --id Docker.DockerDesktop      # Windows PowerShell(관리자)
```

- Docker Desktop → Settings → Resources → WSL Integration 에서 Ubuntu 통합 ON
- `docker` 그룹 추가: `sudo usermod -aG docker $USER` 후 터미널 재시작 (→ 트러블슈팅 2 참고)

![Docker Desktop WSL 통합](docs/images/04-docker-desktop.png)

#### (6) VS Code + WSL — 확장 설치

```bash
$ code --install-extension eamodio.gitlens
$ code --install-extension github.vscode-pull-request-github
$ code --install-extension ms-python.python
$ code --list-extensions
```

VS Code에서 `Open Folder`로 `/home/{{ 사용자 }}/projects/myapp`을 열어 WSL에 연결했다. (좌하단 `WSL: Ubuntu` 확인)

![VS Code WSL 연결](docs/images/04-vscode-wsl.png)
![확장 설치](docs/images/04-extensions.png)

#### (7) (선택) 터미널 꾸미기 — zsh / Oh My Zsh / tmux / btop

```bash
$ echo $SHELL
$ tmux -V
$ btop --version
```

![선택 도구](docs/images/05-optional.png)

---

### 4-1. 터미널 조작 로그


### 위치 확인 · 생성 · 목록
$ pwd
$ mkdir -p ~/projects/practice
$ cd ~/projects/practice
$ touch empty.txt .hidden
$ echo "hello codyssey" > note.txt
$ ls
$ ls -la
<img width="642" height="143" alt="image" src="https://github.com/user-attachments/assets/7c09922c-f391-4b39-a6dd-6481920f9aa0" />

### 내용 확인 · 복사
$ cat note.txt
$ cp note.txt note_copy.txt
$ ls
<img width="555" height="202" alt="image" src="https://github.com/user-attachments/assets/d8207e4e-dd5e-42bf-90c3-1efb2d0898fc" />

### 이동 · 이름 변경
$ mkdir backup
$ mv note_copy.txt backup/
$ mv empty.txt renamed.txt
$ ls && ls backup
<img width="723" height="150" alt="image" src="https://github.com/user-attachments/assets/773cc3be-2b6c-4986-9c34-654038bad9c0" />

### 삭제
$ rm renamed.txt
$ rm -r backup
$ ls -la
```
<img width="723" height="150" alt="image" src="https://github.com/user-attachments/assets/742eb0b7-051c-4843-9946-8d6663d77648" />

**관찰**
- `ls`는 숨김 파일(`.hidden`)을 보여주지 않고, `ls -la`는 숨김 파일과 권한·소유자·크기를 함께 보여준다.
- `mv`는 대상이 다른 디렉토리면 **이동**, 같은 디렉토리에서 이름만 다르면 **이름 변경**으로 동작한다.
- `rm -r`은 디렉토리를 안의 내용까지 삭제하며 되돌릴 수 없다.
 
**📝 절대 경로 vs 상대 경로**
- 절대 경로: `/`부터 시작하는 전체 경로. 예) `/home/ikorv525031/projects/practice/note.txt`
- 상대 경로: 현재 위치 기준 경로. 예) 위치가 `~/projects`일 때 `practice/note.txt`, `../myapp`
````


---

# PART 2. 4-2 권한 실습
 

**1) 변경 전 상태**
$ cd ~/projects/practice
$ echo "secret" > perm.txt
$ mkdir permdir
$ ls -l perm.txt
$ ls -ld permdir
```
<img width="723" height="198" alt="image" src="https://github.com/user-attachments/assets/59f068bb-ccdd-41a7-aff4-ff7301b487e4" />

**2) 파일 권한 변경**
 
```bash
$ chmod 600 perm.txt && ls -l perm.txt
$ chmod 000 perm.txt && ls -l perm.txt
$ cat perm.txt                     # Permission denied 확인
$ chmod 644 perm.txt && ls -l perm.txt
$ cat perm.txt                     # 복구 후 정상 출력
```
<img width="750" height="304" alt="image" src="https://github.com/user-attachments/assets/4ceb7657-d302-4cd5-b386-2b3babad68c5" />

**3) 디렉토리 권한 변경**
 
```bash
$ chmod 700 permdir && ls -ld permdir
$ chmod 000 permdir && ls -ld permdir
$ cd permdir                       # permission denied 확인
$ chmod 755 permdir && ls -ld permdir
$ cd permdir && pwd                # 복구 후 진입 성공
```
<img width="694" height="286" alt="image" src="https://github.com/user-attachments/assets/524c1804-9686-4fdb-b560-cd0440b0a0a1" />

**변경 전/후 비교**
 
| 대상 | 변경 전 | 변경 후 | 명령 | 관찰 |
|------|---------|---------|------|------|
| 파일 `perm.txt` | `-rw-r--r--` | `-rw-------` → `----------` | `chmod 600`, `chmod 000` | 000일 때 `cat`이 Permission denied |
| 디렉토리 `permdir` | `drwxr-xr-x` | `drwx------` → `d---------` | `chmod 700`, `chmod 000` | 000일 때 `cd`가 permission denied |

**📝 권한 표기 해석 (r/w/x, 755, 644)**
- `r`=읽기(4), `w`=쓰기(2), `x`=실행(1). **소유자 / 그룹 / 기타** 순서로 숫자를 더해 표기한다.
- `755` = `rwxr-xr-x` : 소유자는 읽기+쓰기+실행, 그룹·기타는 읽기+실행
- `644` = `rw-r--r--` : 소유자는 읽기+쓰기, 그룹·기타는 읽기만
- 파일의 `x`는 실행 권한이고, 디렉토리의 `x`는 안으로 들어갈 수 있는 권한이다. (`permdir`에 000을 걸면 `cd`가 막힘)

### 4-3. Docker 설치 및 기본 점검

```bash
$ docker --version
$ docker info
```

- 데몬 동작 확인: `Server:` 항목에 엔진 정보가 출력됨

![docker version / info](docs/images/12-docker-info.png)

---

### 4-4. Docker 기본 운영 명령

```bash
$ docker pull nginx:alpine           # 이미지 다운로드
$ docker images                      # 이미지 목록
$ docker run -d --name ops-test nginx:alpine
$ docker ps                          # 실행 중 컨테이너
$ docker logs ops-test               # 로그 확인
$ docker stats --no-stream           # 리소스 확인
$ docker stop ops-test               # 중지
$ docker ps -a                       # 전체 컨테이너
$ docker rm ops-test                 # 정리
```

![운영 명령 1](docs/images/13-docker-ops-1.png)
![운영 명령 2](docs/images/13-docker-ops-2.png)

---

### 4-5. 컨테이너 실행 실습

**hello-world**

```bash
$ docker run hello-world
```

![hello-world](docs/images/14-hello-world.png)

**ubuntu 컨테이너 진입**

```bash
$ docker run -it --name ubuntu-test ubuntu bash
root@xxxx:/# ls
root@xxxx:/# echo "hello container"
root@xxxx:/# exit
$ docker ps -a
```

![ubuntu 컨테이너](docs/images/15-ubuntu.png)

**종료/유지 차이 관찰 (attach vs exec)**

```bash
$ docker run -dit --name ubuntu-keep ubuntu bash
$ docker exec -it ubuntu-keep bash      # 진입 후 exit
$ docker ps                             # 컨테이너 상태 확인
$ docker attach ubuntu-keep             # 진입 후 exit
$ docker ps -a                          # 컨테이너 상태 확인
```

| 방식 | 명령 | 관찰 결과 |
|------|------|-----------|
| run -it | `docker run -it ubuntu bash` | {{ exit 시 컨테이너도 종료 }} |
| exec | `docker exec -it <name> bash` | {{ 별도 프로세스로 진입, exit 해도 컨테이너 유지 }} |
| attach | `docker attach <name>` | {{ 메인 프로세스(PID 1)에 연결, exit 시 컨테이너 종료 }} |

![attach / exec 비교](docs/images/16-attach-exec.png)

---

### 4-6. Dockerfile 기반 커스텀 이미지

- **선택 방식**: (A) 웹 서버 베이스 이미지 활용 + 정적 콘텐츠 교체
- **기존 베이스**: `nginx:alpine` (Docker Hub 공식 이미지)

**site/index.html**

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>My Custom Web</title></head>
<body>
  <h1>Hello from my custom image!</h1>
  <p>version: 1.0</p>
</body>
</html>
```

**Dockerfile** (직접 작성)

```dockerfile
FROM nginx:alpine
LABEL org.opencontainers.image.title="my-custom-nginx"
ENV APP_ENV=dev
COPY site/ /usr/share/nginx/html/
EXPOSE 80
```

**커스텀 포인트와 목적**

| 커스텀 포인트 | 목적 |
|---------------|------|
| `FROM nginx:alpine` | {{ 가볍고 검증된 웹 서버 이미지를 그대로 재사용 }} |
| `LABEL` | {{ 이미지 메타데이터 명시, 식별 용이 }} |
| `ENV APP_ENV=dev` | {{ 실행 모드를 환경 변수로 구분 }} |
| `COPY site/ ...` | {{ 기본 페이지를 내 콘텐츠로 교체 }} |
| `EXPOSE 80` | {{ 컨테이너가 사용하는 포트를 문서화 }} |

**빌드 / 실행**

```bash
$ docker build -t my-web:1.0 .
$ docker images | grep my-web
$ docker run -d --name my-web my-web:1.0
$ docker ps
$ docker exec my-web printenv APP_ENV      # ENV 반영 확인
```

![빌드 결과](docs/images/17-build.png)

---

### 4-7. 포트 매핑 접속 증거

```bash
$ docker rm -f my-web
$ docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
$ curl http://localhost:8080
$ docker run -d -p 8081:80 --name my-web-8081 my-web:1.0
$ curl http://localhost:8081
$ docker ps
```

![curl 응답](docs/images/18-curl.png)
![브라우저 접속 8080 (주소창 포함)](docs/images/18-browser-8080.png)
![브라우저 접속 8081 (주소창 포함)](docs/images/18-browser-8081.png)

---

### 4-8. 바인드 마운트 (변경 반영)

```bash
$ cd ~/projects/myapp
$ docker run -d -p 8082:80 --name bind-test \
    -v "$(pwd)/site":/usr/share/nginx/html my-web:1.0
$ curl http://localhost:8082                         # 변경 전

$ echo "<h1>Changed on host!</h1>" > site/index.html  # 호스트에서 수정
$ curl http://localhost:8082                         # 변경 후 (재빌드 없이 반영)
```

| 구분 | 결과 |
|------|------|
| 변경 전 | {{ }} |
| 변경 후 | {{ }} |

![바인드 마운트 변경 전](docs/images/19-bind-before.png)
![바인드 마운트 변경 후](docs/images/19-bind-after.png)

> 확인 후 `site/index.html`을 원래 내용으로 복구한다.

---

### 4-9. Docker 볼륨 (데이터 영속성)

```bash
$ docker volume create mydata
$ docker volume ls
$ docker run -d --name vol-test -v mydata:/data ubuntu sleep infinity
$ docker exec vol-test bash -lc "echo hi > /data/hello.txt && cat /data/hello.txt"
hi
$ docker rm -f vol-test                       # 컨테이너 삭제

$ docker run -d --name vol-test2 -v mydata:/data ubuntu sleep infinity
$ docker exec vol-test2 bash -lc "cat /data/hello.txt"
hi                                            # 삭제 후에도 데이터 유지
```

![볼륨 생성/연결/기록](docs/images/20-volume-1.png)
![컨테이너 삭제 후 데이터 유지](docs/images/20-volume-2.png)

---

### 4-10. Git 설정 및 GitHub / VS Code 연동

```bash
$ git config --list
```

![git config --list](docs/images/21-git-config-list.png)

**VS Code GitHub 로그인 및 저장소 연동**

1. VS Code 좌하단 계정 아이콘 → `Sign in with GitHub` → 브라우저 승인
2. GitHub Pull Requests 확장 / Source Control 패널에서 저장소 연동 확인

![VS Code GitHub 로그인](docs/images/22-vscode-github-login.png)
![VS Code 저장소 연동](docs/images/22-vscode-repo.png)

**저장소 생성 및 push**

```bash
$ cd ~/projects/myapp
$ printf ".venv/\n" > .gitignore
$ git init
$ git add .
$ git commit -m "Add dev workstation mission"
$ gh repo create {{ 저장소 이름 }} --public --source=. --remote=origin --push
$ git log --oneline
```

![GitHub 저장소 페이지](docs/images/23-github-repo.png)

- 저장소 링크: {{ https://github.com/사용자/저장소 }}

---

## 5. 개념 정리

<!-- 본인 말로 다시 쓰세요 -->

**절대 경로 vs 상대 경로**
- 절대 경로는 루트(`/`)부터 시작하는 전체 경로다. 예: `/home/{{ 사용자 }}/projects/practice/note.txt`
- 상대 경로는 현재 위치 기준이다. 예: 위치가 `~/projects`일 때 `practice/note.txt`, `../myapp`

**파일 권한 (r/w/x, 755, 644)**
- `r`=읽기(4), `w`=쓰기(2), `x`=실행(1)이며 소유자 / 그룹 / 기타 순서로 합산한다.
- `755` = `rwxr-xr-x` (소유자 전체, 나머지는 읽기+실행), `644` = `rw-r--r--` (소유자 읽기+쓰기, 나머지는 읽기)

**포트 매핑이 필요한 이유**
- 컨테이너는 격리된 네트워크를 가지므로 호스트에서 바로 접근할 수 없다. `-p 호스트포트:컨테이너포트`로 연결해야 브라우저/curl로 접속할 수 있다. 같은 이미지를 호스트 포트만 바꿔 여러 개 띄울 수도 있다.

**Docker 볼륨**
- 컨테이너의 생명주기와 분리되어 데이터를 보존하는 Docker 관리 저장소다. 컨테이너를 삭제해도 볼륨의 데이터는 남는다. (바인드 마운트는 호스트의 특정 경로를 직접 연결하는 방식)

**이미지와 컨테이너**
- 이미지는 실행 환경의 설계도(읽기 전용), 컨테이너는 이미지를 실행한 인스턴스다. 같은 이미지로 컨테이너를 여러 번 만들어도 동일한 환경이 재현된다.

**Git vs GitHub**
- Git은 내 컴퓨터(로컬)에서 변경 이력을 관리하는 버전 관리 도구, GitHub는 Git 저장소를 원격에 올려 공유·협업하는 플랫폼이다.

---

## 6. 검증 방법 요약

| 항목 | 사용한 명령 | 확인한 내용 | 증거 |
|------|-------------|-------------|------|
| 터미널 조작 | `pwd`, `ls -la`, `cp`, `mv`, `rm`, `cat`, `touch` | 생성/이동/삭제 정상 | [4-1](#4-1-터미널-조작-로그) |
| 권한 | `ls -l`, `chmod` | 변경 전/후 권한, denied 확인 | [4-2](#4-2-권한-실습) |
| Docker 점검 | `docker --version`, `docker info` | 엔진/데몬 정상 | [4-3](#4-3-docker-설치-및-기본-점검) |
| Docker 운영 | `images`, `ps -a`, `logs`, `stats` | 이미지/컨테이너/리소스 확인 | [4-4](#4-4-docker-기본-운영-명령) |
| 컨테이너 실행 | `run hello-world`, `exec`, `attach` | 실행 및 종료/유지 차이 | [4-5](#4-5-컨테이너-실행-실습) |
| 커스텀 이미지 | `docker build`, `docker run` | 빌드/실행 성공, ENV 반영 | [4-6](#4-6-dockerfile-기반-커스텀-이미지) |
| 포트 매핑 | `docker run -p`, `curl`, 브라우저 | 8080/8081 접속 성공 | [4-7](#4-7-포트-매핑-접속-증거) |
| 바인드 마운트 | `-v $(pwd)/site:...` | 호스트 수정이 즉시 반영 | [4-8](#4-8-바인드-마운트-변경-반영) |
| 볼륨 | `docker volume create`, `-v` | 컨테이너 삭제 후 데이터 유지 | [4-9](#4-9-docker-볼륨-데이터-영속성) |
| Git/GitHub | `git config --list`, VS Code 로그인 | 설정 및 연동 완료 | [4-10](#4-10-git-설정-및-github--vs-code-연동) |

---

## 7. 트러블슈팅

### 트러블슈팅 1: `gh auth login` 브라우저 열기 실패

- **문제**: `Failed opening a web browser ... xdg-open ... executable file not found in $PATH`
- **원인 가설**: WSL에는 GUI 브라우저와 이를 호출하는 도구(`xdg-open`, `wslview`)가 없다.
- **확인**: 에러 메시지의 `executable file not found` 확인, `which xdg-open` 결과 없음
- **해결/대안**: 안내된 URL(`https://github.com/login/device`)을 Windows 브라우저에서 직접 열어 일회용 코드를 입력, `gh auth status`로 로그인 확인

![트러블슈팅 1](docs/images/90-trouble-1.png)

### 트러블슈팅 2: `docker run` 시 `docker.sock` permission denied

- **문제**: `permission denied while trying to connect to the docker API at unix:///var/run/docker.sock`
- **원인 가설**: 소켓 접근이 `docker` 그룹으로 제한되는데 내 계정이 그 그룹에 속하지 않는다.
- **확인**: `groups` 출력에 `docker` 없음, `ls -l /var/run/docker.sock`으로 그룹 확인
- **해결/대안**: `sudo usermod -aG docker $USER` 후 새 터미널(또는 `wsl --shutdown` 후 재접속), `docker run hello-world` 성공 확인

![트러블슈팅 2](docs/images/90-trouble-2.png)

### 트러블슈팅 3: Ubuntu에서 `code .` 실행 시 command not found

- **문제**: `Command 'code' not found`
- **원인 가설**: Windows의 VS Code 경로가 WSL의 `$PATH`에 연결되지 않았다.
- **확인**: `which code` 결과 없음
- **해결/대안**: VS Code 창에서 `Open Folder`로 `/home/{{ 사용자 }}/projects/myapp`을 직접 열어 WSL에 연결

![트러블슈팅 3](docs/images/90-trouble-3.png)

---

## 8. 보너스 과제 (선택)

- [ ] Docker Compose 기초 (단일 서비스)
- [ ] Docker Compose 멀티 컨테이너 (컨테이너 간 통신 확인)
- [ ] Compose 운영 명령 (`up`, `down`, `ps`, `logs`)
- [ ] 환경 변수 활용 (포트/모드 변경)
- [ ] GitHub SSH 키 설정

---

## 9. 재현 방법 (평가자용)

```bash
$ git clone {{ 저장소 URL }}
$ cd {{ 폴더명 }}
$ docker build -t my-web:1.0 .
$ docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
$ curl http://localhost:8080
```

- 필요 환경: Docker (Docker Desktop + WSL2 또는 Linux)
- 주의사항: 실습 로그의 `/home/{{ 사용자 }}` 경로는 본인 환경에 맞게 바꿔서 실행한다. 랩 PC 초기화 시 WSL 데이터가 삭제될 수 있으므로 결과물은 GitHub에 push해 보관했다.

---

## 10. 보안 체크

- [ ] 토큰, 비밀번호, 개인키, 인증코드가 README / 로그 / 스크린샷에 없음 (`gh auth login` 코드, `hosts.yml` 내용 등)
- [ ] 노출이 의심되면 히스토리에서 제거하고 토큰을 재발급함
- [ ] `.venv/` 등 불필요한 파일이 저장소에 올라가지 않음
