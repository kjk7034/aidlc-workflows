# 빌드 지침

## 사전 요구사항
- **빌드 도구**: `hatchling` (PEP 517 빌드 백엔드)
- **런타임**: Python 3.13+
- **패키지 관리자**: `uv`(권장) 또는 `pip`
- **의존성**:
  - `fastapi>=0.115.0`
  - `uvicorn[standard]>=0.34.0`
- **개발 의존성**:
  - `httpx>=0.28.0`
  - `pytest>=8.3.0`
  - `pytest-asyncio>=0.25.0`
  - `pytest-cov>=6.0.0`
  - `ruff>=0.9.0`

## 빌드 단계

### 1. 의존성 설치
```bash
# uv 사용 (권장)
uv sync --all-extras

# 또는 pip 사용
pip install -e ".[dev]"
```

### 2. 환경 설정
```bash
# 설치 없이 소스에서 실행할 때 PYTHONPATH 설정
export PYTHONPATH=src    # Linux/macOS
set PYTHONPATH=src       # Windows
```

### 3. 패키지 빌드
```bash
# wheel 및 sdist 빌드
uv build
# 또는: python -m build
```

### 4. 빌드 성공 검증
- **예상 출력**: `dist/sci_calc-0.1.0-py3-none-any.whl`
- **패키지 구조**: engine, models, routes 하위 패키지가 있는 `src/sci_calc/`
- **진입점**: `sci_calc.app:app` (ASGI 애플리케이션)

## 서버 실행
```bash
# 개발 서버
uvicorn sci_calc.app:app --reload --host 0.0.0.0 --port 8000
```

## 문제 해결

### 의존성 오류로 빌드 실패
- **원인**: 네트워크 사용 불가 또는 PyPI 접근 불가
- **해결**: 로컬 패키지 캐시와 `--find-links` 사용 또는 의존성 사전 설치

### import 오류(오래된 site-packages)
- **원인**: 전역에 설치된 sci-calc 구버전
- **해결**: `PYTHONPATH=src` 설정 또는 `pip install -e .`로 덮어쓰기

### Windows에서 asyncio 손상(WinError 10106)
- **원인**: 손상된 Windows Winsock 공급자(`_overlapped` DLL)
- **해결**: 관리자 권한으로 `netsh winsock reset` 실행, 또는 WSL2 사용
- **우회**: `-p no:anyio -p no:asyncio` pytest 플래그와 동기 테스트 클라이언트 사용
