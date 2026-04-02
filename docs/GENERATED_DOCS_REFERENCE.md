# 생성된 aidlc-docs/ 참조

AI-DLC 워크플로를 실행하면 모든 문서 산출물이 워크스페이스 루트의 `aidlc-docs/` 디렉터리 안에 생성됩니다. 실제로 만들어지는 파일은 프로젝트 유형(그린필드 vs 브라운필드), 복잡도, 워크플로가 실행하거나 건너뛴 단계에 따라 달라집니다.

아래는 모든 단계와 스테이지에 걸쳐 가능한 모든 파일을 보여 주는 완전한 구조입니다. 조건부 파일은 나타나는 시점을 설명하는 메모로 표시했습니다.

```
aidlc-docs/
├── aidlc-state.md                                          # Workflow state tracker — project info, stage progress, current status
├── audit.md                                                # Complete audit trail — every user input, AI response, and approval with timestamps
│
├── inception/                                              # 🔵 INCEPTION PHASE — determines WHAT to build and WHY
│   ├── plans/
│   │   ├── execution-plan.md                               # Workflow visualization and phase execution decisions (always created)
│   │   ├── story-generation-plan.md                        # Story development methodology and questions (if User Stories executes)
│   │   ├── user-stories-assessment.md                      # Assessment of whether user stories add value (if User Stories executes)
│   │   ├── application-design-plan.md                      # Component and service design plan with questions (if Application Design executes)
│   │   └── unit-of-work-plan.md                            # System decomposition plan with questions (if Units Generation executes)
│   │
│   ├── reverse-engineering/                                # Created only for brownfield projects (existing codebase detected)
│   │   ├── business-overview.md                            # Business context, transactions, and dictionary
│   │   ├── architecture.md                                 # System architecture diagrams, component descriptions, data flow
│   │   ├── code-structure.md                               # Build system, key classes/modules, design patterns, file inventory
│   │   ├── api-documentation.md                            # REST APIs, internal APIs, and data models
│   │   ├── component-inventory.md                          # Inventory of all packages by type (application, infrastructure, shared, test)
│   │   ├── technology-stack.md                             # Languages, frameworks, infrastructure, build tools, testing tools
│   │   ├── dependencies.md                                 # Internal and external dependency graphs and relationships
│   │   ├── code-quality-assessment.md                      # Test coverage, code quality indicators, technical debt, patterns
│   │   └── reverse-engineering-timestamp.md                # Analysis metadata and artifact checklist
│   │
│   ├── requirements/
│   │   ├── requirements.md                                 # Functional and non-functional requirements with intent analysis (always created)
│   │   └── requirement-verification-questions.md           # Clarifying questions with [Answer]: tags for user input (always created)
│   │
│   ├── user-stories/                                       # Created only if User Stories stage executes
│   │   ├── stories.md                                      # User stories following INVEST criteria with acceptance criteria
│   │   └── personas.md                                     # User archetypes, characteristics, and persona-to-story mappings
│   │
│   └── application-design/                                 # Created only if Application Design and/or Units Generation execute
│       ├── application-design.md                           # Consolidated design document (if Application Design executes)
│       ├── components.md                                   # Component definitions, responsibilities, and interfaces
│       ├── component-methods.md                            # Method signatures, purposes, and input/output types
│       ├── services.md                                     # Service definitions, responsibilities, and orchestration patterns
│       ├── component-dependency.md                         # Dependency matrix and communication patterns between components
│       ├── unit-of-work.md                                 # Unit definitions and responsibilities (if Units Generation executes)
│       ├── unit-of-work-dependency.md                      # Dependency matrix between units (if Units Generation executes)
│       └── unit-of-work-story-map.md                       # Mapping of user stories to units (if Units Generation executes)
│
├── construction/                                           # 🟢 CONSTRUCTION PHASE — determines HOW to build it
│   ├── plans/
│   │   ├── {unit-name}-functional-design-plan.md           # Business logic design plan with questions (per unit, if Functional Design executes)
│   │   ├── {unit-name}-nfr-requirements-plan.md            # NFR assessment plan with questions (per unit, if NFR Requirements executes)
│   │   ├── {unit-name}-nfr-design-plan.md                  # NFR design patterns plan with questions (per unit, if NFR Design executes)
│   │   ├── {unit-name}-infrastructure-design-plan.md       # Infrastructure mapping plan with questions (per unit, if Infrastructure Design executes)
│   │   └── {unit-name}-code-generation-plan.md             # Detailed code generation steps with checkboxes (per unit, always created)
│   │
│   ├── {unit-name}/                                        # Per-unit artifacts — one directory per unit of work
│   │   ├── functional-design/                              # Created only if Functional Design executes for this unit
│   │   │   ├── business-logic-model.md                     # Detailed business logic and algorithms
│   │   │   ├── business-rules.md                           # Business rules, validation logic, and constraints
│   │   │   ├── domain-entities.md                          # Domain models with entities and relationships
│   │   │   └── frontend-components.md                      # UI component hierarchy, props, state, interactions (if unit has frontend)
│   │   │
│   │   ├── nfr-requirements/                               # Created only if NFR Requirements executes for this unit
│   │   │   ├── nfr-requirements.md                         # Scalability, performance, availability, security requirements
│   │   │   └── tech-stack-decisions.md                     # Technology choices and rationale
│   │   │
│   │   ├── nfr-design/                                     # Created only if NFR Design executes for this unit
│   │   │   ├── nfr-design-patterns.md                      # Resilience, scalability, performance, and security patterns
│   │   │   └── logical-components.md                       # Logical infrastructure components (queues, caches, etc.)
│   │   │
│   │   ├── infrastructure-design/                          # Created only if Infrastructure Design executes for this unit
│   │   │   ├── infrastructure-design.md                    # Cloud service mappings and infrastructure components
│   │   │   └── deployment-architecture.md                  # Deployment model, networking, scaling configuration
│   │   │
│   │   └── code/                                           # Markdown summaries of generated code (always created per unit)
│   │       └── *.md                                        # Code generation summaries (actual code goes to workspace root)
│   │
│   ├── shared-infrastructure.md                            # Shared infrastructure across units (if applicable)
│   │
│   └── build-and-test/                                     # Always created after all units complete code generation
│       ├── build-instructions.md                           # Prerequisites, build steps, troubleshooting
│       ├── unit-test-instructions.md                       # Unit test execution commands and expected results
│       ├── integration-test-instructions.md                # Integration test scenarios, setup, and execution
│       ├── performance-test-instructions.md                # Load/stress test configuration and execution (if performance NFRs exist)
│       ├── contract-test-instructions.md                   # API contract validation between services (if microservices)
│       ├── security-test-instructions.md                   # Vulnerability scanning and security testing (if security NFRs exist)
│       ├── e2e-test-instructions.md                        # End-to-end user workflow testing (if applicable)
│       └── build-and-test-summary.md                       # Overall build status, test results, and readiness assessment
│
└── operations/                                             # 🟡 OPERATIONS PHASE — placeholder for future expansion
```

