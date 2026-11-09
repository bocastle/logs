# TypeScript Optional Chaining 타입 추론 정리

## 핵심 요약

- Optional Chaining은 중간 객체가 nullish일 때 예외를 내지 않고 결과 타입에 undefined를 정확히 남긴다.
- 필수여야 하는 관계에 `?.`를 연쇄 적용하면 데이터 손상을 조용한 undefined로 바꿔 원인을 숨긴다.
- 표시 기본값에는 `??`를 사용해 0과 빈 문자열은 유효 값으로 보존하고 nullish만 대체한다.

## 개념 설명

Optional Chaining은 nullish 값이면 접근을 멈추고 결과 타입에 `undefined` 가능성을 남기는 문법이다.

`?.` 뒤 결과를 바로 값으로 쓰려면 기본값 병합이나 분기 좁히기로 undefined를 처리해야 한다.

## 예시

```ts
const city: string = user.address?.city ?? "unknown";
```

중첩 값이 없을 때도 city가 string이 되도록 nullish coalescing으로 기본값을 둔다.

## 면접 답변 예시

> 추론된 `string | undefined`가 호출자에게 기본값이나 분기 처리를 요구해 누락 상태를 드러낸다. `?.`는 0, 빈 문자열, false에서는 멈추지 않으므로 기존 `&&`의 falsy 동작과 같다고 가정하면 결과가 달라진다. undefined, null, 0, 빈 문자열 사례를 테스트해 optional chain과 fallback의 실제 경계를 확인한다.

## 장점

- `obj.method?.()`는 메서드 존재 여부를 확인하면서 원래 receiver와 호출 관계를 유지한다.

## 단점

- Optional Chaining 결과로 바로 산술 연산을 하면 undefined가 런타임 NaN으로 바뀔 수 있다.

## 주의사항 / 실무 팁

- 도메인상 필수인 중간 값은 초기에 guard로 좁힌 뒤 후속 코드에서 optional 접근을 반복하지 않는다.
