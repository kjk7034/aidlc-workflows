# 컴포넌트 메서드

## Catalog Service

### BookService 메서드

| 메서드 | 입력 | 출력 | 목적 |
|--------|-------|--------|---------|
| `create_book` | `BookCreateRequest(title, author, isbn, category, total_copies)` | `Book` | 도서 생성, available_copies = total_copies 설정 |
| `get_book` | `book_id: str` | `Book` | ID로 조회 또는 NOT_FOUND |
| `list_books` | None | `List[Book]` | 전체 도서 반환 |
| `update_book` | `book_id: str, BookUpdateRequest` | `Book` | 필드 갱신, total_copies >= 대출 중 권수 검증 |
| `delete_book` | `book_id: str` | `None` | 활성 대출 없을 때 삭제 |
| `search_books` | `query: str, category: str, available: bool` | `List[Book]` | 제목/저자 부분 일치, 선택 필터 |
| `check_availability` | `book_id: str` | `AvailabilityInfo(book_id, title, total_copies, available_copies)` | Lending Service용 내부 API |
| `update_availability` | `book_id: str, delta: int` | `Book` | available_copies를 delta(+1 또는 -1)만큼 조정 |

### BookRepository 메서드

| 메서드 | 입력 | 출력 | 목적 |
|--------|-------|--------|---------|
| `create` | `Book` | `Book` | 새 도서 저장 |
| `get_by_id` | `book_id: str` | `Book | None` | 기본 키로 조회 |
| `list_all` | None | `List[Book]` | 전체 레코드 목록 |
| `update` | `Book` | `Book` | 기존 레코드 갱신 |
| `delete` | `book_id: str` | `None` | 레코드 삭제 |
| `search` | `query: str, category: str, available: bool` | `List[Book]` | 필터 검색 |

---

## Lending Service

### AuthService 메서드

| 메서드 | 입력 | 출력 | 목적 |
|--------|-------|--------|---------|
| `hash_password` | `password: str` | `str` | Bcrypt 해시 |
| `verify_password` | `plain: str, hashed: str` | `bool` | 비밀번호 검증 |
| `create_token` | `member_id: str, email: str, role: str` | `str` | 24시간 만료 JWT |
| `decode_token` | `token: str` | `TokenPayload(member_id, email, role)` | 검증 및 디코드 |

### MemberService / MemberRepository 메서드

| 메서드 | 입력 | 출력 | 목적 |
|--------|-------|--------|---------|
| `register` | `MemberRegisterRequest(name, email, password)` | `Member` | bcrypt 해시로 생성 |
| `login` | `email: str, password: str` | `str (JWT)` | 인증 후 토큰 반환 |
| `get_profile` | `member_id: str` | `Member` | 본인 프로필 |
| `update_profile` | `member_id: str, data` | `Member` | 이름/이메일 갱신 |
| `deactivate` | `member_id: str` | `Member` | is_active = False |

### CheckoutService 메서드

| 메서드 | 입력 | 출력 | 목적 |
|--------|-------|--------|---------|
| `checkout` | `member_id: str, book_id: str` | `Checkout` | 전체 검증 + 생성 |
| `return_book` | `checkout_id: str, member_id: str, role: str` | `ReturnResult` | 반납 + 수수료 + 예약 |
| `renew` | `checkout_id: str, member_id: str` | `Checkout` | 기한 연장 |
| `list_checkouts` | `member_id: str, status: str` | `List[Checkout]` | 상태 필터 |

### HoldService 메서드

| 메서드 | 입력 | 출력 | 목적 |
|--------|-------|--------|---------|
| `place_hold` | `member_id: str, book_id: str` | `Hold` | 검증 + FIFO 생성 |
| `cancel_hold` | `hold_id: str, member_id: str, role: str` | `None` | 취소 + 재정렬 |
| `get_holds_for_book` | `book_id: str` | `List[Hold]` | 도서별 대기열 |
| `get_member_holds` | `member_id: str` | `List[Hold]` | 회원 예약 |
| `fulfill_next_hold` | `book_id: str` | `Hold | None` | 첫 대기 → ready |

### FeeService 메서드

| 메서드 | 입력 | 출력 | 목적 |
|--------|-------|--------|---------|
| `calculate_late_fee` | `due_date, return_date` | `Decimal` | 일 $0.25, 상한 $10 |
| `create_fee` | `member_id, checkout_id, amount` | `Fee` | 수수료 기록 |
| `get_member_fees` | `member_id: str` | `List[Fee]` | 수수료 목록 |
| `get_outstanding_balance` | `member_id: str` | `Decimal` | 미결제 합계 |
| `process_payment` | `member_id, amount` | `Payment` | 납부 적용 |
