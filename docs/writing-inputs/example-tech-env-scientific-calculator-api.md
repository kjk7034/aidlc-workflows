# 기술 환경 문서: CalcEngine 과학 계산기 API

## 프로젝트 기술 요약

- **프로젝트 이름**: CalcEngine
- **프로젝트 유형**: Greenfield
- **주 런타임 환경**: Cloud
- **클라우드 제공자**: AWS
- **목표 배포 모델**: Serverless (API Gateway + Lambda)
- **패키지 관리자**: uv
- **팀 규모**: 4명(백엔드 2, 문서 포털용 프런트엔드 1, QA 1)
- **팀 경험**: Python 백엔드 강함, AWS 중간, 수학 라이브러리 개발 경험 없음. FastAPI와 Flask를 업무로 사용. pytest 익숙. CDK 경험 제한적(예시 필요).

---

## 프로그래밍 언어

### 필수 언어

| Language | Version | Purpose | Rationale |
|----------|---------|---------|-----------|
| Python | 3.12+ | API 서비스, 수학 엔진, Lambda 핸들러, CDK 인프라 | 팀 주력 언어. 풍부한 수학 생태계(mpmath, numpy, scipy). uv로 빠르고 안정적인 의존성 관리. |
| HTML/CSS/JS | ES2022+ | 문서 포털(정적 사이트) | API 문서용 최소 프런트엔드. 프레임워크 불필요; Jinja2 템플릿으로 정적 생성. |

### 허용 언어

| Language | Conditions for Use |
|----------|-------------------|
| Rust | Python 성능이 부족할 때 성능이 중요한 수학 함수(예: 식 파서)에 승인. 도입 전 프로파일링 근거 필요. PyO3/maturin으로 Python에 노출. |
| TypeScript | 팀이 Python CDK 대신 TypeScript CDK를 선호할 때 CDK 인프라에 승인. 결정은 construction 시작 전에 내려야 하며 프로젝트 중간에 바꾸지 않음. |

### 금지 언어

| Language | Reason | Use Instead |
|----------|--------|-------------|
| Java | 팀 전문성 없음. 운영 복잡도 증가(Lambda에서 JVM 콜드 스타트). | Python |
| Go | 팀 전문성 없음. Python이 현재 요구를 모두 충족. | Python |
| C/C++ | 네이티브 확장 유지 부담. | 네이티브 성능이 필요하면 PyO3 경유 Rust |

---

## 패키지 및 환경 관리

### 표준 도구로서의 uv

uv는 이 프로젝트의 **유일한 패키지 및 환경 관리 도구**입니다. pip, pip-tools, poetry, pipenv, conda를 사용하지 마세요.

### uv 사용 표준

```
# Project initialization (already done; do not re-run)
uv init calcengine
cd calcengine

# Adding dependencies
uv add fastapi                      # Add a runtime dependency
uv add uvicorn[standard]            # Add with extras
uv add --dev pytest pytest-cov      # Add a development dependency
uv add --dev mypy ruff              # Add dev tooling

# Removing dependencies
uv remove requests                  # Remove a dependency

# Running commands in the project environment
uv run python -m calcengine.main    # Run application
uv run pytest                       # Run tests
uv run mypy src/                    # Run type checker
uv run ruff check src/              # Run linter

# Syncing environment from lockfile
uv sync                             # Install all dependencies from uv.lock
uv sync --dev                       # Include dev dependencies

# Lockfile management
# uv.lock is auto-generated. NEVER edit it manually.
# uv.lock MUST be committed to version control.
```

### 의존성 파일 표준

| File | Purpose | Committed to Git |
|------|---------|-----------------|
| `pyproject.toml` | 프로젝트 메타데이터, 의존성 선언, 도구 설정 | Yes |
| `uv.lock` | 정확히 해석된 버전의 결정적 lockfile | Yes |
| `.python-version` | 프로젝트용 Python 버전 고정(예: `3.12`) | Yes |

### pyproject.toml 관례

모든 프로젝트 설정은 `pyproject.toml`에 둡니다. pyproject.toml 설정을 지원하는 도구에 대해 별도 설정 파일을 만들지 마세요.

```toml
[project]
name = "calcengine"
version = "0.1.0"
description = "Scientific calculator REST API"
requires-python = ">=3.12"
dependencies = [
    # Runtime dependencies listed here by uv add
]

[dependency-groups]
dev = [
    # Dev dependencies listed here by uv add --dev
]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short --strict-markers"
markers = [
    "unit: Unit tests (fast, no external dependencies)",
    "integration: Integration tests (may require services)",
    "accuracy: Mathematical accuracy validation tests",
]

[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "W", "I", "N", "UP", "B", "A", "SIM", "TCH"]

[tool.coverage.run]
source = ["src/calcengine"]
branch = true

[tool.coverage.report]
fail_under = 90
show_missing = true
```

---

## 프레임워크 및 라이브러리

### 필수 프레임워크

| Framework/Library | Version | Domain | Rationale |
|-------------------|---------|--------|-----------|
| FastAPI | 0.115+ | REST API 프레임워크 | 비동기 지원, 자동 OpenAPI 스펙 생성, Pydantic 검증, 강한 Python 타입 통합. |
| Pydantic | 2.x | 요청/응답 검증, 설정 관리 | 타입 안전 데이터 모델, JSON 직렬화, FastAPI에 필수. |
| uvicorn | 0.30+ | ASGI 서버 | FastAPI용 표준 프로덕션 서버. 로컬 및 Mangum 경유 Lambda에서 사용. |
| Mangum | 1.x | Lambda 어댑터 | FastAPI ASGI 앱을 AWS Lambda 핸들러로 감쌈. 설정 없는 어댑터. |
| pytest | 8.x | 테스트 프레임워크 | 팀 표준. 풍부한 플러그인 생태계. |
| mypy | 1.x | 정적 타입 검사 | 런타임 전 타입 오류 포착. 엄격 모드 시행. |
| ruff | 0.8+ | 린트 및 포맷 | flake8, isort, black을 하나의 빠른 도구로 대체. |
| structlog | 24.x+ | 구조화 JSON 로깅 | 모든 Lambda 핸들러와 API 엔드포인트는 구조화 JSON 로그를 내보내야 함. 공유 모듈에서 한 번 설정. |
| aws-cdk-lib | 2.x | Infrastructure as Code | AWS 배포. 모든 인프라에 Python CDK 구성 요소. |

### 권장 라이브러리

역량이 필요할 때만 사용합니다. 선제적으로 추가하지 마세요.

| Library | Purpose | Use When |
|---------|---------|----------|
| mpmath | 임의 정밀 산술 | Phase 2: 임의 정밀 모드 구현 시. MVP에는 불필요(IEEE 754 double로 충분). |
| numpy | 배열 연산, 선형대수 | Phase 2: 행렬/벡터 연산 구현 시. 기본 산술에는 사용하지 않음. |
| scipy | 통계 분포, 수치 적분 | Phase 2 이상: 고급 통계·미적분 구현 시. |
| httpx | 비동기 HTTP 클라이언트 | 아웃바운드 HTTP 호출(예: Phase 3 환율 조회). 비동기 호환을 위해 requests보다 선호. |
| boto3 | AWS SDK | 배포 시 CDK로 처리되지 않는 직접 AWS 서비스 상호작용(예: DynamoDB 쿼리, 런타임 Secrets Manager 읽기). |
| pytest-cov | 테스트 커버리지 보고 | 항상. 프로젝트 시작부터 dev 의존성에 포함. |
| pytest-asyncio | 비동기 테스트 지원 | 비동기 FastAPI 엔드포인트나 비동기 함수 테스트 시. |
| hypothesis | Property-based testing | 수학 함수 테스트. 엣지 케이스를 찾기 위해 무작위 입력 생성. 모든 수학 모듈에 강력 권장. |
| freezegun | 시간 모킹 | 시간 의존 로직(rate limiting, 토큰 만료, 감사 타임스탬프) 테스트 시. |

