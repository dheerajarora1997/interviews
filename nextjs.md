# Next.js Interview Questions and Answers

## ✅ Basics (Beginner Level)

### 1. What is Next.js and why use it over React?

**Answer:** Next.js is a React framework that provides server-side rendering, static site generation, and other features out of the box. It simplifies development by handling routing, code splitting, and performance optimization. Compared to React alone, Next.js offers better SEO and faster initial load.

### 2. What are the main features of Next.js?

**Answer:**

* Server-side rendering (SSR)
* Static site generation (SSG)
* Incremental Static Regeneration (ISR)
* API Routes
* Image Optimization
* Built-in CSS and Sass support
* File-based routing
* Middleware support
* TypeScript support
* Internationalization (i18n)

### 3. How does server-side rendering (SSR) work in Next.js?

**Answer:** In SSR, Next.js renders the HTML on the server on every request using `getServerSideProps`. This improves SEO and provides dynamic content.

### 4. What is the difference between SSR, SSG, ISR, and CSR in Next.js?

**Answer:**

* **SSR (Server-side rendering):** Renders page at request time.
* **SSG (Static Site Generation):** Renders at build time.
* **ISR (Incremental Static Regeneration):** Combines SSG and SSR by updating static pages after deployment.
* **CSR (Client-side rendering):** Renders content on the client using JavaScript.

### 5. How do you create dynamic routes in Next.js?

**Answer:** Use square brackets in the `pages` folder, e.g., `pages/post/[id].js`. Use `getStaticPaths` with `getStaticProps` or `getServerSideProps` to fetch data.

### 6. What is the purpose of the `pages` directory in Next.js?

**Answer:** It defines the routes of the application. Each file in the `pages` directory corresponds to a route based on its file name.

### 7. What is the role of the `_app.js` and `_document.js` files?

**Answer:**

* `_app.js`: Customizes and persists layout between pages.
* `_document.js`: Used for augmenting the HTML document structure. Useful for adding language, metadata, or scripts.

### 8. How do you use environment variables in Next.js?

**Answer:** Define variables in `.env.local`, then access with `process.env.VAR_NAME`. Variables prefixed with `NEXT_PUBLIC_` are available on the client.

### 9. How do you add custom metadata (title, description) for SEO in a page?

**Answer:** Use the `Head` component from `next/head`:

```jsx
import Head from 'next/head';

<Head>
  <title>My Page</title>
  <meta name="description" content="Page description here" />
</Head>
```

### 10. How do you handle navigation in Next.js?

**Answer:** Use the `Link` component from `next/link`:

```jsx
import Link from 'next/link';
<Link href="/about">About</Link>
```

---

## ⚙️ Intermediate Level

### 11. What is API routing in Next.js and how do you create API routes?

**Answer:** Create a file in `pages/api`, e.g., `pages/api/hello.js`. It exports a function handling requests and responses like an Express endpoint.

### 12. How does `getStaticProps` differ from `getServerSideProps` and `getStaticPaths`?

**Answer:**

* `getStaticProps`: Runs at build time, for static generation.
* `getServerSideProps`: Runs on each request, for dynamic content.
* `getStaticPaths`: Defines dynamic routes to be generated at build time with `getStaticProps`.

### 13. Explain Image Optimization in Next.js using `next/image`.

**Answer:** The `next/image` component provides automatic image resizing, optimization, lazy loading, and WebP support.

```jsx
import Image from 'next/image';
<Image src="/me.jpg" width={500} height={500} alt="Me" />
```

### 14. How do you handle authentication in Next.js apps?

**Answer:** Use libraries like `next-auth`, `Auth0`, or JWT. Middleware or API routes can protect backend logic, while session data can be handled with React context or hooks.

### 15. What are middleware functions in Next.js (introduced in v12+)?

**Answer:** Middleware runs before a request is completed. It can be used to redirect, rewrite, or modify requests/responses. Placed in `middleware.js`.

### 16. How do you configure redirects and rewrites in Next.js?

**Answer:** Add to `next.config.js`:

```js
module.exports = {
  async redirects() {
    return [
      { source: '/old', destination: '/new', permanent: true },
    ];
  },
};
```

### 17. What are the differences between Next.js and CRA (Create React App)?

**Answer:**

* CRA is for client-side apps only; Next.js supports SSR and SSG.
* Next.js has routing and performance features out of the box.
* CRA is simpler; Next.js is more powerful and production-ready.

### 18. Explain Incremental Static Regeneration (ISR).

**Answer:** ISR allows static pages to be updated after deployment. You specify a `revalidate` time in `getStaticProps`.

```js
export async function getStaticProps() {
  return {
    props: {},
    revalidate: 60, // seconds
  };
}
```

### 19. What is `next/head` and why is it used?

**Answer:** It is used to modify the HTML `<head>` for each page, such as setting the title, meta tags, and link tags.

### 20. How would you implement internationalization (i18n) in Next.js?

**Answer:** Configure locales in `next.config.js`:

```js
module.exports = {
  i18n: {
    locales: ['en', 'fr'],
    defaultLocale: 'en',
  },
};
```

Use `useRouter().locale` and route accordingly.

---

## 🚀 Advanced Level

### 21. Explain how Next.js handles performance optimization.

**Answer:**

* Code splitting
* Lazy loading
* Static generation
* Image optimization
* Prefetching
* Edge rendering
* Compression

### 22. How does Next.js support edge functions and serverless functions?

**Answer:** You can use `pages/api` for serverless functions and `middleware` for edge functions, deployed on the edge network (e.g., via Vercel).

### 23. What’s the role of the `next.config.js` file?

**Answer:** It configures settings like redirects, rewrites, environment variables, Webpack customizations, and more.

### 24. How does Next.js handle static file serving?

**Answer:** Static assets (images, fonts, etc.) are placed in the `public` folder and accessed directly via `/filename`.

### 25. How would you optimize a large Next.js app for faster performance?

**Answer:**

* Use static generation where possible
* Optimize images with `next/image`
* Implement lazy loading
* Analyze bundle size with `next build`
* Use middleware to reduce server load
* Minimize third-party scripts

### 26. How would you set up analytics in a Next.js application?

**Answer:** Integrate tools like Google Analytics by adding script tags in `_app.js` or using libraries like `next-gtag` or `nextjs-google-analytics`.

### 27. How does Next.js 13+ (App Router) differ from the traditional Pages Router?

**Answer:**

* Uses `app/` instead of `pages/`
* Supports React Server Components
* Introduces new conventions like `layout.js`, `loading.js`, and `error.js`
* Built-in support for nested routing

### 28. What are React Server Components and how are they used in Next.js?

**Answer:** RSCs allow components to render on the server without sending their JS to the client. Improves performance by reducing client bundle size. Used in `app/` directory with server-only logic.

### 29. Explain the use of `useRouter()` hook.

**Answer:** Provided by `next/router`, it lets you access route parameters, query strings, push/replace navigation programmatically.

```js
import { useRouter } from 'next/router';
const router = useRouter();
```

### 30. How would you deploy a Next.js application to Vercel / custom server?

**Answer:**

* **Vercel:** Connect GitHub repo, Vercel auto-detects and deploys.
* **Custom server:** Build using `next build`, then use `next start` or integrate with Node.js/Express for custom routing.
