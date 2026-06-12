# React Hook Form 정리

## 핵심 요약
- `React Hook Form`은 비제어 입력(ref 기반) 중심으로 동작해 불필요한 렌더링을 줄여 폼 성능을 개선한다.
- `register`, `handleSubmit`, `watch`, `setError` 같은 API를 통해 상태 관리와 검증을 한 곳에서 다룬다.
- 복잡한 폼(다단계 입력, 배열 입력, 조건부 필드)에서 `Controller`를 최소화하면 코드 유지보수성이 좋아진다.
- 면접에서 자주 나오는 포인트는 `성능`, `검증 전략`, `SSR/CSR 환경`, `접근성 연동`이다.

## 개념 설명
폼 입력은 값이 바뀔 때마다 컴포넌트 렌더링이 발생하기 쉬워 성능 이슈가 생긴다.  
`React Hook Form`은 브라우저 기본 폼 동작을 최대한 활용하면서 필요한 순간에만 React 상태를 읽고 쓰는 방식으로 동작한다.

핵심 구성은 다음과 같다.
- `register`로 입력을 폼 시스템에 등록해 기본값/검증 룰/에러 상태를 연결
- `handleSubmit`로 제출 타이밍을 통제하고 `resolver`로 유효성 라이브러리(Yup, Zod 등) 통합
- `mode`(`onChange`, `onBlur`, `onSubmit`)로 언제 에러를 노출할지 제어
- `formState`로 `isSubmitting`, `isValid`, `isDirty` 등 제출/검증 상태 확인

## 예시
```tsx
import { useForm } from "react-hook-form"

type LoginForm = {
  email: string
  password: string
}

export function Login() {
  const { register, handleSubmit, formState: { errors } } = useForm<LoginForm>({
    mode: "onBlur",
  })

  const onSubmit = (data: LoginForm) => {
    console.log("제출 데이터", data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("email", { required: "이메일은 필수입니다." })}
        aria-label="이메일"
      />
      {errors.email && <p>{errors.email.message}</p>}

      <input
        {...register("password", {
          required: "비밀번호는 필수입니다.",
          minLength: { value: 8, message: "8자 이상 입력하세요." },
        })}
        type="password"
        aria-label="비밀번호"
      />
      {errors.password && <p>{errors.password.message}</p>}
      <button type="submit">로그인</button>
    </form>
  )
}
```

## 면접 답변 예시
- 질문: "`React Hook Form`을 선택한 이유가 무엇인가요?"  
  답변: "필드가 많을수록 상태를 하나씩 관리하는 방식보다 재렌더링을 줄일 수 있어서요. `register` 기반으로 브라우저 폼 특성을 살리면 성능 이슈를 줄이기 쉽습니다."
- 질문: "어떤 상황에서 `Controller`를 쓰나요?"  
  답변: "MUI/AntD 같은 제어 컴포넌트가 내부 입력을 추상화한 경우, 값과 이벤트를 수동으로 바인딩해야 해서 `Controller`가 유용합니다. 순수 input라면 직접 `register`가 더 간단합니다."

## 장점
- 성능 부담이 비교적 낮다
- 검증 규칙을 일관되게 관리한다
- 중복된 로컬 상태 코드를 줄일 수 있다

## 단점
- 복잡한 커스텀 컴포넌트는 `Controller` 사용량이 늘 수 있다
- 팀이 리액트 폼 패턴에 익숙하지 않으면 초기 학습 비용이 있다

## 주의사항 / 실무 팁
- `defaultValues`와 비동기 초기값 주입 시 타이밍을 명확히 하고, `reset` 호출을 정교하게 분기한다.
- 폼 오류 메시지는 `aria-live` 등 접근성 속성과 함께 제공해 스크린리더 사용성을 확보한다.
- 제출 중 상태를 무시하면 중복 제출이 생기므로 `isSubmitting`로 버튼 비활성화를 걸어준다.
