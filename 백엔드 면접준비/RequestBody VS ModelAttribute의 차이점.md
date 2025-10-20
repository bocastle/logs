# 🧩 RequestBody VS ModelAttribute의 차이점

이들은 클라이언트 측에서 보낸 데이터를 Java 객체로 만들어주는데  
**RequestBody**는 요청의 본문(Body)에 있는 값을 바인딩할 때 사용하고,  
**ModelAttribute**는 요청 파라미터나 multipart/form-data 형식을 바인딩할 때 사용합니다.

---

## 📦 RequestBody

- 클라이언트가 보내는 요청의 **본문(Body)**을 자바 객체로 변환합니다.
- 내부적으로 **HttpMessageConverter**를 거치는데,  
  이때 **ObjectMapper**를 통해 JSON 값을 java 객체로 역직렬화합니다.
- 따라서 변환될 java 객체에 **기본 생성자**를 정의해야 하고,  
  **getter나 setter**를 선언해야 합니다.

### 💡 참고

**cf. record에 기본 생성자를 따로 정의하지 않았는데 역직렬화가 되는 이유**  
record 는 기본생성자를 자동으로 제공하지 않는 대신,  
‘모든 필드를 초기화하는 생성자’를 제공합니다.  
**jackson**은 일반 객체와 달리, record를 역직렬화할 때는  
‘모든 필드를 초기화하는 생성자’를 사용해 역직렬화하기 때문입니다.

---

## 🧭 ModelAttribute

두 가지 사용법이 있습니다.

1. **메서드 단에서의 사용법**  
   JSP의 **Model**에 하나 이상의 속성을 추가하고 싶을 때 사용합니다.  
   예시:
   ```java
   model.addAttribute("속성 이름", "속성 값");
   ```
2. **인자 단에서의 사용법**
   클라이언트가 보내는 **요청의 파라미터나 multipart/form-data 형식의 데이터**를  
   자바 객체로 변환합니다.  
   내부적으로 **ModelAttributeMethodProcessor**를 거치는데,  
   이때 지정된 클래스의 생성자를 찾아 객체로 변환합니다.
