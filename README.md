# React-TodoList_V2

> [https://react.vlpt.us/mashup-todolist/](https://react.vlpt.us/mashup-todolist/)

> styled-components & Context API 활용 TodoList

# 1. `styled-components` - createGlobalStyle

- 특정 컴포넌트를 만들어서 스타일링 하는게 아니라 글로벌 스타일을 추가할 때

```javascript
// App.js

...

const GlobalStyle = createGlobalStyle`
  body {
    background: #e9ecef;
  }
`;
```

# 2. [`react-icons`](https://react-icons.github.io/react-icons/#/icons/md)

## 2.1 Install

```bash
> npm i react-icons
```

### 2.1.1 Usage

```javascript
import { FaBeer } from "react-icons/fa";
class Question extends React.Component {
  render() {
    return (
      <h3>
        {" "}
        Lets go for a <FaBeer />?{" "}
      </h3>
    );
  }
}
```

## 2.2 Use Material Design icons

### 2.2.1 Import

```javascript
import { IconName } from "react-icons/md";
```

### 2.2.2 Usage

```javascript
import { MdDone, MdDelete } from 'react-icons/md';

...

<TodoItemBlock>
  <CheckCircle done={done}>
    {done && <MdDone />}
  </CheckCircle>
  <Text done={done}>
    {text}
  </Text>
  <Remove>
    <MdDelete />
  </Remove>
</TodoItemBlock>
```

# 3. [`Component Selector`](https://styled-components.com/docs/advanced#referring-to-other-components)

## 3.1 Use

- `TodoItemBlock` 위에 커서가 있을 때, `Remove` 컴포넌트를 보여주라는 의미

```javascript

...

const Remove = styled.div`
  display: flex;
  align-items: center;
  justify-content: center;
  color: #dee2e6;
  font-size: 24px;
  cursor: pointer;
  &:hover {
    color: #ff6b6b;
  }
  display: none;
`;

const TodoItemBlock = styled.div`
  display: flex;
  align-items: center;
  padding-top: 12px;
  padding-bottom: 12px;
  &:hover {
    ${Remove} {
      display: initial;
    }
  }
`;
```

# 4. Context API

- 현재 TodoList 상태 관리의 구조

  - `App` 에서 `todos` 상태와, `onToggle`, `onRemove`, `onCreate` 함수를 지니고 있게 하고, 해당 값들을 `props` 를 사용해서 자식 컴포넌트들에게 전달해주는 방식
  - 프로젝트의 규모가 커지게 된다면 최상위 컴포넌트인 `App` 에서 모든 상태 관리를 하기엔 `App` 컴포넌트의 코드가 너무 복잡해질 수도 있고, `props` 를 전달해줘야 하는 컴포넌트가 너무 깊숙히 있을 수도 있다.
    ![hX8jjXG](https://user-images.githubusercontent.com/103430498/217409784-38f5a860-0ad8-4303-853d-a24345a64828.png)

- `Context API` 를 활용
  - ![lYiiIZF](https://user-images.githubusercontent.com/103430498/217409794-e060efdd-b86f-4c11-88d1-39d8b505e0b4.png)

## 4.1 `useReducer`를 사용하여 상태를 관리하는 컴포넌트 만들기

```javascript
// TodoContext.js

import React, { useReducer } from "react";

const initialTodos = [
  {
    id: 1,
    text: "프로젝트 생성하기",
    done: true,
  },
  {
    id: 2,
    text: "컴포넌트 스타일링하기",
    done: true,
  },
  {
    id: 3,
    text: "Context 만들기",
    done: false,
  },
  {
    id: 4,
    text: "기능 구현하기",
    done: false,
  },
];

function todoReducer(state, action) {
  switch (action.type) {
    case "CREATE":
      return state.concat(action.todo);
    case "TOGGLE":
      return state.map((todo) =>
        todo.id === action.id ? { ...todo, done: !todo.dobe } : todo
      );
    case "REMOVE":
      return state.fliter((todo) => todo.id !== action.id);
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialTodos);

  return children;
}
```

## 4.2 Context 만들기

- `state` 와 `dispatch` 를 Context 통하여 다른 컴포넌트에서 바로 사용 할 수 있게 만들기
  - 하나의 Context 를 만들어서 `state` 와 `dispatch` 를 함께 넣어주는 대신에, 두개의 Context 를 만들어서 따로 따로 넣어줌.
  - 이렇게 하면 dispatch 만 필요한 컴포넌트에서 불필요한 렌더링을 방지 할 수 있다.

```javascript
// TodoContext.js
import React, { useReducer, createContext } from "react";

...

const TodoStateContext = createContext();
const TodoDispatchContext = createContext();

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialTodos);
  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatchContext.Provider value={dispatch}>
        {children}
      </TodoDispatchContext.Provider>
    </TodoStateContext.Provider>
  );
}
```

- Context 에서 사용 할 값을 지정 할 때에는 위와 같이 Provider 컴포넌트를 렌더링 하고 `value` 를 설정
- props 로 받아온 `children` 값을 내부에 렌더링

  - 이렇게 하면 다른 컴포넌트에서 `state` 나 `dispatch`를 사용하고 싶을 때 아래와 같이 할 수 있다.

    ```javascript
    import React, { useContext } from "react";
    import { TodoStateContext, TodoDispatchContext } from "../TodoContext";

    function Sample() {
      const state = useContext(TodoStateContext);
      const dispatch = useContext(TodoDispatchContext);
      return <div>Sample</div>;
    }
    ```

## 4.3 Custom Hook 만들기

- 컴포넌트에서 `useContext` 를 직접 사용하는 대신에, `useContext` 를 사용하는 커스텀 Hook 을 만들어서 내보내기
