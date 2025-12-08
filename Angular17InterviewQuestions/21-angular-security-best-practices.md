# 🔐 Angular Security Best Practices: Complete Guide

## 🎯 **Question Overview**

_"What are the security best practices for Angular applications, including XSS prevention, CSRF protection, and secure authentication?"_

## 🛡️ **Understanding Angular Security**

Angular security involves **multiple layers of protection** against common web vulnerabilities like **XSS**, **CSRF**, **injection attacks**, and **unauthorized access**. Angular provides **built-in security features** while requiring **developer best practices**.

Modern Angular security leverages **Content Security Policy**, **sanitization**, **secure authentication**, and **proper data validation**! 🔒

## 🧱 **XSS Prevention Strategies**

### **1. 🧼 Content Sanitization**

```typescript
// src/app/services/sanitization.service.ts - Advanced Sanitization Service
import { Injectable, inject } from "@angular/core";
import {
  DomSanitizer,
  SafeHtml,
  SafeUrl,
  SafeResourceUrl,
} from "@angular/platform-browser";

export interface SanitizationOptions {
  allowedTags?: string[];
  allowedAttributes?: { [tag: string]: string[] };
  allowedSchemes?: string[];
  stripTags?: boolean;
}

@Injectable({
  providedIn: "root",
})
export class SanitizationService {
  private sanitizer = inject(DomSanitizer);

  // Default allowed HTML tags for rich content
  private readonly DEFAULT_ALLOWED_TAGS = [
    "p",
    "br",
    "strong",
    "em",
    "u",
    "i",
    "b",
    "h1",
    "h2",
    "h3",
    "h4",
    "h5",
    "h6",
    "ul",
    "ol",
    "li",
    "blockquote",
    "code",
    "pre",
  ];

  // Default allowed attributes
  private readonly DEFAULT_ALLOWED_ATTRIBUTES = {
    a: ["href", "title"],
    img: ["src", "alt", "width", "height"],
    code: ["class"],
    pre: ["class"],
  };

  // Safe URL schemes
  private readonly SAFE_URL_SCHEMES = ["http", "https", "mailto", "tel"];

  /**
   * Sanitize HTML content with custom options
   */
  sanitizeHtml(content: string, options: SanitizationOptions = {}): SafeHtml {
    if (!content || typeof content !== "string") {
      return this.sanitizer.bypassSecurityTrustHtml("");
    }

    const allowedTags = options.allowedTags || this.DEFAULT_ALLOWED_TAGS;
    const allowedAttributes =
      options.allowedAttributes || this.DEFAULT_ALLOWED_ATTRIBUTES;

    // Remove script tags and event handlers
    let sanitizedContent = this.removeScriptTags(content);
    sanitizedContent = this.removeEventHandlers(sanitizedContent);
    sanitizedContent = this.filterAllowedTags(sanitizedContent, allowedTags);
    sanitizedContent = this.filterAllowedAttributes(
      sanitizedContent,
      allowedAttributes
    );

    if (options.stripTags) {
      sanitizedContent = this.stripHtmlTags(sanitizedContent);
    }

    return this.sanitizer.bypassSecurityTrustHtml(sanitizedContent);
  }

  /**
   * Sanitize URLs to prevent javascript: and data: schemes
   */
  sanitizeUrl(url: string): SafeUrl {
    if (!url || typeof url !== "string") {
      return this.sanitizer.bypassSecurityTrustUrl("");
    }

    // Check for dangerous schemes
    const lowerUrl = url.toLowerCase().trim();

    if (
      lowerUrl.startsWith("javascript:") ||
      lowerUrl.startsWith("data:") ||
      lowerUrl.startsWith("vbscript:")
    ) {
      return this.sanitizer.bypassSecurityTrustUrl("");
    }

    // Validate against allowed schemes
    const hasValidScheme = this.SAFE_URL_SCHEMES.some((scheme) =>
      lowerUrl.startsWith(`${scheme}:`)
    );

    if (
      !hasValidScheme &&
      !lowerUrl.startsWith("/") &&
      !lowerUrl.startsWith("#")
    ) {
      return this.sanitizer.bypassSecurityTrustUrl("");
    }

    return this.sanitizer.bypassSecurityTrustUrl(url);
  }

  /**
   * Sanitize resource URLs (for iframes, etc.)
   */
  sanitizeResourceUrl(
    url: string,
    allowedDomains: string[] = []
  ): SafeResourceUrl {
    if (!url || typeof url !== "string") {
      return this.sanitizer.bypassSecurityTrustResourceUrl("");
    }

    try {
      const urlObj = new URL(url);

      // Check if domain is in allowed list
      if (allowedDomains.length > 0) {
        const isAllowed = allowedDomains.some(
          (domain) =>
            urlObj.hostname === domain || urlObj.hostname.endsWith(`.${domain}`)
        );

        if (!isAllowed) {
          return this.sanitizer.bypassSecurityTrustResourceUrl("");
        }
      }

      // Ensure HTTPS for external resources
      if (urlObj.protocol !== "https:" && urlObj.hostname !== "localhost") {
        return this.sanitizer.bypassSecurityTrustResourceUrl("");
      }

      return this.sanitizer.bypassSecurityTrustResourceUrl(url);
    } catch {
      return this.sanitizer.bypassSecurityTrustResourceUrl("");
    }
  }

  /**
   * Escape user input for display in HTML
   */
  escapeHtml(input: string): string {
    if (!input || typeof input !== "string") return "";

    return input
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#x27;")
      .replace(/\//g, "&#x2F;");
  }

  /**
   * Validate and sanitize user input for SQL injection prevention
   */
  sanitizeInput(
    input: string,
    type: "text" | "email" | "numeric" | "alphanumeric" = "text"
  ): string {
    if (!input || typeof input !== "string") return "";

    let sanitized = input.trim();

    switch (type) {
      case "email":
        // Remove non-email characters
        sanitized = sanitized.replace(/[^a-zA-Z0-9@._-]/g, "");
        break;
      case "numeric":
        // Keep only numbers and decimal points
        sanitized = sanitized.replace(/[^0-9.-]/g, "");
        break;
      case "alphanumeric":
        // Keep only letters, numbers, and common safe characters
        sanitized = sanitized.replace(/[^a-zA-Z0-9\s._-]/g, "");
        break;
      case "text":
      default:
        // Remove potential SQL injection patterns
        sanitized = this.removeSqlInjectionPatterns(sanitized);
        break;
    }

    return sanitized;
  }

  // Private helper methods
  private removeScriptTags(content: string): string {
    return content.replace(
      /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi,
      ""
    );
  }

  private removeEventHandlers(content: string): string {
    // Remove common event handlers
    const eventHandlers = [
      "onload",
      "onerror",
      "onclick",
      "onmouseover",
      "onmouseout",
      "onfocus",
      "onblur",
      "onchange",
      "onsubmit",
      "onkeydown",
      "onkeyup",
      "onkeypress",
      "ontouchstart",
      "ontouchend",
    ];

    let result = content;
    eventHandlers.forEach((handler) => {
      const regex = new RegExp(`\\s*${handler}\\s*=\\s*[^\\s>]*`, "gi");
      result = result.replace(regex, "");
    });

    return result;
  }

  private filterAllowedTags(content: string, allowedTags: string[]): string {
    const tagRegex = /<\/?([a-zA-Z][a-zA-Z0-9]*)\b[^>]*>/gi;

    return content.replace(tagRegex, (match, tagName) => {
      if (allowedTags.includes(tagName.toLowerCase())) {
        return match;
      }
      return "";
    });
  }

  private filterAllowedAttributes(
    content: string,
    allowedAttributes: { [tag: string]: string[] }
  ): string {
    const tagWithAttributesRegex = /<([a-zA-Z][a-zA-Z0-9]*)\s+([^>]*)>/gi;

    return content.replace(
      tagWithAttributesRegex,
      (match, tagName, attributes) => {
        const allowedAttrs = allowedAttributes[tagName.toLowerCase()] || [];

        if (allowedAttrs.length === 0) {
          return `<${tagName}>`;
        }

        const filteredAttrs = this.filterAttributes(attributes, allowedAttrs);
        return `<${tagName}${filteredAttrs ? " " + filteredAttrs : ""}>`;
      }
    );
  }

  private filterAttributes(attributes: string, allowedAttrs: string[]): string {
    const attrRegex = /(\w+)\s*=\s*["']([^"']*)["']/gi;
    const validAttrs: string[] = [];

    let match;
    while ((match = attrRegex.exec(attributes)) !== null) {
      const [, attrName, attrValue] = match;

      if (allowedAttrs.includes(attrName.toLowerCase())) {
        // Additional validation for href attributes
        if (attrName.toLowerCase() === "href") {
          const safeUrl = this.sanitizeUrl(attrValue);
          if (safeUrl) {
            validAttrs.push(`${attrName}="${attrValue}"`);
          }
        } else {
          validAttrs.push(`${attrName}="${this.escapeHtml(attrValue)}"`);
        }
      }
    }

    return validAttrs.join(" ");
  }

  private stripHtmlTags(content: string): string {
    return content.replace(/<[^>]*>/g, "");
  }

  private removeSqlInjectionPatterns(input: string): string {
    const dangerousPatterns = [
      /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION)\b)/gi,
      /(--|\#|\/\*|\*\/)/g,
      /(\b(OR|AND)\s+\d+\s*=\s*\d+)/gi,
      /('\s*(OR|AND)\s*')/gi,
    ];

    let result = input;
    dangerousPatterns.forEach((pattern) => {
      result = result.replace(pattern, "");
    });

    return result;
  }
}
```