### 금지 라이브러리

| Library | Reason | Alternative |
|---------|--------|-------------|
| Flask | 프로젝트는 FastAPI 사용. 웹 프레임워크 혼용 금지. | FastAPI |
| Django | API 서비스에는 과함. ORM 불필요. | FastAPI + 직접 DynamoDB 접근 |
| requests | 동기 전용. FastAPI의 비동기 이벤트 루프를 블로킹. | httpx |
| sympy | MVP 범위에 비해 무거움. 큰 의존성 트리. | 식 파서를 직접 구현. Phase 3 기호 연산 재평가. |
| pandas | 불필요. CalcEngine은 개별 계산을 처리하며 dataframe이 아님. | 필요 시 표준 Python 또는 numpy로 배열 연산. |
| SQLAlchemy | MVP에 관계형 DB 없음. 데이터 저장소는 DynamoDB. | boto3 DynamoDB resource/client |
| celery | MVP에 불필요한 복잡도. 모든 계산은 동기이고 빠름(<50ms). | Phase 3 배치 처리 재평가. 더 일찍 비동기가 필요하면 SQS + Lambda. |
| poetry / pipenv / pip-tools | 프로젝트는 uv 전용. 대체 패키지 관리자 도입 금지. | uv |
| black / isort / flake8 | 셋을 합친 ruff로 대체. | ruff |

### 라이브러리 승인 절차

필수 또는 권장 목록에 없는 라이브러리를 추가하려면:

1. "Dependency Request: [library-name]" 제목의 GitHub 이슈 생성
2. 포함: 목적, 검토한 대안, 라이선스(MIT, Apache 2.0, BSD여야 함), 유지보수 상태(마지막 릴리스 날짜, 열린 이슈 수), 크기 영향
3. 테크 리드가 검토해 승인 또는 거부
4. 승인되면 `uv add`로 추가하고 이 문서 갱신

---

## 클라우드 환경

### 클라우드 제공자

- **Primary Provider**: AWS
- **Account Structure**: MVP는 단일 AWS 계정. Phase 2에 dev/staging/prod 분리 계정.
- **Regions**: `us-east-1`(주). MVP에는 재해 복구 리전 없음. Phase 2에 다중 리전 계획.

### 서비스 허용 목록

| Service | Approved Use Cases | Constraints |
|---------|-------------------|-------------|
| AWS Lambda | API 요청 핸들러, 수학 계산 | Python 3.12 런타임. MVP 최대 256MB 메모리(프로파일링상 필요 시 증가). 30초 타임아웃. |
| Amazon API Gateway (HTTP API) | 공개 REST API 엔드포인트 | HTTP API 유형(REST API 유형 아님). TLS가 있는 커스텀 도메인. rate limiting용 사용 계획. |
| Amazon DynamoDB | API 키 저장, 사용량 측정, rate limit 카운터 | 온디맨드 용량 모드. 단일 테이블 설계. rate limit 윈도우용 TTL. |
| Amazon S3 | OpenAPI 스펙 호스팅, 정적 문서 사이트, Lambda 배포 패키지 | 버킷 암호화 활성화. 문서 사이트 버킷(CloudFront 배포) 외 공개 접근 차단. |
| Amazon CloudFront | 문서 포털 및 API 스펙용 CDN | HTTPS만. 정적 자산 적극 캐시. |
| Amazon CloudWatch | 로깅, 메트릭, 알람, 대시보드 | 모든 Lambda의 구조화 JSON 로그. 계산 횟수, 지연 백분위, 오류율 커스텀 메트릭. |
| AWS Secrets Manager | Stripe API 키, 서명 키 | 지원 시 자동 로테이션. Lambda가 콜드 스타트에 읽고 메모리에 캐시. |
| AWS Certificate Manager | 커스텀 도메인용 TLS 인증서 | API Gateway 및 CloudFront와 함께 사용. |
| Amazon Cognito | 문서 포털 및 API 키 관리용 개발자 계정 인증 | 개발자 가입/로그인용 사용자 풀. API 호출 인증에는 사용하지 않음(API 키 사용). |
| Amazon SQS | 실패한 비동기 작업용 데드 레터 큐 | 표준 큐. 실패한 결제 이벤트 및 오류 캡처에 사용. MVP에서는 계산 요청에 사용하지 않음. |
| AWS CDK | Infrastructure as Code 배포 | Python CDK. 모든 인프라를 CDK로 정의. 수동 콘솔 변경 없음. |
| AWS CloudTrail | API 감사 로깅 | 모든 관리 이벤트에 대해 활성화. 프로덕션에서 S3 및 Lambda 데이터 이벤트. |
| AWS IAM | 서비스 권한 | Lambda 함수당 최소 권한 정책. 와일드카드 리소스 권한 없음. |

### 서비스 비허용 목록

| Service | Reason | Alternative |
|---------|--------|-------------|
| Amazon EC2 | 운영 부담. Serverless 모델 선호. | 컴퓨트는 Lambda. |
| Amazon ECS / Fargate | MVP 요청/응답 부하에 과한 설계. | Lambda. 콜드 스타트가 문제가 되면 재평가. |
| Amazon RDS / Aurora | 관계형 DB 불필요. API 키·사용 데이터는 DynamoDB에 적합. | DynamoDB. |
| Amazon ElastiCache / Redis | MVP에 캐시 레이어 불필요. 계산은 상태 없고 빠름. | 필요 시 Lambda 실행 컨텍스트 내 인메모리 캐싱. |
| AWS Elastic Beanstalk | IaC 모델에 맞지 않음. | CDK + Lambda. |
| Amazon Kinesis | 스트리밍 불필요. 모든 계산은 동기 요청/응답. | 비동기 처리가 필요하면 SQS. |
| AWS Step Functions | MVP에 다단계 오케스트레이션 없음. | 직접 Lambda 호출. Phase 3 배치 처리 재평가. |
| Amazon SNS | MVP에 pub/sub 불필요. | 데드 레터 큐는 SQS. |

### 서비스 승인 절차

허용 목록에 없는 서비스를 사용하려면:

1. "AWS Service Request: [service-name]" 제목의 GitHub 이슈 생성
2. 포함: 사용 사례, 비용 추정(월), 보안 영향, 운영 부담, 허용 서비스로는 요구를 충족할 수 없는 이유
3. 테크 리드 검토. PII 접근 또는 네트워크 노출이 있는 서비스는 추가 보안 검토 필요.
4. 승인되면 CDK 구성 요소를 추가하고 이 문서 갱신

---

## 선호 기술 및 패턴

### 아키텍처 패턴

**Serverless 함수로 배포되는 모듈형 모놀리스.**

CalcEngine은 내부 모듈(arithmetic, trigonometry, statistics 등)을 가진 단일 Python 패키지이며, 단일 FastAPI 애플리케이션으로 노출되고 API Gateway 뒤 AWS Lambda에 배포됩니다. 마이크로서비스 아키텍처가 아닙니다.

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Architecture style | Modular monolith | 소규모 팀(4명), 단일 도메인, MVP에서 모듈별 독립 스케일링 요구 없음. |
| Deployment model | Mangum으로 모든 API 라우트를 처리하는 단일 Lambda 함수 | 단순함. 배포 산출물 하나. 모든 엔드포인트에 콜드 스타트 분산. |
| Module boundaries | `src/calcengine/` 내 Python 패키지 | 별도 서비스 운영 비용 없이 깔끔한 내부 경계. 특정 엔드포인트에 다른 메모리/타임아웃이 필요하면 나중에 별도 Lambda로 분리 가능. |

### API 설계 표준

