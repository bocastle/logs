# CSS text-wrap balance 정리

## 핵심 요약

- 제목의 시각적 균형을 높인다.
- 긴 본문에 적용하면 줄바꿈 예측이 어려워진다.
- 짧은 heading과 callout에 쓴다.

## 개념 설명

`text-wrap: balance`는 브라우저가 짧은 블록 텍스트의 줄바꿈 후보를 재평가해 각 줄의 길이를 비슷하게 맞추는 CSS 텍스트 속성이다.

`text-wrap` shorthand의 wrapping mode로 `balance`를 지정하면 줄 수를 늘리기보다 주어진 폭 안에서 줄바꿈 위치를 균형 있게 고른다.

## 예시

```css
h1 {
  text-wrap: balance;
}
```

짧은 `h1`에 balance wrapping을 켜 한 줄만 과도하게 짧아지는 것을 줄인다.

## 면접 답변 예시

> `text-wrap: balance`는 짧은 heading의 줄바꿈 후보를 다시 계산해 각 줄 길이가 지나치게 불균형하지 않게 만드는 속성입니다. 반응형 폭마다 `<br>`을 직접 넣는 것보다 content와 layout의 결합을 줄일 수 있습니다. 다만 본문처럼 긴 text에 남용하면 결과 예측과 계산 비용이 커지고 font가 바뀌면 줄바꿈도 달라집니다. 짧은 title과 callout에 제한해 사용하고 최종 web font와 대표 viewport에서 실제 줄 수를 확인하겠습니다.

## 장점

- 반응형 폭에서 자동으로 줄바꿈을 다시 고른다.

## 단점

- 컨테이너 폭과 폰트가 바뀌면 결과도 바뀐다.

## 주의사항 / 실무 팁

- 대표 viewport별로 줄 수를 확인한다.
