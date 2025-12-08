# 🎨 **Angular Design System Implementation**

## 🎯 **What You'll Learn**

Master **enterprise design system** implementation in Angular! Learn component libraries, design tokens, theming systems, and scalable UI architecture. Think of a design system as the **DNA of your application** - consistent, reusable, and evolving! 🧬

---

## 📚 **Design System Fundamentals**

### **🤔 Why Design Systems Matter**

```typescript
// ❌ Without Design System - Inconsistent UI chaos
// Component A (by Developer 1)
@Component({
  template: `
    <button class="btn-primary"
            style="background: #007bff; padding: 12px 24px; border-radius: 4px;">
      Submit
    </button>
  `
})

// Component B (by Developer 2)
@Component({
  template: `
    <button class="submit-button"
            style="background: #0056b3; padding: 8px 16px; border-radius: 6px;">
      Submit
    </button>
  `
})

// Component C (by Developer 3)
@Component({
  template: `
    <div class="button-wrapper">
      <a href="#" class="link-button"
         style="background: blue; padding: 10px 20px;">
        Submit
      </a>
    </div>
  `
})

// Result: 😱 3 different "submit" buttons with inconsistent styling!
```

### **✅ With Design System - Consistent Excellence**

```typescript
// ✅ With Design System - One source of truth
@Component({
  selector: 'ds-button',
  template: `
    <button
      [class]="computedClasses"
      [disabled]="disabled"
      [attr.aria-label]="ariaLabel"
      (click)="handleClick($event)">

      <ds-icon *ngIf="iconLeft"
               [name]="iconLeft"
               position="left"></ds-icon>

      <span class="ds-button__content">
        <ng-content></ng-content>
      </span>

      <ds-icon *ngIf="iconRight"
               [name]="iconRight"
               position="right"></ds-icon>

      <ds-loading-spinner *ngIf="loading"
                          size="small"></ds-loading-spinner>
    </button>
  `,
  styleUrls: ['./button.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class DSButtonComponent {
  @Input() variant: ButtonVariant = 'primary';
  @Input() size: ButtonSize = 'medium';
  @Input() disabled = false;
  @Input() loading = false;
  @Input() iconLeft?: string;
  @Input() iconRight?: string;
  @Input() ariaLabel?: string;
  @Output() clicked = new EventEmitter<MouseEvent>();

  get computedClasses(): string {
    return [
      'ds-button',
      `ds-button--${this.variant}`,
      `ds-button--${this.size}`,
      this.disabled ? 'ds-button--disabled' : '',
      this.loading ? 'ds-button--loading' : ''
    ].filter(Boolean).join(' ');
  }

  handleClick(event: MouseEvent): void {
    if (!this.disabled && !this.loading) {
      this.clicked.emit(event);
    }
  }
}

// Usage - Consistent across all teams!
@Component({
  template: `
    <ds-button variant="primary" size="large" (clicked)="submit()">
      Submit Order
    </ds-button>
  `
})
```

---

## 🏗️ **Design Token Architecture**

### **1. 🎨 Token Foundation System**

