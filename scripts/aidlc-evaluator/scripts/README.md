# AIDLC 평가 스크립트

이 디렉터리에는 AIDLC 평가 프레임워크용 전용 실행 스크립트가 있습니다.

## 개요

모든 실행 스크립트는 구성을 위해 이 `scripts/` 디렉터리로 모았습니다. 메인 진입점은 저장소 루트의 `run.py`이며, 평가 모드에 따라 이 전용 스크립트로 분기합니다.

## 스크립트

### 핵심 평가 스크립트

- **run_evaluation.py** - 전체 평가 파이프라인(AIDLC 워크플로 실행 + 출력 채점)
  - 여섯 단계 조율: 실행, 실행 후 테스트, 정량 분석, 계약 테스트, 정성 평가, 보고
  - `--test` 플래그로 테스트 모드 실행 가능

- **run_cli_evaluation.py** - CLI 기반 평가
  - CLI AI 어시스턴트(kiro-cli, claude-code 등)로 평가 실행
  - `packages/cli-harness`의 어댑터 사용

- **run_ide_evaluation.py** - IDE 기반 평가
  - IDE AI 어시스턴트(cursor, cline, kiro)로 평가 실행
  - `packages/ide-harness`의 어댑터 사용

### 배치 처리 스크립트

- **run_batch_evaluation.py** - 배치 평가 러너
  - 여러 Bedrock 모델에 대해 순차적으로 AIDLC 평가 실행
  - `config/` 디렉터리의 모델 설정 읽기
  - 모델마다 `run_evaluation.py`에 위임

- **run_comparison_report.py** - 모델 간 비교
  - 배치 실행 결과 집계
  - Markdown 및 YAML 형식의 비교 행렬 생성
  - 골든 기준선과 비교

- **run_extension_test.py** - 확장 훅 테스트
  - 확장 옵트인 설정이 다른 AIDLC 평가 테스트
  - “모두 예”와 “모두 아니오” 옵트인으로 여러 번 평가 실행
  - 확장 선택 영향을 보여 주는 비교 보고서 생성
  - 확장 훅 기능 브랜치 사용(`feat/extension_hook_question_split`)

### 트렌드 보고

- **run_trend_report.py** - 릴리스 간 트렌드 보고서 생성
  - GitHub 릴리스와 Actions 아티팩트에서 평가 번들 가져오기
  - 릴리스 간 지표를 비교하는 HTML, Markdown, YAML 트렌드 보고서 생성
  - `packages/trend-reports` 패키지 사용
  - 요약 카드: 정성 점수, 계약 테스트, 단위 테스트 통과율(%), 린트 발견, 실행 시간, 총 토큰
  - 실행 시간과 총 토큰은 “낮을수록 좋음” 지표(낮은 값이 바람직하므로 녹색 표시)

## 사용법

### 마스터 진입점 사용(권장)

평가 실행은 저장소 루트의 마스터 `run.py` 스크립트를 쓰는 것을 권장합니다.

```bash
# Full pipeline evaluation
python run.py full --vision test_cases/sci-calc/vision.md

# CLI evaluation
python run.py cli --cli kiro-cli --scenario sci-calc

# IDE evaluation
python run.py ide --ide cursor --scenario sci-calc

# Batch evaluation across models
python run.py batch --models all --scenario sci-calc

# Generate comparison report
python run.py compare --scenario sci-calc

# Test extension hooks (all yes vs all no)
python run.py ext-test --scenario sci-calc

# Generate trend report across releases
python run.py trend --baseline test_cases/sci-calc/golden.yaml

# Run tests
python run.py test
```

### 스크립트 직접 호출

필요하면 스크립트를 직접 호출할 수도 있습니다.

```bash
# Full evaluation
python scripts/run_evaluation.py --vision test_cases/sci-calc/vision.md

# CLI evaluation
python scripts/run_cli_evaluation.py --cli kiro-cli --scenario sci-calc

# Batch evaluation
python scripts/run_batch_evaluation.py --models all --scenario sci-calc

# Extension hook testing
python scripts/run_extension_test.py --scenario sci-calc

# Trend report
python scripts/run_trend_report.py --baseline test_cases/sci-calc/golden.yaml
```

## 경로 해석

모든 스크립트는 저장소 루트를 기준으로 경로를 올바르게 해석하므로, 다음 어디서 호출해도 동작합니다.
- 마스터 `run.py` 스패처를 통한 호출
- 저장소 루트에서 직접
- `scripts/` 디렉터리에서 직접

## 아키텍처 참고

- **REPO_ROOT**: 모든 스크립트가 `Path(__file__).resolve().parent.parent`로 저장소 루트를 찾습니다
- **출력**: 실행 출력은 기본적으로 `runs/<scenario>/`에 기록됩니다
- **설정**: 설정 파일은 저장소 루트의 `config/`에서 읽습니다
- **테스트 케이스**: 시나리오는 저장소 루트의 `test_cases/`에 있습니다
