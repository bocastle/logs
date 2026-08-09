# Permissions API 정리

## 핵심 요약

- Permissions API는 일부 browser 기능의 `granted`, `prompt`, `denied` 상태를 조회하고 변화를 관찰한다.
- `query()`는 permission prompt를 띄우지 않으며 모든 permission name과 browser 조합을 지원하지도 않는다.
- 조회 결과는 UI 힌트로 사용하고 실제 기능 호출의 성공·실패를 최종 기준으로 처리한다.

## 개념 설명

Permissions API는 geolocation과 notification 같은 일부 기능의 permission 상태를 공통 enum으로 확인하게 한다. 상태는 `granted`, `prompt`, `denied` 중 하나지만 지원하는 permission descriptor는 browser마다 다르다.

`navigator.permissions.query()`는 `PermissionStatus`를 반환할 뿐 자체로 사용자 prompt를 열지 않는다. 설정 화면이나 다른 tab에서 권한이 바뀌면 `change` event로 UI를 갱신할 수 있다. Query가 예외를 던지는 경우는 기능 전체 실패가 아니라 상태를 미리 알 수 없는 경우로 처리한다.

## 예시

```ts
const status = await navigator.permissions.query({ name: "geolocation" });
function syncPermissionUI() {
  locateButton.disabled = status.state === "denied";
  permissionHint.textContent = status.state;
}
status.addEventListener("change", syncPermissionUI);
syncPermissionUI();
```

권한 상태를 설명에 반영하되 `denied`라고 버튼을 영구적으로 없애기보다 설정 변경 방법이나 대체 흐름을 제공한다. `granted`여도 device 없음, OS policy와 실제 API 오류로 기능이 실패할 수 있다.

## 면접 답변 예시

> Permissions API는 일부 browser 기능의 현재 상태를 `granted`, `prompt`, `denied`로 조회하는 API입니다. Query 자체가 prompt를 띄우지는 않으며 permission name 지원도 browser마다 다릅니다. 결과는 버튼 설명과 대체 안내를 준비하는 데 사용하고 실제 기능 호출의 성공과 오류를 최종 기준으로 처리하겠습니다. `PermissionStatus`의 change listener는 화면 cleanup에서 제거하고 설정 변경 뒤 UI를 다시 계산합니다.

## 장점

- 설정 화면에서 바뀐 permission을 새로고침 없이 UI에 반영할 수 있다.
- Prompt 전 상태에 맞는 설명과 대체 행동을 준비할 수 있다.

## 단점

- 모든 permission name을 모든 browser에서 query할 수 있는 것은 아니다.
- `granted` 상태도 device 없음과 OS 제한 때문에 실제 기능 성공을 보장하지 않는다.

## 주의사항 / 실무 팁

- Query 예외를 상태 미확인으로 처리하고 실제 API 호출의 성공과 오류를 별도로 다룬다.
- `PermissionStatus` change listener를 component cleanup에서 제거한다.
