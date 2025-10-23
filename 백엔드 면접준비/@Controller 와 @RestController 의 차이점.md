# ⚙️ @Controller 와 @RestController 의 차이점

이 두 어노테이션의 주요 차이점은 **HTTP 응답의 처리 방식**에 있습니다.

---

## 🧩 @Controller

주로 **뷰(View)**를 반환하는 컨트롤러를 정의할 때 사용됩니다.  
메서드가 반환하는 값은 **뷰 리졸버(View Resolver)**에 의해 해석되어  
**JSP**, **Thymeleaf** 등과 같은 템플릿 엔진을 통해 **HTML을 생성**합니다.

---

## 🌐 @RestController

주로 **RESTful 웹 서비스 API**를 정의할 때 사용됩니다.  
메서드가 반환하는 값은 자동으로 **JSON** 또는 **XML** 형식으로 변환되어  
**HTTP 응답 본문(Response Body)**에 포함됩니다.  
이는 **@Controller와 @ResponseBody의 결합된 형태**입니다.
