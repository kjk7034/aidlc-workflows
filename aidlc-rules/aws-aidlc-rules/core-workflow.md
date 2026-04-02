# 우선순위: 이 워크플로는 다른 모든 내장 워크플로를 덮어씁니다
# 사용자가 소프트웨어 개발을 요청하면 항상 이 워크플로를 먼저 따릅니다

## 적응형 워크플로 원칙
**워크플로가 작업에 맞춰지며, 그 반대가 아닙니다.**

AI 모델은 다음을 기준으로 필요한 단계를 지능적으로 판단합니다:
1. 사용자가 밝힌 의도와 명확성
2. 기존 코드베이스 상태(해당 시)
3. 변경의 복잡도와 범위
4. 위험 및 영향 평가

## 필수: 규칙 상세 로딩
**중요**: 어떤 단계를 수행할 때도 규칙 상세 파일의 관련 내용을 반드시 읽고 사용해야 합니다. 다음 경로를 순서대로 확인하고, 존재하는 첫 번째를 사용합니다:
- `.aidlc-rule-details/` (Cursor, Cline, Claude Code, GitHub Copilot)
- `.kiro/aws-aidlc-rule-details/` (Kiro IDE 및 CLI)
- `.amazonq/aws-aidlc-rule-details/` (Amazon Q Developer)

이후 규칙 상세 파일 참조(예: `common/process-overview.md`, `inception/workspace-detection.md`)는 위에서 해석된 규칙 상세 디렉터리를 기준으로 한 상대 경로입니다.

**공통 규칙**: 워크플로 시작 시 항상 공통 규칙을 로드합니다:
- 워크플로 개요를 위해 `common/process-overview.md` 로드
- 세션 재개 안내를 위해 `common/session-continuity.md` 로드
- 콘텐츠 검증 요구사항을 위해 `common/content-validation.md` 로드
- 질문 형식 규칙을 위해 `common/question-format-guide.md` 로드
- 워크플로 실행 전반에서 이를 참조합니다

## 필수: 확장 로딩(컨텍스트 최적화)
**중요**: 워크플로 시작 시 `extensions/` 디렉터리를 재귀적으로 스캔하되, 가벼운 opt-in 파일만 로드합니다 — 전체 규칙 파일은 로드하지 않습니다. 전체 규칙 파일은 사용자가 옵트인한 뒤 필요 시 로드합니다.

**로딩 절차**:
1. `extensions/` 아래 모든 하위 디렉터리를 나열합니다(예: `extensions/security/`, `extensions/compliance/`)
2. 각 하위 디렉터리에서는 `*.opt-in.md` 파일만 로드합니다 — 확장의 opt-in 프롬프트가 들어 있습니다. 대응 규칙 파일은 관례에 따라 이름에서 `.opt-in.md` 접미사를 제거하고 `.md`를 붙입니다(예: `security-baseline.opt-in.md` → `security-baseline.md`)
3. 이 단계에서는 전체 규칙 파일(예: `security-baseline.md`)을 로드하지 않습니다

**지연 규칙 로딩**:
- 요구사항 분석 중에 로드된 `*.opt-in.md`의 opt-in 프롬프트를 사용자에게 제시합니다
- 사용자가 확장에 옵트인하면 그 시점에 대응 규칙 파일(이름 관례에 따라)을 로드합니다
- 사용자가 옵트아웃하면 전체 규칙 파일은 로드하지 않습니다 — 컨텍스트를 절약합니다
- `*.opt-in.md`가 없는 확장은 항상 강제됩니다 — 워크플로 시작 시 즉시 해당 규칙 파일을 로드합니다

