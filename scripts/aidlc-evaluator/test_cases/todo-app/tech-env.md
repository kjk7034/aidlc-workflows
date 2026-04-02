# 기술 환경: 할 일 목록 애플리케이션

## 언어 및 패키지 관리자

- **Node.js 22** (LTS)
- 패키지 관리에 **npm**
- 프로젝트 및 스크립트 설정은 `package.json`

## 백엔드 프레임워크

- REST API 서버에 **Express.js**
- 메모리 내 데이터 저장(일반 JavaScript Map/Array) — MVP에 DB 불필요
- todo ID 생성에 **uuid** 패키지

## 프론트엔드 프레임워크

- 함수형 컴포넌트와 훅이 있는 **React 19**
- 빌드 도구 및 개발 서버로 **Vite**
- 순수 CSS(CSS 프레임워크 불필요)

## 프로젝트 구조

```
todo-app/
├── package.json
├── server/
│   ├── index.js          # Express 서버 진입점
│   ├── routes/
│   │   └── todos.js      # Todo CRUD 라우트
│   └── store.js          # 메모리 내 todo 저장소
├── client/
│   ├── index.html
│   ├── src/
│   │   ├── main.jsx      # React 진입점
│   │   ├── App.jsx       # 루트 컴포넌트
│   │   ├── components/
│   │   │   ├── TodoInput.jsx
│   │   │   ├── TodoList.jsx
│   │   │   ├── TodoItem.jsx
│   │   │   └── FilterBar.jsx
│   │   ├── hooks/
│   │   │   └── useTodos.js
│   │   └── styles/
│   │       └── App.css
│   └── vite.config.js
└── tests/
    ├── server/
    │   └── todos.test.js  # API 통합 테스트
    └── client/
        └── App.test.jsx   # 컴포넌트 테스트
```

## 테스트

- 서버·클라이언트 테스트 모두 **Vitest**
- 컴포넌트 테스트에 **React Testing Library**
- API 통합 테스트에 **supertest**
- `npm test`로 테스트 실행

## 개발 스크립트

```json
{
  "scripts": {
    "dev": "concurrently \"npm run dev:server\" \"npm run dev:client\"",
    "dev:server": "node server/index.js",
    "dev:client": "vite client",
    "build": "vite build client",
    "test": "vitest run",
    "start": "node server/index.js"
  }
}
```

## 규칙

- ES 모듈(`package.json`에 `"type": "module"`)
- 서버는 포트 3001에서 수신, Vite 개발 서버 포트 5173에서 프록시
- 모든 API 라우트는 `/api/` 접두사
- 헬스 엔드포인트는 `/health`(`/api/` 접두사 없음)
- 표준 HTTP 상태 코드: 200, 201, 400, 404, 500
