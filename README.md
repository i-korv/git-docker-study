# git-docker-study
Git과 Docker를 학습하고 실습한 프로젝트입니다.

---

## 📌 프로젝트 개요
- **목표**: Git 버전관리 + Docker 컨테이너 기초 습득
- **기간**: [2026.11.03] ~ [완료일]
- **주요 성과**: Nginx 웹서버 이미지 빌드 및 3가지 실행 방식 검증

---

## 🖥️ 1) 실행 환경

| 항목 | 버전 |
|------|------|
| OS | Windows 11 + WSL2 Ubuntu 22.04 |
| Shell | bash |
| Docker | 29.9.2 |
| Git | 2.53.0 |
| 에디터 | VSCode |

---

## ✅ 2) 수행 체크리스트

### 기초 환경 설정
- [x] WSL2 Ubuntu 완전 재설치
- [x] bash 터미널 기본 조작 (pwd, mkdir, ls, cd)
- [x] 폴더 구조 설계 및 생성
- [x] 파일 권한 변경 (chmod)

### Docker 학습
- [x] Docker 설치 및 버전 확인
- [x] hello-world 컨테이너 실행
- [x] Dockerfile 작성 (Nginx 기반)
- [x] 이미지 빌드 (my-nginx:1.0)
- [x] 컨테이너 실행 - 포트 매핑 (8080:80)
- [x] 컨테이너 실행 - 바인드마운트 (8081:80)
- [x] 컨테이너 실행 - 볼륨 연결 (8082:80)

### 볼륨 & 영속성
- [x] 볼륨 생성 (my-data)
- [x] 볼륨 마운트 및 데이터 저장
- [x] 컨테이너 삭제 후 볼륨 데이터 유지 확인 ✨
- [x] 바인드마운트 실시간 반영 테스트

### Git & GitHub
- [x] Git 초기 설정 (user.name, user.email)
- [x] GitHub 저장소 생성 (git-docker-study)
- [x] VSCode GitHub 연동
- [x] README.md 작성 및 커밋

---

## 📂 3) 폴더 구조
