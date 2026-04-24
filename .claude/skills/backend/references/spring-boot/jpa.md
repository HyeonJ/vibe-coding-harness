# Spring Boot + JPA/Hibernate 가이드

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
- [ ] 디렉토리 구조 (controller/service/repository/domain)
- [ ] Entity 클래스 패턴 (`@Entity`, `@Table`, 연관관계)
- [ ] Repository 패턴 (`JpaRepository` 상속, `@Query`, QueryDSL)
- [ ] N+1 문제 방지 (`@EntityGraph`, fetch join)
- [ ] DTO 변환 (Entity ↔ DTO, MapStruct 또는 수동)
- [ ] `@Transactional` 트랜잭션 경계 설정
- [ ] 영속성 컨텍스트 관리 (Persistence Context)
- [ ] Auditing (`@CreatedDate`, `@LastModifiedDate`)
- [ ] 단위 테스트 (`@DataJpaTest` + Testcontainers)
- [ ] Flyway 마이그레이션 (Entity와 별도 관리)

## 참조 (글로벌 CLAUDE.md)
- 네이밍/로그/예외 처리 규칙은 동일
- DB는 운영 호환 우선 (H2 금지, Testcontainers 사용)
