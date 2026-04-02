# AIDLC 평가 보고서

> **실행:** `20260218T125810-b84d042dff254a72b4ffec926fe5ea99`
> **생성:** 2026-02-18T13:45:16+00:00

## 판정

| 차원 | 결과 |
|-----------|--------|
| 단위 테스트 | ✅ **192/192** 통과 |
| 계약 테스트 | ✅ **88/88** 통과 |
| 코드 품질 | ❌ 발견 18건 (오류 5건) |
| 정성 점수 | 🟢 **0.89** |

## 실행 개요

| 속성 | 값 |
|----------|-------|
| 상태 | `Status.COMPLETED` |
| 실행기 모델 | `global.anthropic.claude-opus-4-6-v1` |
| 시뮬레이터 모델 | `us.anthropic.claude-sonnet-4-5-20250929-v1:0` |
| 리전 | `us-west-2` |
| 실제 경과 시간 | 24.1분 |
| 핸드오프 | 3회 (executor → simulator → executor) |
| 시작 | 2026-02-18T12:58:13.159285+00:00 |
| 완료 | 2026-02-18T13:22:44.249897+00:00 |

## 토큰 사용량

| 에이전트 | 입력 | 출력 | 합계 |
|-------|------:|-------:|------:|
| Executor | 5.7M | 77K | 5.7M |
| Simulator | 180K | 2K | 182K |
| **합계** | **9.7M** | **140K** | **9.8M** |

## 핸드오프 타임라인

| # | 에이전트 | 소요 시간 |
|--:|-------|----------|
| 1 | executor | 16.3분 |
| 2 | simulator | 1.1분 |
| 3 | executor | 6.7분 |

## 생성된 산출물

| 범주 | 개수 |
|----------|------:|
| 소스 파일 | 17 |
| 테스트 파일 | 18 |
| 설정 파일 | 4 |
| 파일 합계 | 72 |
| 코드 줄 수 | 3,522 |
| AIDLC 문서 (inception) | 8 |
| AIDLC 문서 (construction) | 5 |
| AIDLC 문서 합계 | 15 |

## 단위 테스트

**✅ 192 통과** / 192 전체

**커버리지:** 91.3%

## 계약 테스트 (API 명세)

**✅ 88/88** 엔드포인트 검증됨

### Health ✅ 1/1

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ 헬스 체크 | GET | `/health` | 200 | 14ms |


### Arithmetic ✅ 15/15

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ 양의 정수 덧셈 | POST | `/api/v1/arithmetic/add` | 200 | 4ms |
| ✅ 음수 덧셈 | POST | `/api/v1/arithmetic/add` | 200 | 2ms |
| ✅ 부동소수 덧셈 | POST | `/api/v1/arithmetic/add` | 200 | 2ms |
| ✅ 필드 누락 → 422 | POST | `/api/v1/arithmetic/add` | 422 | 2ms |
| ✅ 뺄셈 | POST | `/api/v1/arithmetic/subtract` | 200 | 2ms |
| ✅ 곱셈 | POST | `/api/v1/arithmetic/multiply` | 200 | 2ms |
| ✅ 0 곱하기 | POST | `/api/v1/arithmetic/multiply` | 200 | 2ms |
| ✅ 나눗셈 | POST | `/api/v1/arithmetic/divide` | 200 | 3ms |
| ✅ 0으로 나눗셈 → 오류 | POST | `/api/v1/arithmetic/divide` | 400 | 2ms |
| ✅ 나머지 | POST | `/api/v1/arithmetic/modulo` | 200 | 2ms |
| ✅ 0으로 나머지 → 오류 | POST | `/api/v1/arithmetic/modulo` | 400 | 2ms |
| ✅ 음수 절댓값 | POST | `/api/v1/arithmetic/abs` | 200 | 2ms |
| ✅ 양수 절댓값 | POST | `/api/v1/arithmetic/abs` | 200 | 1ms |
| ✅ 양수 부호 반전 | POST | `/api/v1/arithmetic/negate` | 200 | 1ms |
| ✅ 음수 부호 반전 | POST | `/api/v1/arithmetic/negate` | 200 | 2ms |


