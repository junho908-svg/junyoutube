# AGENT CONTROL 대시보드 구축 작업 기록

> **작업일**: 2026년 4월 23일 (목)
> **작업자**: 대표님 + 클로드 (총괄 매니저)
> **프로젝트**: 준 인스타그램 에이전트 — 시네마틱 프라이빗 대시보드
> **버전**: v1.0 (한글화 v2 적용)
> **저장 위치**: `~/dev/insta_dashboard/`

---

## 0. 요약 (Executive Summary)

하루 동안 인스타그램 자동 운영 에이전트의 **조종실(Cockpit) 대시보드**를 0에서 완성도 높은 상태까지 구축했다. HTML 단일 파일 프론트엔드 + FastAPI Python 백엔드 구조로, Instagram Graph API v23.0 발행 3단계 시퀀스 전체와 state 영속 관리, 에이전트 로그 스트림, 시스템 상태 인디케이터, 주간 콘텐츠 캘린더가 모두 정상 작동하는 상태이다. DRY-RUN 모드 덕에 실제 인스타그램 계정에 영향을 주지 않고 전체 워크플로우를 안전하게 검증할 수 있다.

작업은 **시작 코드 분석 → 채널 전략 수립 → Day 1 콘텐츠 기획 → 백엔드 API 설계 → UI 풀 목업 → 실제 서버 통합 → 로컬 구동 → 한글화** 순으로 진행되었다.

---

## 1. 프로젝트 배경

### 1.1 출발점
대표님이 기존에 사용 중이던 `main_automation.py` 코드와 `state.json` 파일을 업로드하면서 시작. 기존 코드는 단일 스크립트 형태로 트렌드 수집 → 이미지 프롬프트 생성 → 캡션 생성 → 인스타그램 업로드의 일일 파이프라인을 돌리는 구조였으나, 다음과 같은 결함이 있었다.

### 1.2 발견된 결함 7가지

| # | 결함 | 심각도 |
|---|---|---|
| 1 | `state.json`의 커스텀 필드(`last_updated`, `status`, `target_channel`)가 코드에서 한 번도 갱신되지 않고 영원히 멈춤 | 🔴 치명 |
| 2 | 이미지 생성이 사실상 미구현 — `fallback_image_url`만 사용해 매일 같은 이미지 발행 | 🔴 치명 |
| 3 | 예외 처리 전무 (토큰 만료, rate limit, 컨테이너 폴링 에러 모두 무방어) | 🟠 중대 |
| 4 | 캡션·해시태그가 너무 정적 (5개 고정, 첫 줄이 알고리즘 후킹 신호인데 약함) | 🟡 보통 |
| 5 | `datetime.now()` 타임존 미지정 → 서버 로케일 의존 | 🟡 보통 |
| 6 | `.env` 파서 조악 (따옴표·주석·공백 처리 없음) | 🟡 보통 |
| 7 | 동시성 보호 없음 (cron 중복 트리거 시 state.json 덮어쓰기 위험) | 🟢 경미 |

### 1.3 "에이전트"라기엔 부족한 부분
- 성과 회수 부재 (좋아요·댓글·저장·도달을 Graph API Insights로 회수하는 로직 없음)
- 콘텐츠 다양성 정책 부재 (포맷 로테이션, 주제 카테고리 로테이션 미정의)
- 브랜드 가드레일 부재 ("준 인스타그램 에이전트" 채널의 톤·색감·페르소나 미정의)

---

## 2. 채널 전략 확정

### 2.1 채널 정체성
- **타겟**: AI/자동화에 관심 있는 MZ세대
- **콘셉트**: AI 에이전트가 직접 운영하는 인스타그램 채널
- **톤**: 전문적이면서 트렌디, 너무 딱딱하지 않게
- **메타 훅**: "에이전트가 직접 운영한다" — 7일 내내 포기하지 않을 핵심 내러티브
- **채널의 해자(moat)**: 평범한 AI 관심사 계정과 차별화되는 메타 운영 콘셉트

