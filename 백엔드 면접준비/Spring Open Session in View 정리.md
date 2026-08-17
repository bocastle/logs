# Spring Open Session in View 정리

## 핵심 요약

- 켜진 경우 view에서 lazy loading 편의가 있다.
- view 반복 접근이 N+1 SQL을 만든다.
- API는 open-in-view=false를 우선 검토한다.

## 개념 설명

Open Session in View(OSIV)는 HTTP 요청의 view 렌더링이 끝날 때까지 Hibernate 영속성 컨텍스트를 열어 두는 패턴이다.

service transaction이 끝난 뒤에도 proxy의 lazy loading이 가능하지만 각 접근이 추가 SQL과 DB connection 사용을 일으켜 N+1을 숨길 수 있다.

## 예시

```yaml
spring:
  jpa:
    open-in-view: false
```
```java
// transaction 안에서 fetch join 또는 DTO projection
```

API 서버에서는 OSIV를 끄고 필요한 연관 데이터를 transaction 안에서 명시적으로 로딩하면 view 계층의 예측 못 한 DB 접근을 차단한다.

## 면접 답변 예시

> OSIV는 HTTP response rendering이 끝날 때까지 Hibernate session을 열어 view에서도 lazy loading을 가능하게 합니다. 편리하지만 service transaction 밖에서 SQL이 실행돼 N+1과 일관성 경계가 숨을 수 있고 느린 rendering 중 connection을 다시 점유할 수도 있습니다. API server에서는 `open-in-view=false`를 우선 검토하고 transaction 안에서 fetch join, batch fetch나 DTO projection으로 필요한 데이터를 명시적으로 준비하겠습니다. 변경 뒤에는 요청별 SQL 수와 connection checkout을 확인합니다.

## 장점

- 영속성 컨텍스트의 entity identity가 요청 동안 유지된다.

## 단점

- 느린 응답 동안 DB connection을 다시 점유한다.

## 주의사항 / 실무 팁

- fetch join·DTO projection·batch fetch를 명시한다.
