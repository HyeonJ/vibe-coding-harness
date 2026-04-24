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
- 자동화 스크립트: `scripts/` (figma-react-lite-harness 흡수 — 토큰 추출, 품질 게이트, Figma REST, 부트스트랩)
- React 부트스트랩 템플릿: `templates/vite-react-ts/`

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
