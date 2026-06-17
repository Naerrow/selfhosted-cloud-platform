# C4 Level 1 — System Context (v0)

> C4 모델의 최상위 레벨. 본 플랫폼(System of Interest)을 하나의 박스로 보고,
> **누가(페르소나) 사용하는가**와 **무엇(외부 시스템)에 의존하는가**를 그린다.
> 내부 구조는 [Level 2 — Containers](./c4-level2-containers.md)에서 분해한다.
>
> 본 문서는 **Phase 0 Exit 게이트** 산출물 v0다. Phase별로 갱신한다.

## 전제 (입력 문서)

- 페르소나 정의: [`../00-overview/personas.md`](../00-overview/personas.md) (P-OP/P-CSP/P-TA/P-EU)
- 컨트롤 플레인 ↔ 데이터 플레인 분리: [`../00-overview/glossary.md`](../00-overview/glossary.md) §A
- 데이터 플레인 = Proxmox VE: [ADR-0002](../02-adr/0002-use-proxmox-as-hypervisor.md)
- 자체 오케스트레이션 레이어 (Proxmox 자체 클러스터링 미사용): [ADR-0003](../02-adr/0003-not-using-proxmox-clustering.md)
- PG는 어댑터 + Mock까지만: [`../00-overview/scope.md`](../00-overview/scope.md), [ADR-0008](../02-adr/0008-phase3-billing-scope.md)

## 다이어그램

> ⚠ Mermaid의 C4 다이어그램 문법(`C4Context`)은 **experimental**이다. 렌더러(GitHub 등)가 사용하는 Mermaid 버전에 따라 표시가 다를 수 있다. 의미론은 본문 서술로도 완결되게 작성했다.

```mermaid
C4Context
    title System Context — 셀프호스팅 클라우드 플랫폼 (v0)

    Person(pop, "P-OP 플랫폼 운영자", "컨트롤 플레인 자체를 운영·노드 등록·빌링 무결성 검증")
    Person(pcsp, "P-CSP CSP 사업자", "가격 정책·청구·수금 정책의 최종 결정자")
    Person(pta, "P-TA 테넌트 관리자", "한 테넌트 안의 자원·사용자·예산 책임")
    Person(peu, "P-EU 엔드 유저", "테넌트 안에서 VM/스토리지를 실제로 생성·사용")

    System(platform, "셀프호스팅 클라우드 플랫폼", "Proxmox 위에 얹는 얇은 컨트롤 플레인. 멀티테넌시·미터링·종량제 빌링·감사를 제공한다.")

    System_Ext(proxmox, "Proxmox VE 노드 집합", "데이터 플레인. VM(KVM)/LXC를 실제로 실행. REST API(/api2/json) + API Token으로만 접근.")
    System_Ext(idp, "외부 IdP (OIDC)", "Phase 2 이후. 외부 신원 공급자. 예: Keycloak 셀프호스트.")
    System_Ext(pg, "PG (결제 대행)", "어댑터 인터페이스 + Mock 구현체까지만 (ADR-0008). 운영 PG 직접 연결은 OUT-OF-SCOPE.")

    Rel(pop, platform, "운영·노드 수명주기 관리·감사 로그 조회", "웹 콘솔 / API / CLI")
    Rel(pcsp, platform, "가격 정책 설정·청구서 발행·비용 보고서", "웹 콘솔 / API")
    Rel(pta, platform, "테넌트 자원·사용자·예산 관리", "웹 콘솔 / API / CLI")
    Rel(peu, platform, "VM/스토리지 생성·콘솔 접속·사용량 조회", "웹 콘솔 / API / CLI")

    Rel(platform, proxmox, "VM/CT 수명주기·메트릭 조회", "HTTPS REST + API Token")
    Rel(platform, idp, "인증 위임 (Phase 2+)", "OIDC")
    Rel(platform, pg, "청구서 결제 요청 (Mock)", "어댑터 IF")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## 핵심 신뢰 경계 (요약)

상세는 [`./trust-boundaries.md`](./trust-boundaries.md). Level 1 관점에서 가장 바깥 경계만 표시:

1. **사용자 ↔ 플랫폼**: 모든 페르소나 요청은 인증·인가(RBAC)를 통과해야 한다. 모든 통신 TLS ([`../00-overview/vision.md`](../00-overview/vision.md) §3.2 보안 게이트 원칙).
2. **플랫폼 ↔ Proxmox**: 컨트롤 플레인은 Proxmox를 **단일 어댑터 계층**으로만 호출한다. 비즈니스 로직은 Proxmox 엔드포인트를 직접 부르지 않는다 ([ADR-0002](../02-adr/0002-use-proxmox-as-hypervisor.md), [`./c4-level2-containers.md`](./c4-level2-containers.md) Proxmox 어댑터).
3. **플랫폼 ↔ 외부 서비스 (IdP/PG)**: 외부 신뢰 영역. 경계를 넘는 데이터는 검증된다.

## 페르소나 간 권한 비대칭 (위협 모델 입력)

본 컨텍스트에서 4 페르소나는 동등하지 않다. [`../00-overview/personas.md`](../00-overview/personas.md) §7에 따라,
위협 모델은 **P-EU → P-TA → P-CSP → P-OP 방향의 권한 상승(Elevation of Privilege) 시도**를 항상 가정한다.
특히 **P-CSP(가격 정책)와 P-OP(인프라 운영)는 SoD로 분리**되어 1인 운영 환경에서도 동일 자격증명으로 합치지 않는다.

→ 이 비대칭은 [STRIDE 위협 모델 v0](../03-security/threat-model.md)의 신뢰 경계 입력이 된다.

## 범위 밖 (이 다이어그램에 그리지 않는 것)

- 다수 퍼블릭 클라우드: Cloud Bursting은 AWS 1종(Phase 7)으로 한정 (scope.md). v0에는 미표시.
- 데스크탑/모바일 네이티브 클라이언트: OUT-OF-SCOPE (scope.md). 클라이언트는 웹/API/CLI 3종.

## 변경 이력

- v0 (Phase 0): 최초 작성. 4 페르소나 + Proxmox/IdP/PG 외부 시스템.
- v0 (수정): Mermaid C4 experimental 고지 추가. threat-model 링크에 (예정) 표기.
- v0 (수정 2): `threat-model.md` 작성 완료로 (예정) 표기 해제.
- v0 (수정 3): 본 공개 문서의 내부 전용 지침 인용을 공개 근거(vision §3.2, ADR, C4 L2)로 치환 (ADR-0009 준수).
