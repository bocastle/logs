# FormData와 URLSearchParams의 차이를 설명하는 법 정리

## 핵심 요약

- `FormData`는 문자열과 파일을 폼 데이터로 담고, `URLSearchParams`는 문자열 쌍을 URL query나 form-urlencoded 본문으로 표현한다.
- 파일 업로드가 있으면 `FormData`, 공유·복원할 검색 상태라면 `URLSearchParams`가 자연스럽다.
- `fetch`에 `FormData`를 보낼 때는 브라우저가 multipart boundary를 붙이도록 `Content-Type`을 직접 지정하지 않는다.

## 개념 설명

`FormData`는 폼의 문자열 필드와 `File`·`Blob`을 함께 담을 수 있는 자료구조다. `fetch` body로 전달하면 브라우저가 `multipart/form-data` 형식과 boundary를 만든다. `URLSearchParams`는 문자열 이름-값 목록을 query string이나 `application/x-www-form-urlencoded` 본문으로 직렬화한다.

두 API 모두 같은 키를 여러 번 가질 수 있지만 값의 타입과 사용 위치가 다르다. 파일 업로드와 네이티브 폼 제출 흐름은 `FormData`가 맞고, 검색 필터처럼 주소에 남겨 공유할 상태는 `URLSearchParams`가 적합하다. URL에는 민감정보를 넣지 않는다.

## 예시

```ts
const body = new FormData(form);
const query = new URLSearchParams({ page: "1", sort: "recent" });
```

첫 줄은 파일을 포함할 수 있는 제출 본문을 만들고, 두 번째 줄은 주소에 표현할 문자열 query를 만든다. 반복 필드는 `append()`와 `getAll()`을 사용하고 서버의 동일 키 처리 규칙과 맞춘다.

## 면접 답변 예시

> `FormData`는 문자열과 파일을 함께 담아 multipart 폼으로 전송할 때 사용하고, `URLSearchParams`는 문자열 쌍을 query나 form-urlencoded 본문으로 표현할 때 사용합니다. 파일 업로드가 있으면 `FormData`, 링크로 공유할 검색 필터라면 `URLSearchParams`를 선택하겠습니다. `FormData`를 fetch로 보낼 때는 브라우저가 boundary를 만들도록 `Content-Type`을 직접 설정하지 않습니다. 두 API의 반복 키는 서버 계약과 `getAll()` 동작까지 확인합니다.

## 장점

- `FormData`는 파일과 텍스트 입력을 네이티브 폼 구조에서 그대로 수집하기 쉽다.
- `URLSearchParams`는 검색 필터를 URL로 공유하고 새로고침·뒤로가기로 복원하기 쉽다.

## 단점

- multipart `Content-Type`을 직접 만들면 boundary가 빠져 서버가 본문을 파싱하지 못할 수 있다.
- URL에 민감한 값을 넣으면 방문 기록, 분석 도구와 서버 로그에 남는다.

## 주의사항 / 실무 팁

- 반복 키와 빈 값의 처리 방식을 클라이언트와 서버 계약에서 동일하게 정한다.
- 민감정보와 큰 payload는 query string에 올리지 않는다.
