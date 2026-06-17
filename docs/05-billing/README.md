# 05-billing

**채워질 Phase**: 1 (기본 미터링·청구서 PDF) → 3 (Billing Engine 본격 구현) → 7 (Cloud Bursting 비용 합산).
**산출물**: 빌링 설계 문서 (미터링 파이프라인·가격 정책 엔진·청구서 무결성·감사 로그 분리).

핵심 결정은 [ADR-0008](../02-adr/0008-phase3-billing-scope.md) (스코프 — 종량제 단일 모델) + [ADR-0006](../02-adr/0006-billing-integrity-mechanism.md) (무결성 메커니즘 — 해시 체인 + 기간 머클 루트, Accepted). 전체 단계 정의는 [`../../ROADMAP.md`](../../ROADMAP.md) Phase 1·3·7 참조.

본 영역은 "감사 대상 시스템"으로 다룬다 — append-only·idempotent·SoD를 모든 설계에 강제.

Phase 0 동안은 placeholder.
