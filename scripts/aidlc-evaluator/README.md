# AI-DLC 워크플로 평가 및 보고 프레임워크

[AI-DLC workflows](https://github.com/awslabs/aidlc-workflows) 저장소 변경을 검증하기 위한 자동화 테스트 및 보고 프레임워크입니다.

## 개요

이 프레임워크는 여섯 가지 주요 작업 흐름(“big rocks”)으로 구성됩니다.

1. **골든 테스트 케이스** — 모든 평가의 참조 입력으로 사용되는, 선별된 기준 테스트 케이스(AIDLC 문서 + 코드 출력)
2. **실행 프레임워크** — 테스트 케이스를 평가 파이프라인으로 실행하는 핵심 오케스트레이션
3. **의미 평가** — 출력의 정확성·완전성·적절성에 대한 AI 기반 평가(비결정성을 고려해 @k로 보고)
4. **코드 평가** — 생성 코드에 대한 정적 분석(린트, 보안 스캔, 중복 탐지)
5. **NFR 평가** — 비기능 요구사항 테스트(토큰 소비, 실행 시간, 모델 간 일관성)
6. **GitHub CI/CD 통합 및 관리** — PR에서 평가를 트리거하고 보고서를 첨부하는 자동 파이프라인

## 빠른 시작

```bash
# Install dependencies
uv sync

# Run all unit tests
uv run python run.py test
# Note: On Windows, 7 tests in test_run_command.py are expected to fail
# because they use Unix shell commands (echo, exit, sleep, etc.) not available on Windows.

# Build sandbox docker image
./docker/sandbox/build.sh

# Full pipeline: execute AIDLC workflow + evaluate + report (requires Bedrock) with defaults
uv run python run.py full

# Full pipeline: execute AIDLC workflow + evaluate + report (requires Bedrock)
uv run python run.py full \
    --vision test_cases/sci-calc/vision.md \
    --tech-env test_cases/sci-calc/tech-env.md \
    --golden test_cases/sci-calc/golden-aidlc-docs \
    --openapi test_cases/sci-calc/openapi.yaml

# Evaluate an existing run (skip execution, just score via Bedrock)
uv run python run.py full \
    --evaluate-only runs/<run-folder>/aidlc-docs \
    --golden test_cases/sci-calc/golden-aidlc-docs \
    --openapi test_cases/sci-calc/openapi.yaml
```

## 평가 파이프라인

평가 파이프라인(`run.py full` 또는 `scripts/run_evaluation.py`)은 여섯 단계를 조율합니다.

| 단계 | 패키지 | 설명 |
| ------- | --------- | ------------- |
| 1. 실행 | `packages/execution` | AIDLC 두 에이전트 워크플로를 실행해 문서 + 코드 생성 |
| 2. 실행 후 | (실행 패키지 내부) | 의존성 설치 후 생성 프로젝트 테스트 실행 |
| 3. 정량 | `packages/quantitative` | 생성 코드에 대해 린트, 보안 스캔, 중복 검사 |
| 4. 계약 | `packages/contracttest` | 생성 앱을 띄우고 API 엔드포인트 검증 |
| 5. 정성 | `packages/qualitative` | Bedrock LLM으로 생성 문서를 골든 기준선과 비교 |
| 6. 보고 | `packages/reporting` | 통합 Markdown + HTML 보고서 생성 |

각 실행의 출력은 `runs/` 아래 타임스탬프 폴더에 기록됩니다.

```txt
runs/<timestamp>/
  ├── aidlc-docs/                    # AIDLC workflow documents
  ├── workspace/                     # Generated application code
  ├── run-meta.yaml                  # Run identity + config
  ├── run-metrics.yaml               # Tokens, timing, artifacts, errors
  ├── test-results.yaml              # Post-run test output
  ├── quality-report.yaml            # Lint + security + duplication findings
  ├── contract-test-results.yaml     # API endpoint validation
  ├── qualitative-comparison.yaml    # Semantic scoring
  ├── evaluation-config.yaml         # Full resolved config snapshot
  ├── report.md                      # Consolidated Markdown report
  └── report.html                    # Consolidated HTML report
```

## 설정

### 설정 파일 (`config/default.yaml`)

`default.yaml`은 AWS 설정, 모델, swarm 파라미터, 타임아웃, 도구 경로를 제어합니다. 기본값을 바꾸려면 이 파일을 수정하거나 `--config`로 사용자 정의 설정을 넘깁니다.

```yaml
aws:
  profile: "default"
  region: "us-east-1"

models:
  executor:
    provider: "bedrock"
    model_id: "global.anthropic.claude-opus-4-6-v1"
  simulator:
    provider: "bedrock"
    model_id: "global.anthropic.claude-opus-4-6-v1"
  scorer:
    provider: "bedrock"
    model_id: "global.anthropic.claude-opus-4-6-v1"

aidlc:
  rules_source: "git"   # "git" or "local"
  rules_repo: "https://github.com/awslabs/aidlc-workflows.git"
  rules_ref: "main"
  rules_local_path: null

swarm:
  max_handoffs: 200
  max_iterations: 200
  execution_timeout: 14400
  node_timeout: 3600

runs:
  output_dir: "./runs"

execution:
  enabled: true
  command_timeout: 120
  post_run_tests: true
  post_run_timeout: 300

execution:
  sandbox:
    enabled: true
    image: aidlc-sandbox:latest
    memory: 2g
    cpus: 2

tools:
  pmd_path: null   # Path to PMD executable; if null, looks for 'pmd' on PATH
```

우선순위: `CLI 플래그 > YAML 설정 > 내장 기본값`

### 모델별 설정

`config/`의 모델별 파일은 `default.yaml`에서 나머지를 상속하면서 executor 모델만 덮어씁니다.

| 파일 | 모델 |
| ------ | ------- |
| `config/opus-4-6.yaml` | Claude Opus 4.6 |
| `config/opus-4-5.yaml` | Claude Opus 4.5 |
| `config/sonnet-4-6.yaml` | Claude Sonnet 4.6 |
| `config/sonnet-4-5.yaml` | Claude Sonnet 4.5 |
| `config/nova-premier.yaml` | Amazon Nova Premier |
| `config/nova-pro.yaml` | Amazon Nova Pro |
| `config/nova-lite.yaml` | Amazon Nova Lite |
| `config/mistral-large-3.yaml` | Mistral Large 3 (675B) |
| `config/devstral-2.yaml` | Mistral Devstral 2 (123B, code-specialized) |

### Docker 샌드박스

평가 프레임워크는 신뢰할 수 없는 AI 생성 코드가 호스트에 영향을 주지 않도록 격리된 Docker 컨테이너에서 실행합니다. 샌드박스 이미지에는 Python 3.13 + uv, Node.js 22 + npm, 일반 빌드 도구가 포함되며 비루트 사용자로 실행됩니다.

#### 사전 요구사항

호스트에 Docker가 설치되어 실행 중이어야 합니다.

#### 샌드박스 이미지 빌드

```bash
# Build the image (one-time setup, or after Dockerfile changes)
./docker/sandbox/build.sh

# Or build manually
docker build -t aidlc-sandbox:latest docker/sandbox/
```

기본 설정에서 참조하는 `aidlc-sandbox:latest` 이미지가 생성됩니다.

#### 설정

샌드박스 설정은 `config/default.yaml`의 `execution.sandbox`에 있습니다.

```yaml
execution:
  sandbox:
    enabled: true                # Set to false to run generated code directly on the host
    image: aidlc-sandbox:latest  # Docker image name (must be built first)
    memory: 2g                   # Container memory limit
    cpus: 2                      # Container CPU limit
```

샌드박스가 켜져 있으면 실행 후 테스트(2단계)와 계약 테스트 서버(4단계)가 컨테이너 안에서 실행됩니다. 생성된 `workspace/` 디렉터리는 컨테이너의 `/workspace`에 마운트됩니다. Docker를 쓸 수 없거나 `enabled`가 `false`이면 명령은 호스트에서 직접 실행됩니다.

### 도구 설정

**PMD(코드 중복 탐지):** 3단계에서 복사-붙여넣기 탐지에 PMD CPD를 사용합니다. `config/default.yaml`에서 경로를 설정합니다.

```yaml
tools:
  pmd_path: /path/to/pmd    # Absolute path to PMD executable
  # pmd_path: null           # null = search PATH automatically
```

PMD를 찾지 못하면 중복 분석은 비고와 함께 건너뛰며 평가 자체는 실패하지 않습니다.

### 파이프라인 CLI 플래그

```bash
uv run python run.py full \
    --vision test_cases/sci-calc/vision.md \
    --tech-env test_cases/sci-calc/tech-env.md \
    --golden test_cases/sci-calc/golden-aidlc-docs \
    --openapi test_cases/sci-calc/openapi.yaml \
    --config config/default.yaml \
    --profile my-aws-profile \
    --region us-west-2 \
    --executor-model global.anthropic.claude-opus-4-6-v1 \
    --scorer-model us.anthropic.claude-sonnet-4-5-20250929-v1:0 \
    --report-format both
```

지원 플래그:

- `--config` — YAML 설정 파일 경로(기본: `config/default.yaml`)
- `--test` — 단위 테스트만 실행
- `--vision`, `--tech-env` — 실행 입력
- `--evaluate-only` — 실행을 다시 하지 않고 기존 `aidlc-docs` 폴더만 채점
- `--golden` — 참조 기준선 문서 디렉터리
- `--openapi` — 계약 테스트 스펙
- `--report-format` — `markdown`, `html`, 또는 `both`
- `--baseline` — `golden.yaml` 경로 재정의(지정하지 않으면 `--golden` 옆에서 자동 탐색)
- `--output-dir` — 실행 출력 폴더 재정의
- `--results` — 정성 결과 YAML을 사용자 지정 경로에 기록
- `--profile`, `--region` — Bedrock용 AWS 자격 증명/리전
- `--executor-model` — 실행 모델 재정의
- `--scorer-model` — 정성 채점 모델 재정의
- `--rules-ref` — AIDLC 규칙용 Git ref(브랜치/태그/커밋)

## 배치 평가

여러 Bedrock 모델에 대해 평가 파이프라인을 순차 실행한 뒤 모델 간 비교 보고서를 생성합니다.

### 사용 가능한 모델 목록

```bash
uv run python run.py batch --list
```

### 배치 평가 실행

```bash
# Run all configured models
uv run python run.py batch --models all

# Run specific models (names match config file stems in config/)
uv run python run.py batch --models nova-pro,sonnet-4-5

# Override AWS settings
uv run python run.py batch --models all \
    --profile my-aws-profile \
    --region us-east-1
```

모델별 실행은 `runs/<model-name>/`에 전체 평가 산출물과 함께 저장됩니다. `batch-summary.yaml`이 runs 디렉터리에 기록되며 모델별 타이밍과 통과/실패 상태를 담습니다.

### 모델 간 비교 생성

배치 평가가 끝난 뒤 비교 행렬을 생성합니다.

```bash
# Compare all model runs found under runs/
uv run python run.py compare

# Compare specific models against golden baseline
uv run python run.py compare \
    --models nova-pro,sonnet-4-5 \
    --baseline test_cases/sci-calc/golden.yaml
```

`runs/comparison/comparison-report.md`와 `runs/comparison/comparison-data.yaml`이 생성되며, 모든 모델에 대해 단위 테스트·계약 테스트·코드 품질·정성 점수·토큰 사용량·타이밍을 나란히 비교합니다.

## CLI 평가

CLI 기반 AI 어시스턴트(Claude Code, Kiro CLI)로 AIDLC 평가를 실행하려면 CLI 하네스(`packages/cli-harness`)를 사용합니다.

### 사용 가능한 어댑터 목록

```bash
uv run python run.py cli --list
```

지원 어댑터: `claude-code`, `kiro-cli`.

### CLI 평가 실행

```bash
# Run evaluation through Claude Code
uv run python run.py cli --cli claude-code \
    --vision test_cases/sci-calc/vision.md \
    --golden test_cases/sci-calc/golden-aidlc-docs

# Run through Kiro CLI with a specific model
uv run python run.py cli --cli kiro-cli \
    --vision test_cases/sci-calc/vision.md \
    --golden test_cases/sci-calc/golden-aidlc-docs \
    --model claude-sonnet-4

# Check prerequisites for an adapter
uv run python run.py cli --cli claude-code --check-only
```

출력은 `runs/<cli-name>-<timestamp>-<uuid>/`에 기록됩니다. CLI 하네스는 어댑터를 실행한 뒤 `scripts/run_evaluation.py --evaluate-only`로 채점(2–6단계)을 호출합니다.

## IDE 평가

서드파티 IDE AI 어시스턴트로 AIDLC 평가를 실행하려면 IDE 하네스(`packages/ide-harness`)를 사용합니다.

### 사용 가능한 어댑터 목록

```bash
uv run python run.py ide --list
```

지원 어댑터: Cursor, Cline, Copilot, Kiro, Windsurf, Antigravity.

### IDE 평가 실행

```bash
# Run evaluation through Cursor
uv run python run.py ide --ide cursor \
    --vision test_cases/sci-calc/vision.md \
    --golden test_cases/sci-calc/golden-aidlc-docs

# Check prerequisites for an IDE adapter
uv run python run.py ide --ide kiro --check-only
```

출력은 `runs/ide-<adapter-name>/`에 기록됩니다.

## 확장 훅 테스트

규칙 확장 설정이 다른 AIDLC 워크플로를 테스트합니다. 확장 훅 기능은 옵트인 질문에 따라 확장(보안, 성능, 관측 가능성)을 단계적으로 로드할 수 있습니다.

```bash
# List available extension configurations
uv run python run.py ext-test --list-configs

# Run standard test (all extensions vs no extensions)
uv run python run.py ext-test --scenario sci-calc

# Use specific rules branch with extension support
uv run python run.py ext-test --scenario sci-calc \
    --rules-ref feat/extension_hook_question_split
```

평가가 두 번 실행됩니다.
1. 모든 확장 옵트인에 “YES”(최대 가이드)
2. 모든 확장 옵트인에 “NO”(기준선만)

결과는 `runs/<scenario>/extension-test/`에 저장되며, 확장 설정별 영향을 보여 주는 비교 보고서가 포함됩니다.

자세한 내용은 [확장 훅 테스트 가이드](./docs/extension-hook-testing.md)를 참고하세요.

## 트렌드 보고

시간에 따른 평가 지표를 추적하는 릴리스 간 트렌드 보고서를 생성합니다. GitHub 릴리스와 Actions 아티팩트에서 평가 번들을 가져와 HTML, Markdown, YAML 보고서를 렌더링합니다.

```bash
# Generate trend report (requires gh CLI authenticated)
uv run python run.py trend --baseline test_cases/sci-calc/golden.yaml

# HTML only with verbose output
uv run python run.py trend --baseline test_cases/sci-calc/golden.yaml --format html -v

# Include local evaluation bundles
uv run python run.py trend --baseline test_cases/sci-calc/golden.yaml \
    --local-bundle runs/my-run/report.zip

# Gate mode (exit non-zero on regressions)
uv run python run.py trend --baseline test_cases/sci-calc/golden.yaml --gate
```

HTML 요약에는 여섯 가지 지표 카드가 표시됩니다.

- **정성 점수** — 골든 기준선 대비 의미 품질(높을수록 좋음)
- **계약 테스트** — API 통과율(통과/전체, 높을수록 좋음)
- **단위 테스트** — 통과율을 백분율로 표시(높을수록 좋음)
- **린트 발견** — 정적 분석 이슈(낮을수록 좋음)
- **실행 시간** — 생성 소요 시간(낮을수록 좋음)
- **총 토큰** — LLM 토큰 소비(낮을수록 좋음)

출력은 출력 디렉터리(기본: `runs/`) 아래 타임스탬프 폴더에 기록됩니다.

샘플 HTML 보고서는 [`packages/trend-reports/examples/trend-report.html`](./packages/trend-reports/examples/trend-report.html)에서 볼 수 있습니다.

## 실행 컴포넌트 직접 실행

실행 수준 제어를 모두 쓰려면 `aidlc-runner`를 직접 실행할 수 있습니다.

```bash
uv run aidlc-runner \
    --vision test_cases/sci-calc/vision.md \
    --tech-env test_cases/sci-calc/tech-env.md \
    --config config/default.yaml \
    --aws-profile my-aws-profile \
    --aws-region us-west-2 \
    --executor-model global.anthropic.claude-opus-4-6-v1 \
    --simulator-model us.anthropic.claude-sonnet-4-5-20250929-v1:0 \
    --output-dir ./runs
```

실행 전용 옵션:

- `--rules-path <local-rules-dir>` — 로컬 규칙 소스 강제
- `--no-exec` — 워크플로 내 명령 실행 비활성화
- `--no-post-tests` — 실행 후 테스트 비활성화

## 저장소 구조

```txt
.
├── run.py                     # Master entry point — dispatches to evaluation modes
├── scripts/                   # Specialized run scripts
│   ├── run_evaluation.py      # Single-model evaluation pipeline
│   ├── run_batch_evaluation.py    # Multi-model batch evaluation
│   ├── run_comparison_report.py   # Cross-model comparison report generator
│   ├── run_cli_evaluation.py      # CLI adapter evaluation runner
│   ├── run_ide_evaluation.py      # IDE adapter evaluation runner
│   ├── run_extension_test.py      # Extension hook testing (opt-in configurations)
│   ├── run_trend_report.py        # Cross-release trend report generation
│   └── README.md              # Scripts documentation
├── config/
│   ├── default.yaml           # Default configuration (models, AWS, timeouts, tools)
│   ├── nova-premier.yaml      # Amazon Nova Premier executor override
│   ├── nova-pro.yaml          # Amazon Nova Pro executor override
│   ├── sonnet-4-5.yaml        # Claude Sonnet 4.5 executor override
│   └── sonnet-4-6.yaml        # Claude Sonnet 4.6 executor override
├── packages/
│   ├── execution/             # AIDLC workflow runner (two-agent Strands orchestrator)
│   ├── qualitative/           # Semantic evaluation — intent & design similarity via Bedrock
│   ├── quantitative/          # Code evaluation — linting, security, duplication (PMD CPD)
│   ├── contracttest/          # API contract testing against OpenAPI specs
│   ├── nonfunctional/         # NFR evaluation — tokens, timing, consistency
│   ├── reporting/             # Consolidated report generation (Markdown + HTML)
│   ├── trend-reports/         # Cross-release trend reporting (HTML, Markdown, YAML)
│   ├── cli-harness/           # CLI adapter framework (Claude Code, Kiro CLI)
│   ├── ide-harness/           # IDE adapter framework (Cursor, Cline, Kiro, etc.)
│   └── shared/                # Common utilities
├── test_cases/                # Golden test cases (vision + tech-env + golden aidlc-docs)
├── runs/                      # Run output folders (one per evaluation run)
├── docker/
│   └── sandbox/               # Dockerfile + build script for isolated execution
├── docs/                      # Additional documentation
│   ├── extension-hook-testing.md  # Extension hook testing guide
│   ├── ide-harness-design.md      # IDE adapter architecture
│   └── file-structure.md          # Project file organization reference
├── pyproject.toml             # Workspace configuration
└── uv.lock                    # Dependency lock file
```

## 문서

- [FAQ](./FAQ.md) — 자주 묻는 질문
- [기여](./CONTRIBUTING.md) — 변경 제출 가이드
- [아키텍처](./ARCHITECTURE.md) — 시스템 설계 및 구현 세부
- [확장 훅 테스트](./docs/extension-hook-testing.md) — 확장 설정이 다른 AIDLC 테스트
- [IDE 하네스 설계](./docs/ide-harness-design.md) — IDE 어댑터 프레임워크 구조
- [파일 구조](./docs/file-structure.md) — 프로젝트 파일 구성 참고

## 기여

변경 제출 가이드는 [CONTRIBUTING.md](./CONTRIBUTING.md)를 참고하세요.

## 라이선스

[License information to be added]
