# ADR-0004: 백엔드 언어/프레임워크 선택

- **상태(Status)**: Proposed
- **결정일(Date)**: TBD (Phase 1 진입 전 확정)
- **결정자(Decider)**: 프로젝트 메인테이너
- **관련 ADR**: 0002 (Proxmox 채택), 0003 (자체 오케스트레이션)

## Context — 맥락

본 플랫폼의 컨트롤 플레인은 Proxmox REST API를 어댑터로 격리한 위에(ADR-0002·0003) 다음 책임을 가진다.

- HTTP/REST API + WebSocket(noVNC 프록시 등)
- 노드·VM 메타데이터 영속화 (관계형 DB)
- 미터링 파이프라인 (수집 → 정규화 → 시계열 저장)
- 백그라운드 작업 (스케줄러·페일오버 감지·청구서 생성)
- 멀티테넌시 강제 (모든 쿼리에 테넌트 ID 필터)
- 빌링 무결성 (append-only, 해시 체인/머클트리 — ADR-0006)

언어·프레임워크 선택은 다음에 직접 영향을 준다.

- **개발 속도 vs 런타임 성능**: Phase 1 시연 일정과 Phase 4+ 다중 노드 부하 사이 균형.
- **타입 안전성**: 멀티테넌시 격리·빌링 무결성을 컴파일 타임에 강제할 수 있는가.
- **에코시스템**: Proxmox API 클라이언트 라이브러리, OIDC, 시계열 DB 클라이언트, PDF 생성기.
- **운영 부담**: 1인 운영 가능 범위 안에서 런타임·의존성 관리가 단순한가.
- **라이선스 호환성**: AGPLv3 Proxmox 의존성과 본 프로젝트 라이선스(미정, ADR-별도) 정합.

본 ADR은 Phase 1 진입 전 확정해야 한다. 결정이 지연되면 Phase 1 코드가 즉시 막힌다.

## Decision — 결정

[Phase 1 진입 전 확정. 본 섹션은 결정 시점에 채워진다.]

## Alternatives Considered — 대안 비교

[Phase 1 진입 전 확정. 후보로 검토 예정인 조합:]

| 대안 | 강점(예상) | 약점(예상) | 결정 |
|---|---|---|---|
| Go + Gin/Echo + sqlc | 단일 바이너리, 동시성, 정적 타입 | 제네릭 도입 이전 코드 패턴 잔존 | TBD |
| Python + FastAPI + SQLAlchemy | 생태계·생산성·문서화 우수 | 런타임 성능, 배포 복잡도 | TBD |
| TypeScript (Node.js) + NestJS/Fastify + Prisma | 단일 언어 풀스택, 타입 우수 | Node 운영·메모리 모델 | TBD |
| Rust + Axum + SQLx | 메모리 안전·성능 | 개발 속도 비용 큼 | TBD |
| Elixir + Phoenix | 동시성·내결함성 우수 | 1인 운영자 학습 곡선 | TBD |

## Consequences — 결과

[결정 시점에 채워진다.]

## Open Questions

- 단일 모놀리식 백엔드인가, 처음부터 컨트롤 플레인 / 미터링 / 빌링 3분할인가?
- DB는 단일 PostgreSQL인가, 시계열은 별도 (TimescaleDB/InfluxDB)인가?
- 백그라운드 작업 큐 (Celery/RQ/Sidekiq/Asynq/BullMQ 등) 채택 시점은 Phase 1인가, Phase 3 빌링 진입 시점인가?
- 프런트엔드 (웹 콘솔)와 같은 언어 스택으로 묶을 가치가 있는가?
- 결정의 회귀 트리거: Phase 5 (Adjacent Services) 도입 시 폭증하는 코드 표면적을 본 스택이 감당 가능한가?
