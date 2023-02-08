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

```javascript
// TodoContext.js
import React, { useReducer, createContext, useContext } from "react";

...

export function TodoProvider({ children }) {
  ...
}

export function useTodoState() {
  return useContext(TodoStateContext);
}

export function useTodoDispatch() {
  return useContext(TodoDispatchContext);
}
```

- 이렇게 해주면 후에 아래와 같이 사용할 수 있다.

```javascript
import React from "react";
import { useTodoState, useTodoDispatch } from "../TodoContext";

function Sample() {
  const state = useTodoState();
  const dispatch = useTodoDispatch();
  return <div>Sample</div>;
}
```

## 4.4 nextId 값 관리하기

- `state` 를 위한 Context 와 `dispatch` 를 위한 Context 를 만들었는데,
- 여기서 추가적으로 `nextId` 값을 위한 Context 를 만들어주기.
  - `nextId` 값을 위한 Context 를 만들 때에도 마찬가지로 `useTodoNextId` 라는 커스텀 Hook을 만듬

```javascript
// TodoContext.js
import React, { useReducer, createContext, useContext, useRef } from 'react';

...

const TodoStateContext = createContext();
const TodoDispatchContext = createContext();
const TodoNextIdContext = createContext();

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialTodos);
  const nextId = useRef(5);

  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatchContext.Provider value={dispatch}>
        <TodoNextIdContext.Provider value={nextId}>
          {children}
        </TodoNextIdContext.Provider>
      </TodoDispatchContext.Provider>
    </TodoStateContext.Provider>
  );
}

export function useTodoState() {
  return useContext(TodoStateContext);
}

export function useTodoDispatch() {
  return useContext(TodoDispatchContext);
}
export function useTodoNextId() {
  return useContext(TodoNextIdContext);
}

```

## 4.5 Custom Hook 에서 에러 처리

- `useTodoState`, `useTodoDispatch`, `useTodoNextId` Hook 을 사용하려면, 해당 컴포넌트가 TodoProvider 컴포넌트 내부에 렌더링되어 있어야 함.
  - ex) App 컴포넌트에서 모든 내용을 TodoProvider 로 감싸기
- 만약 TodoProvider 로 감싸져있지 않다면 에러를 발생시키도록 커스텀 Hook 을 수정

<br>

- Context 사용을 위한 커스텀 Hook 을 만들 때 이렇게 에러 처리를 해준다면, 후에 실수를 하게 됐을 때 문제점을 빨리 발견 할 수 있다.

```javascript
// TodoContext.js

...

export function TodoProvider({ children }) {
  ...
}

export function useTodoState() {
  const context = useContext(TodoStateContext);
  if (!context) {
    throw new Error('Cannot find TodoProvider');
  }
  return context;
}

export function useTodoDispatch() {
  const context = useContext(TodoDispatchContext);
  if (!context) {
    throw new Error('Cannot find TodoProvider');
  }
  return context;
}

export function useTodoNextId() {
  const context = useContext(TodoNextIdContext);
  if (!context) {
    throw new Error('Cannot find TodoProvider');
  }
  return context;
}
```

## 4.6 컴포넌트 TodoProvider 로 감싸기

- 프로젝트 모든 곳에서 Todo 관련 Context 들을 사용 할 수 있도록, App 컴포넌트에서 TodoProvider 를 불러와서 모든 내용을 TodoProvider 로 감싸기

```javascript
// App.js
import { TodoProvider } from "./TodoContext";

...

function App() {
  return (
    <>
      <TodoProvider>
        <GlobalStyle />
        <TodoTemplate>
          <TodoHead />
          <TodoList />
          <TodoCreate />
        </TodoTemplate>
      </TodoProvider>
    </>
  );
}
```

- TodoHead 컴포넌트에서 useTodoState 를 사용

```javascript
// components/TodoHeader.js

import { useTodoState } from "../TodoContext";

...

function TodoHead() {
  const todos = useTodoState();
  console.log(todos);
  return (
    <TodoHeadBlock>
      <h1>2023년 01월 01일</h1>
      <div className="day">월요일</div>
      <div className="tasks-left">할일 0개 남음</div>
    </TodoHeadBlock>
  );
}
```

> **console.log(todos)** 의 결과
>
> - Array(4)
>   - 0: {id: 1, text: '프로젝트 생성하기', done: true}
>   - 1: {id: 2, text: '컴포넌트 스타일링하기', done: true}
>   - 2: {id: 3, text: 'Context 만들기', done: false}
>   - 3: {id: 4, text: '기능 구현하기', done: false}
>   - length: 4
>   - [[Prototype]]: Array(0)
