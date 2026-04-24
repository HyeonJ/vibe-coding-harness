# vibe-coding-harness

한국 SI 웹 프로젝트의 vibe coding 워크플로우를 자동화하는 하네스. 5명 에이전트 팀(design, backend, publisher, frontend, reviewer)이 협업하여 한 슬라이스(기능 1개)를 end-to-end로 완성한다.

## 하네스: vibe-coding (SI 웹 풀스택)

**목표:** 한국 SI 웹 프로젝트의 9개 phase를 5명 에이전트 팀으로 자동화. 수직 슬라이스 단위로 빠르게 완성.

**트리거:** 신규 기능 개발, 슬라이스 구현, 풀스택 작업, vibe coding 워크플로우 실행 시 `vibe-coding-orchestrator` 스킬 사용. 단일 단계만 필요할 때(예: "DB 설계만")는 오케스트레이터가 해당 에이전트로 라우팅. 단순 질문/탐색은 직접 응답 가능.

**전제:**
- 신규 프로젝트는 먼저 `setup` 스킬로 `.claude/project-profile.yaml` 생성 필요
- 사용자 글로벌 `~/.claude/CLAUDE.md`의 코딩 컨벤션을 따름
- `_workspace/handoff/`만 git commit (나머지는 ignore)

**구성:**
- 에이전트: `.claude/agents/` 5명 (design, backend, publisher, frontend, reviewer)
- 스킬: `.claude/skills/` 8개 (5명 + setup, deploy-checklist, spec-extract) + 오케스트레이터 1개
- 프로파일: `.claude/project-profile.yaml` (스택 동적 분기)
- 템플릿: `templates/project-profile.yaml`, `templates/_workspace.gitignore`

## 방향성 원칙 (외부 피드백 수용)

다음 원칙을 모든 후속 작업이 따른다.

1. **발명보다 참여** — Claude Code의 SKILL.md 표준, awesome-agent-skills 같은 공개 생태계 포맷을 따른다. "vibe-coding-harness" 라는 자체 네이밍이 "로컬 방언"이 되지 않도록, 공개 가능한 형태로 진화 검토.
2. **검증 전에 추가 금지** — 실 프로젝트 1~2개 완주 전에는 게이트/스킬/에이전트 추가 자제. 데이터 없는 설계는 v0.1.0이 가장 피해야 할 것 (CLAUDE.md "알려진 갭" 참조).
3. **constraint → feedback → gate 순서** — 업계 권고 (Plan-Execute-Verify 패턴). 현재 constraint(에이전트 룰)와 gate 후보(reviewer)는 있으나, **feedback loop(워커 자동 재시도, 실패 분석 후 재실행) 부재**. 다음 우선순위.
4. **visual regression 도입 검토** — Applitools/Percy 추세에 따라 Playwright + pixelmatch (또는 동등) 로 Figma vs 프로덕션 직접 diff. lite 원칙과 공존 가능.
5. **진짜 한국 SI 특화는 데이터 기반** — 정부 웹접근성(WA) 인증 체커, 퍼블리셔 전통 관행(부모 div 래핑), PDF 출력 호환성 같은 것은 실 프로젝트에서 발견된 실제 요구로만 도입. 마케팅 문구로 끝나지 않게.

**v0.1.0 지원 스택:**
- backend: spring-boot + mybatis, spring-boot + jpa
- frontend: thymeleaf + jquery, react

## 알려진 갭 (Known Gaps — v0.1.0)

다음 두 항목은 **첫 슬라이스 시나리오 실측 후 결정**한다. 현재 추측 기반 결정 보류.

### 갭 1: 통합 테스트 작성 책임 미정
- **현재 상태**: 단위 테스트는 backend가 작성, 실행은 reviewer. 통합 테스트(Repository 통합, E2E API 시나리오) 작성 책임 명시 안 됨.
- **검토했으나 보류한 옵션**:
  - reviewer가 E2E 작성 → reviewer "자체 수정 금지" 원칙의 인지적 분리 깨짐 (self-validation 함정)
  - backend가 모두 작성 → 한국 SI 실무(QA 별도 직군)와 매핑 안 맞음
- **유력한 후보**: test 에이전트 신설 (옵션 D, 5→6명) 또는 트리거 기반 (사용자 명시 시 backend가 추가 작성)
- **결정 시점**: 첫 슬라이스(order-api-slice) 실행 후 어떤 통합 테스트가 실제로 필요했는지 데이터 확보 후

### 갭 2: 배포 워크플로우 통합 미정
- **현재 상태**: deploy-checklist 스킬은 사용자가 명시적으로 호출 (오케스트레이터 외부)
- **검토했으나 보류한 옵션**:
  - 오케스트레이터에 선택적 Phase 7.5 (배포 준비 게이트) → 한국 SI는 슬라이스마다 배포 X, 여러 슬라이스 누적 → 스프린트 → 배포가 표준. 매번 묻는 게이트는 노이즈가 됨
- **유력한 후보**: 별도 deploy-orchestrator 스킬 (다중 슬라이스 → 배포 사이클 전담)
- **결정 시점**: 다중 슬라이스 시나리오 누적 후 배포 통합이 정말 필요한 시점에

**원칙**: 데이터 없는 설계 결정은 v0.1.0이 가장 피해야 할 것. 먼저 돌려보고 결정한다.

**변경 이력:**

| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-04-24 | 초기 구성 | 전체 (5 에이전트 + 7 스킬 골격) | 신규 하네스 구축 (v0.1.0 골격) |
| 2026-04-24 | spec-extract 스킬 추가 + design/publisher/markup/오케스트레이터 수정 | 8개 파일 | 다양한 입력 형식(Figma/PDF/PPT/Excel/Word/이미지)을 design-spec.md로 통일하여 컨텍스트 효율 + 작업 안정성 + 사람 검토 가능성 확보. design 에이전트의 첫 단계로 spec-extract 호출. publisher는 원본 디자인 도구 직접 접근 금지, design-spec.md만 참조. |
| 2026-04-24 | Known Gaps 문서화 + orchestrator Phase 7 안내 추가 | CLAUDE.md, orchestrator | 통합 테스트 책임/배포 통합 두 갭에 대해 추측 결정 보류. 코드-리뷰어 지적 반영 — self-validation 함정 회피, SI 배포 사이클 매핑 정확성 확보. 결정은 첫 슬라이스 실측 후. |
| 2026-04-24 | figma-react-lite-harness 자산 흡수 (옵션 A) | scripts/, templates/vite-react-ts/ | 검증된 자동화 스크립트(check-token-usage, check-text-ratio, measure-quality, extract-tokens, figma-rest-image, bootstrap, doctor, setup-figma-token) + Vite React 부트스트랩 템플릿 통째 복사. 향후 본문 채우기 시 setup/spec-extract/markup/reviewer references에서 이 자산들 호출. 차륜 재발명 회피. |
| 2026-04-24 | references 계층형 구조 변환 (Phase 8 준비) | backend/markup/interaction의 references + 3개 SKILL.md | 다중 변형 framework(react, spring-boot)는 `{framework}/_common.md + {variant}.md` 패턴으로 변환. 단일 변형(thymeleaf, jquery)은 평면 유지. 새 placeholder 추가: `backend/references/spring-boot/_common.md`, `markup/references/react/tailwind.md` (사용자 프로젝트 React+Tailwind 즉시 필요). Phase 8 본문 채우기는 위치 결정 끝난 상태로 시작. |
| 2026-04-24 | Phase 8 1차 — figma-react-lite 자산 본문 흡수 | spec-extract/references/figma.md, markup/references/react/_common.md + tailwind.md, reviewer/references/conventions.md, reviewer/SKILL.md | figma-rest-image.sh + extract-tokens.sh 호출 패턴, dumb 컴포넌트 원칙 + DS 인벤토리, Tier 1/2 반응형 + mobile-first 변환표, G4/G5/G6/G7/G8 게이트 + measure-quality.sh 흡수. 사용자 프로젝트(React + Tailwind v4) 작업 가능 수준. 잔여 references(Spring Boot, jquery, thymeleaf, setup, deploy)는 차후 슬라이스 시도하며 보강. |
| 2026-04-24 | v0.1.2 — 첫 슬라이스 실측 사전 정비 (profile 스키마 + tailwind 버전 분기 정식화 + v4 템플릿 + bootstrap 가드) | project-profile.yaml, publisher.md, markup/SKILL.md, markup/references/react/tailwind/_common.md + v3.md + v4.md (신설), setup/SKILL.md, templates/vite-react-ts/*, scripts/bootstrap.sh | 첫 실측 bootstrap 전 발견 4갭 해소. (1) profile 키 3곳 불일치 → `frontend.framework` + `frontend.styling` + `frontend.language` + `frontend.bundler` 로 정리 + `backend.enabled`/`frontend.enabled` 추가. (2) Tailwind 버전 분기축 부재 → `react/tailwind/{_common,v3,v4}.md` 3파일로 분해 + SKILL.md 에 `package.json` major 감지 Step 1.5 정식화. (3) vite-react-ts 템플릿 Tailwind v3 → v4 (`@tailwindcss/vite`, `@theme` 블록, `tailwind.config.ts`/`postcss.config.js` 제거). (4) bootstrap.sh docs 복사 가드 + 말미 legacy 메시지(section-worker / figma-react-lite 스킬) 교체. 근거: 데이터 없는 설계 결정 회피 원칙 — 실측 돌리기 전 blocking 갭만 해소. |
| 2026-04-24 | **v0.1.3 — figma-react-lite 자산 revert (외부 피드백 수용, 옵션 B-2)** | scripts/ 전체 삭제, templates/vite-react-ts/ 삭제, 5개 references 본문 → TODO 복원 (spec-extract/figma.md, markup/react/_common.md + tailwind/{_common,v3,v4}.md, reviewer/conventions.md), reviewer/SKILL.md measure-quality 호출 제거, setup/SKILL.md bootstrap.sh 안내 제거, "방향성 원칙" 섹션 신설 | **외부 피드백 수용**: "1 commit, 0 실 프로젝트 검증" 상태에서 게이트 추가는 이른 최적화. constraint→feedback→gate 순서가 뒤집힘 (feedback loop 부재). G1 (visual regression) 부재. 한국 SI 도메인 특화 부재. → 흡수 자산을 모두 비우고 첫 슬라이스 실측 후 진짜 필요한 부분만 자체 구현 또는 외부 의존. 골격(에이전트 5명 + 스킬 8개 + 오케스트레이터)은 유지. |
| 2026-04-24 | ADR-001 — publish-harness 별도 plugin 분리 결정 | docs/architecture/publish-harness-integration.md (신설) | publishing 횡단 관심사를 vibe-coding-harness 내부에 누적하지 않고 별도 GitHub 레포 + Claude Code plugin (publish-harness)으로 분리, publisher.md 는 얇은 코디네이터로 위임. input N × output M 매트릭스 폭증을 vibe-coding 외부에서 처리. 외부 피드백 5항목 모두 충족 ("발명보다 참여" + 단일 책임 + 재사용 + 독립 릴리즈). 시점: publish-harness Stage 1 안정화 후. |
