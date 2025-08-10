기술에 대한 소개보다 이 기술을 왜 사용하는 지에 대한 이해에 집중한 문서.
### 개요

GraphQL은 2012년 Facebook(현 Meta)에서 개발하고 2015년에 공개한 **데이터 쿼리 언어**입니다. 클라이언트가 필요한 데이터의 구조를 명시하면, 서버는 해당 구조에 정확히 맞춰 데이터를 반환하는 **타입 시스템 기반의 서버 사이드 런타임**입니다. 이 방식을 통해 REST API의 한계로 지적되던 불필요한 데이터를 받거나, 필요한 데이터를 모두 받기 위해 여러 번 요청해야 하는 문제를 해결합니다.

GraphQL의 주요 클라이언트는 Apollo Client와 Relay이며, 자바스크립트, 파이썬, 루비, 자바 등 다양한 언어로 서버를 구현할 수 있습니다.

> 정확하게는 GraphQL은 데이터 쿼리 언어이자 그 쿼리를 검증·실행하기 위한 명세에 가까우며, 실제 런타임은 각 구현체(graphql-java, graphql-js)가 제공합니다.
---
### GraphQL 등장 배경: 기존 Rest API의 한계

기존 REST API는 다음과 같은 문제로 인해 비효율성이 발생했습니다.

1. **Over-fetching & Under-fetching:** REST는 고정된 응답 구조를 가지므로, 클라이언트가 필요 이상으로 많은 필드를 받거나(Over-fetching), 필요한 데이터를 모두 얻기 위해 여러 엔드포인트를 호출해야 하는(Under-fetching) 문제가 빈번했습니다.
    
2. **버전 관리의 복잡성:** API 변경 시 `v1`, `v2`와 같은 버저닝이 누적되어 클라이언트 호환성 관리와 배포 동기화 비용이 증가했습니다.
    
3. **다양한 플랫폼 대응의 비효율:** 웹, 모바일, 워치 등 각기 다른 데이터 요구사항을 가진 클라이언트에 일률적인 REST 응답을 제공하여 맞춤형 데이터 전달이 어려웠습니다.
    
4. **불분명한 스키마:** 엔드포인트별 응답 구조가 명확히 정의되지 않거나 문서가 산재하는 경우가 많아 개발 효율성이 저하되었습니다.

---

### REST API와 GraphQL의 비교

| 구분         | REST API                  | GraphQL                              |
| ---------- | ------------------------- | ------------------------------------ |
| **엔드포인트**  | 여러 개의 엔드포인트               | **단일 엔드포인트**                         |
| **요청 방식**  | 엔드포인트에 정의된 응답 수신          | 쿼리 문서에 따라 원하는 데이터만 수신                |
| **데이터 모델** | **리소스 중심** (`/users/123`) | **그래프 중심** (`user(id: 123) { ... }`) |
| **버전 관리**  | URL/헤더 기반의 버저닝            | **무버전** 확장 (비파괴적 필드 추가)              |
| **캐싱**     | HTTP 캐시 활용이 직관적           | 클라이언트/필드 레벨 캐시 전략 필요                 |

그래프 중심 데이터 모델이란?
> GraphQL에서는 모든 데이터를 노드와 간선으로 봅니다.
> 노드 : 객체 타입(User, Post 등)
> 간선 : 필드가 다른 필드를 가리키는 참조
> 조인 연산 없이 직접적인 관계를 탐색합니다.
> 연결된 데이터 그래프를 클라이언트가 필요에 맞게 따라가며 뽑아오는 모델이라고 할 수 있습니다.

---

### GraphQL의 주요 개념 및 기능

1. **스키마 (Schema & SDL):** 스키마 정의 언어(SDL)를 사용하여 API의 구조를 명시적으로 정의합니다. 쿼리, 뮤테이션, 객체 타입, 필드, 관계 등이 포함됩니다.
“이 API에서 어떤 타입과 필드가 있고, 어떤 연산(Query/Mutation/Subscription)을 할 수 있는가”를 정의한다. 클라이언트–서버 사이의 인터페이스라고 볼 수 있음.
```graphql
schema {
  query: Query
  mutation: Mutation
}

type Query {
  user(id: ID!): User
  searchUsers(keyword: String, first: Int = 20, after: String): UserConnection!
}

type Mutation {
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
}

type User { id: ID!, name: String!, email: String @deprecated(reason: "Use contact.email") contact: Contact! }
type Contact { email: String!, phone: String }
input UpdateUserInput { id: ID!, name: String }
type UpdateUserPayload { ok: Boolean!, user: User }

# Relay 스타일 페이지네이션 예시
type UserConnection { edges: [UserEdge!]!, pageInfo: PageInfo! }
type UserEdge { node: User!, cursor: String! }
type PageInfo { hasNextPage: Boolean!, endCursor: String }
```
    
