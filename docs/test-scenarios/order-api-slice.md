# 테스트 시나리오: 주문 등록 API 슬라이스

**목적**: vibe-coding-harness v0.1.0의 **풀 슬라이스 모드 end-to-end 완주** 검증.
**전제**: 사내 표준 스택 (Spring Boot + MyBatis + Thymeleaf + jQuery).

---

## 슬라이스 정의

**기능명**: 주문 등록

**요구사항**:
- 회원만 주문 가능 (인증 필수)
- 입력: 상품 ID, 수량
- 검증:
  - 상품 존재 + 재고 있음
  - 수량 1 이상 100 이하
- 처리:
  - 주문 생성 (상태: 'PENDING')
  - 재고 차감
- 응답: 주문 ID + 생성 시각

**스코프 외**: 결제, 배송지, 쿠폰 (다음 슬라이스)

---

## 사전 준비

### 테스트 프로젝트 생성

```bash
# 신규 프로젝트 디렉토리
mkdir -p ~/workspace/test-order-project
cd ~/workspace/test-order-project

# vibe-coding-harness 적용
cp -r ~/workspace/vibe-coding-harness/.claude .
cp ~/workspace/vibe-coding-harness/templates/project-profile.yaml .claude/project-profile.yaml
mkdir -p _workspace/handoff
cp ~/workspace/vibe-coding-harness/templates/_workspace.gitignore _workspace/.gitignore
```

### project-profile.yaml 셋업

```yaml
backend:
  framework: spring-boot
  orm: mybatis
  java_version: 17
  db: postgresql
frontend:
  markup: thymeleaf
  interaction: jquery
  bundler: none
  ts: false
testing:
  unit: junit5
  integration: testcontainers
  e2e: none
deploy:
  ci: jenkins
  target: tomcat
project:
  name: test-order
  group_id: com.test
  artifact_id: test-order
```

### 셋업 실행

Claude Code 세션에서:
```
/setup spring-boot-thymeleaf 보일러플레이트 만들어줘
```

→ Gradle Wrapper, .gitattributes, 디렉토리 구조 생성, 빌드 통과 검증.

---

## 시나리오 실행

### 사용자 입력

```
주문 등록 API 만들어줘.
- 회원만 주문 가능
- 상품 ID + 수량 입력
- 수량은 1~100 범위
- 상품 존재 + 재고 확인
- 주문 생성하고 재고 차감
- 응답: 주문 ID + 생성 시각
```

### 예상 동작

#### Phase 0: 컨텍스트 확인
- ✅ project-profile.yaml 발견 → 진행
- ✅ _workspace/ 비어있음 → 신규 슬라이스
- 슬라이스명 자동 추출: "order-create" 또는 사용자에게 확인

#### Phase 1: 준비
- _workspace/ 디렉토리 생성
- 사용자 입력을 _workspace/00_input.md 에 저장

#### Phase 2: TeamCreate
- 5명 팀 구성

#### Phase 3: design
**산출물**:
- `_workspace/handoff/openapi.yaml`
  - POST /api/orders 정의
  - Request: { productId: number, quantity: number(1~100) }
  - Response: { success: true, data: { orderId, createdAt } }
  - Error 400/401/404 응답 정의
- `_workspace/handoff/erd.md`
  - orders 테이블 (id, member_id, product_id, quantity, status, created_at)
  - products 테이블 참조
- `_workspace/handoff/schema.sql`
  - V1__create_orders.sql

**Gate**: openapi.yaml 파일 존재 확인 ✅

#### Phase 4: backend ∥ publisher

**backend 산출물**:
- `OrderController.java` — POST /api/orders 엔드포인트
- `OrderService.java` — 트랜잭션, 재고 차감 로직
- `OrderMapper.java` + `OrderMapper.xml` — INSERT 쿼리
- `Order.java` (엔티티 또는 DTO)
- `CreateOrderRequest.java`, `OrderResponse.java`
- `OrderControllerTest.java` (Testcontainers)
- Flyway: `V1__create_orders.sql`

