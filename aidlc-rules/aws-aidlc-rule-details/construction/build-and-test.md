# Build and Test

**목적**: 모든 단위를 빌드하고 포괄적인 테스트 전략을 실행합니다

## 전제 조건
- 모든 단위에 대해 Code Generation이 완료되어야 함
- 모든 코드 산출물이 생성되어야 함
- 프로젝트가 빌드 및 테스트 준비가 되어 있어야 함

---

## Step 1: 테스트 요구사항 분석

프로젝트를 분석해 적절한 테스트 전략을 결정합니다:
- **Unit tests**: 코드 생성 중 단위별로 이미 생성됨
- **Integration tests**: 단위/서비스 간 상호작용 테스트
- **Performance tests**: 부하, 스트레스, 확장성 테스트
- **End-to-end tests**: 전체 사용자 워크플로
- **Contract tests**: 서비스 간 API 계약 검증
- **Security tests**: 취약점 스캔, 침투 테스트

---

## Step 2: 빌드 지침 생성

`aidlc-docs/construction/build-and-test/build-instructions.md` 생성:

```markdown
# Build Instructions

## Prerequisites
- **Build Tool**: [Tool name and version]
- **Dependencies**: [List all required dependencies]
- **Environment Variables**: [List required env vars]
- **System Requirements**: [OS, memory, disk space]

## Build Steps

### 1. Install Dependencies
\`\`\`bash
[Command to install dependencies]
# Example: npm install, mvn dependency:resolve, pip install -r requirements.txt
\`\`\`

### 2. Configure Environment
\`\`\`bash
[Commands to set up environment]
# Example: export variables, configure credentials
\`\`\`

### 3. Build All Units
\`\`\`bash
[Command to build all units]
# Example: mvn clean install, npm run build, brazil-build
\`\`\`

### 4. Verify Build Success
- **Expected Output**: [Describe successful build output]
- **Build Artifacts**: [List generated artifacts and locations]
- **Common Warnings**: [Note any acceptable warnings]

## Troubleshooting

### Build Fails with Dependency Errors
- **Cause**: [Common causes]
- **Solution**: [Step-by-step fix]

### Build Fails with Compilation Errors
- **Cause**: [Common causes]
- **Solution**: [Step-by-step fix]
```

---

## Step 3: 단위 테스트 실행 지침 생성

`aidlc-docs/construction/build-and-test/unit-test-instructions.md` 생성:

```markdown
# Unit Test Execution

## Run Unit Tests

### 1. Execute All Unit Tests
\`\`\`bash
[Command to run all unit tests]
# Example: mvn test, npm test, pytest tests/unit
\`\`\`

### 2. Review Test Results
- **Expected**: [X] tests pass, 0 failures
- **Test Coverage**: [Expected coverage percentage]
- **Test Report Location**: [Path to test reports]

### 3. Fix Failing Tests
If tests fail:
1. Review test output in [location]
2. Identify failing test cases
3. Fix code issues
4. Rerun tests until all pass
```

---

## Step 4: 통합 테스트 지침 생성

`aidlc-docs/construction/build-and-test/integration-test-instructions.md` 생성:

```markdown
# Integration Test Instructions

## Purpose
Test interactions between units/services to ensure they work together correctly.

## Test Scenarios

### Scenario 1: [Unit A] → [Unit B] Integration
- **Description**: [What is being tested]
- **Setup**: [Required test environment setup]
- **Test Steps**: [Step-by-step test execution]
- **Expected Results**: [What should happen]
- **Cleanup**: [How to clean up after test]

### Scenario 2: [Unit B] → [Unit C] Integration
[Similar structure]

## Setup Integration Test Environment

### 1. Start Required Services
\`\`\`bash
[Commands to start services]
# Example: docker-compose up, start test database
\`\`\`

### 2. Configure Service Endpoints
\`\`\`bash
[Commands to configure endpoints]
# Example: export API_URL=http://localhost:8080
\`\`\`

## Run Integration Tests

### 1. Execute Integration Test Suite
\`\`\`bash
[Command to run integration tests]
# Example: mvn integration-test, npm run test:integration
\`\`\`

### 2. Verify Service Interactions
- **Test Scenarios**: [List key integration test scenarios]
- **Expected Results**: [Describe expected outcomes]
- **Logs Location**: [Where to check logs]

### 3. Cleanup
\`\`\`bash
[Commands to clean up test environment]
# Example: docker-compose down, stop test services
\`\`\`
```

---

## Step 5: 성능 테스트 지침 생성(해당 시)

`aidlc-docs/construction/build-and-test/performance-test-instructions.md` 생성:

