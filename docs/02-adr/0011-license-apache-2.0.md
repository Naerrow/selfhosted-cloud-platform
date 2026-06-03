# ADR-0011: 본 프로젝트 라이선스로 Apache License 2.0을 채택한다

- **상태(Status)**: Accepted
- **결정일(Date)**: 2026-06-03
- **결정자(Decider)**: 프로젝트 메인테이너
- **관련 ADR**: 0001 (ADR 운영 체계), 0002 (Proxmox 채택), 0008 (Phase 3 빌링 스코프), 0009 (내부 전용 산출물)

## Context — 맥락

본 프로젝트는 ADR-0001에서 **공개 개발 리포**로 운영하기로 결정했고, 라이선스는 "Phase 1 진입 시점에 별도 ADR로 결정"으로 미뤘다. 그러나 **첫 공개 커밋 시점이 곧 라이선스 결정의 효력 발생 시점**이며, 라이선스가 부재한 상태로 공개되면 다음 문제가 발생한다.

1. **GitHub 기본값**: 라이선스가 없으면 저작권법 기본값(모든 권리 보유)이 적용된다. 외부 기여자가 fork·수정·재배포할 법적 권리가 없으며, 사실상 "공개 코드처럼 보이지만 실제로는 폐쇄" 상태가 된다.
2. **Vision 충돌**: `docs/00-overview/vision.md` §3.1은 "누군가 본 플랫폼을 **fork·운영**할 때 무엇을 왜 그렇게 만들었는지 의사결정 흐름을 그대로 따라갈 수 있게 한다"를 일차 목표로 둔다. 라이선스 부재는 이 vision을 직접 부정한다.
3. **사후 추가의 모호성**: 첫 커밋 이후 라이선스를 추가해도, 추가 이전 시점의 커밋·이슈·PR에 대한 라이선스 적용 모호성이 영구히 남는다.
4. **R-008 완화**: 리스크 레지스터 R-008(라이선스·법적 이슈)은 본 결정의 필요성을 사전 식별했다.

본 ADR은 첫 공개 커밋 직전에 결정되어야 한다.

## Decision — 결정

본 프로젝트의 라이선스로 **Apache License, Version 2.0**을 채택한다.

- 리포 루트에 표준 [`LICENSE`](../../LICENSE) 파일을 둔다 (`apache.org/licenses/LICENSE-2.0.txt`의 본문 그대로).
- README.md §"라이선스" 섹션을 본 ADR 결과로 갱신한다.
- SPDX 식별자: `Apache-2.0`.
- 저작권 표기는 LICENSE 본문에 채우지 않고, 각 소스 파일 헤더 또는 별도 NOTICE 파일에서 다룬다 (Phase 1 코드 작성 시점에 정리).
- 외부 의존성 추가 시 라이선스 호환 검증은 Phase 1 CI 게이트의 SBOM·라이선스 스캔에서 수행한다 (ROADMAP Phase 1 DevSecOps 항목).

## Alternatives Considered — 대안 비교

| 대안 | 강점 | 약점 | 결정 |
|---|---|---|---|
| **Apache License 2.0** | 특허·상표 조항 명시(특허 grant + 종료 조건), 기업·관공서 친화, IaaS 생태계(Apache CloudStack, AWS SDK 등) 표준 라이선스 하나, OSI 승인, Proxmox(AGPLv3)와 호환 가능 (본 프로젝트가 Proxmox 코드를 포함하지 않고 REST API만 호출하므로 — ADR-0002) | LICENSE 본문이 비교적 길다 (운영상 무영향) | **채택** |
| **MIT License** | 가장 짧고 단순, 가장 널리 사용 | 특허·상표 조항 부재 → IaaS·빌링 영역에서 잠재 분쟁 가능성, 본 프로젝트가 미래에 특허 등록 가능한 무결성 메커니즘(ADR-0006)을 다룰 가능성을 고려하면 보호 약함 | 거부 |
| **AGPL-3.0** | Proxmox와 동일 라이선스, copyleft 강함, SaaS 형태 제공 시 소스 공개 의무로 commons 강화 | vision §3.1의 "fork·운영 가능성"에는 **상용 fork**도 포함될 수 있는데, AGPL은 그 경로를 제한. 종량제 청구·SaaS 형태로 본 플랫폼을 활용하려는 사용자(vision의 일차 사용자)에게 추가 의무 부과 → 진입 장벽 증가 | 거부 |
| **BSD-3-Clause** | MIT 유사 + "no endorsement" 조항, 짧고 단순 | 특허 조항 부재, Apache-2.0 대비 보호 약함 | 거부 |

