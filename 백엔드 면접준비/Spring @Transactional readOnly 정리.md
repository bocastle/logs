# Spring @Transactional readOnly 정리

## 핵심 요약

- `@Transactional(readOnly = true)`는 transaction manager와 ORM에 조회 의도를 전달하는 최적화 hint이자 경계다.
- Hibernate의 flush·snapshot 비용과 read replica routing에 활용할 수 있지만 모든 조합에서 쓰기를 물리적으로 막지는 않는다.
- 강한 일관성이 필요한 조회와 self-invocation, 실제 SQL·flush 동작을 통합 테스트로 확인한다.

## 개념 설명

`@Transactional(readOnly = true)`는 Spring transaction metadata에 조회 전용 의도를 표시한다. 사용 중인 transaction manager, JPA provider와 JDBC driver가 이 값을 어떻게 해석하는지에 따라 실제 효과가 달라진다.

Hibernate 조합에서는 flush와 entity snapshot 비용을 줄일 수 있고 routing datasource가 replica 선택 신호로 사용할 수도 있다. 일부 DB transaction은 쓰기를 거절하지만 annotation만 보안 제약처럼 믿어서는 안 된다. 조회 중 entity를 변경하면 flush되지 않거나 예상치 못한 시점에 차이가 나타날 수 있다.

## 예시

```java
@Transactional(readOnly = true)
public OrderView find(long id) {
  return repository.findView(id);
}
```

실제 효과는 사용하는 Spring·Hibernate·driver와 database 조합의 integration test로 확인한다. 같은 class의 method를 직접 호출하는 self-invocation은 proxy를 우회할 수 있다는 점도 transaction 경계 테스트에 포함한다.

## 면접 답변 예시

> `@Transactional(readOnly = true)`는 조회 transaction이라는 의도를 manager와 ORM에 전달하는 hint입니다. Hibernate flush와 snapshot 비용을 줄이거나 replica routing 신호로 활용할 수 있지만 annotation만으로 모든 쓰기가 차단된다고 보지는 않습니다. 저장 직후처럼 read-your-writes가 필요한 조회는 primary 경로로 분리합니다. 사용하는 proxy, provider와 DB 조합에서 실제 SQL·flush와 self-invocation 동작을 통합 테스트하겠습니다.

## 장점

- 조회 use case의 불필요한 flush와 dirty checking 비용을 줄일 수 있다.
- Read replica routing의 명시적인 신호로 사용할 수 있다.

## 단점

- Self-invocation은 Spring proxy를 우회해 기대한 transaction metadata가 적용되지 않을 수 있다.
- Replica routing을 자동 적용하면 저장 직후 조회에서 stale data가 보일 수 있다.

## 주의사항 / 실무 팁

- 쓰기 방지는 DB 권한과 실제 read-only transaction 설정으로 겹쳐 보호한다.
- Entity 변경, native write SQL, self-invocation과 primary fallback 경로를 integration test에 포함한다.