### Powers ✅ 11/11

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ 2^10 | POST | `/api/v1/powers/power` | 200 | 3ms |
| ✅ 5^0 | POST | `/api/v1/powers/power` | 200 | 1ms |
| ✅ sqrt(16) | POST | `/api/v1/powers/sqrt` | 200 | 1ms |
| ✅ sqrt(0) | POST | `/api/v1/powers/sqrt` | 200 | 1ms |
| ✅ sqrt(-1) → 정의역 오류 | POST | `/api/v1/powers/sqrt` | 400 | 2ms |
| ✅ cbrt(27) | POST | `/api/v1/powers/cbrt` | 200 | 2ms |
| ✅ cbrt(-8) | POST | `/api/v1/powers/cbrt` | 200 | 2ms |
| ✅ square(5) | POST | `/api/v1/powers/square` | 200 | 2ms |
| ✅ square(-3) | POST | `/api/v1/powers/square` | 200 | 1ms |
| ✅ 16의 4제곱근 | POST | `/api/v1/powers/nth_root` | 200 | 2ms |
| ✅ nth_root 음수·짝수근 → 정의역 오류 | POST | `/api/v1/powers/nth_root` | 400 | 1ms |


### Trigonometry ✅ 20/20

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ sin(0) | POST | `/api/v1/trigonometry/sin` | 200 | 4ms |
| ✅ sin(90 deg) | POST | `/api/v1/trigonometry/sin` | 200 | 2ms |
| ✅ cos(0) | POST | `/api/v1/trigonometry/cos` | 200 | 2ms |
| ✅ tan(0) | POST | `/api/v1/trigonometry/tan` | 200 | 2ms |
| ✅ asin(0) | POST | `/api/v1/trigonometry/asin` | 200 | 2ms |
| ✅ asin(1) | POST | `/api/v1/trigonometry/asin` | 200 | 1ms |
| ✅ asin(2) → 정의역 오류 | POST | `/api/v1/trigonometry/asin` | 400 | 1ms |
| ✅ acos(1) | POST | `/api/v1/trigonometry/acos` | 200 | 2ms |
| ✅ acos(2) → 정의역 오류 | POST | `/api/v1/trigonometry/acos` | 400 | 2ms |
| ✅ atan(0) | POST | `/api/v1/trigonometry/atan` | 200 | 2ms |
| ✅ atan2(0, 1) | POST | `/api/v1/trigonometry/atan2` | 200 | 2ms |
| ✅ atan2(1, 0) | POST | `/api/v1/trigonometry/atan2` | 200 | 1ms |
| ✅ sinh(0) | POST | `/api/v1/trigonometry/sinh` | 200 | 2ms |
| ✅ cosh(0) | POST | `/api/v1/trigonometry/cosh` | 200 | 2ms |
| ✅ tanh(0) | POST | `/api/v1/trigonometry/tanh` | 200 | 2ms |
| ✅ asinh(0) | POST | `/api/v1/trigonometry/asinh` | 200 | 2ms |
| ✅ acosh(1) | POST | `/api/v1/trigonometry/acosh` | 200 | 1ms |
| ✅ acosh(0.5) → 정의역 오류 | POST | `/api/v1/trigonometry/acosh` | 400 | 1ms |
| ✅ atanh(0) | POST | `/api/v1/trigonometry/atanh` | 200 | 2ms |
| ✅ atanh(1) → 정의역 오류 | POST | `/api/v1/trigonometry/atanh` | 400 | 1ms |


### Logarithmic ✅ 11/11

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ ln(1) | POST | `/api/v1/logarithmic/ln` | 200 | 3ms |
| ✅ ln(e) | POST | `/api/v1/logarithmic/ln` | 200 | 2ms |
| ✅ ln(0) → 정의역 오류 | POST | `/api/v1/logarithmic/ln` | 400 | 2ms |
| ✅ ln(-1) → 정의역 오류 | POST | `/api/v1/logarithmic/ln` | 400 | 1ms |
| ✅ log10(100) | POST | `/api/v1/logarithmic/log10` | 200 | 1ms |
| ✅ log10(1) | POST | `/api/v1/logarithmic/log10` | 200 | 2ms |
| ✅ log2(8) | POST | `/api/v1/logarithmic/log2` | 200 | 2ms |
| ✅ log(8, base=2) | POST | `/api/v1/logarithmic/log` | 200 | 2ms |
| ✅ 밑 1 로그 → 정의역 오류 | POST | `/api/v1/logarithmic/log` | 400 | 2ms |
| ✅ exp(0) | POST | `/api/v1/logarithmic/exp` | 200 | 2ms |
| ✅ exp(1) | POST | `/api/v1/logarithmic/exp` | 200 | 1ms |


