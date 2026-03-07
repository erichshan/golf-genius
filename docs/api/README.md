# Golf Genius API Reference

Golf Genius API v2에 대한 레퍼런스 문서 및 테스트 도구입니다.

---

## 파일 구조

```
api/
├── README.md                    # 현재 문서
├── golfgeniusapiv2.apib         # 원본 API Blueprint (참고용)
├── openapi/
│   ├── golfgenius-api-v2.yaml   # OpenAPI 3.0 스펙 (YAML)
│   └── golfgenius-api-v2.json   # OpenAPI 3.0 스펙 (JSON)
├── http/
│   ├── golfgenius-api.http      # IntelliJ HTTP Client 요청 파일
│   ├── http-client.env.json     # 환경 변수 (dev/prod)
│   └── http-client.private.env.json.example  # 프라이빗 환경 변수 예제
└── postman/
    └── golfgenius-api-v2.postman_collection.json  # Postman Collection
```

---

## 인증 방식

Golf Genius API는 두 가지 인증 방식을 사용합니다:

| 인증 방식 | 용도 | 예시 |
|-----------|------|------|
| URL Path Parameter | 조회 API (GET) | `/api_v2/{api_key}/events` |
| Bearer Token | 쓰기 API (POST/PUT/DELETE) | `Authorization: Bearer {api_key}` |

---

## 도구별 사용법

### 1. SwaggerUI / Swagger Editor

OpenAPI 스펙 파일을 로드하여 API를 탐색할 수 있습니다.

```bash
# 온라인 에디터 사용
# https://editor.swagger.io 에서 openapi/golfgenius-api-v2.yaml 파일 로드

# 또는 로컬 SwaggerUI 실행
docker run -p 8080:8080 -e SWAGGER_JSON=/api/golfgenius-api-v2.json \
  -v $(pwd)/openapi:/api swaggerapi/swagger-ui
```

### 2. IntelliJ IDEA / WebStorm HTTP Client

1. **프라이빗 환경 파일 생성**
   ```bash
   cd docs/api/http
   cp http-client.private.env.json.example http-client.private.env.json
   ```

2. **API 키 설정**
   ```json
   {
     "dev": {
       "api_key": "your-actual-api-key"
     }
   }
   ```

3. **요청 실행**
   - `golfgenius-api.http` 파일 열기
   - 환경 선택 (dev/prod)
   - 원하는 요청 옆의 ▶️ 클릭

### 3. Postman

1. **Collection Import**
   - Postman 열기
   - Import → File → `postman/golfgenius-api-v2.postman_collection.json`

2. **변수 설정**
   - Collection 선택 → Variables 탭
   - `api_key` 값 설정

3. **요청 실행**
   - 원하는 요청 선택 → Send

### 4. cURL

```bash
# 조회 API (GET) - API Key in URL
curl "https://www.golfgenius.com/api_v2/YOUR_API_KEY/seasons"

# 쓰기 API (POST) - Bearer Token
curl -X POST "https://www.golfgenius.com/api_v2/events" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Test Event", "event_type": "event"}'
```

---

## API 엔드포인트 개요

### 조회 API (GET - URL Path Parameter)

| 카테고리 | 엔드포인트 | 설명 |
|----------|-----------|------|
| Seasons | `/api_v2/{api_key}/seasons` | 시즌 목록 |
| Categories | `/api_v2/{api_key}/categories` | 카테고리 목록 |
| Directories | `/api_v2/{api_key}/directories` | 디렉토리 목록 |
| Master Roster | `/api_v2/{api_key}/master_roster` | 마스터 로스터 |
| Events | `/api_v2/{api_key}/events` | 이벤트 목록 |
| Rounds | `/api_v2/{api_key}/events/{event_id}/rounds` | 라운드 목록 |
| Courses | `/api_v2/{api_key}/events/{event_id}/courses` | 코스 및 티 정보 |
| Tee Sheet | `/api_v2/{api_key}/events/{event_id}/rounds/{round_id}/tee_sheet` | 티시트 및 스코어 |
| Tournaments | `/api_v2/{api_key}/events/{event_id}/rounds/{round_id}/tournaments/{id}.{format}` | 토너먼트 결과 |

### 쓰기 API (POST/PUT/DELETE - Bearer Token)

| 카테고리 | 메서드 | 엔드포인트 | 설명 |
|----------|--------|-----------|------|
| Events | POST | `/api_v2/events` | 이벤트 생성 |
| Events | PUT | `/api_v2/events/{event_id}` | 이벤트 수정 |
| Events | DELETE | `/api_v2/events/{event_id}` | 이벤트 삭제 |
| Rounds | POST | `/api_v2/events/{event_id}/rounds` | 라운드 생성 |
| Rounds | PUT/DELETE | `/api_v2/events/{event_id}/rounds/{round_id}` | 라운드 수정/삭제 |
| Divisions | POST | `/api_v2/events/{event_id}/divisions` | 디비전 생성 |
| Members | POST | `/api_v2/events/{event_id}/members` | 멤버 등록 |
| Pairing Groups | POST | `/api_v2/events/{event_id}/rounds/{round_id}/pairing_groups` | 페어링 그룹 생성 |

---

## 주의사항

1. **API 키 보안**
   - `http-client.private.env.json`은 `.gitignore`에 포함되어 있음
   - API 키를 코드에 하드코딩하지 않기

2. **Rate Limiting**
   - Golf Genius API는 rate limiting이 적용될 수 있음
   - 대량 요청 시 적절한 간격 유지

3. **ID 형식**
   - Golf Genius ID는 Numeric 22 또는 String 형식
   - API 응답의 `id`와 `id_str` 모두 지원

---

## 관련 문서

- [설계 문서](../design/index.md) - Golf Genius 연동 설계
- [원본 API Blueprint](./golfgeniusapiv2.apib) - Golf Genius 제공 원본 문서
