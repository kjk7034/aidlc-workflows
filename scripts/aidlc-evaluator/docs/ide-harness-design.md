# IDE 테스트 하네스 — 아키텍처 설계

## 문제

AIDLC 평가 프레임워크는 Bedrock에서 두 에이전트 Strands swarm으로 실행됩니다. IDE 기반 AI 코딩 어시스턴트를 평가하려면 각 IDE의 AI 채팅 인터페이스로 동일한 AIDLC 프로세스를 구동하고, 기존 평가 파이프라인(2–6단계)과 호환되는 형식으로 출력을 수집해야 합니다.

## 입출력 계약

### 입력(각 IDE 어댑터에 제공)
- `vision.md` — 애플리케이션 비전 문서
- `tech-env.md` — 기술 환경 사양
- AIDLC 규칙 — 전체 AIDLC 워크플로 규칙(`aidlc-workflows` 저장소에서)
- 초기 프롬프트 템플릿 — IDE AI가 AIDLC 프로세스를 따르도록 하는 지시

### 출력(각 IDE 어댑터에서 수집)
- `aidlc-docs/` — 생성된 AIDLC 문서(Strands 실행과 동일한 구조)
  - `inception/requirements/`, `inception/plans/`, `inception/application-design/`
  - `construction/plans/`, `construction/build-and-test/`
  - `aidlc-state.md`, `audit.md`
- `workspace/` — 생성된 애플리케이션 소스와 테스트
- `run-meta.yaml` — 실행 메타데이터(어댑터 생성, collector 스키마와 일치)
- `test-results.yaml` — 실행 후 테스트 결과(IDE 완료 후 어댑터가 테스트 실행)

### 출력 정규화

IDE 출력은 Strands run 폴더 레이아웃과 정확히 일치하지 않습니다. 각 어댑터는 기대 구조에 맞게 출력을 정규화해야 합니다.

```
<run-folder>/
  run-meta.yaml          # adapter generates this
  run-metrics.yaml       # adapter generates (tokens if available, timing always)
  test-results.yaml      # adapter runs tests post-generation
  aidlc-docs/            # extracted/copied from IDE workspace
  workspace/             # extracted/copied from IDE workspace
```

이렇게 하면 `run_evaluation.py --evaluate-only <run-folder>/aidlc-docs`로 출력을 채점할 수 있습니다.

## 어댑터 인터페이스

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from pathlib import Path

@dataclass
class AdapterConfig:
    """Configuration for an IDE adapter run."""
    vision_path: Path           # path to vision.md
    tech_env_path: Path | None  # path to tech-env.md (optional)
    rules_path: Path            # path to cloned aidlc-workflows rules
    output_dir: Path            # where to write normalized output
    prompt_template: str        # initial prompt to send to IDE AI
    timeout_seconds: int = 7200 # max time to wait for IDE completion

@dataclass
class AdapterResult:
    """Result from an IDE adapter run."""
    success: bool
    output_dir: Path
    aidlc_docs_dir: Path | None
    workspace_dir: Path | None
    error: str | None = None
    elapsed_seconds: float = 0.0
    token_estimate: int | None = None  # if IDE reports token usage

class IDEAdapter(ABC):
    """Abstract base for IDE-specific automation adapters."""

    @property
    @abstractmethod
    def name(self) -> str:
        """Human-readable IDE name."""
        ...

    @abstractmethod
    def check_prerequisites(self) -> tuple[bool, str]:
        """Verify IDE is installed, configured, and accessible.

        Returns (ok, message).
        """
        ...

    @abstractmethod
    def run(self, config: AdapterConfig) -> AdapterResult:
        """Execute the AIDLC process through the IDE and capture outputs.

        Steps:
        1. Set up a clean workspace directory
        2. Copy/symlink vision.md, tech-env.md, and rules into the workspace
        3. Launch IDE (or connect to running instance)
        4. Send the initial prompt to the IDE's AI chat
        5. Monitor for completion (all AIDLC phases done)
        6. Extract aidlc-docs/ and workspace/ from IDE output
        7. Generate run-meta.yaml with timing and adapter info
        """
        ...
```

## 실행 오케스트레이션

```
run_ide_evaluation.py
  ├── parse args (--ide <name>, --vision, --golden, etc.)
  ├── load adapter by name
  ├── adapter.check_prerequisites()
  ├── adapter.run(config) → AdapterResult
  ├── post_run_tests(result.workspace_dir)  → test-results.yaml
  └── run_evaluation.py --evaluate-only <result.aidlc_docs_dir> --golden <golden>