## 참고

- `{unit-name}`은 실제 작업 단위 이름으로 바뀝니다(예: `api-service`, `frontend-app`, `data-processor`). 단일 작업 단위 프로젝트에서는 보통 `construction/` 아래에 작업 단위 디렉터리가 하나입니다.
- 더 단순한 단일 작업 단위 프로젝트에서는 모델이 이름을 단순화할 수 있습니다. 예: `construction/plans/code-generation-plan.md`처럼 `construction/plans/{unit-name}-code-generation-plan.md` 대신 쓰거나, 개별 컴포넌트 파일 없이 `application-design.md` 하나로 통합할 수 있습니다.
- `build-and-test/` 디렉터리에는 항상 `build-and-test-summary.md`가 포함됩니다. 개별 안내 파일(`build-instructions.md`, `unit-test-instructions.md`, `integration-test-instructions.md` 등)은 프로젝트 복잡도와 테스트 필요에 따라 생성됩니다.
- `inception/plans/`와 `construction/plans/` 안의 계획에는 사용자가 입력하는 `[Answer]:` 태그와 실행 진행을 추적하는 `[ ]`/`[x]` 체크박스가 있습니다.
- 애플리케이션 코드는 `aidlc-docs/` 안에 두지 않고 워크스페이스 루트에 둡니다. 여기에는 마크다운 문서만 있습니다.
- `audit.md`는 추가 전용이며 ISO 8601 타임스탬프와 함께 모든 상호작용을 기록합니다.
- `aidlc-state.md`는 완료·건너뜀·진행 중인 스테이지와 확장 구성을 추적합니다.
