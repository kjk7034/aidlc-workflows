# 비전 문서 가이드

## 목적

비전 문서는 AI-DLC 워크플로에 들어가기 전에 프로젝트의 **비즈니스 목표**, **목표 성과**, **범위 경계**를 정의합니다. Inception Phase의 주 입력으로, AI 모델과 팀이 프로젝트가 무엇을 달성하려 하는지와 왜 중요한지 공유된 이해를 갖게 합니다.

잘 쓴 비전 문서는 Requirements Analysis의 모호함을 줄이고 사용자 스토리 품질을 높이며 Construction에서 범위 확대를 막습니다.

## 비전 문서를 쓰는 시점

- 새 프로젝트나 큰 이니셔티브를 시작하기 전
- 새 제품·기능 묶음·플랫폼을 제안할 때
- 기존 제품을 새 방향으로 전환할 때
- 개발 전에 여러 이해관계자가 목표를 맞춰야 할 때

## 문서 구조

### 1. Executive Summary

프로젝트의 핵심을 담은 짧은 단락(3~5문장)입니다. 이 절만 읽어도 프로젝트가 무엇인지, 누구를 위한지, 왜 존재하는지 알 수 있어야 합니다.

**템플릿:**

```markdown
## Executive Summary

[Project Name] is a [type of system/product] that enables [target users] to [core capability].
It addresses [business problem or opportunity] by [approach or differentiation].
The expected outcome is [measurable business result].
```

**예시:**

```markdown
## Executive Summary

OrderFlow is a web-based order management platform that enables mid-size retailers to
track inventory, process customer orders, and manage supplier relationships in a single
interface. It addresses the fragmented tooling problem that causes fulfillment delays
and inventory mismatches. The expected outcome is a 30% reduction in order processing
time and elimination of manual inventory reconciliation.
```

---

### 2. Business Context

비즈니스 환경, 해결하는 문제, 지금 해결해야 하는 이유를 설명합니다.

**포함할 절:**

```markdown
## Business Context

### Problem Statement
[What specific business problem or pain point does this project address?
Be concrete. Avoid vague statements like "improve efficiency."]

### Business Drivers
[Why is this project being pursued now? What market conditions, competitive
pressures, regulatory changes, or internal needs make this timely?]

### Target Users and Stakeholders
[Who will use the system? Who has a stake in its success?
List user types with a brief description of each.]

| User Type | Description | Primary Need |
|-----------|-------------|--------------|
| [Role]    | [Who they are] | [What they need from this system] |

### Business Constraints
[Budget limits, regulatory requirements, organizational policies, timeline
pressures, or other non-negotiable boundaries.]

### Success Metrics
[How will the business measure whether this project succeeded?
Use specific, measurable criteria.]

| Metric | Current State | Target State | Measurement Method |
|--------|--------------|--------------|-------------------|
| [Metric name] | [Baseline] | [Goal] | [How measured] |
```

---

### 3. Full Scope Vision

제품 또는 시스템의 **완전한 장기 비전**을 설명합니다. 의도적으로 지향적이며, 처음에 만들 것만이 아니라 프로젝트가 될 수 있는 전부를 다룹니다.

**포함할 절:**

```markdown
## Full Scope Vision

### Product Vision Statement
[A single sentence or short paragraph that captures the long-term aspirational
state of the product. What does the world look like when this product is fully
realized?]

### Feature Areas
[Organize the full feature set into logical groups. For each area, describe
what the system will do at full maturity.]

#### Feature Area 1: [Name]
- **Description**: [What this area covers]
- **Key Capabilities**:
  - [Capability 1]
  - [Capability 2]
  - [Capability 3]
- **User Value**: [Why this matters to users]

#### Feature Area 2: [Name]
[Same structure]

### Integration Points
[What external systems, APIs, or data sources will the full system integrate
with at maturity?]

- [System/Service] - [Purpose of integration]

### User Journeys (Full Vision)
[Describe 2-3 end-to-end user journeys that represent the complete product
experience. These should reflect the full scope, not the MVP.]

#### Journey 1: [Name]
1. [Step]
2. [Step]
3. [Step]
**Outcome**: [What the user achieves]

### Scalability and Growth
[How is the product expected to grow? New markets, user types, geographies,
data volumes, or feature categories?]

### Long-Term Roadmap (Optional)
[If known, outline the high-level phases or milestones beyond the MVP.
This is directional, not committal.]

| Phase | Focus | Timeframe (if known) |
|-------|-------|---------------------|
| MVP | [Core scope] | [Target] |
| Phase 2 | [Expansion area] | [Target] |
| Phase 3 | [Further expansion] | [Target] |
```

---

### 4. MVP Scope

