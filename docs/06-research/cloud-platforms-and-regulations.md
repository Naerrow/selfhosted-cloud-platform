# 클라우드 플랫폼 및 법적 규제 조사 자료

> 작성일: 2026-05-25 · 개정: 2026-05-25 (1차 출처 인용 추가, Proxmox 노드 한계 수정; § 7.1 요약 표 자체 모순 정정) · 2026-06-02 ([L11] 전기통신사업법 URL 오류 수정, [L12] 전기통신사업법 시행령 출처를 시행본 고정 URL로 갱신; ADR-0010 분류 2) · 2026-06-03 ([L11] 출처 설명을 국가법령정보센터 공식 표기[전기통신사업법 시행 2026.5.19., 법률 제21652호, 일부개정]에 맞추고, 제22조 조문정보 중 [시행 2026.11.20.] 미래 시행본에서 소규모 부가통신사업 신고 간주 조항이 제5항 체계로 이동함을 부기. §6.4 본문 제22조 제4항 인용은 현재 시행 기준으로 유효이며, 2026-11-20 시행일 도래 시 재확인 트리거 본문 명시. 더불어 §6 머리말에 적용 범위 단락 추가 — 본 §6 법규 항목은 ADR-0008에 따라 Phase 3까지 PG Mock 구현체만 제공되는 현재 단계에서는 직접 준수 요건이 아니며, 사업화 전환 시 ADR-0008 회귀 트리거를 따라 활용됨을 명시. §6 출처 표기 규칙·§6.5 출처 제목·§8.1 1차 출처 안내 3곳의 확인일 메타데이터를 [L11] 2026-06-03·[L12] 2026-06-02로 분리 갱신; ADR-0010 분류 2)
> 대상: OpenStack / CloudStack / Proxmox VE / AWS / Cloud Bursting / 국내 법적 규제
>
> **인용 정책**: § 3 Proxmox VE API, § 4 AWS, § 6 법적 규제의 모든 사실 주장은 공식 1차 출처를 문단 단위로 인용한다 (각 절의 출처 목록 참고). § 1 OpenStack, § 2 CloudStack, § 5 Cloud Bursting 은 일반 개념 정리이며 § 8.2 보조 자료를 참고할 것.

---

## 1. OpenStack

### 1.1 개요
OpenStack은 IaaS(Infrastructure as a Service)를 구축하기 위한 오픈소스 클라우드 플랫폼이다. 컴퓨팅·네트워크·스토리지 자원을 가상화·풀링하여 API와 대시보드를 통해 제공하며, 여러 모듈식 컴포넌트가 메시지 큐(RabbitMQ)와 공용 DB를 매개로 협력하는 구조를 가진다.

### 1.2 핵심 컴포넌트 역할

| 컴포넌트 | 코드명 | 역할 |
|---|---|---|
| **Nova** | Compute | VM 인스턴스의 생명주기 관리(생성·기동·정지·삭제). 하이퍼바이저(KVM, QEMU, Xen, VMware 등) 위에서 실행되며, nova-api / nova-scheduler / nova-conductor / nova-compute 로 구성 |
| **Neutron** | Networking | SDN 기반 가상 네트워크 제공. 테넌트별 가상 라우터·서브넷·플로팅 IP·시큐리티 그룹·LBaaS·FWaaS·VPNaaS 지원. ML2 플러그인으로 OVS, Linux Bridge, OVN 등을 백엔드로 사용 |
| **Cinder** | Block Storage | VM에 부착하는 영구 블록 스토리지 제공. LVM, Ceph RBD, NetApp, EMC 등 다양한 백엔드 지원. 스냅샷·볼륨 복제·QoS 제어 |
| **Keystone** | Identity | 인증·인가 중앙 서비스. 사용자/프로젝트(테넌트)/역할(Role)/서비스 카탈로그/엔드포인트 관리. Fernet 토큰 기반, LDAP·SAML·OIDC 연동 가능 |
| **Glance** | Image | VM 이미지(QCOW2, RAW, VMDK 등) 저장·검색·메타데이터 관리. 백엔드는 파일시스템, Swift, Ceph 등 사용 가능 |
| **Horizon** | Dashboard | Django 기반 웹 UI. 사용자·관리자 모두 사용하는 통합 콘솔로, 각 컴포넌트의 API를 호출하여 자원 운용 가능 |
| **Heat** | Orchestration | HOT(Heat Orchestration Template, YAML) 또는 AWS CloudFormation 호환 템플릿으로 다중 자원을 선언적으로 배포·관리. 스택 단위 라이프사이클 관리 |
| **CloudKitty** | Rating / Billing | 자원 사용량 수집(Ceilometer/Gnocchi 연동) → 요율 적용 → 과금 데이터 산출. 해시맵(HashMap), PyScript 등 다양한 레이팅 모듈 지원 |

### 1.3 IaaS 구조
- **컨트롤 노드(Control Node)**: Keystone, Glance, Horizon, Nova API, Neutron Server 등 API/스케줄러 계층 배치
- **컴퓨트 노드(Compute Node)**: Nova-compute + 하이퍼바이저 + Neutron L2 에이전트가 동작하며 실제 VM 실행
- **네트워크 노드(Network Node)**: L3 라우팅, DHCP, 메타데이터 서비스 담당 (OVN 사용 시 분산화 가능)
- **스토리지 노드(Storage Node)**: Cinder-volume, Swift, Ceph 등 운영
- **메시지 큐**: RabbitMQ가 컴포넌트 간 비동기 RPC를 매개
- **DB**: MariaDB / Galera Cluster가 메타데이터 영속화

### 1.4 멀티 테넌시(Multi-tenancy)
- **프로젝트(Project, 구 Tenant)** 단위로 자원·할당량·네트워크·접근권한이 격리됨
- **Domain → Project → User → Role** 4계층 RBAC 구조 (Keystone v3)
- 네트워크 격리는 VLAN, VXLAN, GENEVE 등의 테넌트 네트워크 ID로 구현
- 스토리지 격리: Cinder는 프로젝트별 볼륨·할당량 분리, Swift는 어카운트 단위 격리
- 리소스 쿼터(Quota)로 vCPU, RAM, 인스턴스 수, 볼륨 용량 등 제한 가능

