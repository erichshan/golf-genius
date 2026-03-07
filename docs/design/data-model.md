# Golf Genius 데이터 모델

[← 용어 정의](./terminology.md) | [다음: 시스템 아키텍처 →](./architecture.md)

---

## 계층 관계 다이어그램

```mermaid
flowchart TD
    A[Organization / Facility / Association\n운영 주체 - 문서상 직접 확정은 아님] --> B[League]
    B --> C[Season]
    C --> D[Event]
    D --> E[Round]
    D --> F[Roster]
    F --> G[Player]
```

> 상단의 Organization / Facility / Association 계층은 Golf Genius 공개 사이트에서
> "clubs, associations, resorts, tours"라는 운영 주체 설명에 기반한 상위 개념 해석이다.
> API 문서에 반드시 동일 명칭으로 존재한다고 단정할 수는 없다.

---

## League / Season / Event 계층 상세

```mermaid
flowchart TD
    L[League\n예: 2026 Smartscore Weekly League]
    S1[Season\n2026 Spring]
    S2[Season\n2026 Summer]
    E1[Event\nWeek 1]
    E2[Event\nWeek 2]
    E3[Event\nWeek 3]
    E4[Event\nSummer Opener]
    R1[Round 1]
    R2[Round 1]
    R3[Round 1]
    R4[Round 1 / Round 2]

    L --> S1
    L --> S2
    S1 --> E1
    S1 --> E2
    S1 --> E3
    S2 --> E4
    E1 --> R1
    E2 --> R2
    E3 --> R3
    E4 --> R4
```

---

## ERD (Entity Relationship Diagram)

### 기본 ERD

```mermaid
erDiagram
    PLAYER ||--o{ ROSTER_ENTRY : participates_in
    EVENT  ||--o{ ROSTER_ENTRY : has

    PLAYER {
      string player_id
      string name
      string email
    }

    EVENT {
      string event_id
      string event_name
      string status
    }

    ROSTER_ENTRY {
      string roster_entry_id
      string event_id
      string player_id
      string event_handicap
      string registration_status
      string division
      string team_id
    }
```

### 전체 ERD

```mermaid
erDiagram
    LEAGUE ||--o{ SEASON : contains
    SEASON ||--o{ EVENT : groups
    EVENT ||--|{ ROUND : has
    EVENT ||--|{ ROSTER_ENTRY : enrolls
    PLAYER ||--o{ ROSTER_ENTRY : participates_as
    ROUND ||--o{ SCORECARD : records
    ROSTER_ENTRY ||--o{ SCORECARD : owns
    EVENT ||--o{ PAIRING : schedules
    ROUND ||--o{ PAIRING : applies_to
    EVENT ||--o{ LEADERBOARD_SNAPSHOT : summarizes
    SEASON ||--o{ SEASON_POINT : accumulates

    LEAGUE {
      string league_id
      string name
      string type
    }
    SEASON {
      string season_id
      string league_id
      string name
      boolean current
    }
    EVENT {
      string event_id
      string season_id
      string name
      string status
      datetime start_at
      datetime end_at
    }
    ROUND {
      string round_id
      string event_id
      int round_no
      date play_date
      string status
    }
    PLAYER {
      string player_id
      string name
      string email
      string gender
      decimal handicap_index
    }
    ROSTER_ENTRY {
      string roster_entry_id
      string event_id
      string player_id
      string registration_status
      string division
      string tee
      decimal playing_handicap
    }
    SCORECARD {
      string scorecard_id
      string round_id
      string roster_entry_id
      int total_strokes
      int net_score
      string scoring_status
    }
    PAIRING {
      string pairing_id
      string round_id
      string event_id
      string tee_time
      string tee
    }
    LEADERBOARD_SNAPSHOT {
      string leaderboard_id
      string event_id
      datetime captured_at
      string scope
    }
    SEASON_POINT {
      string point_id
      string season_id
      string player_id
      decimal points
    }
```

---

## 운영 흐름 시퀀스 다이어그램

```mermaid
sequenceDiagram
    participant Admin as Golf Genius Admin
    participant GG as Golf Genius
    participant SS as Smartscore

    Admin->>GG: League 생성/운영
    Admin->>GG: Season 생성 또는 현재 시즌 지정
    Admin->>GG: Event 생성
    Admin->>GG: Roster 등록(참가자 편성)
    Admin->>GG: Round/Pairing/Tee Time 설정
    Player->>GG: 모바일/현장 입력으로 스코어 기록
    GG-->>SS: 실시간 점수/상태 전달(이상적 구조)
    SS->>SS: 이벤트/라운드/참가자 매핑 후 저장
    SS->>SS: 리더보드/알림/통계 반영
```

---

## 권장 내부 테이블 모델

### 마스터 테이블

| 테이블명 | 용도 |
|----------|------|
| `gg_league` | 리그 정보 |
| `gg_season` | 시즌 정보 |
| `gg_event` | 이벤트/대회 정보 |
| `gg_round` | 라운드 정보 |
| `gg_player` | 플레이어 마스터 |

### 관계/운영 테이블

| 테이블명 | 용도 |
|----------|------|
| `gg_event_participant` | Roster Entry 용 (이벤트 참가자) |
| `gg_round_pairing` | 조편성/티타임 |
| `gg_scorecard` | 스코어카드 |
| `gg_leaderboard_snapshot` | 리더보드 스냅샷 |
| `gg_season_point` | 시즌 포인트 |

### 기술 운영 테이블

| 테이블명 | 용도 |
|----------|------|
| `gg_sync_cursor` | 동기화 커서 관리 |
| `gg_sync_raw` | 원본 JSON 저장 |
| `gg_sync_job_history` | 동기화 작업 이력 |
| `gg_sync_error` | 동기화 오류 기록 |

### 추천 Player 클래스 다이어그램

```mermaid
classDiagram
    class Player {
      +playerId
      +firstName
      +lastName
      +email
      +gender
      +birthDate
      +defaultHandicap
    }
```

---

## 권장 식별 구조

스코어 데이터의 권장 식별 구조:

```text
league_id?
season_id?
event_id
round_id
roster_entry_id
hole_number
score_type
score_value
updated_at
```

---

## 핵심 관계 요약

- League는 여러 Season을 가진다.
- Season은 여러 Event를 가진다.
- Event는 하나 이상의 Round를 가진다.
- Event는 하나의 Roster를 가지며, Roster에는 여러 RosterEntry가 포함된다.
- 각 RosterEntry는 하나의 Player를 참조한다.
- Score는 Round + RosterEntry(또는 Player) 기준으로 기록된다.
- Leaderboard는 Event/Season/League 범위에서 집계된다.

---

[← 용어 정의](./terminology.md) | [다음: 시스템 아키텍처 →](./architecture.md)