### **2. 🔍 Content Security Policy**

```typescript
// src/app/services/csp.service.ts - Content Security Policy Service
import { Injectable, DOCUMENT, inject } from "@angular/core";

export interface CSPConfig {
  defaultSrc?: string[];
  scriptSrc?: string[];
  styleSrc?: string[];
  imgSrc?: string[];
  connectSrc?: string[];
  fontSrc?: string[];
  objectSrc?: string[];
  mediaSrc?: string[];
  frameSrc?: string[];
  childSrc?: string[];
  formAction?: string[];
  frameAncestors?: string[];
  baseUri?: string[];
  upgradeInsecureRequests?: boolean;
  blockAllMixedContent?: boolean;
  reportUri?: string;
}

@Injectable({
  providedIn: "root",
})
export class CSPService {
  private document = inject(DOCUMENT);

  private readonly DEFAULT_CSP_CONFIG: CSPConfig = {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"], // Remove unsafe-inline in production
    styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
    imgSrc: ["'self'", "data:", "https:"],
    connectSrc: ["'self'", "https://api.example.com"],
    fontSrc: ["'self'", "https://fonts.gstatic.com"],
    objectSrc: ["'none'"],
    mediaSrc: ["'self'"],
    frameSrc: ["'none'"],
    frameAncestors: ["'none'"],
    baseUri: ["'self'"],
    formAction: ["'self'"],
    upgradeInsecureRequests: true,
    blockAllMixedContent: true,
  };

  /**
   * Apply Content Security Policy
   */
  applyCSP(config: Partial<CSPConfig> = {}): void {
    const cspConfig = { ...this.DEFAULT_CSP_CONFIG, ...config };
    const cspHeader = this.buildCSPHeader(cspConfig);

    // Set CSP via meta tag (for client-side enforcement)
    this.setCSPMetaTag(cspHeader);

    // Log CSP for debugging
    console.log("Applied CSP:", cspHeader);
  }

  /**
   * Validate if a script source is allowed by current CSP
   */
  isScriptSourceAllowed(src: string): boolean {
    // This would typically check against the current CSP policy
    // Implementation depends on your specific CSP configuration
    return this.validateSource(src, this.DEFAULT_CSP_CONFIG.scriptSrc || []);
  }

  /**
   * Report CSP violations
   */
  reportViolation(violation: any): void {
    console.warn("CSP Violation:", violation);

    // Send to monitoring service
    if (this.DEFAULT_CSP_CONFIG.reportUri) {
      fetch(this.DEFAULT_CSP_CONFIG.reportUri, {
        method: "POST",
        headers: {
          "Content-Type": "application/csp-report",
        },
        body: JSON.stringify({ "csp-report": violation }),
      }).catch((err) => console.error("Failed to report CSP violation:", err));
    }
  }

  /**
   * Setup CSP violation reporting
   */
  setupViolationReporting(): void {
    this.document.addEventListener("securitypolicyviolation", (e) => {
      this.reportViolation({
        "blocked-uri": e.blockedURI,
        "document-uri": e.documentURI,
        "effective-directive": e.effectiveDirective,
        "original-policy": e.originalPolicy,
        referrer: e.referrer,
        "source-file": e.sourceFile,
        "status-code": e.statusCode,
        "violated-directive": e.violatedDirective,
        "line-number": e.lineNumber,
        "column-number": e.columnNumber,
      });
    });
  }

  /**
   * Create nonce for inline scripts
   */
  generateNonce(): string {
    const array = new Uint8Array(16);
    crypto.getRandomValues(array);
    return btoa(String.fromCharCode(...array));
  }

  /**
   * Add script with nonce
   */
  addScriptWithNonce(scriptContent: string, nonce?: string): void {
    const actualNonce = nonce || this.generateNonce();
    const script = this.document.createElement("script");

    script.nonce = actualNonce;
    script.textContent = scriptContent;

    this.document.head.appendChild(script);
  }

  // Private helper methods
  private buildCSPHeader(config: CSPConfig): string {
    const directives: string[] = [];

    // Add each directive if present
    if (config.defaultSrc) {
      directives.push(`default-src ${config.defaultSrc.join(" ")}`);
    }

    if (config.scriptSrc) {
      directives.push(`script-src ${config.scriptSrc.join(" ")}`);
    }

    if (config.styleSrc) {
      directives.push(`style-src ${config.styleSrc.join(" ")}`);
    }

    if (config.imgSrc) {
      directives.push(`img-src ${config.imgSrc.join(" ")}`);
    }

    if (config.connectSrc) {
      directives.push(`connect-src ${config.connectSrc.join(" ")}`);
    }

    if (config.fontSrc) {
      directives.push(`font-src ${config.fontSrc.join(" ")}`);
    }

    if (config.objectSrc) {
      directives.push(`object-src ${config.objectSrc.join(" ")}`);
    }

    if (config.mediaSrc) {
      directives.push(`media-src ${config.mediaSrc.join(" ")}`);
    }

    if (config.frameSrc) {
      directives.push(`frame-src ${config.frameSrc.join(" ")}`);
    }

    if (config.childSrc) {
      directives.push(`child-src ${config.childSrc.join(" ")}`);
    }

    if (config.formAction) {
      directives.push(`form-action ${config.formAction.join(" ")}`);
    }

    if (config.frameAncestors) {
      directives.push(`frame-ancestors ${config.frameAncestors.join(" ")}`);
    }

    if (config.baseUri) {
      directives.push(`base-uri ${config.baseUri.join(" ")}`);
    }

    if (config.upgradeInsecureRequests) {
      directives.push("upgrade-insecure-requests");
    }

    if (config.blockAllMixedContent) {
      directives.push("block-all-mixed-content");
    }

    if (config.reportUri) {
      directives.push(`report-uri ${config.reportUri}`);
    }

    return directives.join("; ");
  }

  private setCSPMetaTag(cspHeader: string): void {
    // Remove existing CSP meta tag
    const existing = this.document.querySelector(
      'meta[http-equiv="Content-Security-Policy"]'
    );
    if (existing) {
      existing.remove();
    }

    // Add new CSP meta tag
    const meta = this.document.createElement("meta");
    meta.httpEquiv = "Content-Security-Policy";
    meta.content = cspHeader;

    this.document.head.appendChild(meta);
  }

  private validateSource(src: string, allowedSources: string[]): boolean {
    // Check if source is explicitly allowed
    if (allowedSources.includes(src)) return true;

    // Check for wildcard matches
    if (allowedSources.includes("'self'") && this.isSameOrigin(src))
      return true;
    if (allowedSources.includes("*")) return true;

    // Check for protocol matches
    if (allowedSources.includes("https:") && src.startsWith("https:"))
      return true;
    if (allowedSources.includes("data:") && src.startsWith("data:"))
      return true;

    return false;
  }

  private isSameOrigin(url: string): boolean {
    try {
      const urlObj = new URL(url, this.document.baseURI);
      return urlObj.origin === this.document.location.origin;
    } catch {
      return false;
    }
  }
}
```

