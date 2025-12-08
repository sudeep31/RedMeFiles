# 🎛️ State Management in Angular: Complete Guide

## 🎯 **Question Overview**

_"How do you manage state in Angular?"_

## 🔍 **Understanding State Management**

State management in Angular refers to how you **store**, **access**, and **modify** application data that needs to be shared across components, persisted across navigation, or synchronized with external sources.

Think of state as your application's **memory** - it remembers what the user has done, what data has been loaded, and what the current state of the UI should be! 🧠

## 🔄 **Core State Concepts: Mutation vs Query**

Before diving into implementation approaches, let's understand the two fundamental operations in state management:

### **🛠️ State Mutation (Write Operations)**

State mutation refers to **changing** or **updating** the application state. These are the "write" operations that modify your data.

#### **1. 📝 What is State Mutation?**

State mutation is any operation that changes the current state of your application. Think of it like editing a document - you're modifying the existing content to reflect new information.

**Key Characteristics:**

- **Modifies existing data** in your application
- **Triggers reactive updates** across your app
- Should be **immutable** (create new state rather than modifying existing)
- Often **asynchronous** (API calls, user interactions)

```typescript
// Examples of State Mutations
class UserStateService {
  private userSubject = new BehaviorSubject<User | null>(null);

  // 🔹 Mutation: Adding new user data
  loginUser(user: User): void {
    this.userSubject.next(user); // ✅ State changed!
  }

  // 🔹 Mutation: Updating existing user
  updateUser(updates: Partial<User>): void {
    const current = this.userSubject.value;
    if (current) {
      const updated = { ...current, ...updates }; // ✅ Immutable update
      this.userSubject.next(updated); // ✅ State changed!
    }
  }

  // 🔹 Mutation: Removing user data
  logout(): void {
    this.userSubject.next(null); // ✅ State changed!
  }
}
```

#### **2. 🎯 Types of State Mutations**

**2.1 🔵 Create Operations (C in CRUD)**

```typescript
// Adding new items to state
addProduct(product: Product): void {
  const currentProducts = this.productsSubject.value;
  const updatedProducts = [...currentProducts, product]; // ✅ Immutable add
  this.productsSubject.next(updatedProducts);
}

addMultipleUsers(users: User[]): void {
  this.usersSubject.update(current => [...current, ...users]);
}
```

**2.2 🟡 Update Operations (U in CRUD)**

```typescript
// Modifying existing items
updateProduct(id: string, updates: Partial<Product>): void {
  const currentProducts = this.productsSubject.value;
  const updatedProducts = currentProducts.map(product =>
    product.id === id ? { ...product, ...updates } : product
  );
  this.productsSubject.next(updatedProducts);
}

// Partial updates
updateUserProfile(profileUpdates: Partial<UserProfile>): void {
  this.userSubject.update(user =>
    user ? { ...user, profile: { ...user.profile, ...profileUpdates } } : user
  );
}
```

**2.3 🔴 Delete Operations (D in CRUD)**

```typescript
// Removing items from state
removeProduct(productId: string): void {
  const currentProducts = this.productsSubject.value;
  const filteredProducts = currentProducts.filter(p => p.id !== productId);
  this.productsSubject.next(filteredProducts);
}

clearAllData(): void {
  this.dataSubject.next([]); // ✅ Clear all state
}
```

**2.4 🟢 Batch Operations**

```typescript
// Multiple mutations in one operation
batchUpdateUsers(userUpdates: Array<{id: string, updates: Partial<User>}>): void {
  const currentUsers = this.usersSubject.value;
  const updatedUsers = currentUsers.map(user => {
    const update = userUpdates.find(u => u.id === user.id);
    return update ? { ...user, ...update.updates } : user;
  });
  this.usersSubject.next(updatedUsers);
}
```

#### **3. ⚡ Async State Mutations**

**3.1 🌐 API-based Mutations**

```typescript
// Async mutations with loading states
async createUser(userData: CreateUserRequest): Promise<void> {
  // 🔹 Set loading state
  this.loadingSubject.next(true);
  this.errorSubject.next(null);

  try {
    // 🔹 API call
    const newUser = await this.http.post<User>('/api/users', userData).toPromise();

    // 🔹 Update state with new data
    const currentUsers = this.usersSubject.value;
    this.usersSubject.next([...currentUsers, newUser]);

    // 🔹 Show success state
    this.successSubject.next('User created successfully!');
  } catch (error: any) {
    // 🔹 Handle error state
    this.errorSubject.next(`Failed to create user: ${error.message}`);
  } finally {
    // 🔹 Clear loading state
    this.loadingSubject.next(false);
  }
}
```