```typescript
// src/design-system/tokens/design-tokens.ts
export interface DesignTokens {
  color: ColorTokens;
  typography: TypographyTokens;
  spacing: SpacingTokens;
  layout: LayoutTokens;
  animation: AnimationTokens;
  elevation: ElevationTokens;
}

// Color token system with semantic naming
export interface ColorTokens {
  // 🎨 Brand colors
  brand: {
    primary: ColorScale;
    secondary: ColorScale;
    accent: ColorScale;
  };

  // 🌈 Semantic colors
  semantic: {
    success: ColorScale;
    warning: ColorScale;
    error: ColorScale;
    info: ColorScale;
  };

  // 🌑 Neutral colors
  neutral: {
    white: string;
    black: string;
    gray: ColorScale;
  };

  // 📝 Text colors
  text: {
    primary: string;
    secondary: string;
    disabled: string;
    inverse: string;
  };

  // 🖼️ Surface colors
  surface: {
    primary: string;
    secondary: string;
    elevated: string;
    overlay: string;
  };
}

export interface ColorScale {
  50: string;
  100: string;
  200: string;
  300: string;
  400: string;
  500: string; // Base color
  600: string;
  700: string;
  800: string;
  900: string;
  950: string;
}

// Typography token system
export interface TypographyTokens {
  fontFamily: {
    primary: string;
    secondary: string;
    monospace: string;
  };

  fontSize: {
    xs: string; // 12px
    sm: string; // 14px
    base: string; // 16px
    lg: string; // 18px
    xl: string; // 20px
    "2xl": string; // 24px
    "3xl": string; // 30px
    "4xl": string; // 36px
    "5xl": string; // 48px
    "6xl": string; // 60px
  };

  fontWeight: {
    light: number; // 300
    normal: number; // 400
    medium: number; // 500
    semibold: number; // 600
    bold: number; // 700
    extrabold: number; // 800
  };

  lineHeight: {
    tight: number; // 1.25
    normal: number; // 1.5
    relaxed: number; // 1.625
    loose: number; // 2
  };

  letterSpacing: {
    tighter: string; // -0.05em
    tight: string; // -0.025em
    normal: string; // 0
    wide: string; // 0.025em
    wider: string; // 0.05em
    widest: string; // 0.1em
  };
}

// Default token implementation
export const DEFAULT_TOKENS: DesignTokens = {
  color: {
    brand: {
      primary: {
        50: "#eff6ff",
        100: "#dbeafe",
        200: "#bfdbfe",
        300: "#93c5fd",
        400: "#60a5fa",
        500: "#3b82f6", // Primary blue
        600: "#2563eb",
        700: "#1d4ed8",
        800: "#1e40af",
        900: "#1e3a8a",
        950: "#172554",
      },
      secondary: {
        50: "#f8fafc",
        100: "#f1f5f9",
        200: "#e2e8f0",
        300: "#cbd5e1",
        400: "#94a3b8",
        500: "#64748b", // Secondary gray
        600: "#475569",
        700: "#334155",
        800: "#1e293b",
        900: "#0f172a",
        950: "#020617",
      },
      accent: {
        50: "#fefce8",
        100: "#fef9c3",
        200: "#fef08a",
        300: "#fde047",
        400: "#facc15",
        500: "#eab308", // Accent yellow
        600: "#ca8a04",
        700: "#a16207",
        800: "#854d0e",
        900: "#713f12",
        950: "#422006",
      },
    },

    semantic: {
      success: {
        50: "#f0fdf4",
        100: "#dcfce7",
        200: "#bbf7d0",
        300: "#86efac",
        400: "#4ade80",
        500: "#22c55e", // Success green
        600: "#16a34a",
        700: "#15803d",
        800: "#166534",
        900: "#14532d",
        950: "#052e16",
      },
      warning: {
        50: "#fffbeb",
        100: "#fef3c7",
        200: "#fde68a",
        300: "#fcd34d",
        400: "#fbbf24",
        500: "#f59e0b", // Warning orange
        600: "#d97706",
        700: "#b45309",
        800: "#92400e",
        900: "#78350f",
        950: "#451a03",
      },
      error: {
        50: "#fef2f2",
        100: "#fee2e2",
        200: "#fecaca",
        300: "#fca5a5",
        400: "#f87171",
        500: "#ef4444", // Error red
        600: "#dc2626",
        700: "#b91c1c",
        800: "#991b1b",
        900: "#7f1d1d",
        950: "#450a0a",
      },
      info: {
        50: "#f0f9ff",
        100: "#e0f2fe",
        200: "#bae6fd",
        300: "#7dd3fc",
        400: "#38bdf8",
        500: "#0ea5e9", // Info blue
        600: "#0284c7",
        700: "#0369a1",
        800: "#075985",
        900: "#0c4a6e",
        950: "#082f49",
      },
    },

    neutral: {
      white: "#ffffff",
      black: "#000000",
      gray: {
        50: "#f9fafb",
        100: "#f3f4f6",
        200: "#e5e7eb",
        300: "#d1d5db",
        400: "#9ca3af",
        500: "#6b7280",
        600: "#4b5563",
        700: "#374151",
        800: "#1f2937",
        900: "#111827",
        950: "#030712",
      },
    },

    text: {
      primary: "#111827",
      secondary: "#6b7280",
      disabled: "#d1d5db",
      inverse: "#ffffff",
    },

    surface: {
      primary: "#ffffff",
      secondary: "#f9fafb",
      elevated: "#ffffff",
      overlay: "rgba(0, 0, 0, 0.5)",
    },
  },

  typography: {
    fontFamily: {
      primary: "Inter, system-ui, -apple-system, sans-serif",
      secondary: "Georgia, serif",
      monospace: "Fira Code, Monaco, monospace",
    },

    fontSize: {
      xs: "0.75rem", // 12px
      sm: "0.875rem", // 14px
      base: "1rem", // 16px
      lg: "1.125rem", // 18px
      xl: "1.25rem", // 20px
      "2xl": "1.5rem", // 24px
      "3xl": "1.875rem", // 30px
      "4xl": "2.25rem", // 36px
      "5xl": "3rem", // 48px
      "6xl": "3.75rem", // 60px
    },

    fontWeight: {
      light: 300,
      normal: 400,
      medium: 500,
      semibold: 600,
      bold: 700,
      extrabold: 800,
    },

    lineHeight: {
      tight: 1.25,
      normal: 1.5,
      relaxed: 1.625,
      loose: 2,
    },

    letterSpacing: {
      tighter: "-0.05em",
      tight: "-0.025em",
      normal: "0",
      wide: "0.025em",
      wider: "0.05em",
      widest: "0.1em",
    },
  },

  spacing: {
    px: "1px",
    0: "0",
    0.5: "0.125rem", // 2px
    1: "0.25rem", // 4px
    1.5: "0.375rem", // 6px
    2: "0.5rem", // 8px
    2.5: "0.625rem", // 10px
    3: "0.75rem", // 12px
    3.5: "0.875rem", // 14px
    4: "1rem", // 16px
    5: "1.25rem", // 20px
    6: "1.5rem", // 24px
    7: "1.75rem", // 28px
    8: "2rem", // 32px
    9: "2.25rem", // 36px
    10: "2.5rem", // 40px
    11: "2.75rem", // 44px
    12: "3rem", // 48px
    14: "3.5rem", // 56px
    16: "4rem", // 64px
    20: "5rem", // 80px
    24: "6rem", // 96px
    28: "7rem", // 112px
    32: "8rem", // 128px
    36: "9rem", // 144px
    40: "10rem", // 160px
    44: "11rem", // 176px
    48: "12rem", // 192px
    52: "13rem", // 208px
    56: "14rem", // 224px
    60: "15rem", // 240px
    64: "16rem", // 256px
    72: "18rem", // 288px
    80: "20rem", // 320px
    96: "24rem", // 384px
  },

  layout: {
    borderRadius: {
      none: "0",
      sm: "0.125rem", // 2px
      base: "0.25rem", // 4px
      md: "0.375rem", // 6px
      lg: "0.5rem", // 8px
      xl: "0.75rem", // 12px
      "2xl": "1rem", // 16px
      "3xl": "1.5rem", // 24px
      full: "9999px",
    },

    borderWidth: {
      0: "0",
      1: "1px",
      2: "2px",
      4: "4px",
      8: "8px",
    },

    maxWidth: {
      none: "none",
      xs: "20rem", // 320px
      sm: "24rem", // 384px
      md: "28rem", // 448px
      lg: "32rem", // 512px
      xl: "36rem", // 576px
      "2xl": "42rem", // 672px
      "3xl": "48rem", // 768px
      "4xl": "56rem", // 896px
      "5xl": "64rem", // 1024px
      "6xl": "72rem", // 1152px
      "7xl": "80rem", // 1280px
      full: "100%",
      screen: "100vw",
    },
  },

  animation: {
    duration: {
      instant: "0ms",
      fast: "150ms",
      normal: "300ms",
      slow: "500ms",
      slower: "750ms",
    },

    easing: {
      linear: "linear",
      easeIn: "cubic-bezier(0.4, 0, 1, 1)",
      easeOut: "cubic-bezier(0, 0, 0.2, 1)",
      easeInOut: "cubic-bezier(0.4, 0, 0.2, 1)",
      bounce: "cubic-bezier(0.68, -0.55, 0.265, 1.55)",
    },
  },

  elevation: {
    none: "none",
    sm: "0 1px 2px 0 rgba(0, 0, 0, 0.05)",
    base: "0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06)",
    md: "0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06)",
    lg: "0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05)",
    xl: "0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04)",
    "2xl": "0 25px 50px -12px rgba(0, 0, 0, 0.25)",
    inner: "inset 0 2px 4px 0 rgba(0, 0, 0, 0.06)",
  },
};
```