## Decision Drivers — 핵심 판단 근거

- **IaaS 생태계 정합**: 본 프로젝트가 참조하는 AWS API·Apache CloudStack 등이 Apache-2.0 또는 호환 라이선스. vision의 "준-CSP 환경" 포지셔닝과 표준이 일치.
- **특허·상표 보호**: 빌링 무결성 메커니즘(ADR-0006)·멀티테넌시 격리 모델(ADR-0005) 등 본 프로젝트의 깊이 가치 영역이 특허 가능 영역과 가까움. Apache-2.0의 명시적 특허 grant + 특허 분쟁 시 종료 조항이 양방향 보호 제공.
- **외부 fork 진입 장벽 최소화**: vision §3.1의 "fork·운영 가능성"이 상용 fork를 포함할 수 있으므로 copyleft 부담이 약한 permissive 계열이 적합. AGPL 거부 이유.
- **MIT/BSD 대비 특허 조항 우위**: 단순함의 가치는 인정하나, IaaS 영역의 특허 위험을 우선.

## Consequences — 결과

### 긍정적

- 외부 기여자의 **fork·수정·재배포 권리가 명시**되어 vision §3.1의 일차 목표 달성.
- 특허·상표 분쟁에 대한 양방향 보호.
- IaaS 생태계 표준 라이선스 하나로, 외부 사용자·기여자가 친숙하게 받아들임.
- R-008(라이선스·법적 이슈) 완화.

### 부정적

- LICENSE 본문이 약 200줄로 비교적 김 (운영상 무영향, 표준 파일).
- AGPL의 강한 copyleft 효과(SaaS 형태 사용자도 소스 공개)는 포기 — 본 프로젝트는 commons 강화보다 fork·운영 가능성을 우선.

### 중립적

- 외부 의존성 추가 시 라이선스 호환 검증 의무 발생 (Phase 1 CI 게이트로 자동화).
- 향후 외부 기여자가 합류하면 DCO(Developer Certificate of Origin) 또는 CLA 도입 여부를 별도 검토 (Open Questions 참조).

## Operational Implications — 운영 영향

- **NOTICE 파일·소스 헤더**: Phase 1 코드 작성 시점에 NOTICE 파일과 소스 파일 헤더 정책을 별도 정리. ADR-0011 결정으로 충족되는 기본 의무는 LICENSE 파일 존재 + README 명시 두 가지.
- **CI 게이트**: Phase 1 진입 시 SBOM 생성 + 라이선스 호환성 자동 검증을 CI에 추가 (ROADMAP Phase 1 DevSecOps 항목과 통합).
- **외부 기여 정책**: 기여 시 라이선스 동의 절차(DCO 또는 CLA)는 외부 기여자 첫 발생 시점에 별도 ADR로 결정.

## Open Questions

- DCO(Developer Certificate of Origin) vs CLA(Contributor License Agreement) 도입 시점·형태 → 외부 기여자 첫 합류 시점 또는 Phase 2 IAM ADR과 함께 재검토.
- NOTICE 파일 채택 여부 및 형식 → Phase 1 코드 첫 작성 시점에 별도 결정 (별도 ADR 불필요, 운영 결정).
- 소스 파일 헤더에 SPDX-License-Identifier 의무화 여부 → Phase 1 CI 게이트 설계 시 결정.