**3.2 🔄 Optimistic Updates**

```typescript
// Update UI immediately, rollback if API fails
async updateUserOptimistic(userId: string, updates: Partial<User>): Promise<void> {
  const currentUsers = this.usersSubject.value;
  const userIndex = currentUsers.findIndex(u => u.id === userId);

  if (userIndex === -1) return;

  const originalUser = currentUsers[userIndex];
  const optimisticUser = { ...originalUser, ...updates };

  // 🔹 Immediate UI update (optimistic)
  const optimisticUsers = [...currentUsers];
  optimisticUsers[userIndex] = optimisticUser;
  this.usersSubject.next(optimisticUsers);

  try {
    // 🔹 Confirm with server
    const serverUser = await this.http.put<User>(`/api/users/${userId}`, updates).toPromise();

    // 🔹 Update with server response
    const confirmedUsers = [...currentUsers];
    confirmedUsers[userIndex] = serverUser;
    this.usersSubject.next(confirmedUsers);
  } catch (error) {
    // 🔹 Rollback on failure
    this.usersSubject.next(currentUsers);
    this.errorSubject.next('Update failed - changes reverted');
  }
}
```

### **🔍 State Query (Read Operations)**

State query refers to **reading** and **selecting** data from your application state. These are the "read" operations that retrieve information without modifying it.

#### **1. 📖 What is State Query?**

State query is any operation that retrieves data from your application state without modifying it. Think of it like reading a book - you're accessing the information without changing the content.

**Key Characteristics:**

- **Reads data** without modification
- Can **filter, transform, or combine** state
- Should be **pure functions** (same input = same output)
- Often **reactive** (automatically update when state changes)

```typescript
// Examples of State Queries
class UserStateService {
  private userSubject = new BehaviorSubject<User | null>(null);
  private usersSubject = new BehaviorSubject<User[]>([]);

  // 🔍 Query: Get current user
  getCurrentUser(): User | null {
    return this.userSubject.value; // ✅ Read without changing
  }

  // 🔍 Query: Check authentication status
  isAuthenticated(): boolean {
    return !!this.userSubject.value; // ✅ Derived state
  }

  // 🔍 Query: Get reactive user stream
  getUser$(): Observable<User | null> {
    return this.userSubject.asObservable(); // ✅ Reactive query
  }
}
```

#### **2. 🎯 Types of State Queries**

**2.1 🔵 Direct Queries**

```typescript
// Simple data retrieval
getCurrentUser(): User | null {
  return this.userSubject.value;
}

getAllProducts(): Product[] {
  return this.productsSubject.value;
}

getLoadingState(): boolean {
  return this.loadingSubject.value;
}
```

**2.2 🟡 Filtered Queries**

```typescript
// Queries with filtering logic
getActiveUsers$(): Observable<User[]> {
  return this.users$.pipe(
    map(users => users.filter(user => user.isActive))
  );
}

getProductsByCategory$(category: string): Observable<Product[]> {
  return this.products$.pipe(
    map(products => products.filter(p => p.category === category))
  );
}

getUsersByRole(role: UserRole): User[] {
  return this.usersSubject.value.filter(user => user.role === role);
}
```

**2.3 🔴 Computed/Derived Queries**

```typescript
// Queries that derive new data from existing state
getUserStats$(): Observable<UserStats> {
  return this.users$.pipe(
    map(users => ({
      total: users.length,
      active: users.filter(u => u.isActive).length,
      premium: users.filter(u => u.isPremium).length,
      averageAge: users.reduce((sum, u) => sum + u.age, 0) / users.length
    }))
  );
}

getShoppingCartTotal$(): Observable<number> {
  return this.cartItems$.pipe(
    map(items => items.reduce((total, item) => total + (item.price * item.quantity), 0))
  );
}
```

**2.4 🟢 Combined Queries**

