# Selfhosted Cloud Platform

> Proxmox VE를 하이퍼바이저 엔진으로 활용해, 서버부터 미니 PC(홈랩)까지 다양한 하드웨어 환경에서
> **AWS와 유사한 IaaS와 빌링 기능**을 셀프호스팅으로 제공하는 플랫폼.

설계 원칙은 **기능의 폭이 아니라 운영·보안·감사 가능성의 깊이**다.
모든 주요 의사결정은 ADR로, 운영 시나리오는 런북으로, 위협은 위협 모델로 명시한다.

---

## 한 줄 가치 명제

> "단일 미니 PC부터 다중 서버까지, **동일한 콘솔과 빌링 흐름**으로 운영하는 셀프호스팅 클라우드.
> 단, 기능의 폭보다 **운영·보안·감사 가능성의 깊이**를 우선한다."

## 현재 상태

- **Phase 0 — Foundation** (설계 문서 작성 중)
- 다음 진입: Phase 1 (Vertical Slice MVP)

전체 로드맵은 [`ROADMAP.md`](./ROADMAP.md) 참고.

## 빠른 둘러보기

| 보고 싶은 것 | 경로 |
|---|---|
| 비전·스코프·페르소나 | [`docs/00-overview/`](./docs/00-overview/) |
| 아키텍처 다이어그램(C4) | [`docs/01-architecture/`](./docs/01-architecture/) |
| 모든 설계 의사결정(ADR) | [`docs/02-adr/`](./docs/02-adr/) |
| 위협 모델·컴플라이언스 매핑 | [`docs/03-security/`](./docs/03-security/) |
| 운영·관측·SLO·런북 | [`docs/04-operations/`](./docs/04-operations/) |
| 빌링 설계 | [`docs/05-billing/`](./docs/05-billing/) |
| 사전 조사 자료 | [`docs/06-research/`](./docs/06-research/) |
| 일자별 작업 일지·리스크 | [`docs/90-meta/`](./docs/90-meta/) |

## 기술 스택 (잠정)

구체 스택은 ADR-0004 이후 확정. 현재까지 결정된 사항:

- **하이퍼바이저**: Proxmox VE (ADR-0002)
- **오케스트레이션**: 자체 컨트롤 플레인 (Proxmox 자체 클러스터링 미사용 — ADR-0003)
- 이외 항목은 Phase 0 ADR로 순차 확정.

## 이 프로젝트가 "안 하는" 것

명세 기준으로 의도적으로 제외한 항목은 [`docs/00-overview/scope.md`](./docs/00-overview/scope.md)에
사유와 함께 명시되어 있다. 스코프와 그 사유를 함께 읽으면 설계 의도가 더 명확해진다.

## 법적 면책

본 프로젝트의 한국 규제·법령 인용(CSAP, 개인정보보호법, 전기통신사업법, 전자상거래법 등)은
[`docs/06-research/cloud-platforms-and-regulations.md`](./docs/06-research/cloud-platforms-and-regulations.md)
작성 시점의 1차 출처 기반 일반 정보 제공이며, **법률 자문이 아니다**.
실제 사업 적용·인증 신청·사업자 등록 등의 의사결정에는 변호사·관할 기관 검토가 별도로 필요하다.

## 라이선스

[Apache License 2.0](./LICENSE) (ADR-0011).
