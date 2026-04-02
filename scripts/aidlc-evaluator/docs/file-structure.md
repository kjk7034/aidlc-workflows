# AI-DLC 평가 프레임워크 - 파일 구조

```
aidlc-regression/
├── README.md                          # Project overview
├── VISION.md                          # Project vision and goals
├── FAQ.md                             # Frequently asked questions
├── OPERATING_PRINCIPLES.md            # Decision-making guidelines
├── CONTRIBUTING.md                    # Contribution guidelines
├── pyproject.toml                     # Workspace configuration
├── uv.lock                           # Dependency lock file
│
├── aidlc-runner/                      # Execution framework (two-agent AIDLC runner)
│   ├── pyproject.toml
│   ├── config/
│   │   └── default.yaml
│   ├── src/
│   │   └── aidlc_runner/
│   │       ├── cli.py                # CLI entry point
│   │       ├── config.py             # Configuration loading
│   │       ├── runner.py             # Orchestration core
│   │       ├── metrics.py            # NFR metrics collection
│   │       ├── post_run.py           # Post-run test evaluation
│   │       ├── progress.py           # Progress handlers
│   │       ├── agents/               # Executor and simulator agent factories
│   │       └── tools/                # Sandboxed file ops, rule loader, run_command
│   ├── tests/
│   └── planning/                     # Phase plans and backlog
│
├── packages/                          # Evaluation packages (monorepo)
│   ├── qualitative/                   # Semantic evaluation
│   │   ├── pyproject.toml
│   │   ├── src/
│   │   │   └── qualitative/
│   │   │       ├── __init__.py
│   │   │       ├── comparator.py     # Comparison orchestration
│   │   │       ├── document.py       # Document loading and phase mapping
│   │   │       ├── scorer.py         # Scoring protocol + implementations
│   │   │       └── models.py         # Result data models
│   │   └── tests/
│   │
│   ├── quantitative/                  # Code evaluation
│   │   ├── pyproject.toml
│   │   └── src/
│   │       └── quantitative/
│   │           ├── __init__.py
│   │           ├── linting.py        # Ruff/eslint checks
│   │           ├── security.py       # Semgrep/bandit integration
│   │           └── organization.py   # Code duplication, structure
│   │
│   ├── nonfunctional/                 # NFR evaluation
│   │   ├── pyproject.toml
│   │   └── src/
│   │       └── nonfunctional/
│   │           ├── __init__.py
│   │           ├── tokens.py         # Token consumption tracking
│   │           ├── timing.py         # Execution time measurement
│   │           └── consistency.py    # Cross-model consistency
│   │
│   ├── reporting/                     # Report generation
│   │   ├── pyproject.toml
│   │   └── src/
│   │       └── reporting/
│   │           ├── __init__.py
│   │           └── generate.py       # Main report generator
│   │
│   └── shared/                        # Common utilities
│       ├── pyproject.toml
│       └── src/
│           └── shared/
│               └── __init__.py
│
├── test_cases/                        # Golden test cases (AIDLC inputs)
│   ├── instructions.md
│   └── sci-calc/
│       ├── vision.md
│       └── tech-env.md
│
├── runs/                              # Evaluation run outputs
│   └── {timestamp}-{uuid}/
│       ├── run-meta.yaml
│       ├── run-metrics.yaml
│       ├── test-results.yaml
│       ├── vision.md
│       ├── tech-env.md
│       ├── aidlc-docs/               # Generated AIDLC documentation
│       └── workspace/                # Generated application code
│
├── overall_project/                   # Broader project tenets and strategy
│
└── docs/                              # Additional documentation
    └── writing-inputs/                # Guides for writing vision/tech-env docs
```

## Big Rocks → 패키지 매핑

```
1. Golden Test Case        → test_cases/
2. Execution Framework     → aidlc-runner/
3. Semantic Evaluation     → packages/qualitative/
4. Code Evaluation         → packages/quantitative/
5. NFR Evaluation          → packages/nonfunctional/
6. GitHub CI/CD            → .github/workflows/  (planned)
```

## 패키지 의존성

```
aidlc-runner (standalone — runs the AIDLC workflow and produces run folders)

qualitative
├── shared
quantitative
├── shared
nonfunctional
├── shared
reporting
├── shared
├── qualitative  (reads semantic evaluation results)
├── quantitative (reads code evaluation results)
└── nonfunctional (reads NFR results)
```

## 주요 설계 결정

1. **uv 워크스페이스 모노레포:** 의존성 관리와 패키지 간 개발을 단순화
2. **Python 3.13:** 최신 안정 Python과 최신 기능
3. **평가 유형별 패키지 분리:** 관심사 분리, 독립적 진화
4. **aidlc-runner를 실행 엔진으로:** 평가 패키지가 소비하는 run 폴더 생성
5. **골든 테스트 케이스를 버전 관리된 입력으로:** 일관된 평가를 위한 선별된 기준선
6. **공유 유틸리티 패키지:** 모든 평가 패키지에서 재사용하는 공통 코드
7. **보고가 전체 집계:** 포괄 보고서를 만드는 단일 진입점
