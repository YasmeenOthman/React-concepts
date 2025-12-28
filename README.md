# Resetting state with a key

## 🎯 The Goal

You want to reset component state when something changes — like going back to its initial state without manually calling multiple setState calls.

## 💡 The Trick: Use a key prop

In React, the key prop tells React how to identify a component in the tree.
If the key changes, React unmounts(delete) the old component and mounts(create) a brand new one — with fresh state.

So, if you wrap your stateful component in another component and change its key, you effectively reset all its internal state.

##  Example

### ✅ Without key (normal behavior)

```jsx
function Form() {
  const [text, setText] = useState("");
  return <input value={text} onChange={(e) => setText(e.target.value)} />;
}

export default function App() {
  return <Form />;
}
```

- Typing into the input updates text, and it stays even if you re-render App.

### 🚀 With key to reset state

```jsx
function Form() {
  const [text, setText] = useState("");
  return <input value={text} onChange={(e) => setText(e.target.value)} />;
}
export default function App() {
  const [version, setVersion] = useState(0);

  return (
    <>
      <button onClick={() => setVersion((v) => v + 1)}>Reset Form</button>
      <Form key={version} />
    </>
  );
}
```

Here’s what happens:

- Each time you click “Reset Form”, the version number changes.

- That changes the key of `<Form />`.

- React says: “New key = new component instance.”

- It throws away the old Form (and its state) and mounts a new one.

## ⚙️ You can use this anywhere

You can reset state for:

- Entire components

- Sections of UI

- Expensive forms, lists, etc.

Just wrap them and update the key to force a reset.

## Summary

| Concept       | What it does                                                                               |
| ------------- | ------------------------------------------------------------------------------------------ |
| `key` changes | React unmounts + remounts component                                                        |
| Effect        | All internal state is reset                                                                |
| When to use   | When you want a clean slate after some event (e.g., reset form, restart game, reload data) |

## Resources

- [Resetting state with a key ](https://react.dev/reference/react/useState#resetting-state-with-a-key:~:text=Show%20more-,Resetting%20state%20with%20a%20key,-You%E2%80%99ll%20often%20encounter)

---

## ToDo List example

###  Original component (for reference)

- You currently have something like this:

```jsx
import { useState } from "react";

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({ id: i, text: "Item " + (i + 1) });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState("");

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button
        onClick={() => {
          setText("");
          setTodos([{ id: todos.length, text }, ...todos]);
        }}
      >
        Add
      </button>

      <ul>
        {todos.map((item) => (
          <li key={item.id}>{item.text}</li>
        ))}
      </ul>
    </>
  );
}
```

### 🚀 Add Reset Using a key

We’ll lift the state one level up to a parent component, and use the key trick:

```jsx
import { useState } from "react";

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({ id: i, text: "Item " + (i + 1) });
  }
  return initialTodos;
}

function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState("");

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button
        onClick={() => {
          setText("");
          setTodos([{ id: todos.length, text }, ...todos]);
        }}
      >
        Add
      </button>

      <ul>
        {todos.map((item) => (
          <li key={item.id}>{item.text}</li>
        ))}
      </ul>
    </>
  );
}

export default function App() {
  const [key, setKey] = useState(0);

  return (
    <>
      <button
        onClick={() => setKey((prev) => prev + 1)}
        style={{ marginBottom: "1rem" }}
      >
        Reset TodoList
      </button>

      {/* 👇 When key changes, TodoList is remounted and all its state resets */}
      <TodoList key={key} />
    </>
  );
}
```

### 🧠 What Happens

- The App component holds a piece of state called key.
- Each time you click “Reset TodoList”, key increments (0 → 1 → 2 → ...).
- React notices the `<TodoList key={key} />` has a new key.
- React unmounts the old TodoList (throwing away its state) and mounts a fresh one.
- useState(createInitialTodos) runs again, giving you a brand-new set of 50 todos and an empty input.

### ✅ Result

- You get a fresh TodoList every time you hit “Reset”.
- You don’t have to manually call setTodos(createInitialTodos()) or setText('').
- React handles the reset cleanly by remounting the component.

---

If you don’t want to use the key trick (full remount), you can manually reset state by calling the set functions yourself
Let’s look at it clearly 👇

## ✅ Manual Reset (no key)

```jsx
import { useState } from "react";

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({ id: i, text: "Item " + (i + 1) });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState("");

  function handleAdd() {
    setTodos([{ id: todos.length, text }, ...todos]);
    setText("");
  }

  // 👇 Manual reset
  function handleReset() {
    setTodos(createInitialTodos()); // <-- important: call the function here
    setText(""); // reset text input
  }

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleAdd}>Add</button>
      <button onClick={handleReset}>Reset</button>

      <ul>
        {todos.map((item) => (
          <li key={item.id}>{item.text}</li>
        ))}
      </ul>
    </>
  );
}
```

### 🧠 Why this works

- setTodos(createInitialTodos())

  - → Calls your function immediately to make a new array of todos.
  - → React replaces the state with that new array.

- setText('')
  - → Clears the input text.

So you’re manually bringing the component back to its “initial” state.

## ⚙️ Difference from the key method

| Method                                           | What it does                                       | Scope                                     |
| ------------------------------------------------ | -------------------------------------------------- | ----------------------------------------- |
| `setTodos(createInitialTodos())` + `setText('')` | Manually resets specific pieces of state           | Partial (you choose what resets)          |
| `key` prop trick                                 | Fully unmounts & remounts component (new instance) | Whole component resets (all state inside) |

---

- ✅ Use manual reset when you just want to reset some state.
- 🧹 Use the key trick when you want to reset everything cleanly in one go.
