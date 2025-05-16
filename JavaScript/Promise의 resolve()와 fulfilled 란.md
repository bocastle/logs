# Promise의 resolve()와 fulfilled 이해하기

프론트엔드 개발에서 Promise는 비동기 작업을 다룰 때 자주 사용됩니다. 이 글에서는 Promise의 `resolve()`와 `fulfilled` 개념을 명확히 짚어보겠습니다.

## 🌟 resolve()란?

`resolve()`는 Promise를 성공적으로 완료시키는 함수입니다. 비동기 작업이 성공적으로 끝나면, `resolve()`를 호출해 결과를 전달합니다. 이로 인해 Promise의 상태는 **fulfilled(이행됨)**로 전환됩니다.

### 예시 코드

```javascript
new Promise((resolve, reject) => {
  resolve("작업 완료!");
});
```

위 코드에서 `resolve("작업 완료!")`를 호출하면 Promise는 `fulfilled` 상태가 되며, 전달된 값(`"작업 완료!"`)을 후속 처리에서 사용할 수 있습니다.

## 🌟 fulfilled란?

`fulfilled`는 Promise가 성공적으로 완료된 상태를 의미합니다. 즉, `resolve()`가 호출된 이후의 결과 상태입니다.

- `resolve()`: Promise를 성공 상태로 만드는 **행위**
- `fulfilled`: 그 행위로 인해 도달한 **상태**

## ❓ resolve()에서 실패를 처리할 수 있나요?

아닙니다. `resolve()`는 오직 성공적인 작업의 완료를 나타낼 때 사용됩니다.

- 작업이 성공하면: `resolve()` 호출 → Promise는 `fulfilled` 상태
- 작업이 실패하면: `reject()` 호출 → Promise는 `rejected` 상태

실패 처리에는 `reject()`를 사용하며, 이는 `catch()`를 통해 처리됩니다.

## 🌟 요약

- **`resolve()`**: 비동기 작업의 성공 결과를 전달하며 Promise를 `fulfilled` 상태로 전환
- **`fulfilled`**: Promise가 성공적으로 완료된 상태
- **`reject()`**: 실패를 알리며 Promise를 `rejected` 상태로 전환
- **`then()`**: `resolve()`로 전달된 값을 처리
- **`catch()`**: `reject()`로 전달된 오류를 처리

이해를 돕기 위해, `resolve()`는 "약속을 지켰다"고 알리는 행동이고, `fulfilled`는 그 약속이 지켜진 상태라고 생각하면 쉽습니다!
