# Practice-React-TodoList-V2

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
