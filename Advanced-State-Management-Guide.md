# Advanced State Management & Modern Frontend Architecture Guide 🚀

This comprehensive guide covers everything you need to know about building scalable, secure, and maintainable frontend applications with focus on state management, API integration, security, and modern UI frameworks.

## Table of Contents

1. [State Management Patterns](#state-management-patterns)
2. [API Integration & Response Filtering](#api-integration--response-filtering)
3. [Custom Hooks & Advanced Patterns](#custom-hooks--advanced-patterns)
4. [Security Implementation](#security-implementation)
5. [Guards & Route Protection](#guards--route-protection)
6. [Advanced Routing Strategies](#advanced-routing-strategies)
7. [UI Framework Integration: Material + Tailwind](#ui-framework-integration-material--tailwind)
8. [Performance Optimization](#performance-optimization)
9. [Testing Strategies](#testing-strategies)
10. [Real-World Implementation Examples](#real-world-implementation-examples)

---

## State Management Patterns

### 🎯 Choosing the Right State Management Solution

**Decision Matrix for State Management:**

| Complexity Level | Application Size             | Team Size | Best Solution           | Why?                                 |
| ---------------- | ---------------------------- | --------- | ----------------------- | ------------------------------------ |
| Simple           | Small (< 10 components)      | 1-2 devs  | Local State + Context   | Minimal overhead, easy to understand |
| Medium           | Medium (10-50 components)    | 2-5 devs  | Zustand/Redux Toolkit   | Balance of simplicity and power      |
| Complex          | Large (50+ components)       | 5+ devs   | NgRx/Redux + Saga       | Structured, scalable, debuggable     |
| Enterprise       | Very Large (100+ components) | 10+ devs  | NgRx + Effects + Entity | Full feature set, strict patterns    |

### 🔥 Modern State Management Implementation

#### React: Zustand + React Query Pattern

```typescript
// stores/auth.store.ts - Simple yet powerful auth state
import { create } from "zustand";
import { devtools, persist } from "zustand/middleware";
import { immer } from "zustand/middleware/immer";

interface User {
  id: string;
  email: string;
  name: string;
  role: "admin" | "user" | "moderator";
  permissions: string[];
  avatar?: string;
}

interface AuthState {
  // State
  user: User | null;
  token: string | null;
  isLoading: boolean;
  error: string | null;
  loginAttempts: number;

  // Actions
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
  updateProfile: (updates: Partial<User>) => Promise<void>;
  resetError: () => void;
}

interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      immer((set, get) => ({
        // Initial state
        user: null,
        token: null,
        isLoading: false,
        error: null,
        loginAttempts: 0,

        // Login action with comprehensive error handling
        login: async (credentials) => {
          set((state) => {
            state.isLoading = true;
            state.error = null;
          });

          try {
            // Rate limiting check
            if (get().loginAttempts >= 5) {
              throw new Error("Too many login attempts. Please try again later.");
            }

            const response = await fetch("/api/auth/login", {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify(credentials),
            });

            if (!response.ok) {
              const errorData = await response.json();
              throw new Error(errorData.message || "Login failed");
            }

            const { user, token, refreshToken } = await response.json();

            // Store tokens securely
            if (credentials.rememberMe) {
              localStorage.setItem("refresh_token", refreshToken);
            } else {
              sessionStorage.setItem("refresh_token", refreshToken);
            }

            set((state) => {
              state.user = user;
              state.token = token;
              state.isLoading = false;
              state.loginAttempts = 0;
            });
          } catch (error) {
            set((state) => {
              state.error = error instanceof Error ? error.message : "Login failed";
              state.isLoading = false;
              state.loginAttempts += 1;
            });
            throw error;
          }
        },

        // Logout with cleanup
        logout: () => {
          // Clear tokens
          localStorage.removeItem("refresh_token");
          sessionStorage.removeItem("refresh_token");

          // Clear user data
          set((state) => {
            state.user = null;
            state.token = null;
            state.error = null;
            state.loginAttempts = 0;
          });

          // Optional: Notify server
          fetch("/api/auth/logout", { method: "POST" }).catch(() => {
            // Silent fail for logout
          });
        },

        // Token refresh logic
        refreshToken: async () => {
          const refreshToken = localStorage.getItem("refresh_token") || sessionStorage.getItem("refresh_token");

          if (!refreshToken) {
            get().logout();
            return;
          }

          try {
            const response = await fetch("/api/auth/refresh", {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify({ refreshToken }),
            });

            if (!response.ok) {
              throw new Error("Token refresh failed");
            }

            const { token } = await response.json();

            set((state) => {
              state.token = token;
            });
          } catch (error) {
            get().logout();
            throw error;
          }
        },

        // Profile update
        updateProfile: async (updates) => {
          const { token, user } = get();
          if (!token || !user) throw new Error("Not authenticated");

          set((state) => {
            state.isLoading = true;
          });

          try {
            const response = await fetch("/api/user/profile", {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${token}`,
              },
              body: JSON.stringify(updates),
            });

            if (!response.ok) {
              throw new Error("Profile update failed");
            }

            const updatedUser = await response.json();

            set((state) => {
              state.user = { ...state.user, ...updatedUser };
              state.isLoading = false;
            });
          } catch (error) {
            set((state) => {
              state.error = error instanceof Error ? error.message : "Update failed";
              state.isLoading = false;
            });
            throw error;
          }
        },

        resetError: () => {
          set((state) => {
            state.error = null;
          });
        },
      })),
      {
        name: "auth-storage", // localStorage key
        partialize: (state) => ({
          user: state.user,
          token: state.token,
        }), // Only persist essential data
      }
    ),
    {
      name: "auth-store", // DevTools name
    }
  )
);

// Selectors for optimized renders
export const useUser = () => useAuthStore((state) => state.user);
export const useIsAuthenticated = () => useAuthStore((state) => !!state.token);
export const useAuthLoading = () => useAuthStore((state) => state.isLoading);
export const useAuthError = () => useAuthStore((state) => state.error);
```

#### Angular: NgRx with Entity Management

```typescript
// state/user/user.models.ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: "admin" | "user" | "moderator";
  permissions: string[];
  avatar?: string;
  isActive: boolean;
  lastLogin?: Date;
  createdAt: Date;
  updatedAt: Date;
}

export interface UserFilters {
  role?: string;
  isActive?: boolean;
  search?: string;
  page: number;
  limit: number;
  sortBy?: keyof User;
  sortOrder?: "asc" | "desc";
}

// state/user/user.actions.ts
import { createAction, props } from "@ngrx/store";
import { User, UserFilters } from "./user.models";

// Load users actions
export const loadUsers = createAction("[User List] Load Users", props<{ filters: UserFilters }>());

export const loadUsersSuccess = createAction(
  "[User API] Load Users Success",
  props<{
    users: User[];
    total: number;
    filters: UserFilters;
  }>()
);

export const loadUsersFailure = createAction("[User API] Load Users Failure", props<{ error: string }>());

// CRUD actions
export const createUser = createAction("[User Form] Create User", props<{ user: Omit<User, "id" | "createdAt" | "updatedAt"> }>());

export const createUserSuccess = createAction("[User API] Create User Success", props<{ user: User }>());

export const updateUser = createAction("[User Form] Update User", props<{ id: string; changes: Partial<User> }>());

export const updateUserSuccess = createAction("[User API] Update User Success", props<{ user: User }>());

export const deleteUser = createAction("[User List] Delete User", props<{ id: string }>());

export const deleteUserSuccess = createAction("[User API] Delete User Success", props<{ id: string }>());

// Bulk actions
export const bulkUpdateUsers = createAction("[User List] Bulk Update Users", props<{ ids: string[]; changes: Partial<User> }>());

export const bulkDeleteUsers = createAction("[User List] Bulk Delete Users", props<{ ids: string[] }>());

// state/user/user.reducer.ts
import { createReducer, on } from "@ngrx/store";
import { EntityState, EntityAdapter, createEntityAdapter } from "@ngrx/entity";
import { User, UserFilters } from "./user.models";
import * as UserActions from "./user.actions";

export interface UserState extends EntityState<User> {
  selectedUserId: string | null;
  filters: UserFilters;
  total: number;
  loading: boolean;
  error: string | null;
  lastUpdated: Date | null;
}

// Entity adapter for normalized state management
export const userAdapter: EntityAdapter<User> = createEntityAdapter<User>({
  selectId: (user: User) => user.id,
  sortComparer: (a: User, b: User) => a.name.localeCompare(b.name),
});

export const initialUserState: UserState = userAdapter.getInitialState({
  selectedUserId: null,
  filters: {
    page: 1,
    limit: 20,
    sortBy: "name",
    sortOrder: "asc",
  },
  total: 0,
  loading: false,
  error: null,
  lastUpdated: null,
});

export const userReducer = createReducer(
  initialUserState,

  // Load users
  on(UserActions.loadUsers, (state, { filters }) => ({
    ...state,
    loading: true,
    error: null,
    filters,
  })),

  on(UserActions.loadUsersSuccess, (state, { users, total, filters }) =>
    userAdapter.setAll(users, {
      ...state,
      loading: false,
      total,
      filters,
      lastUpdated: new Date(),
    })
  ),

  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error,
  })),

  // Create user
  on(UserActions.createUser, (state) => ({
    ...state,
    loading: true,
    error: null,
  })),

  on(UserActions.createUserSuccess, (state, { user }) =>
    userAdapter.addOne(user, {
      ...state,
      loading: false,
      total: state.total + 1,
    })
  ),

  // Update user
  on(UserActions.updateUser, (state) => ({
    ...state,
    loading: true,
    error: null,
  })),

  on(UserActions.updateUserSuccess, (state, { user }) =>
    userAdapter.updateOne(
      { id: user.id, changes: user },
      {
        ...state,
        loading: false,
      }
    )
  ),

  // Delete user
  on(UserActions.deleteUser, (state) => ({
    ...state,
    loading: true,
    error: null,
  })),

  on(UserActions.deleteUserSuccess, (state, { id }) =>
    userAdapter.removeOne(id, {
      ...state,
      loading: false,
      total: state.total - 1,
    })
  ),

  // Bulk operations
  on(UserActions.bulkUpdateUsers, (state, { ids, changes }) =>
    userAdapter.updateMany(
      ids.map((id) => ({ id, changes })),
      state
    )
  ),

  on(UserActions.bulkDeleteUsers, (state, { ids }) =>
    userAdapter.removeMany(ids, {
      ...state,
      total: state.total - ids.length,
    })
  )
);

// state/user/user.selectors.ts
import { createFeatureSelector, createSelector } from "@ngrx/store";
import { UserState, userAdapter } from "./user.reducer";

export const selectUserState = createFeatureSelector<UserState>("users");

// Entity selectors
const { selectIds, selectEntities, selectAll, selectTotal } = userAdapter.getSelectors();

export const selectAllUsers = createSelector(selectUserState, selectAll);
export const selectUserEntities = createSelector(selectUserState, selectEntities);
export const selectUserIds = createSelector(selectUserState, selectIds);
export const selectUserTotal = createSelector(selectUserState, selectTotal);

// Custom selectors
export const selectUsersLoading = createSelector(selectUserState, (state: UserState) => state.loading);

export const selectUsersError = createSelector(selectUserState, (state: UserState) => state.error);

export const selectUserFilters = createSelector(selectUserState, (state: UserState) => state.filters);

export const selectSelectedUser = createSelector(selectUserState, selectUserEntities, (state: UserState, entities) => (state.selectedUserId ? entities[state.selectedUserId] : null));

// Memoized filtered selectors
export const selectFilteredUsers = createSelector(selectAllUsers, selectUserFilters, (users: User[], filters: UserFilters) => {
  let filtered = users;

  // Apply role filter
  if (filters.role) {
    filtered = filtered.filter((user) => user.role === filters.role);
  }

  // Apply active filter
  if (filters.isActive !== undefined) {
    filtered = filtered.filter((user) => user.isActive === filters.isActive);
  }

  // Apply search filter
  if (filters.search) {
    const search = filters.search.toLowerCase();
    filtered = filtered.filter((user) => user.name.toLowerCase().includes(search) || user.email.toLowerCase().includes(search));
  }

  // Apply sorting
  if (filters.sortBy) {
    filtered = [...filtered].sort((a, b) => {
      const aValue = a[filters.sortBy!];
      const bValue = b[filters.sortBy!];

      if (aValue < bValue) return filters.sortOrder === "asc" ? -1 : 1;
      if (aValue > bValue) return filters.sortOrder === "asc" ? 1 : -1;
      return 0;
    });
  }

  return filtered;
});

// Pagination selector
export const selectPaginatedUsers = createSelector(selectFilteredUsers, selectUserFilters, (users: User[], filters: UserFilters) => {
  const start = (filters.page - 1) * filters.limit;
  const end = start + filters.limit;
  return users.slice(start, end);
});

