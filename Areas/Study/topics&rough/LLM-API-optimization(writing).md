https://swalloow.github.io/rag-application-for-production-1/
https://blog.leaphop.co.kr/blogs/95/LLM_%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%9D%98_API_%ED%86%A0%ED%81%B0_%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0__%EC%B8%A1%EC%A0%95%EB%B6%80%ED%84%B0_%EC%A0%88%EC%95%BD%EA%B9%8C%EC%A7%80
https://easy-to-read.tistory.com/67
https://yooniverse1007.tistory.com/35


## 백엔드에서의 LLM API 호출 최적화

### 1. 왜 필요한가?

- 외부 LLM API(OpenAI, Claude, Gemini 등)는 **비용, 성능, 안정성** 문제가 직결됨
    
- 단순히 “API 호출”이 아니라 **백엔드 아키텍처 수준의 최적화**가 필요
    

---

### 2. 최적화 관점별 접근

#### (1) 비용 최적화

- 토큰 관리: 프롬프트 압축, 요약, 불필요한 맥락 제거
    
- 응답 캐싱: 같은 질문 반복 시 Redis 캐시 활용
    
- 사용자별 호출 제한: Free/Premium Tier 구분, 쿼터 기반 관리
    

#### (2) 성능 최적화

- **비동기 호출**: non-blocking I/O로 API 지연 최소화
    
- **스트리밍 응답**: SSE, WebSocket으로 체감 속도 개선
    
- **Bulk 요청 최적화**: 여러 요청을 하나의 배치로 묶어 호출
    

#### (3) 안정성 & Rate Limit 대응

- API Gateway에서 큐잉, 백프레셔 적용
    
- Retry & Fallback 전략: 외부 API 장애 시 대체 경로 (예: 캐시 응답)
    
- 사용자별 Rate Limit 정책 (서비스 남용 방지)
    

#### (4) 보안

- API 키 서버사이드 보관 (클라이언트 노출 금지)
    
- Key Rotation 및 Access Token 관리
    
- Prompt Injection 방어: 입력/출력 필터링
    

---

### 3. 실무 사례

- SaaS API 호출 비용이 급등한 사례 → 프롬프트 최적화/캐싱으로 절감
    
- 대규모 사용자 트래픽 → 비동기 + 스트리밍 응답으로 서버 부하 완화
    
- 토스/네이버 등 대형 서비스 블로그에서 언급한 Rate Limit 대응법
    

---

### 4. 앞으로의 방향

- 외부 API와 로컬 모델의 하이브리드 운영 가능성
    
- Observability(로그/메트릭 기반 모니터링)로 비용·성능 실시간 추적
    
- 오픈소스 미들웨어(vLLM proxy, LangSmith 등) 활용