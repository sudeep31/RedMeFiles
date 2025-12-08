# 🧪 Advanced Angular Testing Strategies: Complete Guide

## 🎯 **Question Overview**

_"What are advanced testing strategies in Angular, including testing async operations, components with dependencies, and E2E testing?"_

## 🔍 **Understanding Advanced Testing**

Advanced Angular testing goes **beyond basic unit tests** to cover complex scenarios like **async operations**, **component interactions**, **service integration**, **state management**, and **end-to-end user workflows**.

Modern Angular testing leverages **TestBed**, **async testing utilities**, **mock strategies**, and **comprehensive E2E frameworks**! 🎯

## 🏗️ **Testing Architecture**

### **1. 🎭 Testing Pyramid Structure**

```typescript
// src/testing/test-config.ts - Global Test Configuration
import { TestBed } from "@angular/core/testing";
import { BrowserAnimationsModule } from "@angular/platform-browser/animations";
import { HttpClientTestingModule } from "@angular/common/http/testing";
import { RouterTestingModule } from "@angular/router/testing";

export interface TestConfig {
  imports?: any[];
  declarations?: any[];
  providers?: any[];
  schemas?: any[];
}

export class TestingUtilities {
  static configureTestBed(config: TestConfig = {}): void {
    beforeEach(async () => {
      await TestBed.configureTestingModule({
        imports: [
          BrowserAnimationsModule,
          HttpClientTestingModule,
          RouterTestingModule,
          ...(config.imports || []),
        ],
        declarations: config.declarations || [],
        providers: config.providers || [],
        schemas: config.schemas || [],
      }).compileComponents();
    });
  }

  static createComponent<T>(componentClass: any): {
    component: T;
    fixture: ComponentFixture<T>;
    element: HTMLElement;
  } {
    const fixture = TestBed.createComponent(componentClass);
    const component = fixture.componentInstance;
    const element = fixture.nativeElement;

    return { component, fixture, element };
  }

  static async flushPromises(): Promise<void> {
    await new Promise((resolve) => setTimeout(resolve, 0));
  }

  static triggerAsyncOperation<T>(operation: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      operation().then(resolve).catch(reject);
    });
  }
}

// Custom matchers for better assertions
declare global {
  namespace jasmine {
    interface Matchers<T> {
      toHaveBeenCalledWithSignal(value: any): boolean;
      toEmitEvent(eventName: string, data?: any): boolean;
      toBeInLoadingState(): boolean;
    }
  }
}

beforeEach(() => {
  jasmine.addMatchers({
    toHaveBeenCalledWithSignal: () => ({
      compare: (actual: jasmine.Spy, expected: any) => {
        const calls = actual.calls.all();
        const found = calls.some(
          (call) => call.args.length > 0 && call.args[0] === expected
        );

        return {
          pass: found,
          message: `Expected spy to have been called with signal value ${expected}`,
        };
      },
    }),

    toEmitEvent: () => ({
      compare: (actual: any, eventName: string, data?: any) => {
        // Custom matcher for event emission testing
        return {
          pass: true, // Implementation depends on your event system
          message: `Expected ${eventName} to be emitted`,
        };
      },
    }),

    toBeInLoadingState: () => ({
      compare: (actual: any) => {
        const isLoading = actual.loading && actual.loading() === true;
        return {
          pass: isLoading,
          message: `Expected component to be in loading state`,
        };
      },
    }),
  });
});
```

### **2. 🔧 Test Doubles & Mocking**

