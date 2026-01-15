# `equals`와 `hashCode`는 왜 함께 재정의해야 하는가

> 본 문서는 Java를 기준으로 **동등성 계약(equality contract)**, **해시 기반 컬렉션의 동작**, **실패 사례와 해결법**, **주의점/체크리스트**를 정리합니다.

---

## 1) 핵심 요지

- **계약(Contract)**: `equals`가 **true**를 반환하는 두 객체는 **항상 같은 `hashCode`** 를 반환해야 합니다.
- 이유: `HashMap`/`HashSet` 등 해시 기반 컬렉션은 **버킷 선택에 `hashCode`**, **동등성 최종 판정에 `equals`** 를 사용합니다.  
  두 메서드가 함께 일관되게 정의되지 않으면 **동일한 객체임에도 다른 버킷**에 들어가 **중복/탐색 실패**가 발생합니다.

---

## 2) 해시 기반 컬렉션의 판정 절차

1. 키/요소의 `hashCode()`로 **버킷 인덱스**를 계산합니다.
2. 같은 버킷(또는 트리 노드) 내 후보와 **`equals()` 비교**로 최종 동일성을 확인합니다.

> 즉, **같은 객체로 취급되려면** 두 객체가 **동일한 버킷**에 도달해야 하므로 `hashCode`가 같아야 하며, **마지막에 `equals`가 true** 여야 합니다.

---

## 3) 실패 사례(질의의 예제 요약)

```java
class Subscribe {
    private final String email;
    private final String category;

    @Override
    public boolean equals(Object o) { /* 필드 동등성 비교 */ }
    // hashCode 미구현 → Object.hashCode() 사용(객체 식별자 기반, 보통 서로 다름)
}
```

- `HashSet`에 **값이 같은 두 인스턴스**를 넣어도, `hashCode`가 서로 달라 **서로 다른 버킷**에 들어가 **중복 제거가 실패**합니다.

---

## 4) 올바른 구현 예시

### 4.1 수동 구현

```java
import java.util.Objects;

public final class Subscribe {
    private final String email;
    private final String category;

    public Subscribe(String email, String category) {
        this.email = email;
        this.category = category;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Subscribe that = (Subscribe) o;
        return Objects.equals(email, that.email)
            && Objects.equals(category, that.category);
    }

    @Override
    public int hashCode() {
        return Objects.hash(email, category);
    }
}
```

### 4.2 `record` 활용(Java 16+)

```java
public record Subscribe(String email, String category) { }
// 자동으로 내용 기반 equals/hashCode/toString 생성
```

### 4.3 Lombok

```java
@lombok.EqualsAndHashCode
@lombok.ToString
@AllArgsConstructor
public class Subscribe {
    private final String email;
    private final String category;
}
```

---

## 5) 계약 상세(요건 정리)

### `equals`의 일반 계약

- **반사성**: `x.equals(x)` 는 항상 true
- **대칭성**: `x.equals(y)` ↔ `y.equals(x)` 동일
- **추이성**: `x.equals(y)` && `y.equals(z)` ⇒ `x.equals(z)`
- **일관성**: 비교 대상 속성이 변하지 않으면 결과도 변하지 않음
- `x.equals(null)` 은 항상 false

### `hashCode`의 일반 계약

- **일관성**: 객체 상태가 변하지 않으면, 같은 실행 중에는 같은 값 반환
- **equals true ⇒ hashCode 동일**(필수)
- **equals false ⇒ hashCode가 달라야만 하는 것은 아님**(권장: 충돌을 줄이도록 충분히 분산)

---

## 6) 흔한 함정과 주의 사항

1. **가변 필드를 포함한 동등성**

   - 키/요소로 사용 중인 객체의 **동등성에 참여하는 필드가 변경**되면, 컬렉션 내부에서 **탐색/삭제 실패**가 발생할 수 있습니다.
   - 해결: **불변(immutable) 설계** 또는 **키로 쓰는 동안 상태 변경 금지**.

2. **`equals`/`hashCode` 필드 불일치**

   - `equals`에는 포함했지만 `hashCode`에는 누락(또는 반대) → **계약 위반**.
   - 두 메서드에서 **동일한 필드 집합**을 사용하십시오.

3. **상속과 대칭성 위반**

   - `equals`에 `instanceof`를 쓸지 `getClass()`를 쓸지에 따라 **서브타입 간 동등성**이 달라집니다.
   - 도메인 요구에 맞춰 **대칭성**이 깨지지 않도록 일관된 정책을 선택하십시오.

4. **성능만을 위한 취약한 해시**

   - 단순 상수/편중된 해시는 **충돌 증가**로 성능 저하를 유발합니다.  
     `Objects.hash(...)` 또는 **좋은 분산 함수**를 사용하십시오.

5. **컬렉션별 비교 규칙 혼동**
   - `TreeSet`/`TreeMap`은 `Comparator`/`compareTo` 기준으로 **정렬 동등성**을 사용(해시 미사용).  
     `compareTo`와 `equals` 기준이 불일치하면 **의외의 포함 여부**가 발생할 수 있습니다.

---

## 7) 실전 체크리스트

- [ ] `equals`가 **내용 기반**으로 올바른 대칭/추이/일관성을 만족하는가
- [ ] `hashCode`가 **equals와 같은 필드**로 계산되는가
- [ ] 동등성에 참여하는 필드는 **가변인가?** 키로 쓰는 동안 변경을 막았는가
- [ ] 상속 구조에서 **대칭성/일관성**을 보장하는가
- [ ] 대형 컬렉션에서도 해시가 **충분히 분산**되는가(충돌률 점검)

---

## 8) 데모: `HashSet` 중복 제거 확인

```java
Set<Subscribe> set = new HashSet<>();
set.add(new Subscribe("team.maeilmail@gmail.com", "backend"));
set.add(new Subscribe("team.maeilmail@gmail.com", "backend"));
System.out.println(set.size()); // 1 (equals + hashCode 모두 구현 시)
```

---

## 9) 요약

- 해시 기반 컬렉션은 **`hashCode`로 버킷 선택 → `equals`로 최종 판정**을 수행합니다.
- 따라서 `equals`만 재정의하거나, 두 메서드가 **다른 필드를 기준**으로 구현되면 **계약 위반**으로 **중복 처리/탐색 실패**가 발생합니다.
- **불변 값 타입 설계**, `record`/Lombok 활용, 일관된 필드 집합 사용으로 안전하고 예측 가능한 동작을 보장하십시오.
