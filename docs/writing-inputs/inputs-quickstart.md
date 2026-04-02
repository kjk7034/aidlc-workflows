# AI-DLC 빠른 시작

AI-DLC(AI-Driven Development Life Cycle)는 AI 보조를 계획·설계·구축까지 안내하는 구조화된 워크플로입니다. 프로젝트를 시작하기 전에 AI에게 **무엇을 만들지**와 **어떤 도구를 쓸지**를 알려 주는 문서 두 가지를 제공합니다.

---

## 제공해야 할 것

### 1. 비전 문서 — 무엇을 왜 만들지

| 섹션 | 작성 내용 | 분량 |
|---------|--------------|----------|
| **요약** | 한 단락: 무엇인지, 누구를 위한지, 왜 중요한지 | 3~5문장 |
| **문제 정의** | 해결하려는 구체적 비즈니스 문제 | 1~2단락 |
| **대상 사용자** | 누가 쓰는지, 사용자 유형별 필요 | 사용자 유형당 한 행인 표 |
| **성공 지표** | 프로젝트 성공을 어떻게 측정할지 | 측정 가능한 목표가 있는 표 |
| **전체 비전 범위** | 성숙 시 제품이 될 수 있는 전 범위를 기능 영역별로 | 필요한 만큼 |
| **MVP 범위 — 포함** | 첫 릴리스에 넣는 모든 기능과 근거 | 표. 여기 없으면 MVP에 없음. |
| **MVP 범위 — 제외** | MVP에서 의도적으로 뺀 기능, 이유와 목표 단계 | 표. 범위 확대 방지. |
| **위험과 미결 질문** | 잘못될 수 있는 일, 아직 결정되지 않은 일 | 표와 글머리 |

**핵심 원칙:** 전체 비전과 MVP를 분리하세요. 전체 비전은 지향점이고, MVP는 가치를 전달하는 최소 단위입니다.

전체 가이드: [vision-document-guide.md](vision-document-guide.md)  
예시: [example-vision-scientific-calculator-api.md](example-vision-scientific-calculator-api.md)

---

### 2. 기술 환경 문서 — 어떤 도구를 쓸지

| 섹션 | 작성 내용 | 분량 |
|---------|--------------|----------|
| **언어** | 필수·허용·금지 언어와 버전 | 범주별 표 |
| **프레임워크와 라이브러리** | 필수·선호·금지와 근거·대안 | 범주별 표 |
| **클라우드 서비스** | 허용 목록과 비허용 목록, 제약 | 목록별 표 |
| **아키텍처와 패턴** | API 스타일, 데이터 패턴, 메시징, 프로젝트 구조 | 짧은 절과 표 |
| **보안** | 인증 방식, 암호화, 입력 검증, 시크릿 관리, 선택한 보안 규정 준수 프레임워크와 범주별 통제 문서화 | 여러 하위 절 |
| **테스트** | 테스트 유형, 커버리지 목표, 도구, CI/CD 게이트 | 표 |
| **예시 코드** | 엔드포인트·함수·테스트·인프라의 표준 패턴을 보여 주는 템플릿 코드 | `examples/` 디렉터리의 동작하는 코드 |

**핵심 원칙:** 허용되는 것과 안 되는 것을 명시하세요. 허용/비허용 목록이 있으면 AI가 추측하지 않습니다.

전체 가이드: [technical-environment-guide.md](technical-environment-guide.md)  
예시: [example-tech-env-scientific-calculator-api.md](example-tech-env-scientific-calculator-api.md)

---

## 최소 입력

빠르게 시작하고 나중에 채우려면 최소한 다음을 제공하세요.

### 비전(최소)

```
1. 무엇을 누구를 위해 만드는지 한 단락
2. MVP 기능 목록(포함 범위)
3. MVP에 넣지 않는 것 목록
4. 미결 질문 — 이미 불확실하다고 아는 것들
```

미결 질문은 선택이지만 가치가 큽니다. Requirements Analysis에 미리 알려 둔 모호함으로 들어가므로, AI-DLC가 설계 중간에 깜짝 이슈로 끌고 오지 않고 초기에 다룹니다.

예시: [example-minimal-vision-scientific-calculator-api.md](example-minimal-vision-scientific-calculator-api.md)

### 기술 환경(최소)

```
1. 언어와 버전
2. 패키지 관리자
3. 웹 프레임워크(해당 시)
4. 클라우드 제공자와 배포 모델(또는 "로컬만")
5. 테스트 프레임워크
6. 금지 라이브러리와 서비스 — 표: 금지 항목 | 이유 | 대신 쓸 것
7. 보안 기본(인증 방식, 입력 검증, 시크릿 관리)
8. 예시 코드 패턴 — 일반적인 엔드포인트, 함수, 테스트 각각 짧은 예 하나
```

