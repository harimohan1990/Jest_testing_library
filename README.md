# Jest_testing_library

Here's a deep guide covering **Jest Testing Library** with **React.js**, structured to walk through each core topic:

---

## ✅ 1. Getting Started

### Install Jest and React Testing Library:

```bash
npm install --save-dev jest @testing-library/react @testing-library/jest-dom
```

### Setup `jest.config.js`:

```js
export default {
  testEnvironment: "jsdom",
  moduleFileExtensions: ["js", "jsx", "ts", "tsx"],
  transform: {
    "^.+\\.(js|jsx|ts|tsx)$": "babel-jest",
  },
};
```

### Basic Test Example:

```js
// MyComponent.jsx
export const MyComponent = () => <h1>Hello World</h1>;

// MyComponent.test.jsx
import { render, screen } from "@testing-library/react";
import { MyComponent } from "./MyComponent";

test("renders Hello World", () => {
  render(<MyComponent />);
  expect(screen.getByText("Hello World")).toBeInTheDocument();
});
```

---

## ✅ 2. Using Matchers

Extend with `@testing-library/jest-dom` for better matchers:

```js
import "@testing-library/jest-dom";

expect(element).toBeInTheDocument();
expect(button).toHaveTextContent("Submit");
expect(input).toHaveValue("test");
expect(spinner).not.toBeVisible();
```

---

## ✅ 3. Testing Asynchronous Code

### Example: Wait for async data

```js
test("loads user data", async () => {
  render(<User />);
  expect(await screen.findByText("John Doe")).toBeInTheDocument();
});
```

Use:

* `findBy*` for promises
* `waitFor(() => ...)` to await conditions

---

## ✅ 4. Setup and Teardown

### Global setup: `setupTests.js`

```js
import "@testing-library/jest-dom";
```

### Using `beforeEach` and `afterEach`:

```js
beforeEach(() => {
  // common setup
});
afterEach(() => {
  // cleanup
});
```

---

## ✅ 5. Mock Functions

```js
const mockFn = jest.fn();
mockFn("first");
expect(mockFn).toHaveBeenCalledWith("first");
```

### Mock API or module:

```js
jest.mock("axios");
axios.get.mockResolvedValue({ data: { user: "John" } });
```

### Mock Components:

```js
jest.mock("./HeavyComponent", () => () => <div>Mocked!</div>);
```

---

## ✅ 6. Jest Platform Features

* **Watch mode**: `jest --watch`
* **Code coverage**: `jest --coverage`
* **Snapshot testing**: `expect(tree).toMatchSnapshot()`

---

## ✅ 7. Jest Community & Ecosystem

