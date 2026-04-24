# React + Tailwind 마크업 가이드

> ⚠️ 골격 단계 — Phase 8에서 채울 예정.
> 사용자 프로젝트(web-admin/web-store)에서 즉시 필요.

## 채울 항목 (TODO)
- [ ] Tailwind v3 vs v4 차이 (v4: `@theme` 블록 + native oklch 지원)
- [ ] tailwind.config.ts (v3) vs CSS `@theme` (v4) 토큰 정의
- [ ] 디자인 토큰 → Tailwind 변환 (figma-react-lite의 `extract-tokens.sh` 활용)
- [ ] 반응형 Tier 1 (Desktop만 → mobile-first 휴리스틱) — figma-react-lite 패턴 흡수
- [ ] 반응형 Tier 2 (Figma에 Mobile/Tablet 디자인 있음) — figma-react-lite 패턴 흡수
- [ ] mobile-first 변환표 (Desktop 패턴 → Mobile-first 클래스):
  - 3~4열 그리드 → `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3`
  - 좌우 flex-row → `flex flex-col md:flex-row`
  - 큰 타이포 → `text-3xl md:text-5xl lg:text-6xl`
  - 등 (figma-react-lite section-worker.md 참조)
- [ ] DS 인벤토리 패턴 (공통 컴포넌트 사전 식별 → required_imports로 강제) — figma-react-lite 흡수
- [ ] SVG 배치 패턴 (부모 div + 원본 사이즈 img)
- [ ] 시맨틱 HTML 강제 (`<section>`, `<header>`, `<nav>`, `<h1>~<h3>`, `<button>`)
- [ ] 터치 타겟 최소 44×44px

## 활용 가능 자산
- `scripts/check-token-usage.mjs` — hex literal 검출 (G4)
- `scripts/check-text-ratio.mjs` — text-baked raster 차단 (G6)

## react/_common.md 와의 관계
- `_common.md`: JSX 규칙, props, a11y, dumb 컴포넌트 원칙
- `tailwind.md`: 위에 추가로 Tailwind 토큰 + 반응형 + 디자인 토큰 매핑
- 두 파일 모두 로드되어야 완전한 가이드
