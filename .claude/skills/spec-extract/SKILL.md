---
name: spec-extract
description: "다양한 형식의 디자인/요구사항 문서(Figma, PDF, PPT, Excel, Word, 이미지)를 단일 마크다운 스펙(_workspace/handoff/design-spec.md)으로 동적 추출. 입력 형식 자동 감지 + 형식별 도구 호출. 추출된 md는 design/publisher/frontend가 모두 공유하는 single source of truth. '디자인 추출', '문서 변환', 'Figma → md', 'PPT 분석', 'PDF 시안 변환', '요구사항 추출', '스펙 추출', '재추출' 등의 요청 시 반드시 이 스킬을 사용할 것."
---

# Spec Extract Skill — 모든 입력을 design-spec.md로 통일

## When to use
- design 에이전트 작업 시작 시 (첫 단계로)
- 디자인 변경 후 재추출 요청
- 새 요구사항 문서 (PDF/PPT/Excel) 추가
- publisher가 디자인 의도 재확인 필요할 때

## 핵심 원칙
**원본 도구를 매번 호출하지 않고, 1회 추출 후 md 파일로 협업.**
- 컨텍스트 효율: 5명 에이전트가 같은 md 참조
- 작업 안정성: 슬라이스 동안 디자인 스냅샷 고정
- 사람 검토 가능: 추출된 md를 사용자가 검수
- 입력 형식 통일: Figma든 PDF든 결국 같은 구조의 md

## Workflow

### Step 1: 기존 스펙 확인
1. `_workspace/handoff/design-spec.md` 존재 확인
   - 존재 + 사용자가 "변경 없음" → 그대로 사용 (재추출 X, 즉시 종료)
   - 존재 + 사용자가 "업데이트/재추출" → 백업 후 재생성
   - 미존재 → 신규 추출

### Step 2: 입력 형식 감지

| 입력 | 감지 | 사용 도구 |
|---|---|---|
| Figma URL (`figma.com/design/...`, `figma.com/board/...`) | URL 패턴 | `plugin:figma:figma`의 `get_design_context`, `get_screenshot` |
| `.pdf` | 파일 확장자 | `office-pdf` 스킬 |
| `.pptx`, `.ppt` | 파일 확장자 | `office-pptx` 스킬 |
| `.xlsx`, `.xls` | 파일 확장자 | `office-xlsx` 스킬 |
| `.docx`, `.doc` | 파일 확장자 | `office-docx` 스킬 |
| `.png`, `.jpg`, `.jpeg`, `.webp` | 파일 확장자 | `Read` 도구 (Claude vision) |
| Notion/Confluence URL | URL 패턴 | `WebFetch` |
| 일반 텍스트/마크다운 | 그 외 | 그대로 처리 |

여러 입력이 섞여 있으면 모두 추출 후 통합한다 (예: PPT + Figma + PDF).

### Step 3: 형식별 추출 (references 분기)

`.claude/skills/spec-extract/references/` 하위 가이드 로드:
- `references/figma.md` — Figma 추출 패턴
- `references/pdf.md` — PDF 텍스트/이미지 추출
- `references/pptx.md` — PPT 슬라이드별 추출
- `references/xlsx.md` — 엑셀 표 → 구조화 데이터
- `references/image.md` — 이미지 시각 분석
- `references/docx.md` — Word 문서

각 reference에 **무엇을 추출할지** 명시 (디자인 의도, 비즈니스 규칙, 데이터 구조 등).

### Step 4: 통합 design-spec.md 생성

표준 구조 (모든 형식 공통):

```markdown
# Design Spec — {슬라이스명}

## 메타
- 추출 시각: {YYYY-MM-DD HH:mm}
- 입력 출처:
  - Figma: {URL} (있으면)
  - PDF: {파일 경로} (있으면)
  - PPT: {파일 경로} (있으면)
  - 그 외: ...
- 추출자: spec-extract skill

## 화면 (있는 경우)
### {화면명 1}
- 스크린샷: `_workspace/handoff/design-assets/{이름}.png`
- 컴포넌트:
  - {컴포넌트명}: {역할}
    - props: {props 목록 + 타입}
    - 디자이너 노트: {Figma description 또는 시안의 메모}
- 인터랙션:
  - {상태/동작}: {설명}
- 디자인 토큰:
  - 색상: {hex 목록}
  - 폰트: {폰트명}
  - 스페이싱: {규칙}

### {화면명 2}
...

## 비즈니스 규칙 (PDF/PPT/요구사항 문서에서 추출)
- {규칙 1}: {설명}
  - 근거: {원본 문서 위치 — 예: "PDF 12쪽"}
- {규칙 2}: ...

## 데이터 구조 (Excel/Word에서 추출)
| 필드 | 타입 | 설명 | 출처 |
|---|---|---|---|
| ... | ... | ... | ... |

## 미해결 사항 (사용자 확인 필요)
- [ ] {모호한 항목 1}
- [ ] {모호한 항목 2}
```

### Step 5: 비주얼 자산 보존
- 화면 스크린샷이 있으면 `_workspace/handoff/design-assets/{이름}.png`로 저장
- design-spec.md에서 상대 경로로 참조
- publisher가 마크업 생성 시 시각 참조용으로 활용

### Step 6: 사용자 검토 (필수)
추출 완료 시 사용자에게 보고:
```
디자인 스펙 추출 완료: _workspace/handoff/design-spec.md
- 화면 N개, 비즈니스 규칙 M개 추출
- 미해결 사항 K개 (확인 필요)
검토 후 design 에이전트 작업 진행하시겠어요?
```

→ 잘못 해석한 부분을 backend/publisher 작업 시작 전에 잡을 기회.

## 절대 원칙
- **원본을 변경하지 않음** — 추출만, 원본 파일/Figma는 read-only
- **추측 금지** — 디자인 의도가 모호하면 "미해결 사항"에 명시, 진행 X
- **출처 명시** — 모든 비즈니스 규칙에 원본 문서 위치 기록 (감사 추적)

## 출력
- **필수**: `_workspace/handoff/design-spec.md`
- **선택**: `_workspace/handoff/design-assets/*.png` (시각 자료가 있을 때)
- **선택**: `_workspace/handoff/raw/{원본 파일명}` (원본 보존이 필요한 경우)

## 글로벌 CLAUDE.md 준수
- 추출된 md는 한국어로 작성 (사용자 환경)
- 코드 예시 등 영어 유지가 자연스러운 부분은 영어 유지

## References
- `references/figma.md` — Figma 추출 (plugin:figma:figma MCP 활용)
- `references/pdf.md` — PDF (office-pdf 스킬)
- `references/pptx.md` — PowerPoint (office-pptx 스킬)
- `references/xlsx.md` — Excel (office-xlsx 스킬)
- `references/image.md` — 이미지 (Claude vision)
- `references/docx.md` — Word (office-docx 스킬)
