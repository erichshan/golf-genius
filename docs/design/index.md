# Golf Genius 연동 문서 개요

작성일: 2026-03-07
최종 수정일: 2026-03-08

---

## 문서 목적

이 문서 시리즈는 Smartscore와 Golf Genius를 연동하여 **골프 라운드 스코어를 실시간 또는 준실시간으로 동기화**하려는 관점에서 작성되었다.

### 다루는 내용

1. Golf Genius 데이터 모델 및 용어 관계 정리
2. Smartscore 연동용 시스템 아키텍처 설계
3. Webhook vs Polling 구조 분석 및 적용 방식
4. Golf Genius API Endpoint 정리 (공개 접근 가능한 범위 중심)
5. 실전 구현 가이드
6. GG Tee Sheet 연동 설계 (ERP Only, GG Only, ERP + GG)

---

## 핵심 결론 (한눈에 보기)

### 데이터 모델 결론

- Golf Genius는 대회/리그 운영 플랫폼이며, **Event / League / Season / Roster / Round / Live Scoring / Leaderboard**가 핵심 축이다.
- **League와 Event는 동일 레벨**이다. League는 `event_type="league"`인 Event이다.
- `Roster`는 단순한 사람(Person) 개념이 아니라, **특정 Event에 참가한 Player의 이벤트 단위 참여 레코드**다.
- Season은 선택적 그룹핑이며, `Category → Event/League → Season(선택) → Round` 구조이다.
- **Division**은 Event 내 참가자 그룹 분류 (남자부/여자부 등), **Team**은 Division 내 팀 경기 그룹

### DB 스키마 결론

- 기존 SS 테이블(`ss_club`, `ss_member`)을 유지하고 **별도 연동 테이블** 생성
- `ss_club_golf_genius`: 골프장별 GG API Key, Webhook 설정
- `ss_member_golf_genius`: SS 회원 ↔ GG Member 매핑 (골프장별)
- `ss_gg_event`, `ss_gg_event_roster`, `ss_gg_round`: GG 데이터 동기화 및 매칭 결과 저장
- **회원 매칭 전략**: GG Event Roster 기준으로 이메일/전화번호로 SS 회원 매칭

### 데이터 흐름 결론

- **양방향 연동 구조**가 필요하다.
  - **Inbound (GG → SS)**: Hybrid (Polling + Webhook)
  - **Outbound (SS → GG)**: 스코어 데이터 실시간 Push
- 스코어 입력은 **Smartscore Tablet (카트 설치)**에서 이루어지며, Smartscore Server를 통해 Golf Genius로 전송된다.
- 라운드 종료 시 전체 스코어 재전송으로 정합성을 보장한다.

### 아키텍처 결론

- **Inbound (마스터 데이터)**: Hybrid 전략 (Polling으로 Event 감지 + API로 Webhook 자동 설정)
- **Outbound (스코어 데이터)**: Smartscore Tablet → Smartscore Server → Golf Genius로 스코어를 Push한다.
- 내부 전달은 이벤트 버스/캐시/실시간 push를 사용한다.
- 데이터 모델은 `Player`와 `Roster Entry`를 반드시 분리해야 한다.
- Webhook은 10가지 종류 지원: courses, pairings, players, scores, settings, matches, match_results, team_results, teams, event_roster_members

### Webhook 연동 결론

- Golf Genius는 **10가지 Webhook을 지원**하며, API를 통해 자동 설정 가능하다.
- 골프장 관리자가 수동으로 Webhook을 설정할 필요 없음 (Smartscore가 자동 처리)
- **사전 조건**: Webhook Integration 기능이 Customer에 활성화되어 있어야 함

### 구현 우선순위 결론

1. Event
2. Roster
3. Round
4. Score / Leaderboard
5. Pairing
6. Season Points

### 운영 결론

- raw 저장, hash 비교, 멱등 upsert, DLQ, 재동기화 배치는 필수다.
- **Hybrid 전략**: Polling으로 Event 감지 + Webhook으로 실시간 변경 수신
- 진행 중 Event는 1시간마다 Webhook 재설정 (GG Admin 수동 변경 대응)

### Tee Sheet 연동 결론

- **TeeSheetAdapter 패턴**으로 ERP/GG 데이터 소스를 추상화
- 3가지 시나리오 지원: ERP Only, GG Only, ERP + GG
- ERP + GG 시 **GG 우선 전략** 적용 (설정으로 변경 가능)
- 스코어는 설정에 따라 ERP, GG, 또는 양쪽 모두에 전송
- 기존 `CInterlock{ErpType}` 패턴 확장으로 `CInterlockGolfGenius` 구현

### 확인 필요 사항

Golf Genius 측에 다음을 확인해야 한다:
1. API Rate Limit 상세 (요청 수, 적용 단위)
2. Webhook Integration 기능 일괄 활성화 방법
3. Webhook payload 상세 스펙
4. **`players` webhook 범위**: Event 참가자만 해당하는지, Master Roster 전체 변경도 포함하는지

---

## 문서 구성

| 파일 | 내용 |
|------|------|
| [terminology.md](./terminology.md) | 용어 정의 및 관계 설명 |
| [data-model.md](./data-model.md) | 데이터 모델, ERD, 시퀀스 다이어그램 |
| [architecture.md](./architecture.md) | 시스템 아키텍처 설계 |
| [sync-strategy.md](./sync-strategy.md) | 동기화 전략 (Webhook vs Polling) |
| [implementation-guide.md](./implementation-guide.md) | 실전 구현 가이드 |
| [teesheet-integration-design.md](./teesheet-integration-design.md) | **GG Tee Sheet 연동 설계** (ERP Only, GG Only, ERP+GG) |
| [API Reference](../api/README.md) | API 문서 (OpenAPI, Postman, HTTP Client) |

---

## 중요 참고사항

> **중요**
> Golf Genius API 문서 페이지는 Golf Genius의 공개 제품/리그 소개 페이지를 함께 대조하여 분석했다.
> 따라서 이 문서의 내용은 **공개 문서에서 확인된 사실**과 **도메인 모델 관점에서의 합리적 해석**을 구분해 설명한다.

---

## 공개 자료 출처

1. Golf Genius API v2 Docs: https://www.golfgenius.com/api/v2/docs
2. Golf Genius League product page: https://www.golfgenius.com/products/tm/what/leagues
3. Golf Genius Tournament Management overview: https://golfgenius.com/products/tm
4. Golf Genius product updates / release notes: https://golfgenius.com/products/tm/product-updates
5. Golf Genius public live/event pages indexed by search results
6. Golf Genius Webinars / Knowledge Base
7. Golf Genius Guides 페이지
8. TM Certification 페이지

---

[다음: 용어 정의 →](./terminology.md)