2. **선언적 데이터 질의 (Queries):** 클라이언트는 필요한 필드를 명확하게 선택하여 한 번의 요청으로 계층적인 데이터를 가져올 수 있습니다.
```graphql
query GetUser($id: ID!, $first: Int = 10) {
  me: user(id: $id) { id name }
  friends: searchUsers(keyword: "kim", first: $first) {
    edges { node { id name } }
    pageInfo { hasNextPage endCursor }
  }
}
```

3. **데이터 변경 (Mutations):** 데이터의 생성, 수정, 삭제 작업을 명시적으로 정의된 타입으로 수행합니다.

```graphql
mutation UpdateUser($input: UpdateUserInput!) {
  updateUser(input: $input) { ok user { id name } }
}
```

4. **리졸버 (Resolvers):** 스키마의 각 필드에 대한 실제 데이터 조회 로직을 구현하는 함수입니다. 데이터베이스, REST API 등 다양한 데이터 소스를 통합할 수 있습니다.

```graphql
resolver(parent, args, context, info)
```
- parent: 상위 필드의 반환값(중첩 해결에 사용)
- args: 필드 인자
- context: 인증 정보, 로더, 트랜잭션 핸들 등 공용 컨텍스트
- info: 실행 경로, 선택된 필드 등 메타정보

5. **인트로스펙션 (Introspection):** 스키마 자체를 질의하여 자동 문서화, 타입 검사, IDE 자동 완성 등 개발 편의 기능을 제공합니다.
```graphql
{
  __type(name: "User") { name fields { name type { kind name } } }
}
```

6. Subscriptions : 서버 푸시가 필요한 이벤트를 스트림으로 구독합니다. 전송은 WebSocket(SSE를 쓰기도 함) 기반이 일반적입니다.
   GraphQL은 전송 독립적이기 때문에, 운영 시 재연결·순서보장·백프레셔 고려가 필요합니다.
```graphql
type Subscription {
  userUpdated(id: ID!): User!
}
```


---

### GraphQL의 장점과 단점

#### 장점

- **효율성:** 클라이언트가 필요한 데이터만 요청하므로 Over-fetching 및 Under-fetching 문제가 해소되어 네트워크 비용과 페이로드 크기가 감소합니다.
    
- **유연성:** 스키마에 새로운 타입과 필드를 추가하는 방식으로 API를 확장할 수 있어, REST API에 비해 더욱 유연한 버저닝을 제공해 클라이언트와의 동기화 비용이 절감됩니다.
  *버저닝은 규칙이 아닌 원칙임을 참고하세요.*
    
