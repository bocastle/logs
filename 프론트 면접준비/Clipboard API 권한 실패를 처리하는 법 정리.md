# Clipboard API 권한 실패를 처리하는 법 정리

## 핵심 요약

- Clipboard API는 HTTPS 같은 secure context와 사용자 활성화, 브라우저 정책에 따라 실패할 수 있다.
- 복사 실패를 단순 오류로 끝내지 말고 텍스트 선택과 수동 복사가 가능한 fallback을 제공한다.
- 클립보드 읽기는 쓰기보다 민감하므로 정말 필요한 흐름인지부터 검토한다.

## 개념 설명

Clipboard API는 `navigator.clipboard`를 통해 텍스트와 일부 데이터를 비동기로 읽고 쓴다. Secure context가 필요하고, 브라우저는 사용자 활성화와 문서 포커스, iframe 권한 정책에 따라 호출을 거절할 수 있다.

`writeText()`는 복사 버튼처럼 사용자가 직접 누른 이벤트 안에서 호출하는 편이 안전하다. `readText()`는 사용자의 기존 클립보드 내용을 노출하므로 더 엄격한 권한과 UX 설명이 필요하다. 권한 상태를 미리 추측하기보다 Promise 실패를 정상 분기로 처리한다.

## 예시

```ts
try {
  await navigator.clipboard.writeText(inviteUrl);
  showToast("링크를 복사했습니다.");
} catch {
  showManualCopy(inviteUrl);
}
```

실패하면 초대 URL을 선택된 input이나 별도 대화상자에 보여 줘 사용자가 직접 복사할 수 있게 한다. 성공 토스트는 실제 Promise가 resolve된 뒤에만 표시한다.

## 면접 답변 예시

> Clipboard API는 secure context와 브라우저의 사용자 활성화 정책에 따라 실패할 수 있는 비동기 API입니다. 복사 버튼의 click handler에서 `writeText()`를 호출하고 Promise가 성공한 뒤에만 완료 메시지를 보여 주겠습니다. 실패하면 텍스트를 선택해 수동으로 복사할 수 있는 경로를 제공해야 합니다. 읽기 기능은 더 민감하므로 필요성을 따로 검토하고, iframe이라면 Permissions Policy도 확인합니다.

## 장점

- 비동기 성공·실패를 UI 흐름 안에서 명확히 처리할 수 있다.
- 수동 복사 fallback을 두면 브라우저 정책이 달라도 사용자가 작업을 마칠 수 있다.

## 단점

- 사용자 활성화 없이 자동 호출하면 브라우저마다 거절될 수 있다.
- iframe에서는 상위 문서의 Permissions Policy 설정이 추가로 필요할 수 있다.

## 주의사항 / 실무 팁

- `catch` 경로에 선택 가능한 원문과 수동 복사 안내를 둔다.
- HTTPS, 활성 탭, iframe, 모바일 브라우저에서 실제 성공과 거절 UX를 테스트한다.
