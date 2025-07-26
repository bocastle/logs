# async-mutex를 사용한 간단한 뮤텍스(Mutex) 설명

## 뮤텍스(Mutex)란?

뮤텍스(Mutual Exclusion, 상호 배제)는 여러 작업이 동시에 공유 자원에 접근하지 못하도록 잠금(lock) 기능을 제공하는 동기화 메커니즘입니다.  
즉, 한 번에 오직 하나의 작업만이 특정 코드 영역(크리티컬 섹션)에 진입할 수 있도록 보장하여, 경쟁 상태(Race Condition)를 방지합니다.

---

## async-mutex 라이브러리

`async-mutex`는 Node.js와 같은 비동기 환경에서 뮤텍스 기능을 쉽게 구현할 수 있도록 도와주는 라이브러리입니다.  
특히 Promise 기반 비동기 코드에서 동기화가 필요한 상황에 적합합니다.

---

## 동작 원리

- 작업이 뮤텍스를 요청(acquire)하면, 이미 다른 작업이 뮤텍스를 점유 중이라면 대기한다.
- 점유 중인 작업이 뮤텍스를 해제(release)하면, 대기 중인 다음 작업이 뮤텍스를 점유한다.
- 뮤텍스가 점유된 동안에는 다른 작업이 동시에 해당 코드 영역에 진입할 수 없다.

---

## 예시 코드 설명

```javascript
const { Mutex } = require("async-mutex");

const mutex = new Mutex(); // 뮤텍스 객체 생성
let sharedCounter = 0;

async function incrementCounter() {
  const release = await mutex.acquire(); // 뮤텍스 잠금 요청, 점유 성공 시 release 함수 반환
  try {
    // 크리티컬 섹션: 공유 변수 접근
    let current = sharedCounter;
    await new Promise((resolve) => setTimeout(resolve, 100)); // 비동기 작업 시뮬레이션
    sharedCounter = current + 1;
    console.log("Counter:", sharedCounter);
  } finally {
    release(); // 뮤텍스 해제하여 다른 작업이 접근 가능하게 함
  }
}

// 여러 비동기 작업에서 안전하게 공유 변수 증가
incrementCounter();
incrementCounter();
incrementCounter();
```

- `mutex.acquire()`로 뮤텍스를 요청하면, 다른 작업이 잠금 중이면 대기함.
- 잠금을 획득하면 `release` 함수를 받아, 작업 종료 후 반드시 호출해서 뮤텍스를 해제해야 함.
- 이렇게 하면 동시에 여러 `incrementCounter`가 실행되어도 `sharedCounter` 변수 값이 꼬이지 않고 순차적으로 안전하게 증가함.

---

### 요약

- `async-mutex`는 비동기 환경에서 뮤텍스를 쉽게 구현하도록 돕는 라이브러리.
- 뮤텍스는 여러 작업의 공유 자원 접근을 직렬화하여 경쟁 상태를 방지함.
- `mutex.acquire()`로 잠금을 요청하고, 반환된 `release()`를 호출해 잠금을 해제함.
- Node.js 같은 비동기 환경에서 데이터 무결성을 유지하는 데 유용함.

### 예시

```javascript
import { Mutex } from "async-mutex";

const mutex = new Mutex();
let sharedCounter = 0;

async function incrementCounter() {
  // 뮤텍스 잠금 요청 (대기 가능)
  const release = await mutex.acquire();

  try {
    // 공유 자원 안전하게 접근
    const current = sharedCounter;
    // 인위적 딜레이 (예: 비동기 작업)
    await new Promise((resolve) => setTimeout(resolve, 100));
    sharedCounter = current + 1;
    console.log(`Counter incremented to: ${sharedCounter}`);
  } finally {
    // 작업 종료 후 반드시 잠금 해제
    release();
  }
}

// 동시에 여러 incrementCounter 호출해도 안전함
async function run() {
  await Promise.all([
    incrementCounter(),
    incrementCounter(),
    incrementCounter(),
    incrementCounter(),
    incrementCounter(),
  ]);
  console.log(`Final counter value: ${sharedCounter}`);
}

run();
```

이 코드는 동시에 여러 비동기 함수가 sharedCounter를 안전하게 1씩 증가시키는 예제입니다.

mutex.acquire() 호출 시, 이미 잠금이 걸려 있으면 대기했다가 잠금 획득 후 작업 진행하며, 작업이 끝나면 release()를 호출해 잠금을 해제합니다.

이 덕분에 동시 접근으로 인한 값 꼬임을 방지할 수 있어요!