// Statistics selectors
export const selectUserStats = createSelector(selectAllUsers, (users: User[]) => ({
  total: users.length,
  active: users.filter((u) => u.isActive).length,
  inactive: users.filter((u) => !u.isActive).length,
  admins: users.filter((u) => u.role === "admin").length,
  moderators: users.filter((u) => u.role === "moderator").length,
  regularUsers: users.filter((u) => u.role === "user").length,
}));
```

---

## API Integration & Response Filtering

### 🌐 Advanced API Service Architecture

#### React: Custom Hook with React Query

```typescript
// hooks/useApi.ts - Advanced API hook with caching and error handling
import { useQuery, useMutation, useQueryClient, UseQueryOptions, UseMutationOptions } from "@tanstack/react-query";
import { useAuthStore } from "../stores/auth.store";

// Types for API responses
interface ApiResponse<T> {
  data: T;
  meta?: {
    total?: number;
    page?: number;
    limit?: number;
    hasNext?: boolean;
    hasPrev?: boolean;
  };
  message?: string;
  status: "success" | "error";
}

interface ApiError {
  message: string;
  code: string;
  details?: Record<string, any>;
  timestamp: string;
}

// Advanced filtering and sorting
interface ApiFilters {
  search?: string;
  filters?: Record<string, any>;
  sort?: {
    field: string;
    direction: "asc" | "desc";
  }[];
  pagination?: {
    page: number;
    limit: number;
  };
  include?: string[]; // Related data to include
  fields?: string[]; // Specific fields to return
}

// HTTP client with interceptors
class ApiClient {
  private baseURL: string;
  private defaultHeaders: Record<string, string>;

  constructor(baseURL: string = "/api") {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      "Content-Type": "application/json",
    };
  }

  // Request interceptor
  private async request<T>(endpoint: string, options: RequestInit = {}): Promise<ApiResponse<T>> {
    const { token } = useAuthStore.getState();

    const url = `${this.baseURL}${endpoint}`;
    const config: RequestInit = {
      ...options,
      headers: {
        ...this.defaultHeaders,
        ...(token && { Authorization: `Bearer ${token}` }),
        ...options.headers,
      },
    };

    try {
      const response = await fetch(url, config);

      // Handle different response types
      if (!response.ok) {
        const errorData: ApiError = await response.json().catch(() => ({
          message: "Network error occurred",
          code: "NETWORK_ERROR",
          timestamp: new Date().toISOString(),
        }));

        // Handle specific error cases
        if (response.status === 401) {
          useAuthStore.getState().logout();
          throw new Error("Authentication required");
        }

        throw new Error(errorData.message || "API request failed");
      }

      // Handle empty responses
      if (response.status === 204) {
        return { data: null as T, status: "success" };
      }

      const data: ApiResponse<T> = await response.json();
      return data;
    } catch (error) {
      console.error("API Request Error:", error);
      throw error;
    }
  }

  // GET request with advanced filtering
  async get<T>(endpoint: string, filters?: ApiFilters): Promise<ApiResponse<T>> {
    const queryParams = this.buildQueryParams(filters);
    const url = queryParams ? `${endpoint}?${queryParams}` : endpoint;

    return this.request<T>(url, { method: "GET" });
  }

  // POST request
  async post<T>(endpoint: string, data?: any): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: "POST",
      body: data ? JSON.stringify(data) : undefined,
    });
  }

  // PUT request
  async put<T>(endpoint: string, data?: any): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: "PUT",
      body: data ? JSON.stringify(data) : undefined,
    });
  }

  // PATCH request
  async patch<T>(endpoint: string, data?: any): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: "PATCH",
      body: data ? JSON.stringify(data) : undefined,
    });
  }

  // DELETE request
  async delete<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, { method: "DELETE" });
  }

  // Build query parameters from filters
  private buildQueryParams(filters?: ApiFilters): string {
    if (!filters) return "";

    const params = new URLSearchParams();

    // Add search
    if (filters.search) {
      params.append("search", filters.search);
    }

    // Add filters
    if (filters.filters) {
      Object.entries(filters.filters).forEach(([key, value]) => {
        if (value !== undefined && value !== null && value !== "") {
          params.append(`filter[${key}]`, String(value));
        }
      });
    }

    // Add sorting
    if (filters.sort && filters.sort.length > 0) {
      filters.sort.forEach((sort, index) => {
        params.append(`sort[${index}][field]`, sort.field);
        params.append(`sort[${index}][direction]`, sort.direction);
      });
    }

    // Add pagination
    if (filters.pagination) {
      params.append("page", String(filters.pagination.page));
      params.append("limit", String(filters.pagination.limit));
    }

    // Add includes
    if (filters.include && filters.include.length > 0) {
      params.append("include", filters.include.join(","));
    }

    // Add field selection
    if (filters.fields && filters.fields.length > 0) {
      params.append("fields", filters.fields.join(","));
    }

    return params.toString();
  }
}

// Global API client instance
export const apiClient = new ApiClient();

// Custom hook for GET requests with caching
export function useApiQuery<T>(key: (string | number | object)[], endpoint: string, filters?: ApiFilters, options?: Omit<UseQueryOptions<ApiResponse<T>>, "queryKey" | "queryFn">) {
  return useQuery({
    queryKey: [key, filters].filter(Boolean),
    queryFn: () => apiClient.get<T>(endpoint, filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    gcTime: 10 * 60 * 1000, // 10 minutes
    ...options,
  });
}

// Custom hook for mutations with optimistic updates
export function useApiMutation<TData, TVariables>(mutationFn: (variables: TVariables) => Promise<ApiResponse<TData>>, options?: UseMutationOptions<ApiResponse<TData>, Error, TVariables>) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn,
    onSuccess: () => {
      // Invalidate and refetch relevant queries
      queryClient.invalidateQueries();
    },
    ...options,
  });
}

// Example: Users API hook
export function useUsers(filters?: ApiFilters) {
  return useApiQuery<User[]>(["users"], "/users", filters, {
    select: (response) => ({
      ...response,
      data:
        response.data?.map((user) => ({
          ...user,
          // Transform/filter response data
          displayName: `${user.name} (${user.role})`,
          isOnline: user.lastLogin && new Date(user.lastLogin) > new Date(Date.now() - 15 * 60 * 1000), // 15 min
        })) || [],
    }),
  });
}

// Example: Create user mutation
export function useCreateUser() {
  return useApiMutation((userData: Omit<User, "id" | "createdAt" | "updatedAt">) => apiClient.post<User>("/users", userData), {
    onSuccess: () => {
      // Show success notification
      console.log("User created successfully");
    },
    onError: (error) => {
      // Show error notification
      console.error("Failed to create user:", error.message);
    },
  });
}

// Example: Bulk operations hook
export function useBulkOperations() {
  const queryClient = useQueryClient();

  const bulkUpdate = useApiMutation((data: { ids: string[]; updates: Partial<User> }) => apiClient.patch<User[]>("/users/bulk", data));

  const bulkDelete = useApiMutation((ids: string[]) => apiClient.post<{ deleted: number }>("/users/bulk-delete", { ids }), {
    onSuccess: (response, ids) => {
      // Optimistically remove from cache
      queryClient.setQueryData(["users"], (old: any) => ({
        ...old,
        data: old?.data?.filter((user: User) => !ids.includes(user.id)) || [],
      }));
    },
  });

  return { bulkUpdate, bulkDelete };
}
```

#### Angular: Advanced HTTP Interceptors & Services

```typescript
// services/api.service.ts - Enterprise-grade API service
import { Injectable } from "@angular/core";
import { HttpClient, HttpParams, HttpHeaders } from "@angular/common/http";
import { Observable, throwError, BehaviorSubject } from "rxjs";
import { map, catchError, retry, timeout, shareReplay, finalize } from "rxjs/operators";

interface ApiResponse<T> {
  data: T;
  meta?: {
    total?: number;
    page?: number;
    limit?: number;
    hasNext?: boolean;
    hasPrev?: boolean;
  };
  message?: string;
  status: "success" | "error";
  timestamp: string;
}

interface ApiFilters {
  search?: string;
  filters?: Record<string, any>;
  sort?: Array<{
    field: string;
    direction: "asc" | "desc";
  }>;
  pagination?: {
    page: number;
    limit: number;
  };
  include?: string[];
  fields?: string[];
}

@Injectable({
  providedIn: "root",
})
export class ApiService {
  private readonly baseUrl = "/api";
  private readonly defaultTimeout = 30000; // 30 seconds
  private loadingSubject = new BehaviorSubject<boolean>(false);

  // Loading state observable
  public loading$ = this.loadingSubject.asObservable();

  constructor(private http: HttpClient) {}

  // GET request with advanced filtering
  get<T>(
    endpoint: string,
    filters?: ApiFilters,
    options?: {
      timeout?: number;
      cache?: boolean;
      retries?: number;
    }
  ): Observable<ApiResponse<T>> {
    const url = `${this.baseUrl}${endpoint}`;
    const params = this.buildHttpParams(filters);

    this.setLoading(true);

    let request$ = this.http
      .get<ApiResponse<T>>(url, {
        params,
        headers: this.getHeaders(),
      })
      .pipe(
        timeout(options?.timeout || this.defaultTimeout),
        retry(options?.retries || 2),
        map((response) => this.transformResponse(response)),
        catchError((error) => this.handleError(error)),
        finalize(() => this.setLoading(false))
      );

    // Add caching for GET requests if requested
    if (options?.cache) {
      request$ = request$.pipe(shareReplay(1));
    }

    return request$;
  }

  // POST request with validation
  post<T>(
    endpoint: string,
    data: any,
    options?: {
      timeout?: number;
      validateResponse?: boolean;
    }
  ): Observable<ApiResponse<T>> {
    const url = `${this.baseUrl}${endpoint}`;

    this.setLoading(true);

    return this.http
      .post<ApiResponse<T>>(url, data, {
        headers: this.getHeaders(),
      })
      .pipe(
        timeout(options?.timeout || this.defaultTimeout),
        map((response) => this.transformResponse(response, options?.validateResponse)),
        catchError((error) => this.handleError(error)),
        finalize(() => this.setLoading(false))
      );
  }

  // PUT request
  put<T>(endpoint: string, data: any): Observable<ApiResponse<T>> {
    const url = `${this.baseUrl}${endpoint}`;

    this.setLoading(true);

    return this.http
      .put<ApiResponse<T>>(url, data, {
        headers: this.getHeaders(),
      })
      .pipe(
        timeout(this.defaultTimeout),
        map((response) => this.transformResponse(response)),
        catchError((error) => this.handleError(error)),
        finalize(() => this.setLoading(false))
      );
  }

  // PATCH request for partial updates
  patch<T>(endpoint: string, data: Partial<any>): Observable<ApiResponse<T>> {
    const url = `${this.baseUrl}${endpoint}`;

    this.setLoading(true);

    return this.http
      .patch<ApiResponse<T>>(url, data, {
        headers: this.getHeaders(),
      })
      .pipe(
        timeout(this.defaultTimeout),
        map((response) => this.transformResponse(response)),
        catchError((error) => this.handleError(error)),
        finalize(() => this.setLoading(false))
      );
  }

  // DELETE request
  delete<T>(endpoint: string): Observable<ApiResponse<T>> {
    const url = `${this.baseUrl}${endpoint}`;

    this.setLoading(true);

    return this.http
      .delete<ApiResponse<T>>(url, {
        headers: this.getHeaders(),
      })
      .pipe(
        timeout(this.defaultTimeout),
        map((response) => this.transformResponse(response)),
        catchError((error) => this.handleError(error)),
        finalize(() => this.setLoading(false))
      );
  }

  // Bulk operations
  bulkOperation<T>(operation: "update" | "delete", data: { ids: string[]; updates?: Partial<any> }): Observable<ApiResponse<T>> {
    const endpoint = `/bulk/${operation}`;
    return this.post<T>(endpoint, data);
  }

  // File upload with progress
  uploadFile(endpoint: string, file: File, additionalData?: Record<string, any>): Observable<ApiResponse<any>> {
    const url = `${this.baseUrl}${endpoint}`;
    const formData = new FormData();

    formData.append("file", file);

    if (additionalData) {
      Object.entries(additionalData).forEach(([key, value]) => {
        formData.append(key, String(value));
      });
    }

    this.setLoading(true);

    return this.http
      .post<ApiResponse<any>>(url, formData, {
        headers: this.getHeaders(true), // Skip Content-Type for FormData
        reportProgress: true,
        observe: "response",
      })
      .pipe(
        map((response) => response.body!),
        catchError((error) => this.handleError(error)),
        finalize(() => this.setLoading(false))
      );
  }

  // Private helper methods
  private buildHttpParams(filters?: ApiFilters): HttpParams {
    let params = new HttpParams();

    if (!filters) return params;

    // Add search
    if (filters.search) {
      params = params.set("search", filters.search);
    }

    // Add filters
    if (filters.filters) {
      Object.entries(filters.filters).forEach(([key, value]) => {
        if (value !== undefined && value !== null && value !== "") {
          params = params.set(`filter[${key}]`, String(value));
        }
      });
    }

    // Add sorting
    if (filters.sort && filters.sort.length > 0) {
      filters.sort.forEach((sort, index) => {
        params = params.set(`sort[${index}][field]`, sort.field);
        params = params.set(`sort[${index}][direction]`, sort.direction);
      });
    }

    // Add pagination
    if (filters.pagination) {
      params = params.set("page", String(filters.pagination.page));
      params = params.set("limit", String(filters.pagination.limit));
    }

    // Add includes
    if (filters.include && filters.include.length > 0) {
      params = params.set("include", filters.include.join(","));
    }

    // Add field selection
    if (filters.fields && filters.fields.length > 0) {
      params = params.set("fields", filters.fields.join(","));
    }

    return params;
  }

