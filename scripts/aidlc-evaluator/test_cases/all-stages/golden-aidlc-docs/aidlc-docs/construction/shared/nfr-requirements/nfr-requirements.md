# NFR 요구사항 — 두 서비스 공통

## 성능 요구사항

| 요구사항 | 목표 | 적용 대상 |
|---|---|---|
| API 응답 시간(p95) | 100ms 미만 | 두 서비스 |
| 전문 검색 지연 | 200ms 미만 | Catalog Service |
| 동시 사용자 | 100명 | 두 서비스 |
| 콜드 스타트 허용 | 3초 미만 | 두 서비스 |

## 가용성 요구사항

| 요구사항 | 목표 |
|---|---|
| API 가동 시간 | 99.9% |

## 보안 요구사항

| 요구사항 | 상세 |
|---|---|
| 인증 | 애플리케이션 수준 JWT(PyJWT + bcrypt) |
| 인가 | Admin, Librarian, Member 역할의 RBAC |
| 비밀번호 저장 | bcrypt 적응형 해싱 |
| 토큰 만료 | 24시간, 리프레시 토큰 없음(MVP) |
| 저장 시 암호화 | 모든 데이터 저장소에 필요 |
| 전송 중 암호화 | TLS 1.2+ |
| PII 보호 | 회원 이메일/이름은 로그에 남기지 않음 |
| 입력 검증 | 모든 엔드포인트에 길이 제약이 있는 Pydantic 모델 |
| CORS | 제한된 출처(설정 가능, 인증 엔드포인트에 와일드카드 없음) |

## 테스트 요구사항

| 테스트 유형 | 목표 |
|---|---|
| 단위 테스트 커버리지 | 서비스당 라인 커버리지 90% 이상 |
| 통합 테스트 | 모든 API 엔드포인트 테스트 |
| 인증 테스트 | 엔드포인트별 허가·비허가 접근 모두 |

## 확장성

- 각 서비스는 독립적으로 확장
- Catalog: 읽기 위주(검색/탐색)에 최적화
- Lending: 쓰기 위주(대출/반납)에 최적화
- MVP는 단일 도서관 배포
- 목표: 도서 10,000권, 회원 2,000명

## 데이터베이스 결정

**결정: 추상 기본 클래스가 있는 메모리 내 저장소로 MVP 개발 및 테스트.**

**근거:**
- tech-env가 NFR 단계까지 DB 선택(DynamoDB vs PostgreSQL)을 연기함
- CDK 인프라가 연기됨 — 클라우드 DB가 프로비저닝되지 않음
- 로컬 개발 및 테스트에는 메모리 내 저장소가 의존성 제로 운영을 제공
- 추상 저장소 패턴으로 이후 DynamoDB 또는 PostgreSQL을 구체 구현으로 교체 가능
- 메모리 내 접근은 "애플리케이션 코드만" MVP 범위와 일치
- 모든 비즈니스 로직과 API 동작을 외부 의존성 없이 완전히 테스트 가능

**저장소 패턴:**
- `BaseRepository`(추상)가 인터페이스 정의(create, get, list, update, delete, search)
- `InMemoryRepository`가 Python dict로 구현 — 개발 및 테스트용
- 향후: `DynamoDBRepository` 또는 `PostgresRepository`를 드롭인 대체로 추가 가능

## 인증 결정

**결정: PyJWT + passlib[bcrypt]를 사용한 애플리케이션 수준 JWT**

**근거:**
- MVP에 Cognito보다 단순
- 외부 의존성 없음(테스트 실행에 AWS 계정 불필요)
- 두 서비스가 공유 비밀로 JWT를 독립 검증
- JWT 비밀은 설정에 저장(로컬 개발은 환경 변수, 프로덕션은 Secrets Manager)

## 구조화 로깅

- JSON 포매터가 있는 Python `logging` 모듈
- 필드: timestamp, correlation_id(요청 ID), level, message, service_name
- PII 필터링: 로그 출력에서 회원 이메일·이름 제거
- stdout으로 로그(배포 시 CloudWatch 호환)
