# 자바에서 클래스 정보는 어떻게 알아낼 수 있나요? (Reflection API)

## 핵심 요약

- 자바에서 런타임에 클래스/메서드/필드 정보를 가져오려면 **Reflection API**를 사용합니다.
- 대표 타입은 `Class`, `Method`, `Field`이며, **구체적인 타입을 몰라도** 클래스 정보에 접근할 수 있습니다.
- 강력하지만 **코드 복잡도 증가**, **캡슐화 약화(강결합 위험)**, **성능 저하 가능성(JIT 최적화 어려움)** 같은 단점이 있습니다.

## 개요

자바에서 클래스 정보를 가져오기 위해서 **Reflection API**를 사용할 수 있습니다. `reflection` 패키지에서 제공하는 클래스를 사용하면, JVM에 로딩되어 있는 클래스와 메서드의 정보를 읽어올 수 있습니다. 대표적으로 `Class` 클래스, `Method` 클래스, `Field` 클래스가 존재합니다.

Reflection API를 사용하면 구체적인 클래스의 타입을 몰라도, 클래스의 정보에 접근할 수 있습니다. 개발자는 이러한 특성을 이용하여 인스턴스를 감싸는 프록시를 만들거나, 사용자로부터 전달된 값을 처리할 메서드를 유연하게 선택하는 등 다양한 구현을 할 수 있습니다. Reflection API는 특히 프레임워크나 라이브러리를 개발하는 과정에서 사용되는 경우가 많습니다. 프레임워크나 라이브러리의 개발자는 사용자가 작성한 클래스에 대한 정보를 알 수 없기 때문입니다.

## Reflection API로 알 수 있는 것들

- 클래스 자체 정보: 이름, 패키지, 상속/구현 정보 등
- 메서드 정보: 메서드 목록, 이름, 파라미터/리턴 타입 등
- 필드 정보: 필드 목록, 타입 등

## 예시 코드

```java
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ReflectionExample {
    public static void main(String[] args) throws Exception {
        Object obj = new Person("Alice", 25);

        // 1) 클래스 정보
        Class<?> clazz = obj.getClass();
        System.out.println("class: " + clazz.getName());

        // 2) 메서드 정보 및 호출
        Method getName = clazz.getMethod("getName");
        Object name = getName.invoke(obj);
        System.out.println("name: " + name);

        // 3) 필드 정보 접근(예시)
        Field age = clazz.getDeclaredField("age");
        age.setAccessible(true);
        System.out.println("age(before): " + age.get(obj));
        age.set(obj, 26);
        System.out.println("age(after): " + age.get(obj));
    }

    static class Person {
        private final String name;
        private int age;

        Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }
    }
}
```

## 장점

- 구체적인 클래스 타입을 몰라도 런타임에 클래스/메서드/필드 정보에 접근할 수 있습니다.
- 프록시 생성, 동적 메서드 선택 등 **유연한 구현**이 가능합니다.
- 프레임워크/라이브러리처럼 “사용자가 어떤 클래스를 작성할지 모르는 상황”에서 특히 유용합니다.

## 단점

- 일반 코드보다 **복잡한 코드가 필요**할 수 있습니다.
- **캡슐화가 약화**되어 강결합으로 이어질 수 있습니다.
- 일반적인 메서드 호출 대비 `Method#invoke`는 **JIT 최적화가 어려워져 성능이 저하될 가능성**이 있습니다.
  - 단, 이 부분은 사용 중인 JVM의 버전과 프로그램 상황에 따라 다를 수 있습니다.

## 주의사항 및 실무 팁

- 애플리케이션 비즈니스 로직에서 무분별하게 사용하기보다는, 프레임워크/공통 모듈처럼 “정말 동적 해석이 필요한 지점”에 한정하는 것이 좋습니다.
- `setAccessible(true)` 같은 접근 제어 우회는 캡슐화를 깨뜨릴 수 있으므로, 사용 범위와 목적을 명확히 합니다.
- 성능 이슈가 의심된다면, 반복 호출 지점의 캐싱(예: `Method`/`Field` 객체 캐싱)과 함께 실제 프로파일링을 통해 확인합니다.