```typescript
// Queries that combine multiple state sources
getUserWithPosts$(userId: string): Observable<UserWithPosts | null> {
  return combineLatest([
    this.getUser$(userId),
    this.getUserPosts$(userId)
  ]).pipe(
    map(([user, posts]) =>
      user ? { ...user, posts } : null
    )
  );
}

getDashboardData$(): Observable<DashboardData> {
  return combineLatest([
    this.userStats$,
    this.recentOrders$,
    this.systemHealth$
  ]).pipe(
    map(([userStats, orders, health]) => ({
      userStats,
      recentOrders: orders,
      systemHealth: health,
      lastUpdated: new Date()
    }))
  );
}
```

#### **3. ⚡ Advanced Query Patterns**

**3.1 🎯 Selector Pattern**

```typescript
// Reusable selectors for complex queries
class UserSelectors {
  static selectUser = (state: AppState) => state.user;
  static selectUsers = (state: AppState) => state.users;

  static selectCurrentUser = createSelector(
    UserSelectors.selectUser,
    (user) => user.currentUser
  );

  static selectUserById = (id: string) =>
    createSelector(UserSelectors.selectUsers, (users) =>
      users.find((u) => u.id === id)
    );

  static selectActiveUsers = createSelector(
    UserSelectors.selectUsers,
    (users) => users.filter((u) => u.isActive)
  );
}
```

**3.2 🔍 Memoized Queries**

```typescript
// Cache expensive computations
class OptimizedStateService {
  private memoizedQueries = new Map<string, any>();

  getExpensiveComputation$(key: string): Observable<ComplexResult> {
    if (this.memoizedQueries.has(key)) {
      return of(this.memoizedQueries.get(key));
    }

    return this.rawData$.pipe(
      map((data) => this.performExpensiveCalculation(data)),
      tap((result) => this.memoizedQueries.set(key, result)),
      shareReplay(1) // Cache the result
    );
  }

  // Clear cache when data changes
  clearMemoizedQueries(): void {
    this.memoizedQueries.clear();
  }
}
```

#### **4. 🚀 Query Performance Optimization**

**4.1 📊 Efficient Filtering**

```typescript
// Optimize queries for large datasets
class PerformantQueriesService {
  // Use indices for fast lookups
  private userIndexById = new Map<string, User>();
  private usersByRole = new Map<UserRole, User[]>();

  buildIndices(users: User[]): void {
    this.userIndexById.clear();
    this.usersByRole.clear();

    users.forEach((user) => {
      this.userIndexById.set(user.id, user);

      if (!this.usersByRole.has(user.role)) {
        this.usersByRole.set(user.role, []);
      }
      this.usersByRole.get(user.role)!.push(user);
    });
  }

  // O(1) lookup instead of O(n) filtering
  getUserById(id: string): User | undefined {
    return this.userIndexById.get(id);
  }

  getUsersByRole(role: UserRole): User[] {
    return this.usersByRole.get(role) || [];
  }
}
```

**4.2 🎭 Reactive Query Optimization**

```typescript
// Minimize unnecessary emissions
getOptimizedUsers$(): Observable<User[]> {
  return this.users$.pipe(
    distinctUntilChanged((prev, curr) =>
      prev.length === curr.length &&
      prev.every((user, index) => user.id === curr[index]?.id)
    ),
    debounceTime(100), // Avoid rapid-fire updates
    shareReplay(1) // Share computation across subscribers
  );
}
```

## 📊 **Types of State in Angular**

### **1. 🏠 Local Component State**

Data that belongs to a single component and doesn't need to be shared.

### **2. 🔗 Shared State**

Data that multiple components need to access and modify.

### **3. 🌐 Global Application State**

Data that affects the entire application (user authentication, app configuration).

### **4. 💾 Persistent State**

Data that needs to survive browser refreshes and sessions.

### **5. 🔄 Server State**

Data synchronized with backend APIs.

## 🛠️ **State Management Approaches**

Now that we understand state mutation and queries, let's explore different implementation approaches:

### **1. 🎯 Angular Services (Reactive Approach)**

The most common and Angular-native way to manage state using RxJS observables.

#### **1.1 📝 Service-Based State Structure**

Angular services provide a clean, injectable way to manage state that can be shared across components:

#### **1.2 🚀 Complete Implementation Example**

