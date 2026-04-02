# 사용자 스토리

## 에픽 1: 장서 관리

### US-CAT-001: 도서 추가
**역할** 사서, **원하는 것** 제목, 저자, ISBN, 카테고리, 부본 수로 장서에 신규 도서 추가, **이유** 회원이 발견하고 대출할 수 있도록.

**인수 기준:**
- 사서 또는 관리자로 인증된 경우
- 유효한 도서 데이터로 POST /api/v1/books 시
- 도서가 고유 ID로 생성되고 available_copies = total_copies이며 응답에 포함됨
- 필수 필드 누락·무효 시 422 VALIDATION_ERROR
- Member이면 403 FORBIDDEN

### US-CAT-002: 도서 수정
**역할** 사서, **원하는 것** 도서 메타데이터(제목, 저자, ISBN, 카테고리, total_copies) 수정, **이유** 장서가 정확히 유지되도록.

**인수 기준:**
- 사서 또는 관리자로 인증된 경우
- PUT /api/v1/books/{book_id}로 필드 갱신 시
- 도서가 갱신되고 갱신 레코드가 반환됨
- 도서가 없으면 404 NOT_FOUND
- total_copies를 현재 대출 중인 권수 아래로 줄이면 409 CONFLICT

### US-CAT-003: 도서 조회
**역할** 회원, **원하는 것** 특정 도서 상세 보기, **이유** 대출 여부를 결정하기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/books/{book_id} 시
- available_copies를 포함한 도서 상세 수신
- 없으면 404 NOT_FOUND

### US-CAT-004: 전체 도서 목록
**역할** 회원, **원하는 것** 전체 장서 탐색, **이유** 도서관에서 이용 가능한 도서를 발견하기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/books 시
- 가용성과 함께 모든 도서 목록 수신

### US-CAT-005: 도서 삭제
**역할** 사서, **원하는 것** 장서에서 도서 제거, **이유** 폐기된 도서가 더 이상 보이지 않도록.

**인수 기준:**
- 사서 또는 관리자로 인증된 경우
- DELETE /api/v1/books/{book_id} 시
- 장서에서 도서 제거됨
- 활성 대출이 있으면 409 CONFLICT
- 없으면 404 NOT_FOUND

### US-CAT-006: 도서 검색
**역할** 회원, **원하는 것** 제목·저자로 검색하고 카테고리·가용성으로 필터, **이유** 특정 도서를 빠르게 찾기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/books/search?q=python&category=programming&available=true 시
- 제목 또는 저자에 검색어가 일치하고 카테고리·가용성으로 필터된 도서 수신
- 일치 없으면 빈 목록(오류 아님)

---

## 에픽 2: 회원 관리 및 인증

### US-AUTH-001: 회원으로 등록
**역할** 신규 사용자, **원하는 것** 이름·이메일·비밀번호로 등록, **이유** 도서 대출을 시작하기 위해.

**인수 기준:**
- 인증되지 않음(공개 엔드포인트)
- POST /api/v1/members/register에 name, email, password 시
- "member" 역할로 계정 생성되고 프로필 수신(비밀번호 제외)
- 이메일이 이미 등록되어 있으면 409 CONFLICT
- 비밀번호가 너무 짧으면(8자 미만) 422 VALIDATION_ERROR

### US-AUTH-002: 로그인
**역할** 등록된 사용자, **원하는 것** 이메일·비밀번호로 로그인, **이유** API 접근용 JWT를 받기 위해.

**인수 기준:**
- 인증되지 않음(공개 엔드포인트)
- POST /api/v1/members/login에 유효한 자격 증명 시
- member_id, email, role이 포함된 24시간 만료 JWT 수신
- 자격 증명이 무효면 401 UNAUTHORIZED

### US-AUTH-003: 내 프로필 보기
**역할** 회원, **원하는 것** 프로필 정보 보기, **이유** 계정 정보를 확인하기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/members/me 시
- 프로필 수신(이름, 이메일, 역할, 활성 상태)

### US-AUTH-004: 내 프로필 수정
**역할** 회원, **원하는 것** 이름·이메일 수정, **이유** 정보를 최신으로 유지하기 위해.

**인수 기준:**
- 인증된 경우
- PUT /api/v1/members/me로 이름 또는 이메일 갱신 시
- 프로필이 갱신되고 새 프로필이 반환됨
- 새 이메일이 이미 사용 중이면 409 CONFLICT

### US-AUTH-005: 임의 회원 프로필 보기(관리자/사서)
**역할** 관리자 또는 사서, **원하는 것** 임의 회원 프로필 보기, **이유** 회원 계정을 관리하기 위해.

**인수 기준:**
- 관리자 또는 사서로 인증된 경우
- GET /api/v1/members/{member_id} 시
- 해당 회원 프로필 수신
- 회원이 없으면 404 NOT_FOUND
- Member이면 403 FORBIDDEN

### US-AUTH-006: 회원 계정 비활성화
**역할** 관리자, **원하는 것** 회원 계정 비활성화, **이유** 기존 의무는 추적하면서 신규 동작은 막기 위해.

