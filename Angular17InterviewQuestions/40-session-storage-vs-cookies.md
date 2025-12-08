# 🍪 **Session Storage vs Cookies: Complete Comparison Guide**

## 🎯 **What You'll Learn**

This comprehensive guide explores the differences between Session Storage and Cookies, their use cases, implementation patterns, and best practices for web applications with practical Angular examples.

---

## 📚 **The Basics: Storage Mechanisms Overview**

### **🤔 What Are These Storage Options?**

**Session Storage:**

- 🗄️ **Browser-based storage** that persists for the browser session
- 📱 **Per-tab isolation** - data not shared between tabs
- 🔒 **Client-side only** - never sent to server automatically
- 📏 **Larger capacity** - typically 5-10MB per origin

**Cookies:**

- 🍪 **Small text files** stored by the browser
- 🌐 **Server communication** - sent with every HTTP request
- 🔄 **Persistent** - can survive browser restarts
- 📏 **Limited size** - typically 4KB per cookie

---

## 🔍 **Detailed Comparison Table**

| Feature                  | 🍪 Cookies               | 🗄️ Session Storage | 🏪 Local Storage       |
| ------------------------ | ------------------------ | ------------------ | ---------------------- |
| **Storage Capacity**     | ~4KB per cookie          | ~5-10MB            | ~5-10MB                |
| **Persistence**          | Until expiry date        | Until tab closed   | Until manually cleared |
| **Server Communication** | Sent automatically       | Never sent         | Never sent             |
| **Scope**                | Domain + subdomain       | Origin + tab       | Origin (all tabs)      |
| **Accessibility**        | Client + Server          | Client only        | Client only            |
| **Security**             | HttpOnly, Secure flags   | XSS vulnerable     | XSS vulnerable         |
| **Performance**          | Impacts HTTP requests    | No network impact  | No network impact      |
| **Use Cases**            | Authentication, tracking | Temporary UI state | User preferences       |

---

## 🛠️ **Implementation in Angular**

### **🍪 Cookie Service Implementation**

```typescript
// src/app/services/cookie.service.ts
import { Injectable, Inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";

export interface CookieOptions {
  expires?: Date | string | number;
  path?: string;
  domain?: string;
  secure?: boolean;
  httpOnly?: boolean;
  sameSite?: "strict" | "lax" | "none";
}

@Injectable({
  providedIn: "root",
})
export class CookieService {
  constructor(@Inject(PLATFORM_ID) private platformId: Object) {}

  // 🍪 SET COOKIE
  setCookie(name: string, value: string, options: CookieOptions = {}): void {
    if (!isPlatformBrowser(this.platformId)) {
      return; // Skip in SSR environment
    }

    let cookieString = `${encodeURIComponent(name)}=${encodeURIComponent(
      value
    )}`;

    // Add expiration
    if (options.expires) {
      const expires = this.getExpiresDate(options.expires);
      cookieString += `; expires=${expires.toUTCString()}`;
    }

    // Add path
    if (options.path) {
      cookieString += `; path=${options.path}`;
    } else {
      cookieString += "; path=/";
    }

    // Add domain
    if (options.domain) {
      cookieString += `; domain=${options.domain}`;
    }

    // Add secure flag
    if (options.secure) {
      cookieString += "; secure";
    }

    // Add SameSite
    if (options.sameSite) {
      cookieString += `; samesite=${options.sameSite}`;
    }

    // Note: httpOnly can only be set on the server
    document.cookie = cookieString;

    console.log(`🍪 Cookie set: ${name}`);
  }

  // 🔍 GET COOKIE
  getCookie(name: string): string | null {
    if (!isPlatformBrowser(this.platformId)) {
      return null;
    }

    const cookies = document.cookie.split(";");

    for (let cookie of cookies) {
      const [cookieName, cookieValue] = cookie.trim().split("=");

      if (decodeURIComponent(cookieName) === name) {
        return decodeURIComponent(cookieValue);
      }
    }

    return null;
  }

  // 🗑️ DELETE COOKIE
  deleteCookie(name: string, path: string = "/", domain?: string): void {
    if (!isPlatformBrowser(this.platformId)) {
      return;
    }

    let cookieString = `${encodeURIComponent(
      name
    )}=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=${path}`;

    if (domain) {
      cookieString += `; domain=${domain}`;
    }

    document.cookie = cookieString;
    console.log(`🗑️ Cookie deleted: ${name}`);
  }

  // 📋 GET ALL COOKIES
  getAllCookies(): { [key: string]: string } {
    if (!isPlatformBrowser(this.platformId)) {
      return {};
    }

    const cookies: { [key: string]: string } = {};

    if (document.cookie) {
      document.cookie.split(";").forEach((cookie) => {
        const [name, value] = cookie.trim().split("=");
        if (name && value) {
          cookies[decodeURIComponent(name)] = decodeURIComponent(value);
        }
      });
    }

    return cookies;
  }

  // 🧹 CLEAR ALL COOKIES
  clearAllCookies(): void {
    const cookies = this.getAllCookies();

    Object.keys(cookies).forEach((cookieName) => {
      this.deleteCookie(cookieName);
    });

    console.log("🧹 All cookies cleared");
  }

  // ✅ CHECK IF COOKIE EXISTS
  hasCookie(name: string): boolean {
    return this.getCookie(name) !== null;
  }

  // 📊 GET COOKIE SIZE
  getCookieSize(name: string): number {
    const value = this.getCookie(name);
    return value ? new Blob([value]).size : 0;
  }

  // 📈 GET TOTAL COOKIES SIZE
  getTotalCookiesSize(): number {
    if (!isPlatformBrowser(this.platformId)) {
      return 0;
    }

    return new Blob([document.cookie]).size;
  }

  // 🔧 Private Helper Methods

  private getExpiresDate(expires: Date | string | number): Date {
    if (expires instanceof Date) {
      return expires;
    }

    if (typeof expires === "string") {
      return new Date(expires);
    }

    if (typeof expires === "number") {
      return new Date(Date.now() + expires * 24 * 60 * 60 * 1000);
    }

    return new Date();
  }
}
```

