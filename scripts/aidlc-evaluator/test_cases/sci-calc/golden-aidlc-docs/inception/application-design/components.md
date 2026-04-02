# 컴포넌트

## 1. 애플리케이션 진입점 — `sci_calc/app.py`
**목적**: FastAPI 애플리케이션 팩토리, 미들웨어, 오류 핸들러, 라우터 등록.
**책임**:
- 메타데이터(title, version)와 함께 FastAPI 앱 인스턴스 생성
- 모든 라우트 모듈 등록
- 표준 오류 래퍼를 쓰도록 기본 검증 오류 핸들러 재정의
- 전역 예외 핸들러 등록(예상치 못한 오류용)
- 헬스 체크 엔드포인트

## 2. Routes 계층 — `sci_calc/routes/`

### 2.1 `arithmetic.py`
**목적**: 산술 연산 요청 처리.
**책임**: 입력 파싱, 엔진에 위임, 성공/오류 래퍼로 결과 감싸기.

### 2.2 `powers.py`
**목적**: 거듭제곱 및 루트 연산 요청 처리.
**책임**: 입력 파싱, 정의역 제약 검증, 엔진에 위임.

### 2.3 `trigonometry.py`
**목적**: 삼각함수 연산 요청 처리.
**책임**: 입력 파싱, angle_unit 변환 처리, 엔진에 위임.

### 2.4 `logarithmic.py`
**목적**: 로그 연산 요청 처리.
**책임**: 입력 파싱, 정의역 제약 검증, 엔진에 위임.

### 2.5 `statistics.py`
**목적**: 통계 연산 요청 처리.
**책임**: 입력 파싱, 리스트 크기 제약 검증, 엔진에 위임.

### 2.6 `constants.py`
**목적**: 수학 상수 제공.
**책임**: 개별 또는 전체 상수 반환.

### 2.7 `conversions.py`
**목적**: 단위 변환 요청 처리.
**책임**: 입력 파싱, 단위 검증, 엔진에 위임.

## 3. Models 계층 — `sci_calc/models/`

### 3.1 `requests.py`
**목적**: 모든 연산에 대한 Pydantic v2 요청 모델.
**책임**: Pydantic을 통한 입력 검증, 타입 강제, NaN 거부.

### 3.2 `responses.py`
**목적**: Pydantic v2 응답 모델(성공 및 오류 래퍼).
**책임**: 표준 응답 구조 정의, 오류 코드 열거형.

## 4. Engine 계층 — `sci_calc/engine/`

### 4.1 `math_engine.py`
**목적**: 순수 계산 로직 — HTTP/FastAPI 의존성 없음.
**책임**:
- 산술 연산(add, subtract, multiply, divide, modulo, abs, negate)
- 거듭제곱 연산(power, sqrt, cbrt, square, nth_root)
- 삼각함수 연산(각도 단위 지원을 포함한 삼각함수 14개 전부)
- 로그 연산(ln, log10, log2, log, exp)
- 통계 연산(mean, median, mode, stdev, variance 등)
- 상수 조회
- 단위 변환(각도, 온도, 길이, 무게)
- 정의역별 예외 발생(DomainError, DivisionByZeroError, OverflowError)
