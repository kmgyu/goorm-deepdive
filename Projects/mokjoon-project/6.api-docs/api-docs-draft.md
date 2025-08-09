
형식 REST + JSON, 버전 프리픽스(/api/v1)
표준 페이징·오류 규격 포함.

# API Docs

## 개요·컨벤션

- Base URL: /api/v1
    
- Media type: application/json; charset=utf-8
    
- Versioning: URL 프리픽스(/v1). 호환 깨질 변경 시 /v2 추가
    
- 시간: ISO 8601(UTC), 예: 2025-08-08T12:34:56Z
    
- 페이징: page(0..), size(1..100), sort=field,asc|desc
    
- 공통 쿼리: orgId(학교/단체 식별자, 기본값: 목포대), from, to(기간)
    
- 인증: 현재 없음. ADMIN 영역은 추후 토큰 기반 보호 예정
    
- 에러 포맷
    
    ```
    {
      "timestamp": "2025-08-08T12:34:56Z",
      "status": 404,
      "error": "Not Found",
      "code": "USER_NOT_FOUND",
      "message": "User 'alice' not found",
      "traceId": "b9b8f7..."
    }
    ```
    

## 스키마 요약

- User
    
    ```
    {
      "handle": "alice",
      "orgId": "mokpo-univ",
      "bojProfileUrl": "https://www.acmicpc.net/user/alice",
      "solvedAcLinked": true,
      "tier": 13,
      "solvedCount": 1024,
      "lastSyncedAt": "2025-08-08T00:00:00Z"
    }
    ```
    
- RankingEntry
    
    ```
    { "rank": 1, "handle": "alice", "delta": +2, "score": 1024 }
    ```
    
- RankingHistoryItem
    
    ```
    { "date": "2025-08-01", "rank": 3, "score": 980 }
    ```
    
- Problem
    
    ```
    {
      "problemId": 1000,
      "title": "A+B",
      "difficulty": 1,
      "tags": ["implementation"],
      "acceptedCount": 123456
    }
    ```
    
- Submission
    
    ```
    {
      "submissionId": 12345678,
      "problemId": 1000,
      "handle": "alice",
      "verdict": "AC",
      "language": "Java 11",
      "submittedAt": "2025-08-07T11:22:33Z",
      "tags": ["implementation"]
    }
    ```
    
- TagStat
    
    ```
    { "tag": "graph", "solvedCount": 42 }
    ```
    
- UpdateJob
    
    ```
    {
      "jobId": "upd_01H...",
      "type": "BAEKJOON_RANKINGS",
      "scope": { "orgId": "mokpo-univ" },
      "status": "RUNNING",   // QUEUED|RUNNING|SUCCEEDED|FAILED
      "startedAt": "2025-08-08T12:00:00Z",
      "finishedAt": null,
      "summary": null
    }
    ```
    

## 엔드포인트

### Rankings

1. 학교 랭킹 조회  
    GET /api/v1/rankings/school  
    query: orgId, page, size, sort=score,desc
    

```
200
{
  "content": [ { "rank":1,"handle":"alice","delta":+2,"score":1024 }, ... ],
  "page": 0, "size": 50, "totalElements": 812, "totalPages": 17
}
```

2. 개인 랭킹 변화 히스토리  
    GET /api/v1/rankings/users/{handle}/history  
    query: from, to
    

```
200
{ "handle":"alice", "history":[ { "date":"2025-08-01","rank":3,"score":980 }, ... ] }
```

3. 태그 랭킹(태그별 상위 해결자 수)  
    GET /api/v1/rankings/tags  
    query: orgId, tag(optional), page, size
    

```
200
{
  "content": [ { "tag":"graph", "handle":"alice", "solvedCount":42 }, ... ],
  "page":0,"size":50,"totalElements":1200,"totalPages":24
}
```

### Users

1. 사용자 검색  
    GET /api/v1/users  
    query: q(부분 일치), orgId, page, size
    

```
200
{
  "content":[
    { "handle":"alice","orgId":"mokpo-univ","solvedAcLinked":true,"tier":13,"solvedCount":1024 }
  ],
  "page":0,"size":20,"totalElements":1,"totalPages":1
}
```

2. 사용자 상세  
    GET /api/v1/users/{handle}
    

```
200
{
  "handle":"alice",
  "orgId":"mokpo-univ",
  "bojProfileUrl":"https://www.acmicpc.net/user/alice",
  "solvedAcLinked":true,
  "tier":13,
  "solvedCount":1024,
  "lastSyncedAt":"2025-08-08T00:00:00Z"
}
```

3. 사용자 제출 기록  
    GET /api/v1/users/{handle}/submissions  
    query: from, to, verdict(AC|WA|...), problemId, page, size, sort=submittedAt,desc
    

```
200
{
  "content":[
    { "submissionId":123,"problemId":1000,"handle":"alice","verdict":"AC","language":"Java 17","submittedAt":"2025-08-07T11:22:33Z","tags":["implementation"] }
  ],
  "page":0,"size":50,"totalElements":523,"totalPages":11
}
```

4. 사용자 태그 통계  
    GET /api/v1/users/{handle}/tags  
    query: top(기본 20)
    

```
200
{ "handle":"alice", "stats":[ { "tag":"graph","solvedCount":42 }, { "tag":"dp","solvedCount":35 } ] }
```