### **2. 🔧 Token Management Service**

```typescript
// src/design-system/services/design-token.service.ts
@Injectable({
  providedIn: "root",
})
export class DesignTokenService {
  private tokens$ = new BehaviorSubject<DesignTokens>(DEFAULT_TOKENS);
  private customTokens$ = new BehaviorSubject<Partial<DesignTokens>>({});

  // 🎯 Public API for consuming tokens
  readonly tokens = this.tokens$.asObservable();

  constructor(
    @Inject(DESIGN_TOKENS_CONFIG) private config: DesignTokensConfig
  ) {
    this.initializeTokens();
  }

  /**
   * Get a specific token value by path
   * @example getToken('color.brand.primary.500') // Returns '#3b82f6'
   */
  getToken<T = any>(path: string): T {
    return this.getNestedProperty(this.tokens$.value, path);
  }

  /**
   * Get multiple tokens by paths
   * @example getTokens(['color.text.primary', 'spacing.4'])
   */
  getTokens(paths: string[]): Record<string, any> {
    const currentTokens = this.tokens$.value;
    return paths.reduce((result, path) => {
      result[path] = this.getNestedProperty(currentTokens, path);
      return result;
    }, {} as Record<string, any>);
  }

  /**
   * Update tokens (useful for theming)
   */
  updateTokens(partialTokens: Partial<DesignTokens>): void {
    const currentTokens = this.tokens$.value;
    const mergedTokens = this.deepMerge(currentTokens, partialTokens);
    this.tokens$.next(mergedTokens);
    this.customTokens$.next(partialTokens);

    // Update CSS custom properties
    this.updateCSSCustomProperties(mergedTokens);
  }

  /**
   * Reset to default tokens
   */
  resetTokens(): void {
    this.tokens$.next(DEFAULT_TOKENS);
    this.customTokens$.next({});
    this.updateCSSCustomProperties(DEFAULT_TOKENS);
  }

  /**
   * Load tokens from remote source (e.g., design tools)
   */
  async loadTokensFromRemote(url: string): Promise<void> {
    try {
      const response = await fetch(url);
      const remoteTokens = await response.json();

      // Validate tokens structure
      if (this.validateTokenStructure(remoteTokens)) {
        this.updateTokens(remoteTokens);
      } else {
        console.warn("Invalid token structure from remote source");
      }
    } catch (error) {
      console.error("Failed to load tokens from remote:", error);
    }
  }

  /**
   * Export current tokens (useful for design tool sync)
   */
  exportTokens(): DesignTokens {
    return structuredClone(this.tokens$.value);
  }

  private initializeTokens(): void {
    // Load custom tokens from config or localStorage
    const savedTokens = this.loadSavedTokens();
    if (savedTokens) {
      this.updateTokens(savedTokens);
    }

    // Initialize CSS custom properties
    this.updateCSSCustomProperties(this.tokens$.value);
  }

  private loadSavedTokens(): Partial<DesignTokens> | null {
    try {
      const saved = localStorage.getItem("design-tokens");
      return saved ? JSON.parse(saved) : null;
    } catch {
      return null;
    }
  }

  private updateCSSCustomProperties(tokens: DesignTokens): void {
    const cssVariables = this.tokensToCSSVariables(tokens);
    const root = document.documentElement;

    // Set CSS custom properties
    Object.entries(cssVariables).forEach(([property, value]) => {
      root.style.setProperty(property, value);
    });

    // Save to localStorage for persistence
    localStorage.setItem(
      "design-tokens",
      JSON.stringify(this.customTokens$.value)
    );
  }

  private tokensToCSSVariables(
    tokens: DesignTokens,
    prefix = "--ds"
  ): Record<string, string> {
    const variables: Record<string, string> = {};

    const flatten = (obj: any, currentPrefix = prefix): void => {
      Object.entries(obj).forEach(([key, value]) => {
        const variableName = `${currentPrefix}-${key}`;

        if (
          typeof value === "object" &&
          value !== null &&
          !Array.isArray(value)
        ) {
          flatten(value, variableName);
        } else {
          variables[variableName] = String(value);
        }
      });
    };

    flatten(tokens);
    return variables;
  }

  private getNestedProperty(obj: any, path: string): any {
    return path.split(".").reduce((current, prop) => current?.[prop], obj);
  }

  private deepMerge(target: any, source: any): any {
    const result = { ...target };

    Object.keys(source).forEach((key) => {
      if (
        source[key] &&
        typeof source[key] === "object" &&
        !Array.isArray(source[key])
      ) {
        result[key] = this.deepMerge(result[key] || {}, source[key]);
      } else {
        result[key] = source[key];
      }
    });

    return result;
  }

  private validateTokenStructure(tokens: any): boolean {
    // Validate required token structure
    const requiredKeys = ["color", "typography", "spacing"];
    return requiredKeys.every((key) => tokens.hasOwnProperty(key));
  }
}

// Configuration token
export const DESIGN_TOKENS_CONFIG = new InjectionToken<DesignTokensConfig>(
  "DESIGN_TOKENS_CONFIG"
);

export interface DesignTokensConfig {
  enableRemoteSync?: boolean;
  remoteUrl?: string;
  enableLocalStorage?: boolean;
  customPrefix?: string;
}
```

