# 기능 설계 — Lending Service — 비즈니스 규칙

## 인증 규칙

### BR-AUTH-001: 등록
- 이름 필수, 최대 100자
- 이메일 필수, 유효한 형식, 전체 회원에서 고유
- 비밀번호 필수, 최소 8자
- 비밀번호는 bcrypt 해시로 저장(평문 절대 금지)
- 자동 부여 역할: "member"
- 계정은 활성으로 생성(is_active = true)

### BR-AUTH-002: 로그인
- 이메일 존재 및 계정 활성 검증
- bcrypt 해시로 비밀번호 검증
- JWT에 반환: member_id, email, role, exp(현재부터 24시간)
- 잘못된 자격 증명은 401 반환(이메일 오류인지 비밀번호 오류인지 노출하지 않음)

### BR-AUTH-003: JWT 검증
- 서명, 만료, 필수 클레임(member_id, email, role) 검증
- 만료된 토큰은 401로 거부
- 유효한 토큰에서 member_id, email, role 추출

## 회원 규칙

### BR-MEM-001: 프로필 수정
- 회원은 자신의 이름과 이메일만 수정 가능
- 새 이메일은 고유해야 함
- 프로필 수정으로 역할 또는 활성 상태 변경 불가

### BR-MEM-002: 계정 비활성화
- Admin만 계정 비활성화 가능
- 비활성 회원: is_active = false
- 기존 대출은 반납까지 유지
- 기존 예약은 유지
- 차단 동작: 신규 대출, 신규 예약, 신규 갱신
- 자기 자신 비활성화 불가

## 대출 규칙

### BR-CHK-001: 대출 검증
1. 도서가 존재해야 함(Catalog Service로 검증)
2. available_copies > 0이어야 함
3. 회원이 활성이어야 함(is_active = true)
4. 회원의 활성 대출 수 < MAX_CHECKOUTS (5)
5. 회원 미결제 수수료 잔액 <= FEE_THRESHOLD ($10.00)
6. 대출 생성 전 모든 검증이 통과해야 함

### BR-CHK-002: 대출 생성
- checkout_date = now (UTC)
- due_date = checkout_date + 14일
- status = "active"
- renewal_count = 0
- Catalog Service에서 available_copies 감소
- Catalog Service 호출이 실패하면 대출 중단

### BR-CHK-003: 반납 처리
1. 대출이 존재하고 활성이어야 함
2. 회원은 자신의 대출만 반납 가능; 사서/관리자는 모든 대출 반납 가능
3. return_date = now (UTC), status = "returned"
4. 연체 시 연체료 계산:
   - days_overdue = (return_date.date() - due_date.date()).days(양수일 때만)
   - fee = days_overdue * $0.25
   - 대출당 수수료 상한 $10.00
   - fee > 0이면 수수료 기록 생성
5. Catalog Service에서 available_copies 증가
6. 예약 대기열 확인: 해당 도서에 대기 예약이 있으면 다음 이행

### BR-CHK-004: 갱신
1. 대출이 존재하고 활성이어야 함
2. 요청 회원의 대출이어야 함
3. 회원이 활성이어야 함
4. renewal_count < MAX_RENEWALS (2)
5. 해당 도서에 대해 status = "waiting"인 활성 예약이 없어야 함
6. 현재 due_date부터 due_date를 14일 연장
7. renewal_count 증가

## 예약 규칙

### BR-HLD-001: 예약 등록
1. 도서가 존재해야 함(Catalog Service로 검증)
2. 도서 available_copies는 0이어야 함(예약은 대여 불가 도서에만)
3. 회원이 활성이어야 함
4. 회원 활성 예약 수 < MAX_HOLDS (3) — status가 "waiting" 또는 "ready"인 예약만 집계
5. 해당 도서에 이미 활성 예약이 있으면 안 됨
6. status = "waiting"으로 예약 생성
7. 대기열 위치 = 해당 도서의 최대 위치 + 1(FIFO)

### BR-HLD-002: 예약 취소
1. 예약이 존재해야 함
2. 회원은 자신의 예약만 취소; 사서/관리자는 모든 예약 취소 가능
3. status = "cancelled"
4. 대기열 재정렬: 같은 도서에서 더 높은 위치의 모든 예약의 위치를 1 감소

### BR-HLD-003: 예약 이행(반납 시)
1. 도서가 반납되면 해당 도서의 대기 예약을 확인
2. queue_position이 가장 작고 status = "waiting"인 예약을 찾음
3. 해당 예약의 status를 "ready"로 갱신
4. available_copies를 감소시키지 않음("ready" 예약이 개념적으로 부본을 예약함)
   - 실제로: 반납 시 available_copies는 이미 증가했음. "ready" 회원이 정상 대출하면 다시 감소함. 따라서 available_copies는 물리적 가용성을 정확히 반영함.

## 수수료 규칙

### BR-FEE-001: 연체료 계산
- 요율: 연체일당 $0.25
- 상한: 대출당 $10.00
- 반납 시에만 부과, 실시간 누적 아님
- UTC 날짜(전일 단위)로 계산

### BR-FEE-002: 수수료 납부
- 사서/관리자가 납부 처리
- 부분 납부 허용
- 납부는 가장 오래된 미결제 수수료부터 적용
- 납부 금액은 0보다 커야 함
- 납부는 총 미결제 잔액을 초과할 수 없음

### BR-FEE-003: 미결제 잔액
- status = "outstanding"인 모든 수수료 합에서 적용된 납부를 뺀 값
- 대출 임계값 검사에 사용(BR-CHK-001 규칙 5)

## 보고 규칙

### BR-RPT-001: 연체 보고
- status = "active"이고 due_date < now인 모든 대출
- 포함: 회원 이름, 이메일, 도서 제목, checkout_date, due_date, days_overdue
- 도서 제목은 Catalog Service에서 조회하거나 대출에 비정규화 저장
- 사서 및 관리자만 접근

### BR-RPT-002: 장서 요약
- total_books: Catalog Service에서 개수
- total_members: 전체 회원 수
- books_checked_out: 활성 대출 수
- books_available: total_books - books_checked_out
- total_outstanding_fees: 미결제 수수료 금액 합
- 관리자만 접근

## 설정 상수

| 상수 | 값 | 설명 |
|----------|-------|-------------|
| MAX_CHECKOUTS | 5 | 회원당 최대 활성 대출 수 |
| MAX_RENEWALS | 2 | 대출당 최대 갱신 횟수 |
| MAX_HOLDS | 3 | 회원당 최대 활성 예약 수 |
| LOAN_PERIOD_DAYS | 14 | 대출 기간 일수 |
| LATE_FEE_PER_DAY | 0.25 | 일일 연체료(달러) |
| LATE_FEE_CAP | 10.00 | 대출당 최대 연체료 |
| FEE_THRESHOLD | 10.00 | 대출 허용 미결제 수수료 한도 |
| JWT_EXPIRY_HOURS | 24 | JWT 토큰 만료(시간) |
