# Docker + GitHub Actions 배포 가이드

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
- [ ] Dockerfile 템플릿 (멀티 스테이지 빌드: build → runtime)
- [ ] `.dockerignore`
- [ ] docker-compose.yml (로컬 + prod 분리)
- [ ] GitHub Actions workflow (`.github/workflows/deploy.yml`):
  - build → test → docker build → push → deploy
- [ ] 컨테이너 레지스트리 (GHCR / Docker Hub / ECR / Harbor)
- [ ] 배포 타겟 (서버 SSH / Kubernetes / ECS)
- [ ] 환경변수 (GitHub Secrets, secret manager)
- [ ] 헬스 체크 (HEALTHCHECK 또는 외부 모니터)
- [ ] 롤백 (이전 이미지 태그로 재배포)

## 사람이 수행해야 하는 것
- GitHub Secrets 등록 (시크릿)
- 첫 배포 후 모니터링 확인
- 롤백 트리거 (필요 시)