- **Style**: HTTPS상 REST. JSON 요청·응답 본문.
- **Base URL**: `https://api.calcengine.io/v1/`
- **Versioning**: URL 경로 접두사(`/v1/`, `/v2/`). 메이저 버전만. 비호환 깨지지 않는 변경은 버전을 올리지 않음.
- **Documentation**: FastAPI가 자동 생성하는 OpenAPI 3.1 스펙. `https://docs.calcengine.io`에 호스팅.
- **Naming Convention**: JSON 필드명은 snake_case(Python 관례). URL 경로는 kebab-case.
- **Content Type**: 모든 요청·응답에 `application/json`. XML 미지원.

**표준 요청 형식:**

```json
{
  "expression": "sin(pi/4) * 2 + sqrt(16)",
  "options": {
    "angle_mode": "radians",
    "precision": 15
  }
}
```

**표준 성공 응답 형식:**

```json
{
  "result": 5.414213562373095,
  "expression": "sin(pi/4) * 2 + sqrt(16)",
  "computation_time_ms": 2.3,
  "engine_version": "0.1.0"
}
```

**표준 오류 응답 형식:**

```json
{
  "error": {
    "code": "DOMAIN_ERROR",
    "message": "Cannot compute logarithm of a negative number",
    "detail": "log(-5) is undefined for real numbers",
    "parameter": "expression",
    "documentation_url": "https://docs.calcengine.io/errors/DOMAIN_ERROR"
  }
}
```

**오류 코드(MVP):**

| Code | HTTP Status | Meaning |
|------|------------|---------|
| `PARSE_ERROR` | 400 | 식을 파싱할 수 없음. 잘못된 구문. |
| `DOMAIN_ERROR` | 422 | 수학적으로 정의되지 않음(log(-1), sqrt(-1), 0으로 나눔). |
| `OVERFLOW_ERROR` | 422 | 결과가 표현 가능 범위를 초과. |
| `INVALID_PARAMETER` | 400 | 요청 매개변수의 타입 또는 값이 잘못됨. |
| `EXPRESSION_TOO_LONG` | 400 | 식이 허용 최대 길이를 초과. |
| `RATE_LIMIT_EXCEEDED` | 429 | API 키가 rate limit을 초과. |
| `UNAUTHORIZED` | 401 | API 키 없음 또는 잘못됨. |
| `INTERNAL_ERROR` | 500 | 예기치 않은 서버 오류. |

### 데이터 패턴

- **Primary Data Store**: DynamoDB(단일 테이블 설계)
- **Entities in DynamoDB**: API 키, 사용량 카운터(키·월별), rate limit 윈도우(키·분별)
- **Access Pattern**: 모든 읽기·쓰기는 기본 키(API 키 ID)로. 스캔 없음. 복잡한 쿼리 없음.
- **Caching**: 외부 캐시 없음. Lambda는 웜 호출 간 DynamoDB 연결 재사용. API 키 검증 결과는 Lambda 메모리에 60초 캐시.
- **No relational database**: 관계형 쿼리가 필요해지면(보고, 분석) RDS 추가 전 DynamoDB → S3 내보내기 + Athena 평가.

### 로깅 패턴

모든 로그 출력은 structlog를 통한 구조화 JSON이어야 합니다. 사람이 읽기 쉬운 콘솔 출력은 로컬 개발 전용.

```python
import structlog

logger = structlog.get_logger()

# Standard log call
logger.info(
    "calculation_completed",
    expression=expression,
    result=result,
    computation_time_ms=elapsed,
    api_key_id=api_key_id,
)

# Error log call
logger.error(
    "calculation_failed",
    expression=expression,
    error_code="DOMAIN_ERROR",
    error_detail=str(e),
    api_key_id=api_key_id,
)
```

**모든 API 요청에 필요한 로그 필드:**

| Field | Description |
|-------|-------------|
| `request_id` | 요청당 고유 ID(API Gateway에서 오거나 생성) |
| `api_key_id` | 해시된 API 키 식별자(원시 키는 절대 로그하지 않음) |
| `endpoint` | 호출된 API 경로 |
| `http_method` | GET, POST 등 |
| `http_status` | 응답 상태 코드 |
| `duration_ms` | 총 요청 처리 시간 |
| `timestamp` | ISO 8601 타임스탬프 |

---

## 보안 요구사항

### 인증 및 권한 부여

- **API Call Authentication**: API 키는 `Authorization: Bearer {key}` 헤더로 전달. API 키는 32자 무작위 문자열이며 DynamoDB에 bcrypt 해시로 저장.
- **Developer Portal Authentication**: Amazon Cognito 사용자 풀. 이메일 + 비밀번호 가입 및 이메일 검증.
- **Authorization Model**: 평면적. 모든 API 키가 모든 엔드포인트에 접근. 티어별 rate limit(무료, starter, professional)은 엔드포인트 수준이 아니라 사용량 측정으로 시행.
- **API Key Management**: 개발자는 개발자 포털에서 키를 생성·로테이션·폐기. 계정당 최대 3개 활성 키.

### 데이터 보호

- **Encryption at Rest**: DynamoDB는 AWS 관리 KMS 키로 암호화. S3 버킷은 SSE-S3로 암호화.
- **Encryption in Transit**: API Gateway 커스텀 도메인과 CloudFront 배포에 TLS 1.2+ 시행. HTTP(평문) 엔드포인트 없음.
- **PII Handling**: 개발자 계정은 이메일과 해시된 비밀번호만 저장. 그 외 PII 수집 없음. 수학 식은 PII 아님. 식은 디버깅용으로 로그되나 영구 저장하지 않음(CloudWatch 로그 보관 30일).
- **Data Classification**: API 키 = 기밀. 개발자 이메일 = 내부. 수학 식과 결과 = 공개.

### 입력 검증

- **Expression length limit**: 최대 4,096자. 더 긴 식은 `EXPRESSION_TOO_LONG`으로 거부.
- **Expression character allowlist**: 영숫자, 산술 연산자(`+ - * / ^ %`), 괄호, 소수점, 쉼표, 공백, 인식된 함수 이름. 인식되지 않은 문자 거부.
- **No code execution**: 식 파서는 `eval()`, `exec()`, `compile()` 또는 동적 코드 실행을 호출하면 안 됨. 식은 AST로 파싱되어 수학 엔진이 평가.
- **Recursion depth limit**: 식 파서는 중첩 깊이를 100단계로 제한. `(((((...)))))` 같은 깊은 중첩에서 스택 오버플로 방지.
- **Numeric range validation**: IEEE 754 double 정밀도 범위를 초과하는 결과는 `Infinity`나 `NaN` 대신 `OVERFLOW_ERROR` 반환.

### 비밀 관리

- **Stripe API Keys**: AWS Secrets Manager에 저장. Lambda가 콜드 스타트에 읽고 메모리에 캐시.
- **Cognito Client Secret**: AWS Secrets Manager에 저장.
- **Prohibited Practices**:
  - Git에 커밋된 `pyproject.toml`, 소스 코드, `.env`에 비밀 없음
  - Lambda 환경 변수에 비밀 없음(런타임에 Secrets Manager 사용)
  - 코드에 AWS 액세스 키 없음(Lambda는 IAM 실행 역할 사용)
  - `.env`는 로컬 개발 전용, `.gitignore`에 포함

### 의존성 보안

- **Scanning**: Python 의존성에 GitHub Dependabot 활성화. 알려진 취약점 알림.
- **License Policy**: 허용: MIT, Apache 2.0, BSD(2·3절), PSF, ISC. 금지: GPL, LGPL, AGPL, SSPL, 독점. 새 의존성 추가 전 `uv tree`로 확인.
- **Update Policy**: Critical/High CVE는 7일 이내 패치. Medium은 30일 이내. Low는 분기별 평가.

### OWASP Top 10 준수(2021)

