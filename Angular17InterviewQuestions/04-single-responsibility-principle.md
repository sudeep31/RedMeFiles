# 🎯 Single Responsibility Principle (SRP) in Angular

## 🎯 **Question Overview**

_"What is SRP - Single Responsibility Principle?"_

## 🔍 **Understanding Single Responsibility Principle**

The **Single Responsibility Principle (SRP)** is the first principle of SOLID design principles, coined by Robert C. Martin. It states:

> **"A class should have only one reason to change, meaning it should have only one job or responsibility."**

In simpler terms: **Do one thing, and do it well!** 🎯

## 📚 **Core Concepts**

### **What SRP Means**

- Each class/module should be responsible for **one specific functionality**
- Changes to one aspect of the software should only affect one class
- High **cohesion** within a class, loose **coupling** between classes
- Makes code more **maintainable**, **testable**, and **readable**

### **Why SRP Matters**

1. **🔧 Easier Maintenance**: Less code to understand and modify
2. **🧪 Better Testing**: Isolated functionality is easier to test
3. **🚀 Improved Reusability**: Single-purpose components can be reused
4. **🛡️ Reduced Risk**: Changes in one area don't break other functionality
5. **👥 Better Collaboration**: Team members can work on separate concerns

## 🛠️ **SRP in Angular Context**

### **❌ Violating SRP - Bad Example**

```typescript
// ❌ BAD: This component has too many responsibilities
@Component({
  selector: "app-user-management",
  template: `
    <div class="user-management">
      <!-- User list display -->
      <div class="user-list">
        <div *ngFor="let user of users" class="user-item">
          {{ user.name }} - {{ user.email }}
          <button (click)="editUser(user)">Edit</button>
          <button (click)="deleteUser(user.id)">Delete</button>
        </div>
      </div>

      <!-- User form -->
      <form (ngSubmit)="saveUser()">
        <input [(ngModel)]="currentUser.name" placeholder="Name" />
        <input [(ngModel)]="currentUser.email" placeholder="Email" />
        <button type="submit">Save</button>
      </form>

      <!-- Notifications -->
      <div class="notifications" *ngIf="message">
        {{ message }}
      </div>
    </div>
  `,
})
export class UserManagementComponent {
  users: User[] = [];
  currentUser: User = { id: 0, name: "", email: "" };
  message: string = "";

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.loadUsers();
  }

  // Responsibility 1: Data fetching and HTTP operations
  loadUsers() {
    this.http.get<User[]>("/api/users").subscribe({
      next: (users) => {
        this.users = users;
        this.showMessage("Users loaded successfully");
      },
      error: (error) => {
        this.showMessage("Error loading users: " + error.message);
        this.logError("Failed to load users", error);
      },
    });
  }

  saveUser() {
    const url = this.currentUser.id
      ? `/api/users/${this.currentUser.id}`
      : "/api/users";
    const request = this.currentUser.id
      ? this.http.put(url, this.currentUser)
      : this.http.post(url, this.currentUser);

    request.subscribe({
      next: () => {
        this.loadUsers();
        this.resetForm();
        this.showMessage("User saved successfully");
      },
      error: (error) => {
        this.showMessage("Error saving user: " + error.message);
        this.logError("Failed to save user", error);
      },
    });
  }

  deleteUser(id: number) {
    this.http.delete(`/api/users/${id}`).subscribe({
      next: () => {
        this.loadUsers();
        this.showMessage("User deleted successfully");
      },
      error: (error) => {
        this.showMessage("Error deleting user: " + error.message);
        this.logError("Failed to delete user", error);
      },
    });
  }

  // Responsibility 2: Form management
  editUser(user: User) {
    this.currentUser = { ...user };
  }

  resetForm() {
    this.currentUser = { id: 0, name: "", email: "" };
  }

  validateUser(): boolean {
    return (
      this.currentUser.name.length > 0 && this.currentUser.email.includes("@")
    );
  }

  // Responsibility 3: UI state management
  showMessage(message: string) {
    this.message = message;
    setTimeout(() => {
      this.message = "";
    }, 3000);
  }

  // Responsibility 4: Logging and error handling
  logError(message: string, error: any) {
    console.error(message, error);
    // Send to error tracking service
    // Save to local storage
    // Send analytics event
  }

  // Responsibility 5: Data transformation
  formatUserDisplayName(user: User): string {
    return `${user.name} (${user.email})`;
  }

  sortUsersByName(): User[] {
    return this.users.sort((a, b) => a.name.localeCompare(b.name));
  }
}
```