**강제 적용**(로드/활성화된 확장에만 적용):
- 확장 규칙은 선택적 안내가 아니라 하드 제약입니다
- 각 단계에서 모델은 단계 목적, 산출물, 작업 맥락을 바탕으로 어떤 확장 규칙이 적용되는지 지능적으로 판단합니다 — 관련 있는 규칙만 강제합니다
- 현재 단계에 적용되지 않는 규칙은 규정 준수 요약에서 N/A로 표시합니다(차단 이슈가 아님)
- 적용 가능한 활성화된 확장 규칙을 위반하는 것은 **차단 이슈**입니다 — 해결될 때까지 단계 완료를 제시하지 않습니다
- 단계 완료를 제시할 때 확장 규칙 준수 요약을 포함합니다(규칙별 준수/비준수/N/A, N/A 판단에 대한 간단한 근거)

**조건부 강제**: 확장은 조건부로 활성화/비활성화될 수 있습니다. 옵트인 메커니즘은 `inception/requirements-analysis.md`를 참고하세요. 어떤 단계에서든 확장을 강제하기 전에 `aidlc-docs/aidlc-state.md`의 `## Extension Configuration`에서 해당 `Enabled` 상태를 확인합니다. 비활성화된 확장은 건너뛰고 audit.md에 기록합니다. 설정이 없으면 기본적으로 강제합니다.

## 필수: 콘텐츠 검증
**중요**: 어떤 파일을 만들기 전에도 `common/content-validation.md` 규칙에 따라 콘텐츠를 검증해야 합니다:
- Mermaid 다이어그램 문법 검증
- ASCII 아트 다이어그램 검증(`common/ascii-diagram-standards.md` 참고)
- 특수 문자 이스케이프
- 복잡한 시각 콘텐츠에 대한 텍스트 대안 제공
- 콘텐츠 파싱 호환성 테스트

## 필수: 질문 파일 형식
**중요**: 어떤 단계에서 질문할 때도 질문 형식 지침을 따라야 합니다.

**`common/question-format-guide.md`에서 다음을 포함한 전체 질문 형식 규칙을 확인하세요**:
- 객관식 형식(A, B, C, D, E 옵션)
- [Answer]: 태그 사용
- 답변 검증 및 모호성 해소

## 필수: 사용자 정의 환영 메시지
**중요**: 어떤 소프트웨어 개발 요청을 시작할 때도 환영 메시지를 표시해야 합니다.

**환영 메시지 표시 방법**:
1. (해석된 규칙 상세 디렉터리의) `common/welcome-message.md`에서 환영 메시지를 로드합니다
2. 전체 메시지를 사용자에게 표시합니다
3. 새 워크플로 시작 시 한 번만 수행합니다
4. 이후 상호작용에서는 컨텍스트 공간을 절약하기 위해 이 파일을 로드하지 않습니다

# 적응형 소프트웨어 개발 워크플로

---

# INCEPTION 단계

**목적**: 계획, 요구사항 수집, 아키텍처 결정

**초점**: 무엇을 만들지(WHAT), 왜 만들지(WHY)를 결정합니다

**INCEPTION 단계의 단계**:
- Workspace Detection (항상)
- Reverse Engineering (조건부 — Brownfield만)
- Requirements Analysis (항상 — 적응형 깊이)
- User Stories (조건부)
- Workflow Planning (항상)
- Application Design (조건부)
- Units Generation (조건부)

---

## Workspace Detection (항상 실행)

1. **필수**: `audit.md`에 초기 사용자 요청을 전체 원문 입력과 함께 기록합니다
2. `inception/workspace-detection.md`의 모든 단계를 로드합니다
3. 워크스페이스 탐지를 실행합니다:
   - 기존 aidlc-state.md 존재 여부 확인(발견 시 재개)
   - 워크스페이스에서 기존 코드 스캔
   - brownfield 또는 greenfield 판단
   - 기존 reverse engineering 산출물 존재 여부 확인
4. 다음 단계 결정: Reverse Engineering(brownfield이고 산출물 없음) 또는 Requirements Analysis
5. **필수**: `audit.md`에 조사 결과를 기록합니다
6. 사용자에게 완료 메시지를 제시합니다(메시지 형식은 workspace-detection.md 참고)
7. 자동으로 다음 단계로 진행합니다

## Reverse Engineering (조건부 — Brownfield만)

