# 🎨 **Angular Style Tags & Style Guides Mastery**

## 🎯 **What You'll Learn**

Master **Angular styling approaches** and **official style guides** - the essential practices that make your Angular code consistent, maintainable, and professional! Think of this as your **coding etiquette handbook**.

---

## 📚 **The Basics (Start Here If You're New)**

### **What Are Angular Style Tags? 🤔**

Angular provides multiple ways to style your components, like having **different paintbrushes** for different artwork:

- 🖌️ **Inline styles** - Quick touch-ups
- 🎨 **Component stylesheets** - Dedicated component styling
- 🏗️ **Global styles** - Site-wide theming
- 💅 **Style libraries** - Professional styling frameworks

### **What Are Angular Style Guides? 📋**

Style guides are like **coding grammar rules** - they ensure your team writes consistent, readable code that everyone can understand and maintain.

---

## 🎨 **Angular Styling Approaches**

### **1. 💅 Component-Level Styling**

```typescript
// src/app/components/user-card/user-card.component.ts
import { Component } from "@angular/core";

@Component({
  selector: "app-user-card",
  template: `
    <div class="user-card">
      <img [src]="user.avatar" class="avatar" alt="User avatar" />
      <div class="user-info">
        <h3 class="user-name">{{ user.name }}</h3>
        <p class="user-email">{{ user.email }}</p>
        <span class="user-status" [class.online]="user.isOnline">
          {{ user.isOnline ? "Online" : "Offline" }}
        </span>
      </div>
      <div class="action-buttons">
        <button class="btn btn-primary" (click)="viewProfile()">View</button>
        <button class="btn btn-secondary" (click)="sendMessage()">
          Message
        </button>
      </div>
    </div>
  `,

  // 🎯 METHOD 1: External Stylesheet (Recommended)
  styleUrls: ["./user-card.component.scss"],

  // 🎯 METHOD 2: Inline Styles (For simple components)
  // styles: [`
  //   .user-card {
  //     background: white;
  //     border-radius: 8px;
  //     padding: 20px;
  //     box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  //   }
  // `]
})
export class UserCardComponent {
  user = {
    name: "John Doe",
    email: "john@example.com",
    avatar: "/assets/avatars/john.jpg",
    isOnline: true,
  };

  viewProfile() {
    console.log("Viewing profile...");
  }

  sendMessage() {
    console.log("Sending message...");
  }
}
```

```scss
// src/app/components/user-card/user-card.component.scss
// 🎨 Component-Scoped Styles (Encapsulated by default)

