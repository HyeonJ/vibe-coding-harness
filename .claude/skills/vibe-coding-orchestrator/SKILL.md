---
name: vibe-coding-orchestrator
description: "한국 SI 웹 프로젝트의 vibe coding 워크플로우를 조율하는 메인 오케스트레이터. design + backend + publisher + frontend + reviewer 5명 팀이 협업하여 한 슬라이스(기능 1개)를 end-to-end로 완성. '신규 기능 개발', '주문 API 만들어줘', '회원 가입 화면 추가', '슬라이스 개발', '풀 스택 구현', 'vibe coding', 'WBS 진행' 등의 요청 시 반드시 이 스킬을 사용. 후속 작업 키워드: '재실행', '수정', '보완', '슬라이스 수정', '특정 단계만 다시', '이전 슬라이스 개선'."
---

# Vibe Coding Orchestrator — SI 웹 풀스택 조율

한국 SI 웹 프로젝트의 5명 에이전트 팀(design, backend, publisher, frontend, reviewer)을 조율하여 **수직 슬라이스**(기능 1개를 end-to-end)로 완성한다.

## 실행 모드

이 오케스트레이터는 **명시적 모드**로 동작한다 (자동 판단 X).

| 사용자 요청 유형 | 모드 | 동작 |
|---|---|---|
| 풀 슬라이스 ("주문 API 만들어줘", "회원 가입 화면 추가") | **에이전트 팀** | TeamCreate로 5명 팀 구성, 파이프라인 실행 |
| 단일 단계 ("DB 설계만 도와줘", "이 API만 리뷰해줘") | **서브 에이전트** | Agent 도구로 해당 에이전트 1명만 호출 |
| Phase 일괄 ("전체 설계만 먼저", "백엔드 전체 구현") | **에이전트 팀 (제한)** | 해당 단계 에이전트만 팀에 포함 |

**원칙**: 사용자 요청에서 모드가 명백하지 않으면 사용자에게 묻는다. 추측 금지.

## 에이전트 구성

| 에이전트 | 타입 | 역할 | 스킬 | 핵심 출력 |
|---|---|---|---|---|
| design | 커스텀 | 디자인 추출 + 프로그램·DB·API 계약 | design + spec-extract | `_workspace/handoff/design-spec.md`, `openapi.yaml`, `erd.md`, `schema.sql` |
| backend | 커스텀 | API 구현 (스택 동적) | backend | 백엔드 코드 + 단위 테스트 |
| publisher | 커스텀 | 정적 마크업 | markup | 마크업 코드 + `_workspace/publisher/components.md` |
| frontend | 커스텀 | 동적 동작 + API 연동 | interaction | 동적 코드 + `_workspace/frontend/progress.md` |
| reviewer | 커스텀 | 컨벤션·계약·테스트 검증 | reviewer | `_workspace/reviewer/review-{슬라이스}.md` |

## 워크플로우 (풀 슬라이스 모드 — 기본)

### Phase 0: 컨텍스트 확인 (필수)

1. **프로젝트 프로파일 확인**
   - `.claude/project-profile.yaml` 존재 확인
   - 미존재 시 → `setup` 스킬로 라우팅하고 사용자에게 셋업 먼저 안내. 이 슬라이스 작업은 중단.
2. **이전 산출물 확인**
   - `_workspace/handoff/` 존재 여부
   - 사용자 요청이 "신규 슬라이스"인지 "기존 수정"인지 판별:
     - **신규 슬라이스**: 기존 `_workspace/`를 `_workspace_archive/{YYYYMMDD_HHMMSS}/`로 이동, 새로 시작
     - **기존 수정**: 기존 `_workspace/` 유지, 수정 대상만 덮어쓰기
3. **슬라이스 정의 확인**
   - 사용자 요청에서 "어떤 기능"인지 추출 (예: "주문 등록 API")
   - 모호하면 사용자에게 1~3줄 질문

### Phase 1: 준비

1. `_workspace/` 디렉토리 생성 (없으면):
   ```
   _workspace/
   ├── handoff/      ← design 산출물 (다음 단계 입력)
   ├── design/       ← design 중간 메모
   ├── publisher/    ← publisher 메모 + components.md
   ├── backend/      ← backend 진행 메모
   ├── frontend/     ← frontend 진행 메모
   └── reviewer/     ← reviewer 보고서
   ```
2. 사용자 요구사항을 `_workspace/00_input.md`에 저장 (감사 추적)

### Phase 2: 팀 구성

