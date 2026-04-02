# 코드 생성 계획 — Catalog Service + Lending Service

## 유닛 맥락
- **워크스페이스 루트**: workspace/
- **프로젝트 유형**: Greenfield 다중 유닛(마이크로서비스)
- **구조**: workspace/catalog-service/ 및 workspace/lending-service/

---

## Catalog Service 단계

- [ ] 1단계: Catalog Service 프로젝트 구조 생성(pyproject.toml, src 레이아웃, tests 디렉터리)
- [ ] 2단계: 도메인 엔티티 생성(Book, AvailabilityInfo)
- [ ] 3단계: Pydantic 요청/응답 모델 생성
- [ ] 4단계: 코어 모듈 생성(exceptions, response helpers, logging, config)
- [ ] 5단계: 추상 저장소 및 메모리 내 구현 생성
- [ ] 6단계: BookService 생성(비즈니스 로직)
- [ ] 7단계: 인증 미들웨어 생성(JWT 검증)
- [ ] 8단계: API 라우트 및 FastAPI 앱 생성
- [ ] 9단계: Catalog Service 단위 테스트 생성
- [ ] 10단계: Catalog Service 통합 테스트 생성

## Lending Service 단계

- [ ] 11단계: Lending Service 프로젝트 구조 생성(pyproject.toml, src 레이아웃, tests 디렉터리)
- [ ] 12단계: 도메인 엔티티 생성(Member, Checkout, Hold, Fee, Payment)
- [ ] 13단계: Pydantic 요청/응답 모델 생성
- [ ] 14단계: 코어 모듈 생성(exceptions, response helpers, logging, config)
- [ ] 15단계: 추상 저장소 및 메모리 내 구현 생성
- [ ] 16단계: AuthService 생성(JWT + bcrypt)
- [ ] 17단계: MemberService 생성
- [ ] 18단계: CheckoutService 생성
- [ ] 19단계: HoldService 생성
- [ ] 20단계: FeeService 생성
- [ ] 21단계: ReportService 생성
- [ ] 22단계: CatalogClient 생성(HTTP 클라이언트)
- [ ] 23단계: 인증 미들웨어 생성(JWT 검증)
- [ ] 24단계: API 라우트 생성(member, checkout, hold, fee, report)
- [ ] 25단계: FastAPI 앱 진입점 생성
- [ ] 26단계: Lending Service 단위 테스트 생성
- [ ] 27단계: Lending Service 통합 테스트 생성

**합계**: 두 서비스를 아우르는 27단계
