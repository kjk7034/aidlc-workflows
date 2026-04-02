# Reverse Engineering

**목적**: 기존 코드베이스를 분석하고 포괄적인 설계 산출물을 생성합니다

**실행 시**: Brownfield 프로젝트(워크스페이스에 기존 코드 발견)

**건너뛸 시**: Greenfield 프로젝트(기존 코드 없음)

**재실행 동작**: 재실행은 workspace-detection.md가 제어합니다. 기존 reverse engineering 산출물이 있고 여전히 최신이면 로드되고 reverse engineering은 건너뜁니다. 산출물이 오래되었거나(코드베이스의 마지막 유의미한 수정보다 이전) 사용자가 명시적으로 재실행을 요청하면, 산출물이 현재 코드 상태를 반영하도록 reverse engineering이 다시 실행됩니다

## Step 1: 다중 패키지 탐색

### 1.1 워크스페이스 스캔
- 모든 패키지(언급된 것만이 아님)
- 설정 파일을 통한 패키지 관계
- 패키지 유형: Application, CDK/Infrastructure, Models, Clients, Tests

### 1.2 비즈니스 맥락 이해
- 시스템이 전반적으로 구현하는 핵심 비즈니스
- 각 패키지의 비즈니스 개요
- 시스템에 구현된 비즈니스 트랜잭션 목록

### 1.3 인프라 탐색
- CDK 패키지(CDK 의존성이 있는 package.json)
- Terraform(.tf 파일)
- CloudFormation(.yaml/.json 템플릿)
- 배포 스크립트

### 1.4 빌드 시스템 탐색
- 빌드 시스템: Brazil, Maven, Gradle, npm
- 빌드 시스템 선언용 설정 파일
- 패키지 간 빌드 의존성

### 1.5 서비스 아키텍처 탐색
- Lambda 함수(핸들러, 트리거)
- 컨테이너 서비스(Docker/ECS 설정)
- API 정의(Smithy 모델, OpenAPI 스펙)
- 데이터 저장소(DynamoDB, S3 등)

### 1.6 코드 품질 분석
- 프로그래밍 언어와 프레임워크
- 테스트 커버리지 지표
- 린트 설정
- CI/CD 파이프라인

## Step 2: 비즈니스 개요 문서 생성

`aidlc-docs/inception/reverse-engineering/business-overview.md` 생성:

```markdown
# Business Overview

## Business Context Diagram
[비즈니스 맥락을 보여주는 Mermaid 다이어그램]

## Business Description
- **Business Description**: [시스템이 하는 일에 대한 전체 비즈니스 설명]
- **Business Transactions**: [시스템이 구현하는 비즈니스 트랜잭션 목록과 설명]
- **Business Dictionary**: [시스템이 따르는 비즈니스 용어 사전과 의미]

## Component Level Business Descriptions
### [패키지/컴포넌트 이름]
- **Purpose**: [비즈니스 관점에서 하는 일]
- **Responsibilities**: [주요 책임]
```

## Step 3: 아키텍처 문서 생성

`aidlc-docs/inception/reverse-engineering/architecture.md` 생성:

```markdown
# System Architecture

## System Overview
[시스템에 대한 고수준 설명]

## Architecture Diagram
[모든 패키지, 서비스, 데이터 저장소, 관계를 보여주는 Mermaid 다이어그램]

## Component Descriptions
### [패키지/컴포넌트 이름]
- **Purpose**: [하는 일]
- **Responsibilities**: [주요 책임]
- **Dependencies**: [의존 대상]
- **Type**: [Application/Infrastructure/Model/Client/Test]

## Data Flow
[주요 워크플로의 Mermaid 시퀀스 다이어그램]

## Integration Points
- **External APIs**: [목적과 함께 목록]
- **Databases**: [목적과 함께 목록]
- **Third-party Services**: [목적과 함께 목록]

## Infrastructure Components
- **CDK Stacks**: [목적과 함께 목록]
- **Deployment Model**: [설명]
- **Networking**: [VPC, 서브넷, 보안 그룹]
```

## Step 4: 코드 구조 문서 생성

`aidlc-docs/inception/reverse-engineering/code-structure.md` 생성:

```markdown
# Code Structure

## Build System
- **Type**: [Maven/Gradle/npm/Brazil]
- **Configuration**: [주요 빌드 파일과 설정]

## Key Classes/Modules
[Mermaid 클래스 다이어그램 또는 모듈 계층]

### Existing Files Inventory
[수정 후보가 되는 모든 소스 파일과 목적 — brownfield 프로젝트에서]

**예시 형식**:
- `[path/to/file]` - [목적/책임]

## Design Patterns
### [패턴 이름]
- **Location**: [사용 위치]
- **Purpose**: [사용 이유]
- **Implementation**: [구현 방식]

## Critical Dependencies
### [의존성 이름]
- **Version**: [버전 번호]
- **Usage**: [사용 방식과 위치]
- **Purpose**: [필요 이유]
```

## Step 5: API 문서 생성

