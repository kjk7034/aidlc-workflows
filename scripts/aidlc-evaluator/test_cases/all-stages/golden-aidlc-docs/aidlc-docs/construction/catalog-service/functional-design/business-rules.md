# 기능 설계 — Catalog Service — 비즈니스 규칙

## BR-CAT-001: 도서 생성
- 제목, 저자, 카테고리는 필수
- ISBN은 선택
- total_copies는 1 이상이어야 함
- 생성 시 available_copies는 total_copies로 설정
- 문자열 필드는 최대 길이 제약을 준수해야 함

## BR-CAT-002: 도서 수정
- 제공된 필드만 부분 업데이트
- total_copies가 변경되면 새 값은 (total_copies - available_copies) 이상이어야 함
  - 즉, 현재 대출 중인 권수 아래로 줄일 수 없음
- total_copies가 증가하면 available_copies도 같은 델타만큼 증가
- total_copies가 감소하면 available_copies도 같은 델타만큼 감소(단 0 미만은 불가)

## BR-CAT-003: 도서 삭제
- 활성 대출이 있는 도서(available_copies < total_copies)는 삭제 불가
- 삭제가 막히면 409 CONFLICT 반환

## BR-CAT-004: 도서 검색
- 검색어는 제목 또는 저자 부분 문자열과 일치(대소문자 무시)
- 카테고리 필터는 정확 일치(대소문자 무시)
- Available 필터: true이면 available_copies > 0인 도서만 반환
- 모든 필터는 선택이며 조합 가능
- 결과가 없으면 빈 목록 반환(오류 아님)

## BR-CAT-005: 가용성 업데이트
- 델타 -1: available_copies 감소(대출)
  - available_copies == 0이면 CONFLICT로 실패
- 델타 +1: available_copies 증가(반납)
  - available_copies == total_copies이면 CONFLICT로 실패
- 델타는 -1과 +1만 허용

## BR-CAT-006: RBAC 규칙
- **Public**: 헬스 체크만
- **Member**: 읽기 작업(get, list, search)
- **Librarian**: 모든 CRUD 작업
- **Admin**: 모든 작업
- **Internal**: 가용성 엔드포인트(서비스 간, 공유 비밀 또는 JWT로 검증)