**실행 조건**:
- 기존 코드베이스가 탐지됨
- 이전 reverse engineering 산출물이 없음

**건너뛰기 조건**:
- Greenfield 프로젝트
- 이전 reverse engineering 산출물이 존재함

**실행**:
1. **필수**: `audit.md`에 reverse engineering 시작을 기록합니다
2. `inception/reverse-engineering.md`의 모든 단계를 로드합니다
3. reverse engineering을 실행합니다:
   - 모든 패키지와 컴포넌트 분석
   - 비즈니스 트랜잭션을 포괄하는 전체 시스템의 비즈니스 개요 생성
   - 아키텍처 문서 생성
   - 코드 구조 문서 생성
   - API 문서 생성
   - 컴포넌트 인벤토리 생성
   - 컴포넌트 간 비즈니스 트랜잭션 구현을 나타내는 Interaction Diagram 생성
   - 기술 스택 문서 생성
   - 의존성 문서 생성

4. **명시적 승인 대기**: 상세 완료 메시지를 제시합니다(메시지 형식은 reverse-engineering.md 참고) — 사용자가 확인할 때까지 진행하지 않습니다
5. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

## Requirements Analysis (항상 실행 — 적응형 깊이)

**항상 실행**되나 요청 명확성과 복잡도에 따라 깊이가 달라집니다:
- **Minimal**: 단순·명확한 요청 — 의도 분석만 문서화
- **Standard**: 일반 복잡도 — 기능·비기능 요구사항 수집
- **Comprehensive**: 복잡·고위험 — 추적 가능한 상세 요구사항

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `inception/requirements-analysis.md`의 모든 단계를 로드합니다
3. 요구사항 분석을 실행합니다:
   - reverse engineering 산출물 로드(brownfield인 경우)
   - 사용자 요청 분석(의도 분석)
   - 필요한 요구사항 깊이 결정
   - 현재 요구사항 평가
   - 명확화 질문(필요 시)
   - 요구사항 문서 생성
4. 적절한 깊이(minimal/standard/comprehensive)로 실행합니다
5. **명시적 승인 대기**: requirements-analysis.md 상세 단계의 승인 형식을 따릅니다 — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

## User Stories (조건부)

**지능형 평가**: 다요인 분석으로 user stories가 가치를 더하는지 판단합니다:

**항상 실행** (높은 우선 지표):
- 새로운 사용자 대면 기능 또는 동작
- 사용자 워크플로우나 상호작용에 영향을 주는 변경
- 여러 사용자 유형 또는 페르소나 관련
- 수락 기준이 필요한 복잡한 비즈니스 요구사항
- 교차 기능 팀 협업 필요
- 고객 대면 API 또는 서비스 변경
- 새 제품 역량 또는 개선

**실행 가능성 높음** (중간 우선 — 복잡도 평가):
- 기존 사용자 대면 기능 수정
- 사용자 경험에 간접 영향을 주는 백엔드 변경
- 사용자 워크플로에 영향을 주는 통합 작업
- 사용자에게 보이는 이점이 있는 성능 개선
- 사용자 상호작용에 영향을 주는 보안 강화
- 사용자 데이터나 보고서에 영향을 주는 데이터 모델 변경

**복잡도 기반 평가**: 중간 우선인 경우 다음이면 user stories를 실행합니다:
- 요청이 여러 컴포넌트나 서비스에 걸침
- 변경이 여러 사용자 접점에 걸침
- 비즈니스 로직이 복잡하거나 시나리오가 여러 개
- 요구사항에 모호함이 있어 스토리로 명확해질 수 있음
- 구현이 여러 사용자 여정에 영향
- 변경이 비즈니스 영향이나 위험이 큼

**다음 경우에만 건너뜀** (낮은 우선 — 단순 사례):
- 사용자 영향이 전혀 없는 순수 내부 리팩터링
- 범위가 명확하고 고립된 단순 버그 수정
- 사용자 대면 효과가 없는 인프라 변경
- 기능 변경 없는 기술 부채 정리
- 개발자 도구나 빌드 프로세스 개선
- 문서만 수정