#### A01:2021 - Broken Access Control

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Authorization enforcement | 라우트 핸들러 실행 전 모든 요청에서 FastAPI 미들웨어(`api/middleware/auth.py`)로 API 키 검증. 유효한 키 없이는 어떤 엔드포인트도 접근 불가. |
| Default deny | API Gateway가 게이트웨이 수준에서 `Authorization` 헤더 없는 요청 거부(401). Lambda 핸들러가 잘못되었거나 폐기된 키 요청 거부(401). |
| Resource ownership | 각 API 키는 Cognito 계정에 연결. 개발자는 자신의 키만 나열·로테이션·폐기 가능. DynamoDB 쿼리는 인증된 사용자의 파티션 키로 범위 제한. |
| Rate limiting | 미들웨어(`api/middleware/rate_limit.py`)에서 키당 rate limit 시행. 무료: 월 1만/초 10회. Starter: 월 100만/초 50회. Professional: 월 1천만/초 200회. 초과 시 429. |
| CORS policy | API Gateway CORS는 문서 포털 출처(`https://docs.calcengine.io`)만 허용. 와일드카드 출처 없음. `GET` 및 `POST`만. |
| Directory traversal / path manipulation | 해당 없음. CalcEngine은 파일을 서빙하지 않고 파일 경로를 입력으로 받지 않음. |

#### A02:2021 - Cryptographic Failures

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Data in transit | API Gateway 커스텀 도메인과 CloudFront에 TLS 1.2+ 시행. HTTP 엔드포인트 없음. API Gateway는 `SecurityPolicy: TLS_1_2` 설정. |
| Data at rest | DynamoDB는 AWS 관리 KMS 키로 암호화. S3는 SSE-S3. CloudWatch 로그는 서비스 관리 키로 암호화. |
| Password/credential storage | 개발자 포털 비밀번호는 bcrypt 해시(Cognito 관리). API 키는 DynamoDB에 bcrypt 해시로 저장. 원시 API 키는 생성 시 한 번만 응답 본문으로 반환되며 이후 저장·로그 없음. |
| Sensitive data in responses | API 응답에 API 키, 계정 자격증명, 내부 식별자 없음. 오류 메시지에 테이블 이름, ARN, 스택 트레이스 유출 없음. |
| Sensitive data in logs | API key ID(키 자체가 아닌 해시 식별자)만 로그. 원시 API 키는 절대 로그하지 않음. 계산 로그에 개발자 이메일 없음. |

#### A03:2021 - Injection

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Expression injection | 식 파서는 엄격한 문법으로 AST를 구축. `eval()`, `exec()`, `compile()` 또는 Python 코드 실행 메커니즘을 **사용하지 않음**. 인식된 토큰(숫자, 연산자, 괄호, 화이트리스트 함수명)만 허용. 인식되지 않은 토큰은 `PARSE_ERROR`(400). |
| Character allowlist | 식 입력은 숫자, 소수점, 산술 연산자(`+ - * / ^ %`), 괄호, 쉼표, 공백, 고정 함수명 집합(`sin`, `cos`, `tan`, `log`, `sqrt` 등)으로 제한. 파싱 전 나머지 문자 모두 거부. |
| NoSQL injection | DynamoDB 쿼리는 매개변수화된 키 조건으로 boto3 SDK 사용. 사용자 입력을 쿼리 식에 문자열 연결하지 않음. 파티션·정렬 키는 프로그램으로만 설정, 요청 본문에서 보간하지 않음. |
| HTTP header injection | FastAPI와 Pydantic이 모든 요청 입력을 검증·타입 검사. 응답 헤더는 프레임워크가 프로그램으로 설정, 사용자 입력에서 오지 않음. |
| Log injection | structlog가 로그 값의 특수 문자 이스케이프. 사용자 제공 식은 구조화 JSON 필드 내 문자열 값으로 로그, 로그 형식 문자열에 보간하지 않음. |

#### A04:2021 - Insecure Design

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Threat modeling | AIDLC NFR Requirements 단계에서 위협 모델 작성. 새 엔드포인트·통합 지점 추가 시 검토. 주요 위협: 식 인젝션, 리소스 고갈, API 키 악용. |
| Defense in depth | 세 계층 검증: (1) API Gateway 요청 검증 (2) FastAPI의 Pydantic 모델 검증 (3) 엔진 함수의 도메인 검증. 각 계층이 독립적으로 거부. |
| Business logic limits | 식 길이 최대 4,096자. 파서 재귀 깊이 최대 100단계. 통계 엔드포인트 배열 최대 10,000개 원소. 정당한 사용에 영향 없이 리소스 고갈 방지. |
| Abuse case testing | 테스트 스위트에 부정/악용 테스트 포함: 과대 식, 깊게 중첩된 괄호, 느린 평가를 유도하는 식, rate limit 초과 연속 요청, 잘못됨/만료/폐기된 API 키. |

#### A05:2021 - Security Misconfiguration

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Infrastructure as Code | 모든 인프라를 AWS CDK(Python)로 정의. 수동 콘솔 변경 없음. 배포 전 PR에서 CDK diff 검토. |
| Default credentials | 어떤 환경에도 기본 API 키, 관리자 계정, 하드코딩 비밀번호 없음. Cognito 사용자 풀은 이메일 검증 필요. |
| Error messages | 프로덕션 오류 응답은 CalcEngine 오류 코드, 사용자 친화 메시지, 문서 URL만 반환. Python 트레이스백, Lambda ARN, DynamoDB 테이블 이름, 내부 파일 경로 노출 없음. 프로덕션에서 FastAPI `debug=False`. |
| Unnecessary features | 프로덕션 Lambda에 `/docs` 또는 `/redoc` 대화형 엔드포인트 노출 없음. OpenAPI 스펙은 정적 문서 사이트에서만 제공. `engine_version` 이상의 버전 세부를 드러내는 헬스 체크 엔드포인트 없음. |
| Security headers | API Gateway 응답에 `Strict-Transport-Security: max-age=31536000; includeSubDomains`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, API 응답에 `Cache-Control: no-store` 포함. CloudFront가 문서 사이트에 보안 헤더 추가. |
| Lambda configuration | Lambda는 필요 최소 메모리(256MB) 사용. 타임아웃 30초. 예약 동시 실행으로 무한 스케일링 방지. 비밀이 있는 환경 변수 없음(런타임 Secrets Manager). |

#### A06:2021 - Vulnerable and Outdated Components

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Dependency scanning | GitHub Dependabot 활성화. `pyproject.toml`과 `uv.lock`에서 알려진 취약점 스캔. 알림이 자동으로 GitHub 이슈 생성. |
| Patch SLA | Critical/High CVE: 7일 이내 패치. Medium: 30일. Low: 분기별 평가. |
| License compliance | 허용: MIT, Apache 2.0, BSD, PSF, ISC. 금지: GPL, LGPL, AGPL, SSPL, 독점. 의존성 추가 전 `uv tree`로 확인. |
| Lockfile integrity | `uv.lock`을 Git에 커밋하고 CI에서 시행. CI의 `uv sync --locked`는 lockfile이 오래되면 실패. CI에서 임의 `uv add` 없음. |
| Minimal dependencies | 금지 라이브러리 목록으로 비대한 의존성 트리 방지(MVP에 pandas, Django, SQLAlchemy, sympy 없음). 새 의존성마다 정당화가 있는 GitHub 이슈 필요. |

#### A07:2021 - Identification and Authentication Failures