### 1.5 빌링(Billing)
OpenStack은 자체 결제 시스템을 내장하지 않고 **사용량 수집 + 외부 과금 엔진** 조합으로 구현한다.

| 단계 | 컴포넌트 | 역할 |
|---|---|---|
| 사용량 수집 | Ceilometer | VM, 네트워크, 볼륨의 미터링 데이터 수집 |
| 시계열 저장 | Gnocchi | 수집된 메트릭의 시계열 DB 저장 |
| 이벤트 처리 | Panko (Deprecated) → Ceilometer events | 자원 생성/삭제 이벤트 처리 |
| 요율 산정 | **CloudKitty** | 사용량 × 요율 = 청구액 계산, 보고서 출력 |
| 결제 연동 | 외부 ERP / PG | 실제 결제·청구서 발행은 외부 시스템에 위임 |

---

## 2. CloudStack

### 2.1 개요
Apache CloudStack은 Citrix가 기증한 IaaS 플랫폼으로, "단일 패키지 + 단일 DB"의 통합형 아키텍처가 특징이다. OpenStack보다 설치·운영이 간단하고 ISP·통신사 환경의 멀티 테넌트 퍼블릭 클라우드 구축에 많이 사용된다.

### 2.2 IaaS 구조 (계층형 인프라 모델)
CloudStack은 자원 계층을 4단계로 명확하게 추상화한다.

```
Region (지역)
 └─ Zone (데이터센터 단위 가용 영역)
     └─ Pod (랙 단위, 동일 L2 네트워크)
         └─ Cluster (동일 하이퍼바이저 풀)
             └─ Host (물리 서버)
```

- **Management Server**: Java 기반 단일 관리 서버(또는 HA 클러스터). API, UI, 오케스트레이션, 모니터링 모두 담당
- **MySQL DB**: 모든 메타데이터 단일 저장
- **System VM**: SSVM(Secondary Storage VM), CPVM(Console Proxy VM), Virtual Router로 네트워크·콘솔·스토리지 게이트웨이 제공
- **하이퍼바이저 지원**: KVM, XenServer/XCP-ng, VMware vSphere, Hyper-V, LXC
- **네트워크 모드**: Basic(공유 L2) / Advanced(VLAN, VXLAN 기반 격리 + Virtual Router로 NAT·LB·VPN)

### 2.3 빌링
- **Usage Server** 컴포넌트가 모든 자원의 사용량을 주기적(기본 1시간)으로 수집해 `cloud_usage` DB에 저장
- VM 실행 시간, 네트워크 송수신량, 스토리지 점유량, 공인 IP 임대 시간, 부하분산기 사용 시간 등이 표준 메트릭
- `listUsageRecords` API로 외부 빌링 시스템이 데이터 인출 → 통신사 BSS/OSS, ERP와 연동하여 청구서 발행
- CloudKitty 같은 별도 레이팅 엔진이 필요 없는 만큼 OpenStack 대비 통합도가 높음

### 2.4 Proxmox VE 대비 차이

| 항목 | CloudStack | Proxmox VE |
|---|---|---|
| 포지셔닝 | 멀티 테넌트 퍼블릭/대규모 프라이빗 IaaS | 중소 규모 VM/컨테이너 가상화 플랫폼 |
| 멀티 테넌시 | Domain/Account/Project 계층 RBAC + 자원 격리 (네이티브 지원) | 풀(Pool)·권한 단위로 분리하지만 본격적 테넌트 격리 기능 약함 |
| 네트워크 | Advanced Zone에서 VLAN/VXLAN, Virtual Router로 NAT/LB/VPN 자동화 | Linux Bridge/OVS 직접 구성, SDN 기능은 추가 작업 필요 |
| 스토리지 추상화 | Primary/Secondary Storage 추상화, 다양한 SAN/NAS 지원 | ZFS, Ceph, LVM 등 직접 연결 |
| 빌링 | Usage Server 내장, 청구용 사용량 데이터 표준 제공 | 내장 빌링/유저빌 사용량 미터링 없음 |
| API | REST/Query API, 사용자별 API Key + Secret 서명 인증 | REST API + API Token (PVE Cluster 단위) |
| 확장성 | 수천~수만 호스트, Zone 단위 수평 확장 | 공식적으로 명시적 노드 한계 없음(50+ 노드 사례 보고됨), 단일 데이터센터·저지연 LAN 환경 권장, 대규모 멀티사이트는 부적합 |
| 운영 난이도 | 컴포넌트 많고 SDN/스토리지 설계 필요 | 데비안 기반, 설치·운영 매우 간단 |

---

## 3. Proxmox VE API

> 출처 표기 규칙: 본 절의 모든 기술적 사실은 Proxmox 공식 위키 및 공식 매뉴얼을 1차 출처로 한다. 각 문단 끝의 `[P#]` 마커는 § 3.6 출처 목록의 항목 번호이며, 모든 URL은 **확인일 2026-05-25** 기준으로 접근·교차 확인했다.

### 3.1 REST API 전체 구조
Proxmox VE는 REST 스타일의 API를 사용하며, HTTPS 프로토콜로 동작하고 서버는 **포트 8006**에서 수신한다. 기본 URL은 `https://your.server:8006/api2/json/` 이며, 반환 포맷은 `json`(기본), `extjs`, `html`, `text` 중 선택할 수 있다. 데이터 포맷으로 JSON Schema를 채택하여 API 전체가 형식적으로 정의되어 있다. 파라미터는 URL이나 `x-www-form-urlencoded` 본문으로 전달한다. [P1]

주요 리소스 경로(루트 서브디렉터리)는 다음과 같이 공식 위키에서 인용된다: `version`, `cluster`, `nodes`, `storage`, `access`, `pools`. 각 노드·VM·컨테이너·스토리지·풀의 세부 엔드포인트 트리는 공식 API Viewer에서 확인할 수 있다. [P1][P2]