**평가 기준**: 판단이 어려우면 user stories 포함을 선호합니다:
- 비즈니스 이해관계자가 관여하는 요청
- 사용자 수락 테스트가 필요한 변경
- 구현 접근이 여러 가지인 기능
- 팀 공유 이해에 도움이 되는 작업
- 요구사항 명확성이 가치 있는 프로젝트

**평가 절차**:
1. 요청 복잡도와 범위 분석
2. 사용자 영향(직·간접) 식별
3. 비즈니스 맥락과 이해관계자 요구 평가
4. 팀 협업 이점 고려
5. 경계선 사례는 기본적으로 포함

**참고**: Requirements Analysis를 실행했다면 스토리는 해당 요구사항을 참조·확장할 수 있습니다.

**User Stories는 한 단계 안에 두 부분**으로 구성됩니다:
1. **Part 1 - Planning**: 질문이 있는 스토리 계획 작성, 답변 수집, 모호성 분석, 승인
2. **Part 2 - Generation**: 승인된 계획을 실행해 스토리와 페르소나 생성

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `inception/user-stories.md`의 모든 단계를 로드합니다
3. **필수**: user-stories.md의 Step 1과 같이 지능형 평가를 수행해 user stories 필요 여부를 검증합니다
4. reverse engineering 산출물을 로드합니다(brownfield인 경우)
5. 요구사항이 있으면 스토리 작성 시 참조합니다
6. 적절한 깊이(minimal/standard/comprehensive)로 실행합니다
7. **PART 1 - Planning**: 질문이 있는 스토리 계획 작성, 사용자 답변 대기, 모호성 분석, 승인
8. **PART 2 - Generation**: 승인된 계획을 실행해 스토리와 페르소나 생성
9. **명시적 승인 대기**: user-stories.md 상세 단계의 승인 형식을 따릅니다 — 사용자가 확인할 때까지 진행하지 않습니다
10. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

## Workflow Planning (항상 실행)

1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `inception/workflow-planning.md`의 모든 단계를 로드합니다
3. **필수**: `common/content-validation.md`에서 콘텐츠 검증 규칙을 로드합니다
4. 이전 맥락을 모두 로드합니다:
   - reverse engineering 산출물(brownfield인 경우)
   - 의도 분석
   - 요구사항(실행한 경우)
   - user stories(실행한 경우)
5. workflow planning을 실행합니다:
   - 실행할 단계 결정
   - 단계별 깊이 수준 결정
   - multi-package 변경 순서 생성(brownfield인 경우)
   - 워크플로 시각화 생성(작성 전 Mermaid 문법 검증)
6. **필수**: `content-validation.md` 규칙에 따라 파일 생성 전 모든 콘텐츠를 검증합니다
7. **명시적 승인 대기**: workflow-planning.md Step 9의 문구로 권장을 제시하고, 사용자가 권장을 재정의할 수 있음을 강조합니다 — 사용자가 확인할 때까지 진행하지 않습니다
8. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

## Application Design (조건부)

**실행 조건**:
- 새 컴포넌트나 서비스가 필요함
- 컴포넌트 메서드와 비즈니스 규칙 정의가 필요함
- 서비스 레이어 설계가 필요함
- 컴포넌트 의존성을 명확히 해야 함

**건너뛰기 조건**:
- 기존 컴포넌트 경계 내 변경
- 새 컴포넌트나 메서드 없음
- 순수 구현 변경

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `inception/application-design.md`의 모든 단계를 로드합니다
3. reverse engineering 산출물을 로드합니다(brownfield인 경우)
4. 적절한 깊이(minimal/standard/comprehensive)로 실행합니다
5. **명시적 승인 대기**: 상세 완료 메시지를 제시합니다(메시지 형식은 application-design.md 참고) — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

## Units Generation (조건부)

**실행 조건**:
- 시스템을 여러 작업 단위로 분해해야 함
- 여러 서비스나 모듈이 필요함
- 구조적 분해가 필요한 복잡한 시스템

