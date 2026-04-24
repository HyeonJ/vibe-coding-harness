---
name: backend
description: "백엔드 API 구현. project-profile.yaml의 backend.framework + backend.orm 조합에 따라 Spring Boot+MyBatis, Spring Boot+JPA 등으로 동적 분기. design 스킬이 만든 openapi.yaml을 입력받아 컨트롤러/서비스/리포지토리/테스트를 생성. 'API 개발', '백엔드 구현', 'Spring 컨트롤러', 'Service 레이어', 'Mapper/Repository', '단위 테스트', '재구현', '백엔드 수정' 등의 요청 시 반드시 이 스킬을 사용할 것."
---

# Backend Skill — API 구현 (스택 동적)

## When to use
- design 산출물(openapi.yaml) 기반 백엔드 구현
- 기존 API 수정/확장
- 단위 테스트 작성

## Workflow

### Step 1: 스택 식별 + references 로드 (계층형)
1. `.claude/project-profile.yaml` 읽기
2. `backend.framework` 로 framework 폴더 결정 후 **공통 + 변형 둘 다 로드**:
   - `spring-boot + mybatis`:
     - Read `references/spring-boot/_common.md` (Spring 공통)
     - Read `references/spring-boot/mybatis.md` (ORM 변형)
   - `spring-boot + jpa`:
     - Read `references/spring-boot/_common.md`
     - Read `references/spring-boot/jpa.md`
   - 향후 `nodejs/_common.md + express.md`, `python/_common.md + fastapi.md` 등
   - 미지원 조합 → 사용자에게 보고 + design 스킬에 재정의 요청
3. 글로벌 `~/.claude/CLAUDE.md` (Java/Spring 코딩 규칙) 컨텍스트 유지

### Step 2: 입력 검증
- `_workspace/handoff/openapi.yaml` 존재 확인 (없으면 design 스킬 호출 권장)
- `_workspace/handoff/erd.md`, `schema.sql` 확인

### Step 3: 구현 (스택별 references 따라)
- 컨트롤러 → DTO (Request/Response) → Service → Mapper/Repository 순
- 단위 테스트 동시 작성 (TDD 권장)
- Flyway 마이그레이션 파일 생성 (`src/main/resources/db/migration/V{n}__{name}.sql`)

### Step 4: 자체 검증
- 빌드 통과 확인 (`./gradlew build`)
- 단위 테스트 통과
- 글로벌 CLAUDE.md 체크리스트 (Slf4j 로그, 와일드카드 import 금지 등)

### Step 5: 진행 메모 기록
`_workspace/backend/progress.md`에 구현한 모듈/엔드포인트 목록 기록 (frontend/reviewer 참고용)

## 글로벌 CLAUDE.md 준수 (필수)
- Slf4j 로그: Controller 진입, Service 진입/완료, catch 블록 모두
- 네이밍: `~Controller`, `~Service`, `~Repository`, `~Request`, `~Response`
- 메서드 30~50줄, 파라미터 3개 이하
- `@Transactional` 명시, 읽기 전용은 `readOnly = true`
- MyBatis: `Map<String, Object>` 금지 → 엔티티/DTO 사용
- 와일드카드 import 금지

## References (계층형 구조)
```
backend/references/
├── spring-boot/
│   ├── _common.md          ← Spring 공통 (항상 로드)
│   ├── mybatis.md          ← ORM 변형 (사내 표준)
│   └── jpa.md              ← ORM 변형
└── (향후) nodejs/, python/ 등
```
- 작동: `{framework}/_common.md` + `{framework}/{variant}.md` 둘 다 로드
- 단일 변형뿐인 경우 `_common.md` 없이 평면 파일 가능 (예: 향후 `astro.md`)
