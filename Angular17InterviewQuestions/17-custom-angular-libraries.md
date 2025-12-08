# 📚 Building Custom Angular Libraries: Complete Guide

## 🎯 **Question Overview**

_"How do you create and distribute custom Angular libraries?"_

## 🔍 **Understanding Angular Libraries**

Angular libraries are **reusable packages** that encapsulate functionality, components, services, and utilities that can be shared across multiple applications. They enable **code reuse**, **maintainability**, and **consistency** across projects.

Modern Angular libraries support **standalone components**, **tree-shaking**, **secondary entry points**, and **flexible distribution** strategies! 📦

## 🏗️ **Library Creation & Structure**

### **1. 🚀 Generate Library Project**

```bash
# Create new Angular workspace with library
ng new my-workspace --create-application=false
cd my-workspace

# Generate library
ng generate library ui-components

# Generate additional libraries
ng generate library data-access
ng generate library utilities

# Generate demo application
ng generate application demo-app

# Project structure
my-workspace/
├── projects/
│   ├── ui-components/          # Main UI library
│   │   ├── src/
│   │   │   ├── lib/
│   │   │   ├── public-api.ts   # Public API exports
│   │   │   └── test.ts         # Test setup
│   │   ├── ng-package.json     # Library build config
│   │   ├── package.json        # Library metadata
│   │   └── README.md
│   ├── data-access/            # Data services library
│   └── utilities/              # Utility functions library
├── demo-app/                   # Demo application
├── angular.json
└── package.json
```

### **2. 📁 Library Architecture**

```typescript
// projects/ui-components/src/lib/ structure
ui-components/
├── components/
│   ├── button/
│   │   ├── button.component.ts
│   │   ├── button.component.spec.ts
│   │   └── index.ts
│   ├── modal/
│   │   ├── modal.component.ts
│   │   ├── modal.service.ts
│   │   ├── modal.interfaces.ts
│   │   └── index.ts
│   ├── form-controls/
│   │   ├── input/
│   │   ├── select/
│   │   ├── checkbox/
│   │   └── index.ts
│   └── index.ts
├── directives/
│   ├── highlight/
│   ├── tooltip/
│   └── index.ts
├── pipes/
│   ├── format-currency/
│   ├── truncate-text/
│   └── index.ts
├── services/
│   ├── theme/
│   ├── notification/
│   └── index.ts
├── utils/
│   ├── validators/
│   ├── helpers/
│   └── index.ts
├── types/
│   ├── interfaces.ts
│   └── types.ts
└── index.ts
```

### **3. 📦 Package Configuration**

```json
// projects/ui-components/package.json
{
  "name": "@my-org/ui-components",
  "version": "1.0.0",
  "description": "Reusable UI components for Angular applications",
  "keywords": ["angular", "ui", "components", "design-system"],
  "author": "Your Name <your.email@example.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/my-org/ui-components.git"
  },
  "bugs": {
    "url": "https://github.com/my-org/ui-components/issues"
  },
  "homepage": "https://my-org.github.io/ui-components",
  "peerDependencies": {
    "@angular/common": "^17.0.0 || ^18.0.0 || ^19.0.0",
    "@angular/core": "^17.0.0 || ^18.0.0 || ^19.0.0",
    "rxjs": "^7.0.0"
  },
  "dependencies": {},
  "sideEffects": false
}
```

```json
// projects/ui-components/ng-package.json
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/ui-components",
  "lib": {
    "entryFile": "src/public-api.ts",
    "umdModuleIds": {
      "lodash": "lodash",
      "@my-org/utilities": "my-org-utilities"
    }
  },
  "assets": ["src/assets/**/*", "README.md", "CHANGELOG.md"],
  "allowedNonPeerDependencies": ["tslib"]
}
```

## 🧩 **Library Components Development**

### **1. 🎨 Standalone Component Library**

