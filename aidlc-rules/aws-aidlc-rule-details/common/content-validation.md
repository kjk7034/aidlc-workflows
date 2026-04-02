# 콘텐츠 검증 규칙

## 필수: 파일 생성 전 콘텐츠 검증

**중요**: 파싱 오류를 막기 위해 생성된 모든 콘텐츠는 파일에 쓰기 전에 검증해야 합니다.

## ASCII 다이어그램 표준

**중요**: ASCII 다이어그램이 있는 **어떤** 파일을 만들기 전에:

1. **로드** `common/ascii-diagram-standards.md`
2. **검증** 각 다이어그램:
   - 줄당 문자 수(모든 줄은 **같은 너비**여야 함)
   - **오직** 사용: `+` `-` `|` `^` `v` `<` `>` 및 공백
   - 유니코드 박스 그리기 문자 없음
   - 공백만(탭 없음)
3. **테스트** 박스 모서리가 수직으로 맞는지 정렬 확인

**패턴과 검증 체크리스트는 `common/ascii-diagram-standards.md`를 참고하세요.**

## Mermaid 다이어그램 검증

### 필수 검증 단계
1. **문법 검사**: 파일 생성 전 Mermaid 문법 검증
2. **문자 이스케이프**: 특수 문자가 올바르게 이스케이프되었는지 확인
3. **대체 콘텐츠**: Mermaid 검증 실패 시 텍스트 대안 제공

### Mermaid 검증 규칙
```markdown
## BEFORE creating any file with Mermaid diagrams:

1. Check for invalid characters in node IDs (use alphanumeric + underscore only)
2. Escape special characters in labels: " → \" and ' → \'
3. Validate flowchart syntax: node connections must be valid
4. Test diagram parsing with simple validation

## FALLBACK: If Mermaid validation fails, use text-based workflow representation
```

### 구현 패턴
```markdown
## Workflow Visualization

### Mermaid Diagram (if syntax valid)
```mermaid
[validated diagram content]
```

### Text Alternative (always include)
```
Phase 1: INCEPTION
- Stage 1: Workspace Detection (COMPLETED)
- Stage 2: Requirements Analysis (COMPLETED)
[continue with text representation]
```

## 일반 콘텐츠 검증

### 생성 전 검증 체크리스트
- [ ] 임베디드 코드 블록(Mermaid, JSON, YAML) 검증
- [ ] 특수 문자 이스케이프 확인
- [ ] 마크다운 문법 정확성 확인
- [ ] 콘텐츠 파싱 호환성 테스트
- [ ] 복잡한 요소에 대한 대체 콘텐츠 포함

### 오류 방지 규칙
1. **파일 쓰기 도구·명령 사용 전 항상 검증**: 검증되지 않은 콘텐츠는 쓰지 않음
2. **특수 문자 이스케이프**: 특히 다이어그램과 코드 블록
3. **대안 제공**: 시각 콘텐츠의 텍스트 버전 포함
4. **문법 테스트**: 복잡한 콘텐츠 구조 검증

## 검증 실패 처리

### 검증이 실패할 때
1. **오류 기록**: 무엇이 검증에 실패했는지 기록
2. **대체 콘텐츠 사용**: 텍스트 기반 대안으로 전환
3. **워크플로 계속**: 콘텐츠 검증 실패로 워크플로를 막지 않음
4. **사용자 안내**: 파싱 제약으로 인해 단순화된 콘텐츠를 사용했다고 알림