| 경로 | 내용 |
|---|---|
| `/access` | 인증, 사용자, 그룹, 역할(Role), ACL, 도메인, API Token |
| `/cluster` | 클러스터 전역 설정, 백업 스케줄, HA 그룹, 방화벽, SDN, 복제 |
| `/nodes/{node}` | 노드별 자원 (CPU/RAM/디스크), 서비스, 작업, 로그 |
| `/nodes/{node}/qemu/{vmid}` | KVM VM의 생성·기동·정지·스냅샷·복제·마이그레이션·콘솔 |
| `/nodes/{node}/lxc/{vmid}` | LXC 컨테이너 관리 |
| `/nodes/{node}/storage/{store}` | 스토리지별 ISO/디스크 이미지/백업 파일 |
| `/storage` | 스토리지 정의(NFS, Ceph, ZFS, LVM, iSCSI 등) |
| `/pools` | 자원 풀(다중 VM/스토리지 묶음, RBAC 단위) |

### 3.2 API 안정성 정책
공식 위키는 "메이저 릴리스 내에서 API 호환성 유지를 시도한다(예: 6.0의 호출은 6.4에서도 동작하지만 7.0에서는 보장하지 않음)" 라고 명시한다. 엔드포인트 제거·이동, 기존 파라미터 제거, 반환 타입의 비호환 변경은 breaking change로 분류되고, 새 파라미터 추가, 반환 객체에의 새 필드 추가, 새 엔드포인트 추가 등은 breaking change가 아니다. [P1]

### 3.3 인증
공식 문서에 따르면 Proxmox VE는 **ticket 또는 token 기반 인증**을 사용하며, 모든 API 요청은 쿠키 헤더에 ticket을 담거나 Authorization 헤더에 API token을 담아 전송해야 한다. [P1]

**1) Ticket Cookie (웹 UI 방식)**
- `POST /access/ticket`에 username·password를 보내면 `PVEAuthCookie` 와 `CSRFPreventionToken` 이 반환된다. 이 ticket은 클러스터 전역 인증 키로 서명된 무작위 텍스트 값에 사용자명·생성 시각이 포함된 형태이며, 키는 하루에 한 번 회전된다. [P1]
- 모든 쓰기 요청(POST/PUT/DELETE)에는 HTTP 헤더에 `CSRFPreventionToken` 을 함께 보내야 한다. [P1]
- **Ticket 유효 기간은 2시간**이며, 만료 전에 기존 ticket을 password 자리에 넣어 `/access/ticket`을 다시 호출하면 갱신 가능하다. [P1]

**2) API Token (자동화·외부 통합용)**
- 공식 문서 정의: "API tokens allow stateless access to most parts of the REST API by another system, software or API client. Tokens can be generated for individual users and can be given separate permissions and expiration dates" — 사용자별로 발급 가능하고, 별도 권한과 만료일 부여가 가능하며, 유출 시 사용자 자체를 비활성화하지 않고 token만 폐기 가능. [P1]
- 헤더 형식: `Authorization: PVEAPIToken=USER@REALM!TOKENID=UUID` (공식 위키 표기를 그대로 따름). [P1]
- 공식 문서가 특히 명시하는 차이: "API tokens do not need CSRF values for POST, PUT or DELETE. Tokens are normally not used in a browser context, so the main attack vector of CSRF is not applicable in the first place." — 즉 CSRF 헤더가 필요 없다. [P1]
- 추가 옵션은 `pveum` 문서를 통해 확인되며, 별도 권한 분리(privilege separation), 만료일 설정 등이 지원된다. [P3]

### 3.4 기능 범위
공식 매뉴얼에 따르면 클러스터 기능은 다음을 포함한다: 중앙집중식 웹 관리, 멀티 마스터 클러스터(모든 노드에서 관리 작업 수행 가능), pmxcfs 기반 설정 파일 실시간 복제, VM/컨테이너의 노드 간 이주(migration), 신속 배포, 방화벽 및 HA 같은 클러스터 전역 서비스. [P2]

### 3.5 클러스터링 동작 원리와 한계
- **Corosync 사용**: Proxmox VE는 안정적 그룹 통신을 위해 **Corosync Cluster Engine**을 사용한다. [P2]
- **pmxcfs**: "Proxmox Cluster File System"으로, 클러스터 설정을 모든 노드에 투명하게 분산한다(공식 매뉴얼 본문 그대로). [P2]
- **Quorum**: "Proxmox VE는 모든 클러스터 노드 간 일관된 상태를 보장하기 위해 quorum 기반 기법을 사용한다. 네트워크 분할 시 상태 변경에는 과반수 노드가 온라인이어야 하고, **quorum을 잃으면 클러스터는 읽기 전용 모드로 전환**된다. 기본적으로 노드 1개당 1표가 부여된다." [P2]
- **노드 수 한계 ─ 수정 사항**: 일반적으로 회자되는 "32노드 한계"는 **공식 입장이 아니다**. 공식 매뉴얼은 다음과 같이 명시한다: *"There's no explicit limit for the number of nodes in a cluster. In practice, the actual possible node count may be limited by the host and network performance. Currently (2021), there are reports of clusters (using high-end enterprise hardware) with over 50 nodes in production."* [P2]
- **네트워크 요구사항**: "all nodes via UDP ports 5405-5412 for corosync", 시간 동기화, SSH(TCP/22) 터널이 필요. HA를 위해서는 최소 3노드 권장, 2노드 클러스터는 QDevice로 3번째 표를 보조 가능. 권장 NIC는 corosync 전용 1Gbit. [P2]
- **네트워크 지연**: "The Proxmox VE cluster stack requires a reliable network with latencies under 5 milliseconds (LAN performance) between all nodes to operate stably. While on setups with a small node count a network with higher latencies may work, this is not guaranteed and gets rather unlikely with more than three nodes and latencies above around 10 ms." — 즉 광역(WAN) 묶음은 부적합. [P2]
- **HA 동작과 펜싱**: HA 리소스가 있는 노드는 quorum이 끊긴 상태가 약 1분 지속되면 스스로 fence(자가 차단)된다. [P2]
- **본격적 멀티 테넌시 부족**: 풀(Pool) + 권한 모델로 구분은 가능하지만, OpenStack/CloudStack과 같은 테넌트 단위 네트워크·스토리지 격리 기능은 제공되지 않는다. (공식 문서에는 "multi-tenancy"라는 표현 자체가 사용되지 않으며, 권한 모델은 사용자/그룹/풀에 한정됨 — [P3] 참조)
- **글로벌 스케줄러 미제공**: VM 마이그레이션은 사용자 또는 HA 그룹이 트리거하며, OpenStack Nova-scheduler 같은 자동 배치 스케줄러는 제공되지 않는다. (공식 매뉴얼은 "qm migrate" 명령과 HA 매니저만 다룸 — [P2] 참조)

