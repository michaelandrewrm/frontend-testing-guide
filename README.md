# Comprehensive Testing Guide for Frontend Development

## Table of Contents

1. [Core Testing Principles](#core-testing-principles)
2. [Unit Tests vs Integration Tests](#unit-tests-vs-integration-tests)
3. [Test Structure and Organization](#test-structure-and-organization)
4. [Naming Conventions](#naming-conventions)
5. [Best Practices](#best-practices)
6. [Do's and Don'ts](#dos-and-donts)
7. [Common Pitfalls](#common-pitfalls)
8. [Vue 3 Specific Guidelines](#vue-3-specific-guidelines)
9. [Testing Patterns](#testing-patterns)
10. [Tools and Setup](#tools-and-setup)

---

## Core Testing Principles

### The Testing Pyramid

```
    /\
   /  \  E2E Tests (Few)
  /____\
 /      \ Integration Tests (Some)
/________\
Unit Tests (Many)
```

### Fundamental Principles

#### 1. **Test Behavior, Not Implementation**

```javascript
// ❌ BAD - Testing implementation details
it('should call setUsername method', () => {
  const setUsernameSpy = vi.spyOn(wrapper.vm, 'setUsername');
  wrapper.find('input').setValue('john');
  expect(setUsernameSpy).toHaveBeenCalled();
});

// ✅ GOOD - Testing behavior
it('should update username when input value changes', async () => {
  await wrapper.find('input').setValue('john');
  expect(wrapper.vm.username).toBe('john');
});
```

#### 2. **Arrange, Act, Assert (AAA Pattern)**

```javascript
it('should calculate total price with tax', () => {
  // Arrange
  const price = 100;
  const taxRate = 0.1;

  // Act
  const total = calculateTotalWithTax(price, taxRate);

  // Assert
  expect(total).toBe(110);
});
```

#### 3. **Single Responsibility**

Each test should verify one specific behavior or outcome.

```javascript
// ❌ BAD - Testing multiple things
it('should handle user registration', async () => {
  await wrapper.find('#email').setValue('test@example.com');
  await wrapper.find('#password').setValue('password123');
  await wrapper.find('#submit').trigger('click');

  expect(wrapper.vm.isLoading).toBe(false);
  expect(wrapper.vm.user.email).toBe('test@example.com');
  expect(wrapper.emitted('success')).toBeTruthy();
  expect(mockApi.register).toHaveBeenCalled();
});

// ✅ GOOD - Separate concerns
describe('User Registration', () => {
  it('should set loading state during registration', async () => {
    await submitRegistrationForm();
    expect(wrapper.vm.isLoading).toBe(true);
  });

  it('should emit success event on successful registration', async () => {
    mockApi.register.mockResolvedValue({ success: true });
    await submitRegistrationForm();
    expect(wrapper.emitted('success')).toBeTruthy();
  });

  it('should update user data after registration', async () => {
    await submitRegistrationForm();
    expect(wrapper.vm.user.email).toBe('test@example.com');
  });
});
```

---

## Unit Tests vs Integration Tests

### Unit Tests

**Purpose**: Test individual components, functions, or modules in isolation.

**Characteristics**:

- Fast execution
- Isolated from external dependencies
- Mock all external interactions
- Focus on single unit of code

```javascript
// Unit test example - Testing a pure function
describe('formatCurrency', () => {
  it('should format number as currency with two decimal places', () => {
    expect(formatCurrency(1234.5)).toBe('$1,234.50');
  });

  it('should handle zero values', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });

  it('should handle negative values', () => {
    expect(formatCurrency(-100)).toBe('-$100.00');
  });
});
```

### Integration Tests

**Purpose**: Test interaction between multiple components or systems.

**Characteristics**:

- Test component interactions
- May include real dependencies
- Slower than unit tests
- Test user workflows

```javascript
// Integration test example - Testing component interaction
describe('TodoApp Integration', () => {
  it('should add new todo and display it in the list', async () => {
    const wrapper = mount(TodoApp, {
      global: {
        plugins: [createTestingPinia()],
      },
    });

    // Add new todo
    await wrapper.find('[data-testid="todo-input"]').setValue('Buy groceries');
    await wrapper.find('[data-testid="add-button"]').trigger('click');

    // Verify it appears in the list
    const todoItems = wrapper.findAll('[data-testid="todo-item"]');
    expect(todoItems).toHaveLength(1);
    expect(todoItems[0].text()).toContain('Buy groceries');
  });
});
```

---

## Test Structure and Organization

### File Organization

```
src/
├── components/
│   ├── Button.vue
│   └── Button.spec.js          # Co-located tests
├── stores/
│   ├── user.js
│   └── user.spec.js
├── utils/
│   ├── formatting.js
│   └── formatting.spec.js
└── __tests__/                  # Global test utilities
    ├── helpers/
    ├── mocks/
    └── setup.js
```

### Test Suite Structure

```javascript
describe('ComponentName', () => {
  // Setup and teardown
  let wrapper;
  let mockStore;

  beforeEach(() => {
    mockStore = createMockStore();
    wrapper = mount(ComponentName, {
      global: { plugins: [mockStore] },
    });
  });

  afterEach(() => {
    wrapper.unmount();
  });

  // Group related tests
  describe('Rendering', () => {
    it('should render component with default props', () => {
      expect(wrapper.exists()).toBe(true);
    });
  });

  describe('User Interactions', () => {
    it('should handle click events', async () => {
      await wrapper.find('button').trigger('click');
      expect(wrapper.emitted('click')).toBeTruthy();
    });
  });

  describe('Store Integration', () => {
    it('should update store on user action', async () => {
      await wrapper.vm.updateUser('newName');
      expect(mockStore.updateUser).toHaveBeenCalledWith('newName');
    });
  });

  describe('Edge Cases', () => {
    it('should handle empty data gracefully', () => {
      wrapper = mount(ComponentName, {
        props: { data: [] },
      });
      expect(wrapper.find('.empty-state').exists()).toBe(true);
    });
  });
});
```

---

## Naming Conventions

### Test File Names

- **Component tests**: `ComponentName.spec.js` or `ComponentName.test.js`
- **Unit tests**: `functionName.spec.js`
- **Integration tests**: `featureName.integration.spec.js`

### Describe Blocks

```javascript
// ✅ GOOD - Clear hierarchy
describe('UserProfile', () => {
  describe('when user is authenticated', () => {
    describe('and has profile data', () => {
      it('should display user information', () => {});
    });
  });
});
```

### Test Names

Use descriptive names that explain the scenario and expected outcome:

```javascript
// ❌ BAD
it('should work', () => {});
it('test button click', () => {});

// ✅ GOOD
it('should emit "save" event when save button is clicked', () => {});
it('should display error message when email format is invalid', () => {});
it('should disable submit button when form is invalid', () => {});
```

### Pattern Templates

```javascript
// Pattern: should [expected behavior] when [condition]
it('should display loading spinner when data is being fetched', () => {});
it('should show error message when API request fails', () => {});

// Pattern: given [condition] should [expected behavior]
it('given invalid email should show validation error', () => {});

// Pattern: [action] should [expected result]
it('clicking submit should call onSubmit prop', () => {});
```

---

## Best Practices

### 1. **Use Data Test IDs**

```vue
<!-- Component template -->
<template>
  <div>
    <button data-testid="submit-button" @click="submit">Submit</button>
    <input data-testid="email-input" v-model="email" />
    <div data-testid="error-message" v-if="error">{{ error }}</div>
  </div>
</template>
```

```javascript
// Test
it('should show error when email is invalid', async () => {
  await wrapper.find('[data-testid="email-input"]').setValue('invalid-email');
  await wrapper.find('[data-testid="submit-button"]').trigger('click');

  expect(wrapper.find('[data-testid="error-message"]').text()).toBe('Please enter a valid email address');
});
```

### 2. **Create Reusable Test Utilities**

```javascript
// test-utils.js
export const createWrapper = (component, options = {}) => {
  const defaultOptions = {
    global: {
      plugins: [createTestingPinia()],
      stubs: ['router-link', 'router-view'],
    },
  };

  return mount(component, merge(defaultOptions, options));
};

export const findByTestId = (wrapper, testId) => {
  return wrapper.find(`[data-testid="${testId}"]`);
};

// Usage in tests
const wrapper = createWrapper(MyComponent, {
  props: { userId: '123' },
});
const submitButton = findByTestId(wrapper, 'submit-button');
```

### 3. **Mock External Dependencies**

```javascript
// Mock API calls
vi.mock('@/api/users', () => ({
  fetchUser: vi.fn(),
  updateUser: vi.fn(),
}));

// Mock router
const mockRouter = {
  push: vi.fn(),
  replace: vi.fn(),
  currentRoute: { value: { params: { id: '123' } } },
};

// Mock store
const createMockStore = (initialState = {}) => {
  return createTestingPinia({
    initialState: {
      user: {
        currentUser: null,
        isLoading: false,
        ...initialState,
      },
    },
  });
};
```

### 4. **Test Async Operations Properly**

```javascript
// ✅ GOOD - Proper async testing
it('should load user data on mount', async () => {
  const mockUser = { id: 1, name: 'John' };
  mockApi.fetchUser.mockResolvedValue(mockUser);

  const wrapper = createWrapper(UserProfile);

  // Wait for async operations to complete
  await flushPromises();

  expect(wrapper.vm.user).toEqual(mockUser);
  expect(wrapper.find('.user-name').text()).toBe('John');
});

// Handle loading states
it('should show loading spinner while fetching data', () => {
  mockApi.fetchUser.mockImplementation(() => new Promise(() => {})); // Never resolves

  const wrapper = createWrapper(UserProfile);

  expect(wrapper.find('.loading-spinner').exists()).toBe(true);
});
```

### 5. **Test Error Scenarios**

```javascript
it('should display error message when API call fails', async () => {
  const errorMessage = 'Network error';
  mockApi.fetchUser.mockRejectedValue(new Error(errorMessage));

  const wrapper = createWrapper(UserProfile);
  await flushPromises();

  expect(wrapper.find('.error-message').text()).toContain(errorMessage);
});
```

---

## Do's and Don'ts

### ✅ DO's

#### **DO: Test User-Facing Behavior**

```javascript
// Test what users see and interact with
it('should show welcome message after successful login', async () => {
  await login('user@example.com', 'password');
  expect(screen.getByText('Welcome back!')).toBeVisible();
});
```

#### **DO: Use Descriptive Assertions**

```javascript
// Be specific about what you're testing
expect(wrapper.find('.error-message').text()).toBe('Email is required');

expect(wrapper.vm.users).toHaveLength(3);
expect(wrapper.emitted('user-selected')[0][0]).toEqual({ id: 1, name: 'John' });
```

#### **DO: Group Related Tests**

```javascript
describe('Form Validation', () => {
  describe('Email Field', () => {
    it('should show error for empty email', () => {});
    it('should show error for invalid email format', () => {});
    it('should accept valid email', () => {});
  });
});
```

#### **DO: Test Edge Cases**

```javascript
describe('UserList', () => {
  it('should handle empty user list', () => {
    const wrapper = mount(UserList, { props: { users: [] } });
    expect(wrapper.find('.empty-state').exists()).toBe(true);
  });

  it('should handle very long user names', () => {
    const longName = 'a'.repeat(100);
    const wrapper = mount(UserList, {
      props: { users: [{ name: longName }] },
    });
    expect(wrapper.find('.user-name').classes()).toContain('truncated');
  });
});
```

### ❌ DON'Ts

#### **DON'T: Test Implementation Details**

```javascript
// ❌ BAD - Testing internal method calls
it('should call calculateTotal method', () => {
  const spy = vi.spyOn(wrapper.vm, 'calculateTotal');
  wrapper.vm.updateCart();
  expect(spy).toHaveBeenCalled();
});

// ✅ GOOD - Testing the result
it('should update total when cart is updated', () => {
  wrapper.vm.updateCart();
  expect(wrapper.vm.total).toBe(150);
});
```

#### **DON'T: Write Overly Complex Tests**

```javascript
// ❌ BAD - Too complex, hard to debug
it('should handle complex user workflow', async () => {
  // 50 lines of setup and interactions
  // Multiple assertions
  // Complex conditional logic
});

// ✅ GOOD - Break into smaller, focused tests
describe('User Checkout Flow', () => {
  it('should add item to cart', () => {});
  it('should calculate shipping cost', () => {});
  it('should process payment', () => {});
  it('should send confirmation email', () => {});
});
```

#### **DON'T: Use Real Dependencies in Unit Tests**

```javascript
// ❌ BAD - Real HTTP calls in unit tests
it('should fetch user data', async () => {
  const response = await fetch('/api/users/1'); // Real API call
  expect(response.ok).toBe(true);
});

// ✅ GOOD - Mock dependencies
it('should fetch user data', async () => {
  mockApi.fetchUser.mockResolvedValue({ id: 1, name: 'John' });
  const user = await fetchUser(1);
  expect(user.name).toBe('John');
});
```

#### **DON'T: Write Flaky Tests**

```javascript
// ❌ BAD - Time-dependent, unreliable
it('should show notification for 3 seconds', async () => {
  showNotification('Hello');
  setTimeout(() => {
    expect(wrapper.find('.notification').exists()).toBe(false);
  }, 3000);
});

// ✅ GOOD - Control timing explicitly
it('should hide notification after timeout', async () => {
  vi.useFakeTimers();
  showNotification('Hello');

  expect(wrapper.find('.notification').exists()).toBe(true);

  vi.advanceTimersByTime(3000);
  await wrapper.vm.$nextTick();

  expect(wrapper.find('.notification').exists()).toBe(false);
  vi.useRealTimers();
});
```

---

## Common Pitfalls

### 1. **Not Waiting for Async Operations**

```javascript
// ❌ PROBLEM
it('should update UI after API call', () => {
  wrapper.vm.loadData(); // Async operation
  expect(wrapper.find('.data').text()).toBe('Loaded data'); // Fails
});

// ✅ SOLUTION
it('should update UI after API call', async () => {
  await wrapper.vm.loadData();
  // or
  await flushPromises();
  // or
  await wrapper.vm.$nextTick();

  expect(wrapper.find('.data').text()).toBe('Loaded data');
});
```

### 2. **Shared State Between Tests**

```javascript
// ❌ PROBLEM - State persists between tests
let userData = { name: 'John' };

it('should update user name', () => {
  userData.name = 'Jane';
  expect(userData.name).toBe('Jane');
});

it('should have original name', () => {
  // This fails because userData.name is still 'Jane'
  expect(userData.name).toBe('John');
});

// ✅ SOLUTION - Fresh state for each test
beforeEach(() => {
  userData = { name: 'John' };
});
```

### 3. **Testing Too Many Things at Once**

```javascript
// ❌ PROBLEM
it('should handle user registration and login flow', async () => {
  // Register user
  await wrapper.find('#email').setValue('test@example.com');
  await wrapper.find('#password').setValue('password');
  await wrapper.find('#register').trigger('click');

  // Login user
  await wrapper.find('#login-email').setValue('test@example.com');
  await wrapper.find('#login-password').setValue('password');
  await wrapper.find('#login').trigger('click');

  // Check dashboard
  expect(wrapper.find('#dashboard').exists()).toBe(true);
});

// ✅ SOLUTION - Separate tests
describe('User Authentication', () => {
  it('should register new user', async () => {
    // Test registration only
  });

  it('should login existing user', async () => {
    // Test login only
  });

  it('should redirect to dashboard after login', async () => {
    // Test redirect only
  });
});
```

### 4. **Incorrect Mock Usage**

```javascript
// ❌ PROBLEM - Mock doesn't match real implementation
const mockUserService = {
  getUser: vi.fn().mockReturnValue({ name: 'John' }), // Returns sync
};

// But real service is async
const realUserService = {
  async getUser() {
    return await fetch('/api/user');
  },
};

// ✅ SOLUTION - Mock matches real behavior
const mockUserService = {
  getUser: vi.fn().mockResolvedValue({ name: 'John' }), // Returns Promise
};
```

---

## Vue 3 Specific Guidelines

### 1. **Testing Composition API**

```javascript
import { ref, computed } from 'vue';

// composable.js
export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  const doubled = computed(() => count.value * 2);

  const increment = () => count.value++;
  const decrement = () => count.value--;

  return { count, doubled, increment, decrement };
}

// composable.spec.js
describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { count } = useCounter();
    expect(count.value).toBe(0);
  });

  it('should initialize with custom value', () => {
    const { count } = useCounter(10);
    expect(count.value).toBe(10);
  });

  it('should compute doubled value correctly', () => {
    const { count, doubled } = useCounter(5);
    expect(doubled.value).toBe(10);
  });

  it('should increment count', () => {
    const { count, increment } = useCounter();
    increment();
    expect(count.value).toBe(1);
  });
});
```

### 2. **Testing with Pinia**

```javascript
import { setActivePinia, createPinia } from 'pinia';
import { createTestingPinia } from '@pinia/testing';

describe('Component with Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('should interact with store', () => {
    const wrapper = mount(Component, {
      global: {
        plugins: [
          createTestingPinia({
            createSpy: vi.fn,
            initialState: {
              user: { currentUser: { name: 'John' } },
            },
          }),
        ],
      },
    });

    const store = useUserStore();
    expect(store.currentUser.name).toBe('John');
  });
});
```

### 3. **Testing Props and Emits**

```javascript
describe('ButtonComponent', () => {
  it('should render with correct props', () => {
    const wrapper = mount(ButtonComponent, {
      props: {
        type: 'primary',
        disabled: true,
        label: 'Click me',
      },
    });

    expect(wrapper.classes()).toContain('btn-primary');
    expect(wrapper.attributes('disabled')).toBeDefined();
    expect(wrapper.text()).toBe('Click me');
  });

  it('should emit click event with payload', async () => {
    const wrapper = mount(ButtonComponent);

    await wrapper.trigger('click');

    expect(wrapper.emitted()).toHaveProperty('click');
    expect(wrapper.emitted('click')).toHaveLength(1);
    expect(wrapper.emitted('click')[0]).toEqual([{ timestamp: expect.any(Number) }]);
  });
});
```

### 4. **Testing Slots**

```javascript
describe('Modal Component', () => {
  it('should render default slot content', () => {
    const wrapper = mount(Modal, {
      slots: {
        default: '<p>Modal content</p>',
      },
    });

    expect(wrapper.find('p').text()).toBe('Modal content');
  });

  it('should render named slots', () => {
    const wrapper = mount(Modal, {
      slots: {
        header: '<h2>Modal Title</h2>',
        footer: '<button>Close</button>',
      },
    });

    expect(wrapper.find('h2').text()).toBe('Modal Title');
    expect(wrapper.find('button').text()).toBe('Close');
  });
});
```

### 5. **Testing Provide/Inject**

```javascript
// Parent component provides value
const ParentComponent = {
  template: '<ChildComponent />',
  components: { ChildComponent },
  provide() {
    return {
      theme: 'dark',
    };
  },
};

// Child component injects value
const ChildComponent = {
  template: '<div :class="themeClass">Content</div>',
  inject: ['theme'],
  computed: {
    themeClass() {
      return `theme-${this.theme}`;
    },
  },
};

// Test
it('should receive injected value from parent', () => {
  const wrapper = mount(ParentComponent);
  const child = wrapper.findComponent(ChildComponent);

  expect(child.classes()).toContain('theme-dark');
});

// Test injection directly
it('should use provided theme value', () => {
  const wrapper = mount(ChildComponent, {
    global: {
      provide: {
        theme: 'light',
      },
    },
  });

  expect(wrapper.classes()).toContain('theme-light');
});
```

---

## Testing Patterns

### 1. **Page Object Model**

```javascript
// page-objects/LoginPage.js
export class LoginPage {
  constructor(wrapper) {
    this.wrapper = wrapper;
  }

  get emailInput() {
    return this.wrapper.find('[data-testid="email-input"]');
  }

  get passwordInput() {
    return this.wrapper.find('[data-testid="password-input"]');
  }

  get submitButton() {
    return this.wrapper.find('[data-testid="submit-button"]');
  }

  get errorMessage() {
    return this.wrapper.find('[data-testid="error-message"]');
  }

  async login(email, password) {
    await this.emailInput.setValue(email);
    await this.passwordInput.setValue(password);
    await this.submitButton.trigger('click');
  }

  async expectErrorMessage(message) {
    expect(this.errorMessage.text()).toBe(message);
  }
}

// Usage in tests
describe('Login Flow', () => {
  let loginPage;

  beforeEach(() => {
    const wrapper = mount(LoginComponent);
    loginPage = new LoginPage(wrapper);
  });

  it('should show error for invalid credentials', async () => {
    await loginPage.login('invalid@email.com', 'wrongpassword');
    await loginPage.expectErrorMessage('Invalid credentials');
  });
});
```

### 2. **Factory Pattern for Test Data**

```javascript
// factories/userFactory.js
export const createUser = (overrides = {}) => ({
  id: Math.random().toString(36),
  name: 'John Doe',
  email: 'john@example.com',
  role: 'user',
  isActive: true,
  createdAt: new Date().toISOString(),
  ...overrides,
});

export const createAdmin = (overrides = {}) =>
  createUser({
    role: 'admin',
    permissions: ['read', 'write', 'delete'],
    ...overrides,
  });

// Usage
it('should display admin badge for admin users', () => {
  const adminUser = createAdmin({ name: 'Jane Admin' });
  const wrapper = mount(UserCard, { props: { user: adminUser } });

  expect(wrapper.find('.admin-badge').exists()).toBe(true);
});
```

### 3. **Custom Matchers**

```javascript
// test-utils/custom-matchers.js
expect.extend({
  toBeVisible(received) {
    const pass = received.isVisible();
    return {
      message: () => `expected element to ${pass ? 'not ' : ''}be visible`,
      pass,
    };
  },

  toHaveEmitted(wrapper, event, times = 1) {
    const emitted = wrapper.emitted(event);
    const pass = emitted && emitted.length === times;

    return {
      message: () => `expected component to have emitted "${event}" ${times} times, but got ${emitted?.length || 0}`,
      pass,
    };
  },
});

// Usage
it('should show modal when button is clicked', async () => {
  await wrapper.find('button').trigger('click');
  expect(wrapper.find('.modal')).toBeVisible();
  expect(wrapper).toHaveEmitted('modal-opened', 1);
});
```

---

## Tools and Setup

### Recommended Testing Stack

```json
{
  "devDependencies": {
    "vitest": "^1.0.0",
    "@vue/test-utils": "^2.4.0",
    "@pinia/testing": "^0.1.0",
    "jsdom": "^23.0.0",
    "happy-dom": "^12.0.0"
  }
}
```

### Vitest Configuration

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/__tests__/setup.js'],
    globals: true,
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      exclude: ['node_modules/', 'src/__tests__/', 'src/**/*.spec.js', 'src/**/*.test.js'],
    },
  },
  resolve: {
    alias: {
      '@': '/src',
    },
  },
});
```

### Test Setup File

```javascript
// src/__tests__/setup.js
import { vi } from 'vitest';
import { config } from '@vue/test-utils';

// Global test utilities
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}));

// Global component stubs
config.global.stubs = {
  'router-link': true,
  'router-view': true,
  'font-awesome-icon': true,
};

// Mock IntersectionObserver
global.IntersectionObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  disconnect: vi.fn(),
}));

// Suppress console errors in tests unless explicitly testing them
const originalError = console.error;
beforeAll(() => {
  console.error = (...args) => {
    if (typeof args[0] === 'string' && args[0].includes('Warning')) {
      return;
    }
    originalError.call(console, ...args);
  };
});

afterAll(() => {
  console.error = originalError;
});
```

---

## Conclusion

Effective testing is crucial for maintaining code quality and preventing regressions. Remember these key principles:

1. **Test behavior, not implementation**
2. **Keep tests simple and focused**
3. **Use descriptive names and organize tests logically**
4. **Mock external dependencies appropriately**
5. **Handle async operations correctly**
6. **Test both happy paths and edge cases**
7. **Make tests maintainable and readable**

By following these guidelines and patterns, you'll create a robust testing suite that gives you confidence in your code and helps maintain high-quality applications.

---

**Additional Resources:**

- [Vue Test Utils Documentation](https://test-utils.vuejs.org/)
- [Vitest Documentation](https://vitest.dev/)
- [Testing Library Principles](https://testing-library.com/docs/guiding-principles)
- [Pinia Testing Guide](https://pinia.vuejs.org/cookbook/testing.html)