### **✅ Following SRP - Good Example**

Let's break down the above component into separate responsibilities:

#### **1. User Service (Data Access)**

```typescript
// ✅ GOOD: Service responsible only for user data operations
@Injectable({
  providedIn: "root",
})
export class UserService {
  private readonly apiUrl = "/api/users";

  constructor(
    private http: HttpClient,
    private errorHandler: ErrorHandlerService
  ) {}

  getUsers(): Observable<User[]> {
    return this.http
      .get<User[]>(this.apiUrl)
      .pipe(
        catchError((error) =>
          this.errorHandler.handleError("Failed to load users", error)
        )
      );
  }

  createUser(user: Omit<User, "id">): Observable<User> {
    return this.http
      .post<User>(this.apiUrl, user)
      .pipe(
        catchError((error) =>
          this.errorHandler.handleError("Failed to create user", error)
        )
      );
  }

  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http
      .put<User>(`${this.apiUrl}/${id}`, user)
      .pipe(
        catchError((error) =>
          this.errorHandler.handleError("Failed to update user", error)
        )
      );
  }

  deleteUser(id: number): Observable<void> {
    return this.http
      .delete<void>(`${this.apiUrl}/${id}`)
      .pipe(
        catchError((error) =>
          this.errorHandler.handleError("Failed to delete user", error)
        )
      );
  }
}
```

#### **2. User Validator Service (Validation Logic)**

```typescript
// ✅ GOOD: Service responsible only for user validation
@Injectable({
  providedIn: "root",
})
export class UserValidatorService {
  validateUser(user: Partial<User>): ValidationResult {
    const errors: string[] = [];

    if (!user.name || user.name.trim().length < 2) {
      errors.push("Name must be at least 2 characters long");
    }

    if (!user.email || !this.isValidEmail(user.email)) {
      errors.push("Please enter a valid email address");
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  private isValidEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }

  validateName(name: string): boolean {
    return name && name.trim().length >= 2;
  }

  validateEmail(email: string): boolean {
    return this.isValidEmail(email);
  }
}
```

#### **3. User List Component (Display Users)**

```typescript
// ✅ GOOD: Component responsible only for displaying user list
@Component({
  selector: "app-user-list",
  template: `
    <div class="user-list">
      <app-user-item
        *ngFor="let user of users"
        [user]="user"
        (edit)="onEdit($event)"
        (delete)="onDelete($event)"
      >
      </app-user-item>
    </div>
  `,
  standalone: true,
  imports: [CommonModule, UserItemComponent],
})
export class UserListComponent {
  @Input() users: User[] = [];
  @Output() editUser = new EventEmitter<User>();
  @Output() deleteUser = new EventEmitter<number>();

  onEdit(user: User) {
    this.editUser.emit(user);
  }

  onDelete(userId: number) {
    this.deleteUser.emit(userId);
  }
}
```

#### **4. User Form Component (Form Management)**

```typescript
// ✅ GOOD: Component responsible only for user form
@Component({
  selector: "app-user-form",
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div class="form-group">
        <input
          formControlName="name"
          placeholder="Name"
          [class.error]="nameControl?.invalid && nameControl?.touched"
        />
        <div
          *ngIf="nameControl?.invalid && nameControl?.touched"
          class="error-message"
        >
          {{ getNameErrorMessage() }}
        </div>
      </div>

      <div class="form-group">
        <input
          formControlName="email"
          placeholder="Email"
          [class.error]="emailControl?.invalid && emailControl?.touched"
        />
        <div
          *ngIf="emailControl?.invalid && emailControl?.touched"
          class="error-message"
        >
          {{ getEmailErrorMessage() }}
        </div>
      </div>

      <div class="form-actions">
        <button type="submit" [disabled]="userForm.invalid">
          {{ isEditing ? "Update" : "Create" }} User
        </button>
        <button type="button" (click)="onReset()">Reset</button>
      </div>
    </form>
  `,
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
})
export class UserFormComponent implements OnInit, OnChanges {
  @Input() user: User | null = null;
  @Output() save = new EventEmitter<Omit<User, "id"> | User>();
  @Output() reset = new EventEmitter<void>();

  userForm!: FormGroup;
  isEditing = false;

  constructor(
    private fb: FormBuilder,
    private validator: UserValidatorService
  ) {}

  ngOnInit() {
    this.initializeForm();
  }

  ngOnChanges(changes: SimpleChanges) {
    if (changes["user"] && this.userForm) {
      this.updateForm();
    }
  }