### Statistics ✅ 12/12

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ mean | POST | `/api/v1/statistics/mean` | 200 | 4ms |
| ✅ 중앙값 홀수 개수 | POST | `/api/v1/statistics/median` | 200 | 2ms |
| ✅ 중앙값 짝수 개수 | POST | `/api/v1/statistics/median` | 200 | 2ms |
| ✅ mode | POST | `/api/v1/statistics/mode` | 200 | 2ms |
| ✅ stdev | POST | `/api/v1/statistics/stdev` | 200 | 2ms |
| ✅ variance | POST | `/api/v1/statistics/variance` | 200 | 2ms |
| ✅ pstdev | POST | `/api/v1/statistics/pstdev` | 200 | 2ms |
| ✅ pvariance | POST | `/api/v1/statistics/pvariance` | 200 | 2ms |
| ✅ min | POST | `/api/v1/statistics/min` | 200 | 2ms |
| ✅ max | POST | `/api/v1/statistics/max` | 200 | 2ms |
| ✅ sum | POST | `/api/v1/statistics/sum` | 200 | 1ms |
| ✅ count | POST | `/api/v1/statistics/count` | 200 | 1ms |


### Constants ✅ 10/10

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ 모든 상수 조회 | GET | `/api/v1/constants` | 200 | 3ms |
| ✅ pi 조회 | GET | `/api/v1/constants/pi` | 200 | 2ms |
| ✅ e 조회 | GET | `/api/v1/constants/e` | 200 | 1ms |
| ✅ tau 조회 | GET | `/api/v1/constants/tau` | 200 | 2ms |
| ✅ golden_ratio 조회 | GET | `/api/v1/constants/golden_ratio` | 200 | 3ms |
| ✅ sqrt2 조회 | GET | `/api/v1/constants/sqrt2` | 200 | 2ms |
| ✅ ln2 조회 | GET | `/api/v1/constants/ln2` | 200 | 2ms |
| ✅ ln10 조회 | GET | `/api/v1/constants/ln10` | 200 | 2ms |
| ✅ inf 조회 | GET | `/api/v1/constants/inf` | 200 | 1ms |
| ✅ nan 조회 | GET | `/api/v1/constants/nan` | 200 | 2ms |


### Conversions ✅ 7/7

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ 180도를 라디안으로 | POST | `/api/v1/conversions/angle` | 200 | 3ms |
| ✅ 끓는점 C→F | POST | `/api/v1/conversions/temperature` | 200 | 2ms |
| ✅ 어는점 C→K | POST | `/api/v1/conversions/temperature` | 200 | 2ms |
| ✅ 1미터를 피트로 | POST | `/api/v1/conversions/length` | 200 | 2ms |
| ✅ 1마일을 킬로미터로 | POST | `/api/v1/conversions/length` | 200 | 2ms |
| ✅ 1kg를 파운드로 | POST | `/api/v1/conversions/weight` | 200 | 1ms |
| ✅ 1스톤을 킬로그램으로 | POST | `/api/v1/conversions/weight` | 200 | 1ms |


### Nonexistent ✅ 1/1

| 테스트 | 메서드 | 경로 | 상태 | 지연 |
|------|--------|------|:------:|--------:|
| ✅ 알 수 없는 엔드포인트 → 404 | GET | `/api/v1/nonexistent` | 404 | 1ms |


## 코드 품질

**❌ 발견 18건** (오류 5건, 경고 13건)

**Linter:** ruff 0.15.1

