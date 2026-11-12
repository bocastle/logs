# Web NFC API 정리

## 핵심 요약

- 물리 tag를 제품 정보나 현장 작업 흐름과 연결할 수 있다.
- 악성 tag가 피싱 URL이나 과도한 payload를 제공할 수 있다.
- recordType, payload 길이, URL protocol과 origin을 모두 검증한다.

## 개념 설명

Web NFC API는 가까운 NFC tag의 NDEF message를 읽거나 쓰는 기능을 웹 애플리케이션에 제공하는 근거리 데이터 교환 API다.

top-level secure context의 사용자 동작에서 NDEFReader를 만들고 `scan`을 요청한 뒤 reading 이벤트의 NDEF record type과 payload를 검증해 처리한다.

## 예시

```ts
const reader = new NDEFReader();
await reader.scan();
reader.addEventListener("reading", ({ message }) => {
  for (const record of message.records) {
    if (record.recordType === "url") handleTagUrl(decodeNdefText(record.data));
  }
});
```

secure context에서 NDEFReader.scan 권한을 받은 뒤 url record만 선별한다. tag 데이터는 외부 입력이므로 URL scheme과 허용 origin을 추가로 검사한다.

## 면접 답변 예시

> 카메라 QR 인식 없이 가까운 거리의 명시적 접촉을 사용할 수 있다. 백그라운드나 iframe에서 같은 동작을 기대하면 권한과 컨텍스트 제한에 걸린다. NFC가 없는 기기에는 동일 식별자를 수동 입력하거나 QR로 읽는 경로를 제공한다.

## 장점

- NDEF 표준 record로 단순한 텍스트와 URL을 교환할 수 있다.

## 단점

- 금속 환경과 기기 안테나 위치에 따라 scan 성공률이 달라진다.

## 주의사항 / 실무 팁

- 사용자에게 tag를 댈 위치와 읽기 완료 상태를 명확히 표시한다.
