id: a1b2c3d4-e5f6-7890-abcd-ef0123456789
category: "🛠️ Projects"
confidence_score: 0.98
tags: [시스템 설계, FastAPI, HTML, 분리 구조]
last_reinforced: 2026-04-23
---
# 시스템 아키텍처 개요

## 📌 한 줄 통찰
> 단일 사용자 프라이빗 대시보드 구축을 위해 'HTML + FastAPI' 경로를 채택하여 최소 의존성과 유지보수성을 확보함.

## 📖 구조화된 지식
- **채택 경로**: HTML (프론트엔드) + FastAPI (백엔드).
- **선택 이유**: 다중 사용자/SEO 불필요, 오버엔지니어링 회피, 높은 유지보수성.
- **데이터 흐름**: `dashboard.html` ↔ `/api/*` 엔드포인트 ↔ `server.py`.
- **핵심 모듈 분리**: 상태 관리(`state_manager.py`), 업로드 로직(`insta_uploader.py`), 이벤트 로그(`event_log.py`)를 명확히 분리하여 기능적 독립성 확보.

## 🔗 지식 연결
- Parent: [[00_Raw/2026-04-23/AGENT_CONTROL_____________2026-04-23.md]]
- Related: [[10_Wiki/Project_Workflow_Audio_Video.md]], [[10_Wiki/Technical_Rules.md]]
- Raw Source: [[00_Raw/2026-04-23/AGENT_CONTROL_____________2026-04-23.md]]