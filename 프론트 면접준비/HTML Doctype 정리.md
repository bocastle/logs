# HTML Doctype 정리

## 1) Doctype의 정의와 위치

- **Doctype(\<!DOCTYPE ...\>)**은 브라우저에게 _이 문서가 어떤 표준(스펙)을 기준으로 작성되었는지_ 알려주는 선언문입니다.
- 문서 **최상단(첫 줄)** 에 위치해야 하며, 공백·주석보다도 앞에 위치하는 것이 안전합니다.
- 대소문자를 구분하지 않지만, 관례적으로 **대문자**를 사용합니다.

```html
<!DOCTYPE html>
<!-- HTML5 표준 모드 권장 선언 -->
<html lang="ko">
  <head>
    ...
  </head>
  <body>
    ...
  </body>
</html>
```

## 2) 왜 필요한가? (렌더링 모드와의 관계)

Doctype은 브라우저의 렌더링 모드 결정에 직접적인 영향을 줍니다.

- **Standards Mode (표준 모드)**: 최신 표준에 맞게 레이아웃/박스모델 등을 해석합니다.
- **Almost Standards Mode (부분 호환 표준 모드)**: 대부분 표준과 동일하나, 오래된 테이블/이미지 라인-박스 처리 등 극히 일부만 과거 호환 규칙 적용.
- **Quirks Mode (쿼크 모드)**: 과거(레거시) 브라우저 호환을 위한 비표준 동작을 폭넓게 허용합니다. 박스 모델 계산, 높이/라인박스 등에서 **예상치 못한 레이아웃 차이**가 커질 수 있습니다.

> **중요**: Doctype을 생략하거나 잘못 선언하면 **Quirks Mode**로 떨어질 수 있어, 브라우저·페이지별로 레이아웃이 들쭉날쭉해질 위험이 큽니다.

## 3) HTML5 이후의 간소화

- HTML5부터는 다음 한 줄로 충분합니다.

```html
<!DOCTYPE html>
```

- 더 이상 DTD(문서타입 정의) URL을 붙이지 않습니다. 선언이 간결하고, 모든 최신 브라우저에서 **표준 모드**를 안정적으로 유도합니다.

## 4) 과거(레거시) Doctype 예시

다음과 같은 긴 형태는 **HTML4/XHTML 시절**의 선언입니다. 유지보수나 마이그레이션 과정에서 마주할 수 있습니다.

```html
<!-- HTML 4.01 Strict -->
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

<!-- XHTML 1.0 Transitional -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

이들 선언은 DTD 링크와 공개 식별자를 포함하며, 선언에 따라 브라우저가 표준/부분표준/쿼크 모드 중 하나를 선택합니다. 현재 신규 프로젝트에서는 사용할 이유가 거의 없습니다.

## 5) 실무 체크리스트

1. **반드시 선언**: `<!DOCTYPE html>` 을 문서 최상단에 둡니다.
2. **인코딩 명시**: 렌더링 초기 해석 혼란을 줄이기 위해 `meta charset`을 head 최상단에 배치합니다.
   ```html
   <!DOCTYPE html>
   <html lang="ko">
     <head>
       <meta charset="utf-8" />
       <meta name="viewport" content="width=device-width, initial-scale=1" />
       <title>문서</title>
     </head>
     <body>
       ...
     </body>
   </html>
   ```
3. **서버/템플릿 전역 설정**: 프레임워크(예: Next.js, Express 템플릿, JSP, Thymeleaf 등) 기본 레이아웃에 Doctype이 빠지지 않도록 보일러플레이트를 고정합니다.
4. **이상 징후 감지**: 레이아웃이 브라우저별로 다르게 보이거나, 박스 모델 계산이 예상과 다르면 **렌더링 모드**부터 의심합니다.
5. **문서 모드 확인**: DevTools(Chrome: `document.compatMode`)로 현재 모드를 확인할 수 있습니다.
   - `"CSS1Compat"` → Standards/Almost Standards
   - `"BackCompat"` → Quirks Mode (수정 필요)

## 6) 자주 묻는 질문(FAQ)

- **Q. Doctype을 빼도 요즘 브라우저는 똑똑하니 괜찮나요?**  
  A. 아닙니다. 여전히 모드 결정에 사용되므로, 선언이 없으면 쿼크 모드로 떨어질 수 있습니다.
- **Q. XHTML을 써야 하나요?**  
  A. 특별한 사유가 없다면 HTML5 표준을 사용하세요. 복잡도가 낮고, 호환성과 도구 지원이 좋습니다.
- **Q. 대문자/소문자 차이가 있나요?**  
  A. 기능상 차이는 없지만, 관례적으로 대문자를 사용합니다.

## 7) 한 줄 결론

- **항상** `<!DOCTYPE html>` 을 **문서 첫 줄**에 선언하세요.  
  이것만으로 최신 표준 기반의 **일관된 렌더링**을 확보할 수 있습니다.
