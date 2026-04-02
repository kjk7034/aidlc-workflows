# 테스트 케이스: all-stages

## 목적

적응형 AI-DLC 워크플로가 **모든 조건부 단계**를 실행하도록 하고, 아무 단계도 건너뛰지 않게 합니다. sci-calc 테스트 케이스는 의도적으로 단순합니다(단일 유닛, 사용자 스토리 없음, NFR 설계 없음, 인프라 설계 없음). 이 테스트 케이스는 모델이 합리적으로 무엇이든 건너뛰기 어렵게 설계되었습니다.

## 트리거되는 단계

| 단계 | 트리거 방식 |
|---|---|
| Workspace Detection | 항상 실행 |
| Reverse Engineering | **트리거되지 않음** — brownfield 필요(아래 참고) |
| Requirements Analysis | 항상 실행 |
| User Stories | 서로 다른 워크플로를 가진 3명의 사용자 페르소나 |
| Workflow Planning | 항상 실행 |
| Application Design | 새 컴포넌트, 서비스, 비즈니스 규칙 |
| Units Generation | 분해가 필요한 2개 모듈 |
| Functional Design(유닛별) | 복잡한 비즈니스 규칙(대출 한도, 수수료, 홀드) |
| NFR Requirements(유닛별) | 명시적 성능 SLA, 보안, 확장성 |
| NFR Design(유닛별) | 복원력, 캐싱, 속도 제한 패턴 필요 |
| Infrastructure Design(유닛별) | AWS 서비스 명시(Lambda, DynamoDB, Cognito, SQS) |
| Code Generation(유닛별) | 항상 실행 |
| Build and Test | 항상 실행 |

## 파일

| 파일 | 설명 |
|---|---|
| `vision.md` | 프로젝트 비전 — BookShelf 커뮤니티 도서관 API |
| `tech-env.md` | 기술 환경 — Python/FastAPI, AWS 서버리스 |
| `openapi.yaml` | 계약 테스트용 `x-test-cases`가 있는 API 계약 |
| `golden-aidlc-docs/` | 골든 참조 문서(첫 성공 실행 후 생성) |
| `golden.yaml` | 골든 실행에서의 기준 메트릭(승격 후 생성) |

---

## run_evaluation.py로 실행

`run_evaluation.py`는 전체 여섯 단계 평가 파이프라인을 조율합니다. 기본값이 모두 `test_cases/sci-calc/`를 가리키므로 이 테스트 케이스는 입력 경로를 재정의해야 합니다.

### 첫 실행(아직 골든 기준선 없음)

첫 실행에는 비교할 `golden-aidlc-docs/`가 없습니다. 실행만 하고 정성 비교는 건너뛰며, 출력을 수동으로 검토합니다.

```bash
# Pick a model config from config/ (e.g., opus.yaml, sonnet-4-5.yaml)
python run_evaluation.py \
  --config  config/opus.yaml \
  --vision  test_cases/all-stages/vision.md \
  --tech-env test_cases/all-stages/tech-env.md \
  --openapi test_cases/all-stages/openapi.yaml \
  --golden  test_cases/all-stages/golden-aidlc-docs
```

`--golden` 경로가 아직 없으므로 정성 단계는 실패합니다. 예상된 동작입니다. 실행 자체는 다음을 생성합니다.

```
runs/<timestamp>/
  aidlc-docs/          # Generated AIDLC documentation
  workspace/           # Generated application code
  test-results.yaml    # Post-run unit test results
  run-meta.yaml        # Execution metadata
```

### 실행을 골든으로 승격

성공한 실행을 검토하고 출력이 올바른지 확인한 뒤:

```bash
# Copy the aidlc-docs from the run into the test case as the golden reference
cp -r runs/<timestamp>/aidlc-docs test_cases/all-stages/golden-aidlc-docs
```

회귀 비교용 `golden.yaml` 기준선을 만들려면 실행의 주요 메트릭을 캡처합니다(형식은 `test_cases/sci-calc/golden.yaml` 참고).

### 이후 실행(골든 기준선 있음)

`golden-aidlc-docs/`가 있으면 전체 파이프라인이 끝까지 동작합니다.

```bash
python run_evaluation.py \
  --config   config/opus.yaml \
  --vision   test_cases/all-stages/vision.md \
  --tech-env test_cases/all-stages/tech-env.md \
  --openapi  test_cases/all-stages/openapi.yaml \
  --golden   test_cases/all-stages/golden-aidlc-docs
```

`--baseline` 플래그는 `--golden`과 같은 디렉터리의 `golden.yaml`에서 자동 탐색되므로, `test_cases/all-stages/golden.yaml`이 있으면 회귀 비교에 자동 사용됩니다.

### 기존 실행만 재채점(실행 생략)

AIDLC 워크플로를 다시 실행하지 않고 이전 실행만 다시 채점하려면:

```bash
python run_evaluation.py \
  --evaluate-only runs/<timestamp>/aidlc-docs \
  --golden  test_cases/all-stages/golden-aidlc-docs \
  --openapi test_cases/all-stages/openapi.yaml
```