**최소 기능 제품(MVP)** 을 정의합니다. 측정 가능한 가치를 내고 핵심 비즈니스 가설을 검증하는 최소 기능 집합입니다. 여기 나열된 것은 제품을 출시하거나 평가하기 전에 모두 구현되어야 합니다.

**포함할 절:**

```markdown
## MVP Scope

### MVP Objective
[What is the single most important thing the MVP must prove or deliver?
Keep this to 1-2 sentences.]

### MVP Success Criteria
[How will you know the MVP succeeded? These should be testable and specific.]

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

### Features In Scope (MVP)
[List every feature that is included in the MVP. Be explicit. If it is not
listed here, it is not in the MVP.]

| Feature | Description | Priority | Rationale for Inclusion |
|---------|-------------|----------|------------------------|
| [Feature name] | [Brief description] | Must Have | [Why it cannot be deferred] |

### Features Explicitly Out of Scope (MVP)
[List features from the Full Scope Vision that are deliberately excluded
from the MVP. State why each is deferred. This prevents scope creep.]

| Feature | Reason for Deferral | Target Phase |
|---------|-------------------|--------------|
| [Feature name] | [Why it can wait] | [Phase 2/3/TBD] |

### MVP User Journeys
[Describe the user journeys that the MVP must support. These are subsets
or simplified versions of the Full Vision journeys.]

#### Journey 1: [Name]
1. [Step]
2. [Step]
3. [Step]
**Outcome**: [What the user achieves]
**Limitation vs Full Vision**: [What is simplified or missing compared to full scope]

### MVP Constraints and Assumptions
[What assumptions is the MVP built on? What known limitations are accepted?]

- **Assumption**: [Statement] - **Risk if wrong**: [Consequence]
- **Accepted Limitation**: [What is intentionally limited and why]

### MVP Definition of Done
[What must be true for the MVP to be considered complete and ready for
evaluation or launch?]

- [ ] All "Must Have" features implemented and tested
- [ ] [Additional criteria specific to this project]
- [ ] [Deployment or accessibility requirement]
- [ ] [Stakeholder sign-off requirement]
```

---

### 5. Risks and Dependencies

```markdown
## Risks and Dependencies

### Key Risks
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [Risk description] | High/Medium/Low | High/Medium/Low | [Mitigation strategy] |

### External Dependencies
[List anything outside the team's control that the project depends on.]

- [Dependency] - [Owner] - [Status]

### Open Questions
[List unresolved questions that need answers before or during development.
These feed directly into the Requirements Analysis clarifying questions.]

- [ ] [Question]
- [ ] [Question]
```

---

## 작성 지침

### 할 것

- 구체적이고 측정 가능하게 쓰세요. "주문 처리 시간 30% 단축"은 "더 빠르게"보다 낫습니다.
- 전체 비전과 MVP를 분리하세요. 섞으면 범위가 불어납니다.
- "범위 밖" 목록을 포함하세요. "범위 안"만큼 가치가 있습니다.
- 경영진이 아니라 팀을 위해 쓰세요. 마케팅 언어는 피하세요.
- 가정을 명시해 반박할 수 있게 하세요.
- 실제로 검증할 수 있는 성공 기준을 넣으세요.

### 하지 말 것

- "world-class," "seamless," "intuitive," "best-in-class" 같은 모호한 표현.
- 기술이나 구현 세부를 나열하지 마세요. 그것은 Technical Environment Document에 둡니다.
- MVP 절을 빼지 마세요. 모든 프로젝트에 시작 경계가 필요합니다.
- 기능과 사용자 여정을 섞지 마세요. 기능은 시스템이 하는 일이고, 여정은 사용자가 경험하는 방식입니다.
- 독자가 비즈니스 맥락을 안다고 가정하지 마세요. 당연해 보여도 Problem Statement를 쓰세요.

---

## 이 문서가 AI-DLC에 어떻게 쓰이는지

| 비전 문서 절 | AI-DLC 단계 | 사용 방식 |
|------------------------|--------------|----------------|
| Executive Summary | Workspace Detection | 프로젝트 분류를 위한 초기 맥락 |
| Business Context | Requirements Analysis | 명확화 질문과 요구 사항 깊이를 이끎 |
| Full Scope Vision | User Stories, Application Design | 페르소나·컴포넌트 식별에 반영 |
| MVP Scope | Workflow Planning | 실행할 단계와 범위 경계 결정 |
| Features In/Out of Scope | Code Generation | 이번 반복에서 무엇을 만들지 정의 |
| Risks and Dependencies | All stages | 위험 평가와 오류 처리에 반영 |
| Open Questions | Requirements Analysis | 질문 파일의 명확화 질문이 됨 |
