# React의 Props와 State

## Props

**Props**는 부모 컴포넌트가 자식 컴포넌트에 전달하는 데이터입니다.  
props는 **읽기 전용**이며, 자식 컴포넌트는 props를 수정할 수 없습니다.

```jsx
function ChildComponent(props) {
  props.name = "New Name"; // 오류 발생 가능
  return <div>{props.name}</div>;
}
```

### Props의 특징

- **읽기 전용**: 자식 컴포넌트는 props를 직접 수정할 수 없습니다.
- **단방향 데이터 흐름**: 부모 → 자식 방향으로만 데이터가 흐릅니다.
- **재사용성 향상**: 외부로부터 주어진 값만 사용하므로 독립적인 컴포넌트 작성 가능.
- **예측 가능성**: 데이터 흐름이 명확하여 디버깅이 쉬움.

### 왜 props는 자식 컴포넌트에서 변경할 수 없을까? 🤔

props가 자식 컴포넌트에서 변하지 않는 이유는 **리액트의 단방향 데이터 흐름 원칙** 때문입니다.  
리액트는 부모 컴포넌트가 자식 컴포넌트에 데이터를 전달할 때 **단방향으로만 전달**하도록 설계되었습니다.

이렇게 하면 컴포넌트 간의 데이터 흐름을 예측 가능하고 일관성 있게 만들 수 있어 **애플리케이션 상태 관리가 간단해집니다.**

- props는 **읽기 전용**이므로, 자식 컴포넌트 내에서 임의로 변경되지 않습니다.
- 특정 상태가 어디서 어떻게 변했는지를 **예측**할 수 있어 **버그 발생 가능성을 줄이고 디버깅을 쉽게** 합니다.
- 자식 컴포넌트는 독립적으로 동작할 수 있어 **재사용성이 높아지고 캡슐화**가 강화됩니다.

## State

**State**는 컴포넌트 내부에서 관리되는 데이터입니다.  
state는 **동적으로 변경**될 수 있으며, 컴포넌트의 **렌더링에 영향을 미칩니다.**

state를 변경하면 컴포넌트는 다시 렌더링되며, UI가 업데이트됩니다.  
주로 사용자 입력이나 네트워크 요청의 응답에 따라 변하는 데이터를 관리할 때 사용됩니다.

## 자식 컴포넌트에서 props를 변경해야 한다면? 🧐

자식 컴포넌트가 props를 변경해야 할 필요가 있다면, **부모 컴포넌트에서 상태(state)**로 해당 데이터를 관리하고,  
**상태 변경 함수(setState)를 자식 컴포넌트로 전달**하는 방식으로 구현해야 합니다.

이를 **상태 끌어올리기(lifting state up)** 라고 합니다.

```jsx
function ParentComponent() {
  const [name, setName] = useState("John");

  return <ChildComponent name={name} onChangeName={setName} />;
}

function ChildComponent({ name, onChangeName }) {
  return (
    <div>
      <p>{name}</p>
      <button onClick={() => onChangeName("Jane")}>Change Name</button>
    </div>
  );
}
```

이렇게 하면 데이터는 여전히 단방향으로 흐르며, 상태는 부모가 관리하기 때문에 **일관성과 유지보수성**이 높아집니다.
