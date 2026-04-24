---
name: markup
description: "Figma 디자인을 정적 마크업(HTML/CSS 또는 React JSX)으로 변환. project-profile.yaml의 frontend.markup에 따라 thymeleaf, react-jsx, html-vanilla로 동적 분기. 상태/이벤트/API 호출 없이 props만 받는 dumb 컴포넌트만 생성. '퍼블리싱', '마크업', 'Figma 컴포넌트화', '정적 화면', 'HTML/CSS 작성', '재퍼블리싱', '마크업 수정' 등의 요청 시 반드시 이 스킬을 사용할 것."
---

# Markup Skill — 디자인 → 정적 마크업 (스택 동적)

## When to use
- Figma 디자인을 코드 컴포넌트로 변환
- 정적 화면 마크업 (Thymeleaf 템플릿 또는 React 컴포넌트)
- 디자인 토큰화 (색상/폰트/스페이싱)

## Workflow

### Step 1: 스택 식별 + references 로드 (계층형)
1. `.claude/project-profile.yaml` 읽기
2. `frontend.framework + frontend.styling` 조합으로 분기:
   - **다중 변형 framework** (react, nextjs, vue 등) → `_common.md + {styling}.md` 둘 다 로드
     - `react + tailwind` → Read `references/react/_common.md` + `references/react/tailwind.md`
     - `react + css-modules` → Read `references/react/_common.md` + `references/react/css-modules.md` (향후)
     - `react-native + nativewind` → Read `references/react-native/_common.md` + `references/react-native/nativewind.md` (v0.2.0)
     - `nextjs + tailwind` → Read `references/nextjs/_common.md` + `references/nextjs/{router}.md` (향후)
   - **단일 변형 framework** → 평면 파일 직접 로드
     - `thymeleaf` → Read `references/thymeleaf.md`
     - `html-vanilla` → Read `references/html.md` (향후)
3. 디자인 가이드(색상/폰트) 컨텍스트 확인

### Step 2: 디자인 스펙 입력 (원본 직접 접근 X)
- `_workspace/handoff/design-spec.md` 읽기 (필수, 없으면 design 에이전트 호출)
- `_workspace/handoff/design-assets/*.png` 시각 참조 (있으면)
- design-spec.md의 "미해결 사항" 점검 — 있으면 진행 중단 + design 에이전트에 보고
- **원본 도구(Figma/PDF/PPT) 직접 호출 금지** — spec-extract가 이미 추출해놓음

### Step 3: 컴포넌트 분해 + 마크업 생성
1. 화면 → 재사용 가능 단위로 분해
2. 각 컴포넌트의 props 명세 정의
3. 정적 마크업 작성 (스택별 references 따라)
4. **동적 코드 절대 금지**: useState, onClick, fetch 모두 X

### Step 4: 컴포넌트 카탈로그 갱신
`_workspace/publisher/components.md`에 추가:
- 컴포넌트 이름
- props 목록 + 타입
- 사용 예시
- (frontend 에이전트가 이를 보고 동적 동작 추가)

## 절대 원칙 (위반 시 즉시 중단)
- 동적 코드 작성 금지 (이는 frontend 스킬의 영역)
- 디자인 토큰 무시하고 하드코딩 색상 사용 금지
- 기존 컴포넌트와 중복 생성 금지 (재사용 우선)

## 글로벌 CLAUDE.md 준수
- HTML/Thymeleaf: 속성 순서 (id → class → th:* → data-* → 이벤트)
- CSS: BEM 또는 케밥케이스, `!important` 금지, 셀렉터 깊이 3단계 이하
- fragment로 레이아웃 분리

## References (계층형 구조)
```
markup/references/
├── react/
│   ├── _common.md          ← JSX 규칙, props, a11y (모든 React 공통)
│   ├── tailwind.md         ← Tailwind v3/v4 + 반응형 + DS 인벤토리
│   ├── css-modules.md      (향후)
│   └── styled.md           (향후)
├── react-native/           (v0.2.0)
│   ├── _common.md
│   └── nativewind.md
├── nextjs/                 (향후)
│   ├── _common.md
│   ├── app-router.md       (SEO)
│   └── pages-router.md
├── thymeleaf.md            ← 평면 (단일 변형)
├── vue.md                  (향후)
└── html.md                 (향후)
```
- 다중 변형 framework: `_common.md` + 변형 .md 둘 다 로드
- 단일 변형: 평면 파일 그대로
