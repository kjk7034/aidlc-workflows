# Workspace Detection

**목적**: 워크스페이스 상태를 파악하고 기존 AI-DLC 프로젝트 여부를 확인합니다

## Step 1: 기존 AI-DLC 프로젝트 확인

`aidlc-docs/aidlc-state.md` 존재 여부 확인:
- **있으면**: 마지막 phase부터 재개(이전 phase의 컨텍스트 로드)
- **없으면**: 신규 프로젝트 평가 계속

## Step 2: 기존 코드를 위해 워크스페이스 스캔

**워크스페이스에 기존 코드가 있는지 판단:**
- 소스 코드 파일 스캔(.java, .py, .js, .ts, .jsx, .tsx, .kt, .kts, .scala, .groovy, .go, .rs, .rb, .php, .c, .h, .cpp, .hpp, .cc, .cs, .fs 등)
- 빌드 파일 확인(pom.xml, package.json, build.gradle 등)
- 프로젝트 구조 단서 탐색
- 워크스페이스 루트 디렉터리 식별(aidlc-docs/ **아님**)

**조사 결과 기록:**
```markdown
## Workspace State
- **Existing Code**: [Yes/No]
- **Programming Languages**: [List if found]
- **Build System**: [Maven/Gradle/npm/etc. if found]
- **Project Structure**: [Monolith/Microservices/Library/Empty]
- **Workspace Root**: [Absolute path]
```

## Step 3: 다음 단계 결정

**워크스페이스가 비어 있음(기존 코드 없음)**:
- 플래그 설정: `brownfield = false`
- 다음 단계: Requirements Analysis

**워크스페이스에 기존 코드가 있음**:
- 플래그 설정: `brownfield = true`
- `aidlc-docs/inception/reverse-engineering/`에 기존 reverse engineering 산출물이 있는지 확인
- **reverse engineering 산출물이 있으면**:
    - 산출물이 오래되었는지 확인(산출물 타임스탬프와 코드베이스의 마지막 유의미한 수정 시점 비교)
    - **산출물이 최신이면**: 로드하고 Requirements Analysis로 건너뜀
    - **산출물이 오래되었으면**: 다음 단계는 Reverse Engineering(산출물 갱신)
    - **사용자가 명시적으로 재실행을 요청하면**: 오래됨과 관계없이 다음 단계는 Reverse Engineering
- **reverse engineering 산출물이 없으면**: 다음 단계는 Reverse Engineering

## Step 4: 초기 상태 파일 생성

`aidlc-docs/aidlc-state.md` 생성:

```markdown
# AI-DLC State Tracking

## Project Information
- **Project Type**: [Greenfield/Brownfield]
- **Start Date**: [ISO timestamp]
- **Current Stage**: INCEPTION - Workspace Detection

## Workspace State
- **Existing Code**: [Yes/No]
- **Reverse Engineering Needed**: [Yes/No]
- **Workspace Root**: [Absolute path]

## Code Location Rules
- **Application Code**: Workspace root (NEVER in aidlc-docs/)
- **Documentation**: aidlc-docs/ only
- **Structure patterns**: See code-generation.md Critical Rules

## Stage Progress
[Will be populated as workflow progresses]
```

## Step 5: 완료 메시지 제시

**Brownfield 프로젝트:**
```markdown
# 🔍 Workspace Detection Complete

Workspace analysis findings:
• **Project Type**: Brownfield project
• [AI-generated summary of workspace findings in bullet points]
• **Next Step**: Proceeding to **Reverse Engineering** to analyze existing codebase...
```

**Greenfield 프로젝트:**
```markdown
# 🔍 Workspace Detection Complete

Workspace analysis findings:
• **Project Type**: Greenfield project
• **Next Step**: Proceeding to **Requirements Analysis**...
```

## Step 6: 자동 진행

- **사용자 승인 불필요** — 정보 제공 목적만 해당
- 자동으로 다음 단계 진행:
  - **Brownfield**: Reverse Engineering(기존 산출물 없을 때) 또는 Requirements Analysis(산출물이 있을 때)
  - **Greenfield**: Requirements Analysis
