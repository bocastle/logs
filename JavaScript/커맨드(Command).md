# 커맨드 (Command)

---

## 개념

- 요청을 객체의 형태로 캡슐화하여 서로 다른 요청, 큐잉, 로깅, 취소 등을 지원하는 디자인 패턴
- 실행할 작업을 객체로 만들어 요청자(invoker)와 수행자(receiver)를 분리

## 왜 사용하는가?

- 요청을 큐에 저장하거나 로그로 남길 수 있음
- 실행 취소(undo) 기능 구현이 용이
- 클라이언트와 실행자 간 결합도 감소

## 특징

- 커맨드 객체는 실행할 작업에 대한 정보를 가지고 있음
- 요청자(invoker)는 커맨드 객체를 실행만 함
- 수행자(receiver)는 실제 작업 수행

## JavaScript 예제

```javascript
// 수행자
class Light {
  on() {
    console.log("Light is ON");
  }
  off() {
    console.log("Light is OFF");
  }
}

// 커맨드 인터페이스 역할
class Command {
  execute() {}
  undo() {}
}

// 켜기 커맨드
class LightOnCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  execute() {
    this.light.on();
  }
  undo() {
    this.light.off();
  }
}

// 끄기 커맨드
class LightOffCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  execute() {
    this.light.off();
  }
  undo() {
    this.light.on();
  }
}

// 요청자
class RemoteControl {
  setCommand(command) {
    this.command = command;
  }
  pressButton() {
    this.command.execute();
  }
  pressUndo() {
    this.command.undo();
  }
}

// 사용 예
const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();

remote.setCommand(lightOn);
remote.pressButton(); // Light is ON
remote.pressUndo(); // Light is OFF

remote.setCommand(lightOff);
remote.pressButton(); // Light is OFF
remote.pressUndo(); // Light is ON
```