```typescript
// src/testing/mocks/service-mocks.ts - Service Mock Factory
import { Observable, of, throwError, BehaviorSubject } from "rxjs";
import { delay } from "rxjs/operators";

export class MockBuilder<T> {
  private mockObject: Partial<T> = {};

  method(methodName: keyof T, returnValue: any): MockBuilder<T> {
    this.mockObject[methodName] = jasmine
      .createSpy(String(methodName))
      .and.returnValue(returnValue);
    return this;
  }

  asyncMethod(
    methodName: keyof T,
    returnValue: any,
    delayMs = 0
  ): MockBuilder<T> {
    const observable = of(returnValue).pipe(delay(delayMs));
    this.mockObject[methodName] = jasmine
      .createSpy(String(methodName))
      .and.returnValue(observable);
    return this;
  }

  errorMethod(methodName: keyof T, error: any): MockBuilder<T> {
    this.mockObject[methodName] = jasmine
      .createSpy(String(methodName))
      .and.returnValue(throwError(() => error));
    return this;
  }

  property(propertyName: keyof T, value: any): MockBuilder<T> {
    this.mockObject[propertyName] = value;
    return this;
  }

  signal(propertyName: keyof T, initialValue: any): MockBuilder<T> {
    const signalMock = {
      [propertyName]: jasmine
        .createSpy(String(propertyName))
        .and.returnValue(initialValue),
      [`set${String(propertyName)}`]: jasmine.createSpy(
        `set${String(propertyName)}`
      ),
      [`update${String(propertyName)}`]: jasmine.createSpy(
        `update${String(propertyName)}`
      ),
    };
    Object.assign(this.mockObject, signalMock);
    return this;
  }

  build(): jasmine.SpyObj<T> {
    return this.mockObject as jasmine.SpyObj<T>;
  }
}

// HTTP Service Mock
export interface User {
  id: string;
  name: string;
  email: string;
  role: string;
}

export interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

export class MockHttpService {
  static createUserService() {
    return new MockBuilder<any>()
      .asyncMethod("getUsers", {
        data: [
          {
            id: "1",
            name: "John Doe",
            email: "john@example.com",
            role: "admin",
          },
          {
            id: "2",
            name: "Jane Smith",
            email: "jane@example.com",
            role: "user",
          },
        ],
        success: true,
        message: "Users retrieved successfully",
      } as ApiResponse<User[]>)
      .asyncMethod("getUserById", {
        data: {
          id: "1",
          name: "John Doe",
          email: "john@example.com",
          role: "admin",
        },
        success: true,
        message: "User retrieved successfully",
      } as ApiResponse<User>)
      .asyncMethod("createUser", {
        data: {
          id: "3",
          name: "New User",
          email: "new@example.com",
          role: "user",
        },
        success: true,
        message: "User created successfully",
      } as ApiResponse<User>)
      .asyncMethod("updateUser", {
        data: {
          id: "1",
          name: "Updated User",
          email: "updated@example.com",
          role: "admin",
        },
        success: true,
        message: "User updated successfully",
      } as ApiResponse<User>)
      .errorMethod("deleteUser", new Error("Deletion failed"))
      .build();
  }

  static createAuthService() {
    const currentUserSubject = new BehaviorSubject<User | null>(null);

    return new MockBuilder<any>()
      .method(
        "login",
        of({ token: "mock-token", user: { id: "1", name: "Test User" } })
      )
      .method("logout", of(true))
      .method("refreshToken", of({ token: "refreshed-token" }))
      .property("currentUser$", currentUserSubject.asObservable())
      .property("isAuthenticated$", of(true))
      .method("hasRole", true)
      .method("hasPermission", true)
      .build();
  }
}

// Component Mock Utilities
export class MockComponentBuilder {
  static createProductCard() {
    return {
      product: null,
      loading: false,
      onProductClick: jasmine.createSpy("onProductClick"),
      onAddToCart: jasmine.createSpy("onAddToCart"),
      onFavorite: jasmine.createSpy("onFavorite"),
    };
  }

  static createModal() {
    return {
      isOpen: false,
      title: "",
      content: "",
      open: jasmine.createSpy("open"),
      close: jasmine.createSpy("close"),
      confirm: jasmine.createSpy("confirm"),
    };
  }
}
```

## 🧩 **Component Testing Strategies**

### **1. 🎨 Testing Components with Signals**

