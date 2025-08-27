# ⚡ Cluster

## 📌 개념

- Node.js는 싱글 스레드이므로 CPU 코어를 다 활용하지 못함
- Cluster 모듈은 **멀티 프로세스**를 띄워 모든 코어를 활용
- 주로 서버 애플리케이션에서 성능 확장(Scaling) 용도로 사용

---

## 🛠 사용 예시

```js
// cluster.js
const cluster = require("cluster");
const http = require("http");
const os = require("os");

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  console.log(`마스터 프로세스 ${process.pid} 실행`);

  // 워커 생성
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", (worker) => {
    console.log(`워커 ${worker.process.pid} 종료`);
    cluster.fork(); // 죽으면 새로 생성
  });
} else {
  // 워커 프로세스 (서버 역할)
  http
    .createServer((req, res) => {
      res.writeHead(200);
      res.end(`Hello from worker ${process.pid}`);
    })
    .listen(3000);

  console.log(`워커 프로세스 ${process.pid} 시작`);
}
```

---

## ✅ 특징

- 각 워커는 **별도의 프로세스** → 메모리 독립적
- IPC(프로세스 간 통신)로 데이터 주고받음
- CPU 코어 개수만큼 워커를 띄워 병렬 처리 가능
