# aidlc-runner

전체 AI-DLC(AI 주도 개발 수명 주기) 워크플로를 구동하는 두 에이전트 오케스트레이터입니다. 비전 문서와 선택적 기술 환경 문서가 주어지면 aidlc-runner는 **Executor** 에이전트와 **Human Simulator** 에이전트를 조율해 요구사항부터 코드 생성까지 소프트웨어 프로젝트를 이끌며, 모든 문서 산출물과 동작하는 애플리케이션 코드를 생성합니다.

## 동작 방식

aidlc-runner는 서로 핸드오프하는 두 에이전트로 [Strands Agents](https://github.com/strands-agents) swarm을 만듭니다.

1. **Executor** — AIDLC 워크플로를 단계별로 진행합니다. 각 단계에 맞는 규칙 파일을 로드하고, 산출물(요구사항, 설계, 코드)을 만들며, 인간 입력이 필요할 때 시뮬레이터로 넘깁니다.
2. **Human Simulator** — 지식 있는 인간 이해관계자 역할을 합니다. 비전과 기술 환경 문서를 바탕으로 명확화 질문에 답하고, 문서를 승인하고, 생성 코드를 검토한 뒤 executor로 다시 넘깁니다.

에이전트는 전체 Inception·Construction 단계를 거쳐 전체 애플리케이션이 생성될 때까지 이 핸드오프 루프를 반복합니다.

### 워크플로 단계

**Inception 단계** — 무엇을 만들고 왜 만드는지:

| 단계 | 조건 |
|---|---|
| Workspace Detection | 항상 |
| Reverse Engineering | Brownfield만 |
| Requirements Analysis | 항상 |
| User Stories | 복잡도에 따라 |
| Workflow Planning | 항상 |
| Application Design | 조건부 |
| Units Generation | 조건부 |

**Construction 단계** — 어떻게 만들지(작업 단위마다 실행):

| 단계 | 조건 |
|---|---|
| Functional Design | 조건부 |
| NFR Requirements | 조건부 |
| NFR Design | 조건부 |
| Infrastructure Design | 조건부 |
| Code Generation | 항상 |
| Build and Test | 항상 |

## 사전 요구사항

- Python 3.13+
- [uv](https://github.com/astral-sh/uv)
- Git(AIDLC 규칙 클론용; `--rules-path` 사용 시 불필요)
- Amazon Bedrock 접근 권한이 있는 프로파일로 구성된 AWS CLI

## 설치

저장소 루트에서:

```bash
cd aidlc-runner
uv sync
```

## 사용법

```bash
uv run aidlc-runner --vision <path-to-vision-file> [--tech-env <path-to-tech-env-file>] [options]
```

필수 인자는 `--vision`뿐이며, 만들 내용을 설명하는 마크다운 파일 경로입니다. 선택적으로 `--tech-env`로 어떻게 만들지(언어, 프레임워크, 보안 통제, 테스트 표준)를 정의하는 기술 환경 문서를 줄 수 있습니다. 문서 작성 방법은 [입력 문서 가이드](GUIDE_TO_WRITING_VISION_DOCS.md)를 참고하세요.

### 예시

최소 — 기본값만 사용:

```bash
uv run aidlc-runner --vision ./my-project-vision.md
```

기술 환경 문서 포함:

```bash
uv run aidlc-runner --vision ./my-project-vision.md \
  --tech-env ./my-project-tech-env.md
```

사용자 정의 AWS 프로파일과 리전:

```bash
uv run aidlc-runner --vision ./my-project-vision.md \
  --aws-profile my-profile \
  --aws-region us-east-1
```

GitHub 대신 로컬 AIDLC 규칙 사용:

```bash
uv run aidlc-runner --vision ./my-project-vision.md \
  --rules-path /opt/aidlc-workflows
```

출력 디렉터리와 설정 파일 지정:

```bash
uv run aidlc-runner --vision ./my-project-vision.md \
  --config ./my-config.yaml \
  --output-dir ./my-runs
```

모델 ID 재정의:

```bash
uv run aidlc-runner --vision ./my-project-vision.md \
  --executor-model us.anthropic.claude-opus-4-20250514-v1:0 \
  --simulator-model us.anthropic.claude-opus-4-20250514-v1:0
```

### CLI 참고

| 플래그 | 필수 | 기본값 | 설명 |
|---|---|---|---|
| `--vision PATH` | 예 | — | 비전/제약 마크다운 파일 경로 |
| `--tech-env PATH` | 아니오 | — | 기술 환경 마크다운 파일 경로 |
| `--config PATH` | 아니오 | 내장 기본값 | YAML 설정 파일 경로 |
| `--aws-profile TEXT` | 아니오 | `default` | AWS 프로파일 이름 |
| `--aws-region TEXT` | 아니오 | `us-west-2` | Bedrock용 AWS 리전 |
| `--executor-model TEXT` | 아니오 | Claude Opus 4 | Executor 에이전트 모델 ID |
| `--simulator-model TEXT` | 아니오 | Claude Sonnet 4.5 | Simulator 에이전트 모델 ID |
| `--output-dir PATH` | 아니오 | `../runs` | 실행 폴더가 생성되는 디렉터리 |
| `--rules-path PATH` | 아니오 | Git에서 클론 | 로컬 AIDLC 규칙 디렉터리 경로 |
| `--no-exec` | 아니오 | 활성화 | 워크플로 내 명령 실행 비활성화 |
| `--no-post-tests` | 아니오 | 활성화 | 실행 후 테스트 비활성화 |

## 설정

설정은 **CLI 플래그 > YAML 설정 > 내장 기본값** 순으로 적용됩니다.

### YAML 설정 파일

YAML 파일을 만들고 `--config`로 넘깁니다. 지정하지 않은 값은 내장 기본값으로 떨어집니다.

```yaml
aws:
  profile: "my-profile"
  region: "us-east-1"

models:
  executor:
    provider: "bedrock"
    model_id: "us.anthropic.claude-opus-4-20250514-v1:0"
  simulator:
    provider: "bedrock"
    model_id: "us.anthropic.claude-opus-4-20250514-v1:0"

aidlc:
  rules_source: "git"                  # "git" or "local"
  rules_repo: "https://github.com/awslabs/aidlc-workflows.git"
  rules_local_path: null               # set when rules_source is "local"

swarm:
  max_handoffs: 200
  max_iterations: 200
  execution_timeout: 14400             # 4 hours, in seconds
  node_timeout: 3600                   # 1 hour, in seconds

runs:
  output_dir: "../runs"
```

### 내장 기본값

내장 기본값은 위와 동일합니다. 기본 설정은 `aidlc-runner/config/default.yaml`에 포함됩니다.

## 실행 출력

각 호출은 출력 디렉터리 아래에 타임스탬프 실행 폴더를 만듭니다.

```
runs/
└── 20260212T143022-a1b2c3d4e5f6.../
    ├── run-meta.yaml              # Metadata: timestamps, config snapshot, status
    ├── run-metrics.yaml           # NFR metrics: tokens, timing, artifacts, errors
    ├── test-results.yaml          # Test pass/fail results (if post-run tests enabled)
    ├── vision.md                  # Copy of the input vision file
    ├── tech-env.md                # Copy of the input tech-env file (if provided)
    ├── aidlc-rules/               # AIDLC workflow rules (cloned or copied)
    │   ├── aws-aidlc-rules/
    │   └── aws-aidlc-rule-details/
    ├── aidlc-docs/                # Documentation artifacts from the workflow
    │   ├── inception/             # Requirements, user stories, designs, etc.
    │   ├── construction/          # Functional design, code plans, reviews
    │   ├── aidlc-state.md         # Current workflow state tracker
    │   └── audit.md               # Timestamped audit log of all stages
    └── workspace/                 # Generated application code
        ├── src/
        ├── tests/
        ├── pyproject.toml
        └── ...
```

`run-meta.yaml`에는 전체 실행 맥락이 기록됩니다 — 시작/종료 시각, 상태, 총 핸드오프 수, 노드 이력, 사용된 설정 스냅샷. `run-metrics.yaml`에는 NFR 메트릭이 담깁니다 — 토큰 사용량(총·에이전트별), 핸드오프 타이밍과 패턴, 생성 산출물 수와 코드 줄 수, 오류/재시도 이벤트.

## 개발

### 테스트 실행

```bash
cd aidlc-runner
uv run pytest
```

### 린트

```bash
uv run ruff check . && uv run ruff format .
```

### 프로젝트 구조

```
aidlc-runner/
├── config/
│   └── default.yaml                # Default configuration
├── src/aidlc_runner/
│   ├── __init__.py                 # Package version (0.1.0)
│   ├── __main__.py                 # python -m aidlc_runner entry point
│   ├── cli.py                      # Argument parsing and main()
│   ├── config.py                   # Configuration dataclasses and loading
│   ├── runner.py                   # Run folder creation, rules setup, swarm orchestration
│   ├── metrics.py                  # Metrics collection, artifact scanning, YAML output
│   ├── progress.py                 # Callback handlers and swarm hooks for progress reporting
│   ├── post_run.py                 # Post-run test evaluation
│   ├── agents/
│   │   ├── executor.py             # Executor agent factory
│   │   └── simulator.py            # Simulator agent factory
│   └── tools/
│       ├── file_ops.py             # Sandboxed read/write/list file tools
│       ├── rule_loader.py          # AIDLC rule file loader with path resolution
│       └── run_command.py          # Sandboxed shell command execution tool
├── tests/
│   ├── test_config.py              # Configuration unit tests
│   ├── test_metrics.py             # Metrics collection and artifact scanning tests
│   ├── test_post_run.py            # Post-run evaluation tests
│   ├── test_run_command.py         # Command execution and sandboxing tests
│   └── test_two_inputs.py          # Two-input-document (vision + tech-env) tests
└── pyproject.toml
```

### 주요 모듈

- **cli.py** — CLI 인자 파싱(`--vision`, 선택적 `--tech-env` 포함), 설정 로드, `runner.run()` 호출.
- **config.py** — `RunnerConfig`와 중첩 dataclass(`AwsConfig`, `ModelConfig`, `SwarmConfig` 등) 정의. 기본값, YAML, CLI 재정의 병합.
- **runner.py** — 실행 폴더 생성, 비전·선택적 tech-env 복사, 규칙 설정, 두 에이전트 빌드, `Swarm` 생성·실행, 메트릭 기록.
- **metrics.py** — `MetricsCollector`가 실행 중 핸드오프 타이밍과 오류 이벤트를 누적하고, 실행 후 토큰 사용량·산출물 수·핸드오프 패턴을 `run-metrics.yaml`로 조립.
- **progress.py** — `AgentProgressHandler`가 도구 호출을 출력하고 에이전트별 오류 이벤트를 감지. `SwarmProgressHook`이 Strands 훅 이벤트로 노드 단위 핸드오프 타이밍을 추적.
- **post_run.py** — 실행 후 테스트 평가: `workspace/`에서 프로젝트 유형 감지, 의존성 설치, 테스트 실행, `test-results.yaml` 기록.
- **agents/executor.py** — 파일 도구, 규칙 로더 도구, 선택적 `run_command` 도구로 executor `Agent` 구축. 시스템 프롬프트에 전체 AIDLC 단계 순서와 핸드오프 프로토콜 인코딩.
- **agents/simulator.py** — 파일 도구로 simulator `Agent` 구축. 시스템 프롬프트에 비전 문서 내용을 동적으로 삽입하고, 제공 시 기술 환경 문서도 포함.
- **tools/file_ops.py** — `make_file_tools(run_folder)`가 실행 폴더로 범위가 제한된 `read_file`, `write_file`, `list_files`를 반환하며 경로 순회 탐지 방지.
- **tools/rule_loader.py** — `make_rule_loader(rules_dir)`가 `load_rule`을 반환해 축약 경로(예: `"inception/requirements-analysis"`)를 전체 규칙 파일 경로로 해석.
- **tools/run_command.py** — `make_run_command(run_folder)`가 Build and Test 중 실행 폴더 내 셸 명령용 샌드박스 `run_command`를 반환.