```typescript
// src/app/components/user-profile/user-profile.component.spec.ts
import {
  ComponentFixture,
  TestBed,
  fakeAsync,
  tick,
  flush,
} from "@angular/core/testing";
import { signal } from "@angular/core";
import { By } from "@angular/platform-browser";
import { DebugElement } from "@angular/core";

import { UserProfileComponent } from "./user-profile.component";
import { UserService } from "../../services/user.service";
import { NotificationService } from "../../services/notification.service";
import {
  MockBuilder,
  MockHttpService,
} from "../../testing/mocks/service-mocks";

describe("UserProfileComponent", () => {
  let component: UserProfileComponent;
  let fixture: ComponentFixture<UserProfileComponent>;
  let mockUserService: jasmine.SpyObj<any>;
  let mockNotificationService: jasmine.SpyObj<any>;

  beforeEach(async () => {
    mockUserService = MockHttpService.createUserService();
    mockNotificationService = new MockBuilder<any>()
      .method("showSuccess", undefined)
      .method("showError", undefined)
      .method("showInfo", undefined)
      .build();

    await TestBed.configureTestingModule({
      imports: [UserProfileComponent], // Standalone component
      providers: [
        { provide: UserService, useValue: mockUserService },
        { provide: NotificationService, useValue: mockNotificationService },
      ],
    }).compileComponents();

    fixture = TestBed.createComponent(UserProfileComponent);
    component = fixture.componentInstance;
  });

  describe("Signal-based State Management", () => {
    it("should initialize with loading state", () => {
      expect(component.loading()).toBeTruthy();
      expect(component.user()).toBeNull();
      expect(component.error()).toBeNull();
    });

    it("should load user data on init", fakeAsync(() => {
      component.userId.set("1");
      component.ngOnInit();

      expect(component.loading()).toBeTruthy();
      expect(mockUserService.getUserById).toHaveBeenCalledWith("1");

      tick(100); // Wait for async operation
      fixture.detectChanges();

      expect(component.loading()).toBeFalsy();
      expect(component.user()).toBeTruthy();
      expect(component.user()?.name).toBe("John Doe");
    }));

    it("should handle load error gracefully", fakeAsync(() => {
      mockUserService.getUserById.and.returnValue(
        throwError(() => new Error("User not found"))
      );

      component.userId.set("999");
      component.ngOnInit();

      tick(100);
      fixture.detectChanges();

      expect(component.loading()).toBeFalsy();
      expect(component.user()).toBeNull();
      expect(component.error()).toBe("Failed to load user profile");
      expect(mockNotificationService.showError).toHaveBeenCalledWith(
        "Failed to load user profile"
      );
    }));

    it("should update user data reactively", fakeAsync(() => {
      // Initial load
      component.userId.set("1");
      component.ngOnInit();
      tick(100);
      fixture.detectChanges();

      // Update user
      const updatedUserData = {
        id: "1",
        name: "Updated Name",
        email: "updated@example.com",
        role: "admin",
      };

      component.updateUser(updatedUserData);
      tick(100);
      fixture.detectChanges();

      expect(mockUserService.updateUser).toHaveBeenCalledWith(
        "1",
        updatedUserData
      );
      expect(component.user()?.name).toBe("Updated Name");
      expect(mockNotificationService.showSuccess).toHaveBeenCalledWith(
        "Profile updated successfully"
      );
    }));
  });

  describe("Template Integration", () => {
    it("should display loading spinner when loading", () => {
      component.loading.set(true);
      fixture.detectChanges();

      const loadingElement = fixture.debugElement.query(
        By.css('[data-testid="loading"]')
      );
      expect(loadingElement).toBeTruthy();
    });

    it("should display user data when loaded", fakeAsync(() => {
      component.userId.set("1");
      component.ngOnInit();
      tick(100);
      fixture.detectChanges();

      const nameElement = fixture.debugElement.query(
        By.css('[data-testid="user-name"]')
      );
      const emailElement = fixture.debugElement.query(
        By.css('[data-testid="user-email"]')
      );

      expect(nameElement.nativeElement.textContent).toContain("John Doe");
      expect(emailElement.nativeElement.textContent).toContain(
        "john@example.com"
      );
    }));

    it("should display error message when error occurs", () => {
      component.error.set("Failed to load user");
      fixture.detectChanges();

      const errorElement = fixture.debugElement.query(
        By.css('[data-testid="error-message"]')
      );
      expect(errorElement).toBeTruthy();
      expect(errorElement.nativeElement.textContent).toContain(
        "Failed to load user"
      );
    });

    it("should enable edit mode when edit button clicked", () => {
      component.user.set({
        id: "1",
        name: "John Doe",
        email: "john@example.com",
        role: "user",
      });
      component.canEdit.set(true);
      fixture.detectChanges();

      const editButton = fixture.debugElement.query(
        By.css('[data-testid="edit-button"]')
      );
      editButton.nativeElement.click();
      fixture.detectChanges();

      expect(component.isEditing()).toBeTruthy();

      const editForm = fixture.debugElement.query(
        By.css('[data-testid="edit-form"]')
      );
      expect(editForm).toBeTruthy();
    });
  });

  describe("Form Handling", () => {
    beforeEach(() => {
      component.user.set({
        id: "1",
        name: "John Doe",
        email: "john@example.com",
        role: "user",
      });
      component.isEditing.set(true);
      fixture.detectChanges();
    });

    it("should validate form inputs", () => {
      const nameInput = fixture.debugElement.query(
        By.css('[data-testid="name-input"]')
      );
      const emailInput = fixture.debugElement.query(
        By.css('[data-testid="email-input"]')
      );

      // Test empty name
      nameInput.nativeElement.value = "";
      nameInput.nativeElement.dispatchEvent(new Event("input"));
      fixture.detectChanges();

      expect(component.editForm.get("name")?.invalid).toBeTruthy();

      // Test invalid email
      emailInput.nativeElement.value = "invalid-email";
      emailInput.nativeElement.dispatchEvent(new Event("input"));
      fixture.detectChanges();

      expect(component.editForm.get("email")?.invalid).toBeTruthy();
    });

    it("should save valid form data", fakeAsync(() => {
      const nameInput = fixture.debugElement.query(
        By.css('[data-testid="name-input"]')
      );
      const emailInput = fixture.debugElement.query(
        By.css('[data-testid="email-input"]')
      );
      const saveButton = fixture.debugElement.query(
        By.css('[data-testid="save-button"]')
      );

      nameInput.nativeElement.value = "Updated Name";
      nameInput.nativeElement.dispatchEvent(new Event("input"));

      emailInput.nativeElement.value = "updated@example.com";
      emailInput.nativeElement.dispatchEvent(new Event("input"));

      fixture.detectChanges();

      saveButton.nativeElement.click();
      tick(100);
      fixture.detectChanges();

      expect(mockUserService.updateUser).toHaveBeenCalledWith(
        "1",
        jasmine.objectContaining({
          name: "Updated Name",
          email: "updated@example.com",
        })
      );
      expect(component.isEditing()).toBeFalsy();
    }));

    it("should cancel edit mode without saving", () => {
      const cancelButton = fixture.debugElement.query(
        By.css('[data-testid="cancel-button"]')
      );

      cancelButton.nativeElement.click();
      fixture.detectChanges();

      expect(component.isEditing()).toBeFalsy();
      expect(mockUserService.updateUser).not.toHaveBeenCalled();
      expect(component.editForm.value.name).toBe("John Doe"); // Form should reset
    });
  });

  describe("Permission-based UI", () => {
    it("should show edit button only when user has permission", () => {
      component.user.set({
        id: "1",
        name: "John Doe",
        email: "john@example.com",
        role: "user",
      });

      // Without edit permission
      component.canEdit.set(false);
      fixture.detectChanges();

      let editButton = fixture.debugElement.query(
        By.css('[data-testid="edit-button"]')
      );
      expect(editButton).toBeFalsy();

      // With edit permission
      component.canEdit.set(true);
      fixture.detectChanges();

      editButton = fixture.debugElement.query(
        By.css('[data-testid="edit-button"]')
      );
      expect(editButton).toBeTruthy();
    });

    it("should handle role-based feature visibility", () => {
      component.user.set({
        id: "1",
        name: "Admin User",
        email: "admin@example.com",
        role: "admin",
      });
      fixture.detectChanges();

      const adminPanel = fixture.debugElement.query(
        By.css('[data-testid="admin-panel"]')
      );
      expect(adminPanel).toBeTruthy();

      // Switch to regular user
      component.user.update((user) => ({ ...user!, role: "user" }));
      fixture.detectChanges();

      const adminPanelAfter = fixture.debugElement.query(
        By.css('[data-testid="admin-panel"]')
      );
      expect(adminPanelAfter).toBeFalsy();
    });
  });
});
```

