# Java에서 Object를 String으로 변환: `(String) value` vs `String.valueOf(value)`

## 핵심 요약

- **타입 캐스팅**(`(String) value`)은 `value`가 **실제로 String일 때만** 안전하며, 아니면 **`ClassCastException`**이 발생합니다. `null`이면 `null`을 그대로 반환해 이후 사용 시 **`NullPointerException`** 위험이 있습니다.
- **`String.valueOf(value)`**는 `null`이면 **"null" 문자열**을 반환하고, 그 외에는 내부적으로 **`value.toString()`**을 호출해 문자열로 변환하므로 **예외를 줄이는 편**입니다.

## 개요

두 방식 모두 결과적으로 `String`을 얻기 위한 방법이지만, **동작 방식**과 **예외/널 처리**에서 중요한 차이가 있습니다.

## 동작 방식과 예외 처리 차이

### 1) 타입 캐스팅: `(String) value`

- `value`가 **String 타입이 아닌 경우** → **`ClassCastException` 발생**
- `value`가 **null인 경우** → `null`을 그대로 반환  
  이후 `String` 메서드를 호출하면 **`NullPointerException`**이 발생할 수 있습니다.
- **타입 안정성이 부족**하므로, **캐스팅 대상 타입이 확실할 때만** 사용하는 것이 좋습니다.

```java
Object intValue = 10;
String str1 = (String) intValue; // ClassCastException

Object nullValue = null;
String str2 = (String) nullValue; // null
str2.concat("maeilmail"); // NullPointerException
```

### 2) `String.valueOf(value)`

- `value`가 **String 타입이 아닌 경우** → 내부적으로 **`value.toString()`**을 호출해 문자열로 변환
- `value`가 **null인 경우** → **"null" 문자열**을 반환

```java
Object intValue = 10;
String str1 = String.valueOf(intValue); // "10"

Object nullValue = null;
String str2 = String.valueOf(nullValue); // "null"
str2.concat("maeilmail"); // "nullmaeilmail"
```

## 장점

### 타입 캐스팅의 장점

- `value`가 **이미 String임이 보장**될 때는 **명확하고 직접적**입니다.

### `String.valueOf()`의 장점

- 타입 캐스팅처럼 런타임 `ClassCastException`을 유발하지 않으며,
- `null` 처리까지 포함해 **상대적으로 안전**하게 문자열을 얻을 수 있습니다.

## 단점

### 타입 캐스팅의 단점

- 런타임에 **`ClassCastException`**이 발생할 수 있습니다.
- `null`이면 그대로 `null`이 되어, 이후 로직에서 **`NullPointerException`** 위험이 있습니다.

### `String.valueOf()`의 단점

- `null`이 **"null" 문자열**로 바뀌므로, 상황에 따라 **의도치 않은 데이터**를 만들 수 있습니다.

## 타입 캐스팅 시 `ClassCastException`을 방지하는 방법

캐스팅 전에 **타입이 맞는지 확인**하면 예외를 방지할 수 있습니다. `instanceof`를 사용하면 안전하게 처리할 수 있습니다.

```java
Object intValue = 10;

if (intValue instanceof String str) {
    System.out.println(str);
} else {
    // ...
}
```

## `String.valueOf(null)`이 "null"을 반환하는 것이 문제가 될 수도 있나요?

- **"null" 문자열**과 **`null` 자체**는 다른 의미를 가질 수 있어 문제가 될 수 있습니다.
- 특히 **JSON 변환**이나 **DB 저장** 시, `null`이 `"null"`로 저장되면 오류/혼선을 유발할 가능성이 있습니다.

### 실무 팁

- 원치 않는 `"null"` 문자열을 방지하려면:
  - **사전에 null 여부를 검증**해서 분기 처리하거나
  - `Objects.toString()`을 사용해 **null일 때 대체 문자열**을 지정하는 방법을 고려할 수 있습니다.

```java
import java.util.Objects;

Object value = null;

// null이면 기본값을 사용
String s1 = Objects.toString(value, "");        // ""
String s2 = Objects.toString(value, "(none)");  // "(none)"
```