| Control | CalcEngine Implementation |
|---------|--------------------------|
| API key hashing | API 키는 `secrets.token_urlsafe`로 만든 32자 암호학적 무작위 문자열. bcrypt 해시로 저장. 조회는 키 접두사(처음 8자, 평문 저장)로 레코드를 찾은 뒤 bcrypt로 전체 키 검증. |
| Brute force protection | API Gateway 스로틀링: 모든 엔드포인트에서 IP당 초 100요청. 실패한 인증(잘못된 키)은 `api_key_prefix`와 출처 IP로 로그. 5분 안에 단일 IP에서 50회 실패 시 WAF 규칙으로 임시 IP 차단. |
| Developer portal auth | Cognito 시행: 최소 12자 비밀번호, 이메일 검증 필수, 로그인 5회 실패 시 계정 잠금. |
| Key rotation | 개발자가 이전 키를 폐기하기 전에 새 키 생성 가능(무중단 로테이션을 위한 겹침 기간). 계정당 최대 3개 활성 키로 키 비축 방지. |
| Credential exposure | API 키는 생성 시(HTTP 응답 본문) 정확히 한 번만 반환. 어디에도 평문 저장 없음. 이메일에 포함 없음. 생성 후 개발자 포털에 표시 없음. |
| Multi-factor authentication | MVP에 필수 아님. Cognito MFA 지원은 있으며 팀/엔터프라이즈 계정이 도입되는 Phase 2에 옵션으로 활성화 예정. |

#### A08:2021 - Software and Data Integrity Failures

| Control | CalcEngine Implementation |
|---------|--------------------------|
| CI/CD pipeline security | GitHub Actions. `main` 브랜치 보호: PR 필요, 최소 1인 리뷰, 모든 CI 통과. `main`에 직접 푸시 금지. 배포 워크플로는 `main` 머지 시에만 트리거. |
| Dependency integrity | `uv.lock`에 모든 의존성 해시 포함. `uv sync --locked`가 설치 시 해시 검증. PR의 lockfile 변경은 명시적으로 검토. |
| Deployment artifact integrity | Lambda 배포 패키지는 깨끗한 `uv sync --locked` 설치로 CI에서 빌드. 로컬 빌드를 프로덕션에 배포하지 않음. CDK 배포는 개발자 머신이 아닌 CI 파이프라인에서만 실행. |
| Deserialization safety | Pydantic v2 모델이 들어오는 JSON을 파싱·검증. `pickle`, `yaml.load()`(안전하지 않은 로더), `marshal` 사용 없음. Pydantic JSON 파싱을 통한 `json.loads()`만. Pydantic `model_config`에 `extra = "forbid"`로 예상치 못한 필드 거부. |

#### A09:2021 - Security Logging and Monitoring Failures

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Security events logged | 아래 이벤트를 모두 CloudWatch에 구조화 JSON으로 로그: 인증 실패(잘못됨/만료/폐기된 키), rate limit 초과(429), 입력 검증 실패(400), 권한 이상, 모든 5xx 오류. |
| Log protection | CloudWatch 로그 30일 보관. 로그 그룹 리소스 정책으로 Lambda 실행 역할의 삭제 방지. CloudTrail이 관리 이벤트를 객체 잠금이 있는 별도 S3 버킷에 로그. |
| Alerting | CloudWatch 알람: 5분간 5xx 오류율 > 1%, 인증 실패율 > 분당 100회, 단일 API 키가 시도에서 자신의 rate limit의 10배 초과, Lambda 동시 실행 > 예약 동시 실행의 80%. 알람은 SNS로 온콜 이메일/SMS에 알림. |
| Monitoring dashboard | CloudWatch 대시보드: 요청 수, 오류율(4xx 및 5xx), p50/p95/p99 지연, 인증 실패 수, rate limit 적중 수, Lambda 콜드 스타트 비율, DynamoDB 소비 용량. 주간 검토. |

#### A10:2021 - Server-Side Request Forgery (SSRF)

| Control | CalcEngine Implementation |
|---------|--------------------------|
| Applicability | **MVP에서 위험 낮음.** CalcEngine은 사용자 입력 기반 아웃바운드 HTTP 요청을 하지 않음. 식 파서는 수학 식을 평가하며 URL 가져오기, 호스트명 해석, 네트워크 호출을 하지 않음. |
| Outbound requests | Lambda의 아웃바운드 네트워크 호출은 (1) AWS SDK를 통한 DynamoDB 쿼리(엔드포인트는 AWS 리전으로 결정, 사용자 입력 아님) (2) 콜드 스타트의 Secrets Manager 읽기(비밀 이름은 설정에 하드코딩, 사용자 입력 아님)뿐. |
| Phase 3 consideration | Phase 3에 통화 변환이 추가되면 금융 데이터 제공자에서 환율을 가져옴. 그때 제공자 URL은 환경 변수(사용자 입력 아님), 요청은 허용 목록 호스트명 사용, 응답은 사용 전 예상 스키마에 대해 검증. Phase 3 출시 전 이 절을 갱신해야 함. |
| Network segmentation | Lambda는 AWS 관리 VPC에서 실행(MVP에 고객 VPC 없음). 공개 엔드포인트를 통해서만 AWS 서비스에 도달. 이 구성에서 Lambda는 내부 서비스, DB, 메타데이터 엔드포인트에 도달 불가. |

---

## 테스트 요구사항

### 테스트 전략 개요

| Test Type | Required | Coverage Target | Tooling |
|-----------|----------|----------------|---------|
| Unit Tests | Yes | 라인 90%, 분기 80% | pytest + pytest-cov |
| Mathematical Accuracy Tests | Yes | 구현된 함수 100% | pytest + hypothesis |
| Integration Tests | Yes | 모든 API 엔드포인트, DynamoDB 상호작용 | pytest + moto(AWS 모킹) |
| Load Tests | Yes(출시 전) | 동시 요청 1,000건, p50 < 50ms | Locust |
| Security Tests | Yes | 입력 검증, 인젝션 방지 | pytest(커스텀) + 수동 OWASP 검토 |
| End-to-End Tests | Conditional | 배포된 스테이징에 대한 핵심 사용자 여정 | pytest + httpx로 라이브 API |

### 단위 테스트 표준

- **Coverage Minimum**: 라인 90%, 분기 80%. `pyproject.toml`의 `pytest-cov`와 `fail_under = 90`으로 시행.
- **Mocking Policy**: moto로 AWS 서비스(DynamoDB, Secrets Manager) 모킹. freezegun으로 시간 모킹. 내부 수학 함수는 모킹하지 않음. 수학 함수는 실제 계산으로 테스트.
- **Naming Convention**: 테스트 파일은 소스를 미러링. `src/calcengine/trig.py`는 `tests/unit/test_trig.py`에서 테스트. 테스트 함수명 `test_<function>_<scenario>`(예: `test_sin_zero_returns_zero`, `test_sin_negative_pi_returns_zero`).
- **Test Location**: 별도 `tests/` 디렉터리 트리. 소스와 공동 위치하지 않음.

```
tests/
  unit/
    test_arithmetic.py
    test_trig.py
    test_statistics.py
    test_expression_parser.py
    test_error_handling.py
  integration/
    test_api_evaluate.py
    test_api_trig.py
    test_api_keys.py
    test_rate_limiting.py
  accuracy/
    test_trig_accuracy.py
    test_arithmetic_accuracy.py
    test_statistics_accuracy.py
  conftest.py
```

### 수학 정확도 테스트

대부분의 프로젝트에 없는 CalcEngine 전용 테스트 범주입니다.

- **Reference implementation**: 모든 수학 함수는 Python `math` 모듈, 고정밀 `mpmath` 라이브러리, 또는 공개된 수학 표와 대조해 테스트해야 함.
- **Property-based testing with hypothesis**: hypothesis로 무작위 유효 입력을 생성해 성질이 성립하는지 검증(예: `sin(x)^2 + cos(x)^2 == 1`, `log(a*b) == log(a) + log(b)`).
- **Edge cases**: 모든 함수에 대해 명시적 테스트: 0, 음의 0, 매우 작은 수(엡실론 근처), 매우 큰 수, 도메인 경계(예: asin(1), asin(1.0000001)), 특수 값(pi, e, 삼각법의 pi/2 배수).
- **Tolerance**: 기본 함수는 참조 값과 1 ULP(최하위 비트 단위) 이내로 일치해야 함. 더 넓은 허용을 받는 함수는 근거와 함께 문서화.

