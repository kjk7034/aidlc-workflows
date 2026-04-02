# 기술 환경: Scientific Calculator API

## 언어 및 패키지 관리자

- **Python 3.13**
- 모든 패키지 관리에 **uv** 사용 (pip, poetry, conda 없음)
- 프로젝트 및 도구 설정은 `pyproject.toml`

## 웹 프레임워크

- 요청/응답 검증에 **FastAPI**와 Pydantic v2
- ASGI 서버로 **uvicorn**

## 프로젝트 구조

```
sci-calc/
├── pyproject.toml
├── src/
│   └── sci_calc/
│       ├── __init__.py
│       ├── app.py
│       ├── routes/
│       │   ├── __init__.py
│       │   ├── arithmetic.py
│       │   ├── trigonometry.py
│       │   ├── logarithmic.py
│       │   ├── powers.py
│       │   ├── statistics.py
│       │   ├── constants.py
│       │   └── conversions.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── requests.py
│       │   └── responses.py
│       └── engine/
│           ├── __init__.py
│           └── math_engine.py
└── tests/
    ├── __init__.py
    ├── conftest.py
    ├── test_arithmetic.py
    ├── test_trigonometry.py
    ├── test_logarithmic.py
    ├── test_powers.py
    ├── test_statistics.py
    ├── test_constants.py
    └── test_conversions.py
```

## 테스트

- **pytest**와 pytest-asyncio, httpx(비동기 테스트 클라이언트)
- **pytest-cov**, 라인 커버리지 최소 90%
- 단위 테스트는 `math_engine.py`를 알려진 값 테이블로 직접 검증
- 통합 테스트는 FastAPI TestClient와 `httpx.AsyncClient` 사용
- 경계 테스트로 모든 정의역 제약이 올바른 오류 코드를 내는지 검증
- 실행 명령: `uv run pytest`

## 린트 및 포맷

- **ruff** (줄 길이 100, 대상 py313)

## 빌드 백엔드

- **hatchling**

## 사용하지 말 것

| 금지 | 이유 | 대신 사용 |
|-----------|--------|-------------|
| Flask, Django | 프로젝트는 FastAPI 사용 | FastAPI |
| requests | 비동기 이벤트 루프를 블로킹 | httpx |
| sympy | 이 범위에는 과함 | Python `math` 표준 라이브러리 |
| pandas, numpy | 단일 계산에는 불필요 | 표준 Python |
| pip, poetry, pipenv | 프로젝트는 uv만 사용 | uv |
| black, flake8, isort | ruff로 대체 | ruff |

## 비기능 요구사항

| 요구사항 | 목표 |
|---|---|
| 시작 시간 | 2초 미만 |
| 응답 지연(p95) | 단일 연산당 50ms 미만 |
| 테스트 커버리지 | 라인 커버리지 90% 이상 |
| 부동소수점 일치 | 결과가 Python `math` 표준 라이브러리와 1 ULP 이내 |
| 최대 요청 본문 크기 | 1 MB |
| Python 버전 | 3.13.x (`requires-python = ">=3.13"`로 강제) |

## 개발 워크플로

```bash
uv sync
uv run uvicorn sci_calc.app:app --reload --port 8000
uv run pytest
uv run ruff check . && uv run ruff format .
```
