## ✅ String 객체는 가변일까요, 불변일까요?

String 객체는 **불변 (Immutable)** 입니다.

- String 클래스는 내부적으로 `final` 키워드가 선언된 `byte[]` 또는 `char[]` 필드를 사용해서 문자열을 저장합니다.
- `String`은 참조 타입이므로, `concat()`, `replace()`, `toUpperCase()` 같은 메서드를 호출하면 새로운 String 객체를 반환하고 기존 객체는 변하지 않습니다.

## 📌 String을 불변으로 설계한 이유

1. **String Constant Pool 사용 가능**

   - 동일한 문자열은 동일한 객체를 재사용하여 메모리 사용을 최적화합니다.

2. **Thread-safe 보장**

   - 불변 객체는 멀티스레드 환경에서도 안전하게 공유할 수 있습니다.

3. **HashCode 캐싱**

   - 불변이기 때문에 해시코드를 한 번만 계산하고 재사용할 수 있어 성능이 향상됩니다.

4. **보안 강화**
   - 비밀번호, 토큰, URL 같은 민감한 데이터가 변경 불가능하여 보안성이 높습니다.

---

### 📌 리터럴 vs 생성자 차이 예시

```java
String first = "hello"; // 리터럴
String second = new String("hello"); // 생성자
String third = "hello"; // 동일 리터럴

System.out.println(System.identityHashCode(first));   // 예: 498931366
System.out.println(System.identityHashCode(second));  // 예: 2060468723
System.out.println(System.identityHashCode(third));   // 예: 498931366
```

- first와 third는 같은 리터럴이므로 String Constant Pool에서 동일한 객체를 참조합니다.

- second는 new 키워드로 생성되었기 때문에 Heap 메모리에 새로운 객체가 생성됩니다.

```java
String first = "hello";
String second = new String("hello");
String third = second.intern(); // String Constant Pool에 등록

System.out.println(System.identityHashCode(first));   // 예: 498931366
System.out.println(System.identityHashCode(second));  // 예: 2060468723
System.out.println(System.identityHashCode(third));   // 예: 498931366
```

- intern() 메서드를 사용하면 Heap에 있는 문자열을 String Constant Pool로 등록하고 해당 참조를 반환합니다.

- 따라서 first와 third는 같은 객체를 참조하게 됩니다.
