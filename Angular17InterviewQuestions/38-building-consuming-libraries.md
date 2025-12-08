# 📚 **Building & Consuming Angular Libraries**

## 🎯 **What You'll Learn**

This guide covers the complete process of building custom Angular libraries, from initial setup to publishing and consuming them in applications. Essential knowledge for creating reusable, maintainable components and services across multiple projects.

---

## 📚 **The Basics: Angular Library Fundamentals**

### **🏗️ What Are Angular Libraries?**

Angular libraries are **reusable packages** that can contain:

- **Components** - UI elements and widgets
- **Services** - Business logic and utilities
- **Directives** - DOM manipulation helpers
- **Pipes** - Data transformation tools
- **Models** - TypeScript interfaces and types

**Why Build Libraries?**

- ✅ **Code Reusability** across multiple projects
- ✅ **Consistent Design Systems**
- ✅ **Faster Development** with pre-built components
- ✅ **Better Testing** with isolated, focused packages
- ✅ **Team Collaboration** with shared components

---

## 🏗️ **Creating Your First Angular Library**

### **🚀 Library Generation & Setup**

```bash
# 📦 Generate workspace with library
ng new my-workspace --create-application=false
cd my-workspace

# 🏗️ Generate library
ng generate library ui-components

# 📱 Generate application for testing
ng generate application demo-app

# 🎯 Generate components in library
ng generate component button --project=ui-components
ng generate component card --project=ui-components
ng generate service theme --project=ui-components
```

### **📁 Library Project Structure**

```
projects/ui-components/
├── src/
│   ├── lib/
│   │   ├── button/
│   │   │   ├── button.component.ts
│   │   │   ├── button.component.html
│   │   │   ├── button.component.scss
│   │   │   └── button.component.spec.ts
│   │   ├── card/
│   │   │   └── [card files]
│   │   ├── services/
│   │   │   └── theme.service.ts
│   │   ├── ui-components.module.ts
│   │   └── ui-components.service.ts
│   ├── public-api.ts          // 📋 Public API exports
│   └── test.ts                // 🧪 Test configuration
├── ng-package.json            // 📦 Build configuration
├── package.json               // 📄 Library metadata
├── tsconfig.lib.json          // 📝 TypeScript config
└── tsconfig.spec.json         // 🧪 Test TypeScript config
```

---

## 💎 **Advanced Library Development**

### **🎨 Professional Button Component Example**

```typescript
// projects/ui-components/src/lib/button/button.component.ts
import {
  Component,
  Input,
  Output,
  EventEmitter,
  ChangeDetectionStrategy,
  ViewEncapsulation,
  HostBinding,
  HostListener,
} from "@angular/core";

export type ButtonVariant =
  | "primary"
  | "secondary"
  | "success"
  | "warning"
  | "danger";
export type ButtonSize = "small" | "medium" | "large";

@Component({
  selector: "ui-button",
  template: `
    <button
      [type]="type"
      [disabled]="disabled"
      [class.loading]="loading"
      class="ui-button"
    >
      <!-- Loading Spinner -->
      <span *ngIf="loading" class="ui-button__spinner">
        <svg viewBox="0 0 24 24">
          <circle
            cx="12"
            cy="12"
            r="10"
            stroke="currentColor"
            stroke-width="2"
            fill="none"
            stroke-linecap="round"
            stroke-dasharray="31.416"
            stroke-dashoffset="31.416"
          >
            <animate
              attributeName="stroke-dasharray"
              dur="2s"
              values="0 31.416;15.708 15.708;0 31.416"
              repeatCount="indefinite"
            />
          </circle>
        </svg>
      </span>

      <!-- Icon -->
      <span *ngIf="icon && !loading" class="ui-button__icon">
        <ng-content select="[slot=icon]"></ng-content>
      </span>

      <!-- Content -->
      <span class="ui-button__content">
        <ng-content></ng-content>
      </span>

      <!-- Ripple Effect -->
      <span class="ui-button__ripple" #ripple></span>
    </button>
  `,
  styleUrls: ["./button.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
  encapsulation: ViewEncapsulation.None,
})
export class ButtonComponent {
  @Input() variant: ButtonVariant = "primary";
  @Input() size: ButtonSize = "medium";
  @Input() type: "button" | "submit" | "reset" = "button";
  @Input() disabled = false;
  @Input() loading = false;
  @Input() icon = false;
  @Input() fullWidth = false;
  @Input() ariaLabel?: string;

  @Output() buttonClick = new EventEmitter<MouseEvent>();

  @HostBinding("class") get hostClasses(): string {
    return [
      "ui-button-host",
      `ui-button-host--${this.variant}`,
      `ui-button-host--${this.size}`,
      this.fullWidth ? "ui-button-host--full-width" : "",
      this.disabled ? "ui-button-host--disabled" : "",
      this.loading ? "ui-button-host--loading" : "",
    ]
      .filter(Boolean)
      .join(" ");
  }

  @HostBinding("attr.aria-label") get ariaLabelAttr(): string | null {
    return this.ariaLabel || null;
  }

  @HostListener("click", ["$event"])
  onClick(event: MouseEvent): void {
    if (!this.disabled && !this.loading) {
      this.createRippleEffect(event);
      this.buttonClick.emit(event);
    }
  }

  @HostListener("keydown.enter", ["$event"])
  @HostListener("keydown.space", ["$event"])
  onKeydown(event: KeyboardEvent): void {
    if (!this.disabled && !this.loading) {
      event.preventDefault();
      this.buttonClick.emit(event as any);
    }
  }

  private createRippleEffect(event: MouseEvent): void {
    const button = event.currentTarget as HTMLElement;
    const ripple = button.querySelector(".ui-button__ripple") as HTMLElement;

    if (!ripple) return;

    const rect = button.getBoundingClientRect();
    const size = Math.max(rect.width, rect.height);
    const x = event.clientX - rect.left - size / 2;
    const y = event.clientY - rect.top - size / 2;

    ripple.style.width = ripple.style.height = size + "px";
    ripple.style.left = x + "px";
    ripple.style.top = y + "px";
    ripple.style.transform = "scale(0)";

    // Trigger animation
    requestAnimationFrame(() => {
      ripple.style.transform = "scale(1)";
      ripple.style.opacity = "0.3";

      setTimeout(() => {
        ripple.style.opacity = "0";
        ripple.style.transform = "scale(1.2)";
      }, 300);
    });
  }
}
```