```typescript
// projects/ui-components/src/lib/components/button/button.component.ts
import {
  Component,
  Input,
  Output,
  EventEmitter,
  computed,
  signal,
  ChangeDetectionStrategy,
} from "@angular/core";
import { CommonModule } from "@angular/common";

export type ButtonVariant =
  | "primary"
  | "secondary"
  | "outline"
  | "ghost"
  | "danger"
  | "success";
export type ButtonSize = "small" | "medium" | "large";

export interface ButtonConfig {
  variant?: ButtonVariant;
  size?: ButtonSize;
  disabled?: boolean;
  loading?: boolean;
  fullWidth?: boolean;
  rounded?: boolean;
}

@Component({
  selector: "ui-button",
  standalone: true,
  imports: [CommonModule],
  template: `
    <button
      [type]="type"
      [class]="buttonClasses()"
      [disabled]="disabled || loading()"
      (click)="handleClick($event)"
      (focus)="handleFocus($event)"
      (blur)="handleBlur($event)"
    >
      <!-- Loading spinner -->
      <span *ngIf="loading()" class="button-spinner" aria-hidden="true"> </span>

      <!-- Start icon -->
      <span
        *ngIf="startIcon && !loading()"
        class="button-icon button-icon-start"
        [innerHTML]="startIcon"
        aria-hidden="true"
      >
      </span>

      <!-- Content -->
      <span class="button-content">
        <ng-content></ng-content>
      </span>

      <!-- End icon -->
      <span
        *ngIf="endIcon && !loading()"
        class="button-icon button-icon-end"
        [innerHTML]="endIcon"
        aria-hidden="true"
      >
      </span>
    </button>
  `,
  styleUrls: ["./button.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
  host: {
    "[class.ui-button-host]": "true",
  },
})
export class ButtonComponent {
  @Input() type: "button" | "submit" | "reset" = "button";
  @Input() variant: ButtonVariant = "primary";
  @Input() size: ButtonSize = "medium";
  @Input() disabled = false;
  @Input() fullWidth = false;
  @Input() rounded = false;
  @Input() startIcon?: string;
  @Input() endIcon?: string;
  @Input() ariaLabel?: string;

  @Output() buttonClick = new EventEmitter<MouseEvent>();
  @Output() buttonFocus = new EventEmitter<FocusEvent>();
  @Output() buttonBlur = new EventEmitter<FocusEvent>();

  // Loading state
  loading = signal(false);

  // Computed CSS classes
  buttonClasses = computed(() => {
    const classes = [
      "ui-button",
      `ui-button--${this.variant}`,
      `ui-button--${this.size}`,
    ];

    if (this.disabled) classes.push("ui-button--disabled");
    if (this.fullWidth) classes.push("ui-button--full-width");
    if (this.rounded) classes.push("ui-button--rounded");
    if (this.loading()) classes.push("ui-button--loading");

    return classes.join(" ");
  });

  handleClick(event: MouseEvent): void {
    if (!this.disabled && !this.loading()) {
      this.buttonClick.emit(event);
    }
  }

  handleFocus(event: FocusEvent): void {
    this.buttonFocus.emit(event);
  }

  handleBlur(event: FocusEvent): void {
    this.buttonBlur.emit(event);
  }

  // Public API for loading state
  setLoading(loading: boolean): void {
    this.loading.set(loading);
  }

  async performAsyncAction<T>(action: () => Promise<T>): Promise<T> {
    this.setLoading(true);
    try {
      return await action();
    } finally {
      this.setLoading(false);
    }
  }
}
```

```scss
// projects/ui-components/src/lib/components/button/button.component.scss
.ui-button {
  // Base styles
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border: none;
  border-radius: 0.375rem;
  font-weight: 600;
  text-decoration: none;
  transition: all 0.2s ease-in-out;
  cursor: pointer;
  position: relative;
  white-space: nowrap;
  user-select: none;

  // Focus styles
  &:focus-visible {
    outline: 2px solid var(--ui-primary-500);
    outline-offset: 2px;
  }

  // Disabled state
  &--disabled {
    opacity: 0.6;
    cursor: not-allowed;
    pointer-events: none;
  }

  // Loading state
  &--loading {
    color: transparent;

    .button-spinner {
      position: absolute;
      width: 1rem;
      height: 1rem;
      border: 2px solid currentColor;
      border-top: 2px solid transparent;
      border-radius: 50%;
      animation: spin 1s linear infinite;
    }
  }

  // Sizes
  &--small {
    padding: 0.5rem 0.75rem;
    font-size: 0.875rem;
    line-height: 1.25rem;
  }

  &--medium {
    padding: 0.625rem 1rem;
    font-size: 1rem;
    line-height: 1.5rem;
  }

  &--large {
    padding: 0.75rem 1.5rem;
    font-size: 1.125rem;
    line-height: 1.75rem;
  }

  // Variants
  &--primary {
    background-color: var(--ui-primary-600);
    color: var(--ui-white);

    &:hover:not(.ui-button--disabled) {
      background-color: var(--ui-primary-700);
    }

    &:active:not(.ui-button--disabled) {
      background-color: var(--ui-primary-800);
    }
  }

  &--secondary {
    background-color: var(--ui-gray-600);
    color: var(--ui-white);

    &:hover:not(.ui-button--disabled) {
      background-color: var(--ui-gray-700);
    }

    &:active:not(.ui-button--disabled) {
      background-color: var(--ui-gray-800);
    }
  }

  &--outline {
    background-color: transparent;
    color: var(--ui-primary-600);
    border: 1px solid var(--ui-primary-600);

    &:hover:not(.ui-button--disabled) {
      background-color: var(--ui-primary-50);
    }

    &:active:not(.ui-button--disabled) {
      background-color: var(--ui-primary-100);
    }
  }

  &--ghost {
    background-color: transparent;
    color: var(--ui-primary-600);

    &:hover:not(.ui-button--disabled) {
      background-color: var(--ui-primary-50);
    }

    &:active:not(.ui-button--disabled) {
      background-color: var(--ui-primary-100);
    }
  }

  &--danger {
    background-color: var(--ui-red-600);
    color: var(--ui-white);

    &:hover:not(.ui-button--disabled) {
      background-color: var(--ui-red-700);
    }

    &:active:not(.ui-button--disabled) {
      background-color: var(--ui-red-800);
    }
  }

  &--success {
    background-color: var(--ui-green-600);
    color: var(--ui-white);

    &:hover:not(.ui-button--disabled) {
      background-color: var(--ui-green-700);
    }

    &:active:not(.ui-button--disabled) {
      background-color: var(--ui-green-800);
    }
  }

  // Modifiers
  &--full-width {
    width: 100%;
  }

  &--rounded {
    border-radius: 9999px;
  }

  // Icons
  .button-icon {
    display: flex;
    align-items: center;

    &-start {
      margin-right: -0.125rem;
    }

    &-end {
      margin-left: -0.125rem;
    }
  }

  .button-content {
    display: flex;
    align-items: center;
  }
}

// Animations
@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

// CSS Custom Properties (Design tokens)
:root {
  --ui-primary-50: #eff6ff;
  --ui-primary-500: #3b82f6;
  --ui-primary-600: #2563eb;
  --ui-primary-700: #1d4ed8;
  --ui-primary-800: #1e40af;

  --ui-gray-600: #4b5563;
  --ui-gray-700: #374151;
  --ui-gray-800: #1f2937;

  --ui-red-600: #dc2626;
  --ui-red-700: #b91c1c;
  --ui-red-800: #991b1b;

  --ui-green-600: #16a34a;
  --ui-green-700: #15803d;
  --ui-green-800: #166534;

  --ui-white: #ffffff;
}
```

