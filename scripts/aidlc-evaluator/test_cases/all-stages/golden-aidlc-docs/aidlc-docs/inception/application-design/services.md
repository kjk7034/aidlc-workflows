# 서비스 설계

## Catalog Service — 서비스 계층

### BookService
- **책임**: 도서 CRUD, 검색, 가용성 관리 오케스트레이션
- **메서드**:
  - `create_book(data) -> Book` — 검증 후 도서 생성
  - `get_book(book_id) -> Book` — ID로 조회
  - `list_books() -> List[Book]` — 전체 목록
  - `update_book(book_id, data) -> Book` — 메타데이터 수정
  - `delete_book(book_id) -> None` — 삭제(활성 대출 시 실패)
  - `search_books(query, category, available) -> List[Book]` — 필터가 있는 전문 검색
  - `check_availability(book_id) -> AvailabilityInfo` — 서비스 간 호출용 가용성 반환
  - `update_availability(book_id, delta) -> Book` — available_copies 증감

---

## Lending Service — 서비스 계층

### AuthService
- **책임**: 인증 및 토큰 관리
- **메서드**:
  - `hash_password(password) -> str` — bcrypt 해시
  - `verify_password(plain, hashed) -> bool` — bcrypt 검증
  - `create_token(member) -> str` — member_id, email, role, 24시간 만료 JWT 생성
  - `decode_token(token) -> TokenPayload` — JWT 검증 및 디코드

### MemberService
- **책임**: 회원 생명주기 관리
- **메서드**:
  - `register(data) -> Member` — 해시된 비밀번호로 회원 생성, member 역할 자동 부여
  - `login(email, password) -> str` — 자격 증명 검증, JWT 반환
  - `get_profile(member_id) -> Member` — 프로필 조회
  - `update_profile(member_id, data) -> Member` — 이름/이메일 수정
  - `get_member(member_id) -> Member` — 관리자/사서 조회
  - `deactivate(member_id) -> Member` — active=False 설정

### CheckoutService
- **책임**: 대출, 반납, 갱신 오케스트레이션
- **메서드**:
  - `checkout(member_id, book_id) -> Checkout` — 한도/수수료/가용성 검증, 대출 생성, 가용성 감소
  - `return_book(checkout_id, member_id, role) -> ReturnResult` — 반납 처리, 수수료 계산, 가용성 증가, 예약 이행
  - `renew(checkout_id, member_id) -> Checkout` — 갱신 한도/예약 검증, 기한 연장
  - `list_checkouts(member_id, status) -> List[Checkout]` — 선택적 상태 필터로 대출 목록

### HoldService
- **책임**: 예약 대기열 관리
- **메서드**:
  - `place_hold(member_id, book_id) -> Hold` — 한도/가용성/중복 검증, FIFO 위치로 예약 생성
  - `cancel_hold(hold_id, member_id, role) -> None` — 취소 및 대기열 재정렬
  - `get_holds_for_book(book_id) -> List[Hold]` — 도서별 예약 대기열
  - `get_member_holds(member_id) -> List[Hold]` — 회원의 모든 예약
  - `fulfill_next_hold(book_id) -> Hold | None` — 반납 시 호출, 첫 대기 예약을 ready로 갱신

### FeeService
- **책임**: 수수료 계산 및 납부 처리
- **메서드**:
  - `calculate_late_fee(due_date, return_date) -> Decimal` — 일 $0.25, 대출당 상한 $10.00
  - `create_fee(member_id, checkout_id, amount) -> Fee` — 수수료 기록 생성
  - `get_member_fees(member_id) -> List[Fee]` — 회원 수수료 목록
  - `get_outstanding_balance(member_id) -> Decimal` — 미결제 합계
  - `process_payment(member_id, amount) -> Payment` — 납부 기록, 미결제 감소

### ReportService
- **책임**: 운영 보고
- **메서드**:
  - `get_overdue_checkouts() -> List[OverdueItem]` — 연체 전체, 회원 정보 및 연체 일수
  - `get_collection_summary() -> CollectionSummary` — 집계 통계

### CatalogClient
- **책임**: Catalog Service로의 HTTP 클라이언트
- **메서드**:
  - `check_availability(book_id) -> AvailabilityInfo` — GET /api/v1/books/{book_id}/availability
  - `decrement_availability(book_id) -> None` — POST /api/v1/books/{book_id}/availability, delta=-1
  - `increment_availability(book_id) -> None` — POST /api/v1/books/{book_id}/availability, delta=+1