```scss
// projects/ui-components/src/lib/button/button.component.scss
.ui-button-host {
  display: inline-block;
  position: relative;

  &--full-width {
    width: 100%;
  }
}

.ui-button {
  // Base button styles
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 6px;
  font-family: inherit;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  overflow: hidden;
  outline: none;
  user-select: none;

  // Focus styles
  &:focus-visible {
    outline: 2px solid var(--focus-color, #4f46e5);
    outline-offset: 2px;
  }

  // Disabled state
  &:disabled,
  &.loading {
    cursor: not-allowed;
    opacity: 0.6;
  }

  // Loading state
  &.loading {
    pointer-events: none;
  }
}

// Size variants
.ui-button-host--small .ui-button {
  padding: 8px 16px;
  font-size: 14px;
  min-height: 32px;
}

.ui-button-host--medium .ui-button {
  padding: 12px 24px;
  font-size: 16px;
  min-height: 40px;
}

.ui-button-host--large .ui-button {
  padding: 16px 32px;
  font-size: 18px;
  min-height: 48px;
}

// Color variants
.ui-button-host--primary .ui-button {
  background-color: #4f46e5;
  color: white;

  &:hover:not(:disabled) {
    background-color: #4338ca;
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(79, 70, 229, 0.3);
  }

  &:active:not(:disabled) {
    transform: translateY(0);
    box-shadow: 0 2px 6px rgba(79, 70, 229, 0.3);
  }
}

.ui-button-host--secondary .ui-button {
  background-color: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;

  &:hover:not(:disabled) {
    background-color: #e5e7eb;
    border-color: #9ca3af;
  }
}

.ui-button-host--success .ui-button {
  background-color: #10b981;
  color: white;

  &:hover:not(:disabled) {
    background-color: #059669;
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
  }
}

.ui-button-host--warning .ui-button {
  background-color: #f59e0b;
  color: white;

  &:hover:not(:disabled) {
    background-color: #d97706;
    box-shadow: 0 4px 12px rgba(245, 158, 11, 0.3);
  }
}

.ui-button-host--danger .ui-button {
  background-color: #ef4444;
  color: white;

  &:hover:not(:disabled) {
    background-color: #dc2626;
    box-shadow: 0 4px 12px rgba(239, 68, 68, 0.3);
  }
}

// Button content layout
.ui-button__content {
  display: flex;
  align-items: center;
  gap: 8px;
}

.ui-button__icon {
  display: flex;
  align-items: center;

  ::ng-deep svg {
    width: 1em;
    height: 1em;
    fill: currentColor;
  }
}

.ui-button__spinner {
  width: 16px;
  height: 16px;
  margin-right: 8px;

  svg {
    width: 100%;
    height: 100%;
    animation: spin 1s linear infinite;
  }
}

.ui-button__ripple {
  position: absolute;
  background-color: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  pointer-events: none;
  transition: transform 0.3s ease, opacity 0.3s ease;
  transform-origin: center;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

// Full width variant
.ui-button-host--full-width .ui-button {
  width: 100%;
}

// Responsive adjustments
@media (max-width: 768px) {
  .ui-button-host--large .ui-button {
    padding: 14px 28px;
    font-size: 16px;
    min-height: 44px;
  }

  .ui-button-host--medium .ui-button {
    padding: 10px 20px;
    font-size: 15px;
    min-height: 36px;
  }
}
```