### **2. 🔧 Service Library**

```typescript
// projects/ui-components/src/lib/services/theme/theme.service.ts
import {
  Injectable,
  signal,
  computed,
  effect,
  Inject,
  DOCUMENT,
} from "@angular/core";

export type Theme = "light" | "dark" | "auto";
export type ColorScheme = "light" | "dark";

export interface ThemeConfig {
  defaultTheme: Theme;
  storageKey: string;
  enableSystemPreference: boolean;
}

const DEFAULT_THEME_CONFIG: ThemeConfig = {
  defaultTheme: "auto",
  storageKey: "ui-theme",
  enableSystemPreference: true,
};

@Injectable({
  providedIn: "root",
})
export class ThemeService {
  private readonly _currentTheme = signal<Theme>("auto");
  private readonly _systemPreference = signal<ColorScheme>("light");
  private readonly config: ThemeConfig;

  // Public readonly signals
  readonly currentTheme = this._currentTheme.asReadonly();
  readonly systemPreference = this._systemPreference.asReadonly();

  // Computed color scheme
  readonly colorScheme = computed<ColorScheme>(() => {
    const theme = this._currentTheme();
    if (theme === "auto") {
      return this._systemPreference();
    }
    return theme as ColorScheme;
  });

  // Computed boolean helpers
  readonly isDarkMode = computed(() => this.colorScheme() === "dark");
  readonly isLightMode = computed(() => this.colorScheme() === "light");
  readonly isAutoMode = computed(() => this._currentTheme() === "auto");

  constructor(
    @Inject(DOCUMENT) private document: Document,
    @Inject("THEME_CONFIG") config: Partial<ThemeConfig> = {}
  ) {
    this.config = { ...DEFAULT_THEME_CONFIG, ...config };
    this.initialize();
  }

  private initialize(): void {
    // Load saved theme or use default
    const savedTheme = this.loadFromStorage();
    this._currentTheme.set(savedTheme || this.config.defaultTheme);

    // Setup system preference detection
    this.setupSystemPreferenceDetection();

    // Setup theme application effect
    this.setupThemeEffect();
  }

  private setupSystemPreferenceDetection(): void {
    if (!this.config.enableSystemPreference) return;

    const mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");

    // Set initial value
    this._systemPreference.set(mediaQuery.matches ? "dark" : "light");

    // Listen for changes
    mediaQuery.addEventListener("change", (e) => {
      this._systemPreference.set(e.matches ? "dark" : "light");
    });
  }

  private setupThemeEffect(): void {
    effect(() => {
      const colorScheme = this.colorScheme();
      this.applyTheme(colorScheme);
    });
  }

  private applyTheme(colorScheme: ColorScheme): void {
    const documentElement = this.document.documentElement;

    // Remove existing theme classes
    documentElement.classList.remove("theme-light", "theme-dark");

    // Add new theme class
    documentElement.classList.add(`theme-${colorScheme}`);

    // Set data attribute for CSS selectors
    documentElement.setAttribute("data-theme", colorScheme);

    // Set color-scheme meta tag for native elements
    const colorSchemeMetaTag = this.document.querySelector(
      'meta[name="color-scheme"]'
    );
    if (colorSchemeMetaTag) {
      colorSchemeMetaTag.setAttribute("content", colorScheme);
    } else {
      const metaTag = this.document.createElement("meta");
      metaTag.name = "color-scheme";
      metaTag.content = colorScheme;
      this.document.head.appendChild(metaTag);
    }
  }

  // Public API
  setTheme(theme: Theme): void {
    this._currentTheme.set(theme);
    this.saveToStorage(theme);
  }

  toggleTheme(): void {
    const current = this._currentTheme();
    let newTheme: Theme;

    if (current === "light") {
      newTheme = "dark";
    } else if (current === "dark") {
      newTheme = "auto";
    } else {
      newTheme = "light";
    }

    this.setTheme(newTheme);
  }

  toggleColorScheme(): void {
    const currentColorScheme = this.colorScheme();
    this.setTheme(currentColorScheme === "light" ? "dark" : "light");
  }

  resetTheme(): void {
    this.setTheme(this.config.defaultTheme);
  }

  // Storage methods
  private loadFromStorage(): Theme | null {
    try {
      const saved = localStorage.getItem(this.config.storageKey);
      return (saved as Theme) || null;
    } catch {
      return null;
    }
  }

  private saveToStorage(theme: Theme): void {
    try {
      localStorage.setItem(this.config.storageKey, theme);
    } catch {
      // Storage not available
    }
  }

  // Utility methods
  getAvailableThemes(): Theme[] {
    return ["light", "dark", "auto"];
  }

  getThemeDisplayName(theme: Theme): string {
    const names: Record<Theme, string> = {
      light: "Light",
      dark: "Dark",
      auto: "System",
    };
    return names[theme];
  }

  // Observable for backward compatibility
  get theme$() {
    return new Observable((observer) => {
      const cleanup = effect(() => {
        observer.next(this._currentTheme());
      });
      return () => cleanup.destroy();
    });
  }

  get colorScheme$() {
    return new Observable((observer) => {
      const cleanup = effect(() => {
        observer.next(this.colorScheme());
      });
      return () => cleanup.destroy();
    });
  }
}

// Provider function for configuration
export function provideTheme(config: Partial<ThemeConfig> = {}) {
  return [{ provide: "THEME_CONFIG", useValue: config }, ThemeService];
}
```

