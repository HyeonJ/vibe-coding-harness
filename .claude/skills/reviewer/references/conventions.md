# 컨벤션 체크리스트 (글로벌 CLAUDE.md 기반)

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
글로벌 CLAUDE.md 규칙을 자동/수동 검증 항목으로 변환:

### Java / Spring Boot
- [ ] 네이밍: `~Controller`, `~Service`, `~Repository`, `~Request`, `~Response`
- [ ] Boolean: `is/has/can` 접두사
- [ ] 상수: `UPPER_SNAKE_CASE`
- [ ] 메서드 30~50줄 이내, 파라미터 3개 이하
- [ ] `@Transactional` 명시, 읽기 전용은 `readOnly = true`
- [ ] catch 블록 비우지 않음 (최소 `log.error`)
- [ ] null 대신 Optional + `orElseThrow()`
- [ ] 와일드카드 import (`*`) 금지
- [ ] 미사용 import 제거
- [ ] MyBatis: `Map<String, Object>` 금지 → 엔티티/DTO

### Slf4j 로그
- [ ] Controller 진입 시 `log.info("[HTTP /경로] param={}", value)`
- [ ] Service 진입/완료 시 `log.info("[메서드명] key={}", value)`
- [ ] catch 블록: `log.error("[메서드명] 실패", e)` (스택트레이스 필수)

### API 응답 형식
- [ ] 성공: `{ "success": true, "data": { ... } }`
- [ ] 에러: `{ "success": false, "message": "..." }`
- [ ] HTTP 상태코드 적절성 (200/400/403/404/500)

### 프론트엔드
- [ ] HTML/Thymeleaf: 속성 순서 (id → class → th:* → data-* → 이벤트)
- [ ] CSS: BEM 또는 케밥케이스, `!important` 금지, 셀렉터 깊이 3단계 이하
- [ ] JS: `var` 금지, `const` 기본, 템플릿 리터럴 사용
- [ ] jQuery 셀렉터 캐싱

### 보안
- [ ] 시크릿 키 하드코딩 금지 (환경변수 + 없으면 시작 시 에러)
- [ ] Refresh Token 갱신 시 이전 토큰 명시적 삭제

### 빌드/실행
- [ ] clone 후 바로 빌드 가능 (Gradle Wrapper 포함)
- [ ] `.gitattributes` 존재 (CRLF/LF 통일)

### API 계약 정합성 (경계면 교차 비교)
- [ ] backend Controller 응답 타입 = openapi.yaml schema
- [ ] frontend fetch 응답 처리 = openapi.yaml schema
- [ ] 위 셋이 모두 일치