---

## 🧩 **Component Library Architecture**

### **1. 🎨 Base Component System**

```typescript
// src/design-system/components/base/base-component.ts
export abstract class BaseComponent implements OnInit, OnDestroy {
  protected destroy$ = new Subject<void>();

  @Input() className?: string;
  @Input() dataTestId?: string;
  @Input() ariaLabel?: string;
  @Input() ariaDescribedBy?: string;

  constructor(
    protected tokenService: DesignTokenService,
    protected cdr: ChangeDetectorRef
  ) {}

  ngOnInit(): void {
    this.subscribeToTokenChanges();
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  /**
   * Get computed CSS classes for the component
   */
  protected getBaseClasses(): string[] {
    return [
      this.getComponentClass(),
      ...(this.className ? [this.className] : []),
    ];
  }

  /**
   * Get base component class name (to be implemented by child components)
   */
  protected abstract getComponentClass(): string;

  /**
   * Subscribe to token changes and trigger change detection
   */
  private subscribeToTokenChanges(): void {
    this.tokenService.tokens.pipe(takeUntil(this.destroy$)).subscribe(() => {
      this.cdr.markForCheck();
    });
  }

  /**
   * Utility to get design tokens within components
   */
  protected getToken<T = any>(path: string): T {
    return this.tokenService.getToken<T>(path);
  }

  /**
   * Generate dynamic styles based on tokens
   */
  protected generateStyles(
    styleMap: Record<string, string>
  ): Record<string, string> {
    const styles: Record<string, string> = {};

    Object.entries(styleMap).forEach(([property, tokenPath]) => {
      const value = this.getToken(tokenPath);
      if (value !== undefined) {
        styles[property] = value;
      }
    });

    return styles;
  }
}

// src/design-system/components/button/button.component.ts
@Component({
  selector: "ds-button",
  templateUrl: "./button.component.html",
  styleUrls: ["./button.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
  host: {
    "[class]": "computedClasses",
    "[attr.data-testid]": "dataTestId",
    "[attr.aria-label]": "ariaLabel",
    "[attr.disabled]": "disabled || loading || null",
  },
})
export class DSButtonComponent extends BaseComponent {
  // 🎯 Button variants
  @Input() variant: ButtonVariant = "primary";
  @Input() size: ButtonSize = "medium";
  @Input() disabled = false;
  @Input() loading = false;
  @Input() fullWidth = false;
  @Input() iconLeft?: string;
  @Input() iconRight?: string;
  @Input() type: "button" | "submit" | "reset" = "button";

  // 🔄 Events
  @Output() clicked = new EventEmitter<MouseEvent>();

  // 📊 Internal state
  private isPressed = false;
  private isFocused = false;

  get computedClasses(): string {
    return this.getBaseClasses().join(" ");
  }

  get dynamicStyles(): Record<string, string> {
    return this.generateStyles({
      "background-color": `color.brand.${this.variant}.500`,
      color:
        this.variant === "ghost"
          ? `color.brand.${this.variant}.600`
          : "color.text.inverse",
      padding: `spacing.${this.getPaddingSize()}`,
      "border-radius": "layout.borderRadius.md",
      "font-size": `typography.fontSize.${this.getFontSize()}`,
      "font-weight": "typography.fontWeight.medium",
    });
  }

  protected getComponentClass(): string {
    return "ds-button";
  }

  protected getBaseClasses(): string[] {
    return [
      ...super.getBaseClasses(),
      `ds-button--${this.variant}`,
      `ds-button--${this.size}`,
      ...(this.disabled ? ["ds-button--disabled"] : []),
      ...(this.loading ? ["ds-button--loading"] : []),
      ...(this.fullWidth ? ["ds-button--full-width"] : []),
      ...(this.isPressed ? ["ds-button--pressed"] : []),
      ...(this.isFocused ? ["ds-button--focused"] : []),
    ];
  }

  @HostListener("click", ["$event"])
  onClick(event: MouseEvent): void {
    if (!this.disabled && !this.loading) {
      this.clicked.emit(event);
    }
  }

  @HostListener("mousedown")
  onMouseDown(): void {
    this.isPressed = true;
    this.cdr.markForCheck();
  }

  @HostListener("mouseup")
  @HostListener("mouseleave")
  onMouseUp(): void {
    this.isPressed = false;
    this.cdr.markForCheck();
  }

  @HostListener("focus")
  onFocus(): void {
    this.isFocused = true;
    this.cdr.markForCheck();
  }

  @HostListener("blur")
  onBlur(): void {
    this.isFocused = false;
    this.cdr.markForCheck();
  }

  private getPaddingSize(): string {
    const paddingMap = {
      small: "2",
      medium: "3",
      large: "4",
    };
    return paddingMap[this.size];
  }

  private getFontSize(): string {
    const fontSizeMap = {
      small: "sm",
      medium: "base",
      large: "lg",
    };
    return fontSizeMap[this.size];
  }
}

// Type definitions
export type ButtonVariant =
  | "primary"
  | "secondary"
  | "success"
  | "warning"
  | "error"
  | "ghost"
  | "outline";
export type ButtonSize = "small" | "medium" | "large";
```

