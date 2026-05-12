id: b2c3d4e5-f678-90ab-cdef-012345678901
category: "⚖️ Decisions"
confidence_score: 0.99
tags: [상태 관리, 원자성, JSON 처리]
last_reinforced: 2026-04-23
---
# 상태 관리 프로토콜 (state.json 안전 관리)

## 📌 한 줄 통찰
> `state.json`의 데이터 정합성과 원자성을 보장하기 위해 '기본값 머지'와 '원자적 쓰기(os.replace)' 전략을 사용함.

## 📖 구조화된 지식
- **핵심 설계**: `{**DEFAULT_STATE, **loaded}` 패턴으로 누락된 필드는 기본값을 채우고 기존 값은 유지하여 데이터 손실 방지.
- **원자적 쓰기**: `tempfile` 생성 후 `os.replace()`를 사용하여 파일 시스템 수준의 원자성 보장.
- **데이터 무결성**: JSON 파손 시 `.corrupted.json`으로 백업 후 기본값 복구 메커니즘 포함.
- **시간 관리**: 모든 저장 시점에 `last_updated` 필드를 KST ISO 포맷으로 자동 갱신하여 시간대 문제를 방지.

## 🔗 지식 연결
- Parent: [[00_Raw/2026-04-23/AGENT_CONTROL_____________2026-04-23.md]]
- Related: [[10_Wiki/Technical_Rules.md]], [[4.1 `state_manager.py` 설명]]
- Raw Source: [[00_Raw/2026-04-23/AGENT_CONTROL_____________2026-04-23.md]]