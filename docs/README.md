# AI-DLC (AI-Driven Development Life Cycle)

> [!IMPORTANT]
> 생성형 AI는 오류를 만들 수 있습니다. 선택한 AI 모델과 에이전트 코딩 보조 도구가 생성한 모든 출력과 비용을 검토하는 것이 좋습니다. [AWS Responsible AI Policy](https://aws.amazon.com/ai/responsible-ai/policy/)를 참고하세요.

<!-- TODO: Replace this Amplify URL with a permanent/stable URL when available -->
AI-DLC는 요구에 맞게 조정되고 품질 기준을 유지하며, 과정을 사용자가 통제할 수 있게 하는 지능형 소프트웨어 개발 워크플로입니다. AI-DLC 방법론에 대해 더 알아보려면 이 [블로그](https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/)와 그 안에서 인용하는 [방법 정의 논문(Method Definition Paper)](https://prod.d13rzhkk8cj2z0.amplifyapp.com/)을 읽으세요.

## 목차

- [빠른 시작](#quick-start)
- [플랫폼별 설정](#platform-specific-setup)
- [사용 방법](#usage)
- [3단계 적응형 워크플로](#three-phase-adaptive-workflow)
- [주요 기능](#key-features)
- [확장](#extensions)
- [원칙](#tenets)
- [사전 요구 사항](#prerequisites)
- [문제 해결](#troubleshooting)
- [버전 관리 권장 사항](#version-control-recommendations)
- [추가 자료](#additional-resources)
- [보안](#security)
- [라이선스](#license)

---

## Quick Start

1. [Releases 페이지](../../releases/latest)에서 최신 릴리스 zip을 프로젝트 디렉터리 **밖**의 폴더(예: `~/Downloads`)에 내려받습니다.
2. zip을 풀면 `aidlc-rules/` 폴더와 그 안의 두 하위 디렉터리가 있습니다.
   - `aws-aidlc-rules/` — 핵심 AI-DLC 워크플로 규칙
   - `aws-aidlc-rule-details/` — 핵심 규칙이 조건부로 참조하는 상세 규칙
3. 아래에서 사용하는 코딩 에이전트와 플랫폼에 맞는 설정 안내를 따릅니다.

---

## Platform-Specific Setup

  - [Kiro](#kiro)
  - [Amazon Q Developer IDE 플러그인](#amazon-q-developer-ide-pluginextension)
  - [Cursor IDE](#cursor-ide)
  - [Cline](#cline)
  - [Claude Code](#claude-code)
  - [GitHub Copilot](#github-copilot)
  - [기타 에이전트](#other-agents)

---

### Kiro

AI-DLC는 프로젝트 워크스페이스 안에서 [Kiro Steering Files](https://kiro.dev/docs/cli/steering/)를 사용합니다.

아래 명령은 zip을 `Downloads` 폴더에 풀었다고 가정합니다. 다른 위치를 썼다면 `Downloads`를 실제 폴더 경로로 바꾸세요.

On macOS/Linux:
```bash
mkdir -p .kiro/steering
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rules .kiro/steering/
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details .kiro/
```

On Windows (PowerShell):
```powershell
New-Item -ItemType Directory -Force -Path ".kiro\steering"
Copy-Item -Recurse "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules" ".kiro\steering\"
Copy-Item -Recurse "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details" ".kiro\"
```

On Windows (CMD):
```cmd
mkdir .kiro\steering
xcopy %USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules .kiro\steering\aws-aidlc-rules\ /E /I
xcopy %USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details .kiro\aws-aidlc-rule-details\ /E /I
```

프로젝트 구조는 다음과 같아야 합니다.
```
<project-root>/
    ├── .kiro/
    │     ├── steering/
    │     │      ├── aws-aidlc-rules/
    │     ├── aws-aidlc-rule-details/
```

규칙이 로드되었는지 확인하려면:

#### Kiro IDE에서 확인

Steering 파일 패널을 열고 아래 스크린샷처럼 `Workspace` 아래에 `core-workflow` 항목이 보이는지 확인합니다.

<img src="./assets/images/kiro-ide-aidlc-rules-loaded.png?raw=true" alt="AI-DLC Rules in Kiro IDE" width="700" height="450">

AI-DLC 워크플로를 실행할 때는 Kiro IDE를 Vibe 모드로 사용합니다. 그래야 AI-DLC 워크플로가 Kiro에서 개발 흐름을 안내합니다. 가끔 Kiro가 spec 모드로 전환하라고 제안할 수 있습니다. Vibe 모드를 유지하려면 그런 프롬프트에는 `No`를 선택하세요.

<img src="./assets/images/kiro-sdd-nudge.png?raw=true" alt="Staying in Kiro Vibe mode" width="500" height="175">

#### Kiro CLI에서 확인
`kiro-cli`를 실행한 뒤 `/context show`를 입력하고 `.kiro/steering/aws-aidlc-rules` 항목이 있는지 확인합니다.

<img src="./assets/images/kiro-cli-aidlc-rules-loaded.png?raw=true" alt="AI-DLC Rules in Kiro CLI" width="700" height="660">

---

### Amazon Q Developer IDE Plugin/Extension

AI-DLC는 프로젝트 워크스페이스 안에서 [Amazon Q Rules](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/context-project-rules.html)를 사용합니다.

아래 명령은 zip을 `Downloads` 폴더에 풀었다고 가정합니다. 다른 위치를 썼다면 `Downloads`를 실제 폴더 경로로 바꾸세요.

On macOS/Linux:
```bash
mkdir -p .amazonq/rules
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rules .amazonq/rules/
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details .amazonq/
```

On Windows (PowerShell):
```powershell
New-Item -ItemType Directory -Force -Path ".amazonq\rules"
Copy-Item -Recurse "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules" ".amazonq\rules\"
Copy-Item -Recurse "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details" ".amazonq\"
```

On Windows (CMD):
```cmd
mkdir .amazonq\rules
xcopy %USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules .amazonq\rules\aws-aidlc-rules\ /E /I
xcopy %USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details .amazonq\aws-aidlc-rule-details\ /E /I
```

프로젝트 구조는 다음과 같아야 합니다.
```
<project-root>/
    ├── .amazonq/
    │     ├── rules/
    │     │     ├── aws-aidlc-rules/
    │     ├── aws-aidlc-rule-details/
```

규칙이 로드되었는지 확인하려면:

1. Amazon Q Chat 창에서 오른쪽 아래 `Rules` 버튼을 클릭합니다.
2. `.amazonq/rules/aws-aidlc-rules` 항목이 보이는지 확인합니다.

<img src="./assets/images/q-ide-aidlc-rules-loaded.png?raw=true" alt="AI-DLC Rules in Q Developer IDE plugin" width="700" height="400">

---

### Cursor IDE

AI-DLC는 지능형 워크플로를 구현하기 위해 [Cursor Rules](https://cursor.com/docs/context/rules)를 사용합니다.

아래 명령은 zip을 `Downloads` 폴더에 풀었다고 가정합니다. 다른 위치를 썼다면 `Downloads`를 실제 폴더 경로로 바꾸세요.

#### 옵션 1: Project Rules(권장)

**Unix/Linux/macOS:**
```bash
mkdir -p .cursor/rules

cat > .cursor/rules/ai-dlc-workflow.mdc << 'EOF'
---
description: "AI-DLC (AI-Driven Development Life Cycle) adaptive workflow for software development"
alwaysApply: true
---

EOF
cat ~/Downloads/aidlc-rules/aws-aidlc-rules/core-workflow.md >> .cursor/rules/ai-dlc-workflow.mdc

mkdir -p .aidlc-rule-details
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details/* .aidlc-rule-details/
```

**Windows PowerShell:**
```powershell
New-Item -ItemType Directory -Force -Path ".cursor\rules"

$frontmatter = @"
---
description: "AI-DLC (AI-Driven Development Life Cycle) adaptive workflow for software development"
alwaysApply: true
---

"@
$frontmatter | Out-File -FilePath ".cursor\rules\ai-dlc-workflow.mdc" -Encoding utf8

Get-Content "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" | Add-Content ".cursor\rules\ai-dlc-workflow.mdc"

New-Item -ItemType Directory -Force -Path ".aidlc-rule-details"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details\*" ".aidlc-rule-details\" -Recurse
```

**Windows CMD:**
```cmd
mkdir .cursor\rules

(
echo ---
echo description: "AI-DLC (AI-Driven Development Life Cycle) adaptive workflow for software development"
echo alwaysApply: true
echo ---
echo.
) > .cursor\rules\ai-dlc-workflow.mdc

type "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" >> .cursor\rules\ai-dlc-workflow.mdc

mkdir .aidlc-rule-details
xcopy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details" ".aidlc-rule-details\" /E /I
```

#### 옵션 2: AGENTS.md(간단한 대안)

**Unix/Linux/macOS:**
```bash
cp ~/Downloads/aidlc-rules/aws-aidlc-rules/core-workflow.md ./AGENTS.md
mkdir -p .aidlc-rule-details
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details/* .aidlc-rule-details/
```

**Windows PowerShell:**
```powershell
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".\AGENTS.md"
New-Item -ItemType Directory -Force -Path ".aidlc-rule-details"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details\*" ".aidlc-rule-details\" -Recurse
```

**Windows CMD:**
```cmd
copy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".\AGENTS.md"
mkdir .aidlc-rule-details
xcopy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details" ".aidlc-rule-details\" /E /I
```

**설정 확인:**
1. **Cursor 설정 → Rules, Commands**를 엽니다.
2. **Project Rules**에서 `ai-dlc-workflow`가 목록에 있는지 확인합니다.
3. `AGENTS.md`는 자동으로 감지되어 적용됩니다.

![AI-DLC Rules in Cursor](./assets/images/cursor-ide-aidlc-rules-loaded.png?raw=true "AI-DLC Rules in Cursor")

**디렉터리 구조(옵션 1):**
```
<my-project>/
├── .cursor/
│   └── rules/
│       └── ai-dlc-workflow.mdc
└── .aidlc-rule-details/
    ├── common/
    ├── inception/
    ├── construction/
    ├── extensions/
    └── operations/
```

---

### Cline

AI-DLC는 지능형 워크플로를 구현하기 위해 Cline Rules를 사용합니다.

아래 명령은 zip을 `Downloads` 폴더에 풀었다고 가정합니다. 다른 위치를 썼다면 `Downloads`를 실제 폴더 경로로 바꾸세요.

#### 옵션 1: .clinerules 디렉터리(권장)

**Unix/Linux/macOS:**
```bash
mkdir -p .clinerules
cp ~/Downloads/aidlc-rules/aws-aidlc-rules/core-workflow.md .clinerules/
mkdir -p .aidlc-rule-details
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details/* .aidlc-rule-details/
```

**Windows PowerShell:**
```powershell
New-Item -ItemType Directory -Force -Path ".clinerules"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".clinerules\"
New-Item -ItemType Directory -Force -Path ".aidlc-rule-details"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details\*" ".aidlc-rule-details\" -Recurse
```

**Windows CMD:**
```cmd
mkdir .clinerules
copy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".clinerules\"
mkdir .aidlc-rule-details
xcopy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details" ".aidlc-rule-details\" /E /I
```

#### 옵션 2: AGENTS.md(대안)

**Unix/Linux/macOS:**
```bash
cp ~/Downloads/aidlc-rules/aws-aidlc-rules/core-workflow.md ./AGENTS.md
mkdir -p .aidlc-rule-details
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details/* .aidlc-rule-details/
```

**Windows PowerShell:**
```powershell
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".\AGENTS.md"
New-Item -ItemType Directory -Force -Path ".aidlc-rule-details"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details\*" ".aidlc-rule-details\" -Recurse
```

**Windows CMD:**
```cmd
copy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".\AGENTS.md"
mkdir .aidlc-rule-details
xcopy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details" ".aidlc-rule-details\" /E /I
```

**설정 확인:**
1. Cline 채팅에서 입력란 아래 Rules 팝오버를 찾습니다.
2. `core-workflow.md`가 목록에 있고 활성화되어 있는지 확인합니다.
3. 필요에 따라 규칙 파일을 켜고 끌 수 있습니다.

![AI-DLC Rules in Cline](./assets/images/cline-ide-aidlc-rules-loaded.png?raw=true "AI-DLC Rules in Cline")

**디렉터리 구조(옵션 1):**
```
<my-project>/
├── .clinerules/
│   └── core-workflow.md
└── .aidlc-rule-details/
    ├── common/
    ├── inception/
    ├── construction/
    ├── extensions/
    └── operations/
```

---

### Claude Code

AI-DLC는 지능형 워크플로를 구현하기 위해 Claude Code의 프로젝트 메모리 파일(`CLAUDE.md`)을 사용합니다.

아래 명령은 zip을 `Downloads` 폴더에 풀었다고 가정합니다. 다른 위치를 썼다면 `Downloads`를 실제 폴더 경로로 바꾸세요.

#### 옵션 1: 프로젝트 루트(권장)

**Unix/Linux/macOS:**
```bash
cp ~/Downloads/aidlc-rules/aws-aidlc-rules/core-workflow.md ./CLAUDE.md
mkdir -p .aidlc-rule-details
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details/* .aidlc-rule-details/
```

**Windows PowerShell:**
```powershell
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".\CLAUDE.md"
New-Item -ItemType Directory -Force -Path ".aidlc-rule-details"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details\*" ".aidlc-rule-details\" -Recurse
```

**Windows CMD:**
```cmd
copy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".\CLAUDE.md"
mkdir .aidlc-rule-details
xcopy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details" ".aidlc-rule-details\" /E /I
```

#### 옵션 2: .claude 디렉터리

**Unix/Linux/macOS:**
```bash
mkdir -p .claude
cp ~/Downloads/aidlc-rules/aws-aidlc-rules/core-workflow.md .claude/CLAUDE.md
mkdir -p .aidlc-rule-details
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details/* .aidlc-rule-details/
```

**Windows PowerShell:**
```powershell
New-Item -ItemType Directory -Force -Path ".claude"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".claude\CLAUDE.md"
New-Item -ItemType Directory -Force -Path ".aidlc-rule-details"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details\*" ".aidlc-rule-details\" -Recurse
```

**Windows CMD:**
```cmd
mkdir .claude
copy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".claude\CLAUDE.md"
mkdir .aidlc-rule-details
xcopy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details" ".aidlc-rule-details\" /E /I
```

**설정 확인:**
1. 프로젝트 디렉터리에서 Claude Code를 시작합니다(CLI: `claude` 또는 VS Code 확장).
2. `/config` 명령으로 현재 설정을 봅니다.
3. Claude에게 "이 프로젝트에서 현재 어떤 지침이 활성화되어 있나요?"라고 물어봅니다.

**디렉터리 구조(옵션 1):**
```
<my-project>/
├── CLAUDE.md
└── .aidlc-rule-details/
    ├── common/
    ├── inception/
    ├── construction/
    ├── extensions/
    └── operations/
```

---

### GitHub Copilot

AI-DLC는 지능형 워크플로를 구현하기 위해 [GitHub Copilot 사용자 지정 지침](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)을 사용합니다. `.github/copilot-instructions.md` 파일은 자동으로 감지되어 워크스페이스의 모든 채팅 요청에 적용됩니다.

아래 명령은 zip을 `Downloads` 폴더에 풀었다고 가정합니다. 다른 위치를 썼다면 `Downloads`를 실제 폴더 경로로 바꾸세요.

**Unix/Linux/macOS:**
```bash
mkdir -p .github
cp ~/Downloads/aidlc-rules/aws-aidlc-rules/core-workflow.md .github/copilot-instructions.md
mkdir -p .aidlc-rule-details
cp -R ~/Downloads/aidlc-rules/aws-aidlc-rule-details/* .aidlc-rule-details/
```

**Windows PowerShell:**
```powershell
New-Item -ItemType Directory -Force -Path ".github"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".github\copilot-instructions.md"
New-Item -ItemType Directory -Force -Path ".aidlc-rule-details"
Copy-Item "$env:USERPROFILE\Downloads\aidlc-rules\aws-aidlc-rule-details\*" ".aidlc-rule-details\" -Recurse
```

**Windows CMD:**
```cmd
mkdir .github
copy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rules\core-workflow.md" ".github\copilot-instructions.md"
mkdir .aidlc-rule-details
xcopy "%USERPROFILE%\Downloads\aidlc-rules\aws-aidlc-rule-details" ".aidlc-rule-details\" /E /I
```

**설정 확인:**
1. VS Code에서 프로젝트 폴더를 엽니다.
2. Copilot Chat 패널을 엽니다(Cmd/Ctrl+Shift+I).
3. **Configure Chat**(톱니 아이콘) > **Chat Instructions**에서 `copilot-instructions`가 목록에 있는지 확인합니다.
4. 또는 채팅 입력에 `/instructions`를 입력해 활성 지침을 봅니다.

**디렉터리 구조:**
```
<my-project>/
├── .github/
│   └── copilot-instructions.md
└── .aidlc-rule-details/
    ├── common/
    ├── inception/
    ├── construction/
    ├── extensions/
    └── operations/
```

---

### 기타 에이전트

AI-DLC는 프로젝트 수준 규칙이나 steering 파일을 지원하는 모든 코딩 에이전트와 함께 동작합니다. 일반적인 방법은 다음과 같습니다.

1. 에이전트가 프로젝트 규칙을 읽는 위치에 `aws-aidlc-rules/`를 둡니다(에이전트 문서를 참고하세요).
2. 규칙이 참조할 수 있도록 `aws-aidlc-rule-details/`는 형제 수준에 둡니다.

에이전트에 규칙 파일 관례가 없으면 두 폴더를 모두 프로젝트 루트에 두고, 에이전트가 규칙 디렉터리로 `aws-aidlc-rules/`를 바라보게 하면 됩니다.

---

## Usage

1. 채팅에서 **"Using AI-DLC, ..."**로 시작하는 문구로 의도를 말해 어떤 소프트웨어 개발 프로젝트든 시작합니다.
2. AI-DLC 워크플로가 자동으로 활성화되어 이후를 안내합니다.
3. AI-DLC가 묻는 구조화된 질문에 답합니다.
4. AI가 만든 모든 계획을 꼼꼼히 검토하고, 감독과 검증을 제공합니다.
5. 실행 계획을 검토해 어떤 단계가 실행될지 확인합니다.
6. 산출물을 꼼꼼히 검토하고 통제를 유지하려면 각 단계를 승인합니다.
7. 모든 산출물은 `aidlc-docs/` 디렉터리에 생성됩니다.

---

## Three-Phase Adaptive Workflow

AI-DLC는 프로젝트 복잡도에 맞게 조정되는 구조화된 3단계 방식을 따릅니다.

### 🔵 INCEPTION PHASE
**무엇을** 만들고 **왜** 만드는지 정합니다.
- 요구 사항 분석 및 검증
- 사용자 스토리 작성(해당될 때)
- 애플리케이션 설계 및 병렬 개발을 위한 작업 단위 생성
- 위험 평가 및 복잡도 평가

### 🟢 CONSTRUCTION PHASE
**어떻게** 만들지 정합니다.
- 상세 컴포넌트 설계
- 코드 생성 및 구현
- 빌드 구성 및 테스트 전략
- 품질 보증 및 검증

### 🟡 OPERATIONS PHASE
배포 및 모니터링(향후)
- 배포 자동화 및 인프라
- 모니터링 및 관측 가능성 설정
- 프로덕션 준비 검증

---

## Key Features

| 기능 | 설명 |
|---------|-------------|
| **적응형 지능** | 특정 요청에 가치를 더하는 단계만 실행합니다 |
| **맥락 인식** | 기존 코드베이스와 복잡도 요구를 분석합니다 |
| **위험 기반** | 복잡한 변경은 포괄적으로, 단순한 변경은 효율적으로 다룹니다 |
| **질문 주도** | 채팅이 아니라 파일 안의 구조화된 객관식 질문을 사용합니다 |
| **항상 통제** | 실행 계획을 검토하고 각 단계를 승인합니다 |
| **확장 가능** | 보안, 규정 준수, 조직별 규칙 등 사용자 정의 규칙을 핵심 워크플로 위에 겹쳐 씁니다 |

---

## Extensions

AI-DLC는 핵심 워크플로 위에 추가 규칙을 겹칠 수 있는 확장 시스템을 지원합니다. 확장은 `aws-aidlc-rule-details/extensions/` 아래에 마크다운 파일로 정리되며 범주별로 묶입니다(예: `security/`, `testing/`).

### 확장 작동 방식

각 확장은 같은 디렉터리에 두 파일로 구성됩니다.

- 확장 규칙이 담긴 **rules 파일**(예: `security-baseline.md`).
- 요구 사항 분석 중 사용자에게 제시되는 구조화된 객관식 질문이 담긴 **opt-in 파일**(예: `security-baseline.opt-in.md`).

워크플로 시작 시 AI-DLC는 `extensions/` 디렉터리를 검사하고 `*.opt-in.md` 파일만 로드합니다. 요구 사항 분석 중에는 각 opt-in 프롬프트를 사용자에게 보여 줍니다. 사용자가 옵트인하면 해당 rules 파일이 로드됩니다(이름 규칙: `.opt-in.md`를 제거하고 `.md`를 붙임). 옵트아웃하면 rules 파일은 로드되지 않습니다. 짝이 되는 `*.opt-in.md`가 없는 확장은 항상 강제됩니다.

활성화되면 확장 규칙은 차단 제약이 됩니다. 각 단계에서 모델이 준수 여부를 확인한 뒤 단계를 진행할 수 있습니다.

### 기본 제공 확장

`extensions/` 디렉터리에는 다음이 포함되어 있습니다(시간이 지나며 새 확장이 추가될 수 있음).

```
aws-aidlc-rule-details/
└── extensions/
    ├── security/                      # Extension category
    │   └── baseline/
    │       ├── security-baseline.md          # Baseline security rules
    │       └── security-baseline.opt-in.md   # Opt-in prompt
    └── testing/                       # Extension category
        └── property-based/
            ├── property-based-testing.md          # Property-based testing rules
            └── property-based-testing.opt-in.md   # Opt-in prompt
```

> [!IMPORTANT]
> 보안 확장 규칙은 AI-DLC 워크플로 안에서 효과적인 보안 규칙을 만드는 방향성 참고용으로 제공됩니다. 각 조직은 프로덕션 워크플로에 배포하기 전에 자체 보안 규칙을 만들고, 맞춤화하고, 철저히 테스트해야 합니다.

### 사용자 정의 확장 추가

기존 범주를 확장하거나 완전히 새 범주를 만들 수 있습니다.

1. `extensions/` 아래에 디렉터리를 만듭니다(예: `security/compliance/` 또는 `performance/baseline/`).
2. **rules 파일**(예: `compliance.md`)을 추가합니다. `security-baseline.md`와 같은 구조를 따릅니다.
   - 각 규칙을 `## Rule <PREFIX-NN>: <Title>` 형식의 제목으로 정의합니다. PREFIX는 짧은 범주 식별자이고 NN은 순번입니다(예: `COMPLIANCE-01`, `COMPLIANCE-02`). 이 ID는 감사 로그와 규정 준수 요약에서 참조되므로, 로드된 모든 확장에서 고유해야 합니다.
   - 요구 사항을 설명하는 **Rule** 섹션을 포함합니다.
   - 모델이 평가해야 할 구체적 검사가 담긴 **Verification** 섹션을 포함합니다.
3. `<name>.opt-in.md` 명명 규칙으로 짝이 되는 **opt-in 파일**을 추가합니다(예: `compliance.opt-in.md`). 기대 형식은 `security-baseline.opt-in.md`를 참고하세요. 이 파일이 없으면 사용자 옵트아웃 없이 확장이 항상 강제됩니다.
4. 규칙은 기본적으로 차단형입니다. 검증 기준을 충족하지 않으면 조치가 해결될 때까지 단계를 진행할 수 없습니다.

---

## Tenets

의사 결정을 이끄는 핵심 원칙입니다.

- **중복 없음**. 진실은 한 곳에만 둡니다. 특정 파일이 필요한 새 도구나 형식을 지원하면 원본에서 생성하고 별도 사본을 유지하지 않습니다.

- **방법론 우선**. AI-DLC는 근본적으로 방법론이지 도구가 아닙니다. 시작하려고 무엇이든 설치할 필요는 없어야 합니다. 다만 방법론 채택·확장을 돕는 편의 도구(스크립트, CLI)는 열어 둡니다.

- **재현 가능**. 규칙은 서로 다른 모델이 비슷한 결과를 내도록 충분히 명확해야 합니다. 모델마다 동작은 다르지만, 방법론은 명시적 지침으로 분산을 줄여야 합니다.

- **도구 중립**. 방법론은 어떤 IDE, 에이전트, 모델과도 동작합니다. 특정 도구나 벤더에 묶이지 않습니다.

- **사람 개입**. 중요한 결정은 사용자의 명시적 확인이 필요합니다. 에이전트가 제안하고 사람이 승인합니다.

---

## Prerequisites

보조 AI 코딩용으로 지원하는 플랫폼/도구 중 하나를 설치해 두세요.

| 플랫폼 | 설치 링크 |
|----------|------------------|
| Kiro | [Install](https://kiro.dev/) |
| Kiro CLI | [Install](https://kiro.dev/cli/) |
| Amazon Q Developer IDE Plugin | [Install](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-in-IDE.html) |
| Cursor IDE | [Install](https://cursor.com/) |
| Cline VS Code Extension | [Install](https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev) |
| Claude Code CLI | [Install](https://github.com/anthropics/claude-code) |
| GitHub Copilot | [Install](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) + [Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) |

---

## Troubleshooting

### 일반 문제

| 문제 | 해결 |
|---------|----------|
| 규칙이 로드되지 않음 | 플랫폼에 맞는 위치에 파일이 있는지 확인 |
| 파일 인코딩 문제 | 파일이 UTF-8인지 확인 |
| 세션에 규칙이 적용되지 않음 | 파일 변경 후 새 채팅 세션 시작 |
| 규칙 상세가 로드되지 않음 | `.aidlc-rule-details/`와 하위 디렉터리가 있는지 확인 |

### 플랫폼별 문제

#### Kiro
- Kiro CLI에서 `/context show`로 규칙 로드 여부 확인
- `.kiro/steering/` 디렉터리 구조 확인
- 참고: Kiro는 `.kiro/` 아래에 `aws-aidlc-rule-details`를 사용합니다(`.aidlc-rule-details/` 아님)

#### Amazon Q Developer
- `.amazonq/rules/` 디렉터리 구조 확인
- Amazon Q Chat Rules 패널에 규칙이 나열되는지 확인
- 참고: Amazon Q는 `.amazonq/` 아래에 `aws-aidlc-rule-details`를 사용합니다(`.aidlc-rule-details/` 아님)

#### Cursor
- "Apply Intelligently"를 쓰려면 frontmatter에 description이 정의되어 있어야 함
- **Cursor 설정 → Rules**에서 규칙이 켜져 있는지 확인
- 규칙이 너무 크면(500줄 초과) 여러 개의 집중 규칙으로 나눔

#### Cline
- 채팅 입력란 아래 Rules 팝오버 확인
- 팝오버 UI로 규칙 파일을 필요에 따라 켜고 끔

#### Claude Code
- `/config` 명령으로 현재 설정 확인
- "이 프로젝트에서 현재 어떤 지침이 활성화되어 있나요?"라고 질문

#### GitHub Copilot
- **Configure Chat**(톱니 아이콘) > **Chat Instructions**에서 지침 로드 여부 확인
- 채팅 입력에 `/instructions`로 활성 지침 파일 확인
- 워크스페이스 루트에 `.github/copilot-instructions.md`가 있는지 확인

### Windows의 파일 경로 문제
- 마크다운 안의 파일 경로에는 슬래시 `/` 사용
- 백슬래시가 있는 Windows 경로는 제대로 동작하지 않을 수 있음

---

## Version Control Recommendations

**저장소에 커밋:**
```gitignore
# These should be version controlled
CLAUDE.md
AGENTS.md
.amazonq/rules/
.amazonq/aws-aidlc-rule-details/
.kiro/steering/
.kiro/aws-aidlc-rule-details/
.cursor/rules/
.clinerules/
.github/copilot-instructions.md
.aidlc-rule-details/
```

**선택 — 필요하면 `.gitignore`에 추가:**
```gitignore
# Local-only settings
.claude/settings.local.json
```

---

## Generated aidlc-docs/ Reference

AI-DLC 워크플로가 생성하는 모든 문서 산출물에 대한 전체 참조는 [docs/GENERATED_DOCS_REFERENCE.md](docs/GENERATED_DOCS_REFERENCE.md)를 보세요.

---

## Additional Resources

<!-- TODO: Replace this Amplify URL with a permanent/stable URL when available -->
| 자료 | 링크 |
|----------|------|
| AI-DLC Method Definition Paper | [논문](https://prod.d13rzhkk8cj2z0.amplifyapp.com/) |
| AI-DLC 방법론 블로그 | [AWS Blog](https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/) |
| AI-DLC 오픈소스 출시 블로그 | [AWS Blog](https://aws.amazon.com/blogs/devops/open-sourcing-adaptive-workflows-for-ai-driven-development-life-cycle-ai-dlc/) |
| AI-DLC 예시 워크스루 블로그 | [AWS Blog](https://aws.amazon.com/blogs/devops/building-with-ai-dlc-using-amazon-q-developer/) |
| Amazon Q Developer 문서 | [문서](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-in-IDE.html) |
| Kiro CLI 문서 | [문서](https://kiro.dev/docs/cli/steering/) |
| Cursor Rules 문서 | [문서](https://cursor.com/docs/context/rules) |
| Claude Code 문서 | [GitHub](https://github.com/anthropics/claude-code) |
| GitHub Copilot 문서 | [문서](https://docs.github.com/en/copilot) |
| AI-DLC 사용하기(상호작용 패턴과 팁) | [docs/WORKING-WITH-AIDLC.md](docs/WORKING-WITH-AIDLC.md) |
| 기여 가이드 | [CONTRIBUTING.md](CONTRIBUTING.md) |
| 행동 강령 | [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) |

---

## Security

자세한 내용은 [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications)을 참고하세요.

## License

이 라이브러리는 MIT-0 라이선스로 제공됩니다. [LICENSE](LICENSE) 파일을 참고하세요.