**건너뛰기 조건**:
- 단일 단순 단위
- 분해 불필요
- 단일 컴포넌트 구현이 직관적

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `inception/units-generation.md`의 모든 단계를 로드합니다
3. reverse engineering 산출물을 로드합니다(brownfield인 경우)
4. 적절한 깊이(minimal/standard/comprehensive)로 실행합니다
5. **명시적 승인 대기**: 상세 완료 메시지를 제시합니다(메시지 형식은 units-generation.md 참고) — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

---

# 🟢 CONSTRUCTION 단계

**목적**: 상세 설계, NFR 구현, 코드 생성

**초점**: 어떻게 만들지(HOW)를 결정합니다

**CONSTRUCTION 단계의 단계**:
- 단위별 루프(각 단위마다 실행):
  - Functional Design (조건부, 단위별)
  - NFR Requirements (조건부, 단위별)
  - NFR Design (조건부, 단위별)
  - Infrastructure Design (조건부, 단위별)
  - Code Generation (항상, 단위별)
- Build and Test (항상 — 모든 단위 완료 후)

**참고**: 각 단위는 다음 단위로 넘어가기 전에 설계와 코드를 모두 완료합니다.

---

## 단위별 루프(각 작업 단위마다 실행)

**각 작업 단위에 대해 다음 단계를 순서대로 실행합니다:**

### Functional Design (조건부, 단위별)

**실행 조건**:
- 새 데이터 모델이나 스키마
- 복잡한 비즈니스 로직
- 비즈니스 규칙의 상세 설계 필요

**건너뛰기 조건**:
- 단순 로직 변경
- 새 비즈니스 로직 없음

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `construction/functional-design.md`의 모든 단계를 로드합니다
3. 해당 단위에 대해 functional design을 실행합니다
4. **필수**: functional-design.md에 정의된 표준 2옵션 완료 메시지를 제시합니다 — 임의의 3옵션 동작을 사용하지 않습니다
5. **명시적 승인 대기**: 사용자는 "Request Changes"와 "Continue to Next Stage" 중 선택해야 합니다 — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

### NFR Requirements (조건부, 단위별)

**실행 조건**:
- 성능 요구사항 존재
- 보안 고려 필요
- 확장성 우려 존재
- 기술 스택 선택 필요

**건너뛰기 조건**:
- NFR 요구사항 없음
- 기술 스택이 이미 결정됨

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `construction/nfr-requirements.md`의 모든 단계를 로드합니다
3. 해당 단위에 대해 NFR 평가를 실행합니다
4. **필수**: nfr-requirements.md에 정의된 표준 2옵션 완료 메시지를 제시합니다 — 임의 동작을 사용하지 않습니다
5. **명시적 승인 대기**: 사용자는 "Request Changes"와 "Continue to Next Stage" 중 선택해야 합니다 — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

### NFR Design (조건부, 단위별)

**실행 조건**:
- NFR Requirements 단계가 실행됨
- NFR 패턴을 반영해야 함

**건너뛰기 조건**:
- NFR 요구사항 없음
- NFR Requirements를 건너뜀

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `construction/nfr-design.md`의 모든 단계를 로드합니다
3. 해당 단위에 대해 NFR design을 실행합니다
4. **필수**: nfr-design.md에 정의된 표준 2옵션 완료 메시지를 제시합니다 — 임의 동작을 사용하지 않습니다
5. **명시적 승인 대기**: 사용자는 "Request Changes"와 "Continue to Next Stage" 중 선택해야 합니다 — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

### Infrastructure Design (조건부, 단위별)

**실행 조건**:
- 인프라 서비스 매핑 필요
- 배포 아키텍처 필요
- 클라우드 리소스 명세 필요

**건너뛰기 조건**:
- 인프라 변경 없음
- 인프라가 이미 정의됨

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `construction/infrastructure-design.md`의 모든 단계를 로드합니다
3. 해당 단위에 대해 infrastructure design을 실행합니다
4. **필수**: infrastructure-design.md에 정의된 표준 2옵션 완료 메시지를 제시합니다 — 임의 동작을 사용하지 않습니다
5. **명시적 승인 대기**: 사용자는 "Request Changes"와 "Continue to Next Stage" 중 선택해야 합니다 — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