```
TeamCreate(
  team_name: "vibe-coding-team",
  members: [
    { name: "design",    agent_type: "design",    model: "opus", prompt: "당신은 design 에이전트. 슬라이스: {슬라이스명}. .claude/agents/design.md 참조." },
    { name: "backend",   agent_type: "backend",   model: "opus", prompt: "당신은 backend 에이전트. design 산출물 대기. .claude/agents/backend.md 참조." },
    { name: "publisher", agent_type: "publisher", model: "opus", prompt: "당신은 publisher 에이전트. design 산출물 대기. .claude/agents/publisher.md 참조." },
    { name: "frontend",  agent_type: "frontend",  model: "opus", prompt: "당신은 frontend 에이전트. publisher + design 산출물 대기. .claude/agents/frontend.md 참조." },
    { name: "reviewer",  agent_type: "reviewer",  model: "opus", prompt: "당신은 reviewer 에이전트. 각 모듈 완성 직후 즉시 검증. .claude/agents/reviewer.md 참조." }
  ]
)
```

### Phase 3a: 디자인/요구 스펙 추출 (design + spec-extract)

design 에이전트가 spec-extract 스킬을 호출하여 입력 문서(Figma/PDF/PPT/Excel/Word/이미지) → `_workspace/handoff/design-spec.md` 생성.

```
TaskCreate(tasks: [
  { title: "디자인/요구 스펙 추출",
    description: "spec-extract 스킬 호출. 사용자가 제공한 모든 입력 형식을 design-spec.md로 통일. 완료 시 사용자에게 검토 요청.",
    assignee: "design" }
])
```

**Gate (필수)**: `_workspace/handoff/design-spec.md` 존재 확인 + 사용자 검토 통과 후 Phase 3b 진행.
- design-spec.md의 "미해결 사항"이 있으면 사용자 응답 대기
- 사용자가 "OK" 하면 Phase 3b로 진행

### Phase 3b: 설계 (design 단독, 스펙 기반)

```
TaskCreate(tasks: [
  { title: "OpenAPI/ERD/schema 생성",
    description: "_workspace/handoff/design-spec.md 기반 OpenAPI yaml + ERD + schema.sql 생성. 완료 시 backend, publisher에게 SendMessage로 '계약 확정' 알림.",
    assignee: "design",
    depends_on: ["디자인/요구 스펙 추출"] }
])
```

리더는 design 완료 메시지 또는 유휴 알림 대기.

**Gate**: `_workspace/handoff/openapi.yaml` 존재 확인 후 다음 phase 진행.

### Phase 4: 병렬 구현 (backend + publisher)

```
TaskCreate(tasks: [
  { title: "API 구현",
    description: "_workspace/handoff/openapi.yaml + design-spec.md 기반 컨트롤러/Service/Mapper 구현 + 단위 테스트. 완료 시 reviewer에게 'API 리뷰 요청' 메시지.",
    assignee: "backend" },
  { title: "마크업 작성",
    description: "_workspace/handoff/design-spec.md + design-assets/ 기반 정적 컴포넌트 생성. 원본 Figma/PDF 직접 접근 X. _workspace/publisher/components.md 갱신. 완료 시 frontend에게 'markup 완료' 메시지.",
    assignee: "publisher" }
])
```

두 에이전트 병렬 실행. 각각 완료 시 다음 단계 트리거.

### Phase 5: 프론트 동작 (frontend)

publisher 완료 메시지 수신 후:

```
TaskCreate(tasks: [
  { title: "프론트 동작 + API 연동",
    description: "publisher 컴포넌트에 상태/이벤트/API 호출 추가. backend 미완료면 mock으로 선개발. 완료 시 reviewer에게 'frontend 리뷰 요청' 메시지.",
    assignee: "frontend",
    depends_on: ["마크업 작성"] }
])
```

### Phase 6: 통합 검증 (reviewer)

reviewer는 backend/frontend가 각각 완료할 때마다 incremental 검증을 이미 수행. Phase 6에서는 **슬라이스 전체 통합 검증**:

```
TaskCreate(tasks: [
  { title: "슬라이스 통합 검증",
    description: "backend + frontend + handoff yaml 3중 경계면 교차 비교. 빌드 + 테스트 실행. 보고서 _workspace/reviewer/review-{슬라이스}.md 작성.",
    assignee: "reviewer",
    depends_on: ["API 구현", "프론트 동작 + API 연동"] }
])
```

리뷰 보고서의 **Critical**이 0개여야 다음 phase. 있으면 해당 에이전트 재호출 (Phase 4 또는 5 부분 재실행).

### Phase 7: 정리

