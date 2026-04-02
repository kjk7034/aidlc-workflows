# BookShelf Community Library API — 요구사항 문서

## 의도 분석 요약

| 속성 | 값 |
|-----------|-------|
| **사용자 요청** | 두 개의 독립 배포 가능 서비스(Catalog Service, Lending Service)로 클라우드 배포 BookShelf 플랫폼 구축 |
| **요청 유형** | 신규 프로젝트 (Greenfield) |
| **범위 추정** | 시스템 전반 — 공유 인증, 이벤트 기반 통신, 포괄적 비즈니스 규칙을 갖춘 두 마이크로서비스 |
| **복잡도 추정** | 복잡 — 다중 서비스, RBAC, 비동기 이벤트, 대출 정책 집행, 서비스 간 통신 |
| **요구사항 깊이** | 포괄적 |

---

## 1. 기능 요구사항

### 1.1 Catalog Service

#### FR-CAT-001: 도서 CRUD 작업
- 제목, 저자, ISBN(선택), 카테고리, total_copies로 도서 추가
- 도서 메타데이터 수정(제목, 저자, ISBN, 카테고리, total_copies)
- ID로 단일 도서 조회
- 전체 도서 목록
- 도서 삭제(소프트 삭제 또는 하드 삭제 — 사서/관리자만)
- 모든 문자열 필드에 최대 길이 제약

#### FR-CAT-002: 도서 검색
- 제목·저자 필드 전문 검색(부분 문자열 일치)
- 카테고리로 필터
- 가용성으로 필터(available copies > 0)
- 검색 + 필터 조합 지원

#### FR-CAT-003: 도서 가용성 추적
- 각 도서가 `total_copies`와 `available_copies` 추적
- 대출 시 `available_copies` 감소(Lending Service의 이벤트 또는 API 호출)
- 반납 시 `available_copies` 증가(Lending Service의 이벤트 또는 API 호출)
- `available_copies`는 0 미만 또는 `total_copies` 초과 불가

#### FR-CAT-004: 도서 가용성 API
- Lending Service가 대출/예약 전 도서 존재 및 가용 부본을 검증하기 위한 내부 API 엔드포인트
- 도서 ID, 제목, total_copies, available_copies 반환

#### FR-CAT-005: 헬스 체크
- `GET /api/v1/catalog/health`가 서비스 상태와 버전 반환

### 1.2 Lending Service

#### FR-LND-001: 회원 등록
- 이름, 이메일, 비밀번호로 등록
- 이메일은 고유해야 함
- 적응형 해싱(bcrypt)으로 비밀번호 저장
- 등록 시 "member" 역할 자동 부여
- 회원 ID와 프로필 반환(비밀번호 제외)

#### FR-LND-002: 회원 인증
- 이메일과 비밀번호로 로그인
- 24시간 만료 JWT 토큰 반환
- JWT에 포함: member_id, email, role
- 두 서비스가 JWT를 독립 검증

#### FR-LND-003: 회원 프로필 관리
- 회원은 자신의 프로필 조회·수정(이름, 이메일)
- 관리자는 임의 회원 프로필 조회
- 사서는 임의 회원 프로필 조회

#### FR-LND-004: 역할 기반 접근 제어
- 세 역할: Admin, Librarian, Member
- **Admin**: 두 서비스의 모든 엔드포인트에 전체 접근
- **Librarian**: 장서 관리(CRUD, 검색), 대출 작업(반납 처리, 예약 관리), 보고 조회
- **Member**: 셀프 서비스(본인 대출, 예약, 수수료, 프로필, 장서 검색)
- **Public**: 등록, 로그인, 헬스 체크 — JWT 불필요

#### FR-LND-005: 대출
- 회원이 도서 ID로 대출
- 대출 전 검증:
  - 도서 존재(Catalog Service HTTP 호출로 검증)
  - Available copies > 0
  - 회원 활성 대출 수 < 5(설정 가능 최대)
  - 회원 미결 수수료 ≤ $10.00 임계값(설정 가능); 초과 시 차단
- 대출 성공 시:
  - member_id, book_id, checkout_date(UTC), due_date(checkout_date + 14일), status=active, renewal_count=0로 대출 기록
  - Catalog Service에서 available_copies 감소
- 기한: 대출일로부터 14일

#### FR-LND-006: 반납
- 회원 또는 사서/관리자가 checkout ID로 반납
- 반납 시:
  - 연체 시 연체료 계산: 일 $0.25, 대출당 상한 $10.00
  - 연체료 > 0이면 회원에 수수료 기록 생성
  - 대출 상태를 "returned"로, return_date 설정
  - Catalog Service에서 available_copies 증가
  - 예약 대기열 동기 확인: 해당 도서에 예약이 있으면 다음 예약 이행(hold 상태를 "ready"로)
