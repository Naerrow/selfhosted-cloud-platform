# 04-operations

본 디렉터리는 운영 산출물을 보관한다. 본 프로젝트의 가치 명제 "운영·보안·감사 가능성의 깊이"가 직접 반영되는 영역.

| 하위 항목 | 채워질 Phase | 산출물 |
|---|---|---|
| [`runbooks/`](./runbooks/) | Phase 1 (사고·작업별 절차서) → Phase별 추가 | 헬스체크 실패, 빌링 검증, 노드 등록·해제, 시크릿 회전 등 |
| `slo.md` (예정) | Phase 1 (관측 가능성 설계와 함께) | SLO/SLI 정의, 오류 예산 정책 |
| `observability.md` (예정) | Phase 1 | 메트릭·로그·트레이스 수집 설계 |

전체 단계 정의는 [`../../ROADMAP.md`](../../ROADMAP.md) 횡단 원칙 + Phase 1·4 참조.

## 원칙

- **사람 승인 모드 유지**: 자율 자동 조치는 [`../00-overview/scope.md`](../00-overview/scope.md) OUT-OF-SCOPE. 런북은 사람 승인 단계를 항상 명시한다.
- **감사 로그 캡처**: 모든 상태 변경 이벤트는 누가/언제/무엇을/어디서 + 페르소나 코드 (P-OP/P-CSP/P-TA/P-EU) 기록.
- **변경 절차**: [ADR-0010](../02-adr/0010-research-doc-correction-exception.md) 4단계 분류 체계 적용.
