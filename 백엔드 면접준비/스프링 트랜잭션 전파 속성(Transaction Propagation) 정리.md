# 스프링 트랜잭션 전파 속성(Transaction Propagation) 정리

## 핵심 요약

- 트랜잭션 전파는 **호출 시점에 기존 트랜잭션이 있을 때/없을 때** 해당 메서드가 **어떻게 트랜잭션 경계를 형성할지**를 결정합니다.
- `@Transactional(propagation = ...)` 값으로 지정하며, 대표적으로 **기존 트랜잭션 재사용**, **새 트랜잭션 생성**, **트랜잭션 없이 실행**, **예외 발생**, **중첩 트랜잭션(SAVEPOINT)** 등의 동작을 선택합니다.

## 트랜잭션 전파(Transaction Propagation) 개요

스프링에서 트랜잭션 전파(Transaction Propagation)는 트랜잭션의 경계에서 이미 진행 중인 트랜잭션이 있을 때 또는 없을 때 어떻게 동작할 것인가를 결정하는 기능입니다. 가령, `@Transactional` 어노테이션이 존재하는 메서드를 호출했을 때, 기존에 트랜잭션이 존재하면 재사용할지, 예외를 던질지 등 행동을 결정할 수 있습니다.

트랜잭션 전파 속성에는 `REQUIRED`, `REQUIRES_NEW`, `MANDATORY`, `SUPPORTS`, `NOT_SUPPORTED`, `NESTED`, `NEVER`가 존재하며, `@Transactional` 어노테이션의 `propagation` 속성에 값을 설정할 수 있습니다.

## 전파 속성별 동작

### REQUIRED

- 트랜잭션이 존재하는 경우 **해당 트랜잭션을 사용**하고, 트랜잭션이 없는 경우 **트랜잭션을 생성**합니다.

### REQUIRES_NEW

- 트랜잭션이 존재하는 경우 **기존 트랜잭션을 잠시 보류(suspend)** 시키고, **신규 트랜잭션을 생성**하여 사용합니다.

### MANDATORY

- 트랜잭션이 **반드시 있어야** 합니다.
- 트랜잭션이 없다면 **예외가 발생**합니다.
- 트랜잭션이 존재한다면 **해당 트랜잭션을 사용**합니다.

### SUPPORTS

- 트랜잭션이 존재하는 경우 **트랜잭션을 사용**하고,
- 트랜잭션이 없다면 **트랜잭션 없이 실행**합니다.

### NOT_SUPPORTED

- 트랜잭션이 존재하는 경우 **트랜잭션을 잠시 보류(suspend)** 하고,
- **트랜잭션이 없는 상태로 처리**합니다.

### NESTED

- 트랜잭션이 있다면 **SAVEPOINT를 남기고 중첩 트랜잭션을 시작**합니다.
- 트랜잭션이 없는 경우에는 **새로운 트랜잭션을 시작**합니다.

### NEVER

- 트랜잭션이 존재하는 경우 **예외를 발생**시키고,
- 트랜잭션이 없다면 **생성하지 않습니다**.

## 장점

- 상황에 따라 트랜잭션 경계를 유연하게 설계할 수 있습니다(재사용/분리/비트랜잭션/강제/중첩).
- 모듈화된 서비스 호출 구조에서도 일관된 트랜잭션 정책을 적용할 수 있습니다.

## 단점

- 전파 속성 조합에 따라 실제 커밋/롤백 단위가 직관과 달라질 수 있어(특히 `REQUIRES_NEW`, `NESTED`) 운영/장애 분석 난이도가 올라갈 수 있습니다.
- 트랜잭션을 보류(suspend)하거나 중첩(SAVEPOINT)을 쓰는 경우, 데이터소스/드라이버/환경에 따라 성능 및 지원 여부에 영향을 받을 수 있습니다.

## 주의사항 및 실무 팁

- `REQUIRES_NEW`, `NOT_SUPPORTED`는 기존 트랜잭션을 **보류(suspend)** 하므로, 호출 경로가 길어질수록 트랜잭션 흐름이 복잡해질 수 있습니다.
- `NESTED`는 **SAVEPOINT 기반**이므로, 실제로 SAVEPOINT를 지원하는지(사용하는 트랜잭션 매니저/DB/드라이버/설정) 확인이 필요합니다.
- 예외 정책(체크/언체크 예외)과 롤백 규칙(`rollbackFor`, `noRollbackFor`)을 함께 고려하지 않으면 기대와 다른 커밋/롤백이 발생할 수 있습니다.
- 트랜잭션 전파는 **프록시 기반 AOP**로 동작하므로, 같은 클래스 내부 호출(self-invocation)에서는 `@Transactional`이 적용되지 않는 전형적인 함정을 주의합니다.

## 코드 예시

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Service
public class SampleService {

    @Transactional(propagation = Propagation.REQUIRED)
    public void required() {
        // 기존 트랜잭션 참여, 없으면 생성
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void requiresNew() {
        // 기존 트랜잭션 보류 후 새 트랜잭션 생성
    }

    @Transactional(propagation = Propagation.MANDATORY)
    public void mandatory() {
        // 트랜잭션 없으면 예외
    }

    @Transactional(propagation = Propagation.SUPPORTS)
    public void supports() {
        // 있으면 참여, 없으면 비트랜잭션
    }

    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void notSupported() {
        // 기존 트랜잭션 보류, 비트랜잭션으로 실행
    }

    @Transactional(propagation = Propagation.NESTED)
    public void nested() {
        // 트랜잭션 있으면 SAVEPOINT 기반 중첩
    }

    @Transactional(propagation = Propagation.NEVER)
    public void never() {
        // 트랜잭션 있으면 예외, 없으면 비트랜잭션
    }
}
```
