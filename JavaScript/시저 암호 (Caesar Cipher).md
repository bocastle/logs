# 🔐 JavaScript - 시저 암호 (Caesar Cipher)

## 📝 문제 설명

어떤 문장의 각 **알파벳을 일정한 거리만큼 밀어서** 다른 알파벳으로 바꾸는 암호화 방식을 **시저 암호(Caesar Cipher)** 라고 합니다.

예를 들어 `"AB"`는 1만큼 밀면 `"BC"`, 3만큼 밀면 `"DE"`가 됩니다.  
또한 `"z"`는 1만큼 밀면 `"a"`가 됩니다.

---

## 📌 제한 사항

- **공백**은 아무리 밀어도 **공백**입니다.
- `s`는 **알파벳 소문자, 대문자, 공백**으로만 이루어져 있습니다.
- `s`의 길이는 **8000 이하**입니다.
- `n`은 **1 이상, 25 이하**의 자연수입니다.

---

## ✅ 입출력 예

| s         | n   | result    |
| --------- | --- | --------- |
| `"AB"`    | 1   | `"BC"`    |
| `"z"`     | 1   | `"a"`     |
| `"a B z"` | 4   | `"e F d"` |

---

## 💡 풀이 아이디어

1. 문자열을 문자 배열로 쪼갭니다.
2. 각 문자를 순회하면서:
   - 공백은 그대로 둡니다.
   - 대소문자를 구분하여 밀어줍니다.
   - 범위를 벗어나는 경우 알파벳 순환 처리 (`% 26`)를 적용합니다.
3. 변환된 문자를 다시 합쳐 최종 문자열을 반환합니다.

---

## 💻 내 풀이 (JavaScript)

```javascript
function solution(s, n) {
  const array = s.split("");
  let answer = "";

  for (let i = 0; i < array.length; i++) {
    if (array[i] === " ") {
      answer += " ";
      continue;
    }

    let isUpperType = array[i] === array[i].toUpperCase();
    const startIndex = isUpperType ? "A".charCodeAt(0) : "a".charCodeAt(0);
    const nextIndex = (array[i].charCodeAt(0) - startIndex + n) % 26;
    const transText = String.fromCharCode(nextIndex + startIndex);

    answer += transText;
  }

  return answer;
}
```
