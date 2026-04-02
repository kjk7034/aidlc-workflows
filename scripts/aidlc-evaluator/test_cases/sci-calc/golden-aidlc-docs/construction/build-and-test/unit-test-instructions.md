# 단위 테스트 실행

## 단위 테스트 실행

### 1. 모든 테스트 실행 (표준 환경)
```bash
cd workspace
uv run pytest tests/ -v --cov=sci_calc --cov-report=term-missing --cov-fail-under=90
```

### 2. 모든 테스트 실행 (Windows asyncio 손상 환경)
```bash
cd workspace
set PYTHONPATH=src
python -m pytest tests/ -v -p no:anyio -p no:asyncio
```

### 3. 테스트 결과 검토

#### 예상: 192개 테스트 통과, 실패 0

| 테스트 모듈         | 엔진 테스트 | API 테스트 | 합계 |
|---------------------|-------------|-----------|-------|
| test_arithmetic.py  | 20          | 12        | 32    |
| test_constants.py   | 12          | 4         | 16    |
| test_conversions.py | 21          | 7         | 28    |
| test_logarithmic.py | 17          | 9         | 26    |
| test_powers.py      | 16          | 9         | 25    |
| test_statistics.py  | 18          | 14        | 32    |
| test_trigonometry.py| 25          | 8         | 33    |
| **합계**           | **129**     | **63**    | **192** |

- **테스트 커버리지 목표**: ≥90%
- **테스트 보고서 위치**: stdout(`--cov-report=term-missing` 통해)

### 4. 실패한 테스트 수정
테스트가 실패하면:
1. 어떤 테스트가 왜 실패했는지 보여 주는 상세 출력 검토
2. 오류 트레이스백 확인
3. `src/sci_calc/` 또는 `tests/`의 코드 문제 수정
4. 192개 모두 통과할 때까지 테스트 재실행

## 테스트 아키텍처

### 엔진 단위 테스트 (129)
- `sci_calc.engine.math_engine`에서 직접 import
- 유효·엣지·오류 입력으로 모든 수학 함수 테스트
- 사용자 정의 예외 검증(`MathDomainError`, `MathDivisionByZeroError`, `MathOverflowError`)
- HTTP 또는 프레임워크 의존성 없음

### API 통합 테스트 (63)
- 테스트 클라이언트로 전체 HTTP 엔드포인트 경로 호출
- 상태 코드, 응답 구조, 오류 래퍼 검증
- 정상 경로, 정의역 오류, 검증 오류, 404 테스트
- Pydantic 모델 검증(NaN 거부, 타입 강제) 검증

### 테스트 클라이언트
- **표준**: `starlette.testclient.TestClient`(동작하는 asyncio 필요)
- **대체**: `conftest.py`의 사용자 정의 `SyncTestClient` — 비동기 핸들러를 동기로 구동,
  `asyncio`를 import하지 않음 — Windows에서 `_overlapped` DLL이 손상된 환경에서 사용
