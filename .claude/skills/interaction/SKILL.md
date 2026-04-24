---
name: interaction
description: "정적 마크업에 동적 동작 추가 — 상태 관리, 이벤트, API 호출, 폼 검증, 라우팅. project-profile.yaml의 frontend.interaction에 따라 jquery, react, vue로 동적 분기. publisher가 만든 컴포넌트를 가져다 쓰며 마크업 자체는 변경하지 않음. '프론트 동작', 'API 연동', 'JS 작성', '상태 관리', '이벤트 핸들러', '폼 검증', '재구현', '동작 수정' 등의 요청 시 반드시 이 스킬을 사용할 것."
---

# Interaction Skill — 동적 동작 + API 연동 (스택 동적)

## When to use
- publisher 산출물에 상태/이벤트/API 호출 추가
- 폼 검증, 라우팅, 인증 흐름
- backend API 연동

## Workflow

### Step 1: 스택 식별 + references 로드 (계층형)
1. `.claude/project-profile.yaml` 읽기
2. `frontend.framework` (또는 별도 `frontend.interaction`)로 분기:
   - **다중 변형 framework** → `_common.md + {variant}.md` 로드
     - `react` 기본 → Read `references/react/_common.md`
     - `react + react-query` → 위 + Read `references/react/react-query.md` (향후)
     - `react-native` → Read `references/react-native/_common.md` (v0.2.0)
     - `nextjs` → Read `references/nextjs/_common.md` + 추가 변형 (향후)
   - **단일 변형 framework** → 평면 파일
     - `jquery` → Read `references/jquery.md`
     - `vue` → Read `references/vue.md` (향후)

### Step 2: 입력 확인
- `_workspace/handoff/openapi.yaml` (API 계약, 필수)
- `_workspace/publisher/components.md` (사용 가능 컴포넌트, 필수)
- backend 미완료 시 → openapi.yaml 기반 mock 응답 생성해서 선개발

### Step 3: 동적 코드 작성
1. 상태 관리 정의 (useState / Context / jQuery 변수)
2. 이벤트 핸들러 + API 호출
3. 응답 처리 (글로벌 응답 포맷 `{success, data, message}` 공통 처리)
4. 폼 검증, 에러 핸들링

### Step 4: 자체 검증
- 빌드 성공 (npm/yarn build, 또는 Thymeleaf+jQuery는 브라우저 동작 확인)
- API 호출 형식이 openapi.yaml과 일치하는지 확인
- 콘솔 에러 없음

### Step 5: 진행 메모
`_workspace/frontend/progress.md`에 구현한 페이지/기능 기록

## 절대 원칙 (위반 시 즉시 중단)
- 마크업 구조 직접 수정 금지 → publisher 스킬에 메시지 발송
- API 응답 형식이 yaml과 다르면 자체 우회 금지 → backend에 메시지 발송
- 계약 위반은 design에 변경 요청

## 글로벌 CLAUDE.md 준수
- `var` 금지, `const` 기본, 재할당만 `let`
- jQuery 셀렉터 변수 캐싱: `const $btn = $('.submit-btn')`
- 템플릿 리터럴 사용 (string concat 금지)
- API 응답 처리 시 `{success, data, message}` 형식 가정

## References (계층형 구조)
```
interaction/references/
├── react/
│   ├── _common.md          ← hooks, fetch, 글로벌 응답 처리
│   ├── react-query.md      (향후)
│   ├── zustand.md          (향후)
│   └── context.md          (향후)
├── react-native/           (v0.2.0)
│   ├── _common.md
│   ├── async-storage.md
│   └── navigation.md
├── nextjs/                 (향후)
│   ├── _common.md
│   └── server-actions.md
├── jquery.md               ← 평면 (단일 변형, 사내 표준)
└── vue.md                  (향후)
```
