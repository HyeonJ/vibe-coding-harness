# Spring Boot 공통 가이드 (ORM 무관)

> ⚠️ 골격 단계 — Phase 8 (v0.1.0 본문 채우기)에서 채울 예정.
> 이 파일은 mybatis.md / jpa.md 모두에서 공통 로드됨.

## 채울 항목 (TODO)
- [ ] 디렉토리 구조 (controller/service/repository or mapper/domain/config)
- [ ] 컨트롤러 작성 패턴 (`@RestController`, `@RequestMapping`)
  - 글로벌 응답 포맷 `{success, data, message}` 공통 처리
  - HTTP 상태코드 (200/400/403/404/500)
- [ ] Service 패턴 (`@Service`, `@Transactional` 트랜잭션 경계)
- [ ] DTO (Request/Response) 클래스 작성
- [ ] 예외 처리 (커스텀 RuntimeException + ControllerAdvice)
- [ ] 단위 테스트 (JUnit5 + Mockito)
- [ ] 통합 테스트 환경 (Testcontainers, H2 금지)
- [ ] Flyway 마이그레이션 (`V{n}__{name}.sql`)
- [ ] OpenAPI yaml → Spring 컨트롤러 변환 가이드
- [ ] application.yml 프로파일 분리 (local/dev/stg/prod)

## 글로벌 CLAUDE.md 준수
- Slf4j 로그 (Controller/Service/catch 모두)
- 네이밍 (~Controller, ~Service, ~Repository, ~Request, ~Response)
- 메서드 30~50줄, 파라미터 3개 이하
- 와일드카드 import 금지