| 파일 | 줄 | 코드 | 메시지 | 심각도 |
|------|-----:|------|---------|----------|
| `app.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `math_engine.py` | 7 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `math_engine.py` | 12 | `F401` | `typing.Any`가 import되었으나 미사용 | 🟡 경고 |
| `arithmetic.py` | 65 | `E501` | 줄이 너무 김 (101 > 100) | 🔴 오류 |
| `arithmetic.py` | 78 | `E501` | 줄이 너무 김 (107 > 100) | 🔴 오류 |
| `logarithmic.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `logarithmic.py` | 72 | `E501` | 줄이 너무 김 (108 > 100) | 🔴 오류 |
| `powers.py` | 74 | `E501` | 줄이 너무 김 (103 > 100) | 🔴 오류 |
| `trigonometry.py` | 75 | `E501` | 줄이 너무 김 (109 > 100) | 🔴 오류 |
| `conftest.py` | 8 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `test_arithmetic.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `test_arithmetic.py` | 9 | `F401` | `sci_calc.engine.math_engine.MathOverflowError`가 import되었으나 미사용 | 🟡 경고 |
| `test_constants.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `test_conversions.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `test_logarithmic.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `test_powers.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `test_statistics.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |
| `test_trigonometry.py` | 3 | `I001` | import 블록이 정렬·포맷되지 않음 | 🟡 경고 |

*보안 스캐너(bandit)를 사용할 수 없었습니다.*

## 정성 평가(의미 유사도)

**전체 점수: 🟢 0.8910**

### Inception 단계

| 차원 | 점수 |
|-----------|------:|
| 의도 | 0.90 |
| 설계 | 0.89 |
| 완전성 | 0.88 |
| **전체** | **0.89** |

| 문서 | 의도 | 설계 | 완전성 | 전체 |
|----------|-------:|-------:|---------:|--------:|
| `component-dependency.md` | 1.00 | 0.95 | 0.90 | 0.96 |
| `component-methods.md` | 1.00 | 0.95 | 0.85 | 0.95 |
| `components.md` | 1.00 | 1.00 | 1.00 | 1.00 |
| `services.md` | 0.95 | 0.90 | 0.85 | 0.91 |
| `application-design-plan.md` | 1.00 | 1.00 | 1.00 | 1.00 |
| `execution-plan.md` | 1.00 | 0.95 | 0.95 | 0.97 |
| `requirement-verification-questions.md` | 0.30 | 0.40 | 0.50 | 0.38 |
| `requirements.md` | 0.95 | 0.95 | 0.95 | 0.95 |

<details><summary><code>component-dependency.md</code> — 0.96</summary>

두 문서 모두 동일한 의도를 담습니다: 관심사가 분리된 FastAPI 수학 서비스의 컴포넌트 의존성을 문서화합니다. 설계는 거의 동일하며 동일한 아키텍처(routes, models, engine), 동일한 의존 패턴, 동일한 핵심 제약(엔진은 프레임워크 의존성 제로, 라우트는 얇은 어댑터)을 갖습니다. 사소한 차이: 후보는 모듈 표기 대신 파일 경로(.py 확장자)를 쓰고, 의존성 흐름도 대신 데이터 흐름도를 포함하며, 외부 의존성 표와 예외 핸들러 등록 세부는 생략합니다. 후보는 동기 호출 및 async/DB/큐 없음에 대한 설명을 추가합니다. 전반적으로 사소한 표현 차이만 있는 높은 일치입니다.

</details>

<details><summary><code>component-methods.md</code> — 0.95</summary>

의도는 동일합니다: 동일한 수학 연산, 요청/응답 모델, API 구조를 정의합니다. 설계는 거의 동일하며 동일한 계층 아키텍처(routes, models, engine), 동일한 함수 시그니처, 동일한 예외 처리 방식을 갖습니다. 사소한 차이: 후보는 모델 이름이 약간 다르고(BinaryOperationRequest vs TwoOperandRequest 등) 상세 라우트 경로/메서드 표는 생략합니다. HTTP 메서드와 경로가 있는 상세 라우팅 표가 없고 create_app() 함수나 사용자 정의 예외 클래스를 별도 엔티티로 명시하지 않으나 기능은 함축되어 있습니다. 조직적 차이만 있는 매우 강한 일치입니다.

