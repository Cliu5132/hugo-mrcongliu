---
title: "UseState vs UseReducer"
date: 2023-06-13
---
## Introduction

Learning React often involves learncing Hooks. Two common ones are `useState` and `useReducer`. In this blog post, we will explore these two Hooks and understand their usage and benefits.

### 1. useState

The `useState` Hook is used for managing state within a functional component in React. It allows us to declare state variables and update them, triggering a re-render of the component when the state changes. This Hook takes in an initial value as an argument and returns an array with two elements: the current state value and a function to update the state.

```jsx
const [count, setCount] = useState(0);
```

In the example above, we initialize a state variable called count with an initial value of 0. We also receive a function setCount that we can use to update the value of count. Whenever setCount is called, React will re-render the component, reflecting the updated state value.

### 2. useReducer

The `useReducer` Hook is another way of managing state in React. It is especially useful for keeping track of multiple pieces of state that rely on complex logic. Instead of directly updating the state like `useState`, `useReducer` follows the concept of reducers commonly found in Redux.

The `useReducer` Hook takes in two arguments: a reducer function and an initial state. The reducer function contains the custom state logic, and the initial state can be a simple value or an object.

```jsx
const initialState = { count: 0 };

const reducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
};

const [state, dispatch] = useReducer(reducer, initialState);
```

In the example above, we define an initial state object with a `count` property set to 0. We also create a `reducer` function that handles different actions to update the state based on the action type. The `useReducer` Hook returns the current `state` and a `dispatch` function. The `dispatch` function is used to send actions to the reducer, triggering updates to the `state`.

## Usage

### 1. useState

The `useState` Hook is typically used when we need to manage simple state values within a component. It is straightforward to use and perfect for scenarios where the state updates are not too complex.

```jsx
const [name, setName] = useState('');
```

In the example above, we initialize a state variable called name with an empty string. We can update the value of name using the `setName` function, which triggers a re-render of the component.

### 2. useReducer

The `useReducer` Hook is beneficial when we have more complex state management requirements. It allows us to handle state updates based on different actions and provides a centralized place to define the state logic.

```jsx
const initialState = { count: 0 };

const reducer = (state, action) => {
  // ...
};

const [state, dispatch] = useReducer(reducer, initialState);
```

In the example above, we define an initial state object with a count property set to 0. We also create a `reducer` function that handles different `actions` to update the `state`. The `useReducer` Hook returns the current `state` and the `dispatch` function, which we can use to send `actions` to the `reducer`.

## Conclusion

In this blog post, we explored the `useState` and `useReducer` Hooks in React. Keep in mind, there is no better option generally. We should use the most suitable one for our specific needs.