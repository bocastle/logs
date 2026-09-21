# TypeScript Recursive Type 정리

## 핵심 요약

- Recursive Type으로 Json과 트리 노드를 표현하면 모든 중첩 단계에 같은 값 규칙을 재사용할 수 있다.
- 깊은 conditional recursion은 TypeScript의 instantiation 한도를 넘어 컴파일 오류와 에디터 지연을 만든다.
- leaf나 primitive 같은 명확한 종료 variant를 먼저 정의한 뒤 recursive branch가 그 union을 참조하게 한다.

## 개념 설명

Recursive Type은 자기 자신을 참조해 트리, JSON, 메뉴처럼 중첩 구조를 표현하는 타입이다.

재귀 깊이가 큰 conditional type은 컴파일 비용을 키울 수 있어 실제 데이터 구조의 경계를 정해야 한다.

## 예시

```ts
type Json = string | number | boolean | null | Json[] | { [key: string]: Json };
```

JSON처럼 중첩 가능한 값을 표현하지만 너무 깊은 타입 연산은 빌드 성능을 확인해야 한다.

## 면접 답변 예시

> base case와 재귀 branch가 union에 드러나 leaf 처리 누락을 좁히기에서 발견할 수 있다. Json 타입을 지나치게 넓게 쓰면 Date나 undefined처럼 실제 직렬화 결과가 달라지는 값의 경계가 흐려진다. 외부 그래프를 순회할 때 visited set과 최대 깊이를 적용하고 JSON 입력은 런타임 parser로 확인한다.

## 장점

- generic Tree<T>가 children에서도 T를 보존해 순회 함수와 렌더러가 node payload를 안전하게 다룬다.

## 단점

- 타입이 tree를 표현해도 런타임 객체의 순환 참조는 막지 못해 순회가 무한 반복될 수 있다.

## 주의사항 / 실무 팁

- 고정 깊이만 필요한 경로 타입은 depth counter를 두어 타입 연산이 무한히 확장되지 않게 제한한다.