</details>

<details><summary><code>components.md</code> — 1.00</summary>

두 문서 모두 동일한 컴포넌트 아키텍처를 설명하며 동일한 네 계층(앱 진입점, routes, models, engine) 구조를 갖습니다. 라우트 모듈 일곱 개가 모두 있고 목적이 일치합니다. models 계층은 요청과 응답을 동일하게 구분합니다. engine 계층 책임은 순수 함수 설계, 표준 라이브러리만 의존, 정의역별 예외를 포함해 동등합니다. 스타일 차이(포맷, 연산 나열의 상세도)는 있으나 아키텍처 의도, 설계 결정, 주제 범위는 기능적으로 동일합니다.

</details>

<details><summary><code>services.md</code> — 0.91</summary>

두 문서 모두 별도 서비스 계층 없이 라우트에서 엔진으로 직접 위임하는 얇은 서비스 아키텍처를 설명합니다. 의도는 거의 동일합니다. 설계는 동일한 오류 처리 흐름과 패턴으로 매우 유사하나, 후보는 404 처리를 추가하고 CORS 미들웨어 세부는 생략합니다. 후보는 CORS 설정을 언급하지 않지만 참조에 없는 헬스 체크 세부를 추가해 약간 덜 완전할 수 있습니다.

</details>

<details><summary><code>application-design-plan.md</code> — 1.00</summary>

두 문서 모두 동일한 의도를 담습니다: FastAPI와 Pydantic v2를 쓰는 Scientific Calculator API의 3계층(Routes, Models, Engine) 아키텍처. tech-env가 완전히 명시되어 설계 질문이 불필요하다고 둘 다 명시합니다. 동일한 산출물(components.md 등)과 검증 단계를 포함합니다. 후보가 맥락 설명을 약간 더 제공하나 참조와 완전히 정렬됩니다.

</details>

<details><summary><code>execution-plan.md</code> — 0.97</summary>

두 문서 모두 동일한 의도와 목표를 가지며 동일한 요구사항과 실행 전략을 담습니다. 설계 접근은 컴포넌트 구조와 생략/실행 결정이 동일해 거의 같습니다. 사소한 차이: 참조는 성공 기준이 더 상세함(1 ULP 정밀도, HTTP 상태 코드, 구조화 래퍼)이고 워크플로 시각화 형식이 약간 다릅니다. 후보는 더 간결하나 주요 주제는 모두 다룹니다. 전반적으로 매우 높은 일치입니다.

</details>

<details><summary><code>requirement-verification-questions.md</code> — 0.38</summary>

두 문서 모두 요구사항 확정 전 모호함을 줄이려 하지만 거의 다른 관심사를 다룹니다. 참조는 부동소수점 처리, 배열 한계, CORS, NaN 직렬화, 정밀도, API 문서에 초점을 둡니다. 후보는 오류 래퍼 구조, mode 반환 형식, 오버플로 처리, 알 수 없는 단위, 커버리지 강제, NaN 입력 처리에 초점을 둡니다. 질문 1(부동소수점/오버플로)과 4(NaN 처리)만 주제가 겹치나 구체적 질문은 다릅니다. 둘 다 질문 6개와 유사한 구조이나 실질 내용은 크게 달라 각 inception 실행에서 다른 불확실 영역이 식별되었음을 시사합니다.

</details>

<details><summary><code>requirements.md</code> — 0.95</summary>

두 문서 모두 과학 계산기 API에 대해 거의 동일한 의도, 요구사항, 기술 접근을 담습니다. 사소한 차이: 참조는 FR-011(NaN/Infinity 문자열 직렬화), FR-013(명시적 CORS)이 있으나 후보는 생략합니다. 후보는 FR-10.3/10.4(오버플로/NaN 입력 처리)를 더 명시합니다. 후보는 FR-1.1 형식, 참조는 FR-001 형식이나 내용은 동등합니다. 연산, 오류 코드, 기술 스택, 제약은 동일합니다. 후보는 CORS와 특수 NaN/Infinity 직렬화 형식을 명시하지 않아 사소하나 눈에 띄는 공백이 있습니다.

