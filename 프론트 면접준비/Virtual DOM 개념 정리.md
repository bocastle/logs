# 🧠 Virtual DOM 개념 정리 (React 중심)

## 📌 Virtual DOM이란?

Virtual DOM은 **실제 DOM(Document Object Model)**을 **JavaScript 객체 형태로 복제한 가벼운 메모리 내 구조**입니다.

- **목적:** 복잡한 실제 DOM 조작의 성능 비용 절감을 위해
- **도입 배경:** 직접 DOM을 조작하는 것은 느리고 비용이 큼 → 이를 최적화하려고 Virtual DOM을 사용

---

## ⚙️ 동작 원리

1. **상태 변경**  
   컴포넌트의 `state`나 `props`가 변경되면 새로운 Virtual DOM 트리가 생성됩니다.

2. **재조정 (Reconciliation)**  
   이전 Virtual DOM과 새로운 Virtual DOM을 비교하여 **차이점(Diff)**을 계산합니다.

3. **최소 업데이트 수행**  
   변경된 부분만 실제 DOM에 반영합니다.

- 이 모든 비교 작업은 **브라우저 DOM이 아닌 메모리 상**에서 빠르게 처리됩니다.

---

## ⚡ React의 Diffing 알고리즘 최적화

DOM 트리 비교는 이론적으로 O(n³)이지만, React는 **휴리스틱**을 통해 O(n)으로 최적화했습니다.

### 🔑 React의 휴리스틱 가정 두 가지

---

### 1. 서로 다른 타입은 완전히 다른 트리로 간주

- 예:  
  기존 → `<div class="before" title="stuff" />`  
  변경 → `<span class="after" title="stuff" />`

→ 이 경우, 전체 노드와 자식을 통째로 새로 만듭니다.

- **같은 타입의 경우**, 속성 차이만 업데이트

```javascript
<div className="before" title="stuff" />
→
<div className="after" title="stuff" />
```

### 2️⃣ key prop을 통한 자식 요소 추적

- **같은 수준의 자식 요소들을 비교할 때**, `key`를 사용해 어떤 요소가 **유지 / 이동 / 삭제**되었는지 판단합니다.
- `key`가 없으면 **순서 기반 비교**가 이루어져서, 리스트 전체를 불필요하게 리렌더링할 수 있습니다.

> 🔍 React는 `key`를 통해 요소의 identity를 추적함으로써 리스트 리렌더링 성능을 최적화합니다.

```javascript
// 예시: key가 있는 경우
<ul>
  {items.map((item) => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>
```

### ✅ 정리

| 항목           | 설명                                 |
| -------------- | ------------------------------------ |
| Virtual DOM    | 실제 DOM의 JS 객체 버전              |
| Reconciliation | 변경된 Virtual DOM과 이전 DOM의 비교 |
| 최적화 방식    | 휴리스틱 기반 비교 (O(n))            |
| 효율성 핵심    | 타입 기반 분기, key 기반 비교        |
