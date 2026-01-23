# 방어적 복사(Defensive Copy) 정리

## 1) 개념

- **방어적 복사**는 외부로부터 전달받은 가변 객체(또는 내부 가변 상태)를 **별도의 복사본**으로 보관·반환하여, **원본 변경이 내부 상태에 영향을 주지 않도록** 막는 기법입니다.
- 핵심 목적: **불변성 강화, 캡슐화 보존, 사이드 이펙트 차단, 보안/무결성 보장**.

---

## 2) 언제 쓰나

- 생성자 인자로 **가변 객체**(예: `List`, `Date`, 가변 POJO)를 전달받을 때
- 필드 값을 반환하는 **getter**가 가변 객체를 되돌려 줄 때
- 컬렉션, 배열, 날짜/시간(구 `java.util.Date`) 등 **변경 가능한 타입**을 다룰 때

---

## 3) 자바에서의 일반 패턴

### 생성자에서

```java
public class User {
  private final List<String> tags;

  public User(List<String> tags) {
    // 1) 먼저 복사
    List<String> copy = new ArrayList<>(tags);
    // 2) 검증은 복사본에 대해 수행
    validate(copy);
    // 3) 필드에 대입
    this.tags = copy;
  }
}
```

- **복사 → 검증 → 대입** 순서를 권장합니다. 검증 전에 원본이 바뀌면 _(TOCTTOU: check-then-use 문제)_ 무결성이 깨질 수 있습니다.

### getter에서

```java
public List<String> getTags() {
  return Collections.unmodifiableList(tags); // 읽기 전용 뷰
  // 또는 필요시 완전한 복사본을 반환: return new ArrayList<>(tags);
}
```

- 외부에서 수정 시도를 해도 `UnsupportedOperationException`을 발생시켜 내부 컬렉션 보호.

### 배열

```java
public class Box {
  private final byte[] payload;

  public byte[] getPayload() {
    return payload.clone(); // 배열은 반드시 복제
  }
}
```

---

## 4) 컬렉션 반환 시 전략

- **읽기 전용 뷰**: `Collections.unmodifiableList/Set/Map` 또는 `List.copyOf(...)`(Java 10+)
- **완전 복사**: 변경 가능 컬렉션의 **새 인스턴스**를 만들어 반환
- 주의: 읽기 전용 뷰는 **원본이 바뀌면 뷰도 바뀜**. 외부 변경 전파를 **완전히** 차단하려면 **깊은 복사**가 필요합니다.

---

## 5) 얕은 복사 vs 깊은 복사

- **얕은 복사**: 컨테이너만 새로 만들고 **요소(참조)는 공유**  
  → 요소 객체가 가변이면 외부 변경이 **내부로 전파**될 수 있음.
- **깊은 복사**: 요소까지 **새 인스턴스**로 복제  
  → 비용이 크지만 **완전한 격리** 보장.
- 가장 좋은 해결은 **요소 타입 자체를 불변(Immutable)** 으로 만드는 것. (예: `record`, 불변 VO)

---

## 6) 질문의 예: `Lotto` 코드의 문제점과 개선안

### 문제 코드

```java
public class Lotto {
  private final List<LottoNumber> numbers;

  public Lotto(List<LottoNumber> numbers) {
      validateSize(numbers);
      this.numbers = new ArrayList<>(numbers);  // 방어적 복사
  }
}
```

### 문제점 1) TOCTTOU(검증-사용 시점 불일치)

- `validateSize(numbers)` 이후 `new ArrayList<>(numbers)` 사이 **원본 컬렉션이 외부에서 변경**될 수 있습니다.
- 결과적으로 검증은 통과했지만, **복사 시점의 내용은 달라져 무결성이 깨질 수 있음**.

### 문제점 2) 얕은 복사로 인한 내부 요소 변이 노출

- `new ArrayList<>(numbers)`는 **컨테이너만 복제**하고 요소인 `LottoNumber`는 **동일 객체를 참조**합니다.
- `LottoNumber`가 **가변 타입**이면 외부에서 `numbers.get(0).changeNumber(...)` 같은 변경이 **내부에 전파**됩니다.

### 개선안 A) 복사 → 검증 순서로 변경

```java
public Lotto(List<LottoNumber> numbers) {
  List<LottoNumber> copy = new ArrayList<>(numbers); // 먼저 복사
  validateSize(copy);                                 // 복사본 검증
  this.numbers = Collections.unmodifiableList(copy);  // 읽기 전용 뷰 적용(선택)
}
```

### 개선안 B) 요소의 불변화 또는 깊은 복사

- **권장**: `LottoNumber`를 **불변 타입**으로 설계 (예: `final` 필드, 상태 변경 메서드 제거, `record` 활용)
- 불변화가 어렵다면 **깊은 복사** 수행:

```java
List<LottoNumber> deepCopy = numbers.stream()
    .map(LottoNumber::copy) // 복제 팩터리/복사 생성자 제공
    .toList();
```

### 개선안 C) Getter에서도 방어적 복사/읽기 전용 뷰

```java
public List<LottoNumber> getNumbers() {
  return numbers; // numbers가 이미 unmodifiable이라면 그대로 반환해도 무방
  // 또는 필요시 return new ArrayList<>(numbers);
}
```

---

## 7) 성능·설계 상의 고려사항

- **비용**: 복사는 CPU/메모리 비용이 듭니다. 경계(외부 노출 지점)에만 선택적으로 적용하십시오.
- **불변 우선**: 가능한 한 **불변 객체**를 채택하면 **깊은 복사 비용**과 **버그 가능성**을 크게 낮출 수 있습니다.
- **스레드 안전성**: 방어적 복사는 **경합을 줄이고 가시성 문제를 완화**하는 데도 유용합니다(특히 불변과 병행).
- **API 설계**: 외부에 **가변 컬렉션 반환 지양**. 반환이 필요하면 읽기 전용 뷰나 복사본을 제공합니다.

---

## 8) 체크리스트

- [ ] 생성자에서 **복사 → 검증 → 대입** 순서를 지켰는가
- [ ] getter가 **가변 상태를 그대로 노출**하지 않는가
- [ ] 요소 타입이 **불변**인가. 아니면 **깊은 복사**가 필요한가
- [ ] 읽기 전용 뷰(`unmodifiable*`, `List.copyOf`)를 적절히 사용했는가
- [ ] 방어적 복사로 인한 **비용**과 **이득**을 균형 있게 평가했는가
