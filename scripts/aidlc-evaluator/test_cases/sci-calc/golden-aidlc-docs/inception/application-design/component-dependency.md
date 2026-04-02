# 컴포넌트 의존성

## 의존성 행렬

| 컴포넌트 | 의존 대상 | 의존되는 쪽 |
|---|---|---|
| `app.py` | 모든 routes, responses(모델) | — (진입점) |
| `routes/arithmetic.py` | `models/requests.py`, `models/responses.py`, `engine/math_engine.py` | `app.py` |
| `routes/powers.py` | `models/requests.py`, `models/responses.py`, `engine/math_engine.py` | `app.py` |
| `routes/trigonometry.py` | `models/requests.py`, `models/responses.py`, `engine/math_engine.py` | `app.py` |
| `routes/logarithmic.py` | `models/requests.py`, `models/responses.py`, `engine/math_engine.py` | `app.py` |
| `routes/statistics.py` | `models/requests.py`, `models/responses.py`, `engine/math_engine.py` | `app.py` |
| `routes/constants.py` | `models/responses.py`, `engine/math_engine.py` | `app.py` |
| `routes/conversions.py` | `models/requests.py`, `models/responses.py`, `engine/math_engine.py` | `app.py` |
| `models/requests.py` | — (독립 Pydantic 모델) | 모든 routes |
| `models/responses.py` | — (독립 Pydantic 모델) | 모든 routes, `app.py` |
| `engine/math_engine.py` | Python `math` 표준 라이브러리, `statistics` 표준 라이브러리 | 모든 routes |

## 데이터 흐름

```
HTTP Request
    |
    v
app.py (FastAPI) --> route handler (validates via Pydantic model)
    |                    |
    |                    v
    |               math_engine.py (pure computation)
    |                    |
    |                    v
    |               result or exception
    |                    |
    v                    v
SuccessResponse or ErrorResponse (Pydantic model)
    |
    v
HTTP Response (JSON)
```

## 통신 패턴
- **동기 함수 호출** — 엔진에 async 호출 불필요(CPU 바운드 수학이 빠름)
- **메시지 큐, 이벤트, 외부 서비스 없음**
- **데이터베이스 연결 없음**
- 엔진에서 라우트로 **예외 기반 오류 신호**

## 주요 설계 결정
1. 엔진은 **순수 모듈**로 독립 함수만 — 클래스 인스턴스화 없음, 상태 없음
2. 라우트가 엔진 함수를 **직접 import** — 이 범위에서는 의존성 주입 불필요
3. 사용자 정의 예외(`DomainError`, `DivisionByZeroError`)는 엔진 모듈에 정의
4. 엔진은 **HTTP/프레임워크 의존성 제로** — 단독으로 테스트 가능
