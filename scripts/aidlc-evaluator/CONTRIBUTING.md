# AI-DLC 평가 프레임워크에 기여하기

AI-DLC 워크플로 평가 및 보고 프레임워크에 기여해 주셔서 감사합니다.

## 시작하기

### 사전 요구사항

- Python 3.13+
- [uv](https://github.com/astral-sh/uv) 패키지 매니저
- Git

### 설정

```bash
# Clone the repository
git clone <repository-url>
cd aidlc-evaluation-framework

# Install dependencies
uv sync

# Run tests to verify setup
uv run pytest
```

## 개발 워크플로

### 1. 브랜치 만들기

```bash
git checkout -b feature/your-feature-name
```

### 2. 변경 작업

적절한 패키지에서 작업합니다.
- `aidlc-runner/` - 실행 프레임워크(AIDLC 두 에이전트 워크플로 러너)
- `packages/qualitative/` - 의미 평가(의도·설계 유사도 채점)
- `packages/quantitative/` - 코드 평가(린트, 보안, 구조)
- `packages/nonfunctional/` - NFR 평가(토큰, 시간, 일관성)
- `packages/reporting/` - 보고서 생성
- `packages/shared/` - 공통 유틸리티

또는 다른 작업 흐름에 기여할 수 있습니다.
- `test_cases/` - 골든 테스트 케이스(기준 입력)
- `docs/writing-inputs/` - 비전·기술 환경 문서 가이드
- `.github/workflows/` - GitHub CI/CD 통합 및 관리

### 3. 테스트 실행

```bash
# Run all tests
uv run pytest

# Run specific package tests
uv run pytest tests/test_qualitative.py

# Run with coverage
uv run pytest --cov
```

### 4. 린트

```bash
# Check code style
uv run ruff check .

# Auto-fix issues
uv run ruff check --fix .

# Format code
uv run ruff format .
```

### 5. 커밋

명확하고 설명적인 커밋 메시지를 작성합니다.

```bash
git add .
git commit -m "Add token tracking to nonfunctional package"
```

### 6. Pull Request 제출

- 브랜치를 저장소에 푸시합니다
- 변경 사항을 명확히 설명하는 PR을 엽니다
- 관련 이슈가 있으면 링크합니다
- 자동 테스트가 통과할 때까지 기다립니다
- 리뷰 피드백을 반영합니다

## 작업 흐름

프로젝트는 여섯 가지 big rock으로 구성됩니다. 변경은 보통 그중 하나 이상에 해당합니다.

| 작업 흐름 | 설명 | 패키지 / 영역 |
|---|---|---|
| **골든 테스트 케이스** | 선별된 기준 테스트 입력 | `test_cases/` |
| **실행 프레임워크** | AIDLC 두 에이전트 워크플로 러너(담당: Jeff) | `aidlc-runner/` |
| **의미 평가** | 의도·설계 유사도 채점 | `packages/qualitative/` |
| **코드 평가** | 린트, 보안, 구조 | `packages/quantitative/` |
| **NFR 평가** | 토큰, 시간, 일관성 | `packages/nonfunctional/` |
| **GitHub CI/CD** | 파이프라인 통합 및 관리 | `.github/workflows/` |

## 코드 기준

### Python 스타일

- PEP 8 준수(Ruff로 강제)
- 타입 힌트 사용
- 최대 줄 길이: 100자
- 공개 함수·클래스에는 독스트링 작성

### 테스트

- 새 기능에는 테스트 작성
- 코드 커버리지 유지 또는 개선
- 설명적인 테스트 이름: `test_<what>_<condition>_<expected>`

### 문서

- 새 기능을 추가하면 README.md 갱신
- 새 모듈·함수에는 독스트링 추가
- `docs/`의 관련 문서 갱신

## 패키지 의존성

의존성을 추가할 때:

1. `packages/<package>/` 또는 `aidlc-runner/`의 해당 `pyproject.toml`에 추가
2. `uv sync`로 lock 파일 갱신
3. PR에 해당 의존성이 필요한 이유를 문서화

## 이슈 보고

버그나 기능 요청 시:

- GitHub Issues 사용
- 재현 단계를 명확히 기술
- 관련 로그나 오류 메시지 포함
- 영향받는 패키지 명시

## 질문이 있으면?

- 자주 묻는 질문은 [FAQ.md](./FAQ.md) 참고
- 의사결정 가이드는 [OPERATING_PRINCIPLES.md](./OPERATING_PRINCIPLES.md) 참고
- PR 댓글이나 Discussion에 질문

## 행동 강령

- 서로 존중하고 건설적으로 대화합니다
- 사람이 아니라 코드에 집중합니다
- 다양한 관점을 환영합니다
- 서로 배우고 성장하는 데 도움을 줍니다

AI-DLC 평가 프레임워크 개선에 참여해 주셔서 감사합니다.