### 주요 CLI 플래그 참고

| 플래그 | 용도 | 기본값 |
|---|---|---|
| `--config` | 모델 설정 YAML(AWS 자격, executor 모델, swarm 설정) | `config/default.yaml` |
| `--vision` | 비전 문서 | `test_cases/sci-calc/vision.md` |
| `--tech-env` | 기술 환경 문서 | `test_cases/sci-calc/tech-env.md` |
| `--openapi` | `x-test-cases`가 있는 API 계약 스펙 | `test_cases/sci-calc/openapi.yaml` |
| `--golden` | 정성 채점용 골든 참조 aidlc-docs | `test_cases/sci-calc/golden-aidlc-docs` |
| `--baseline` | 회귀 비교용 `golden.yaml` | `--golden` 상위에서 자동 탐색 |
| `--executor-model` | 설정의 executor 모델 ID 재정의 | 설정 YAML에서 |
| `--scorer-model` | 정성 채점용 Bedrock 모델 | 설정 YAML에서 |
| `--rules-ref` | AIDLC 규칙용 Git ref(브랜치/태그/커밋) | 설정 YAML에서 |
| `--output-dir` | 실행 출력 디렉터리 재정의 | `runs/<timestamp>` |
| `--report-format` | `markdown`, `html`, 또는 `both` | `both` |

---

## run_batch_evaluation.py로 실행

배치 러너는 동일 테스트 케이스에 대해 여러 모델을 평가합니다. 모델마다 `run_evaluation.py`를 한 번씩 호출하고, 결과를 `runs/` 아래 모델별 하위 디렉터리에 모읍니다.

```bash
# All models defined in config/*.yaml
python run_batch_evaluation.py \
  --vision   test_cases/all-stages/vision.md \
  --tech-env test_cases/all-stages/tech-env.md \
  --openapi  test_cases/all-stages/openapi.yaml \
  --golden   test_cases/all-stages/golden-aidlc-docs \
  --models all

# Specific models only
python run_batch_evaluation.py \
  --vision   test_cases/all-stages/vision.md \
  --tech-env test_cases/all-stages/tech-env.md \
  --openapi  test_cases/all-stages/openapi.yaml \
  --golden   test_cases/all-stages/golden-aidlc-docs \
  --models opus sonnet-4-5

# List available model configs
python run_batch_evaluation.py --list
```

---

## 평가 파이프라인 단계

참고로 이 테스트 케이스에서 각 단계의 역할은 다음과 같습니다.

| # | 단계 | 하는 일 |
|---|---|---|
| 1 | **실행** | 두 에이전트 AIDLC 워크플로(executor + simulator)를 실행해 `aidlc-docs/`와 `workspace/` 코드 생성 |
| 2 | **실행 후 테스트** | `workspace/`에 의존성 설치 후 `uv run pytest` 실행(실행 단계에 내장) |
| 3 | **정량** | 생성 코드에 ruff로 린트, bandit으로 보안 스캔 |
| 4 | **계약 테스트** | `workspace/`에서 FastAPI 앱 기동, `openapi.yaml`의 `x-test-cases`로 요청 전송, 응답 검증 |
| 5 | **정성** | Bedrock으로 생성 `aidlc-docs/`와 `golden-aidlc-docs/`의 의미 유사도 비교 채점 |
| 6 | **보고** | 모든 메트릭이 담긴 통합 Markdown + HTML 보고서 생성 |

---

## Reverse Engineering 참고

Reverse Engineering 단계는 brownfield 프로젝트(기존 코드 감지)에서만 실행됩니다. 현재 러너는 초기 프롬프트에 `"this is a greenfield project"`를 하드코딩합니다(`packages/execution/src/aidlc_runner/runner.py` 224행). Reverse Engineering을 테스트하려면:

1. 이 테스트 케이스 안에 `existing-code/` 디렉터리에 기존 코드를 둡니다
2. 실행 전에 `runner.py`가 `existing-code/*`를 `workspace/`로 복사하도록 수정합니다
3. 초기 프롬프트에서 greenfield 가정을 제거합니다

이 테스트 케이스가 다루지 않는 AIDLC 단계는 이것뿐입니다.

---

## 설계 근거

도메인(도서관 대출)은 누구나 이해할 수 있을 만큼 단순하지만, 모든 조건부 단계를 강제할 만큼 구조적인 복잡도를 포함합니다.

- **여러 사용자 페르소나**(사서, 회원, 관리자)와 서로 다른 워크플로 → 사용자 스토리 유도
- **두 개의 논리 모듈**(Catalog, Lending) → 애플리케이션 설계와 유닛 생성 유도
- **비즈니스 규칙**(대출 한도, 연체료, 홀드 대기열, 대여 연장) → 기능 설계 유도
- **명시적 성능 목표, 보안 요구사항, 확장성 제약** → NFR 요구사항 및 NFR 설계 유도
- **AWS 클라우드 배포 요구사항** → 인프라 설계 유도
