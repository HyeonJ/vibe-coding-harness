# Figma 추출 가이드

> ⚠️ 골격 단계 — 첫 슬라이스 실측 시 채울 예정. figma-react-lite 자산 흡수가 revert되어 본문 제거됨.

## 채울 항목 (TODO)
- [ ] Figma URL 파싱 (fileKey, nodeId 추출)
- [ ] Figma REST Images API 호출 패턴 (`X-Figma-Token` 헤더, S3 URL → PNG 다운로드)
- [ ] FIGMA_TOKEN 환경변수 셋업 안내
- [ ] 페이지 전체 + 섹션별 baseline PNG 다운로드 → `_workspace/handoff/design-assets/`
- [ ] 반응형 프레임 자동 감지 (Desktop/Tablet/Mobile nodeId)
- [ ] 디자인 토큰 추출 (REST `/v1/files/{fileKey}` → tokens.css)
- [ ] 노드 구조 추출 (`get_design_context` MCP 또는 REST `/v1/files/.../nodes`)
- [ ] design-spec.md 작성 (spec-extract/SKILL.md의 표준 구조)
- [ ] leaf nodeId 사용 원칙 (부모 frame 사용 금지 — text-baked raster 안티패턴)
- [ ] MCP vs REST 분기 (쿼터/등록 상태에 따라)

## 결정 보류 (실측 후)
- 토큰 자동 추출 도구 — 자체 구현 vs 외부 도구(예: Specify, Figma Tokens) 의존
- visual regression 도입 (G1) — Playwright + pixelmatch with Figma baseline
