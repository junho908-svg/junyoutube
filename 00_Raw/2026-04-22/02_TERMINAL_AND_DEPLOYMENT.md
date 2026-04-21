# 🛠️ 터미널 서버 구축 & GitHub 배포 완벽 가이드

> **대상**: 앱 코드 로컬 개발 및 GitHub/Vercel 배포  
> **작성일**: 2026.04.21  
> **작업 환경**: macOS (Junho-ui-MacBookPro-14), zsh, Node.js

---

## 📑 목차

1. [개발 환경 준비](#1-개발-환경-준비)
2. [프로젝트 구조](#2-프로젝트-구조)
3. [로컬 서버 운영 (일상)](#3-로컬-서버-운영-일상)
4. [Git 기초 워크플로우](#4-git-기초-워크플로우)
5. [GitHub 업로드 상세](#5-github-업로드-상세)
6. [Vercel 배포 상세](#6-vercel-배포-상세)
7. [자동 배포 흐름 (CI/CD)](#7-자동-배포-흐름-cicd)
8. [문제 해결 (Troubleshooting)](#8-문제-해결-troubleshooting)
9. [자주 쓰는 명령어 치트시트](#9-자주-쓰는-명령어-치트시트)

---

## 1. 개발 환경 준비

### 1.1 필수 도구

| 도구 | 용도 | 설치 여부 |
|---|---|---|
| **Node.js 18+** | JavaScript 런타임 | 이미 설치됨 |
| **npm** | 패키지 관리자 | Node.js와 함께 |
| **Git** | 버전 관리 | macOS 기본 포함 |
| **GitHub Desktop** | 시각적 Git 도구 | 설치됨 |
| **Chrome** | 개발 및 테스트 | 설치됨 |
| **Terminal** | 명령어 실행 | macOS 기본 |

### 1.2 개발 환경 정보

```
OS: macOS (Junho-ui-MacBookPro-14)
사용자: bluediamond
Shell: zsh (mlx_env 활성화)
프로젝트 경로: /Users/bluediamond/family-worship-app
서버 포트: 5173 (Vite 기본)
브라우저: Chrome
```

### 1.3 처음 한 번만 설정 (이미 완료)

```bash
# Node.js 버전 확인
node --version    # v18 이상이어야 함

# npm 버전 확인  
npm --version     # 9 이상이어야 함

# Git 설정 (최초 1회)
git config --global user.name "junho908-svg"
git config --global user.email "대표님이메일@gmail.com"
```

---

## 2. 프로젝트 구조

### 2.1 파일 트리

```
/Users/bluediamond/family-worship-app/
├── 📁 src/                    # 앱 코드
│   ├── App.jsx               # 메인 앱 (약 2,900줄)
│   ├── main.jsx              # 엔트리 포인트
│   └── index.css             # Tailwind + 글로벌 스타일
│
├── 📁 public/                 # 정적 파일 (아이콘 등, 현재 비어있음)
│
├── 📁 node_modules/           # 의존성 (자동 생성, Git에 안 올림)
│
├── 📁 dist/                   # 빌드 결과 (자동 생성, Git에 안 올림)
│
├── 📁 .git/                   # Git 저장소 (숨김, 자동 생성)
│
├── 📄 package.json            # 프로젝트 설정 + 의존성 목록
├── 📄 package-lock.json       # 의존성 잠금 (자동 생성)
│
├── 📄 vite.config.js          # Vite 빌드 설정
├── 📄 tailwind.config.js      # Tailwind 설정
├── 📄 postcss.config.js       # PostCSS 설정
│
├── 📄 index.html              # 브라우저 진입 파일
├── 📄 README.md               # GitHub 방문자용 문서
└── 📄 .gitignore              # Git 제외 목록
```

### 2.2 `package.json` 핵심 내용

```json
{
  "name": "family-worship-app",
  "version": "0.9.0",
  "type": "module",
  "scripts": {
    "dev": "vite",           // npm run dev → 개발 서버
    "build": "vite build",   // npm run build → 프로덕션 빌드
    "preview": "vite preview"// npm run preview → 빌드 결과 테스트
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "lucide-react": "^0.383.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.1",
    "vite": "^5.3.1",
    "tailwindcss": "^3.4.4",
    "postcss": "^8.4.38",
    "autoprefixer": "^10.4.19"
  }
}
```

### 2.3 `.gitignore` 내용

```
node_modules
dist
.DS_Store
.env
.env.local
*.log
.vite
```

이 파일에 적힌 폴더/파일들은 **Git에 올라가지 않음**. `node_modules`는 용량이 커서 반드시 제외해야 함.

---

## 3. 로컬 서버 운영 (일상)

### 3.1 서버 시작 (가장 자주 씀)

**터미널을 열고:**

```bash
cd ~/family-worship-app
npm run dev
```

**결과:**
```
VITE v5.x.x  ready in 1114 ms

➜ Local:   http://localhost:5173/
➜ Network: use --host to expose
➜ press h + enter to show help
```

Chrome에서 `http://localhost:5173/` 접속 → 앱 열림 🎵

### 3.2 서버 중지

서버가 돌아가는 **바로 그 터미널 창**에서:

```
Ctrl + C  (macOS도 Ctrl, Cmd 아님!)
```

### 3.3 서버 재시작 (자주 씀)

```bash
# 1. 기존 서버 멈추기
Ctrl + C

# 2. 다시 시작
npm run dev
```

> 💡 **언제 재시작이 필요한가?**
> - 새 패키지 설치 후
> - `vite.config.js`, `tailwind.config.js` 수정 후
> - 핫 리로드(자동 새로고침)가 이상하게 멈췄을 때

### 3.4 Chrome 개발자 도구 (F12)

**Console 탭**: 에러 메시지 확인
**Network 탭**: 이미지/BGM 로딩 확인
**Elements 탭**: HTML/CSS 구조 확인

### 3.5 강제 새로고침 (중요!)

Chrome에서 변경사항이 안 보일 때:

```
⌘ + Shift + R    (반드시 Shift!)
```

일반 `⌘+R`은 캐시를 사용할 수 있음. **`⌘+Shift+R`은 캐시 완전 무시**.

### 3.6 패키지 추가 설치

```bash
# 예: lodash 설치
npm install lodash

# 개발용 패키지 설치 (devDependency)
npm install -D @types/node

# 설치 후 서버 재시작 필요
Ctrl + C
npm run dev
```

### 3.7 새 코드 적용하기 (클로드에서 받은 파일)

```bash
# 다운로드 폴더에서 src/ 로 복사
cp ~/Downloads/App.jsx ~/family-worship-app/src/App.jsx

# 한 번에 여러 파일 적용
cp ~/Downloads/App.jsx ~/family-worship-app/src/App.jsx && \
cp ~/Downloads/package.json ~/family-worship-app/package.json && \
cp ~/Downloads/README.md ~/family-worship-app/README.md && \
cp ~/Downloads/index.html ~/family-worship-app/index.html && \
echo "✅ 적용 완료"
```

Vite는 파일 변경을 **자동 감지**해서 Chrome이 알아서 새로고침됩니다. 감지 안 되면 Chrome에서 `⌘+Shift+R`.

---

## 4. Git 기초 워크플로우

### 4.1 Git이란?

**Git = 시간 여행 도구 + 협업 도구**

- 모든 변경 이력을 저장 (실수해도 되돌릴 수 있음)
- GitHub와 연동 → 여러 컴퓨터에서 같이 작업
- Vercel과 연동 → 자동 배포

### 4.2 핵심 3개 동작

```
[작업 폴더]          [스테이징]         [저장소]
수정/추가 ──git add──▶ 준비됨 ──git commit──▶ 기록됨 ──git push──▶ GitHub
```

| 명령어 | 의미 |
|---|---|
| `git add .` | 모든 변경사항을 "스테이징" (다음 커밋에 포함할 것) |
| `git commit -m "메시지"` | 변경사항을 저장 (로컬에) |
| `git push` | 로컬 커밋을 GitHub에 업로드 |
| `git pull` | GitHub의 변경사항을 로컬로 다운로드 |
| `git status` | 현재 상태 확인 |
| `git log` | 변경 이력 보기 |

### 4.3 일상 워크플로우

#### 방법 A: 터미널

```bash
cd ~/family-worship-app

# 1. 파일 수정 (코드 편집)

# 2. 상태 확인
git status

# 3. 변경사항 스테이징
git add .

# 4. 커밋 (로컬에 저장)
git commit -m "fix: 로고몽 사이즈 조정"

# 5. GitHub에 푸시
git push
```

#### 방법 B: GitHub Desktop (GUI)

1. GitHub Desktop 열기
2. 좌측에 변경된 파일 자동 표시
3. 하단 **Summary** 입력: `fix: 로고몽 사이즈 조정`
4. **Commit to main** 클릭
5. 상단 **Push origin** 클릭

### 4.4 커밋 메시지 컨벤션

좋은 커밋 메시지 = 미래의 자신/팀원에게 선물

```
타입: 요약 (50자 이내)

상세 설명 (필요 시, 72자로 줄바꿈)
```

**타입 종류**:
- `feat`: 새 기능 추가
- `fix`: 버그 수정
- `style`: 디자인만 변경 (로직 X)
- `refactor`: 코드 정리 (기능 X)
- `docs`: 문서 업데이트
- `chore`: 설정·패키지 등
- `release`: 버전 배포

**예시**:
```
feat: 로고몽 9곳에 통합
fix: 웰컴 스크린 자동재생 오류 해결
style: 시간대별 컬러 그라디언트 조정
docs: README v0.9 BETA 정보 추가
release: v0.9 BETA 첫 공개
```

---

## 5. GitHub 업로드 상세

### 5.1 GitHub 저장소 구조 (우리 프로젝트)

```
👤 junho908-svg (계정)
│
├── 📦 family-worship-app    (앱 코드 저장소)
│   ├── src/
│   ├── package.json
│   └── ...
│
└── 📦 family-worship-bgm    (미디어 파일 저장소)
    ├── MP3 15개
    └── logomong.png
```

**두 저장소로 분리한 이유**:
- 앱 코드: 수시로 업데이트, 용량 작음
- 미디어: 가끔 추가, 용량 큼

이렇게 분리하면 Git 히스토리가 깔끔하고 빌드도 빠름.

### 5.2 처음 GitHub 업로드 (이미 완료했지만 기록)

#### 단계별 명령어

```bash
# 1. 프로젝트 폴더로 이동
cd /Users/bluediamond/family-worship-app

# 2. Git 초기화
git init

# 3. GitHub에서 빈 리포지토리 생성 (웹에서)
#    - github.com 로그인
#    - New repository 클릭
#    - Name: family-worship-app
#    - Public 선택
#    - Initialize with README: 체크 해제 (이미 로컬에 있음)
#    - Create repository

# 4. 원격 저장소 연결
git remote add origin https://github.com/junho908-svg/family-worship-app.git

# 5. main 브랜치 생성
git checkout -b main

# 6. 모든 파일 스테이징
git add .

# 7. 첫 커밋
git commit -m "Add 우리집 가정예배 앱 - 로고몽과 함께"

# 8. README 충돌이 있을 경우
git fetch origin
git config pull.rebase false
git pull origin main --allow-unrelated-histories --no-edit
git checkout --ours README.md     # 로컬 버전 선택
git add README.md
git commit -m "Resolve README conflict - keep app version"

# 9. GitHub로 푸시!
git push -u origin main
```

### 5.3 인증 — Personal Access Token (PAT)

#### 왜 필요한가?
2021년 8월부터 GitHub은 **비밀번호로 push 금지**. PAT 필수.

#### Token 만드는 법

1. Chrome에서 [github.com/settings/tokens/new](https://github.com/settings/tokens/new)
2. **Note**: `family-worship-deploy`
3. **Expiration**: `90 days`
4. **Select scopes**: ☑️ `repo` 체크 (가장 위 큰 박스)
5. 페이지 하단 **Generate token** 클릭
6. 생성된 `ghp_xxxxxxxx...` 토큰을 **복사해서 안전한 곳에 메모**

> ⚠️ 토큰은 한 번만 보여줌. 잃어버리면 새로 만들어야 함.

#### 사용법

터미널이 이런 걸 물으면:
```
Username for 'https://github.com': 
```
→ `junho908-svg` 입력

```
Password for 'https://junho908-svg@github.com': 
```
→ **GitHub 비밀번호 아님!** 위의 토큰 붙여넣기 (입력해도 화면엔 안 보임. 정상)

### 5.4 일상 업로드 (v0.9 BETA 이후)

**파일 수정 후 한 번에 올리기**:

```bash
cd ~/family-worship-app && \
git add . && \
git commit -m "변경사항 설명" && \
git push
```

이 한 줄이면 GitHub 업로드 + Vercel 자동 배포까지 시작됩니다.

### 5.5 미디어 파일 업로드 (family-worship-bgm)

새 BGM이나 이미지 추가할 때:

#### 방법 A: GitHub 웹에서 직접 업로드 (쉬움)

1. [github.com/junho908-svg/family-worship-bgm](https://github.com/junho908-svg/family-worship-bgm)
2. 녹색 "Code" 버튼 옆 **"Add file"** → **"Upload files"**
3. 파일 드래그 앤 드롭
4. **Commit changes** 클릭
5. 10~30초 후 jsDelivr에서 사용 가능

#### 방법 B: GitHub Desktop

1. 앱에서 family-worship-bgm 리포지토리 선택
2. **Repository → Show in Finder**
3. Finder에서 파일 드래그해서 폴더에 넣기
4. GitHub Desktop으로 돌아가기
5. Summary: `Add 새 곡 제목`
6. **Commit to main → Push origin**

### 5.6 캐시 무효화 (미디어 파일 변경 시)

파일을 덮어썼는데 앱에서 옛날 버전이 보이면:

```
https://purge.jsdelivr.net/gh/junho908-svg/family-worship-bgm@main/파일명.png
```

위 URL을 Chrome에서 열면 즉시 캐시 비워짐.

---

## 6. Vercel 배포 상세

### 6.1 Vercel이란?

**Vercel = "푸시하면 자동 배포해주는 웹 호스팅 서비스"**

- 무료 Hobby 플랜 (개인 프로젝트용)
- GitHub 연동으로 `git push`만 하면 자동 배포
- HTTPS 자동 (보안 좋음)
- 전 세계 CDN (빠름)
- 한국 서울 노드 있음

### 6.2 배포 단계 (최초 1회, 이미 완료)

#### 1. Vercel 가입 (30초)

[vercel.com/signup](https://vercel.com/signup) → **"Continue with GitHub"**

GitHub 계정으로 자동 가입. 카드 등록 불필요.

#### 2. 새 프로젝트 생성

대시보드에서: **"Add New..."** → **"Project"**

또는 [vercel.com/new](https://vercel.com/new)

#### 3. GitHub 리포지토리 연결

**"Continue with GitHub"** → Authorize → family-worship-app 선택

**Install Vercel GitHub App**:
- "Only select repositories" 추천
- `family-worship-app` 체크
- **Install**

#### 4. 리포지토리 Import

목록에서 `family-worship-app` 옆 **"Import"**

#### 5. 프로젝트 설정 (자동 감지됨)

| 항목 | 값 |
|---|---|
| Project Name | `family-worship-app` |
| Framework Preset | **Vite** (자동 감지) ✅ |
| Root Directory | `./` |
| Build Command | `npm run build` |
| Output Directory | `dist` |
| Install Command | `npm install` |

**모두 그대로 두고 Deploy 클릭!**

#### 6. 빌드 대기 (1~2분, 첫 빌드는 15분까지 가능)

진행 로그:
```
⏳ Installing dependencies (npm install)
⏳ Building (vite build)
⏳ Deploying
```

#### 7. 🎉 Congratulations!

콘페티 + "Congratulations!" 메시지 + 앱 미리보기
→ **진짜 인터넷 URL 발급!**

### 6.3 우리 프로젝트 배포 정보

```
Production URL: https://family-worship-app-pi.vercel.app
GitHub: junho908-svg/family-worship-app
Branch: main
Framework: Vite
Build Command: npm run build
Output Directory: dist
Install Command: npm install
Node Version: 18.x (자동)
Region: Auto (글로벌 CDN)
```

### 6.4 Vercel 대시보드 주요 기능

**Deployments 탭**: 모든 배포 이력 (실패 배포도 확인)
**Analytics 탭**: 방문자 수, 페이지 로딩 속도 (무료 Hobby에서 제한적)
**Settings 탭**: 환경 변수, 도메인, 빌드 설정
**Domains**: `vercel.app` 외에 커스텀 도메인 연결 가능 (예: `urijip-yebae.com`)

### 6.5 빌드 실패 시 확인 방법

1. Vercel 대시보드 → **Deployments**
2. 실패한 배포 클릭
3. **"View Function Logs"** 또는 **"Build Logs"**
4. 빨간 에러 메시지 읽기

**자주 발생하는 원인**:
- `package.json`에 없는 패키지 import
- 오타로 인한 파싱 에러
- 환경 변수 누락
- 빌드 메모리 부족 (우리 앱은 가벼워서 거의 없음)

---

## 7. 자동 배포 흐름 (CI/CD)

### 7.1 전체 파이프라인

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  📝 로컬 개발 (VSCode / 에디터)                   │
│        ↓                                        │
│  💾 git add . && git commit                     │
│        ↓                                        │
│  ⬆️  git push                                    │
│        ↓                                        │
│  📦 GitHub 저장 (2초)                            │
│        ↓                                        │
│  🔔 Webhook 발송 → Vercel                       │
│        ↓                                        │
│  🤖 Vercel 자동 빌드                            │
│     - npm install (30초)                        │
│     - vite build (30초)                         │
│        ↓                                        │
│  🚀 Cloudflare CDN 배포 (10초)                  │
│        ↓                                        │
│  ✨ 전 세계 가족의 핸드폰에 새 버전!              │
│                                                 │
│  ⏱️  총 소요: 1~2분                              │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 7.2 브랜치 전략 (선택)

v1.0부터 쓰면 좋은 전략:

```
main (프로덕션)      ──── 항상 안정적 ───→ family-worship-app-pi.vercel.app
  ↑
develop (개발)       ──── 테스트 중 ────→ family-worship-app-dev.vercel.app
  ↑
feature/알림기능     ─── 신기능 개발 ───→ 브랜치별 preview URL
```

- `feature/*`: 새 기능 만들 때
- `develop`: 통합 테스트
- `main`: 가족에게 공개되는 버전

지금(v0.9)은 그냥 `main`만 써도 충분. v1.0부터 도입 검토.

### 7.3 Preview Deployments

Vercel은 **모든 브랜치와 PR(Pull Request)마다 자동으로 미리보기 URL** 만듦:

```
main 브랜치          → family-worship-app-pi.vercel.app
feature/알림 브랜치   → family-worship-app-pi-git-feature-alarm.vercel.app
```

→ 기능 개발하면서 실제 배포 전에 테스트 가능.

---

## 8. 문제 해결 (Troubleshooting)

### 8.1 "command not found: npm"

**원인**: Node.js 경로 문제, 셸 환경 문제

**해결**:
```bash
# 1. 새 터미널 열기
# 2. Node.js 확인
node --version

# 3. 그래도 안 되면 재설치 필요
# https://nodejs.org/ 에서 LTS 버전 설치
```

### 8.2 "Port 5173 is already in use"

**원인**: 다른 서버가 이미 5173 포트 사용 중

**해결**:
```bash
# 모든 node 프로세스 종료
killall node

# 다시 시작
cd ~/family-worship-app
npm run dev
```

### 8.3 Vite 자동 새로고침 안 됨

**원인**: 파일 감시 실패 (드래그 방식 이동 시 자주 발생)

**해결**: 서버 재시작
```bash
Ctrl + C
npm run dev
```

### 8.4 "Permission denied" 에러

**원인**: 권한 문제

**해결**:
```bash
# 소유권 확인
ls -la ~/family-worship-app

# 소유권 변경 (필요 시)
sudo chown -R $(whoami) ~/family-worship-app
```

### 8.5 Git push 거부됨 (rejected)

**원인**: 원격에 내가 없는 커밋이 있음

**해결**:
```bash
# 원격 가져오기
git pull --rebase

# 충돌 없으면 push
git push

# 충돌 있으면 수동 해결 후
git add .
git rebase --continue
git push
```

### 8.6 "Updates were rejected because the remote contains work"

**해결**:
```bash
# 우리 버전으로 강제 (주의: 원격 변경 사라짐)
git push --force-with-lease

# 또는 원격 우선 병합
git pull --rebase origin main
# 충돌 해결 후
git push
```

### 8.7 Vercel 빌드 실패

**확인 순서**:
1. Vercel 대시보드 → Deployments → 실패 배포 클릭
2. Build Logs 확인
3. 에러 메시지 읽기

**자주 있는 원인 + 해결**:

```
❌ Cannot find module 'xxx'
→ npm install xxx && git add package.json package-lock.json && git commit && git push

❌ Unexpected token
→ JavaScript 문법 오류. 로컬에서 npm run build 먼저 테스트

❌ Out of memory
→ 큰 이미지 import 확인. 외부 CDN(jsDelivr) 사용 권장
```

### 8.8 브라우저에서 변경이 안 보임

**순서대로 시도**:

1. `⌘+Shift+R` (강제 새로고침)
2. Chrome DevTools (F12) → Network 탭 → "Disable cache" 체크
3. 시크릿 창에서 열기
4. 다른 브라우저 테스트
5. Vercel 대시보드에서 배포 완료 확인

### 8.9 로컬은 되는데 Vercel에서는 안 됨

**원인**:
- 환경 변수 누락
- 대소문자 파일명 차이 (macOS는 관대, 리눅스는 엄격)
- 절대 경로 사용

**해결**:
```bash
# 로컬에서 프로덕션 빌드 테스트
npm run build
npm run preview

# 여기서 재현되면 원인 쉽게 발견
```

---

## 9. 자주 쓰는 명령어 치트시트

### 🔥 매일 쓰는 TOP 5

```bash
cd ~/family-worship-app        # 프로젝트로 이동
npm run dev                    # 개발 서버 시작
Ctrl + C                       # 서버 중지
git add . && git commit -m "..." && git push   # 한 번에 배포
⌘ + Shift + R                  # Chrome 강제 새로고침
```

### 📦 프로젝트 설정

```bash
# 의존성 설치 (처음 또는 package.json 변경 후)
npm install

# 패키지 추가
npm install 패키지명
npm install -D 패키지명        # 개발용 패키지

# 패키지 업데이트
npm update

# 보안 취약점 확인
npm audit
npm audit fix
```

### 🏗️ 빌드 & 테스트

```bash
# 개발 서버
npm run dev

# 프로덕션 빌드
npm run build

# 빌드 결과 테스트 (로컬에서 프로덕션처럼)
npm run preview
```

### 📂 파일 작업

```bash
# 특정 파일 존재 확인
ls ~/family-worship-app/src/App.jsx

# 파일 용량 확인
du -h ~/family-worship-app/src/App.jsx

# 라인 수 확인
wc -l ~/family-worship-app/src/App.jsx

# 특정 단어 검색
grep -n "Logomong" ~/family-worship-app/src/App.jsx
```

### 🌿 Git 일상

```bash
# 현재 상태
git status

# 변경 내용 보기
git diff

# 최근 커밋 보기
git log --oneline -10

# 특정 파일 변경 이력
git log --follow -p -- src/App.jsx

# 브랜치 확인
git branch

# 원격 저장소 확인
git remote -v

# 태그 추가 (버전 마킹)
git tag v0.9.0
git push --tags
```

### 🔧 문제 해결

```bash
# 포트 프로세스 확인
lsof -i :5173

# 모든 Node 프로세스 종료
killall node

# npm 캐시 비우기
npm cache clean --force

# node_modules 완전 재설치
rm -rf node_modules package-lock.json
npm install

# Git 변경 모두 취소 (주의!)
git reset --hard HEAD
git clean -fd
```

### 🚀 한 방 배포 (자주 씀)

```bash
# 다운받은 App.jsx 적용 + GitHub push + 자동 배포
cp ~/Downloads/App.jsx ~/family-worship-app/src/App.jsx && \
cd ~/family-worship-app && \
git add . && \
git commit -m "feat: 새 기능 추가" && \
git push && \
echo "✅ 배포 시작! 1~2분 후 family-worship-app-pi.vercel.app 에 반영"
```

### 🎨 자주 찾는 URL

```
로컬 개발: http://localhost:5173/
프로덕션: https://family-worship-app-pi.vercel.app
GitHub 앱: https://github.com/junho908-svg/family-worship-app
GitHub BGM: https://github.com/junho908-svg/family-worship-bgm
Vercel 대시보드: https://vercel.com/junho908-3556s-projects
jsDelivr 캐시 무효화: https://purge.jsdelivr.net/gh/junho908-svg/family-worship-bgm@main/파일명
```

---

## 💡 보너스: 편한 작업 환경 만들기

### 1. Terminal Alias (단축어)

`~/.zshrc` 파일에 추가:

```bash
# 파일 열기
open ~/.zshrc

# 파일 안에 추가
alias worship='cd ~/family-worship-app'
alias dev='cd ~/family-worship-app && npm run dev'
alias push='git add . && git commit -m "update" && git push'
```

적용:
```bash
source ~/.zshrc
```

이제부터:
```bash
dev     # 프로젝트 폴더로 가서 서버 시작
push    # 한 번에 GitHub 업로드
```

### 2. VSCode 설치 (선택)

무료 에디터 ([code.visualstudio.com](https://code.visualstudio.com))

**필수 확장**:
- **ES7+ React/Redux/React-Native** snippets
- **Tailwind CSS IntelliSense**
- **GitLens**
- **Prettier - Code formatter**

### 3. Chrome 북마크 모음

북마크 폴더 "우리집 예배":
- 🌐 Production: family-worship-app-pi.vercel.app
- 💻 Localhost: localhost:5173
- 📦 GitHub 앱: github.com/junho908-svg/family-worship-app
- 🎵 GitHub 미디어: github.com/junho908-svg/family-worship-bgm
- 🚀 Vercel 대시보드
- 🧹 jsDelivr 캐시 purge (자주 씀)

---

## 🎯 실전 시나리오

### 시나리오 A: 오타 하나만 수정

```bash
# 1. 파일 직접 수정 (에디터에서)

# 2. 빠른 배포
cd ~/family-worship-app
git add .
git commit -m "fix: 오타 수정"
git push
```

**총 소요**: 10초 + Vercel 빌드 1~2분

### 시나리오 B: 클로드에서 받은 새 기능 적용

```bash
# 1. 파일 적용
cp ~/Downloads/App.jsx ~/family-worship-app/src/App.jsx

# 2. 로컬 테스트 (이미 dev 서버 켜져있으면 자동 새로고침)
# Chrome에서 ⌘+Shift+R

# 3. 문제 없으면 배포
cd ~/family-worship-app
git add .
git commit -m "feat: 로고몽 축하 애니메이션 추가"
git push
```

### 시나리오 C: BGM 새 곡 10개 추가

```bash
# 1. GitHub Desktop 사용
# - family-worship-bgm 리포지토리 열기
# - Repository → Show in Finder
# - 10개 MP3 드래그해서 폴더에 넣기
# - GitHub Desktop으로 돌아가서 Commit → Push

# 2. App.jsx에서 BGM 목록에 추가
cp ~/Downloads/App.jsx ~/family-worship-app/src/App.jsx

# 3. 배포
cd ~/family-worship-app
git add .
git commit -m "feat: BGM 10곡 추가"
git push
```

### 시나리오 D: 긴급 롤백 (v0.9로 되돌리기)

```bash
cd ~/family-worship-app

# 최근 커밋 이력
git log --oneline -10

# 원하는 커밋으로 되돌리기 (커밋 해시 복사)
git revert 커밋해시

# GitHub에 반영
git push
```

Vercel이 자동으로 이전 버전으로 재배포합니다.

---

## 📝 마무리

이 문서를 북마크해두고 막힐 때마다 열어보세요.

특히 자주 쓸 것들:
- ✅ **로컬 서버 시작/중지** (3.1, 3.2)
- ✅ **GitHub 일상 업로드** (5.4)
- ✅ **문제 해결** (8장)
- ✅ **한 방 배포 명령어** (9장)

대표님 IT 전문가시라 반은 이미 아실 거예요. 나머지 반은 몇 번 하면 익숙해집니다.

**질문이나 막힌 부분 생기면 언제든 물어보세요!** 🙏

---

**문서 버전**: v1.0  
**작성일**: 2026.04.21  
**작성자**: 전준호 (with Claude)
