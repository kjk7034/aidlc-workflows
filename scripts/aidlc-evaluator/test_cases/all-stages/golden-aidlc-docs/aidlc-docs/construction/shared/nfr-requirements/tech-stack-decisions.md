# 기술 스택 결정 — 두 서비스 공통

## 코어 스택

| 기술 | 버전 | 용도 |
|---|---|---|
| Python | 3.13+ | 런타임 |
| FastAPI | 0.115+ | REST API 프레임워크 |
| Pydantic | 2.x | 요청/응답 검증 |
| uvicorn | 0.34+ | ASGI 서버 |
| PyJWT | 2.x | JWT 생성 및 검증 |
| passlib[bcrypt] | 1.7+ | 비밀번호 해싱 |
| httpx | 0.28+ | HTTP 클라이언트(Lending → Catalog, 테스트 클라이언트) |

## 테스트 스택

| 기술 | 버전 | 용도 |
|---|---|---|
| pytest | 8.x | 테스트 러너 |
| pytest-asyncio | 0.24+ | 비동기 테스트 지원 |
| pytest-cov | 6.x | 커버리지 보고 |
| httpx | 0.28+ | 통합 테스트용 AsyncClient |

## 개발 도구

| 기술 | 버전 | 용도 |
|---|---|---|
| uv | latest | 패키지 관리 |
| ruff | 0.9+ | 린트 + 포맷 |

## 데이터베이스 전략

| 계층 | 기술 | 용도 |
|---|---|---|
| 저장소 인터페이스 | 추상 기본 클래스 | 데이터 접근 계약 정의 |
| MVP 구현 | 메모리 내(Python dict) | 의존성 제로 개발/테스트 |
| 향후 프로덕션 | DynamoDB 또는 PostgreSQL | 클라우드 배포(2단계) |

## 금지 기술(tech-env.md 기준)

| 금지 | 대신 사용 |
|---|---|
| Flask, Django | FastAPI |
| requests | httpx |
| pandas, numpy | 표준 Python |
| pip, poetry, pipenv | uv |
| black, flake8, isort | ruff |
| EC2(직접) | Lambda 또는 Fargate(향후) |
| Elastic Beanstalk | CDK(향후) |

## 프로젝트 구조

각 서비스는 독립적인 Python 패키지입니다:
- `catalog-service/` — pyproject.toml, src/catalog_service/, tests/
- `lending-service/` — pyproject.toml, src/lending_service/, tests/

둘 다 `uv`를 패키지 관리자로 사용하며 의존성은 `pyproject.toml`로 명시합니다.
