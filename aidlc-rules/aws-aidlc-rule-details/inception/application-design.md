# Application Design - 상세 단계

## 목적
**상위 수준 컴포넌트 식별 및 서비스 레이어 설계**

Application Design은 다음에 초점을 맞춥니다:
- 주요 기능 컴포넌트와 책임 식별
- 컴포넌트 인터페이스 정의(상세 비즈니스 로직 아님)
- 오케스트레이션을 위한 서비스 레이어 설계
- 컴포넌트 의존성과 통신 패턴 확립

**참고**: 상세 비즈니스 로직 설계는 이후 CONSTRUCTION 단계의 Functional Design(단위별)에서 수행됩니다.

## 전제 조건
- Workspace Detection이 완료되어야 함
- Requirements Analysis 권장(기능 맥락 제공)
- User Stories 권장(user stories가 설계 결정을 안내)
- 실행 계획에 Application Design 단계 실행이 포함되어야 함

## 단계별 실행

### 1. 맥락 분석
- `aidlc-docs/inception/requirements/requirements.md`와 `aidlc-docs/inception/user-stories/stories.md` 읽기
- 핵심 비즈니스 역량과 기능 영역 식별
- 설계 범위와 복잡도 판단

### 2. Application Design 계획 생성
- application design용 체크박스 []가 있는 계획 생성
- 컴포넌트, 책임, 메서드, 비즈니스 규칙, 서비스에 초점
- 각 단계와 하위 단계에 체크박스 [] 포함

### 3. 계획에 필수 설계 산출물 포함
- **반드시** 설계 계획에 다음 필수 산출물 포함:
  - [ ] 컴포넌트 정의와 고수준 책임이 담긴 components.md 생성
  - [ ] 메서드 시그니처가 담긴 component-methods.md 생성(비즈니스 규칙 상세는 이후 Functional Design에서)
  - [ ] 서비스 정의와 오케스트레이션 패턴이 담긴 services.md 생성
  - [ ] 의존 관계와 통신 패턴이 담긴 component-dependency.md 생성
  - [ ] 설계 완전성·일관성 검증

### 4. 맥락에 맞는 질문 생성
**지시**: 요구사항과 스토리를 분석해 **이** 애플리케이션 설계에 맞는 질문을 생성합니다. 아래 범주를 안내로 사용합니다. 각 범주를 평가하고 적용 여부가 의심스러우면 건너뛰지 말고 질문합니다 — 과신은 나쁜 결과로 이어집니다(overconfidence-prevention.md 참고).

- [Answer]: 태그 형식으로 질문 삽입
- 모호함, 누락 정보, 명확화가 필요한 영역에 초점
- 설계 결정을 개선하려면 사용자 입력이 필요한 곳마다 질문 생성
- **의심스러우면 질문한다** — 과신은 나쁜 설계로 이어짐

**평가할 질문 범주**(모든 범주 고려):
- **컴포넌트 식별** — 컴포넌트 경계, 구성, 그룹화 전략
- **컴포넌트 메서드** — 메서드 시그니처, 입출력 기대, 인터페이스 계약(상세 비즈니스 규칙은 이후)
- **서비스 레이어 설계** — 서비스 오케스트레이션, 경계, 조정 패턴
- **컴포넌트 의존성** — 통신 패턴, 의존성 관리, 결합 우려
- **디자인 패턴** — 아키텍처 스타일 선호, 패턴 선택, 설계 제약

### 5. Application Design 계획 저장
- `aidlc-docs/inception/plans/application-design-plan.md`로 저장
- 사용자 입력용 [Answer]: 태그 모두 포함
- 설계 측면 전체를 다루는지 확인

### 6. 사용자 입력 요청
- 사용자에게 계획 문서에서 [Answer]: 태그를 직접 채우도록 요청
- 설계 결정의 중요성 강조
- [Answer]: 태그 완료 방법을 명확히 안내

### 7. 답변 수집
- 사용자가 문서에서 [Answer]: 태그로 모든 질문에 답할 때까지 대기
- 모든 [Answer]: 태그가 채워질 때까지 진행하지 않음
- 빈 [Answer]: 태그가 없는지 문서 검토

