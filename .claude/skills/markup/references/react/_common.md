# React JSX 마크업 공통 가이드 (정적 컴포넌트만)

> ⚠️ 골격 단계 — 첫 슬라이스 실측 시 채울 예정. figma-react-lite 자산 흡수가 revert되어 본문 제거됨.

publisher 에이전트가 React 계열 스택(react, react-native, nextjs)에서 작업 시 항상 로드. 변형별 추가 가이드는 `tailwind/`, `nativewind.md` 등.

## 채울 항목 (TODO)
- [ ] dumb 컴포넌트 원칙 (props만, 상태/이벤트/API 호출 금지)
- [ ] 시맨틱 HTML 강제 패턴 (`<section>`, `<header>`, `<nav>`, `<button>` 등)
- [ ] 텍스트는 JSX 트리에 (raster baked-in 텍스트 금지)
- [ ] 타입 안전성 (`any`/`unknown` 금지, props는 readonly interface)
- [ ] 디렉토리 구조 (ui/, sections/, layout/)
- [ ] 함수형 컴포넌트 패턴
- [ ] 컴포넌트 합성 (children 패턴)
- [ ] SVG / 이미지 배치 패턴 (부모 div + 원본 사이즈 img)
- [ ] DS 인벤토리 패턴 (공통 컴포넌트 사전 식별)
- [ ] 글로벌 CLAUDE.md 준수 (var 금지, const 기본, 템플릿 리터럴)

## 절대 금지 (interaction 스킬의 영역)
- useState, useEffect 등 hooks 사용 금지
- onClick, onChange 등 이벤트 핸들러 정의 금지 (props로 받기만)
- fetch, axios 등 API 호출 금지

## 결정 보류 (실측 후)
- 자동 검증 도구 도입 — eslint-plugin-react, jsx-a11y 등
- DS 인벤토리 자동화 vs 수동 명시