### Code Generation (항상 실행, 단위별)

**각 단위마다 항상 실행됩니다**

**Code Generation은 한 단계 안에 두 부분**으로 구성됩니다:
1. **Part 1 - Planning**: 명시적 단계가 있는 상세 코드 생성 계획 작성
2. **Part 2 - Generation**: 승인된 계획을 실행해 코드, 테스트, 산출물 생성

**실행**:
1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `construction/code-generation.md`의 모든 단계를 로드합니다
3. **PART 1 - Planning**: 체크박스가 있는 코드 생성 계획 작성, 사용자 승인
4. **PART 2 - Generation**: 승인된 계획을 실행해 해당 단위 코드 생성
5. **필수**: code-generation.md에 정의된 표준 2옵션 완료 메시지를 제시합니다 — 임의 동작을 사용하지 않습니다
6. **명시적 승인 대기**: 사용자는 "Request Changes"와 "Continue to Next Stage" 중 선택해야 합니다 — 사용자가 확인할 때까지 진행하지 않습니다
7. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

---

## Build and Test (항상 실행)

1. **필수**: 이 단계 중 사용자 입력은 모두 `audit.md`에 기록합니다
2. `construction/build-and-test.md`의 모든 단계를 로드합니다
3. 포괄적인 빌드 및 테스트 지침을 생성합니다:
   - 모든 단위에 대한 빌드 지침
   - 단위 테스트 실행 지침
   - 통합 테스트 지침(단위 간 상호작용 테스트)
   - 성능 테스트 지침(해당 시)
   - 필요에 따른 추가 테스트 지침(계약 테스트, 보안 테스트, e2e 테스트)
4. `build-and-test/` 하위에 지침 파일 생성: build-instructions.md, unit-test-instructions.md, integration-test-instructions.md, performance-test-instructions.md, build-and-test-summary.md
5. **명시적 승인 대기**: "**Build and test instructions complete. Ready to proceed to Operations stage?**"라고 질문합니다 — 사용자가 확인할 때까지 진행하지 않습니다
6. **필수**: `audit.md`에 사용자 응답을 전체 원문 입력과 함께 기록합니다

---

# 🟡 OPERATIONS 단계

**목적**: 향후 배포·모니터링 워크플로를 위한 자리 표시자

**초점**: 배포·실행 방법(향후 확장)

**OPERATIONS 단계의 단계**:
- Operations (PLACEHOLDER)

---

## Operations (PLACEHOLDER)

**상태**: 이 단계는 현재 향후 확장을 위한 자리 표시자입니다.

Operations 단계에는 최종적으로 다음이 포함될 예정입니다:
- 배포 계획 및 실행
- 모니터링·관측 가능성 설정
- 사고 대응 절차
- 유지보수·지원 워크플로
- 프로덕션 준비 체크리스트

**현재 상태**: 모든 빌드 및 테스트 활동은 CONSTRUCTION 단계에서 처리됩니다.

## 핵심 원칙

- **적응형 실행**: 가치를 더하는 단계만 실행
- **투명한 계획**: 시작 전 항상 실행 계획 표시
- **사용자 제어**: 사용자가 단계 포함/제외를 요청할 수 있음
- **진행 추적**: 실행·건너뛴 단계를 `aidlc-state.md`에 반영
- **완전한 감사 추적**: `audit.md`에 타임스탬프와 함께 모든 사용자 입력과 AI 응답 기록
  - **중요**: 사용자의 **전체 원문 입력**을 제공된 그대로 캡처
  - **중요**: 감사 로그에서 사용자 입력을 요약하거나 의역하지 않음
  - **중요**: 승인만이 아니라 모든 상호작용 기록
