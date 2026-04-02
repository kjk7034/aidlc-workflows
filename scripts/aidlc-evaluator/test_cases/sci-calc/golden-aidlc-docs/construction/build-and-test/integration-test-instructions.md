# 통합 테스트 지침

## 목적
7개 API 도메인 전체에 요청→route→engine→response 파이프라인을 테스트합니다.

## 테스트 시나리오

### 시나리오 1: 산술 연산
- **엔드포인트**: POST `/api/v1/arithmetic/{add,subtract,multiply,divide,modulo,abs,negate}`
- **테스트**: 통합 테스트 12개
- **포함**: 이항 연산, 단항 연산, 0으로 나눗셈, 잘못된 입력, NaN 거부, 404

### 시나리오 2: 거듭제곱과 루트
- **엔드포인트**: POST `/api/v1/powers/{power,sqrt,cbrt,square,nth_root}`
- **테스트**: 통합 테스트 9개
- **포함**: 모든 연산, 정의역 오류, 오버플로, 404

### 시나리오 3: 삼각함수
- **엔드포인트**: POST `/api/v1/trigonometry/{sin,cos,tan,asin,acos,atan,atan2,sinh,cosh,tanh,asinh,acosh,atanh}`
- **테스트**: 통합 테스트 8개
- **포함**: 라디안/도, 정의역 오류, 쌍곡선 함수, 404

### 시나리오 4: 로그
- **엔드포인트**: POST `/api/v1/logarithmic/{ln,log10,log2,log,exp}`
- **테스트**: 통합 테스트 9개
- **포함**: 모든 연산, 정의역 오류, 오버플로, 404

### 시나리오 5: 통계
- **엔드포인트**: POST `/api/v1/statistics/{mean,median,mode,stdev,variance,pstdev,pvariance,min,max,sum,count}`
- **테스트**: 통합 테스트 14개
- **포함**: 모든 연산, 정의역 오류, 빈 입력 검증, 404

### 시나리오 6: 상수
- **엔드포인트**: GET `/api/v1/constants/` 및 GET `/api/v1/constants/{name}`
- **테스트**: 통합 테스트 4개
- **포함**: 단일 상수 조회, 전체 목록, 알 수 없는 상수 404

### 시나리오 7: 변환 및 헬스
- **엔드포인트**: POST `/api/v1/conversions/{angle,temperature,length,weight}`, GET `/health`
- **테스트**: 통합 테스트 7개
- **포함**: 모든 변환 유형, 알 수 없는 카테고리, 알 수 없는 단위, 헬스 체크

## 통합 테스트 실행
```bash
# 통합 테스트는 단위 테스트와 동일한 테스트 파일에 있음
# conftest.py의 `client` fixture 사용
set PYTHONPATH=src
python -m pytest tests/ -v -k "API" -p no:anyio -p no:asyncio
```

## 예상 결과
- **통합 테스트 총계**: 63
- **모두 통과**: ✅
- **응답 형식 검증**: status, operation, inputs, result(또는 error)