### 3.6 출처 (확인일 2026-05-25)
- **[P1]** Proxmox 공식 위키, *Proxmox VE API* (마지막 편집일 2026-02-04). <https://pve.proxmox.com/wiki/Proxmox_VE_API>
- **[P2]** Proxmox 공식 매뉴얼, *Cluster Manager* (버전 9.1.2, 2025-12-12 갱신). <https://pve.proxmox.com/pve-docs/chapter-pvecm.html>
- **[P3]** Proxmox 공식 매뉴얼, *User Management* (pveum). <https://pve.proxmox.com/pve-docs/chapter-pveum.html> (특히 §"API Tokens" 절: <https://pve.proxmox.com/pve-docs/chapter-pveum.html#pveum_tokens>)
- **[P4]** Proxmox 공식 API Viewer (HTML 기반 엔드포인트 브라우저). <https://pve.proxmox.com/pve-docs/api-viewer/index.html>

---

## 4. AWS 서비스 구조

> 출처 표기 규칙: 본 절의 API/기능 주장은 AWS 공식 문서(`docs.aws.amazon.com`)를 1차 출처로 한다. 각 항목 끝의 `[A#]` 마커는 § 4.9 출처 목록의 항목 번호이며, **확인일은 모두 2026-05-25**이다.

### 4.1 EC2 (Elastic Compute Cloud)
가상 머신(인스턴스)을 제공한다. 인스턴스 생성은 AMI(이미지) + 인스턴스 타입 + 키 페어 + 시큐리티 그룹 + 서브넷의 조합으로 이루어지며, 공식 *EC2 API Reference* 에서 `RunInstances` 가 인스턴스 시작 API로 정의되어 있다. [A1]

- **요소**: AMI, 인스턴스 타입, 키 페어, 보안 그룹(스테이트풀 방화벽), 사용자 데이터(부트 스크립트), 인스턴스 메타데이터(IMDSv2 권장), 배치 그룹, ENI
- **요금 모델**: On-Demand, Reserved Instance, Savings Plans, Spot, Dedicated Host
- **주요 API**: `RunInstances`(인스턴스 시작), `TerminateInstances`(종료), `DescribeInstances`(조회), `StartInstances`/`StopInstances`, `CreateImage`, `ModifyInstanceAttribute` — 모두 *Amazon EC2 API Reference* 의 정식 액션. [A1][A2]

### 4.2 VPC (Virtual Private Cloud)
사용자 정의 격리 가상 네트워크. CIDR 블록 지정, 가용 영역별 서브넷 구성, IGW/NAT 게이트웨이/엔드포인트 등 *Amazon VPC User Guide* 에 정의된 구성 요소를 사용한다. [A3]

- **요소**: 라우팅 테이블, 인터넷 게이트웨이(IGW), NAT 게이트웨이, VPC 엔드포인트(Gateway/Interface), 시큐리티 그룹, 네트워크 ACL, VPC 피어링, Transit Gateway, Site-to-Site VPN, Direct Connect
- **주요 API**: `CreateVpc`, `CreateSubnet`, `CreateRouteTable`, `AttachInternetGateway`, `CreateNatGateway`, `CreateVpcEndpoint` — *Amazon EC2/VPC API Reference* 의 정식 액션. [A3]

### 4.3 EBS (Elastic Block Store)
EC2에 부착되는 영구 블록 스토리지. AZ에 종속되며, *Amazon EBS User Guide* 가 볼륨 타입과 API를 정의한다. [A4]

- **타입**: gp3/gp2(범용 SSD), io2/io1(고성능 SSD), st1(처리량 최적 HDD), sc1(저비용 HDD)
- **주요 API**: `CreateVolume`, `AttachVolume`, `DetachVolume`, `CreateSnapshot`, `ModifyVolume` — *Amazon EC2 API Reference* 에 정의. [A4]

### 4.4 S3 (Simple Storage Service)
객체 스토리지. Bucket → Object 구조이며, 스토리지 클래스·버저닝·라이프사이클·암호화 등은 *Amazon S3 User Guide* 가 1차 출처. [A5]

- **스토리지 클래스**: Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier Instant/Flexible/Deep Archive
- **주요 API**: `PutObject`, `GetObject`, `ListObjectsV2`, `CopyObject`, `DeleteObject`, `CreateMultipartUpload`, `PutBucketPolicy` — *Amazon S3 API Reference* 의 정식 액션. [A5]

### 4.5 IAM (Identity and Access Management)
- **요소**: User, Group, Role, Policy(JSON), Identity Provider(SAML/OIDC), Permission Boundary, SCP(Organizations) — *IAM User Guide* 에 정의. [A6]
- **정책 평가 로직 (공식 인용)**: AWS 공식 문서에 따르면 "IAM policy evaluation uses an explicit deny, which means that if a single permissions policy includes a denied action, IAM denies the entire request and stops evaluating. Because requests are denied by default, the applicable permissions policies must allow every part of your request for IAM to authorize your request." — 즉 (1) 기본은 묵시적 deny, (2) 명시적 Allow 가 있어야 허용, (3) 명시적 Deny 는 모든 Allow 를 override. [A7]
- **모범 사례**: 루트 계정 사용 금지·MFA, 최소 권한 원칙, 장기 액세스 키 대신 IAM Role + STS, IAM Identity Center 도입 — *IAM Best Practices* 문서 참고. [A6]
- **주요 API**: `CreateUser`, `CreateRole`, `AttachRolePolicy`, `AssumeRole`(STS), `GetCallerIdentity`. [A6]

