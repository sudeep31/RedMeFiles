# 🧪 **Angular Testing Strategies & Advanced Frameworks**

## 🎯 **What You'll Learn**

This comprehensive guide covers enterprise-grade testing strategies for Angular applications, including unit testing, integration testing, E2E testing, performance testing, and advanced testing patterns that ensure code quality and reliability in production.

---

## 📚 **The Basics: Testing Pyramid & Strategy**

### **🏗️ Enterprise Testing Architecture**

```
                    🔺
                   /   \
                  /  E2E  \     ← Few, High-Value
                 /  Tests  \
                /-----------\
               /             \
              / Integration   \   ← More, Focused
             /    Tests       \
            /-----------------\
           /                   \
          /    Unit Tests       \  ← Many, Fast
         /_____________________ \
```

**Testing Strategy Principles:**

- **70% Unit Tests** - Fast, isolated, developer-focused
- **20% Integration Tests** - Component interactions, API contracts
- **10% E2E Tests** - Critical user journeys, browser automation

---

## 🔬 **Advanced Unit Testing with Jest & Testing Library**

### **🚀 Comprehensive Unit Testing Setup**

```typescript
// src/testing/test-setup.ts
import "zone.js/dist/zone-testing";
import { getTestBed } from "@angular/core/testing";
import {
  BrowserDynamicTestingModule,
  platformBrowserDynamicTesting,
} from "@angular/platform-browser-dynamic/testing";
import { configure } from "@testing-library/angular";

// Configure Angular testing environment
getTestBed().initTestEnvironment(
  BrowserDynamicTestingModule,
  platformBrowserDynamicTesting()
);

// Configure Testing Library
configure({
  testIdAttribute: "data-testid",
  defaultHidden: false,
  asyncUtilTimeout: 5000,
});

// Global test utilities
declare global {
  namespace jest {
    interface Matchers<R> {
      toBeInTheDocument(): R;
      toHaveClass(className: string): R;
      toBeVisible(): R;
      toBeDisabled(): R;
      toHaveValue(value: string | number): R;
    }
  }
}

// Mock common browser APIs
Object.defineProperty(window, "matchMedia", {
  writable: true,
  value: jest.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};

// Mock ResizeObserver
global.ResizeObserver = class ResizeObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};
```

### **🎭 Advanced Testing Utilities & Factories**

