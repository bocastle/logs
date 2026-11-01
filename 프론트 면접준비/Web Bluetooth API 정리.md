# Web Bluetooth API 정리

## 핵심 요약

- BLE 센서 값을 설치형 앱 없이 실시간으로 읽을 수 있다.
- 전파 간섭과 절전 때문에 GATT 연결이 자주 끊길 수 있다.
- gattserverdisconnected에서 UI를 중지하고 사용자 주도 재연결을 제공한다.

## 개념 설명

Web Bluetooth API는 주변 Bluetooth Low Energy 장치의 GATT service와 characteristic을 사용자가 선택한 웹 페이지에 제한적으로 노출하는 API다.

`navigator.bluetooth.requestDevice`에서 service UUID 필터를 선언하고 device.gatt.connect 뒤 primary service와 characteristic을 찾아 값을 읽거나 notification을 구독한다.

## 예시

```ts
const device = await navigator.bluetooth.requestDevice({
  filters: [{ services: [HEART_RATE_SERVICE] }],
});
if (!device.gatt) throw new Error("GATT is unavailable for this device");
const server = await device.gatt.connect();
const service = await server.getPrimaryService(HEART_RATE_SERVICE);
const characteristic = await service.getCharacteristic(HEART_RATE_MEASUREMENT);
await characteristic.startNotifications();
characteristic.addEventListener("characteristicvaluechanged", readHeartRate);
```

서비스 필터로 BLE 장치를 선택하고 GATT 연결 뒤 측정 characteristic notification을 구독한다. UUID와 payload byte order는 기기 프로필을 따른다.

## 면접 답변 예시

> Web Bluetooth는 사용자가 선택한 BLE device의 GATT service와 characteristic을 web page에 제한적으로 연결합니다. Service UUID로 선택 범위를 좁히고 GATT 지원 여부, characteristic flags와 payload length·byte order를 profile에 맞춰 검증하겠습니다. Wireless interference와 절전으로 연결이 자주 끊길 수 있어 `gattserverdisconnected`에서 notification 상태를 정리하고 사용자 주도 재연결을 제공합니다. 건강·위치성 sensor data는 민감정보로 보고 저장·전송 동의와 retention을 명시합니다.

## 장점

- service 필터로 요청 기능과 관련된 장치만 보여줄 수 있다.

## 단점

- 같은 서비스 UUID라도 제조사별 payload 확장이 다를 수 있다.

## 주의사항 / 실무 팁

- characteristic decoder에 길이와 flag bit 검증을 둔다.
