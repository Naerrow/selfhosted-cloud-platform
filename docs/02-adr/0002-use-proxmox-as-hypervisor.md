# ADR-0002: 하이퍼바이저로 Proxmox VE를 채택한다

- **상태(Status)**: Accepted
- **결정일(Date)**: 2026-05-25
- **결정자(Decider)**: 프로젝트 메인테이너
- **관련 ADR**: 0003 (Proxmox 자체 클러스터링 미사용)

## Context — 맥락

본 프로젝트의 비전(`docs/00-overview/vision.md`)은 "데이터센터급 서버부터 미니 PC(홈랩)까지 동일한 콘솔로 운영"을 명시한다. 이를 위해 데이터 플레인에서 VM/컨테이너를 실행할 **하이퍼바이저 엔진**을 선택해야 한다.

주요 제약:

1. **저사양 하드웨어 지원**: Intel NUC, N100 미니 PC 등 4코어/16GB 환경에서도 동작해야 한다.
2. **운영 단순성**: 1인 운영자가 다룰 수 있어야 한다.
3. **API 가용성**: 컨트롤 플레인이 외부에서 자원을 조작할 수 있는 REST API가 필요하다.
4. **컨테이너 + VM**: VM(KVM)과 경량 컨테이너(LXC) 양쪽이 필요하다.
5. **오픈소스 + 상용 가능**: 라이선스가 셀프호스팅·코드 공개를 모두 허용해야 한다.

## Decision — 결정

데이터 플레인 하이퍼바이저로 **Proxmox VE**를 채택한다.

- 본 프로젝트의 **컨트롤 플레인**은 Proxmox의 **REST API(`/api2/json/...`)** 만을 외부 인터페이스로 사용한다. Proxmox 내부 구조에 깊이 결합하지 않는다.
- 인증은 **API Token** 방식(`PVEAPIToken=USER@REALM!TOKENID=UUID`)을 표준으로 한다. 자세한 사유는 추후 시크릿 관리 ADR(0007)에서 다룬다.
- VM(KVM)과 LXC 양쪽을 1급 시민으로 다룬다.

## Alternatives Considered — 대안 비교

| 대안 | 강점 | 약점 | 결정 |
|---|---|---|---|
| **Proxmox VE** | KVM+LXC 통합, 데비안 기반 운영 단순성, 미니 PC에서 동작, REST API + API Token 인증 지원, 활발한 커뮤니티 | 자체 클러스터링은 단일 LAN·저지연 전제(멀티사이트·광역 묶음 부적합), AGPLv3 (서브스크립션은 별도) | **채택** |
| **OpenStack** | 풀스택 IaaS, 강력한 멀티 테넌시, CloudKitty 등 풍부한 생태계 | 분산 마이크로서비스 구조로 설치·운영 매우 복잡, 미니 PC급 단일 노드 환경에 부적합, 본 프로젝트는 OpenStack을 다시 만드는 것이 목표가 아님 | 거부 |
| **Apache CloudStack** | 통합형 아키텍처, 통신사·ISP 환경에서 검증, Usage Server 내장 | Java + MySQL 단일 관리 서버 구조, 대규모 환경 가정, 미니 PC 단독 시작에 부적합 | 거부 |
| **베어 KVM/QEMU + libvirt** | 가장 가벼움, 직접 제어 가능 | 운영 UX 부재, 백업·HA·스토리지 추상화를 모두 자체 구현해야 함, 개발 공수 폭증 | 거부 |
| **oVirt** | RHV 계열, 엔터프라이즈 기능 | 미니 PC급 환경 부적합, Red Hat이 RHV 단종 후 커뮤니티 동력 약화 | 거부 |
| **VMware ESXi** | 안정성·생태계 | 라이선스·정책 불확실성(Broadcom 인수 이후), 셀프호스팅·코드 공개 모델에 부적합 | 거부 |
| **XCP-ng / XenServer** | 오픈소스 Xen 기반, 안정적 | 커뮤니티 규모가 Proxmox 대비 작고, LXC 통합이 약함 | 거부 |

## Decision Drivers — 핵심 판단 근거

> 인용 출처: `docs/06-research/cloud-platforms-and-regulations.md` (사전 조사 자료, 작성일 2026-05-25).

- Proxmox VE는 데비안 기반으로 미니 PC 환경에서 검증된 동작 사례가 많고, KVM과 LXC를 단일 플랫폼에서 다룬다.
- REST API가 잘 정의되어 있고(`https://<host>:8006/api2/json/...`), **API Token** 방식이 자동화·외부 통합에 적합하다 (CSRF 토큰 불필요, 토큰별 만료·권한 분리 가능).
- 사전 조사상 OpenStack/CloudStack 대비 운영 난이도가 가장 낮다.
- 본 프로젝트의 가치는 **Proxmox 위에 얹는 컨트롤 플레인의 설계**에 있으며, 하이퍼바이저 자체를 재발명하지 않는다.

## Consequences — 결과

### 긍정적
- 단일 미니 PC에서도 시작 가능 → 프로젝트의 비전과 정렬.
- REST API + API Token이라는 깔끔한 추상화 경계가 확보된다.
- VM(KVM) + 컨테이너(LXC) 양쪽을 동일 인터페이스로 다룬다.
- 커뮤니티가 활발해 학습 자료가 풍부하다.

### 부정적
- Proxmox의 자체 클러스터링(Corosync + pmxcfs)은 단일 데이터센터·저지연 LAN(노드 간 지연 5ms 미만)·quorum 전제를 가지며, 본격적 멀티테넌시도 제공하지 않는다. 본 플랫폼은 이 제약을 그대로 떠안게 된다. → **ADR-0003에서 해결**: 본 플랫폼은 Proxmox 자체 클러스터링을 사용하지 않고 자체 오케스트레이션 레이어를 만든다.
- AGPLv3 라이선스의 의미를 항상 인지해야 한다(서브스크립션 가입은 선택, 본 프로젝트는 무가입으로 시작).
- Proxmox API 변경에 대한 추적 의무가 생긴다.

### 중립적
- 본 프로젝트의 컨트롤 플레인은 추상화를 잘 두면, 미래에 다른 하이퍼바이저(libvirt 직접, ESXi 등) 어댑터를 추가하는 길은 열려 있다. 다만 Phase 7 이후 검토.

## Open Questions

- Proxmox Backup Server(PBS) 통합 시점 → Phase 1 또는 Phase 4에서 별도 ADR.
- Proxmox SDN 기능(VLAN/VXLAN/EVPN)을 어디까지 사용할지 → Phase 5 네트워킹 ADR에서 결정.
