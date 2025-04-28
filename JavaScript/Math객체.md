# 완전제곱수 판별 함수 (자바스크립트)

## ✅ 함수 코드

```javascript
function solution(n) {
  const x = Math.sqrt(n);
  return Number.isInteger(x) ? (x + 1) ** 2 : -1;
}
```

## 📌 사용 예시

```javascript
solution(121); // 144
solution(3);   // -1
```

## 🧠 관련 개념

### Math 객체

| 함수 | 설명 |
|------|------|
| `Math.sqrt(x)` | 제곱근 |
| `Math.pow(x, y)` | 거듭제곱 |
| `Math.floor(x)` | 내림 |
| `Math.ceil(x)` | 올림 |
| `Math.round(x)` | 반올림 |

### 제곱근이란?

- `Math.sqrt(9)` → `3` (3 × 3 = 9)  
- `Math.sqrt(2)` → `1.4142...` (정수가 아님)

---

## 🔁 반복문을 이용한 버전

```javascript
function solution(n) {
  for (let i = 1; i * i <= n; i++) {
    if (i * i === n) return (i + 1) ** 2;
  }
  return -1;
}
```