### 8. 답변 분석 (필수)
진행하기 전에 **반드시** 모든 사용자 답변을 꼼꼼히 검토합니다:
- **모호하거나 애매한 답**: "mix of", "somewhere between", "not sure", "depends" 등
- **정의되지 않은 기준이나 용어**: 명확한 정의 없이 언급된 개념
- **상충하는 답**: 서로 충돌하는 응답
- **설계 세부 부족**: 구체적 안내가 부족한 답
- **옵션을 합친 답**: 명확한 결정 규칙 없이 여러 접근을 섞은 응답

### 9. 필수 후속 질문
8단계 분석에서 **어떤** 모호한 답이라도 드러나면 **반드시**:
- [Answer]: 태그로 계획 문서에 구체적 후속 질문 추가
- 모든 모호함이 해소될 때까지 승인 단계로 가지 않음
- 필요한 후속 질문 예:
  - "'A와 B의 혼합'이라고 하셨는데, A와 B 중 언제 무엇을 쓸지 판단하는 구체적 기준은 무엇인가요?"
  - "'A와 B 사이'라고 하셨는데, 정확한 중간 지점 접근을 정의해 주실 수 있나요?"
  - "'잘 모르겠다'고 하셨는데, 결정에 도움이 될 추가 정보는 무엇인가요?"
  - "'복잡도에 따라 다름'이라고 하셨는데, 복잡도 수준은 어떻게 정의하시나요?"

### 10. Application Design 산출물 생성
- 승인된 계획을 실행해 설계 산출물 생성
- `aidlc-docs/inception/application-design/components.md` 생성:
  - 컴포넌트 이름과 목적
  - 컴포넌트 책임
  - 컴포넌트 인터페이스
- `aidlc-docs/inception/application-design/component-methods.md` 생성:
  - 컴포넌트별 메서드 시그니처
  - 메서드별 고수준 목적
  - 입출력 타입
  - 참고: 상세 비즈니스 규칙은 Functional Design에서 정의(CONSTRUCTION phase, 단위별)
- `aidlc-docs/inception/application-design/services.md` 생성:
  - 서비스 정의
  - 서비스 책임
  - 서비스 상호작용과 오케스트레이션
- `aidlc-docs/inception/application-design/component-dependency.md` 생성:
  - 관계를 보여주는 의존성 매트릭스
  - 컴포넌트 간 통신 패턴
  - 데이터 흐름 다이어그램
- 위에서 만든 여러 설계 문서를 하나로 통합한 `aidlc-docs/inception/application-design/application-design.md` 생성

### 11. 승인 로깅
- `aidlc-docs/audit.md`에 타임스탬프와 함께 승인 프롬프트 기록
- 승인 프롬프트 전문 포함
- ISO 8601 타임스탬프 형식 사용

### 12. 완료 메시지 제시

```markdown
# 🏗️ Application Design Complete

[application-design에서 생성된 산출물에 대한 AI 생성 요약 — 글머리 기호]

> **📋 <u>**검토 필요:**</u>**  
> 다음 경로의 application design 산출물을 검토하세요: `aidlc-docs/inception/application-design/`

> **🚀 <u>**다음 단계**</u>**
>
> **선택할 수 있는 항목:**
>
> 🔧 **변경 요청** — 필요 시 application design 수정을 요청합니다
> [Units Generation을 건너뛴 경우:]
> 📝 **Units Generation 추가** — **Units Generation** 단계를 포함하도록 선택합니다(현재 건너뜀)
> ✅ **승인 후 계속** — 설계를 승인하고 **[Units Generation/CONSTRUCTION PHASE]**로 진행합니다
```

### 13. 명시적 승인 대기
- 사용자가 application design을 명시적으로 승인할 때까지 진행하지 않음
- 승인은 명확하고 모호하지 않아야 함
- 사용자가 변경을 요청하면 설계를 수정하고 승인 절차 반복

### 14. 승인 응답 기록
- `aidlc-docs/audit.md`에 타임스탬프와 함께 사용자 승인 응답 기록
- 사용자 응답 원문 그대로 포함
- 승인 상태를 명확히 표시

### 15. 진행 갱신
- `aidlc-docs/aidlc-state.md`에서 Application Design 단계 완료 표시
- "Current Status" 섹션 갱신
- 다음 단계 전환 준비
