# Phase 6 — 검증 보고서

생성 일자: 2026-04-24
대상 버전: v0.1.0 (골격)

## 1. 구조 검증

### 1-1. 파일 존재 확인

| 카테고리 | 개수 | 결과 |
|---|---|---|
| 에이전트 정의 (.claude/agents/*.md) | 5 | ✅ design, backend, publisher, frontend, reviewer |
| 스킬 SKILL.md | 8 | ✅ design, backend, markup, interaction, reviewer, setup, deploy-checklist + vibe-coding-orchestrator |
| 스킬 references | 11 | ✅ 모든 v0.1.0 지원 스택 placeholder 존재 |
| 템플릿 파일 | 2 | ✅ project-profile.yaml, _workspace.gitignore |
| 하네스 메타 | 2 | ✅ CLAUDE.md, README.md |

### 1-2. Frontmatter 검증

- 13개 파일 모두 YAML frontmatter 보유 (`---` 마커 + name + description)
- 에이전트 5개 모두 `model: opus` 명시 ✅ (harness 규칙 준수)
- 스킬은 model 필드 불필요 (트리거 기반 로딩이라)

### 1-3. 에이전트 ↔ 스킬 매핑

| 에이전트 | 사용 스킬 | 매핑 일치 여부 |
|---|---|---|
| design | design | ✅ 동일명 |
| backend | backend | ✅ 동일명 |
| publisher | **markup** | ✅ 의도적 분리 (publisher 역할이 markup 스킬 사용) |
| frontend | **interaction** | ✅ 의도적 분리 (frontend 역할이 interaction 스킬 사용) |
| reviewer | reviewer | ✅ 동일명 |

### 1-4. 데드 링크 검증

| 참조 | 존재 여부 |
|---|---|
| `.claude/agents/{name}.md` (오케스트레이터에서 참조) | ✅ 5개 모두 존재 |
| `.claude/project-profile.yaml` (런타임 생성, 템플릿 제공) | ✅ 템플릿 존재 |
| `_workspace/handoff/openapi.yaml` (런타임 산출물) | ✅ 런타임 생성 (Gate 체크 있음) |
| `~/.claude/CLAUDE.md` (글로벌 컨벤션 참조) | ✅ 사용자 보유 |

→ 데드 링크 없음.

---

## 2. 트리거 검증

### 2-1. vibe-coding-orchestrator (메인)

**Should-trigger (이 스킬이 트리거되어야 하는 쿼리)**:
1. "주문 등록 API 만들어줘"
2. "회원 가입 화면 추가하고 싶어"
3. "전체 슬라이스 한번 진행해줘"
4. "vibe coding으로 풀스택 작업 시작"
5. "결제 기능 신규 개발"
6. "방금 만든 주문 슬라이스 수정해줘" (후속)
7. "API 명세 다시 짜고 backend도 다시" (후속)
8. "이전 슬라이스 개선" (후속)

**Should-NOT-trigger (다른 스킬/도구가 적합한 near-miss)**:
1. "이 함수 어떻게 작동해?" → 일반 질문, 도구 호출만
2. "버그 좀 디버깅해줘" → systematic-debugging 스킬
3. "Figma 디자인 시안 받아서 코드만 변환" → frontend-design 또는 figma-implement-design (단일 변환)
4. "Slack에 메시지 보내줘" → 무관
5. "Excel로 정리해줘" → office-xlsx
6. "DB 설계만 도와줘" → 오케스트레이터가 받되 단일 모드(Agent design)로 라우팅
7. "git status 보여줘" → bash 직접

### 2-2. 개별 스킬 트리거 키워드 충돌

| 스킬 | 핵심 키워드 | 충돌 가능성 |
|---|---|---|
| design | 설계, ERD, OpenAPI, 명세 | ✅ 도메인 한정적 |
| backend | API 개발, Spring 컨트롤러 | ✅ 명확 |
| markup | 퍼블리싱, 마크업, Figma 컴포넌트화 | ⚠️ figma-implement-design과 부분 겹침 — 의도: vibe-coding 컨텍스트 안에선 markup, 단독이면 figma-* |
| interaction | 프론트 동작, API 연동, JS 작성 | ⚠️ frontend-design과 부분 겹침 — 의도: vibe-coding 컨텍스트면 interaction |
| reviewer | 코드 리뷰, QA, 검증 | ⚠️ superpowers:code-reviewer와 겹침 — 의도: 이 하네스 내 검증은 이 스킬 |
| setup | 신규 프로젝트, 보일러플레이트 | ✅ 명확 |
| deploy-checklist | 배포 준비, 배포 체크리스트 | ✅ 명확 |

→ 약간의 트리거 충돌 있음. **해결 방향**: 오케스트레이터가 진입점이 되어 컨텍스트를 잡아주면 자연스럽게 내부 스킬로 라우팅됨. 외부 스킬(figma-*, frontend-design 등)은 vibe-coding 컨텍스트 밖에서만 사용.

---

## 3. 드라이런 (Phase 순서 + 데이터 흐름)

### 3-1. 풀 슬라이스 모드 흐름

```
사용자 입력
   ↓
[Phase 0] project-profile.yaml 확인
   ├─ 미존재 → setup 스킬 안내 (중단)
   └─ 존재 → 다음
   ↓
[Phase 0 cont.] _workspace/ 확인
   ├─ 미존재 → 신규 슬라이스 모드
   └─ 존재 → 신규/수정 사용자 의도 확인
   ↓
[Phase 1] _workspace/ 디렉토리 생성, 입력 저장
   ↓
[Phase 2] TeamCreate (5명)
   ↓
[Phase 3] design 단독 실행
   ↓ Gate: openapi.yaml 존재 확인
   ↓
[Phase 4] backend ∥ publisher 병렬
   ↓ publisher 완료 시점에 frontend 시작 트리거
   ↓
[Phase 5] frontend (publisher 후)
   ↓
[Phase 6] reviewer (incremental + 통합 검증)
   ↓ Gate: Critical 0 확인
   ├─ Critical 있음 → 해당 에이전트 1회 재호출 → Gate 재검사
   └─ Critical 없음 → 다음
   ↓
[Phase 7] TeamDelete + 사용자 보고
```

### 3-2. 데이터 흐름 검증

| 산출물 | 생성자 | 소비자 | 경로 | 상태 |
|---|---|---|---|---|
| `openapi.yaml` | design | backend, frontend | `_workspace/handoff/` | ✅ 양쪽 다 명시 |
| `erd.md` | design | backend | `_workspace/handoff/` | ✅ |
| `schema.sql` | design | backend (Flyway) | `_workspace/handoff/` | ✅ |
| `components.md` | publisher | frontend | `_workspace/publisher/` | ✅ |
| `progress.md` | backend, frontend | reviewer | `_workspace/{backend,frontend}/` | ✅ |
| `review-{슬라이스}.md` | reviewer | 리더 (사용자 보고) | `_workspace/reviewer/` | ✅ |

**Dead link 0건.**

### 3-3. 에러 흐름 검증

| 시나리오 | 처리 경로 | 검증 |
|---|---|---|
| design 실패 | 즉시 중단, 사용자 보고 | ✅ orchestrator 명시 |
| backend 빌드 실패 | 1회 자체 재시도 → 실패 시 리더 보고 | ✅ |
| reviewer Critical 발견 | 해당 에이전트 1회 재호출 | ✅ 무한 루프 방지 |
| 사용자 요청 모호 | 추측 금지, 질문 | ✅ Phase 0에 명시 |

---

## 4. 1 슬라이스 시나리오 (테스트 명세)

`docs/test-scenarios/order-api-slice.md` 별도 파일 참조.

이 시나리오는 v0.1.0 본문 채우기 작업의 **완주 검증 기준**:
- 시나리오 통과 = v0.1.0 배포 가능
- 시나리오 실패 = 어느 단계에서 막혔는지 보고 + skill 본문 보완

---

## 5. 종합 평가

| 항목 | 결과 |
|---|---|
| 파일 구조 | ✅ 완전 |
| Frontmatter | ✅ 모두 유효 |
| 에이전트-스킬 매핑 | ✅ 일관 |
| Dead link | ✅ 없음 |
| 트리거 description | ⚠️ 약간 충돌 가능 (외부 스킬과) — 오케스트레이터 진입점으로 완화 |
| Phase 순서 | ✅ 논리적, gate 명확 |
| 데이터 흐름 | ✅ 모든 산출물 경로 일치 |
| 에러 핸들링 | ✅ 명시됨 |

**v0.1.0 골격 완료. 다음 단계: 본문 채우기 (각 references 파일 + 첫 슬라이스 검증).**
