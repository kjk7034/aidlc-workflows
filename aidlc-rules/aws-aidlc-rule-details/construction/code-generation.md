# Code Generation — 상세 단계

## 개요
이 단계는 두 개의 연속된 부분을 통해 각 작업 단위에 대한 코드를 생성합니다:
- **Part 1 - Planning**: 명시적 단계가 있는 상세 코드 생성 계획 작성
- **Part 2 - Generation**: 승인된 계획을 실행해 코드, 테스트, 산출물 생성

**참고**: brownfield 프로젝트에서는 "generate"는 적절할 때 기존 파일을 수정한다는 뜻이며, 중복 생성이 아닙니다.

## 전제 조건
- 해당 단위에 대해 Unit Design Generation이 완료되어야 함
- (실행한 경우) 해당 단위에 대해 NFR Implementation이 완료되어야 함
- 모든 단위 설계 산출물을 사용할 수 있어야 함
- 단위가 코드 생성 준비가 되어 있어야 함

---

# PART 1: PLANNING

## Step 1: 단위 맥락 분석
- [ ] Unit Design Generation에서 단위 설계 산출물 읽기
- [ ] 할당된 스토리를 이해하기 위해 단위 스토리 맵 읽기
- [ ] 단위 의존성과 인터페이스 식별
- [ ] 단위가 코드 생성 준비가 되었는지 검증

## Step 2: 상세 단위 코드 생성 계획 작성
- [ ] `aidlc-docs/aidlc-state.md`에서 워크스페이스 루트와 프로젝트 유형 읽기
- [ ] 코드 위치 결정(구조 패턴은 Critical Rules 참고)
- [ ] **Brownfield만**: 수정할 기존 파일을 위해 reverse engineering의 code-structure.md 검토
- [ ] 정확한 경로 문서화(절대 aidlc-docs/ 아님)
- [ ] 단위 생성을 위한 명시적 스텝 문서화:
  - Project Structure Setup (greenfield만)
  - Business Logic Generation
  - Business Logic Unit Testing
  - Business Logic Summary
  - API Layer Generation
  - API Layer Unit Testing
  - API Layer Summary
  - Repository Layer Generation
  - Repository Layer Unit Testing
  - Repository Layer Summary
  - Frontend Components Generation (해당 시)
  - Frontend Components Unit Testing (해당 시)
  - Frontend Components Summary (해당 시)
  - Database Migration Scripts (데이터 모델이 있을 때)
  - Documentation Generation (API 문서, README 갱신)
  - Deployment Artifacts Generation
- [ ] 각 스텝에 순번 부여
- [ ] 스토리 매핑 참조 포함
- [ ] 각 스텝에 체크박스 [ ] 추가

## Step 3: 단위 생성 맥락 포함
- [ ] 이 단위에 대해 다음 포함:
  - 이 단위가 구현하는 스토리
  - 다른 단위/서비스에 대한 의존성
  - 예상 인터페이스와 계약
  - 이 단위가 소유하는 데이터베이스 엔티티
  - 서비스 경계와 책임

## Step 4: 단위 계획 문서 작성
- [ ] 전체 계획을 `aidlc-docs/construction/plans/{unit-name}-code-generation-plan.md`로 저장
- [ ] 스텝 번호 포함(Step 1, Step 2 등)
- [ ] 단위 맥락과 의존성 포함
- [ ] 스토리 추적 가능성 포함
- [ ] 계획이 스텝별로 실행 가능한지 확인
- [ ] 이 계획이 Code Generation의 단일 진실 공급원임을 강조

## Step 5: 단위 계획 요약
- [ ] 사용자에게 단위 코드 생성 계획 요약 제공
- [ ] 단위 생성 접근 강조
- [ ] 스텝 순서와 스토리 범위 설명
- [ ] 총 스텝 수와 예상 범위 언급

## Step 6: 승인 프롬프트 로깅
- [ ] 사용자에게 묻기 전에 `aidlc-docs/audit.md`에 타임스탬프와 함께 프롬프트 기록
- [ ] 완전한 단위 코드 생성 계획에 대한 참조 포함
- [ ] ISO 8601 타임스탬프 형식 사용

## Step 7: 명시적 승인 대기
- [ ] 사용자가 단위 코드 생성 계획 전체를 명시적으로 승인할 때까지 진행하지 않음
- [ ] 승인은 전체 계획과 생성 순서를 포함해야 함
- [ ] 사용자가 변경을 요청하면 계획을 갱신하고 승인 절차 반복

## Step 8: 승인 응답 기록
- [ ] `aidlc-docs/audit.md`에 사용자 승인 응답을 타임스탬프와 함께 기록
- [ ] 정확한 사용자 응답 텍스트 포함
- [ ] 승인 상태를 명확히 표시

## Step 9: 진행 갱신
- [ ] `aidlc-state.md`에서 Code Generation Part 1(Planning) 완료 표시
- [ ] "Current Status" 섹션 갱신
- [ ] Code Generation으로 전환 준비

---

# PART 2: GENERATION

## Step 10: 단위 코드 생성 계획 로드
- [ ] `aidlc-docs/construction/plans/{unit-name}-code-generation-plan.md`에서 전체 계획 읽기
- [ ] 다음 미완료 스텝 식별(첫 [ ] 체크박스)
- [ ] 해당 스텝의 맥락 로드(단위, 의존성, 스토리)

## Step 11: 현재 스텝 실행
- [ ] 계획에서 대상 디렉터리 검증(절대 aidlc-docs/ 아님)
- [ ] **Brownfield만**: 대상 파일 존재 여부 확인
- [ ] 현재 스텝이 설명하는 것만 정확히 생성:
  - **파일이 있으면**: 제자리에서 수정(`ClassName_modified.java`, `ClassName_new.java` 등 생성 금지)
  - **파일이 없으면**: 새 파일 생성
