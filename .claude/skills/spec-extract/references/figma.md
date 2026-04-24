# Figma 추출 가이드

Figma URL을 입력으로 받아 `_workspace/handoff/design-spec.md` + `_workspace/handoff/design-assets/*.png` + 디자인 토큰을 자동 추출.

## 전제: FIGMA_TOKEN 환경변수

`scripts/`의 모든 Figma 스크립트는 Figma Personal Access Token이 필요하다. 미설정 시 즉시 안내:

```bash
bash scripts/setup-figma-token.sh   # 대화형 등록 (한 번만)
```

→ 사용자가 토큰을 셋업한 후 새 터미널 세션부터 자동 로드됨.

## 추출 순서

### Step 1: Figma URL 파싱

`figma.com/design/<fileKey>/<fileName>?node-id=<nodeId>` 에서:
- **fileKey**: `/design/` 다음 부분
- **nodeId**: query string `node-id=10-5282` → `10:5282` (대시 → 콜론, 자동 변환됨)

### Step 2: 페이지 전체 + 섹션별 baseline PNG 다운로드

`scripts/figma-rest-image.sh`로 Figma REST Images API 호출:

```bash
# 페이지 전체 (Desktop)
bash scripts/figma-rest-image.sh <fileKey> <pageNodeId> \
  _workspace/handoff/design-assets/{page}-full.png --scale 2

# 섹션별 (선택, design-spec.md에 섹션 단위 분리 시)
bash scripts/figma-rest-image.sh <fileKey> <sectionNodeId> \
  _workspace/handoff/design-assets/{page}-{section}.png --scale 2
```

**중요**: leaf nodeId만 사용. 부모 frame nodeId로 export하면 text-bearing raster 안티패턴 발생 (G6 게이트 FAIL).

### Step 3: 반응형 프레임 자동 감지

페이지에 Tablet/Mobile 디자인 별도 존재하는지 확인. 감지 단서:
- **프레임 이름 키워드**: `Desktop`, `Tablet`, `Mobile`, `768`, `1920`, `375` 등
- **프레임 너비**: 1920/1440/1280=Desktop · 768/1024=Tablet · 375/390/360=Mobile
- **별도 페이지 분리**: 페이지 이름에 `Mobile` 등 포함

감지된 뷰포트 nodeId를 design-spec.md에 기록:
```markdown
## 화면: 주문 등록
- Desktop nodeId: 12:345 (필수)
- Tablet nodeId: 12:456 (선택, 감지된 경우만)
- Mobile nodeId: 12:567 (선택)
```

→ 이후 markup 스킬이 Tier 1(Desktop만) vs Tier 2(반응형 디자인 있음) 경로 결정에 사용.

### Step 4: 디자인 토큰 추출

`scripts/extract-tokens.sh`로 자동 추출:

```bash
# 권장: Component/Design System 페이지 지정 (네이밍 품질 ↑)
bash scripts/extract-tokens.sh <fileKey> --component-page <component-page-nodeId>

# fallback: 전체 파일 휴리스틱 스캔
bash scripts/extract-tokens.sh <fileKey>
```

산출:
- `src/styles/tokens.css` — `:root { --brand-*, --surface-*, --text-*, --space-*, --radius-* }`
- `src/styles/fonts.css` — `@font-face` 블록
- `docs/token-audit.md` — 토큰 인벤토리 (markup 스킬이 참조)
- `tmp/figma-raw.json` — 원본 (디버깅용)

**중요**: tokens.css / fonts.css는 **이 스크립트만이 쓴다**. 사람이나 다른 에이전트가 직접 수정 금지 (재추출 시 덮어써짐).

### Step 5: 노드 구조 추출 (선택)

복잡 섹션의 경우 `get_design_context` MCP 호출 또는 REST 직접:

```bash
# REST fallback (MCP 쿼터 소진 또는 미등록 시)
curl -sS -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/<fileKey>/nodes?ids=<nodeId>&depth=3"
```

응답의 `document.children[].children[]`에서 layout/fill/style 추출.

### Step 6: design-spec.md 작성

추출 결과를 `_workspace/handoff/design-spec.md`에 통합 (spec-extract/SKILL.md의 표준 구조 따름):

```markdown
# Design Spec — {슬라이스명}

## 메타
- 입력 출처: Figma <URL>
- 추출 시각: 2026-04-24 14:30
- fileKey: <fileKey>
- 추출 도구: spec-extract (figma references)

## 화면: {화면명}
- 스크린샷: `_workspace/handoff/design-assets/{file}.png`
- Desktop nodeId: ...
- Tablet nodeId: ... (감지된 경우)
- Mobile nodeId: ... (감지된 경우)

### 컴포넌트
- {컴포넌트명}: {역할}
  - props: ...
  - 디자이너 노트: ... (Figma description 추출)

### 인터랙션 명세
- ...

### 디자인 토큰
- (tokens.css에 자동 추출됨, docs/token-audit.md 참조)

## 미해결 사항 (사용자 확인 필요)
- [ ] ...
```

## 주의 사항

### Framelink MCP 금지
이전 세대 Framelink MCP는 폐기됨. 항상 figma-rest-image.sh (REST API) 사용.

### MCP vs REST
- 노드 tree 1회 조회: `get_design_context` MCP (있으면)
- baseline PNG / 에셋: 항상 figma-rest-image.sh (REST, 쿼터 넉넉)
- MCP 미등록 또는 쿼터 소진: REST fallback

### 부모 frame nodeId 금지
- ❌ 부모 frame export → text가 raster로 baked (G6 FAIL)
- ✅ leaf nodeId (실제 이미지/아이콘) export

## 활용 자산
- `scripts/setup-figma-token.sh` — 토큰 대화형 등록
- `scripts/figma-rest-image.sh` — 노드 PNG 다운로드 (재시도/검증 포함)
- `scripts/extract-tokens.sh` — 토큰 추출
- `scripts/_extract-tokens-analyze.mjs` — 토큰 분석 (extract-tokens가 내부 호출)
- `scripts/_lib/load-figma-token.sh` — 환경변수 로드 (다른 스크립트가 source)
