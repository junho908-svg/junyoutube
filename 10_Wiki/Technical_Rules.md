---
id: f5e6d7c8-90ab-cdef-1234-567890abcde
category: "10_Wiki/Technical"
confidence_score: 1.0
tags: [Rule, Engine, API]
last_reinforced: 2026-04-20
---
# 기술 및 운영 규칙 (Technical & Operational Rules)

## 📌 한 줄 통찰
모든 출력물은 '시크하게 일하고, 경건하게 출력한다'는 원칙에 따라 데이터 기반의 효율적인 AI 자동화 시스템을 운영한다.

## 📖 구조화된 지식
- **Rule 1: Music Generation & Structure (Lyria + Suno)**:
    - 생성 길이 최적화: Suno API 연동으로 3~5분 이상의 완전한 곡 생성 우선.
    - 콘텐츠 구조: 시리즈별 길이 기준(새벽기도 60분 / QT 30~45분 등) 준수 및 루프 기법 적용.
    - 믹스 마스터링: 크로스페이드 2초, LUFS -14dB 정규화 필수.
- **Rule 2: API Integration & Error Handling**:
    - API 호출 원칙: 할당량 체크 및 캐싱 기본 적용.
    - 에러 핸들링: API 장애 시, 대안 루트를 찾아 프로세스를 완수하는 '선제작 후보고' 체계 자동 가동.
- **Rule 3: File & Asset Management**:
    - 분리 보관: 코드는 `/src/`, 미디어는 `assets/media/`에 격리하여 인덱싱 부하 최소화.
- **Rule 4: Bible Citation Rules**:
    - 번역본 고정: 개역개정 사용.
    - 출처 표기: `"구절" – 성경 이름` 형식 준수.
    - 영상당 최소 2회 인용 (설명란 및 고정 댓글).

## 🔗 지식 연결
- Parent: [[10_Wiki/Missions]]
- Related: [API, Multimedia, Citation]
- Raw Source: [[00_Raw/2026-04-20/agent-malsum-chanyang_SKILL_v2.0.md]]