  private getHeaders(skipContentType = false): HttpHeaders {
    let headers = new HttpHeaders();

    if (!skipContentType) {
      headers = headers.set("Content-Type", "application/json");
    }

    headers = headers.set("Accept", "application/json");

    // Add other headers as needed (auth token handled by interceptor)
    return headers;
  }

  private transformResponse<T>(response: ApiResponse<T>, validate = false): ApiResponse<T> {
    // Add timestamp if not present
    if (!response.timestamp) {
      response.timestamp = new Date().toISOString();
    }

    // Validate response structure if requested
    if (validate && !this.isValidResponse(response)) {
      throw new Error("Invalid response format");
    }

    return response;
  }

  private isValidResponse(response: any): boolean {
    return response && typeof response === "object" && "data" in response && "status" in response;
  }

  private handleError(error: any): Observable<never> {
    let errorMessage = "An error occurred";

    if (error.error?.message) {
      errorMessage = error.error.message;
    } else if (error.message) {
      errorMessage = error.message;
    } else if (error.status) {
      switch (error.status) {
        case 400:
          errorMessage = "Bad request";
          break;
        case 401:
          errorMessage = "Unauthorized";
          break;
        case 403:
          errorMessage = "Forbidden";
          break;
        case 404:
          errorMessage = "Not found";
          break;
        case 500:
          errorMessage = "Internal server error";
          break;
        default:
          errorMessage = `Error ${error.status}`;
      }
    }

    console.error("API Error:", error);
    return throwError(() => new Error(errorMessage));
  }

  private setLoading(loading: boolean): void {
    this.loadingSubject.next(loading);
  }
}

// HTTP Interceptor for authentication and error handling
import { Injectable } from "@angular/core";
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent, HttpErrorResponse } from "@angular/common/http";
import { Observable, throwError, BehaviorSubject } from "rxjs";
import { catchError, filter, take, switchMap } from "rxjs/operators";
import { AuthService } from "./auth.service";

@Injectable()
export class ApiInterceptor implements HttpInterceptor {
  private isRefreshing = false;
  private refreshTokenSubject: BehaviorSubject<any> = new BehaviorSubject<any>(null);

  constructor(private authService: AuthService) {}

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Add auth token to request
    const authToken = this.authService.getToken();

    if (authToken) {
      request = this.addToken(request, authToken);
    }