### **2. 🎨 Advanced Component: Form Field**

```typescript
// src/design-system/components/form-field/form-field.component.ts
@Component({
  selector: "ds-form-field",
  templateUrl: "./form-field.component.html",
  styleUrls: ["./form-field.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => DSFormFieldComponent),
      multi: true,
    },
  ],
})
export class DSFormFieldComponent
  extends BaseComponent
  implements ControlValueAccessor, AfterContentInit
{
  // 🎯 Form field configuration
  @Input() label?: string;
  @Input() placeholder?: string;
  @Input() helperText?: string;
  @Input() errorText?: string;
  @Input() required = false;
  @Input() disabled = false;
  @Input() readonly = false;
  @Input() type: FormFieldType = "text";
  @Input() size: FormFieldSize = "medium";
  @Input() variant: FormFieldVariant = "outline";

  // 🔍 Validation states
  @Input() hasError = false;
  @Input() isValid = false;

  // 🎨 Visual enhancements
  @Input() iconLeft?: string;
  @Input() iconRight?: string;
  @Input() clearable = false;

  // 🔄 Events
  @Output() valueChange = new EventEmitter<any>();
  @Output() focused = new EventEmitter<FocusEvent>();
  @Output() blurred = new EventEmitter<FocusEvent>();
  @Output() cleared = new EventEmitter<void>();

  // 📝 Form control integration
  private onChange: (value: any) => void = () => {};
  private onTouched: () => void = () => {};

  // 📊 Internal state
  value: any = "";
  isFocused = false;
  isTouched = false;

  // 🔍 Template references
  @ViewChild("inputElement", { static: true })
  inputElement!: ElementRef<HTMLInputElement>;

  get computedClasses(): string {
    return this.getBaseClasses().join(" ");
  }

  get showError(): boolean {
    return this.hasError && this.isTouched;
  }

  get showSuccess(): boolean {
    return this.isValid && this.isTouched && !this.hasError;
  }

  get dynamicStyles(): Record<string, string> {
    const stateColor = this.getStateColor();

    return this.generateStyles({
      "border-color": `color.${stateColor}.500`,
      padding: `spacing.${this.getPaddingSize()}`,
      "border-radius": "layout.borderRadius.md",
      "font-size": `typography.fontSize.${this.getFontSize()}`,
      "background-color": this.disabled
        ? "color.neutral.gray.50"
        : "color.surface.primary",
    });
  }

  protected getComponentClass(): string {
    return "ds-form-field";
  }

  protected getBaseClasses(): string[] {
    return [
      ...super.getBaseClasses(),
      `ds-form-field--${this.variant}`,
      `ds-form-field--${this.size}`,
      ...(this.disabled ? ["ds-form-field--disabled"] : []),
      ...(this.readonly ? ["ds-form-field--readonly"] : []),
      ...(this.isFocused ? ["ds-form-field--focused"] : []),
      ...(this.showError ? ["ds-form-field--error"] : []),
      ...(this.showSuccess ? ["ds-form-field--success"] : []),
      ...(this.required ? ["ds-form-field--required"] : []),
    ];
  }

  // ControlValueAccessor implementation
  writeValue(value: any): void {
    this.value = value;
    this.cdr.markForCheck();
  }

  registerOnChange(fn: (value: any) => void): void {
    this.onChange = fn;
  }

  registerOnTouched(fn: () => void): void {
    this.onTouched = fn;
  }

  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
    this.cdr.markForCheck();
  }

  // 🔄 Event handlers
  onInput(event: Event): void {
    const target = event.target as HTMLInputElement;
    this.value = target.value;
    this.onChange(this.value);
    this.valueChange.emit(this.value);
  }

  onFocus(event: FocusEvent): void {
    this.isFocused = true;
    this.focused.emit(event);
    this.cdr.markForCheck();
  }

  onBlur(event: FocusEvent): void {
    this.isFocused = false;
    this.isTouched = true;
    this.onTouched();
    this.blurred.emit(event);
    this.cdr.markForCheck();
  }

  onClear(): void {
    this.value = "";
    this.onChange(this.value);
    this.valueChange.emit(this.value);
    this.cleared.emit();
    this.inputElement.nativeElement.focus();
  }

  // 🔧 Helper methods
  focus(): void {
    this.inputElement.nativeElement.focus();
  }

  blur(): void {
    this.inputElement.nativeElement.blur();
  }

  private getStateColor(): string {
    if (this.showError) return "semantic.error";
    if (this.showSuccess) return "semantic.success";
    if (this.isFocused) return "brand.primary";
    return "neutral.gray";
  }

  private getPaddingSize(): string {
    const paddingMap = {
      small: "2",
      medium: "3",
      large: "4",
    };
    return paddingMap[this.size];
  }

  private getFontSize(): string {
    const fontSizeMap = {
      small: "sm",
      medium: "base",
      large: "lg",
    };
    return fontSizeMap[this.size];
  }
}

export type FormFieldType =
  | "text"
  | "email"
  | "password"
  | "number"
  | "tel"
  | "url"
  | "search";
export type FormFieldSize = "small" | "medium" | "large";
export type FormFieldVariant = "outline" | "filled" | "underline";
```