### **2. 🔄 Testing Async Operations**

```typescript
// src/app/components/data-table/data-table.component.spec.ts
import {
  ComponentFixture,
  TestBed,
  fakeAsync,
  tick,
  discardPeriodicTasks,
} from "@angular/core/testing";
import { of, throwError, timer } from "rxjs";
import { delay, map } from "rxjs/operators";

import { DataTableComponent } from "./data-table.component";
import { DataService } from "../../services/data.service";

describe("DataTableComponent - Async Operations", () => {
  let component: DataTableComponent;
  let fixture: ComponentFixture<DataTableComponent>;
  let mockDataService: jasmine.SpyObj<DataService>;

  beforeEach(async () => {
    const dataSpy = jasmine.createSpyObj("DataService", [
      "getData",
      "deleteItem",
      "exportData",
    ]);

    await TestBed.configureTestingModule({
      imports: [DataTableComponent],
      providers: [{ provide: DataService, useValue: dataSpy }],
    }).compileComponents();

    fixture = TestBed.createComponent(DataTableComponent);
    component = fixture.componentInstance;
    mockDataService = TestBed.inject(
      DataService
    ) as jasmine.SpyObj<DataService>;
  });

  describe("Data Loading with Delays", () => {
    it("should handle immediate data load", fakeAsync(() => {
      const mockData = [
        { id: "1", name: "Item 1", status: "active" },
        { id: "2", name: "Item 2", status: "inactive" },
      ];

      mockDataService.getData.and.returnValue(of(mockData));

      component.loadData();

      expect(component.loading()).toBeTruthy();

      tick(); // Process immediate observable
      fixture.detectChanges();

      expect(component.loading()).toBeFalsy();
      expect(component.data()).toEqual(mockData);
      expect(component.error()).toBeNull();
    }));

    it("should handle delayed data load", fakeAsync(() => {
      const mockData = [{ id: "1", name: "Delayed Item" }];

      mockDataService.getData.and.returnValue(
        of(mockData).pipe(delay(2000)) // 2 second delay
      );

      component.loadData();

      expect(component.loading()).toBeTruthy();
      expect(component.data()).toEqual([]);

      tick(1000); // 1 second passed
      fixture.detectChanges();

      expect(component.loading()).toBeTruthy(); // Still loading

      tick(1000); // Total 2 seconds
      fixture.detectChanges();

      expect(component.loading()).toBeFalsy();
      expect(component.data()).toEqual(mockData);
    }));

    it("should handle timeout scenarios", fakeAsync(() => {
      mockDataService.getData.and.returnValue(
        timer(10000).pipe(map(() => [])) // 10 second delay, should timeout
      );

      component.setTimeout(5000); // 5 second timeout
      component.loadData();

      tick(5000); // Wait for timeout
      fixture.detectChanges();

      expect(component.loading()).toBeFalsy();
      expect(component.error()).toBe("Request timeout");
      expect(component.data()).toEqual([]);

      discardPeriodicTasks(); // Clean up timer
    }));
  });

  describe("Error Handling", () => {
    it("should handle network errors", fakeAsync(() => {
      const networkError = new Error("Network connection failed");
      mockDataService.getData.and.returnValue(
        throwError(() => networkError).pipe(delay(100))
      );

      component.loadData();

      tick(100);
      fixture.detectChanges();

      expect(component.loading()).toBeFalsy();
      expect(component.error()).toBe(
        "Failed to load data: Network connection failed"
      );
      expect(component.data()).toEqual([]);
    }));

    it("should handle server errors with retry", fakeAsync(() => {
      let callCount = 0;
      mockDataService.getData.and.callFake(() => {
        callCount++;
        if (callCount < 3) {
          return throwError(() => new Error("Server error"));
        }
        return of([{ id: "1", name: "Success after retry" }]);
      });

      component.setRetryCount(3);
      component.loadData();

      // First attempt fails
      tick(100);
      expect(component.loading()).toBeTruthy();

      // Second attempt fails
      tick(1000); // Retry delay
      tick(100);
      expect(component.loading()).toBeTruthy();

      // Third attempt succeeds
      tick(1000); // Retry delay
      tick(100);
      fixture.detectChanges();

      expect(component.loading()).toBeFalsy();
      expect(component.data()).toEqual([
        { id: "1", name: "Success after retry" },
      ]);
      expect(callCount).toBe(3);
    }));
  });

  describe("Pagination and Filtering", () => {
    it("should handle paginated data loading", fakeAsync(() => {
      const pageData = {
        items: [{ id: "1", name: "Page 1 Item" }],
        totalCount: 50,
        currentPage: 1,
        pageSize: 10,
      };

      mockDataService.getData.and.returnValue(of(pageData).pipe(delay(100)));

      component.loadPage(1);

      tick(100);
      fixture.detectChanges();

      expect(component.currentPage()).toBe(1);
      expect(component.totalItems()).toBe(50);
      expect(component.data()).toEqual(pageData.items);
    }));

    it("should debounce filter changes", fakeAsync(() => {
      mockDataService.getData.and.returnValue(of([]));

      // Rapid filter changes
      component.updateFilter("a");
      component.updateFilter("ab");
      component.updateFilter("abc");

      // Only one call should be made after debounce delay
      expect(mockDataService.getData).not.toHaveBeenCalled();

      tick(500); // Debounce delay

      expect(mockDataService.getData).toHaveBeenCalledTimes(1);
      expect(mockDataService.getData).toHaveBeenCalledWith(
        jasmine.objectContaining({ filter: "abc" })
      );
    }));
  });

  describe("Bulk Operations", () => {
    it("should handle bulk delete with progress tracking", fakeAsync(() => {
      const itemsToDelete = ["1", "2", "3", "4", "5"];

      let deleteCallCount = 0;
      mockDataService.deleteItem.and.callFake((id: string) => {
        deleteCallCount++;
        return of(true).pipe(delay(200)); // Each delete takes 200ms
      });

      component.bulkDelete(itemsToDelete);

      expect(component.bulkOperation.inProgress()).toBeTruthy();
      expect(component.bulkOperation.progress()).toBe(0);

      // After first item
      tick(200);
      fixture.detectChanges();
      expect(component.bulkOperation.progress()).toBe(20); // 1/5 = 20%

      // After all items
      tick(800); // Remaining 4 items
      fixture.detectChanges();

      expect(component.bulkOperation.inProgress()).toBeFalsy();
      expect(component.bulkOperation.progress()).toBe(100);
      expect(deleteCallCount).toBe(5);
    }));

    it("should handle partial bulk operation failures", fakeAsync(() => {
      const itemsToDelete = ["1", "2", "3"];

      mockDataService.deleteItem.and.callFake((id: string) => {
        if (id === "2") {
          return throwError(() => new Error("Delete failed")).pipe(delay(200));
        }
        return of(true).pipe(delay(200));
      });

      component.bulkDelete(itemsToDelete);

      tick(600); // All operations complete
      fixture.detectChanges();

      expect(component.bulkOperation.inProgress()).toBeFalsy();
      expect(component.bulkOperation.successCount()).toBe(2);
      expect(component.bulkOperation.failureCount()).toBe(1);
      expect(component.bulkOperation.errors()).toContain(
        "Failed to delete item 2: Delete failed"
      );
    }));
  });

  describe("Real-time Updates", () => {
    it("should handle websocket updates", fakeAsync(() => {
      // Mock websocket service
      const mockUpdate = {
        id: "1",
        name: "Updated Item",
        timestamp: Date.now(),
      };

      component.subscribeToUpdates();

      // Simulate websocket message
      component.handleRealtimeUpdate(mockUpdate);

      tick();
      fixture.detectChanges();

      const updatedItem = component.data().find((item) => item.id === "1");
      expect(updatedItem?.name).toBe("Updated Item");
    }));

    it("should handle connection loss and reconnection", fakeAsync(() => {
      component.subscribeToUpdates();

      // Simulate connection loss
      component.handleConnectionLoss();
      tick();

      expect(component.connectionStatus()).toBe("disconnected");

      // Verify reconnection attempts
      tick(5000); // First reconnect attempt
      tick(5000); // Second reconnect attempt

      // Simulate successful reconnection
      component.handleReconnection();
      tick();

      expect(component.connectionStatus()).toBe("connected");
    }));
  });
});
```

