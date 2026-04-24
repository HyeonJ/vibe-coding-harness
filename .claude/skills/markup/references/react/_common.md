# React JSX 마크업 공통 가이드 (정적 컴포넌트만)

publisher 에이전트가 React 계열 스택(react, react-native, nextjs)에서 작업 시 항상 로드. 변형별 추가 가이드는 `tailwind.md`, `nativewind.md` 등.

## 절대 원칙 (위반 시 즉시 중단)

1. **dumb 컴포넌트만** — props만 받음, 상태/이벤트/API 호출 금지
   - `useState`, `useEffect`, `useRef` 등 hooks 금지 (interaction 스킬의 영역)
   - `onClick`, `onChange`, `onSubmit` 핸들러 정의 금지 → props로 받기만
   - `fetch`, `axios`, `useQuery` 등 데이터 호출 금지

2. **시맨틱 HTML 강제** — `<section>`, `<header>`, `<nav>`, `<footer>`, `<h1>~<h3>`, `<ul>`, `<button>`
   - `<div onClick>` 금지 (G5 FAIL — eslint jsx-a11y)
   - 버튼은 반드시 `<button>` (Link면 `<a>`)

3. **텍스트는 JSX 트리에**
   - 문장을 `<img alt="...">` 한 줄로 밀어넣지 말 것
   - 배경/장식 raster만 `<img>` 허용
   - 본문/제목/리스트는 `<h2>`, `<p>`, `<li>` 로 재구성
   - **이유**: G6 게이트 (text-baked raster 차단), G8 게이트 (i18n 가능성), 검색 엔진 / 스크린리더 접근성

4. **타입 안전성** — `any` / `unknown` 금지. props는 `readonly` interface

## 디렉토리 구조

```
src/components/
├── ui/                    공통 컴포넌트 (Button, Input, Card, Wordmark 등)
├── sections/              페이지 단위 큰 섹션
│   └── home/
│       ├── HomeHero.tsx
│       └── HomeFeatures.tsx
└── layout/                Header / Footer / Sidebar
```

## 컴포넌트 작성 패턴

### 함수형 컴포넌트
```tsx
interface ProductCardProps {
  readonly name: string;
  readonly price: number;
  readonly imageSrc: string;
  readonly onSelect?: (id: string) => void;  // ← 이벤트는 받기만
}

export function ProductCard({ name, price, imageSrc, onSelect }: ProductCardProps) {
  return (
    <article className="product-card">
      <img src={imageSrc} alt={name} />
      <h3>{name}</h3>
      <p>{price.toLocaleString()}원</p>
    </article>
  );
}
```

### 컴포넌트 합성 (children 패턴)
```tsx
interface CardProps {
  readonly title: string;
  readonly children: React.ReactNode;
}

export function Card({ title, children }: CardProps) {
  return (
    <article>
      <h3>{title}</h3>
      {children}
    </article>
  );
}
```

## SVG / 이미지 배치 패턴

부모 div + 원본 사이즈 img:
```tsx
<div className="w-[28px] h-[28px] flex items-center justify-center">
  <img src={icon} className="w-[21px] h-[9px]" alt="" />
</div>
```

**금지**: Figma REST PNG는 baked-in 합성 사진이므로 CSS에서 `rotate()`, `mix-blend-*`, 배경색 재적용 금지 (이미지 자체에 이미 적용된 효과를 다시 적용하면 이중 처리).

## DS 인벤토리 패턴 (figma-react-lite 흡수)

publisher 작업 시작 전, design-spec.md에서 **3+ 화면에 반복 등장하는 컴포넌트** 사전 식별:

| 식별 기준 | 예시 |
|---|---|
| 로고 / 워드마크 (Figma 심볼이 아니어도) | Wordmark, BrandMark |
| 반복 아이콘 | IconButton, NavIcon |
| 반복 카드 패턴 | ProductCard, FeatureCard |
| 반복 문구 | CTA, FormError |

→ `_workspace/publisher/components.md`에 사전 등록 후, 후속 컴포넌트들이 **인라인 재구현 금지** (DRY 위반).

**놓치면 발생하는 문제**: 같은 로고를 Nav가 `<a>text</a>`, Footer가 `<p>text</p>`, Header가 `<img>` 로 각자 구현 → 사후 리팩터 비용 폭증.

## 글로벌 CLAUDE.md 준수 (JS 일반)
- `var` 금지, `const` 기본, 재할당 시만 `let`
- 템플릿 리터럴 `` `${var}` `` 사용 (string concat 금지)
- 와일드카드 import 금지

## 검증 (reviewer가 자동 실행)
- G4: 디자인 토큰만 사용 (hex literal 금지) — `scripts/check-token-usage.mjs`
- G5: 시맨틱 HTML — eslint jsx-a11y
- G6: 텍스트:이미지 비율 — `scripts/check-text-ratio.mjs`
- G8: i18n 가능성 (literal text 검출)