.user-card {
  background: var(--surface-color, #ffffff);
  border-radius: var(--border-radius, 12px);
  padding: var(--spacing-lg, 24px);
  box-shadow: var(--shadow-sm, 0 2px 8px rgba(0, 0, 0, 0.1));
  border: 1px solid var(--border-color, #e1e5e9);
  transition: all 0.2s ease;

  &:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-md, 0 4px 16px rgba(0, 0, 0, 0.15));
  }

  // 🖼️ Avatar Styling
  .avatar {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    object-fit: cover;
    border: 2px solid var(--primary-color, #007bff);
    margin-bottom: var(--spacing-md, 16px);
  }

  // 👤 User Information
  .user-info {
    flex: 1;

    .user-name {
      margin: 0 0 var(--spacing-xs, 8px) 0;
      font-size: var(--font-lg, 1.25rem);
      font-weight: var(--font-weight-semibold, 600);
      color: var(--text-primary, #1a1a1a);
    }

    .user-email {
      margin: 0 0 var(--spacing-sm, 12px) 0;
      font-size: var(--font-sm, 0.875rem);
      color: var(--text-secondary, #6b7280);
    }

    .user-status {
      display: inline-flex;
      align-items: center;
      font-size: var(--font-xs, 0.75rem);
      font-weight: var(--font-weight-medium, 500);
      padding: var(--spacing-xs, 4px) var(--spacing-sm, 8px);
      border-radius: var(--border-radius-sm, 6px);
      background: var(--error-light, #fee2e2);
      color: var(--error-dark, #dc2626);

      &::before {
        content: "";
        width: 6px;
        height: 6px;
        border-radius: 50%;
        background: currentColor;
        margin-right: var(--spacing-xs, 4px);
      }

      &.online {
        background: var(--success-light, #d1fae5);
        color: var(--success-dark, #065f46);
      }
    }
  }

  // 🔘 Action Buttons
  .action-buttons {
    display: flex;
    gap: var(--spacing-sm, 8px);
    margin-top: var(--spacing-md, 16px);

    .btn {
      padding: var(--spacing-sm, 8px) var(--spacing-md, 16px);
      border-radius: var(--border-radius-sm, 6px);
      border: none;
      font-size: var(--font-sm, 0.875rem);
      font-weight: var(--font-weight-medium, 500);
      cursor: pointer;
      transition: all 0.2s ease;

      &:hover {
        transform: translateY(-1px);
      }

      &:active {
        transform: translateY(0);
      }

      &.btn-primary {
        background: var(--primary-color, #007bff);
        color: white;

        &:hover {
          background: var(--primary-dark, #0056b3);
        }
      }

      &.btn-secondary {
        background: var(--gray-100, #f8f9fa);
        color: var(--text-primary, #1a1a1a);
        border: 1px solid var(--border-color, #e1e5e9);

        &:hover {
          background: var(--gray-200, #e9ecef);
        }
      }
    }
  }
}

// 📱 Responsive Design
@media (max-width: 480px) {
  .user-card {
    padding: var(--spacing-md, 16px);

    .action-buttons {
      flex-direction: column;

      .btn {
        width: 100%;
      }
    }
  }
}

// 🌙 Dark Mode Support
:host-context(.dark-theme) .user-card {
  background: var(--dark-surface, #1f2937);
  border-color: var(--dark-border, #374151);
  color: var(--dark-text, #f9fafb);

  .user-name {
    color: var(--dark-text-primary, #ffffff);
  }

  .user-email {
    color: var(--dark-text-secondary, #9ca3af);
  }
}
```

### **2. 🌍 Global Styling Setup**

```scss
// src/styles.scss - Global styles
@import "styles/variables";
@import "styles/typography";
@import "styles/components";
@import "styles/utilities";

// 🎨 CSS Custom Properties (Design System)
:root {
  // 🎯 Brand Colors
  --primary-50: #eff6ff;
  --primary-100: #dbeafe;
  --primary-200: #bfdbfe;
  --primary-300: #93c5fd;
  --primary-400: #60a5fa;
  --primary-500: #3b82f6; // Main brand color
  --primary-600: #2563eb;
  --primary-700: #1d4ed8;
  --primary-800: #1e40af;
  --primary-900: #1e3a8a;

  // 🔘 Neutral Colors
  --gray-50: #f9fafb;
  --gray-100: #f3f4f6;
  --gray-200: #e5e7eb;
  --gray-300: #d1d5db;
  --gray-400: #9ca3af;
  --gray-500: #6b7280;
  --gray-600: #4b5563;
  --gray-700: #374151;
  --gray-800: #1f2937;
  --gray-900: #111827;

  // 🎯 Semantic Colors
  --success-light: #d1fae5;
  --success: #10b981;
  --success-dark: #065f46;

  --warning-light: #fef3c7;
  --warning: #f59e0b;
  --warning-dark: #92400e;

  --error-light: #fee2e2;
  --error: #ef4444;
  --error-dark: #dc2626;

  // 📏 Spacing Scale
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  --spacing-2xl: 48px;
  --spacing-3xl: 64px;

  // 📖 Typography
  --font-family-sans: "Inter", "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  --font-family-mono: "Fira Code", "Consolas", "Monaco", monospace;

  --font-size-xs: 0.75rem; // 12px
  --font-size-sm: 0.875rem; // 14px
  --font-size-base: 1rem; // 16px
  --font-size-lg: 1.125rem; // 18px
  --font-size-xl: 1.25rem; // 20px
  --font-size-2xl: 1.5rem; // 24px
  --font-size-3xl: 1.875rem; // 30px

  --font-weight-light: 300;
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  // 🎨 Border & Radius
  --border-radius-sm: 6px;
  --border-radius: 8px;
  --border-radius-lg: 12px;
  --border-radius-full: 9999px;

  // 🌫️ Shadows
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);

  // ⚡ Transitions
  --transition-fast: 0.15s ease;
  --transition-base: 0.2s ease;
  --transition-slow: 0.3s ease;
}

// 🌙 Dark Theme
.dark-theme {
  --surface-color: var(--gray-800);
  --text-primary: var(--gray-100);
  --text-secondary: var(--gray-400);
  --border-color: var(--gray-700);
}

// 🔄 CSS Reset & Base Styles
*,
*::before,
*::after {
  box-sizing: border-box;
}

html {
  line-height: 1.15;
  -webkit-text-size-adjust: 100%;
}

body {
  margin: 0;
  font-family: var(--font-family-sans);
  font-size: var(--font-size-base);
  line-height: 1.5;
  color: var(--text-primary, var(--gray-900));
  background-color: var(--surface-color, var(--gray-50));
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

// 🎯 Utility Classes
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

.text-center {
  text-align: center;
}
.text-left {
  text-align: left;
}
.text-right {
  text-align: right;
}

.flex {
  display: flex;
}
.inline-flex {
  display: inline-flex;
}
.grid {
  display: grid;
}
.block {
  display: block;
}
.inline-block {
  display: inline-block;
}
.hidden {
  display: none;
}

.items-center {
  align-items: center;
}
.items-start {
  align-items: flex-start;
}
.items-end {
  align-items: flex-end;
}

.justify-center {
  justify-content: center;
}
.justify-between {
  justify-content: space-between;
}
.justify-start {
  justify-content: flex-start;
}
.justify-end {
  justify-content: flex-end;
}
```

### **3. 🎯 View Encapsulation Strategies**

```typescript
// src/app/components/styled-component/styled-component.component.ts
import { Component, ViewEncapsulation } from "@angular/core";

// 🔒 METHOD 1: Default Encapsulation (Recommended)
@Component({
  selector: "app-default-encapsulation",
  template: `
    <div class="container">
      <h2 class="title">Default Encapsulation</h2>
      <p class="content">Styles are scoped to this component only</p>
    </div>
  `,
  styles: [
    `
      .container {
        background: lightblue;
        padding: 20px;
      }
      .title {
        color: darkblue;
      }
    `,
  ],
  // ViewEncapsulation.Emulated is the default
  encapsulation: ViewEncapsulation.Emulated,
})
export class DefaultEncapsulationComponent {}

// 🌍 METHOD 2: No Encapsulation (Global Styles)
@Component({
  selector: "app-global-styles",
  template: `
    <div class="global-container">
      <h2 class="global-title">Global Styles</h2>
      <p class="global-content">These styles affect the entire application</p>
    </div>
  `,
  styles: [
    `
      .global-container {
        background: lightcoral;
        padding: 20px;
      }
      .global-title {
        color: darkred;
      }
    `,
  ],
  encapsulation: ViewEncapsulation.None, // ⚠️ Use carefully!
})
export class GlobalStylesComponent {}

// 🎯 METHOD 3: Shadow DOM (Native Encapsulation)
@Component({
  selector: "app-shadow-dom",
  template: `
    <div class="shadow-container">
      <h2 class="shadow-title">Shadow DOM</h2>
      <p class="shadow-content">Uses native browser Shadow DOM</p>
    </div>
  `,
  styles: [
    `
      .shadow-container {
        background: lightgreen;
        padding: 20px;
      }
      .shadow-title {
        color: darkgreen;
      }
    `,
  ],
  encapsulation: ViewEncapsulation.ShadowDom, // Modern browsers only
})
export class ShadowDomComponent {}
```

---

## 📋 **Angular Official Style Guide**

### **1. 🏗️ File Naming Conventions**

```typescript
// ✅ CORRECT: Descriptive, kebab-case names
user - profile.component.ts; // Component files
user - profile.component.html; // Template files
user - profile.component.scss; // Stylesheet files
user - profile.component.spec.ts; // Test files

authentication.service.ts; // Service files
form - validation.directive.ts; // Directive files
currency.pipe.ts; // Pipe files
auth.guard.ts; // Guard files
user.interface.ts; // Interface files
api.constants.ts; // Constants files

// ❌ INCORRECT: Poor naming
UserProfile.component.ts; // PascalCase files
userprofile.component.ts; // No separation
user_profile.component.ts; // Snake_case
userProfileComp.ts; // Abbreviated
UP.component.ts; // Too short
```

### **2. 🎯 Component Structure Standards**

```typescript
// src/app/features/user-management/components/user-list/user-list.component.ts

// ✅ CORRECT: Well-structured component
import {
  Component,
  OnInit,
  OnDestroy,
  Input,
  Output,
  EventEmitter,
} from "@angular/core";
import { Observable, Subject } from "rxjs";
import { takeUntil } from "rxjs/operators";

import { User } from "../../models/user.interface";
import { UserService } from "../../services/user.service";

@Component({
  selector: "app-user-list", // ✅ Prefixed with 'app'
  templateUrl: "./user-list.component.html",
  styleUrls: ["./user-list.component.scss"],
})
export class UserListComponent implements OnInit, OnDestroy {
  // 📋 PUBLIC PROPERTIES (Template bindings)
  @Input() users: User[] = [];
  @Input() loading = false;
  @Input() searchTerm = "";

  @Output() userSelected = new EventEmitter<User>();
  @Output() userDeleted = new EventEmitter<string>();

  // 🔍 PUBLIC COMPUTED PROPERTIES
  get filteredUsers(): User[] {
    if (!this.searchTerm) return this.users;

    return this.users.filter(
      (user) =>
        user.name.toLowerCase().includes(this.searchTerm.toLowerCase()) ||
        user.email.toLowerCase().includes(this.searchTerm.toLowerCase())
    );
  }

  // 🔒 PRIVATE PROPERTIES
  private destroy$ = new Subject<void>();
  private readonly pageSize = 20;

  // 🏗️ CONSTRUCTOR (Dependency Injection)
  constructor(private userService: UserService) {}

  // 🚀 LIFECYCLE HOOKS
  ngOnInit(): void {
    this.loadUsers();
    this.setupUserUpdates();
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  // 🎯 PUBLIC METHODS (Template interactions)
  onUserClick(user: User): void {
    this.userSelected.emit(user);
  }

  onDeleteUser(userId: string): void {
    if (confirm("Are you sure you want to delete this user?")) {
      this.userDeleted.emit(userId);
    }
  }

  trackByUserId(index: number, user: User): string {
    return user.id; // ✅ Always provide trackBy for *ngFor
  }

  // 🔒 PRIVATE METHODS (Internal logic)
  private loadUsers(): void {
    this.userService
      .getUsers()
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (users) => {
          this.users = users;
        },
        error: (error) => {
          console.error("Failed to load users:", error);
        },
      });
  }

  private setupUserUpdates(): void {
    this.userService.userUpdates$
      .pipe(takeUntil(this.destroy$))
      .subscribe((updatedUser) => {
        const index = this.users.findIndex((u) => u.id === updatedUser.id);
        if (index >= 0) {
          this.users[index] = updatedUser;
        }
      });
  }
}
```

### **3. 📝 Template Best Practices**

```html
<!-- src/app/features/user-management/components/user-list/user-list.component.html -->

<!-- ✅ CORRECT: Clean, semantic template -->
<div class="user-list-container">
  <!-- 🔍 Search Section -->
  <div class="search-section">
    <label for="user-search" class="sr-only">Search users</label>
    <input
      id="user-search"
      type="text"
      [(ngModel)]="searchTerm"
      placeholder="Search users..."
      class="search-input"
      autocomplete="off"
    />
  </div>

  <!-- 📊 Loading State -->
  <div
    *ngIf="loading"
    class="loading-container"
    role="status"
    aria-live="polite"
  >
    <div class="loading-spinner" aria-hidden="true"></div>
    <span class="loading-text">Loading users...</span>
  </div>

  <!-- 📋 Users List -->
  <div *ngIf="!loading && filteredUsers.length > 0" class="users-grid">
    <div
      *ngFor="let user of filteredUsers; trackBy: trackByUserId"
      class="user-card"
      [attr.aria-label]="'User: ' + user.name"
      tabindex="0"
      role="button"
      (click)="onUserClick(user)"
      (keydown.enter)="onUserClick(user)"
      (keydown.space)="onUserClick(user)"
    >
      <!-- 👤 User Avatar -->
      <img
        [src]="user.avatar || '/assets/default-avatar.svg'"
        [alt]="user.name + ' avatar'"
        class="user-avatar"
        loading="lazy"
      />

      <!-- 📝 User Info -->
      <div class="user-info">
        <h3 class="user-name">{{ user.name }}</h3>
        <p class="user-email">{{ user.email }}</p>
        <span
          class="user-status"
          [class.online]="user.isOnline"
          [attr.aria-label]="user.isOnline ? 'Online' : 'Offline'"
        >
          {{ user.isOnline ? 'Online' : 'Offline' }}
        </span>
      </div>

      <!-- 🔘 Actions -->
      <div class="user-actions">
        <button
          type="button"
          class="btn btn-danger btn-sm"
          (click)="onDeleteUser(user.id); $event.stopPropagation()"
          [attr.aria-label]="'Delete user ' + user.name"
        >
          <span aria-hidden="true">🗑️</span>
          Delete
        </button>
      </div>
    </div>
  </div>

  <!-- 🚫 Empty State -->
  <div *ngIf="!loading && filteredUsers.length === 0" class="empty-state">
    <div class="empty-icon" aria-hidden="true">👤</div>
    <h3 class="empty-title">No users found</h3>
    <p class="empty-description">
      <span *ngIf="searchTerm; else noUsersMessage">
        No users match your search for "{{ searchTerm }}"
      </span>
      <ng-template #noUsersMessage> There are no users to display </ng-template>
    </p>
  </div>
</div>
```

### **4. 🔧 Service Structure Standards**

```typescript
// src/app/core/services/user.service.ts

// ✅ CORRECT: Well-structured service
import { Injectable } from "@angular/core";
import { HttpClient, HttpErrorResponse } from "@angular/common/http";
import { BehaviorSubject, Observable, throwError } from "rxjs";
import { map, catchError, tap, retry } from "rxjs/operators";

import {
  User,
  CreateUserRequest,
  UpdateUserRequest,
} from "../models/user.interface";
import { ApiResponse, PaginatedResponse } from "../models/api.interface";
import { NotificationService } from "./notification.service";

@Injectable({
  providedIn: "root", // ✅ Use providedIn: 'root' for singletons
})
export class UserService {
  // 🔗 API Configuration
  private readonly apiUrl = "/api/users";

  // 📊 State Management
  private usersSubject = new BehaviorSubject<User[]>([]);
  private loadingSubject = new BehaviorSubject<boolean>(false);

  // 📡 Public Observables
  readonly users$ = this.usersSubject.asObservable();
  readonly loading$ = this.loadingSubject.asObservable();
  readonly userUpdates$ = new BehaviorSubject<User | null>(null);

  // 🏗️ Constructor
  constructor(
    private http: HttpClient,
    private notificationService: NotificationService
  ) {}

  // 📋 GET METHODS
  getUsers(page = 1, limit = 20): Observable<User[]> {
    this.loadingSubject.next(true);

    return this.http
      .get<PaginatedResponse<User>>(`${this.apiUrl}`, {
        params: { page: page.toString(), limit: limit.toString() },
      })
      .pipe(
        map((response) => response.data),
        tap((users) => {
          this.usersSubject.next(users);
          this.loadingSubject.next(false);
        }),
        catchError(this.handleError.bind(this)),
        retry(2) // ✅ Retry failed requests
      );
  }

  getUserById(id: string): Observable<User> {
    return this.http.get<ApiResponse<User>>(`${this.apiUrl}/${id}`).pipe(
      map((response) => response.data),
      catchError(this.handleError.bind(this))
    );
  }

  searchUsers(query: string): Observable<User[]> {
    if (!query.trim()) {
      return this.users$;
    }

    return this.http
      .get<PaginatedResponse<User>>(`${this.apiUrl}/search`, {
        params: { q: query },
      })
      .pipe(
        map((response) => response.data),
        catchError(this.handleError.bind(this))
      );
  }

  // ✏️ CREATE/UPDATE METHODS
  createUser(userData: CreateUserRequest): Observable<User> {
    return this.http.post<ApiResponse<User>>(this.apiUrl, userData).pipe(
      map((response) => response.data),
      tap((newUser) => {
        // Update local state
        const currentUsers = this.usersSubject.value;
        this.usersSubject.next([newUser, ...currentUsers]);

        // Notify about the update
        this.userUpdates$.next(newUser);
        this.notificationService.success("User created successfully");
      }),
      catchError(this.handleError.bind(this))
    );
  }

  updateUser(id: string, userData: UpdateUserRequest): Observable<User> {
    return this.http
      .put<ApiResponse<User>>(`${this.apiUrl}/${id}`, userData)
      .pipe(
        map((response) => response.data),
        tap((updatedUser) => {
          // Update local state
          const currentUsers = this.usersSubject.value;
          const index = currentUsers.findIndex((u) => u.id === id);
          if (index >= 0) {
            currentUsers[index] = updatedUser;
            this.usersSubject.next([...currentUsers]);
          }

          // Notify about the update
          this.userUpdates$.next(updatedUser);
          this.notificationService.success("User updated successfully");
        }),
        catchError(this.handleError.bind(this))
      );
  }

  // 🗑️ DELETE METHOD
  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      tap(() => {
        // Update local state
        const currentUsers = this.usersSubject.value;
        const filteredUsers = currentUsers.filter((u) => u.id !== id);
        this.usersSubject.next(filteredUsers);

        this.notificationService.success("User deleted successfully");
      }),
      catchError(this.handleError.bind(this))
    );
  }

  // 🔄 UTILITY METHODS
  refreshUsers(): Observable<User[]> {
    return this.getUsers();
  }

  clearCache(): void {
    this.usersSubject.next([]);
  }

  // 🚨 ERROR HANDLING
  private handleError(error: HttpErrorResponse): Observable<never> {
    let errorMessage = "An unknown error occurred";

    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Error: ${error.error.message}`;
    } else {
      // Server-side error
      switch (error.status) {
        case 400:
          errorMessage = "Bad request. Please check your input.";
          break;
        case 401:
          errorMessage = "Unauthorized. Please log in again.";
          break;
        case 403:
          errorMessage = "Forbidden. You don't have permission.";
          break;
        case 404:
          errorMessage = "User not found.";
          break;
        case 500:
          errorMessage = "Server error. Please try again later.";
          break;
        default:
          errorMessage = `Error ${error.status}: ${error.message}`;
      }
    }

    // Show user-friendly error
    this.notificationService.error(errorMessage);

    // Reset loading state
    this.loadingSubject.next(false);

    // Return error for component handling
    return throwError(() => new Error(errorMessage));
  }
}
```

---

## 🎯 **Advanced Style Guide Patterns**

### **1. 🏗️ Folder Structure Standards**

```
src/app/
├── core/                          # 🏛️ Singleton services, guards, interceptors
│   ├── guards/
│   ├── interceptors/
│   ├── services/
│   └── models/
├── shared/                        # 🤝 Reusable components, pipes, directives
│   ├── components/
│   ├── directives/
│   ├── pipes/
│   └── utils/
├── features/                      # 🎯 Feature modules
│   ├── authentication/
│   │   ├── components/
│   │   ├── services/
│   │   ├── models/
│   │   └── authentication.module.ts
│   └── user-management/
│       ├── components/
│       │   ├── user-list/
│       │   │   ├── user-list.component.ts
│       │   │   ├── user-list.component.html
│       │   │   ├── user-list.component.scss
│       │   │   └── user-list.component.spec.ts
│       │   └── user-detail/
│       ├── services/
│       ├── models/
│       └── user-management.module.ts
└── layout/                        # 📱 App layout components
    ├── header/
    ├── footer/
    └── sidebar/
```

### **2. 📛 Naming Convention Cheat Sheet**

```typescript
// 🎯 COMPONENTS
UserProfileComponent; // ✅ PascalCase class names
user - profile.component.ts < // ✅ kebab-case file names
  app - user - profile > // ✅ kebab-case selectors
  // 🔧 SERVICES
  UserManagementService; // ✅ PascalCase, descriptive
authentication.service.ts; // ✅ kebab-case file names

// 🚪 DIRECTIVES
HighlightDirective[appHighlight]; // ✅ PascalCase class names // ✅ camelCase selectors with prefix

// 🔄 PIPES
CurrencyFormatPipe; // ✅ PascalCase class names
currency - format.pipe.ts; // ✅ kebab-case file names
{
  {
    price | currencyFormat;
  }
} // ✅ camelCase pipe usage

// 📋 INTERFACES/MODELS
export interface User {
  // ✅ PascalCase, no prefix/suffix
  id: string;
  name: string;
}

// 🔐 ENUMS
export enum UserRole { // ✅ PascalCase
  ADMIN = "admin",
  USER = "user",
  GUEST = "guest",
}

// 📊 CONSTANTS
export const API_ENDPOINTS = {
  // ✅ UPPER_SNAKE_CASE
  USERS: "/api/users",
  AUTH: "/api/auth",
};
```

### **3. 💬 Documentation Standards**

````typescript
/**
 * User management service providing CRUD operations for user data
 *
 * @example
 * ```typescript
 * constructor(private userService: UserService) {}
 *
 * ngOnInit() {
 *   this.userService.getUsers().subscribe(users => {
 *     console.log('Loaded users:', users);
 *   });
 * }
 * ```
 *
 * @since 1.0.0
 * @author Development Team
 */
@Injectable({
  providedIn: "root",
})
export class UserService {
  /**
   * Retrieves a paginated list of users
   *
   * @param page - Page number (1-based)
   * @param limit - Number of users per page
   * @returns Observable of user array
   *
   * @example
   * ```typescript
   * this.userService.getUsers(1, 10).subscribe(users => {
   *   this.displayUsers = users;
   * });
   * ```
   */
  getUsers(page = 1, limit = 20): Observable<User[]> {
    // Implementation...
  }

  /**
   * Creates a new user account
   *
   * @param userData - User information for account creation
   * @returns Observable of created user
   * @throws {ValidationError} When user data is invalid
   * @throws {ConflictError} When email already exists
   *
   * @example
   * ```typescript
   * const newUser = {
   *   name: 'John Doe',
   *   email: 'john@example.com',
   *   role: UserRole.USER
   * };
   *
   * this.userService.createUser(newUser).subscribe({
   *   next: (user) => console.log('Created:', user),
   *   error: (error) => console.error('Creation failed:', error)
   * });
   * ```
   */
  createUser(userData: CreateUserRequest): Observable<User> {
    // Implementation...
  }
}
````

---

## ⚡ **Performance & Accessibility Guidelines**

### **🚀 Performance Best Practices**

```typescript
// ✅ CORRECT: OnPush strategy for performance
@Component({
  selector: "app-high-performance",
  template: `
    <div class="performance-optimized">
      <!-- ✅ Use trackBy for large lists -->
      <div *ngFor="let item of items; trackBy: trackByItemId">
        {{ item.name }}
      </div>

      <!-- ✅ Async pipe for observables -->
      <div *ngIf="user$ | async as user">Welcome, {{ user.name }}!</div>

      <!-- ✅ Lazy loading for images -->
      <img [src]="imageUrl" loading="lazy" alt="Description" />
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush, // ✅ OnPush strategy
})
export class HighPerformanceComponent {
  // ✅ Proper trackBy function
  trackByItemId(index: number, item: any): string {
    return item.id; // Use unique identifier, not index
  }

  // ✅ Unsubscribe from observables
  private destroy$ = new Subject<void>();

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### ♿ **Accessibility Standards**

```html
<!-- ✅ CORRECT: Accessible component template -->
<section class="user-form" role="main">
  <h1 id="form-title">Create User Account</h1>

  <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
    <!-- ✅ Proper labeling -->
    <div class="form-field">
      <label for="user-name">Full Name *</label>
      <input
        id="user-name"
        type="text"
        formControlName="name"
        [attr.aria-describedby]="nameError ? 'name-error' : null"
        [attr.aria-invalid]="nameError ? 'true' : 'false'"
        required
      />
      <div
        id="name-error"
        *ngIf="nameError"
        class="error-message"
        role="alert"
        aria-live="polite"
      >
        {{ nameError }}
      </div>
    </div>

    <!-- ✅ Button accessibility -->
    <button
      type="submit"
      [disabled]="userForm.invalid"
      [attr.aria-describedby]="userForm.invalid ? 'form-errors' : null"
    >
      <span *ngIf="loading" aria-hidden="true">⏳</span>
      {{ loading ? 'Creating...' : 'Create User' }}
    </button>

    <!-- ✅ Error summary -->
    <div
      id="form-errors"
      *ngIf="userForm.invalid && userForm.touched"
      class="error-summary"
      role="alert"
      aria-live="assertive"
    >
      <h2>Please fix the following errors:</h2>
      <ul>
        <li *ngFor="let error of formErrors">{{ error }}</li>
      </ul>
    </div>
  </form>
</section>
```

---

## 🎉 **Summary: Angular Style Mastery**

You now know how to:

### **🏗️ What You've Mastered:**

✅ **Component Styling** - Multiple approaches and best practices  
✅ **View Encapsulation** - Understanding different encapsulation strategies  
✅ **Global Style Systems** - CSS custom properties and design systems  
✅ **File Organization** - Proper folder structure and naming conventions  
✅ **Code Standards** - Angular's official style guide compliance  
✅ **Performance Optimization** - Fast, efficient styling patterns  
✅ **Accessibility** - WCAG-compliant styling and markup

### **🚀 Real-World Benefits:**

- **Consistent codebase** that any team member can understand
- **Maintainable styles** with proper encapsulation and organization
- **Better performance** with optimized styling strategies
- **Accessible applications** that work for all users
- **Professional code quality** following industry standards

### **🎯 Key Style Guide Areas:**

- **Naming Conventions** - Files, components, services, and variables
- **Code Organization** - Folder structure and file placement
- **Component Structure** - Property order and method organization
- **Template Standards** - Clean, semantic HTML with accessibility
- **Documentation** - Proper JSDoc comments and examples
- **Performance** - OnPush strategy and optimization techniques

**Remember**: Following style guides isn't just about rules - it's about creating code that's easy to read, maintain, and scale! 🌟
