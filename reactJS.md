# React.js Interview Questions & Answers

A comprehensive list of frequently asked **React.js interview questions** with clear and concise answers. Covers beginner to advanced topics.

---

## ✅ Beginner Level

### 1. What is React.js?
React is a **JavaScript library** for building **user interfaces**, developed by Facebook. It’s component-based and supports declarative views.

---

### 2. What are the features of React?
- JSX – JavaScript XML
- Components
- Virtual DOM
- One-way data binding
- Lifecycle methods
- Hooks (from React 16.8)

---

### 3. What is JSX?
JSX stands for **JavaScript XML**. It allows writing HTML elements in JavaScript and placing them in the DOM without `createElement()` or `appendChild()`.

---

### 4. What is the Virtual DOM?
A **lightweight copy of the real DOM** used by React to perform efficient updates by comparing virtual DOM trees (diffing) before updating the real DOM.

---

### 5. What are components in React?
Components are **reusable pieces of UI**, split into:
- **Functional Components** – simple functions.
- **Class Components** – ES6 classes (less common in modern React).

---

### 6. What is the difference between Functional and Class Components?
| Feature          | Functional          | Class            |
|------------------|----------------------|------------------|
| Syntax           | Function             | ES6 Class        |
| Lifecycle        | useEffect Hook       | lifecycle methods|
| State            | useState Hook        | this.state       |

---

### 7. What is the use of `state` in React?
`state` is a built-in object used to store **data or UI changes** within a component. Updating state triggers re-rendering.

---

### 8. What is `props` in React?
`props` (short for properties) are **read-only attributes** used to pass data from parent to child components.

---

### 9. What is the difference between state and props?
- **State** is local and mutable.
- **Props** are external and immutable.

---

### 10. How do you handle events in React?
React handles events similarly to DOM, but uses **camelCase**:
```jsx
<button onClick={handleClick}>Click Me</button>

---

## ⚙️ Intermediate Level

### 11. What is a React Hook?
Hooks are functions that let you **use state and other React features** in functional components (e.g., `useState`, `useEffect`).

---

### 12. What is `useState()`?
A hook that allows you to **add state** to functional components.
```jsx
const [count, setCount] = useState(0);



### 13.  What is useEffect()?
A hook that lets you perform side effects like fetching data or setting up subscriptions.

useEffect(() => {
  fetchData();
}, []);
