# 🧠 BigInt로 문자열 숫자 비교하기

## ✨ 개요

자바스크립트에서 **아주 큰 숫자**가 문자열로 주어졌을 때,  
이를 정확하게 비교하려면 `BigInt`를 사용하는 것이 안전합니다.

---

## 🚨 왜 `Number`가 아닌 `BigInt`?

- 자바스크립트의 `Number`는 IEEE-754 표준을 따르는 **64비트 부동소수점 숫자**
- 약 ±9007199254740991 (2^53 - 1) 이상의 정수는 정확하게 표현하지 못함
- `Number("999999999999999999") === Number("1000000000000000000")` ➜ `true` 😱

---

## ✅ 해결책: `BigInt`

- `BigInt`는 자바스크립트에서 **정수의 범위를 넘어 정확한 표현**이 가능하도록 도입된 타입
- 문자열을 그대로 `BigInt`로 변환하여 비교 가능

```javascript
const a = BigInt("999999999999999999");
const b = BigInt("1000000000000000000");

console.log(a < b); // true
```

## 📦 활용 예시

문자열 안에서 숫자 부분 비교하기

```javascript
function countLessOrEqual(t, p) {
  const target = BigInt(p);
  const len = p.length;
  let count = 0;

  for (let i = 0; i <= t.length - len; i++) {
    const part = BigInt(t.slice(i, i + len));
    if (part <= target) count++;
  }

  return count;
}
```

## 📝 요약

- 숫자가 정확하게 표현되어야 할 경우, 특히 긴 숫자 문자열 비교 시 BigInt를 사용
- 문자열을 그대로 BigInt()로 감싸면 안전하게 비교 가능
- 브라우저 지원은 현대 브라우저에서 대부분 제공됨 (ES2020 이후)
