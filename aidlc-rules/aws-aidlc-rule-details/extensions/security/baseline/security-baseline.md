# Baseline Security Rules

## 개요
이 보안 규칙은 모든 AI-DLC phase에 걸쳐 적용되는 **필수** 횡단 제약입니다. 선택적 안내가 아니라 — 질문 생성, 설계 산출물, 코드 생성, 완료 메시지 제시 시 단계가 **반드시** 준수해야 하는 하드 제약입니다.

**강제 적용**: 해당하는 각 단계에서 모델은 사용자에게 단계 완료 메시지를 제시하기 **전에** 이 규칙 준수를 검증해야 합니다.

### 차단 보안 이슈 동작
**차단 보안 이슈**는 다음을 의미합니다:
1. 이슈는 단계 완료 메시지의 "Security Findings" 섹션에 SECURITY 규칙 ID와 설명과 함께 **반드시** 나열됩니다
2. 차단 이슈가 모두 해소될 때까지 단계는 "Continue to Next Stage" 옵션을 **제시하면 안 됩니다**
3. 모델은 무엇을 바꿔야 하는지 명확히 설명한 **"Request Changes" 옵션만** 제시해야 합니다
4. 이슈는 SECURITY 규칙 ID, 설명, 단계 맥락과 함께 `aidlc-docs/audit.md`에 **반드시** 기록됩니다

SECURITY 규칙이 현재 프로젝트에 적용되지 않으면(예: 데이터 저장소가 없을 때 SECURITY-01), 규정 준수 요약에서 **N/A**로 표시합니다 — 이는 차단 이슈가 아닙니다.

### 기본 강제
이 문서의 모든 규칙은 기본적으로 **차단**입니다. 어떤 규칙의 검증 기준도 충족되지 않으면 차단 보안 이슈입니다 — 위에 정의된 차단 이슈 동작을 따릅니다.

### 검증 기준 형식
이 문서의 검증 항목은 준수 점검을 설명하는 일반 글머리 기호입니다. 단계 계획 파일에 쓰이는 `- [ ]` / `- [x]` 진행 체크박스와는 다릅니다. 검토 시 각 항목을 준수/비준수로 평가합니다.

---

## Rule SECURITY-01: 저장 시 및 전송 시 암호화

**규칙**: 모든 데이터 지속 저장소(데이터베이스, 객체 스토리지, 파일 시스템, 캐시 또는 이에 상당하는 것)는 다음을 **반드시** 갖춰야 합니다:
- 관리형 키 서비스 또는 고객 관리 키를 사용한 저장 시 암호화 활성화
- 전송 시 암호화 강제(저장소 입출력 모든 데이터 이동에 TLS 1.2+)

**검증**:
- 암호화 설정 블록 없이 저장소 리소스를 정의하지 않음
- 암호화되지 않은 프로토콜을 사용하는 DB 연결 문자열 없음
- 객체 스토리지가 저장 시 암호화를 강제하고 정책으로 비 TLS 요청 거부
- DB 인스턴스에 스토리지 암호화가 켜져 있고 TLS 연결을 강제

---

## Rule SECURITY-02: 네트워크 중간 계층의 액세스 로깅

**규칙**: 외부 트래픽을 처리하는 모든 네트워크 대면 중간 계층은 액세스 로깅이 **반드시** 활성화되어야 합니다. 여기에는 다음이 포함됩니다:
- 로드 밸런서 → 지속 저장소로의 액세스 로그
- API 게이트웨이 → 중앙 로그 서비스로의 실행 로그 및 액세스 로그
- CDN 배포 → 표준 로깅 또는 실시간 로그

**검증**:
- 액세스 로깅 없이 로드 밸런서 리소스를 정의하지 않음
- 액세스 로깅 없이 API 게이트웨이 스테이지를 정의하지 않음
- 로깅 설정 없이 CDN 배포를 정의하지 않음

---

## Rule SECURITY-03: 애플리케이션 수준 로깅

**규칙**: 배포된 모든 애플리케이션 컴포넌트는 구조화된 로깅 인프라를 **반드시** 포함해야 합니다:
- 로깅 프레임워크가 **반드시** 구성됨
- 로그 출력은 **반드시** 중앙 로그 서비스로 전달됨
- 로그에는 **반드시** 포함: 타임스탬프, 상관/요청 ID, 로그 레벨, 메시지
- 민감 데이터(비밀번호, 토큰, PII)는 로그 출력에 **나와서는 안 됨**