```typescript
// User State Service
@Injectable({
  providedIn: "root",
})
export class UserStateService {
  private userSubject = new BehaviorSubject<User | null>(null);
  private loadingSubject = new BehaviorSubject<boolean>(false);
  private errorSubject = new BehaviorSubject<string | null>(null);

  // Public observables
  user$ = this.userSubject.asObservable();
  loading$ = this.loadingSubject.asObservable();
  error$ = this.errorSubject.asObservable();

  // Computed observables
  isAuthenticated$ = this.user$.pipe(map((user) => !!user));

  userProfile$ = this.user$.pipe(
    filter((user) => !!user),
    map((user) => ({
      name: user!.name,
      email: user!.email,
      avatar: user!.avatar,
    }))
  );

  constructor(private http: HttpClient) {
    // Initialize from localStorage
    this.initializeFromStorage();
  }

  // State mutations
  async loginUser(credentials: LoginCredentials): Promise<void> {
    this.loadingSubject.next(true);
    this.errorSubject.next(null);

    try {
      const user = await this.http
        .post<User>("/api/login", credentials)
        .toPromise();
      this.userSubject.next(user);
      this.saveToStorage(user);
    } catch (error) {
      this.errorSubject.next("Login failed: " + error.message);
    } finally {
      this.loadingSubject.next(false);
    }
  }

  updateUser(updates: Partial<User>): void {
    const currentUser = this.userSubject.value;
    if (currentUser) {
      const updatedUser = { ...currentUser, ...updates };
      this.userSubject.next(updatedUser);
      this.saveToStorage(updatedUser);
    }
  }

  logout(): void {
    this.userSubject.next(null);
    this.clearStorage();
  }

  // State queries
  getCurrentUser(): User | null {
    return this.userSubject.value;
  }

  isUserLoggedIn(): boolean {
    return !!this.userSubject.value;
  }

  private initializeFromStorage(): void {
    const savedUser = localStorage.getItem("user");
    if (savedUser) {
      try {
        const user = JSON.parse(savedUser);
        this.userSubject.next(user);
      } catch (error) {
        console.warn("Failed to parse saved user data");
        this.clearStorage();
      }
    }
  }

  private saveToStorage(user: User): void {
    localStorage.setItem("user", JSON.stringify(user));
  }

  private clearStorage(): void {
    localStorage.removeItem("user");
  }
}
```

#### **1.3 🧩 Component Integration Pattern**

Here's how components consume and interact with the reactive state service:

```typescript
// Using the service in components
@Component({
  selector: "app-user-profile",
  template: `
    <div class="profile" *ngIf="userProfile$ | async as profile">
      <img [src]="profile.avatar" [alt]="profile.name" />
      <h2>{{ profile.name }}</h2>
      <p>{{ profile.email }}</p>
      <button (click)="editProfile()">Edit Profile</button>
      <button (click)="logout()">Logout</button>
    </div>

    <div class="loading" *ngIf="loading$ | async">Updating profile...</div>

    <div class="error" *ngIf="error$ | async as error">
      {{ error }}
    </div>
  `,
  standalone: true,
  imports: [CommonModule],
})
export class UserProfileComponent {
  userProfile$ = this.userService.userProfile$;
  loading$ = this.userService.loading$;
  error$ = this.userService.error$;

  constructor(private userService: UserStateService) {}

  editProfile() {
    // Navigate to edit profile
  }

  logout() {
    this.userService.logout();
  }
}
```

### **2. 🚀 Angular Signals (Angular 16+)**

Modern reactive state management with signals offering better performance and simpler syntax.

#### **2.1 🎯 Signal-Based Architecture**

Signals provide a more efficient and intuitive approach to reactive state management:

#### **2.2 📝 Complete Signal Service Implementation**

