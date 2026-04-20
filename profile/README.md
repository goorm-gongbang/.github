# 구름공방 · Playball

<p align="center">
  <img src="https://raw.githubusercontent.com/goorm-gongbang/001-finalPR/main/img/playball_logo.svg" alt="Playball Logo" width="200"/>
</p>

<p align="center">
  <b>야구 티켓팅 플랫폼 Playball</b><br/>
  <i>대규모 트래픽에서도 원하는 자리를 쉽고 빠르게</i>
</p>

<p align="center">
  <a href="https://playball.one"><img src="https://img.shields.io/badge/Service-playball.one-00AD82?style=for-the-badge"/></a>
  <a href="https://docs.playball.one/planning/overview"><img src="https://img.shields.io/badge/Docs-docs.playball.one-1f6feb?style=for-the-badge"/></a>
</p>

---

## 목차

- [Brand Message](#brand-message)
- [시장 문제와 솔루션](#시장-문제와-솔루션)
- [비즈니스 임팩트](#비즈니스-임팩트)
- [주요 기능](#주요-기능)
- [시스템 아키텍처](#시스템-아키텍처)
- [보안 아키텍처](#보안-아키텍처)
- [핵심 기술](#핵심-기술)
- [핵심 성과](#핵심-성과)
- [기술 스택](#기술-스택)
- [링크](#링크)
- [Team](#team)

---

## Brand Message

> **Play Ball!**
> 야구 경기 시작을 알리기 위해 심판이 외치는 신호.
> 경기가 시작되는 순간처럼 티켓팅 또한 **지금 시작된다**는 긴장감과 속도를 가지고 있습니다.

---

## 시장 문제와 솔루션

### 현재 스포츠 티켓팅의 구조적 문제

2025 KBO 총 관중 **12,312,519명** · 좌석 점유율 **82.9%** · 경기당 평균 **17,101명** · 연간 티켓 시장 약 **1,290억 원**.
2년 연속 최다 관중 갱신, 2030 여성 팬층 유입 등 시장은 커지고 있지만 구조적 문제도 함께 커졌습니다.

| Pain Point | 현상 |
| --- | --- |
| **암표 시장 과열** | 1·2차 티켓 구매 경험자의 **36%가 2차 판매(암표) 구매**, 36.5%는 구매 고려 (마크로밀엠브레인 조사) |
| **매크로 / 봇 공격** | 단순 봇·자동화 봇·Residential Proxy 고급 봇까지 — 전통 탐지 규칙은 오탐·미탐 트레이드오프에 갇힘 |
| **자동배정 불만족** | 결제 전까지 좌석 위치 확인 불가 · 관람 경험 저하 |
| **직접 선택의 비효율** | 빠른 확보 우선으로 좌석이 띄엄띄엄 확보되어 **연석 확보 어려움** · 네트워크 빠른 사용자만 유리한 불공정 |

### 솔루션 — 분산 추천으로 수요 분포 재설계

```
온보딩 선호 수집  →  추천 구역 기반 티켓팅  →  구역 내 좌석 자동 배정
  (응원구단·뷰포인트         (온보딩 데이터 +           (블록 단위 분산 락 +
   ·응원석 근접)              인원 수 기반 추천)          조건부 UPDATE)
```

**핫스팟에 집중되던 트래픽을 선호 기반 블록으로 분산**시켜 구조적 경합을 해소합니다.

---

## 비즈니스 임팩트

| 이해관계자 | 가치 |
| --- | --- |
| **팬 (B2C)** | 원하는 구역 연석 확보 성공률 ↑ · 2차 티켓 구매 비용 절감 · 구매 비효율 개선 |
| **운영사 (B2B)** | 동일 좌석 대비 판매율 ↑ · 빅매치 장애 리스크 감소 · 서버 비용 저하 |
| **구단 (B2B)** | CS / 환불 민원 감소 · 팬 경험 표준화 · 경기별 예매 데이터 인사이트 |
| **KBO (시장)** | 리그 신뢰도 ↑ · 신규 팬층 이탈 방지 · 정상 예매 생태계 복원 |

---

## 주요 기능

### 1. 온보딩 기반 맞춤 추천

- 선호 구역 **최대 10개** 지정 (좌석맵에서 블록 직접 선택)
- 뷰포인트 우선순위 **1~3순위** · 응원 구단 · 응원석 근접 선호 (필수)
- 관람 환경 · 응원 분위기 · 시야 민감도 · 가격대 (선택, 부가 가중치)

### 2. 추천 기반 좌석 배정

- **취향 점수 계산 (최대 70점)**: 뷰포인트 매칭 + 응원구단 + 응원석 근접도
- **연석 탐색 2단계**: 실연석 (앞줄 · 통로 · 왼쪽 우선) → 준연석 fallback (앞줄 합 · 겹침 · 평균 통로)
- **블록 단위 Redisson 분산 락 + Watch Dog** 자동 연장
- **조건부 UPDATE**로 중복 선점 차단, 5분 Hold TTL

### 3. 대기열 (Queue)

- Redis ZSET + Lua 스크립트 기반 순번 관리
- Admission Token 발급으로 좌석 API 진입 권한 제어
- 결제 완료 / Hold 확정 이벤트 시 READY 슬롯 즉시 회수

### 4. 인증 / 인가 (Auth-Guard)

- 카카오 OAuth2 로그인
- JWT Access / Refresh 분리 · HttpOnly Cookie
- AES-256-GCM 필드 암호화 (PII 보호)

### 5. VQA 보안 인증

- 기존 문자 캡차와 다른 **시각 질의응답(VQA)** 방식
- 실제 티켓팅 전 **사전 체험 페이지**로 사용자가 난이도 · 조작 방식을 미리 학습
- 매크로 · 봇 자동 풀이 방지

### 6. AI 공격 / 방어 에이전트

> **공격자가 되어 방어를 검증한다** — 기존 AI 방어의 맹점을 스스로 뚫어보며 약점을 보완.

- **공격 에이전트**: FlowState 상태머신 · 사람 궤적 재현(곡선·미세 떨림) · VQA 자동 풀이 · 스웜 인프라 · LLM 코디네이터 (전략 실시간 조정)
- **방어 에이전트**: 실시간 룰 기반 5개 행동 지표(직선도 35% · 손떨림 25% · 머뭇거림 15% · 경로 15% · 속도 10%) 가중합 → Tier(T0~T3) 판정 → VQA 게이트 필수 통과
- **LLM 사후 판단 (Track A/B 병렬)**: T1·T2 애매 세션 재검토 + 탐지 기준 점진 적용 (5% → 20% → 50% → 100%) · 장애 시 규칙 기반 자동 전환
- **실측 결과** (30 trace 기준): Runtime Action THROTTLE 63.3% · BLOCK 0% · 공격 Hold 도달률 66.7% · VQA 통과율 83.3%

### 7. 주문 / 결제

- 카카오 / 토스 / 무통장 입금 지원
- `@TxEventListener(AFTER_COMMIT)` 기반 Transactional Outbox 유사 구조
- Kafka 이벤트 기반 좌석 상태 동기화 (payment-completed · order-cancelled · bank-transfer-expired)

---

## 시스템 아키텍처

<p align="center">
  <img src="https://raw.githubusercontent.com/goorm-gongbang/001-finalPR/main/Group%202085665504.svg" alt="System Architecture" width="100%"/>
</p>

- **Entry**: API-Gateway `:8085` (Spring Cloud Gateway + WebFlux) — JWT 검증 · X-User-Id 주입 · Rate Limit
- **Service Mesh**: Istio + ArgoCD GitOps
- **MSA 모듈**: Auth-Guard `:8080` · Queue `:8081` · Seat `:8082` · Order-Core `:8083` · common-core (공유 라이브러리)
- **Data**: Amazon RDS PostgreSQL 16 (서비스별 스키마 · AES-256-GCM 필드 암호화) · ElastiCache Redis (ZSET 대기열 · Redisson RLock · Admission Token · 블랙리스트)
- **Messaging**: Apache Kafka (KRaft · 3 brokers · P=3 · acks=all · DLT) — `payment-completed` · `order-cancelled` · `bank-transfer-expired` · `seat-hold-completed` · `user-blocked`

**비동기 정합성 보장 4중 안전 장치**: `@TxEventListener(AFTER_COMMIT)` Transactional Outbox · 파티션 키 기반 순서 보장 · `acks=all` + DLT 재시도 · 멱등 소비.

---

## 보안 아키텍처

<p align="center">
  <img src="https://raw.githubusercontent.com/goorm-gongbang/001-finalPR/main/Group%202085665504.svg" alt="Security Architecture" width="100%"/>
</p>

- **authz-adapter** (Go · gRPC `:9001`): API Gateway 요청을 가로채 권한/봇 판단을 AI-defense에 위임 · 정책 양방향 주입
- **AI-defense** (FastAPI `:8000`): 실시간 Tier 판정(T0~T3) + VQA 게이트 + LLM 사후 판단 · Redis/ClickHouse 이벤트 스트림
- **VQA 필수 게이트**: 좌석 선택 진입 시 등급과 무관하게 전원 1회 통과 필수 (정답 + 풀이 과정 2중 검증)
- **개인정보 보호**: AES-256-GCM 필드 암호화(PII) · HttpOnly Cookie 기반 Refresh/admissionToken 분리

---

## 핵심 기술

### 좌석 배정 & 동시성 제어

```sql
-- 조건부 UPDATE (낙관적 동시성 제어)
UPDATE match_seats
   SET sale_status = 'BLOCKED'
 WHERE id = :matchSeatId
   AND sale_status = 'AVAILABLE'

-- return 0 → 이미 선점됨 → 롤백 후 재시도 (최대 3회)
```

- **블록 단위 분산 락**: 동일 블록 접근 유저 중 1명만 획득 → 좌석 탐색 **직렬화** → 중복 선점 차단
- **Watch Dog 자동 연장**: Redisson이 작업 지속 시 TTL 자동 갱신
- **재시도 각 최대 3회**: 실연석 / 준연석 각각 다른 구간 탐색
- **Hold 5분 TTL + Cleanup Scheduler**: 60초 간격 만료 Hold 자동 해제

---

## 핵심 성과

### 부하 테스트 (k6 · 1,000 VU · Phase 0 → Phase 4)

| 메트릭 | Phase 0 (AS-IS) | Phase 4 (TO-BE) | 개선률 |
| --- | --- | --- | --- |
| Queue P99 | 8,342 ms | **898 ms** | **−89%** |
| Seat P99 | 8,048 ms | 4,414 ms | −45% |
| 시나리오 완주 시간 | 2분 38초 | **15초** | **−90.5%** |
| RPS (Queue + Seat) | 33.8 | **237.6** | **×7** |
| 5xx 에러 | 1~2% | **0건** | 완전 제거 |
| DB 커넥션 peak | 270 (한계) | ~100 | −63% |

> **DB 인스턴스 업그레이드 없이** 앱 레벨 4단계 최적화 (Phase 1 Caffeine 로컬 캐시 → Phase 2 Redis 분산 캐시 → Phase 3 응답 캐시 + HikariCP 튜닝 → Phase 4 OSIV OFF + Redis Lua 원자 연산 + Resilience4j 서킷브레이커)만으로 달성.

### 추천 알고리즘 실측 비교 (1,000 VU · 동일 경기 경합)

| 지표 | 추천 OFF (포도알 직접 선택) | 추천 ON (서버 자동 배정) | 효과 |
| --- | --- | --- | --- |
| 이선좌 (409) 경합 | **2,472건** | **0건** | 경합 완전 제거 |
| 시도 성공률 | 60.4% | **100%** | +39.6%p |
| 가상 유저 1명당 HOLD 시도 | 6.24회 | **1회** | −5.24회 · 재시도 제거 |

---

## 기술 스택

### Backend

![Java 21](https://img.shields.io/badge/Java-21-007396?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-4.0.2-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=for-the-badge&logo=springsecurity&logoColor=white)
![Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Redisson](https://img.shields.io/badge/Redisson-D12229?style=for-the-badge&logo=redis&logoColor=white)

### Frontend

![Next.js](https://img.shields.io/badge/Next.js-15-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)

### AI

![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)
![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=for-the-badge&logo=playwright&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![ClickHouse](https://img.shields.io/badge/ClickHouse-FFCC01?style=for-the-badge&logo=clickhouse&logoColor=black)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)

### Cloud · Infra

![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Kubernetes](https://img.shields.io/badge/EKS-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Istio](https://img.shields.io/badge/Istio-466BB0?style=for-the-badge&logo=istio&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![TeamCity](https://img.shields.io/badge/TeamCity-000000?style=for-the-badge&logo=teamcity&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)

### Design

![Figma](https://img.shields.io/badge/Figma-F24E1E?style=for-the-badge&logo=figma&logoColor=white)
![Pretendard](https://img.shields.io/badge/Pretendard-000000?style=for-the-badge)

---

## 링크

| 항목 | URL |
| --- | --- |
| 서비스 운영 | [playball.one](https://playball.one/) |
| 프로젝트 Docs | [docs.playball.one](https://docs.playball.one/planning/overview) |
| WBS | [Google Sheets](https://docs.google.com/spreadsheets/d/1axI4Ynw2SzQXTDA9udrfF_bvB5g_KfeJ8SLNYf0UtSE/edit?usp=sharing) |
| QA 시트 | [Google Sheets](https://docs.google.com/spreadsheets/d/13FvuJ2gUbNocKz6nOm2aRKB3uSNBFATWn5DN3W8KRUY/edit?gid=0#gid=0) |
| 구름공방 노션 | [Notion](https://www.notion.so/4-2e79e3e335cc8032979bc17eb053e93a?source=copy_link) |
| 개발팀 기술 문서 · 그라운드 룰 · 명세 · 작업 계획서 | [Notion](https://www.notion.so/2ed9e3e335cc801ea7bae1cf7378e707?source=copy_link) |

---

## Team

### 팀 리더십

|                           팀장 · 백엔드 리드 · 개발 그룹장                            |                            부팀장 · PM 파트장                            |
|:---------------------------------------------------------------------:|:------------------------------------------------------------------:|
|       <img height="200" src="https://github.com/Seulgi0117.png" />        |       <img height="200" src="https://github.com/j-hyunn.png" />        |
|       강슬기 <br/> [@Seulgi0117](https://github.com/Seulgi0117)        |         김제현 <br/> [@j-hyunn](https://github.com/j-hyunn)          |

### Backend (BE) · Fullstack (FS)

|              JWT 인증 / 인가 · 추천 좌석 알고리즘 · 티켓팅 서버 구현              |                       티켓팅 · 주문 서버 구현                        |                    대기열 서버 구현 · OAuth 연동 · 시큐어코딩                    |
|:-------------------------------------------------------------:|:---------------------------------------------------------------:|:------------------------------------------------------------------:|
|  <img height="200" src="https://github.com/Seulgi0117.png" />   |     <img height="200" src="https://github.com/ejinn1.png" />      |       <img height="200" src="https://github.com/siyeon13.png" />       |
|  강슬기 <br/> [@Seulgi0117](https://github.com/Seulgi0117)   |       유의진 <br/> [@ejinn1](https://github.com/ejinn1)       |         황시연 <br/> [@siyeon13](https://github.com/siyeon13)         |

### Frontend (FE)

|                                   프론트엔드 총괄                                    |
|:---------------------------------------------------------------------:|
|        <img height="200" src="https://github.com/806hyogi.png" />         |
|         최광혁 <br/> [@806hyogi](https://github.com/806hyogi)         |

### Cloud Native (CN)

|                      클라우드보안 그룹장 · CN 파트장                       |                        CI / CD 파이프라인 (TeamCity)                         |                         모니터링 · 시각화 · 알림 체계 (Grafana)                         |
|:---------------------------------------------------------------:|:----------------------------------------------------------------------:|:------------------------------------------------------------------------:|
|     <img height="200" src="https://github.com/212clab.png" />     |       <img height="200" src="https://github.com/Chemuchi.png" />       |         <img height="200" src="https://github.com/jjyeah.png" />         |
|      이원이 <br/> [@212clab](https://github.com/212clab)      |        정재형 <br/> [@Chemuchi](https://github.com/Chemuchi)         |          정지혜 <br/> [@jjyeah](https://github.com/jjyeah)           |

### Security (SC)

|                        보안 파트장 · 모의 해킹 · 보안 총괄                        |                        보안 인프라 · 운영 보안                         |                               인증 / 인가 · 앱단 보안                                |
|:---------------------------------------------------------------:|:-----------------------------------------------------------:|:----------------------------------------------------------------------------:|
|    <img height="200" src="https://github.com/hapumpum.png" />     |   <img height="200" src="https://github.com/hej-13.png" />    |      <img height="200" src="https://github.com/vanillacustardcream.png" />       |
|     정민욱 <br/> [@hapumpum](https://github.com/hapumpum)      |    정완우 <br/> [@hej-13](https://github.com/hej-13)     | 안지서 <br/> [@vanillacustardcream](https://github.com/vanillacustardcream)  |

### AI

|                         AI 파트장 · AI 공격 에이전트 구현                         |                               AI 방어 서버 구축                               |
|:---------------------------------------------------------------------:|:---------------------------------------------------------------------:|
|        <img height="200" src="https://github.com/wkdwlgus.png" />         |       <img height="200" src="https://github.com/DDong-Gosu.png" />        |
|         장지현 <br/> [@wkdwlgus](https://github.com/wkdwlgus)         |        최동훈 <br/> [@DDong-Gosu](https://github.com/DDong-Gosu)         |

### Product Design (PD)

| 이름 | 담당 |
| --- | --- |
| 박세영 | 프로덕트 디자인 그룹장 |
| 윤정빈 | 디자인 파트장 |

---

<sub>© 2026 구름공방 </sub>
