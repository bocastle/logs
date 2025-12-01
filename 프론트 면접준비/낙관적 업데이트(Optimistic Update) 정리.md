# 낙관적 업데이트(Optimistic Update) 정리

> **정의**: 서버가 **성공할 것이라고 가정**하고 **응답 전에 UI 상태를 먼저 변경**한 뒤, 결과에 맞춰 **확정 또는 롤백**하는 상호작용 패턴. 느린 네트워크에서도 **즉시성(instant feedback)** 으로 체감 성능을 높이는 데 핵심적입니다.

---

## 1) 왜 쓰나 — 장점

- **즉각 반응**: 클릭 즉시 UI가 변하므로 체감 지연 감소, 이탈율 감소.
- **작업 흐름 유지**: 리스트/피드에서 연속 상호작용(좋아요/북마크/팔로우)을 자연스럽게.
- **모바일/저품질 네트워크**에서 UX 개선.

> 단, **실패 시 잠시 잘못된 상태가 노출**될 수 있으므로 **정확한 롤백 설계**가 필수입니다.

---

## 2) 언제 쓰나 — 적용 판단 기준

**적합**

- 성공 확률이 높고(> 95% 등), 실패해도 **치명적이지 않다**: 좋아요, 북마크, 투표, 가벼운 프로필 설정.
- **서버의 최종 상태가 예측 가능**: 가산/감산 토글, 단순 생성/삭제 등.

**신중/비적합**

- **금전·재무·주문** 등 **불변/법적 기록**(영수증, 결제, 송금).
- 충돌 가능성이 높은 **동시 편집**(문서 협업) — CRDT/OT 등 별도 전략 필요.
- **실패율이 높은 네트워크**(롤백 빈번) — 오히려 혼란 가중.

---

## 3) 핵심 설계 포인트

1. **원자적 롤백**: 실패 시 **이전 스냅샷으로 정확히 복원**.
2. **중복·순서 보장**: 재시도/지연에서 **요청 순서 역전** 고려(마지막 쓰기 승리 vs 버전 비교).
3. **멱등성**: 동일 요청의 중복 도착에도 서버 상태가 안정적으로 유지. `Idempotency-Key` 권장.
4. **낙관적 UI 표시**: 진행 중임을 **시각적 피드백**(스피너/희미한 스타일/“저장 중…”)로 알림.
5. **에러 피드백**: 롤백 + 토스트/배너로 이유 설명, 재시도 버튼 제공.
6. **오프라인 우선**: 큐잉/재시도(백오프+지터), 브라우저 재가동 후 재전송.
7. **접근성**: 상태 변화에 대해 **aria‑live** 등으로 보조공학 알림.

---

## 4) TanStack Query(React Query) 패턴