```typescript
// Signal-based State Service
@Injectable({
  providedIn: "root",
})
export class UserSignalService {
  // Writable signals
  private _user = signal<User | null>(null);
  private _loading = signal(false);
  private _error = signal<string | null>(null);

  // Read-only signals
  readonly user = this._user.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly error = this._error.asReadonly();

  // Computed signals
  readonly isAuthenticated = computed(() => !!this._user());
  readonly userProfile = computed(() => {
    const user = this._user();
    return user
      ? {
          name: user.name,
          email: user.email,
          avatar: user.avatar,
        }
      : null;
  });

  readonly userInitials = computed(() => {
    const user = this._user();
    if (!user) return "";
    return user.name
      .split(" ")
      .map((part) => part[0])
      .join("")
      .toUpperCase();
  });

  constructor(private http: HttpClient) {
    this.initializeFromStorage();

    // Effect for persistence
    effect(() => {
      const user = this._user();
      if (user) {
        localStorage.setItem("user", JSON.stringify(user));
      } else {
        localStorage.removeItem("user");
      }
    });
  }

  async loginUser(credentials: LoginCredentials): Promise<void> {
    this._loading.set(true);
    this._error.set(null);

    try {
      const user = await firstValueFrom(
        this.http.post<User>("/api/login", credentials)
      );
      this._user.set(user);
    } catch (error: any) {
      this._error.set("Login failed: " + error.message);
      throw error;
    } finally {
      this._loading.set(false);
    }
  }

  updateUser(updates: Partial<User>): void {
    this._user.update((user) => (user ? { ...user, ...updates } : null));
  }

  logout(): void {
    this._user.set(null);
    this._error.set(null);
  }

  private initializeFromStorage(): void {
    const savedUser = localStorage.getItem("user");
    if (savedUser) {
      try {
        const user = JSON.parse(savedUser);
        this._user.set(user);
      } catch (error) {
        console.warn("Failed to parse saved user data");
        localStorage.removeItem("user");
      }
    }
  }
}
```

#### **2.3 🤖 Signal-Based Component Integration**

Components using signals benefit from automatic change detection optimization:

```typescript
// Using signals in components
@Component({
  selector: "app-user-dashboard",
  template: `
    @if (userProfile(); as profile) {
    <div class="profile">
      <div class="avatar">{{ userInitials() }}</div>
      <h2>Welcome, {{ profile.name }}!</h2>
      <p>{{ profile.email }}</p>
      <button (click)="editProfile()">Edit Profile</button>
      <button (click)="logout()">Logout</button>
    </div>
    } @else {
    <app-login (loginSuccess)="onLoginSuccess()"></app-login>
    } @if (loading()) {
    <div class="loading">Please wait...</div>
    } @if (error(); as errorMessage) {
    <div class="error">{{ errorMessage }}</div>
    }
  `,
  standalone: true,
  imports: [CommonModule, LoginComponent],
})
export class UserDashboardComponent {
  userProfile = this.userService.userProfile;
  userInitials = this.userService.userInitials;
  loading = this.userService.loading;
  error = this.userService.error;

  constructor(private userService: UserSignalService) {}

  editProfile() {
    // Navigate to edit profile
  }

  logout() {
    this.userService.logout();
  }

  onLoginSuccess() {
    console.log("Login successful!");
  }
}
```

### **3. 🗃️ NgRx (Redux Pattern)**

For complex applications with intricate state management needs using the Redux pattern.

#### **3.1 🏗️ NgRx Architecture Overview**

NgRx follows the Redux pattern with unidirectional data flow: Actions → Reducers → Store → Selectors → Components

#### **3.2 🎯 Actions & State Definition**

Define the actions and state structure for your application:

````typescript
// Actions
export const UserActions = createActionGroup({
  source: "User",
  events: {
    "Load User": emptyProps(),
    "Load User Success": props<{ user: User }>(),
    "Load User Failure": props<{ error: string }>(),
    "Update User": props<{ updates: Partial<User> }>(),
    "Update User Success": props<{ user: User }>(),
    Logout: emptyProps(),
  },
});

// State interface
export interface UserState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  user: null,
  loading: false,
  error: null,
};

// Reducer
export const userReducer = createReducer(
  initialState,
  on(UserActions.loadUser, (state) => ({
    ...state,
    loading: true,
    error: null,
  })),
  on(UserActions.loadUserSuccess, (state, { user }) => ({
    ...state,
    user,
    loading: false,
    error: null,
  })),
  on(UserActions.loadUserFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error,
  })),
  on(UserActions.updateUser, (state) => ({
    ...state,
    loading: true,
  })),
  on(UserActions.updateUserSuccess, (state, { user }) => ({
    ...state,
    user,
    loading: false,
  })),
  on(UserActions.logout, (state) => ({
    ...state,
    user: null,
    error: null,
  }))
);

// Selectors
export const selectUserState = createFeatureSelector<UserState>("user");

export const selectUser = createSelector(
  selectUserState,
  (state) => state.user
);