    return next.handle(request).pipe(
      catchError((error) => {
        if (error instanceof HttpErrorResponse && error.status === 401) {
          return this.handle401Error(request, next);
        } else {
          return throwError(() => error);
        }
      })
    );
  }

  private addToken(request: HttpRequest<any>, token: string): HttpRequest<any> {
    return request.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`,
      },
    });
  }

  private handle401Error(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    if (!this.isRefreshing) {
      this.isRefreshing = true;
      this.refreshTokenSubject.next(null);

      return this.authService.refreshToken().pipe(
        switchMap((token: any) => {
          this.isRefreshing = false;
          this.refreshTokenSubject.next(token);
          return next.handle(this.addToken(request, token));
        }),
        catchError((error) => {
          this.isRefreshing = false;
          this.authService.logout();
          return throwError(() => error);
        })
      );
    } else {
      return this.refreshTokenSubject.pipe(
        filter((token) => token != null),
        take(1),
        switchMap((jwt) => {
          return next.handle(this.addToken(request, jwt));
        })
      );
    }
  }
}
```

---

## Custom Hooks & Advanced Patterns

### 🎣 React Custom Hooks for Complex Logic

#### Advanced Data Fetching Hook

```typescript
// hooks/useAdvancedQuery.ts - Professional data fetching with caching
import { useState, useEffect, useRef, useCallback } from "react";
import { useDebounce } from "./useDebounce";

interface UseAdvancedQueryOptions<T> {
  enabled?: boolean;
  refetchOnWindowFocus?: boolean;
  refetchInterval?: number;
  staleTime?: number;
  cacheTime?: number;
  retryCount?: number;
  retryDelay?: number;
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
  transform?: (data: any) => T;
  placeholderData?: T;
}

interface QueryResult<T> {
  data: T | undefined;
  error: Error | null;
  isLoading: boolean;
  isError: boolean;
  isSuccess: boolean;
  refetch: () => Promise<void>;
  invalidate: () => void;
}

// In-memory cache for queries
const queryCache = new Map<
  string,
  {
    data: any;
    timestamp: number;
    staleTime: number;
  }
>();

export function useAdvancedQuery<T>(queryKey: string[], queryFn: () => Promise<T>, options: UseAdvancedQueryOptions<T> = {}): QueryResult<T> {
  const {
    enabled = true,
    refetchOnWindowFocus = false,
    refetchInterval,
    staleTime = 5 * 60 * 1000, // 5 minutes
    cacheTime = 10 * 60 * 1000, // 10 minutes
    retryCount = 3,
    retryDelay = 1000,
    onSuccess,
    onError,
    transform,
    placeholderData,
  } = options;

  const [state, setState] = useState<{
    data: T | undefined;
    error: Error | null;
    isLoading: boolean;
  }>({
    data: placeholderData,
    error: null,
    isLoading: false,
  });

  const retryCountRef = useRef(0);
  const cacheKey = queryKey.join("-");
  const abortControllerRef = useRef<AbortController>();

  // Check cache for existing data
  const getCachedData = useCallback((): T | undefined => {
    const cached = queryCache.get(cacheKey);
    if (cached && Date.now() - cached.timestamp < cached.staleTime) {
      return cached.data;
    }
    return undefined;
  }, [cacheKey]);

  // Execute query with retry logic
  const executeQuery = useCallback(async (): Promise<void> => {
    if (!enabled) return;

    // Check cache first
    const cachedData = getCachedData();
    if (cachedData) {
      setState((prev) => ({ ...prev, data: cachedData, isLoading: false }));
      return;
    }

    setState((prev) => ({ ...prev, isLoading: true, error: null }));

    // Abort previous request
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();

    try {
      const rawData = await queryFn();
      const data = transform ? transform(rawData) : rawData;

      // Cache the result
      queryCache.set(cacheKey, {
        data,
        timestamp: Date.now(),
        staleTime,
      });

      setState({
        data,
        error: null,
        isLoading: false,
      });

      retryCountRef.current = 0;
      onSuccess?.(data);
    } catch (error) {
      const err = error instanceof Error ? error : new Error("Unknown error");

      // Retry logic
      if (retryCountRef.current < retryCount && err.name !== "AbortError") {
        retryCountRef.current++;
        setTimeout(() => {
          executeQuery();
        }, retryDelay * retryCountRef.current);
        return;
      }

      setState({
        data: placeholderData,
        error: err,
        isLoading: false,
      });

      retryCountRef.current = 0;
      onError?.(err);
    }
  }, [enabled, queryFn, transform, onSuccess, onError, retryCount, retryDelay, getCachedData, placeholderData, cacheKey, staleTime]);

  // Refetch function
  const refetch = useCallback(async (): Promise<void> => {
    queryCache.delete(cacheKey); // Clear cache
    await executeQuery();
  }, [executeQuery, cacheKey]);

  // Invalidate cache
  const invalidate = useCallback((): void => {
    queryCache.delete(cacheKey);
  }, [cacheKey]);

  // Initial fetch
  useEffect(() => {
    executeQuery();
  }, [executeQuery]);

  // Refetch on window focus
  useEffect(() => {
    if (!refetchOnWindowFocus) return;

    const handleFocus = () => {
      if (document.visibilityState === "visible") {
        executeQuery();
      }
    };

    document.addEventListener("visibilitychange", handleFocus);
    return () => document.removeEventListener("visibilitychange", handleFocus);
  }, [executeQuery, refetchOnWindowFocus]);

  // Polling interval
  useEffect(() => {
    if (!refetchInterval) return;

    const interval = setInterval(() => {
      executeQuery();
    }, refetchInterval);

    return () => clearInterval(interval);
  }, [executeQuery, refetchInterval]);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []);

  return {
    data: state.data,
    error: state.error,
    isLoading: state.isLoading,
    isError: !!state.error,
    isSuccess: !!state.data && !state.error,
    refetch,
    invalidate,
  };
}

// Example usage hook
export function useUsers(filters?: UserFilters) {
  const debouncedFilters = useDebounce(filters, 300);

  return useAdvancedQuery(["users", JSON.stringify(debouncedFilters)], () => apiClient.get<User[]>("/users", debouncedFilters), {
    transform: (response) => response.data,
    onSuccess: (users) => {
      console.log(`Loaded ${users.length} users`);
    },
    onError: (error) => {
      console.error("Failed to load users:", error);
    },
    staleTime: 2 * 60 * 1000, // 2 minutes
    refetchOnWindowFocus: true,
  });
}
```

#### Form Management Hook

```typescript
// hooks/useAdvancedForm.ts - Comprehensive form handling
import { useState, useCallback, useEffect } from "react";

interface FormField {
  value: any;
  error?: string;
  touched: boolean;
  dirty: boolean;
}

interface ValidationRule {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  custom?: (value: any) => string | undefined;
  dependencies?: string[]; // Other fields this validation depends on
}

interface FormConfig<T> {
  initialValues: T;
  validationRules?: Partial<Record<keyof T, ValidationRule>>;
  onSubmit?: (values: T) => Promise<void> | void;
  validateOnChange?: boolean;
  validateOnBlur?: boolean;
  enableReinitialize?: boolean;
}

interface FormMethods<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  dirty: Partial<Record<keyof T, boolean>>;
  isValid: boolean;
  isSubmitting: boolean;
  isDirty: boolean;

  // Field methods
  setValue: (field: keyof T, value: any) => void;
  setError: (field: keyof T, error: string) => void;
  setTouched: (field: keyof T, touched?: boolean) => void;

  // Form methods
  setValues: (values: Partial<T>) => void;
  resetForm: (newValues?: T) => void;
  validateField: (field: keyof T) => Promise<boolean>;
  validateForm: () => Promise<boolean>;
  handleSubmit: (e?: React.FormEvent) => Promise<void>;

  // Utility methods
  getFieldProps: (field: keyof T) => {
    value: any;
    onChange: (e: React.ChangeEvent<any>) => void;
    onBlur: (e: React.FocusEvent<any>) => void;
    error?: string;
    helperText?: string;
  };
}

export function useAdvancedForm<T extends Record<string, any>>(config: FormConfig<T>): FormMethods<T> {
  const { initialValues, validationRules = {}, onSubmit, validateOnChange = true, validateOnBlur = true, enableReinitialize = false } = config;

  // Form state
  const [fields, setFields] = useState<Record<keyof T, FormField>>(() => {
    const initialFields: Record<keyof T, FormField> = {} as any;
    Object.keys(initialValues).forEach((key) => {
      initialFields[key as keyof T] = {
        value: initialValues[key as keyof T],
        touched: false,
        dirty: false,
      };
    });
    return initialFields;
  });

  const [isSubmitting, setIsSubmitting] = useState(false);

  // Reinitialize form if initial values change
  useEffect(() => {
    if (enableReinitialize) {
      setFields((prev) => {
        const newFields = { ...prev };
        Object.keys(initialValues).forEach((key) => {
          const fieldKey = key as keyof T;
          if (newFields[fieldKey] && !newFields[fieldKey].dirty) {
            newFields[fieldKey] = {
              ...newFields[fieldKey],
              value: initialValues[fieldKey],
            };
          }
        });
        return newFields;
      });
    }
  }, [initialValues, enableReinitialize]);

  // Validation function
  const validateField = useCallback(
    async (field: keyof T): Promise<boolean> => {
      const rule = validationRules[field];
      const value = fields[field]?.value;

      if (!rule) return true;

      let error: string | undefined;

      // Required validation
      if (rule.required && (!value || (typeof value === "string" && !value.trim()))) {
        error = `${String(field)} is required`;
      }

      // Length validations
      else if (value && typeof value === "string") {
        if (rule.minLength && value.length < rule.minLength) {
          error = `${String(field)} must be at least ${rule.minLength} characters`;
        } else if (rule.maxLength && value.length > rule.maxLength) {
          error = `${String(field)} must not exceed ${rule.maxLength} characters`;
        }
      }

      // Pattern validation
      if (!error && rule.pattern && value && !rule.pattern.test(String(value))) {
        error = `${String(field)} format is invalid`;
      }

      // Custom validation
      if (!error && rule.custom) {
        error = rule.custom(value);
      }

      // Dependency validation
      if (!error && rule.dependencies) {
        const dependentValues: Record<string, any> = {};
        rule.dependencies.forEach((dep) => {
          dependentValues[dep] = fields[dep as keyof T]?.value;
        });

        // You can implement custom dependency validation here
        // For example, password confirmation
        if (field === "confirmPassword" && value !== dependentValues.password) {
          error = "Passwords do not match";
        }
      }

      // Update field error
      setFields((prev) => ({
        ...prev,
        [field]: {
          ...prev[field],
          error,
        },
      }));

      return !error;
    },
    [fields, validationRules]
  );

  // Validate entire form
  const validateForm = useCallback(async (): Promise<boolean> => {
    const validationPromises = Object.keys(fields).map((field) => validateField(field as keyof T));

    const results = await Promise.all(validationPromises);
    return results.every((result) => result);
  }, [fields, validateField]);

  // Set field value
  const setValue = useCallback(
    (field: keyof T, value: any) => {
      setFields((prev) => ({
        ...prev,
        [field]: {
          ...prev[field],
          value,
          dirty: value !== initialValues[field],
          touched: true,
        },
      }));

      if (validateOnChange) {
        setTimeout(() => validateField(field), 0);
      }
    },
    [validateOnChange, validateField, initialValues]
  );

  // Set field error
  const setError = useCallback((field: keyof T, error: string) => {
    setFields((prev) => ({
      ...prev,
      [field]: {
        ...prev[field],
        error,
      },
    }));
  }, []);

  // Set field touched
  const setTouched = useCallback(
    (field: keyof T, touched = true) => {
      setFields((prev) => ({
        ...prev,
        [field]: {
          ...prev[field],
          touched,
        },
      }));

      if (validateOnBlur && touched) {
        setTimeout(() => validateField(field), 0);
      }
    },
    [validateOnBlur, validateField]
  );

  // Set multiple values
  const setValues = useCallback(
    (values: Partial<T>) => {
      setFields((prev) => {
        const newFields = { ...prev };
        Object.entries(values).forEach(([key, value]) => {
          const fieldKey = key as keyof T;
          newFields[fieldKey] = {
            ...newFields[fieldKey],
            value,
            dirty: value !== initialValues[fieldKey],
            touched: true,
          };
        });
        return newFields;
      });
    },
    [initialValues]
  );

  // Reset form
  const resetForm = useCallback(
    (newValues?: T) => {
      const resetValues = newValues || initialValues;
      setFields(() => {
        const resetFields: Record<keyof T, FormField> = {} as any;
        Object.keys(resetValues).forEach((key) => {
          resetFields[key as keyof T] = {
            value: resetValues[key as keyof T],
            touched: false,
            dirty: false,
          };
        });
        return resetFields;
      });
      setIsSubmitting(false);
    },
    [initialValues]
  );

  // Handle form submission
  const handleSubmit = useCallback(
    async (e?: React.FormEvent) => {
      e?.preventDefault();

      setIsSubmitting(true);

      try {
        const isValid = await validateForm();

        if (isValid && onSubmit) {
          const values: T = {} as T;
          Object.keys(fields).forEach((key) => {
            values[key as keyof T] = fields[key as keyof T].value;
          });

          await onSubmit(values);
        }
      } catch (error) {
        console.error("Form submission error:", error);
      } finally {
        setIsSubmitting(false);
      }
    },
    [fields, validateForm, onSubmit]
  );

  // Get field props for easy form integration
  const getFieldProps = useCallback(
    (field: keyof T) => ({
      value: fields[field]?.value || "",
      onChange: (e: React.ChangeEvent<any>) => {
        const value = e.target.type === "checkbox" ? e.target.checked : e.target.value;
        setValue(field, value);
      },
      onBlur: () => setTouched(field, true),
      error: fields[field]?.touched && !!fields[field]?.error,
      helperText: fields[field]?.touched ? fields[field]?.error : undefined,
    }),
    [fields, setValue, setTouched]
  );

  // Computed values
  const values: T = {} as T;
  const errors: Partial<Record<keyof T, string>> = {};
  const touched: Partial<Record<keyof T, boolean>> = {};
  const dirty: Partial<Record<keyof T, boolean>> = {};

  Object.keys(fields).forEach((key) => {
    const fieldKey = key as keyof T;
    values[fieldKey] = fields[fieldKey].value;
    if (fields[fieldKey].error) errors[fieldKey] = fields[fieldKey].error;
    if (fields[fieldKey].touched) touched[fieldKey] = fields[fieldKey].touched;
    if (fields[fieldKey].dirty) dirty[fieldKey] = fields[fieldKey].dirty;
  });

  const isValid = Object.values(errors).length === 0;
  const isDirty = Object.values(dirty).some(Boolean);

  return {
    values,
    errors,
    touched,
    dirty,
    isValid,
    isSubmitting,
    isDirty,
    setValue,
    setError,
    setTouched,
    setValues,
    resetForm,
    validateField,
    validateForm,
    handleSubmit,
    getFieldProps,
  };
}

// Example usage
export function useUserForm(initialUser?: Partial<User>) {
  return useAdvancedForm({
    initialValues: {
      name: initialUser?.name || "",
      email: initialUser?.email || "",
      role: initialUser?.role || "user",
      password: "",
      confirmPassword: "",
      isActive: initialUser?.isActive ?? true,
    },
    validationRules: {
      name: {
        required: true,
        minLength: 2,
        maxLength: 50,
      },
      email: {
        required: true,
        pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
        custom: (value) => {
          // Custom async validation could go here
          return undefined;
        },
      },
      password: {
        required: true,
        minLength: 8,
        pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
        custom: (value) => {
          if (!value) return undefined;
          if (!/(?=.*[a-z])/.test(value)) return "Password must contain lowercase letter";
          if (!/(?=.*[A-Z])/.test(value)) return "Password must contain uppercase letter";
          if (!/(?=.*\d)/.test(value)) return "Password must contain number";
          if (!/(?=.*[@$!%*?&])/.test(value)) return "Password must contain special character";
          return undefined;
        },
      },
      confirmPassword: {
        required: true,
        dependencies: ["password"],
      },
    },
    onSubmit: async (values) => {
      // Remove confirmPassword before submission
      const { confirmPassword, ...userData } = values;

      if (initialUser?.id) {
        await apiClient.put(`/users/${initialUser.id}`, userData);
      } else {
        await apiClient.post("/users", userData);
      }
    },
    validateOnChange: true,
    validateOnBlur: true,
  });
}
```

### 🎯 Angular Advanced Patterns

#### Custom Directive for Form Validation

```typescript
// directives/form-validation.directive.ts
import { Directive, Input, OnInit, OnDestroy, ElementRef, Renderer2, HostListener } from "@angular/core";
import { NgControl, AbstractControl, ValidationErrors, Validators } from "@angular/forms";
import { Subject, debounceTime, takeUntil } from "rxjs";

interface ValidationConfig {
  showErrors?: boolean;
  errorClass?: string;
  successClass?: string;
  validateOnBlur?: boolean;
  validateOnChange?: boolean;
  debounceTime?: number;
  customMessages?: Record<string, string>;
}

@Directive({
  selector: "[appFormValidation]",
  standalone: true,
})
export class FormValidationDirective implements OnInit, OnDestroy {
  @Input() validationConfig: ValidationConfig = {};

  private destroy$ = new Subject<void>();
  private valueChanges$ = new Subject<any>();
  private errorElement?: HTMLElement;

  private defaultConfig: ValidationConfig = {
    showErrors: true,
    errorClass: "error",
    successClass: "success",
    validateOnBlur: true,
    validateOnChange: true,
    debounceTime: 300,
    customMessages: {
      required: "This field is required",
      email: "Please enter a valid email address",
      minlength: "Input is too short",
      maxlength: "Input is too long",
      pattern: "Invalid format",
    },
  };

  constructor(private elementRef: ElementRef, private renderer: Renderer2, private ngControl: NgControl) {}

  ngOnInit() {
    if (!this.ngControl) return;

    const config = { ...this.defaultConfig, ...this.validationConfig };

    // Setup debounced validation
    this.valueChanges$.pipe(debounceTime(config.debounceTime || 300), takeUntil(this.destroy$)).subscribe(() => {
      this.updateValidationState();
    });

    // Listen to control value changes
    if (config.validateOnChange) {
      this.ngControl.valueChanges?.pipe(takeUntil(this.destroy$)).subscribe(() => {
        this.valueChanges$.next(null);
      });
    }

    // Listen to status changes
    this.ngControl.statusChanges?.pipe(takeUntil(this.destroy$)).subscribe(() => {
      this.updateValidationState();
    });
  }

  @HostListener("blur")
  onBlur() {
    const config = { ...this.defaultConfig, ...this.validationConfig };
    if (config.validateOnBlur) {
      this.updateValidationState();
    }
  }

  private updateValidationState() {
    if (!this.ngControl?.control) return;

    const control = this.ngControl.control;
    const config = { ...this.defaultConfig, ...this.validationConfig };

    // Remove existing classes
    this.renderer.removeClass(this.elementRef.nativeElement, config.errorClass);
    this.renderer.removeClass(this.elementRef.nativeElement, config.successClass);

    // Remove existing error message
    this.removeErrorMessage();

    if (control.touched || control.dirty) {
      if (control.invalid && control.errors) {
        // Add error class
        this.renderer.addClass(this.elementRef.nativeElement, config.errorClass!);

        // Show error message
        if (config.showErrors) {
          this.showErrorMessage(control.errors, config.customMessages);
        }
      } else if (control.valid) {
        // Add success class
        this.renderer.addClass(this.elementRef.nativeElement, config.successClass!);
      }
    }
  }

  private showErrorMessage(errors: ValidationErrors, customMessages?: Record<string, string>) {
    const errorKey = Object.keys(errors)[0];
    const errorValue = errors[errorKey];

    let message = customMessages?.[errorKey];

    if (!message) {
      // Generate default messages
      switch (errorKey) {
        case "required":
          message = "This field is required";
          break;
        case "email":
          message = "Please enter a valid email address";
          break;
        case "minlength":
          message = `Minimum length is ${errorValue.requiredLength} characters`;
          break;
        case "maxlength":
          message = `Maximum length is ${errorValue.requiredLength} characters`;
          break;
        case "pattern":
          message = "Invalid format";
          break;
        default:
          message = "Invalid input";
      }
    }

    this.errorElement = this.renderer.createElement("div");
    this.renderer.addClass(this.errorElement, "validation-error");
    this.renderer.setProperty(this.errorElement, "textContent", message);

    const parent = this.elementRef.nativeElement.parentNode;
    this.renderer.insertBefore(parent, this.errorElement, this.elementRef.nativeElement.nextSibling);
  }

  private removeErrorMessage() {
    if (this.errorElement) {
      this.renderer.removeChild(this.errorElement.parentNode, this.errorElement);
      this.errorElement = undefined;
    }
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
    this.removeErrorMessage();
  }
}

// Usage example
@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <mat-form-field>
        <mat-label>Email</mat-label>
        <input
          matInput
          formControlName="email"
          appFormValidation
          [validationConfig]="{
            customMessages: {
              required: 'Email is required',
              email: 'Please enter a valid email'
            }
          }"
        />
      </mat-form-field>

      <mat-form-field>
        <mat-label>Password</mat-label>
        <input
          matInput
          type="password"
          formControlName="password"
          appFormValidation
          [validationConfig]="{
            validateOnChange: true,
            debounceTime: 500
          }"
        />
      </mat-form-field>

      <button mat-raised-button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `,
})
export class UserFormComponent {
  userForm = this.fb.group({
    email: ["", [Validators.required, Validators.email]],
    password: ["", [Validators.required, Validators.minLength(8)]],
  });

  constructor(private fb: FormBuilder) {}

  onSubmit() {
    if (this.userForm.valid) {
      console.log("Form submitted:", this.userForm.value);
    }
  }
}
```

---

## Security Implementation

### 🔒 Authentication & Authorization

#### JWT Token Management

```typescript
// services/auth.service.ts - Secure authentication service
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { BehaviorSubject, Observable, throwError, timer } from "rxjs";
import { map, catchError, switchMap, take } from "rxjs/operators";
import { Router } from "@angular/router";

interface AuthTokens {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
  tokenType: string;
}

interface User {
  id: string;
  email: string;
  name: string;
  role: string;
  permissions: string[];
}

interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}

@Injectable({
  providedIn: "root",
})
export class AuthService {
  private readonly ACCESS_TOKEN_KEY = "access_token";
  private readonly REFRESH_TOKEN_KEY = "refresh_token";
  private readonly USER_KEY = "user_data";

  private currentUserSubject = new BehaviorSubject<User | null>(null);
  private isAuthenticatedSubject = new BehaviorSubject<boolean>(false);
  private tokenRefreshInProgress = false;

  public currentUser$ = this.currentUserSubject.asObservable();
  public isAuthenticated$ = this.isAuthenticatedSubject.asObservable();

  constructor(private http: HttpClient, private router: Router) {
    this.initializeAuth();
  }

  private initializeAuth(): void {
    const token = this.getAccessToken();
    const user = this.getStoredUser();

    if (token && user && !this.isTokenExpired(token)) {
      this.currentUserSubject.next(user);
      this.isAuthenticatedSubject.next(true);
      this.scheduleTokenRefresh();
    } else {
      this.logout();
    }
  }