  get nameControl() {
    return this.userForm.get("name");
  }
  get emailControl() {
    return this.userForm.get("email");
  }

  private initializeForm() {
    this.userForm = this.fb.group({
      name: ["", [Validators.required, Validators.minLength(2)]],
      email: ["", [Validators.required, Validators.email]],
    });
  }

  private updateForm() {
    if (this.user) {
      this.isEditing = true;
      this.userForm.patchValue({
        name: this.user.name,
        email: this.user.email,
      });
    } else {
      this.isEditing = false;
      this.userForm.reset();
    }
  }

  onSubmit() {
    if (this.userForm.valid) {
      const formValue = this.userForm.value;

      if (this.isEditing && this.user) {
        this.save.emit({ ...this.user, ...formValue });
      } else {
        this.save.emit(formValue);
      }
    }
  }

  onReset() {
    this.userForm.reset();
    this.isEditing = false;
    this.reset.emit();
  }

  getNameErrorMessage(): string {
    if (this.nameControl?.hasError("required")) {
      return "Name is required";
    }
    if (this.nameControl?.hasError("minlength")) {
      return "Name must be at least 2 characters";
    }
    return "";
  }

  getEmailErrorMessage(): string {
    if (this.emailControl?.hasError("required")) {
      return "Email is required";
    }
    if (this.emailControl?.hasError("email")) {
      return "Please enter a valid email";
    }
    return "";
  }
}
```

#### **5. Notification Service (UI Feedback)**

```typescript
// ✅ GOOD: Service responsible only for notifications
@Injectable({
  providedIn: "root",
})
export class NotificationService {
  private notificationSubject = new BehaviorSubject<Notification | null>(null);
  public notification$ = this.notificationSubject.asObservable();

  showSuccess(message: string, duration: number = 3000) {
    this.showNotification({
      type: "success",
      message,
      duration,
    });
  }

  showError(message: string, duration: number = 5000) {
    this.showNotification({
      type: "error",
      message,
      duration,
    });
  }

  showInfo(message: string, duration: number = 3000) {
    this.showNotification({
      type: "info",
      message,
      duration,
    });
  }

  private showNotification(notification: Notification) {
    this.notificationSubject.next(notification);

    if (notification.duration > 0) {
      setTimeout(() => {
        this.clearNotification();
      }, notification.duration);
    }
  }

  clearNotification() {
    this.notificationSubject.next(null);
  }
}
```

#### **6. Main Container Component (Orchestration)**

```typescript
// ✅ GOOD: Component responsible only for orchestrating user management
@Component({
  selector: "app-user-management-container",
  template: `
    <div class="user-management">
      <h1>User Management</h1>

      <app-user-form
        [user]="selectedUser"
        (save)="onSaveUser($event)"
        (reset)="onResetForm()"
      >
      </app-user-form>

      <app-user-list
        [users]="users$ | async || []"
        (editUser)="onEditUser($event)"
        (deleteUser)="onDeleteUser($event)"
      >
      </app-user-list>

      <app-notification></app-notification>
    </div>
  `,
  standalone: true,
  imports: [
    CommonModule,
    UserFormComponent,
    UserListComponent,
    NotificationComponent,
  ],
})
export class UserManagementContainerComponent implements OnInit {
  users$ = this.userService.getUsers();
  selectedUser: User | null = null;

  constructor(
    private userService: UserService,
    private notificationService: NotificationService
  ) {}

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    this.users$ = this.userService.getUsers();
  }

  onSaveUser(user: Omit<User, "id"> | User) {
    const operation =
      "id" in user
        ? this.userService.updateUser(user.id, user)
        : this.userService.createUser(user);

    operation.subscribe({
      next: () => {
        this.loadUsers();
        this.onResetForm();
        const message =
          "id" in user
            ? "User updated successfully"
            : "User created successfully";
        this.notificationService.showSuccess(message);
      },
      error: (error) => {
        this.notificationService.showError(
          `Failed to save user: ${error.message}`
        );
      },
    });
  }

  onEditUser(user: User) {
    this.selectedUser = { ...user };
  }

  onDeleteUser(userId: number) {
    if (confirm("Are you sure you want to delete this user?")) {
      this.userService.deleteUser(userId).subscribe({
        next: () => {
          this.loadUsers();
          this.notificationService.showSuccess("User deleted successfully");
        },
        error: (error) => {
          this.notificationService.showError(
            `Failed to delete user: ${error.message}`
          );
        },
      });
    }
  }

  onResetForm() {
    this.selectedUser = null;
  }
}
```

## 🔧 **SRP in Angular Architecture**

### **Service Layer Separation**

```typescript
// ✅ Data Access Layer
@Injectable({
  providedIn: "root",
})
export class UserApiService {
  // Only responsible for HTTP operations
}

