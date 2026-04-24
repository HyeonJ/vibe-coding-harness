# 컨벤션 체크리스트

reviewer 스킬이 backend/frontend/publisher 산출물을 검증할 때 사용. 글로벌 `~/.claude/CLAUDE.md` 규칙 + figma-react-lite 게이트 흡수.

## 자동 검증 도구 (스택별)

### React + Tailwind
```bash
# 한방 실행 (G4/G5/G6/G7/G8 모두)
bash scripts/measure-quality.sh <섹션명> <섹션 디렉토리>
```

`tests/quality/{섹션명}.json` 결과 저장. JSON에 G4/G5/G6/G7/G8 각각 PASS/FAIL/SKIP.

### Spring Boot
```bash
./gradlew build      # 컴파일
./gradlew test       # 단위 테스트
./gradlew check      # Checkstyle (있으면)
```

### React/Node
```bash
npm run build
npm test
npm run lint
```

## G게이트 체크리스트 (figma-react-lite 흡수, React 작업)

### G4 — 디자인 토큰 사용 (hex literal 차단)
- 도구: `scripts/check-token-usage.mjs <dir>`
- 검사:
  - hex literal (`#1a2b3c` 등) JSX/CSS 내 직접 사용 금지
  - `style={{ backgroundColor: "#..." }}` 인라인 hex 금지
  - tailwind arbitrary `bg-[#1a2b3c]` 금지
- 예외 화이트리스트: `#fff`, `#000`
- FAIL 처리: publisher에 메시지 → tokens.css 변수로 교체

### G5 — 시맨틱 HTML (eslint jsx-a11y)
- 도구: `npx eslint <dir>`
- 검사:
  - `<div onClick>` 금지 → `<button>` 사용
  - 이미지 `alt` 누락
  - `<a>` 빈 href
  - heading 순서 위반 (`<h2>` 후 `<h4>` 등)
- FAIL 처리: publisher에 메시지 → 시맨틱 태그 교체

### G6 — 텍스트:이미지 비율
- 도구: `scripts/check-text-ratio.mjs <dir>`
- 검사: 텍스트가 raster 이미지에 baked-in 되어있는지 (대형 PNG에 텍스트 박혀있음)
- 이유: 검색 엔진 / 스크린리더 / 다국어 지원 불가
- FAIL 처리: publisher → 이미지에서 텍스트 분리 → JSX `<h2>`, `<p>` 등으로 재구성

### G7 — Lighthouse a11y/SEO (선택)
- 도구: `npx lighthouse http://127.0.0.1:5173/__preview/<섹션> --only-categories=accessibility,seo`
- 기준: a11y ≥ 95, seo ≥ 90
- 환경 미비 시 SKIP 허용
- FAIL 처리: 보고서 첨부, 사용자 결정

### G8 — i18n 가능성 (literal text)
- 도구: `scripts/check-text-ratio.mjs <dir>` (g8 필드)
- 검사: JSX에 한국어/영어 literal text 직접 박혀있는지
- 이유: 향후 i18n 시 일괄 추출 가능해야 함
- FAIL 처리: 단계적 적용 권장 — 첫 슬라이스는 OK, 다국어 도입 시 일괄 변환

## 글로벌 CLAUDE.md 검증 항목

### Java / Spring Boot
- [ ] **네이밍**: `~Controller`, `~Service`, `~Repository`, `~Request`, `~Response`
- [ ] **Boolean**: `is/has/can` 접두사 (`isActive`, `hasPermission`)
- [ ] **상수**: `UPPER_SNAKE_CASE`
- [ ] **메서드**: 30~50줄 이내, 파라미터 3개 이하
- [ ] **`@Transactional`**: 명시. 읽기 전용은 `readOnly = true`
- [ ] **catch 블록**: 비우지 않음 (최소 `log.error`)
- [ ] **null 처리**: `Optional + orElseThrow()`
- [ ] **import**: 와일드카드(`*`) 금지, 미사용 import 제거
- [ ] **MyBatis**: `Map<String, Object>` 금지 → 엔티티/DTO 사용
- [ ] **mapper XML**: `resources/mapper/` 하위 위치