```typescript
// src/testing/test-utils.ts
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { render, RenderResult, screen } from "@testing-library/angular";
import { userEvent } from "@testing-library/user-event";
import { BehaviorSubject, Observable, of } from "rxjs";
import {
  Component,
  Injectable,
  Input,
  Output,
  EventEmitter,
} from "@angular/core";

// 🏭 TEST DATA FACTORIES
export class TestDataFactory {
  static createUser(overrides: Partial<User> = {}): User {
    return {
      id: this.generateId(),
      email: "test@example.com",
      firstName: "John",
      lastName: "Doe",
      role: "user",
      isActive: true,
      createdAt: new Date(),
      ...overrides,
    };
  }

  static createProduct(overrides: Partial<Product> = {}): Product {
    return {
      id: this.generateId(),
      name: "Test Product",
      description: "A test product description",
      price: 99.99,
      category: "electronics",
      inStock: true,
      tags: ["test", "product"],
      ...overrides,
    };
  }

  static createOrder(overrides: Partial<Order> = {}): Order {
    return {
      id: this.generateId(),
      userId: this.generateId(),
      items: [
        {
          productId: this.generateId(),
          quantity: 2,
          price: 99.99,
        },
      ],
      status: "pending",
      total: 199.98,
      createdAt: new Date(),
      ...overrides,
    };
  }

  private static generateId(): string {
    return Math.random().toString(36).substr(2, 9);
  }
}

// 🎭 MOCK BUILDERS
export class MockBuilder {
  static service<T>(type: new (...args: any[]) => T): MockedService<T> {
    const mock = {} as MockedService<T>;

    // Get prototype methods
    const prototype = type.prototype;
    const methodNames = Object.getOwnPropertyNames(prototype).filter(
      (name) => name !== "constructor" && typeof prototype[name] === "function"
    );

    // Create jest mocks for each method
    methodNames.forEach((methodName) => {
      mock[methodName] = jest.fn();
    });

    return mock;
  }

  static component<T>(
    component: new (...args: any[]) => T,
    inputs: Partial<T> = {}
  ): MockedComponent<T> {
    @Component({
      selector: "mock-component",
      template: '<div data-testid="mock-component">Mock Component</div>',
    })
    class MockComponent implements Partial<T> {
      // Copy inputs from original component
      constructor() {
        Object.assign(this, inputs);
      }

      // Mock common component methods
      ngOnInit = jest.fn();
      ngOnDestroy = jest.fn();
      ngOnChanges = jest.fn();
      ngAfterViewInit = jest.fn();
    }

    return MockComponent as any;
  }

  static httpResponse<T>(
    data: T,
    options: { status?: number; delay?: number } = {}
  ): Observable<T> {
    const response = of(data);

    if (options.delay) {
      return new Observable((subscriber) => {
        setTimeout(() => {
          subscriber.next(data);
          subscriber.complete();
        }, options.delay);
      });
    }

    return response;
  }
}

// 📋 CUSTOM RENDER UTILITIES
export interface CustomRenderOptions {
  route?: string;
  user?: User;
  permissions?: string[];
  providers?: any[];
  imports?: any[];
  declarations?: any[];
}

export async function customRender<T>(
  component: new (...args: any[]) => T,
  options: CustomRenderOptions = {}
): Promise<RenderResult<T> & { user: typeof userEvent }> {
  const { route = "/", user, permissions = [], ...renderOptions } = options;

  // Setup mock providers
  const providers = [
    {
      provide: Router,
      useValue: {
        navigate: jest.fn(),
        url: route,
      },
    },
    {
      provide: AuthService,
      useValue: {
        currentUser$: of(user),
        hasPermission: jest.fn((permission: string) =>
          permissions.includes(permission)
        ),
        isAuthenticated$: of(!!user),
      },
    },
    ...(renderOptions.providers || []),
  ];

  const result = await render(component, {
    ...renderOptions,
    providers,
  });

  return {
    ...result,
    user: userEvent.setup(),
  };
}

// 🎯 PAGE OBJECT MODEL
export abstract class PageObject {
  constructor(
    protected fixture: ComponentFixture<any>,
    protected screen = screen
  ) {}

  // Common interactions
  async clickButton(testId: string): Promise<void> {
    const button = this.screen.getByTestId(testId);
    await userEvent.click(button);
    this.fixture.detectChanges();
  }

  async fillInput(testId: string, value: string): Promise<void> {
    const input = this.screen.getByTestId(testId);
    await userEvent.clear(input);
    await userEvent.type(input, value);
    this.fixture.detectChanges();
  }

  async selectOption(testId: string, option: string): Promise<void> {
    const select = this.screen.getByTestId(testId);
    await userEvent.selectOptions(select, option);
    this.fixture.detectChanges();
  }

  // Assertions
  expectElementToBeVisible(testId: string): void {
    const element = this.screen.getByTestId(testId);
    expect(element).toBeVisible();
  }

  expectElementToHaveText(testId: string, text: string): void {
    const element = this.screen.getByTestId(testId);
    expect(element).toHaveTextContent(text);
  }

  expectElementToBeDisabled(testId: string): void {
    const element = this.screen.getByTestId(testId);
    expect(element).toBeDisabled();
  }

  // Wait utilities
  async waitForElement(testId: string, timeout = 3000): Promise<HTMLElement> {
    return this.screen.findByTestId(testId, {}, { timeout });
  }

  async waitForElementToDisappear(
    testId: string,
    timeout = 3000
  ): Promise<void> {
    await waitForElementToBeRemoved(() => this.screen.queryByTestId(testId), {
      timeout,
    });
  }
}

// 🧪 SPECIFIC PAGE OBJECTS
export class UserProfilePageObject extends PageObject {
  // Getters for elements
  get firstNameInput() {
    return this.screen.getByTestId("first-name-input");
  }

  get lastNameInput() {
    return this.screen.getByTestId("last-name-input");
  }

  get emailInput() {
    return this.screen.getByTestId("email-input");
  }

  get saveButton() {
    return this.screen.getByTestId("save-button");
  }

  get cancelButton() {
    return this.screen.getByTestId("cancel-button");
  }

  // Actions
  async fillUserForm(user: Partial<User>): Promise<void> {
    if (user.firstName) {
      await this.fillInput("first-name-input", user.firstName);
    }

    if (user.lastName) {
      await this.fillInput("last-name-input", user.lastName);
    }

    if (user.email) {
      await this.fillInput("email-input", user.email);
    }
  }

  async saveProfile(): Promise<void> {
    await this.clickButton("save-button");
  }

  // Assertions
  expectFormToBeValid(): void {
    expect(this.saveButton).not.toBeDisabled();
  }

  expectFormToBeInvalid(): void {
    expect(this.saveButton).toBeDisabled();
  }

  expectSuccessMessage(): void {
    this.expectElementToBeVisible("success-message");
  }
}

// 🎭 TYPE DEFINITIONS
type MockedService<T> = {
  [K in keyof T]: T[K] extends (...args: any[]) => any
    ? jest.MockedFunction<T[K]>
    : T[K];
};

type MockedComponent<T> = new (...args: any[]) => Partial<T> & {
  ngOnInit: jest.MockedFunction<any>;
  ngOnDestroy: jest.MockedFunction<any>;
  ngOnChanges: jest.MockedFunction<any>;
  ngAfterViewInit: jest.MockedFunction<any>;
};

// 🚀 ADVANCED COMPONENT TESTING EXAMPLE
describe("UserProfileComponent", () => {
  let pageObject: UserProfilePageObject;
  let mockUserService: MockedService<UserService>;
  let mockNotificationService: MockedService<NotificationService>;

  beforeEach(async () => {
    // Setup mocks
    mockUserService = MockBuilder.service(UserService);
    mockNotificationService = MockBuilder.service(NotificationService);

    // Setup successful responses by default
    mockUserService.updateUser.mockReturnValue(of({ success: true }));
    mockUserService.getCurrentUser.mockReturnValue(
      of(TestDataFactory.createUser())
    );

    // Render component with mocks
    const { fixture } = await customRender(UserProfileComponent, {
      providers: [
        { provide: UserService, useValue: mockUserService },
        { provide: NotificationService, useValue: mockNotificationService },
      ],
    });

    pageObject = new UserProfilePageObject(fixture);
  });

  describe("Form Validation", () => {
    it("should disable save button when form is invalid", async () => {
      // Arrange
      const invalidUser = { firstName: "", email: "invalid-email" };

      // Act
      await pageObject.fillUserForm(invalidUser);

      // Assert
      pageObject.expectFormToBeInvalid();
    });

    it("should enable save button when form is valid", async () => {
      // Arrange
      const validUser = TestDataFactory.createUser();

      // Act
      await pageObject.fillUserForm(validUser);

      // Assert
      pageObject.expectFormToBeValid();
    });
  });

  describe("User Profile Update", () => {
    it("should successfully update user profile", async () => {
      // Arrange
      const updatedUser = TestDataFactory.createUser({
        firstName: "Updated",
        lastName: "User",
      });

      // Act
      await pageObject.fillUserForm(updatedUser);
      await pageObject.saveProfile();

      // Assert
      expect(mockUserService.updateUser).toHaveBeenCalledWith(
        expect.objectContaining({
          firstName: "Updated",
          lastName: "User",
        })
      );

      pageObject.expectSuccessMessage();

      expect(mockNotificationService.showSuccess).toHaveBeenCalledWith(
        "Profile updated successfully"
      );
    });

    it("should handle update error gracefully", async () => {
      // Arrange
      const errorMessage = "Update failed";
      mockUserService.updateUser.mockReturnValue(
        throwError(new Error(errorMessage))
      );

      const user = TestDataFactory.createUser();

      // Act
      await pageObject.fillUserForm(user);
      await pageObject.saveProfile();

      // Assert
      expect(mockNotificationService.showError).toHaveBeenCalledWith(
        "Failed to update profile. Please try again."
      );
    });
  });

  describe("Loading States", () => {
    it("should show loading spinner during profile update", async () => {
      // Arrange
      const delayedResponse = MockBuilder.httpResponse(
        { success: true },
        { delay: 1000 }
      );
      mockUserService.updateUser.mockReturnValue(delayedResponse);

      const user = TestDataFactory.createUser();

      // Act
      await pageObject.fillUserForm(user);
      await pageObject.saveProfile();

      // Assert
      pageObject.expectElementToBeVisible("loading-spinner");
      expect(pageObject.saveButton).toBeDisabled();
    });
  });

  describe("Accessibility", () => {
    it("should have proper ARIA labels", () => {
      // Assert
      expect(pageObject.firstNameInput).toHaveAttribute(
        "aria-label",
        "First Name"
      );
      expect(pageObject.emailInput).toHaveAttribute(
        "aria-label",
        "Email Address"
      );
    });

    it("should show validation messages with proper ARIA", async () => {
      // Act
      await pageObject.fillInput("email-input", "invalid-email");

      // Assert
      const errorMessage = pageObject.screen.getByRole("alert");
      expect(errorMessage).toBeInTheDocument();
      expect(pageObject.emailInput).toHaveAttribute(
        "aria-describedby",
        expect.stringContaining("email-error")
      );
    });
  });

  describe("Performance", () => {
    it("should not re-render unnecessarily", () => {
      // This would use performance testing utilities
      const renderSpy = jest.spyOn(UserProfileComponent.prototype, "ngOnInit");

      // Simulate props that should not trigger re-render
      // ... performance assertions

      expect(renderSpy).toHaveBeenCalledTimes(1);
    });
  });
});
```

