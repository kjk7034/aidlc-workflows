# 세션 연속성 템플릿

## 돌아온 사용자용 프롬프트 템플릿
사용자가 기존 AI-DLC 프로젝트 작업을 이어갈 때 다음 프롬프트를 제시합니다:

```markdown
**Welcome back! I can see you have an existing AI-DLC project in progress.**

Based on your aidlc-state.md, here's your current status:
- **Project**: [project-name]
- **Current Phase**: [INCEPTION/CONSTRUCTION/OPERATIONS]
- **Current Stage**: [Stage Name]
- **Last Completed**: [Last completed step]
- **Next Step**: [Next step to work on]

**What would you like to work on today?**

A) Continue where you left off ([Next step description])
B) Review a previous stage ([Show available stages])

[Answer]: 
```

## 필수: 세션 연속성 지침
1. 기존 프로젝트가 감지되면 **항상 먼저 aidlc-state.md를 읽습니다**
2. 워크플로 파일에서 현재 상태를 파싱해 프롬프트를 채웁니다
3. **필수: 이전 단계 산출물 로드** — 어떤 단계를 재개하기 전에도 이전 단계의 관련 산출물을 자동으로 모두 읽습니다:
   - **Reverse Engineering**: architecture.md, code-structure.md, api-documentation.md 읽기
   - **Requirements Analysis**: requirements.md, requirement-verification-questions.md 읽기
   - **User Stories**: stories.md, personas.md, story-generation-plan.md 읽기
   - **Application Design**: application-design 산출물 읽기(components.md, component-methods.md, services.md)
   - **Design (Units)**: unit-of-work.md, unit-of-work-dependency.md, unit-of-work-story-map.md 읽기
   - **단위별 설계**: functional-design.md, nfr-requirements.md, nfr-design.md, infrastructure-design.md 읽기
   - **코드 단계**: 모든 코드 파일, 계획, **그리고** 모든 이전 산출물 읽기
4. **단계별 스마트 컨텍스트 로딩**:
   - **초기 단계(Workspace Detection, Reverse Engineering)**: 워크스페이스 분석 로드
   - **Requirements/Stories**: reverse engineering + 요구사항 산출물 로드
   - **설계 단계**: 요구사항 + 스토리 + 아키텍처 + 설계 산출물 로드
   - **코드 단계**: 모든 산출물 + 기존 코드 파일 로드
5. 아키텍처 선택과 현재 단계에 맞게 옵션을 조정합니다
6. 일반적인 설명보다 **구체적인 다음 단계**를 표시합니다
7. 연속성 프롬프트를 타임스탬프와 함께 `audit.md`에 기록합니다
8. **컨텍스트 요약**: 산출물 로드 후 사용자 인지를 위해 무엇을 로드했는지 간단히 요약합니다
9. **질문하기**: 명확화나 사용자 피드백 질문은 **항상** `.md` 파일에 넣습니다. 객관식 질문을 채팅에 인라인으로 넣지 **않습니다**.

## 오류 처리
세션 재개 중 산출물이 없거나 손상된 경우 복구 절차는 [error-handling.md](error-handling.md)를 참고하세요.