</details>

### Construction 단계

| 차원 | 점수 |
|-----------|------:|
| 의도 | 0.93 |
| 설계 | 0.85 |
| 완전성 | 0.90 |
| **전체** | **0.89** |

| 문서 | 의도 | 설계 | 완전성 | 전체 |
|----------|-------:|-------:|---------:|--------:|
| `build-and-test-summary.md` | 0.95 | 0.90 | 0.95 | 0.93 |
| `build-instructions.md` | 0.85 | 0.75 | 0.80 | 0.80 |
| `integration-test-instructions.md` | 0.85 | 0.75 | 0.90 | 0.82 |
| `unit-test-instructions.md` | 1.00 | 0.90 | 0.95 | 0.95 |
| `sci-calc-code-generation-plan.md` | 1.00 | 0.95 | 0.90 | 0.96 |

<details><summary><code>build-and-test-summary.md</code> — 0.93</summary>

두 문서 모두 동일한 핵심 의도를 담습니다: sci-calc 프로젝트의 빌드·테스트 결과를 요약하고 모든 테스트 통과 및 배포 준비 완료를 나타냅니다. 설계 접근은 거의 동일합니다(FastAPI, hatchling, pytest, 동일 모듈 구조). 사소한 차이: 후보는 테스트 192개, 참조 187개(테스트 정제 가능), 후보는 NaN 검증기 수정 등 버그 수정 문서가 상세하며 Windows asyncio 문제용 SyncTestClient 우회를 사용합니다. 후보는 모듈별 테스트 분해가 더 세분화됩니다. 둘 다 품질 게이트를 충족하고 배포 준비를 선언합니다. 커버리지 보고는 다름(참조: 95.20% 측정, 후보: CI로 연기). 파일 수는 약간 다르나 핵심 구조는 동등합니다. 구현 변형만 있는 매우 유사한 문서입니다.

</details>

<details><summary><code>build-instructions.md</code> — 0.80</summary>

두 문서 모두 Python 3.13+와 uv로 sci-calc 프로젝트 빌드 지침을 제공한다는 동일한 핵심 의도를 공유합니다. 후보는 빌드 백엔드(hatchling), 명시적 의존성 버전, 패키지 빌드 단계, 문제 해결 절 등 참조에 없는 추가 세부를 포함합니다. 참조는 더 단순한 검증과 개발 워크플로에 초점을 둡니다. 설계는 유사하나(uv 기반, FastAPI/uvicorn) 후보가 빌드 도구 세부를 더합니다. 후보는 참조의 주요 주제(사전 요구사항, 설치, 검증, 서버 실행, 린트)를 모두 다루고 추가 항목도 있으나 헬스 체크 curl 명령 등 일부 참조 요소는 없습니다.

</details>

<details><summary><code>integration-test-instructions.md</code> — 0.82</summary>

두 문서 모두 동일한 FastAPI 계산기 애플리케이션의 통합 테스트를 설명하며 유사한 목표(HTTP 요청/응답 주기, 검증, 오류 처리)를 갖습니다. 후보는 7개 도메인에 63개 테스트로 더 세분화되고 참조는 5개 일반 시나리오입니다. 설계는 유사하나(httpx.AsyncClient, ASGI 전송, 동일 위치 테스트) 후보가 구체적 경로와 테스트 수를 추가합니다. 후보는 참조 시나리오 전부와 상수·변환·헬스 등 추가 도메인을 포함합니다. 실행 명령은 약간 다르나 둘 다 pytest를 사용합니다. 후보가 세부를 강화한 강한 일치입니다.

</details>

<details><summary><code>unit-test-instructions.md</code> — 0.95</summary>

두 문서 모두 동일한 의도를 공유합니다: pytest와 ≥90% 커버리지 목표로 sci_calc 단위 테스트 실행 지침을 제공합니다. 설계는 pytest/커버리지 명령으로 매우 유사하나 후보는 Windows asyncio 우회와 테스트 아키텍처 분해를 더 상세히 합니다. 후보는 테스트 192개, 참조 187개이며 대체 테스트 클라이언트 문서를 추가합니다. 참조는 모듈별 커버리지 표(95.20% 달성)가 있고 후보는 모듈별 테스트 개수 분해에 초점을 둡니다. 둘 다 construction 단계 테스트 지침으로 완전하며 구조 차이만 사소합니다.