### **3. 🎭 Directive Library**

```typescript
// projects/ui-components/src/lib/directives/tooltip/tooltip.directive.ts
import {
  Directive,
  Input,
  ElementRef,
  Renderer2,
  HostListener,
  OnDestroy,
  Injectable,
  signal,
} from "@angular/core";

export type TooltipPosition = "top" | "bottom" | "left" | "right";

export interface TooltipConfig {
  position: TooltipPosition;
  showDelay: number;
  hideDelay: number;
  maxWidth: string;
  className?: string;
}

@Injectable({
  providedIn: "root",
})
export class TooltipService {
  private tooltipElement: HTMLElement | null = null;
  private showTimeout?: number;
  private hideTimeout?: number;

  constructor(private renderer: Renderer2) {}

  show(
    targetElement: HTMLElement,
    content: string,
    config: TooltipConfig
  ): void {
    // Clear existing timeouts
    this.clearTimeouts();

    this.showTimeout = window.setTimeout(() => {
      this.createTooltip(targetElement, content, config);
    }, config.showDelay);
  }

  hide(config: TooltipConfig): void {
    this.clearTimeouts();

    this.hideTimeout = window.setTimeout(() => {
      this.removeTooltip();
    }, config.hideDelay);
  }

  private createTooltip(
    targetElement: HTMLElement,
    content: string,
    config: TooltipConfig
  ): void {
    // Remove existing tooltip
    this.removeTooltip();

    // Create tooltip element
    this.tooltipElement = this.renderer.createElement("div");
    this.renderer.addClass(this.tooltipElement, "ui-tooltip");
    this.renderer.addClass(
      this.tooltipElement,
      `ui-tooltip--${config.position}`
    );

    if (config.className) {
      this.renderer.addClass(this.tooltipElement, config.className);
    }

    // Set content
    this.renderer.setProperty(this.tooltipElement, "textContent", content);

    // Set styles
    this.renderer.setStyle(this.tooltipElement, "maxWidth", config.maxWidth);
    this.renderer.setStyle(this.tooltipElement, "position", "absolute");
    this.renderer.setStyle(this.tooltipElement, "zIndex", "10000");
    this.renderer.setStyle(this.tooltipElement, "pointerEvents", "none");

    // Append to document body
    this.renderer.appendChild(document.body, this.tooltipElement);

    // Position tooltip
    this.positionTooltip(targetElement, config.position);

    // Add show class for animation
    this.renderer.addClass(this.tooltipElement, "ui-tooltip--show");
  }

  private positionTooltip(
    targetElement: HTMLElement,
    position: TooltipPosition
  ): void {
    if (!this.tooltipElement) return;

    const targetRect = targetElement.getBoundingClientRect();
    const tooltipRect = this.tooltipElement.getBoundingClientRect();
    const scrollX = window.pageXOffset;
    const scrollY = window.pageYOffset;

    let top = 0;
    let left = 0;

    switch (position) {
      case "top":
        top = targetRect.top + scrollY - tooltipRect.height - 8;
        left =
          targetRect.left +
          scrollX +
          (targetRect.width - tooltipRect.width) / 2;
        break;
      case "bottom":
        top = targetRect.bottom + scrollY + 8;
        left =
          targetRect.left +
          scrollX +
          (targetRect.width - tooltipRect.width) / 2;
        break;
      case "left":
        top =
          targetRect.top +
          scrollY +
          (targetRect.height - tooltipRect.height) / 2;
        left = targetRect.left + scrollX - tooltipRect.width - 8;
        break;
      case "right":
        top =
          targetRect.top +
          scrollY +
          (targetRect.height - tooltipRect.height) / 2;
        left = targetRect.right + scrollX + 8;
        break;
    }

    // Ensure tooltip stays within viewport
    const maxLeft = window.innerWidth - tooltipRect.width - 8;
    const maxTop = window.innerHeight - tooltipRect.height - 8;

    left = Math.max(8, Math.min(left, maxLeft));
    top = Math.max(8, Math.min(top, maxTop));

    this.renderer.setStyle(this.tooltipElement, "top", `${top}px`);
    this.renderer.setStyle(this.tooltipElement, "left", `${left}px`);
  }

  private removeTooltip(): void {
    if (this.tooltipElement) {
      this.renderer.removeChild(document.body, this.tooltipElement);
      this.tooltipElement = null;
    }
  }

  private clearTimeouts(): void {
    if (this.showTimeout) {
      clearTimeout(this.showTimeout);
      this.showTimeout = undefined;
    }
    if (this.hideTimeout) {
      clearTimeout(this.hideTimeout);
      this.hideTimeout = undefined;
    }
  }

  destroy(): void {
    this.clearTimeouts();
    this.removeTooltip();
  }
}

@Directive({
  selector: "[uiTooltip]",
  standalone: true,
})
export class TooltipDirective implements OnDestroy {
  @Input("uiTooltip") content = "";
  @Input() tooltipPosition: TooltipPosition = "top";
  @Input() tooltipShowDelay = 500;
  @Input() tooltipHideDelay = 200;
  @Input() tooltipMaxWidth = "200px";
  @Input() tooltipClass?: string;

  private get config(): TooltipConfig {
    return {
      position: this.tooltipPosition,
      showDelay: this.tooltipShowDelay,
      hideDelay: this.tooltipHideDelay,
      maxWidth: this.tooltipMaxWidth,
      className: this.tooltipClass,
    };
  }

  constructor(
    private elementRef: ElementRef<HTMLElement>,
    private tooltipService: TooltipService
  ) {}

  @HostListener("mouseenter")
  onMouseEnter(): void {
    if (this.content) {
      this.tooltipService.show(
        this.elementRef.nativeElement,
        this.content,
        this.config
      );
    }
  }

  @HostListener("mouseleave")
  onMouseLeave(): void {
    this.tooltipService.hide(this.config);
  }

  @HostListener("focus")
  onFocus(): void {
    if (this.content) {
      this.tooltipService.show(
        this.elementRef.nativeElement,
        this.content,
        this.config
      );
    }
  }

  @HostListener("blur")
  onBlur(): void {
    this.tooltipService.hide(this.config);
  }

  ngOnDestroy(): void {
    this.tooltipService.destroy();
  }
}
```

