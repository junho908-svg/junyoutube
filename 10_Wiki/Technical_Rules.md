---
id: f5e6d7c8-90ab-cdef-1234-567890abcde
category: "10_Wiki/Technical"
confidence_score: 1.0
tags: [Music_Tech, API, Workflow]
last_reinforced: 2026-04-20
---
# 핵심 기술 규칙 (Core Technical Rules)

## 📌 한 줄 통찰
음악 생성 엔진의 효율성과 저작권 준수를 최우선으로 하며, API 오류 발생 시에도 프로세스를 중단하지 않고 대안을 모색한다.

## 📖 구조화된 지식
- **Rule 1: Music Generation & Structure (Lyria + Suno)**:
    - **생성 길이**: 롱폼(3~5분) 및 루프 기법 적용으로 최소 60분 이상의 시청 지속 시간 보장.
    - **구조**: 시리즈별 길이 기준(새벽기도 60분 / QT 30~45분 등)을 엄수한다.
    - **믹스 마스터링**: 크로스페이드 2초 적용, LUFS -14dB 정규화 필수.
- **Rule 2: API Integration & Error Handling**:
    - **API 호출**: 할당량 체크 및 캐싱 기본 적용.
    - **에러 처리**: API 장애 시 작업 중단 금지. '선제작 후보고' 체계로 대안 루트를 자동 탐색하여 프로세스 완수.
- **Rule 3: File & Asset Management**:
    - **분리 보관**: 미디어 파일은 `assets/media/`에 격리하고, 구버전 스크립트는 `/utils/archive/`에 저장한다.
- **Rule 4: Bible Citation Rules (Critical)**:
    - **번역본 고정**: 개역개정 사용.
    - **출처 표기**: 모든 성경 인용 시 출처를 먼저 밝히고 큰따옴표 사용 (`"성경 구절" – 책장`).
    - **영상당 최소 2회 인용**: 설명란 및 고정 댓글 양쪽에 반드시 포함한다.

## 🔗 지식 연결
- Parent: [[10_Wiki/Workflow]]
- Related: [API, Copyright, Quality_Control]
- Raw Source: [[00_Raw/2026-04-20/agent-malsum-chanyang_SKILL_v2.0.md]]