# Publish-harness 통합 설계 (ADR-001)

| 항목 | 값 |
|---|---|
| 상태 | 결정됨 (Accepted) — 구현은 publish-harness Stage 1 안정화 후 |
| 결정 일자 | 2026-04-24 |
| 작성자 | vibe-coding-harness 메인테이너 |
| 영향 범위 | vibe-coding-harness `publisher` 에이전트, `markup` 스킬, `project-profile.yaml` |

---

## 컨텍스트

`figma-react-lite-harness` 가 범용 `publish-harness` 로 진화 중이며, input × output 매트릭스가 폭증할 예정.

### Stage 1 (완료/진행 중)
- figma × vite-react-ts (figma-react-lite 계승)
- spec × vite-react-ts (핸드오프 번들 임포트)
- G1 visual regression (선택적, Playwright + pixelmatch)
- Feedback loop (3단계 자동 재시도 + 9개 실패 카테고리 분류)

### Stage 2 (예정)
- figma × html-static (정적 랜딩/마케팅)
- G9 brand-guardrails (spec 모드 Forbidden Patterns)

### Stage 3+ (실제 use case 기반)
- {figma, spec} × server-side templates (Thymeleaf, JSP, Blade, Django template)
- {figma, spec} × next-app-router
- {figma, spec} × rn-expo
- Storybook 소스 모드

→ **input N × output M = N×M 조합 폭증**. publisher 에이전트가 매번 따라가면 비대화.

## 질문

publish-harness 가 input/output 매트릭스를 확장할 때마다 vibe-coding-harness 의 `publisher` 에이전트를 직접 업데이트해야 하는가, 아니면 publish-harness 를 별도 레포 + plugin 으로 분리하여 참조하는 구조로 가는가.

## 검토한 옵션

### Option A — vibe-coding-harness 내부에 직접 업데이트
- publisher 에이전트와 markup 스킬 references 가 매번 업데이트
- 단일 레포 / 단일 plugin
- 자체 완결성

### Option B — publish-harness 별도 레포 + plugin 으로 분리, 참조
- vibe-coding-harness 는 publishing 워크플로우 오케스트레이션만
- publish-harness 는 input × output 변환 전담
- 두 plugin 독립 릴리즈

## 결정

**Option B 선택.**

### 결정 근거 (가중치 순)

1. **단일 책임 원칙 (Single Responsibility)** — vibe-coding 은 SI 풀스택 워크플로우 조율, publish 는 디자인/스펙→코드 변환. 책임 영역이 명확히 다름.
2. **input×output 매트릭스 폭증 대응** — publisher 에이전트가 매번 수정되면 매트릭스 폭발 시 유지보수 비용 비선형 증가. publish-harness 안에서 처리하면 vibe-coding 무관.
3. **외부 피드백 정렬** ("발명보다 참여") — publish-harness 를 awesome-agent-skills 같은 공개 생태계 시민으로 만들 수 있음. SKILL.md 표준 준수.
4. **재사용성** — publish-harness 는 vibe-coding 외 다른 컨텍스트(단독 변환, 다른 메타-하네스)에서도 사용 가능.
5. **독립 릴리즈 사이클** — publish-harness v0.5 안정화가 vibe-coding-harness 릴리즈를 막지 않음.
6. **공개 가능성** — publish-harness 의 노하우(Stage 1 검증된 G1-G8 + feedback loop)가 vibe-coding 의 "사내 도구" 우산 안에 갇히지 않음.

## 아키텍처 상세

### 두 plugin 분리

```
~/.claude/plugins/
├── publish-harness/                   ← 별도 GitHub 레포 + plugin
│   ├── .claude-plugin/marketplace.json
│   └── .claude/
│       └── skills/
│           ├── figma-to-react/        ← Stage 1
│           ├── spec-to-react/         ← Stage 1
│           ├── figma-to-html/         ← Stage 2
│           ├── figma-to-next/         ← Stage 3
│           ├── figma-to-rn/           ← Stage 3
│           ├── figma-to-thymeleaf/    ← Stage 3
│           └── ... (input × output 매트릭스)
│
└── vibe-coding-harness/               ← 별도 GitHub 레포 + plugin
    ├── .claude-plugin/marketplace.json
    └── .claude/
        ├── agents/
        │   └── publisher.md           ← 얇은 코디네이터 (publish-harness 호출)
        └── skills/
            └── ... (vibe-coding 워크플로우 전용)
```

