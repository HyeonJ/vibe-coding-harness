# React + Tailwind 마크업 가이드

`react/_common.md`와 함께 로드. Tailwind 토큰 + 반응형 + 디자인 토큰 매핑 추가.

## Tailwind v3 vs v4

| 항목 | v3 | v4 (권장 — 신규 프로젝트) |
|---|---|---|
| 설정 파일 | `tailwind.config.ts` | CSS의 `@theme` 블록 |
| 색공간 | sRGB hex | native oklch 지원 |
| 빌드 | PostCSS plugin | Vite/Next 네이티브 통합 |
| NativeWind 호환 | v4-aligned | (NativeWind v4는 v3 alignment) |

### v3 토큰 정의 (tailwind.config.ts)
```ts
export default {
  content: ["./src/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: {
        brand: { 500: "var(--brand-500)" },
        surface: { 100: "var(--surface-100)" },
      },
      fontFamily: {
        sans: ["Pretendard", "sans-serif"],
      },
    },
  },
};
```

### v4 토큰 정의 (CSS @theme)
```css
@import "tailwindcss";

@theme {
  --color-brand-500: oklch(0.7 0.18 250);
  --color-surface-100: oklch(0.96 0.01 90);
  --font-sans: "Pretendard", sans-serif;
}
```

→ 둘 다 `extract-tokens.sh`가 생성하는 `src/styles/tokens.css`의 `var(--*)` 변수를 그대로 활용.

## 디자인 토큰 사용 원칙 (G4 게이트)

### ✅ OK
```tsx
<div className="bg-[var(--brand-500)] text-[var(--text-primary)]">
  또는
<div className="bg-brand-500 text-text-primary">  // tailwind.config에 등록 시
```

### ❌ 금지 (G4 FAIL)
```tsx
<div className="bg-[#2563EB]">                    // hex literal 직접
<div style={{ backgroundColor: "#2563EB" }}>      // inline hex
```

**예외 화이트리스트**: `#fff` / `#000` 중립값만 허용.

## 반응형 — Mobile-first 필수

### Breakpoint 표준 (Tailwind 기본)
- Mobile: `<768px` — 클래스 prefix 없음 (기본값)
- Tablet: `md:` (`>=768px`)
- Desktop: `lg:` (`>=1024px`)

**원칙**: 기본 className = Mobile, `md:` / `lg:` 로 상향 덮어쓰기.

### Tier 결정 (design-spec.md 보고)

design-spec.md에서 화면별 nodeId 확인:

| 상태 | 경로 |
|---|---|
| Desktop + Tablet + Mobile nodeId 모두 있음 | **Tier 2** — 각 뷰포트의 Figma 디자인 충실 반영 |
| Desktop만 있음 | **Tier 1** — 휴리스틱으로 Mobile/Tablet 변환 (아래 표) |

### Tier 2: Figma에 반응형 디자인 있음
1. 각 뷰포트 PNG (`_workspace/handoff/design-assets/{name}-{tablet|mobile}.png`) Read 도구로 시각 확인
2. Mobile PNG → 기본 className에 반영
3. `md:` prefix → Tablet PNG 기준
4. `lg:` prefix → Desktop PNG 기준
5. Figma가 일부 뷰포트 미제공 시 → Tier 1 휴리스틱으로 보완

### Tier 1: Desktop만 있음 (휴리스틱)

| Desktop 패턴 | Mobile-first 클래스 |
|---|---|
| 3~4열 그리드 | `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3` |
| 2열 그리드 | `grid grid-cols-1 md:grid-cols-2` |
| 좌우 `flex-row` | `flex flex-col md:flex-row` |
| 고정 폭 `w-[1280px]` | `w-full max-w-[1280px] mx-auto px-6 md:px-12` |
| 큰 타이포 (Figma 60px+) | `text-3xl md:text-5xl lg:text-6xl` |
| 중간 타이포 (Figma 32~48px) | `text-2xl md:text-3xl lg:text-4xl` |
| 가로 Nav (5+ 링크) | 햄버거 (Mobile) → `hidden md:flex` 풀 Nav |
| Hero 배경 + 텍스트 오버레이 | Mobile은 `aspect-[4/5]` 또는 `aspect-square` 세로 |
| absolute 겹침 레이아웃 | Mobile은 relative 스택, `md:absolute md:inset-0` |
| 큰 이미지 사이드 배치 | `flex-col md:flex-row`, 이미지 `w-full md:w-1/2` |
| `gap-12` 큰 간격 | `gap-6 md:gap-12` 단계 축소 |
| `py-24` 큰 패딩 | `py-12 md:py-24` 단계 축소 |

### 금지 패턴
- ❌ 고정 폭 단독 (`w-[1280px]` 만 있고 대응 없음) → Mobile 가로 스크롤
- ❌ `overflow-visible` 로 큰 요소 유출 (section 기본 `overflow-hidden` 검토)
- ❌ `text-[...]` arbitrary 크기 Mobile/Tablet 대응 없이 단독 사용
- ❌ 터치 타겟 44px 미만 버튼/링크

### 허용 타협
- Figma에 없는 Mobile 디자인 → publisher 자체 판단으로 합리적 변환 (디자이너 역할 대행)
- Mobile에서 복잡 overlap을 스택으로 단순화 (의도 유지가 목표)
- 햄버거 메뉴 내부 디테일 단순화

## 자체 점검 (구현 직전 체크리스트)

- [ ] Mobile 375px에서 가로 스크롤 생길 요소 있나? (고정 width, 큰 이미지)
- [ ] 큰 타이포가 Mobile에서 overflow 안 하나?
- [ ] 이미지가 Mobile에서 비율 왜곡 없나? (`object-cover` / `aspect-ratio`)
- [ ] 터치 타겟 최소 44×44px 확보?
- [ ] Nav 가로 메뉴가 Mobile에서 햄버거로 전환되나?
- [ ] G4 통과 — hex literal 0개?
- [ ] design-spec.md의 디자이너 노트가 코드에 반영됐나?

## components.md 갱신

작업 완료 후 `_workspace/publisher/components.md`에 추가:

```markdown
## ProductCard

- **위치**: src/components/ui/ProductCard.tsx
- **props**:
  - name: string
  - price: number
  - imageSrc: string
  - onSelect?: (id: string) => void
- **사용 예**:
  ```tsx
  <ProductCard name="후드티" price={29000} imageSrc="..." onSelect={handleSelect} />
  ```
- **반응형**: Mobile 1열 / Tablet 2열 / Desktop 3열 그리드 환경에서 사용
- **디자인 토큰**: `var(--surface-100)`, `var(--text-primary)`
```

→ frontend 에이전트가 이 컴포넌트 import해서 동적 동작 추가 시 참조.

## 활용 자산
- `scripts/check-token-usage.mjs` — G4 게이트 (publisher 자체 검증 + reviewer가 다시 검증)
- `scripts/check-text-ratio.mjs` — G6/G8 게이트
- `scripts/measure-quality.sh` — 모든 게이트 한방
- 디자인 토큰 출처: `src/styles/tokens.css` (extract-tokens.sh가 생성)
