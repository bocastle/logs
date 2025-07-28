# 퍼사드 (Facade)

---

## 개념

- 복잡한 서브시스템들을 단순한 인터페이스로 감싸서 클라이언트가 쉽게 사용할 수 있도록 하는 패턴
- 여러 인터페이스를 하나의 통합된 인터페이스로 제공하여 사용 편의성을 높임

## 왜 사용하는가?

- 복잡한 내부 시스템을 감추고 단순한 API 제공
- 서브시스템 간 결합도를 낮춤
- 클라이언트와 서브시스템 간의 의존성 줄임

## 특징

- 퍼사드는 서브시스템 객체들을 포함하고 있음
- 클라이언트는 퍼사드 객체와만 상호작용함
- 퍼사드는 내부적으로 여러 서브시스템 호출을 조정함

## JavaScript 예제

```javascript
// 서브시스템 1
class CPU {
  freeze() {
    console.log("CPU freeze");
  }
  jump(position) {
    console.log(`CPU jump to position ${position}`);
  }
  execute() {
    console.log("CPU execute");
  }
}

// 서브시스템 2
class Memory {
  load(position, data) {
    console.log(`Memory load data ${data} at position ${position}`);
  }
}

// 서브시스템 3
class Disk {
  read(position, size) {
    console.log(`Disk read ${size} bytes from position ${position}`);
    return "data";
  }
}

// 퍼사드
class ComputerFacade {
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.disk = new Disk();
  }

  start() {
    this.cpu.freeze();
    const data = this.disk.read(0, 1024);
    this.memory.load(0, data);
    this.cpu.jump(0);
    this.cpu.execute();
  }
}

// 사용 예
const computer = new ComputerFacade();
computer.start();

// 출력:
// CPU freeze
// Disk read 1024 bytes from position 0
// Memory load data data at position 0
// CPU jump to position 0
// CPU execute
```
