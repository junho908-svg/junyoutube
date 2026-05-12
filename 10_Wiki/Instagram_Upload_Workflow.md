id: c3d4e5f6-7890-abcd-ef01-234567890123
category: "🚀 Skills"
confidence_score: 0.99
tags: [API 연동, Graph API, 시퀀스]
last_reinforced: 2026-04-23
---
# 인스타그램 업로드 워크플로우 (Graph API v23.0)

## 📌 한 줄 통찰
> 성공적인 게시물 발행을 위해 Media Container 생성 → 상태 폴링 → Media Publish의 3단계 시퀀스를 엄격하게 따름.

## 📖 구조화된 지식
- **Stage 1: Media Container 생성**:
    - API: `POST /{ig-user-id}/media`
    - 결과: `container_id` 획득.
- **Stage 2: 상태 폴링 (Polling)**:
    - API: `GET /{container_id}?fields=status_code`
    - 목적: `IN_PROGRESS`에서 `FINISHED` 상태로 전환될 때까지 대기.
    - 제약: 분당 1회, 최대 5분 폴링 권고.
- **Stage 3: Media Publish**:
    - API: `POST /{ig-user-id}/media_publish`
    - 목적: 최종 게시물 ID(`post_id`) 획득.
- **안전 장치**: DRY-RUN 모드를 통해 실제 호출을 방지하고, 에러 발생 시 `UploadError`로 래핑하여 예외 처리 강화.

## 🔗 지식 연결
- Parent: [[00_Raw/2026-04-23/AGENT_CONTROL_____________2026-04-23.md]]
- Related: [[10_Wiki/Risk_Management/Crisis_Response_Protocol.md]], [[4.2 `insta_uploader.py` 설명]]
- Raw Source: [[00_Raw/2026-04-23/AGENT_CONTROL_____________2026-04-23.md]]