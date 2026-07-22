# Server Action 에러를 폼 필드와 연결하는 법 정리

## 핵심 요약

- Server Action의 검증 오류는 input `name`과 같은 key로 반환해 각 필드 설명에 연결한다.
- `aria-invalid`와 `aria-describedby`로 관계를 표현하고 여러 오류를 각각 `role="alert"`로 읽게 하지 않는다.
- 제출 뒤에는 오류 요약이나 첫 오류로 포커스를 이동하되 사용자가 입력한 안전한 값은 유지한다.

## 개념 설명

Server Action은 클라이언트 검증을 우회한 요청도 받으므로 서버에서 schema와 권한을 다시 확인해야 한다. 검증 실패 결과를 필드 key와 form-level 오류로 나눠 반환하면 화면에서 수정 위치를 정확히 연결할 수 있다.

오류가 있는 input에는 `aria-invalid="true"`를 설정하고 오류 문장의 ID를 `aria-describedby`에 추가한다. 여러 필드 오류에 모두 `role="alert"`를 붙이면 한꺼번에 읽힐 수 있으므로 상단 요약 live region 하나를 사용하거나 제출 뒤 요약으로 포커스를 보낸다.

## 예시

```tsx
<input name="email" aria-invalid={!!errors.email} aria-describedby="email-error" />
<p id="email-error">{errors.email}</p>
```

서버에서 받은 email 오류를 해당 입력의 설명으로 연결한다. 기존 help text가 있다면 `aria-describedby`에 두 ID를 모두 포함한다. 상단 토스트만 보여 주면 어떤 필드를 고쳐야 하는지 찾기 어렵다.

## 면접 답변 예시

> Server Action에서는 서버 검증 결과를 input `name`과 같은 field key로 반환하겠습니다. 오류가 있는 input에 `aria-invalid`를 설정하고 오류 문장을 `aria-describedby`로 연결해 보조 기술도 관계를 알게 합니다. 오류가 여러 개라면 각 문장을 모두 alert로 읽게 하기보다 요약 영역과 첫 오류 포커스 이동을 사용합니다. 재제출과 성공 시 오래된 오류가 남지 않는지, 사용자가 입력한 안전한 값은 유지되는지도 확인합니다.

## 장점

- 시각 사용자와 보조 기술 사용자가 오류와 해당 입력의 관계를 바로 찾을 수 있다.
- 서버 검증과 클라이언트 검증을 같은 필드 UI 규칙으로 표현할 수 있다.

## 단점

- 오류마다 `role="alert"`를 붙이면 제출 순간 여러 문장이 겹쳐 읽힐 수 있다.
- 서버 field key와 input name이 어긋나면 오류가 화면에서 조용히 누락된다.

## 주의사항 / 실무 팁

- `aria-invalid`와 기존 help text를 포함한 `aria-describedby`를 함께 갱신한다.
- 이전 오류가 있는 상태에서 재제출, form-level 오류와 성공 후 cleanup을 테스트한다.