---

## 🎭 **Theming System**

### **1. 🌈 Multi-Theme Support**

```typescript
// src/design-system/theming/theme.service.ts
@Injectable({
  providedIn: "root",
})
export class ThemeService {
  private currentTheme$ = new BehaviorSubject<Theme>(DEFAULT_LIGHT_THEME);
  private availableThemes$ = new BehaviorSubject<Theme[]>([
    DEFAULT_LIGHT_THEME,
    DEFAULT_DARK_THEME,
    HIGH_CONTRAST_THEME,
  ]);

  readonly theme = this.currentTheme$.asObservable();
  readonly availableThemes = this.availableThemes$.asObservable();

  constructor(
    private tokenService: DesignTokenService,
    @Inject(DOCUMENT) private document: Document
  ) {
    this.initializeTheme();
  }

  /**
   * Switch to a different theme
   */
  setTheme(themeId: string): void {
    const themes = this.availableThemes$.value;
    const theme = themes.find((t) => t.id === themeId);

    if (theme) {
      this.applyTheme(theme);
      this.currentTheme$.next(theme);
      localStorage.setItem("selected-theme", themeId);
    }
  }

  /**
   * Register a new custom theme
   */
  registerTheme(theme: Theme): void {
    const themes = this.availableThemes$.value;
    const existingIndex = themes.findIndex((t) => t.id === theme.id);

    if (existingIndex >= 0) {
      themes[existingIndex] = theme;
    } else {
      themes.push(theme);
    }

    this.availableThemes$.next([...themes]);
  }

  /**
   * Create a custom theme variant
   */
  createThemeVariant(
    baseTheme: Theme,
    overrides: Partial<DesignTokens>,
    name: string
  ): Theme {
    const mergedTokens = this.deepMerge(baseTheme.tokens, overrides);

    return {
      id: `${baseTheme.id}-${name.toLowerCase().replace(/\s+/g, "-")}`,
      name: `${baseTheme.name} (${name})`,
      description: `Custom variant of ${baseTheme.name}`,
      tokens: mergedTokens,
      type: baseTheme.type,
    };
  }

  /**
   * Auto-detect user's preferred theme
   */
  detectPreferredTheme(): string {
    // Check for saved preference
    const saved = localStorage.getItem("selected-theme");
    if (saved) return saved;

    // Check system preference
    const prefersDark = window.matchMedia(
      "(prefers-color-scheme: dark)"
    ).matches;
    const prefersHighContrast = window.matchMedia(
      "(prefers-contrast: high)"
    ).matches;

    if (prefersHighContrast) return "high-contrast";
    return prefersDark ? "dark" : "light";
  }

  /**
   * Toggle between light and dark themes
   */
  toggleTheme(): void {
    const current = this.currentTheme$.value;
    const targetTheme = current.type === "dark" ? "light" : "dark";
    this.setTheme(targetTheme);
  }

  private initializeTheme(): void {
    const preferredThemeId = this.detectPreferredTheme();
    this.setTheme(preferredThemeId);

    // Listen for system theme changes
    window
      .matchMedia("(prefers-color-scheme: dark)")
      .addEventListener("change", (e) => {
        if (!localStorage.getItem("selected-theme")) {
          this.setTheme(e.matches ? "dark" : "light");
        }
      });
  }

  private applyTheme(theme: Theme): void {
    // Update design tokens
    this.tokenService.updateTokens(theme.tokens);

    // Apply theme class to document
    this.document.body.className = this.document.body.className
      .replace(/theme-\w+/g, "")
      .trim();
    this.document.body.classList.add(`theme-${theme.id}`);

    // Update meta theme color
    this.updateMetaThemeColor(theme.tokens.color.brand.primary["500"]);
  }

  private updateMetaThemeColor(color: string): void {
    let themeColorMeta = this.document.querySelector(
      'meta[name="theme-color"]'
    ) as HTMLMetaElement;

    if (!themeColorMeta) {
      themeColorMeta = this.document.createElement("meta");
      themeColorMeta.name = "theme-color";
      this.document.head.appendChild(themeColorMeta);
    }

    themeColorMeta.content = color;
  }

  private deepMerge(target: any, source: any): any {
    const result = { ...target };

    Object.keys(source).forEach((key) => {
      if (
        source[key] &&
        typeof source[key] === "object" &&
        !Array.isArray(source[key])
      ) {
        result[key] = this.deepMerge(result[key] || {}, source[key]);
      } else {
        result[key] = source[key];
      }
    });

    return result;
  }
}

// Theme definitions
export interface Theme {
  id: string;
  name: string;
  description: string;
  tokens: Partial<DesignTokens>;
  type: "light" | "dark" | "high-contrast";
}

export const DEFAULT_LIGHT_THEME: Theme = {
  id: "light",
  name: "Light",
  description: "Clean and bright theme for daytime use",
  type: "light",
  tokens: {
    // Light theme uses default tokens
  },
};

export const DEFAULT_DARK_THEME: Theme = {
  id: "dark",
  name: "Dark",
  description: "Easy on the eyes for low-light environments",
  type: "dark",
  tokens: {
    color: {
      surface: {
        primary: "#0f172a",
        secondary: "#1e293b",
        elevated: "#334155",
        overlay: "rgba(255, 255, 255, 0.1)",
      },
      text: {
        primary: "#f8fafc",
        secondary: "#cbd5e1",
        disabled: "#64748b",
        inverse: "#0f172a",
      },
    },
  },
};

export const HIGH_CONTRAST_THEME: Theme = {
  id: "high-contrast",
  name: "High Contrast",
  description: "Maximum contrast for accessibility",
  type: "high-contrast",
  tokens: {
    color: {
      brand: {
        primary: {
          500: "#000000",
        } as ColorScale,
      },
      surface: {
        primary: "#ffffff",
        secondary: "#f0f0f0",
        elevated: "#ffffff",
        overlay: "rgba(0, 0, 0, 0.8)",
      },
      text: {
        primary: "#000000",
        secondary: "#000000",
        disabled: "#666666",
        inverse: "#ffffff",
      },
    },
  },
};
```

