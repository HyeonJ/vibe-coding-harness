# vibe-coding-harness

한국 SI 웹 프로젝트의 vibe coding 워크플로우 자동화 하네스.

## 이게 뭐?

한 슬라이스(기능 1개)를 design → backend ∥ publisher → frontend → reviewer 순으로 5명 에이전트가 협업하여 end-to-end 완성합니다.

## 설치 (개발자 PC에서)

### 1. clone
```bash
git clone <레포 URL> ~/workspace/vibe-coding-harness
```

### 2. 신규 프로젝트에 적용
```bash
cd ~/workspace/your-new-project
cp -r ~/workspace/vibe-coding-harness/.claude .
cp ~/workspace/vibe-coding-harness/templates/project-profile.yaml .claude/project-profile.yaml
mkdir -p _workspace/handoff
cp ~/workspace/vibe-coding-harness/templates/_workspace.gitignore _workspace/.gitignore
```

### 3. project-profile.yaml 편집
프로젝트 스택에 맞게 값 채우기 (예: spring-boot + mybatis + thymeleaf + jquery).

### 4. Claude Code에서 사용
- 신규 슬라이스: "주문 등록 API 만들어줘 (회원만, 상품 ID + 수량)"
- 단일 단계: "DB 설계만 도와줘"
- 재실행: "방금 만든 주문 API의 검증 부분 다시 해줘"

## 구성

```
.claude/
├── agents/         5명 에이전트 정의
├── skills/         7개 스킬 (오케스트레이터 + 6개 작업 스킬)
└── project-profile.yaml   ← 프로젝트별로 작성

_workspace/         에이전트 협업 작업 영역
├── handoff/        ← git commit (다음 개발자에게 전달)
└── (나머지는 gitignore)
```

## v0.1.0 지원 스택

- **Backend**: Spring Boot + MyBatis, Spring Boot + JPA
- **Frontend (markup)**: Thymeleaf, React JSX
- **Frontend (interaction)**: jQuery, React
- **Deploy**: Tomcat + Jenkins, Docker + GitHub Actions

새 스택은 `.claude/skills/{스킬}/references/` 에 파일 추가만 하면 자동 인식.

## 글로벌 CLAUDE.md와의 관계

사용자의 `~/.claude/CLAUDE.md` 코딩 컨벤션(Slf4j 로그, 네이밍, API 응답 포맷 등)을 모든 에이전트가 자동 적용. reviewer 에이전트가 준수 여부 검증.

## 상태

- v0.1.0 골격 단계 (에이전트/스킬 정의 + references placeholder)
- 본문 채우는 작업이 아직 진행 중
- 첫 슬라이스 시나리오("주문 API 1개") 완주 검증 후 v0.1.1 배포 예정
