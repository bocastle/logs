# JPA N+1 문제 정리

> 본 문서는 Spring Data JPA + Hibernate를 전제로 설명합니다.

## 1. N+1 문제란

- **정의**: 1건의 “루트 쿼리”(예: `select u from User u`)로 **N건**의 엔티티를 조회한 뒤, 각 엔티티의 **연관 엔티티**를 접근하는 순간 **추가 쿼리 N건**이 발생하는 현상입니다.
- **전형적 예시**: `User` ↔ `Post(OneToMany)` 관계에서 모든 `User`를 조회한 후, 각 사용자에 대해 `posts`를 접근하면 사용자 수만큼 `select ... from post where user_id = ?`가 반복 실행됩니다.

---

## 2. `findAll()` 과 글로벌 패치 전략(EAGER/LAZY)

Spring Data JPA의 `Repository#findAll()`은 내부적으로 다음 **JPQL**을 실행합니다.

```jpql
select u from User u
```

중요 포인트는 **JPQL은 매핑의 글로벌 fetch 전략을 고려하지 않고** 실행된다는 점입니다. 이후 연관 필드 접근 시점에 따라 N+1이 발생합니다.

### 2.1 글로벌 **EAGER** 일 때

- `findAll()`로 `User` 목록을 읽은 뒤, Hibernate는 즉시로딩 규칙에 따라 각 연관을 **추가 쿼리**로 가져오려 시도합니다.
- 다대일(ManyToOne) 등은 조인으로 함께 가져오기도 하지만, **일대다(OneToMany)** 는 대개 **사용자마다 별도 쿼리**가 발생하여 **N+1**이 쉽게 나타납니다.
- 정리: **EAGER + findAll() ⇒ N+1 발생 위험 높음**

### 2.2 글로벌 **LAZY** 일 때

- `findAll()`은 루트만 조회하고, 연관 컬렉션은 **프록시**(초기화 전)로 주입됩니다.
- 단, 이후 코드에서 `u.getPosts()` 등 **실제 접근이 발생하면 그때 추가 쿼리**가 실행됩니다.
- 정리: **findAll() 시점에는 N+1이 보이지 않지만**, 뷰 렌더링/직렬화/로직 수행 중 접근하면 **결국 N+1**이 될 수 있습니다.

> 결론: **글로벌 패치 전략만으로 N+1을 근본 해결할 수 없습니다.** “어떤 시점에 무엇을 로드할지”를 **쿼리 단위**로 통제해야 합니다.

---

## 3. 재현 예시

```java
@Entity
class User {
  @Id @GeneratedValue Long id;
  String name;

  @OneToMany(mappedBy = "user")
  List<Post> posts = new ArrayList<>();
}

@Entity
class Post {
  @Id @GeneratedValue Long id;
  String title;

  @ManyToOne(fetch = FetchType.LAZY)
  User user;
}

// 서비스
List<User> users = userRepository.findAll(); // select u from User u
for (User u : users) {
  // 여기서 접근하는 순간 사용자 수만큼 추가 쿼리
  System.out.println(u.getPosts().size());
}
```

---

## 4. 해결 전략

### 4.1 `fetch join` (가장 직접적)

연관을 **한 번에** 로드합니다.

```java
// Repository
@Query("""
  select distinct u
  from User u
  left join fetch u.posts
""")
List<User> findAllWithPosts();
```

- **장점**: 단일 쿼리로 즉시 로딩, N+1 차단.
- **주의**:
  - **중복행** → `distinct` 필요(엔티티 중복 제거).
  - **페이징 주의**: JPA 기준 **컬렉션 fetch join + 페이징**은 메모리에서 페이징 처리되어 성능/메모리 이슈가 생길 수 있습니다. (to-one 위주 페치 조합 또는 별도 전략 고려)

### 4.2 `@EntityGraph`

JPQL 작성 없이 **로딩 그래프**를 선언적으로 지정합니다.

```java
@EntityGraph(attributePaths = {"posts"}, type = EntityGraph.EntityGraphType.FETCH)
@Query("select u from User u")
List<User> findAllWithPostsByEntityGraph();

// 또는 메서드 이름 기반에도 적용 가능
@EntityGraph(attributePaths = {"posts"})
List<User> findAll();
```

