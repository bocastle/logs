# @ExceptionHandler 어노테이션

`@ExceptionHandler` 애너테이션은 **Spring MVC**에서
컨트롤러(`@Controller`)나 전역 예외 처리를 위한 `@ControllerAdvice`
클래스의 메서드에서 발생하는 예외를 처리하는 데 사용됩니다.\
즉, 특정 예외를 처리할 메서드를 지정하여, 예외 상황에서도 사용자에게
안정적이고 일관된 응답을 제공할 수 있게 합니다.

---

## 동작 방식

1.  Spring MVC 애플리케이션에서 예외 발생 시, `DispatcherServlet`은
    적절한 **HandlerExceptionResolver**를 찾아 예외를 처리합니다.
2.  Spring에는 기본적으로 세 가지 `HandlerExceptionResolver`가 등록되어
    있으며, 우선순위에 따라 실행됩니다.
3.  이 중 **ExceptionHandlerExceptionResolver**가 가장 먼저 동작합니다.
    - 발생한 예외가 `@ExceptionHandler`에 등록되어 있으면 해당
      메서드가 실행됩니다.\
    - 처리할 수 없는 예외라면 다음 리졸버로 넘어갑니다.
4.  `ExceptionHandlerExceptionResolver`는 예외를 **WAS(Web Application
    Server)까지 전달하지 않고 직접 처리**합니다.

---

## 사용 예시

```java
@Controller
public class MyController {

    @ExceptionHandler(NullPointerException.class)
    public String handleNullPointerException(NullPointerException ex, Model model) {
        model.addAttribute("errorMessage", "데이터가 존재하지 않습니다.");
        return "errorPage";
    }
}
```

또는 전역 처리 방식으로 `@ControllerAdvice`를 활용할 수 있습니다.

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleAllExceptions(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                             .body("서버 오류가 발생했습니다: " + ex.getMessage());
    }
}
```

---

## 장점

- 예외 발생 시 **사용자 친화적인 메시지**를 반환 가능\
- **로깅, 모니터링, 알림 처리** 등 부가 작업 수행 가능\
- 컨트롤러 코드에서 예외 처리 로직을 분리하여 **가독성과 유지보수성
  향상**

---

## 정리

- `@ExceptionHandler`는 특정 예외에 대한 맞춤형 처리 로직을 제공하는
  애너테이션\
- `DispatcherServlet` → `HandlerExceptionResolver` →
  `ExceptionHandlerExceptionResolver` 순으로 동작\
- WAS에 예외를 던지지 않고 Spring 내에서 처리 가능\
- **사용자 경험 향상과 시스템 안정성을 높이는 핵심 기능**
