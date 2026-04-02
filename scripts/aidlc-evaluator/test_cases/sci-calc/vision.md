# Scientific Calculator API

## 요약

수학 연산, 삼각함수, 로그, 거듭제곱, 통계, 단위 변환을 수행하는 무상태 HTTP API입니다. 수학 라이브러리를 설치하지 않고도 모든 HTTP 클라이언트가 사용할 수 있습니다. 이 계산기는 처리량보다 정확성, 정밀도, 명확한 오류 보고를 우선합니다. 골든 테스트 케이스 애플리케이션 역할을 합니다: 전체를 충분히 이해할 만큼 작으면서도, 코드 생성 도구를 여러 측면에서 검증하기에 충분히 풍부합니다.

## 범위 내 기능 (MVP)

- 산술: add, subtract, multiply, divide, modulo, abs, negate
- 거듭제곱과 루트: power, sqrt, cbrt, nth_root, square
- 삼각함수: sin, cos, tan, asin, acos, atan, atan2, sinh, cosh, tanh, asinh, acosh, atanh (도·라디안 모드)
- 로그: ln, log10, log2, log (임의 밑), exp
- 통계: mean, median, mode, stdev, variance, pstdev, pvariance, min, max, sum, count
- 상수: pi, e, tau, inf, nan, golden_ratio, sqrt2, ln2, ln10
- 단위 변환: 각도, 온도, 길이, 무게
- 헬스 체크 엔드포인트
- 모든 실패 케이스에 대한 구조화된 오류 응답
- 단위 및 통합 테스트

## 명시적으로 범위 밖인 기능 (MVP)

- 영구 저장소 또는 사용자 계정
- 그래픽 또는 터미널 UI
- 기호 연산 / 컴퓨터 대수(CAS) 기능
- Python 표준 `decimal` 모듈을 넘어서는 임의 정밀도 또는 큰 수 라이브러리
- 인증, 속도 제한, 또는 프로덕션 하드닝
- 문자열 입력으로부터의 식 평가

## API 명세

모든 엔드포인트는 `application/json`을 받고 반환합니다.

### 응답 래퍼

**성공:**

```json
{ "status": "ok", "operation": "<name>", "inputs": { ... }, "result": <number | object> }
```

**오류:**

```json
{ "status": "error", "operation": "<name>", "inputs": { ... }, "error": { "code": "<CODE>", "message": "..." } }
```

| 오류 코드 | HTTP 상태 | 의미 |
|---|---|---|
| `INVALID_INPUT` | 422 | 요청 본문 검증 실패 |
| `DIVISION_BY_ZERO` | 400 | 0으로 나눗셈 또는 나머지 연산 |
| `DOMAIN_ERROR` | 400 | 수학적 정의역 밖의 입력 (예: sqrt(-1), log(0)) |
| `OVERFLOW` | 400 | 결과가 표현 가능 범위를 초과 |
| `NOT_FOUND` | 404 | 알 수 없는 엔드포인트 |

### 엔드포인트

**`GET /health`** — `{"status": "ok", "version": "0.1.0"}`을 반환합니다.

**`POST /api/v1/arithmetic/{operation}`** — `add`, `subtract`, `multiply`, `divide`, `modulo`는 `{"a": N, "b": N}`을 받습니다. `abs`, `negate`는 `{"a": N}`을 받습니다.

**`POST /api/v1/powers/{operation}`** — `power`는 `{"base": N, "exponent": N}`을 받습니다. `sqrt`, `cbrt`, `square`는 `{"a": N}`을 받습니다. `nth_root`는 `{"a": N, "n": int}`를 받습니다. sqrt에서 `a < 0`이면 정의역 오류; nth_root에서 `a < 0`이고 `n`이 짝수이면 정의역 오류입니다.

**`POST /api/v1/trigonometry/{operation}`** — 대부분 `{"a": N, "angle_unit": "radians"|"degrees"}`를 받습니다(기본값 라디안). `atan2`는 `{"y": N, "x": N, "angle_unit": ...}`를 받습니다. 정의역 제약: asin/acos는 -1 <= a <= 1, acosh는 a >= 1, atanh는 -1 < a < 1.

**`POST /api/v1/logarithmic/{operation}`** — `ln`, `log10`, `log2`는 `{"a": N}`을 받습니다(a <= 0이면 정의역 오류). `log`는 `{"a": N, "base": N}`을 받습니다(a <= 0, base <= 0, 또는 base = 1이면 정의역 오류). `exp`는 `{"a": N}`을 받습니다.

**`POST /api/v1/statistics/{operation}`** — 모두 `{"values": [N, ...]}`를 받습니다. 최소 1개 요소 필요. `stdev`/`variance`는 최소 2개 요소 필요. `pstdev`/`pvariance`는 최소 1개. `mode`는 동점일 때 가장 작은 최빈값을 반환합니다.

**`GET /api/v1/constants/{name}`** — 이름이 지정된 상수를 반환합니다. `GET /api/v1/constants`는 모두 맵으로 반환합니다.

**`POST /api/v1/conversions/{category}`** — `{"value": N, "from_unit": "...", "to_unit": "..."}`를 받습니다. 카테고리: angle(도/라디안/그라드), temperature(섭씨/화씨/켈빈), length(미터/피트/인치/센티미터/밀리미터/킬로미터/마일/야드), weight(킬로그램/파운드/온스/그램/밀리그램/톤/스톤).

## 오류 처리 원칙

1. 절대 맨 500을 반환하지 않습니다. 수학 정의역 및 오버플로 오류를 잡아 구조화된 오류 래퍼로 변환합니다.
2. FastAPI/Pydantic이 스키마 검증 오류를 처리하도록 하고, 기본 422 핸들러를 오류 래퍼에 맞게 재정의합니다.
3. 예상치 못한 예외는 ERROR 수준으로 로깅하고 일반 `INTERNAL_ERROR` 응답을 반환합니다.

## 성공 지표

- 모든 테스트가 라인 커버리지 90% 이상으로 통과
- 결과가 표준 연산에 대해 Python `math` 표준 라이브러리와 1 ULP 이내로 일치
- 단일 연산에 대한 응답 지연 p95 < 50ms

## 버전 관리

API는 URL 접두사(`/api/v1/...`)로 버전이 매겨집니다. 초기 릴리스는 v0.1.0입니다. 시맨틱 버전을 적용합니다.
