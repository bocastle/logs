## 🧮 왜 `reduce()`를 사용하는가?

---

### 📌 핵심 개념

`reduce()`는 배열을 단일 값으로 축약하는 **고차 함수**로, 내부적으로 반복 처리를 최적화하고 깔끔한 코드 구성을 돕습니다.

---

### ✅ 왜 `reduce()`를 사용하면 좋은가?

#### 1. **명확한 목적을 가진 순회**

- `reduce()`는 배열의 각 요소를 하나씩 순회하면서 **누적 결과값**을 만들기 때문에,
- **누적하는 작업**에는 가장 자연스럽고 직관적인 도구입니다.

```javascript
photo.reduce((sum, person) => sum + (scoreMap[person] || 0), 0);
```

- sum: 누적값
- scoreMap[person]: 현재 사람의 점수
- || 0: 해당 인물이 없을 경우 0점 처리

---

#### 2. forEach보다 선언적이고 함수형 스타일

- for 혹은 forEach는 순회하면서 수동으로 상태를 관리해야 하지만,
- reduce()는 하나의 반환값으로 축약하므로 코드의 목적이 분명합니다.

```javascript
// imperative (명령형)
let sum = 0;
for (let i = 0; i < photo.length; i++) {
  sum += scoreMap[photo[i]] || 0;
}

// declarative (선언형)
photo.reduce((acc, cur) => acc + (scoreMap[cur] || 0), 0);
```

#### 3. 가독성과 재사용성 향상

- 복잡한 누산 로직이 생기더라도, reduce()는 함수로 추출해 재사용하기 좋습니다.

- 코드의 의도를 더 명확하게 표현합니다. (예: "이건 축약이다")

---

## 🧠 그럼에도 주의할 점

- reduce()는 로직이 길어지면 가독성이 떨어질 수 있으니, 단순한 누산용으로 사용하는 게 가장 적절합니다.

- 로직이 복잡해질 경우는 함수 분리와 주석을 활용하세요.

## 📚 결론

- reduce()는 단순한 반복을 넘어서 의미 있는 집계 작업에 적합

- 코드를 선언적으로 표현하여 가독성과 유지보수성 향상

- 배열을 한 번만 순회하므로 성능 측면에서도 효율적

즉, reduce()는 단순히 짧아서 쓰는 게 아니라, 그 동작이 문제에 딱 맞기 때문에 사용하는 함수입니다.