### Slf4j 로그
- [ ] **Controller 진입**: `log.info("[HTTP /경로] param={}", value)`
- [ ] **Service 진입/완료**: `log.info("[메서드명] key={}", value)`
- [ ] **catch 블록**: `log.error("[메서드명] 실패", e)` — 스택트레이스 필수

### API 응답 형식
- [ ] 성공: `{ "success": true, "data": { ... } }`
- [ ] 에러: `{ "success": false, "message": "..." }`
- [ ] HTTP 상태코드: 200 / 400 / 403 / 404 / 500 적절성

### 프론트엔드
- [ ] **HTML/Thymeleaf 속성 순서**: id → class → th:* → data-* → 이벤트
- [ ] **CSS**: BEM 또는 케밥케이스, `!important` 금지, 셀렉터 깊이 3단계 이하
- [ ] **JS**: `var` 금지, `const` 기본, 템플릿 리터럴 사용
- [ ] **jQuery**: 셀렉터 변수 캐싱 (`const $btn = $('.submit')`)

### 보안
- [ ] **시크릿**: 환경변수 + 없으면 시작 시 에러 (하드코딩 금지)
- [ ] **Refresh Token**: 갱신 시 이전 토큰 명시적 삭제

### 빌드/실행
- [ ] **`./gradlew build` 통과** (글로벌 규칙: clone 후 바로 빌드 가능)
- [ ] **`.gitattributes` 존재** (CRLF/LF 통일)
- [ ] **Gradle Wrapper 포함** (gradlew, gradlew.bat, gradle-wrapper.jar)
- [ ] **`.env.example` 존재** (시크릿 키 목록)

## API 계약 정합성 (경계면 교차 비교 — 핵심)

reviewer의 가장 중요한 검증. **개별 검증 X, 두 면을 동시에 읽고 shape 비교**.

### 비교 대상 3중
1. **openapi.yaml schema** (design 산출물)
2. **backend Controller 응답 코드** (실제 반환되는 객체)
3. **frontend 호출 코드** (어떤 형식으로 받아 처리하는지)

### 불일치 패턴
- yaml: `{ orderId: number }`, backend: `Long` 반환, frontend: `string` 처리
- yaml: 응답 `data` 필드 누락, backend는 `data` 감쌈
- yaml에 없는 필드를 backend가 반환, frontend가 사용
- HTTP 상태코드 불일치 (yaml 400, backend 422 반환)

### 발견 시
→ design 에이전트에 메시지 ("yaml과 구현 불일치, 어느 쪽이 맞는지 확인 필요")
→ 자체 수정 X

## 보고서 형식

`_workspace/reviewer/review-{슬라이스명}.md`:

```markdown
# Review — {슬라이스명}
검증 일시: 2026-04-24 15:00

## Critical (수정 필수)
1. **[backend/OrderController.java:45]** 응답에 `success` 누락
   - openapi.yaml은 `{success, data, message}` 형식 명시
   - 글로벌 CLAUDE.md 규칙 위반
   - 수정: ResponseEntity 래핑 시 ApiResponse.success(data) 사용

## Important (수정 권장)
1. **[publisher/OrderForm.tsx:23]** G4 FAIL — hex literal `#2563EB`
   - tokens.css의 `--brand-primary` 사용 권장

## Minor (개선 제안)
1. **[backend/OrderService.java:67]** Slf4j 로그 누락
   - service 메서드 진입 로그 권장

## 검증 통과
- [x] Gradle build PASS
- [x] 단위 테스트 통과 (15/15)
- [x] G4: PASS
- [ ] G5: FAIL (jsx-a11y 위반 1건 → Critical 항목 참조)
- [x] G6: PASS
- [x] G8: PASS
- [x] OpenAPI 계약 일치 (3중 비교 통과)
```