**정확도 테스트 예시 패턴:**

```python
import math
import pytest
from hypothesis import given, strategies as st
from calcengine.trig import sin, cos

class TestSinAccuracy:
    """Validate sin() accuracy against math.sin and known exact values."""

    @pytest.mark.accuracy
    @pytest.mark.parametrize("input_val, expected", [
        (0, 0.0),
        (math.pi / 6, 0.5),
        (math.pi / 4, math.sqrt(2) / 2),
        (math.pi / 2, 1.0),
        (math.pi, 0.0),
        (3 * math.pi / 2, -1.0),
        (2 * math.pi, 0.0),
        (-math.pi / 2, -1.0),
    ])
    def test_sin_known_values(self, input_val: float, expected: float) -> None:
        result = sin(input_val)
        assert result == pytest.approx(expected, abs=1e-15)

    @pytest.mark.accuracy
    @given(st.floats(min_value=-1e6, max_value=1e6, allow_nan=False, allow_infinity=False))
    def test_sin_matches_stdlib(self, x: float) -> None:
        assert sin(x) == pytest.approx(math.sin(x), rel=1e-15)

    @pytest.mark.accuracy
    @given(st.floats(min_value=-1e6, max_value=1e6, allow_nan=False, allow_infinity=False))
    def test_pythagorean_identity(self, x: float) -> None:
        assert sin(x) ** 2 + cos(x) ** 2 == pytest.approx(1.0, abs=1e-14)
```

### 통합 테스트 표준

- **Scope**: FastAPI 테스트 클라이언트로 전체 API 요청/응답 주기 테스트. moto로 DynamoDB 상호작용 테스트.
- **Environment**: 로컬. 배포된 서비스 불필요. `moto`가 모든 AWS 서비스를 모킹.
- **Data Management**: 각 테스트가 moto fixture로 자체 DynamoDB 테이블을 만들고 이후 정리. 공유 테스트 상태 없음.

### CI/CD 테스트 게이트

| Pipeline Stage | Required Tests | Tooling | Failure Action |
|---------------|---------------|---------|----------------|
| Pre-commit | ruff check, ruff format --check, mypy | pre-commit 훅의 ruff, mypy | 커밋 차단 |
| Pull Request | 단위, 정확도, 통합 테스트, 커버리지 검사 | pytest, GitHub Actions | 머지 차단 |
| Pre-deploy (staging) | 모든 PR 테스트 + 부하 테스트(동시 100, 60초) | pytest + Locust, GitHub Actions | 배포 차단 |
| Post-deploy (production) | 스모크 테스트(라이브 API에 대해 대표 계산 10건) | pytest + httpx | 온콜 알림. 실패 >50% 시 자동 롤백. |

### 로컬에서 테스트 실행

```bash
# Run all tests
uv run pytest

# Run only unit tests
uv run pytest tests/unit/ -m unit

# Run only accuracy tests
uv run pytest tests/accuracy/ -m accuracy

# Run with coverage report
uv run pytest --cov --cov-report=term-missing

# Run type checking
uv run mypy src/

# Run linter
uv run ruff check src/ tests/

# Run formatter check (no changes)
uv run ruff format --check src/ tests/

# Run formatter (apply changes)
uv run ruff format src/ tests/
```

---

## 프로젝트 구조

```
calcengine/
  .github/
    workflows/
      ci.yml                         # GitHub Actions: lint, type check, test on PR
      deploy.yml                     # GitHub Actions: CDK deploy on merge to main
  src/
    calcengine/
      __init__.py
      main.py                        # FastAPI app creation, Mangum handler
      config.py                      # Settings via Pydantic BaseSettings
      api/
        __init__.py
        router.py                    # Top-level API router
        endpoints/
          __init__.py
          evaluate.py                # POST /v1/evaluate (expression evaluation)
          arithmetic.py              # POST /v1/arithmetic/{operation}
          trigonometry.py            # POST /v1/trigonometry/{function}
          statistics.py              # POST /v1/statistics/{function}
          constants.py               # GET  /v1/constants/{name}
        middleware/
          __init__.py
          auth.py                    # API key validation middleware
          rate_limit.py              # Rate limiting middleware
          request_logging.py         # Structured request/response logging
        models/
          __init__.py
          requests.py                # Pydantic request models
          responses.py               # Pydantic response models
          errors.py                  # Error response models and error codes
      engine/
        __init__.py
        expression_parser.py         # Tokenizer, AST builder, evaluator
        arithmetic.py                # Basic math operations
        trigonometry.py              # Trig functions with domain validation
        statistics.py                # Descriptive statistics functions
        constants.py                 # Mathematical constants
        combinatorics.py             # Factorial, permutations, combinations
        logarithmic.py               # Log, ln, exp functions
        validation.py                # Input validation, domain checking
        errors.py                    # Math-domain exception types
      storage/
        __init__.py
        dynamodb.py                  # DynamoDB client, table operations
        api_keys.py                  # API key CRUD, validation, hashing
        usage.py                     # Usage metering, rate limit counters
      logging.py                     # structlog configuration
  infrastructure/
    app.py                           # CDK app entry point
    stacks/
      __init__.py
      api_stack.py                   # Lambda, API Gateway, custom domain
      data_stack.py                  # DynamoDB tables
      monitoring_stack.py            # CloudWatch dashboards, alarms
      auth_stack.py                  # Cognito user pool
      docs_stack.py                  # S3 + CloudFront for documentation site
  tests/
    unit/
      test_arithmetic.py
      test_trig.py
      test_statistics.py
      test_expression_parser.py
      test_combinatorics.py
      test_logarithmic.py
      test_validation.py
      test_api_keys.py
    integration/
      test_api_evaluate.py
      test_api_arithmetic.py
      test_api_trig.py
      test_api_statistics.py
      test_api_auth.py
      test_api_rate_limiting.py
    accuracy/
      test_trig_accuracy.py
      test_arithmetic_accuracy.py
      test_statistics_accuracy.py
      test_logarithmic_accuracy.py
      test_expression_parser_accuracy.py
    conftest.py                      # Shared fixtures (FastAPI test client, moto mocks)
  examples/
    api-endpoint/
      README.md
      example_endpoint.py
      test_example_endpoint.py
    math-function/
      README.md
      example_function.py
      test_example_function.py
    cdk-construct/
      README.md
      example_stack.py
  docs/
    static/                          # Documentation portal source (Jinja2 templates)
  pyproject.toml
  uv.lock
  .python-version                    # Contains: 3.12
  .gitignore
  .pre-commit-config.yaml
  README.md
```

### 디렉터리 규칙

| Directory | Contains | Rules |
|-----------|----------|-------|
| `src/calcengine/` | 모든 애플리케이션 소스 코드 | Python만. 설정 파일, 테스트, 문서 없음. |
| `src/calcengine/engine/` | 순수 수학 함수 | AWS import 없음. HTTP import 없음. 부작용 없음. 순수 함수만. 모킹 없이 테스트 가능해야 함. |
| `src/calcengine/api/` | FastAPI 라우트, 미들웨어, 모델 | HTTP 레이어만. 엔진 함수 호출. 수학 로직 포함하지 않음. |
| `src/calcengine/storage/` | DynamoDB 접근 레이어 | 모든 AWS 데이터 접근을 여기에 격리. 비즈니스 로직 없음. |
| `infrastructure/` | CDK 스택 | Python CDK만. 애플리케이션 코드 없음. |
| `tests/` | 모든 테스트 | `src/` 구조를 미러링. `unit/`, `integration/`, `accuracy/` 디렉터리 분리. |
| `examples/` | 패턴용 템플릿 코드 | 테스트와 README가 있는 동작 코드. 표준 변경 시 갱신. |

