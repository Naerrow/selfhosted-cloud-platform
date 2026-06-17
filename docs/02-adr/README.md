# Architecture Decision Records (ADR)

본 디렉터리는 본 프로젝트의 **모든 설계 의사결정**을 기록한다.
"왜 그렇게 했는가"의 단일 진실 공급원이다.

## 작성 규칙

ADR의 형식·라이프사이클·번호 부여 규칙은 [`0001-record-architecture-decisions.md`](./0001-record-architecture-decisions.md)를 따른다.

## 인덱스

| 번호 | 제목 | 상태 |
|---|---|---|
| [0001](./0001-record-architecture-decisions.md) | 아키텍처 의사결정을 ADR로 기록한다 | Accepted |
| [0002](./0002-use-proxmox-as-hypervisor.md) | 하이퍼바이저로 Proxmox VE를 채택한다 | Accepted |
| [0003](./0003-not-using-proxmox-clustering.md) | Proxmox 자체 클러스터링을 사용하지 않고 자체 오케스트레이션 레이어를 구현한다 | Accepted |
| [0004](./0004-backend-language-and-framework.md) | 백엔드 언어/프레임워크 선택 | Accepted |
| [0005](./0005-multitenancy-isolation-model.md) | 멀티테넌시 격리 모델 | Accepted |
| [0006](./0006-billing-integrity-mechanism.md) | 빌링 데이터 무결성 보장 방식 | Accepted |
| [0007](./0007-secret-management.md) | 시크릿 관리 방식 | Proposed |
| [0008](./0008-phase3-billing-scope.md) | Phase 3 빌링 스코프를 종량제 단일 모델로 한정한다 | Accepted |
| [0009](./0009-internal-only-design-docs.md) | 일부 설계 산출물을 내부 전용으로 분리한다 | Accepted |
| [0010](./0010-research-doc-correction-exception.md) | 문서 정정과 설계 변경을 구분한다 | Accepted |
| [0011](./0011-license-apache-2.0.md) | 본 프로젝트 라이선스로 Apache License 2.0을 채택한다 | Accepted |

> 번호는 결정 순서가 아니라 **주제별로 미리 할당**하는 경우가 있다. 인덱스의 상태를 항상 신뢰한다.

## 상태 라이프사이클

```
Proposed → Accepted → (Deprecated 또는 Superseded)
```

상태 정의는 0001 참조.