- [ ] 올바른 위치에 쓰기:
  - **Application Code**: 프로젝트 구조에 따라 워크스페이스 루트
  - **Documentation**: `aidlc-docs/construction/{unit-name}/code/` (markdown만)
  - **Build/Config Files**: 워크스페이스 루트
- [ ] 단위 스토리 요구사항 준수
- [ ] 의존성과 인터페이스 존중

## Step 12: 진행 갱신
- [ ] 단위 코드 생성 계획에서 완료된 스텝을 [x]로 표시
- [ ] 생성이 끝난 단위 스토리를 [x]로 표시
- [ ] `aidlc-docs/aidlc-state.md` 현재 상태 갱신
- [ ] **Brownfield만**: 중복 파일이 생성되지 않았는지 확인(예: `ClassName.java` 옆에 `ClassName_modified.java` 없음)
- [ ] 생성된 모든 산출물 저장

## Step 13: 생성 계속 또는 완료
- [ ] 남은 스텝이 있으면 Step 10으로 돌아감
- [ ] 모든 스텝이 완료되면 완료 메시지 제시로 진행

## Step 14: 완료 메시지 제시
- 다음 구조로 완료 메시지 제시:
     1. **완료 알림** (필수): 항상 다음으로 시작:

```markdown
# 💻 Code Generation Complete - [unit-name]
```

     2. **AI 요약** (선택): 구조화된 글머리 요약
        - **Brownfield**: 수정 vs 생성 파일 구분(예: "• Modified: `src/services/user-service.ts`", "• Created: `src/services/auth-service.ts`")
        - **Greenfield**: 경로와 함께 생성 파일 나열(예: "• Created: `src/services/user-service.ts`")
        - 경로와 함께 테스트, 문서, 배포 산출물 나열
        - 사실만, 워크플로 지침 없음
     3. **형식화된 워크플로 메시지** (필수): 항상 다음 형식으로 끝냄:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the generated code at:
> - **Application Code**: `[actual-workspace-path]`
> - **Documentation**: `aidlc-docs/construction/[unit-name]/code/`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the generated code based on your review  
> ✅ **Continue to Next Stage** - Approve code generation and proceed to **[next-unit/Build & Test]**

---
```

## Step 15: 명시적 승인 대기
- 사용자가 생성된 코드를 명시적으로 승인할 때까지 진행하지 않음
- 승인은 명확하고 모호하지 않아야 함
- 변경 요청 시 코드를 수정하고 승인 절차 반복

## Step 16: 승인 기록 및 진행 갱신
- 타임스탬프와 함께 audit.md에 승인 기록
- 타임스탬프와 함께 사용자 승인 응답 기록
- aidlc-state.md에서 해당 단위의 Code Generation 단계 완료 표시

---

## Critical Rules

### 코드 위치 규칙
- **Application code**: 워크스페이스 루트만(절대 aidlc-docs/ 아님)
- **Documentation**: aidlc-docs/만(markdown 요약)
- 코드 생성 전 **워크스페이스 루트**를 aidlc-state.md에서 읽기

**프로젝트 유형별 구조 패턴**:
- **Brownfield**: 기존 구조 사용(예: `src/main/java/`, `lib/`, `pkg/`)
- **Greenfield 단일 단위**: 워크스페이스 루트에 `src/`, `tests/`, `config/`
- **Greenfield 다중 단위(마이크로서비스)**: `{unit-name}/src/`, `{unit-name}/tests/`
- **Greenfield 다중 단위(모놀리스)**: `src/{unit-name}/`, `tests/{unit-name}/`

### Brownfield 파일 수정 규칙
- 생성 전 파일 존재 여부 확인
- 있으면: 제자리 수정(`ClassName_modified.java` 같은 복사본 생성 금지)
- 없으면: 새 파일 생성
- 생성 후 중복 파일 없음 확인(Step 12)

### Planning 단계 규칙
- 모든 생성 활동에 대해 명시적 번호 스텝 작성
- 계획에 스토리 추적 가능성 포함
- 단위 맥락과 의존성 문서화
- 생성 전 명시적 사용자 승인

### Generation 단계 규칙
- **하드코딩 로직 금지**: 단위 계획에 쓰인 것만 실행
- **계획을 정확히 따름**: 스텝 순서에서 벗어나지 않음
- **체크박스 갱신**: 각 스텝 완료 직후 [x] 표시
- **스토리 추적**: 기능 구현 시 단위 스토리 [x] 표시
- **의존성 존중**: 단위 의존성이 충족될 때만 구현

### 자동화 친화적 코드 규칙
웹·모바일·데스크톱 UI 코드 생성 시 요소가 자동화에 적합하도록 합니다:
- 상호작용 요소(버튼, 입력, 링크, 폼)에 `data-testid` 속성 추가
- 일관된 이름: `{component}-{element-role}` (예: `login-form-submit-button`, `user-list-search-input`)
- 렌더마다 바뀌는 동적·자동 생성 ID는 피함
- 요소 목적이 바뀔 때만 `data-testid` 값 변경 등 안정적으로 유지

## 완료 기준
- 완전한 단위 코드 생성 계획이 작성·승인됨
- 단위 코드 생성 계획의 모든 스텝이 [x]
- 계획에 따라 모든 단위 스토리가 구현됨
- 모든 코드와 테스트가 생성됨(테스트는 Build & Test 단계에서 실행)
- 배포 산출물이 생성됨
- 빌드·검증 준비가 된 완전한 단위