### 4.6 CloudWatch
- **Metrics 해상도 (공식 인용)**: "Metrics produced by AWS services are standard resolution by default. When you publish a custom metric, you can define it as either standard resolution or high resolution. When you publish a high-resolution metric, CloudWatch stores it with a resolution of 1 second, and you can read and retrieve it with a period of 1 second, 5 seconds, 10 seconds, 30 seconds, or any multiple of 60 seconds." [A8]
- 즉 AWS 서비스가 게시하는 메트릭은 기본 1분(standard resolution), 커스텀 메트릭은 게시 시 standard 또는 high resolution(1초) 중 선택 가능. [A8]
- **Logs**: 로그 그룹/스트림, Logs Insights 쿼리, 메트릭 필터, 구독 필터 — *CloudWatch Logs User Guide* 참조. [A9]
- **Alarms / EventBridge**: 메트릭 기준 알람 트리거, EventBridge 의 이벤트 버스·규칙. [A9]
- **주요 API**: `PutMetricData`, `GetMetricStatistics`, `PutMetricAlarm`, `DescribeAlarms`, `FilterLogEvents` — *CloudWatch API Reference* 의 액션. [A8][A9]

### 4.7 CloudFormation
- IaC 서비스로 JSON/YAML 템플릿으로 스택 단위 자원 배포. [A10]
- **요소**: Parameters, Mappings, Conditions, Resources, Outputs, Transform, Change Sets, Stack Sets(다중 계정/리전) — *CloudFormation User Guide* 에 정의. [A10]
- **주요 API**: `CreateStack`, `UpdateStack`, `DeleteStack`, `CreateChangeSet`, `ExecuteChangeSet`, `DescribeStacks` — *AWS CloudFormation API Reference* 의 액션. [A10]

### 4.8 Billing (결제 및 비용 관리)
- **AWS Billing and Cost Management** 가 청구서, 결제 수단, 크레딧, 세금 설정의 1차 콘솔. [A11]
- **Cost Explorer**, **Budgets**, **Cost and Usage Report (CUR)**, **Savings Plans**, **AWS Organizations 통합 결제** — 모두 *AWS Billing and Cost Management User Guide* 의 공식 기능. [A11]
- **주요 API**: AWS Cost Explorer API (`GetCostAndUsage`, `GetReservationUtilization`), AWS Budgets API (`CreateBudget`, `DescribeBudgets`), AWS Price List API — *AWS Cost Explorer Service / AWS Budgets API Reference*. [A11][A12]

### 4.9 출처 (확인일 2026-05-25)
- **[A1]** *Amazon EC2 API Reference — RunInstances*. <https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RunInstances.html>
- **[A2]** *Amazon EC2 API Reference — DescribeInstances*. <https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstances.html>
- **[A3]** *Amazon VPC User Guide*. <https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html>
- **[A4]** *Amazon EBS User Guide*. <https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html>
- **[A5]** *Amazon S3 User Guide* (및 API Reference). <https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html> / <https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html>
- **[A6]** *AWS IAM User Guide*. <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html>
- **[A7]** *AWS IAM — How IAM works*. <https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html> (정책 평가 로직: <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html>)
- **[A8]** *Amazon CloudWatch — Metrics concepts*. <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html>
- **[A9]** *Amazon CloudWatch User Guide*. <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html>
- **[A10]** *AWS CloudFormation User Guide / API Reference*. <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html> / <https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/Welcome.html>
- **[A11]** *AWS Billing and Cost Management User Guide*. <https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html>
- **[A12]** *AWS Cost Explorer Service API Reference*. <https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/Welcome.html>

---

## 5. Cloud Bursting

### 5.1 개념
**Cloud Bursting**은 평소에는 온프레미스(또는 프라이빗 클라우드)에서 워크로드를 처리하다가, **부하가 임계치를 초과하면 일부 또는 신규 워크로드를 퍼블릭 클라우드로 일시 확장**하는 하이브리드 클라우드 아키텍처 패턴이다.

목적:
- 평상시 비용 최적화 (자체 자원 우선 사용)
- 피크 타임(이벤트, 시즌, 광고 캠페인 등)에 무한 탄력성 확보
- DR(재해복구)·BCP 보조 수단

### 5.2 동작 원리
1. **모니터링**: 온프레미스의 CPU/메모리/큐 길이/응답시간 등 임계치 감시 (Prometheus, CloudWatch Agent 등)
2. **트리거**: 임계치 초과 시 오케스트레이터(Kubernetes HPA/Cluster Autoscaler, Terraform, 자체 스크립트)가 동작
3. **프로비저닝**: 퍼블릭 클라우드에서 신규 VM/컨테이너/Pod 생성
4. **네트워크 확장**: Site-to-Site VPN, AWS Direct Connect, Azure ExpressRoute로 사설망 연결 → 사내망의 일부처럼 동작
5. **트래픽 분산**: 글로벌 로드밸런서(Route 53, NGINX, F5, Cloudflare 등)가 신규 인스턴스로 트래픽 라우팅
6. **데이터 동기화**: 공유 스토리지(S3/Blob, 또는 데이터베이스 복제)로 상태 일관성 유지
7. **축소(Scale-In)**: 부하가 안정화되면 퍼블릭 측 자원을 해제하여 비용 절감

### 5.3 사전 고려사항
- **데이터 중력(Data Gravity)**: 대용량 데이터의 이동 비용·지연
- **네트워크 대역폭과 지연시간**: 워크로드의 stateful 정도에 따라 결정적
- **데이터 주권/규제**: 개인정보·금융 데이터의 국외 이전 제약
- **이미지·구성 일치성**: 양쪽에서 동일하게 동작하도록 Packer/Ansible/Terraform 활용
- **비용 모델**: 데이터 송신(egress) 요금이 크다는 점 주의

### 5.4 하이브리드 클라우드 사례