## 🎭 **Service Testing Strategies**

### **1. 🔧 HTTP Service Testing**

```typescript
// src/app/services/api.service.spec.ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { HttpErrorResponse } from "@angular/common/http";

import { ApiService } from "./api.service";
import { environment } from "../../environments/environment";

describe("ApiService", () => {
  let service: ApiService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [ApiService],
    });

    service = TestBed.inject(ApiService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify(); // Verify no outstanding HTTP requests
  });

  describe("GET Requests", () => {
    it("should retrieve users list", () => {
      const mockUsers = [
        { id: "1", name: "John", email: "john@example.com" },
        { id: "2", name: "Jane", email: "jane@example.com" },
      ];

      service.getUsers().subscribe((users) => {
        expect(users.length).toBe(2);
        expect(users).toEqual(mockUsers);
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users`);
      expect(req.request.method).toBe("GET");
      expect(req.request.headers.get("Accept")).toBe("application/json");

      req.flush(mockUsers);
    });

    it("should handle pagination parameters", () => {
      const paginationParams = { page: 2, size: 10, sort: "name" };

      service.getUsers(paginationParams).subscribe();

      const req = httpMock.expectOne(
        `${environment.apiUrl}/users?page=2&size=10&sort=name`
      );
      expect(req.request.method).toBe("GET");

      req.flush({ items: [], totalCount: 0 });
    });

    it("should include authentication headers", () => {
      service.setAuthToken("test-token");

      service.getUsers().subscribe();

      const req = httpMock.expectOne(`${environment.apiUrl}/users`);
      expect(req.request.headers.get("Authorization")).toBe(
        "Bearer test-token"
      );

      req.flush([]);
    });
  });

  describe("POST Requests", () => {
    it("should create a new user", () => {
      const newUser = {
        name: "New User",
        email: "new@example.com",
        role: "user",
      };
      const createdUser = { id: "3", ...newUser, createdAt: "2023-01-01" };

      service.createUser(newUser).subscribe((user) => {
        expect(user).toEqual(createdUser);
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users`);
      expect(req.request.method).toBe("POST");
      expect(req.request.body).toEqual(newUser);
      expect(req.request.headers.get("Content-Type")).toBe("application/json");

      req.flush(createdUser);
    });

    it("should handle validation errors", () => {
      const invalidUser = { name: "", email: "invalid-email" };
      const validationErrors = {
        errors: {
          name: ["Name is required"],
          email: ["Invalid email format"],
        },
      };

      service.createUser(invalidUser).subscribe({
        next: () => fail("Should not succeed"),
        error: (error: HttpErrorResponse) => {
          expect(error.status).toBe(400);
          expect(error.error).toEqual(validationErrors);
        },
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users`);
      req.flush(validationErrors, { status: 400, statusText: "Bad Request" });
    });
  });

  describe("PUT/PATCH Requests", () => {
    it("should update user data", () => {
      const userId = "1";
      const updateData = { name: "Updated Name", email: "updated@example.com" };
      const updatedUser = {
        id: userId,
        ...updateData,
        updatedAt: "2023-01-01",
      };

      service.updateUser(userId, updateData).subscribe((user) => {
        expect(user).toEqual(updatedUser);
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users/${userId}`);
      expect(req.request.method).toBe("PUT");
      expect(req.request.body).toEqual(updateData);

      req.flush(updatedUser);
    });

    it("should handle partial updates with PATCH", () => {
      const userId = "1";
      const patchData = { name: "Patched Name" };

      service.patchUser(userId, patchData).subscribe();

      const req = httpMock.expectOne(`${environment.apiUrl}/users/${userId}`);
      expect(req.request.method).toBe("PATCH");
      expect(req.request.body).toEqual(patchData);

      req.flush({ id: userId, name: "Patched Name" });
    });
  });

  describe("DELETE Requests", () => {
    it("should delete a user", () => {
      const userId = "1";

      service.deleteUser(userId).subscribe((result) => {
        expect(result).toBe(true);
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users/${userId}`);
      expect(req.request.method).toBe("DELETE");

      req.flush(null, { status: 204, statusText: "No Content" });
    });

    it("should handle delete conflicts", () => {
      const userId = "1";
      const conflictError = { message: "User has active dependencies" };

      service.deleteUser(userId).subscribe({
        next: () => fail("Should not succeed"),
        error: (error: HttpErrorResponse) => {
          expect(error.status).toBe(409);
          expect(error.error).toEqual(conflictError);
        },
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users/${userId}`);
      req.flush(conflictError, { status: 409, statusText: "Conflict" });
    });
  });

  describe("Error Handling", () => {
    it("should handle network errors", () => {
      service.getUsers().subscribe({
        next: () => fail("Should not succeed"),
        error: (error) => {
          expect(error.message).toContain("Network error");
        },
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users`);
      req.error(new ErrorEvent("Network error"), {
        status: 0,
        statusText: "Unknown Error",
      });
    });

    it("should handle server errors with retry", () => {
      let attemptCount = 0;

      service.getUsersWithRetry().subscribe({
        next: (users) => {
          expect(users).toEqual([]);
          expect(attemptCount).toBe(3); // Should retry twice before succeeding
        },
      });

      // First attempt - server error
      attemptCount++;
      const req1 = httpMock.expectOne(`${environment.apiUrl}/users`);
      req1.flush(null, { status: 500, statusText: "Internal Server Error" });

      // Second attempt - server error
      attemptCount++;
      const req2 = httpMock.expectOne(`${environment.apiUrl}/users`);
      req2.flush(null, { status: 500, statusText: "Internal Server Error" });

      // Third attempt - success
      attemptCount++;
      const req3 = httpMock.expectOne(`${environment.apiUrl}/users`);
      req3.flush([]);
    });

    it("should handle timeout scenarios", fakeAsync(() => {
      service.getUsersWithTimeout(1000).subscribe({
        next: () => fail("Should timeout"),
        error: (error) => {
          expect(error.message).toContain("timeout");
        },
      });

      const req = httpMock.expectOne(`${environment.apiUrl}/users`);

      tick(1000); // Simulate timeout

      // Request should be cancelled
      expect(req.cancelled).toBeTruthy();
    }));
  });

  describe("Interceptor Testing", () => {
    it("should add request timestamps", () => {
      service.getUsers().subscribe();

      const req = httpMock.expectOne(`${environment.apiUrl}/users`);
      expect(req.request.headers.get("X-Request-Timestamp")).toBeTruthy();

      req.flush([]);
    });

    it("should handle token refresh", () => {
      // Mock initial request with expired token
      service.setAuthToken("expired-token");

      service.getUsers().subscribe();

      // First request with expired token
      const req1 = httpMock.expectOne(`${environment.apiUrl}/users`);
      req1.flush(null, { status: 401, statusText: "Unauthorized" });

      // Token refresh request
      const tokenReq = httpMock.expectOne(`${environment.apiUrl}/auth/refresh`);
      tokenReq.flush({ token: "new-token" });

      // Retry original request with new token
      const req2 = httpMock.expectOne(`${environment.apiUrl}/users`);
      expect(req2.request.headers.get("Authorization")).toBe(
        "Bearer new-token"
      );
      req2.flush([]);
    });
  });
});
```

## 🎯 **Part 1 Summary**

This first part covers:

- **🧪 Testing Architecture** - Global test configuration and utilities
- **🎭 Mock Strategies** - Advanced mocking with builders and factories
- **🧩 Component Testing** - Signal-based components with async operations
- **🔧 Service Testing** - HTTP services with comprehensive scenarios

**Coming in Part 2:**

- Integration testing strategies
- E2E testing with modern frameworks
- Performance testing approaches
- Testing micro-frontends and complex architectures