### publisher.md 변환 (얇아짐)

**Before (현재 + A 옵션이면 점점 비대화)**

```markdown
당신은 publisher 에이전트.
스택 분기 → 자체 references 로드 → 직접 변환
- react + tailwind v3 → references/react/tailwind/v3.md
- react + tailwind v4 → references/react/tailwind/v4.md
- nextjs → references/nextjs/...
- thymeleaf → references/thymeleaf.md
... (매번 추가)
```

**After (얇은 코디네이터)**

```markdown
당신은 publisher 에이전트.
입력: design-spec.md + design-assets/
출력: 정적 마크업 + components.md

워크플로우:
1. project-profile.yaml 읽기 → input_type (figma | spec) + output_type (vite-react | html-static | next | rn | ...)
2. publish-harness 플러그인의 `{input_type}-to-{output_type}` 스킬 호출
3. 결과 검증 + components.md 갱신

publish-harness 미설치 시:
- 사용자에게 안내: "publish-harness 플러그인 설치 필요"
- 설치 명령 제공
```

publisher.md 가 **20줄 이하**로 줄어들고, "어떻게 변환하나" 는 publish-harness 가 전담.

### project-profile.yaml 확장

```yaml
publishing:
  input: figma                # figma | spec | both
  output: vite-react          # vite-react | next-app-router | html-static | rn-expo | thymeleaf | ...
  options:
    visual_regression: true   # G1 활성화 (publish-harness 가 처리)
    brand_guardrails: true    # G9 (Stage 2)
```

vibe-coding-harness 는 이 프로파일만 보고 publish-harness 에 위임.

### publish-harness 권장 구조 (별도 레포 만들 때)

```
publish-harness/
├── README.md (사용법, 지원 input × output 매트릭스, 게이트 목록)
├── .claude-plugin/marketplace.json
├── .claude/
│   ├── agents/                        ← 워커들
│   │   ├── section-worker.md          (figma-react-lite 계승)
│   │   ├── spec-worker.md             (Stage 1 추가)
│   │   └── ...
│   └── skills/
│       ├── figma-to-react/
│       │   ├── SKILL.md
│       │   └── references/
│       │       ├── _common.md         (공통)
│       │       ├── tailwind/{v3,v4}.md
│       │       └── ...
│       ├── spec-to-react/
│       ├── figma-to-html/             (Stage 2)
│       └── ...
├── templates/                          (각 output 별 부트스트랩)
│   ├── vite-react-ts/
│   ├── next-app-router/                (Stage 3)
│   ├── html-static/                    (Stage 2)
│   └── rn-expo/                        (Stage 3)
├── scripts/                            (G1 visual regression 등)
│   ├── visual-diff.sh                  (Playwright + pixelmatch)
│   ├── extract-tokens.sh
│   ├── figma-rest-image.sh
│   ├── measure-quality.sh              (G4-G8)
│   └── brand-guardrails.mjs            (G9, Stage 2)
└── docs/
    ├── stages.md                       (Stage 1/2/3+ 로드맵)
    └── adapters.md                     (각 input × output 변환 가이드)
```

### 통합 시점 워크플로우

```
사용자: "이 Figma URL → React 컴포넌트 만들어줘"
   ↓
[vibe-coding-orchestrator] 인식 → 단일 단계 (publisher 모드)
   ↓
[publisher 에이전트] (vibe-coding-harness)
   1. project-profile.yaml 읽기 → publishing.input=figma, publishing.output=vite-react
   2. publish-harness 의 "figma-to-react" 스킬 호출
   ↓
[figma-to-react 스킬] (publish-harness)
   - figma-rest-image.sh 로 PNG 다운로드
   - extract-tokens.sh 로 tokens.css
   - section-worker 워커 호출 (4단계 실행)
   - G1-G9 게이트 + feedback loop
   ↓
결과 → publisher 가 vibe-coding 의 컨벤션(_workspace/handoff/)에 정리
   ↓
[reviewer 에이전트] (vibe-coding-harness)
   - 슬라이스 차원 검증 (publisher 가 제대로 publish-harness 호출했는지)
```

