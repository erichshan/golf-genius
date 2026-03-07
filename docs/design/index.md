# Golf Genius 연동 문서 개요

작성일: 2026-03-07

---

## 문서 목적

이 문서 시리즈는 Golf Genius와 Smartscore를 연동하여 **골프 라운드 스코어를 실시간 또는 준실시간으로 동기화**하려는 관점에서 작성되었다.

### 다루는 내용

1. Golf Genius 데이터 모델 및 용어 관계 정리
2. Smartscore 연동용 시스템 아키텍처 제안
3. Webhook vs Polling 구조 분석 및 권장안
4. Golf Genius API Endpoint 정리 (공개 접근 가능한 범위 중심)
5. 실전 구현 가이드

---

## 핵심 결론 (한눈에 보기)

### 데이터 모델 결론

- Golf Genius는 대회/리그 운영 플랫폼이며, **Event / League / Season / Roster / Round / Live Scoring / Leaderboard**가 핵심 축이다.
- `Roster`는 단순한 사람(Person) 개념이 아니라, **특정 Event에 참가한 Player의 이벤트 단위 참여 레코드**다.
- 운영 관점에서는 `League → Season → Event → Round` 구조 위에 `Roster → Player`가 연결된다.

### 아키텍처 결론

- **현재 기준 최적안은 Polling-first 구조**다.
- 외부 연동은 API pull, 내부 전달은 이벤트 버스/캐시/실시간 push를 사용한다.
- 데이터 모델은 `Player`와 `Roster Entry`를 반드시 분리해야 한다.
- 리그 서비스라면 `League → Season → Event → Round`를 표준 계층으로 설계하는 것이 향후 확장에 유리하다.

### 구현 우선순위 결론

1. Event
2. Roster
3. Round
4. Score / Leaderboard
5. Pairing
6. Season Points

### 운영 결론

- raw 저장, hash 비교, 멱등 upsert, DLQ, 재동기화 배치는 필수다.
- webhook이 확인되더라도 polling을 완전히 버리기보다 **보조 트리거**로 쓰는 것이 안정적이다.

---

## 문서 구성

| 파일 | 내용 |
|------|------|
| [terminology.md](./terminology.md) | 용어 정의 및 관계 설명 |
| [data-model.md](./data-model.md) | 데이터 모델, ERD, 시퀀스 다이어그램 |
| [architecture.md](./architecture.md) | 시스템 아키텍처 설계 |
| [sync-strategy.md](./sync-strategy.md) | 동기화 전략 (Webhook vs Polling) |
| [implementation-guide.md](./implementation-guide.md) | 실전 구현 가이드 |
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
