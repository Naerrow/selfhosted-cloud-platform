# Glossary

> 본 문서는 프로젝트 내에서 사용하는 용어의 **단일 진실 공급원(Single Source of Truth)** 이다.
> 새 용어가 등장하면 즉시 이곳에 등록하고, 다른 문서는 이 용어집을 따른다.

## A. 도메인 — 플랫폼 추상화

| 용어 | 정의 |
|---|---|
| **컨트롤 플레인 (Control Plane)** | 본 플랫폼의 API/스케줄러/대시보드 계층. 실제 워크로드는 실행하지 않고 노드들을 지휘한다. |
| **데이터 플레인 (Data Plane)** | 실제 VM/컨테이너가 실행되는 Proxmox 노드 집합. |
| **노드 (Node)** | 본 플랫폼이 관리하는 개별 Proxmox 서버. 서버급일 수도, 미니 PC일 수도 있다. |
| **풀 (Pool)** | 여러 노드를 묶어 단일 자원 풀로 추상화한 단위. 스케줄러가 풀 단위로 배치 결정을 내린다. |
| **하이퍼바이저 (Hypervisor)** | 본 프로젝트에서는 항상 Proxmox VE (KVM/LXC) 를 의미한다. |
| **테넌트 (Tenant)** | 자원·사용자·예산이 격리된 고객 단위. OpenStack의 Project, AWS의 Account에 대응한다. |
| **도메인 (Domain)** | 테넌트들을 묶는 상위 단위. 보통 단일 조직·법인에 해당한다. |
| **인스턴스 (Instance)** | 실행 중인 VM 또는 LXC 컨테이너. |
| **인스턴스 타입 (Instance Type)** | vCPU·메모리·기본 스토리지 조합을 정의한 카탈로그 항목. 단가의 기본 단위. |

## B. IAM — 인증·인가

| 용어 | 정의 |
|---|---|
| **사용자 (User)** | 사람 또는 자동화 주체에 부여되는 자격증명 주체. |
| **역할 (Role)** | 권한의 묶음. 사용자에게 부여한다. |
| **정책 (Policy)** | 어떤 자원에 대해 어떤 행위를 허용/거부하는지 기술한 규칙. |
| **API 키 (API Key) / 토큰 (Token)** | 프로그래밍 방식 접근을 위한 자격증명. 사용자보다 짧은 수명·좁은 권한을 갖는다. |
| **RBAC** | Role-Based Access Control. 본 플랫폼의 기본 인가 모델. |
| **SoD (Segregation of Duties)** | 권한 분리. 단가 수정과 빌링 발행 등 충돌 가능한 권한을 동일인이 모두 가질 수 없게 하는 원칙. |
| **2-eye 승인** | 한 사람의 요청 + 다른 사람의 승인이 모두 있어야 실행되는 흐름. |
| **OIDC** | OpenID Connect. 외부 IdP 연동 시 사용하는 표준 프로토콜. |

## C. 빌링

| 용어 | 정의 |
|---|---|
| **미터링 (Metering)** | 자원 사용량의 원천 측정·기록. 본 시스템에서는 append-only 로만 다룬다. |
| **레이팅 (Rating)** | 미터링 데이터에 단가를 곱해 청구 가능 금액으로 변환하는 단계. |
| **빌링 (Billing)** | 레이팅 결과를 기간 단위로 묶어 청구서를 생성하는 단계. |
| **종량제 (Pay-as-you-go)** | 실제 사용량 비례 과금. 본 프로젝트의 **유일한** 빌링 모델. |
| **가격 정책 (Pricing Policy)** | 인스턴스 타입·스토리지·트래픽 단가 정의서. 변경은 감사 대상. |
| **청구서 (Invoice)** | 한 테넌트의 한 기간(보통 월) 사용량과 금액을 집계한 문서. PDF로 발행. |
| **무결성 보장 (Integrity)** | 청구서·미터링 데이터가 사후 변조되지 않았음을 검증할 수 있게 하는 메커니즘 (해시 체인/머클 트리). |
| **Idempotency** | 같은 입력으로 같은 청구서가 재생산되는 성질. 빌링 재실행이 안전하게 가능해야 한다. |
| **PG (Payment Gateway)** | 외부 결제 대행 서비스. 본 프로젝트에서는 어댑터 인터페이스 + Mock 까지만 구현. |

