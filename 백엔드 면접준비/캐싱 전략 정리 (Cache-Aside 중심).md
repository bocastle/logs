# 캐싱 전략 정리 (Cache-Aside 중심)

> 목적: **지연 시간 단축**, **원본(DB) 부하 완화**, **처리량 향상**을 위해 캐시를 활용합니다. 서비스 특성과 **일관성/지연/가용성**의 균형에 따라 전략을 선택하셔야 합니다.

---

## 1) Cache-Aside (Lazy Loading)

### 동작 개요

1. **읽기**: 캐시에 키가 있으면(Hit) 반환, 없으면(Miss) **DB 조회 후 캐시에 적재**하고 반환
2. **쓰기**: DB를 먼저 갱신한 뒤, **캐시 무효화(삭제)** 또는 **동일 값으로 갱신**

```
읽기 흐름 (Cache-Aside)
if cache.has(key):
    return cache.get(key)           # Hit
value = db.query(key)                # Miss → DB
cache.set(key, value, TTL)           # 적재(선택적 TTL)
return value
```

### 장점

- **요청된 데이터만** 캐시에 올라 **메모리 효율**이 좋습니다.
- 캐시 장애 시에도 **DB 우회**가 가능해 내결함성이 높습니다.

### 단점/주의

- 초기/냉 캐시 구간에서 **미스 폭주**로 DB 부하가 증가할 수 있습니다.
- **캐시 불일치** 가능성이 있습니다(쓰기 순서·동시성 이슈).

### 보완 팁

- **TTL**(만료) + **버전/태그 키**로 변경 전파
- **캐시 스탬피드 방지**: 확률적 재계산, **뮤텍스 잠금**, **early refresh**
- **핫키 보호**: 슬라이딩 TTL, 다층 캐시(L1 in‑proc + L2 외부 캐시)

---

## 2) 캐시 불일치 해소를 위한 쓰기 전략

### 2.1 Write‑Through

- **쓰기 경로**: `App → Cache → DB 동기 반영`
- 캐시는 **항상 최신**이며, 재조회 시 이점이 큽니다.

```
writeThrough(key, value):
    cache.set(key, value)        # 실패 시 전체 실패로 처리(트랜잭션적)
    db.update(key, value)
```

- **장점**: 읽기 일관성↑, 콜드 미스 감소
- **단점**: **쓰기 지연 증가**(2회 쓰기), 캐시 장애 시 전체 쓰기 실패 처리 필요
- **권장**: **강한 읽기 일관성**이 중요한 read‑heavy 서비스

### 2.2 Cache Invalidation (무효화)

- **쓰기 경로**: `DB 갱신 → 캐시 키 삭제/만료`
- 다음 읽기 시 **Cache‑Aside 흐름**으로 최신화

```
update:
    db.update(key, value)
    cache.del(key)  # or set TTL very small
```

- **장점**: 쓰기 가벼움, **이중 쓰기 비용 없음**
- **단점**: 무효화 실패/지연 시 잠시 **stale** 노출
- **권장**: 변경이 **자주** 일어나나 강한 순간 일관성이 덜 중요한 경우

### 2.3 Write‑Behind(Write‑Back)

- **쓰기 경로**: `Cache 먼저 업데이트 → 비동기 배치로 DB 반영`
- 디스크/DB 쓰기를 **버퍼링/병합**하여 처리량↑

```
writeBack(key, value):
    cache.set(key, value)
    queue.enqueue({key, value})   # 백그라운드 플러시
# 배경 프로세스: 주기/트랜잭션 단위로 DB 반영 + 재시도/재전송
```

- **장점**: **쓰기 TPS 향상**, 스파이크 완화
- **단점**: 캐시와 DB가 **일시 불일치**, 장애 시 **데이터 유실 위험**(내구성 장치 필요)
- **권장**: **쓰기 빈번**·일시적 불일치를 **허용**하는 상황(예: 카운터/랭킹)

---

## 3) 전략 선택 가이드

| 기준        | Cache‑Aside              | Write‑Through          | Write‑Behind                 |
| ----------- | ------------------------ | ---------------------- | ---------------------------- |
| 읽기 일관성 | 보통(무효화 타이밍 의존) | **높음**               | 낮음(지연 반영)              |
| 쓰기 지연   | 낮음                     | **높음**               | 낮음                         |
| 구현 난이도 | 낮음                     | 중간(트랜잭션 관리)    | **높음**(내구성·재처리 필요) |
| 장애 허용성 | 캐시 장애 우회 용이      | 캐시 장애 시 쓰기 영향 | 플러시 경로 장애 리스크      |
| 적합 사례   | 범용, 읽기 위주          | 강한 읽기 일관성       | 쓰기 폭주, 카운팅/배치       |

> 기본은 **Cache‑Aside + Invalidation**을 우선 고려하시고, **읽기 일관성**이 매우 중요하면 **Write‑Through**, **쓰기 처리량**이 병목이면 **Write‑Behind**를 검토하십시오.

---

## 4) 실무 패턴·안전장치

1. **원자적 갱신 순서**

   - 보편 패턴: `DB 업데이트 → 캐시 무효화` (무효화 실패 시 재시도 큐/보상 작업)
   - 동일 트랜잭션 경계가 어려우면 **Outbox 패턴 + 소비자**로 무효화 이벤트를 보장

2. **스탬피드(동시 미스) 방지**

   - **Mutex**(setnx/분산락)로 **1인만 재계산**, 나머지는 캐시 혹은 백오프
   - **조기 갱신**(TTL 임계에서 백그라운드 리프레시), **TTL 지터**로 동시 만료 분산

3. **일시적 실패 내성**

   - 캐시/DB/큐에 **타임아웃·재시도·회로 차단기** 적용
   - 캐시 불가 시 **우회 경로** 마련(읽기: DB, 쓰기: 로그 적재 후 보상)

4. **키 설계**

   - 네임스페이스/버전 포함: `user:v2:{id}`
   - 관계형 집계는 **파생 키**로 관리: `post:{id}:count:likes`

5. **관찰 가능성**
   - 히트율, 미스 유형(냉/만료/축출), 재계산 시간, 플러시 지연, 실패율 모니터링
   - 경보 임계: 히트율 급락, 큐 적체, DB QPS 급증

---

## 5) 간단 예시 (Redis, 의사코드)

```ts
async function getUser(id: string) {
  const key = `user:v1:${id}`;
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  // stampede 방지: 분산락
  const lock = await redis.setNX(`${key}:lock`, "1", { px: 3000 });
  if (!lock) {
    await sleep(50);
    return getUser(id); // 재시도
  }

  try {
    const user = await db.selectUser(id);
    await redis.set(key, JSON.stringify(user), { EX: 3600, EXAT: undefined });
    return user;
  } finally {
    await redis.del(`${key}:lock`);
  }
}

async function updateUser(id: string, patch: any) {
  const user = await db.updateUser(id, patch); // DB 먼저
  await redis.del(`user:v1:${id}`); // 무효화
  return user;
}
```

---

## 6) 요약

- **Cache‑Aside**는 기본값으로 채택하기 좋은 전략이며, **무효화·TTL·스탬피드 방지**를 함께 설계하셔야 합니다.
- **Write‑Through**는 읽기 일관성을, **Write‑Behind**는 쓰기 처리량을 극대화합니다.
- 장애·불일치·동시성에 대비한 **보상/재시도/관찰 가능성**을 내장하시길 권장드립니다.
