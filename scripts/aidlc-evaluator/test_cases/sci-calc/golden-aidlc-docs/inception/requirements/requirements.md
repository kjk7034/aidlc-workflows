# 요구사항: Scientific Calculator API

## 의도 분석

| 속성 | 값 |
|---|---|
| **사용자 요청** | 과학 수학 연산을 위한 무상태 HTTP API 구축 |
| **요청 유형** | 신규 프로젝트 (greenfield) |
| **범위 추정** | 단일 애플리케이션 — 여러 컴포넌트(routes, models, engine) |
| **복잡도 추정** | 보통 — 잘 정의된 API 표면, 많은 연산, 철저한 오류 처리 |
| **요구사항 깊이** | 표준 |

---

## 1. 기능 요구사항

### FR-1: 헬스 체크
- **FR-1.1**: `GET /health`는 HTTP 200으로 `{"status": "ok", "version": "0.1.0"}`을 반환합니다.

### FR-2: 산술 연산
- **FR-2.1**: `POST /api/v1/arithmetic/{operation}`는 다음을 지원합니다: `add`, `subtract`, `multiply`, `divide`, `modulo`, `abs`, `negate`.
- **FR-2.2**: 이항 연산(`add`, `subtract`, `multiply`, `divide`, `modulo`)은 `{"a": N, "b": N}`을 받습니다.
- **FR-2.3**: 단항 연산(`abs`, `negate`)은 `{"a": N}`을 받습니다.
- **FR-2.4**: `divide`와 `modulo`는 `b == 0`일 때 `DIVISION_BY_ZERO` 오류를 반환합니다.

### FR-3: 거듭제곱과 루트
- **FR-3.1**: `POST /api/v1/powers/{operation}`는 다음을 지원합니다: `power`, `sqrt`, `cbrt`, `square`, `nth_root`.
- **FR-3.2**: `power`는 `{"base": N, "exponent": N}`을 받습니다.
- **FR-3.3**: `sqrt`, `cbrt`, `square`는 `{"a": N}`을 받습니다.
- **FR-3.4**: `nth_root`는 `{"a": N, "n": int}`를 받습니다.
- **FR-3.5**: `sqrt`는 `a < 0`일 때 `DOMAIN_ERROR`를 반환합니다.
- **FR-3.6**: `nth_root`는 `a < 0`이고 `n`이 짝수일 때 `DOMAIN_ERROR`를 반환합니다.