```

오케스트레이터 스크립트는:
1. 대상 IDE용 어댑터를 인스턴스화
2. 어댑터를 실행해 출력 생성
3. 생성 후 테스트 실행(의존성 설치 + pytest/npm test)
4. evaluate-only 모드로 기존 평가 파이프라인 호출

## 어댑터 구현 전략

### 범주 A: CLI로 스크립트 가능한 IDE
프롬프트 전송·응답 수신을 위한 CLI 또는 API 지원이 있는 IDE.

- **Cursor** — CLI(`cursor` 명령). `--chat` 등 지원 가능.
- **Kiro** — AWS IDE, Bedrock 연동 가능성. CLI 확인.

접근: 서브프로세스 호출, stdout/stderr 파싱, 출력 파일을 위해 워크스페이스 모니터링.

### 범주 B: VS Code 확장 IDE
독립 CLI 없이 VS Code 확장으로 동작하는 IDE.

- **Cline** — VS Code 확장. VS Code 자동화 필요.
- **GitHub CoPilot** — VS Code 확장. 채팅 패널 자동화 필요.

접근: `@vscode/test-electron` 또는 Playwright 기반 VS Code 자동화.

### 범주 C: VS Code 포크 IDE
내장 AI가 있는 VS Code 포크 단독 IDE.

- **Windsurf** — Codeium 포크. Electron 앱, VS Code 내부.
- **Antigravity** — AI 코딩 어시스턴트.

접근: Playwright 또는 네이티브 확장 API로 Electron 자동화.

### 공통 실행 후 단계(모든 어댑터)
1. 워크스페이스에서 `aidlc-docs/` 디렉터리 구조 스캔
2. `workspace/` 또는 프로젝트 루트 아래 생성 소스 코드 식별
3. 기대 스키마에 맞게 파일 레이아웃 정규화
4. 프로젝트 유형 감지(Python/Node/Rust/Go)
5. 의존성 설치 및 테스트 실행
6. `run-meta.yaml` 및 `run-metrics.yaml` 생성

## 패키지 구조

```
packages/ide-harness/
  pyproject.toml
  src/ide_harness/
    __init__.py
    adapter.py          # Abstract adapter interface + AdapterConfig/Result
    orchestrator.py     # Run orchestration (invoke adapter + evaluation)
    normalizer.py       # Output normalization utilities
    post_run.py         # Reuse/adapt execution package's post-run test logic
    prompt_template.py  # Standard AIDLC prompt template for IDE AI
    adapters/
      __init__.py
      kiro.py
      cursor.py
      cline.py
      copilot.py
      windsurf.py
      antigravity.py
  tests/
    test_normalizer.py
    test_orchestrator.py
```

## 프롬프트 템플릿

각 IDE AI에 보내는 프롬프트는 AIDLC 프로세스를 따르도록 지시해야 합니다.

```
You are tasked with building an application following the AIDLC (AI Development
Life Cycle) process. The AIDLC rules are provided in the `aidlc-rules/` directory.

Please read the vision document at `vision.md` and follow the complete AIDLC process:

1. INCEPTION PHASE:
   - Read the AIDLC rules for the inception phase
   - Create requirements, plans, and application design documents
   - Output these to `aidlc-docs/inception/`

2. CONSTRUCTION PHASE:
   - Read the AIDLC rules for the construction phase
   - Create build plans and test instructions
   - Generate the application source code and tests
   - Output documents to `aidlc-docs/construction/`
   - Output code to the project root (which becomes `workspace/`)

3. Generate `aidlc-docs/aidlc-state.md` tracking your progress through each phase.

Follow every AIDLC rule precisely. Do not skip phases or documents.
```

## 열린 질문

1. **완료 감지**: IDE AI가 모든 AIDLC 단계를 끝냈는지 어떻게 감지할까?
   - 파일 기반: `aidlc-state.md`로 construction 완료 표시까지 감시
   - 시간 기반: N분 후 타임아웃
   - 프롬프트 기반: IDE AI에 완료 신호 요청

2. **다중 턴 상호작용**: AIDLC에는 인간 시뮬레이터 핸드오프가 포함됩니다. IDE에서는:
   - 단일 포괄 프롬프트를 보내 IDE가 모두 처리하게 할까?
   - 다중 턴 상호작용을 스크립트(단계 전환마다 승인)할까?
   - 반자동(사람이 모니터링, 스크립트가 수집)할까?

3. **토큰 추적**: 대부분의 IDE는 토큰 사용량을 노출하지 않습니다. 선택지:
   - 출력 크기로 추정
   - Bedrock CloudWatch 메트릭 수집(IDE가 Bedrock 사용 시)
   - IDE 실행에서는 토큰 메트릭을 “N/A”로 수용
