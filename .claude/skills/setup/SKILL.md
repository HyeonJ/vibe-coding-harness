---
name: setup
description: "신규 프로젝트 부트스트랩 — 보일러플레이트 생성, Gradle Wrapper, .gitattributes, 디렉토리 구조, project-profile.yaml 생성, 첫 빌드 검증까지. project-profile.yaml의 backend.framework + frontend.markup 조합에 따라 spring-boot-init, react-init 등 동적 분기. '프로젝트 시작', '신규 프로젝트', '보일러플레이트', '초기 셋업', '환경 구성', 'project-profile' 등의 요청 시 반드시 이 스킬을 사용할 것."
---

# Setup Skill — 신규 프로젝트 부트스트랩

## When to use
- 신규 프로젝트 시작 시 (1회성)
- 기존 프로젝트에 vibe-coding-harness 도입 시
- project-profile.yaml만 새로 만들고 싶을 때

## Workflow

### Step 1: 스택 결정 (사용자 인터뷰)
사용자에게 묻기 (옵션 제시):
- backend: spring-boot+mybatis | spring-boot+jpa | (향후 추가)
- frontend.markup: thymeleaf | react-jsx | (향후 추가)
- frontend.interaction: jquery | react | (향후 추가)
- db: postgresql | mysql | oracle | h2(테스트 only)
- ci: github-actions | jenkins | gitlab-ci

→ 결정사항을 `.claude/project-profile.yaml`에 저장

### Step 2: references 로드 + 부트스트랩 실행
스택 조합에 따라:
- `spring-boot + thymeleaf+jquery` → Read `references/spring-boot-thymeleaf.md`
- `spring-boot + react` → Read `references/spring-boot-react.md`

### Step 3: 필수 파일 생성 (글로벌 CLAUDE.md 규칙)
- `.gitattributes` (CRLF/LF 통일: `* text=auto` + 확장자별 `eol=lf`)
- `.gitignore` (스택별)
- `gradlew`, `gradlew.bat`, `gradle/wrapper/gradle-wrapper.jar` (Spring 시)
- 또는 `package.json`, `node_modules/` 셋업 (React 시)
- `_workspace/.gitkeep` + `_workspace/.gitignore`(handoff 외 모두 ignore)
- 환경 변수 `.env.example`

### Step 4: 디렉토리 구조 생성
스택별 표준 구조 (Spring 예):
```
src/main/java/{group_id}/{artifact}/
  ├── controller/
  ├── service/
  ├── repository/  (or mapper)
  ├── domain/
  └── config/
src/main/resources/
  ├── templates/  (Thymeleaf 시)
  ├── static/     (Thymeleaf 시)
  ├── mapper/     (MyBatis 시)
  └── db/migration/  (Flyway)
src/test/java/...
```

### Step 5: README 생성
clone 후 바로 빌드/실행 가능한 가이드 (글로벌 규칙):
- 사전 요구사항 (JDK 버전, Node 버전)
- `./gradlew bootRun` 또는 `npm run dev` 명령
- 환경 변수 셋업 방법

### Step 6: 첫 빌드 검증 (글로벌 규칙)
- `./gradlew build` 실행 → 통과 확인
- 통과 못 하면 사용자에게 보고 + 수정 (이 단계 통과 없이 완료 보고 금지)

### Step 7: 첫 커밋
변경된 파일 commit (메시지: `chore: vibe-coding-harness 초기 셋업`)

## 절대 원칙 (글로벌 CLAUDE.md)
- 보일러플레이트는 **clone 후 바로 빌드 가능**해야 함
- Gradle Wrapper 필수 포함
- `.gitattributes` 필수 (CRLF 이슈 사전 방지)
- 시크릿 키 하드코딩 금지 → 환경변수 + 없으면 시작 시 에러
- 빌드 + 테스트 통과 검증 후 완료 보고

## References (스택 조합별)
- `references/spring-boot-thymeleaf.md` — Spring Boot + Thymeleaf + jQuery (사내 표준)
- `references/spring-boot-react.md` — Spring Boot + React (frontend 디렉토리 분리)
- 향후: `references/nodejs-react.md` 등