### **🎨 Theme Service for Design System**

```typescript
// projects/ui-components/src/lib/services/theme.service.ts
import { Injectable, Inject, InjectionToken, Optional } from "@angular/core";
import { BehaviorSubject, Observable } from "rxjs";

export interface ThemeConfig {
  primary: string;
  secondary: string;
  success: string;
  warning: string;
  danger: string;
  background: string;
  surface: string;
  text: string;
  textSecondary: string;
  border: string;
  focus: string;
}

export interface ThemeVariables {
  light: ThemeConfig;
  dark: ThemeConfig;
}

export const THEME_CONFIG = new InjectionToken<ThemeVariables>("THEME_CONFIG");

export const DEFAULT_THEME: ThemeVariables = {
  light: {
    primary: "#4f46e5",
    secondary: "#6366f1",
    success: "#10b981",
    warning: "#f59e0b",
    danger: "#ef4444",
    background: "#ffffff",
    surface: "#f9fafb",
    text: "#111827",
    textSecondary: "#6b7280",
    border: "#e5e7eb",
    focus: "#4f46e5",
  },
  dark: {
    primary: "#6366f1",
    secondary: "#818cf8",
    success: "#34d399",
    warning: "#fbbf24",
    danger: "#f87171",
    background: "#111827",
    surface: "#1f2937",
    text: "#f9fafb",
    textSecondary: "#9ca3af",
    border: "#374151",
    focus: "#6366f1",
  },
};

@Injectable({
  providedIn: "root",
})
export class ThemeService {
  private currentTheme$ = new BehaviorSubject<"light" | "dark">("light");
  private themeConfig: ThemeVariables;

  constructor(@Optional() @Inject(THEME_CONFIG) config?: ThemeVariables) {
    this.themeConfig = config || DEFAULT_THEME;
    this.initializeTheme();
  }

  // 🎨 GET CURRENT THEME
  getCurrentTheme(): Observable<"light" | "dark"> {
    return this.currentTheme$.asObservable();
  }

  // 🔄 SET THEME
  setTheme(theme: "light" | "dark"): void {
    this.currentTheme$.next(theme);
    this.applyTheme(theme);
    localStorage.setItem("ui-theme", theme);
  }

  // ⚡ TOGGLE THEME
  toggleTheme(): void {
    const current = this.currentTheme$.value;
    const newTheme = current === "light" ? "dark" : "light";
    this.setTheme(newTheme);
  }

  // 🎯 GET THEME VARIABLES
  getThemeVariables(theme?: "light" | "dark"): ThemeConfig {
    const activeTheme = theme || this.currentTheme$.value;
    return this.themeConfig[activeTheme];
  }

  // 🏗️ Private Methods

  private initializeTheme(): void {
    // Get saved theme or detect system preference
    const savedTheme = localStorage.getItem("ui-theme") as "light" | "dark";
    const systemPrefersDark = window.matchMedia(
      "(prefers-color-scheme: dark)"
    ).matches;

    const initialTheme = savedTheme || (systemPrefersDark ? "dark" : "light");
    this.setTheme(initialTheme);

    // Listen for system theme changes
    window
      .matchMedia("(prefers-color-scheme: dark)")
      .addEventListener("change", (e) => {
        if (!localStorage.getItem("ui-theme")) {
          this.setTheme(e.matches ? "dark" : "light");
        }
      });
  }

  private applyTheme(theme: "light" | "dark"): void {
    const config = this.themeConfig[theme];
    const root = document.documentElement;

    // Apply CSS custom properties
    Object.entries(config).forEach(([key, value]) => {
      root.style.setProperty(`--ui-${key}`, value);
    });

    // Add theme class to body
    document.body.className = document.body.className
      .replace(/ui-theme-\w+/g, "")
      .trim();
    document.body.classList.add(`ui-theme-${theme}`);
  }
}
```

---

## 📦 **Building and Publishing Your Library**

### **🏗️ Build Configuration**

```json
// projects/ui-components/ng-package.json
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/ui-components",
  "lib": {
    "entryFile": "src/public-api.ts"
  },
  "allowedNonPeerDependencies": ["tslib"]
}
```

