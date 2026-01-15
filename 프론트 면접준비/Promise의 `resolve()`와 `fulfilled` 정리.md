# Promise의 `resolve()`와 `fulfilled` 정리

> 요약: **`resolve()`**는 *완료를 통보하는 함수(행위)*이고, **`fulfilled`**는 _완료된 상태(state)_ 입니다. `resolve(value)`가 호출되면 해당 Promise는 **이행(fulfilled)** 으로 전이되며, 핸들러 체인(`then`)이 **마이크로태스크 큐**에서 실행됩니다.

---

## 1) 용어 구분

- **`resolve(value)`**: executor 또는 외부에서 호출되는 **함수**. 성공 결과 `value`를 전달하며, Promise를 **이행(fulfilled)** 상태로 전환하거나 **다른 thenable/Promise의 상태를 채택**합니다.
- **`fulfilled`**: Promise의 **상태** 중 하나(다른 하나는 `rejected`). 한 번 정해지면 **불변(immutable)** 입니다.
- (참고) **resolved(용어 혼동 주의)**: 비표준적·혼용되는 표현입니다. Promises/A+ 문맥에서 *resolved*는 *이미 다른 결과(또는 다른 Promise)를 채택한 상태*를 가리키기도 합니다. 일반 실무 문서에서는 대부분 **fulfilled**와 같은 뜻으로 느슨히 쓰이지만, **명확히는 다릅니다**.

---

## 2) 기본 동작 예시

```js
const p = new Promise((resolve, reject) => {
  // 비동기 작업 성공
  setTimeout(() => resolve("OK"), 0);
});

p.then((v) => console.log(v)); // 'OK'
```

- `resolve('OK')` 호출 → 상태가 **fulfilled** 로 전환 → 연결된 `then` 핸들러가 **마이크로태스크**로 스케줄링되어 실행됩니다.

---

## 3) `resolve()`의 중요한 규칙

1. **한 번 정해지면 끝(단일 결론)**  
   동일 Promise에서 `resolve()`/`reject()`를 **여러 번 호출**해도 **첫 호출만 유효**합니다.

   ```js
   new Promise((resolve, reject) => {
     resolve(1);
     resolve(2); // 무시됨
     reject(new Error()); // 무시됨
   });
   ```

2. **thenable/Promise 채택(Adoption)**  
   `resolve()`에 **다른 Promise/thenable**을 넘기면, **그 대상의 최종 상태**를 **채택**합니다.

   ```js
   const p1 = Promise.resolve(10);
   const p2 = new Promise((resolve) => resolve(p1)); // p1의 결과를 채택
   p2.then((v) => console.log(v)); // 10
   ```

3. **동기처럼 보여도 비동기(마이크로태스크)**  
   `then`/`catch` 핸들러는 **현재 콜스택이 끝난 뒤** 마이크로태스크로 실행됩니다.

   ```js
   Promise.resolve("A").then(console.log);
   console.log("B");
   // 출력 순서: B → A
   ```

4. **`resolve()`는 실패를 표현하지 않음**  
   실패는 **`reject(error)`** 로만 표현합니다. `resolve()`는 **이행(성공)** 을 의미합니다.

5. **`resolve(undefined)`와 `resolve()`**  
   둘 다 결과 값이 `undefined`인 **fulfilled** 를 만듭니다.

---

## 4) `fulfilled` 상태와 핸들러 체인

- `fulfilled`가 되면 연결된 `then(onFulfilled)` 이 **순서대로** 실행됩니다.
- 체인이 **값을 반환**하면 다음 `then`으로 **전달**되고, **예외/`throw`/거부된 Promise** 를 반환하면 다음 `catch`로 **전파**됩니다.

```js
Promise.resolve(1)
  .then((v) => v + 1) // 2
  .then((v) => {
    throw new Error("oops");
  })
  .catch((e) => "recovered") // 'recovered'
  .then((v) => console.log(v)); // 'recovered'
```

---

## 5) `resolve()`와 에러의 관계(자주 하는 질문)

- **Q. `resolve()`가 실패할 수 있나요?**  
  **아니요.** `resolve()`는 성공을 의미합니다. 실패는 **`reject()`** 로 표현합니다.
- **Q. 비동기 작업이 내부적으로 실패했는데 `resolve()`를 호출하면?**  
  그건 **논리 오류**입니다. 실패 분기는 `reject(error)` 또는 `throw`(async 함수에서는 `throw`가 자동으로 reject)로 표현해야 합니다.
- **Q. `then`과 `catch`의 역할?**  
  `then`은 **이행 결과 처리**, `catch`는 **거부(오류) 처리**, `finally`는 **정리 작업**을 담당합니다.

---

## 6) async/await와의 관계

- `async` 함수는 **암묵적으로 Promise를 반환**합니다.
- `return value` → **fulfilled**(value), `throw error` → **rejected**(error)  
  내부에서 `resolve`/`reject`를 직접 다루지 않아도 **동일한 의미**를 갖습니다.

```js
async function f() {
  const v = await Promise.resolve(1); // 이행을 기다림
  return v + 1; // fulfilled(2)
}
f().then(console.log);
```

---

## 7) 체크리스트

- [ ] 성공은 `resolve(value)`, 실패는 `reject(error)` 로 **명확히 분기**했는가
- [ ] 외부 Promise/thenable을 `resolve()`에 넘길 때 **채택(adopt)** 동작을 이해하고 있는가
- [ ] 핸들러 실행 타이밍(마이크로태스크)을 고려해 **레이스 컨디션**을 피했는가
- [ ] 첫 결론 이후 추가 `resolve/reject` 호출이 **무시**됨을 전제하고 코드를 작성했는가

---

## 8) 결론

- **`resolve()` = 이행을 *시작*하는 함수**, **`fulfilled` = 이행이 _완료된 상태_** 입니다.
- 실패는 **`reject()`/예외** 로 표현하고, 핸들러는 **마이크로태스크**에서 실행됩니다.
- thenable/Promise 채택 규칙과 단일 결론 원칙을 이해하면, **안정적이고 예측 가능한 비동기 코드**를 작성할 수 있습니다.