**인수 기준:**
- 관리자로 인증된 경우
- PUT /api/v1/members/{member_id}/deactivate 시
- 회원 계정이 비활성으로 표시됨
- 기존 대출과 예약은 활성으로 유지
- 신규 대출·예약·갱신 불가
- 관리자가 아니면 403 FORBIDDEN

---

## 에픽 3: 대출 작업

### US-LND-001: 도서 대출
**역할** 회원, **원하는 것** 도서 대출, **이유** 14일간 빌리기 위해.

**인수 기준:**
- Member로 인증된 경우
- POST /api/v1/checkouts에 book_id 시
- due_date = now + 14일, status = active인 대출 기록 생성
- Catalog Service에서 available_copies 감소
- 도서가 없으면 404 NOT_FOUND
- 가용 부본이 없으면 409 CONFLICT
- 활성 대출이 이미 5건이면 409 CONFLICT, 메시지 "checkout limit exceeded"
- 미결 수수료가 $10.00을 초과하면 409 CONFLICT, 메시지 "outstanding fees exceed threshold"

### US-LND-002: 도서 반납
**역할** 회원 또는 사서, **원하는 것** 대출된 도서 반납, **이유** 다른 사람이 이용할 수 있도록.

**인수 기준:**
- 인증된 경우
- POST /api/v1/checkouts/{checkout_id}/return 시
- 대출 상태가 return_date와 함께 "returned"로 갱신됨
- Catalog Service에서 available_copies 증가
- 연체면 연체료 계산(일 $0.25, 대출당 상한 $10.00) 및 수수료 기록 생성
- 해당 도서에 활성 예약이 있으면 FIFO 순으로 첫 예약이 "ready"로 갱신됨
- Member은 본인 대출만 반납; 사서/관리자는 모든 대출 반납 가능

### US-LND-003: 대출 갱신
**역할** 회원, **원하는 것** 대출 갱신, **이유** 14일 더 보유하기 위해.

**인수 기준:**
- 대출 소유자로 인증된 경우
- POST /api/v1/checkouts/{checkout_id}/renew 시
- due_date가 14일 연장되고 renewal_count 증가
- renewal_count가 이미 2이면 409 CONFLICT, "renewal limit exceeded"
- 해당 도서에 활성 예약이 있으면 409 CONFLICT, "book has active holds"
- 대출이 활성이 아니면 409 CONFLICT

### US-LND-004: 내 활성 대출 보기
**역할** 회원, **원하는 것** 활성 대출 보기, **이유** 어떤 도서를 가졌는지와 기한을 알기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/checkouts?status=active 시
- book_id, book_title, checkout_date, due_date, renewal_count가 포함된 활성 대출 수신
- 관리자/사서는 member_id 쿼리로 임의 회원 대출 조회 가능

---

## 에픽 4: 예약 관리

### US-HLD-001: 예약 등록
**역할** 회원, **원하는 것** 대여 불가 도서에 예약, **이유** 반납 시 공정하게 접근하기 위해.

**인수 기준:**
- Member로 인증된 경우
- POST /api/v1/holds에 book_id 시
- status=waiting 및 FIFO 대기열 위치로 예약 기록 생성
- 가용 부본이 있으면 409 CONFLICT, "book is available for checkout"
- 해당 도서에 이미 예약이 있으면 409 CONFLICT, "duplicate hold"
- 활성 예약이 이미 3건이면 409 CONFLICT, "hold limit exceeded"

### US-HLD-002: 예약 취소
**역할** 회원, **원하는 것** 본인 예약 취소, **이유** 다른 사람이 대기열에서 앞으로 오도록.

**인수 기준:**
- 인증된 경우
- DELETE /api/v1/holds/{hold_id} 시
- 예약이 취소되고 나머지 대기열 위치가 재정렬됨
- Member은 본인 예약만 취소; 사서/관리자는 모든 예약 취소 가능

### US-HLD-003: 예약 대기열 보기
**역할** 회원, **원하는 것** 도서에 대한 대기열에서 본인 순번 보기, **이유** 대기 시간을 가늠하기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/holds?book_id={book_id} 시
- 위치와 상태가 있는 예약 대기열 수신
- 대기열에서 본인 순번 식별 가능

### US-HLD-004: 내 예약 보기
**역할** 회원, **원하는 것** 모든 활성 예약 보기, **이유** 예약 대기열을 관리하기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/holds/me 시
- book_id, status(waiting/ready/cancelled), queue_position, hold_date가 포함된 모든 예약 수신

---

## 에픽 5: 수수료 관리

### US-FEE-001: 내 수수료 보기
**역할** 회원, **원하는 것** 미결 수수료 보기, **이유** 얼마를 갚아야 하는지 알기 위해.

**인수 기준:**
- 인증된 경우
- GET /api/v1/fees/me 시
- fee_id, amount, status, created_date, checkout_id가 있는 수수료 목록 수신
- 총 미결 잔액 확인 가능