  login(credentials: LoginCredentials): Observable<User> {
    return this.http
      .post<{
        user: User;
        tokens: AuthTokens;
      }>("/api/auth/login", credentials)
      .pipe(
        map((response) => {
          this.handleAuthSuccess(response.user, response.tokens);
          return response.user;
        }),
        catchError((error) => {
          console.error("Login failed:", error);
          return throwError(() => error);
        })
      );
  }

  register(userData: { name: string; email: string; password: string }): Observable<User> {
    return this.http
      .post<{
        user: User;
        tokens: AuthTokens;
      }>("/api/auth/register", userData)
      .pipe(
        map((response) => {
          this.handleAuthSuccess(response.user, response.tokens);
          return response.user;
        }),
        catchError((error) => {
          console.error("Registration failed:", error);
          return throwError(() => error);
        })
      );
  }

  logout(): void {
    // Clear tokens from storage
    this.clearStorage();

    // Update subjects
    this.currentUserSubject.next(null);
    this.isAuthenticatedSubject.next(false);

    // Notify server
    const refreshToken = this.getRefreshToken();
    if (refreshToken) {
      this.http
        .post("/api/auth/logout", { refreshToken })
        .pipe(take(1))
        .subscribe({
          error: (error) => console.warn("Logout notification failed:", error),
        });
    }

    // Redirect to login
    this.router.navigate(["/login"]);
  }

  refreshToken(): Observable<string> {
    if (this.tokenRefreshInProgress) {
      // Wait for ongoing refresh
      return this.currentUser$.pipe(
        switchMap(() => {
          const token = this.getAccessToken();
          if (!token) {
            throw new Error("No access token available");
          }
          return token;
        }),
        take(1)
      );
    }

    const refreshToken = this.getRefreshToken();
    if (!refreshToken) {
      this.logout();
      return throwError(() => new Error("No refresh token available"));
    }

    this.tokenRefreshInProgress = true;

    return this.http
      .post<{
        user: User;
        tokens: AuthTokens;
      }>("/api/auth/refresh", { refreshToken })
      .pipe(
        map((response) => {
          this.handleAuthSuccess(response.user, response.tokens);
          this.tokenRefreshInProgress = false;
          return response.tokens.accessToken;
        }),
        catchError((error) => {
          this.tokenRefreshInProgress = false;
          console.error("Token refresh failed:", error);
          this.logout();
          return throwError(() => error);
        })
      );
  }

  // Check if user has specific permission
  hasPermission(permission: string): boolean {
    const user = this.currentUserSubject.value;
    return user?.permissions?.includes(permission) ?? false;
  }

  // Check if user has specific role
  hasRole(role: string): boolean {
    const user = this.currentUserSubject.value;
    return user?.role === role;
  }

  // Check if user has any of the specified roles
  hasAnyRole(roles: string[]): boolean {
    const user = this.currentUserSubject.value;
    return user ? roles.includes(user.role) : false;
  }

  // Get current user
  getCurrentUser(): User | null {
    return this.currentUserSubject.value;
  }

  // Get access token
  getAccessToken(): string | null {
    return localStorage.getItem(this.ACCESS_TOKEN_KEY) || sessionStorage.getItem(this.ACCESS_TOKEN_KEY);
  }

  // Private helper methods
  private handleAuthSuccess(user: User, tokens: AuthTokens): void {
    // Store tokens securely
    this.storeTokens(tokens);

    // Store user data
    this.storeUser(user);

    // Update subjects
    this.currentUserSubject.next(user);
    this.isAuthenticatedSubject.next(true);

    // Schedule token refresh
    this.scheduleTokenRefresh();
  }

  private storeTokens(tokens: AuthTokens): void {
    const storage = tokens.expiresIn > 3600 ? localStorage : sessionStorage;

    storage.setItem(this.ACCESS_TOKEN_KEY, tokens.accessToken);
    storage.setItem(this.REFRESH_TOKEN_KEY, tokens.refreshToken);
  }

  private storeUser(user: User): void {
    const storage = this.getAccessToken() ? (localStorage.getItem(this.ACCESS_TOKEN_KEY) ? localStorage : sessionStorage) : sessionStorage;

    storage.setItem(this.USER_KEY, JSON.stringify(user));
  }

  private getStoredUser(): User | null {
    const userData = localStorage.getItem(this.USER_KEY) || sessionStorage.getItem(this.USER_KEY);

    return userData ? JSON.parse(userData) : null;
  }

  private getRefreshToken(): string | null {
    return localStorage.getItem(this.REFRESH_TOKEN_KEY) || sessionStorage.getItem(this.REFRESH_TOKEN_KEY);
  }

  private clearStorage(): void {
    [localStorage, sessionStorage].forEach((storage) => {
      storage.removeItem(this.ACCESS_TOKEN_KEY);
      storage.removeItem(this.REFRESH_TOKEN_KEY);
      storage.removeItem(this.USER_KEY);
    });
  }

  private isTokenExpired(token: string): boolean {
    try {
      const payload = JSON.parse(atob(token.split(".")[1]));
      return Date.now() >= payload.exp * 1000;
    } catch {
      return true;
    }
  }

  private scheduleTokenRefresh(): void {
    const token = this.getAccessToken();
    if (!token) return;

    try {
      const payload = JSON.parse(atob(token.split(".")[1]));
      const expiresAt = payload.exp * 1000;
      const refreshAt = expiresAt - 5 * 60 * 1000; // Refresh 5 minutes before expiry
      const delay = refreshAt - Date.now();

      if (delay > 0) {
        timer(delay)
          .pipe(take(1))
          .subscribe(() => {
            this.refreshToken().subscribe({
              error: () => this.logout(),
            });
          });
      }
    } catch (error) {
      console.error("Failed to schedule token refresh:", error);
    }
  }
}
```

#### Content Security Policy (CSP) Implementation

```typescript
// security/csp.service.ts - Content Security Policy management
import { Injectable } from "@angular/core";

interface CSPConfig {
  "default-src"?: string[];
  "script-src"?: string[];
  "style-src"?: string[];
  "img-src"?: string[];
  "connect-src"?: string[];
  "font-src"?: string[];
  "frame-src"?: string[];
  "media-src"?: string[];
  "object-src"?: string[];
  "base-uri"?: string[];
  "form-action"?: string[];
  "frame-ancestors"?: string[];
  "upgrade-insecure-requests"?: boolean;
  "block-all-mixed-content"?: boolean;
}

@Injectable({
  providedIn: "root",
})
export class CSPService {
  private defaultConfig: CSPConfig = {
    "default-src": ["'self'"],
    "script-src": [
      "'self'",
      "'unsafe-inline'", // Only for development
      "https://cdn.jsdelivr.net",
      "https://unpkg.com",
    ],
    "style-src": [
      "'self'",
      "'unsafe-inline'", // Needed for Angular Material
      "https://fonts.googleapis.com",
    ],
    "img-src": ["'self'", "data:", "https://images.unsplash.com", "https://via.placeholder.com"],
    "connect-src": ["'self'", "https://api.example.com", "wss://api.example.com"],
    "font-src": ["'self'", "https://fonts.gstatic.com"],
    "frame-src": ["'none'"],
    "object-src": ["'none'"],
    "base-uri": ["'self'"],
    "form-action": ["'self'"],
    "frame-ancestors": ["'none'"],
    "upgrade-insecure-requests": true,
    "block-all-mixed-content": true,
  };

  generateCSPHeader(customConfig?: Partial<CSPConfig>): string {
    const config = { ...this.defaultConfig, ...customConfig };
    const directives: string[] = [];

    Object.entries(config).forEach(([directive, value]) => {
      if (typeof value === "boolean") {
        if (value) {
          directives.push(directive);
        }
      } else if (Array.isArray(value)) {
        directives.push(`${directive} ${value.join(" ")}`);
      }
    });

    return directives.join("; ");
  }

  // Validate CSP policy
  validateCSP(policy: string): { valid: boolean; errors: string[] } {
    const errors: string[] = [];

    // Check for common security issues
    if (policy.includes("'unsafe-eval'")) {
      errors.push("'unsafe-eval' allows dangerous code execution");
    }

    if (policy.includes("*") && !policy.includes("data:")) {
      errors.push("Wildcard (*) allows all sources - too permissive");
    }

    if (!policy.includes("'self'")) {
      errors.push("Missing 'self' directive - may block own resources");
    }

    return {
      valid: errors.length === 0,
      errors,
    };
  }

  // Set CSP meta tag (for client-side implementation)
  setCSPMetaTag(customConfig?: Partial<CSPConfig>): void {
    const policy = this.generateCSPHeader(customConfig);

    // Remove existing CSP meta tag
    const existingTag = document.querySelector('meta[http-equiv="Content-Security-Policy"]');
    if (existingTag) {
      existingTag.remove();
    }

    // Create new CSP meta tag
    const metaTag = document.createElement("meta");
    metaTag.setAttribute("http-equiv", "Content-Security-Policy");
    metaTag.setAttribute("content", policy);
    document.head.appendChild(metaTag);
  }
}

// Example usage in app initialization
export function initializeCSP(): () => void {
  return () => {
    const cspService = new CSPService();

    // Set CSP for development/production
    const isDevelopment = !environment.production;

    const cspConfig: Partial<CSPConfig> = {
      "script-src": isDevelopment ? ["'self'", "'unsafe-inline'", "'unsafe-eval'", "https://cdn.jsdelivr.net"] : ["'self'", "https://cdn.jsdelivr.net"],
      "connect-src": ["'self'", environment.apiUrl],
    };

    cspService.setCSPMetaTag(cspConfig);
  };
}
```

## Guards & Route Protection

### 🛡️ Advanced Route Guards

#### Angular Route Guards

```typescript
// guards/auth.guard.ts - Advanced authentication guard
import { Injectable } from "@angular/core";
import { CanActivate, CanActivateChild, CanDeactivate, CanLoad, ActivatedRouteSnapshot, RouterStateSnapshot, Router, UrlTree } from "@angular/router";
import { Observable, of } from "rxjs";
import { map, catchError, take } from "rxjs/operators";
import { AuthService } from "../services/auth.service";
import { NotificationService } from "../services/notification.service";

export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

@Injectable({
  providedIn: "root",
})
export class AuthGuard implements CanActivate, CanActivateChild, CanLoad {
  constructor(private authService: AuthService, private router: Router, private notificationService: NotificationService) {}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    return this.checkAuth(route, state.url);
  }

  canActivateChild(childRoute: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    return this.checkAuth(childRoute, state.url);
  }

  canLoad(): Observable<boolean> | Promise<boolean> | boolean {
    return this.authService.isAuthenticated$.pipe(
      take(1),
      map((isAuthenticated) => {
        if (!isAuthenticated) {
          this.router.navigate(["/login"]);
          return false;
        }
        return true;
      })
    );
  }

  private checkAuth(route: ActivatedRouteSnapshot, url: string): Observable<boolean | UrlTree> {
    return this.authService.isAuthenticated$.pipe(
      take(1),
      map((isAuthenticated) => {
        if (isAuthenticated) {
          // Check for route-specific permissions
          const requiredPermissions = route.data?.["permissions"] as string[];
          const requiredRoles = route.data?.["roles"] as string[];

          if (requiredPermissions && requiredPermissions.length > 0) {
            const hasPermission = requiredPermissions.some((permission) => this.authService.hasPermission(permission));

            if (!hasPermission) {
              this.notificationService.error("You do not have permission to access this page");
              return this.router.parseUrl("/dashboard");
            }
          }

          if (requiredRoles && requiredRoles.length > 0) {
            const hasRole = this.authService.hasAnyRole(requiredRoles);

            if (!hasRole) {
              this.notificationService.error("Your role does not allow access to this page");
              return this.router.parseUrl("/dashboard");
            }
          }

          return true;
        } else {
          // Store attempted URL for redirect after login
          this.authService.setRedirectUrl(url);
          return this.router.parseUrl("/login");
        }
      }),
      catchError(() => {
        this.router.navigate(["/login"]);
        return of(false);
      })
    );
  }
}

// guards/role.guard.ts - Role-based access control
@Injectable({
  providedIn: "root",
})
export class RoleGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router, private notificationService: NotificationService) {}

  canActivate(route: ActivatedRouteSnapshot): Observable<boolean | UrlTree> | boolean | UrlTree {
    const requiredRoles = route.data?.["roles"] as string[];
    const requiredPermissions = route.data?.["permissions"] as string[];

    if (!requiredRoles && !requiredPermissions) {
      return true; // No restrictions
    }

    const user = this.authService.getCurrentUser();

    if (!user) {
      return this.router.parseUrl("/login");
    }

    // Check roles
    if (requiredRoles && requiredRoles.length > 0) {
      const hasRole = requiredRoles.includes(user.role);
      if (!hasRole) {
        this.notificationService.error(`Access denied. Required roles: ${requiredRoles.join(", ")}`);
        return this.router.parseUrl("/unauthorized");
      }
    }

    // Check permissions
    if (requiredPermissions && requiredPermissions.length > 0) {
      const hasAllPermissions = requiredPermissions.every((permission) => user.permissions.includes(permission));

      if (!hasAllPermissions) {
        this.notificationService.error(`Access denied. Missing permissions: ${requiredPermissions.join(", ")}`);
        return this.router.parseUrl("/unauthorized");
      }
    }

    return true;
  }
}

