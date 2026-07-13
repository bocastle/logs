# Circuit Breaker half-open probe 설계 정리

## 핵심 요약

- Half-open은 cooldown 뒤 제한된 실제 요청만 통과시켜 dependency 회복 여부를 확인하는 단계다.
- 표본이 적을 때 p95 같은 백분위수보다 연속 성공 수, 실패 수와 개별 지연 상한을 사용하는 편이 해석하기 쉽다.
- Probe가 쓰기라면 멱등성을 보장하고, 단순 health endpoint가 실제 호출 경로를 대표하는지도 확인한다.

## 개념 설명

Circuit Breaker가 open 상태에서 cooldown을 보낸 뒤 곧바로 전체 트래픽을 허용하면 회복 중인 dependency를 다시 무너뜨릴 수 있다. Half-open은 소수 요청만 통과시켜 실제 복구 여부를 확인하는 중간 단계다.

동시에 통과시킬 probe 수와 성공 판정 기준을 미리 정한다. 표본이 다섯 개뿐인데 p95를 계산하는 것보다 “다섯 요청 중 네 번 성공하고 각 지연이 300ms 이내”처럼 직접 해석 가능한 규칙이 낫다. 실패하면 다시 open으로 보내고 cooldown을 늘릴지도 결정한다.

## 예시

```text
open 30s -> half-open
allow 5 probes
success >= 4 and each latency < 300ms -> close
otherwise -> open
```

Probe는 가능하면 실제 사용자 호출 경로를 대표해야 한다. 가벼운 health endpoint만 성공하고 실제 쿼리는 계속 실패한다면 회로를 잘못 닫게 된다. 쓰기 요청을 사용한다면 timeout 뒤 재시도돼도 안전한 멱등 기준이 필요하다.

## 면접 답변 예시

> Half-open은 cooldown이 끝난 뒤 일부 요청만 통과시켜 dependency가 실제로 회복했는지 확인하는 단계입니다. Probe 수가 적다면 p95보다 필요한 연속 성공 수와 요청별 지연 상한처럼 명확한 기준을 사용하겠습니다. 단순 health check가 실제 호출 경로를 대표하지 못할 수 있으므로 가능한 한 대표 요청을 사용하고, 쓰기라면 멱등성을 보장합니다. 실패하면 다시 open으로 전환하며 dependency 특성에 맞춰 cooldown과 동시 probe 수를 조정합니다.

## 장점

- 회복 중인 dependency에 전체 트래픽을 한꺼번에 보내는 일을 막는다.
- 성공과 지연 조건을 기준으로 자동 복구 여부를 판단할 수 있다.

## 단점

- Probe가 너무 많으면 open 상태의 보호 효과가 줄어들고 재장애를 만들 수 있다.
- 대표성이 없는 health probe는 실제 요청이 여전히 실패하는데도 회로를 닫을 수 있다.

## 주의사항 / 실무 팁

- half-open의 통과·거절 수, probe 결과와 지연을 open·closed 전이 이유와 함께 남긴다.
- 적은 표본에 백분위수를 적용하기보다 직접 설명 가능한 성공·실패 규칙을 사용한다.