### FR-4: 삼각함수 연산
- **FR-4.1**: `POST /api/v1/trigonometry/{operation}`는 다음을 지원합니다: `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `sinh`, `cosh`, `tanh`, `asinh`, `acosh`, `atanh`.
- **FR-4.2**: 대부분의 연산은 기본값 `"radians"`로 `{"a": N, "angle_unit": "radians"|"degrees"}`를 받습니다.
- **FR-4.3**: `atan2`는 `{"y": N, "x": N, "angle_unit": "radians"|"degrees"}`를 받습니다.
- **FR-4.4**: 정의역 제약: `asin`/`acos`는 `-1 <= a <= 1`, `acosh`는 `a >= 1`, `atanh`는 `-1 < a < 1`.
- **FR-4.5**: 정의역 위반은 `DOMAIN_ERROR`를 반환합니다.

### FR-5: 로그 연산
- **FR-5.1**: `POST /api/v1/logarithmic/{operation}`는 다음을 지원합니다: `ln`, `log10`, `log2`, `log`, `exp`.
- **FR-5.2**: `ln`, `log10`, `log2`는 `{"a": N}`을 받습니다 — `a <= 0`이면 `DOMAIN_ERROR`.
- **FR-5.3**: `log`는 `{"a": N, "base": N}`을 받습니다 — `a <= 0`, `base <= 0`, 또는 `base == 1`이면 `DOMAIN_ERROR`.
- **FR-5.4**: `exp`는 `{"a": N}`을 받습니다.

### FR-6: 통계 연산
- **FR-6.1**: `POST /api/v1/statistics/{operation}`는 다음을 지원합니다: `mean`, `median`, `mode`, `stdev`, `variance`, `pstdev`, `pvariance`, `min`, `max`, `sum`, `count`.
- **FR-6.2**: 모두 최소 1개 요소가 있는 `{"values": [N, ...]}`를 받습니다.
- **FR-6.3**: `stdev`/`variance`는 최소 2개 요소가 필요합니다.
- **FR-6.4**: `pstdev`/`pvariance`는 최소 1개 요소가 필요합니다.
- **FR-6.5**: `mode`는 단일 숫자 값을 반환합니다; 동점이면 가장 작은 최빈값을 반환합니다.

### FR-7: 수학 상수
- **FR-7.1**: `GET /api/v1/constants/{name}`는 이름이 지정된 상수를 반환합니다.
- **FR-7.2**: `GET /api/v1/constants`는 모든 상수를 맵으로 반환합니다.
- **FR-7.3**: 지원 상수: `pi`, `e`, `tau`, `inf`, `nan`, `golden_ratio`, `sqrt2`, `ln2`, `ln10`.

### FR-8: 단위 변환
- **FR-8.1**: `POST /api/v1/conversions/{category}`는 `{"value": N, "from_unit": "...", "to_unit": "..."}`를 받습니다.
- **FR-8.2**: 각도: `degrees`, `radians`, `gradians`.
- **FR-8.3**: 온도: `celsius`, `fahrenheit`, `kelvin`.
- **FR-8.4**: 길이: `meters`, `feet`, `inches`, `centimeters`, `millimeters`, `kilometers`, `miles`, `yards`.
- **FR-8.5**: 무게: `kilograms`, `pounds`, `ounces`, `grams`, `milligrams`, `tonnes`, `stones`.
- **FR-8.6**: 알 수 없는 단위는 `INVALID_INPUT`(422)을 반환합니다.

### FR-9: 응답 래퍼
- **FR-9.1**: 성공 응답: `{"status": "ok", "operation": "<name>", "inputs": {...}, "result": <number|object>}`.
- **FR-9.2**: 오류 응답: `{"status": "error", "operation": "<name>", "inputs": {...}, "error": {"code": "<CODE>", "message": "..."}}`.
- **FR-9.3**: 모든 엔드포인트는 `application/json`을 받고 반환합니다.

### FR-10: 오류 처리
- **FR-10.1**: 오류 코드: `INVALID_INPUT`(422), `DIVISION_BY_ZERO`(400), `DOMAIN_ERROR`(400), `OVERFLOW`(400), `NOT_FOUND`(404).
- **FR-10.2**: Pydantic 검증 오류는 `INVALID_INPUT` 코드로 동일한 오류 래퍼에 담습니다.
- **FR-10.3**: 결과가 `inf` 또는 `-inf`가 될 경우 `OVERFLOW` 오류를 반환합니다.
- **FR-10.4**: `NaN` 입력은 `INVALID_INPUT` 오류로 거부합니다.
- **FR-10.5**: 알 수 없는 엔드포인트는 `NOT_FOUND`를 반환합니다.
- **FR-10.6**: 예상치 못한 예외는 ERROR 수준으로 로깅하고 일반 `INTERNAL_ERROR` 응답을 반환합니다; 맨 500은 반환하지 않습니다.

---

## 2. 비기능 요구사항

### NFR-1: 성능
- **NFR-1.1**: 시작 시간 2초 미만.
- **NFR-1.2**: 단일 연산에 대해 응답 지연 p95 50ms 미만.

### NFR-2: 정확성
- **NFR-2.1**: 표준 연산에 대해 결과가 Python `math` 표준 라이브러리와 1 ULP 이하로 일치합니다.

### NFR-3: 테스트
- **NFR-3.1**: 모든 테스트가 라인 커버리지 90% 이상으로 통과합니다(강제 — 90% 미만이면 테스트 실행 실패).
- **NFR-3.2**: 단위 테스트는 알려진 값 테이블로 `math_engine.py`를 직접 검증합니다.
- **NFR-3.3**: 통합 테스트는 FastAPI TestClient와 `httpx.AsyncClient`를 사용합니다.
- **NFR-3.4**: 경계 테스트로 모든 정의역 제약이 올바른 오류 코드를 내는지 검증합니다.

### NFR-4: 보안
- **NFR-4.1**: 최대 요청 본문 크기 1 MB.
- **NFR-4.2**: MVP에는 인증, 속도 제한, 프로덕션 하드닝이 없습니다.

### NFR-5: 유지보수성
- **NFR-5.1**: 코드는 `ruff`로 린트 및 포맷(줄 길이 100, 대상 py313).
- **NFR-5.2**: 관심사 분리: routes, models, engine.

---

## 3. 기술적 제약

| 제약 | 값 |
|---|---|
| 언어 | Python 3.13 |
| 패키지 관리자 | uv (pip, poetry, conda 없음) |
| 프레임워크 | FastAPI + Pydantic v2 |
| ASGI 서버 | uvicorn |
| 빌드 백엔드 | hatchling |
| 테스트 프레임워크 | pytest + pytest-asyncio + httpx + pytest-cov |
| 린터/포매터 | ruff |
| 금지 | Flask, Django, requests, sympy, pandas, numpy, pip, poetry, pipenv, black, flake8, isort |

---

## 4. API 버전 관리
- URL 접두사: `/api/v1/...`
- 초기 릴리스: v0.1.0
- 시맨틱 버전 적용.

---

## 5. 범위 밖 (MVP)
- 영구 저장소 또는 사용자 계정
- 그래픽 또는 터미널 UI
- 기호 연산 / CAS 기능
- Python `decimal` 모듈을 넘어선 임의 정밀도
- 문자열 입력으로부터의 식 평가
