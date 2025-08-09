### GraphQL 소개

GraphQL은 2012년 Facebook(현 Meta)에서 개발하여 2015년에 공개한 **데이터 쿼리 언어**입니다. 클라이언트가 필요한 데이터의 구조를 지정하면, 서버는 그 구조에 정확히 맞춰 데이터를 반환하는 **타입 시스템 기반의 서버사이드 런타임**이죠. 이 방식을 통해 REST의 단점인 불필요한 데이터를 받거나 필요한 데이터를 받지 못하는 문제를 해결합니다.

GraphQL의 주요 클라이언트로는 아폴로 클라이언트와 Relay가 있으며, 자바스크립트, 파이썬, 루비, 자바 등 다양한 언어로 서버를 구현할 수 있습니다.

---

### 왜 등장했나: 기존 API 이슈

1. **과다/과소 요청 문제** REST에서는 하나의 화면 데이터를 위해 여러 엔드포인트를 호출하거나(n+1 fetch), 하나의 엔드포인트가 너무 많은 필드를 반환해 낭비가 발생하는 문제(over-fetching/under-fetching)가 있었습니다.
    
2. **버전 관리의 어려움** 엔드포인트 버저닝(v1, v2)이 누적되면 클라이언트 파편화와 배포 동기화 비용이 증가합니다.
    
3. **다채널·다단 해상도 대응의 비효율** 웹, 모바일, 워치 등 다양한 기기에서 각기 다른 데이터가 필요하지만, REST의 응답은 일률적이어서 맞춤형 전달이 어렵습니다.
    
4. **스키마/문서화 부재** 엔드포인트별 응답 구조가 명확히 정의되어 있지 않거나 문서가 산재하는 경우가 많았습니다.
    

---

### REST와의 간단 비교

GraphQL과 REST의 주요 차이점은 다음과 같습니다.

|구분|REST|GraphQL|
|---|---|---|
|**엔드포인트**|여러 개의 엔드포인트|**하나의 엔드포인트** (게이트웨이/페더레이션 환경에서는 여러 개 사용 가능)|
|**요청**|엔드포인트에 정해진 응답을 받음|쿼리 문서에 따라 원하는 데이터만 응답 받음|
|**데이터 모델**|리소스 중심 (`/users/123`)|**그래프(관계) 중심** (`user(id: 123) { posts { ... } }`)|
|**버전 관리**|경로/헤더를 통한 버저닝이 흔함|필드를 추가하는 비파괴적 확장으로 **무버전** 진화|
|**캐시**|HTTP 캐시를 활용하기 직관적|클라이언트/필드 레벨 캐시 전략 설계 필요|

---

### 핵심 개념과 기능

1. **타입 기반 스키마 (Schema & SDL)** **스키마 정의 언어(SDL)**로 쿼리, 뮤테이션, 객체 타입, 필드, 관계를 명시하여 API의 구조를 정의합니다.
    
2. **선언적 데이터 가져오기 (Queries)** 클라이언트가 필요한 필드만 선택해 한 번의 요청으로 계층형 데이터를 가져올 수 있습니다.
    
3. **서버 측 데이터 제어 (Resolvers)** 필드 단위로 데이터 조회 로직을 구현하는 기능입니다. 데이터베이스, REST API, gRPC 등 다양한 소스를 통합할 수 있습니다.
    
4. **데이터 변경 (Mutations)** 리소스의 생성, 수정, 삭제를 명시적 타입으로 정의합니다.
    
5. **실시간 (Subscriptions)** 웹소켓 등을 사용해 서버에서 발생하는 이벤트를 실시간 스트림으로 클라이언트에 전달합니다.
    
6. **강력한 개발자 도구 (Introspection)** 스키마 자체를 질의하여 자동 문서화, IDE의 자동 완성, 타입 검사 등의 기능을 활용할 수 있습니다.
    

---

### 장점

1. **필요한 것만 받는다** 클라이언트가 원하는 데이터만 요청하므로 **Over/Under-fetching**을 해소하고, 네트워크 요청 횟수와 페이로드 크기를 줄입니다. 모바일 환경에 특히 최적화되어 있습니다.
    
2. **유연한 API 진화** 스키마에 필드를 추가하는 방식으로 **버전 없이 API를 확장**할 수 있어, 클라이언트와의 동기화 비용이 줄어듭니다.
    
3. **강한 타입과 자가 문서화** 스키마 기반으로 타입 안정성이 보장되며, Introspection 기능을 통해 API 탐색성과 테스트 용이성이 높아집니다.
    
4. **효율적인 데이터 집계 및 조인** 연관된 여러 리소스를 한 번의 질의로 계층적으로 가져올 수 있어, 여러 번의 REST 요청을 줄입니다.
    