---

## 🔗 **Integration Testing Strategies**

### **🌐 API Integration Testing**

```typescript
// src/testing/integration/api-integration.spec.ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { of } from "rxjs";

describe("API Integration Tests", () => {
  let userService: UserService;
  let httpController: HttpTestingController;
  let httpClient: HttpClient;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService],
    });

    userService = TestBed.inject(UserService);
    httpController = TestBed.inject(HttpTestingController);
    httpClient = TestBed.inject(HttpClient);
  });

  afterEach(() => {
    httpController.verify();
  });

  describe("User CRUD Operations", () => {
    it("should get user by id", () => {
      // Arrange
      const userId = "123";
      const expectedUser = TestDataFactory.createUser({ id: userId });

      // Act
      userService.getUserById(userId).subscribe((user) => {
        expect(user).toEqual(expectedUser);
      });

      // Assert
      const req = httpController.expectOne(`/api/users/${userId}`);
      expect(req.request.method).toBe("GET");
      req.flush(expectedUser);
    });

    it("should handle API errors gracefully", () => {
      // Arrange
      const userId = "123";
      const errorMessage = "User not found";

      // Act
      userService.getUserById(userId).subscribe({
        next: () => fail("Should have failed"),
        error: (error) => {
          expect(error.message).toBe(errorMessage);
        },
      });

      // Assert
      const req = httpController.expectOne(`/api/users/${userId}`);
      req.flush(
        { message: errorMessage },
        {
          status: 404,
          statusText: "Not Found",
        }
      );
    });

    it("should retry failed requests", fakeAsync(() => {
      // Arrange
      const userId = "123";
      const expectedUser = TestDataFactory.createUser();

      let callCount = 0;
      spyOn(httpClient, "get").and.returnValue(
        defer(() => {
          callCount++;
          if (callCount < 3) {
            return throwError(new Error("Network error"));
          }
          return of(expectedUser);
        })
      );

      // Act
      userService.getUserById(userId).subscribe((user) => {
        expect(user).toEqual(expectedUser);
        expect(callCount).toBe(3);
      });

      // Advance time for retry delays
      tick(5000);
    }));
  });

  describe("Caching Behavior", () => {
    it("should cache user data and not make duplicate requests", () => {
      // Arrange
      const userId = "123";
      const expectedUser = TestDataFactory.createUser();

      // Act - Make two requests
      userService.getUserById(userId).subscribe();
      userService.getUserById(userId).subscribe();

      // Assert - Only one HTTP request should be made
      const req = httpController.expectOne(`/api/users/${userId}`);
      req.flush(expectedUser);

      // Verify no additional requests
      httpController.verify();
    });

    it("should invalidate cache after update", () => {
      // Arrange
      const userId = "123";
      const originalUser = TestDataFactory.createUser();
      const updatedUser = { ...originalUser, firstName: "Updated" };

      // Act - Get, Update, Get again
      userService.getUserById(userId).subscribe();

      let req = httpController.expectOne(`/api/users/${userId}`);
      req.flush(originalUser);

      userService.updateUser(userId, updatedUser).subscribe();

      req = httpController.expectOne(`/api/users/${userId}`);
      req.flush(updatedUser);

      userService.getUserById(userId).subscribe((user) => {
        expect(user.firstName).toBe("Updated");
      });

      // Should make another request since cache was invalidated
      req = httpController.expectOne(`/api/users/${userId}`);
      req.flush(updatedUser);
    });
  });

  describe("Concurrent Request Handling", () => {
    it("should handle concurrent requests correctly", fakeAsync(() => {
      // Arrange
      const userIds = ["1", "2", "3"];
      const users = userIds.map((id) => TestDataFactory.createUser({ id }));

      // Act - Make concurrent requests
      const responses: User[] = [];
      userIds.forEach((id) => {
        userService.getUserById(id).subscribe((user) => {
          responses.push(user);
        });
      });

      // Assert - Handle all requests
      userIds.forEach((id, index) => {
        const req = httpController.expectOne(`/api/users/${id}`);
        req.flush(users[index]);
      });

      tick();

      expect(responses).toHaveLength(3);
      expect(responses[0].id).toBe("1");
      expect(responses[1].id).toBe("2");
      expect(responses[2].id).toBe("3");
    }));
  });
});
```

This is **Part 1** of the Testing Strategies guide. Would you like me to continue with **Part 2** covering:

- 🎭 **E2E Testing with Playwright/Cypress**
- 🚀 **Performance Testing & Load Testing**
- 🔄 **Test Automation & CI/CD Integration**
- 📊 **Visual Regression Testing**

Should I proceed with Part 2? 🧪