// guards/unsaved-changes.guard.ts - Prevent navigation with unsaved changes
@Injectable({
  providedIn: "root",
})
export class UnsavedChangesGuard implements CanDeactivate<CanComponentDeactivate> {
  constructor(private notificationService: NotificationService) {}

  canDeactivate(component: CanComponentDeactivate): Observable<boolean> | Promise<boolean> | boolean {
    if (component.canDeactivate && !component.canDeactivate()) {
      return this.notificationService.confirm("You have unsaved changes. Are you sure you want to leave?", "Unsaved Changes");
    }
    return true;
  }
}

// Usage in routing module
const routes: Routes = [
  {
    path: "admin",
    loadChildren: () => import("./admin/admin.module").then((m) => m.AdminModule),
    canLoad: [AuthGuard],
    canActivate: [AuthGuard, RoleGuard],
    data: {
      roles: ["admin"],
      permissions: ["admin.access"],
    },
  },
  {
    path: "user-profile",
    component: UserProfileComponent,
    canActivate: [AuthGuard],
    canDeactivate: [UnsavedChangesGuard],
    data: {
      permissions: ["profile.edit"],
    },
  },
  {
    path: "reports",
    loadChildren: () => import("./reports/reports.module").then((m) => m.ReportsModule),
    canLoad: [AuthGuard],
    canActivateChild: [RoleGuard],
    data: {
      roles: ["admin", "manager"],
      permissions: ["reports.view"],
    },
  },
];
```

#### React Route Protection

```typescript
// components/ProtectedRoute.tsx - Advanced route protection for React
import React, { ReactNode, useEffect, useState } from "react";
import { Navigate, useLocation } from "react-router-dom";
import { useAuthStore } from "../stores/auth.store";
import { LoadingSpinner } from "./ui/LoadingSpinner";
import { UnauthorizedPage } from "../pages/UnauthorizedPage";

interface ProtectedRouteProps {
  children: ReactNode;
  requiredRoles?: string[];
  requiredPermissions?: string[];
  fallback?: ReactNode;
  redirectTo?: string;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ children, requiredRoles = [], requiredPermissions = [], fallback = <UnauthorizedPage />, redirectTo = "/login" }) => {
  const { user, token, isLoading } = useAuthStore();
  const location = useLocation();
  const [isChecking, setIsChecking] = useState(true);

  useEffect(() => {
    // Simulate async permission check (could be API call)
    const checkPermissions = async () => {
      // Add delay to prevent flash
      await new Promise((resolve) => setTimeout(resolve, 100));
      setIsChecking(false);
    };

    checkPermissions();
  }, []);

  // Show loading while checking auth status
  if (isLoading || isChecking) {
    return <LoadingSpinner />;
  }

  // Redirect to login if not authenticated
  if (!token || !user) {
    return <Navigate to={redirectTo} state={{ from: location }} replace />;
  }

  // Check role requirements
  if (requiredRoles.length > 0) {
    const hasRole = requiredRoles.includes(user.role);
    if (!hasRole) {
      console.warn(`Access denied. User role: ${user.role}, Required: ${requiredRoles.join(", ")}`);
      return <>{fallback}</>;
    }
  }

  // Check permission requirements
  if (requiredPermissions.length > 0) {
    const hasAllPermissions = requiredPermissions.every((permission) => user.permissions.includes(permission));

    if (!hasAllPermissions) {
      const missingPermissions = requiredPermissions.filter((permission) => !user.permissions.includes(permission));
      console.warn(`Access denied. Missing permissions: ${missingPermissions.join(", ")}`);
      return <>{fallback}</>;
    }
  }

  // All checks passed - render protected content
  return <>{children}</>;
};

// Higher-order component for route protection
export function withRouteProtection<P extends object>(Component: React.ComponentType<P>, options: Omit<ProtectedRouteProps, "children">) {
  return function ProtectedComponent(props: P) {
    return (
      <ProtectedRoute {...options}>
        <Component {...props} />
      </ProtectedRoute>
    );
  };
}

// Hook for permission checking
export function usePermissions() {
  const { user } = useAuthStore();

  const hasPermission = (permission: string): boolean => {
    return user?.permissions?.includes(permission) ?? false;
  };

  const hasAnyPermission = (permissions: string[]): boolean => {
    return permissions.some((permission) => hasPermission(permission));
  };

  const hasAllPermissions = (permissions: string[]): boolean => {
    return permissions.every((permission) => hasPermission(permission));
  };

  const hasRole = (role: string): boolean => {
    return user?.role === role;
  };

  const hasAnyRole = (roles: string[]): boolean => {
    return user ? roles.includes(user.role) : false;
  };

  const canAccess = (requiredRoles?: string[], requiredPermissions?: string[]): boolean => {
    if (!user) return false;

    if (requiredRoles && requiredRoles.length > 0) {
      if (!hasAnyRole(requiredRoles)) return false;
    }

    if (requiredPermissions && requiredPermissions.length > 0) {
      if (!hasAllPermissions(requiredPermissions)) return false;
    }

    return true;
  };

  return {
    hasPermission,
    hasAnyPermission,
    hasAllPermissions,
    hasRole,
    hasAnyRole,
    canAccess,
    user,
  };
}

// Component for conditional rendering based on permissions
interface PermissionGateProps {
  children: ReactNode;
  requiredRoles?: string[];
  requiredPermissions?: string[];
  fallback?: ReactNode;
  operator?: "AND" | "OR"; // How to combine multiple permissions
}

export const PermissionGate: React.FC<PermissionGateProps> = ({ children, requiredRoles = [], requiredPermissions = [], fallback = null, operator = "AND" }) => {
  const { canAccess, hasAnyPermission, hasAllPermissions, hasAnyRole } = usePermissions();

  let hasAccess = true;

  if (requiredRoles.length > 0 && requiredPermissions.length > 0) {
    // Both roles and permissions specified
    if (operator === "AND") {
      hasAccess = hasAnyRole(requiredRoles) && hasAllPermissions(requiredPermissions);
    } else {
      hasAccess = hasAnyRole(requiredRoles) || hasAnyPermission(requiredPermissions);
    }
  } else {
    // Use the canAccess helper for single type checks
    hasAccess = canAccess(requiredRoles.length > 0 ? requiredRoles : undefined, requiredPermissions.length > 0 ? requiredPermissions : undefined);
  }

  return hasAccess ? <>{children}</> : <>{fallback}</>;
};

// Usage examples
export const App: React.FC = () => {
  return (
    <Router>
      <Routes>
        {/* Public routes */}
        <Route path="/login" element={<LoginPage />} />
        <Route path="/register" element={<RegisterPage />} />

        {/* Protected routes */}
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <DashboardPage />
            </ProtectedRoute>
          }
        />

        {/* Admin only routes */}
        <Route
          path="/admin/*"
          element={
            <ProtectedRoute requiredRoles={["admin"]} requiredPermissions={["admin.access"]}>
              <AdminLayout />
            </ProtectedRoute>
          }
        />

        {/* Manager and admin routes */}
        <Route
          path="/reports/*"
          element={
            <ProtectedRoute requiredRoles={["admin", "manager"]} requiredPermissions={["reports.view"]}>
              <ReportsLayout />
            </ProtectedRoute>
          }
        />
      </Routes>
    </Router>
  );
};

// Component with conditional rendering
export const UserActions: React.FC = () => {
  return (
    <div className="user-actions">
      {/* Always visible */}
      <button>View Profile</button>

      {/* Only for users with edit permission */}
      <PermissionGate requiredPermissions={["profile.edit"]}>
        <button>Edit Profile</button>
      </PermissionGate>

      {/* Only for admins or managers */}
      <PermissionGate requiredRoles={["admin", "manager"]} operator="OR">
        <button>Manage Users</button>
      </PermissionGate>

      {/* Complex permission logic */}
      <PermissionGate requiredRoles={["admin"]} requiredPermissions={["system.configure"]} fallback={<span>Access Denied</span>}>
        <button>System Settings</button>
      </PermissionGate>
    </div>
  );
};
```

---

## Advanced Routing Strategies

### 🛣️ Dynamic Route Loading

#### Angular Lazy Loading with Preloading

```typescript
// routing/preloading-strategy.service.ts - Custom preloading strategy
import { Injectable } from "@angular/core";
import { PreloadingStrategy, Route } from "@angular/router";
import { Observable, of, timer } from "rxjs";
import { switchMap } from "rxjs/operators";

@Injectable({
  providedIn: "root",
})
export class CustomPreloadingStrategy implements PreloadingStrategy {
  private preloadedModules: Set<string> = new Set();

  preload(route: Route, fn: () => Observable<any>): Observable<any> {
    // Don't preload if already preloaded
    if (this.preloadedModules.has(route.path || "")) {
      return of(null);
    }

    // Check preloading conditions
    if (this.shouldPreload(route)) {
      console.log(`Preloading module: ${route.path}`);
      this.preloadedModules.add(route.path || "");

      // Add delay to prevent blocking main thread
      return timer(this.getPreloadDelay(route)).pipe(switchMap(() => fn()));
    }

    return of(null);
  }

  private shouldPreload(route: Route): boolean {
    // Preload based on route data
    if (route.data?.["preload"] === false) {
      return false;
    }

    // Preload high-priority routes immediately
    if (route.data?.["priority"] === "high") {
      return true;
    }

    // Preload based on user role
    if (route.data?.["roles"]) {
      // Check if user has required role (would need AuthService)
      return true; // Simplified for example
    }

    // Default preload for routes with preload flag
    return route.data?.["preload"] === true;
  }

  private getPreloadDelay(route: Route): number {
    const priority = route.data?.["priority"];

    switch (priority) {
      case "high":
        return 100; // Preload almost immediately
      case "medium":
        return 2000; // Preload after 2 seconds
      case "low":
        return 5000; // Preload after 5 seconds
      default:
        return 1000; // Default 1 second delay
    }
  }
}

// app-routing.module.ts - Advanced routing configuration
const routes: Routes = [
  {
    path: "",
    redirectTo: "/dashboard",
    pathMatch: "full",
  },
  {
    path: "dashboard",
    loadChildren: () => import("./dashboard/dashboard.module").then((m) => m.DashboardModule),
    data: {
      preload: true,
      priority: "high",
      breadcrumb: "Dashboard",
    },
  },
  {
    path: "users",
    loadChildren: () => import("./users/users.module").then((m) => m.UsersModule),
    canLoad: [AuthGuard],
    data: {
      preload: true,
      priority: "medium",
      roles: ["admin", "manager"],
      breadcrumb: "User Management",
    },
  },
  {
    path: "reports",
    loadChildren: () => import("./reports/reports.module").then((m) => m.ReportsModule),
    canLoad: [AuthGuard],
    data: {
      preload: false, // Load only when needed
      priority: "low",
      roles: ["admin"],
      breadcrumb: "Reports",
    },
  },
  {
    path: "settings",
    loadChildren: () => import("./settings/settings.module").then((m) => m.SettingsModule),
    data: {
      preload: true,
      priority: "low",
      breadcrumb: "Settings",
    },
  },
  {
    path: "**",
    loadChildren: () => import("./not-found/not-found.module").then((m) => m.NotFoundModule),
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      // Enable router preloading with custom strategy
      preloadingStrategy: CustomPreloadingStrategy,

      // Enable tracing for debugging (disable in production)
      enableTracing: !environment.production,

      // Scroll to top on route change
      scrollPositionRestoration: "top",

      // Preserve query params and fragments
      paramsInheritanceStrategy: "always",

      // Cancel pending navigations
      canceledNavigationResolution: "replace",
    }),
  ],
  exports: [RouterModule],
  providers: [CustomPreloadingStrategy],
})
export class AppRoutingModule {}
```

#### React Code Splitting with Suspense

```typescript
// utils/lazy-loading.tsx - Advanced lazy loading utilities
import React, { Suspense, ComponentType, lazy, ReactNode } from "react";
import { ErrorBoundary } from "./ErrorBoundary";
import { LoadingSpinner } from "../components/ui/LoadingSpinner";

interface LazyComponentOptions {
  fallback?: ReactNode;
  errorFallback?: ComponentType<{ error: Error; retry: () => void }>;
  retryCount?: number;
  timeout?: number;
}

// Enhanced lazy loading with retry mechanism
export function createLazyComponent<T extends ComponentType<any>>(factory: () => Promise<{ default: T }>, options: LazyComponentOptions = {}): ComponentType<any> {
  const { fallback = <LoadingSpinner />, errorFallback: ErrorFallback, retryCount = 3, timeout = 10000 } = options;

  let retries = 0;

  const LazyComponent = lazy(() => {
    return Promise.race([
      factory().catch((error) => {
        // Retry logic
        if (retries < retryCount) {
          retries++;
          console.warn(`Failed to load component, retrying... (${retries}/${retryCount})`);
          return factory();
        }
        throw error;
      }),
      // Timeout promise
      new Promise<never>((_, reject) => {
        setTimeout(() => {
          reject(new Error(`Component loading timed out after ${timeout}ms`));
        }, timeout);
      }),
    ]);
  });

  return function WrappedLazyComponent(props: any) {
    const handleRetry = () => {
      retries = 0;
      window.location.reload(); // Simple retry strategy
    };

    return (
      <ErrorBoundary fallback={ErrorFallback ? (error) => <ErrorFallback error={error} retry={handleRetry} /> : undefined}>
        <Suspense fallback={fallback}>
          <LazyComponent {...props} />
        </Suspense>
      </ErrorBoundary>
    );
  };
}

