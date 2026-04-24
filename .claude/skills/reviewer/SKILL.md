---
name: reviewer
description: "코드 품질 + API 계약 정합성 + 테스트 검증. 글로벌 CLAUDE.md 코딩 컨벤션 준수 + backend 응답이 openapi.yaml과 일치하는지 경계면 교차 비교 + 단위/통합 테스트 실행. 각 모듈 완성 직후 점진적 실행 (incremental QA). '코드 리뷰', '검증', 'QA', '테스트 실행', '컨벤션 체크', '재검토', '리뷰 결과 확인' 등의 요청 시 반드시 이 스킬을 사용할 것."
---

# Reviewer Skill — 컨벤션·계약·테스트 검증

## When to use
- backend/frontend 모듈 완성 직후 (incremental)
- 슬라이스 전체 완료 후 통합 검증
- 빌드/배포 직전 최종 점검

## Workflow

### Step 1: 검증 범위 식별
1. 사용자 요청에서 범위 파악:
   - 특정 모듈: 해당 파일/디렉토리만
   - 슬라이스 전체: backend + frontend + handoff yaml 모두
2. `.claude/project-profile.yaml`로 스택 확인 (스택별 컨벤션 적용)

### Step 2: 자동 검증 (스크립트)
가능한 한 명령으로 실행:
- 빌드: `./gradlew build` (Spring) 또는 `npm run build` (React)
- 테스트: `./gradlew test` 또는 `npm test`
- 린트: 사용 가능한 linter (ESLint, Checkstyle 등)
- 추가 자동 게이트 (visual regression, token usage 등) 도입은 첫 슬라이스 실측 후 결정 (CLAUDE.md "알려진 갭" 참조)

### Step 3: 경계면 교차 비교 (핵심)
**개별 검증 X, 두 면을 동시에 읽고 shape 비교**:
- backend Controller의 `@GetMapping("/api/orders/{id}")` 응답 타입
- frontend의 `fetch('/api/orders/' + id)` 처리 코드
- 둘이 일치하는지 + openapi.yaml과도 일치하는지 3중 비교

### Step 4: 컨벤션 체크 (글로벌 CLAUDE.md 기반)
Read `references/conventions.md` 후 항목별 검증.

### Step 5: 보고서 작성
`_workspace/reviewer/review-{슬라이스명}.md` 생성:
```markdown
## Critical (수정 필수)
- [파일:라인] 문제 설명 + 이유

## Important (수정 권장)
- ...

## Minor (개선 제안)
- ...

## 검증 통과
- [x] 빌드 성공
- [x] 테스트 통과
- [ ] OpenAPI 계약 일치 (불일치 항목 위 명시)
```

### Step 6: 후속 액션
- Critical 있으면 → backend/frontend에 메시지 + 수정 요청
- 모두 통과 → 리더에게 "검증 완료" 보고

## 절대 원칙
- **자체 수정 금지** — 발견만 하고 수정은 backend/frontend의 책임
- **Critical은 즉시 알림** — 빌드 실패, 계약 위반은 보고서 작성 전이라도 즉시 메시지

## References
- `references/conventions.md` — 글로벌 CLAUDE.md 기반 체크리스트
- 향후: `references/security-checklist.md`, `references/perf-checklist.md`