### 2.2 디자인 시스템
- **색상**: 사이버네틱 다크 테마
  - 배경 `#070D17` (거의 검정)
  - 패널 `#0D1B2A`
  - 액센트 시안 `#00D9FF`
  - 보조 코랄 `#FF6B9D`, 앰버 `#FFB84D`
  - 텍스트 `#E8F1F2` / `#A3B8C7` / `#5C7A92`
- **타이포**: Pretendard Variable (한글) + JetBrains Mono (수치)
- **무드**: 여백 큼, 정보 밀도 낮음, 미세한 글로우, 모노스페이스 터미널 감성
- **인터랙션**: hover 시 subtle glow, 클릭 시 미세 scale transform
- **레퍼런스**: NASA 관제실 × Bloomberg Terminal × Apple Vision Pro

### 2.3 주간 1주차 콘텐츠 캘린더 (Week 1 Arc)

| Day | 요일 | 포맷 | 역할 | 주제 | 핵심 감정 |
|---|---|---|---|---|---|
| 1 | 수 (4/23) | 피드 | 선언 | AI 에이전트의 출사표 | 호기심 |
| 2 | 목 (4/24) | 릴스 | Behind-the-Scenes | 에이전트의 24시간 | 투명성 |
| 3 | 금 (4/25) | 캐러셀 | 교육 | 에이전트 vs 자동화, 뭐가 다를까? | 유익 |
| 4 | 토 (4/26) | 릴스 | 밈·공감 | AI가 본 개발자의 월요일 | 재미 |
| 5 | 일 (4/27) | 피드 | 시네마틱 감상 | 시네마틱 일요일 · 서울 스카이라인 | 미감 |
| 6 | 월 (4/28) | 캐러셀 | 실전 | 나도 AI 에이전트 만들기 · 체크리스트 | 실용 |
| 7 | 화 (4/29) | 릴스 | 1주차 회고 | 에이전트가 직접 쓴 회고 | 친밀 |

**포맷 분배**: 릴스 3 / 캐러셀 2 / 피드 2 (3R/2C/2F)

**Week 1 KPI**:
- 팔로워: 500+
- 게시물당 평균 저장률: 2.5%
- 댓글 50개 이상 (수동 응답 필수)

### 2.4 Day 1 발행 디테일

**선택된 키워드**: `AI Agent` (점수 94.2)

**헤드라인**: "이 포스트를 기획한 건 사람이 아닙니다."

