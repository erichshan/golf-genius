# Golf Genius 연동 문서

이 문서는 Smartscore와 Golf Genius 간의 연동을 위한 설계 문서 및 API 레퍼런스를 포함합니다.

---

## 문서 구조

```
docs/
├── README.md              # 현재 문서
├── design/                # 설계/기획 문서
│   ├── index.md           # 연동 개요 및 핵심 결론
│   ├── terminology.md     # 용어 정의
│   ├── data-model.md      # 데이터 모델
│   ├── architecture.md    # 시스템 아키텍처
│   ├── sync-strategy.md   # 동기화 전략
│   └── implementation-guide.md  # 구현 가이드
└── api/                   # API 레퍼런스 및 도구
    ├── README.md          # API 사용 가이드
    ├── golfgeniusapiv2.apib     # 원본 API Blueprint
    ├── openapi/           # OpenAPI 3.0 스펙
    ├── http/              # IntelliJ HTTP Client
    └── postman/           # Postman Collection
```

---

## 빠른 시작

### 1. 설계 문서 읽기

Golf Genius 연동의 전체 그림을 이해하려면:

1. [설계 개요](./design/index.md) - 핵심 결론 및 문서 구성
2. [용어 정의](./design/terminology.md) - Golf Genius 도메인 용어
3. [데이터 모델](./design/data-model.md) - ERD 및 엔티티 관계
4. [아키텍처](./design/architecture.md) - 시스템 설계
5. [동기화 전략](./design/sync-strategy.md) - Polling vs Webhook
6. [구현 가이드](./design/implementation-guide.md) - 실전 구현 단계

### 2. API 테스트

API를 직접 테스트하려면:

- [API 사용 가이드](./api/README.md) - 도구별 설정 방법

---

## 핵심 결론 요약

### 아키텍처
- **Polling-first 구조** 권장
- 외부 연동은 API pull, 내부 전달은 이벤트 버스/캐시/실시간 push

### 데이터 모델
- `League → Season → Event → Round` 계층 구조
- `Player`와 `Roster Entry` 분리 필수

### 구현 우선순위
1. Event
2. Roster
3. Round
4. Score / Leaderboard
5. Pairing
6. Season Points

---

## 참고 자료

- [Golf Genius 공식 사이트](https://www.golfgenius.com)
- [Golf Genius API v2 Docs](https://www.golfgenius.com/api/v2/docs)
