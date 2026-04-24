# React JSX 마크업 가이드 (정적 컴포넌트만)

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
- [ ] 컴포넌트 디렉토리 (`src/components/`, atomic 또는 feature 기반)
- [ ] 함수형 컴포넌트 패턴 (props만, 상태 X, 이벤트 X)
- [ ] Props 타입 정의 (TypeScript 또는 PropTypes)
- [ ] CSS 전략 (Tailwind / CSS Modules / Styled Components)
- [ ] 디자인 토큰 → Tailwind config 또는 CSS 변수
- [ ] 컴포넌트 합성 (children 패턴)
- [ ] 반응형 (Tailwind responsive 또는 미디어 쿼리)
- [ ] 접근성 (semantic HTML, aria-*)
- [ ] Storybook 셋업 (선택)
- [ ] Figma → React 컴포넌트 변환 패턴

## 절대 금지 (interaction-skill의 영역)
- useState, useEffect 등 hooks 사용 금지 (props만)
- onClick, onChange 등 이벤트 핸들러 정의 금지 (props로 받기만)
- fetch, axios 등 API 호출 금지

## 참조
- 사용자 글로벌 CLAUDE.md의 일반 JS 규칙 (var 금지, const 기본)
