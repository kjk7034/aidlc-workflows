# 기능 설계 — Lending Service — 도메인 엔티티

## 엔티티: Member

| 필드 | 타입 | 제약 | 설명 |
|-------|------|-------------|-------------|
| id | str (UUID) | 자동 생성 | 고유 식별자 |
| name | str | 필수, 최대 100자 | 회원 전체 이름 |
| email | str | 필수, 고유, 최대 255자, 유효한 이메일 형식 | 로그인 이메일 |
| password_hash | str | 시스템 관리 | 비밀번호의 Bcrypt 해시 |
| role | str (enum) | "admin", "librarian", "member" | RBAC 역할 |
| is_active | bool | 기본값: true | 계정 활성 상태 |
| created_at | datetime (UTC) | 자동 생성 | 가입 시각 |

## 엔티티: Checkout

| 필드 | 타입 | 제약 | 설명 |
|-------|------|-------------|-------------|
| id | str (UUID) | 자동 생성 | 고유 식별자 |
| member_id | str | 필수, Member FK | 대출 회원 |
| book_id | str | 필수 | Catalog Service의 도서 |
| checkout_date | datetime (UTC) | 자동 생성 | 대출 시각 |
| due_date | datetime (UTC) | checkout_date + 14일 | 반납 기한 |
| return_date | datetime (UTC) | 반납 전까지 Null | 반납 시각 |
| status | str (enum) | "active", "returned" | 대출 상태 |
| renewal_count | int | 기본값: 0, 최대 2 | 갱신 횟수 |

## 엔티티: Hold

| 필드 | 타입 | 제약 | 설명 |
|-------|------|-------------|-------------|
| id | str (UUID) | 자동 생성 | 고유 식별자 |
| member_id | str | 필수, Member FK | 예약 요청 회원 |
| book_id | str | 필수 | Catalog Service의 도서 |
| hold_date | datetime (UTC) | 자동 생성 | 예약 시각 |
| status | str (enum) | "waiting", "ready", "cancelled", "fulfilled" | 예약 상태 |
| queue_position | int | >= 1 | FIFO 대기열 위치 |

## 엔티티: Fee

| 필드 | 타입 | 제약 | 설명 |
|-------|------|-------------|-------------|
| id | str (UUID) | 자동 생성 | 고유 식별자 |
| member_id | str | 필수, Member FK | 수수료 부담 회원 |
| checkout_id | str | 필수, Checkout FK | 관련 대출 |
| amount | Decimal | > 0, 최대 $10.00 | 수수료 금액 |
| status | str (enum) | "outstanding", "paid" | 수수료 상태 |
| created_at | datetime (UTC) | 자동 생성 | 수수료 생성 시각 |

## 엔티티: Payment

| 필드 | 타입 | 제약 | 설명 |
|-------|------|-------------|-------------|
| id | str (UUID) | 자동 생성 | 고유 식별자 |
| member_id | str | 필수, Member FK | 결제 회원 |
| amount | Decimal | > 0 | 결제 금액 |
| payment_date | datetime (UTC) | 자동 생성 | 결제 처리 시각 |

## 엔티티: TokenPayload (값 객체)

| 필드 | 타입 | 설명 |
|-------|------|-------------|
| member_id | str | 회원 UUID |
| email | str | 회원 이메일 |
| role | str | 회원 역할 |
| exp | datetime | 토큰 만료 |

## 엔티티: ReturnResult (값 객체)

| 필드 | 타입 | 설명 |
|-------|------|-------------|
| checkout | Checkout | 갱신된 대출 기록 |
| fee | Fee | None | 해당 시 연체료 |
| hold_fulfilled | Hold | None | 해당 시 이행된 예약 |