| 사례 | 특징 |
|---|---|
| **AWS Outposts** | AWS가 관리하는 물리 랙을 고객 데이터센터에 설치, 동일한 AWS API/서비스(EC2, EBS, RDS, EKS 등)를 온프레미스에서 사용. 리전과 일관된 컨트롤 플레인 |
| **AWS Local Zones / Wavelength** | 특정 메트로(또는 5G 통신사 엣지)에 AWS 인프라 배치하여 저지연 워크로드 처리. 버스팅의 경량 형태 |
| **Azure Stack Hub** | Azure 호환 IaaS/PaaS를 자체 데이터센터에서 운영. 연결·차단(disconnected) 환경 모두 지원 |
| **Azure Stack HCI** | 하이퍼컨버지드 인프라 솔루션. Azure 서비스(Arc, Backup, Monitor)와 연동 |
| **Azure Arc** | 온프레미스/타사 클라우드 자원까지 Azure Resource Manager로 통합 관리. 정책·거버넌스를 하이브리드 전반에 적용 |
| **Google Anthos** | GKE 기반 하이브리드/멀티클라우드 플랫폼. 온프레미스·AWS·Azure에 동일 K8s 환경 제공 |
| **Google Distributed Cloud** | 엣지·온프레미스용 GCP 확장 (구 GDC Edge / Hosted) |
| **VMware Cloud on AWS** | VMware SDDC를 AWS 베어메탈에서 운영, vMotion으로 온프레미스 ↔ 클라우드 간 마이그레이션 |
| **IBM Cloud Satellite** | 고객 위치·타사 클라우드에 IBM Cloud 서비스 배포 |
| **OpenStack + Public Cloud (Federation)** | Keystone-to-Keystone Federation, K8s Cluster API로 OpenStack과 퍼블릭 클라우드 연계 |

---

## 6. 법적 규제 (대한민국)

> ⚠️ 본 문서는 일반 정보 제공 목적이며 법률 자문이 아니다. 실제 규제 적용 여부는 변호사·관할 기관과 별도 검토가 필요하다.
>
> 적용 범위: 본 §6의 법규 항목은 ADR-0008에 따라 Phase 3까지 PG Mock 구현체만 제공되는 본 프로젝트의 **현재 단계**에서는 **직접 준수 요건이 아니다**. 사업화 전환(실제 PG 연동·유료 판매·공공기관 대상 제공)이 발생할 때 ADR-0008 회귀 트리거를 따라 별도 법무 검토와 함께 본 매핑을 활용한다.
>
> 출처 표기 규칙: 본 절은 국가법령정보센터(`law.go.kr`), 한국인터넷진흥원(`kisa.or.kr`), 개인정보보호위원회(`privacy.go.kr`), 공정거래위원회(`ftc.go.kr`) 등 **소관 부처·법령 1차 출처**를 사용한다. 각 항목 끝의 `[L#]` 마커는 § 6.5 출처 목록의 항목 번호이며, **기본 확인일은 2026-05-25이며, [L11]은 2026-06-03, [L12]는 2026-06-02에 재확인·갱신**했다.

### 6.1 CSAP (클라우드 보안 인증제, Cloud Security Assurance Program)
- **시행 근거**: 한국인터넷진흥원(KISA) 공식 페이지는 시행 근거를 「**클라우드컴퓨팅 발전 및 이용자 보호에 관한 법률 제23조의2** (클라우드컴퓨팅서비스의 보안인증)」로 명시한다. [L1]
- **정책기관·인증기관**: 정책기관은 **과학기술정보통신부**, 인증기관은 **한국인터넷진흥원(KISA)**, 평가기관은 (사)한국정보통신진흥협회, (주)한국아이티평가원, 한국시스템보증(주) — 모두 KISA 공식 페이지의 설명. [L1]
- **인증 대상**: 클라우드서비스(**IaaS, PaaS, SaaS** 등) 제공사업자로서, **국가·공공기관 등의 업무를 위해** 클라우드서비스를 제공하려는 자. [L1]
- **인증 등급(상·중·하)**: KISA 공식 안내는 "클라우드서비스 보안인증 등급은 **상·중·하**로 구분된다"고 명시하며, 등급별 평가기준 차등화(상=강화, 중=현행 유지, 하=완화)가 도입되었음을 확인할 수 있다. 등급제 도입은 2023년 1월에 발표되었고, **하 등급**은 고시에 반영되어 시행 중이다(상·중 등급의 평가기준은 2024년 행정예고 등을 거쳐 순차 시행). [L1][L2]
- **인증 평가 종류와 유효기간**: 최초평가 → 사후평가 → 갱신평가의 3단계로 운영되며, 최초평가 통과 시 **5년의 유효기간**이 부여되고, 사후평가는 유효기간 내 매년 시행되며, 갱신평가 통과 시 5년이 다시 부여된다. [L1]
- **평가 기준 근거**: 「**클라우드컴퓨팅서비스 보안인증에 관한 고시**」 제15조(보안인증기준). [L1]

### 6.2 개인정보보호법 (PIPA)
- **근거 법령**: 「**개인정보 보호법**」 (법률 — 국가법령정보센터). [L3]
- **국외 이전 (제28조의8)**: 본문은 다음과 같다. *"개인정보처리자는 개인정보를 국외로 제공(조회되는 경우를 포함한다)ㆍ처리위탁ㆍ보관(이하 이 절에서 '이전'이라 한다)하여서는 아니 된다."* 다만 다음 5가지 경로 중 하나에 해당하면 국외 이전이 가능하다고 규정한다. [L4]
  1. **정보주체로부터 국외 이전에 관한 별도의 동의**를 받은 경우
  2. **법률, 대한민국 당사자 조약 또는 그 밖의 국제협정**에 개인정보의 국외 이전에 관한 특별한 규정이 있는 경우
  3. **정보주체와의 계약의 체결 및 이행을 위하여** 개인정보의 처리위탁·보관이 필요한 경우로서, 법정 고지사항을 개인정보 처리방침에 공개하거나 전자우편 등으로 알린 경우
  4. **개인정보를 이전받는 자가 개인정보보호위원회가 정하는 인증을 받은 경우**로서 인증의 유지를 위해 필요한 조치 및 정보주체 권리 보장 조치 등을 한 경우
  5. **개인정보가 이전되는 국가 또는 국제기구가 보호위원회로부터 개인정보 보호 수준 인정을 받은 경우** [L4]