5. **프론트엔드와 백엔드 개발자 부담 완화** API의 요청/응답 형태에 대한 의존도가 낮아져, 프론트엔드 개발자가 백엔드 개발자의 도움 없이도 필요한 데이터를 가져올 수 있습니다.
    

---

### 단점과 고려사항

#### 개발·운영

- **서버 복잡도 증가:** 필드 단위 리졸버, 캐시, 권한 등을 설계해야 하므로 서버의 복잡도가 높아집니다.
    
- **캐싱 전략 재설계:** URL 기반의 REST 캐시 전략을 그대로 사용하기 어려우므로, 클라이언트/필드 레벨 캐시를 설계해야 합니다.
    
- **단순 CRUD에는 과도:** 단순한 요청/응답만 필요하다면 REST가 더 효율적일 수 있습니다.
    

#### 성능

- **N+1 쿼리:** 필드 리졸버가 반복적으로 외부 소스를 호출하면 N+1 쿼리 문제가 발생하기 쉽습니다. **DataLoader**를 사용해 배칭(Batching)으로 해결할 수 있습니다.
    
- **권한 부여로 인한 추가 I/O:** 필드별 권한 검사가 또 다른 I/O를 유발할 수 있습니다.
    

#### 보안

- **공격 표면 확대:** 쿼리 언어와 스키마가 노출될 수 있습니다. 프로덕션 환경에서는 인트로스펙션을 제한하는 것이 좋습니다.
    
- **필드 레벨 권한의 난이도:** 엔드포인트 단위보다 필드 단위 권한 검사가 더 복잡합니다.
    

---

### 스키마와 질의 예시

**스키마(SDL):**

GraphQL

```
schema {
  query: Query
  mutation: Mutation
}

type Query {
  me: User
  post(id: ID!): Post
  posts(limit: Int = 10, cursor: ID): PostConnection!
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
}

type PostConnection {
  edges: [Post!]!
  pageInfo: PageInfo!
}

type PageInfo {
  endCursor: ID
  hasNextPage: Boolean!
}

input CreatePostInput {
  title: String!
  body: String!
  tags: [String!] = []
}

type User {
  id: ID!
  name: String!
  email: String!
  posts(limit: Int = 5): [Post!]!
}

type Post {
  id: ID!
  title: String!
  body: String!
  author: User!
  tags: [String!]!
}
```

**클라이언트 질의:**

GraphQL

```
query Home($limit: Int!) {
  me {
    id
    name
  }
  posts(limit: $limit) {
    edges {
      id
      title
      author { id name }
      tags
    }
    pageInfo { endCursor hasNextPage }
  }
}
```

**응답(요청한 필드만):**

JSON

```
{
  "data": {
    "me": { "id": "u1", "name": "민규" },
    "posts": {
      "edges": [
        {
          "id": "p10",
          "title": "GraphQL 도입기",
          "author": { "id": "u2", "name": "Alice" },
          "tags": ["api", "graphql"]
        }
      ],
      "pageInfo": { "endCursor": "p10", "hasNextPage": true }
    }
  }
}
```

---

### 언제 적합한가

- 여러 뷰(웹/모바일/위젯)에서 서로 다른 데이터 조합이 필요한 경우
    
- 네트워크 회수를 줄여야 하는 모바일 환경
    
- 프론트엔드 팀이 데이터 형태를 자율적으로 실험/진화해야 하는 조직
    

### 도입 팁

- **스키마 우선 설계:** 도메인 언어로 SDL부터 합의하는 것이 중요합니다.
    
- **DataLoader/배칭:** N+1 쿼리 방지를 위해 기본적으로 탑재하는 것을 권장합니다.
    
- **점진적 도입:** 백엔드 포 프론트엔드(BFF) 또는 REST 위에 GraphQL을 얹는 방식으로 시작해 리스크를 줄일 수 있습니다.
    

---

### 한 줄 요약

GraphQL은 타입 안전한 스키마를 기반으로 클라이언트가 필요한 데이터만 한 번에 가져오게 해, 과다/과소 요청과 엔드포인트 난립 문제를 해결하는 선언적 API 모델입니다.




### 참고자료
[간단정리] GraphQL이란? (REST api와 차이점)
https://hahahoho5915.tistory.com/63

graphql 공식 문서 - 모범 사례
https://graphql-kr.github.io/learn/best-practices/

graphql을 그만둔 이유
https://news.hada.io/topic?id=15097

위키백과 - GraphQL
https://ko.wikipedia.org/wiki/GraphQL

GraphQL-github Pages 공식문서
https://graphql-kr.github.io/