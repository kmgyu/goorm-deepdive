하는 김에 칸반 보드 연습하기
아래는 GPT가 짜준 로드맵, 수정 필요

# Spring Template Project Kanban Roadmap

## Roadmap Strategy and Priorities

The **Spring Template** project is designed as a long-term maintainable Spring Boot backend, prioritizing stability over chasing every new framework[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L3-L5). This roadmap outlines a sequence of versioned milestones (V1.1 through V2.0 and beyond) to incrementally introduce features, improve infrastructure, enhance code quality, and refine documentation. Each version’s goals are grouped into four categories – **Feature Development**, **Infrastructure/Architecture**, **Code Quality**, and **Documentation & Developer Experience** – ensuring balanced progress in all areas. The plan assumes an initial V1.0 release established the core framework (e.g. basic user auth and initial DB schema for users, posts, comments[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/V1.md#L3-L8)). From V1.1 onward, the focus shifts to implementing missing features (like post/comment CRUD), then progressively tackling architectural scalability (microservices) once the foundation is solid.

To manage these tasks, we use a **Kanban workflow** with columns _Backlog, Ready, In Progress, Code Review, Blocked,_ and _Done_. In the tables/checklists below, each task is labeled with a suggested current **Status** (column) to illustrate its progression. The Kanban approach will be used in GitHub Projects (or Jira), meaning tasks move from **Backlog** (planned work) to **Ready** (groomed for development), then **In Progress**, through **Code Review**, and finally **Done**. If any task is waiting on an external dependency or decision, it’s marked **Blocked** until the impediment is resolved. This system ensures transparency and focus: highest-priority items are pulled from Ready to In Progress each iteration, while completed work is promptly reviewed and closed. The roadmap below provides a clear path from the current state (V1.1) to future vision (V2.0 microservices and beyond), aligning with the project’s goal of a **reliable, stable template** for the long run[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L3-L5)[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L48-L50).

## Version 1.1 – Core Features & Environment Setup

_Focus:_ Complete the basic CRUD features for the blogging/forum functionality and establish a solid development environment. Version 1.1 emphasizes implementing **Post** and **Comment** management (the core “board” features) and setting up essential devOps tooling to streamline development and deployment.

**Feature Development:**

-  **User Authentication:** Implement basic signup & login flows (Spring Security, password encoding, default roles) – **Status: Done**. _(_Basic auth was implemented in the initial release, providing a foundation for other features._)_
    
-  **Post CRUD:** Create, read, update, and delete posts (backend + Thymeleaf UI) – **Status: In Progress**. _(Lay out Post entity, repository, service & controller; add post listing and editing views. This builds on the existing `posts` table[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/V1.md#L3-L8) defined in V1.0.)_
    
-  **Comment CRUD:** Enable adding, editing, and deleting comments on posts – **Status: In Progress**. _(Link comments to posts via a Post detail page; reuse the `comments` table from initial schema[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/V1.md#L3-L8) to store comments.)_
    

**Infrastructure/Architecture:**

-  **Docker Compose Setup:** Add Dockerfile and Docker Compose for local development (containerize the Spring app with a MariaDB database) – **Status: Code Review**. _(This was a planned enhancement[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L25-L29); allows new developers to spin up the app and DB quickly in a consistent environment.)_
    
-  **CI Pipeline (GitHub Actions):** Configure Continuous Integration to run tests and builds on each push – **Status: In Progress**. _(Set up a GitHub Actions workflow to compile the project and execute the test suite on pull requests, ensuring no regressions.)_
    

**Code Quality:**

-  **Basic Unit Tests:** Start a test suite covering core features (e.g. user registration, post creation) – **Status: Ready**. _(Aim for initial coverage ~30%. Establish testing frameworks and CI integration for quality gates.)_
    
-  **Code Refactoring:** Improve project structure for clarity – **Status: Backlog**. _(E.g. reorganize packages – possibly remove the temporary `playground` package naming – and apply consistent naming conventions once initial features are in place.)_
    

**Documentation & Developer Experience:**

-  **Setup Guide:** Update the README with clear setup and run instructions – **Status: In Progress**. _(Cover environment setup, how to apply Flyway migrations, and how to run the app; this lowers the barrier for new contributors.)_
    
-  **Basic API Documentation:** Introduce simple API docs for key endpoints – **Status: Backlog**. _(For example, integrate Springdoc OpenAPI or add an `api.md` listing the main REST endpoints. This will be expanded as the project grows.)_
    

## Version 1.2 – Enhancements & Tech Integrations

_Focus:_ Extend the application with admin capabilities and integrate supporting technologies. V1.2 targets an **Admin UI** for content management, introduces caching and background jobs for better performance, and improves the deployment pipeline. Code quality efforts continue with higher test coverage and static analysis.

**Feature Development:**

-  **Admin UI & Management Features:** Implement an admin dashboard for managing posts/users – **Status: Backlog**. _(Leverage the existing `ADMIN` role[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/java/example/spring_template/playground/user/UserRole.java#L6-L8) to restrict access. Provide interfaces for admins to delete inappropriate posts, manage user roles, etc.)_
    

**Infrastructure/Architecture:**

-  **Redis Caching:** Integrate Redis for caching and session management – **Status: Ready**. _(Improve performance by caching frequent read operations, e.g. post lists. Use Spring Boot’s Redis support to store sessions or cache queries.)_
    
-  **Scheduled Batch Job:** Add a scheduled batch process – **Status: Backlog**. _(For example, a nightly job to purge old posts or send summary emails. This introduces Spring’s scheduling or Spring Batch, showcasing timed background tasks.)_
    
-  **Continuous Deployment Pipeline:** Extend CI to CD – automating deployments to a dev/staging environment – **Status: **🚫** Blocked**. _(Set up Docker image builds and push to a registry, and auto-deploy to a test server or container. **Blocked:** awaiting credentials/infrastructure for the staging environment.)_
    

**Code Quality:**

-  **QueryDSL Integration:** Add QueryDSL for type-safe database queries[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L39-L42) – **Status: Backlog**. _(Enhance the repository layer with fluent, typesafe queries. This is planned once the team is comfortable with QueryDSL.)_
    
-  **Increase Test Coverage:** Expand tests for new features (aim ~50% coverage) – **Status: Ready**. _(Cover Admin controllers, security tests for role-based access, and integration tests for Redis caching behavior.)_
    
-  **Static Analysis & Linting:** Introduce code quality tools – **Status: Ready**. _(E.g. enable Checkstyle/SpotBugs or integrate SonarQube in CI. Enforce coding standards and catch bugs/security issues early.)_
    

**Documentation & Developer Experience:**

-  **API Documentation (Expanded):** Publish interactive API docs – **Status: Ready**. _(Integrate Swagger UI or Springdoc OpenAPI to auto-generate documentation for REST endpoints. This improves consumer understanding of the API.)_
    
-  **User Guide Updates:** Update docs for new features and tools – **Status: Ready**. _(Document the Admin UI usage, Redis setup requirements, and how to run the batch jobs. Ensure the README and wiki reflect these additions so developers can easily configure and use them.)_
    

## Version 1.3 – Hardening & Architecture Planning

_Focus:_ Hardening the application and laying the groundwork for a microservices transition. V1.3 is about **modularizing** the codebase and final polishing before a major architectural shift. Feature work is lighter here, allowing focus on design and quality: increasing test coverage, performing thorough refactoring, and preparing documentation for the upcoming split into services.

**Feature Development:**

-  **User Profile Management:** Enable users to manage their profile (update email, password, etc.) – **Status: Backlog**. _(Adds a personal settings page, demonstrating how to handle user-specific data and form updates in the template.)_
    

**Infrastructure/Architecture:**

-  **Microservices Design Blueprint:** Define a clear plan to split into services – **Status: Backlog**. _(Identify service boundaries – e.g. a **User/Auth Service** vs **Post/Comment Service** – and design how they will communicate (REST APIs, etc.). Decide on database segregation or shared DB, and plan required supporting services like API Gateway or service registry.)_
    
-  **Kubernetes Dev Setup:** Prepare a local K8s environment for testing (e.g. Minikube or Docker Desktop K8s) – **Status: Ready**. _(This allows testing deployment of multiple services in anticipation of the move to K8s. Define Kubernetes manifests or Helm charts in draft form.)_
    

**Code Quality:**

-  **Modularize Codebase:** Refactor into multi-module structure for domains – **Status: Ready**. _(E.g. separate modules for “user", “post", etc., within the project. This enforces clearer boundaries in the monolith, making the eventual extraction to microservices easier and improving maintainability.)_
    
-  **Increase Test Coverage:** Further expand automated tests (aim ~70% coverage) – **Status: Ready**. _(Include integration tests covering module boundaries and ensure new profile features and admin tools are well-tested. Establish a habit of writing tests for each bug fix or new feature.)_
    
-  **Performance Audit:** Conduct basic load testing and profiling – **Status: Backlog**. _(Use tools like JMeter or Gatling to ensure the application performs well with the current architecture. Identify any bottlenecks or memory issues to address before microservices deployment.)_
    

**Documentation & Developer Experience:**

-  **Contribution Guidelines:** Create a contributing guide – **Status: Ready**. _(Outline the project structure, coding style, how to fork and submit PRs, and the Kanban workflow for issue tracking. This lowers the barrier for external contributors and standardizes code contributions.)_
    
-  **Architecture Overview:** Document the planned microservice architecture – **Status: Ready**. _(Prepare a design document or wiki page describing each proposed service, its responsibilities, and interactions. This will be useful for onboarding and for the team to agree on the approach before moving to implementation.)_
    

## Version 2.0 – Microservices Launch & Deployment

_Focus:_ **Major architecture overhaul** – migrating the application from a monolith to a microservices architecture, and deploying it on Kubernetes for scalability. V2.0 delivers the same functionality as 1.x but re-packaged into independent services, accompanied by a robust CI/CD pipeline and production-ready infrastructure. This version is a significant milestone aimed at scalability, tech modernization, and serving as a blueprint for cloud-native Spring Boot apps.

**Feature Development:**

-  **New Feature Expansion:** _(TBD)_ Incorporate additional end-user features based on feedback – **Status: Backlog**. _(Any new functional improvements (e.g. real-time notifications, search enhancements) will be planned here if needed. Feature development in 2.0 is minimal to allow focus on architecture, but user needs will drive any extras.)_
    

**Infrastructure/Architecture:**

-  **Microservice: User/Auth Service:** Extract the user management and authentication into its own Spring Boot service – **Status: Ready**. _(This service handles user registration, login, roles, etc., with its own database or schema for user data.)_
    
-  **Microservice: Post/Comment Service:** Extract post and comment functionality into a separate service – **Status: Ready**. _(This service manages blog posts and comments, possibly with its own database. It will expose REST APIs for the front-end or other services to consume.)_
    
-  **API Gateway & Routing:** Implement an API Gateway to unify service endpoints – **Status: Ready**. _(Use Spring Cloud Gateway or NGINX to route client requests to the appropriate service. This provides a single entry point and can handle cross-cutting concerns like authentication, rate-limiting.)_
    
-  **Containerization & K8s Deployment:** Containerize all services and deploy on Kubernetes – **Status: Ready**. _(Write Dockerfiles for each service and Kubernetes manifests (Deployments, Services). Leverage the K8s ecosystem for service discovery and scaling. The app should be able to run as a multi-container deployment on a local or cloud K8s cluster.)_
    
-  **CI/CD for Microservices:** Set up pipelines to build, test, and deploy each service – **Status: Ready**. _(Extend the CI pipeline so each microservice is built and containerized on every commit. Implement CD to the K8s cluster – possibly using GitHub Actions or Jenkins – so that merging to main triggers a deployment update. This includes managing configuration (e.g., using ConfigMaps/Secrets) and ensuring zero-downtime deployments.)_
    

**Code Quality:**

-  **High Test Coverage:** Achieve ~90% test coverage on each service – **Status: Backlog**. _(Since each microservice is smaller and focused, enforce a high unit test and integration test coverage to guarantee reliability. Also set up contract tests or integration tests between services to verify inter-service communication (for example, test that the Auth service and Post service integrate correctly via the API gateway).)_
    
-  **Performance & Resilience Testing:** Load-test the distributed system and harden it – **Status: Backlog**. _(Perform end-to-end stress tests in a staging environment to observe system behavior under load. Identify any failure points in the new architecture (e.g., network latency issues, memory usage) and optimize them. Also test resilience by simulating service outages to ensure the system can gracefully handle partial failures.)_
    

**Documentation & Developer Experience:**

-  **Deployment Guide:** Provide documentation for deploying and running the microservices – **Status: Ready**. _(Cover how to set up the Kubernetes cluster (namespace, ingress, etc.), how to deploy the services (maybe with Helm or kubectl apply), and how to monitor the system. Include any troubleshooting tips for common issues in a distributed deployment.)_
    
-  **Service-specific Docs:** Update README/Docs for each service and overall system – **Status: Ready**. _(Each microservice gets its own README describing its purpose, configuration, and usage. Additionally, update the main project documentation to reflect the new architecture. Ensure API documentation is updated to cover the endpoints of each service, possibly aggregating them in the gateway documentation.)_
    

## Beyond V2.0 – Future Outlook

After V2.0, the project will continue to evolve, guided by stability and real-world feedback. Key themes for future planning include:

- **Advanced Features & Improvements:** Introduce new functionality as needed (e.g. real-time features with WebSockets, full-text search, or a mobile-friendly API). Also continually refine the UI/UX of the admin and user-facing components. These features will be added to the **Backlog** and prioritized based on user community feedback and project goals.
    
- **Ecosystem and Tech Upgrades:** Stay current with the tech stack in a measured way. For example, plan upgrades to the next Spring Boot LTS or Java version once they prove stable, ensuring the template remains modern but **compatible** (in line with the project’s focus on stability[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L3-L5)). Similarly, keep an eye on new Spring ecosystem projects that could further improve the architecture (e.g. Spring Cloud improvements, GraphQL integrations) and assess their adoption in future versions.
    
- **Scalability & Optimization:** As the template is used in larger projects, gather performance data to inform scaling best practices. Future work may include introducing **horizontal scaling** patterns, refining Kubernetes deployments (auto-scaling, pod resource optimizations), and enhancing monitoring/observability (integrating tools like Prometheus/Grafana or ELK stack for centralized logging). These infrastructure enhancements would ensure the project remains a **reliable and high-performance starting point** for any scale of application.
    
- **Continuous Quality and DX:** Maintain a high bar for code quality and developer experience. This includes periodically refactoring to eliminate technical debt, expanding the test suite for new edge cases, and improving developer tooling (like setting up a local cluster orchestration script, or container images to simplify setup). The contribution guidelines and documentation will be living documents, updated as the project and community evolve. By keeping docs and processes up-to-date, we ensure new contributors can easily onboard and that the project stays approachable and well-maintained.
    

Overall, this roadmap provides a structured path to grow **spring_template** from a solid monolithic starter into a **full-featured, scalable, microservice-ready platform**, all while preserving the project’s core values of maintainability and stability[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L3-L5). Each version’s Kanban board (Backlog → Done) will guide the development, review, and release of these improvements in an orderly, transparent manner. With disciplined execution of this roadmap, the project will progressively achieve its vision of a robust, enterprise-ready Spring Boot template.

**Sources:**

1. kmgyu/spring_template – _Project README (tech stack and goals)_[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L2-L5)[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L46-L50)
    
2. kmgyu/spring_template – _Planned Docker and QueryDSL support (README)_[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L25-L29)[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L39-L42)
    
3. kmgyu/spring_template – _Initial DB schema (Flyway V1, tables for users/posts/comments)_[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/V1.md#L3-L8)
    
4. kmgyu/spring_template – _User roles definition (ADMIN/USER)_[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/java/example/spring_template/playground/user/UserRole.java#L6-L8)


---

깃 프로젝트에서 쓰도록 된 버전. 믿을 거 못되니 첨삭 필요

# Spring Template 로드맵 (v1.1 ~ v2.0)

## V1.1

**기능 개발:**

-  사용자 인증(로그인/로그아웃 및 회원가입) 기능 구현 ✅ 완료
    
-  게시글 CRUD(작성, 조회, 수정, 삭제) 기본 기능 구현 ✅ 완료[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/V1.md#L3-L9)
    

**인프라/아키텍처:**

-  Flyway 기반 DB 마이그레이션 적용 ✅ 완료[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/V1.md#L5-L9)
    

**코드 품질:**

-  프로젝트 Lombok 적용으로 보일러플레이트 코드 감소 ✅ 완료
    

**문서 및 개발자 경험:**

-  기본 README에 핵심 스택 및 목표 내용 추가 ✅ 완료
    
-  MariaDB 개발 DB 초기 세팅 가이드 작성 ✅ 완료[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/start.md#L2-L10)[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/src/main/resources/db/start.md#L25-L31)
    
-  Spring DevTools 적용으로 코드 변경 시 자동 리로딩 지원 ✅ 완료[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/build.gradle.kts#L35-L38)
    

## V1.2

**기능 개발:**

-  댓글 작성 및 표시 기능 구현 🛠️ 진행중
    
-  게시글 목록 페이지네이션 기능 추가 📌 백로그
    

**인프라/아키텍처:**

-  Dockerfile 작성 및 애플리케이션 컨테이너화 🛠️ 진행중[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L25-L29)
    
-  Docker Compose를 활용한 MariaDB 등 개발환경 자동화 📌 백로그
    

**코드 품질:**

-  핵심 서비스 및 리포지토리에 대한 단위 테스트 작성 시작 🛠️ 진행중
    
-  코드 스타일 가이드 및 린트 도구 설정 📌 백로그
    

**문서 및 개발자 경험:**

-  README에 프로젝트 빌드 및 실행 방법 섹션 추가 🛠️ 진행중
    
-  Docker 컨테이너 실행 가이드 문서화 📌 백로그
    

## V1.3

**기능 개발:**

-  게시글 목록 페이지네이션 지원 ✅ 완료
    
-  게시글 검색(제목/내용 키워드) 기능 구현 📌 백로그
    

**인프라/아키텍처:**

-  GitHub Actions 기반 CI 파이프라인 구축 ✅ 완료
    
-  QueryDSL 통합을 통한 타입 안전 쿼리 지원 📌 백로그[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L39-L42)
    

**코드 품질:**

-  통합 테스트(Mock MVC 등을 활용) 케이스 추가 🛠️ 진행중
    
-  정적 코드 분석 도구(Checkstyle 등) 도입 📌 백로그
    

**문서 및 개발자 경험:**

-  Docker 활용 가이드(컨테이너 실행 방법) 문서화 ✅ 완료
    
-  API 명세 문서(Swagger UI) 도입 📌 백로그
    

## V2.0

**기능 개발:**

-  게시글 검색 기능 구현 ✅ 완료
    
-  REST API 엔드포인트(게시글/댓글 CRUD) 추가 🛠️ 진행중
    
-  관리자 페이지 및 권한 관리 기능 추가 📌 백로그
    

**인프라/아키텍처:**

-  QueryDSL 도입 및 빌드 설정 추가 ✅ 완료[GitHub](https://github.com/kmgyu/spring_template/blob/386df1f9b5b2940160718c3cf33ee60da2b7622f/readme.md#L39-L42)
    
-  Spring Boot 및 의존성 버전 최신화 ✅ 완료
    
-  CI 파이프라인에 Docker 이미지 빌드/배포 단계 추가 🛠️ 진행중
    
-  Spring Boot 4.0 업그레이드 준비 ⛔ 블로킹 (정식 릴리스 대기중)
    

**코드 품질:**

-  테스트 커버리지 80% 이상 달성 ✅ 완료
    
-  정적 분석 도구 적용 및 코드 규칙 준수 ✅ 완료
    

**문서 및 개발자 경험:**

-  최종 버전 README 및 사용자 가이드 업데이트 🛠️ 진행중
    
-  Swagger를 통한 API 문서 제공 ✅ 완료
    
-  운영 배포 가이드 작성 📌 백로그