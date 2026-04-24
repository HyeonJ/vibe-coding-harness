# Spring Boot + MyBatis 가이드 (사내 표준)

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
- [ ] 디렉토리 구조 (controller/service/mapper/domain)
- [ ] 컨트롤러 작성 패턴 (`@RestController`, `@RequestMapping`)
- [ ] Service 패턴 (`@Service`, `@Transactional(readOnly = true)`)
- [ ] MyBatis Mapper Interface + XML (`resources/mapper/`)
- [ ] DTO (Request/Response) 클래스 작성
- [ ] 예외 처리 (커스텀 RuntimeException + ControllerAdvice)
- [ ] 단위 테스트 (JUnit5 + Mockito + Testcontainers)
- [ ] Flyway 마이그레이션 (`V{n}__{name}.sql`)
- [ ] OpenAPI yaml → Spring 컨트롤러 변환 가이드
- [ ] 글로벌 응답 포맷 `{success, data, message}` 공통 처리

## 참조 (글로벌 CLAUDE.md)
- Slf4j 로그 규칙 (Controller/Service/catch 모두)
- 네이밍 컨벤션 (`~Controller`, `~Service`, `~Repository`, `~Request`, `~Response`)
- MyBatis: `Map<String, Object>` 금지 → 엔티티/DTO 사용
- mapper XML은 `resources/mapper/` 하위
