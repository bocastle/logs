# Payment Request API 정리

## 핵심 요약

- 결제 정보 입력 부담을 줄일 수 있다.
- 지원 결제 수단과 브라우저 범위가 제한적이다.
- canMakePayment 결과를 보조 신호로만 사용한다.

## 개념 설명

Payment Request API는 브라우저가 저장된 결제 수단과 주소 UI를 제공해 웹 결제 흐름을 표준화하려는 API다.

앱은 결제 금액, 통화, 배송 옵션을 `PaymentRequest`에 넘기고, 사용자가 승인하면 payment response를 서버 검증과 승인 처리로 이어 간다.

## 예시

```ts
const request = new PaymentRequest(methodData, details, options);
const response = await request.show();
try {
  await chargeOnServer(response);
  await response.complete("success");
} catch (error) {
  await response.complete("fail");
  throw error;
}
```

브라우저 결제 UI는 입력을 도와줄 뿐이고 실제 승인, 재고 확인, 영수증 처리는 서버가 원자적으로 마무리해야 한다.

## 면접 답변 예시

> Payment Request API는 browser가 저장된 결제 수단과 주소 선택 UI를 제공해 checkout 입력을 줄이는 기능입니다. Browser response는 결제 승인을 대신하지 않으므로 server가 amount, inventory와 payment token을 검증해 주문을 확정해야 합니다. Server 처리가 성공한 뒤에만 `complete("success")`를 호출하고 실패·취소·지원 부재에는 기존 checkout으로 돌아갈 수 있게 하겠습니다. `canMakePayment()`는 사용 가능성을 보조하는 hint로만 보고 배송비 변경과 중복 제출도 test합니다.

## 장점

- 브라우저가 주소와 결제 수단 UI를 제공한다.

## 단점

- 서버 검증 없이 클라이언트 응답을 믿으면 결제 위조 위험이 있다.

## 주의사항 / 실무 팁

- 결제 승인과 주문 확정은 서버에서 검증한다.
