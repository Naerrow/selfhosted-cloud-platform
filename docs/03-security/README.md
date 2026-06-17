# 03-security

본 디렉터리는 보안 설계 산출물을 보관한다. 다음 두 영역을 다룬다.

| 하위 항목 | 채워질 Phase | 산출물 |
|---|---|---|
| [`compliance-mapping/`](./compliance-mapping/) | Phase 0 (CSAP·개인정보보호법 v0) → Phase 5 이후 정식화 | 한국 규제 통제 항목 매핑 표 |
| [`threat-model.md`](./threat-model.md) | Phase 0 (STRIDE v0 ✅) → Phase별 갱신 | STRIDE 기반 위협 모델, 신뢰 경계 정의 |

전체 단계 정의는 [`../../ROADMAP.md`](../../ROADMAP.md) Phase 0 / Phase 2 / Phase 3 참조.

## 운영 원칙

- 모든 1차 출처 인용에는 확인일 명시 (R-003·R-005 완화).
- 변경 절차는 [ADR-0010](../02-adr/0010-research-doc-correction-exception.md)의 4단계 분류 체계 적용.
- 본 디렉터리 산출물은 외부 공개 추적 문서다. ADR-0009의 내부 전용 산출물 대상은 아니므로, 공개 문서 규칙을 따른다.