</details>

<details><summary><code>sci-calc-code-generation-plan.md</code> — 0.96</summary>

두 문서 모두 동일한 과학 계산기 API를 대상으로 하며 목표와 요구사항이 동일합니다. 설계는 거의 동일하며 동일한 계층 아키텍처(engine/models/routes), FastAPI, 컴포넌트 분해를 갖습니다. 후보는 연산 유형별 엔진 하위 단계, 명시적 오류 처리 등 구현 세부가 더 세분화되고 참조는 더 넓은 단계를 사용합니다. 후보는 일부 파일을 통합(conftest를 1단계에)하고 테스트 세부를 더 명시합니다. 단계 구성은 약간 다르나 참조 주제를 모두 다루며 구현 특정성을 추가합니다.

</details>

## 기준선 비교

> 골든 기준선 대비: `20260218T125810-b84d042dff254a72b4ffec926fe5ea99`
> 승격: 2026-02-18T13:45:06+00:00

| | 개수 |
|---|------:|
| 🟢 개선 | 0 |
| 🔴 퇴보 | 0 |
| ⚪ 변경 없음 | 20 |

### 단위 테스트

| 지표 | 골든 | 현재 | 델타 | 변경 |
|--------|-------:|--------:|------:|--------|
| 테스트 통과 | 192 | 192 | ⚪ 0 | 변경 없음 |
| 테스트 실패 | 0 | 0 | ⚪ 0 | 변경 없음 |
| 테스트 합계 | 192 | 192 | ⚪ 0 | 변경 없음 |
| 커버리지 % | 91 | 91 | ⚪ 0 | 변경 없음 |

### 계약 테스트

| 지표 | 골든 | 현재 | 델타 | 변경 |
|--------|-------:|--------:|------:|--------|
| 계약 통과 | 88 | 88 | ⚪ 0 | 변경 없음 |
| 계약 실패 | 0 | 0 | ⚪ 0 | 변경 없음 |
| 계약 합계 | 88 | 88 | ⚪ 0 | 변경 없음 |

### 코드 품질

| 지표 | 골든 | 현재 | 델타 | 변경 |
|--------|-------:|--------:|------:|--------|
| 린트 오류 | 5 | 5 | ⚪ 0 | 변경 없음 |
| 린트 경고 | 13 | 13 | ⚪ 0 | 변경 없음 |
| 린트 합계 | 18 | 18 | ⚪ 0 | 변경 없음 |

### 정성

| 지표 | 골든 | 현재 | 델타 | 변경 |
|--------|-------:|--------:|------:|--------|
| 정성 점수 | 0.8910 | 0.8910 | ⚪ 0 | 변경 없음 |
| Inception 점수 | 0.8900 | 0.8900 | ⚪ 0 | 변경 없음 |
| Construction 점수 | 0.8920 | 0.8920 | ⚪ 0 | 변경 없음 |

### 산출물

| 지표 | 골든 | 현재 | 델타 | 변경 |
|--------|-------:|--------:|------:|--------|
| 소스 파일 | 17 | 17 | ⚪ 0 | 변경 없음 |
| 테스트 파일 | 18 | 18 | ⚪ 0 | 변경 없음 |
| 코드 줄 수 | 3,522 | 3,522 | ⚪ 0 | 변경 없음 |
| 문서 파일 | 15 | 15 | ⚪ 0 | 변경 없음 |

### 실행

| 지표 | 골든 | 현재 | 델타 | 변경 |
|--------|-------:|--------:|------:|--------|
| 토큰 합계 | 9,835,935 | 9,835,935 | ⚪ 0 | 변경 없음 |
| 실제 경과(ms) | 1,445,460 | 1,445,460 | ⚪ 0 | 변경 없음 |
| 핸드오프 | 3 | 3 | ⚪ 0 | 변경 없음 |

---
*aidlc-reporting v0.1.0으로 생성된 보고서*
