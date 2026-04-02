# NFR Requirements

## 전제 조건
- 해당 단위에 대해 Functional Design이 완료되어야 함
- 단위 functional design 산출물이 있어야 함
- 실행 계획에 NFR Requirements 단계 실행이 포함되어야 함

## 개요
단위에 대한 비기능 요구사항을 정하고 기술 스택을 선택합니다.

## 실행 단계

### Step 1: Functional Design 분석
- `aidlc-docs/construction/{unit-name}/functional-design/`에서 functional design 산출물 읽기
- 비즈니스 로직 복잡도와 요구사항 이해

### Step 2: NFR Requirements 계획 생성
- NFR 평가용 체크박스 []가 있는 계획 생성
- 확장성, 성능, 가용성, 보안에 초점
- 각 스텝에 체크박스 []

### Step 3: 맥락에 맞는 질문 생성
**지시**: functional design을 철저히 분석해 시스템 품질과 아키텍처 결정을 개선할 NFR 명확화가 필요한 **모든** 영역을 식별합니다. 포괄적 NFR 범위를 위해 선제적으로 질문합니다.

**중요**: 시스템 품질에 영향을 줄 수 있는 **어떤** 모호함이나 세부 누락이 있어도 기본적으로 질문합니다. 잘못된 NFR 가정보다 질문이 많은 편이 낫습니다.

- [Answer]: 태그 형식으로 질문 삽입
- 모호함, 누락 정보, 명확화가 필요한 **모든** 영역에 초점
- NFR 및 기술 스택 결정을 개선할 사용자 입력이 있으면 질문 생성
- **의심스러면 질문** — 과신은 나쁜 시스템 품질로 이어짐

**평가할 질문 범주** (**모든** 범주 고려):
- **Scalability Requirements** — 예상 부하, 성장 패턴, 스케일링 트리거, 용량 계획
- **Performance Requirements** — 응답 시간, 처리량, 지연, 성능 벤치마크
- **Availability Requirements** — 가동 시간 기대, 재해 복구, 페일오버, 연속성
- **Security Requirements** — 데이터 보호, 규정 준수, 인증, 권한, 위협 모델
- **Tech Stack Selection** — 기술 선호, 제약, 기존 시스템, 통합 요구사항
- **Reliability Requirements** — 오류 처리, 장애 허용, 모니터링, 알림
- **Maintainability Requirements** — 코드 품질, 문서, 테스트, 운영 요구사항
- **Usability Requirements** — 사용자 경험, 접근성, 인터페이스 요구사항

### Step 4: 계획 저장
- `aidlc-docs/construction/plans/{unit-name}-nfr-requirements-plan.md`로 저장
- 사용자 입력용 [Answer]: 태그 모두 포함

### Step 5: 답변 수집 및 분석
- 사용자가 모든 [Answer]: 태그를 채울 때까지 대기
- **필수**: **모든** 응답을 꼼꼼히 검토해 모호하거나 애매한 답을 찾음
- **중요**: **어떤** 불명확한 응답에도 후속 질문 추가 — 모호한 채로 진행하지 않음
- "depends", "maybe", "not sure", "mix of", "somewhere between", "standard", "typical" 같은 응답 찾기
- **어떤** 모호함이 보이면 명확화 질문 파일 생성
- **모든** 모호함이 해소될 때까지 진행하지 않음

### Step 6: NFR Requirements 산출물 생성
- `aidlc-docs/construction/{unit-name}/nfr-requirements/nfr-requirements.md` 생성
- `aidlc-docs/construction/{unit-name}/nfr-requirements/tech-stack-decisions.md` 생성

### Step 7: 완료 메시지 제시
- 다음 구조로 완료 메시지 제시:
     1. **완료 알림** (필수): 항상 다음으로 시작:

```markdown
# 📊 NFR Requirements Complete - [unit-name]
```

     2. **AI 요약** (선택): NFR 요구사항에 대한 구조화된 글머리 요약
        - 형식: "NFR requirements assessment has identified [description]:"
        - 주요 확장성, 성능, 가용성 요구사항(글머리)
        - 식별된 보안·규정 준수 요구사항
        - 기술 스택 결정과 근거
        - 워크플로 지침은 넣지 않음
        - 사실 위주로 유지
     3. **형식화된 워크플로 메시지** (필수): 항상 다음 형식으로 끝냄:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the NFR requirements at: `aidlc-docs/construction/[unit-name]/nfr-requirements/`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the NFR requirements based on your review  
> ✅ **Continue to Next Stage** - Approve NFR requirements and proceed to **[next-stage-name]**

---
```

### Step 8: 명시적 승인 대기
- 사용자가 NFR requirements를 명시적으로 승인할 때까지 진행하지 않음
- 승인은 명확하고 모호하지 않아야 함
- 변경 요청 시 요구사항을 수정하고 승인 절차 반복

### Step 9: 승인 기록 및 진행 갱신
- 타임스탬프와 함께 audit.md에 승인 기록
- 타임스탬프와 함께 사용자 승인 응답 기록
- aidlc-state.md에서 NFR Requirements 단계 완료 표시