```scss
// projects/ui-components/src/lib/directives/tooltip/tooltip.styles.scss
.ui-tooltip {
  background: var(--ui-gray-900);
  color: var(--ui-white);
  padding: 0.5rem 0.75rem;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  line-height: 1.25rem;
  font-weight: 500;
  word-wrap: break-word;
  transform: scale(0.8);
  opacity: 0;
  transition: all 0.15s ease-in-out;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);

  &--show {
    transform: scale(1);
    opacity: 1;
  }

  // Positioning arrows
  &::after {
    content: "";
    position: absolute;
    width: 0;
    height: 0;
    border: 4px solid transparent;
  }

  &--top::after {
    top: 100%;
    left: 50%;
    transform: translateX(-50%);
    border-top-color: var(--ui-gray-900);
  }

  &--bottom::after {
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    border-bottom-color: var(--ui-gray-900);
  }

  &--left::after {
    left: 100%;
    top: 50%;
    transform: translateY(-50%);
    border-left-color: var(--ui-gray-900);
  }

  &--right::after {
    right: 100%;
    top: 50%;
    transform: translateY(-50%);
    border-right-color: var(--ui-gray-900);
  }
}

// Dark theme
[data-theme="dark"] .ui-tooltip {
  background: var(--ui-gray-700);
  color: var(--ui-gray-100);

  &--top::after {
    border-top-color: var(--ui-gray-700);
  }

  &--bottom::after {
    border-bottom-color: var(--ui-gray-700);
  }

  &--left::after {
    border-left-color: var(--ui-gray-700);
  }

  &--right::after {
    border-right-color: var(--ui-gray-700);
  }
}
```