### **🗄️ Session Storage Service Implementation**

```typescript
// src/app/services/session-storage.service.ts
import { Injectable, Inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";
import { BehaviorSubject, Observable } from "rxjs";

export interface SessionStorageEvent {
  key: string;
  oldValue: any;
  newValue: any;
  timestamp: Date;
}

@Injectable({
  providedIn: "root",
})
export class SessionStorageService {
  private storageEvents$ = new BehaviorSubject<SessionStorageEvent | null>(
    null
  );

  constructor(@Inject(PLATFORM_ID) private platformId: Object) {
    this.initializeStorageListener();
  }

  // 💾 SET ITEM
  setItem<T>(key: string, value: T): void {
    if (!isPlatformBrowser(this.platformId)) {
      console.warn("SessionStorage not available in SSR environment");
      return;
    }

    try {
      const oldValue = this.getItem(key);
      const serializedValue = JSON.stringify(value);

      sessionStorage.setItem(key, serializedValue);

      // Emit storage event
      this.emitStorageEvent(key, oldValue, value);

      console.log(`💾 SessionStorage set: ${key}`);
    } catch (error) {
      console.error(`Failed to set sessionStorage item '${key}':`, error);

      if (error instanceof DOMException && error.code === 22) {
        console.error("SessionStorage quota exceeded");
        this.handleQuotaExceeded(key);
      }
    }
  }

  // 🔍 GET ITEM
  getItem<T>(key: string): T | null {
    if (!isPlatformBrowser(this.platformId)) {
      return null;
    }

    try {
      const value = sessionStorage.getItem(key);

      if (value === null) {
        return null;
      }

      return JSON.parse(value) as T;
    } catch (error) {
      console.error(`Failed to parse sessionStorage item '${key}':`, error);
      return null;
    }
  }

  // 🗑️ REMOVE ITEM
  removeItem(key: string): void {
    if (!isPlatformBrowser(this.platformId)) {
      return;
    }

    const oldValue = this.getItem(key);
    sessionStorage.removeItem(key);

    // Emit storage event
    this.emitStorageEvent(key, oldValue, null);

    console.log(`🗑️ SessionStorage removed: ${key}`);
  }

  // 🧹 CLEAR ALL
  clear(): void {
    if (!isPlatformBrowser(this.platformId)) {
      return;
    }

    // Get all keys before clearing
    const keys = this.getAllKeys();

    sessionStorage.clear();

    // Emit events for all cleared items
    keys.forEach((key) => {
      this.emitStorageEvent(key, this.getItem(key), null);
    });

    console.log("🧹 SessionStorage cleared");
  }

  // ✅ CHECK IF KEY EXISTS
  hasItem(key: string): boolean {
    return this.getItem(key) !== null;
  }

  // 📋 GET ALL KEYS
  getAllKeys(): string[] {
    if (!isPlatformBrowser(this.platformId)) {
      return [];
    }

    const keys: string[] = [];

    for (let i = 0; i < sessionStorage.length; i++) {
      const key = sessionStorage.key(i);
      if (key) {
        keys.push(key);
      }
    }

    return keys;
  }

  // 📦 GET ALL ITEMS
  getAllItems(): { [key: string]: any } {
    const items: { [key: string]: any } = {};
    const keys = this.getAllKeys();

    keys.forEach((key) => {
      items[key] = this.getItem(key);
    });

    return items;
  }

  // 📏 GET STORAGE SIZE
  getStorageSize(): number {
    if (!isPlatformBrowser(this.platformId)) {
      return 0;
    }

    let totalSize = 0;

    for (let key in sessionStorage) {
      if (sessionStorage.hasOwnProperty(key)) {
        totalSize += sessionStorage[key].length + key.length;
      }
    }

    return totalSize;
  }

  // 📊 GET ITEM SIZE
  getItemSize(key: string): number {
    const value = sessionStorage.getItem(key);

    if (!value) {
      return 0;
    }

    return new Blob([value]).size;
  }

  // 📡 LISTEN TO STORAGE EVENTS
  getStorageEvents(): Observable<SessionStorageEvent | null> {
    return this.storageEvents$.asObservable();
  }

  // 💾 BATCH SET ITEMS
  setItems(items: { [key: string]: any }): void {
    Object.entries(items).forEach(([key, value]) => {
      this.setItem(key, value);
    });
  }

  // 🗑️ BATCH REMOVE ITEMS
  removeItems(keys: string[]): void {
    keys.forEach((key) => {
      this.removeItem(key);
    });
  }

  // 🔧 Private Helper Methods

  private initializeStorageListener(): void {
    if (!isPlatformBrowser(this.platformId)) {
      return;
    }

    // Listen to storage events (only for changes from other tabs)
    window.addEventListener("storage", (event) => {
      if (event.storageArea === sessionStorage && event.key) {
        this.emitStorageEvent(
          event.key,
          event.oldValue ? JSON.parse(event.oldValue) : null,
          event.newValue ? JSON.parse(event.newValue) : null
        );
      }
    });
  }

  private emitStorageEvent(key: string, oldValue: any, newValue: any): void {
    this.storageEvents$.next({
      key,
      oldValue,
      newValue,
      timestamp: new Date(),
    });
  }

  private handleQuotaExceeded(key: string): void {
    console.warn(`SessionStorage quota exceeded. Attempting to free space...`);

    // Strategy: Remove oldest items (you could implement LRU here)
    const keys = this.getAllKeys();

    if (keys.length > 0) {
      // Remove first item and try again
      this.removeItem(keys[0]);
      console.log(`Removed ${keys[0]} to free space`);
    }
  }
}
```