### Problems

1. 문제 검색  
    GET /api/v1/problems  
    query: q, tag, difficultyMin, difficultyMax, page, size, sort=difficulty,asc
    

```
200
{
  "content":[
    { "problemId":1000,"title":"A+B","difficulty":1,"tags":["implementation"],"acceptedCount":123456 }
  ],
  "page":0,"size":20,"totalElements":23000,"totalPages":1150
}
```

2. 문제 상세  
    GET /api/v1/problems/{problemId}
    

```
200
{
  "problemId":1000,
  "title":"A+B",
  "difficulty":1,
  "tags":["implementation"],
  "acceptedCount":123456,
  "firstSeenAt":"2009-01-01T00:00:00Z",
  "lastSyncedAt":"2025-08-08T00:00:00Z"
}
```

### Stats

1. 요약 통계(일/주/월)  
    GET /api/v1/stats/summary  
    query: orgId, window=day|week|month, from, to
    

```
200
{
  "orgId":"mokpo-univ",
  "window":"day",
  "buckets":[
    { "date":"2025-08-01","usersActive":120,"problemsSolved":540 },
    { "date":"2025-08-02","usersActive":98,"problemsSolved":420 }
  ]
}
```

2. 태그 트렌드  
    GET /api/v1/stats/tags  
    query: orgId, tags=graph,dp, from, to
    

```
200
{
  "orgId":"mokpo-univ",
  "series":[
    { "tag":"graph", "points":[ { "date":"2025-08-01","count":42 }, ... ] },
    { "tag":"dp", "points":[ { "date":"2025-08-01","count":35 }, ... ] }
  ]
}
```

3. 풀이 속도/활동량(개인)  
    GET /api/v1/stats/velocity  
    query: handle, from, to
    

```
200
{
  "handle":"alice",
  "daily":[ { "date":"2025-08-01","problemsSolved":12 }, ... ],
  "streak": { "current": 7, "best": 19 }
}
```

### Admin (보호 대상)

1. 즉시 갱신 실행(비동기 잡)  
    POST /api/v1/admin/update/run
    

```
request:
{
  "type": "BAEKJOON_RANKINGS",   // BAEKJOON_RANKINGS|USER_PROFILES|SUBMISSIONS|PROBLEMS|ALL
  "scope": { "orgId":"mokpo-univ", "handles":["alice","bob"] }
}
202
{ "jobId":"upd_01H...", "status":"QUEUED" }
```

2. 갱신 스케줄 설정  
    PUT /api/v1/admin/update/schedule
    

```
request: { "cron": "0 0 3 * * *", "enabled": true }
200     : { "cron":"0 0 3 * * *","enabled":true,"nextRunAt":"2025-08-09T03:00:00Z" }
```

3. 갱신 히스토리 조회  
    GET /api/v1/admin/update/history  
    query: type, status, page, size
    

```
200
{
  "content":[
    { "jobId":"upd_01H...","type":"BAEKJOON_RANKINGS","status":"SUCCEEDED","startedAt":"2025-08-08T03:00:00Z","finishedAt":"2025-08-08T03:02:10Z","summary":{"rows":812} }
  ],
  "page":0,"size":20,"totalElements":35,"totalPages":2
}
```

4. 헬스체크  
    GET /api/v1/admin/health
    

```
200
{
  "status":"UP",
  "db":"UP",
  "crawler":"UP",
  "lastRankingsSync":"2025-08-08T03:02:10Z"
}
```

## 상태 코드 규칙

- 200 OK: 조회 성공
    
- 201 Created: 리소스 생성
    
- 202 Accepted: 비동기 작업 수락
    
- 400 Bad Request: 유효성 오류
    
- 401 Unauthorized / 403 Forbidden: 인증/인가 실패(ADMIN)
    
- 404 Not Found: 대상 없음
    
- 409 Conflict: 중복/경합
    
- 429 Too Many Requests: 레이트 리밋
    
- 500 Internal Server Error
    

## 레이트 리밋·캐시

- 클라이언트 조회: 기본 60 req/min (429 반환, Retry-After 헤더)
    
- 캐싱 헤더
    
    - 정적 메타(문제 상세): Cache-Control: public, max-age=3600
        
    - 랭킹/통계: public, max-age=300
        
    - 개인 히스토리: private, max-age=60
        

## 예시 cURL

```
curl -s 'https://host/api/v1/rankings/school?orgId=mokpo-univ&page=0&size=50'
curl -s 'https://host/api/v1/users/alice/tags'
curl -s -X POST 'https://host/api/v1/admin/update/run' \
  -H 'Content-Type: application/json' \
  -d '{ "type":"ALL", "scope": { "orgId": "mokpo-univ" } }'
```

## 필드 유효성·제약

- handle: ^[a-z0-9_]{2,20}$
    
- orgId: 슬러그 형식, 소문자·하이픈
    
- problemId: 정수(BOJ 문제 번호)
    
- from/to: from ≤ to
    
- page: 0 이상, size: 1..100
    

## 추후 확장 가이드

- 멀티 테넌시 경로 지원: /api/v1/orgs/{orgId}/...
    
- 웹훅 추가: 데이터 갱신 완료 알림 POST
    
- 필터 확장: 난이도 구간, 언어, 태그 논리연산(and/or)
    