```markdown
# Performance Test Instructions

## Purpose
Validate system performance under load to ensure it meets requirements.

## Performance Requirements
- **Response Time**: < [X]ms for [Y]% of requests
- **Throughput**: [X] requests/second
- **Concurrent Users**: Support [X] concurrent users
- **Error Rate**: < [X]%

## Setup Performance Test Environment

### 1. Prepare Test Environment
\`\`\`bash
[Commands to set up performance testing]
# Example: scale services, configure load balancers
\`\`\`

### 2. Configure Test Parameters
- **Test Duration**: [X] minutes
- **Ramp-up Time**: [X] seconds
- **Virtual Users**: [X] users

## Run Performance Tests

### 1. Execute Load Tests
\`\`\`bash
[Command to run load tests]
# Example: jmeter -n -t test.jmx, k6 run script.js
\`\`\`

### 2. Execute Stress Tests
\`\`\`bash
[Command to run stress tests]
# Example: gradually increase load until failure
\`\`\`

### 3. Analyze Performance Results
- **Response Time**: [Actual vs Expected]
- **Throughput**: [Actual vs Expected]
- **Error Rate**: [Actual vs Expected]
- **Bottlenecks**: [Identified bottlenecks]
- **Results Location**: [Path to performance reports]

## Performance Optimization

If performance doesn't meet requirements:
1. Identify bottlenecks from test results
2. Optimize code/queries/configurations
3. Rerun tests to validate improvements
```

---

## Step 6: 추가 테스트 지침 생성(필요 시)

프로젝트 요구사항에 따라 추가 테스트 지침 파일을 생성합니다:

### Contract Tests (마이크로서비스용)
`aidlc-docs/construction/build-and-test/contract-test-instructions.md` 생성:
- 서비스 간 API 계약 검증
- 소비자 주도 계약 테스트
- 스키마 검증

### Security Tests
`aidlc-docs/construction/build-and-test/security-test-instructions.md` 생성:
- 취약점 스캔
- 의존성 보안 검사
- 인증/권한 테스트
- 입력 검증 테스트

### End-to-End Tests
`aidlc-docs/construction/build-and-test/e2e-test-instructions.md` 생성:
- 전체 사용자 워크플로 테스트
- 서비스 간 시나리오
- UI 테스트(해당 시)

---

## Step 7: 테스트 요약 생성

`aidlc-docs/construction/build-and-test/build-and-test-summary.md` 생성:

```markdown
# Build and Test Summary

## Build Status
- **Build Tool**: [Tool name]
- **Build Status**: [Success/Failed]
- **Build Artifacts**: [List artifacts]
- **Build Time**: [Duration]

## Test Execution Summary

### Unit Tests
- **Total Tests**: [X]
- **Passed**: [X]
- **Failed**: [X]
- **Coverage**: [X]%
- **Status**: [Pass/Fail]

### Integration Tests
- **Test Scenarios**: [X]
- **Passed**: [X]
- **Failed**: [X]
- **Status**: [Pass/Fail]

### Performance Tests
- **Response Time**: [Actual] (Target: [Expected])
- **Throughput**: [Actual] (Target: [Expected])
- **Error Rate**: [Actual] (Target: [Expected])
- **Status**: [Pass/Fail]

### Additional Tests
- **Contract Tests**: [Pass/Fail/N/A]
- **Security Tests**: [Pass/Fail/N/A]
- **E2E Tests**: [Pass/Fail/N/A]

## Overall Status
- **Build**: [Success/Failed]
- **All Tests**: [Pass/Fail]
- **Ready for Operations**: [Yes/No]

## Next Steps
[If all pass]: Ready to proceed to Operations phase for deployment planning
[If failures]: Address failing tests and rebuild
```

---

## Step 8: 상태 추적 갱신

`aidlc-docs/aidlc-state.md` 갱신:
- Build and Test 단계를 완료로 표시
- 현재 상태 갱신

---

## Step 9: 사용자에게 결과 제시

다음 구조로 완료 메시지를 제시합니다:
     1. **완료 알림** (필수): 항상 다음으로 시작:

```markdown
# 🔨 Build and Test Complete
```

     2. **AI 요약** (선택): 빌드 및 테스트 결과에 대한 구조화된 글머리 요약
        - 형식: "Build and test has completed with the following results:"
        - 빌드 상태와 산출물 나열
        - 범주별 테스트 결과 나열(unit, integration, performance 등)
        - 생성된 지침 파일 나열
        - 워크플로 지침은 넣지 않음("please review", "let me know" 등)
        - 사실 위주로 유지
     3. **형식화된 워크플로 메시지** (필수): 항상 다음 형식으로 끝냄:

```markdown
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the build and test summary at: `aidlc-docs/construction/build-and-test/build-and-test-summary.md`



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the build and test instructions based on your review
> ✅ **Approve & Continue** - Approve build and test results and proceed to **Operations**

---
```

---

## Step 10: 상호작용 로깅

**필수**: `aidlc-docs/audit.md`에 단계 완료를 기록합니다:

```markdown
## Build and Test Stage
**Timestamp**: [ISO timestamp]
**Build Status**: [Success/Failed]
**Test Status**: [Pass/Fail]
**Files Generated**:
- build-instructions.md
- unit-test-instructions.md
- integration-test-instructions.md
- performance-test-instructions.md
- build-and-test-summary.md

---
```