## 📋 **Public API Definition**

### **1. 🔗 Public API Exports**

```typescript
// projects/ui-components/src/public-api.ts - Main entry point
/*
 * Public API Surface of @my-org/ui-components
 */

// Components
export * from "./lib/components/button/button.component";
export * from "./lib/components/modal/modal.component";
export * from "./lib/components/form-controls";

// Directives
export * from "./lib/directives/tooltip/tooltip.directive";
export * from "./lib/directives/highlight/highlight.directive";

// Services
export * from "./lib/services/theme/theme.service";
export * from "./lib/services/notification/notification.service";

// Pipes
export * from "./lib/pipes/format-currency/format-currency.pipe";
export * from "./lib/pipes/truncate-text/truncate-text.pipe";

// Types & Interfaces
export * from "./lib/types/interfaces";
export * from "./lib/types/types";

// Utils
export * from "./lib/utils/validators";
export * from "./lib/utils/helpers";

// Tokens & Providers
export * from "./lib/providers/theme.providers";
export * from "./lib/providers/notification.providers";

// Library version
export const UI_COMPONENTS_VERSION = "1.0.0";
```

### **2. 🎯 Secondary Entry Points**

```json
// projects/ui-components/ng-package.json - Secondary entry points
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/ui-components",
  "lib": {
    "entryFile": "src/public-api.ts"
  },
  "assets": ["README.md", "CHANGELOG.md"],
  "allowedNonPeerDependencies": ["tslib"]
}
```

```typescript
// projects/ui-components/components/public-api.ts
export * from "../src/lib/components/button/button.component";
export * from "../src/lib/components/modal/modal.component";
export * from "../src/lib/components/form-controls";
```

```typescript
// projects/ui-components/services/public-api.ts
export * from "../src/lib/services/theme/theme.service";
export * from "../src/lib/services/notification/notification.service";
```

```typescript
// projects/ui-components/directives/public-api.ts
export * from "../src/lib/directives/tooltip/tooltip.directive";
export * from "../src/lib/directives/highlight/highlight.directive";
```

```json
// package.json - Entry point configuration
{
  "name": "@my-org/ui-components",
  "exports": {
    ".": {
      "import": "./fesm2022/my-org-ui-components.mjs",
      "require": "./bundles/my-org-ui-components.umd.js",
      "typings": "./index.d.ts"
    },
    "./components": {
      "import": "./fesm2022/my-org-ui-components-components.mjs",
      "require": "./bundles/my-org-ui-components-components.umd.js",
      "typings": "./components/index.d.ts"
    },
    "./services": {
      "import": "./fesm2022/my-org-ui-components-services.mjs",
      "require": "./bundles/my-org-ui-components-services.umd.js",
      "typings": "./services/index.d.ts"
    },
    "./directives": {
      "import": "./fesm2022/my-org-ui-components-directives.mjs",
      "require": "./bundles/my-org-ui-components-directives.umd.js",
      "typings": "./directives/index.d.ts"
    }
  }
}
```

## 🔧 **Build & Distribution**

### **1. 🏗️ Build Configuration**

```bash
# Build library
ng build ui-components

# Build with production optimizations
ng build ui-components --configuration production

# Watch mode for development
ng build ui-components --watch

# Build all libraries
npm run build:libs
```

```json
// angular.json - Build configuration
{
  "projects": {
    "ui-components": {
      "projectType": "library",
      "root": "projects/ui-components",
      "sourceRoot": "projects/ui-components/src",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:ng-packagr",
          "options": {
            "project": "projects/ui-components/ng-package.json"
          },
          "configurations": {
            "production": {
              "tsConfig": "projects/ui-components/tsconfig.lib.prod.json"
            },
            "development": {
              "tsConfig": "projects/ui-components/tsconfig.lib.json"
            }
          }
        },
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "tsConfig": "projects/ui-components/tsconfig.spec.json",
            "polyfills": ["zone.js", "zone.js/testing"]
          }
        }
      }
    }
  }
}
```

