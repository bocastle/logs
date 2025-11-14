# 자바의 얕은 복사 vs 깊은 복사 정리

> 본 문서는 자바에서 **얕은 복사(Shallow Copy)** 와 **깊은 복사(Deep Copy)** 의 개념, 예제, 주의사항, 실무 모범 사례를 정리합니다.

---

## 1) 핵심 개념

- **얕은 복사**: **최상위 객체만 새로 만들고**, 내부 참조 필드는 **원본과 같은 참조**를 공유합니다.
- **깊은 복사**: **최상위 + 내부 참조까지 재귀적으로 새로 생성**하여 **서로 독립**된 객체 그래프를 만듭니다.

> 값 타입(예: `String`은 불변)이면 공유해도 안전하지만, **가변 참조 필드**는 얕은 복사에서 **원본/사본이 서로 영향**을 주게 됩니다.

---

## 2) 예제(질의 코드 기반)

```java
class Book {

    private String name; // 책 이름(불변 String)
    private Author author; // 저자(가변)

    public Book(String name, Author author) {
        this.name = name;
        this.author = author;
    }

    public Book shallowCopy() { // 얕은 복사
        return new Book(this.name, this.author);
    }

    public Book deepCopy() { // 깊은 복사
        Author copiedAuthor = new Author(this.author.getName());
        return new Book(this.name, copiedAuthor);
    }

    public void changeAuthor(String name) { // 저자 이름 변경
        author.setName(name);
    }

    @Override
    public String toString() {
        return "Book name : " + name + ", " + author;
    }

    static class Author {
        private String name;
        public Author(String name) { this.name = name; }
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        @Override public String toString() { return "Author : " + name; }
    }
}
```

- **얕은 복사** 후 사본에서 `changeAuthor()`를 호출하면 **원본의 저자도 함께 변경**됩니다(동일 `Author` 참조 공유).
- **깊은 복사** 후 사본에서 저자를 바꿔도 **원본은 유지**됩니다(서로 다른 `Author` 인스턴스).

---

## 3) 자주 쓰는 복사 패턴과 주의점

### 3-1. Copy Constructor / Factory

- 가장 명시적이고 안전한 방법: **복사 생성자** 또는 **정적 팩토리**로 어떤 필드를 **깊게/얕게** 복사할지 통제합니다.

```java
public Book(Book other) {
    this.name = other.name; // String은 불변 → 공유 가능
    this.author = new Author(other.author.getName()); // 깊은 복사
}
```

- 장점: null 처리/불변식 검증을 함께 수행 가능.

### 3-2. `clone()`/`Cloneable`

- `Cloneable` + `Object#clone()`은 **얕은 복사 기본**이며, 안전하게 쓰려면 **깊은 복사 로직을 직접 구현**해야 합니다.
- 예외 처리(`CloneNotSupportedException`)와 **얕은 복사 함정** 탓에 **일반적으로 비권장**입니다(Effective Java 권고).

### 3-3. 직렬화 기반 복사

- `Serializable` + 직렬화/역직렬화로 **깊은 복사 효과**를 낼 수 있습니다.
- 단점: **무겁고 느림**, 모든 하위 객체가 `Serializable` 이어야 함.

### 3-4. 컬렉션 복사

- `new ArrayList<>(originalList)` 는 **컨테이너만 복사(얕은 복사)** 합니다. 요소 객체는 같은 참조를 가리킵니다.
- **깊은 복사**가 필요하면 **요소 단위**로 복사 생성자/팩토리를 호출하세요.

```java
List<Book> deepCopied = original.stream()
    .map(Book::new) // 복사 생성자
    .toList();
```

### 3-5. 불변(Immutable) 설계

- **깊은 복사 대신 불변**을 택하면 안전성이 크게 향상됩니다.
  - 세터 제거, 모든 필드 `final`, 가변 필드도 **방어적 복사**로 캡슐화.
  - 컬렉션은 `List.copyOf`, `Map.copyOf`, `Collections.unmodifiable*` 로 **불변 뷰** 제공.
- 불변 객체는 복사 비용이 낮고 **공유가 안전**합니다.

---

## 4) 방어적 복사(Defensive Copy)의 원칙

- **주입/반환 경계**에서 가변 참조를 그대로 넘기지 말고 **복사본**을 사용합니다.

```java
public final class Library {
    private final List<Book> books;
    public Library(List<Book> books) {
        // 방어적 복사 + 불변 뷰
        this.books = List.copyOf(books.stream().map(Book::new).toList());
    }
    public List<Book> books() { return books; } // 불변 리스트(수정 시 예외)
}
```

- 이로써 내부 상태가 외부에서 **우회 변경**되는 것을 방지합니다.

---

## 5) 깊은 복사의 난점과 전략

- 객체 그래프가 **깊고 순환 참조**가 있으면 단순 재귀 복사는 **무한 루프/중복 복제** 위험이 있습니다.
  - 전략: **방문 집합(IdentityHashMap)** 으로 이미 복사한 객체를 추적하여 재사용.
- 대규모 그래프의 전체 깊은 복사는 **비용이 큼**: 요구사항에 따라 **부분 깊은 복사**(필요 필드만)로 타협합니다.

---

## 6) 성능 고려

- 얕은 복사는 빠르지만, **공유 참조로 인한 사이드이펙트 비용**이 발생할 수 있습니다.
- 깊은 복사는 안전하지만, **할당/GC/순회 비용**이 큽니다.
- **불변 + 부분 깊은 복사 + 방어적 복사**의 조합이 실무에서 **성능·안전성 균형**에 유리합니다.

---

## 7) 체크리스트

- [ ] 가변 참조 필드가 있는가? → **깊은 복사/불변화/방어적 복사**를 고려했는가
- [ ] 컬렉션을 복사할 때 **요소도 복사**했는가(필요 시)
- [ ] 외부 API에 내부 참조를 **그대로 노출**하지 않았는가
- [ ] 복사 전략을 **문서화/테스트**했는가(얕은/깊은 복사 구분)
- [ ] 순환 참조/대규모 그래프에서 **성능/정합성**을 검증했는가

---

## 8) 요약

- **얕은 복사**: 최상위만 새 객체, 내부 참조는 공유 → **원본·사본 상호 영향** 가능.
- **깊은 복사**: 내부 참조까지 새로 할당 → **독립성 보장**, 비용↑.
- 실무에서는 **불변 설계**와 **부분 깊은 복사**, **방어적 복사**를 조합해 **안전성과 성능**을 함께 확보하는 것이 바람직합니다.
