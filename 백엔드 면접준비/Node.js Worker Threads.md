# 🧵 Worker Threads

## 📌 개념
- Node.js v10.5+ 부터 도입된 **멀티 스레드 지원 모듈**
- CPU 바운드 작업(대규모 계산, 이미지 처리, 암호화 등)에 유용
- 이벤트 루프와 별도의 스레드에서 실행됨

---

## 🛠 사용 예시

```js
// worker.js
const { parentPort } = require("worker_threads");

let sum = 0;
for (let i = 0; i < 1e9; i++) {
  sum += i;
}

parentPort.postMessage(sum);
```

```js
// main.js
const { Worker } = require("worker_threads");

const worker = new Worker("./worker.js");

worker.on("message", (result) => {
  console.log("결과:", result);
});

worker.on("error", (err) => {
  console.error("에러 발생:", err);
});
```

---

## ✅ 특징
- 데이터는 직렬화되어 메시지로 전달됨
- 공유 메모리(Buffer, SharedArrayBuffer)도 활용 가능
- Node.js를 멀티코어 CPU에서 활용할 수 있게 해줌