1. 모든 팀원 종료 (SendMessage)
2. `TeamDelete`
3. `_workspace/` 보존 (감사 추적, 다음 슬라이스의 컨텍스트)
4. 사용자에게 결과 요약:
   ```
   슬라이스 '{이름}' 완료
   - 산출물:
     - 백엔드: {파일 목록}
     - 프론트: {파일 목록}
     - DB 마이그레이션: {파일}
   - 리뷰: Critical 0 / Important {N} / Minor {N}
   - 다음 슬라이스 시작 시 이 오케스트레이터 재호출
   - 배포 준비가 필요하면 deploy-checklist 스킬을 호출하세요
     (한국 SI 배포는 보통 다중 슬라이스 누적 후 진행)
   ```

## 단일 단계 모드 (서브 에이전트)

사용자가 명시적으로 "DB 설계만", "이 API만 리뷰해줘" 같은 단일 단계 요청 시:

```
Phase 0: 컨텍스트 확인 (동일)
Phase 1: 준비 (동일)
Phase 2: 단일 Agent 호출
  Agent(
    subagent_type: "{해당 에이전트}",
    model: "opus",
    prompt: "{사용자 요청 + 컨텍스트}"
  )
Phase 3: 결과 검토 + 사용자 보고
```

**TeamCreate 사용 안 함**, 오버헤드 회피.

## 데이터 흐름 (풀 슬라이스 모드)

```
[사용자]
   ↓ 요구사항
[리더(orchestrator)]
   ↓ TeamCreate
[design] → openapi.yaml ──┐
   │                       ↓
   │    SendMessage ──→ [backend] ──→ 백엔드 코드 ──┐
   │                                                   ↓
   └── SendMessage ──→ [publisher] ──→ 컴포넌트 ──→ [frontend] ──→ 프론트 코드
                                                                       ↓
                                                  [reviewer] ←─── 리뷰 요청
                                                       ↓
                                                  review-{슬라이스}.md
                                                       ↓
                                                  [리더: Critical 0 확인]
                                                       ↓
                                                  사용자 보고
```

## 에러 핸들링

| 상황 | 전략 |
|---|---|
| design 실패 | 즉시 사용자 보고, 슬라이스 중단 (다른 단계 의미 없음) |
| backend/publisher 1명 실패 | 1회 재시도 → 실패 시 사용자에게 알리고 진행 여부 확인 |
| frontend 실패 + backend 완료 | backend는 완료 보존, frontend만 재시도 |
| reviewer Critical 발견 | 해당 에이전트(backend/frontend)에 자동 재호출 + 수정 요청 (1회) |
| 빌드 깨짐 | reviewer 보고 → 원인 에이전트 재호출 |
| 사용자 요청 모호 | 추측 금지, 사용자에게 1~3줄 질문 |

## 테스트 시나리오

### 정상 흐름
1. 사용자: "주문 등록 API 만들어줘 (회원만 가능, 상품 ID + 수량 입력)" + Figma URL + 기획 PDF 첨부
2. Phase 0: project-profile.yaml 존재 확인 (Spring Boot + MyBatis + Thymeleaf + jQuery)
3. Phase 1: `_workspace/` 생성, 입력 저장
4. Phase 2: 5명 팀 구성
5. Phase 3a: design이 spec-extract 호출 → Figma + PDF → design-spec.md + design-assets/*.png 생성 → 사용자 검토 OK
6. Phase 3b: design이 openapi.yaml + ERD + V1__order.sql 생성
7. Phase 4: backend가 OrderController/Service/Mapper 구현, publisher가 design-spec.md 보고 주문 폼 템플릿 작성
8. Phase 5: frontend가 폼 검증 + AJAX 호출 추가
9. Phase 6: reviewer 검증 — Critical 0 확인
10. Phase 7: 팀 정리, 사용자 보고

### 에러 흐름
1. Phase 4에서 backend 빌드 실패 (예: Lombok 누락)
2. backend가 자체 1회 수정 시도 → 실패
3. 리더가 유휴 알림 수신 → backend SendMessage로 상태 확인
4. backend가 "Lombok 의존성 누락" 보고 → 리더가 사용자에게 안내
5. 사용자가 의존성 추가 → backend 재시작
6. 나머지 phase 정상 진행

## 후속 작업 지원 (필수)

다음 사용자 요청은 모두 이 스킬을 트리거:
- "방금 만든 주문 API에 수정사항 반영해줘"
- "슬라이스 다시 검증해줘"
- "백엔드만 다시 만들어줘"
- "리뷰에서 지적된 부분 수정해줘"

→ Phase 0에서 기존 `_workspace/` 감지 후 부분 재실행 모드 진입.
