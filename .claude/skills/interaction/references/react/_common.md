# React 18+ 동적 구현 가이드

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
- [ ] 디렉토리 구조 (`src/features/`, `src/pages/`, `src/hooks/`)
- [ ] 상태 관리 전략 (useState / Context / Zustand / Redux)
- [ ] 서버 상태 (React Query 또는 SWR)
- [ ] HTTP 클라이언트 (axios 인스턴스 + 인터셉터 + 글로벌 응답 포맷 처리)
- [ ] 라우팅 (React Router v6)
- [ ] 폼 관리 (React Hook Form + Zod 검증)
- [ ] 인증 흐름 (토큰 저장, refresh, 보호 라우트)
- [ ] 에러 바운더리
- [ ] Suspense + lazy loading
- [ ] 환경 변수 (Vite/CRA `.env`)

## 글로벌 응답 포맷 공통 처리 (필수)
```typescript
// 글로벌 CLAUDE.md: { success, data, message }
api.interceptors.response.use(res => {
  if (!res.data.success) {
    throw new ApiError(res.data.message);
  }
  return res.data.data;
});
```

## publisher 산출물 사용
- `_workspace/publisher/components.md` 참조
- 정적 컴포넌트는 그대로 import해서 사용, 수정 필요 시 publisher에 메시지

## 참조 (글로벌 CLAUDE.md)
- `var` 금지, `const` 기본
- 템플릿 리터럴 사용
