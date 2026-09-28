# Form submit 이벤트와 requestSubmit 정리

## 핵심 요약

- 커스텀 제출 버튼에서도 native validation을 유지한다.
- `form.submit()`을 쓰면 invalid 폼이 제출될 수 있다.
- 사용자 의도에 맞는 submit button을 `requestSubmit()`에 전달한다.

## 개념 설명

`form.requestSubmit()`은 지정한 submit button을 실제로 누른 것처럼 constraint validation을 거쳐 cancelable `submit` event를 발생시키는 폼 제출 method다.

`requestSubmit(submitter)`의 button은 `SubmitEvent.submitter`와 `formaction`·`formmethod`·name/value에 반영된다. 반면 `form.submit()`은 validation과 submit event를 건너뛰 즉시 제출한다.

## 예시

```js
form.addEventListener("submit", (event) => {
  event.preventDefault();
  save(new FormData(form, event.submitter));
});
form.requestSubmit(saveButton);
```

save button을 submitter로 지정해 validation·submit event·FormData 구성을 일반 제출과 같이 유지한다.

## 면접 답변 예시

> submit event listener·analytics·FormData 흐름을 건너뛰지 않는다. submitter가 해당 form의 submit button이 아니면 예외가 발생한다. invalid·preventDefault·multiple submit·Enter key 경로를 테스트한다.

## 장점

- 여러 submit button의 의도를 `submitter`로 구분한다.

## 단점

- submit listener에서 조건 없이 다시 `requestSubmit()`하면 재귀 제출이 생긴다.

## 주의사항 / 실무 팁

- submit handler에서 `event.submitter`를 로깅해 저장·임시저장을 구분한다.
