# 코드 생성 계획 — sci-calc

## 유닛 맥락
- **유닛**: sci-calc (단일 유닛 — 전체 애플리케이션)
- **프로젝트 유형**: Greenfield, Python 3.13, FastAPI
- **워크스페이스 루트**: workspace/
- **코드 위치**: workspace/ (tech-env 구조: pyproject.toml, src/sci_calc/, tests/)

## 단계 순서

### 1단계: 프로젝트 구조 설정
- [ ] hatchling 빌드 백엔드, 모든 의존성(fastapi, uvicorn, httpx, pytest, pytest-asyncio, pytest-cov, ruff), Python 3.13 요구사항으로 `workspace/pyproject.toml` 생성
- [ ] 버전과 함께 `workspace/src/sci_calc/__init__.py` 생성
- [ ] `workspace/src/sci_calc/routes/__init__.py` 생성
- [ ] `workspace/src/sci_calc/models/__init__.py` 생성
- [ ] `workspace/src/sci_calc/engine/__init__.py` 생성
- [ ] `workspace/tests/__init__.py` 생성
- [ ] 비동기 테스트 클라이언트 fixture가 있는 `workspace/tests/conftest.py` 생성

### 2단계: 엔진 — 사용자 정의 예외
- [ ] `workspace/src/sci_calc/engine/math_engine.py` 생성 — 사용자 정의 예외 정의: `MathDomainError`, `MathDivisionByZeroError`, `MathOverflowError`

### 3단계: 엔진 — 산술 연산
- [ ] `math_engine.py`에 산술 함수 추가: `add`, `subtract`, `multiply`, `divide`, `modulo`, `absolute`, `negate`
- [ ] 오버플로 검출 구현(결과가 inf/-inf → OverflowError 발생)

### 4단계: 엔진 — 거듭제곱과 루트
- [ ] `math_engine.py`에 거듭제곱 함수 추가: `power`, `sqrt_op`, `cbrt`, `square`, `nth_root`
- [ ] 정의역 검증 구현(음수의 sqrt, nth_root 제약)

### 5단계: 엔진 — 삼각함수
- [ ] `math_engine.py`에 삼각 함수 추가: `sin_op`, `cos_op`, `tan_op`, `asin_op`, `acos_op`, `atan_op`, `atan2_op`, `sinh_op`, `cosh_op`, `tanh_op`, `asinh_op`, `acosh_op`, `atanh_op`
- [ ] 각도 단위 변환 구현(도 ↔ 라디안)
- [ ] 역삼각함수에 대한 정의역 검증 구현

### 6단계: 엔진 — 로그 연산
- [ ] `math_engine.py`에 로그 함수 추가: `ln`, `log10_op`, `log2_op`, `log_op`, `exp_op`
- [ ] 정의역 검증 구현(a <= 0, 밑 제약)

### 7단계: 엔진 — 통계
- [ ] `math_engine.py`에 통계 함수 추가: `mean_op`, `median_op`, `mode_op`, `stdev_op`, `variance_op`, `pstdev_op`, `pvariance_op`, `min_op`, `max_op`, `sum_op`, `count_op`
- [ ] 최소 요소 개수 검증 구현

### 8단계: 엔진 — 상수
- [ ] `math_engine.py`에 상수 함수 추가: `get_constant`, `get_all_constants`
- [ ] 상수 맵 정의: pi, e, tau, inf, nan, golden_ratio, sqrt2, ln2, ln10

### 9단계: 엔진 — 단위 변환
- [ ] `math_engine.py`에 변환 함수 추가: `convert_angle`, `convert_temperature`, `convert_length`, `convert_weight`
- [ ] 지원 단위 전체에 대한 변환 계수 테이블 정의

### 10단계: 모델 — 요청 모델
- [ ] NaN 검증이 있는 모든 Pydantic v2 요청 모델로 `workspace/src/sci_calc/models/requests.py` 생성

### 11단계: 모델 — 응답 모델
- [ ] SuccessResponse, ErrorDetail, ErrorResponse 모델로 `workspace/src/sci_calc/models/responses.py` 생성

### 12단계: 라우트 — 산술
- [ ] 모든 산술 연산에 대한 POST 엔드포인트가 있는 APIRouter로 `workspace/src/sci_calc/routes/arithmetic.py` 생성

### 13단계: 라우트 — 거듭제곱
- [ ] 모든 거듭제곱/루트 연산에 대한 POST 엔드포인트가 있는 APIRouter로 `workspace/src/sci_calc/routes/powers.py` 생성

### 14단계: 라우트 — 삼각함수
- [ ] 모든 삼각 연산에 대한 POST 엔드포인트가 있는 APIRouter로 `workspace/src/sci_calc/routes/trigonometry.py` 생성

### 15단계: 라우트 — 로그
- [ ] 모든 로그 연산에 대한 POST 엔드포인트가 있는 APIRouter로 `workspace/src/sci_calc/routes/logarithmic.py` 생성

### 16단계: 라우트 — 통계
- [ ] 모든 통계 연산에 대한 POST 엔드포인트가 있는 APIRouter로 `workspace/src/sci_calc/routes/statistics.py` 생성

### 17단계: 라우트 — 상수
- [ ] 상수에 대한 GET 엔드포인트가 있는 APIRouter로 `workspace/src/sci_calc/routes/constants.py` 생성

### 18단계: 라우트 — 변환
- [ ] 모든 변환에 대한 POST 엔드포인트가 있는 APIRouter로 `workspace/src/sci_calc/routes/conversions.py` 생성

### 19단계: 애플리케이션 진입점
- [ ] FastAPI 앱, 모든 라우터 등록, 사용자 정의 오류 핸들러(422 재정의, 포괄), 헬스 체크 엔드포인트로 `workspace/src/sci_calc/app.py` 생성

### 20단계: 테스트 — 엔진 단위 테스트
- [ ] 산술 엔진 함수 + 경계 테스트용 `workspace/tests/test_arithmetic.py` 생성
- [ ] 거듭제곱 엔진 함수 + 정의역 오류 테스트용 `workspace/tests/test_powers.py` 생성
- [ ] 삼각 엔진 함수 + 정의역 오류 테스트용 `workspace/tests/test_trigonometry.py` 생성
- [ ] 로그 엔진 함수 + 정의역 오류 테스트용 `workspace/tests/test_logarithmic.py` 생성
- [ ] 통계 엔진 함수 + 엣지 케이스용 `workspace/tests/test_statistics.py` 생성
- [ ] 상수 + API 통합 테스트용 `workspace/tests/test_constants.py` 생성
- [ ] 변환 함수 + API 통합 테스트용 `workspace/tests/test_conversions.py` 생성

### 21단계: 테스트 — API 통합 테스트
- [ ] httpx.AsyncClient를 사용해 각 테스트 파일에 통합 테스트 추가
- [ ] 성공 응답이 래퍼 구조와 일치하는지 테스트
- [ ] 오류 응답이 오류 래퍼 구조와 일치하는지 테스트
- [ ] 알 수 없는 엔드포인트에 대해 404 테스트
- [ ] 사용자 정의 422 핸들러가 Pydantic 오류를 감싸는지 테스트

## 합계: 21단계, 소스 파일 약 20개, 테스트 파일 약 7개
