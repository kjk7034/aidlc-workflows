# 컴포넌트 메서드

## 1. 엔진 — `math_engine.py`

### 산술
- `add(a: float, b: float) -> float`
- `subtract(a: float, b: float) -> float`
- `multiply(a: float, b: float) -> float`
- `divide(a: float, b: float) -> float` — DivisionByZeroError 발생
- `modulo(a: float, b: float) -> float` — DivisionByZeroError 발생
- `absolute(a: float) -> float`
- `negate(a: float) -> float`

### 거듭제곱
- `power(base: float, exponent: float) -> float` — OverflowError 발생
- `sqrt(a: float) -> float` — a < 0이면 DomainError
- `cbrt(a: float) -> float`
- `square(a: float) -> float`
- `nth_root(a: float, n: int) -> float` — a < 0이고 n이 짝수이면 DomainError

### 삼각함수
- `sin(a: float, angle_unit: str) -> float`
- `cos(a: float, angle_unit: str) -> float`
- `tan(a: float, angle_unit: str) -> float`
- `asin(a: float, angle_unit: str) -> float` — |a| > 1이면 DomainError
- `acos(a: float, angle_unit: str) -> float` — |a| > 1이면 DomainError
- `atan(a: float, angle_unit: str) -> float`
- `atan2(y: float, x: float, angle_unit: str) -> float`
- `sinh(a: float) -> float`
- `cosh(a: float) -> float`
- `tanh(a: float) -> float`
- `asinh(a: float) -> float`
- `acosh(a: float) -> float` — a < 1이면 DomainError
- `atanh(a: float) -> float` — |a| >= 1이면 DomainError

### 로그
- `ln(a: float) -> float` — a <= 0이면 DomainError
- `log10(a: float) -> float` — a <= 0이면 DomainError
- `log2(a: float) -> float` — a <= 0이면 DomainError
- `log(a: float, base: float) -> float` — DomainError 발생
- `exp(a: float) -> float` — OverflowError 발생

### 통계
- `mean(values: list[float]) -> float`
- `median(values: list[float]) -> float`
- `mode(values: list[float]) -> float` — 동점이면 가장 작은 값
- `stdev(values: list[float]) -> float` — len >= 2 필요
- `variance(values: list[float]) -> float` — len >= 2 필요
- `pstdev(values: list[float]) -> float`
- `pvariance(values: list[float]) -> float`
- `min_val(values: list[float]) -> float`
- `max_val(values: list[float]) -> float`
- `sum_val(values: list[float]) -> float`
- `count(values: list[float]) -> int`

### 상수
- `get_constant(name: str) -> float`
- `get_all_constants() -> dict[str, float]`

### 변환
- `convert_angle(value: float, from_unit: str, to_unit: str) -> float`
- `convert_temperature(value: float, from_unit: str, to_unit: str) -> float`
- `convert_length(value: float, from_unit: str, to_unit: str) -> float`
- `convert_weight(value: float, from_unit: str, to_unit: str) -> float`

## 2. 모델 — `requests.py`

### 요청 모델
- `BinaryOperationRequest(a: float, b: float)` — NaN 검증기 포함
- `UnaryOperationRequest(a: float)` — NaN 검증기 포함
- `PowerRequest(base: float, exponent: float)` — NaN 검증기 포함
- `NthRootRequest(a: float, n: int)` — NaN 검증기 포함
- `TrigRequest(a: float, angle_unit: str = "radians")` — NaN 검증기 포함
- `Atan2Request(y: float, x: float, angle_unit: str = "radians")` — NaN 검증기 포함
- `LogRequest(a: float, base: float)` — NaN 검증기 포함
- `StatisticsRequest(values: list[float])` — NaN 검증기 포함
- `ConversionRequest(value: float, from_unit: str, to_unit: str)` — NaN 검증기 포함

## 3. 모델 — `responses.py`

### 응답 모델
- `SuccessResponse(status: str, operation: str, inputs: dict, result: Any)`
- `ErrorDetail(code: str, message: str)`
- `ErrorResponse(status: str, operation: str, inputs: dict, error: ErrorDetail)`

## 4. 라우트 — 각 라우트 모듈
- URL 접두사가 있는 모듈당 하나의 `APIRouter`
- 라우트 함수: 요청 파싱 → 엔진 호출 → SuccessResponse 또는 ErrorResponse 반환
- 예외 핸들러가 엔진 예외를 잡아 오류 코드로 매핑
