# AI-DLC 용어 사전

## 핵심 용어

### Phase vs Stage

**Phase**: AI-DLC의 세 가지 상위 생명주기 단계 중 하나
- 🔵 **INCEPTION PHASE** — 계획 및 아키텍처(WHAT과 WHY)
- 🟢 **CONSTRUCTION PHASE** — 설계, 구현 및 테스트(HOW)
- 🟡 **OPERATIONS PHASE** — 배포 및 모니터링(향후 확장)

**Stage**: 단계 내 개별 워크플로 활동
- 예: Context Assessment 단계, Requirements Assessment 단계, Code Generation 단계
- 각 단계에는 전제 조건, 단계, 산출물이 있음
- 단계는 ALWAYS-EXECUTE이거나 CONDITIONAL일 수 있음

**사용 예**:
- ✅ "CONSTRUCTION phase에는 7개의 stage가 있다"
- ✅ "Code Generation stage는 항상 실행된다"
- ✅ "INCEPTION phase에서 Requirements Assessment **stage**를 실행 중이다"
- ❌ "Requirements Assessment **phase**" (올바른 표현은 "stage")
- ❌ "CONSTRUCTION **stage**" (올바른 표현은 "phase")

## 세 단계 생명주기

### INCEPTION PHASE
**목적**: 계획 및 아키텍처 결정  
**초점**: 무엇을 만들지(WHAT), 왜 만들지(WHY)  
**위치**: `inception/` 디렉터리

**Stage**:
- Workspace Detection (항상)
- Reverse Engineering (조건부 — Brownfield만)
- Requirements Analysis (항상 — 적응형 깊이)
- User Stories (조건부)
- Workflow Planning (항상)
- Application Design (조건부)
- Design - Units Planning/Generation (조건부)

**산출물**: 요구사항, user stories, 아키텍처 결정, 단위 정의

### CONSTRUCTION PHASE
**목적**: 상세 설계 및 구현  
**초점**: 어떻게 만들지(HOW)  
**위치**: `construction/` 디렉터리

**Stage**:
- Functional Design (조건부, 단위별)
- NFR Requirements (조건부, 단위별)
- NFR Design (조건부, 단위별)
- Infrastructure Design (조건부, 단위별)
- Code Generation (항상) — Part 1: Planning, Part 2: Generation 포함
- Build and Test (항상)

**산출물**: 설계 산출물, NFR 구현, 코드, 테스트

### OPERATIONS PHASE
**목적**: 배포 및 운영 준비  
**초점**: 배포·실행 방법  
**위치**: `operations/` 디렉터리

**Stage**:
- Operations (PLACEHOLDER)

**산출물**: 빌드 지침, 배포 가이드, 모니터링 설정, 검증 절차

---

## 워크플로 단계

### 항상 실행되는 Stage
- **Workspace Detection**: 워크스페이스 상태와 프로젝트 유형의 초기 분석
- **Requirements Analysis**: 요구사항 수집(복잡도에 따라 깊이 변동)
- **Workflow Planning**: 실행할 phase를 정하는 실행 계획 작성
- **Code Generation**: 두 부분으로 구성된 단일 stage — Part 1(Planning)은 상세 구현 계획, Part 2(Generation)은 계획과 이전 산출물을 바탕으로 실제 코드 생성
- **Build and Test**: 모든 단위 빌드 및 포괄적 테스트 실행

### 조건부 Stage
- **Reverse Engineering**: 기존 코드베이스 분석(brownfield만)
- **User Stories**: user stories 및 페르소나 생성(Story Planning 및 Story Generation 포함)
- **Application Design**: 애플리케이션 컴포넌트, 메서드, 비즈니스 규칙, 서비스 설계
- **Design**: 시스템 컴포넌트 설계(Units Planning, Units Generation, 단위별 설계 포함)
- **Functional Design**: 기술 중립적 비즈니스 로직 설계(단위별)
- **NFR Requirements**: NFR 결정 및 기술 스택 선택(단위별)
- **NFR Design**: NFR 패턴 및 논리 컴포넌트 반영(단위별)
- **Infrastructure Design**: 실제 인프라 서비스에 매핑(단위별)