* **Expect matchers**: [jest-extended](https://github.com/jest-community/jest-extended)
* **Mocking tools**: [msw](https://mswjs.io) for API mocking
* **Visual regression**: [jest-image-snapshot](https://github.com/americanexpress/jest-image-snapshot)

---

## ✅ 8. Guides

### React-specific guide:

* Prefer `screen` over destructuring returns from `render`
* Use `userEvent` for simulating real user actions:

```js
import userEvent from "@testing-library/user-event";
await userEvent.click(button);
await userEvent.type(input, "Hello");
```

---

## ✅ 9. Framework Integration

* **Next.js**: use `@testing-library/react`, `jest`, and `msw`
* **Create React App**: Jest comes pre-configured
* **Vite**: configure `vite.config.ts` + `vitest` instead (Jest alternative)

---

## ✅ 10. Upgrade Guides & Resources

* [Jest Official Docs](https://jestjs.io/docs/getting-started)
* [Testing Library Docs](https://testing-library.com/docs/)
* [React Testing Best Practices](https://testing-library.com/docs/react-testing-library/intro/)
* Upgrade Notes: check GitHub changelogs or use `jest --updateSnapshot` when needed.

---

Let's test an **API call in a React component** using **Jest** and **React Testing Library**, with **mocking via `msw`** (Mock Service Worker) — the most robust and scalable approach.

---

### 🧩 Component Example: `UserProfile.jsx`

```jsx
import { useEffect, useState } from "react";

export const UserProfile = () => {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch("/api/user")
      .then(res => res.json())
      .then(data => setUser(data));
  }, []);

  if (!user) return <p>Loading...</p>;

  return <h1>{user.name}</h1>;
};
```

---

### ✅ Install Testing Tools

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom msw
```

---

### 🧪 Test File: `UserProfile.test.jsx`

```jsx
import { render, screen } from "@testing-library/react";
import { rest } from "msw";
import { setupServer } from "msw/node";
import { UserProfile } from "./UserProfile";

// Setup mock server
const server = setupServer(
  rest.get("/api/user", (req, res, ctx) => {
    return res(ctx.json({ name: "John Doe" }));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test("displays user name from API", async () => {
  render(<UserProfile />);
  expect(await screen.findByText("John Doe")).toBeInTheDocument();
});
```

---

### 💡 Why MSW?

* Intercepts real HTTP calls (REST/GraphQL)
* Works in browser and Node.js
* Isolates your test from backend

Here are **most commonly used test cases for API calls in React** using Jest + Testing Library + MSW:

---

### ✅ 1. **Success State** – API returns data

```jsx
test("shows user name from API", async () => {
  server.use(
    rest.get("/api/user", (req, res, ctx) => {
      return res(ctx.json({ name: "Alice" }));
    })
  );
  render(<UserProfile />);
  expect(await screen.findByText("Alice")).toBeInTheDocument();
});
```

---

### ✅ 2. **Loading State**

```jsx
test("shows loading initially", () => {
  render(<UserProfile />);
  expect(screen.getByText("Loading...")).toBeInTheDocument();
});
```

---

### ✅ 3. **Error State** – API fails

```jsx
test("handles error state", async () => {
  server.use(
    rest.get("/api/user", (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  render(<UserProfile />);
  expect(await screen.findByText(/error/i)).toBeInTheDocument(); // assuming component handles error
});
```

---

### ✅ 4. **Empty Response / Fallback**

```jsx
test("handles empty user response", async () => {
  server.use(
    rest.get("/api/user", (req, res, ctx) => {
      return res(ctx.json({}));
    })
  );
  render(<UserProfile />);
  expect(await screen.findByText(/no user/i)).toBeInTheDocument(); // if fallback exists
});
```

---

### ✅ 5. **Retry Mechanism**

If your component retries failed calls:

```jsx
test("retries on failure", async () => {
  let attempt = 0;
  server.use(
    rest.get("/api/user", (req, res, ctx) => {
      attempt++;
      return attempt < 2
        ? res(ctx.status(500))
        : res(ctx.json({ name: "Retry User" }));
    })
  );
  render(<UserProfile />);
  expect(await screen.findByText("Retry User")).toBeInTheDocument();
});
```

Here’s how to test **API calls using React Query, Redux Toolkit Query, and Axios**—all commonly used in production React apps.

---

## ✅ 1. React Query + Jest + MSW

### 🧩 Component

```jsx
import { useQuery } from "@tanstack/react-query";

export const UserProfile = () => {
  const { data, isLoading, error } = useQuery(["user"], () =>
    fetch("/api/user").then(res => res.json())
  );

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  return <h1>{data.name}</h1>;
};
```

### 🧪 Test

```jsx
import { render, screen } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { rest } from "msw";
import { setupServer } from "msw/node";
import { UserProfile } from "./UserProfile";

const server = setupServer(
  rest.get("/api/user", (req, res, ctx) => res(ctx.json({ name: "Alice" })))
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

const renderWithClient = (ui) =>
  render(
    <QueryClientProvider client={new QueryClient()}>
      {ui}
    </QueryClientProvider>
  );

test("React Query fetch success", async () => {
  renderWithClient(<UserProfile />);
  expect(await screen.findByText("Alice")).toBeInTheDocument();
});
```

---

## ✅ 2. Redux Toolkit Query (RTK Query)

### 🧩 API Setup

```ts
// services/api.ts
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

export const userApi = createApi({
  reducerPath: "userApi",
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  endpoints: builder => ({
    getUser: builder.query({ query: () => "user" }),
  }),
});

export const { useGetUserQuery } = userApi;
```

### 🧩 Component

```jsx
import { useGetUserQuery } from "./services/api";

export const UserProfile = () => {
  const { data, isLoading, error } = useGetUserQuery();

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  return <h1>{data.name}</h1>;
};
```

### 🧪 Test

```jsx
import { configureStore } from "@reduxjs/toolkit";
import { Provider } from "react-redux";
import { userApi } from "./services/api";

const store = configureStore({
  reducer: { [userApi.reducerPath]: userApi.reducer },
  middleware: gDM => gDM().concat(userApi.middleware),
});

const renderWithStore = (ui) =>
  render(<Provider store={store}>{ui}</Provider>);

test("RTK Query fetch success", async () => {
  renderWithStore(<UserProfile />);
  expect(await screen.findByText("Alice")).toBeInTheDocument();
});
```

---

## ✅ 3. Axios + Jest (no React Query)

### 🧩 Component

```jsx
import { useEffect, useState } from "react";
import axios from "axios";

export const UserProfile = () => {
  const [user, setUser] = useState(null);
  useEffect(() => {
    axios.get("/api/user").then(res => setUser(res.data));
  }, []);
  return <>{user ? <h1>{user.name}</h1> : <p>Loading...</p>}</>;
};
```

### 🧪 Test

```jsx
import axios from "axios";
jest.mock("axios");

test("Axios fetch success", async () => {
  axios.get.mockResolvedValue({ data: { name: "Alice" } });
  render(<UserProfile />);
  expect(await screen.findByText("Alice")).toBeInTheDocument();
});
```