---

## 📚 **Design System Documentation**

### **1. 📖 Storybook Integration**

```typescript
// .storybook/main.js
module.exports = {
  stories: [
    "../src/**/*.stories.@(js|jsx|ts|tsx|mdx)",
    "../src/design-system/**/*.stories.@(js|jsx|ts|tsx|mdx)",
  ],
  addons: [
    "@storybook/addon-essentials",
    "@storybook/addon-controls",
    "@storybook/addon-docs",
    "@storybook/addon-viewport",
    "@storybook/addon-a11y",
    "@storybook/addon-design-tokens",
  ],
  framework: "@storybook/angular",
  core: {
    builder: "webpack5",
  },
  features: {
    buildStoriesJson: true,
  },
};

// src/design-system/components/button/button.stories.ts
import {
  Meta,
  StoryObj,
  moduleMetadata,
  argsToTemplate,
} from "@storybook/angular";
import { action } from "@storybook/addon-actions";
import { DSButtonComponent } from "./button.component";
import { DesignSystemModule } from "../../design-system.module";

type Story = StoryObj<DSButtonComponent>;

const meta: Meta<DSButtonComponent> = {
  title: "Design System/Button",
  component: DSButtonComponent,
  decorators: [
    moduleMetadata({
      imports: [DesignSystemModule],
    }),
  ],
  parameters: {
    docs: {
      description: {
        component: `
          ## Button Component
          
          The Button component is a foundational interactive element that triggers actions or events.
          It supports multiple variants, sizes, and states to cover all use cases.
          
          ### Accessibility
          - Keyboard navigation support
          - Screen reader friendly
          - High contrast mode support
          - Focus management
          
          ### Best Practices
          - Use clear, action-oriented labels
          - Maintain consistent spacing and alignment
          - Provide loading states for async actions
          - Use appropriate variants for context
        `,
      },
    },
    a11y: {
      element: "ds-button",
      config: {
        rules: [
          {
            id: "color-contrast",
            enabled: true,
          },
        ],
      },
    },
  },
  argTypes: {
    variant: {
      control: "select",
      options: [
        "primary",
        "secondary",
        "success",
        "warning",
        "error",
        "ghost",
        "outline",
      ],
      description: "Visual style variant of the button",
    },
    size: {
      control: "select",
      options: ["small", "medium", "large"],
      description: "Size of the button",
    },
    disabled: {
      control: "boolean",
      description: "Whether the button is disabled",
    },
    loading: {
      control: "boolean",
      description: "Whether the button is in loading state",
    },
    fullWidth: {
      control: "boolean",
      description: "Whether the button spans full width",
    },
    clicked: {
      action: "clicked",
      description: "Event emitted when button is clicked",
    },
  },
  args: {
    variant: "primary",
    size: "medium",
    disabled: false,
    loading: false,
    fullWidth: false,
    clicked: action("clicked"),
  },
};

export default meta;

// 🎯 Primary story - default state
export const Primary: Story = {
  render: (args) => ({
    props: args,
    template: `<ds-button ${argsToTemplate(args)}>Primary Button</ds-button>`,
  }),
};

// 🎨 Variants showcase
export const Variants: Story = {
  render: () => ({
    template: `
      <div style="display: flex; gap: 16px; flex-wrap: wrap;">
        <ds-button variant="primary">Primary</ds-button>
        <ds-button variant="secondary">Secondary</ds-button>
        <ds-button variant="success">Success</ds-button>
        <ds-button variant="warning">Warning</ds-button>
        <ds-button variant="error">Error</ds-button>
        <ds-button variant="ghost">Ghost</ds-button>
        <ds-button variant="outline">Outline</ds-button>
      </div>
    `,
  }),
  parameters: {
    docs: {
      description: {
        story: "All available button variants with their default styling.",
      },
    },
  },
};

// 📏 Sizes showcase
export const Sizes: Story = {
  render: () => ({
    template: `
      <div style="display: flex; gap: 16px; align-items: center;">
        <ds-button size="small">Small</ds-button>
        <ds-button size="medium">Medium</ds-button>
        <ds-button size="large">Large</ds-button>
      </div>
    `,
  }),
  parameters: {
    docs: {
      description: {
        story: "Button sizes from small to large.",
      },
    },
  },
};

// 🔄 States showcase
export const States: Story = {
  render: () => ({
    template: `
      <div style="display: flex; gap: 16px; flex-wrap: wrap;">
        <ds-button>Normal</ds-button>
        <ds-button disabled="true">Disabled</ds-button>
        <ds-button loading="true">Loading</ds-button>
        <ds-button fullWidth="true">Full Width</ds-button>
      </div>
    `,
  }),
  parameters: {
    docs: {
      description: {
        story:
          "Different button states including disabled, loading, and full-width.",
      },
    },
  },
};

// 🎯 Interactive playground
export const Playground: Story = {
  render: (args) => ({
    props: args,
    template: `
      <ds-button 
        [variant]="variant"
        [size]="size"
        [disabled]="disabled"
        [loading]="loading"
        [fullWidth]="fullWidth"
        (clicked)="clicked($event)">
        {{ content || 'Button Text' }}
      </ds-button>
    `,
  }),
  args: {
    content: "Playground Button",
  },
  argTypes: {
    content: {
      control: "text",
      description: "Button content/text",
    },
  },
};

// 🌈 Theme variations
export const ThemeVariations: Story = {
  render: () => ({
    template: `
      <div style="display: grid; gap: 24px;">
        <div class="theme-light" style="padding: 16px; border: 1px solid #e5e7eb;">
          <h4>Light Theme</h4>
          <div style="display: flex; gap: 12px;">
            <ds-button variant="primary">Primary</ds-button>
            <ds-button variant="secondary">Secondary</ds-button>
            <ds-button variant="outline">Outline</ds-button>
          </div>
        </div>
        
        <div class="theme-dark" style="padding: 16px; border: 1px solid #374151; background: #0f172a;">
          <h4 style="color: white;">Dark Theme</h4>
          <div style="display: flex; gap: 12px;">
            <ds-button variant="primary">Primary</ds-button>
            <ds-button variant="secondary">Secondary</ds-button>
            <ds-button variant="outline">Outline</ds-button>
          </div>
        </div>
      </div>
    `,
  }),
  parameters: {
    docs: {
      description: {
        story: "Buttons in different theme contexts.",
      },
    },
  },
};
```