---

## 🚀 **Practical Usage Examples**

### **🔐 Authentication with Cookies**

```typescript
// src/app/services/auth.service.ts
import { Injectable } from "@angular/core";
import { BehaviorSubject, Observable } from "rxjs";
import { CookieService } from "./cookie.service";

export interface User {
  id: string;
  email: string;
  name: string;
  role: string;
}

@Injectable({
  providedIn: "root",
})
export class AuthService {
  private currentUser$ = new BehaviorSubject<User | null>(null);
  private readonly TOKEN_KEY = "auth_token";
  private readonly USER_KEY = "user_data";

  constructor(private cookieService: CookieService) {
    this.initializeAuthState();
  }

  // 🔑 LOGIN
  login(email: string, password: string): Observable<User> {
    return new Observable((observer) => {
      // Simulate API call
      setTimeout(() => {
        const user: User = {
          id: "1",
          email,
          name: "John Doe",
          role: "admin",
        };

        const token = "mock-jwt-token-" + Date.now();

        // Store token in secure cookie (HttpOnly on server)
        this.cookieService.setCookie(this.TOKEN_KEY, token, {
          expires: 7, // 7 days
          secure: location.protocol === "https:",
          sameSite: "strict",
        });

        // Store user data in cookie (or use sessionStorage for sensitive data)
        this.cookieService.setCookie(this.USER_KEY, JSON.stringify(user), {
          expires: 7,
          secure: location.protocol === "https:",
          sameSite: "strict",
        });

        this.currentUser$.next(user);
        observer.next(user);
        observer.complete();
      }, 1000);
    });
  }

  // 🚪 LOGOUT
  logout(): void {
    this.cookieService.deleteCookie(this.TOKEN_KEY);
    this.cookieService.deleteCookie(this.USER_KEY);

    this.currentUser$.next(null);

    console.log("🚪 User logged out");
  }

  // 👤 GET CURRENT USER
  getCurrentUser(): Observable<User | null> {
    return this.currentUser$.asObservable();
  }

  // ✅ CHECK IF AUTHENTICATED
  isAuthenticated(): boolean {
    return this.cookieService.hasCookie(this.TOKEN_KEY);
  }

  // 🔑 GET AUTH TOKEN
  getAuthToken(): string | null {
    return this.cookieService.getCookie(this.TOKEN_KEY);
  }

  private initializeAuthState(): void {
    const userDataCookie = this.cookieService.getCookie(this.USER_KEY);

    if (userDataCookie) {
      try {
        const user = JSON.parse(userDataCookie) as User;
        this.currentUser$.next(user);
      } catch (error) {
        console.error("Failed to parse user data from cookie:", error);
        this.logout();
      }
    }
  }
}
```

