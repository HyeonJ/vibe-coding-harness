---
name: design
description: "프로그램 상세 설계, DB 설계(ERD), API 계약(OpenAPI 3.x yaml) 산출물을 생성. 요구사항 정의서를 입력받아 backend/frontend가 병렬 구현 가능한 contract를 만든다. '설계해줘', 'ERD 만들어줘', 'API 명세 짜줘', '시퀀스 다이어그램', 'OpenAPI yaml', '계약 정의', '재설계', '설계 수정' 등 모든 설계 산출물 요청 시 반드시 이 스킬을 사용할 것."
---

# Design Skill — 프로그램·DB·API 계약 생성

## When to use
- 새 기능/슬라이스의 설계 단계 진입
- 기존 설계 수정 요청
- 계약(OpenAPI) 변경 요청

## Workflow

### Step 1: 컨텍스트 확인
1. `_workspace/handoff/` 존재 여부 확인 → 있으면 후속 작업, 없으면 신규
2. `.claude/project-profile.yaml` 읽기 (DB 종류, ORM 등)
3. 사용자 요구사항 입력 위치 확인 (사용자 메시지 / `_workspace/requirements.md` / 외부 문서)

### Step 2: 산출물 생성 (수직 슬라이스 단위)
**한 번에 한 슬라이스만 설계**한다. "전체 프로젝트 설계 한 방에" 하지 않는다.

생성 산출물:
1. **OpenAPI yaml** → `_workspace/handoff/openapi.yaml`
   - paths, schemas, examples 모두 포함
   - 글로벌 응답 포맷 `{success, data, message}` 적용
2. **ERD (필요 시)** → `_workspace/handoff/erd.md` (mermaid 또는 plain md)
3. **스키마 SQL** → `_workspace/handoff/schema.sql` (Flyway V{n}__{name}.sql 형식)
4. **시퀀스 (복잡 로직)** → `_workspace/design/sequence-{기능명}.md`

### Step 3: 후속 통지
- backend/frontend 호출 시 "이 슬라이스의 계약: `_workspace/handoff/openapi.yaml`" 전달

## Output 컨벤션
- 모든 산출물 위치: `_workspace/handoff/` (다음 에이전트 입력)
- 중간 메모: `_workspace/design/`
- handoff/ 디렉토리만 git commit, 나머지 _workspace는 gitignore

## 글로벌 CLAUDE.md 준수 (필수)
- API 응답 형식: `{success, data, message}` + 적절한 HTTP 상태코드
- DB 마이그레이션은 Flyway 가정 (schema.sql 직접 관리 X)
- DB는 운영 호환 우선 (H2 가정 X)

## References
- 스택 무관 공통 스킬 (현재 references/ 비어있음)
- 향후 도메인별 가이드 추가 가능 (예: `references/finance-api.md`)
