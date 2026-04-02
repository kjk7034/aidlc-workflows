# Infrastructure Design

## 전제 조건
- 해당 단위에 대해 Functional Design이 완료되어야 함
- NFR Design 권장(매핑할 논리 컴포넌트 제공)
- 실행 계획에 Infrastructure Design 단계 실행이 포함되어야 함

## 개요
논리 소프트웨어 컴포넌트를 배포 환경에 맞는 실제 인프라 선택에 매핑합니다.

## 실행 단계

### Step 1: 설계 산출물 분석
- `aidlc-docs/construction/{unit-name}/functional-design/`에서 functional design 읽기
- 존재 시 `aidlc-docs/construction/{unit-name}/nfr-design/`에서 NFR design 읽기
- 인프라가 필요한 논리 컴포넌트 식별

### Step 2: Infrastructure Design 계획 생성
- infrastructure design용 체크박스 []가 있는 계획 생성
- 실제 서비스(AWS, Azure, GCP, on-premise)에 매핑에 초점
- 각 스텝에 체크박스 []

### Step 3: 맥락에 맞는 질문 생성
**지시**: functional 및 NFR design을 철저히 분석해 인프라 결정을 개선할 명확화가 필요한 **모든** 영역을 식별합니다. 포괄적 인프라 범위를 위해 선제적으로 질문합니다.

**중요**: 인프라 품질에 영향을 줄 수 있는 **어떤** 모호함이나 세부 누락이 있어도 기본적으로 질문합니다. 잘못된 인프라 가정보다 질문이 많은 편이 낫습니다.

**필수**: 아래 **모든** 범주를 대상으로 질문하여 각 범주의 적용 여부를 functional·NFR design 증거로 판단합니다 — 명시적 근거 없이 범주를 건너뛰지 않습니다:

- [Answer]: 태그 형식으로 질문 삽입
- 모호함, 누락 정보, 명확화가 필요한 **모든** 영역에 초점
- 인프라 결정을 개선할 사용자 입력이 있으면 질문 생성
- **의심스러면 질문** — 과신은 나쁜 인프라 선택으로 이어짐

**평가할 질문 범주** (**모든** 범주 고려):
- **Deployment Environment** — 클라우드 선호, 환경 설정, 배포 대상
- **Compute Infrastructure** — 컴퓨트 서비스 선택, 크기, 스케일링 요구사항
- **Storage Infrastructure** — 데이터베이스 선택, 스토리지 패턴, 데이터 수명주기
- **Messaging Infrastructure** — 메시징/큐 서비스, 이벤트 기반 패턴, 비동기 처리
- **Networking Infrastructure** — 로드 밸런싱, API 게이트웨이 접근, 네트워크 토폴로지
- **Monitoring Infrastructure** — 관측 가능성 도구, 알림 전략, 로깅 요구사항
- **Shared Infrastructure** — 인프라 공유 전략, 멀티 테넌시, 리소스 격리

### Step 4: 계획 저장
- `aidlc-docs/construction/plans/{unit-name}-infrastructure-design-plan.md`로 저장
- 사용자 입력용 [Answer]: 태그 모두 포함

### Step 5: 답변 수집 및 분석
- 사용자가 모든 [Answer]: 태그를 채울 때까지 대기
- 모호하거나 애매한 응답 검토
- 필요 시 후속 질문 추가

### Step 6: Infrastructure Design 산출물 생성
- `aidlc-docs/construction/{unit-name}/infrastructure-design/infrastructure-design.md` 생성
- `aidlc-docs/construction/{unit-name}/infrastructure-design/deployment-architecture.md` 생성
- 공유 인프라가 있으면: `aidlc-docs/construction/shared-infrastructure.md` 생성

### Step 7: 완료 메시지 제시
- 다음 구조로 완료 메시지 제시:
     1. **완료 알림** (필수): 항상 다음으로 시작:

```markdown
# 🏢 Infrastructure Design Complete - [unit-name]
```

     2. **AI 요약** (선택): infrastructure design에 대한 구조화된 글머리 요약
        - 형식: "Infrastructure design has mapped [description]:"
        - 주요 인프라 서비스와 컴포넌트(글머리)
        - 배포 아키텍처 결정과 근거
        - 클라우드 제공자 선택과 서비스 매핑
        - 워크플로 지침은 넣지 않음
        - 사실 위주로 유지
     3. **형식화된 워크플로 메시지** (필수): 항상 다음 형식으로 끝냄:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the infrastructure design at: `aidlc-docs/construction/[unit-name]/infrastructure-design/`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the infrastructure design based on your review  
> ✅ **Continue to Next Stage** - Approve infrastructure design and proceed to **Code Generation**

---
```

### Step 8: 명시적 승인 대기
- 사용자가 infrastructure design을 명시적으로 승인할 때까지 진행하지 않음
- 승인은 명확하고 모호하지 않아야 함
- 변경 요청 시 설계를 수정하고 승인 절차 반복

### Step 9: 승인 기록 및 진행 갱신
- 타임스탬프와 함께 audit.md에 승인 기록
- 타임스탬프와 함께 사용자 승인 응답 기록
- aidlc-state.md에서 Infrastructure Design 단계 완료 표시