```typescript
// projects/ui-components/src/public-api.ts
/*
 * Public API Surface of ui-components
 */

// 🏗️ Main Module
export * from "./lib/ui-components.module";

// 🎨 Components
export * from "./lib/button/button.component";
export * from "./lib/card/card.component";

// 🔧 Services
export * from "./lib/services/theme.service";

// 🎯 Types & Interfaces
export * from "./lib/models/theme.interface";
export * from "./lib/models/component.types";

// 🎨 Tokens
export * from "./lib/tokens/theme.tokens";
```

```bash
# 📦 Build commands
ng build ui-components

# 🚀 Build for production
ng build ui-components --configuration production

# 📊 Build with stats
ng build ui-components --stats-json

# 👀 Watch mode for development
ng build ui-components --watch
```

### **📤 Publishing to NPM**

```json
// projects/ui-components/package.json
{
  "name": "@company/ui-components",
  "version": "1.0.0",
  "description": "Enterprise UI component library",
  "keywords": ["angular", "components", "ui", "design-system"],
  "repository": {
    "type": "git",
    "url": "https://github.com/company/ui-components.git"
  },
  "homepage": "https://ui-components.company.com",
  "peerDependencies": {
    "@angular/common": "^17.0.0 || ^18.0.0 || ^19.0.0",
    "@angular/core": "^17.0.0 || ^18.0.0 || ^19.0.0"
  },
  "dependencies": {
    "tslib": "^2.3.0"
  },
  "sideEffects": false
}
```

```bash
# 📦 Publishing process
npm run build:ui-components
cd dist/ui-components

# 🔐 Login to npm (if not already)
npm login

# 📤 Publish to npm
npm publish --access public

# 🎯 Publish beta version
npm publish --tag beta
```

---

## 🚀 **Consuming Libraries in Applications**

### **📥 Installing and Importing**

```bash
# 📦 Install your library
npm install @company/ui-components

# 📦 Install peer dependencies if needed
npm install @angular/common @angular/core
```

```typescript
// src/app/app.module.ts
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";

// 📦 Import library module
import { UiComponentsModule } from "@company/ui-components";

import { AppComponent } from "./app.component";

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    UiComponentsModule, // 📦 Add library module
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### **🎯 Using Library Components**

```typescript
// src/app/app.component.ts
import { Component } from "@angular/core";
import { ThemeService } from "@company/ui-components";

@Component({
  selector: "app-root",
  template: `
    <div class="app-container">
      <h1>My Application</h1>

      <!-- 🎨 Use library button component -->
      <ui-button
        variant="primary"
        size="medium"
        [loading]="isLoading"
        (buttonClick)="handleClick()"
      >
        Click Me!
      </ui-button>

      <!-- 🔄 Theme toggle -->
      <ui-button variant="secondary" (buttonClick)="toggleTheme()">
        Toggle Theme
      </ui-button>

      <!-- 📦 Card component -->
      <ui-card>
        <h2 card-title>Welcome</h2>
        <p card-content>This is using our custom library!</p>

        <div card-actions>
          <ui-button variant="success">Save</ui-button>
          <ui-button variant="secondary">Cancel</ui-button>
        </div>
      </ui-card>
    </div>
  `,
  styles: [
    `
      .app-container {
        padding: 20px;
        max-width: 800px;
        margin: 0 auto;
      }
    `,
  ],
})
export class AppComponent {
  isLoading = false;

  constructor(private themeService: ThemeService) {}

  handleClick(): void {
    this.isLoading = true;

    // Simulate API call
    setTimeout(() => {
      this.isLoading = false;
      console.log("Button clicked!");
    }, 2000);
  }

  toggleTheme(): void {
    this.themeService.toggleTheme();
  }
}
```

---

## 🎉 **Summary: Library Development Mastery**

### **✅ What We've Covered:**

🏗️ **Library Creation** - Complete setup and project structure  
🎨 **Professional Components** - Production-ready button with all features  
🔧 **Theme Service** - Complete design system with dark/light modes  
📦 **Build & Distribution** - Publishing to NPM with proper configuration  
🚀 **Library Consumption** - Importing and using in applications

### **🎯 Key Takeaways:**

- ✅ **Reusability First** - Design for multiple use cases
- ✅ **TypeScript Excellence** - Strong typing for better DX
- ✅ **Accessibility** - ARIA support and keyboard navigation
- ✅ **Performance** - OnPush detection and optimizations
- ✅ **Documentation** - Clear API and usage examples
- ✅ **Testing** - Comprehensive unit and integration tests

**You're now ready to build enterprise-grade Angular libraries!** 🚀
