# jQuery + Thymeleaf 동적 동작 가이드 (사내 표준)

> ⚠️ 골격 단계 — 본문은 v0.1.0 빌드 시 채워질 예정.

## 채울 항목 (TODO)
- [ ] JS 디렉토리 구조 (`static/js/`, 페이지별 또는 기능별)
- [ ] 셀렉터 캐싱 패턴 (`const $btn = $('.submit-btn')`)
- [ ] 이벤트 위임 (`$(document).on('click', '.dynamic-btn', handler)`)
- [ ] AJAX 호출 (`$.ajax`, `$.get`, `$.post` + 글로벌 응답 포맷 처리)
- [ ] 폼 직렬화 + 검증 (jQuery Validation 또는 수동)
- [ ] DOM 조작 패턴 (성능 주의: append 일괄 처리)
- [ ] 모달/토스트 (Bootstrap 또는 사내 컴포넌트)
- [ ] 페이지 라이프사이클 (`$(document).ready()`)
- [ ] 글로벌 에러 핸들러 (`$.ajaxSetup`)

## 글로벌 응답 포맷 공통 처리 (필수)
```javascript
// 글로벌 CLAUDE.md: { success, data, message }
function apiCall(url, options) {
  return $.ajax({...options, url}).then(res => {
    if (!res.success) {
      // 공통 에러 처리 (toast 등)
      throw new Error(res.message);
    }
    return res.data;
  });
}
```

## 참조 (글로벌 CLAUDE.md)
- `var` 금지, `const` 기본
- 셀렉터 변수 캐싱
- 템플릿 리터럴 사용
