# 서비스

## 서비스 계층

애플리케이션은 `app.py`가 오케스트레이터 역할을 하는 **얇은 서비스 계층** 패턴을 사용합니다:

### `app.py` — 애플리케이션 오케스트레이터
**패턴**: FastAPI 애플리케이션 팩토리
**책임**:
1. **라우터 등록**: 모든 라우트 모듈을 URL 접두사와 함께 마운트
2. **오류 핸들러 등록**: Pydantic 422 핸들러 재정의, 포괄 예외 핸들러 추가
3. **헬스 체크**: `GET /health`를 직접 제공
4. **미들웨어**: MVP에는 없음(인증·속도 제한 없음)

### 라우트에서 엔진으로 위임
각 라우트 핸들러는 일관된 패턴을 따릅니다:
1. 검증된 Pydantic 요청 모델 수신
2. 적절한 `math_engine` 함수 호출
3. 결과를 `SuccessResponse` 래퍼로 감쌈
4. 예외 시 잡아 `ErrorResponse` 래퍼 반환

**별도 서비스 클래스는 없습니다** — 라우트가 엔진 함수를 직접 호출합니다. 무상태 계산, 영속성 없음, 사용자 세션 없음, 횡단 비즈니스 로직이 없는 프로젝트 범위에 적합합니다.

### 오류 처리 흐름
1. Pydantic 검증 오류 → 사용자 정의 422 핸들러 → `ErrorResponse(code="INVALID_INPUT")`
2. `DivisionByZeroError` → 라우트 핸들러 → `ErrorResponse(code="DIVISION_BY_ZERO", status=400)`
3. `DomainError` → 라우트 핸들러 → `ErrorResponse(code="DOMAIN_ERROR", status=400)`
4. `OverflowError` → 라우트 핸들러 → `ErrorResponse(code="OVERFLOW", status=400)`
5. 알 수 없는 엔드포인트 → FastAPI 404 → 사용자 정의 핸들러 → `ErrorResponse(code="NOT_FOUND", status=404)`
6. 예상치 못한 예외 → 포괄 핸들러 → ERROR로 로깅 → `ErrorResponse(code="INTERNAL_ERROR", status=500)`