## 마이그레이션 경로

현재 상태 → 권장 상태:

| 단계 | 작업 | 책임 |
|---|---|---|
| 1 | publish-harness 별도 레포 생성 (HyeonJ/publish-harness) | publish-harness 메인테이너 |
| 1 | figma-react-lite 자산을 여기로 이전 + Claude Code plugin 으로 패키징 | publish-harness 메인테이너 |
| 2 | vibe-coding-harness 의 publisher.md 를 얇은 코디네이터로 재작성 | vibe-coding-harness 메인테이너 |
| 2 | project-profile.yaml 에 `publishing.{input, output, options}` 추가 | vibe-coding-harness 메인테이너 |
| 3 | 사용자 환경에 두 plugin 모두 설치 (`/plugin install HyeonJ/publish-harness`, `/plugin install HyeonJ/vibe-coding-harness`) | 사용자 |
| 4 | 첫 슬라이스 실측 (방향성 원칙대로) | vibe-coding-harness 메인테이너 |

publish-harness 가 이미 검증된 자산이므로 risk 가 낮음. vibe-coding-harness 는 통합 워크플로우만 검증하면 됨.

## 외부 피드백 충족 매트릭스

| 외부 피드백 항목 | 본 결정으로 해결되는 방식 |
|---|---|
| 발명보다 참여 (SKILL.md 표준 + 공개 생태계) | publish-harness 를 awesome-agent-skills 에 PR 가능한 형태로 |
| 검증 전 추가 금지 | publish-harness 는 이미 사용자가 검증 중 (Stage 1 통과) |
| constraint → feedback → gate 순서 | publish-harness Stage 1 에 feedback loop 이미 있음 |
| visual regression (G1) | publish-harness G1 이미 도입 |
| 한국 SI 진짜 특화 | publish-harness G9 brand-guardrails (Stage 2 예정) |

→ publish-harness 가 사실상 피드백 권고를 정확히 따라가고 있음. vibe-coding-harness 는 그걸 사용하는 클라이언트가 되는 게 가장 자연스러움.

## 트레이드오프 / 위험

### 잠재 위험
- **두 plugin 의존성 관리** — vibe-coding-harness 가 publish-harness 의 특정 SemVer 범위에 의존. 미설치/미호환 시 사용자 디버깅 비용.
- **버전 어긋남 가능성** — publish-harness 가 breaking change 한 경우 vibe-coding-harness 도 따라가야 함.
- **첫 진입 장벽 증가** — 사용자가 두 plugin 을 모두 설치해야 함 (대신 README 에 단일 설치 명령 제공으로 완화).

### 위험 완화책
- vibe-coding-harness 의 publisher.md 가 publish-harness 미설치 시 명확한 안내 + 설치 명령 출력.
- publish-harness 의 SemVer 정책 명시 (major 변경 시 vibe-coding-harness 도 메이저 변경).
- 통합 테스트 시나리오 — `docs/test-scenarios/` 에 publish-harness 호출이 정상 동작하는지 검증 항목 추가.

## 후속 액션

이 결정의 시점은 **publish-harness Stage 1 안정화 직후**. 그 전까지 vibe-coding-harness 는 현재 v0.1.3 상태(figma-react-lite 자산 revert 완료 + 골격 유지) 그대로 대기.

publish-harness 가 별도 레포로 분리되는 시점에 본 ADR 의 "마이그레이션 경로" 단계 2~4 를 실행한다.

## 관련 문서
- 외부 피드백 원문: 본 ADR 작성을 트리거한 피드백 (CLAUDE.md "방향성 원칙" 섹션 참조)
- vibe-coding-harness 알려진 갭: `CLAUDE.md` "알려진 갭 (Known Gaps — v0.1.0)"
- 첫 슬라이스 시나리오: `docs/test-scenarios/order-api-slice.md`