- **서버사이드 배치&캐시:** GraphQL은 필드별로 데이터를 가져오는 리졸버(Resolver) 구조를 채택하고 있습니다. 이는 코드의 가독성을 높이고 역할을 명확히 분리하는 장점이 있지만, 최적화가 없으면 N+1 쿼리 문제와 같은 성능 저하를 일으킬 수 있습니다.
  따라서 GraphQL에서는 주로 서버사이드 배치와 캐싱 기술을 사용해 성능 최적화를 합니다. Facebook의 [DataLoader](https://github.com/facebook/dataloader)와 같은 도구가 그 예시입니다.

DataLoader의 역할
> DataLoader는 다음과 같은 방식으로 N+1 쿼리 문제를 해결합니다.
> 1. **요청 수집:** 여러 리졸버에서 발생하는 동일한 종류의 데이터 요청들을 모아둡니다.
> 2. **단일 요청:** 모든 요청을 모은 후, 이를 하나의 큰 쿼리로 데이터베이스에 전송합니다.
> 3. **데이터 분배:** 데이터베이스에서 반환된 결과를 원래 요청했던 각 리졸버에 맞게 분배해줍니다.
> 이러한 과정을 통해, DataLoader는 데이터베이스 부하를 줄이고 GraphQL 서비스의 응답 속도를 크게 향상시킵니다.

- **Nullability** : GraphQL 타입 시스템에서는 모든 필드가 기본적으로 nullable이기 때문에, 만약 예외가 발생할 경우, 해당 필드가 null로 반환되어 요청이 완전히 실패하지 않도록 보장합니다. non-null 타입을 제공하여 절대 null을 반환하지 않도록 보장할 수도 있습니다.
  *그러나 non-null 타입에서 오류가 발생할 시, 상위 타입에서 null을 반환할 수 있습니다. 경우에 따라 루트 데이터 전체가 null이 될 수도 있습니다.*

#### 단점

- **서버 복잡도 증가:** 필드 단위의 리졸버, 캐싱, 권한 관리 등 서버 측 설계가 REST보다 복잡해질 수 있습니다.
    
- **캐싱 전략 재설계:** URL 기반의 HTTP 캐싱을 활용하기 어려워 클라이언트/필드 레벨의 새로운 캐싱 전략을 수립해야 합니다.

캐싱 전략이란?
> 무엇을, 어디에, 얼마나 오래, 어떤 키로 저장하고 언제 무효화(purge)할지에 대한 규칙입니다.
> 레이어별로 다르게 세웁니다.
> 클라이언트 캐시: 앱 메모리/디스크에 최근 응답을 저장해 재사용
> 서버 캐시: 리졸버/응답 결과를 서버 쪽 메모리·Redis 등에 저장
> 엣지 캐시: CDN/프록시가 응답을 저장해 전 세계에서 빠르게 서빙

캐싱 전략을 활용하기 어려운 이유
> 하나의 URL에 쿼리 본문을 넣어 요청하는 구조이기 때문에, URL 기반 HTTP 캐싱을 하기 힘듬. 따라서 아래와 같은 캐싱 전략을 활용해볼 수 있습니다.
> **클라이언트 캐시:** 아폴로 클라이언트(Apollo Client)나 릴레이(Relay) 같은 도구가 제공하는 정교한 캐싱 기능을 활용하여 이전에 요청했던 데이터를 효율적으로 관리할 수 있습니다.
> **서버 캐시:** 특정 쿼리의 응답을 서버 메모리에 저장하거나, 필드 단위로 캐싱을 적용하는 등 다양한 방법을 사용할 수 있습니다.

- **성능 문제:** 필드 리졸버가 반복적으로 외부 소스를 호출하는 경우 **N+1 쿼리 문제**가 발생하기 쉽습니다. 이는 **DataLoader**를 통해 해결할 수 있습니다.
    
- **보안 취약점:** 쿼리 언어의 특성상 DoS(Denial-of-Service) 공격에 취약할 수 있으므로, 쿼리 심도 및 복잡도 제한 기능을 구현해야 합니다.

> DoS의 예시로, 재귀적 쿼리를 실행시키는 것이 있으며, DB에서 방대한 데이터를 요청하는 쿼리를 실행할 수도 있음.

---

### 스키마와 질의 예시

**스키마(SDL) 예시:**

GraphQL

```
type User {
  id: ID!
  name: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  author: User!
}

type Query {
  user(id: ID!): User
}
```

**클라이언트 질의 예시:**

GraphQL

```
query GetUserAndPosts($userId: ID!) {
  user(id: $userId) {
    name
    posts {
      title
    }
  }
}
```

**응답 예시:**

JSON

```
{
  "data": {
    "user": {
      "name": "민규",
      "posts": [
        { "title": "GraphQL 도입기" },
        { "title": "GraphQL과 REST 비교" }
      ]
    }
  }
}
```

---

### 결론

GraphQL은 여러 뷰에서 서로 다른 데이터가 필요한 환경, 네트워크 효율성이 중요한 모바일 환경, 프론트엔드 팀이 API 형태를 주도적으로 정의해야 하는 조직에 특히 적합한 기술입니다.
도입 시에는 스키마 우선 설계, **DataLoader**를 활용한 성능 최적화, 점진적 도입 전략 등을 고려하는 것이 효과적입니다.