# AIDLC 평가 프레임워크 — 설계 문서

## 1. 목적

이 문서는 **AI-DLC Workflows 평가 및 보고 프레임워크**의 아키텍처, 설계 결정, 데이터 흐름, 내부 동작을 설명합니다. 시스템 동작 이해, 확장, 디버깅이 필요한 개발자를 대상으로 합니다.

프레임워크는 [AI-DLC workflows](https://github.com/awslabs/aidlc-workflows) 저장소 변경을 AI 주도 소프트웨어 개발 수명 주기를 끝까지 실행한 뒤, 기능 정확성, 코드 품질, API 계약 준수, 골든 기준선과의 의미 유사성 등 여러 품질 차원에서 출력을 채점해 검증합니다.

---

## 2. 상위 수준 아키텍처

```
                              ┌──────────────────────┐
                              │   Entry Points (CLI) │
                              └──────────┬───────────┘
                 ┌───────────────────────┼──────────────────────┐
                 │                       │                      │
        run_evaluation.py      run_batch_evaluation.py   run_ide_evaluation.py
         (single model)          (multi-model loop)       (IDE adapter)
                 │                       │                      │
                 └───────────┬───────────┘                      │
                             │                                  │
              ┌──────────────▼──────────────┐   ┌───────────────▼──────────┐
              │       6-Stage Pipeline      │   │     IDE Harness          │
              │  ┌──────────────────────┐   │   │  ┌───────────────────┐   │
              │  │ 1. Execution         │   │   │  │ Adapter (Cursor,  │   │
              │  │    (Strands Swarm)   │   │   │  │ Cline, Kiro, ...) │   │
              │  ├──────────────────────┤   │   │  └────────┬──────────┘   │
              │  │ 2. Post-Run Tests    │   │   │           │              │
              │  ├──────────────────────┤   │   │  ┌────────▼──────────┐   │
              │  │ 3. Quantitative      │   │   │  │ Output Normalizer │   │
              │  ├──────────────────────┤   │   │  └────────┬──────────┘   │
              │  │ 4. Contract Tests    │   │   │           │              │
              │  ├──────────────────────┤   │   └───────────┼──────────────┘
              │  │ 5. Qualitative       │   │               │
              │  ├──────────────────────┤   │    ┌──────────▼──────────┐
              │  │ 6. Report Generation │   │    │ --evaluate-only     │
              │  └──────────────────────┘   │    │ (stages 2-6)        │
              └─────────────────────────────┘    └─────────────────────┘
                             │
              ┌──────────────▼───────────────┐
              │  runs/<timestamp>/           │
              │   ├── aidlc-docs/            │
              │   ├── workspace/             │
              │   ├── run-meta.yaml          │
              │   ├── run-metrics.yaml       │
              │   ├── test-results.yaml      │
              │   ├── quality-report.yaml    │
              │   ├── contract-test-results… │
              │   ├── qualitative-comparison…│
              │   ├── report.md / .html      │
              │   └── evaluation-config.yaml │
              └─────────────────────────────┘
```

---

## 3. 패키지 구조

프로젝트는 루트 `pyproject.toml`에 정의된 **uv workspace**를 사용하며 내부 패키지가 여덟 개입니다. 각 패키지는 자체 `pyproject.toml`, `src/` 레이아웃, `tests/` 디렉터리로 독립 구성됩니다.

| 패키지 | PyPI 이름 | 목적 |
|---------|-----------|---------|
| `packages/execution` | `aidlc-runner` | AIDLC 워크플로를 실행하는 두 에이전트 swarm |
| `packages/qualitative` | `aidlc-qualitative` | 문서와 골든 기준선의 의미 채점 |
| `packages/quantitative` | `aidlc-quantitative` | 정적 분석: 린트, 보안, 중복 |
| `packages/contracttest` | `aidlc-contracttest` | OpenAPI 사양에 대한 API 계약 테스트 |
| `packages/nonfunctional` | `aidlc-nonfunctional` | NFR 평가(토큰, 타이밍, 일관성) |
| `packages/reporting` | `aidlc-reporting` | 통합 보고서 생성(Markdown + HTML) |
| `packages/ide-harness` | (미배포) | 서드파티 AI 어시스턴트용 IDE 어댑터 프레임워크 |
| `packages/shared` | `aidlc-shared` | 패키지 간 공통 유틸리티 |

**의존성 그래프**(단순화):

```
run_evaluation.py ──► execution (aidlc-runner)
                  ──► quantitative
                  ──► contracttest
                  ──► qualitative
                  ──► reporting ──► reporting.collector
                                ──► reporting.baseline
                                ──► reporting.render_md
                                ──► reporting.render_html
```

모든 패키지는 **디스크상의 YAML 파일**로 통신합니다. 평가 패키지 간 인프로세스 라이브러리 수준 의존성은 없으며 — 오케스트레이터(`run_evaluation.py`)가 `python -m <package>`로 각 패키지를 서브프로세스로 호출하고 파일 경로를 인자로 넘깁니다. 이 설계로 패키지를 독립적으로 테스트할 수 있고 각각 단독 실행이 가능합니다.

---

## 4. 설정 시스템

### 4.1 계층적 설정 해석

설정은 세 단계 우선순위 모델을 따릅니다.

```
CLI flags  >  YAML config file  >  Built-in Python defaults
```

1. **내장 기본값**은 `packages/execution/src/aidlc_runner/config.py`의 dataclass 필드 기본값으로 정의됩니다(`RunnerConfig` 및 중첩 dataclass).
2. **YAML 설정**은 `config/default.yaml`(또는 `--config`로 지정한 경로)에서 로드됩니다. `_merge_dict_into_dataclass()`가 YAML 값을 dataclass 트리에 재귀적으로 덮어씁니다.
3. **CLI 플래그**(예: `--executor-model`, `--profile`)는 마지막에 적용되어 YAML과 기본값을 모두 재정의합니다.

### 4.2 설정 Dataclass 계층

```python
RunnerConfig
  ├── aws: AwsConfig          # profile, region
  ├── models: ModelsConfig
  │     ├── executor: ModelConfig   # provider, model_id
  │     └── simulator: ModelConfig
  ├── aidlc: AidlcConfig      # rules_source, rules_repo, rules_ref
  ├── swarm: SwarmConfig       # max_handoffs, max_iterations, timeouts
  ├── runs: RunsConfig         # output_dir
  └── execution: ExecutionConfig  # enabled, command_timeout, post_run_tests
```

### 4.3 모델별 설정 파일

`config/`의 파일(예: `config/sonnet-4-5.yaml`, `config/nova-pro.yaml`)은 `models.executor.model_id` 필드만 덮어씁니다. 배치 러너(`run_batch_evaluation.py`)가 `config/*.yaml`을 스캔하고 `default.yaml`을 제외해 자동으로 발견합니다.

---

## 5. 단계별 파이프라인 설계

### 5.1 1단계: 실행 (`packages/execution`)

프레임워크의 핵심입니다. **Strands SDK** 다중 에이전트 오케스트레이션으로 전체 AIDLC 워크플로를 실행합니다.

#### 두 에이전트 Swarm 아키텍처

```
                    ┌──────────────────────┐
                    │   Strands Swarm      │
                    │                      │
  initial prompt ──►│  ┌────────────────┐  │
                    │  │   Executor     │  │
                    │  │   Agent        │◄─┤── handoff ──┐
                    │  │                ├──┤── handoff ──│
                    │  └────────────────┘  │             │
                    │                      │  ┌──────────▼─┐
                    │                      │  │ Simulator  │
                    │                      │  │ Agent      │
                    │                      │  └────────────┘
                    └──────────────────────┘
```

**Executor 에이전트** — 모든 단계(Inception → Construction)에서 AIDLC 워크플로를 이끕니다.
- `load_rule` 도구로 필요 시 AIDLC 규칙 파일 로드(지연 로딩으로 컨텍스트 창 사용량 감소)
- 샌드박스된 `read_file`, `write_file`, `list_files`로 실행 폴더에서 파일 읽기/쓰기
- `run_command`로 셸 명령 실행(의존성 설치, 테스트 실행)
- 인간 입력이 필요할 때(질문, 승인, 검토) Simulator로 핸드오프

**Simulator 에이전트** — 시뮬레이션된 인간 이해관계자 역할을 합니다.
- 비전 문서(및 선택적 tech-env 문서)가 시스템 프롬프트에 포함됨
- 명확화 질문에 답하고, 문서를 승인하고, 코드를 검토함
- 항상 Executor로 다시 넘겨 워크플로를 계속함

**주요 설계 결정:**
- **샌드박스 파일 작업**: 모든 파일 도구가 `_resolve_safe()`로 실행 폴더 밖 경로 순회를 방지
- **샌드박스 명령 실행**: `run_command`는 제한된 환경(PATH, HOME, LANG만)으로 실행 격리
- **지연 규칙 로딩**: 시스템 프롬프트에 모든 규칙을 미리 넣지 않고 단계 시작 시 하나씩 로드
- **진행 스트리밍**: `AgentProgressHandler`가 전체 LLM 출력 없이 도구 호출을 stderr에 기록; `SwarmProgressHook`이 핸드오프 타이밍 기록
- **메트릭 수집**: `MetricsCollector`가 토큰 사용량, 핸드오프 타이밍, 컨텍스트 크기 샘플, 실행 중 오류 이벤트 기록

#### AIDLC 워크플로 단계

Executor가 다음 순서를 진행합니다(프로젝트 범위에 따라 일부 단계는 조건부).

| # | 단계 | 페이즈 | 조건부? |
|---|-------|-------|-------------|
| 1 | Workspace Detection | Inception | 항상 |
| 2 | Reverse Engineering | Inception | Brownfield만 |
| 3 | Requirements Analysis | Inception | 항상 |
| 4 | User Stories | Inception | 복잡할 때 |
| 5 | Workflow Planning | Inception | 항상 |
| 6 | Application Design | Inception | 필요 시 |
| 7 | Units Generation | Inception | 필요 시 |
| 8 | Functional Design | Construction | 필요 시 |
| 9 | NFR Requirements | Construction | 필요 시 |
| 10 | NFR Design | Construction | 필요 시 |
| 11 | Infrastructure Design | Construction | 필요 시 |
| 12 | Code Generation | Construction | 항상 |
| 13 | Build and Test | Construction | 항상 |

각 단계는 실행 전 해당 규칙 파일(예: `inception/requirements-analysis.md`)을 로드합니다. Executor는 모든 문서 산출물을 `aidlc-docs/`에, 생성 코드를 `workspace/`에 씁니다.

#### 규칙 설정

러너는 다음 중 하나를 수행합니다.
- AIDLC 규칙 저장소(기본: `awslabs/aidlc-workflows`, ref 설정 가능)를 실행 폴더에 **Git clone**한 뒤 `aidlc-rules/` 내용 추출
- `rules_source: "local"`일 때 로컬 경로에서 **복사**

#### 실행 폴더 레이아웃

```
runs/<YYYYMMDDTHHMMSS>-<rules_slug>/
  ├── vision.md                      # Copied input
  ├── tech-env.md                    # Copied input (if provided)
  ├── aidlc-rules/                   # AIDLC workflow rules
  │   ├── aws-aidlc-rules/           # Core workflow definition
  │   └── aws-aidlc-rule-details/    # Per-stage rule files
  ├── aidlc-docs/                    # Generated AIDLC documents
  │   ├── inception/                 # Requirements, user stories, design docs
  │   ├── construction/              # Functional design, code-gen docs
  │   ├── aidlc-state.md             # Workflow state tracker
  │   └── audit.md                   # Timestamped audit log
  ├── workspace/                     # Generated application code
  └── run-meta.yaml                  # Run identity and config snapshot
```

#### 실행 후 테스트 평가

Swarm이 끝나면 `post_run.py`가 자동 테스트를 수행합니다.
1. **프로젝트 감지**: `workspace/`를 최대 3단계 깊이로 BFS 스캔해 마커 파일(`pyproject.toml`, `package.json`, `Cargo.toml`, `go.mod`) 탐색
2. **의존성 설치**: 적절한 설치 명령 실행(예: `uv pip install -e ".[dev]"`)
3. **테스트 실행**: 적절한 테스트 명령 실행(예: `uv run pytest`)
4. **출력 파싱**: 언어별 파서가 테스트 출력에서 통과/실패 수 추출(pytest, Jest/Vitest, cargo test, go test)
5. **결과**: `test-results.yaml`에 기록

### 5.2 2단계: 실행 후 테스트(요약)

1단계가 쓴 `test-results.yaml`을 읽고 사람이 읽기 쉬운 요약을 출력합니다. 실행 단계에 포함되어 있으며 — 오케스트레이터가 요약 표시를 위해 이 파일을 읽습니다.

### 5.3 3단계: 정량 분석 (`packages/quantitative`)

`workspace/`의 생성 코드에 대해 정적 분석 도구를 실행합니다. 분석은 언어를 인지합니다.

#### 프로젝트 유형별 도구 선택

| 프로젝트 유형 | 린터 | 보안 스캐너 | 중복 |
|-------------|--------|-----------------|-------------|
| Python | ruff | bandit + semgrep | PMD CPD |
| Node.js | eslint | npm audit + semgrep | PMD CPD |

#### 분석 흐름

```
scan_workspace(path)
  ├── detect project type (pyproject.toml → Python, package.json → Node)
  ├── run_ruff() or run_eslint()         → LintFinding[]
  ├── run_bandit() or run_npm_audit()    → SecurityFinding[]
  ├── run_semgrep()                       → SecurityFinding[]
  ├── run_cpd()                           → DuplicationFinding[]
  └── compute_summary()                   → QualityReport
```

각 도구 러너는:
1. 도구 사용 가능 여부 확인(`shutil.which` 또는 `uv run --version`)
2. JSON 출력 형식으로 실행
3. 구조화된 출력을 표준화된 finding 모델로 파싱
4. finding과 메타데이터가 담긴 `ToolResult` 반환

**우아한 저하**: 도구가 설치되지 않았으면 해당 도구 분석은 비고와 함께 건너뛰며 평가 자체는 실패하지 않습니다.

출력: `quality-report.yaml`

### 5.4 4단계: 계약 테스트 (`packages/contracttest`)

OpenAPI 3.x 사양에 맞춰 생성 애플리케이션의 API 엔드포인트를 검증합니다.

#### 아키텍처

```
openapi.yaml ──► spec.py (parser) ──► ContractSpec
                                          ├── AppConfig (module, port, framework)
                                          └── TestCase[] (from x-test-cases extensions)

workspace/ ──► server.py (ServerProcess) ──► uvicorn subprocess
                                                  │
                                                  ▼
ContractSpec ──► runner.py ──► HTTP requests ──► CaseResult[]
                                                  │
                                                  ▼
                                          ContractTestResults
```

**핵심 동작:**
- **스펙 파싱**: OpenAPI 스펙에 사용자 정의 `x-app`(서버 설정) 및 `x-test-cases`(작업별 테스트 입력/기대 출력) 확장 사용
- **서버 관리**: `ServerProcess`가 워크스페이스 프로젝트용 격리 venv를 만들고 uvicorn을 기동하며, 준비될 때까지 `/health`를 폴링하고 테스트 후 정리 종료
- **테스트 실행**: 각 테스트 케이스가 HTTP 요청을 보내고 상태 코드 일치, 응답 본문에 기대 키/값 포함(부동소수 허용 오차가 있는 재귀적 깊은 일치)을 검증
- **중단 조건**: 서버 프로세스가 죽거나 연속 3회 연결 오류 시 조기 중단

출력: `contract-test-results.yaml`

### 5.5 5단계: 정성 평가 (`packages/qualitative`)

의미 유사도 채점으로 생성된 AIDLC 문서를 골든 기준선과 비교합니다.

#### 문서 매칭

```
golden aidlc-docs/         candidate aidlc-docs/
  inception/                 inception/
    requirements.md    ◄──►   requirements.md         (paired)
    user-stories.md    ◄──►   user-stories.md         (paired)
  construction/              construction/
    code-generation.md ◄──►   code-generation.md      (paired)
                              extra-doc.md             (unmatched candidate)
```

문서는 상대 경로로 짝을 맞춥니다. 내부 워크플로 파일(`aidlc-state.md`, `audit.md`)은 제외합니다.

#### 채점 차원

문서 쌍마다 세 차원(0.0~1.0)으로 채점합니다.

| 차원 | 가중치 | 측정 내용 |
|-----------|--------|-----------------|
| Intent Similarity | 0.4 | 동일한 목표, 요구사항, 목적 |
| Design Similarity | 0.4 | 동일한 아키텍처, 컴포넌트, 패턴 |
| Completeness | 0.2 | 후보가 참조의 모든 주제를 다룸 |

**문서별 전체** = 0.4 × intent + 0.4 × design + 0.2 × completeness

점수는 페이즈별(inception, construction)로 집계된 뒤 전체 점수로 합산됩니다.

#### 두 가지 채점기 구현

**HeuristicScorer**(오프라인, 결정적):
- Intent: 불용어 제거 후 용어 빈도 코사인 유사도
- Design: 기술 식별자 Jaccard(0.6)와 제목 구조 Jaccard(0.4)의 가중 혼합
- Completeness: 참조 제목 중 후보에 존재하는 비율

**LlmScorer**(기본, Bedrock 필요):
- Bedrock `converse` API로 두 문서를 LLM에 전달
- 프롬프트가 세 차원 점수와 메모를 담은 JSON 요청
- 재현성을 위해 temperature 0.0
- 문서당 내용 15K자로 잘림

출력: `qualitative-comparison.yaml`

### 5.6 6단계: 보고서 생성 (`packages/reporting`)

실행 폴더의 모든 YAML 산출물을 모아 통합 보고서를 생성합니다.

#### 데이터 수집

`reporting.collector.collect(run_folder)`가 모든 YAML을 읽어 `ReportData` dataclass를 조립합니다.
- `RunMeta` — 식별, 타이밍, 모델, 규칙
- `RunMetrics` — 토큰(총+에이전트별), 벽시계, 핸드오프 타임라인, 산출물 수, 오류 수, 컨텍스트 크기 통계
- `TestResults` — 단위 테스트 통과/실패/총계와 통과율
- `QualityReport` — 린트, 보안, 중복 finding
- `ContractResults` — 엔드포인트별 테스트 결과
- `QualitativeResults` — 문서별·페이즈별 의미 점수

#### 기준선 비교

`golden.yaml` 기준 파일이 있으면(`--golden` 디렉터리 옆에서 자동 탐색) 보고서에 회귀 비교가 포함됩니다.
1. `extract_baseline()`이 `ReportData`를 약 30개 수치 필드의 `BaselineMetrics`로 평탄화
2. `compare()`가 델타를 계산하고 지표마다 개선/퇴보/변화 없음으로 분류
3. 분류는 방향을 존중(예: 린트 오류 감소 = 개선, 테스트 통과율 상승 = 개선)

#### 출력 형식

- **Markdown**: `render_markdown()`이 판정 배너, 표, 델타 표시, 접을 수 있는 상세 섹션이 있는 GitHub Flavored Markdown 생성
- **HTML**: `render_html()`이 단독 보기용으로 Markdown을 CSS와 함께 래핑

---

## 6. 오케스트레이터

### 6.1 단일 모델 파이프라인 (`run_evaluation.py`)

메인 진입점입니다. 여섯 단계를 순차 조율합니다.

```
parse CLI args
  │
  ├── --test mode ──► run pytest on all packages ──► exit
  │
  ├── --evaluate-only mode ──► skip Stage 1
  │     ├── Stage 3 (quantitative)
  │     ├── Stage 4 (contract)
  │     ├── Stage 5 (qualitative)
  │     └── Stage 6 (report)
  │
  └── full pipeline mode
        ├── Stage 1 (execution) ──► creates timestamped run folder
        ├── Save evaluation config and repo info
        ├── Stage 2 (read test-results.yaml from Stage 1)
        ├── Stage 3 (quantitative)
        ├── Stage 4 (contract, if --openapi provided)
        ├── Stage 5 (qualitative)
        ├── Stage 6 (report)
        └── Print summary, exit 0 if all pass
```

**복원력**: Strands swarm이 0이 아닌 코드로 종료해도 AIDLC 문서가 생성되었으면 평가는 계속됩니다(모든 문서 작성 후 늦은 핸드오프에서 swarm이 실패할 수 있음).

### 6.2 배치 평가 (`run_batch_evaluation.py`)

선택한 모델 설정마다 루프로 `run_evaluation.py`를 실행합니다.

```
discover_models()     ← scans config/*.yaml, excludes default.yaml
  │
  for each model:
  │  ├── build CLI command with --executor-model override
  │  ├── run as subprocess, capture stdout/stderr to log file
  │  ├── find new timestamped run folder
  │  ├── rename folder: <timestamp>-<slug>-<model-name>
  │  └── write per-model batch-summary.yaml
  │
  write batch-summary.yaml with timing and pass/fail for all models
```

모델별 실행은 완전히 격리됩니다 — 각각 별도 서브프로세스 호출과 전용 실행 폴더.

### 6.3 모델 간 비교 (`run_comparison_report.py`)

배치 평가 후 나란히 비교 행렬을 생성합니다.

```
find_model_runs()     ← discovers run folders by model name suffix
  │
  for each model:
  │  └── collect() + extract_baseline() → BaselineMetrics
  │
  load golden baseline (golden.yaml)
  │
  generate_comparison_markdown()   → comparison-report.md
  generate_comparison_yaml()       → comparison-data.yaml
```

비교 표에는 단위 테스트, 계약 테스트, 코드 품질, 정성 점수, 산출물, 실행 비용, 컨텍스트 크기 등 약 30개 지표가 포함되며 — 골든 기준선 대비 델타 표시(^ 개선, v 악화).

### 6.4 IDE 평가 (`run_ide_evaluation.py`)

서드파티 IDE AI 어시스턴트로 AIDLC 워크플로를 실행합니다.

```
get_adapter(name)     ← lazy import from registry
  │
  ├── check_prerequisites()
  ├── adapter.run(config) ──► IDE-specific automation
  ├── normalize_output()  ──► standard run folder layout
  └── run_evaluation.py --evaluate-only  ──► stages 2-6
```

**어댑터 패턴**: 각 IDE는 `IDEAdapter`의 서브클래스로 세 메서드를 구현합니다.
- `check_prerequisites()` — IDE 설치·설정 확인
- `run(config)` — IDE를 통해 AIDLC 프로세스 실행
- `name` — 사람이 읽기 쉬운 식별자

**출력 정규화**: `normalizer.py`가 IDE별 출력 레이아웃을 평가 파이프라인이 기대하는 표준 실행 폴더 구조로 변환하고, 합성 `run-meta.yaml`과 `run-metrics.yaml`을 생성합니다.

지원 어댑터: Cursor, Cline, Copilot, Kiro, Windsurf, Antigravity.

---

## 7. 데이터 흐름: YAML 산출물 그래프

각 단계는 실행 폴더의 YAML 파일로 통신합니다. 단계 경계를 넘는 인메모리 상태는 없습니다.

```
Stage 1 (execution)
  ├── writes: run-meta.yaml, run-metrics.yaml, test-results.yaml
  ├── writes: aidlc-docs/**/*.md, workspace/**/*
  │
Stage 3 (quantitative) reads: workspace/
  └── writes: quality-report.yaml
  │
Stage 4 (contract) reads: workspace/, openapi.yaml (test input)
  └── writes: contract-test-results.yaml
  │
Stage 5 (qualitative) reads: aidlc-docs/, golden-aidlc-docs/ (test input)
  └── writes: qualitative-comparison.yaml
  │
Stage 6 (report) reads: ALL of the above YAML files + golden.yaml
  └── writes: report.md, report.html
```

오케스트레이터는 `evaluation-config.yaml`(전체 해석된 설정 스냅샷)도 쓰고 평가 수준 필드로 `run-meta.yaml`을 갱신합니다.

---

## 8. 주요 데이터 모델

### 8.1 실행 메트릭 (`run-metrics.yaml`)

```yaml
tokens:
  total: {input_tokens, output_tokens, total_tokens, cache_read_tokens, cache_write_tokens}
  per_agent:
    executor: {input_tokens, output_tokens, total_tokens}
    simulator: {input_tokens, output_tokens, total_tokens}
timing:
  total_wall_clock_ms: int
  handoffs: [{handoff: int, node_id: str, duration_ms: int}, ...]
handoff_patterns:
  total_handoffs: int
  sequence: [str, ...]
  per_agent: {agent: {turn_count, total_duration_ms, avg_turn_duration_ms}}
artifacts:
  workspace: {source_files, test_files, config_files, total_files, total_lines_of_code}
  aidlc_docs: {inception_files, construction_files, total_files}
errors:
  throttle_events, timeout_events, failed_tool_calls, model_error_events, ...
context_size:
  total: {min_tokens, max_tokens, avg_tokens, median_tokens, sample_count}
  per_agent: {executor: {...}, simulator: {...}}
```

### 8.2 정성 점수 (`qualitative-comparison.yaml`)

```yaml
overall_score: float  # 0.0 to 1.0
phases:
  - phase: inception
    avg_intent: float
    avg_design: float
    avg_completeness: float
    avg_overall: float
    documents:
      - path: inception/requirements.md
        intent_similarity: float
        design_similarity: float
        completeness: float
        overall: float
        notes: str
```

### 8.3 골든 기준선 (`golden.yaml`)

승격된 실행에서 약 30개 핵심 지표의 평탄한 수치 스냅샷. 회귀 비교 대상으로 사용됩니다. 필드는 실행 비용, 산출물, 테스트 결과, 코드 품질, 정성 점수를 아우릅니다.

---

## 9. 도구 통합

### 9.1 Strands SDK (다중 에이전트)

실행 패키지는 [Strands Agents SDK](https://github.com/strands-agents/sdk-python)를 사용합니다.
- `Agent` — 시스템 프롬프트와 도구 집합으로 Bedrock 모델 래핑
- `Swarm` — 설정 가능한 한도(최대 핸드오프, 최대 반복, 실행 타임아웃, 노드 타임아웃)로 에이전트 간 핸드오프 조율
- `@tool` 데코레이터 — Python 함수를 에이전트용 호출 가능 도구로 등록
- `BedrockModel` — 설정 가능한 재시도 정책이 있는 Bedrock 모델 제공자
- 훅 시스템 — 진행 추적용 `BeforeNodeCallEvent` / `AfterNodeCallEvent`

### 9.2 Amazon Bedrock

모든 LLM 호출은 boto3를 통해 Amazon Bedrock으로 갑니다. 설정:
- 읽기 타임아웃: 실행 에이전트 900초(15분), 정성 채점기 300초(5분)
- 연결 타임아웃: 30초
- 재시도 정책: 적응 모드로 최대 10회
- 모델: 역할별 설정(executor, simulator, scorer)

### 9.3 정적 분석 도구

| 도구 | 목적 | 출력 형식 | 우아한 저하 |
|------|---------|--------------|---------------------|
| ruff | Python 린트 | JSON | PATH에 없으면 건너뜀 |
| bandit | Python 보안 | JSON | PATH에 없으면 건너뜀 |
| semgrep | 다언어 보안 | JSON | PATH에 없으면 건너뜀 |
| eslint | JS/TS 린트 | JSON | npx로 폴백 |
| npm audit | JS 의존성 보안 | JSON | package-lock.json 필요 |
| PMD CPD | 코드 중복 | XML | 설정 경로 또는 PATH 스캔 |

---

## 10. 보안 모델

### 10.1 파일 샌드박싱

AI 에이전트가 수행하는 모든 파일 작업은 실행 폴더로 샌드박스됩니다.
- `_resolve_safe(run_folder, relative_path)`가 경로를 해석하고 실행 폴더 경계 안에 있는지 검증
- 경로 순회 시도(예: `../../etc/passwd`)는 `ValueError`로 거부
- 적용 대상: `read_file`, `write_file`, `list_files`, `run_command`

### 10.2 명령 샌드박싱

`run_command` 도구는 제한된 셸 환경을 제공합니다.
- `PATH`, `HOME`, `LANG`, `TERM`만 설정(`UV_CACHE_DIR` 등 도구별 변수 추가)
- `HOME`은 실행 폴더로 설정해 호스트 사용자 설정 읽기 방지
- 명령은 설정 가능한 타임아웃(기본 120초)
- 출력은 50K자에서 잘림

### 10.3 서버 격리(계약 테스트)

계약 테스트 서버는 자체 venv에서 실행됩니다.
- `ServerProcess._ensure_venv()`가 워크스페이스 프로젝트에 격리 venv 생성
- `uv run`이 디렉터리 트리를 올라가 부모 프로젝트를 해석하는 것을 방지
- 서버는 해당 venv의 Python 바이너리로 기동

---

## 11. 테스트 케이스

테스트 케이스는 `test_cases/`에 있으며 표준 구조를 따릅니다.

```
test_cases/<case-name>/
  ├── vision.md              # Project vision and constraints
  ├── tech-env.md            # Technical environment requirements
  ├── openapi.yaml           # API contract spec with x-test-cases
  ├── golden-aidlc-docs/     # Reference aidlc-docs output (golden baseline)
  │   ├── inception/
  │   │   ├── requirements.md
  │   │   └── ...
  │   └── construction/
  │       ├── code-generation.md
  │       └── ...
  └── golden.yaml            # Promoted baseline metrics
```

기본 테스트 케이스는 `sci-calc`(과학 계산기 API)입니다. 모든 CLI 기본값이 이 테스트 케이스를 가리킵니다.

---

## 12. 확장 지점

### 새 모델 추가

1. `models.executor.model_id`를 Bedrock 모델 ID로 둔 `config/<model-name>.yaml` 생성
2. 배치 러너가 자동으로 발견

### 새 IDE 어댑터 추가

1. `packages/ide-harness/src/ide_harness/adapters/<name>.py` 생성
2. `IDEAdapter` 추상 클래스 구현(세 메서드: `name`, `check_prerequisites`, `run`)
3. `packages/ide-harness/src/ide_harness/registry.py`의 `_ADAPTER_MAP`에 등록

### 새 정적 분석 도구 추가

1. `packages/quantitative/src/quantitative/analyzers.py`에 분석 함수 추가(`run_ruff` 패턴 따름)
2. 필요 시 `models.py`에 finding 모델 정의
3. 프로젝트 유형 감지에 따라 `scanner.py`에서 호출

### 새 테스트 케이스 추가

1. `test_cases/<case-name>/` 아래 디렉터리 생성
2. `vision.md`, `tech-env.md`, 선택적으로 `openapi.yaml` 제공
3. 골든 기준선을 만들기 위해 전체 파이프라인을 한 번 실행
4. `reporting.baseline.promote()`로 `golden.yaml` 생성
5. 실행의 `aidlc-docs/`를 `golden-aidlc-docs/`로 복사

---

## 13. 의존성 스택

| 구성 요소 | 기술 |
|-----------|-----------|
| 언어 | Python 3.13+ |
| 패키지 매니저 | uv(워크스페이스 모드) |
| AI 오케스트레이션 | Strands Agents SDK |
| LLM 제공자 | Amazon Bedrock (boto3) |
| HTTP 클라이언트 | httpx(계약 테스트) |
| ASGI 서버 | uvicorn(계약 테스트) |
| 테스트 프레임워크 | pytest |
| 직렬화 | PyYAML |
| 린트 | ruff |
| 보안 스캔 | bandit, semgrep |
| 중복 탐지 | PMD CPD(외부) |
