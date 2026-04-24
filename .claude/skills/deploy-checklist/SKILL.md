---
name: deploy-checklist
description: "배포 전 체크리스트 + 배포 스크립트/매니페스트 생성. 실제 배포 트리거는 사람이 수행 (자동 배포 X). project-profile.yaml의 deploy.target에 따라 tomcat, docker, k8s로 동적 분기. '배포 준비', '배포 체크리스트', '배포 스크립트', 'Dockerfile 생성', 'CI/CD 설정', '운영 배포 점검' 등의 요청 시 반드시 이 스킬을 사용할 것."
---

# Deploy Checklist Skill — 배포 준비 (실행은 사람)

## When to use
- 배포 직전 점검
- CI/CD 파이프라인 파일 생성/수정
- 배포 매니페스트 (Dockerfile, k8s YAML 등) 작성
- 롤백 계획 작성

## 핵심 원칙
**이 스킬은 자동 배포를 트리거하지 않는다.**
- 한국 SI 환경: 권한 통제 + 책임 소재로 LLM이 prod에 배포하는 것은 부적절
- 사람이 검토 + 승인 + 트리거하는 게 표준

## Workflow

### Step 1: 환경 식별
1. `.claude/project-profile.yaml` 읽기
2. `deploy.target` + `deploy.ci` 조합으로 references 분기:
   - `tomcat + jenkins` → Read `references/tomcat-jenkins.md`
   - `docker + github-actions` → Read `references/docker-gha.md`
   - 향후: `references/k8s-helm.md` 등

### Step 2: 배포 전 체크리스트 생성
`_workspace/deploy/checklist-{날짜}.md` 생성:
```markdown
## 코드 검증
- [ ] reviewer 스킬 통과 (Critical 0개)
- [ ] 모든 단위 테스트 통과
- [ ] 통합 테스트 통과 (있으면)
- [ ] 빌드 산출물 생성 (jar/war/이미지)

## 환경 변수
- [ ] prod용 환경 변수 모두 셋업 (시크릿 포함)
- [ ] 글로벌 규칙: 시크릿 하드코딩 0건

## DB
- [ ] Flyway 마이그레이션 검토
- [ ] 운영 DB 백업 완료
- [ ] 마이그레이션 롤백 계획

## 모니터링
- [ ] 로그 수집 (Slf4j 출력 위치)
- [ ] 알람 설정
- [ ] 헬스 체크 엔드포인트 동작

## 롤백 계획
- [ ] 이전 버전 백업 위치
- [ ] 롤백 명령 (사람이 실행)

## 사용자 확인 (필수)
- [ ] PM 승인
- [ ] QA 통과
- [ ] 변경 사항 사내 공지
```

### Step 3: 배포 산출물 생성 (스택별)
- Spring + Tomcat: `build/libs/*.war` 빌드 명령 + 배포 스크립트
- Docker: Dockerfile + docker-compose.yml + 빌드/푸시 스크립트
- K8s: Deployment.yaml + Service.yaml + Ingress.yaml

### Step 4: CI/CD 파일 생성/검토
- GitHub Actions: `.github/workflows/deploy.yml`
- Jenkins: `Jenkinsfile`

### Step 5: 사용자에게 핸드오프
출력 메시지:
```
배포 준비 완료. 사람이 수행할 작업:
1. 체크리스트 확인: _workspace/deploy/checklist-*.md
2. 배포 스크립트 검토: scripts/deploy.sh
3. PM 승인 후 배포 트리거: <명령 또는 CI 버튼>
```

## 절대 원칙
- **자동 배포 금지** — 항상 사람의 명시적 승인/트리거 필요
- **시크릿 노출 금지** — 환경변수 또는 secret manager 사용
- **롤백 계획 없이 배포 권유 금지**

## References
- `references/tomcat-jenkins.md` — 사내 표준 (Tomcat + Jenkins)
- `references/docker-gha.md` — 컨테이너 + GitHub Actions
- 향후: `references/k8s-helm.md`, `references/aws-ecs.md`