---

## 예시 및 템플릿 코드

### 예시 1: API 엔드포인트 패턴

`examples/api-endpoint/README.md`:

```markdown
# API Endpoint Pattern

## What This Demonstrates
CalcEngine에 새 계산 엔드포인트를 추가하는 표준 패턴.
다음을 보여줌: 라우트 정의, Pydantic 모델, 엔진 호출, 오류 처리, 로깅.

## When to Use
- 새 계산 엔드포인트 추가 시
- API에 새 HTTP 라우트 추가 시

## When Not to Use
- 내부 엔진 함수(math-function 예시 참고)
- 인프라 변경(cdk-construct 예시 참고)

## Customization Guide
| Element | Customize? | Notes |
|---------|-----------|-------|
| Route path and method | Yes | /v1/{category}/{function} 관례 따름 |
| Request/response models | Yes | 엔드포인트별 Pydantic 모델 정의 |
| Engine function call | Yes | 적절한 엔진 모듈 함수 호출 |
| Error handling structure | No | 항상 CalcEngineError 계층과 error_response() 사용 |
| Logging calls | No | 항상 request_id, api_key_id, duration_ms로 로그 |
| Response envelope | No | 항상 {"result": ..., "expression": ..., "computation_time_ms": ..., "engine_version": ...} 반환 |
```

`examples/api-endpoint/example_endpoint.py`:

```python
"""Example: Standard API endpoint pattern for CalcEngine."""

import time

import structlog
from fastapi import APIRouter, Depends
from pydantic import BaseModel, Field

from calcengine.api.middleware.auth import get_api_key_id
from calcengine.api.models.errors import error_response
from calcengine.api.models.responses import CalculationResponse
from calcengine.engine.errors import CalcEngineError
from calcengine.engine.trigonometry import sin

logger = structlog.get_logger()

router = APIRouter()


class SinRequest(BaseModel):
    """Request model for sine calculation."""

    value: float = Field(..., description="Input angle")
    angle_mode: str = Field(
        default="radians",
        pattern="^(radians|degrees)$",
        description="Angle unit: 'radians' or 'degrees'",
    )


@router.post("/v1/trigonometry/sin", response_model=CalculationResponse)
async def calculate_sin(
    request: SinRequest,
    api_key_id: str = Depends(get_api_key_id),
) -> CalculationResponse | dict:
    """Calculate the sine of the given value."""
    start = time.perf_counter()

    try:
        result = sin(request.value, angle_mode=request.angle_mode)
        elapsed = (time.perf_counter() - start) * 1000

        logger.info(
            "calculation_completed",
            endpoint="/v1/trigonometry/sin",
            input_value=request.value,
            angle_mode=request.angle_mode,
            result=result,
            computation_time_ms=round(elapsed, 3),
            api_key_id=api_key_id,
        )

        return CalculationResponse(
            result=result,
            expression=f"sin({request.value})",
            computation_time_ms=round(elapsed, 3),
        )

    except CalcEngineError as e:
        elapsed = (time.perf_counter() - start) * 1000
        logger.warning(
            "calculation_failed",
            endpoint="/v1/trigonometry/sin",
            input_value=request.value,
            error_code=e.code,
            error_detail=str(e),
            computation_time_ms=round(elapsed, 3),
            api_key_id=api_key_id,
        )
        return error_response(e)
```

`examples/api-endpoint/test_example_endpoint.py`:

```python
"""Example: Standard test pattern for a CalcEngine API endpoint."""

import math

import pytest
from fastapi.testclient import TestClient

from calcengine.main import app


@pytest.fixture
def client() -> TestClient:
    """Create a test client with a mocked API key."""
    return TestClient(app)


class TestSinEndpoint:
    """Tests for POST /v1/trigonometry/sin."""

    @pytest.mark.unit
    def test_sin_zero_radians(self, client: TestClient) -> None:
        response = client.post(
            "/v1/trigonometry/sin",
            json={"value": 0, "angle_mode": "radians"},
            headers={"Authorization": "Bearer test-api-key"},
        )
        assert response.status_code == 200
        data = response.json()
        assert data["result"] == pytest.approx(0.0)
        assert "computation_time_ms" in data

    @pytest.mark.unit
    def test_sin_pi_over_2_radians(self, client: TestClient) -> None:
        response = client.post(
            "/v1/trigonometry/sin",
            json={"value": math.pi / 2, "angle_mode": "radians"},
            headers={"Authorization": "Bearer test-api-key"},
        )
        assert response.status_code == 200
        assert response.json()["result"] == pytest.approx(1.0)

    @pytest.mark.unit
    def test_sin_90_degrees(self, client: TestClient) -> None:
        response = client.post(
            "/v1/trigonometry/sin",
            json={"value": 90, "angle_mode": "degrees"},
            headers={"Authorization": "Bearer test-api-key"},
        )
        assert response.status_code == 200
        assert response.json()["result"] == pytest.approx(1.0)

    @pytest.mark.unit
    def test_sin_invalid_angle_mode(self, client: TestClient) -> None:
        response = client.post(
            "/v1/trigonometry/sin",
            json={"value": 1.0, "angle_mode": "gradians"},
            headers={"Authorization": "Bearer test-api-key"},
        )
        assert response.status_code == 422  # Pydantic validation error

    @pytest.mark.unit
    def test_sin_missing_auth(self, client: TestClient) -> None:
        response = client.post(
            "/v1/trigonometry/sin",
            json={"value": 0},
        )
        assert response.status_code == 401
```

### 예시 2: 순수 수학 함수 패턴

`examples/math-function/README.md`:

```markdown
# Math Function Pattern

## What This Demonstrates
엔진 레이어에서 순수 수학 함수를 구현하는 표준 패턴.
다음을 보여줌: 함수 시그니처, 타입 힌트, 도메인 검증, 오류 발생, docstring 형식.

## When to Use
- src/calcengine/engine/에 새 수학 함수 추가 시

## When Not to Use
- API 엔드포인트(api-endpoint 예시 참고)
- AWS 또는 HTTP 접근이 필요한 함수(api/ 또는 storage/에 속함)

## Key Rules
- api/, storage/, 외부 서비스에서 import 금지
- 순수 함수만: 동일 입력은 항상 동일 출력
- 도메인 오류는 CalcEngineError 하위 클래스로 raise, None이나 NaN 반환 금지
- 모든 매개변수와 반환값에 타입 힌트
```

`examples/math-function/example_function.py`:

```python
"""Example: Standard pattern for a pure math function in CalcEngine engine layer."""

import math

from calcengine.engine.errors import DomainError


def log_base(value: float, base: float = 10.0) -> float:
    """Compute the logarithm of a value with the given base.

    Args:
        value: The number to compute the logarithm of. Must be positive.
        base: The logarithm base. Must be positive and not equal to 1.
              Defaults to 10 (common logarithm).

    Returns:
        The logarithm of value in the given base.

    Raises:
        DomainError: If value <= 0, base <= 0, or base == 1.
    """
    if value <= 0:
        raise DomainError(
            code="DOMAIN_ERROR",
            message=f"Cannot compute logarithm of {value}",
            detail="Logarithm is only defined for positive numbers",
            parameter="value",
        )

    if base <= 0:
        raise DomainError(
            code="DOMAIN_ERROR",
            message=f"Cannot use {base} as logarithm base",
            detail="Logarithm base must be positive",
            parameter="base",
        )

    if base == 1.0:
        raise DomainError(
            code="DOMAIN_ERROR",
            message="Cannot use 1 as logarithm base",
            detail="Logarithm base 1 is undefined (division by zero in change-of-base)",
            parameter="base",
        )

    return math.log(value) / math.log(base)
```