export const selectIsAuthenticated = createSelector(
  selectUser,
  (user) => !!user
);

export const selectUserProfile = createSelector(selectUser, (user) =>
  user
    ? {
        name: user.name,
        email: user.email,
        avatar: user.avatar,
      }
    : null
);

export const selectUserLoading = createSelector(
  selectUserState,
  (state) => state.loading
);

export const selectUserError = createSelector(
  selectUserState,
  (state) => state.error
);

#### **3.3 ⚡ Effects for Side Effects Management**

Effects handle asynchronous operations and side effects:

```typescript
// Effects
@Injectable()
export class UserEffects {
  constructor(
    private actions$: Actions,
    private userService: UserApiService,
    private store: Store
  ) {}

  loadUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.loadUser),
      switchMap(() =>
        this.userService.getCurrentUser().pipe(
          map((user) => UserActions.loadUserSuccess({ user })),
          catchError((error) =>
            of(UserActions.loadUserFailure({ error: error.message }))
          )
        )
      )
    )
  );

  updateUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.updateUser),
      withLatestFrom(this.store.select(selectUser)),
      switchMap(([action, currentUser]) => {
        if (!currentUser) {
          return of(
            UserActions.loadUserFailure({ error: "No user to update" })
          );
        }

        const updatedUser = { ...currentUser, ...action.updates };
        return this.userService.updateUser(updatedUser).pipe(
          map((user) => UserActions.updateUserSuccess({ user })),
          catchError((error) =>
            of(UserActions.loadUserFailure({ error: error.message }))
          )
        );
      })
    )
  );
}

#### **3.4 🧩 Component Integration with NgRx**

Components interact with NgRx through dispatching actions and selecting state:

```typescript
// Component using NgRx
@Component({
  selector: "app-ngrx-user",
  template: `
    <div *ngIf="user$ | async as user" class="user-info">
      <h2>{{ user.name }}</h2>
      <p>{{ user.email }}</p>
      <button (click)="updateProfile()">Update Profile</button>
      <button (click)="logout()">Logout</button>
    </div>

    <div *ngIf="loading$ | async" class="loading">Loading user data...</div>

    <div *ngIf="error$ | async as error" class="error">
      {{ error }}
    </div>
  `,
  standalone: true,
  imports: [CommonModule],
})
export class NgRxUserComponent implements OnInit {
  user$ = this.store.select(selectUser);
  loading$ = this.store.select(selectUserLoading);
  error$ = this.store.select(selectUserError);
  isAuthenticated$ = this.store.select(selectIsAuthenticated);

  constructor(private store: Store) {}

  ngOnInit() {
    this.store.dispatch(UserActions.loadUser());
  }

  updateProfile() {
    const updates = { name: "Updated Name" };
    this.store.dispatch(UserActions.updateUser({ updates }));
  }

  logout() {
    this.store.dispatch(UserActions.logout());
  }
}
````

### **4. 🏪 Akita State Management**

Alternative to NgRx with less boilerplate and more intuitive API.

#### **4.1 🏗️ Akita Architecture Pattern**

Akita uses Entity Stores for managing collections and Query services for reading state:

#### **4.2 🗺️ Entity Store Implementation**

````typescript
// Entity State with Akita
export interface UserState extends EntityState<User> {
  currentUserId: ID | null;
  loading: boolean;
  error: string | null;
}

@Injectable({ providedIn: "root" })
@StoreConfig({ name: "user" })
export class UserStore extends EntityStore<UserState> {
  constructor() {
    super({
      currentUserId: null,
      loading: false,
      error: null,
    });
  }
}

@Injectable({ providedIn: "root" })
export class UserQuery extends QueryEntity<UserState> {
  currentUser$ = this.selectEntity(this.getValue().currentUserId);
  loading$ = this.select((state) => state.loading);
  error$ = this.select((state) => state.error);

  isAuthenticated$ = this.currentUser$.pipe(map((user) => !!user));

  constructor(protected store: UserStore) {
    super(store);
  }
}

#### **4.3 🛠️ Service Layer with Akita**

Service layer manages state mutations and API interactions:

