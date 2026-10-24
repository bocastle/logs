# WebHID API 정리

## 핵심 요약

- 전용 드라이버 UI 없이 커스텀 입력 장치와 통신할 수 있다.
- 잘못된 report 길이 해석이 센서 값과 제어 명령을 손상시킬 수 있다.
- reportId별 byte 길이와 단위를 별도 decoder에서 검증한다.

## 개념 설명

WebHID API는 키보드·게임패드 표준 매핑으로 표현되지 않는 Human Interface Device의 report를 웹 애플리케이션이 읽고 쓰게 하는 장치 API다.

사용자 동작에서 `navigator.hid.requestDevice`로 필터에 맞는 장치를 선택하고 open한 뒤 HIDDevice의 `inputreport` 이벤트에서 reportId와 DataView 데이터를 해석한다.

## 예시

```ts
const [device] = await navigator.hid.requestDevice({
  filters: [{ vendorId: 0x1234, productId: 0x0001 }],
});
if (device) {
  await device.open();
  device.addEventListener("inputreport", (event) => {
    const { reportId, data } = event;
    handleHidReport(reportId, data);
  });
}
```

navigator.hid 선택 결과의 첫 HIDDevice를 열고 inputreport의 reportId에 맞는 report schema로 DataView를 해석한다.

## 면접 답변 예시

> WebHID는 표준 keyboard나 gamepad mapping으로 표현되지 않는 HID device의 report를 web application이 직접 다루는 API입니다. User gesture에서 vendor·product filter로 선택을 요청하되 취소로 빈 결과가 올 수 있고 선택 뒤에도 report ID, byte length와 unit을 별도 decoder에서 검증해야 합니다. Device 연결 해제 중 command를 보내면 실패할 수 있어 `disconnect` event에서 pending 작업과 UI 상태를 정리하겠습니다. 지원·permission fallback과 동일 model 여러 대를 구분하는 사용자 표시도 필요합니다.

## 장점

- vendorId와 productId 필터로 선택 목록을 좁힐 수 있다.

## 단점

- 사용자가 동일한 모델 여러 대를 선택하면 제품 ID만으로 장치를 구분하기 어렵다.

## 주의사항 / 실무 팁

- 선택한 제품명과 연결 상태를 화면에 계속 표시한다.
