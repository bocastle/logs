# inputmode와 autocomplete를 함께 설계하는 기준 정리

## 핵심 요약

- 모바일 입력 속도를 높인다.
- `inputmode="numeric"`은 숫자 형식 검증이 아니다.
- 입력할 문자 종류와 값의 의미를 따로 정한다.

## 개념 설명

`inputmode`는 모바일 입력 패널의 형태를, `autocomplete`는 필드에 채울 값의 의미를 브라우저에 알리는 서로 다른 힌트다.

전화번호에 `inputmode="tel" autocomplete="tel"`을 쓰는 식으로 입력 장치와 데이터 의미를 맞추되, 둘 모두 validation을 대체하지는 않는다.

## 예시

```html
<input name="phone" type="tel" inputmode="tel" autocomplete="tel" />
```

전화번호용 키보드와 저장된 전화번호 autofill을 함께 요청하는 예다.

## 면접 답변 예시

> 키보드 UX와 필드 의미를 독립적으로 설계한다. 잘못된 autocomplete token은 민감 정보를 오입력할 수 있다. iOS와 Android에서 실제 키보드와 autofill을 확인한다.

## 장점

- 올바른 자동완성 후보를 보여 준다.

## 단점

- 소수점·부호 키 제공은 키보드마다 다를 수 있다.

## 주의사항 / 실무 팁

- 클라이언트와 서버 validation을 유지한다.
