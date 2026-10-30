# WebUSB API 정리

## 핵심 요약

- USB 진단기와 개발 보드를 브라우저 도구에서 직접 다룰 수 있다.
- 운영체제 드라이버가 점유한 interface는 claim에 실패할 수 있다.
- 제품 명세의 configuration, interface, endpoint 번호를 코드 상수로 검증한다.

## 개념 설명

WebUSB API는 운영체제가 일반 웹용 클래스로 노출하지 않은 USBDevice의 configuration, interface, endpoint를 웹 코드가 직접 제어하게 하는 API다.

`navigator.usb.requestDevice`로 허용된 장치를 받은 뒤 open, selectConfiguration, claimInterface를 순서대로 수행하고 endpoint 번호로 `transferIn` 또는 transferOut을 호출한다.

## 예시

```ts
const device = await navigator.usb.requestDevice({
  filters: [{ vendorId: 0x1209, productId: 0x0001 }],
});
await device.open();
if (!device.configuration) await device.selectConfiguration(1);
await device.claimInterface(0);
const result = await device.transferIn(1, 64);
if (result.status === "ok" && result.data) {
  consumePacket(result.data);
} else {
  handleTransferFailure(result.status);
}
```

navigator.usb로 고른 USBDevice의 configuration과 interface를 준비한 뒤 endpoint 1에서 transferIn 한다. endpoint와 packet 형식은 장치 명세에 맞춰야 한다.

## 면접 답변 예시

> WebUSB는 browser가 허용한 USB device의 configuration, interface와 endpoint를 직접 제어하는 API입니다. User gesture에서 product까지 좁힌 filter로 선택하고 device 명세의 interface·endpoint와 packet length를 코드 상수로 검증하겠습니다. `transferIn()`은 data가 없거나 stall 같은 status를 돌려줄 수 있어 성공 여부를 확인한 뒤 decode해야 합니다. Disconnect와 작업 종료 시 pending transfer를 정리하고 interface release와 device close를 수행하며 기존 OS driver가 interface를 점유한 실패도 안내합니다.

## 장점

- 장치별 configuration과 interface 선택을 애플리케이션이 통제할 수 있다.

## 단점

- 잘못된 endpoint로 transfer하면 stall이나 데이터 손상이 발생한다.

## 주의사항 / 실무 팁

- stall 결과에는 clearHalt와 재연결 정책을 구분해 적용한다.
