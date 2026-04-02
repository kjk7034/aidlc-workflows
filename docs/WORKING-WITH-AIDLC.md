# AIDLC 사용하기

이 가이드는 AI-DLC(AI-Driven Development Life Cycle)를 최대한 활용하는 방법을 다룹니다. 첫 프롬프트부터 동작하는 코드까지 각 단계에서 AI와 효과적으로 상호작용하는 법을 설명합니다.

각 절에서는 기본부터 읽으세요. 고급 팁은 실제 워크숍 경험에서 나왔으며, 기본에 익숙해진 뒤 팀이 가장 유용하다고 느낀 패턴을 다룹니다.

---

## 목차

1. [일반 규칙](#1-general-rules)
2. [Inception 단계](#2-inception-phase)
3. [Construction 단계](#3-construction-phase)
4. [Vibe 코딩 금지](#4-never-vibe-code)

---

## 1. General Rules

### 문서를 바꾸지 않고 질문하기

일찍 들여야 할 습관 중 하나: **모든 질문이 문서 갱신으로 이어지면 안 됩니다.**

질문을 제한하지 않으면 AI가 변경 요청으로 해석해 설계 문서를 바로 고칠 수 있습니다. 탐색형 질문 앞에는 문서를 바꾸지 않겠다는 명확한 지시를 붙이세요.

**기본 패턴:**

```
Do not update any documents. Help me understand why [this decision] was made.
```

```
Do not update any documents. For [component name], is it reasonable to use [library or technology] here?
```

```
Do not change anything. Assess the impact of [proposed change].
I want to understand the consequences before we decide.
```

이 패턴으로 AI와 대화하며 대안을 평가하고 결정에 도전할 수 있고, 아무것도 확정하지 않을 수 있습니다. 답에 만족하면 필요할 때만 의도적인 갱신 지시를 이어가면 됩니다.

> **팁**: 탐색 메시지는 "Do not update any documents."로 시작하세요. 실행할 준비가 되면 그 제약을 풀면 됩니다.

---

### 질문 → 문서 → 승인 흐름

AIDLC는 채팅 안에 명확화 질문을 넣지 않습니다. 질문을 마크다운 파일에 쓰고, 그곳에 답을 채우기를 기다립니다. 모든 결정의 지속 기록이 되고 팀 전체가 기여하기 쉽습니다.

**1단계 — AIDLC가 질문 파일 생성**

AI가 `aidlc-docs/inception/requirements/requirement-verification-questions.md` 같은 파일을 만들고 멈춥니다. 답할 때까지 진행하지 않습니다.

**2단계 — 답을 채움**

파일을 열고 각 `[Answer]:` 태그를 채웁니다. 질문은 객관식 형식입니다.

```markdown
## Question: Deployment model
Where will this service be deployed?

A) AWS Lambda (serverless)
B) AWS ECS Fargate (containerized)
C) Existing on-premises infrastructure
X) Other (please describe after [Answer]: tag below)

[Answer]: B
```

답할 때 잘 통하는 방법:

- **문자와 함께 라벨을 붙이세요.** `C — financial summary and debt service coverage`는 `C`만 쓰는 것보다 분명합니다.
- **짧은 근거를 넣으세요.** `A — design-first; generate the OpenAPI spec before writing code`는 의도를 확인하고 AI가 이어가 맥락을 줍니다.
- **둘 다 해당하면 둘을 조합하세요.** `B and C — rate limiting at both API Gateway level and application level (not D)`는 모호하지 않습니다.
- **거의 맞는 옵션일 때 단서를 덧붙이세요.** `B — migration is a separate project; however, include a one-time migration into the new data structures.`
- **X를 자유롭게 쓰세요.** 어느 것도 맞지 않으면 억지로 고르지 말고 X가 맞습니다.

**3단계 — AI에게 답을 준비했다고 알림**

채팅으로 돌아가 "We have answered your clarification questions. Please re-read the file and proceed."라고 말합니다.

팁: AI에게 파일을 *다시 읽으라*고 명시하면 디스크에서 답을 읽고, 최신 편집이 반영되지 않은 메모리 버전에 의존하지 않게 할 수 있습니다.

**4단계 — AIDLC가 검증하고 진행**

AI가 답을 읽고 남은 모호함을 표시한 뒤 다음 산출물을 생성합니다.

> **고급 팁**: AI의 질문 일부가 이미 문서에 답이 있다면 스스로 해결하라고 지시할 수 있습니다. "각 질문의 근거를 분석하라. 제공된 문서로 이미 답된 질문은 스스로 답하라. 여전히 불명확할 때만 나에게 물어라." 게이트에서 불필요한 왕복을 줄입니다.

**승인 게이트**

각 단계 끝에서 AIDLC는 두 가지 옵션이 있는 완료 메시지를 보여 줍니다.

- **Request Changes** — 넘어가기 전에 수정 요청
- **Approve and Continue** — 산출물을 받아들이고 진행

승인 전에 생성된 산출물을 읽으세요. 필요하면 팀과 논의합니다. 만족할 때만 승인하세요.

---

### 맥락 관리

맥락은 세션에서 AI의 작업 기억입니다. AIDLC는 일관된 하류 산출물을 만들려면 아티팩트와 지침의 전체 사슬이 맥락에 있어야 합니다. 잘 관리하는 것이 가장 레버리지가 큰 습관입니다.

**핵심 규칙: 자연스러운 결정 지점마다 맥락을 비우세요.**

AIDLC는 게이트를 중심으로 설계되어 있습니다. AI가 멈추고 무언가를 요청하는 순간 — 답할 질문 파일, 승인할 문서, 검토할 계획입니다. 이 멈춤은 단순한 승인점이 아닙니다. 이어가기 전에 새 맥락을 시작하기 좋은 순간입니다.

게이트에서 맥락을 비우는 위험은 낮습니다. AI의 현재 작업은 이미 파일에 저장되어 있기 때문입니다. 다음 맥락은 깨끗하게 시작하고 디스크에서 관련 아티팩트를 읽으며, 이전 단계의 누적된 노이즈를 끌고 가지 않습니다.

여러 게이트에 걸쳐 맥락을 쌓이게 두면 AI는 이전 지침과 아티팩트의 압축되거나 일부만 남은 버전으로 일합니다. 출력 품질이 미묘하게 떨어지고 진단하기 어렵습니다.

**실무:**

- AI가 질문 파일에 답하라고 하면 — 질문에 답한 뒤 **새 맥락을 시작**하고 파일을 다시 읽고 계속하라고 요청
- AI가 승인할 문서를 보여 주면 — 검토한 뒤 **새 맥락을 시작**해 변경 요청 또는 승인 후 계속
- 도구가 워크플로 중간에 "compact context"를 제안하면 **항상 거절** — 압축은 깨끗한 리셋과 같지 않고 잃는 것이 더 큼

**맥락 리셋 후 재개:**

옵션 1 — 상태 파일 방식(권장):
```
Go to aidlc-docs/aidlc-state.md, find the first unchecked item,
then go to the corresponding plan file and resume from that point.
```

옵션 2 — 수동 인계:
```
I am resuming a previously stopped conversation. Here is the context:
[paste summary of last output or recent change]
Please continue with [next action or section X].
```

> **팁**: 맥락을 리셋할 때마다 현재 변경을 모두 커밋하고 푸시하세요. 몇 초 걸리며 항상 깨끗한 복구 지점이 생깁니다.

```
Please commit and push all current changes to the repository.
```

---

### 프롬프트 묶기

모든 프롬프트를 따로 보낼 필요는 없습니다. 워크숍에서 나온 단순한 규칙:

**같은 주제에 밀접하게 묶인 두 가지 변경은 한 프롬프트에 넣으세요. 관련 없는 두 가지 변경은 한 번에 하나씩 하세요.**

과도한 묶기(관련 없는 변경 결합)는 AI가 초점을 잃고 세부를 놓치게 합니다. 과소 묶기(밀접한 것을 여러 프롬프트로 쪼개기)는 불필요한 왕복을 늘립니다. 의심스러우면 나누는 쪽이 낫습니다.

---

### 외부 참조 파일 로드

스키마, 아키텍처 다이어그램, 데이터 사전, API 스펙 등 기존 문서를 AIDLC에 가리켜 현재 단계에 반영할 수 있습니다.

**기본 패턴:**

```
Please read [file path or description]. Use it as the basis for [what you want].
```

```
We have an existing audit table structure. Please add it to the inception documents
and reference it for this service. When we proceed, expect new requirements and
stories related to this service.
```

> **고급 팁**: 문서는 시작할 때만이 아니라 어느 단계에서든 로드할 수 있습니다. Construction 중에 새 제약이 나오면 — 갱신된 보안 정책, 수정된 데이터 모델 — 로드하고 진행하기 전에 AIDLC에 영향을 평가하라고 요청하세요.

> **고급 팁 — 기업 표준을 확장으로**: 조직에 모든 프로젝트에 적용할 보안, 규정 준수, API 지침이 있으면 `aidlc-rules/extensions/`에 마크다운 steering 파일로 추가하세요. AIDLC가 수동 주입 없이 모든 단계에 자동으로 로드합니다.

---

### 독립적인 비평 받기

AIDLC는 자기 이전 결정을 옹호합니다. 산출물에 대한 편향 없는 평가를 원하면 **새 맥락**에서 비평을 요청하세요 — 그 결정을 왜 내렸는지 기억이 없는 맥락입니다.

```
Produce a critique document of [the requirements document / the component design].
Do this in a new context separate from everything else.
```

산출물을 만든 같은 세션에서 비평을 요청하는 것보다 유용하고 객관적인 피드백이 나옵니다.

---

### 깊이 수준

AIDLC는 요청 복잡도에 따라 각 단계를 얼마나 깊게 실행할지 조정합니다. 영향을 줄 수 있습니다.

```
Keep this at minimal depth — we just need the basic structure documented.
```

```
This is a production-critical component. Please run at comprehensive depth.
```

---

## 2. Inception Phase

Inception 단계는 설계나 코드 작업 전에 *무엇을 왜 만들지* AI와 맞추는 곳입니다. 여기서 맥락을 많이 가져올수록 Construction에서 명확화 질문과 재작업이 줄어듭니다.

### 시작 전 입력 준비

AIDLC를 시작하기 전에 가장 효과적인 일은 문서 두 가지를 준비하는 것입니다.

1. **비전 문서** — 무엇을 왜 만들지
2. **기술 환경 문서** — 어떤 도구와 제약이 적용되는지

이 문서는 AIDLC가 묻는 명확화 질문 수를 크게 줄이고, AI가 추측하지 않고 팀의 실제 맥락에서 시작하게 합니다.

**시작점:**

- [writing-inputs/inputs-quickstart.md](writing-inputs/inputs-quickstart.md) — 그린필드와 브라운필드 요약
- [writing-inputs/vision-document-guide.md](writing-inputs/vision-document-guide.md) — 템플릿이 있는 전체 비전 가이드
- [writing-inputs/technical-environment-guide.md](writing-inputs/technical-environment-guide.md) — 전체 기술 환경 가이드

**브라운필드 프로젝트**(기존 코드베이스에 추가)는 조금 다른 입력이 필요합니다. 비전 문서에는 현재 상태 설명과 바꾸면 안 되는 목록이 있어야 합니다. 기술 환경 문서는 바라는 스택이 아니라 기존 스택을 설명하고, 예시 코드는 실제 파일에서 가져와야 합니다. 브라운필드 최소치와 예시는 [writing-inputs/inputs-quickstart.md](writing-inputs/inputs-quickstart.md)를 보세요.

빠르게 시작하려는 경우 **최소 입력**은 다음과 같습니다.

비전: 무엇을 누구를 위해 만드는지 한 단락, MVP 포함 기능 목록, 명시적 제외 기능 목록, 이미 불확실하다고 아는 미결 질문. 미결 질문은 Requirements Analysis에 미리 선언된 모호함으로 들어가 설계 중간 깜짝 이슈 대신 초기에 해결됩니다.

기술 환경: 언어와 버전, 패키지 관리자, 웹 프레임워크, 클라우드 제공자와 배포 모델, 테스트 프레임워크, 금지 라이브러리 표(항목별 이유와 권장 대안), 보안 기본, 일반적인 엔드포인트·함수·테스트 예시 각각 하나 이상.

금지 라이브러리 표는 단순 목록보다 중요합니다. 이유와 대안 열이 AI-DLC에 *왜* 금지인지 알려 더 나은 대체 결정으로 이어집니다. 예시 코드 패턴은 기본 이후 가장 효과가 큰 추가로, 코드 생성 시 스스로 만들지 않고 따를 구체적 패턴을 줍니다.

> **팁**: 미리 채운 빈칸 하나가 Requirements Analysis에서 줄어드는 질문 하나입니다.

---

### 새 프로젝트 시작

입력 문서가 준비되면:

```
I want to start a new project. Please read [path to vision document] and
[path to technical environment document], then begin the AIDLC workflow.
```

AIDLC가 워크스페이스를 스캔해 그린필드 vs 브라운필드를 판단하고, 문서를 주 소스로 Requirements Analysis로 진행합니다 — 문서가 다루지 않는 것만 묻습니다.

브라운필드에서는 AIDLC가 먼저 Reverse Engineering을 실행해 기존 코드베이스를 분석하고 아키텍처·컴포넌트·API 문서를 만듭니다. 이 산출물을 꼼꼼히 검토하세요 — 이후 모든 것의 기반이 됩니다.

---

### 요구 사항 질문에 답하기

[1절](#the-question--doc--approval-flow)의 답변 팁에서 문자 사용, 라벨 추가, 옵션 조합, 사용자 정의 답의 X 사용을 모두 다룹니다. Requirements Analysis에만 해당하는 추가 포인트:

- **전체 비전과 MVP를 분리해 말하세요.** AIDLC가 포함할 기능을 물으면 이름을 명시하세요. 범위 밖이면 그렇다고 하세요 — 모호하게 두지 마세요.
- **의도적인 "아니오"를 분명히.** `D — no caching required at this time`는 의도를 전달합니다. 빈 답은 AI가 추측하도록 초대합니다.
- **단계적 접근을 한 줄에 설명.** `X — simple role-based workflow now; replace with external workflow engine when available`는 현재 솔루션을 올바른 확장 지점과 함께 설계하게 합니다.

> **고급 팁 — Security Extensions**: Requirements Analysis 중 AIDLC가 보안 확장 규칙을 강제할지 묻습니다. 프로덕션급 애플리케이션은 Yes, 프로토타입은 No도 가능합니다. 이 결정은 기록되고 Construction 전체에서 강제되므로 의도적으로 선택하세요.

---

### Inception 전용 상호작용

**진행 중 기능을 미루기:**

```
We are going to backlog the [feature name] capability for the current release.
Please remove it from the component design and flag the related user stories as backlogged.
```

백로그(삭제 대신)로 두면 현재 빌드에 영향을 주지 않고 이후 반복을 위해 작업을 보존합니다.

**기존 데이터 구조 등록:**

```
We have an existing [schema/structure name]. Please add it to the inception documents
and reference it for this service. When we proceed, expect new requirements and
stories related to this service.
```

**Making implicit data sources explicit:**

```
For the [service name], add the understanding that [new data source] is also a
data source for this feature, in addition to [existing data source]. Then review
requirements and user stories to ensure this is captured.
```

**설계 변경 후 상위 영향 확인:**

설계 산출물을 의미 있게 바꾼 뒤에는 AIDLC에 이전 문서가 여전히 일관되는지 확인하라고 요청하세요.

```
Now review the previous steps — user stories and requirements — to ensure
this change does not require updates to any of those documents.
```

> **고급 팁 — 상시 역전파 규칙**: 변경할 때마다 묻는 대신, 단계 시작 시 상시 지시로 두세요. "문서를 갱신할 때마다 요구 사항 문서와 사용자 스토리에 영향이 있는지 확인하고, 있으면 나에게 알려라." 기억을 요구하지 않고 자동 안전망이 됩니다.

**컴포넌트 설계 병렬 팀 검토:**

팀이 나뉘어 여러 컴포넌트를 동시에 검토할 때:

```
Restrict your edits to the files under your team's control. When all teams are done,
we will ask the AI to review all changes and confirm there are no conflicts.
Then we will ask it to review impacts to user stories and requirements.
```

모두 끝나면 충돌 검사를 트리거합니다.

```
We had [N] independent groups editing component design files. Please review all files
and report any conflicts or inconsistencies. Do not edit the files — produce a report
for our review.
```

충돌을 번호로 명시해 해결합니다.

```
For conflict #[number] ([conflict description]):
update [target file] to reflect [your decision].
```

```
For conflict #[number] ([capability name]):
this capability is backlogged. Update the documentation to clearly mark it as
backlogged so code generation does not attempt to implement it.
```

**오래된 설계 파일 보관:**

설계 중 탐색으로 더 이상 필요 없는 파일이 생겼을 때:

```
Move the [file descriptions] to an archive folder — do not delete them.
Then confirm whether they are required for code generation.
```

> **고급 팁 — 컴포넌트 크기 제약**: 한 스프린트에 구현하기 너무 큰 컴포넌트를 막으려면 Application Design에서 스토리 포인트 상한을 둡니다. "컴포넌트 설계 단계에서 다음 지시를 넣어라: 단일 컴포넌트의 합산 스토리 포인트는 [X]를 넘지 않는다. 넘으면 더 작은 하위 컴포넌트로 나눈다."

> **고급 팁 — 단계 중간 맥락 리셋**: 세션이 끊기면 상태를 다시 잡기 위해 이렇게 쓰세요.
>
> ```
> Stop. New context. We just completed [description of recent work].
> Please review [upstream artifacts] to assess any impact of the recent change.
> [Paste the change description here.]
> ```

---

## 3. Construction Phase

Construction 단계는 설계가 코드가 되는 곳입니다. 각 작업 단위는 (조건부) 설계 단계들을 거친 뒤 항상 Code Generation을 거칩니다. 모든 단위가 끝나면 Build and Test로 마무리합니다.

### 설계 검토 과정

작업 단위마다 코드 생성 전에 AIDLC가 아래 설계 단계 중 일부 또는 전부를 실행할 수 있습니다.

- **Functional Design** — 비즈니스 로직, 도메인 모델, 데이터 스키마
- **NFR Requirements** — 성능, 보안, 확장성, 기술 스택 선택
- **NFR Design** — 설계에 NFR 패턴 적용
- **Infrastructure Design** — 설계를 실제 클라우드 서비스에 매핑

각 단계는 `aidlc-docs/construction/{unit-name}/`에 문서를 만듭니다. 각 게이트에서 문서를 읽고 변경 요청 또는 승인을 결정합니다.

**승인 전에 읽으세요.** 설계 문서는 코드 생성의 진실 공급원입니다. 여기서 새는 실수는 나중에 고치기 어렵습니다.

**설계에서 코드로 넘어갈 때:**

Code Generation으로 넘어갈 준비가 되면 AI에 필요한 구조 맥락을 미리 주세요.

```
We have completed component design review. We are ready for code creation.
Please use the following directory and source code structure:
[reference an existing service or folder structure].
Use this pattern for APIs. For the UI, follow the [Vue.js composables/components/store]
directory structure. Please ask any questions you have before proceeding.
```

Inviting questions before generation starts resolves ambiguities in the plan rather than in the middle of file creation.

**Requesting a targeted correction:**

Be precise — name the element, what is wrong, and what it should be:

```
The [endpoint description] should use [correct parameter], not [incorrect parameter].
Please update the [component name] accordingly.
```

**Choosing between AI-presented options:**

```
Please implement Option B — [option description] — for [feature name].
Update all component design documents accordingly.
```

옵션을 문자와 *설명* 모두로 가리키고, 질문이 나온 문서뿐 아니라 영향받는 모든 문서로 갱신 범위를 명시하세요.

**설계 패턴 재정의:**

```
We prefer to deviate from [standard pattern] and use [our preferred approach]
to allow [rationale]. Please update the component design documents accordingly.
```

근거가 중요합니다. AIDLC가 이후 단계로 가져가며 편차가 조용히 되돌아가지 않게 합니다.

> **고급 팁 — 커밋 전 영향 평가**: 의미 있는 설계 변경은 실행 전에 평가하세요.
>
> ```
> Do not change anything. Assess the impact of [proposed change].
> [Describe the proposed change in detail.]
> ```

> **고급 팁 — 인라인 코드 문서화**: 모든 단위에 일관되게 인라인 문서를 넣으려면 Construction 단계 시작 시 상시 규칙으로 한 번만 넣으세요. 단위마다 반복하지 마세요. "construction 단계 표준 규칙으로 인라인 코드 문서를 추가하라."

---

### 코드 생성 과정

Code Generation은 두 부분으로 나뉩니다. 둘 다 명시적 승인이 필요합니다.

**1부 — 계획**

AIDLC가 만들거나 수정할 모든 파일에 대한 번호 매긴 체크박스 계획을 만듭니다. 승인 전에 검토하세요.

- 모든 파일이 올바른 위치에 있는지(애플리케이션 코드는 워크스페이스 루트, `aidlc-docs/` 안 아님)
- 단계가 설계 문서가 명시한 것을 모두 다루는지
- 브라운필드는 기존 파일 수정을 나열하고 옆에 새 중복을 두지 않는지

> **고급 팁 — 내부 라이브러리**: 계획을 승인하기 전에 Q&A 파일이나 구현 계획에 내부 라이브러리 요구를 넣으세요.
>
> ```
> In addition to my answers, you must use the following libraries from our
> [starter project / building blocks]: [list each library explicitly].
> Explain why and when each should be used, not just what it is.
> ```
>
> 저장소만 가리키는 것보다 내부 라이브러리용으로 정리한 마크다운 가이드가 낫습니다. 만들어 코드 생성 입력으로 참조하세요.

> **고급 팁 — Figma 디자인에서 UI**: Figma 디자인 스크린샷을 vision 모델(예: ChatGPT)에 넘겨 스크린샷에서 프레임워크 코드를 생성한 뒤, 그 출력을 AIDLC UI 구현 입력으로 주세요. 디자인 도구 raw 내보내기보다 구체적이고 도구가 읽기 쉬운 명세가 됩니다.

**2부 — 생성**

AIDLC가 각 단계를 순서대로 실행하며 완료할 때마다 체크합니다. 모두 끝나면 생성된 파일 경로와 함께 완료 메시지를 보여 줍니다.

승인 전에 생성된 코드를 검토하세요. 문제가 있으면:

```
Request Changes: [describe specifically what needs to change]
```

> **고급 팁 — 브라운필드 파일 수정**: 기존 코드베이스에서는 AIDLC가 제자리에서 파일을 수정합니다. 원본 옆에 `ClassName_modified.java`나 `service_new.ts`가 보이면 즉시 지적하세요.
>
> ```
> I see [ClassName_modified.java] alongside [ClassName.java]. Please merge the changes
> into the original file and delete the duplicate.
> ```

---

### 빌드 및 테스트

모든 단위가 끝나면 AIDLC가 모든 단위에 대한 빌드·테스트 안내를 생성합니다. 알아 두면 좋은 패턴:

**테스트 도구를 적절한 시점에 주입:**

프로젝트 시작 시점에 테스트 프레임워크나 테스트 관리 시스템 지침을 넣지 마세요. 코드 생성이 시작될 때쯤이면 여러 단계에서 압축되거나 사라졌을 수 있습니다. 필요할 때 맞춰 주입하세요.

```
At the functional test generation step, inject the following instruction:
generate functional tests using the [test management system] format described
in this document: [attach specification]. Use this API endpoint to push the
generated test cases to the [test management system] repository: [endpoint details].
```

도구별 지침은 모두 같은 원칙입니다. 프로젝트 시작이 아니라 필요한 단계에 주입하세요.

**단위 테스트 커버리지 범위:**

```
When generating unit tests, exclude third-party external dependencies from
code coverage calculations. Require a minimum of 80% coverage on internal
code paths only.
```

---

### 코드 생성 후: 변경 역전파

코드 생성 중에 내린 작은 설계 결정, 코딩하며 발견한 조정은 설계 문서로 다시 올라가야 합니다. 코드 다듬기가 끝난 뒤 의도적으로 한 번에 훑으며 하고, 즉흥적으로 하지 마세요.

```
When you have finished polishing the code, review each unit's final design files
and propagate any changes back up the chain to requirements and user stories.
Make a plan for how to do this step by step before executing.
```

실행 전에 계획을 요청하면 모든 단위에 걸쳐 체계적으로 훑고 선택적으로만 하지 않게 됩니다.

> **고급 팁 — 재사용 명세 추출**: 완료된 프로젝트 끝에서 정립한 패턴을 이후 프로젝트용 재사용 명세 문서로 뽑으세요.
>
> ```
> Create a set of reusable specification documents from the patterns expressed
> in this project: one for API design, one for security, one for UI specifications,
> one for the technology stack, and one for directory structure. Use the completed
> units as the source. I will review and approve each document before it is used
> in future projects.
> ```

---

## 4. Never Vibe Code

Vibe 코딩은 설계 문서를 건너뛰고 생성된 코드 파일을 직접 고쳐 빠르게 수정하거나 시험해 보는 것입니다. 당장은 빠르게 느껴지고 곧 문제가 됩니다.

문제는 수정 자체가 아니라 설계 문서 — AIDLC가 이후 모든 작업에서 쓰는 진실 공급원 — 가 실제 코드와 맞지 않게 된다는 점입니다. 관련 단위로 AIDLC가 Code Generation을 다시 돌리거나 세션을 재개하거나 동료가 이어 받을 때 불일치가 혼란과 재작업을 만듭니다.

워크숍에서 한 팀이 이렇게 말했습니다.

> "코드를 직접 고치지 마세요. 이슈를 발견하면 AIDLC로 돌아가 이렇게 말하세요: 이슈 X를 발견했다. 설계를 검토하고 고치는 계획을 세워라. 설계에 영향이 있으면 설계를 갱신한 뒤 코드를 갱신하라."

**규칙: 먼저 설계를 갱신하고, 그다음 코드를 생성하세요.**

---

### 올바른 변경 방법

버그를 찾았든 설계 결정을 바꿨든 새 요구를 받았든 흐름은 같습니다.

**1단계 — 아무것도 건드리지 않고 이슈를 설명:**

```
Do not update any documents yet. I have discovered issue [X].
Review the design and help me understand where this needs to be addressed.
```

**2단계 — 설계 문서를 고침:**

```
Please update [specific design document] to reflect [the fix].
Then check whether any upstream documents — requirements, user stories —
also need to be updated.
```

**3단계 — 영향받는 코드를 다시 생성:**

```
The design for [unit name] has been updated. Please re-run code generation
for the affected files only.
```

파일을 직접 고치는 것보다 몇 분 더 걸리지만 문서를 맞추고 감사 추적을 유지하며 실제로 무엇이 만들어졌는지 팀이 맞출 수 있습니다.

---

### "그냥 파일만 고치자"는 유혹이 들 때

**"한 줄 고치는 것뿐이야."**

설계를 건너뛴 한 줄 수정도 드리프트를 만듭니다. 해당 설계 문서에 수정을 적고 AIDLC가 적용하게 하세요.

```
In [functional-design.md for unit X], update [method or rule] to [the fix].
Then regenerate [the affected file].
```

**"탐색 중이야 — 아직 확정 아니야."**

탐색은 바로 "Do not update any documents"가 있는 이유입니다. 채팅에서는 자유롭게 탐색하세요. 준비됐을 때만 커밋합니다.

**"지금 팀을 막아 둘 수 없어."**

가끔은 빨리 움직여야 합니다. 직접 수정했다면 감사 추적이 정확하도록 솔직히 기록하세요.

```
We made a temporary direct edit to [file] to unblock the team.
The fix was [description]. Please update [design document] to reflect this
and verify no other documents are inconsistent.
```

---

### 드리프트를 막는 상시 규칙

Construction 단계 시작 시 두 가지 상시 지시를 두면 매번 묻지 않아도 일찍 문제를 잡을 수 있습니다.

**갱신마다 역전파:**

```
Every time you update a document, check whether the change impacts the
requirements document and user stories, and prompt me if it does.
```

**코드 결정마다 설계 우선:**

```
When you make a design decision during code generation, always make sure
the documentation reflects this change before proceeding.
```

Construction 시작 시 한 번만 두면 단계 전체에 적용됩니다.

---

### 리포트를 aidlc-docs 밖에 두기

실무 팁: AIDLC에게 사람이 읽는 리포트 — 아키텍처 다이어그램, 컴포넌트 요약, 이해관계자 발표 자료 — 를 만들게 할 때 `aidlc-docs/`에 저장하지 마세요. 이후 단계에서 아티팩트로 로드되어 토큰이 불어나고 무엇이 권위 있는 설계 입력인지 AI가 혼동할 수 있습니다.

별도 `reports/` 폴더를 쓰고, 더 깔끔한 출력을 위해 전용 리포트 명세 파일과 새 맥락에서 리포트를 생성하세요.

```
Pause the process. Start a new context. Read [report specification markdown file]
and produce the report based on the current state of the AIDLC artifacts.
Save the output to a reports/ folder, not aidlc-docs/.
```

---

*입력 문서 준비 가이드는 [writing-inputs/inputs-quickstart.md](writing-inputs/inputs-quickstart.md)를 참고하세요.*
