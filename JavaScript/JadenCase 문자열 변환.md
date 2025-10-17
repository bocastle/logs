# JadenCase 문자열 변환

## 문제 설명

JadenCase란 모든 단어의 첫 문자가 대문자이고, 나머지 알파벳은 소문자인 문자열입니다. 단, 첫 문자가 알파벳이 아닌 경우(예: 숫자) 이어지는 알파벳은 소문자로 유지됩니다. 문자열 `s`가 주어졌을 때, 이를 JadenCase로 변환한 문자열을 반환합니다.

### 입력

- `s`: 알파벳, 숫자, 공백문자로 이루어진 문자열 (1 ≤ `s.length` ≤ 200)
  - 숫자는 단어의 첫 문자로만 등장.
  - 숫자로만 이루어진 단어는 없음.
  - 공백문자가 연속으로 나올 수 있음.

### 출력

- JadenCase로 변환된 문자열

## 해결 코드

```javascript
function solution(s) {
  return s
    .split(" ")
    .map((word) => {
      if (word.length === 0) return word; // 빈 단어 처리
      return word[0].toUpperCase() + word.slice(1).toLowerCase();
    })
    .join(" ");
}
```

## 코드 설명

1. **문자열 분리**:

   - `s.split(' ')`: 문자열을 공백 기준으로 단어 배열로 분리.
   - 연속된 공백은 빈 문자열(`''`)로 처리됨.

2. **단어별 변환**:

   - 각 단어에 대해:
     - `word.length === 0`: 빈 단어는 그대로 반환(연속 공백 처리).
     - `word[0].toUpperCase()`: 첫 문자를 대문자로 변환.
     - `word.slice(1).toLowerCase()`: 나머지 문자를 소문자로 변환.
   - 첫 문자가 숫자인 경우, `toUpperCase()`는 숫자를 변경하지 않으며, 나머지 문자는 소문자로 변환.

3. **결과 합치기**:
   - `join(' ')`: 변환된 단어들을 공백으로 연결해 원래 형태의 문자열로 반환.

### 처리된 제한사항

- **숫자 첫 문자**: `toUpperCase()`는 숫자에 영향을 주지 않으므로, 숫자는 그대로 유지되고 나머지 알파벳만 소문자로 변환.
- **연속 공백**: `split(' ')`은 연속 공백을 빈 문자열로 처리하며, `join(' ')`으로 원래 공백 구조 유지.
- **길이 제한**: `s.length` ≤ 200이므로 성능 문제 없음.

## 예시

### 예시 1

- 입력: `s="3people unFollowed me"`
- 계산:
  - 분리: `["3people", "unFollowed", "me"]`
  - 변환: `["3people", "Unfollowed", "Me"]`
  - 결과: `"3people Unfollowed Me"`
- 출력: `"3people Unfollowed Me"`

### 예시 2

- 입력: `s="for the last week"`
- 계산:
  - 분리: `["for", "the", "last", "week"]`
  - 변환: `["For", "The", "Last", "Week"]`
  - 결과: `"For The Last Week"`
- 출력: `"For The Last Week"`

### 예시 3

- 입력: `s="hello   world"`
- 계산:
  - 분리: `["hello", "", "", "world"]`
  - 변환: `["Hello", "", "", "World"]`
  - 결과: `"Hello   World"`
- 출력: `"Hello   World"`

## 시간 복잡도

- `split(' ')`: `O(n)` (`n`은 문자열 길이)
- `map`과 문자열 변환(`toUpperCase`, `toLowerCase`): 각 단어당 `O(m)` (`m`은 단어 길이), 전체 `O(n)`
- `join(' ')`: `O(n)`
- **총 시간 복잡도**: `O(n)`
- **공간 복잡도**: `O(n)` (단어 배열과 결과 문자열)

## 제한사항 고려

- `s.length` ≤ 200: 매우 작아 성능 문제 없음.
- 연속 공백 처리: `split`과 `join`으로 자연스럽게 처리.
- 숫자 첫 문자: `toUpperCase()`가 숫자를 변경하지 않으므로 별도 처리 불필요.