- **시행령상 보호조치**: 「개인정보 보호법 시행령」 제29조의10은 단서에 따라 국외이전 시 ① 안전성 확보 조치, ② 고충처리·분쟁해결 조치, ③ 그 밖에 정보주체 보호를 위해 필요한 조치를 의무화하고 이를 계약에 반영하도록 규정(2023 신설). [L5]
- **인증 유효기간**: 시행령 제29조의8 ②항은 보호위원회가 인증 고시 시 "**5년의 범위에서 유효기간**을 정하여 고시"할 수 있도록 함. [L5]
- **과징금 (제64조의2)**: *"보호위원회는 다음 각 호의 어느 하나에 해당하는 경우에는 해당 개인정보처리자에게 **전체 매출액의 100분의 3을 초과하지 아니하는 범위**에서 과징금을 부과할 수 있다."* — 즉 위반 시 매출액의 3% 이내 과징금이 법정 한도. (제2항: 위반행위와 무관한 매출액은 제외, 제4항: 비례성·효과성 고려) [L6]
- **소관 부처**: **개인정보보호위원회** (`privacy.go.kr`)가 국외이전·과징금 등 제도 운영의 1차 소관. [L7]

### 6.3 전자상거래법 — 통신판매업
- **근거 법령**: 「**전자상거래 등에서의 소비자보호에 관한 법률**」(약칭: 전자상거래법). [L8]
- **청약 철회 (제17조 ①항)**: 소비자는 다음 기간 이내에 청약 철회가 가능하다. *"계약내용에 관한 서면을 받은 날부터 **7일**. 다만, 그 서면을 받은 때보다 재화등의 공급이 늦게 이루어진 경우에는 재화등을 공급받거나 재화등의 공급이 시작된 날부터 7일"*. 서면 미수령·주소 부재 시에는 통신판매업자의 주소를 안 날부터 7일. [L8][L9]
- **청약 철회 제한 사유 (제17조 ②항)**: 소비자 귀책으로 재화가 멸실·훼손된 경우, 사용·소비로 가치가 현저히 감소한 경우, 시간 경과로 재판매가 곤란한 경우, 복제 가능한 재화의 포장이 훼손된 경우, **디지털 콘텐츠의 제공이 개시된 경우(소비자의 사전 동의·시험사용 제공 등 일정 요건 충족 시)** 등은 제한될 수 있음. [L8][L9]
- **사업자 정보 표시 (제10조)**: 상호, 대표자, 주소, 전화번호, 사업자등록번호, 통신판매신고번호 등을 표시 의무. [L8]
- **통신판매업 신고 (제12조)**: 통신판매업자는 대통령령이 정하는 사항을 공정거래위원회 또는 특별자치시장·특별자치도지사·시장·군수·구청장에게 신고해야 한다. 다만 시행령에 따라 직전년도 거래횟수 50회 미만 또는 부가가치세법상 간이과세자 등은 신고 면제. [L8][L10]
- **소관 부처**: **공정거래위원회** (`ftc.go.kr`). [L10]
- **위반 시**: 시정조치·과태료·영업정지 및 형사 처벌 (법 제40조 이하).

### 6.4 전기통신사업법 — 부가통신사업 신고
- **근거 법령**: 「**전기통신사업법**」 및 같은 법 시행령. [L11][L12]
- **사업 분류**: 전기통신사업법은 **기간통신사업자**(전기통신회선설비 보유)와 **부가통신사업자**(기간통신역무 외 부가역무 제공)로 구분한다. 클라우드·호스팅·콘텐츠 제공 등은 부가통신사업에 해당한다. [L11]
- **신고 의무 (제22조 제1항)**: 부가통신사업을 경영하려는 자는 **과학기술정보통신부장관**에게 신고해야 한다(접수는 중앙전파관리소). [L11]
- **소규모 부가통신사업의 신고 면제 (제22조 제4항 + 시행령 제30조 제1항)**: *"법 제22조제4항제1호에서 '대통령령으로 정하는 기준에 해당하는 소규모 부가통신사업'이란 **인터넷을 이용하여 부가통신역무를 제공하는 자본금 1억 원 이하인 부가통신사업자**를 말한다."* 자본금이 1억 원을 초과하게 되면 **1개월 이내** 신고 의무가 발생한다. [L12][L13]
- **벌칙 (제96조)**: 미신고 부가통신사업 영위 시 **2년 이하의 징역 또는 1억 원 이하의 벌금**. [L11][L13]
- **추가 의무**: 약관 신고·게시, 이용자 정보 보호, 일정 규모 이상 사업자의 **서비스 안정성 확보 의무**("넷플릭스법") 등.