`aidlc-docs/inception/reverse-engineering/api-documentation.md` 생성:

```markdown
# API Documentation

## REST APIs
### [엔드포인트 이름]
- **Method**: [GET/POST/PUT/DELETE]
- **Path**: [/api/path]
- **Purpose**: [하는 일]
- **Request**: [요청 형식]
- **Response**: [응답 형식]

## Internal APIs
### [인터페이스/클래스 이름]
- **Methods**: [시그니처와 함께 목록]
- **Parameters**: [매개변수 설명]
- **Return Types**: [반환 타입 설명]

## Data Models
### [모델 이름]
- **Fields**: [필드 설명]
- **Relationships**: [관련 모델]
- **Validation**: [검증 규칙]
```

## Step 6: 컴포넌트 인벤토리 생성

`aidlc-docs/inception/reverse-engineering/component-inventory.md` 생성:

```markdown
# Component Inventory

## Application Packages
- [패키지 이름] - [목적]

## Infrastructure Packages
- [패키지 이름] - [CDK/Terraform] - [목적]

## Shared Packages
- [패키지 이름] - [Models/Utilities/Clients] - [목적]

## Test Packages
- [패키지 이름] - [Integration/Load/Unit] - [목적]

## Total Count
- **Total Packages**: [개수]
- **Application**: [개수]
- **Infrastructure**: [개수]
- **Shared**: [개수]
- **Test**: [개수]
```

## Step 7: 기술 스택 문서 생성

`aidlc-docs/inception/reverse-engineering/technology-stack.md` 생성:

```markdown
# Technology Stack

## Programming Languages
- [언어] - [버전] - [용도]

## Frameworks
- [프레임워크] - [버전] - [목적]

## Infrastructure
- [서비스] - [목적]

## Build Tools
- [도구] - [버전] - [목적]

## Testing Tools
- [도구] - [버전] - [목적]
```

## Step 8: 의존성 문서 생성

`aidlc-docs/inception/reverse-engineering/dependencies.md` 생성:

```markdown
# Dependencies

## Internal Dependencies
[패키지 의존성을 보여주는 Mermaid 다이어그램]

### [패키지 A]는 [패키지 B]에 의존
- **Type**: [Compile/Runtime/Test]
- **Reason**: [의존이 존재하는 이유]

## External Dependencies
### [의존성 이름]
- **Version**: [버전]
- **Purpose**: [사용 이유]
- **License**: [라이선스 유형]
```

## Step 9: 코드 품질 평가 생성

`aidlc-docs/inception/reverse-engineering/code-quality-assessment.md` 생성:

```markdown
# Code Quality Assessment

## Test Coverage
- **Overall**: [비율 또는 Good/Fair/Poor/None]
- **Unit Tests**: [상태]
- **Integration Tests**: [상태]

## Code Quality Indicators
- **Linting**: [Configured/Not configured]
- **Code Style**: [Consistent/Inconsistent]
- **Documentation**: [Good/Fair/Poor]

## Technical Debt
- [이슈 설명과 위치]

## Patterns and Anti-patterns
- **Good Patterns**: [목록]
- **Anti-patterns**: [위치와 함께 목록]
```

## Step 10: 타임스탬프 파일 생성

`aidlc-docs/inception/reverse-engineering/reverse-engineering-timestamp.md` 생성:

```markdown
# Reverse Engineering Metadata

**Analysis Date**: [ISO 타임스탬프]
**Analyzer**: AI-DLC
**Workspace**: [워크스페이스 경로]
**Total Files Analyzed**: [개수]

## Artifacts Generated
- [x] architecture.md
- [x] code-structure.md
- [x] api-documentation.md
- [x] component-inventory.md
- [x] technology-stack.md
- [x] dependencies.md
- [x] code-quality-assessment.md
```

## Step 11: 상태 추적 갱신

`aidlc-docs/aidlc-state.md` 갱신:

```markdown
## Reverse Engineering Status
- [x] Reverse Engineering - [타임스탬프]에 완료
- **Artifacts Location**: aidlc-docs/inception/reverse-engineering/
```

## Step 12: 사용자에게 완료 메시지 제시

```markdown
# 🔍 Reverse Engineering Complete

[분석에서 얻은 주요 결과에 대한 AI 생성 요약 — 글머리 기호]

> **📋 <u>**검토 필요:**</u>**  
> 다음 경로의 reverse engineering 산출물을 검토하세요: `aidlc-docs/inception/reverse-engineering/`

> **🚀 <u>**다음 단계**</u>**
>
> **선택할 수 있는 항목:**
>
> 🔧 **변경 요청** - 필요 시 reverse engineering 분석 수정을 요청합니다
> ✅ **승인 후 계속** - 분석을 승인하고 **Requirements Analysis**로 진행합니다
```

## Step 13: 사용자 승인 대기

- **필수**: 사용자가 명시적으로 승인할 때까지 진행하지 않음
- **필수**: 사용자 응답을 audit.md에 완전한 원문으로 기록
