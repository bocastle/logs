# API Gateway Rate Limit Key 설계 정리

## 핵심 요약

- 한도 우회를 줄이고 과금 단위와 정책을 맞출 수 있다.
- IP만 키로 쓰면 NAT 뒤 정상 사용자가 함께 막힌다.
- 키 후보마다 cardinality와 우회 가능성을 같이 본다.

## 개념 설명

API Gateway Rate Limit Key는 어떤 호출들을 같은 한도 버킷으로 묶을지 정하는 게이트웨이 정책이다.

키는 인증된 client id, tenant id, route group, plan을 조합하고, 사용자 입력 헤더만으로 한도를 나누지 않는다.

## 예시

```text
rate_key(client) = client_id
rate_key(tenant_route) = tenant_id + ":" + route_group
rate_key(anonymous) = source_ip + ":public"
```

client 전체 폭주와 tenant별 공정성을 모두 막아야 한다면 하나의 복합 key가 아니라 각 범위의 bucket을 함께 검사한다.

## 면접 답변 예시

> Rate-limit key는 어떤 요청을 같은 사용량 bucket으로 묶을지 정하는 정책입니다. 인증된 호출은 client, tenant와 비용이 비슷한 route group을 신뢰 가능한 identity에서 만들고 사용자 임의 header만으로 분리하지 않겠습니다. Client 전체 한도와 tenant별 한도가 모두 필요하면 하나의 복합 key가 아니라 두 bucket을 각각 검사해야 우회를 막을 수 있습니다. 익명 호출은 NAT 공유 영향을 고려한 별도 정책을 두고, 차단 log에는 민감하지 않은 key 식별자와 어느 bucket이 막았는지를 남깁니다.

## 장점

- 비싼 route group을 별도 한도로 보호할 수 있다.

## 단점

- 사용자 조작 가능한 헤더를 키로 쓰면 한도를 쉽게 우회한다.

## 주의사항 / 실무 팁

- anonymous와 authenticated 정책을 분리한다.