**6번:** 이유와 권장 대안을 넣는 것이 중요합니다. 없으면 AI-DLC가 금지는 지킬 수 있어도 의도를 이해하지 못해 좋은 대체 선택을 하기 어렵습니다.

**8번:** 짧은 예 하나만 있어도 코드 생성 시 AI-DLC가 스스로 패턴을 만들지 않고 따를 구체적 패턴이 됩니다. 기본 외에서 가장 효과가 큰 추가입니다.

예시: [example-minimal-tech-env-scientific-calculator-api.md](example-minimal-tech-env-scientific-calculator-api.md)

나머지는 Inception 단계에서 AI-DLC의 명확화 질문으로 채울 수 있습니다. 처음에 많이 줄수록 AI가 묻는 횟수는 줄어듭니다.

---

## 브라운필드 프로젝트

기존 코드베이스에 추가하거나 수정할 때는 입력이 다른 질문에 답해야 합니다. 전체 가이드에 브라운필드가 자세히 나오지만, 최소한 다음입니다.

### 비전(브라운필드 최소)

```
1. 현재 상태 — 오늘 시스템이 무엇을 하는지 한 단락
2. 무엇을 추가·변경하는지 — 변경에 대한 명확한 설명
3. 이번 반복에 포함하는 기능
4. 이번 반복에서 제외하는 기능
5. 바꾸면 안 되는 것 — 새 작업이 건드리면 안 되는 기존 컴포넌트, API, 데이터
6. 미결 질문
```

"바꾸면 안 되는 것" 절이 중요합니다. AI-DLC가 Reverse Engineering 단계로 기존 코드베이스를 분석하지만, 경계를 명시하면 동작 중인 부분을 흔들 제안을 줄일 수 있습니다.

예시: [example-minimal-vision-brownfield.md](example-minimal-vision-brownfield.md)

### 기술 환경(브라운필드 최소)

```
1. 기존 스택 — 언어, 프레임워크, DB, 인프라 — 버전 포함
2. 무엇을 추가할지(새 서비스, 테이블, 컴포넌트)
3. 그대로 둘 것 — 건드리지 않을 서비스, 스키마, 계약, 설정
4. 금지 패턴 — 기존 코드베이스와 충돌하는 라이브러리나 접근
5. 보안 기본 — 기존 시스템의 인증과 시크릿 처리
6. 기존 코드베이스에서 나온 예시 코드 패턴
```

브라운필드에서는 예시 코드 패턴이 특히 중요합니다. AI-DLC는 기존 코드베이스와 어울리는 코드를 생성해야 하며, 옆에 새 규칙을 들이밀지 않아야 합니다. 실제 파일에서 예를 가져오세요.

예시: [example-minimal-tech-env-brownfield.md](example-minimal-tech-env-brownfield.md)

---

## 문서를 준비한 뒤 일어나는 일

AI-DLC는 두 가지 주요 단계를 거칩니다.

**Inception** — 이해하고 계획
1. 워크스페이스 감지(신규 프로젝트 또는 기존 코드)
2. 요구 사항 분석(불명확하면 명확화 질문)
3. 사용자 스토리 생성(프로젝트에 맞을 때)
4. 실행 계획 수립(어떤 단계를 실행·건너뛸지)
5. 컴포넌트와 작업 단위 설계(복잡도에 맞을 때)

**Construction** — 작업 단위마다 설계하고 구축
1. 기능 설계(비즈니스 로직, 도메인 모델)
2. NFR 요구 및 설계(성능, 보안, 확장성)
3. 인프라 설계(실제 클라우드 서비스에 매핑)
4. 코드 생성(코드, 테스트, 배포 산출물 작성)
5. 빌드 및 테스트(빌드 안내, 테스트 실행, 검증)

각 단계는 진행하기 전에 승인을 받습니다. 변경 요청, 건너뛴 단계 추가, 게이트에서 방향 전환을 할 수 있습니다.

---

## 파일 개요

```
docs/writing-inputs/
  inputs-quickstart.md                               <-- You are here
  vision-document-guide.md                           <-- How to write a vision document
  technical-environment-guide.md                     <-- How to write a tech environment document

  -- Greenfield examples (new project from scratch) --
  example-vision-scientific-calculator-api.md        <-- Full example: CalcEngine vision
  example-tech-env-scientific-calculator-api.md      <-- Full example: CalcEngine tech env
  example-minimal-vision-scientific-calculator-api.md<-- Minimal example: CalcEngine vision
  example-minimal-tech-env-scientific-calculator-api.md<-- Minimal example: CalcEngine tech env

  -- Brownfield examples (adding to an existing system) --
  example-minimal-vision-brownfield.md               <-- Minimal example: returns module on existing platform
  example-minimal-tech-env-brownfield.md             <-- Minimal example: returns module on existing platform
```