// Route-based code splitting
export const lazyRoutes = {
  Dashboard: createLazyComponent(() => import("../pages/Dashboard"), { fallback: <div>Loading Dashboard...</div> }),
  UserManagement: createLazyComponent(() => import("../pages/UserManagement"), {
    fallback: <div>Loading User Management...</div>,
    timeout: 15000, // Longer timeout for complex pages
  }),
  Reports: createLazyComponent(() => import("../pages/Reports"), { fallback: <div>Loading Reports...</div> }),
  Settings: createLazyComponent(() => import("../pages/Settings"), { fallback: <div>Loading Settings...</div> }),
};

// Smart preloading hook
export function useRoutePreloading() {
  const [preloadedRoutes, setPreloadedRoutes] = React.useState<Set<string>>(new Set());

  const preloadRoute = React.useCallback(
    (routeName: string) => {
      if (preloadedRoutes.has(routeName)) {
        return; // Already preloaded
      }

      // Preload the component
      const componentLoader = {
        Dashboard: () => import("../pages/Dashboard"),
        UserManagement: () => import("../pages/UserManagement"),
        Reports: () => import("../pages/Reports"),
        Settings: () => import("../pages/Settings"),
      }[routeName];

      if (componentLoader) {
        componentLoader()
          .then(() => {
            setPreloadedRoutes((prev) => new Set([...prev, routeName]));
            console.log(`Preloaded route: ${routeName}`);
          })
          .catch((error) => {
            console.error(`Failed to preload route ${routeName}:`, error);
          });
      }
    },
    [preloadedRoutes]
  );

  const preloadOnHover = React.useCallback(
    (routeName: string) => {
      return {
        onMouseEnter: () => preloadRoute(routeName),
        onFocus: () => preloadRoute(routeName),
      };
    },
    [preloadRoute]
  );

  return { preloadRoute, preloadOnHover, preloadedRoutes };
}

// Advanced routing component with preloading
export const AppRouter: React.FC = () => {
  const { preloadOnHover } = useRoutePreloading();

  return (
    <Router>
      <nav>
        <Link to="/dashboard" {...preloadOnHover("Dashboard")}>
          Dashboard
        </Link>
        <Link to="/users" {...preloadOnHover("UserManagement")}>
          Users
        </Link>
        <Link to="/reports" {...preloadOnHover("Reports")}>
          Reports
        </Link>
        <Link to="/settings" {...preloadOnHover("Settings")}>
          Settings
        </Link>
      </nav>

      <main>
        <Routes>
          <Route path="/dashboard" element={<lazyRoutes.Dashboard />} />

          <Route
            path="/users"
            element={
              <ProtectedRoute requiredRoles={["admin", "manager"]}>
                <lazyRoutes.UserManagement />
              </ProtectedRoute>
            }
          />

          <Route
            path="/reports"
            element={
              <ProtectedRoute requiredRoles={["admin"]}>
                <lazyRoutes.Reports />
              </ProtectedRoute>
            }
          />

          <Route path="/settings" element={<lazyRoutes.Settings />} />

          <Route path="*" element={<NotFoundPage />} />
        </Routes>
      </main>
    </Router>
  );
};
```

---

## UI Framework Integration: Material + Tailwind

### 🎨 Combining Material Design with Tailwind CSS

#### Angular Material + Tailwind Setup

```typescript
// styles/material-tailwind-theme.scss - Unified theme system
@use '@angular/material' as mat;
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';

// Define custom color palette that works with both Material and Tailwind
$custom-primary: (
  50: #e8f4fd,
  100: #c5e4fa,
  200: #9fd2f7,
  300: #78c0f4,
  400: #5bb2f1,
  500: #3ea4ee,
  600: #389cec,
  700: #3092e9,
  800: #2888e7,
  900: #1a76e2,
  A100: #ffffff,
  A200: #d6e9ff,
  A400: #a3d1ff,
  A700: #8ac6ff,
  contrast: (
    50: rgba(black, 0.87),
    100: rgba(black, 0.87),
    200: rgba(black, 0.87),
    300: rgba(black, 0.87),
    400: rgba(black, 0.87),
    500: white,
    600: white,
    700: white,
    800: white,
    900: white,
    A100: rgba(black, 0.87),
    A200: rgba(black, 0.87),
    A400: rgba(black, 0.87),
    A700: rgba(black, 0.87),
  )
);

$custom-accent: (
  50: #fce4ec,
  100: #f8bbd9,
  200: #f48fb1,
  300: #f06292,
  400: #ec407a,
  500: #e91e63,
  600: #d81b60,
  700: #c2185b,
  800: #ad1457,
  900: #880e4f,
  A100: #ff80ab,
  A200: #ff4081,
  A400: #f50057,
  A700: #c51162,
  contrast: (
    50: rgba(black, 0.87),
    100: rgba(black, 0.87),
    200: rgba(black, 0.87),
    300: white,
    400: white,
    500: white,
    600: white,
    700: white,
    800: white,
    900: white,
    A100: rgba(black, 0.87),
    A200: white,
    A400: white,
    A700: white,
  )
);

// Create Material theme
$primary-palette: mat.define-palette($custom-primary, 500);
$accent-palette: mat.define-palette($custom-accent, 500);
$warn-palette: mat.define-palette(mat.$red-palette);

$light-theme: mat.define-light-theme((
  color: (
    primary: $primary-palette,
    accent: $accent-palette,
    warn: $warn-palette,
  ),
  typography: mat.define-typography-config(
    $font-family: '"Inter", sans-serif',
    $headline-1: mat.define-typography-level(32px, 40px, 700),
    $headline-2: mat.define-typography-level(28px, 36px, 600),
    $headline-3: mat.define-typography-level(24px, 32px, 600),
    $headline-4: mat.define-typography-level(20px, 28px, 600),
    $headline-5: mat.define-typography-level(18px, 24px, 500),
    $headline-6: mat.define-typography-level(16px, 22px, 500),
    $body-1: mat.define-typography-level(14px, 20px, 400),
    $body-2: mat.define-typography-level(12px, 18px, 400),
  ),
  density: 0
));

$dark-theme: mat.define-dark-theme((
  color: (
    primary: $primary-palette,
    accent: $accent-palette,
    warn: $warn-palette,
  ),
  typography: mat.define-typography-config(
    $font-family: '"Inter", sans-serif',
  ),
  density: 0
));

// Include Material themes
@include mat.all-component-themes($light-theme);

// Dark theme
.dark-theme {
  @include mat.all-component-colors($dark-theme);
}

// Custom Material component styles that work with Tailwind
.mat-card {
  @apply shadow-lg rounded-lg border border-gray-200;

  &.elevated {
    @apply shadow-xl;
  }

  .mat-card-header {
    @apply border-b border-gray-100 pb-4 mb-4;
  }
}

.mat-button, .mat-raised-button, .mat-flat-button {
  @apply font-medium transition-all duration-200;

  &:hover {
    @apply transform scale-105;
  }
}

.mat-form-field {
  @apply w-full;

  .mat-form-field-outline {
    @apply rounded-lg;
  }

  &.mat-focused .mat-form-field-outline-thick {
    @apply border-blue-500;
  }
}

// Custom utility classes combining Material and Tailwind
.material-card {
  @apply bg-white rounded-xl shadow-lg border border-gray-100 overflow-hidden;

  .card-header {
    @apply px-6 py-4 border-b border-gray-100 bg-gray-50;
  }

  .card-content {
    @apply px-6 py-4;
  }

  .card-actions {
    @apply px-6 py-4 border-t border-gray-100 bg-gray-50 flex justify-end space-x-2;
  }
}

.material-button {
  @apply inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm;
  @apply transition-all duration-200 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-offset-2;

  &.primary {
    @apply text-white bg-blue-600 hover:bg-blue-700 focus:ring-blue-500;
  }

  &.secondary {
    @apply text-gray-700 bg-white hover:bg-gray-50 border-gray-300 focus:ring-blue-500;
  }

  &.accent {
    @apply text-white bg-pink-600 hover:bg-pink-700 focus:ring-pink-500;
  }
}
```

#### Tailwind Configuration for Material Integration

```javascript
// tailwind.config.js - Extended configuration for Material Design
const { createThemes } = require("tw-colors");

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{html,ts,tsx,js,jsx}"],
  darkMode: "class",
  theme: {
    extend: {
      // Material Design color system
      colors: {
        primary: {
          50: "#e8f4fd",
          100: "#c5e4fa",
          200: "#9fd2f7",
          300: "#78c0f4",
          400: "#5bb2f1",
          500: "#3ea4ee",
          600: "#389cec",
          700: "#3092e9",
          800: "#2888e7",
          900: "#1a76e2",
        },
        accent: {
          50: "#fce4ec",
          100: "#f8bbd9",
          200: "#f48fb1",
          300: "#f06292",
          400: "#ec407a",
          500: "#e91e63",
          600: "#d81b60",
          700: "#c2185b",
          800: "#ad1457",
          900: "#880e4f",
        },
        surface: {
          50: "#fafafa",
          100: "#f5f5f5",
          200: "#eeeeee",
          300: "#e0e0e0",
          400: "#bdbdbd",
          500: "#9e9e9e",
          600: "#757575",
          700: "#616161",
          800: "#424242",
          900: "#212121",
        },
      },

      // Material Design typography
      fontFamily: {
        sans: ["Inter", "Roboto", "system-ui", "sans-serif"],
        display: ["Inter", "Roboto", "system-ui", "sans-serif"],
      },

      fontSize: {
        "headline-1": ["32px", { lineHeight: "40px", fontWeight: "700" }],
        "headline-2": ["28px", { lineHeight: "36px", fontWeight: "600" }],
        "headline-3": ["24px", { lineHeight: "32px", fontWeight: "600" }],
        "headline-4": ["20px", { lineHeight: "28px", fontWeight: "600" }],
        "headline-5": ["18px", { lineHeight: "24px", fontWeight: "500" }],
        "headline-6": ["16px", { lineHeight: "22px", fontWeight: "500" }],
        "body-1": ["14px", { lineHeight: "20px", fontWeight: "400" }],
        "body-2": ["12px", { lineHeight: "18px", fontWeight: "400" }],
        caption: ["11px", { lineHeight: "16px", fontWeight: "400" }],
      },

      // Material Design spacing
      spacing: {
        18: "4.5rem",
        88: "22rem",
      },

      // Material Design shadows
      boxShadow: {
        "material-1": "0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24)",
        "material-2": "0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23)",
        "material-3": "0 10px 20px rgba(0, 0, 0, 0.19), 0 6px 6px rgba(0, 0, 0, 0.23)",
        "material-4": "0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22)",
        "material-5": "0 19px 38px rgba(0, 0, 0, 0.30), 0 15px 12px rgba(0, 0, 0, 0.22)",
      },

      // Material Design border radius
      borderRadius: {
        material: "4px",
        "material-lg": "8px",
        "material-xl": "12px",
      },

      // Animation curves
      transitionTimingFunction: {
        "material-standard": "cubic-bezier(0.4, 0.0, 0.2, 1)",
        "material-deceleration": "cubic-bezier(0.0, 0.0, 0.2, 1)",
        "material-acceleration": "cubic-bezier(0.4, 0.0, 1, 1)",
        "material-sharp": "cubic-bezier(0.4, 0.0, 0.6, 1)",
      },

      // Material Design breakpoints
      screens: {
        xs: "600px",
        sm: "960px",
        md: "1280px",
        lg: "1920px",
      },
    },
  },
  plugins: [
    require("@tailwindcss/forms"),
    require("@tailwindcss/typography"),
    require("@tailwindcss/aspect-ratio"),

    // Custom plugin for Material Design utilities
    function ({ addUtilities, addComponents, theme }) {
      // Material Design elevation utilities
      const elevations = {};
      for (let i = 0; i <= 24; i++) {
        elevations[`.elevation-${i}`] = {
          boxShadow: theme(`boxShadow.material-${Math.min(Math.floor(i / 5) + 1, 5)}`),
        };
      }
      addUtilities(elevations);

      // Material Design component classes
      addComponents({
        ".mat-card": {
          backgroundColor: theme("colors.white"),
          borderRadius: theme("borderRadius.material-lg"),
          boxShadow: theme("boxShadow.material-1"),
          padding: theme("spacing.6"),
          transition: "box-shadow 280ms cubic-bezier(0.4, 0, 0.2, 1)",

          "&:hover": {
            boxShadow: theme("boxShadow.material-2"),
          },
        },

        ".mat-button": {
          display: "inline-flex",
          alignItems: "center",
          justifyContent: "center",
          padding: `${theme("spacing.2")} ${theme("spacing.4")}`,
          borderRadius: theme("borderRadius.material"),
          fontSize: theme("fontSize.body-1[0]"),
          fontWeight: theme("fontSize.body-1[1].fontWeight"),
          lineHeight: theme("fontSize.body-1[1].lineHeight"),
          textTransform: "uppercase",
          letterSpacing: "0.025em",
          transition: "all 280ms cubic-bezier(0.4, 0, 0.2, 1)",
          cursor: "pointer",
          border: "none",
          outline: "none",

          "&:focus": {
            boxShadow: `0 0 0 2px ${theme("colors.primary.500")}40`,
          },

          "&:disabled": {
            opacity: "0.38",
            cursor: "not-allowed",
          },
        },

        ".mat-raised-button": {
          backgroundColor: theme("colors.primary.500"),
          color: theme("colors.white"),
          boxShadow: theme("boxShadow.material-1"),

          "&:hover": {
            backgroundColor: theme("colors.primary.600"),
            boxShadow: theme("boxShadow.material-2"),
          },

          "&:active": {
            boxShadow: theme("boxShadow.material-3"),
          },
        },

        ".mat-form-field": {
          position: "relative",
          width: "100%",
          marginBottom: theme("spacing.4"),

          ".mat-form-field-label": {
            position: "absolute",
            top: theme("spacing.3"),
            left: theme("spacing.3"),
            fontSize: theme("fontSize.body-1[0]"),
            color: theme("colors.gray.500"),
            transition: "all 200ms cubic-bezier(0.4, 0, 0.2, 1)",
            pointerEvents: "none",
          },

          ".mat-form-field-input": {
            width: "100%",
            padding: theme("spacing.3"),
            border: `1px solid ${theme("colors.gray.300")}`,
            borderRadius: theme("borderRadius.material"),
            fontSize: theme("fontSize.body-1[0]"),
            outline: "none",
            transition: "border-color 200ms cubic-bezier(0.4, 0, 0.2, 1)",

            "&:focus": {
              borderColor: theme("colors.primary.500"),

              "+ .mat-form-field-label": {
                transform: "translateY(-1.5rem) scale(0.75)",
                color: theme("colors.primary.500"),
              },
            },

            "&:not(:placeholder-shown) + .mat-form-field-label": {
              transform: "translateY(-1.5rem) scale(0.75)",
            },
          },
        },
      });
    },

    // Theme plugin for dark mode
    createThemes({
      light: {
        surface: "#ffffff",
        "on-surface": "#1f2937",
        "surface-variant": "#f3f4f6",
        "on-surface-variant": "#6b7280",
      },
      dark: {
        surface: "#1f2937",
        "on-surface": "#f9fafb",
        "surface-variant": "#374151",
        "on-surface-variant": "#d1d5db",
      },
    }),
  ],
};
```

#### Unified Component Library

```typescript
// components/ui/Card.tsx - Unified Material + Tailwind card component
import React, { ReactNode, forwardRef } from "react";
import { cn } from "../../utils/cn"; // className utility function

