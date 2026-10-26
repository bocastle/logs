# Browser Preload Scanner 정리

## 핵심 요약

- parser blocking 중에도 핵심 resource 발견을 진행한다.
- scanner가 표준 실행 순서를 바꾸는 것은 아니다.
- LCP image와 critical stylesheet URL을 initial HTML에 노출한다.

## 개념 설명

Browser Preload Scanner는 main HTML parser가 blocking script에 멈춘 동안에도 앞쪽 markup을 speculative하게 스캔해 image·stylesheet·script 요청을 미리 발견하는 브라우저 최적화다.

scanner는 HTML attribute에 드러난 URL을 빠르게 찾지만 JS가 나중에 삽입하는 resource, CSS 파싱 후 발견되는 background image, 복잡한 실행 결과는 예측하지 못한다.

## 예시

```html
<link rel="stylesheet" href="/app.css" />
<img src="/hero.avif" fetchpriority="high" alt="…" />
<script src="/legacy.js"></script>
```

hero URL을 initial markup에 두어 parser-blocking script에 도달하기 전 scanner가 요청을 발견할 수 있게 한다.

## 면접 답변 예시

> JS 실행 후 요청을 만드는 구조의 늦은 discovery를 찾는다. HTML에 단서가 있어도 잘못된 priority로 queue에서 기다릴 수 있다. JS-injected resource는 markup 또는 제한적 preload로 앞당길 가치가 있는지 측정한다.

## 장점

- initial HTML에 핵심 resource 단서를 두는 가치를 설명한다.

## 단점

- JS 문자열 안 URL은 안정적으로 preload할 수 없다.

## 주의사항 / 실무 팁

- waterfall에서 discoverer·initiator·request start를 구분한다.