- 모든 타임스탬프는 UTC

#### FR-LND-007: 갱신
- 회원이 checkout ID로 활성 대출 갱신
- 검증:
  - 대출이 활성(반납되지 않음)
  - 대출이 요청 회원 소유
  - 갱신 횟수 < 2(대출당 최대 2회 갱신)
  - 도서에 활성 예약 없음
- 갱신 시: 현재 due_date부터 due_date를 14일 연장, renewal_count 증가

#### FR-LND-008: 활성 대출
- 요청 회원의 활성 대출 목록
- 각 레코드에 checkout_id, book_id, book_title, checkout_date, due_date, renewal_count 포함
- 관리자/사서는 임의 회원의 활성 대출 목록 가능

#### FR-LND-009: 예약 등록
- 회원이 도서 ID로 예약
- 검증:
  - 도서 존재(Catalog Service로 검증)
  - 가용 부본 없음(available_copies == 0)
  - 회원 활성 예약 수 < 3(설정 가능 최대)
  - 해당 도서에 이미 활성 예약 없음
- 등록 시: member_id, book_id, hold_date, status=waiting, queue_position(FIFO) 기록

#### FR-LND-010: 예약 취소
- 회원이 hold ID로 본인 예약 취소
- 사서/관리자는 임의 예약 취소
- 취소 시: 상태를 "cancelled"로, 대기열 위치 재정렬

#### FR-LND-011: 예약 대기열 상태
- 특정 도서의 예약 대기열: 위치와 상태가 있는 예약 목록
- 회원은 대기열에서 본인 순번 확인 가능

#### FR-LND-012: 예약 이행(MVP 동기)
- 도서가 반납되고 해당 도서에 "waiting" 예약이 있으면:
  - FIFO 순으로 첫 예약 이행
  - 예약 상태를 "waiting"에서 "ready"로 갱신
  - 유예 없음 — 즉시 이행
- 진짜 비동기 처리(SQS/EventBridge)는 2단계로 연기

#### FR-LND-013: 수수료 추적
- 회원별 미결 수수료 추적
- 수수료 기록: fee_id, member_id, checkout_id, amount, created_date, status(outstanding/paid/partial)
- 요청 회원의 모든 수수료 목록
- 관리자/사서는 임의 회원 수수료 조회

#### FR-LND-014: 수수료 납부
- 회원 미결 수수료에 대한 납부 기록
- 부분 납부 허용
- 관리자와 사서가 납부 처리
- 납부 기록: payment_id, member_id, amount, payment_date

#### FR-LND-015: 연체 보고
- 현재 연체 중인 모든 대출(due_date < now이고 status=active)
- 포함: 회원 이름, 이메일, 도서 제목, checkout_date, due_date, days_overdue
- 사서 및 관리자 역할만 접근

#### FR-LND-016: 장서 요약
- 총 도서 수, 총 회원 수, 대출 중 도서, 가용 도서, 총 미결 수수료
- 관리자 역할만 접근

#### FR-LND-017: 헬스 체크
- `GET /api/v1/lending/health`가 서비스 상태와 버전 반환

### 1.3 서비스 간 통신

#### FR-ISC-001: 도서 검증
- Lending Service가 대출/예약 전 Catalog Service에 HTTP로 호출하여 도서 존재 및 가용성 검증
- 동기 검증을 위한 직접 HTTP 호출(이벤트 기반 아님)

#### FR-ISC-002: 가용성 갱신
- Lending Service가 대출/반납 시 Catalog Service에 HTTP로 호출하여 available_copies 감소/증가
- 원자적 작업 — 갱신이 실패하면 대출/반납도 실패해야 함

### 1.4 계정 비활성화 동작

#### FR-ACC-001: 계정 비활성화
- 관리자가 회원 계정 비활성화 가능
- 비활성 계정: 기존 예약과 대출은 유지되나 신규 대출, 예약, 갱신 불가
- 계정 정지 자동화는 MVP 범위 밖(2단계로 연기)

---

## 2. 비기능 요구사항

### 2.1 성능
| 지표 | 목표 |
|--------|--------|
| API 응답 시간(p95) | 100ms 미만 |
| 전문 검색 지연 | 200ms 미만 |
| 동시 사용자 | 100명 |
| 서비스 간 호출 오버헤드 | 지연 50ms 미만 추가 |

### 2.2 신뢰성
| 지표 | 목표 |
|--------|--------|
| API 가동 시간 | 99.9% |
| 데이터 내구성 | 다중 AZ 복제 |

### 2.3 보안
- 세 역할 RBAC가 모든 엔드포인트에 집행
- 24시간 만료 JWT 인증
- bcrypt 비밀번호 해싱
- 모든 엔드포인트에 Pydantic 입력 검증
- 로그에 PII 없음
- 저장 및 전송 암호화
- 보안 확장 규칙(SECURITY-01 ~ SECURITY-15) 집행

