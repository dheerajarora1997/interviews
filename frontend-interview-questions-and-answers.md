# Senior Frontend Interview Questions & Answers
## React, Next.js & JavaScript — 7+ Years Experience

> A practical interview guide for senior frontend developers.  
> The answers emphasize **internals, architecture, performance, security, trade-offs, and production experience** rather than memorized definitions.

---

# Table of Contents

- [React](#react)
  - [1. Reconciliation and Fiber](#1-explain-reacts-reconciliation-algorithm-and-how-fiber-improves-it)
  - [2. useLayoutEffect vs useEffect](#2-when-would-you-use-uselayouteffect-over-useeffect)
  - [3. React Compiler and memoization](#3-how-does-react-19s-compiler-change-performance-optimization-strategies)
  - [4. Data-fetching custom hook](#4-design-a-custom-hook-for-data-fetching-with-caching-retries-and-deduplication)
  - [5. Controlled vs uncontrolled](#5-controlled-vs-uncontrolled-components-at-scale)
  - [6. Preventing unnecessary re-renders](#6-prevent-unnecessary-re-renders-in-deeply-nested-component-trees)
  - [7. Hydration mismatch](#7-what-is-hydration-mismatch-and-how-do-you-debug-it)
  - [8. Optimistic UI](#8-how-do-you-implement-optimistic-ui-with-rollback)
  - [9. React 19 `use` hook](#9-how-does-the-use-hook-change-data-fetching-patterns)
  - [10. Design system architecture](#10-how-would-you-architect-a-design-system)
- [Next.js](#nextjs)
  - [11. Partial Prerendering](#11-explain-partial-prerendering-ppr)
  - [12. Next.js caching](#12-how-has-caching-changed-in-nextjs)
  - [13. Server Actions authorization](#13-how-do-you-implement-authorization-in-server-actions)
  - [14. revalidateTag vs revalidatePath](#14-when-would-you-use-revalidatetag-vs-revalidatepath)
  - [15. Slow third-party APIs](#15-how-do-you-handle-slow-third-party-api-calls)
  - [16. Product page and Core Web Vitals](#16-design-a-product-detail-page-for-maximum-core-web-vitals)
  - [17. after()](#17-explain-the-after-api)
  - [18. Middleware authentication](#18-how-do-you-implement-middleware-based-authentication)
  - [19. Bundle bloat](#19-what-are-common-mistakes-that-bloat-a-nextjs-bundle)
  - [20. Pages Router to App Router migration](#20-how-would-you-migrate-a-large-pages-router-app)
- [JavaScript](#javascript)
  - [21. Promise.allSettled](#21-implement-promiseallsettled-from-scratch)
  - [22. Event loop](#22-explain-the-event-loop-with-microtasks-vs-macrotasks)
  - [23. Temporal Dead Zone](#23-how-does-the-temporal-dead-zone-tdz-work)
  - [24. Debouncing and throttling](#24-implement-debouncing-and-throttling)
  - [25. Prototypes vs classes](#25-explain-prototypal-inheritance-vs-class-syntax)
  - [26. WeakMap and WeakSet](#26-how-do-weakmap-and-weakset-help-with-memory-management)
  - [27. Circular deep clone](#27-implement-a-deep-clone-function-that-handles-circular-references)
  - [28. Closures](#28-explain-how-closures-work)
  - [29. Reactive state management](#29-how-would-you-implement-a-reactive-state-management-system)
  - [30. structuredClone vs JSON vs manual copy](#30-structuredclone-vs-jsonparse-stringify-vs-manual-copy)
- [Senior-Level Interview Strategy](#senior-level-interview-strategy)

---

# React

## 1. Explain React's reconciliation algorithm and how Fiber improves it.

### Short answer

**Reconciliation** is React's process of comparing the previous element tree with the new element tree and determining the minimum set of changes required for the UI.

**Fiber** is React's internal architecture that represents work as small units. It allows React to schedule, pause, resume, prioritize, and discard rendering work instead of treating a large render as one indivisible operation.

### How reconciliation works

When state or props change:

1. React creates a new element tree.
2. It compares the new tree with the previous tree.
3. It determines which components/elements can be reused.
4. It calculates updates.
5. The commit phase applies changes to the DOM.

For lists, **keys** are critical because they help React identify stable identities.

```jsx
items.map(item => (
  <Row key={item.id} item={item} />
))
```

Using an array index as a key can cause incorrect state preservation when items are inserted, deleted, or reordered.

### What Fiber adds

Conceptually, Fiber separates work into:

- **Render/reconciliation phase** — calculate what should change.
- **Commit phase** — apply the changes.

The render work can be scheduled according to priority. This is important for concurrent rendering because expensive background work should not unnecessarily block urgent interactions.

### When is this useful?

For example, imagine:

- A page rendering thousands of rows.
- A user types into a search field.
- A large filtering calculation is also running.

With modern React scheduling, urgent input updates can receive higher priority while expensive work can be interrupted or deferred.

### Senior-level point

Fiber does **not** make every component automatically faster. Its major benefit is **better scheduling and responsiveness**, especially for large or interactive applications.

---

## 2. When would you use `useLayoutEffect` over `useEffect`?

### Short answer

Use `useLayoutEffect` when an effect must run **after React has updated the DOM but before the browser paints**.

Use `useEffect` for most side effects that do not need to block painting.

### Example

Suppose a tooltip needs to measure an element and reposition itself before the user sees it:

```jsx
function Tooltip({ targetRef }) {
  const tooltipRef = useRef(null);

  useLayoutEffect(() => {
    const target = targetRef.current;
    const tooltip = tooltipRef.current;

    if (!target || !tooltip) return;

    const rect = target.getBoundingClientRect();

    tooltip.style.left = `${rect.left}px`;
    tooltip.style.top = `${rect.bottom + 8}px`;
  }, []);

  return <div ref={tooltipRef}>Tooltip</div>;
}
```

If this were done in `useEffect`, the browser may paint the initial incorrect position first, producing visible flicker.

### Use `useEffect` for

- API calls
- subscriptions
- analytics
- timers
- logging
- non-visual synchronization

### Important trade-off

`useLayoutEffect` blocks painting. Excessive work inside it can hurt performance.

**Senior answer:** default to `useEffect`; use `useLayoutEffect` only when DOM measurement or synchronous visual adjustment genuinely requires pre-paint timing.

---

## 3. How does React 19's compiler change performance optimization strategies?

### Short answer

React's compiler can automatically optimize many cases involving memoization and stable values/functions. This reduces the need to manually add `useMemo`, `useCallback`, and `memo` everywhere.

The important change is that developers should increasingly optimize based on **measured bottlenecks**, not by mechanically memoizing everything.

### Traditional approach

```jsx
const filtered = useMemo(
  () => products.filter(p => p.category === category),
  [products, category]
);

const handleSelect = useCallback(
  id => onSelect(id),
  [onSelect]
);
```

These APIs still have legitimate uses, but unnecessary memoization can make code harder to understand.

### Better senior-level approach

First measure:

- React DevTools Profiler
- browser Performance panel
- production metrics
- component render frequency

Then optimize the actual bottleneck.

### When manual memoization still matters

Examples include:

- interacting with third-party libraries requiring stable references
- expensive calculations where compiler optimization does not apply
- carefully controlled component boundaries
- integration code where referential identity is part of the API contract

### Key interview point

**Memoization is not automatically a performance improvement.**

It has its own costs:

- dependency tracking
- memory usage
- complexity
- stale dependency bugs

The compiler shifts the default toward simpler React code, while profiling remains essential.

---

## 4. Design a custom hook for data fetching with caching, retries, and deduplication.

A production-grade data-fetching layer should handle:

- cache
- request deduplication
- retries
- cancellation
- stale data
- errors
- loading state
- SSR/CSR boundaries

A simplified example:

```jsx
const cache = new Map();
const inFlight = new Map();

async function fetchWithRetry(url, retries = 2) {
  let lastError;

  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      const response = await fetch(url);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      return response.json();
    } catch (error) {
      lastError = error;

      if (attempt < retries) {
        await new Promise(resolve =>
          setTimeout(resolve, 2 ** attempt * 500)
        );
      }
    }
  }

  throw lastError;
}

function getData(url) {
  if (cache.has(url)) {
    return Promise.resolve(cache.get(url));
  }

  if (inFlight.has(url)) {
    return inFlight.get(url);
  }

  const promise = fetchWithRetry(url)
    .then(data => {
      cache.set(url, data);
      return data;
    })
    .finally(() => {
      inFlight.delete(url);
    });

  inFlight.set(url, promise);

  return promise;
}
```

A hook can subscribe React state to this cache.

### Deduplication

If five components request the same URL simultaneously, they should ideally share one network request rather than create five.

### SSR vs CSR

With SSR/Server Components:

- Fetch data on the server when appropriate.
- Avoid sending unnecessary secrets to the browser.
- Serialize only data required by the client.
- Avoid duplicating server and client requests.

For CSR:

- handle loading states
- cancellation
- retries
- stale cache
- browser-specific behavior

### Production recommendation

For complex applications, use a mature data layer rather than building a full caching system from scratch unless there is a specific requirement.

---

## 5. Controlled vs uncontrolled components at scale

### Controlled

React owns the value:

```jsx
<input
  value={email}
  onChange={e => setEmail(e.target.value)}
/>
```

### Uncontrolled

The DOM owns the value:

```jsx
<input ref={inputRef} defaultValue="" />
```

### Controlled advantages

- easy validation
- predictable state
- conditional UI
- synchronization with other components
- easy state-driven rendering

### Uncontrolled advantages

- fewer React state updates
- useful for very large forms
- simpler for values that do not need continuous React synchronization

### At scale

For a form with hundreds of fields, updating React state on every keystroke can cause unnecessary rendering if the architecture is poorly designed.

A performant form architecture can use:

- uncontrolled inputs
- field-level subscriptions
- isolated components
- selective validation
- batched updates

### Senior-level answer

Don't choose uncontrolled simply because it is "faster." Choose based on **ownership of state and rendering requirements**.

---

## 6. Prevent unnecessary re-renders in deeply nested component trees

A senior engineer should investigate the cause before adding memoization.

### Techniques

#### 1. Split contexts

Instead of:

```jsx
<AppContext.Provider value={{ user, theme, cart }}>
```

separate concerns:

```jsx
<UserContext.Provider>
  <ThemeContext.Provider>
    <CartContext.Provider>
```

A component subscribing only to theme should not need to react to every user/cart update.

#### 2. Stable references

Avoid:

```jsx
<Child config={{ enabled: true }} />
```

if the child is memoized and the object is recreated every render.

#### 3. `React.memo`

Use it around components that benefit from stable props and expensive rendering.

#### 4. Selector pattern

Instead of subscribing to an entire store, select only what is needed:

```js
const userName = useStore(state => state.user.name);
```

### Architecture matters more than `memo`

Often the biggest improvement comes from moving state closer to where it is used.

---

## 7. What is hydration mismatch and how do you debug it?

### Definition

Hydration occurs when React attaches behavior to HTML that was already rendered on the server.

A hydration mismatch happens when the client generates markup that differs from the server-generated markup.

### Common causes

#### Time-dependent rendering

```jsx
<div>{new Date().toLocaleTimeString()}</div>
```

Server and client can produce different values.

#### Random values

```js
Math.random()
```

#### Browser-only APIs during render

```js
window.innerWidth
```

#### Different data

Server and client receive different initial state.

#### Invalid HTML

Incorrect nesting can also produce unexpected DOM structures.

### Debugging strategy

1. Reproduce in production mode.
2. Inspect hydration warnings.
3. Identify nondeterministic rendering.
4. Compare server and client inputs.
5. Move browser-only behavior into effects where appropriate.
6. Use explicit server-provided initial state.

### Senior-level point

Do not blindly suppress hydration warnings. Fix the source of nondeterminism whenever possible.

---

## 8. How do you implement optimistic UI with rollback?

Optimistic UI updates the interface immediately, assuming the server operation will succeed.

Example:

```jsx
async function deleteItem(id) {
  const previous = items;

  setItems(current =>
    current.filter(item => item.id !== id)
  );

  try {
    await api.delete(`/items/${id}`);
  } catch (error) {
    setItems(previous);
    throw error;
  }
}
```

### Production concerns

A real implementation must consider:

- concurrent mutations
- duplicate requests
- retries
- server-generated IDs
- stale responses
- authorization failures
- cache invalidation

### Better architecture

1. Capture previous state.
2. Apply optimistic mutation.
3. Send request.
4. If successful, reconcile with server response.
5. If failed, rollback or refetch.
6. Invalidate affected cache.

### Important point

Optimistic UI is not simply "update the UI before API response."

The client and server must eventually converge on the **same canonical state**.

---

## 9. How does the `use` hook change data-fetching patterns?

React's `use` API allows a component to consume a promise or other supported resource and suspend while the value is unavailable.

Conceptually:

```jsx
function Product({ productPromise }) {
  const product = use(productPromise);

  return <h1>{product.name}</h1>;
}
```

Combined with Suspense:

```jsx
<Suspense fallback={<ProductSkeleton />}>
  <Product productPromise={productPromise} />
</Suspense>
```

### Why this matters

Instead of manually managing:

```jsx
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);
const [data, setData] = useState(null);
```

React's rendering model can coordinate asynchronous resources with Suspense.

### Server Components

In a Server Component architecture, data fetching can often remain on the server, reducing:

- client JavaScript
- network round trips
- exposed credentials
- duplicated data-fetching logic

### Senior-level point

`use` is not a replacement for every data-fetching library. Cache policy, mutations, invalidation, retries, and client synchronization still require architectural decisions.

---

## 10. How would you architect a design system?

A scalable design system should separate:

### 1. Design tokens

```js
const tokens = {
  spacing: {
    sm: "4px",
    md: "8px",
    lg: "16px"
  },
  radius: {
    sm: "4px",
    md: "8px"
  }
};
```

Prefer semantic tokens where possible:

```css
--color-background-primary
--color-text-primary
--color-border-default
```

### 2. Primitive components

Examples:

- Button
- Input
- Stack
- Grid
- Text
- Modal

### 3. Composite components

Examples:

- DatePicker
- SearchBox
- DataTable
- FormField

### 4. Accessibility

Build accessibility into primitives:

- keyboard navigation
- focus management
- semantic HTML
- ARIA only when necessary
- screen-reader behavior
- visible focus states

### 5. Tree-shaking

Avoid a single giant client-side entry point that imports everything.

Prefer modular exports:

```js
import { Button } from "@company/ui/button";
```

rather than forcing the application to load the entire library.

### 6. Theming

Use CSS variables/design tokens so themes can change without duplicating component implementations.

### Senior-level point

A design system is not just a component library. It is a **contract for consistency, accessibility, API stability, and developer experience**.

---

# Next.js

## 11. Explain Partial Prerendering (PPR).

PPR is an architecture where a route can have a statically rendered shell while dynamic portions are streamed or rendered separately.

Conceptually:

```text
Request
   |
   +-- Static shell
   |     Header
   |     Navigation
   |     Product layout
   |
   +-- Dynamic content
         User-specific data
         Inventory
         Recommendations
```

### PPR vs traditional SSR

Traditional SSR can make the response dependent on dynamic work before useful HTML is delivered.

PPR aims to allow the stable portion to be reused while dynamic content is handled separately.

### PPR vs streaming

They are related but not identical.

- **Streaming** progressively sends rendered output.
- **PPR** separates a route into reusable/static and dynamic regions.

### When useful?

Examples:

- ecommerce product pages
- dashboards
- personalized pages
- content sites with small dynamic areas

### Senior-level consideration

The correct choice depends on:

- cacheability
- personalization
- data volatility
- infrastructure
- SEO
- Core Web Vitals

Do not use PPR simply because it is newer.

---

## 12. How has caching changed in Next.js?

One of the most important senior-level topics is that different caching mechanisms should not be treated as one generic "Next.js cache."

Think in layers:

```text
Browser
   ↓
CDN / HTTP cache
   ↓
Next.js / server-side caches
   ↓
Data cache
   ↓
Database / API
```

Different versions of Next.js have changed caching defaults and APIs, so interview answers should be tied to the exact Next.js version being used.

### Important distinction

Ask:

- Is the data cached?
- Is the rendered output cached?
- Is the client Router Cache involved?
- When does revalidation occur?
- What invalidates the cache?

### Senior-level answer

Do not memorize "fetch is cached" or "fetch is not cached."

Instead explain **which cache layer**, **why the data is cacheable**, and **how invalidation works**.

---

## 13. How do you implement authorization in Server Actions?

A Server Action is server-side code callable through a framework mechanism, but it should **never be treated as trusted simply because it is defined on the server**.

Example:

```ts
"use server";

export async function updateProfile(formData: FormData) {
  const session = await getSession();

  if (!session?.user?.id) {
    throw new Error("Unauthorized");
  }

  const name = String(formData.get("name") ?? "").trim();

  if (!name) {
    throw new Error("Name is required");
  }

  await updateUser(session.user.id, { name });
}
```

### Correct security model

Validate:

1. Authentication
2. Authorization
3. Input validation
4. Resource ownership
5. Business rules

For example, do not trust:

```js
userId = formData.get("userId")
```

Instead derive the user identity from the authenticated session.

### Additional concerns

- CSRF protection
- rate limiting
- validation
- audit logging
- least privilege
- secure cookies
- preventing IDOR/resource-access vulnerabilities

### Senior-level answer

The server must enforce authorization **at the action/data boundary**, not only in the UI.

---

## 14. `revalidateTag` vs `revalidatePath`

### `revalidatePath`

Use when a specific route/path should be revalidated.

Conceptually:

```ts
revalidatePath("/products/123");
```

This is useful when the mutation directly affects a particular route.

### `revalidateTag`

Use when multiple pages share data identified by a semantic cache tag.

For example:

```ts
fetch("/api/products/123", {
  next: {
    tags: ["product:123"]
  }
});
```

After updating the product:

```ts
revalidateTag("product:123");
```

Now all relevant cached data associated with that tag can be invalidated.

### Rule of thumb

Use:

- **Path** → "this route needs updating."
- **Tag** → "this underlying piece of data changed."

### Senior-level architecture

Tags scale better when many pages consume the same entity.

For example:

```text
Product detail
Category page
Search results
Recommendations
Admin preview
```

could all depend on the same product entity.

---

## 15. How do you handle slow third-party APIs without blocking the page?

Do not make the entire page wait for an optional slow dependency.

Use Suspense boundaries:

```jsx
export default function ProductPage() {
  return (
    <>
      <ProductHero />

      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews />
      </Suspense>

      <Suspense fallback={<RecommendationSkeleton />}>
        <Recommendations />
      </Suspense>
    </>
  );
}
```

### Good strategy

Identify which data is:

- critical
- useful but optional
- personalized
- expensive
- unreliable

### Example

```text
Product title/image/price
        ↓
     render fast

Reviews API
        ↓
     stream later

Recommendations API
        ↓
     stream later
```

### Production safeguards

Also consider:

- timeout
- retry policy
- circuit breaker
- caching
- stale data
- graceful fallback
- observability

### Senior-level answer

Streaming improves perceived performance, but it does not magically make slow APIs faster. The dependency still needs reliability engineering.

---

## 16. Design a product detail page for maximum Core Web Vitals.

A good architecture separates critical content from secondary content.

### Static/fast shell

Prioritize:

- product title
- primary image
- price
- availability
- essential metadata

### Dynamic/secondary sections

Stream:

- reviews
- recommendations
- recently viewed products
- personalized offers

### Image strategy

Use appropriately sized responsive images and avoid loading large images when they are not visible.

### JavaScript

Keep client components small.

Avoid turning the entire page into a Client Component simply because one interactive section needs browser APIs.

### Core Web Vitals

Focus on:

#### LCP

Make the primary hero content load quickly.

#### INP

Avoid long JavaScript tasks and expensive event handlers.

#### CLS

Reserve layout space for images, ads, and dynamic content.

### Senior-level architecture

```text
Server-rendered product shell
        |
        +-- Product information
        |
        +-- Primary image
        |
        +-- Purchase controls
        |
        +-- Suspense: Reviews
        |
        +-- Suspense: Recommendations
```

The goal is not just "fast server response"; it is **fast useful interaction and visual stability**.

---

## 17. Explain the `after()` API.

`after()` is designed for work that can happen **after the main response or operation has completed**, depending on the execution context.

Typical examples:

- analytics
- logging
- cleanup
- non-critical side effects

Conceptually:

```ts
import { after } from "next/server";

export async function GET() {
  const response = Response.json({ success: true });

  after(async () => {
    await logAnalytics();
  });

  return response;
}
```

### Why it is useful

Without this pattern, developers may unnecessarily make users wait for non-critical work.

### Important caveat

Do not put business-critical operations in a post-response task if the user-facing operation depends on their successful completion.

For example:

**Good candidate:**

```text
Send response
   ↓
log analytics
```

**Bad candidate:**

```text
Send response
   ↓
actually charge the customer
```

If payment is required for the operation to be considered successful, it should not be treated as optional post-response work.

---

## 18. How do you implement middleware-based authentication at the edge?

Middleware can inspect a request before it reaches a route.

Conceptually:

```ts
export async function middleware(request) {
  const token = request.cookies.get("session");

  if (!token) {
    return Response.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}
```

### But middleware is not enough

Authorization should still be enforced at the server/data layer.

Think:

```text
Middleware
   ↓
Fast request filtering
   ↓
Route
   ↓
Server-side authorization
   ↓
Database/resource access
```

### Limitations

Edge execution can have constraints around:

- runtime APIs
- database drivers
- Node-specific dependencies
- latency
- token/session verification
- complex authorization logic

### Senior-level point

Middleware is a **routing/security boundary**, not a replacement for resource-level authorization.

---

## 19. What commonly bloats a Next.js bundle?

### Common causes

- importing large libraries into Client Components
- importing entire utility libraries
- accidentally marking large trees with `"use client"`
- large chart/editor packages
- duplicate dependencies
- unnecessary polyfills
- shipping server-only code to the browser
- large icon imports
- excessive third-party scripts

### Example

Bad architecture:

```text
Page
 ↓
"use client"
 ↓
Entire page becomes client-heavy
 ↓
Large dependency graph shipped to browser
```

Better:

```text
Server Component
 |
 +-- Server-rendered content
 |
 +-- Small Client Component for interaction
```

### Audit strategy

Measure rather than guess:

- bundle analyzer
- browser network panel
- source maps
- Lighthouse/PageSpeed
- production telemetry

### Senior-level point

The biggest bundle optimization is often **reducing the amount of code that needs to be client-side in the first place**.

---

## 20. How would you migrate a large Pages Router app to App Router incrementally?

Do not rewrite the entire application at once.

### Step 1 — Inventory

Identify:

- pages
- API routes
- authentication
- global state
- data fetching
- shared components
- dependencies

### Step 2 — Establish boundaries

Determine which components can become Server Components and which genuinely need:

```js
"use client";
```

### Step 3 — Migrate low-risk routes

Start with:

- static pages
- documentation
- low-traffic routes
- isolated features

### Step 4 — Establish patterns

Standardize:

- loading UI
- error boundaries
- metadata
- data access
- authentication
- cache strategy

### Step 5 — Migrate complex areas

Move high-value routes once the architecture is proven.

### Key architectural differences

Pages Router often centers around:

- `getServerSideProps`
- `getStaticProps`
- page-level data fetching

App Router introduces:

- layouts
- Server Components
- nested loading/error boundaries
- Suspense
- Server Actions
- different caching/data-fetching patterns

### Senior-level answer

Migration should be **incremental, observable, reversible, and driven by business value**, not a technology rewrite for its own sake.

---

# JavaScript

## 21. Implement `Promise.allSettled` from scratch.

`Promise.allSettled()` waits for every promise and reports whether each one fulfilled or rejected.

```js
function allSettled(promises) {
  return new Promise(resolve => {
    const items = Array.from(promises);

    if (items.length === 0) {
      resolve([]);
      return;
    }

    const results = new Array(items.length);
    let completed = 0;

    items.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = {
            status: "fulfilled",
            value
          };
        })
        .catch(reason => {
          results[index] = {
            status: "rejected",
            reason
          };
        })
        .finally(() => {
          completed++;

          if (completed === items.length) {
            resolve(results);
          }
        });
    });
  });
}
```

### Difference from `Promise.all`

`Promise.all()` rejects as soon as one input rejects.

```js
await Promise.all([
  requestA(),
  requestB(),
  requestC()
]);
```

If `requestB()` fails, the combined promise rejects.

`allSettled()` waits for everything:

```js
const results = await Promise.allSettled([
  requestA(),
  requestB(),
  requestC()
]);
```

### Use cases

`allSettled()` is useful when partial success is acceptable:

- dashboard widgets
- batch operations
- analytics
- independent API calls

---

## 22. Explain the event loop with microtasks vs macrotasks.

JavaScript executes synchronous code first.

After the current call stack completes, the runtime processes queued asynchronous work.

A simplified model:

```text
Call Stack
    ↓
Microtasks
    ↓
Rendering opportunity
    ↓
Task/Macrotask
    ↓
Microtasks
    ↓
...
```

### Microtasks

Examples:

- `Promise.then`
- `catch`
- `finally`
- `queueMicrotask`

### Tasks/macrotasks

Examples include:

- timers such as `setTimeout`
- certain DOM events
- message events

### Example

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

Output:

```text
A
D
C
B
```

Why?

1. `A` executes.
2. Timer is scheduled.
3. Promise callback becomes a microtask.
4. `D` executes.
5. Current stack finishes.
6. Microtask `C` executes.
7. Timer callback `B` executes later.

### Senior-level point

Too many chained microtasks can delay rendering and other tasks. "Async" does not automatically mean "non-blocking."

---

## 23. How does the Temporal Dead Zone work?

The TDZ is the period between entering a scope and the point where a `let` or `const` variable is initialized.

Example:

```js
console.log(name);

const name = "John";
```

This throws a `ReferenceError`.

Another example:

```js
{
  console.log(value);

  let value = 10;
}
```

The variable exists in the lexical environment but cannot be accessed before initialization.

### `var`

```js
console.log(value); // undefined

var value = 10;
```

`var` is initialized with `undefined` during hoisting.

### Important distinction

It is misleading to say simply "`let` and `const` are not hoisted."

They are hoisted in the sense that their bindings are created during environment setup, but they remain **uninitialized** until execution reaches their declaration.

---

## 24. Implement debouncing and throttling with `this` binding and cleanup.

### Debounce

Run after calls stop for a specified amount of time.

```js
function debounce(fn, delay) {
  let timer;

  function debounced(...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  }

  debounced.cancel = () => {
    clearTimeout(timer);
  };

  return debounced;
}
```

### Throttle

Run at most once during a time window.

```js
function throttle(fn, interval) {
  let lastCall = 0;
  let timer;

  function throttled(...args) {
    const now = Date.now();
    const remaining = interval - (now - lastCall);

    if (remaining <= 0) {
      clearTimeout(timer);
      timer = undefined;
      lastCall = now;
      fn.apply(this, args);
    } else if (!timer) {
      timer = setTimeout(() => {
        lastCall = Date.now();
        timer = undefined;
        fn.apply(this, args);
      }, remaining);
    }
  }

  throttled.cancel = () => {
    clearTimeout(timer);
    timer = undefined;
  };

  return throttled;
}
```

### When to use

**Debounce:**

- search input
- autosave
- validation after typing

**Throttle:**

- scroll
- resize
- pointer movement
- continuous browser events

### React consideration

Create stable debounced/throttled functions and clean them up when the component unmounts.

---

## 25. Explain prototypal inheritance vs class syntax.

JavaScript uses **prototype-based inheritance**.

Objects can delegate property lookup to another object through the prototype chain.

```js
const animal = {
  speak() {
    console.log("sound");
  }
};

const dog = Object.create(animal);

dog.speak();
```

### Classes

```js
class Animal {
  speak() {
    console.log("sound");
  }
}

class Dog extends Animal {}
```

Classes provide a cleaner syntax over JavaScript's prototype model.

### Important point

Classes do not replace prototypes. Class methods still live on prototypes.

### Performance

Do not claim "classes are always faster" or "prototypes are always faster."

Performance depends on:

- object shapes
- engine optimizations
- allocation patterns
- property access
- polymorphism
- actual workload

A senior engineer should **benchmark** instead of making blanket claims.

---

## 26. How do WeakMap and WeakSet help with memory management?

A `WeakMap` holds weak references to object keys.

```js
const metadata = new WeakMap();

function track(element) {
  metadata.set(element, {
    createdAt: Date.now()
  });
}
```

If the DOM element becomes unreachable elsewhere, its WeakMap entry can become eligible for garbage collection.

### Why not Map?

With:

```js
const map = new Map();
map.set(element, metadata);
```

the Map strongly references the element, potentially keeping it alive.

### Use cases

- DOM metadata
- object-associated caches
- private metadata
- framework internals

### WeakSet

Useful when you only need to know whether an object has been seen:

```js
const visited = new WeakSet();

function process(node) {
  if (visited.has(node)) return;

  visited.add(node);
}
```

### Important limitation

WeakMap is not iterable. This is intentional because garbage collection is nondeterministic.

---

## 27. Implement a deep clone that handles circular references.

A simple recursive clone fails for:

```js
const obj = {};
obj.self = obj;
```

because recursion never terminates.

Use `WeakMap` to track already-cloned objects:

```js
function deepClone(value, seen = new WeakMap()) {
  if (value === null || typeof value !== "object") {
    return value;
  }

  if (seen.has(value)) {
    return seen.get(value);
  }

  if (value instanceof Date) {
    return new Date(value);
  }

  if (value instanceof RegExp) {
    return new RegExp(value.source, value.flags);
  }

  if (Array.isArray(value)) {
    const result = [];
    seen.set(value, result);

    for (const item of value) {
      result.push(deepClone(item, seen));
    }

    return result;
  }

  const result = Object.create(
    Object.getPrototypeOf(value)
  );

  seen.set(value, result);

  for (const key of Reflect.ownKeys(value)) {
    const descriptor = Object.getOwnPropertyDescriptor(
      value,
      key
    );

    if ("value" in descriptor) {
      descriptor.value = deepClone(
        descriptor.value,
        seen
      );
    }

    Object.defineProperty(result, key, descriptor);
  }

  return result;
}
```

### Limitations

A custom clone must decide how to handle:

- functions
- DOM nodes
- WeakMap
- WeakSet
- Map
- Set
- class instances
- private fields
- platform-specific objects

For many applications, `structuredClone()` is preferable when its supported data model matches your requirements.

---

## 28. Explain closures and common pitfalls.

A closure occurs when a function retains access to variables from its lexical scope even after the outer function has returned.

```js
function createCounter() {
  let count = 0;

  return () => {
    count++;
    return count;
  };
}

const counter = createCounter();

counter(); // 1
counter(); // 2
```

The returned function retains access to `count`.

### Loop pitfall

With `var`:

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 0);
}
```

The callbacks share the same function-scoped `i`, so the result is typically:

```text
3
3
3
```

Using `let` creates a new binding for each iteration:

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 0);
}
```

Result:

```text
0
1
2
```

### React closure issue

Event handlers and effects capture values from their render.

This can produce stale closures if dependencies are incorrect.

Senior engineers should understand that many React "state bugs" are really **JavaScript closure + rendering model** issues.

---

## 29. How would you implement a reactive state management system?

A basic reactive system needs:

1. state
2. dependency tracking
3. subscriptions
4. updates
5. batching

Conceptually:

```js
let activeEffect = null;

const deps = new WeakMap();

function track(target, key) {
  if (!activeEffect) return;

  let targetDeps = deps.get(target);

  if (!targetDeps) {
    targetDeps = new Map();
    deps.set(target, targetDeps);
  }

  let keyDeps = targetDeps.get(key);

  if (!keyDeps) {
    keyDeps = new Set();
    targetDeps.set(key, keyDeps);
  }

  keyDeps.add(activeEffect);
}

function trigger(target, key) {
  const targetDeps = deps.get(target);
  const keyDeps = targetDeps?.get(key);

  keyDeps?.forEach(effect => effect());
}
```

A Proxy can intercept property access:

```js
function reactive(target) {
  return new Proxy(target, {
    get(obj, key, receiver) {
      track(obj, key);
      return Reflect.get(obj, key, receiver);
    },

    set(obj, key, value, receiver) {
      const result = Reflect.set(
        obj,
        key,
        value,
        receiver
      );

      trigger(obj, key);

      return result;
    }
  });
}
```

### Batching

Without batching:

```text
state.a = 1 → render
state.b = 2 → render
state.c = 3 → render
```

A batched system can schedule one update:

```text
state.a = 1
state.b = 2
state.c = 3
      ↓
single scheduled update
```

### Senior-level concerns

A production state manager also needs to address:

- dependency cleanup
- nested objects
- computed values
- cycles
- batching
- selectors
- subscription lifecycle
- concurrent updates
- error handling

---

## 30. structuredClone vs JSON vs manual deep copy

### `structuredClone`

```js
const copy = structuredClone(original);
```

Good for many built-in structured data types and circular references.

It handles significantly more cases than JSON serialization.

### JSON approach

```js
const copy = JSON.parse(
  JSON.stringify(original)
);
```

Simple but lossy.

Potential problems include:

- `undefined`
- functions
- `Symbol`
- special numeric values
- `Date` becoming strings
- prototype information
- circular references

It should not be treated as a general-purpose deep-cloning solution.

### Manual clone

Useful when you need precise domain-specific behavior.

For example, you may want to preserve:

- class instances
- custom prototypes
- property descriptors
- special application-specific objects

But implementing a correct general-purpose clone is difficult.

### Comparison

| Approach | Circular refs | Date | Map/Set | Functions | Custom control |
|---|---:|---:|---:|---:|---:|
| `structuredClone` | Yes | Yes | Yes | No | Medium |
| JSON | No | Loses type | No | No | Low |
| Manual | Depends | Depends | Depends | Depends | High |

### Senior-level answer

Use `structuredClone()` when its supported data model fits the problem. Use JSON only for intentionally JSON-compatible data transformations. Use a custom clone only when you have a specific requirement that standard cloning does not satisfy.

---

# Senior-Level Interview Strategy

For a 7+ years frontend interview, knowing definitions is not enough.

The interviewer is usually evaluating whether you can make good decisions under real production constraints.

## 1. Always Explain Trade-offs

Instead of saying:

> "I would use `useMemo`."

Say:

> "I would first verify that the calculation or child rendering is actually expensive. If profiling shows a meaningful cost and the dependencies are stable, I would consider `useMemo`. Otherwise I would prefer simpler code."

That demonstrates engineering judgment.

---

## 2. Connect Answers to Production

Whenever possible, structure your answer like:

```text
Concept
   ↓
Why it matters
   ↓
Implementation
   ↓
Trade-offs
   ↓
Production example
   ↓
Monitoring/debugging
```

---

## 3. Performance Questions

Be ready to discuss:

- LCP
- INP
- CLS
- React Profiler
- browser Performance panel
- bundle analysis
- code splitting
- lazy loading
- image optimization
- server/client boundaries
- caching
- streaming
- memoization
- virtualization

---

## 4. Security Questions

For senior roles, discuss security beyond authentication.

Know:

- authentication vs authorization
- XSS
- CSRF
- IDOR
- secure cookies
- input validation
- output encoding
- Content Security Policy
- rate limiting
- secrets management
- Server Actions security
- least privilege

---

## 5. Architecture Questions

A strong answer should mention:

### Scalability

How does the solution behave when the application becomes 10x larger?

### Maintainability

Can another team understand and modify it?

### Performance

What happens to:

- bundle size?
- server response?
- browser CPU?
- memory?
- network traffic?

### Reliability

What happens when:

- APIs timeout?
- a dependency fails?
- cache becomes stale?
- users submit duplicate requests?

### Observability

How will you know the system is failing?

Think about:

- logs
- metrics
- tracing
- frontend monitoring
- Core Web Vitals
- error tracking

---

# Quick Senior-Level Answer Framework

When an interviewer asks a difficult architecture question, use:

## C — Context

Clarify the requirements.

## O — Options

Present 2–3 possible approaches.

## T — Trade-offs

Explain the advantages and disadvantages.

## D — Decision

Choose one and explain why.

## P — Production

Explain how you would monitor and roll it out.

Example:

> "For this application I would initially use Server Components for data-heavy, non-interactive content and isolate interactive pieces as Client Components. The alternative is making the entire page client-side, but that increases JavaScript and can hurt startup performance. I would validate the decision using bundle size, Core Web Vitals, and production telemetry."

---

# Final Preparation Checklist

Before a senior frontend interview, make sure you can explain these without memorizing a script:

- [ ] React reconciliation
- [ ] Fiber and concurrent rendering
- [ ] Server Components
- [ ] Suspense
- [ ] `use`, `useEffect`, `useLayoutEffect`
- [ ] React Compiler
- [ ] Context optimization
- [ ] State management architecture
- [ ] Hydration
- [ ] Optimistic updates
- [ ] Next.js App Router
- [ ] Server Actions
- [ ] Authentication and authorization
- [ ] Cache layers
- [ ] Cache invalidation
- [ ] Streaming
- [ ] PPR
- [ ] Middleware
- [ ] `after()`
- [ ] Core Web Vitals
- [ ] Bundle optimization
- [ ] Pages → App Router migration
- [ ] Event loop
- [ ] Promises
- [ ] Closures
- [ ] Prototypes
- [ ] WeakMap/WeakSet
- [ ] Debounce/throttle
- [ ] Deep cloning
- [ ] Reactive systems

---

# Important Note on Version-Specific Next.js Topics

Next.js changes rapidly. Topics such as **caching defaults, Partial Prerendering, Server Actions, `after()`, and App Router behavior** should always be checked against the exact Next.js version used by the company.

For an interview, it is better to say:

> "This behavior depends on the Next.js version. In the version I'm working with, the relevant behavior is X, and I would verify the current framework documentation before relying on a specific default."

That demonstrates senior-level awareness rather than relying on outdated framework behavior.