```typescript
@Injectable({ providedIn: "root" })
export class UserService {
  constructor(private userStore: UserStore, private http: HttpClient) {}

  async login(credentials: LoginCredentials): Promise<void> {
    this.userStore.setLoading(true);
    this.userStore.setError(null);

    try {
      const user = await firstValueFrom(
        this.http.post<User>("/api/login", credentials)
      );

      this.userStore.add(user);
      this.userStore.update({ currentUserId: user.id });
    } catch (error: any) {
      this.userStore.setError(error.message);
    } finally {
      this.userStore.setLoading(false);
    }
  }

  updateUser(id: ID, updates: Partial<User>): void {
    this.userStore.update(id, updates);
  }

  logout(): void {
    this.userStore.remove();
    this.userStore.update({ currentUserId: null });
  }
}
````

## 🎯 **Advanced State Management Patterns**

Now let's explore sophisticated patterns for complex state management scenarios:

### **1. 🔄 State Normalization**

Organizing complex relational data for optimal performance and maintainability.

#### **1.1 📊 Normalized Data Structure**

```typescript
// Normalized state structure
interface NormalizedState {
  users: {
    byId: { [id: string]: User };
    allIds: string[];
  };
  posts: {
    byId: { [id: string]: Post };
    allIds: string[];
  };
  ui: {
    selectedUserId: string | null;
    selectedPostId: string | null;
  };
}

@Injectable({
  providedIn: "root",
})
export class NormalizedStateService {
  private stateSubject = new BehaviorSubject<NormalizedState>({
    users: { byId: {}, allIds: [] },
    posts: { byId: {}, allIds: [] },
    ui: { selectedUserId: null, selectedPostId: null },
  });

  state$ = this.stateSubject.asObservable();

  // Selectors
  users$ = this.state$.pipe(
    map((state) => state.users.allIds.map((id) => state.users.byId[id]))
  );

  selectedUser$ = this.state$.pipe(
    map((state) =>
      state.ui.selectedUserId ? state.users.byId[state.ui.selectedUserId] : null
    )
  );

  userPosts$ = this.state$.pipe(
    map((state) => {
      const userId = state.ui.selectedUserId;
      if (!userId) return [];

      return state.posts.allIds
        .map((id) => state.posts.byId[id])
        .filter((post) => post.userId === userId);
    })
  );

  // Actions
  addUsers(users: User[]): void {
    this.updateState((state) => {
      const byId = { ...state.users.byId };
      const allIds = [...state.users.allIds];

      users.forEach((user) => {
        if (!byId[user.id]) {
          byId[user.id] = user;
          allIds.push(user.id);
        }
      });

      return {
        ...state,
        users: { byId, allIds },
      };
    });
  }

  selectUser(userId: string): void {
    this.updateState((state) => ({
      ...state,
      ui: { ...state.ui, selectedUserId: userId },
    }));
  }

  private updateState(
    updater: (state: NormalizedState) => NormalizedState
  ): void {
    const currentState = this.stateSubject.value;
    const newState = updater(currentState);
    this.stateSubject.next(newState);
  }
}
```

### **2. 🏪 State Persistence**

Managing state that survives browser sessions and application restarts.

#### **2.1 💾 Persistent Storage Strategy**

```typescript
@Injectable({
  providedIn: "root",
})
export class PersistentStateService {
  private readonly STORAGE_KEYS = {
    USER: "app_user_state",
    SETTINGS: "app_settings_state",
    CACHE: "app_cache_state",
  };

  // State subjects
  private userSubject = new BehaviorSubject<User | null>(null);
  private settingsSubject = new BehaviorSubject<AppSettings>(defaultSettings);

  // Public observables
  user$ = this.userSubject.asObservable();
  settings$ = this.settingsSubject.asObservable();

  constructor() {
    this.loadPersistedState();
    this.setupStatePersistence();
  }

  // Load state from storage
  private loadPersistedState(): void {
    // Load user state
    const savedUser = this.getFromStorage(this.STORAGE_KEYS.USER);
    if (savedUser) {
      this.userSubject.next(savedUser);
    }

    // Load settings
    const savedSettings = this.getFromStorage(this.STORAGE_KEYS.SETTINGS);
    if (savedSettings) {
      this.settingsSubject.next({ ...defaultSettings, ...savedSettings });
    }
  }