**publisher 산출물**:
- `templates/order/create.html` (Thymeleaf)
- `static/css/order.css`
- `_workspace/publisher/components.md` — orderForm 컴포넌트 명세

#### Phase 5: frontend
**산출물**:
- `static/js/order.js`
  - 폼 검증 (수량 1~100)
  - AJAX POST /api/orders
  - 응답 처리 (성공: 주문 ID 표시, 실패: 메시지)
  - 글로벌 응답 포맷 `{success, data, message}` 공통 처리

#### Phase 6: reviewer
**산출물**: `_workspace/reviewer/review-order-create.md`

**검증 항목**:
- [ ] 빌드 성공 (`./gradlew build`)
- [ ] 단위 테스트 통과
- [ ] OrderController 응답 = openapi.yaml schema
- [ ] frontend AJAX 호출 형식 = openapi.yaml
- [ ] 글로벌 CLAUDE.md:
  - Slf4j 로그 (Controller, Service, catch)
  - 네이밍 (OrderController, OrderService, OrderMapper, CreateOrderRequest, OrderResponse)
  - @Transactional 명시
  - MyBatis 엔티티 클래스 사용
  - 와일드카드 import 없음
  - HTML 속성 순서
  - var 금지

**Critical 0** 확인 → Phase 7 진행.

#### Phase 7: 정리
- TeamDelete
- 사용자 보고:
  ```
  슬라이스 'order-create' 완료
  - 백엔드: src/main/java/com/test/order/ 5개 파일
  - 프론트: templates/order/create.html, static/css/order.css, static/js/order.js
  - DB: V1__create_orders.sql
  - 리뷰: Critical 0 / Important {N} / Minor {N}
  ```

---

## 합격 기준 (v0.1.0)

| # | 기준 | 비고 |
|---|---|---|
| 1 | 모든 7개 Phase 완주 | 중단 없이 |
| 2 | reviewer Critical 0 | 빌드/테스트/계약 모두 통과 |
| 3 | 사용자가 React/Postman 등으로 실제 호출 시 정상 응답 | 단순 코드 생성 X, 동작 검증 |
| 4 | 글로벌 CLAUDE.md 컨벤션 준수율 95% 이상 | reviewer 보고서 기준 |
| 5 | 동료가 README 따라 clone 후 직접 실행 가능 | 1명 이상 검증 |

---

## 실패 시 분석 항목

| 실패 지점 | 보완 대상 | 다음 액션 |
|---|---|---|
| design이 yaml 못 만듦 | `skills/design/SKILL.md` 본문 보완 | reference 추가 또는 워크플로우 명확화 |
| backend가 컨벤션 위반 | `skills/backend/references/spring-boot-mybatis.md` 본문 보완 | 글로벌 CLAUDE.md 규칙을 reference에 더 명시 |
| publisher가 시안 못 받음 | 디자인 입력 방법 명시 | Figma URL? 이미지? 사용자에게 물어보는 패턴 |
| frontend가 mock 못 만듦 | `interaction` 스킬에 mock 패턴 추가 | references/jquery.md에 글로벌 응답 처리 예시 |
| reviewer 통과인데 실제 호출 실패 | reviewer 검증 로직 강화 | 실제 HTTP 호출 테스트 항목 추가 |

---

## 후속 슬라이스 (이 시나리오 통과 후)

| 슬라이스 | 검증 목적 |
|---|---|
| 주문 조회 (GET /api/orders/{id}) | 같은 도메인 두 번째 슬라이스 — 컨텍스트 재사용 효율 |
| 회원 가입 화면 (전혀 다른 도메인) | 도메인 갈아타기 — 일반화 검증 |
| 결제 슬라이스 (외부 API 연동) | 외부 의존성 처리 |
| Spring + React 조합 | 다른 스택 동적 분기 검증 |
