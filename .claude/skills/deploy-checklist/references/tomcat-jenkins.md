# Tomcat + Jenkins 배포 가이드 (사내 표준)

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
- [ ] Jenkinsfile 템플릿 (build → test → deploy stage)
- [ ] WAR 빌드 명령 (`./gradlew bootWar`)
- [ ] Tomcat 배포 위치 (보통 `/var/lib/tomcat/webapps/`)
- [ ] 배포 스크립트 (scp + tomcat 재시작)
- [ ] 무중단 배포 (blue-green 또는 rolling)
- [ ] 환경변수 주입 (`setenv.sh` 또는 systemd)
- [ ] 로그 위치 + 모니터링
- [ ] 롤백 절차 (이전 WAR 백업 위치 + 복원 명령)
- [ ] 헬스 체크 (Spring Actuator `/actuator/health`)

## 사람이 수행해야 하는 것 (LLM이 자동화 X)
- 운영 서버 SSH 접근
- 실 배포 트리거 (Jenkins 버튼 클릭)
- DB 마이그레이션 검토 + 백업
- 사내 변경 공지