## D. 운영·보안

| 용어 | 정의 |
|---|---|
| **ADR (Architecture Decision Record)** | 설계 의사결정 1건을 기록한 짧은 MD 문서. 본 프로젝트의 핵심 산출물. |
| **SLO (Service Level Objective)** | 우리가 스스로에게 부여하는 서비스 수준 목표. |
| **SLI (Service Level Indicator)** | SLO를 측정하기 위한 정량 지표. |
| **런북 (Runbook)** | 특정 사고·작업에 대한 단계별 대응 절차서. |
| **STRIDE** | 위협 모델링 분류 체계: Spoofing / Tampering / Repudiation / Information Disclosure / Denial of Service / Elevation of Privilege. |
| **신뢰 경계 (Trust Boundary)** | 신뢰 수준이 다른 두 영역의 경계. 데이터가 이 경계를 넘을 때 검증·인가가 필요. |
| **SBOM (Software Bill of Materials)** | 소프트웨어 구성 요소 목록. CI에서 자동 생성·서명한다. |
| **감사 로그 (Audit Log)** | 누가/언제/무엇을/어디서 했는지 기록하는 변조 방지 로그. |
| **append-only** | 추가만 가능하고 수정·삭제가 불가능한 데이터 구조. 정정은 보정 레코드로만. |

## E. 한국 규제 (사전 조사 자료 기반, 작성일 2026-05-25 기준)

| 용어 | 정의 |
|---|---|
| **CSAP** | 클라우드 보안 인증제. 공공기관 대상 클라우드 사업의 사실상 필수 인증. 상/중/하 등급. |
| **개인정보보호법 (PIPA)** | 개인정보 처리·이전·안전성 확보 조치를 규정. 국외 이전 5가지 경로 규정. |
| **전기통신사업법** | 부가통신사업 신고 의무의 근거 법령. 신고 면제 대상은 "인터넷을 이용하여 부가통신역무를 제공하는 자본금 1억 원 이하"인 소규모 부가통신사업자이며, 자본금 1억 원 초과 시 신고 의무가 발생한다 (사전 조사 자료 [§6.4](../06-research/cloud-platforms-and-regulations.md) 참조). |
| **부가통신사업** | 기간통신 회선을 이용해 정보 제공·매개 서비스를 제공하는 사업. 클라우드·호스팅 등. |
| **데이터 주권 (Data Sovereignty)** | 데이터가 어느 관할 법령의 적용을 받는가에 관한 원칙. 국외 이전 제한의 근거. |

> 한국 규제 항목은 작성일 기준이며, **실제 사업 적용 시 변호사·관할 기관과 별도 검토**가 필요하다.
> 본 프로젝트의 컴플라이언스 매핑은 `../03-security/compliance-mapping/` 디렉터리를 참고.

## F. 페르소나 코드

[`personas.md`](./personas.md) 참조. 짧은 인용:

| 코드 | 명칭 |
|---|---|
| P-OP | 플랫폼 운영자 |
| P-CSP | CSP 사업자 |
| P-TA | 테넌트 관리자 |
| P-EU | 엔드 유저 |

## G. 약어 모음

| 약어 | 풀이 |
|---|---|
| ADR | Architecture Decision Record |
| API | Application Programming Interface |
| CI/CD | Continuous Integration / Continuous Delivery |
| CSP | Cloud Service Provider |
| DR | Disaster Recovery |
| IaaS | Infrastructure as a Service |
| IaC | Infrastructure as Code |
| IdP | Identity Provider |
| JWT | JSON Web Token |
| LXC | Linux Containers (Proxmox의 컨테이너 형식) |
| MFA | Multi-Factor Authentication |
| PG | Payment Gateway |
| RBAC | Role-Based Access Control |
| SBOM | Software Bill of Materials |
| SLO/SLI | Service Level Objective / Indicator |
| SoD | Segregation of Duties |
| SSO | Single Sign-On |
| STRIDE | Spoofing/Tampering/Repudiation/Info Disclosure/DoS/Elevation |
| VPC | Virtual Private Cloud |
| WAF | Web Application Firewall |

> 신규 약어는 본 문서의 알파벳 순서를 유지하며 추가한다.
