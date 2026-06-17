# 01-architecture

**채워질 Phase**: 0 (C4 Level 1–2 다이어그램 v0 — Phase 0 Exit 게이트) → Phase별 갱신.
**산출물**: C4 모델 다이어그램 (System Context, Containers), 데이터 플로우, 신뢰 경계.

전체 단계 정의는 [`../../ROADMAP.md`](../../ROADMAP.md) Phase 0 산출물 + 횡단 원칙 참조.

## 산출물 (v0 — Phase 0)

| 문서 | 내용 |
|---|---|
| [`c4-level1-context.md`](./c4-level1-context.md) | C4 Level 1 — System Context. 4 페르소나 + Proxmox/IdP/PG 외부 시스템. |
| [`c4-level2-containers.md`](./c4-level2-containers.md) | C4 Level 2 — Containers. Go 모듈러 모놀리스 + PostgreSQL 단일 (ADR-0004 반영). |
| [`trust-boundaries.md`](./trust-boundaries.md) | 신뢰 경계(TB-1~TB-7)·데이터 플로우. STRIDE 위협 모델 v0의 직접 입력. |

> Level 2는 **ADR-0004·ADR-0005·ADR-0006(Accepted)** 의 백엔드 스택·멀티테넌시 격리·빌링 무결성 결정을 반영한다. 시크릿 관리(ADR-0007)는 아직 **Proposed** 이므로, 시크릿 저장소가 Accepted로 확정되면 본 다이어그램을 갱신한다.
