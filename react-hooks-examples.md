# React Hooks Examples (Beginner Friendly)

This file contains simple examples of all React hooks (up to React 19).

---

## 1. useState
```jsx
import React, { useState } from "react";

export default function UseStateExample() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h3>Count: {count}</h3>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

---

## 2. useEffect
```jsx
import React, { useState, useEffect } from "react";

export default function UseEffectExample() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Clicked ${count} times`;
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Click {count}</button>
    </div>
  );
}
```

---

## 3. useContext
```jsx
import React, { createContext, useContext } from "react";

const UserContext = createContext();

function Child() {
  const user = useContext(UserContext);
  return <h3>Hello, {user}</h3>;
}

export default function UseContextExample() {
  return (
    <UserContext.Provider value="Dheeraj">
      <Child />
    </UserContext.Provider>
  );
}
```

---

## 4. useReducer
```jsx
import React, { useReducer } from "react";

function reducer(state, action) {
  switch (action.type) {
    case "inc": return { count: state.count + 1 };
    case "dec": return { count: state.count - 1 };
    default: return state;
  }
}

export default function UseReducerExample() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <h3>Count: {state.count}</h3>
      <button onClick={() => dispatch({ type: "inc" })}>+1</button>
      <button onClick={() => dispatch({ type: "dec" })}>-1</button>
    </div>
  );
}
```

---

## 5. useCallback
```jsx
import React, { useState, useCallback } from "react";

export default function UseCallbackExample() {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => setCount(c => c + 1), []);

  return (
    <div>
      <h3>Count: {count}</h3>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

---

## 6. useMemo
```jsx
import React, { useState, useMemo } from "react";

export default function UseMemoExample() {
  const [count, setCount] = useState(0);

  const double = useMemo(() => count * 2, [count]);

  return (
    <div>
      <h3>Count: {count}, Double: {double}</h3>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

---

## 7. useRef
```jsx
import React, { useRef } from "react";

export default function UseRefExample() {
  const inputRef = useRef();

  return (
    <div>
      <input ref={inputRef} placeholder="Type here" />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
    </div>
  );
}
```

---

## 8. useImperativeHandle
```jsx
import React, { useRef, forwardRef, useImperativeHandle } from "react";

const Child = forwardRef((props, ref) => {
  const sayHello = () => alert("Hello from Child!");
  useImperativeHandle(ref, () => ({ sayHello }));
  return <p>Child Component</p>;
});

export default function UseImperativeHandleExample() {
  const childRef = useRef();
  return (
    <div>
      <Child ref={childRef} />
      <button onClick={() => childRef.current.sayHello()}>Call Child</button>
    </div>
  );
}
```

---

## 9. useLayoutEffect
```jsx
import React, { useLayoutEffect, useRef } from "react";

export default function UseLayoutEffectExample() {
  const divRef = useRef();

  useLayoutEffect(() => {
    divRef.current.style.background = "yellow";
  }, []);

  return <div ref={divRef}>Hello World</div>;
}
```

---

## 10. useDebugValue
```jsx
import React, { useState, useDebugValue } from "react";

function useCount() {
  const [count, setCount] = useState(0);
  useDebugValue(count > 5 ? "High" : "Low");
  return [count, setCount];
}

export default function UseDebugValueExample() {
  const [count, setCount] = useCount();
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

---

## 11. useId (React 18)
```jsx
import React, { useId } from "react";

export default function UseIdExample() {
  const id = useId();
  return (
    <div>
      <label htmlFor={id}>Name: </label>
      <input id={id} type="text" />
    </div>
  );
}
```

---

## 12. useDeferredValue (React 18)
```jsx
import React, { useState, useDeferredValue } from "react";

export default function UseDeferredValueExample() {
  const [text, setText] = useState("");
  const deferredText = useDeferredValue(text);

  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <p>Typing: {deferredText}</p>
    </div>
  );
}
```

---

## 13. useTransition (React 18)
```jsx
import React, { useState, useTransition } from "react";

export default function UseTransitionExample() {
  const [list, setList] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(() => {
      setList(Array(2000).fill("Item"));
    });
  };

  return (
    <div>
      <button onClick={handleClick}>Load Items</button>
      {isPending ? <p>Loading...</p> : list.map((i, idx) => <div key={idx}>{i}</div>)}
    </div>
  );
}
```

---

## 14. useSyncExternalStore (React 18)
```jsx
import React, { useSyncExternalStore } from "react";

function subscribe(callback) {
  window.addEventListener("resize", callback);
  return () => window.removeEventListener("resize", callback);
}

export default function UseSyncExternalStoreExample() {
  const width = useSyncExternalStore(
    subscribe,
    () => window.innerWidth
  );

  return <h3>Window width: {width}</h3>;
}
```

---

## 15. useInsertionEffect (React 18)
```jsx
import React, { useInsertionEffect } from "react";

export default function UseInsertionEffectExample() {
  useInsertionEffect(() => {
    const style = document.createElement("style");
    style.innerHTML = "body { background: #f0f0f0; }";
    document.head.appendChild(style);
  }, []);

  return <h3>Background changed with useInsertionEffect</h3>;
}
```

---

## 16. useFormStatus (React 19 / experimental)
```jsx
// Works only in React 19+ (experimental)
import React from "react";
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? "Submitting..." : "Submit"}</button>;
}

export default function UseFormStatusExample() {
  return (
    <form action="/submit" method="post">
      <input type="text" name="name" />
      <SubmitButton />
    </form>
  );
}
```

---

## 17. useFormState (React 19 / experimental)
```jsx
// Experimental API
import { useFormState } from "react-dom";

function MyForm() {
  const [state, formAction] = useFormState(async (prev, formData) => {
    return { message: "Form submitted!" };
  }, { message: "" });

  return (
    <form action={formAction}>
      <input name="email" type="email" />
      <button type="submit">Submit</button>
      <p>{state.message}</p>
    </form>
  );
}
```

---

## 18. useOptimistic (React 19 / experimental)
```jsx
// Experimental API
import React from "react";
import { useOptimistic } from "react-dom";

export default function UseOptimisticExample() {
  const [messages, addOptimisticMessage] = useOptimistic([], (state, newMsg) => [...state, newMsg]);

  const handleSend = () => {
    addOptimisticMessage("New message (optimistic)");
  };

  return (
    <div>
      <button onClick={handleSend}>Send Message</button>
      <ul>{messages.map((m, i) => <li key={i}>{m}</li>)}</ul>
    </div>
  );
}
```
