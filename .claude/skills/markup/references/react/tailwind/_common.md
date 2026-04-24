# React + Tailwind 공통 가이드 (버전 무관)

> ⚠️ 골격 단계 — 첫 슬라이스 실측 시 채울 예정. figma-react-lite 자산 흡수가 revert되어 본문 제거됨.

`react/_common.md` + `tailwind/v{major}.md` 와 함께 로드. 이 파일은 **Tailwind v3/v4 공통** 규칙 — 버전 분기 케이스는 `v3.md` / `v4.md` 참조.

## 로드 순서 (SKILL.md가 지시)

1. `react/_common.md` — JSX, props, a11y (React 공통)
2. `react/tailwind/_common.md` ← 이 파일 — Tailwind 공통 원칙
3. `react/tailwind/v{major}.md` — 감지된 major 버전 전용 (셋업 방식, 토큰 정의)

**중요**: `package.json` 의 `tailwindcss` 의존성 major 버전으로 자동 선택.

## 채울 항목 (TODO)
- [ ] 디자인 토큰 사용 원칙 (G4 게이트 — hex literal 금지, 예외 화이트리스트)
- [ ] 반응형 — Mobile-first 필수
  - [ ] Breakpoint 표준 (Tailwind 기본 sm/md/lg/xl/2xl)
  - [ ] Tier 결정 (design-spec.md의 nodeId 유무로 Tier 1 vs Tier 2)
  - [ ] Tier 1 (Desktop만): Mobile-first 변환 휴리스틱 표
  - [ ] Tier 2 (반응형 디자인 있음): 각 뷰포트 PNG 시각 확인 → className 작성
- [ ] 금지 패턴 (고정 폭 단독, overflow-visible, 터치 타겟 44px 미만 등)
- [ ] 자체 점검 체크리스트 (Mobile 가로 스크롤, 큰 타이포 overflow, 이미지 비율)
- [ ] components.md 갱신 패턴

## 결정 보류 (실측 후)
- visual regression (G1) 도입 — Playwright + pixelmatch
- 자동 검증 스크립트 (token usage, text:image ratio) — 자체 구현 vs 외부 의존
