# 템플릿 메서드 패턴(Template Method Pattern) 정리

## 핵심 요약

- 템플릿 메서드 패턴은 **기능의 뼈대(절차)** 와 **구현(세부 단계)** 을 분리하는 **행위(Behavioral) 디자인 패턴**입니다.
- **상위 클래스**가 실행 단계의 **절차(알고리즘 흐름)** 를 결정하고, **하위 클래스**가 각 실행 단계를 **구현**합니다.
- 공통 로직을 상위 클래스에 모아 **중복을 줄이고 재사용성을 높일 수** 있지만, 상위 클래스 변경 시 **하위 클래스 전반에 영향**이 갈 수 있습니다.

## 템플릿 메서드 패턴이란?

템플릿 메서드 패턴(Template Method Pattern)은 기능의 뼈대와 구현을 분리하는 행위 디자인 패턴입니다.  
템플릿 메서드 패턴은 실행 단계의 절차를 결정하는 상위 클래스와 실행 단계를 구현하는 하위 클래스로 구성됩니다.

## 예시 코드

아래 예시에서 `doDailyRoutine()`이 **템플릿 메서드**이며, 상위 클래스가 “하루 루틴의 절차”를 정하고 하위 클래스가 각 단계의 구체 동작을 구현합니다.

```java
public abstract class Student {

    public abstract void study();
    public abstract void watchYoutube();
    public abstract void sleep();

    // 템플릿 메서드
    final public void doDailyRoutine() {
       study();
       watchYoutube();
       sleep();
    }
}
class BackendStuduent extends Student {

    @Override
    public void study() {
        System.out.println("영한님 JPA 강의를 수강합니다.");
    }

    @Override
    public void watchYoutube() {
        System.out.println("개발바닥 유튜브를 시청합니다.");
    }

    @Override
    public void sleep() {
        System.out.println("7시간 잠을 잡니다.");
    }
}
```

## 장점

- 공통 로직을 상위 클래스에 모아 **중복 코드를 줄일 수** 있습니다.
- 코드의 **재사용성**을 높일 수 있습니다.

## 단점

- 하위 클래스를 개발할 때, 상위 클래스의 내용을 알기 전까지 **어떤 방식으로 동작할지 예측하기 어려울 수** 있습니다.
- 상위 클래스 수정이 발생하는 경우 **모든 하위 클래스를 변경해야 할 수** 있습니다.

## 주의사항 / 실무 팁

- 템플릿 메서드(`final`로 고정되는 절차)와 “변경 가능한 단계(추상 메서드/훅 메서드)”의 경계를 명확히 두면 유지보수에 도움이 됩니다.
- 상위 클래스 변경이 하위 클래스 전체에 영향을 주기 쉬우므로, 상위 클래스의 공개 API와 호출 순서를 변경할 때는 **하위 클래스 영향 범위를 먼저 점검**하는 편이 안전합니다.