  // Persist state changes automatically
  private setupStatePersistence(): void {
    // Persist user state
    this.user$
      .pipe(
        debounceTime(300), // Avoid too frequent writes
        distinctUntilChanged()
      )
      .subscribe((user) => {
        if (user) {
          this.saveToStorage(this.STORAGE_KEYS.USER, user);
        } else {
          this.removeFromStorage(this.STORAGE_KEYS.USER);
        }
      });

    // Persist settings
    this.settings$
      .pipe(debounceTime(500), distinctUntilChanged())
      .subscribe((settings) => {
        this.saveToStorage(this.STORAGE_KEYS.SETTINGS, settings);
      });
  }

  // Storage utilities
  private getFromStorage<T>(key: string): T | null {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : null;
    } catch (error) {
      console.warn(`Failed to parse ${key} from storage:`, error);
      return null;
    }
  }

  private saveToStorage(key: string, value: any): void {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(`Failed to save ${key} to storage:`, error);
    }
  }

  private removeFromStorage(key: string): void {
    localStorage.removeItem(key);
  }

  // Public methods
  updateSettings(updates: Partial<AppSettings>): void {
    this.settingsSubject.next({
      ...this.settingsSubject.value,
      ...updates,
    });
  }

  clearAllPersistedState(): void {
    Object.values(this.STORAGE_KEYS).forEach((key) => {
      this.removeFromStorage(key);
    });
  }
}
```

## 📊 **State Management Comparison**

| Approach                    | Complexity | Learning Curve | Performance | Use Case             |
| --------------------------- | ---------- | -------------- | ----------- | -------------------- |
| **Angular Services + RxJS** | Low        | Easy           | Good        | Small to medium apps |
| **Angular Signals**         | Low        | Easy           | Excellent   | Modern Angular apps  |
| **NgRx**                    | High       | Steep          | Excellent   | Large, complex apps  |
| **Akita**                   | Medium     | Moderate       | Good        | Medium to large apps |
| **Elf**                     | Medium     | Moderate       | Good        | Flexible state needs |

## 🚨 **Common State Management Pitfalls**

### **❌ Common Mistakes**

1. **State Mutation**

```typescript
// ❌ Bad - Mutating state directly
updateUser(updates: Partial<User>) {
  this.currentUser.name = updates.name; // Direct mutation
  this.userSubject.next(this.currentUser);
}
```

2. **Memory Leaks**

```typescript
// ❌ Bad - Not unsubscribing
ngOnInit() {
  this.userService.user$.subscribe(user => {
    this.currentUser = user;
  }); // No unsubscription
}
```

### **✅ Best Practices**

1. **Immutable Updates**

```typescript
// ✅ Good - Immutable updates
updateUser(updates: Partial<User>) {
  const currentUser = this.userSubject.value;
  if (currentUser) {
    const updatedUser = { ...currentUser, ...updates };
    this.userSubject.next(updatedUser);
  }
}
```

2. **Proper Subscription Management**

```typescript
// ✅ Good - Using takeUntilDestroyed
@Component({})
export class MyComponent {
  private destroy$ = new Subject<void>();

  ngOnInit() {
    this.userService.user$.pipe(takeUntil(this.destroy$)).subscribe((user) => {
      this.currentUser = user;
    });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// ✅ Even better - Angular 16+ with takeUntilDestroyed
@Component({})
export class ModernComponent {
  constructor() {
    this.userService.user$.pipe(takeUntilDestroyed()).subscribe((user) => {
      this.currentUser = user;
    });
  }
}
```

## 🎯 **Key Takeaways**

### **Choose the Right Approach:**

1. **🏠 Simple Component State** → Local component properties
2. **🔗 Shared Component State** → Angular Services + RxJS
3. **🚀 Modern Reactive State** → Angular Signals
4. **🏢 Complex Enterprise Apps** → NgRx or Akita
5. **💾 Persistent State** → Services + LocalStorage/SessionStorage

### **Best Practices:**

1. **Keep state immutable**
2. **Use observables for reactive updates**
3. **Manage subscriptions properly**
4. **Normalize complex state**
5. **Persist important state**
6. **Test state changes thoroughly**

### **Performance Tips:**

1. Use **OnPush change detection** with observables
2. **Debounce frequent updates**
3. **Memoize expensive computations**
4. **Use trackBy functions** in \*ngFor
5. **Lazy load state** when possible

State management is crucial for building scalable Angular applications. Choose the right approach based on your app's complexity and requirements! 🚀