`examples/math-function/test_example_function.py`:

```python
"""Example: Standard test pattern for a pure math function."""

import math

import pytest
from hypothesis import given, strategies as st

from calcengine.engine.errors import DomainError
from calcengine.engine.logarithmic import log_base


class TestLogBase:
    """Tests for log_base function."""

    # --- Known values ---

    @pytest.mark.unit
    def test_log10_of_100(self) -> None:
        assert log_base(100, 10) == pytest.approx(2.0)

    @pytest.mark.unit
    def test_log2_of_8(self) -> None:
        assert log_base(8, 2) == pytest.approx(3.0)

    @pytest.mark.unit
    def test_ln_of_e(self) -> None:
        assert log_base(math.e, math.e) == pytest.approx(1.0)

    @pytest.mark.unit
    def test_log_of_1_any_base(self) -> None:
        assert log_base(1, 10) == pytest.approx(0.0)
        assert log_base(1, 2) == pytest.approx(0.0)
        assert log_base(1, math.e) == pytest.approx(0.0)

    # --- Default base ---

    @pytest.mark.unit
    def test_default_base_is_10(self) -> None:
        assert log_base(1000) == pytest.approx(3.0)

    # --- Domain errors ---

    @pytest.mark.unit
    def test_log_of_zero_raises_domain_error(self) -> None:
        with pytest.raises(DomainError, match="Cannot compute logarithm"):
            log_base(0)

    @pytest.mark.unit
    def test_log_of_negative_raises_domain_error(self) -> None:
        with pytest.raises(DomainError, match="Cannot compute logarithm"):
            log_base(-5)

    @pytest.mark.unit
    def test_log_base_zero_raises_domain_error(self) -> None:
        with pytest.raises(DomainError, match="Cannot use 0"):
            log_base(10, 0)

    @pytest.mark.unit
    def test_log_base_one_raises_domain_error(self) -> None:
        with pytest.raises(DomainError, match="Cannot use 1"):
            log_base(10, 1)

    @pytest.mark.unit
    def test_log_base_negative_raises_domain_error(self) -> None:
        with pytest.raises(DomainError, match="Cannot use -2"):
            log_base(10, -2)

    # --- Property-based: accuracy against stdlib ---

    @pytest.mark.accuracy
    @given(
        st.floats(min_value=1e-300, max_value=1e300, allow_nan=False, allow_infinity=False),
    )
    def test_log10_matches_stdlib(self, x: float) -> None:
        assert log_base(x, 10) == pytest.approx(math.log10(x), rel=1e-14)

    @pytest.mark.accuracy
    @given(
        st.floats(min_value=1e-300, max_value=1e300, allow_nan=False, allow_infinity=False),
    )
    def test_log2_matches_stdlib(self, x: float) -> None:
        assert log_base(x, 2) == pytest.approx(math.log2(x), rel=1e-14)

    # --- Property-based: mathematical identity ---

    @pytest.mark.accuracy
    @given(
        a=st.floats(min_value=1e-100, max_value=1e100, allow_nan=False, allow_infinity=False),
        b=st.floats(min_value=1e-100, max_value=1e100, allow_nan=False, allow_infinity=False),
    )
    def test_log_product_identity(self, a: float, b: float) -> None:
        """log(a * b) should equal log(a) + log(b)."""
        if a * b > 0:
            assert log_base(a * b, 10) == pytest.approx(
                log_base(a, 10) + log_base(b, 10), rel=1e-10
            )
```

### 예시 3: CDK Construct 패턴

`examples/cdk-construct/README.md`:

```markdown
# CDK Construct Pattern

## What This Demonstrates
CalcEngine 인프라를 위한 CDK 스택 정의 표준 패턴.
다음을 보여줌: Lambda 함수, API Gateway 통합, DynamoDB 테이블, IAM 권한.

## When to Use
- 프로젝트에 새 인프라 리소스 추가 시

## Key Rules
- 모든 인프라는 infrastructure/stacks/ 디렉터리
- 논리 그룹(api, data, monitoring, auth, docs)당 스택 하나
- CDK context의 환경 변수 사용, 하드코딩 금지
- 최소 권한 IAM: 각 Lambda는 필요한 권한만
```

`examples/cdk-construct/example_stack.py`:

```python
"""Example: Standard CDK stack pattern for CalcEngine."""

from aws_cdk import Duration, Stack
from aws_cdk import aws_apigatewayv2 as apigwv2
from aws_cdk import aws_dynamodb as dynamodb
from aws_cdk import aws_lambda as lambda_
from aws_cdk import aws_logs as logs
from aws_cdk.aws_apigatewayv2_integrations import HttpLambdaIntegration
from constructs import Construct


class ExampleApiStack(Stack):
    """Example stack showing Lambda + API Gateway + DynamoDB pattern."""

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # DynamoDB table - single table design
        table = dynamodb.Table(
            self,
            "ExampleTable",
            partition_key=dynamodb.Attribute(
                name="PK", type=dynamodb.AttributeType.STRING
            ),
            sort_key=dynamodb.Attribute(
                name="SK", type=dynamodb.AttributeType.STRING
            ),
            billing_mode=dynamodb.BillingMode.PAY_PER_REQUEST,
            encryption=dynamodb.TableEncryption.AWS_MANAGED,
            point_in_time_recovery=True,
        )

        # Lambda function
        handler = lambda_.Function(
            self,
            "ExampleHandler",
            runtime=lambda_.Runtime.PYTHON_3_12,
            handler="calcengine.main.handler",
            code=lambda_.Code.from_asset("src/"),
            memory_size=256,
            timeout=Duration.seconds(30),
            environment={
                "TABLE_NAME": table.table_name,
                "LOG_LEVEL": "INFO",
            },
            log_retention=logs.RetentionDays.ONE_MONTH,
        )

        # Grant Lambda read/write access to DynamoDB (least privilege)
        table.grant_read_write_data(handler)

        # HTTP API Gateway with Lambda integration
        api = apigwv2.HttpApi(
            self,
            "ExampleHttpApi",
            api_name="calcengine-api",
            default_integration=HttpLambdaIntegration(
                "LambdaIntegration",
                handler,
            ),
        )
```

---

## 이 문서가 AI-DLC에 어떻게 쓰이는지

| Section | AI-DLC Stage | How It Is Used |
|---------|--------------|----------------|
| Project Technical Summary | Workspace Detection | Greenfield 분류, 팀 맥락 |
| Programming Languages | Code Generation | Python 3.12 시행, 승인 없이 다른 언어 금지 |
| uv Standards | Code Generation | 모든 의존성 작업에 uv 사용, pyproject.toml이 단일 설정 소스 |
| Frameworks and Libraries | Code Generation, NFR Design | FastAPI + Pydantic + Mangum 스택, 금지 라이브러리 시행 |
| Cloud Services Allow/Disallow | Infrastructure Design | MVP는 Lambda + API Gateway + DynamoDB만 |
| Architecture Pattern | Application Design | 모듈형 모놀리스, engine/ 대 api/ 대 storage/ 모듈 경계 |
| API Design Standards | Functional Design, Code Generation | 엔드포인트 관례, 오류 코드, 응답 형식 |
| Security Requirements | NFR Requirements, NFR Design | 입력 검증 규칙, eval() 금지, API 키 인증 패턴 |
| Testing Requirements | Code Generation, Build and Test | pytest + hypothesis, 90% 커버리지, 정확도 테스트 필수 |
| Project Structure | Code Generation | 정확한 디렉터리 레이아웃과 파일 배치 규칙 |
| Example Code | Code Generation | 엔드포인트, 엔진 함수, CDK 스택의 기준 패턴 |
