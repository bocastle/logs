# modulepreload 정리

## 핵심 요약

- ES module 다운로드 발견 시점을 앞당긴다.
- 사용하지 않는 module을 preload하면 byte와 CPU를 낭비한다.
- 현재 route에서 곧 import할 module에만 적용한다.

## 개념 설명

`modulepreload`는 ES module 스크립트를 실행 전에 가져오고 파싱·컴파일할 수 있게 하는 `<link>` 리소스 힌트다.

`<link rel="modulepreload" href="/route.js">`는 module request semantics와 credentials mode를 사용해 모듈 map에 결과를 준비한다. 의존 module graph의 재귀 preload 범위는 브라우저 행동을 검증한다.

## 예시

```html
<link rel="modulepreload" href="/assets/product-route.js" crossorigin />
<script type="module" src="/assets/app.js"></script>
```

곧 필요한 route module을 미리 준비하고 실제 import와 `crossorigin` 설정을 맞춘다.

## 면접 답변 예시

> `modulepreload`는 곧 사용할 ES module의 다운로드와 module map 준비를 앞당기는 resource hint입니다. 현재 route에서 실제로 import할 module에만 적용하고, 실제 import와 URL·credentials mode를 맞춰 중복 요청을 막겠습니다. 너무 많은 module을 미리 가져오면 CSS와 LCP image의 bandwidth와 CPU를 경쟁합니다. Network panel에서 initiator, 중복 요청과 사용되지 않은 preload를 확인합니다.

## 장점

- module을 실행 전에 파싱·컴파일할 수 있다.

## 단점

- credentials mode가 실제 import와 다르면 중복 요청이 생길 수 있다.

## 주의사항 / 실무 팁

- build output의 URL과 hash가 실제 import와 같은지 확인한다.