interface CardProps {
  children: ReactNode;
  className?: string;
  elevation?: 0 | 1 | 2 | 3 | 4 | 5;
  variant?: "default" | "outlined" | "filled";
  hover?: boolean;
  onClick?: () => void;
}

export const Card = forwardRef<HTMLDivElement, CardProps>(({ children, className, elevation = 1, variant = "default", hover = false, onClick }, ref) => {
  const baseClasses = "rounded-material-lg transition-all duration-300 material-standard";

  const variantClasses = {
    default: "bg-surface border border-gray-200",
    outlined: "bg-surface border-2 border-primary-200",
    filled: "bg-primary-50 border border-primary-200",
  };

  const elevationClasses = {
    0: "shadow-none",
    1: "shadow-material-1",
    2: "shadow-material-2",
    3: "shadow-material-3",
    4: "shadow-material-4",
    5: "shadow-material-5",
  };

  const hoverClasses = hover ? "hover:shadow-material-3 hover:scale-[1.02] cursor-pointer" : "";

  return (
    <div ref={ref} className={cn(baseClasses, variantClasses[variant], elevationClasses[elevation], hoverClasses, className)} onClick={onClick}>
      {children}
    </div>
  );
});

Card.displayName = "Card";

// Card sub-components
export const CardHeader = ({ children, className }: { children: ReactNode; className?: string }) => <div className={cn("px-6 py-4 border-b border-gray-100", className)}>{children}</div>;

export const CardContent = ({ children, className }: { children: ReactNode; className?: string }) => <div className={cn("px-6 py-4", className)}>{children}</div>;

export const CardActions = ({ children, className }: { children: ReactNode; className?: string }) => <div className={cn("px-6 py-4 border-t border-gray-100 flex justify-end space-x-2", className)}>{children}</div>;

// Example usage component
export const UserCard: React.FC<{ user: User }> = ({ user }) => {
  return (
    <Card elevation={2} hover className="max-w-sm">
      <CardHeader>
        <div className="flex items-center space-x-4">
          <img src={user.avatar || "/default-avatar.png"} alt={user.name} className="w-12 h-12 rounded-full object-cover" />
          <div>
            <h3 className="text-headline-6 font-medium text-on-surface">{user.name}</h3>
            <p className="text-body-2 text-on-surface-variant">{user.role}</p>
          </div>
        </div>
      </CardHeader>

      <CardContent>
        <p className="text-body-1 text-on-surface-variant mb-4">{user.email}</p>
        <div className="flex items-center space-x-2">
          <span className={cn("inline-flex items-center px-2 py-1 rounded-full text-caption font-medium", user.isActive ? "bg-green-100 text-green-800" : "bg-red-100 text-red-800")}>{user.isActive ? "Active" : "Inactive"}</span>
          <span className="text-caption text-on-surface-variant">Last login: {user.lastLogin ? new Date(user.lastLogin).toLocaleDateString() : "Never"}</span>
        </div>
      </CardContent>

      <CardActions>
        <button className="mat-button text-primary-600 hover:bg-primary-50">Edit</button>
        <button className="mat-raised-button bg-primary-600 text-white hover:bg-primary-700">View Details</button>
      </CardActions>
    </Card>
  );
};
```

---

## Summary & Best Practices

### 🎯 Key Takeaways

#### 1. State Management Strategy

- **Choose the right tool for your scale**: Context API for simple state → Zustand for moderate complexity → Redux/NgRx for enterprise
- **Implement proper data normalization**: Use entity adapters and normalized data structures
- **Leverage reactive patterns**: Use signals, observables, and reactive programming for optimal performance
- **Separate concerns**: Keep business logic separate from UI state

#### 2. API Integration Excellence

- **Design reusable service layers**: Create consistent, type-safe API clients with proper error handling
- **Implement comprehensive caching**: Use appropriate caching strategies (memory, HTTP, IndexedDB)
- **Handle offline scenarios**: Implement proper offline support and data synchronization
- **Use TypeScript everywhere**: Ensure type safety across API contracts and data models

#### 3. Security First Approach

- **Always validate inputs**: Implement client-side validation with server-side verification
- **Secure authentication flows**: Use proper JWT handling with refresh tokens and secure storage
- **Implement CSP headers**: Use Content Security Policy to prevent XSS attacks
- **Follow OWASP guidelines**: Implement security best practices consistently

#### 4. Performance Optimization

- **Implement lazy loading**: Use virtual scrolling, code splitting, and dynamic imports
- **Monitor performance**: Track Core Web Vitals and implement performance budgets
- **Optimize rendering**: Use memoization, pure components, and efficient change detection
- **Minimize bundle size**: Implement tree shaking and analyze bundle composition

#### 5. Testing Excellence

- **Write comprehensive tests**: Cover unit, integration, and end-to-end scenarios
- **Test user interactions**: Focus on user behavior rather than implementation details
- **Implement accessibility tests**: Ensure your application works for all users
- **Use proper mocking**: Mock external dependencies consistently and correctly

### 🚀 Implementation Roadmap

#### Phase 1: Foundation (Weeks 1-2)

1. Set up basic state management (Zustand/NgRx)
2. Implement core API service layer
3. Add basic authentication and routing
4. Set up testing framework

#### Phase 2: Enhancement (Weeks 3-4)

1. Add advanced state patterns (entity management, caching)
2. Implement custom hooks and reactive patterns
3. Add comprehensive error handling
4. Implement security measures (CSP, input validation)

#### Phase 3: Optimization (Weeks 5-6)

1. Add performance monitoring and optimization
2. Implement virtual scrolling and lazy loading
3. Add offline support and PWA features
4. Optimize bundle size and loading times

#### Phase 4: Polish (Weeks 7-8)

1. Add comprehensive testing coverage
2. Implement accessibility features
3. Add internationalization support
4. Performance tuning and monitoring

### 📊 Decision Matrix: Choosing Technologies

| Requirement      | React Solution         | Angular Solution            | Key Considerations                       |
| ---------------- | ---------------------- | --------------------------- | ---------------------------------------- |
| Simple State     | Context API            | Angular Services            | Small applications, minimal complexity   |
| Complex State    | Zustand                | NgRx                        | Growing applications, team collaboration |
| Enterprise State | Redux Toolkit          | NgRx + Signals              | Large teams, complex business logic      |
| API Calls        | React Query + Axios    | HttpClient + RxJS           | Caching requirements, offline support    |
| Forms            | React Hook Form        | Reactive Forms              | Validation complexity, dynamic forms     |
| Routing          | React Router           | Angular Router              | Route guards, lazy loading needs         |
| UI Framework     | Material-UI + Tailwind | Angular Material + Tailwind | Design system requirements               |
| Testing          | Jest + RTL             | Jasmine + Karma             | Team expertise, CI/CD integration        |

### 🔧 Development Tools & Extensions

#### Essential VS Code Extensions

- **State Management**: Redux DevTools, NgRx Store DevTools
- **Testing**: Jest Runner, Coverage Gutters
- **Code Quality**: ESLint, Prettier, SonarLint
- **Performance**: Bundle Analyzer, Lighthouse CI
- **Accessibility**: axe Accessibility Linter

#### Recommended NPM Packages

```json
{
  "dependencies": {
    "@tanstack/react-query": "^4.0.0",
    "zustand": "^4.0.0",
    "@reduxjs/toolkit": "^1.9.0",
    "axios": "^1.0.0",
    "react-hook-form": "^7.0.0",
    "@mui/material": "^5.0.0",
    "tailwindcss": "^3.0.0"
  },
  "devDependencies": {
    "@testing-library/react": "^13.0.0",
    "@testing-library/jest-dom": "^5.0.0",
    "@testing-library/user-event": "^14.0.0",
    "cypress": "^12.0.0",
    "@storybook/react": "^6.5.0"
  }
}
```

### 🎯 Performance Benchmarks

#### Target Metrics

- **First Contentful Paint**: < 1.5s
- **Largest Contentful Paint**: < 2.5s
- **Cumulative Layout Shift**: < 0.1
- **First Input Delay**: < 100ms
- **Bundle Size**: < 200KB (gzipped)
- **Memory Usage**: < 50MB initial, < 100MB after interaction

#### Monitoring Tools

- **Lighthouse CI**: Automated performance testing
- **Web Vitals**: Core performance metrics
- **Bundle Analyzer**: Bundle size analysis
- **Chrome DevTools**: Performance profiling
- **Sentry**: Error tracking and performance monitoring

### 📚 Additional Resources

#### Documentation & Guides

- [React Query Documentation](https://tanstack.com/query)
- [Zustand GitHub Repository](https://github.com/pmndrs/zustand)
- [NgRx Documentation](https://ngrx.io/)
- [Angular Material Design System](https://material.angular.io/)
- [OWASP Frontend Security Checklist](https://owasp.org/www-project-top-ten/)

#### Community & Learning

- [React Patterns](https://reactpatterns.com/)
- [Angular University](https://angular-university.io/)
- [JavaScript Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
- [Web.dev Performance Guides](https://web.dev/performance/)

#### Tools & Libraries

- [Storybook for Component Development](https://storybook.js.org/)
- [Playwright for E2E Testing](https://playwright.dev/)
- [React Testing Library](https://testing-library.com/)
- [Angular Testing Utilities](https://angular.io/guide/testing)

### 🎓 Final Notes

This guide provides a comprehensive foundation for building modern, scalable, and secure frontend applications. The patterns and practices outlined here have been battle-tested in production environments and represent current industry best practices.

**Remember to:**

- Start simple and gradually add complexity
- Always measure performance before optimizing
- Implement security considerations from day one
- Write tests as you develop, not after
- Document your architectural decisions
- Keep learning and adapting to new patterns

**Key Success Factors:**

1. **Team Alignment**: Ensure your team understands and agrees on the chosen patterns
2. **Consistent Implementation**: Follow established patterns consistently across the codebase
3. **Regular Reviews**: Conduct regular code reviews and architecture discussions
4. **Continuous Learning**: Stay updated with evolving best practices and technologies
5. **User Focus**: Always prioritize user experience and accessibility

The frontend landscape continues to evolve rapidly, but the fundamental principles of good architecture, security, performance, and testing remain constant. Use this guide as a foundation, but be prepared to adapt and evolve your approach as new challenges and opportunities arise.

---

_Built with ❤️ for the frontend development community. Happy coding!_