**캡션 구조**:
- 후킹 첫 줄: 도발/반전형
- 본문 342자
- 해시태그 13개 (대형 3 / 중형 4 / 소형·브랜디드 6 균형형)
- 마무리 정체성 선언 (Agent Log #001 / Generator: Claude Opus 4.7 / Human intervention: 0)

**이미지 콘셉트**: Cybernetic Dark 테마
- 시네마틱 와이드 샷, 미니멀 디자이너 데스크
- 빈 의자 + 김 올라오는 커피잔 + 모니터에 흐르는 neural network
- 창밖 황혼의 서울 야경
- Teal & Orange 그레이딩 (Kodak Portra 400 그레이드)
- 4:5 세로 비율 (피드 점유 면적 +25%)

**업로드 타이밍**: 4월 23일(수) 오후 9시 KST

---

## 3. 시스템 아키텍처

### 3.1 최종 결정 — "경로 A: HTML + FastAPI"

세 가지 경로를 비교한 끝에 채택:

| 경로 | 장점 | 단점 |
|---|---|---|
| **A. HTML + FastAPI** ✅ | 최소 의존성, 한눈에 파악, 유지보수 용이 | 컴포넌트 분해 약함 |
| B. Next.js 풀 프로젝트 | 컴포넌트 재사용, 생태계 풍부 | 파일 20개+, 빌드 도구 학습 필요 |
| C. Antigravity 위탁 | 작업량 최소 | 의도 100% 구현 보장 어려움 |

**채택 사유**: 단일 사용자 프라이빗 대시보드이므로 다중 사용자·SEO·복잡한 라우팅이 불필요. 오버엔지니어링 회피.

### 3.2 데이터 흐름

```
[브라우저: dashboard.html]
    │
    │ fetch('/api/state', '/api/calendar', '/api/logs', '/api/status')
    │ POST /api/publish  (Approve & Upload 클릭 시)
    ▼
[FastAPI 서버: server.py]
    │
    ├─→ state_manager.py     (state.json 안전 읽기/쓰기)
    ├─→ event_log.py         (인메모리 이벤트 버퍼)
    ├─→ insta_uploader.py    (Graph API 3단계 시퀀스)
    │       │
    │       ▼
    │   [Instagram Graph API v23.0]
    │       1) POST /media           → container_id
    │       2) GET /{container}/status_code (폴링)
    │       3) POST /media_publish   → post_id
    │
    └─→ insta_engine_v1/plans/day_NN.json (기획안 프리셋)
```

### 3.3 디렉토리 구조

```
insta_dashboard/
├── server.py                    # FastAPI — 9개 엔드포인트
├── insta_uploader.py            # Graph API v23.0 3단계 + DRY_RUN
├── state_manager.py             # 원자적 쓰기, 필드 보존
├── event_log.py                 # 인메모리 이벤트 버퍼 (max 200)
├── insta_engine_v1/
│   ├── state.json               # 영속 상태 (day, history, target_channel)
│   └── plans/
│       ├── day_01.json          # Day 1 풀 기획안 (캡션, 이미지, 트렌드)
│       └── day_02~07.json       # Week 1 캘린더 프리셋
├── static/
│   └── dashboard.html           # 시네마틱 대시보드 (서버 API 연동)
├── requirements.txt             # fastapi, uvicorn, requests, pydantic
├── .env.example                 # 환경변수 예시 (DRY_RUN=true 기본)
├── .gitignore
├── run.sh                       # 실행 스크립트
└── README.md                    # 프로젝트 문서
```

---

## 4. 핵심 모듈 설명

### 4.1 `state_manager.py` — 안전한 state.json 관리

**해결한 문제**: 기존 코드는 `state.json`의 커스텀 필드를 덮어쓰는 위험이 있었음.

**핵심 설계**:
- **기본값 머지 방식**: `{**DEFAULT_STATE, **loaded}` 패턴으로 누락된 필드만 채우고 기존 값 유지
- **원자적 쓰기**: `tempfile` 생성 → `os.replace()` 로 rename (POSIX 원자성 보장)
- **자동 백업**: JSON 파손 시 `.corrupted.json` 으로 백업 후 기본값 복구
- **KST ISO 자동 갱신**: 매 저장마다 `last_updated` 필드를 KST ISO 포맷으로 갱신
- **`target_channel` 방어선**: 빈 값이면 항상 기본값으로 복구

**핵심 함수**:
- `load_state()` — 머지된 state 반환
- `save_state(state)` — 원자적 쓰기 + last_updated 갱신
- `append_history(post_id, day, trend, topic)` — 발행 기록 추가 + day 증가

### 4.2 `insta_uploader.py` — Graph API 3단계 업로더

**Instagram Graph API v23.0 공식 발행 시퀀스**:

```
Stage 1: POST /{ig-user-id}/media
         ?image_url=<URL>&caption=<CAPTION>&access_token=<TOKEN>
         → container_id 수신

Stage 2: GET /{container_id}?fields=status_code
         → status_code ∈ {IN_PROGRESS, FINISHED, ERROR, EXPIRED}
         (Meta 권고: 분당 1회, 최대 5분)

Stage 3: POST /{ig-user-id}/media_publish
         ?creation_id=<CONTAINER_ID>&access_token=<TOKEN>
         → post_id 수신
```

**주요 기능**:
- **DRY_RUN 모드**: `.env`의 `DRY_RUN=true` 시 실제 API 호출 없이 가짜 `post_id` 반환
- **단계별 콜백**: `on_stage` 콜백으로 UI 로그 실시간 연동
- **타임아웃**: container 생성 30초, 폴링 15초, publish 30초
- **에러 처리**: Graph API 에러를 `UploadError`로 wrap
- **진단**: `check_token()`, `get_publish_quota()` 헬퍼 메서드

**제약사항**:
- Instagram Business 계정만 가능 (Personal 불가)
- 이미지는 공개 HTTPS URL + JPEG만 지원
- 24시간 이동 윈도우 기준 100회 발행 한도
- API 호출 200회/시간 한도

### 4.3 `event_log.py` — 인메모리 이벤트 버퍼

**설계**:
- `collections.deque(maxlen=200)` 원형 버퍼
- 스레드 안전 (`threading.Lock`)
- 시퀀스 ID로 증분 폴링 지원 (`since` 파라미터)
- KST 시각 자동 부여

**부팅 시퀀스 시드** (`seed_boot_sequence`):
```
기동      → 에이전트 기동. 실행 ID=#001
트렌드    → 오늘의 트렌드 수집: AI Agent (점수 94.2)
프롬프트  → 시네마틱 프롬프트 생성 완료. 토큰=287
이미지    → 이미지 렌더링 완료: flow_run_001.jpg
캡션      → 캡션 초안 작성 (342자 · 해시태그 13개)
호스팅    → 이미지 Cloudflare R2 업로드 완료
대기      → 승인 대기 중. 대표님의 승인을 기다립니다.
```

### 4.4 `server.py` — FastAPI 엔드포인트

| Method | Path | 설명 |
|---|---|---|
| GET | `/` | 대시보드 HTML 서빙 |
| GET | `/api/state` | state.json + 오늘의 기획안 통합 응답 |
| GET | `/api/plan/{day}` | 특정 Day 기획안 |
| GET | `/api/calendar` | 주간 캘린더 7일치 (status: today/done/upcoming) |
| GET | `/api/logs?since=N` | 에이전트 로그 (증분 폴링) |
| GET | `/api/status` | 시스템 상태 (토큰/quota/다음실행 카운트다운) |
| POST | `/api/publish` | Day 발행 (idempotency_key 필수) |
| POST | `/api/regenerate` | 재생성 (현재 스텁) |
| GET | `/api/health` | 헬스체크 |

**보안 장치**:
- **idempotency_key 중복 방지**: 같은 키 재요청 시 HTTP 409 "Already processed"
- **Day mismatch 방어**: 클라이언트 day와 서버 day 불일치 시 HTTP 409
- **Lock**: 동시 발행 방지 (`threading.Lock`)

---

## 5. 검증 결과 (8개 시나리오 모두 통과)

| # | 시나리오 | 결과 |
|---|---|---|
| 1 | 초기 상태 로드 (day=1, target_channel 보존) | ✅ |
| 2 | POST /api/publish (Day 1 발행, DRY_RUN) | ✅ HTTP 200, post_id 생성, 2.2초 소요 |
| 3 | 발행 후 state 변화 (day 1→2, history +1, last_updated 갱신, target_channel 보존) | ✅ |
| 4 | 중복 idempotency_key 방어 | ✅ HTTP 409 "Already processed" |
| 5 | Day mismatch 방어 (서버 day=2인데 day=1로 요청) | ✅ HTTP 409 "Day mismatch" |
| 6 | 발행 시퀀스 로그 (12개 이벤트 순차 기록) | ✅ |
| 7 | 캘린더 자동 전이 (Day 1 → done, Day 2 → today) | ✅ |
| 8 | 디스크 state.json 원본 정합성 | ✅ |

**대표적인 발행 시퀀스 로그 예시**:
```
[15:34:25] info 트렌드      오늘의 트렌드 수집: AI Agent (점수 94.2)
[15:34:25] ok   프롬프트    시네마틱 프롬프트 생성 완료. 토큰=287
[15:34:25] ok   이미지      이미지 렌더링 완료: flow_run_001.jpg
[15:34:25] info 캡션        캡션 초안 작성 (342자 · 해시태그 13개)
[15:34:25] info 호스팅      이미지 Cloudflare R2 업로드 완료
[15:34:25] warn 대기        승인 대기 중. 대표님의 승인을 기다립니다.
[15:34:25] info 업로드      1일차 업로드 시퀀스 시작
[15:34:25] ok   1단계       컨테이너 생성 완료 → dryrun_ctr_1776926065
[15:34:26] info 2단계       상태 폴링 #1 → IN_PROGRESS
[15:34:27] ok   2단계       컨테이너 준비 완료 (FINISHED)
[15:34:27] ok   3단계       발행 완료. 게시물 ID = dryrun_post_1776926067
[15:34:27] info 상태        state.day 1 → 2, 히스토리 +1
```

---

## 6. UI 영역별 구성 (대시보드 5개 영역)

### 6.1 상단 바 (Top Bar)
- AGENT CONTROL 로고 (시안 글로우, 3초 주기 펄스)
- 채널명 라벨 (state.target_channel 자동 반영)
- 실시간 KST 시계
- DAY 카운터 (`01 / 365`)
- 상태 (활성/일시정지/중지/오류)
- 에이전트 모델명 (Claude Opus 4.7)
- DRY-RUN 앰버 뱃지 (DRY_RUN=true일 때만 표시)

### 6.2 좌측 사이드바 (Navigation)
- **컨트롤 센터**: 대시보드 / 캘린더 / 성과 분석 / 히스토리
- **자동화**: 트렌드 엔진 / 설정
- 현재 active 표시 (시안 좌측 보더 + 글로우)
- 일자 배지 (D1, D2…)

### 6.3 중앙 Hero — Today's Plan
**좌측: 이미지 영역**
- 4:5 비율 이미지 + 그라디언트 오버레이
- 좌상단 테마 태그 (Cybernetic Dark)
- 우상단 비율 태그 (4:5 · JPEG)
- 좌하단 파일명, 우하단 해상도

**우측: 콘텐츠 영역**
- 트렌드 태그 (필 형태, 시안 글로우)
- 헤드라인 (특정 단어 시안 강조)
- 캡션 프리뷰 박스 (스크롤 가능, 하단 페이드)
- 스탯 3개 (캡션 자수 / 해시태그 개수 / 예정 시각)
- 액션 3개: 승인 후 업로드 / 재생성 / 편집

### 6.4 주간 캘린더 (Weekly Content Arc)
- 7일 카드 그리드
- 각 카드에 일자, 요일(한글), 주제, 포맷 필
- 포맷별 색상: 피드(시안) / 릴스(코랄) / 캐러셀(앰버)
- 상태별 표시:
  - `today` → NOW 배지 + 시안 글로우
  - `done` → ✓ 체크 + 55% 투명도
  - `upcoming` → 기본 상태

### 6.5 발행 히스토리 (Publication History)
- 빈 상태: 점선 박스 + 안내 문구
- 발행 후: 가로 스크롤 카드 타임라인
- 각 카드에 일자, 주제, 게시물 ID 일부 표시

### 6.6 우측 에이전트 로그 (Agent Log)
- 최대 540px 높이, 세로 스크롤
- 2초 주기 폴링 (증분만 가져옴)
- 좌측 시각 / 우측 본문 (태그 + 메시지)
- 태그 색상: info(시안) / ok(녹색) / warn(앰버) / err(빨강)
- 새 이벤트 슬라이드 인 애니메이션

### 6.7 우측 시스템 상태 (System Status)
- 액세스 토큰 (47일 남음 + 게이지)
- API 호출 한도 (3 / 200 · 시간당)
- 발행 한도 (0 / 100 · 24시간)
- 다음 에이전트 실행 (자정까지 카운트다운)
- 연결 상태 (● Graph API v23.0)

---

## 7. 한글화 v2 적용 내역

상단 바·사이드바·Hero 영역·주간 캘린더·히스토리·우측 패널·모달·로그 태그까지 **모든 사용자 가시 텍스트를 한글화**했다.

### 7.1 영역별 변경 표 (요약)

**상단 바**: LIVE/DAY/STATE/AGENT → 실시간/일차/상태/에이전트, active → 활성

**좌측 사이드바**:
- Control Center → 컨트롤 센터
- Dashboard → 대시보드
- Calendar → 캘린더
- Analytics → 성과 분석
- History → 히스토리
- Automation → 자동화
- Trend Engine → 트렌드 엔진
- Settings → 설정

**Hero**:
- Today's Plan → 오늘의 기획안
- AGENT_RUN_ID #001 → 에이전트 실행 ID #001
- TREND // → 트렌드 //
- Caption / Hashtags / Est. ETA → 캡션 / 해시태그 / 예정 시각
- **Approve & Upload → 승인 후 업로드**
- Regen → 재생성
- Edit → 편집
- Publishing... / ✓ Published / ✗ Failed → 업로드 중... / ✓ 발행 완료 / ✗ 실패

**주간 캘린더**:
- Weekly Content Arc — Week 1 → 주간 콘텐츠 · 1주차
- DAY 01 · WED → 01일차 · 수
- FEED / REELS / CAROUSEL → 피드 / 릴스 / 캐러셀

**히스토리**:
- Publication History → 발행 히스토리
- LIFETIME · X POSTS → 누적 X건

**우측 패널**:
- Agent Log Stream → 에이전트 로그
- X events → X건
- System Status → 시스템 상태
- REAL-TIME → 실시간
- Access Token → 액세스 토큰
- API Rate Limit → API 호출 한도
- Publish Quota (24h) → 발행 한도 (24시간)
- Next Agent Run → 다음 에이전트 실행
- Connection → 연결 상태

**발행 모달**:
- Final Confirmation → 최종 확인
- day/trend/format/caption/endpoint/target → 일차/트렌드/포맷/캡션/엔드포인트/발행 대상

**Agent Log 태그**:
- BOOT → 기동
- TREND → 트렌드
- PROMPT → 프롬프트
- IMAGE → 이미지
- CAPTION → 캡션
- HOST → 호스팅
- READY → 대기
- UPLOAD → 업로드
- STAGE 1/2/3 → 1단계/2단계/3단계
- STATE → 상태
- REGEN → 재생성
- CONFIG → 설정

---

## 8. 로컬 구동 방법 (Mac 기준)

### 8.1 최초 설치 (1회)

```bash
# 작업 폴더 준비
mkdir -p ~/dev
cd ~/dev

# zip 압축 해제
mv ~/Downloads/insta_dashboard.zip ~/dev/
unzip insta_dashboard.zip
cd insta_dashboard

# 가상환경 생성 + 활성화
python3 -m venv .venv
source .venv/bin/activate

# 의존성 설치
pip install -r requirements.txt

# 환경변수 설정
cp .env.example .env

# 서버 기동
python3 server.py
```

### 8.2 일상 사용 (3줄)

```bash
cd ~/dev/insta_dashboard
source .venv/bin/activate
python3 server.py
```

브라우저에서 **http://127.0.0.1:8000** 접속. 끄실 때는 터미널에서 `Ctrl + C`.

### 8.3 환경변수 (.env)

```env
# Instagram Graph API
INSTA_ACCOUNT_ID=17841400000000000
INSTA_ACCESS_TOKEN=your_long_lived_token_here
TOKEN_EXPIRES_IN_DAYS=47

# 개발 모드
DRY_RUN=true              # true = 실제 업로드 없이 가짜 post_id 반환

# 서버
PORT=8000
```

### 8.4 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| `ModuleNotFoundError: No module named 'fastapi'` | 가상환경 미활성화 | `source .venv/bin/activate` 후 재실행 |
| `Address already in use` | 이전 서버 프로세스 잔존 | `Ctrl+C` 또는 `.env`에서 `PORT=8001` 변경 |
| 브라우저 "서버 연결 실패" 빨간 뱃지 | 서버 미실행 또는 포트 불일치 | 서버 터미널 상태 확인 |
| 한글 깨짐 | Mac 터미널 인코딩 (드물지만) | `LANG=ko_KR.UTF-8` 설정 |
| 한글화 후에도 영문 표시 | 브라우저 캐시 | `⌘ + Shift + R` 강제 새로고침 |

### 8.5 새 버전 적용 시

```bash
cd ~/dev
mv insta_dashboard insta_dashboard_old   # 기존 백업
unzip ~/Downloads/insta_dashboard.zip    # 새 zip 풀기
cd insta_dashboard
cp .env.example .env                      # 또는 기존 .env 복사
source .venv/bin/activate                 # (또는 새로 만들기)
python3 server.py
```

브라우저는 반드시 **`⌘ + Shift + R`** 강제 새로고침.

---

## 9. 의사결정 기록 (Why)

### 9.1 왜 Next.js 안 갔나?
- 단일 사용자 프라이빗 대시보드
- SEO·다중 사용자·복잡한 라우팅 모두 불필요
- 빌드 도구·React 생태계 학습 비용이 효율 저하
- HTML 단일 파일이 한눈에 파악 가능, 디버깅 쉬움

### 9.2 왜 SQLite 안 쓰고 JSON 파일?
- 일일 1회 발행이라 동시성 부담 없음
- 텍스트 에디터로 직접 열어볼 수 있음 (디버깅 편의)
- 향후 history가 100건 이상 쌓이면 SQLite 마이그레이션 권장

### 9.3 왜 인메모리 로그 (DB 안 씀)?
- 프라이빗 단일 인스턴스이므로 휘발 OK
- 매일 자정 에이전트 재기동 시 로그 리셋이 자연스러움
- 향후 영속 필요 시 SQLite로 교체 가능

### 9.4 왜 DRY_RUN 모드를 1급 시민으로?
- 실제 인스타그램 발행은 **되돌릴 수 없음** (삭제만 가능, 알고리즘 신호는 남음)
- 모든 UI·워크플로우를 안전하게 검증 가능해야 함
- 키 발급 전에도 시스템 전체를 체험 가능
- DRY-RUN 앰버 뱃지로 항상 모드 시각화

### 9.5 왜 idempotency_key를 클라이언트가 생성?
- 사용자가 빠르게 두 번 클릭해도 한 번만 발행되도록
- `crypto.randomUUID()`로 충돌 가능성 사실상 0
- 서버는 set으로 중복 검사 (현재는 메모리, 프로덕션은 Redis 권장)

### 9.6 왜 폴링 (WebSocket 안 씀)?
- 단순함 우선
- 로그 2초·상태 5초·state 30초 주기로 충분
- 향후 실시간성 강화 필요 시 SSE(Server-Sent Events) 도입 가능

---

## 10. 알려진 한계 및 다음 작업

### 10.1 현재 스텁(미구현) 상태인 버튼
| 버튼 | 현재 동작 | 다음 작업 |
|---|---|---|
| 재생성 | 토스트만 표시 | 새 이미지 프롬프트 후보 3개 생성 |
| 편집 | "다음 버전" 안내 | 캡션 인라인 편집 모달 |
| 캘린더 일자 카드 | console.log만 | 클릭 시 해당 Day 상세 패널 |
| 사이드바 5개 (대시보드 외) | onclick 없음 | SPA 라우팅으로 페이지 전환 |

### 10.2 외부 API 연동 (다음 단계)
- **실시간 이슈 주제 선정**: Brave Search API + Claude API 조합 확정
- **이미지 생성 파이프라인**: Imagen 3 / Flux / Ideogram 중 선택
- **성과 회수 루프**: `GET /{media-id}/insights` 로 좋아요·저장·도달 수집

### 10.3 운영 모드 전환 시 체크리스트
- [ ] `.env`의 `DRY_RUN=false` 변경
- [ ] 실제 `INSTA_ACCOUNT_ID` (Instagram Business ID) 입력
- [ ] 실제 `INSTA_ACCESS_TOKEN` (장기 토큰, 60일 만료) 입력
- [ ] 이미지 호스팅 결정 (S3 / Cloudflare R2 / Supabase Storage)
- [ ] 이미지가 공개 HTTPS URL + JPEG인지 검증
- [ ] 토큰 자동 갱신 크론 설정 (매주 권장)
- [ ] Slack/이메일 알림 웹훅 연결 (실패 시 통보)

### 10.4 향후 마이그레이션 권장 시점
- **SQLite**: history 100건 이상 또는 통계 쿼리 필요해질 때
- **Redis**: 다중 인스턴스 또는 백그라운드 워커 도입 시 (idempotency_key 공유)
- **WebSocket/SSE**: 로그 지연이 체감될 때 (현재 2초로도 충분)

---

## 11. 외부 API 발급 가이드 (다음 작업용)

### 11.1 Brave Search API
- **URL**: https://brave.com/search/api/
- **무료 한도**: 월 2,000회, 분당 1회
- **결제수단 등록 필수** (악용 방지)
- **키 형식**: `BSA...`
- **용도**: 실시간 뉴스/트렌드 키워드 수집

### 11.2 Anthropic Claude API
- **URL**: https://console.anthropic.com/
- **결제**: 최소 $5부터 충전 (자동 결제 아님)
- **키 형식**: `sk-ant-api03-...`
- **용도**: Brave가 가져온 키워드를 채널 정체성에 맞게 큐레이션 + 풀패키지 생성
- **예상 비용**: 하루 1회 사용 시 월 $3~5 (Sonnet 기준), $1 미만 (Haiku 기준)

---

## 12. 파일 인벤토리

| 파일 | 역할 | 라인 수 |
|---|---|---|
| `server.py` | FastAPI 서버 | 약 250 |
| `insta_uploader.py` | Graph API 3단계 업로더 | 약 200 |
| `state_manager.py` | state.json 관리 | 약 80 |
| `event_log.py` | 인메모리 로그 | 약 60 |
| `static/dashboard.html` | 대시보드 (HTML+CSS+JS 단일 파일) | 약 1,150 |
| `insta_engine_v1/state.json` | 영속 상태 | 7 |
| `insta_engine_v1/plans/day_01.json` | Day 1 풀 기획안 | 약 50 |
| `insta_engine_v1/plans/day_02~07.json` | Day 2~7 캘린더 프리셋 | 각 1줄 |
| `requirements.txt` | Python 의존성 | 4 |
| `.env.example` | 환경변수 예시 | 12 |
| `run.sh` | 실행 스크립트 | 약 30 |
| `README.md` | 프로젝트 문서 | 약 100 |
| `.gitignore` | Git 제외 목록 | 6 |

**총 17개 파일 / 약 30KB (압축 시)**

---

## 13. 참고 — 기술 스택 정리

| 영역 | 선택 |
|---|---|
| 백엔드 | Python 3.10+ + FastAPI 0.110+ + uvicorn |
| 프론트엔드 | Vanilla HTML/CSS/JS (단일 파일) |
| 데이터 | JSON 파일 (state.json + plans/*.json) |
| HTTP 클라이언트 | requests |
| 검증 | pydantic v2 |
| 폰트 | Pretendard Variable + JetBrains Mono |
| 외부 API | Instagram Graph API v23.0 |
| 가상환경 | venv |
| 패키징 | zip |

---

## 14. 다음 대화 시작용 메모

다음 대화에서 이 프로젝트를 이어가실 때, 클로드에게 이 한 문장만 주시면 됩니다:

> "준 인스타그램 에이전트 대시보드 작업 이어가자. 어디까지 했지?"

클로드는 메모리에서 다음 정보를 즉시 복원합니다:
- 프로젝트 경로 `~/dev/insta_dashboard/`
- HTML + FastAPI 구조
- 한글화 v2 적용 완료
- DRY_RUN 모드 검증 완료
- 다음 작업 후보:
  1. 죽은 버튼 9개 활성화 (재생성/편집/캘린더 카드/사이드바 5개)
  2. 실시간 이슈 기능 (Brave Search + Claude API)
  3. 이미지 생성 파이프라인
  4. Day 2~7 풀 기획안 확장
  5. 성과 회수 루프

---

**문서 작성 완료 — 2026년 4월 23일 KST**

*by 총괄 매니저 (Claude Opus 4.7) · 대표님 전용 작업 기록*