### **💾 UI State with Session Storage**

```typescript
// src/app/services/ui-state.service.ts
import { Injectable } from "@angular/core";
import { SessionStorageService } from "./session-storage.service";
import { BehaviorSubject, Observable } from "rxjs";

export interface UIState {
  sidebarOpen: boolean;
  theme: "light" | "dark";
  currentTab: string;
  filters: any;
  searchQuery: string;
  sortOrder: "asc" | "desc";
}

@Injectable({
  providedIn: "root",
})
export class UIStateService {
  private readonly STATE_KEY = "ui_state";
  private uiState$ = new BehaviorSubject<UIState>(this.getDefaultState());

  constructor(private sessionStorage: SessionStorageService) {
    this.initializeState();
  }

  // 🎨 GET UI STATE
  getUIState(): Observable<UIState> {
    return this.uiState$.asObservable();
  }

  // ⚙️ UPDATE UI STATE
  updateUIState(updates: Partial<UIState>): void {
    const currentState = this.uiState$.value;
    const newState = { ...currentState, ...updates };

    this.uiState$.next(newState);
    this.sessionStorage.setItem(this.STATE_KEY, newState);

    console.log("🎨 UI state updated:", updates);
  }

  // 📱 SIDEBAR METHODS
  toggleSidebar(): void {
    this.updateUIState({
      sidebarOpen: !this.uiState$.value.sidebarOpen,
    });
  }

  setSidebarOpen(open: boolean): void {
    this.updateUIState({ sidebarOpen: open });
  }

  // 🌙 THEME METHODS
  setTheme(theme: "light" | "dark"): void {
    this.updateUIState({ theme });
  }

  toggleTheme(): void {
    const currentTheme = this.uiState$.value.theme;
    this.setTheme(currentTheme === "light" ? "dark" : "light");
  }

  // 📑 TAB METHODS
  setCurrentTab(tabId: string): void {
    this.updateUIState({ currentTab: tabId });
  }

  // 🔍 SEARCH METHODS
  setSearchQuery(query: string): void {
    this.updateUIState({ searchQuery: query });
  }

  // 🗂️ FILTER METHODS
  updateFilters(filters: any): void {
    this.updateUIState({ filters });
  }

  // 📊 SORT METHODS
  setSortOrder(order: "asc" | "desc"): void {
    this.updateUIState({ sortOrder: order });
  }

  // 🔄 RESET STATE
  resetUIState(): void {
    const defaultState = this.getDefaultState();
    this.uiState$.next(defaultState);
    this.sessionStorage.removeItem(this.STATE_KEY);
    console.log("🔄 UI state reset to defaults");
  }

  private initializeState(): void {
    const savedState = this.sessionStorage.getItem<UIState>(this.STATE_KEY);

    if (savedState) {
      // Merge with defaults to handle new properties
      const mergedState = { ...this.getDefaultState(), ...savedState };
      this.uiState$.next(mergedState);
      console.log("🔄 UI state loaded from session storage");
    }
  }

  private getDefaultState(): UIState {
    return {
      sidebarOpen: true,
      theme: "light",
      currentTab: "dashboard",
      filters: {},
      searchQuery: "",
      sortOrder: "asc",
    };
  }
}
```