### **2. 📦 Publishing Strategy**

```json
// package.json - Publishing scripts
{
  "scripts": {
    "build:libs": "ng build ui-components && ng build data-access && ng build utilities",
    "test:libs": "ng test ui-components && ng test data-access && ng test utilities",
    "pack:libs": "npm run build:libs && cd dist/ui-components && npm pack",
    "publish:beta": "npm run build:libs && cd dist/ui-components && npm publish --tag beta",
    "publish:latest": "npm run build:libs && cd dist/ui-components && npm publish",
    "version:patch": "npm version patch && npm run publish:latest",
    "version:minor": "npm version minor && npm run publish:latest",
    "version:major": "npm version major && npm run publish:latest"
  }
}
```

### **3. 🔄 CI/CD Pipeline**

```yaml
# .github/workflows/library.yml
name: Library CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
      - uses: actions/checkout@v3

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm run test:libs

      - name: Build libraries
        run: npm run build:libs

      - name: Upload coverage
        uses: codecov/codecov-action@v3

  publish:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "20.x"
          cache: "npm"
          registry-url: "https://registry.npmjs.org"

      - name: Install dependencies
        run: npm ci

      - name: Build libraries
        run: npm run build:libs

      - name: Semantic Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

## 🧪 **Testing Strategy**

### **1. ✅ Unit Testing**

```typescript
// projects/ui-components/src/lib/components/button/button.component.spec.ts
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { ButtonComponent } from "./button.component";
import { DebugElement } from "@angular/core";
import { By } from "@angular/platform-browser";

describe("ButtonComponent", () => {
  let component: ButtonComponent;
  let fixture: ComponentFixture<ButtonComponent>;
  let buttonElement: DebugElement;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ButtonComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(ButtonComponent);
    component = fixture.componentInstance;
    buttonElement = fixture.debugElement.query(By.css("button"));
  });

  describe("Component Initialization", () => {
    it("should create", () => {
      expect(component).toBeTruthy();
    });

    it("should have default properties", () => {
      expect(component.type).toBe("button");
      expect(component.variant).toBe("primary");
      expect(component.size).toBe("medium");
      expect(component.disabled).toBeFalsy();
      expect(component.fullWidth).toBeFalsy();
      expect(component.rounded).toBeFalsy();
    });
  });

  describe("CSS Classes", () => {
    it("should apply base CSS classes", () => {
      fixture.detectChanges();

      const classes = component.buttonClasses();
      expect(classes).toContain("ui-button");
      expect(classes).toContain("ui-button--primary");
      expect(classes).toContain("ui-button--medium");
    });

    it("should apply variant classes", () => {
      component.variant = "secondary";
      fixture.detectChanges();

      expect(component.buttonClasses()).toContain("ui-button--secondary");
    });

    it("should apply size classes", () => {
      component.size = "large";
      fixture.detectChanges();

      expect(component.buttonClasses()).toContain("ui-button--large");
    });

    it("should apply modifier classes", () => {
      component.disabled = true;
      component.fullWidth = true;
      component.rounded = true;
      fixture.detectChanges();

      const classes = component.buttonClasses();
      expect(classes).toContain("ui-button--disabled");
      expect(classes).toContain("ui-button--full-width");
      expect(classes).toContain("ui-button--rounded");
    });
  });

  describe("Event Handling", () => {
    it("should emit click event when clicked", () => {
      spyOn(component.buttonClick, "emit");

      buttonElement.nativeElement.click();

      expect(component.buttonClick.emit).toHaveBeenCalled();
    });

    it("should not emit click when disabled", () => {
      component.disabled = true;
      spyOn(component.buttonClick, "emit");

      buttonElement.nativeElement.click();

      expect(component.buttonClick.emit).not.toHaveBeenCalled();
    });

    it("should not emit click when loading", () => {
      component.setLoading(true);
      spyOn(component.buttonClick, "emit");

      buttonElement.nativeElement.click();

      expect(component.buttonClick.emit).not.toHaveBeenCalled();
    });

    it("should emit focus and blur events", () => {
      spyOn(component.buttonFocus, "emit");
      spyOn(component.buttonBlur, "emit");

      buttonElement.nativeElement.focus();
      buttonElement.nativeElement.blur();

      expect(component.buttonFocus.emit).toHaveBeenCalled();
      expect(component.buttonBlur.emit).toHaveBeenCalled();
    });
  });

  describe("Loading State", () => {
    it("should show loading spinner when loading", () => {
      component.setLoading(true);
      fixture.detectChanges();

      const spinner = fixture.debugElement.query(By.css(".button-spinner"));
      expect(spinner).toBeTruthy();
    });

    it("should hide icons when loading", () => {
      component.startIcon = "🔍";
      component.setLoading(true);
      fixture.detectChanges();

      const icon = fixture.debugElement.query(By.css(".button-icon"));
      expect(icon).toBeFalsy();
    });

    it("should perform async action with loading state", async () => {
      const mockAction = jest.fn().mockResolvedValue("result");

      expect(component.loading()).toBeFalsy();

      const promise = component.performAsyncAction(mockAction);

      expect(component.loading()).toBeTruthy();

      const result = await promise;

      expect(component.loading()).toBeFalsy();
      expect(result).toBe("result");
      expect(mockAction).toHaveBeenCalled();
    });
  });

  describe("Accessibility", () => {
    it("should have correct button type", () => {
      component.type = "submit";
      fixture.detectChanges();

      expect(buttonElement.nativeElement.type).toBe("submit");
    });

    it("should be disabled when disabled prop is true", () => {
      component.disabled = true;
      fixture.detectChanges();

      expect(buttonElement.nativeElement.disabled).toBeTruthy();
    });

    it("should have aria-label when provided", () => {
      component.ariaLabel = "Custom button label";
      fixture.detectChanges();

      expect(buttonElement.nativeElement.getAttribute("aria-label")).toBe(
        "Custom button label"
      );
    });
  });
});
```

### **2. 🎯 Integration Testing**

```typescript
// projects/ui-components/src/lib/services/theme/theme.service.spec.ts
import { TestBed } from "@angular/core/testing";
import { DOCUMENT } from "@angular/common";
import { ThemeService } from "./theme.service";