- **품질 중심**: 복잡한 변경은 전체 절차, 단순 변경은 효율 유지
- **콘텐츠 검증**: `content-validation.md` 규칙에 따라 파일 생성 전 항상 검증
- **임의 동작 금지**: CONSTRUCTION 단계는 해당 규칙 파일에 정의된 표준 2옵션 완료 메시지를 반드시 사용합니다. 3옵션 메뉴나 기타 임의 탐색 패턴을 만들지 않습니다.

## 필수: 계획 수준 체크박스 강제

### 계획 실행에 대한 필수 규칙
1. **계획 체크박스를 갱신하지 않고는 어떤 작업도 완료로 처리하지 않음**
2. **계획 파일에 기술된 어떤 단계를 완료하자마자 즉시 해당 단계를 [x]로 표시**
3. **작업이 완료된 동일 상호작용 안에서 반드시 수행**
4. **예외 없음**: 계획 단계 완료마다 체크박스 갱신으로 추적

### 두 수준 체크박스 추적
- **계획 수준**: 각 단계 내 상세 실행 진행 추적
- **단계 수준**: `aidlc-state.md`에서 전체 워크플로 진행 추적
- **즉시 갱신**: 작업 완료와 동일한 상호작용에서 모든 진행 갱신

## 프롬프트 로깅 요구사항
- **필수**: 모든 사용자 입력(프롬프트, 질문, 응답)을 `audit.md`에 타임스탬프와 함께 기록
- **필수**: 사용자의 **전체 원문 입력**을 제공된 그대로 캡처(요약하지 않음)
- **필수**: 사용자에게 묻기 전 모든 승인 프롬프트를 타임스탬프와 함께 기록
- **필수**: 수신 후 모든 사용자 응답을 타임스탬프와 함께 기록
- **중요**: 항상 `audit.md`를 **편집(append)**하여 변경합니다. 내용 전체를 덮어쓰는 도구·명령은 사용하지 않습니다
- **중요**: `audit.md` 전체를 덮어쓰는 파일 쓰기 도구·명령은 사용하지 않습니다(중복 발생)
- 타임스탬프는 ISO 8601 형식(YYYY-MM-DDTHH:MM:SSZ)
- 각 항목에 단계 맥락 포함

### 감사 로그 형식:
```markdown
## [Stage Name or Interaction Type]
**Timestamp**: [ISO timestamp]
**User Input**: "[Complete raw user input - never summarized]"
**AI Response**: "[AI's response or action taken]"
**Context**: [Stage, action, or decision made]

---
```

### audit.md에 대한 올바른 도구 사용

✅ 올바름:

1. audit.md 파일을 읽습니다
2. 파일을 append/편집하여 변경합니다

❌ 잘못됨:

1. audit.md 파일을 읽습니다
2. 읽은 내용에 새 변경을 더해 audit.md 전체를 덮어씁니다

## 디렉터리 구조

```text
<WORKSPACE-ROOT>/                   # ⚠️ APPLICATION CODE HERE
├── [project-specific structure]    # Varies by project (see code-generation.md)
│
├── aidlc-docs/                     # 📄 DOCUMENTATION ONLY
│   ├── inception/                  # 🔵 INCEPTION PHASE
│   │   ├── plans/
│   │   ├── reverse-engineering/    # Brownfield only
│   │   ├── requirements/
│   │   ├── user-stories/
│   │   └── application-design/
│   ├── construction/               # 🟢 CONSTRUCTION PHASE
│   │   ├── plans/
│   │   ├── {unit-name}/
│   │   │   ├── functional-design/
│   │   │   ├── nfr-requirements/
│   │   │   ├── nfr-design/
│   │   │   ├── infrastructure-design/
│   │   │   └── code/               # Markdown summaries only
│   │   └── build-and-test/
│   ├── operations/                 # 🟡 OPERATIONS PHASE (placeholder)
│   ├── aidlc-state.md
│   └── audit.md
```

**CRITICAL RULE**:
- Application code: Workspace root (NEVER in aidlc-docs/)
- Documentation: aidlc-docs/ only
- Project structure: See code-generation.md for patterns by project type
