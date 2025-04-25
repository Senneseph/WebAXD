# WebAX Testing Framework Specification

## Overview

The WebAX Testing Framework provides a comprehensive solution for testing WebAX applications. It is designed to be simple, extensible, and integrated with the WebAX ecosystem. The framework is part of the `basic-freedom` library and can be imported as:

```javascript
import { testing } from "basic-freedom";
```

## Design Principles

1. **Simplicity**: The API should be simple and intuitive
2. **Extensibility**: The framework should be extensible to support different testing needs
3. **Integration**: The framework should integrate seamlessly with the WebAX ecosystem
4. **Performance**: The framework should be performant and efficient
5. **Isolation**: Tests should run in isolation to prevent side effects
6. **Readability**: Test code should be readable and maintainable

## Core Components

The testing framework consists of several core components:

### 1. Test Runner

The test runner is responsible for executing tests and reporting results. It supports:

- Running tests in parallel or sequentially
- Filtering tests by name, tag, or file
- Reporting test results in various formats
- Watching files for changes and re-running tests

### 2. Assertion Library

The assertion library provides a set of assertions for validating test conditions. It supports:

- Basic assertions (equality, truthiness, etc.)
- Type assertions
- Exception assertions
- Asynchronous assertions
- Custom assertions

### 3. Mocking Library

The mocking library provides tools for creating test doubles (mocks, stubs, spies). It supports:

- Function mocks
- Object mocks
- Class mocks
- API mocks
- Event mocks

### 4. UI Testing

The UI testing module provides tools for testing WebAX UI components. It supports:

- Rendering components in isolation
- Simulating user interactions
- Asserting component state
- Testing accessibility
- Testing performance

### 5. State Testing

The state testing module provides tools for testing WebAX state management. It supports:

- Testing state transitions
- Testing selectors
- Testing actions
- Testing reducers
- Testing middleware

### 6. API Testing

The API testing module provides tools for testing WebAX API integration. It supports:

- Mocking API responses
- Testing API requests
- Testing error handling
- Testing caching
- Testing offline behavior

### 7. End-to-End Testing

The end-to-end testing module provides tools for testing WebAX applications as a whole. It supports:

- Browser automation
- User flow testing
- Visual regression testing
- Performance testing
- Accessibility testing

## API Design

### Test Definition

Tests are defined using a simple, declarative API:

```javascript
import { test, describe, beforeEach, afterEach } from "basic-freedom/testing";

describe("Calculator", () => {
  let calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  test("adds two numbers", () => {
    const result = calculator.add(2, 3);
    expect(result).toBe(5);
  });

  test("subtracts two numbers", () => {
    const result = calculator.subtract(5, 3);
    expect(result).toBe(2);
  });

  afterEach(() => {
    calculator = null;
  });
});
```

### Assertions

Assertions are made using a fluent, chainable API:

```javascript
import { expect } from "basic-freedom/testing";

// Basic assertions
expect(value).toBe(expected);
expect(value).toEqual(expected);
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();
expect(value).toBeNaN();

// Type assertions
expect(value).toBeType("string");
expect(value).toBeInstanceOf(Class);

// Numeric assertions
expect(value).toBeGreaterThan(expected);
expect(value).toBeGreaterThanOrEqual(expected);
expect(value).toBeLessThan(expected);
expect(value).toBeLessThanOrEqual(expected);
expect(value).toBeCloseTo(expected, precision);

// String assertions
expect(value).toMatch(pattern);
expect(value).toStartWith(prefix);
expect(value).toEndWith(suffix);
expect(value).toContain(substring);

// Array assertions
expect(array).toContain(item);
expect(array).toContainEqual(item);
expect(array).toHaveLength(length);

// Object assertions
expect(object).toHaveProperty(path, value);
expect(object).toMatchObject(partial);
expect(object).toStrictEqual(expected);

// Exception assertions
expect(() => fn()).toThrow();
expect(() => fn()).toThrowError(error);
expect(() => fn()).toThrowErrorMatchingSnapshot();

// Async assertions
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();
```

### Mocks

Mocks are created using a simple, functional API:

```javascript
import { mock, spy, stub } from "basic-freedom/testing";

// Function mock
const mockFn = mock();
mockFn.mockReturnValue(42);
mockFn.mockImplementation((a, b) => a + b);
mockFn.mockResolvedValue(42);
mockFn.mockRejectedValue(new Error("error"));

// Function spy
const spyFn = spy(obj, "method");
spyFn.mockReturnValue(42);

// Object stub
const stubObj = stub({
  method1: () => 42,
  method2: (a, b) => a + b,
});

// Verification
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith(1, 2);
expect(mockFn).toHaveBeenCalledTimes(2);
```

### UI Testing

UI components are tested using a component-specific API:

```javascript
import { render, fireEvent, screen } from "basic-freedom/testing/ui";

test("Button click increments counter", () => {
  // Render the component
  render(<Counter />);

  // Get elements
  const button = screen.getByText("Increment");
  const counter = screen.getByTestId("counter");

  // Initial state
  expect(counter).toHaveTextContent("0");

  // Simulate user interaction
  fireEvent.click(button);

  // Updated state
  expect(counter).toHaveTextContent("1");
});
```

### State Testing

State management is tested using a state-specific API:

```javascript
import { createTestStore } from "basic-freedom/testing/state";

test("counter reducer increments state", () => {
  // Create a test store
  const store = createTestStore({
    reducer: counterReducer,
    initialState: { count: 0 },
  });

  // Dispatch an action
  store.dispatch({ type: "INCREMENT" });

  // Assert state
  expect(store.getState()).toEqual({ count: 1 });
});
```