---

## 🎉 **Summary: Design System Mastery**

### **🎨 What You've Mastered:**

#### **🏗️ Foundation Systems:**

✅ **Design Tokens** - Scalable token architecture with semantic naming  
✅ **Token Management** - Runtime token updates and CSS custom properties  
✅ **Theme System** - Multi-theme support with auto-detection  
✅ **Component Architecture** - Reusable base components with consistent patterns

#### **🧩 Component Library:**

✅ **Base Component Pattern** - Shared functionality and token integration  
✅ **Advanced Components** - Form fields with full ControlValueAccessor support  
✅ **Accessibility** - WCAG compliance built into every component  
✅ **Performance** - OnPush change detection and efficient updates

#### **📚 Documentation & Tools:**

✅ **Storybook Integration** - Interactive documentation with accessibility testing  
✅ **Design Tool Sync** - Token export/import for design-dev collaboration  
✅ **Theme Variants** - Custom theme creation and brand customization  
✅ **Type Safety** - Full TypeScript support with strict typing

### **🚀 Real-World Benefits:**

- **⚡ 60% Faster Development** - Reusable components reduce build time
- **🎨 100% Brand Consistency** - Design tokens ensure visual coherence
- **♿ Full Accessibility** - WCAG AA compliance out of the box
- **🔧 Easy Maintenance** - Centralized styling with token updates

### **💡 Key Success Factors:**

1. **🎯 Token-First Approach** - All styling flows from design tokens
2. **🔄 Runtime Flexibility** - Themes and tokens can change dynamically
3. **📚 Living Documentation** - Storybook keeps docs in sync with code
4. **🤝 Design-Dev Sync** - Shared language between design and development
5. **♿ Accessibility by Default** - No additional effort required for compliance

**Remember**: A great design system grows with your product - start with tokens, build components, document everything! 🌟