## 🔒 **Authentication & Authorization**

### **1. 🎫 JWT Authentication Service**

```typescript
// src/app/services/auth.service.ts - Secure Authentication Service
import { Injectable, signal, computed, inject } from "@angular/core";
import { HttpClient, HttpErrorResponse } from "@angular/common/http";
import { Router } from "@angular/router";
import { Observable, BehaviorSubject, timer, throwError } from "rxjs";
import { tap, catchError, switchMap, filter, take } from "rxjs/operators";

export interface User {
  id: string;
  email: string;
  name: string;
  roles: string[];
  permissions: string[];
  lastLogin?: Date;
  mfaEnabled: boolean;
}

export interface AuthTokens {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
  tokenType: string;
}

export interface LoginCredentials {
  email: string;
  password: string;
  mfaCode?: string;
  rememberMe?: boolean;
}

export interface AuthState {
  user: User | null;
  tokens: AuthTokens | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

@Injectable({
  providedIn: "root",
})
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);

  private readonly TOKEN_KEY = "auth_tokens";
  private readonly REFRESH_THRESHOLD = 5 * 60 * 1000; // 5 minutes

  // Auth state signals
  private _authState = signal<AuthState>({
    user: null,
    tokens: null,
    isAuthenticated: false,
    isLoading: false,
    error: null,
  });

  // Public readonly state
  readonly authState = this._authState.asReadonly();
  readonly currentUser = computed(() => this._authState().user);
  readonly isAuthenticated = computed(() => this._authState().isAuthenticated);
  readonly isLoading = computed(() => this._authState().isLoading);
  readonly authError = computed(() => this._authState().error);

  // Token management
  private refreshTokenSubject = new BehaviorSubject<AuthTokens | null>(null);
  private isRefreshing = false;

  // Observable streams for backward compatibility
  readonly currentUser$ = new BehaviorSubject<User | null>(null);
  readonly isAuthenticated$ = this.currentUser$.pipe(map((user) => !!user));

  constructor() {
    this.initializeAuth();
    this.startTokenRefreshTimer();
  }

  /**
   * Initialize authentication state from stored tokens
   */
  private async initializeAuth(): Promise<void> {
    try {
      const storedTokens = this.getStoredTokens();

      if (storedTokens && !this.isTokenExpired(storedTokens.accessToken)) {
        this.updateAuthState({
          tokens: storedTokens,
          isLoading: true,
        });

        // Verify token with server and get user info
        const user = await this.getCurrentUserInfo();
        this.updateAuthState({
          user,
          isAuthenticated: true,
          isLoading: false,
        });
      }
    } catch (error) {
      this.clearAuthState();
    }
  }

  /**
   * Login with email and password
   */
  async login(credentials: LoginCredentials): Promise<User> {
    this.updateAuthState({ isLoading: true, error: null });

    try {
      const response = await this.http
        .post<{
          user: User;
          tokens: AuthTokens;
          requiresMfa?: boolean;
        }>("/api/auth/login", {
          email: credentials.email.toLowerCase().trim(),
          password: credentials.password,
          mfaCode: credentials.mfaCode,
          rememberMe: credentials.rememberMe,
        })
        .toPromise();

      if (!response) {
        throw new Error("Login failed: No response from server");
      }

      if (response.requiresMfa && !credentials.mfaCode) {
        this.updateAuthState({
          isLoading: false,
          error: "MFA_REQUIRED",
        });
        throw new Error("MFA verification required");
      }

      // Store tokens securely
      this.storeTokens(response.tokens, credentials.rememberMe);

      // Update auth state
      this.updateAuthState({
        user: response.user,
        tokens: response.tokens,
        isAuthenticated: true,
        isLoading: false,
        error: null,
      });

      // Update observable stream
      this.currentUser$.next(response.user);

      return response.user;
    } catch (error) {
      const errorMessage = this.handleAuthError(error);
      this.updateAuthState({
        isLoading: false,
        error: errorMessage,
      });
      throw error;
    }
  }

  /**
   * Logout and clear all auth data
   */
  async logout(): Promise<void> {
    const tokens = this._authState().tokens;

    try {
      // Notify server about logout
      if (tokens?.refreshToken) {
        await this.http
          .post("/api/auth/logout", {
            refreshToken: tokens.refreshToken,
          })
          .toPromise();
      }
    } catch (error) {
      console.warn("Logout request failed:", error);
    } finally {
      this.clearAuthState();
      this.router.navigate(["/login"]);
    }
  }

  /**
   * Refresh access token using refresh token
   */
  async refreshAccessToken(): Promise<AuthTokens | null> {
    const currentTokens = this._authState().tokens;

    if (!currentTokens?.refreshToken || this.isRefreshing) {
      return null;
    }

    this.isRefreshing = true;

    try {
      const response = await this.http
        .post<{
          tokens: AuthTokens;
        }>("/api/auth/refresh", {
          refreshToken: currentTokens.refreshToken,
        })
        .toPromise();

      if (!response) {
        throw new Error("Token refresh failed");
      }

      const newTokens = response.tokens;

      // Store new tokens
      this.storeTokens(newTokens);

      // Update auth state
      this.updateAuthState({
        tokens: newTokens,
      });

      // Notify waiting requests
      this.refreshTokenSubject.next(newTokens);

      return newTokens;
    } catch (error) {
      console.error("Token refresh failed:", error);
      this.clearAuthState();
      this.router.navigate(["/login"]);
      return null;
    } finally {
      this.isRefreshing = false;
    }
  }

  /**
   * Get current access token (with automatic refresh)
   */
  async getAccessToken(): Promise<string | null> {
    const tokens = this._authState().tokens;

    if (!tokens) return null;

    // Check if token needs refresh
    if (this.shouldRefreshToken(tokens.accessToken)) {
      const newTokens = await this.refreshAccessToken();
      return newTokens?.accessToken || null;
    }

    return tokens.accessToken;
  }

  /**
   * Check if user has specific role
   */
  hasRole(role: string): boolean {
    const user = this.currentUser();
    return user?.roles.includes(role) || false;
  }

  /**
   * Check if user has specific permission
   */
  hasPermission(permission: string): boolean {
    const user = this.currentUser();
    return user?.permissions.includes(permission) || false;
  }

  /**
   * Check if user has any of the specified roles
   */
  hasAnyRole(roles: string[]): boolean {
    return roles.some((role) => this.hasRole(role));
  }

  /**
   * Check if user has all specified permissions
   */
  hasAllPermissions(permissions: string[]): boolean {
    return permissions.every((permission) => this.hasPermission(permission));
  }

  /**
   * Update user profile
   */
  async updateProfile(profileData: Partial<User>): Promise<User> {
    const response = await this.http
      .put<{ user: User }>("/api/auth/profile", profileData)
      .toPromise();

    if (!response) {
      throw new Error("Profile update failed");
    }

    // Update auth state
    this.updateAuthState({
      user: response.user,
    });

    // Update observable stream
    this.currentUser$.next(response.user);

    return response.user;
  }

  /**
   * Change password
   */
  async changePassword(
    currentPassword: string,
    newPassword: string
  ): Promise<void> {
    await this.http
      .post("/api/auth/change-password", {
        currentPassword,
        newPassword,
      })
      .toPromise();
  }

  /**
   * Enable/disable MFA
   */
  async setupMFA(
    enable: boolean,
    secret?: string,
    code?: string
  ): Promise<{
    secret?: string;
    qrCode?: string;
  }> {
    const response = await this.http
      .post<{
        secret?: string;
        qrCode?: string;
      }>("/api/auth/mfa", {
        enable,
        secret,
        code,
      })
      .toPromise();

    return response || {};
  }

  // Private helper methods
  private updateAuthState(updates: Partial<AuthState>): void {
    this._authState.update((state) => ({ ...state, ...updates }));
  }

  private clearAuthState(): void {
    this.removeStoredTokens();
    this.updateAuthState({
      user: null,
      tokens: null,
      isAuthenticated: false,
      isLoading: false,
      error: null,
    });
    this.currentUser$.next(null);
  }

  private storeTokens(tokens: AuthTokens, persist = false): void {
    const storage = persist ? localStorage : sessionStorage;

    try {
      storage.setItem(
        this.TOKEN_KEY,
        JSON.stringify({
          ...tokens,
          timestamp: Date.now(),
        })
      );
    } catch (error) {
      console.error("Failed to store tokens:", error);
    }
  }

  private getStoredTokens(): AuthTokens | null {
    try {
      const stored =
        localStorage.getItem(this.TOKEN_KEY) ||
        sessionStorage.getItem(this.TOKEN_KEY);

      if (!stored) return null;

      const parsed = JSON.parse(stored);

      // Check if tokens are expired
      if (this.isTokenExpired(parsed.accessToken)) {
        this.removeStoredTokens();
        return null;
      }

      return parsed;
    } catch (error) {
      console.error("Failed to retrieve tokens:", error);
      this.removeStoredTokens();
      return null;
    }
  }

  private removeStoredTokens(): void {
    try {
      localStorage.removeItem(this.TOKEN_KEY);
      sessionStorage.removeItem(this.TOKEN_KEY);
    } catch (error) {
      console.error("Failed to remove tokens:", error);
    }
  }

  private isTokenExpired(token: string): boolean {
    try {
      const payload = JSON.parse(atob(token.split(".")[1]));
      const currentTime = Math.floor(Date.now() / 1000);
      return payload.exp < currentTime;
    } catch {
      return true;
    }
  }

  private shouldRefreshToken(token: string): boolean {
    try {
      const payload = JSON.parse(atob(token.split(".")[1]));
      const currentTime = Math.floor(Date.now() / 1000);
      const timeUntilExpiry = (payload.exp - currentTime) * 1000;

      return timeUntilExpiry < this.REFRESH_THRESHOLD;
    } catch {
      return true;
    }
  }

  private async getCurrentUserInfo(): Promise<User> {
    const response = await this.http
      .get<{ user: User }>("/api/auth/me")
      .toPromise();

    if (!response) {
      throw new Error("Failed to get user info");
    }

    return response.user;
  }

  private startTokenRefreshTimer(): void {
    // Check token status every minute
    timer(0, 60000).subscribe(() => {
      const tokens = this._authState().tokens;

      if (
        tokens &&
        this.shouldRefreshToken(tokens.accessToken) &&
        !this.isRefreshing
      ) {
        this.refreshAccessToken();
      }
    });
  }

  private handleAuthError(error: any): string {
    if (error instanceof HttpErrorResponse) {
      switch (error.status) {
        case 401:
          return "Invalid credentials";
        case 403:
          return "Account suspended or insufficient permissions";
        case 423:
          return "Account locked due to multiple failed attempts";
        case 429:
          return "Too many login attempts. Please try again later";
        default:
          return error.error?.message || "Authentication failed";
      }
    }

    return error.message || "An unexpected error occurred";
  }
}
```

## 🔐 **Part 1 Summary**

This first part covers:

- **🧼 XSS Prevention** - Advanced content sanitization service with custom options
- **🔍 Content Security Policy** - CSP implementation with violation reporting
- **🔒 Authentication** - Comprehensive JWT-based auth service with MFA support

**Coming in Part 2:**

- CSRF protection strategies
- Authorization guards and role-based access
- Secure HTTP interceptors
- Input validation and data protection
- Security testing approaches
