# XYZ 마트 할인 회원등록 날짜 계산

## 문제 설명

XYZ 마트는 10일 동안 회원 자격을 부여하며, 회원은 매일 한 가지 제품을 할인받을 수 있습니다. 정현이는 원하는 제품(`want`)과 수량(`number`)이 연속된 10일 동안의 할인 목록(`discount`)과 정확히 일치하는 날짜에 회원가입하려 합니다. 가능한 회원등록 날짜의 총 일수를 구합니다. 가능한 날이 없으면 0을 반환합니다.

### 입력

- `want`: 정현이가 원하는 제품 문자열 배열
- `number`: 각 제품의 원하는 수량 배열 (1 ≤ `want.length` = `number.length` ≤ 10)
- `discount`: XYZ 마트의 할인 제품 문자열 배열 (10 ≤ `discount.length` ≤ 100,000)

### 출력

- 원하는 제품과 수량을 모두 할인받을 수 있는 회원등록 날짜의 수 (정수)

## 해결 코드

```javascript
function solution(want, number, discount) {
  // 원하는 제품과 수량을 맵으로 저장
  const wantMap = new Map();
  for (let i = 0; i < want.length; i++) {
    wantMap.set(want[i], number[i]);
  }

  let count = 0;

  // 10일 슬라이딩 윈도우로 확인
  for (let i = 0; i <= discount.length - 10; i++) {
    // 현재 10일 동안의 제품 빈도 계산
    const currentMap = new Map();
    for (let j = i; j < i + 10; j++) {
      const item = discount[j];
      currentMap.set(item, (currentMap.get(item) || 0) + 1);
    }

    // wantMap과 currentMap 비교
    let isMatch = true;
    for (let [item, qty] of wantMap) {
      if (currentMap.get(item) !== qty) {
        isMatch = false;
        break;
      }
    }

    if (isMatch) count++;
  }

  return count;
}
```

## 코드 설명

1. **원하는 제품 맵 생성**:

   - `want`와 `number`를 `Map` 객체(`wantMap`)로 변환하여 제품과 필요한 수량 저장.
   - 예: `want=["banana", "apple"], number=[2, 1]` → `Map { "banana" => 2, "apple" => 1 }`.

2. **슬라이딩 윈도우**:

   - `discount` 배열에서 연속된 10일 구간을 슬라이딩 윈도우로 확인.
   - 각 구간 시작 인덱스 `i`에서 `i`부터 `i+9`까지 제품 빈도를 `currentMap`에 저장.

3. **일치 여부 확인**:

   - `wantMap`의 각 제품과 수량이 `currentMap`과 정확히 일치하는지 확인.
   - 모든 제품의 수량이 일치하면 `count` 증가.

4. **결과 반환**:
   - 가능한 모든 시작 날짜를 확인한 후 `count` 반환.
   - 가능한 날이 없으면 0 반환(자동 처리).

### 최적화 고려

- `Map` 대신 객체를 사용할 수 있지만, `Map`은 키-값 쌍 관리에 명시적이고 안전.
- 슬라이딩 윈도우 이동 시 이전 구간의 데이터를 재활용하면 효율성을 높일 수 있음(예: 첫 구간 계산 후, 다음 구간은 첫 번째 요소 제거, 새 요소 추가).
- 하지만 제한조건(`discount.length` ≤ 100,000, `want.length` ≤ 10) 내에서는 현재 구현으로 충분.

## 예시

### 예시 1

- 입력:
  - `want=["banana", "apple", "rice", "pork", "pot"]`
  - `number=[3, 2, 2, 2, 1]`
  - `discount=["chicken", "apple", "apple", "banana", "rice", "apple", "pork", "banana", "pork", "rice", "pot", "banana", "apple", "banana"]`
- 계산:
  - 필요: `banana:3, apple:2, rice:2, pork:2, pot:1`
  - 날짜 1 (`0~9`): `chicken:1, apple:3, banana:2, rice:2, pork:2` → 불일치 (pot 없음, banana 부족)
  - 날짜 2 (`1~10`): `apple:3, banana:2, rice:2, pork:2, pot:1` → 불일치 (banana 부족)
  - 날짜 3 (`2~11`): `apple:2, banana:3, rice:2, pork:2, pot:1` → 일치
  - 날짜 4 (`3~12`): `banana:3, rice:2, pork:2, apple:2, pot:1` → 일치
  - 날짜 5 (`4~13`): `rice:2, apple:3, pork:2, banana:2, pot:1` → 불일치 (banana 부족)
- 출력: `3`

## 시간 복잡도

- `wantMap` 생성: `O(w)` (`w`는 `want.length` ≤ 10)
- 슬라이딩 윈도우:
  - 윈도우 이동: `O(d)` (`d`는 `discount.length` - 10)
  - 각 윈도우에서 빈도 계산: `O(10) = O(1)`
  - 일치 확인: `O(w)`
- 총 시간 복잡도: `O(d * w)` → 최악 `O(100,000 * 10) = O(1,000,000)`
- 공간 복잡도: `O(w)` (최대 10개의 제품 저장)

## 제한사항 고려

- `want.length`, `number.length` ≤ 10: 맵 처리 효율적.
- `discount.length` ≤ 100,000: 슬라이딩 윈도우로 처리 가능.
- 제품 이름은 문자열, 비교 연산 단순.