### API Testing

API integration is tested using an API-specific API:

```javascript
import { mockApi } from "basic-freedom/testing/api";

test("fetches user data", async () => {
  // Mock API response
  mockApi.get("/api/users/1").reply(200, {
    id: 1,
    name: "John Doe",
  });

  // Call the function that uses the API
  const user = await fetchUser(1);

  // Assert result
  expect(user).toEqual({
    id: 1,
    name: "John Doe",
  });

  // Verify API call
  expect(mockApi.get).toHaveBeenCalledWith("/api/users/1");
});
```

### End-to-End Testing

End-to-end tests are written using a browser automation API:

```javascript
import { browser } from "basic-freedom/testing/e2e";

test("user login flow", async () => {
  // Navigate to the login page
  await browser.goto("/login");

  // Fill in the login form
  await browser.fill("input[name=username]", "john");
  await browser.fill("input[name=password]", "password");

  // Submit the form
  await browser.click("button[type=submit]");

  // Wait for navigation
  await browser.waitForNavigation();

  // Assert that we're on the dashboard
  expect(browser.url()).toContain("/dashboard");

  // Assert that the user is logged in
  expect(await browser.text("h1")).toBe("Welcome, John!");
});
```

## Integration with WebAX

The testing framework integrates seamlessly with the WebAX ecosystem:

### 1. jcule Integration

The framework provides special utilities for testing jcule Business Logic Controllers:

```javascript
import { createTestBusinessLogic } from "basic-freedom/testing/jcule";

test("business logic handles user login", () => {
  // Create a test business logic controller
  const blc = createTestBusinessLogic({
    // Configuration
  });

  // Dispatch an action
  blc.dispatch({
    type: "LOGIN",
    payload: {
      username: "john",
      password: "password",
    },
  });

  // Assert state
  expect(blc.getState().user).toEqual({
    id: 1,
    name: "John Doe",
  });
});
```

### 2. jellicule Integration

The framework provides special utilities for testing jellicule UI components:

```javascript
import { render, fireEvent } from "basic-freedom/testing/jellicule";

test("jellicule button emits click event", () => {
  // Render the component
  const { component, events } = render("jel-button", {
    props: {
      label: "Click me",
    },
  });

  // Simulate user interaction
  fireEvent.click(component);

  // Assert events
  expect(events).toContainEvent("click");
});
```

### 3. WebAXP Integration

The framework provides special utilities for testing WebAXP applications:

```javascript
import { createTestApp } from "basic-freedom/testing/webaxp";

test("WebAXP application renders correctly", () => {
  // Create a test application
  const app = createTestApp({
    // Configuration
  });

  // Navigate to a route
  app.navigate("/products");

  // Assert rendered components
  expect(app.getRenderedComponents()).toContain("ProductList");
});
```

## Test Organization

Tests are organized using a simple, file-based structure:

```
src/
  components/
    Button.tsx
    Button.test.tsx
  state/
    counter.ts
    counter.test.ts
  api/
    users.ts
    users.test.ts
  pages/
    Home.tsx
    Home.test.tsx
  e2e/
    login.test.ts
```

## Configuration

The testing framework is configured using a configuration file:

```javascript
// basic-freedom.config.js
module.exports = {
  testing: {
    // Test runner configuration
    runner: {
      parallel: true,
      watch: true,
      coverage: true,
    },
    // Reporter configuration
    reporter: {
      format: "detailed",
      output: "reports/test-results.json",
    },
    // Browser configuration
    browser: {
      headless: true,
      width: 1280,
      height: 720,
    },
  },
};
```

## CLI

The testing framework includes a command-line interface:

```bash
# Run all tests
basic-freedom test

# Run specific tests
basic-freedom test src/components

# Run tests in watch mode
basic-freedom test --watch

# Run tests with coverage
basic-freedom test --coverage

# Run end-to-end tests
basic-freedom test:e2e
```

## Plugins

The testing framework is extensible through plugins:

```javascript
// basic-freedom.config.js
module.exports = {
  testing: {
    plugins: [
      // Custom reporter
      {
        name: "custom-reporter",
        reporter: {
          onTestStart: (test) => {
            console.log(`Starting test: ${test.name}`);
          },
          onTestEnd: (test, result) => {
            console.log(`Test ${test.name} ${result.status}`);
          },
        },
      },
      // Custom assertion
      {
        name: "custom-assertion",
        assertion: {
          toBeValidEmail: (received) => {
            const pass = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(received);
            return {
              pass,
              message: () =>
                pass
                  ? `Expected ${received} not to be a valid email`
                  : `Expected ${received} to be a valid email`,
            };
          },
        },
      },
    ],
  },
};
```

## Implementation Considerations

### 1. Performance

The testing framework is designed to be performant:

- Tests run in parallel by default
- Test files are loaded on demand
- Test results are cached
- Coverage data is collected incrementally

### 2. Compatibility

The testing framework is compatible with:

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Node.js environments
- WebAX applications
- TypeScript projects
- JavaScript projects

### 3. Extensibility

The testing framework is extensible through:

- Plugins
- Custom assertions
- Custom matchers
- Custom reporters
- Custom test runners

### 4. Integration

The testing framework integrates with:

- CI/CD systems
- Code editors
- Build tools
- Coverage tools
- Linting tools

## Conclusion

The WebAX Testing Framework provides a comprehensive solution for testing WebAX applications. It is designed to be simple, extensible, and integrated with the WebAX ecosystem. By following this specification, developers can create high-quality, maintainable tests for their WebAX applications.
