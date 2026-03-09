# Spring MessageSource `setFallbackToSystemLocale(false)` 정리

## 핵심 요약

- Spring의 `ReloadableResourceBundleMessageSource`는 메시지 파일이
  없을 때 **시스템 Locale을 기준으로 fallback** 할 수 있습니다.
- 이 경우 예상과 다르게 **서버 기본 언어(messages_ko.properties 등)가
  먼저 사용**될 수 있습니다.
- `setFallbackToSystemLocale(false)`를 설정하면 **시스템 Locale
  fallback을 막고 `messages.properties`를 기본 fallback으로 사용**하게
  됩니다.

---

# 개요

Spring에서 다국어(i18n)를 처리할 때 `MessageSource`를 통해 언어별 메시지
파일을 읽어옵니다.

일반적인 메시지 파일 구조:

messages.properties messages_en.properties messages_ko.properties
messages_ja.properties

요청 Locale이 `en`이면 `messages_en.properties`,\
`ko`이면 `messages_ko.properties`를 사용합니다.

문제는 **요청한 언어 파일이 존재하지 않을 때** 발생합니다.

---

# 메시지 파일 fallback 동작

예를 들어 `pl` 요청이 들어왔다고 가정합니다.

/pl/promotion/welcome-bonus

하지만 프로젝트에는 다음 파일만 존재합니다.

messages.properties\
messages_en.properties\
messages_ko.properties

대부분 다음처럼 동작할 것이라고 생각합니다.

messages_pl.properties 없음 → messages.properties 사용

하지만 실제 Spring 기본 동작은 다를 수 있습니다.

---

# 시스템 Locale fallback

Spring은 기본적으로 **시스템 Locale도 fallback 후보**로 사용합니다.

예를 들어 서버 기본 Locale이

ko_KR

이라면 메시지 탐색 순서는 다음처럼 진행될 수 있습니다.

messages_pl.properties 없음\
→ messages_ko.properties\
→ messages.properties

따라서 요청 Locale이 `pl`이어도 **한국어 메시지가 출력되는 상황**이
발생할 수 있습니다.

예:

pl message = 회원가입\
en message = SignUp\
ko message = 회원가입

이 경우 `pl` 메시지가 `messages.properties`가 아니라\
**messages_ko.properties에서 반환된 것**입니다.

---

# 문제점

이 동작은 다음과 같은 문제를 만들 수 있습니다.

- 서버 환경에 따라 **메시지 결과가 달라질 수 있음**
- 개발 환경과 운영 환경의 **Locale 차이로 번역 결과가 달라질 수 있음**
- 지원하지 않는 언어 요청 시 **예상하지 못한 언어가 노출될 수 있음**

예:

messages.properties = 영어\
messages_ko.properties = 한국어

pl 요청 → 한국어 출력

---

# 해결 방법

시스템 Locale fallback을 비활성화합니다.

```java
@Bean
public MessageSource messageSource() {
    ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
    messageSource.setBasename("classpath:/i18n/messages");
    messageSource.setDefaultEncoding("UTF-8");

    // 시스템 Locale fallback 비활성화
    messageSource.setFallbackToSystemLocale(false);

    return messageSource;
}
```

이 설정을 적용하면 fallback 흐름은 다음과 같이 변경됩니다.

messages_pl.properties 없음\
→ messages.properties

즉 **시스템 Locale을 거치지 않고 바로 기본 메시지 파일을 사용**합니다.

---

# 적용 후 동작

예:

messages.properties signup=SignUp

messages_ko.properties signup=회원가입

결과

요청 Locale 결과

---

en SignUp
ko 회원가입
pl SignUp

지원하지 않는 언어는 항상 **messages.properties**로 fallback 됩니다.

---

# 장점

- 서버 Locale에 영향을 받지 않음
- 다국어 fallback 동작이 **일관되게 유지**
- 기본 메시지 파일 기준으로 **예측 가능한 결과 제공**

---

# 주의사항

- `messages.properties`는 보통 **기준 언어(대부분 영어)** 로 작성하는
  것이 좋습니다.
- 기본 파일을 한국어로 두면 **지원하지 않는 언어 요청이 모두 한국어로
  출력**될 수 있습니다.
- 다국어 사이트에서는 **fallback 기준 언어를 명확히 정의**하는 것이
  중요합니다.

---

# 한 줄 정리

setFallbackToSystemLocale(false)는\
메시지 파일이 없을 때 **서버 기본 Locale fallback을 막고** 항상
**messages.properties를 기준 fallback으로 사용하도록 만드는
설정**입니다.
