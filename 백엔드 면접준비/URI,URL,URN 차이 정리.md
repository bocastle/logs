# URI, URL, URN 차이 정리

## 한눈에 보는 정의

- **URI (Uniform Resource Identifier)**: 인터넷 자원을 **식별**하기 위한 모든 식별자 체계의 상위 개념. URL과 URN을 **포함**합니다.
- **URL (Uniform Resource Locator)**: 자원의 **위치(어디에, 어떻게 접근하는가)**를 나타내는 URI.
- **URN (Uniform Resource Name)**: 자원의 **이름(무엇인가)**을 나타내는 URI. 위치와 무관하게 **영구 식별**을 목표로 합니다.

## 구조 비교

| 구분 | 핵심 목적                 | 예시 스킴                                  | 필수 요소                                    | 예시                                                                   |
| ---- | ------------------------- | ------------------------------------------ | -------------------------------------------- | ---------------------------------------------------------------------- |
| URI  | 자원 **식별**의 총칭      | `http`, `https`, `urn`, `mailto`, `ftp` 등 | 식별 가능한 문자열                           | `https://example.com/a` , `urn:isbn:0451450523`                        |
| URL  | 자원 **위치 + 접근 방법** | `http`, `https`, `ftp` 등                  | 스킴 + 네트워크 위치(호스트 등) + 경로(옵션) | `https://www.example.com/path/to/resource?x=1#frag`                    |
| URN  | 자원 **이름(영구 식별)**  | `urn`                                      | 스킴 `urn:` + 네임스페이스 + 네임            | `urn:isbn:0451450523`, `urn:uuid:123e4567-e89b-12d3-a456-426614174000` |

> 모든 **URL**과 **URN**은 **URI**입니다. 그러나 모든 **URI**가 URL 또는 URN인 것은 아닙니다(스킴에 따라 분류).

## URL의 일반 형식

```
scheme "://" authority path? query? fragment?
└http ┘     └ userinfo@host:port ┘ └ /docs/guide ┘ └ ?q=uri ┘ └ #intro ┘
```

- **scheme**: `http`, `https`, `ftp`, `ws`, `mailto` 등
- **authority**: `userinfo@host:port` (대부분 `host[:port]`)
- **path**: 자원 경로
- **query**: 추가 파라미터
- **fragment**: 문서 내 위치 식별

## URN의 일반 형식

```
urn:<namespace>:<namestring>
예) urn:isbn:0451450523, urn:uuid:123e4567-e89b-12d3-a456-426614174000
```

- **namespace**: `isbn`, `uuid`, `issn` 등
- **namestring**: 네임스페이스 정의에 맞는 값

## 어떤 것을 언제 쓰나

- **URL**: 웹·API 접근, 브라우저 네비게이션, 파일 다운로드 등 **실제 접근**이 필요한 경우.
- **URN**: 책(ISBN), 학술자료(ISSN), UUID 등 **위치가 바뀌어도 변하지 않는 식별자**가 필요한 경우.
- **URI**: 규격 또는 문서에서 **포괄적 개념**을 언급할 때(예: “이 필드는 URI를 받습니다”).

## 자주 하는 오해

1. **URI = URL**이라고 생각하는 경우가 많지만, URN도 URI에 포함됩니다.
2. **URN은 바로 접근할 수 없다**: URN 자체는 위치가 아니므로 해석·해결(resolution) 시스템이 있어야 실제 리소스로 연결됩니다.
3. **모든 스킴이 URL**은 아님: `mailto:`, `data:` 같은 일부 스킴은 네트워크 위치가 없을 수 있어 “로케이터” 범주에 엄밀히 들어가지 않을 수 있습니다. 그래도 URI로는 인정됩니다.

## 간단 예시 모음

- **URI(총칭)**: `https://example.org/docs`, `urn:isbn:0451450523`, `mailto:dev@example.org`
- **URL**: `https://api.example.com/v1/users?limit=10`
- **URN**: `urn:uuid:550e8400-e29b-41d4-a716-446655440000`

## 체크리스트

- **접근 방법/주소가 필요**하면 → URL
- **영구 불변 식별이 목적**이면 → URN
- **둘 중 무엇이든 가능**하고 규격상 포괄 표현이 필요하면 → URI