### 2.4 확장성
- 각 서비스는 독립 확장
- Catalog Service는 읽기 위주 부하에 최적화
- Lending Service는 쓰기 위주 부하에 최적화
- MVP는 단일 도서관 배포(2단계에서 멀티 테넌트)

### 2.5 테스트
| 테스트 유형 | 목표 |
|-----------|--------|
| 단위 테스트 커버리지 | 라인 커버리지 90% 이상 |
| 통합 테스트 | 모든 API 엔드포인트 테스트 |
| 계약 테스트 | 모든 엔드포인트가 OpenAPI 명세와 일치 |

### 2.6 유지보수성
- 단독 개발자가 유지 가능
- 인프라 비용 월 $50 미만
- 서비스 간 깔끔한 분리
- 일관된 API 래퍼 형식

---

## 3. API 설계 표준

- **스타일**: JSON을 쓰는 REST
- **버전 관리**: URL 경로 접두사 `/api/v1/`
- **필드 이름**: snake_case
- **성공 래퍼**: `{ "status": "ok", "data": { ... } }`
- **오류 래퍼**: `{ "status": "error", "error": { "code": "ERROR_CODE", "message": "..." } }`
- **오류 코드**: VALIDATION_ERROR (422), NOT_FOUND (404), UNAUTHORIZED (401), FORBIDDEN (403), CONFLICT (409), INTERNAL_ERROR (500)

---

## 4. 기술 스택

- **언어**: Python 3.13+
- **프레임워크**: FastAPI 0.115+ 및 Pydantic 2.x
- **서버**: uvicorn 0.34+
- **테스트**: pytest 8.x, pytest-asyncio, httpx, pytest-cov
- **린트**: ruff 0.9+
- **패키지 관리자**: uv
- **인프라**: AWS CDK 2.x (연기 — MVP는 애플리케이션 코드만)
- **클라우드**: AWS us-east-1
- **데이터베이스**: NFR 요구사항 / 인프라 설계 단계에서 결정

---

## 5. 아키텍처 결정

| 결정 | 선택 | 근거 |
|----------|--------|-----------|
| 서비스 아키텍처 | 두 개의 독립 서비스(Catalog + Lending) | 비전 요구사항; 독립 확장 |
| 인증 | 애플리케이션 수준 JWT(PyJWT + bcrypt); 두 서비스가 JWT 검증 | MVP에 Cognito보다 단순; 회원 데이터는 Lending Service 소유 |
| 서비스 간 통신(동기) | httpx 직접 HTTP 호출 | 단순, 테스트 가능; 도서 검증을 위해 Lending → Catalog |
| 예약 이행 | 반납 시 동기 | MVP에 더 단순; 진짜 비동기는 2단계 |
| 대출 수수료 임계값 | 미결 수수료 > $10.00이면 대출 차단 | 설정 가능 임계값; 집행과 사용성 균형 |
| 연체료 상한 | 대출당 $10.00 | 비전에 명시된 고정 상한 |
| CDK 인프라 | 연기 — MVP는 앱 코드만 | 동작 애플리케이션에 집중; 배포 문서 제공 |
| 데이터베이스 | NFR 단계로 연기 | tech-env가 DynamoDB와 RDS를 옵션으로 나열; 접근 패턴에 따라 선택 |

---

## 6. MVP 범위 경계

### 범위 내
- 도서 CRUD, 검색, 가용성 추적
- 회원 등록, 로그인, JWT 인증, 프로필 관리
- RBAC(Admin, Librarian, Member)
- 정책 집행이 완전한 대출, 반납, 갱신
- 예약 등록, 취소, 대기열 상태, 동기 이행
- 수수료 추적 및 납부
- 연체 보고 및 장서 요약
- 두 서비스 헬스 체크
- 단위 및 통합 테스트(커버리지 ≥90%)
- OpenAPI 명세

### 범위 밖(연기)
- 이메일 알림(2단계)
- 멀티 테넌트(2단계)
- 바코드/ISBN 스캔(2단계)
- 고급 분석(2단계)
- 추천 엔진(3단계)
- 계정 정지 자동화(2단계)
- 비밀번호 재설정(2단계)
- 페이지네이션(2단계)
- CDK 인프라 코드(배포 문서만)

---

## 7. 이후 단계에서의 미결 결정

| 결정 | 단계 |
|----------|-------|
| DB 선택(DynamoDB vs RDS PostgreSQL) | NFR 요구사항 / 인프라 설계 |
| 컴퓨트 선택(Lambda vs Fargate) | NFR 요구사항 / 인프라 설계 |
| API Gateway 구성 | 인프라 설계 |
| 2단계용 메시징 인프라 | 인프라 설계 |