// ✅ Business Logic Layer
@Injectable({
  providedIn: "root",
})
export class UserBusinessService {
  // Only responsible for business rules and logic
}

// ✅ State Management Layer
@Injectable({
  providedIn: "root",
})
export class UserStateService {
  // Only responsible for state management
}

// ✅ Validation Layer
@Injectable({
  providedIn: "root",
})
export class UserValidationService {
  // Only responsible for validation rules
}
```

### **Component Responsibility Separation**

```typescript
// ✅ Presentation Component (Dumb/Pure Component)
@Component({
  selector: "app-user-card",
  template: `
    <div class="user-card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <button (click)="onEdit()">Edit</button>
      <button (click)="onDelete()">Delete</button>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserCardComponent {
  @Input() user!: User;
  @Output() edit = new EventEmitter<User>();
  @Output() delete = new EventEmitter<number>();

  onEdit() {
    this.edit.emit(this.user);
  }

  onDelete() {
    this.delete.emit(this.user.id);
  }
}

// ✅ Container Component (Smart Component)
@Component({
  selector: "app-user-container",
  template: `
    <app-user-card
      *ngFor="let user of users$ | async"
      [user]="user"
      (edit)="editUser($event)"
      (delete)="deleteUser($event)"
    >
    </app-user-card>
  `,
})
export class UserContainerComponent {
  users$ = this.userService.getUsers();

  constructor(private userService: UserService) {}

  editUser(user: User) {
    // Handle edit logic
  }

  deleteUser(userId: number) {
    // Handle delete logic
  }
}
```

## 🚨 **Common SRP Violations in Angular**

### **❌ Fat Components**

- Components doing data fetching, business logic, validation, and UI management
- Components directly making HTTP calls
- Components handling multiple unrelated concerns

### **❌ God Services**

- Services handling multiple unrelated responsibilities
- Services mixing data access with business logic
- Services handling both API calls and local storage

### **❌ Mixed Concerns**

- Validation logic inside components
- Business logic in templates
- UI state mixed with business state

## 📊 **Angular Version Comparison**

| Aspect                     | Angular 15             | Angular 17-19                       |
| -------------------------- | ---------------------- | ----------------------------------- |
| **Component Architecture** | Class-based components | Enhanced with standalone components |
| **Service Injection**      | Constructor injection  | Constructor + inject() function     |
| **State Management**       | RxJS + Services        | Signals + RxJS integration          |
| **Code Organization**      | NgModules required     | Flexible with standalone            |
| **Tree Shaking**           | Module-level           | Component-level optimization        |
| **Bundle Size**            | Larger due to modules  | Smaller with better separation      |

### **Modern Angular 19 SRP Example**

```typescript
// ✅ Angular 19 with Signals and Standalone Components
@Component({
  selector: "app-user-profile",
  template: `
    <div class="profile">
      <h2>{{ user().name }}</h2>
      <p>{{ user().email }}</p>
      <p>Status: {{ isOnline() ? "Online" : "Offline" }}</p>
    </div>
  `,
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserProfileComponent {
  // Single responsibility: Display user profile information
  private userService = inject(UserService);
  private statusService = inject(UserStatusService);

  user = this.userService.currentUser;
  isOnline = this.statusService.isOnline;
}
```

## 🎯 **Key Benefits of SRP in Angular**

### **🔧 Maintainability**

- Easier to understand and modify
- Clear separation of concerns
- Reduced cognitive load

### **🧪 Testability**

- Isolated functionality
- Easier to mock dependencies
- More focused unit tests

### **🚀 Reusability**

- Single-purpose components and services
- Better composition opportunities
- Easier to extract into libraries

### **👥 Team Collaboration**

- Clear ownership boundaries
- Parallel development possible
- Reduced merge conflicts

## 🎯 **Key Takeaways**

1. **One class, one responsibility** - Each class should have a single reason to change
2. **Separate concerns** - Data access, business logic, validation, and UI should be separate
3. **Use services for single purposes** - API calls, validation, state management should be in different services
4. **Keep components focused** - Components should either be smart (containers) or dumb (presentation)
5. **Think before you code** - Ask "What is this class responsible for?" before adding new methods
6. **Refactor regularly** - When a class grows too big, split it up
7. **Test-driven approach** - If it's hard to test, it probably violates SRP

SRP is the foundation of clean, maintainable Angular applications! 🚀