## Application Design 용어

- **Component**: 명확한 책임을 가진 기능 단위
- **Method**: 정의된 비즈니스 규칙을 가진 컴포넌트 내 함수 또는 연산
- **Business Rule**: 메서드 동작과 검증을 관장하는 로직
- **Service**: 컴포넌트 간 비즈니스 로직을 조율하는 오케스트레이션 레이어
- **Component Dependency**: 컴포넌트 간 관계와 통신 패턴

## 아키텍처 용어(인프라)

### Unit of Work
개발 목적으로 user stories를 묶는 논리적 그룹. 계획과 분해 시 사용하는 용어입니다.

**사용 예**: "시스템을 작업 단위로 분해해야 한다"

### Service
마이크로서비스 아키텍처에서 독립 배포 가능한 컴포넌트. 각 서비스는 별도의 작업 단위입니다.

**사용 예**: "Payment Service가 모든 결제 처리를 담당한다"

### Module
단일 서비스나 모놀리스 내 기능의 논리적 묶음. 모듈은 독립 배포되지 않습니다.

**사용 예**: "User Service 내 인증 모듈"

### Component
서비스나 모듈 내 재사용 가능한 빌딩 블록. 특정 기능을 제공하는 클래스, 함수 또는 패키지입니다.

**사용 예**: "EmailValidator 컴포넌트가 이메일 주소를 검증한다"

## 용어 사용 지침

### 각 용어를 쓸 때

**Unit of Work**:
- Units Planning 및 Units Generation stage에서
- 시스템 분해를 논할 때
- 계획 문서와 논의에서
- 예: "이것을 작업 단위로 어떻게 나눌까?"

**Service**:
- 독립 배포 가능한 컴포넌트를 가리킬 때
- 마이크로서비스 아키텍처 맥락에서
- 배포·인프라 논의에서
- 예: "Order Service를 ECS에 배포할 것이다"

**Module**:
- 서비스 내 논리적 묶음을 가리킬 때
- 모놀리스 아키텍처 맥락에서
- 내부 구조를 논할 때
- 예: "보고 모듈이 모든 보고서를 생성한다"

**Component**:
- 특정 클래스, 함수, 패키지를 가리킬 때
- 설계·구현 논의에서
- 재사용 빌딩 블록을 논할 때
- 예: "DatabaseConnection 컴포넌트가 연결을 관리한다"

## Stage 용어

### Planning vs Generation
- **Planning**: 실행을 안내하는 질문과 체크박스가 있는 계획 작성
- **Generation**: 계획을 실행해 산출물 생성

예:
- Story Planning → Story Generation
- Units Planning → Units Generation
- Unit Design Planning → Unit Design Generation
- NFR Planning → NFR Generation
- Code Generation Part 1 (Planning) → Code Generation Part 2 (Generation)

### 깊이 수준
- **Minimal**: 단순 변경을 위한 빠르고 집중된 실행
- **Standard**: 일반 프로젝트를 위한 표준 산출물이 있는 보통 깊이
- **Comprehensive**: 복잡·고위험 프로젝트를 위한 전체 깊이와 모든 산출물

## 산출물 유형

### Plans
실행을 안내하는 체크박스와 질문이 있는 문서.
- `aidlc-docs/plans/`에 위치
- 예: `story-generation-plan.md`, `unit-of-work-plan.md`

### Artifacts
계획 실행으로 생성된 출력.
- `aidlc-docs/`의 여러 하위 디렉터리에 위치
- 예: `requirements.md`, `stories.md`, `design.md`

### State Files
워크플로 진행과 상태를 추적하는 파일.
- `aidlc-state.md`: 전체 워크플로 상태
- `audit.md`: 모든 상호작용의 완전한 감사 추적

## 일반 약어

- **AI-DLC**: AI-Driven Development Life Cycle
- **NFR**: Non-Functional Requirements
- **UOW**: Unit of Work
- **API**: Application Programming Interface
- **CDK**: Cloud Development Kit (AWS)