---

## 🎯 **Use Case Decision Guide**

### **🍪 Use Cookies When:**

✅ **Authentication tokens** - Need server access  
✅ **User preferences** - Persist across sessions  
✅ **Shopping cart** - Survive browser restart  
✅ **Analytics tracking** - Cross-session tracking  
✅ **Security flags** - HttpOnly, Secure needed

### **🗄️ Use Session Storage When:**

✅ **UI state** - Tab-specific preferences  
✅ **Form data** - Temporary user input  
✅ **Wizard progress** - Multi-step processes  
✅ **Shopping cart** - Single session only  
✅ **Sensitive data** - Not sent to server

### **🏪 Use Local Storage When:**

✅ **User settings** - Persist forever  
✅ **Cached data** - Offline functionality  
✅ **User preferences** - Cross-tab sharing  
✅ **Application state** - Survive restart

---

## 🛡️ **Security Considerations**

### **🔒 Cookie Security**

```typescript
// Secure cookie configuration
const secureCookieOptions: CookieOptions = {
  secure: true, // HTTPS only
  httpOnly: true, // Server-side only (set on server)
  sameSite: "strict", // CSRF protection
  expires: 30, // Explicit expiration
};
```

### **🛡️ Storage Security**

```typescript
// Encrypt sensitive data before storage
import CryptoJS from "crypto-js";

class SecureStorageService {
  private readonly SECRET_KEY = "your-secret-key";

  setSecureItem(key: string, value: any): void {
    const encrypted = CryptoJS.AES.encrypt(
      JSON.stringify(value),
      this.SECRET_KEY
    ).toString();

    sessionStorage.setItem(key, encrypted);
  }

  getSecureItem<T>(key: string): T | null {
    const encrypted = sessionStorage.getItem(key);

    if (!encrypted) return null;

    try {
      const decrypted = CryptoJS.AES.decrypt(
        encrypted,
        this.SECRET_KEY
      ).toString(CryptoJS.enc.Utf8);

      return JSON.parse(decrypted) as T;
    } catch {
      return null;
    }
  }
}
```

---

## 🎉 **Summary: Storage Strategy Mastery**

### **✅ What We've Covered:**

🍪 **Cookie Management** - Complete service with security options  
🗄️ **Session Storage** - Full-featured service with event handling  
🔐 **Authentication** - Secure token management with cookies  
🎨 **UI State** - Temporary state management with session storage  
🛡️ **Security** - Best practices for data protection

### **🎯 Key Takeaways:**

- ✅ **Right Tool for Right Job** - Match storage to use case
- ✅ **Security First** - Always consider data sensitivity
- ✅ **Performance Aware** - Minimize cookie size and requests
- ✅ **SSR Compatible** - Handle server-side rendering properly
- ✅ **Error Handling** - Graceful degradation when storage fails

**You're now equipped to make informed storage decisions!** 🚀