- **장점**: 간결, 재사용성.
- **주의**: 내부적으로는 조인(fetch) 전략을 힌트로 전달. 복잡한 조건/다단계 그래프는 가독성 관리 필요.

### 4.3 배치 페치(`@BatchSize`, 글로벌 `hibernate.default_batch_fetch_size`)

LAZY 컬렉션/프록시 초기화 시 **IN 쿼리로 묶어서** 가져옵니다.

```java
@Entity
@BatchSize(size = 50) // 또는 application 설정으로 글로벌 적용
class User { ... }
```

- **효과**: 1 + N → 1 + `ceil(N / batch)` 로 **쿼리 수 감소**.
- **장점**: 기존 코드 변경 최소화, 다단계 로딩에도 유용.
- **한계**: 여전히 **여러 번**의 라운드트립이 남습니다(완전 단일 쿼리는 아님).

### 4.4 DTO 투사(Projection)로 분리 조회

- **전략**: 루트 목록과 연관 컬렉션을 **두 번의 쿼리로 명시적으로** 가져와 애플리케이션에서 **그룹핑/매핑**.
- **장점**: 페이징과 조인 제약을 회피, 필요한 필드만 선택.
- **예시**: 사용자 페이지는 `select u ... limit ...`, 이후 `select p ... where user_id in (...)` 로 한 번 더 가져와 Map으로 결합.

### 4.5 서브셀렉트/특수 페치

- Hibernate의 `@Fetch(FetchMode.SUBSELECT)` 등으로 동일 세션에서 로딩된 컬렉션을 **한 번 더의 서브쿼리**로 모아 가져옴.
- 데이터 패턴에 따라 유리할 수 있으나 **DB/쿼리 플래너 특성**을 고려해야 합니다.

---

## 5. 상황별 선택 가이드

- **목록 + 상세 컬렉션을 즉시 보여줘야 함** → `fetch join` 또는 `@EntityGraph`
- **페이지네이션(대량) + 컬렉션 필요** → **DTO 2‑step 로딩** 또는 **배치 페치** 병행
- **간헐적 접근, 화면 간 이동 많음** → 배치 페치 + 캐시 고려
- **중첩 연관이 깊음** → 필요한 경로만 선별적으로 fetch, 나머지는 배치 페치

---

## 6. Spring Data JPA `findAll()`과 조합

- 기본 `findAll()`은 `select u from User u`만 수행 → **N+1 위험**
- 대안
  - **전용 메서드**에 `@EntityGraph` 부여
  - **커스텀 JPQL + fetch join** 메서드 제공
  - **Querydsl/스펙**으로 조건별 fetch 제어
  - 글로벌 `default_batch_fetch_size`로 방어선 구축

```java
// 예시: 엔티티 그래프 부착
public interface UserRepository extends JpaRepository<User, Long> {

  @Override
  @EntityGraph(attributePaths = {"posts"})
  List<User> findAll();

  // 또는 별도 메서드로 분리
  @EntityGraph(attributePaths = {"posts"})
  List<User> findAllWithPosts();
}
```

---

## 7. 추가 팁

- **양방향 연관 직렬화 주의**: 컨트롤러에서 엔티티 직접 JSON 변환 시 순환 참조/프록시 초기화로 N+1이 터질 수 있습니다. DTO로 **정렬/필터링/절단**하세요.
- **OSIV(Open Session In View)** 활성화 시 뷰 렌더링 단계에서 컬렉션 접근으로 **늦은 시점 N+1**이 발생하기 쉽습니다. 서비스 계층에서 필요한 데이터 로딩을 **완료**하세요.
- **인덱스/카디널리티**: fetch join으로 폭넓게 가져올수록 조인이 무거워질 수 있습니다. **실제 실행 계획**을 계속 확인하십시오.

---

## 요약

- N+1은 “루트 1 + 연관 N”의 **과다한 추가 조회** 문제입니다.
- **글로벌 Fetch(EAGER/LAZY)만으로는 해결 불가**하며, **쿼리 단위**로 로딩 전략을 명시해야 합니다.
- 대표 해법은 **fetch join**, **@EntityGraph**, **배치 페치**, **DTO 분리 조회**이며, **페이징/성능 특성**에 따라 혼합 적용이 필요합니다.