describe("ThemeService", () => {
  let service: ThemeService;
  let document: Document;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        ThemeService,
        { provide: "THEME_CONFIG", useValue: { defaultTheme: "light" } },
      ],
    });

    service = TestBed.inject(ThemeService);
    document = TestBed.inject(DOCUMENT);
  });

  it("should be created", () => {
    expect(service).toBeTruthy();
  });

  it("should initialize with default theme", () => {
    expect(service.currentTheme()).toBe("light");
  });

  it("should apply theme classes to document", () => {
    service.setTheme("dark");

    expect(
      document.documentElement.classList.contains("theme-dark")
    ).toBeTruthy();
    expect(document.documentElement.getAttribute("data-theme")).toBe("dark");
  });

  it("should toggle theme correctly", () => {
    service.setTheme("light");
    service.toggleTheme();

    expect(service.currentTheme()).toBe("dark");

    service.toggleTheme();
    expect(service.currentTheme()).toBe("auto");

    service.toggleTheme();
    expect(service.currentTheme()).toBe("light");
  });

  it("should save theme to localStorage", () => {
    spyOn(localStorage, "setItem");

    service.setTheme("dark");

    expect(localStorage.setItem).toHaveBeenCalledWith("ui-theme", "dark");
  });
});
```

## 📊 **Angular Version Compatibility**

| Feature                    | Angular 16   | Angular 17  | Angular 19  |
| -------------------------- | ------------ | ----------- | ----------- |
| **Standalone Libraries**   | Supported    | Enhanced    | Optimized   |
| **ng-packagr**             | v16          | v17         | v19         |
| **Secondary Entry Points** | Full support | Improved    | Native      |
| **Tree Shaking**           | Good         | Better      | Excellent   |
| **Bundle Size**            | Baseline     | 10% smaller | 20% smaller |

## 🎯 **Key Takeaways**

### **🏗️ Library Development Best Practices:**

1. **Design for Reusability** - Create flexible, configurable components
2. **Follow Angular Guidelines** - Use official patterns and conventions
3. **Implement Tree Shaking** - Use secondary entry points and pure functions
4. **Provide Type Safety** - Export comprehensive TypeScript interfaces
5. **Document Thoroughly** - Include examples and API documentation

### **📦 Distribution Strategy:**

1. **Semantic Versioning** - Follow semver for predictable updates
2. **Multiple Entry Points** - Allow selective imports for smaller bundles
3. **Peer Dependencies** - Avoid version conflicts with consuming apps
4. **CI/CD Pipeline** - Automate testing, building, and publishing
5. **NPM Registry** - Use scoped packages for organization

### **🧪 Testing Approach:**

1. **Unit Tests** - Test individual components and services
2. **Integration Tests** - Test component interactions
3. **E2E Tests** - Test complete user workflows
4. **Visual Regression** - Catch UI changes automatically
5. **Performance Tests** - Monitor bundle size and runtime performance

Creating robust Angular libraries requires careful planning, thorough testing, and thoughtful API design - but the benefits of code reuse and maintainability are immense! 🚀