### US-FEE-002: 임의 회원 수수료 보기
**역할** 사서 또는 관리자, **원하는 것** 임의 회원 수수료 보기, **이유** 수수료 문의를 돕기 위해.

**인수 기준:**
- 사서 또는 관리자로 인증된 경우
- GET /api/v1/fees?member_id={member_id} 시
- 해당 회원 수수료 기록 수신

### US-FEE-003: 수수료 납부 처리
**역할** 사서 또는 관리자, **원하는 것** 회원 수수료 납부 기록, **이유** 잔액이 갱신되도록.

**인수 기준:**
- 사서 또는 관리자로 인증된 경우
- POST /api/v1/fees/payments에 member_id와 amount 시
- 납부가 기록되고 미결 수수료에 적용됨
- 부분 납부 허용(납부 금액만큼 미결 감소)
- 응답에 새 미결 잔액 포함

---

## 에픽 6: 보고

### US-RPT-001: 연체 보고 보기
**역할** 사서, **원하는 것** 모든 연체 대출 보기, **이유** 회원에게 후속 조치하기 위해.

**인수 기준:**
- 사서 또는 관리자로 인증된 경우
- GET /api/v1/reports/overdue 시
- member_name, member_email, book_title, checkout_date, due_date, days_overdue가 포함된 연체 대출 목록 수신
- Member이면 403 FORBIDDEN

### US-RPT-002: 장서 요약 보기
**역할** 관리자, **원하는 것** 도서관 운영 요약 보기, **이유** 대시보드 뷰를 갖기 위해.

**인수 기준:**
- 관리자로 인증된 경우
- GET /api/v1/reports/summary 시
- total_books, total_members, books_checked_out, books_available, total_outstanding_fees 수신
- 관리자가 아니면 403 FORBIDDEN

---

## 에픽 7: 시스템 운영

### US-SYS-001: Catalog 헬스 체크
**역할** 모니터링 시스템, **원하는 것** Catalog Service 헬스 확인, **이유** 장애를 감지하기 위해.

**인수 기준:**
- GET /api/v1/catalog/health(공개 엔드포인트) 시
- 상태 및 버전 정보 수신

### US-SYS-002: Lending 헬스 체크
**역할** 모니터링 시스템, **원하는 것** Lending Service 헬스 확인, **이유** 장애를 감지하기 위해.

**인수 기준:**
- GET /api/v1/lending/health(공개 엔드포인트) 시
- 상태 및 버전 정보 수신

---

## 스토리-페르소나 매핑

| 스토리 | Admin | Librarian | Member | Public |
|-------|-------|-----------|--------|--------|
| US-CAT-001 도서 추가 | ✅ | ✅ | ❌ | ❌ |
| US-CAT-002 도서 수정 | ✅ | ✅ | ❌ | ❌ |
| US-CAT-003 도서 조회 | ✅ | ✅ | ✅ | ❌ |
| US-CAT-004 도서 목록 | ✅ | ✅ | ✅ | ❌ |
| US-CAT-005 도서 삭제 | ✅ | ✅ | ❌ | ❌ |
| US-CAT-006 도서 검색 | ✅ | ✅ | ✅ | ❌ |
| US-AUTH-001 등록 | ❌ | ❌ | ❌ | ✅ |
| US-AUTH-002 로그인 | ❌ | ❌ | ❌ | ✅ |
| US-AUTH-003 내 프로필 | ✅ | ✅ | ✅ | ❌ |
| US-AUTH-004 프로필 수정 | ✅ | ✅ | ✅ | ❌ |
| US-AUTH-005 임의 프로필 | ✅ | ✅ | ❌ | ❌ |
| US-AUTH-006 비활성화 | ✅ | ❌ | ❌ | ❌ |
| US-LND-001 대출 | ✅ | ✅ | ✅ | ❌ |
| US-LND-002 반납 | ✅ | ✅ | ✅(본인) | ❌ |
| US-LND-003 갱신 | ✅ | ✅ | ✅(본인) | ❌ |
| US-LND-004 활성 대출 | ✅(임의) | ✅(임의) | ✅(본인) | ❌ |
| US-HLD-001 예약 | ✅ | ✅ | ✅ | ❌ |
| US-HLD-002 예약 취소 | ✅(임의) | ✅(임의) | ✅(본인) | ❌ |
| US-HLD-003 예약 대기열 | ✅ | ✅ | ✅ | ❌ |
| US-HLD-004 내 예약 | ✅ | ✅ | ✅ | ❌ |
| US-FEE-001 내 수수료 | ✅ | ✅ | ✅ | ❌ |
| US-FEE-002 임의 수수료 | ✅ | ✅ | ❌ | ❌ |
| US-FEE-003 납부 | ✅ | ✅ | ❌ | ❌ |
| US-RPT-001 연체 | ✅ | ✅ | ❌ | ❌ |
| US-RPT-002 요약 | ✅ | ❌ | ❌ | ❌ |
| US-SYS-001 Catalog 헬스 | ✅ | ✅ | ✅ | ✅ |
| US-SYS-002 Lending 헬스 | ✅ | ✅ | ✅ | ✅ |