### 4.1 토글(좋아요) 예시 — 낙관적 반영 + 롤백

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function useToggleLike(postId: string) {
  const qc = useQueryClient();

  return useMutation({
    mutationFn: async (next: boolean) => {
      const res = await fetch(`/api/posts/${postId}/like`, {
        method: next ? "POST" : "DELETE",
      });
      if (!res.ok) throw new Error("Failed");
      return next;
    },
    onMutate: async (next) => {
      // 1) 낙관 전 준비: 관련 쿼리 취소 & 스냅샷
      await qc.cancelQueries({ queryKey: ["post", postId] });
      const prev = qc.getQueryData<{
        id: string;
        liked: boolean;
        likes: number;
      }>(["post", postId]);

      // 2) 낙관적 업데이트
      if (prev) {
        qc.setQueryData(["post", postId], {
          ...prev,
          liked: next,
          likes: prev.likes + (next ? 1 : -1),
        });
      }

      // 3) onError에서 쓸 컨텍스트(스냅샷) 반환
      return { prev };
    },
    onError: (_err, _vars, ctx) => {
      // 4) 실패하면 롤백
      if (ctx?.prev) qc.setQueryData(["post", postId], ctx.prev);
    },
    onSettled: () => {
      // 5) 서버 소스와 재동기화
      qc.invalidateQueries({ queryKey: ["post", postId] });
    },
  });
}
```

### 4.2 리스트 항목 추가(낙관 ID + 재동기화)

```tsx
const createTodo = useMutation({
  mutationFn: async (title: string) => {
    const res = await fetch("/api/todos", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ title }),
    });
    if (!res.ok) throw new Error("create failed");
    return res.json(); // { id, title, ... }
  },
  onMutate: async (title) => {
    await qc.cancelQueries({ queryKey: ["todos"] });
    const prev = qc.getQueryData<any[]>(["todos"]) ?? [];

    const tempId = `temp-${crypto.randomUUID()}`;
    qc.setQueryData(["todos"], [{ id: tempId, title, pending: true }, ...prev]);

    return { prev, tempId };
  },
  onError: (_e, _vars, ctx) => {
    if (ctx?.prev) qc.setQueryData(["todos"], ctx.prev);
  },
  onSuccess: (created, _vars, ctx) => {
    // tempId를 실제 id로 치환
    qc.setQueryData(["todos"], (curr: any[] = []) =>
      curr.map((t) => (t.id === ctx?.tempId ? { ...created } : t))
    );
  },
  onSettled: () => qc.invalidateQueries({ queryKey: ["todos"] }),
});
```

> 팁: **교착 방지**를 위해 onSuccess에서 **invalidate** 대신 **정밀 수정(setQueryData)** 를 먼저 수행하고, 마지막에 **invalidate**로 최종 동기화를 권장합니다.

---

## 5) 충돌 해결(Conflict Resolution)

- **마지막 쓰기 승리(LWW)**: 타임스탬프/버전이 더 최신이면 덮어쓰기(간단하지만 데이터 손실 가능).
- **버전 기반 병합**: 서버가 **버전/ETag**를 요구하고 `If-Match` 실패 시 412 응답 → 클라이언트 **재취득 후 병합**.
- **필드별 병합/연산적 업데이트**: 카운터는 **가산/감산**으로 통합, 태그/세트는 **합집합/차집합** 연산.
- **협업 편집**: OT/CRDT 등 **전문 기법** 필요(낙관적 업데이트 단독 사용 지양).

---

## 6) 실패/재시도 전략

- **재시도 정책**: 지수 백오프 + **지터**(동시 재시도 폭주 방지).
- **네트워크/서버 오류 분리**: 4xx는 즉시 사용자 피드백, 5xx/네트워크는 자동 재시도 후보.
- **오프라인 큐잉**: 요청을 **IndexedDB** 등에 저장 → 온라인 전환 시 순서 보장 전송.
- **데드라인(요청 예산)**: UX 기준 시간 초과 시 사용자에게 **재시도/취소** 선택 제공.

---

## 7) 서버 측 고려사항

- **멱등성 보장**: `Idempotency-Key`, 유니크 제약으로 중복 요청 안전화.
- **사이드 이펙트**: 웹훅/푸시/로그는 **중복 안전**하게.
- **트랜잭션 경계**: DB/캐시/검색 색인을 **원자적**으로 변경(또는 보상 트랜잭션).
- **정합성 신호**: 응답에 **서버 버전/ETag/updatedAt** 포함 → 클라이언트 병합에 활용.

---

## 8) UX/접근성 패턴

- **Pending 상태** 스타일: 투명도/스켈레톤/아이콘 스피너, **취소/되돌리기** 버튼 제공 가능.
- **토스트/배너**: 성공/실패/재시도 안내. `aria-live="polite|assertive"` 로 알림.
- **낙관 표식**: 항목에 `data-pending` 속성을 두어 전역 스타일로 쉽게 제어.

```css
.item[data-pending="true"] {
  opacity: 0.6;
  pointer-events: none;
}
```

---

## 9) 안티패턴

- **스냅샷 없이 덮어쓰기**: 실패 시 복원 불가 → 항상 **이전 상태 보관**.
- **invalidate만 사용**: 서버 응답될 때까지 **UI가 원복**되는 깜빡임 → 낙관 값 유지 + 정밀 수정 권장.
- **랜덤 키 교체**: 리스트 키가 바뀌어 **컴포넌트 상태 초기화**(index 키 사용 금지와 동일 맥락).
- **거대한 그래프 낙관 수정**: 영향 범위를 최소 쿼리로 한정(쿼리 키 설계).

---

## 10) 체크리스트

- _[ ]_ 성공 확률이 높고 실패 영향이 낮은가
- _[ ]_ **onMutate → onError(롤백) → onSettled(재동기화)** 흐름이 구현되어 있는가
- _[ ]_ **스냅샷**을 안전하게 보관/복원하는가
- _[ ]_ **멱등성/중복 안전성**이 서버에서 보장되는가(Idempotency-Key 등)
- _[ ]_ **재시도/백오프/오프라인 큐** 전략이 있는가
- _[ ]_ **충돌 정책**(LWW/버전/병합)이 합의되어 있는가
- _[ ]_ **UX 피드백/접근성**이 반영되어 있는가
- _[ ]_ 로깅/관측(성공률, 롤백률, 평균 복원 시간)을 하고 있는가

---

## 11) 요약

- 낙관적 업데이트는 **즉시성 UX**의 핵심 기술이지만, **정확한 롤백·충돌 해결·멱등성** 없이는 위험합니다.
- TanStack Query의 **onMutate/onError/onSettled** 패턴으로 **스냅샷 → 낙관 반영 → 롤백/재동기화**를 표준화하십시오.
- 결제/주문 등 민감 도메인에는 **신중 적용**하고, 필요 시 **오프라인 큐·버전 관리** 등 보완책을 병행하십시오.