### 6.5 출처 (기본 확인일 2026-05-25; [L11]은 2026-06-03, [L12]는 2026-06-02 재확인·갱신)
- **[L1]** 한국인터넷진흥원, *클라우드서비스 보안인증(CSAP)* 안내(KISA 사업소개, 최종수정일 2026-03-12). <https://www.kisa.or.kr/1050603>
- **[L2]** 한국인터넷진흥원, *클라우드 보안인증제 소개*(ISMS 포털). <https://isms.kisa.or.kr/main/csap/intro/>
- **[L3]** 국가법령정보센터, *개인정보 보호법* (법령 본문). <https://www.law.go.kr/LSW/lsInfoP.do?lsId=011357&ancYnChk=0>
- **[L4]** 국가법령정보센터, *개인정보 보호법* 제28조의8(개인정보의 국외 이전). 원문 확인: <https://www.law.go.kr/LSW/lsInfoP.do?lsId=011357&ancYnChk=0> (조문 참조용 보조: <https://casenote.kr/법령/개인정보_보호법/제28조의8>)
- **[L5]** 국가법령정보센터, *개인정보 보호법 시행령* 제29조의8, 제29조의10. <https://www.law.go.kr/LSW/lsInfoP.do?lsId=011468&ancYnChk=0>
- **[L6]** 국가법령정보센터, *개인정보 보호법* 제64조의2(과징금의 부과). 동일 법령 URL 참조: <https://www.law.go.kr/LSW/lsInfoP.do?lsId=011357&ancYnChk=0>
- **[L7]** 개인정보보호위원회, *국외이전 제도 안내*. <https://www.privacy.go.kr/front/contents/cntntsView.do?contsNo=367>
- **[L8]** 국가법령정보센터, *전자상거래 등에서의 소비자보호에 관한 법률* 본문. <https://www.law.go.kr/lsInfoP.do?lsId=009318&ancYnChk=0>
- **[L9]** 국가법령정보센터, *전자상거래 등에서의 소비자보호에 관한 법률* 제17조(청약철회등) 인용 페이지. <https://law.go.kr/LSW//lsLawLinkInfo.do?chrClsCd=010202&lsId=009318&lsJoLnkSeq=1000527255>
- **[L10]** 공정거래위원회, *전자상거래법 주요내용* 안내. <https://www.ftc.go.kr/www/contents.do?key=153>
- **[L11]** 국가법령정보센터, *전기통신사업법* [시행 2026. 5. 19.] [법률 제21652호, 2026. 5. 19., 일부개정]. 본 문서 §6.4의 제22조 제4항 인용은 2026-06-03 기준 현재 시행 체계를 따른다. 국가법령정보센터의 제22조 조문정보 중 [시행 2026.11.20.] 미래 시행본에서는 소규모 부가통신사업 신고 간주 조항이 제5항 체계로 이동한다 — 2026-11-20 시행일 도래 시 §6.4 인용을 재확인. <https://www.law.go.kr/LSW/lsInfoP.do?chrClsCd=010202&lsId=&lsiSeq=286079&urlMode=lsInfoP>
- **[L12]** 국가법령정보센터, *전기통신사업법 시행령* 제30조(부가통신사업자 신고의 면제; 대통령령 제36281호, 2026-04-28 시행본 고정 URL). <https://law.go.kr/LSW/lsInfoP.do?ancYnChk=0&chrClsCd=010202&efYd=20260428&lsiSeq=285721&urlMode=lsInfoP>
- **[L13]** 과학기술정보통신부 중앙전파관리소, *부가통신사업 신고 제도 안내*. <https://www.crms.go.kr/lay1/S1T54C59/contents.do>

---

## 7. 핵심 비교 요약

### 7.1 IaaS 플랫폼 비교

| 항목 | OpenStack | CloudStack | Proxmox VE |
|---|---|---|---|
| 라이선스 | Apache 2.0 | Apache 2.0 | AGPLv3 (서브스크립션 별도) |
| 아키텍처 | 분산 마이크로서비스 | 통합형 (Java + MySQL) | 노드 분산 + Corosync |
| 멀티 테넌시 | 강함 (Domain/Project/Role) | 강함 (Domain/Account) | 약함 (Pool/Permission) |
| 빌링 | CloudKitty (외부 통합) | Usage Server 내장 | 없음 |
| 적합 규모 | 대규모 (수천 호스트+) | 중대규모, 통신사 | 소·중규모 (단일 LAN 권장, 멀티사이트 부적합) |
| 운영 난이도 | 매우 높음 | 중간 | 낮음 |

### 7.2 법규 준수 체크리스트

| 체크 항목 | 관련 법규 |
|---|---|
| 공공기관 대상 클라우드 영업 | CSAP 인증 (상/중/하 등급) |
| 개인정보를 다루는 모든 서비스 | 개인정보보호법 (안전성 조치, 동의, 처리방침) |
| 개인정보 국외 이전 발생 시 | 개인정보보호법 국외 이전 5가지 경로 중 하나 충족 |
| 클라우드 서비스를 유료로 판매 | 전자상거래법: 통신판매업 신고 + 정보 표시 + 청약 철회 등 |
| 클라우드/IDC/호스팅 사업 | 전기통신사업법: 부가통신사업 신고 (자본금 1억 원 초과) |

---

## 8. 참고 자료

### 8.1 1차 출처와 확인일에 관한 안내
**Proxmox API · AWS API · 한국 법규** 세 영역에 대한 모든 사실 주장은 각 절에서 1차 출처(공식 위키·공식 매뉴얼·공식 API Reference·국가법령정보센터·KISA·개인정보보호위원회·공정거래위원회·중앙전파관리소 등)로 직접 인용되어 있으며, 모든 URL의 접근·교차 확인일은 **기본 2026-05-25이며, [L11]은 2026-06-03, [L12]는 2026-06-02에 재확인·갱신**했다. 인용 마커는 각 절 내부의 출처 목록과 1:1 대응한다.

- Proxmox VE API → § 3.6 출처 목록 (`[P1]`–`[P4]`)
- AWS 서비스 → § 4.9 출처 목록 (`[A1]`–`[A12]`)
- 대한민국 법적 규제 → § 6.5 출처 목록 (`[L1]`–`[L13]`)

### 8.2 그 외 일반 참고 자료
인용 대상이 아닌 보조 학습용 자료. 본 문서의 사실 주장은 위 1차 출처가 우선한다.

- OpenStack 공식 문서: <https://docs.openstack.org/>
- Apache CloudStack 공식 문서: <https://docs.cloudstack.apache.org/>
- AWS Outposts: <https://docs.aws.amazon.com/outposts/>
- Microsoft Azure Stack / Arc: <https://learn.microsoft.com/en-us/azure-stack/>
- Google Anthos / Distributed Cloud: <https://cloud.google.com/anthos/docs>

### 8.3 정보의 시의성·면책
- 한국 법령은 개정 빈도가 높다. 본 문서의 인용은 2026-05-25 시점의 국가법령정보센터 본문과 소관 부처 안내를 따랐으나, 실제 적용 시점에는 최신 본문(특히 시행일, 단서 추가)을 반드시 재확인해야 한다.
- CSAP 등급제(상·중·하)는 시행 일정이 단계적으로 진행 중이다. 신청·평가 기준의 세부 항목은 「클라우드컴퓨팅서비스 보안인증에 관한 고시」의 최신 개정본을 확인할 것.
- Proxmox VE / AWS 공식 문서는 버전·릴리스에 따라 본문이 갱신되므로, 자동화 코드를 작성할 때는 인용한 페이지에서 동일 문장이 유지되는지 다시 확인할 것.