**검증**:
- 모든 서비스/함수 진입점에 구성된 로거 포함
- 프로덕션 코드에서 주 로깅 수단으로 임시 로깅 문만 사용하지 않음
- 로그 구성이 출력을 중앙 로그 서비스로 라우팅
- 비밀, 토큰, PII가 로그에 남지 않음

---

## Rule SECURITY-04: 웹 애플리케이션용 HTTP 보안 헤더

**규칙**: HTML을 제공하는 모든 엔드포인트에 다음 HTTP 응답 헤더를 **반드시** 설정해야 합니다:

| Header | Required Value |
|---|---|
| `Content-Security-Policy` | Define a restrictive policy (at minimum: `default-src 'self'`) |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` (or `SAMEORIGIN` if framing is required) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |

**참고**: `X-XSS-Protection`은 최신 브라우저에서 폐기되었습니다. 대신 `Content-Security-Policy`를 사용하세요.

**검증**:
- 미들웨어 또는 응답 인터셉터가 필수 헤더 모두 설정
- CSP 정책이 문서화된 근거 없이 `unsafe-inline` 또는 `unsafe-eval`을 사용하지 않음
- HSTS max-age가 최소 31536000(1년)

---

## Rule SECURITY-05: 모든 API 매개변수에 대한 입력 검증

**규칙**: 모든 API 엔드포인트(REST, GraphQL, gRPC, WebSocket)는 처리 전에 모든 입력 매개변수를 **반드시** 검증해야 합니다. 검증에는 **반드시** 다음이 포함됩니다:
- **유형 검사**: 예상치 못한 유형 거부
- **길이/크기 한계**: 문자열 최대 길이, 배열·페이로드 최대 크기 강제
- **형식 검증**: 구조화된 입력(이메일, 날짜, ID)에 허용 목록(정규식 또는 스키마) 사용
- **새니타이즈**: XSS 방지를 위해 사용자 제공 문자열의 HTML/스크립트 이스케이프 또는 거부
- **인젝션 방지**: 모든 DB 작업에 매개변수화된 쿼리 사용(문자열 연결 금지)

**검증**:
- 모든 API 핸들러가 검증 라이브러리 또는 스키마 사용
- 원시 사용자 입력을 SQL, NoSQL, OS 명령에 연결하지 않음
- 문자열 입력에 명시적 최대 길이 제약
- 요청 본문 크기 제한이 프레임워크 또는 게이트웨이 수준에서 구성됨

---

## Rule SECURITY-06: 최소 권한 액세스 정책

**규칙**: 모든 신원·액세스 관리 정책, 역할, 권한 경계는 최소 권한을 **반드시** 따라야 합니다:
- 구체적인 리소스 식별자 사용 — API가 리소스 수준 권한을 지원하지 않는 경우가 아니면 와일드카드 리소스 **금지**(예외는 문서화)
- 구체적인 액션 사용 — 와일드카드 액션 **금지**
- 가능하면 조건으로 범위 제한
- 읽기와 쓰기 권한을 별도 정책 문으로 분리

**검증**:
- 문서화된 예외 없이 와일드카드 액션이나 와일드카드 리소스를 정책에 포함하지 않음
- 서비스 역할이 실제 호출보다 넓은 권한을 갖지 않음
- 가능하면 인라인 정책 대신 관리형 정책 사용
- 모든 역할에 특정 서비스 또는 계정으로 범위가 지정된 신뢰 정책

---

## Rule SECURITY-07: 제한적 네트워크 구성

**규칙**: 모든 네트워크 구성(보안 그룹, 네트워크 ACL, 라우트 테이블)은 기본 거부를 **반드시** 따라야 합니다:
- 방화벽 규칙: 애플리케이션에 필요한 특정 포트만 개방
- 공개 로드 밸런서의 80/443이 아닌 한, 출발지 `0.0.0.0/0`인 인바운드 규칙 없음
- 명시적으로 정당화되지 않는 한 모든 포트에 `0.0.0.0/0`인 아웃바운드 규칙 없음
- 프라이빗 서브넷에 인터넷 게이트웨이 직접 경로 **금지**
- 가능하면 클라우드 서비스 액세스에 프라이빗 엔드포인트 사용

**검증**:
- 공개 로드 밸런서의 80/443 외 포트에서 인바운드 `0.0.0.0/0`을 허용하는 방화벽 규칙 없음
- DB·애플리케이션 방화벽 규칙이 출발지를 특정 CIDR 또는 보안 그룹 참조로 제한
- 프라이빗 서브넷이 NAT 게이트웨이를 통해 라우팅(인터넷 게이트웨이 아님)
- 트래픽이 많은 클라우드 서비스 호출에 프라이빗 엔드포인트 사용

---

## Rule SECURITY-08: 애플리케이션 수준 액세스 제어

**규칙**: 리소스에 액세스하거나 변경하는 모든 애플리케이션 엔드포인트는 애플리케이션 레이어에서 권한 검사를 **반드시** 시행해야 합니다:
- **기본 거부**: 공개로 명시되지 않은 한 모든 라우트/엔드포인트는 인증 **필수**
- **객체 수준 권한**: ID로 리소스를 참조하는 모든 요청은 요청 사용자/주체가 해당 리소스를 소유하거나 액세스 권한이 있는지 **반드시** 검증(IDOR 방지)
- **기능 수준 권한**: 관리·특권 작업은 호출자의 역할/권한을 서버 측에서 **반드시** 검증 — 클라이언트 측 숨김에 의존 금지
- **CORS 정책**: 교차 출처 리소스 공유는 명시적으로 허용된 출처로만 제한 — 인증된 엔드포인트에 `Access-Control-Allow-Origin: *` **금지**
- **토큰 검증**: JWT 또는 세션 토큰은 매 요청마다 서버 측에서 **반드시** 검증(서명, 만료, audience, issuer)

**검증**:
- 모든 컨트롤러/핸들러에 권한 미들웨어 또는 가드 적용
- 호출자의 소유권 또는 권한 검증 없이 리소스 ID에 대한 데이터를 반환하는 엔드포인트 없음
- 관리/특권 라우트에 서버 측 명시적 역할 검사
- 인증된 엔드포인트에서 CORS가 와일드카드 출처를 사용하지 않음
- 토큰 검증이 로그인 시에만이 아니라 매 요청마다 서버 측에서 수행

---

## Rule SECURITY-09: 보안 강화 및 오설정 방지

**규칙**: 배포된 모든 구성 요소는 강화 기준선을 **반드시** 따라야 합니다:
- **기본 자격 증명 금지**: 배포 전 기본 사용자명/비밀번호는 **반드시** 변경하거나 비활성화
- **최소 설치**: 사용하지 않는 기능, 프레임워크, 샘플 애플리케이션, 문서 엔드포인트 제거 또는 비활성화
- **오류 처리**: 프로덕션 오류 응답은 엔드 사용자에게 스택 트레이스, 내부 경로, 프레임워크 버전, DB 세부를 **노출하면 안 됨**
- **디렉터리 목록**: 웹 서버는 디렉터리 목록을 **반드시** 비활성화
- **클라우드 스토리지**: 명시적 요구·문서화가 없으면 클라우드 객체 스토리지는 공개 액세스를 **반드시** 차단
- **패치 관리**: 런타임, 프레임워크, OS 이미지는 현재 지원되는 버전을 **반드시** 사용

**검증**:
- 구성 파일, 환경 변수, IaC 템플릿에 기본 자격 증명 없음
- 프로덕션 오류 응답은 일반 메시지만 반환(스택 트레이스나 내부 세부 없음)
- 문서화된 예외가 없으면 클라우드 객체 스토리지에 공개 액세스 차단
- 샘플/데모 애플리케이션이나 기본 페이지 배포 없음
- 프레임워크와 런타임 버전이 현재이며 지원됨


---

## Rule SECURITY-10: 소프트웨어 공급망 보안

**규칙**: 모든 프로젝트는 소프트웨어 공급망을 **반드시** 관리해야 합니다:
- **의존성 고정**: 모든 의존성은 정확한 버전 또는 lock 파일을 **반드시** 사용
- **취약점 스캔**: 의존성 취약점 스캐너를 **반드시** 구성
- **미사용 의존성 금지**: 실제로 쓰지 않는 패키지 제거
- **신뢰 출처만**: 공식 레지스트리 또는 검증된 프라이빗 레지스트리에서만 의존성을 가져옴 — 검증되지 않은 제3자 출처 금지
- **SBOM**: 프로덕션 배포를 위해 Software Bill of Materials를 **반드시** 생성
- **CI/CD 무결성**: 빌드 파이프라인은 고정된 도구 버전과 검증된 베이스 이미지를 **반드시** 사용 — 프로덕션 Dockerfile 또는 CI 설정에 `latest` 태그 금지

**검증**:
- lock 파일이 존재하고 버전 관리에 커밋됨
- CI/CD에 의존성 취약점 스캔 단계가 포함되거나 빌드 지침에 문서화됨
- 미사용 또는 방치된 의존성 없음
- 프로덕션용 Dockerfile과 CI 설정에 `latest` 또는 고정되지 않은 이미지 태그 없음
- 의존성은 공식 또는 검증된 레지스트리에서 가져옴

---

## Rule SECURITY-11: 안전한 설계 원칙

**규칙**: 애플리케이션 설계는 처음부터 보안을 **반드시** 반영해야 합니다:
- **관심사 분리**: 보안에 중요한 로직(인증, 권한, 결제 처리)은 전용 모듈에 **반드시** 격리 — 코드베이스 전체에 흩뿌리지 않음
- **다층 방어**: 단일 통제만이 유일한 방어선이 되어서는 안 됨 — 검증 + 권한 + 암호화 등 층층이 통제
- **속도 제한**: 공개 엔드포인트는 남용 방지를 위해 속도 제한 또는 스로틀링을 **반드시** 구현
- **비즈니스 로직 남용**: 설계는 행복 경로만이 아니라 오용 사례를 **반드시** 고려

**검증**:
- 보안에 중요한 로직이 전용 모듈 또는 서비스에 캡슐화됨
- 공개 API에 속도 제한이 구성됨
- 설계 문서에 오용/남용 시나리오가 최소 하나 다루어짐

---

## Rule SECURITY-12: Authentication and Credential Management

**Rule**: Every application with user authentication MUST implement:
- **Password policy**: Minimum 8 characters, check against breached password lists
- **Credential storage**: Passwords MUST be hashed using adaptive algorithms — never weak or non-adaptive hashing
- **Multi-factor authentication**: MFA MUST be supported for administrative accounts and SHOULD be available for all users
- **Session management**: Sessions MUST have server-side expiration, be invalidated on logout, and use secure/httpOnly/sameSite cookie attributes
- **Brute-force protection**: Login endpoints MUST implement account lockout, progressive delays, or CAPTCHA after repeated failures
- **No hardcoded credentials**: No passwords, API keys, or secrets in source code or IaC templates — use a secrets manager

**Verification**:
- Password hashing uses adaptive algorithms (not weak or non-adaptive hashing)
- Session cookies set `Secure`, `HttpOnly`, and `SameSite` attributes
- Login endpoints have brute-force protection (lockout, delay, or CAPTCHA)
- No hardcoded credentials in source code or configuration files
- MFA is supported for admin accounts
- Sessions are invalidated on logout and have a defined expiration

---

## Rule SECURITY-13: Software and Data Integrity Verification

**Rule**: Systems MUST verify the integrity of software and data:
- **Deserialization safety**: Untrusted data MUST NOT be deserialized without validation — use safe deserialization libraries or allowlists of permitted types
- **Artifact integrity**: Downloaded dependencies, plugins, and updates MUST be verified via checksums or digital signatures
- **CI/CD pipeline security**: Build pipelines MUST restrict who can modify pipeline definitions — separate duties between code authors and deployment approvers
- **CDN and external resources**: Scripts or resources loaded from external CDNs MUST use Subresource Integrity (SRI) hashes
- **Data integrity**: Critical data modifications MUST be auditable (who changed what, when)

**Verification**:
- No unsafe deserialization of untrusted input
- External scripts include SRI integrity attributes when loaded from CDNs
- CI/CD pipeline definitions are access-controlled and changes are auditable
- Critical data changes are logged with actor, timestamp, and before/after values

---

## Rule SECURITY-14: 알림 및 모니터링

**규칙**: 로깅(SECURITY-02, SECURITY-03) 외에 시스템은 다음을 **반드시** 포함해야 합니다:
- **보안 이벤트 알림**: 반복 인증 실패, 권한 상승 시도, 비정상 위치에서의 액세스, 권한 실패 등 고가치 보안 이벤트에 알림을 **반드시** 구성
- **로그 무결성**: 로그는 추가 전용 또는 변조 증명 저장소에 **반드시** 저장 — 애플리케이션 코드가 자체 감사 로그를 삭제·수정할 수 **없음**
- **로그 보관**: 애플리케이션 규정 준수 요구에 맞는 최소 기간 동안 로그를 **반드시** 보관(기본: 최소 90일)
- **모니터링 대시보드**: 주요 운영·보안 지표에 대해 모니터링 대시보드 또는 알람 구성을 **반드시** 정의

**검증**:
- 인증 실패 및 권한 위반에 대한 알림 구성
- 애플리케이션 로그 그룹에 보관 정책 설정(최소 90일)
- 애플리케이션 역할이 자체 로그 그룹/스트림 삭제 권한 없음
- 보안 관련 이벤트(로그인 실패, 액세스 거부, 권한 변경)가 알림 생성

---

## Rule SECURITY-15: 예외 처리 및 안전 실패 기본값

**규칙**: 모든 애플리케이션은 예외 상황을 안전하게 처리해야 합니다:
- **포착 및 처리**: 모든 외부 호출(DB, API, 파일 I/O)에 명시적 오류 처리 **필수** — 프로덕션에서 처리되지 않은 promise 거부나 잡히지 않은 예외 없음
- **안전 쪽으로 실패**: 오류 시 시스템은 액세스를 거부하거나 작업을 중단해야 함 — 절대 느슨하게 실패(fail open) 금지
- **리소스 정리**: 오류 경로에서 연결, 파일 핸들, 잠금 등 리소스를 **반드시** 해제 — try/finally, using 등 동등 패턴 사용
- **사용자 대면 오류**: 사용자에게 보이는 오류 메시지는 일반적이어야 함 — 내부 세부나 시스템 정보 없음
- **전역 오류 핸들러**: 애플리케이션은 잡히지 않은 예외를 포착하고(SECURITY-03에 따라) 로깅하며 안전한 응답을 반환하는 전역/최상위 오류 핸들러를 **반드시** 가져야 함

**검증**:
- 모든 외부 호출(DB, HTTP, 파일 I/O)에 명시적 오류 처리(try/catch, .catch(), 오류 콜백)
- 애플리케이션 진입점에 전역 오류 핸들러 구성
- 오류 경로가 권한 또는 검증 검사를 우회하지 않음(fail closed)
- 오류 경로에서 리소스 정리(연결 종료, 트랜잭션 롤백)
- 애플리케이션 코드에 처리되지 않은 promise 거부나 잡히지 않은 예외 경고 없음

---

## 강제 적용 통합

이 규칙은 모든 AI-DLC 단계에 적용되는 횡단 제약입니다. 각 단계에서:
- 생성된 산출물에 대해 모든 SECURITY 규칙 검증 기준을 평가
- 단계 완료 요약에 "Security Compliance" 섹션을 포함해 각 규칙을 준수/비준수/N/A로 나열
- 어떤 규칙이든 비준수이면 차단 보안 이슈 — 개요에 정의된 차단 이슈 동작을 따름
- 설계 문서와 테스트 지침에 보안 규칙 참조 포함

---

## 부록: OWASP 참조 매핑

<!-- TODO: CRITICAL - This entire OWASP mapping table needs verification. The "2025" edition may not exist; the latest published OWASP Top 10 is 2021. Category IDs (A01-A10), numbering, and names must be validated against the actual published standard before relying on this mapping. -->
검토자를 위해 아래는 SECURITY 규칙을 OWASP Top 10 (2025) 범주에 매핑합니다:

| SECURITY Rule | OWASP Category |
|---|---|
| SECURITY-08 | A01:2025 – Broken Access Control |
| SECURITY-09 | A02:2025 – Security Misconfiguration |
| SECURITY-10 | A03:2025 – Software Supply Chain Failures |
| SECURITY-11 | A06:2025 – Insecure Design |
| SECURITY-12 | A07:2025 – Authentication Failures |
| SECURITY-13 | A08:2025 – Software or Data Integrity Failures |
| SECURITY-14 | A09:2025 – Logging & Alerting Failures |
| SECURITY-15 | A10:2025 – Mishandling of Exceptional Conditions |
