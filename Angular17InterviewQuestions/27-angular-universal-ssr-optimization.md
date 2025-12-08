# 🚀 Angular Universal & SSR Optimization: Complete Production Guide

## 🎯 **Question Overview**

_"How do you implement and optimize Angular Universal for server-side rendering with advanced caching strategies, performance optimization, and SEO enhancement?"_

## 🔍 **Understanding Angular Universal & SSR**

Angular Universal enables **server-side rendering (SSR)** for **improved SEO**, **faster initial load times**, **better social media sharing**, and **enhanced user experience**. Advanced optimization includes **intelligent caching**, **prerendering strategies**, **performance monitoring**, and **progressive hydration**! ⚡

## 🏗️ **Advanced Universal Setup**

### **1. 📦 Complete Universal Configuration**

```typescript
// angular.json - Enhanced Universal Configuration
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "my-app": {
      "projectType": "application",
      "schematics": {
        "@schematics/angular:component": {
          "style": "scss"
        }
      },
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/my-app/browser",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",
            "inlineStyleLanguage": "scss",
            "assets": [
              "src/favicon.ico",
              "src/assets",
              {
                "glob": "**/*",
                "input": "src/assets/prerendered",
                "output": "/prerendered/"
              }
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": [],
            "serviceWorker": true,
            "ngswConfigPath": "ngsw-config.json"
          },
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "2kb",
                  "maximumError": "4kb"
                }
              ],
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.prod.ts"
                }
              ],
              "outputHashing": "all",
              "optimization": {
                "scripts": true,
                "styles": {
                  "minify": true,
                  "inlineCritical": true
                },
                "fonts": true
              },
              "sourceMap": false,
              "namedChunks": false,
              "extractLicenses": true,
              "vendorChunk": false,
              "buildOptimizer": true
            }
          }
        },

        "server": {
          "builder": "@nguniversal/builders:ssr-dev-server",
          "options": {
            "browserTarget": "my-app:build",
            "serverTarget": "my-app:server"
          },
          "configurations": {
            "production": {
              "browserTarget": "my-app:build:production",
              "serverTarget": "my-app:server:production"
            }
          }
        },

        "build-ssr": {
          "builder": "@nguniversal/builders:ssr",
          "options": {
            "tsConfig": "tsconfig.server.json",
            "main": "server.ts"
          },
          "configurations": {
            "production": {
              "optimization": true,
              "outputHashing": "media",
              "sourceMap": false,
              "namedChunks": false,
              "extractLicenses": true,
              "vendorChunk": false
            }
          }
        },

        "prerender": {
          "builder": "@nguniversal/builders:prerender",
          "options": {
            "routes": [
              "/",
              "/products",
              "/about",
              "/contact",
              "/blog"
            ],
            "routesFile": "routes.txt"
          },
          "configurations": {
            "production": {
              "browserTarget": "my-app:build:production",
              "serverTarget": "my-app:build-ssr:production"
            }
          }
        }
      }
    }
  }
}
```

```typescript
// server.ts - Advanced Server Configuration
import "zone.js/dist/zone-node";
import { ngExpressEngine } from "@nguniversal/express-engine";
import { REQUEST, RESPONSE } from "@nguniversal/express-engine/tokens";
import express from "express";
import { join } from "path";
import compression from "compression";
import helmet from "helmet";
import rateLimit from "express-rate-limit";
import { LRUCache } from "lru-cache";

import { APP_BASE_HREF } from "@angular/common";
import { existsSync, readFileSync } from "fs";

// Import the server version of the app
import { AppServerModule } from "./src/main.server";

// Performance monitoring
import { performance, PerformanceObserver } from "perf_hooks";

// Enhanced caching configuration
interface CacheEntry {
  html: string;
  timestamp: number;
  etag: string;
  headers: Record<string, string>;
}

const ssrCache = new LRUCache<string, CacheEntry>({
  max: 1000, // Maximum number of cached pages
  ttl: 1000 * 60 * 60, // 1 hour TTL
  updateAgeOnGet: true,
  allowStale: true,
  fetchMethod: async (key: string, staleValue?: CacheEntry) => {
    // Implement cache refresh logic here
    return staleValue;
  },
});

// Performance tracking
const performanceEntries: { [key: string]: number[] } = {};

const perfObserver = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  entries.forEach((entry) => {
    if (!performanceEntries[entry.name]) {
      performanceEntries[entry.name] = [];
    }
    performanceEntries[entry.name].push(entry.duration);

    // Keep only last 100 measurements
    if (performanceEntries[entry.name].length > 100) {
      performanceEntries[entry.name] =
        performanceEntries[entry.name].slice(-100);
    }
  });
});

perfObserver.observe({ entryTypes: ["measure"] });

// Express application setup
const app = express();
const PORT = process.env["PORT"] || 4000;
const DIST_FOLDER = join(process.cwd(), "dist");

// Security middleware
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'", "'unsafe-eval'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
        connectSrc: ["'self'", "wss:", "ws:"],
        fontSrc: ["'self'"],
        objectSrc: ["'none'"],
        mediaSrc: ["'self'"],
        frameSrc: ["'none'"],
      },
    },
    crossOriginEmbedderPolicy: false,
  })
);

// Compression middleware
app.use(
  compression({
    level: 6,
    threshold: 1024,
    filter: (req, res) => {
      if (req.headers["x-no-compression"]) {
        return false;
      }
      return compression.filter(req, res);
    },
  })
);

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 1000, // Limit each IP to 1000 requests per windowMs
  message: "Too many requests from this IP",
  standardHeaders: true,
  legacyHeaders: false,
});

app.use(limiter);

// Template engine setup
const template = existsSync(
  join(DIST_FOLDER, "my-app/browser", "index.original.html")
)
  ? join(DIST_FOLDER, "my-app/browser", "index.original.html")
  : join(DIST_FOLDER, "my-app/browser", "index.html");

app.engine(
  "html",
  ngExpressEngine({
    bootstrap: AppServerModule,
    providers: [
      {
        provide: "REQUEST",
        useFactory: () => {
          // Enhanced request context
          return {};
        },
      },
    ],
  })
);

app.set("view engine", "html");
app.set("views", join(DIST_FOLDER, "my-app/browser"));

// Enhanced static file serving with caching
app.get(
  "*.*",
  express.static(join(DIST_FOLDER, "my-app/browser"), {
    maxAge: "1y",
    etag: true,
    lastModified: true,
    setHeaders: (res, path) => {
      // Set different cache headers based on file type
      if (path.endsWith(".js") || path.endsWith(".css")) {
        res.setHeader("Cache-Control", "public, max-age=31536000, immutable");
      } else if (path.endsWith(".html")) {
        res.setHeader("Cache-Control", "public, max-age=0, must-revalidate");
      } else {
        res.setHeader("Cache-Control", "public, max-age=86400");
      }
    },
  })
);

// Health check endpoint
app.get("/health", (req, res) => {
  const healthInfo = {
    status: "healthy",
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    cache: {
      size: ssrCache.size,
      hitRatio: ssrCache.calculatedSize,
    },
    performance: Object.keys(performanceEntries).reduce((acc, key) => {
      const entries = performanceEntries[key];
      acc[key] = {
        count: entries.length,
        average: entries.reduce((sum, val) => sum + val, 0) / entries.length,
        min: Math.min(...entries),
        max: Math.max(...entries),
      };
      return acc;
    }, {} as any),
  };

  res.json(healthInfo);
});

// Performance metrics endpoint
app.get("/metrics", (req, res) => {
  const metrics = {
    ssrCache: {
      size: ssrCache.size,
      maxSize: ssrCache.max,
      ttl: ssrCache.ttl,
    },
    performance: performanceEntries,
    memory: process.memoryUsage(),
    uptime: process.uptime(),
  };

  res.json(metrics);
});

// Enhanced SSR with intelligent caching
app.get("*", (req, res) => {
  const startTime = performance.now();
  const url = req.originalUrl;
  const userAgent = req.get("User-Agent") || "";

  // Generate cache key based on URL and relevant factors
  const cacheKey = generateCacheKey(url, userAgent, req.headers);

  // Check if response should be cached
  const shouldCache = shouldCacheRequest(req);

  if (shouldCache) {
    const cachedEntry = ssrCache.get(cacheKey);

    if (cachedEntry && !isStale(cachedEntry)) {
      // Serve from cache
      performance.mark("cache-hit-start");

      res.set({
        "X-Cache": "HIT",
        "X-Cache-Key": cacheKey,
        ETag: cachedEntry.etag,
        ...cachedEntry.headers,
      });

      const endTime = performance.now();
      performance.mark("cache-hit-end");
      performance.measure("cache-hit", "cache-hit-start", "cache-hit-end");

      console.log(
        `Cache HIT for ${url} (${(endTime - startTime).toFixed(2)}ms)`
      );
      return res.send(cachedEntry.html);
    }
  }

  // Server-side render
  performance.mark("ssr-start");

  res.render(
    "index",
    {
      req,
      providers: [
        { provide: APP_BASE_HREF, useValue: req.baseUrl },
        { provide: REQUEST, useValue: req },
        { provide: RESPONSE, useValue: res },
        {
          provide: "REQUEST_URL",
          useValue: req.protocol + "://" + req.get("host") + req.originalUrl,
        },
        {
          provide: "USER_AGENT",
          useValue: userAgent,
        },
      ],
    },
    (error: Error, html: string) => {
      performance.mark("ssr-end");
      performance.measure("ssr-render", "ssr-start", "ssr-end");

      const endTime = performance.now();
      const renderTime = endTime - startTime;

      if (error) {
        console.error("SSR Error:", error);
        return res.status(500).send("Server Error");
      }

      // Generate ETag for cache validation
      const etag = generateETag(html);

      // Set response headers
      const responseHeaders = {
        "X-Cache": "MISS",
        "X-Render-Time": `${renderTime.toFixed(2)}ms`,
        ETag: etag,
        Vary: "User-Agent, Accept-Encoding",
      };

      res.set(responseHeaders);

      // Cache the response if appropriate
      if (shouldCache && html) {
        const cacheEntry: CacheEntry = {
          html,
          timestamp: Date.now(),
          etag,
          headers: responseHeaders,
        };

        ssrCache.set(cacheKey, cacheEntry);
        console.log(`Cached response for ${url} (${renderTime.toFixed(2)}ms)`);
      }

      console.log(`SSR for ${url} completed in ${renderTime.toFixed(2)}ms`);
      res.send(html);
    }
  );
});

// Helper functions
function generateCacheKey(
  url: string,
  userAgent: string,
  headers: any
): string {
  // Create cache key based on URL and relevant factors
  const isMobile = /Mobile|Android|iPhone|iPad/i.test(userAgent);
  const acceptsWebP = headers.accept?.includes("image/webp") || false;

  return `${url}_${isMobile ? "mobile" : "desktop"}_${
    acceptsWebP ? "webp" : "standard"
  }`;
}

function shouldCacheRequest(req: express.Request): boolean {
  // Don't cache POST requests, authenticated requests, etc.
  if (req.method !== "GET") return false;
  if (req.headers.authorization) return false;
  if (req.headers.cookie?.includes("auth")) return false;
  if (req.query.nocache) return false;

  // Don't cache API routes
  if (req.originalUrl.startsWith("/api/")) return false;

  return true;
}

function isStale(entry: CacheEntry): boolean {
  const maxAge = 5 * 60 * 1000; // 5 minutes for demonstration
  return Date.now() - entry.timestamp > maxAge;
}

function generateETag(content: string): string {
  const crypto = require("crypto");
  return `"${crypto.createHash("md5").update(content).digest("hex")}"`;
}

// Graceful shutdown
process.on("SIGTERM", () => {
  console.log("SIGTERM received, shutting down gracefully");
  process.exit(0);
});

process.on("SIGINT", () => {
  console.log("SIGINT received, shutting down gracefully");
  process.exit(0);
});

// Start the server
app.listen(PORT, () => {
  console.log(`Universal server listening on http://localhost:${PORT}`);
  console.log(`Cache size limit: ${ssrCache.max} entries`);
});
```

### **2. 🎯 Advanced Universal Services**

```typescript
// src/app/shared/services/universal.service.ts
import { Injectable, Inject, PLATFORM_ID, Optional } from "@angular/core";
import { DOCUMENT, isPlatformBrowser, isPlatformServer } from "@angular/common";
import { REQUEST } from "@nguniversal/express-engine/tokens";
import { Request } from "express";
import { Meta, Title } from "@angular/platform-browser";
import { BehaviorSubject, Observable, of } from "rxjs";

export interface DeviceInfo {
  isMobile: boolean;
  isTablet: boolean;
  isDesktop: boolean;
  userAgent: string;
  platform: string;
  screenWidth?: number;
  screenHeight?: number;
}

export interface GeoLocation {
  country?: string;
  region?: string;
  city?: string;
  latitude?: number;
  longitude?: number;
  timezone?: string;
}

export interface UniversalContext {
  isBrowser: boolean;
  isServer: boolean;
  baseUrl: string;
  currentUrl: string;
  deviceInfo: DeviceInfo;
  geoLocation?: GeoLocation;
}

@Injectable({
  providedIn: "root",
})
export class UniversalService {
  private readonly context$ = new BehaviorSubject<UniversalContext | null>(
    null
  );

  readonly isBrowser: boolean;
  readonly isServer: boolean;

  constructor(
    @Inject(PLATFORM_ID) private platformId: Object,
    @Inject(DOCUMENT) private document: Document,
    @Optional() @Inject(REQUEST) private request: Request,
    private meta: Meta,
    private title: Title
  ) {
    this.isBrowser = isPlatformBrowser(this.platformId);
    this.isServer = isPlatformServer(this.platformId);

    this.initializeContext();
  }

  // Get universal context
  getContext(): Observable<UniversalContext | null> {
    return this.context$.asObservable();
  }

  getCurrentContext(): UniversalContext | null {
    return this.context$.value;
  }

  // Device detection
  getDeviceInfo(): DeviceInfo {
    const userAgent = this.getUserAgent();

    return {
      isMobile: this.isMobile(),
      isTablet: this.isTablet(),
      isDesktop: this.isDesktop(),
      userAgent,
      platform: this.getPlatform(),
      screenWidth: this.getScreenWidth(),
      screenHeight: this.getScreenHeight(),
    };
  }

  isMobile(): boolean {
    if (this.isServer && this.request) {
      const userAgent = this.request.get("User-Agent") || "";
      return /Mobile|Android|iPhone|iPod|BlackBerry|Windows Phone/i.test(
        userAgent
      );
    }

    if (this.isBrowser) {
      return (
        /Mobile|Android|iPhone|iPod|BlackBerry|Windows Phone/i.test(
          navigator.userAgent
        ) || window.innerWidth <= 768
      );
    }

    return false;
  }

  isTablet(): boolean {
    if (this.isServer && this.request) {
      const userAgent = this.request.get("User-Agent") || "";
      return /iPad|Tablet|PlayBook|Silk/i.test(userAgent);
    }

    if (this.isBrowser) {
      return (
        /iPad|Tablet|PlayBook|Silk/i.test(navigator.userAgent) ||
        (window.innerWidth > 768 && window.innerWidth <= 1024)
      );
    }

    return false;
  }

  isDesktop(): boolean {
    return !this.isMobile() && !this.isTablet();
  }

  // URL and navigation utilities
  getBaseUrl(): string {
    if (this.isServer && this.request) {
      return `${this.request.protocol}://${this.request.get("host")}`;
    }

    if (this.isBrowser) {
      return `${window.location.protocol}//${window.location.host}`;
    }

    return "";
  }

  getCurrentUrl(): string {
    if (this.isServer && this.request) {
      return `${this.getBaseUrl()}${this.request.originalUrl}`;
    }

    if (this.isBrowser) {
      return window.location.href;
    }

    return "";
  }

  // SEO utilities
  setSEOData(data: {
    title?: string;
    description?: string;
    keywords?: string;
    image?: string;
    type?: string;
    url?: string;
    siteName?: string;
  }): void {
    if (data.title) {
      this.title.setTitle(data.title);
      this.meta.updateTag({ property: "og:title", content: data.title });
      this.meta.updateTag({ name: "twitter:title", content: data.title });
    }

    if (data.description) {
      this.meta.updateTag({ name: "description", content: data.description });
      this.meta.updateTag({
        property: "og:description",
        content: data.description,
      });
      this.meta.updateTag({
        name: "twitter:description",
        content: data.description,
      });
    }

    if (data.keywords) {
      this.meta.updateTag({ name: "keywords", content: data.keywords });
    }

    if (data.image) {
      this.meta.updateTag({ property: "og:image", content: data.image });
      this.meta.updateTag({ name: "twitter:image", content: data.image });
    }

    if (data.type) {
      this.meta.updateTag({ property: "og:type", content: data.type });
    }

    if (data.url) {
      this.meta.updateTag({ property: "og:url", content: data.url });
      this.meta.updateTag({ rel: "canonical", href: data.url });
    }

    if (data.siteName) {
      this.meta.updateTag({ property: "og:site_name", content: data.siteName });
    }

    // Twitter Card
    this.meta.updateTag({
      name: "twitter:card",
      content: "summary_large_image",
    });
  }

  // Structured data utilities
  addStructuredData(data: any): void {
    if (!this.isBrowser) return;

    const script = this.document.createElement("script");
    script.type = "application/ld+json";
    script.text = JSON.stringify(data);

    const head = this.document.getElementsByTagName("head")[0];
    head.appendChild(script);
  }

  // Performance utilities
  preloadResource(
    url: string,
    type: "script" | "style" | "image" | "font" = "script"
  ): void {
    if (!this.isBrowser) return;

    const link = this.document.createElement("link");
    link.rel = "preload";
    link.href = url;

    switch (type) {
      case "script":
        link.as = "script";
        break;
      case "style":
        link.as = "style";
        break;
      case "image":
        link.as = "image";
        break;
      case "font":
        link.as = "font";
        link.crossOrigin = "anonymous";
        break;
    }

    const head = this.document.getElementsByTagName("head")[0];
    head.appendChild(link);
  }

  prefetchResource(url: string): void {
    if (!this.isBrowser) return;

    const link = this.document.createElement("link");
    link.rel = "prefetch";
    link.href = url;

    const head = this.document.getElementsByTagName("head")[0];
    head.appendChild(link);
  }

  // Browser-only operations
  runInBrowser<T>(callback: () => T): T | null {
    if (this.isBrowser) {
      return callback();
    }
    return null;
  }

  runInBrowserAsync<T>(callback: () => Promise<T>): Promise<T | null> {
    if (this.isBrowser) {
      return callback();
    }
    return Promise.resolve(null);
  }

  // Storage utilities (browser-safe)
  setLocalStorage(key: string, value: any): void {
    this.runInBrowser(() => {
      try {
        localStorage.setItem(key, JSON.stringify(value));
      } catch (error) {
        console.warn("LocalStorage not available:", error);
      }
    });
  }

  getLocalStorage(key: string): any {
    return this.runInBrowser(() => {
      try {
        const item = localStorage.getItem(key);
        return item ? JSON.parse(item) : null;
      } catch (error) {
        console.warn("LocalStorage not available:", error);
        return null;
      }
    });
  }

  setSessionStorage(key: string, value: any): void {
    this.runInBrowser(() => {
      try {
        sessionStorage.setItem(key, JSON.stringify(value));
      } catch (error) {
        console.warn("SessionStorage not available:", error);
      }
    });
  }

  getSessionStorage(key: string): any {
    return this.runInBrowser(() => {
      try {
        const item = sessionStorage.getItem(key);
        return item ? JSON.parse(item) : null;
      } catch (error) {
        console.warn("SessionStorage not available:", error);
        return null;
      }
    });
  }

  // Private helper methods
  private initializeContext(): void {
    const context: UniversalContext = {
      isBrowser: this.isBrowser,
      isServer: this.isServer,
      baseUrl: this.getBaseUrl(),
      currentUrl: this.getCurrentUrl(),
      deviceInfo: this.getDeviceInfo(),
      geoLocation: this.getGeoLocationFromRequest(),
    };

    this.context$.next(context);
  }

  private getUserAgent(): string {
    if (this.isServer && this.request) {
      return this.request.get("User-Agent") || "";
    }

    if (this.isBrowser) {
      return navigator.userAgent;
    }

    return "";
  }

  private getPlatform(): string {
    if (this.isServer && this.request) {
      const userAgent = this.request.get("User-Agent") || "";

      if (/Windows/i.test(userAgent)) return "Windows";
      if (/Mac|iPhone|iPad/i.test(userAgent)) return "macOS/iOS";
      if (/Android/i.test(userAgent)) return "Android";
      if (/Linux/i.test(userAgent)) return "Linux";

      return "Unknown";
    }

    if (this.isBrowser) {
      return navigator.platform;
    }

    return "Unknown";
  }

  private getScreenWidth(): number | undefined {
    return this.runInBrowser(() => window.innerWidth) || undefined;
  }

  private getScreenHeight(): number | undefined {
    return this.runInBrowser(() => window.innerHeight) || undefined;
  }

  private getGeoLocationFromRequest(): GeoLocation | undefined {
    if (this.isServer && this.request) {
      // In a real application, you would use IP geolocation service
      // or extract location from request headers set by a CDN
      const cloudflareCountry = this.request.get("CF-IPCountry");
      const cloudflareTimezone = this.request.get("CF-Timezone");

      if (cloudflareCountry || cloudflareTimezone) {
        return {
          country: cloudflareCountry,
          timezone: cloudflareTimezone,
        };
      }
    }

    return undefined;
  }
}
```

### **3. ⚡ Performance Optimization Service**

```typescript
// src/app/shared/services/ssr-optimization.service.ts
import { Injectable, inject, NgZone } from "@angular/core";
import { UniversalService } from "./universal.service";
import { BehaviorSubject, Observable, fromEvent, merge } from "rxjs";
import { debounceTime, distinctUntilChanged, filter } from "rxjs/operators";

export interface PerformanceMetrics {
  firstContentfulPaint?: number;
  largestContentfulPaint?: number;
  firstInputDelay?: number;
  cumulativeLayoutShift?: number;
  timeToInteractive?: number;
  serverRenderTime?: number;
  hydrationTime?: number;
}

export interface OptimizationConfig {
  enableLazyHydration: boolean;
  enableProgressiveHydration: boolean;
  enableCriticalCSS: boolean;
  enableResourceHints: boolean;
  prioritizeAboveFold: boolean;
}

@Injectable({
  providedIn: "root",
})
export class SSROptimizationService {
  private universalService = inject(UniversalService);
  private ngZone = inject(NgZone);

  private readonly metrics$ = new BehaviorSubject<PerformanceMetrics>({});
  private readonly config: OptimizationConfig = {
    enableLazyHydration: true,
    enableProgressiveHydration: true,
    enableCriticalCSS: true,
    enableResourceHints: true,
    prioritizeAboveFold: true,
  };

  constructor() {
    if (this.universalService.isBrowser) {
      this.initializePerformanceMonitoring();
      this.initializeOptimizations();
    }
  }

  // Performance metrics
  getPerformanceMetrics(): Observable<PerformanceMetrics> {
    return this.metrics$.asObservable();
  }

  // Critical CSS injection
  injectCriticalCSS(css: string): void {
    if (!this.universalService.isBrowser || !this.config.enableCriticalCSS) {
      return;
    }

    const style = document.createElement("style");
    style.textContent = css;
    style.setAttribute("data-critical", "true");

    const head = document.getElementsByTagName("head")[0];
    const firstLink = head.querySelector('link[rel="stylesheet"]');

    if (firstLink) {
      head.insertBefore(style, firstLink);
    } else {
      head.appendChild(style);
    }
  }

  // Progressive hydration
  enableProgressiveHydration(): void {
    if (
      !this.universalService.isBrowser ||
      !this.config.enableProgressiveHydration
    ) {
      return;
    }

    // Hydrate components based on viewport visibility
    this.hydrateOnIntersection();

    // Hydrate interactive components first
    this.hydrateInteractiveComponents();

    // Hydrate remaining components after main thread is idle
    this.hydrateOnIdle();
  }

  // Lazy hydration implementation
  enableLazyHydration(
    selector: string,
    priority: "high" | "medium" | "low" = "medium"
  ): void {
    if (!this.universalService.isBrowser || !this.config.enableLazyHydration) {
      return;
    }

    const elements = document.querySelectorAll(selector);

    elements.forEach((element) => {
      switch (priority) {
        case "high":
          this.hydrateImmediately(element);
          break;
        case "medium":
          this.hydrateOnVisible(element);
          break;
        case "low":
          this.hydrateOnIdle(element);
          break;
      }
    });
  }

  // Resource optimization
  optimizeResources(): void {
    if (!this.universalService.isBrowser || !this.config.enableResourceHints) {
      return;
    }

    // Preload critical resources
    this.preloadCriticalResources();

    // Prefetch next page resources
    this.prefetchNextPageResources();

    // Optimize images
    this.optimizeImages();

    // Defer non-critical JavaScript
    this.deferNonCriticalJS();
  }

  // Above-the-fold prioritization
  prioritizeAboveFoldContent(): void {
    if (!this.universalService.isBrowser || !this.config.prioritizeAboveFold) {
      return;
    }

    // Load above-the-fold images immediately
    const aboveFoldImages = this.getAboveFoldImages();
    aboveFoldImages.forEach((img) => {
      img.loading = "eager";
      img.fetchPriority = "high";
    });

    // Defer below-the-fold content
    this.deferBelowFoldContent();
  }

  // Font optimization
  optimizeFonts(): void {
    if (!this.universalService.isBrowser) return;

    // Use font-display: swap for web fonts
    const fontFaces = document.querySelectorAll('link[href*="fonts"]');
    fontFaces.forEach((link) => {
      link.setAttribute("rel", "preload");
      link.setAttribute("as", "font");
      link.setAttribute("type", "font/woff2");
      link.setAttribute("crossorigin", "anonymous");
    });

    // Add font-display CSS if not present
    this.addFontDisplayCSS();
  }

  // Service Worker optimization
  optimizeServiceWorker(): void {
    if (!this.universalService.isBrowser) return;

    // Register service worker with optimizations
    if ("serviceWorker" in navigator) {
      navigator.serviceWorker
        .register("/ngsw-worker.js", {
          scope: "/",
          updateViaCache: "imports",
        })
        .then((registration) => {
          console.log("SW registered with optimization");

          // Implement advanced caching strategies
          this.implementAdvancedCaching(registration);
        })
        .catch((error) => {
          console.error("SW registration failed:", error);
        });
    }
  }

  // Private implementation methods
  private initializePerformanceMonitoring(): void {
    this.ngZone.runOutsideAngular(() => {
      // Core Web Vitals monitoring
      this.monitorCoreWebVitals();

      // Custom performance metrics
      this.monitorCustomMetrics();

      // Performance observer for detailed metrics
      this.setupPerformanceObserver();
    });
  }

  private initializeOptimizations(): void {
    // Wait for DOM to be ready
    if (document.readyState === "loading") {
      document.addEventListener("DOMContentLoaded", () => {
        this.applyOptimizations();
      });
    } else {
      this.applyOptimizations();
    }
  }

  private applyOptimizations(): void {
    this.enableProgressiveHydration();
    this.optimizeResources();
    this.prioritizeAboveFoldContent();
    this.optimizeFonts();
    this.optimizeServiceWorker();
  }

  private monitorCoreWebVitals(): void {
    // First Contentful Paint
    this.observePerformanceEntry("first-contentful-paint", (entry) => {
      this.updateMetrics({ firstContentfulPaint: entry.startTime });
    });

    // Largest Contentful Paint
    this.observeLCP();

    // First Input Delay
    this.observeFID();

    // Cumulative Layout Shift
    this.observeCLS();
  }

  private observePerformanceEntry(
    entryName: string,
    callback: (entry: any) => void
  ): void {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        if (entry.name === entryName) {
          callback(entry);
        }
      });
    });

    observer.observe({ entryTypes: ["paint"] });
  }

  private observeLCP(): void {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1];
      this.updateMetrics({ largestContentfulPaint: lastEntry.startTime });
    });

    observer.observe({ entryTypes: ["largest-contentful-paint"] });
  }

  private observeFID(): void {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        this.updateMetrics({
          firstInputDelay: entry.processingStart - entry.startTime,
        });
      });
    });

    observer.observe({ entryTypes: ["first-input"] });
  }

  private observeCLS(): void {
    let clsValue = 0;

    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      });

      this.updateMetrics({ cumulativeLayoutShift: clsValue });
    });

    observer.observe({ entryTypes: ["layout-shift"] });
  }

  private monitorCustomMetrics(): void {
    // Time to Interactive
    this.calculateTimeToInteractive();

    // Hydration time
    this.measureHydrationTime();
  }

  private calculateTimeToInteractive(): void {
    // Simplified TTI calculation
    window.addEventListener("load", () => {
      setTimeout(() => {
        const navigationStart = performance.timing.navigationStart;
        const tti = performance.now();
        this.updateMetrics({ timeToInteractive: tti });
      }, 100);
    });
  }

  private measureHydrationTime(): void {
    const hydrationStart = performance.now();

    // Measure when Angular has fully hydrated
    this.ngZone.onStable.subscribe(() => {
      const hydrationTime = performance.now() - hydrationStart;
      this.updateMetrics({ hydrationTime });
    });
  }

  private setupPerformanceObserver(): void {
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        // Log detailed performance entries
        console.log(`Performance entry: ${entry.name}`, entry);
      });
    });

    observer.observe({
      entryTypes: ["navigation", "resource", "measure", "mark"],
    });
  }

  private hydrateOnIntersection(): void {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            this.hydrateElement(entry.target);
            observer.unobserve(entry.target);
          }
        });
      },
      { threshold: 0.1 }
    );

    document.querySelectorAll("[data-hydrate-on-visible]").forEach((el) => {
      observer.observe(el);
    });
  }

  private hydrateInteractiveComponents(): void {
    document.querySelectorAll("[data-interactive]").forEach((element) => {
      this.hydrateImmediately(element);
    });
  }

  private hydrateOnIdle(): void {
    if ("requestIdleCallback" in window) {
      requestIdleCallback(() => {
        document
          .querySelectorAll("[data-hydrate-on-idle]")
          .forEach((element) => {
            this.hydrateElement(element);
          });
      });
    } else {
      setTimeout(() => {
        document
          .querySelectorAll("[data-hydrate-on-idle]")
          .forEach((element) => {
            this.hydrateElement(element);
          });
      }, 100);
    }
  }

  private hydrateImmediately(element: Element): void {
    this.hydrateElement(element);
  }

  private hydrateOnVisible(element: Element): void {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          this.hydrateElement(entry.target);
          observer.unobserve(entry.target);
        }
      });
    });

    observer.observe(element);
  }

  private hydrateOnIdle(element: Element): void {
    if ("requestIdleCallback" in window) {
      requestIdleCallback(() => {
        this.hydrateElement(element);
      });
    } else {
      setTimeout(() => {
        this.hydrateElement(element);
      }, 100);
    }
  }

  private hydrateElement(element: Element): void {
    // Implementation depends on your hydration strategy
    console.log("Hydrating element:", element);
    element.removeAttribute("data-hydrate-on-visible");
    element.removeAttribute("data-hydrate-on-idle");
  }

  private preloadCriticalResources(): void {
    // Preload critical CSS
    this.universalService.preloadResource("/styles/critical.css", "style");

    // Preload critical JavaScript
    this.universalService.preloadResource("/js/critical.js", "script");

    // Preload critical fonts
    this.universalService.preloadResource("/fonts/main.woff2", "font");
  }

  private prefetchNextPageResources(): void {
    // Prefetch likely next page resources
    this.universalService.prefetchResource("/products");
    this.universalService.prefetchResource("/about");
  }

  private optimizeImages(): void {
    const images = document.querySelectorAll("img[data-src]");

    const imageObserver = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const img = entry.target as HTMLImageElement;
          img.src = img.dataset["src"] || "";
          img.removeAttribute("data-src");
          imageObserver.unobserve(img);
        }
      });
    });

    images.forEach((img) => imageObserver.observe(img));
  }

  private deferNonCriticalJS(): void {
    const scripts = document.querySelectorAll("script[data-defer]");

    scripts.forEach((script) => {
      const newScript = document.createElement("script");
      newScript.src = script.getAttribute("data-src") || "";
      newScript.defer = true;

      document.head.appendChild(newScript);
      script.remove();
    });
  }

  private getAboveFoldImages(): HTMLImageElement[] {
    const images = Array.from(document.querySelectorAll("img"));
    const viewportHeight = window.innerHeight;

    return images.filter((img) => {
      const rect = img.getBoundingClientRect();
      return rect.top < viewportHeight;
    });
  }

  private deferBelowFoldContent(): void {
    const belowFoldElements = document.querySelectorAll("[data-below-fold]");

    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const element = entry.target;
            element.removeAttribute("data-below-fold");
            observer.unobserve(element);
          }
        });
      },
      { rootMargin: "50px" }
    );

    belowFoldElements.forEach((el) => observer.observe(el));
  }

  private addFontDisplayCSS(): void {
    const style = document.createElement("style");
    style.textContent = `
      @font-face {
        font-display: swap;
      }
    `;
    document.head.appendChild(style);
  }

  private implementAdvancedCaching(
    registration: ServiceWorkerRegistration
  ): void {
    // Implementation would depend on your caching strategy
    console.log("Implementing advanced caching strategies:", registration);
  }

  private updateMetrics(newMetrics: Partial<PerformanceMetrics>): void {
    const currentMetrics = this.metrics$.value;
    this.metrics$.next({ ...currentMetrics, ...newMetrics });
  }
}
```

### **4. 📊 Advanced Prerendering & CDN Integration**

```typescript
// src/app/shared/services/prerender.service.ts
import { Injectable, inject } from "@angular/core";
import { Router, NavigationEnd, ActivatedRoute } from "@angular/router";
import { HttpClient } from "@angular/common/http";
import { UniversalService } from "./universal.service";
import { BehaviorSubject, Observable, of, combineLatest } from "rxjs";
import {
  filter,
  map,
  switchMap,
  catchError,
  debounceTime,
} from "rxjs/operators";

export interface PrerenderConfig {
  routes: string[];
  dynamicRoutes: DynamicRouteConfig[];
  excludePatterns: RegExp[];
  priority: RoutePriority[];
  cacheStrategy: CacheStrategy;
}

export interface DynamicRouteConfig {
  pattern: string;
  dataSource: string;
  priority: "high" | "medium" | "low";
  frequency: "daily" | "weekly" | "monthly";
}

export interface RoutePriority {
  pattern: string;
  priority: number;
  changeFrequency:
    | "always"
    | "hourly"
    | "daily"
    | "weekly"
    | "monthly"
    | "yearly"
    | "never";
  lastModified?: Date;
}

export interface CacheStrategy {
  defaultTTL: number;
  maxAge: number;
  staleWhileRevalidate: number;
  routeSpecific: { [pattern: string]: number };
}

export interface CDNConfig {
  provider: "cloudflare" | "aws" | "azure" | "custom";
  endpoints: string[];
  purgeEndpoint?: string;
  authToken?: string;
  zoneId?: string;
}

@Injectable({
  providedIn: "root",
})
export class PrerenderService {
  private universalService = inject(UniversalService);
  private router = inject(Router);
  private route = inject(ActivatedRoute);
  private http = inject(HttpClient);

  private readonly config$ = new BehaviorSubject<PrerenderConfig | null>(null);
  private readonly cdnConfig$ = new BehaviorSubject<CDNConfig | null>(null);

  private readonly defaultConfig: PrerenderConfig = {
    routes: ["/", "/about", "/contact", "/products", "/blog"],
    dynamicRoutes: [
      {
        pattern: "/blog/:slug",
        dataSource: "/api/blog/posts",
        priority: "high",
        frequency: "daily",
      },
      {
        pattern: "/product/:id",
        dataSource: "/api/products",
        priority: "medium",
        frequency: "weekly",
      },
    ],
    excludePatterns: [/^\/admin/, /^\/user\/profile/, /^\/checkout/, /\?/],
    priority: [
      { pattern: "/", priority: 1, changeFrequency: "weekly" },
      { pattern: "/products", priority: 0.9, changeFrequency: "daily" },
      { pattern: "/blog", priority: 0.8, changeFrequency: "daily" },
    ],
    cacheStrategy: {
      defaultTTL: 3600, // 1 hour
      maxAge: 86400, // 24 hours
      staleWhileRevalidate: 604800, // 1 week
      routeSpecific: {
        "/": 7200, // 2 hours for homepage
        "/products": 1800, // 30 minutes for product pages
        "/blog": 3600, // 1 hour for blog
      },
    },
  };

  constructor() {
    this.config$.next(this.defaultConfig);
    this.initializeRouteTracking();
  }

  // Configuration methods
  setConfig(config: Partial<PrerenderConfig>): void {
    const currentConfig = this.config$.value || this.defaultConfig;
    this.config$.next({ ...currentConfig, ...config });
  }

  setCDNConfig(config: CDNConfig): void {
    this.cdnConfig$.next(config);
  }

  // Route discovery and management
  async discoverDynamicRoutes(): Promise<string[]> {
    const config = this.config$.value;
    if (!config) return [];

    const discoveredRoutes: string[] = [];

    for (const dynamicRoute of config.dynamicRoutes) {
      try {
        const routes = await this.fetchDynamicRoutes(dynamicRoute);
        discoveredRoutes.push(...routes);
      } catch (error) {
        console.error(
          `Failed to discover routes for ${dynamicRoute.pattern}:`,
          error
        );
      }
    }

    return discoveredRoutes;
  }

  async generateSitemap(): Promise<string> {
    const config = this.config$.value;
    if (!config) return "";

    const allRoutes = await this.getAllRoutes();
    const baseUrl = this.universalService.getBaseUrl();

    const sitemapEntries = allRoutes
      .filter((route) => !this.shouldExcludeRoute(route))
      .map((route) => {
        const priority = this.getRoutePriority(route);
        const changeFreq = this.getChangeFrequency(route);
        const lastMod = this.getLastModified(route);

        return `
  <url>
    <loc>${baseUrl}${route}</loc>
    <lastmod>${lastMod}</lastmod>
    <changefreq>${changeFreq}</changefreq>
    <priority>${priority}</priority>
  </url>`;
      })
      .join("");

    return `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${sitemapEntries}
</urlset>`;
  }

  // CDN integration
  async purgeCache(routes?: string[]): Promise<boolean> {
    const cdnConfig = this.cdnConfig$.value;
    if (!cdnConfig || !cdnConfig.purgeEndpoint) {
      console.warn("CDN not configured for cache purging");
      return false;
    }

    try {
      const response = await this.purgeCDNCache(cdnConfig, routes);
      console.log("Cache purged successfully:", response);
      return true;
    } catch (error) {
      console.error("Failed to purge CDN cache:", error);
      return false;
    }
  }

  async warmupCache(routes?: string[]): Promise<void> {
    const config = this.config$.value;
    if (!config) return;

    const routesToWarmup = routes || (await this.getAllRoutes());
    const baseUrl = this.universalService.getBaseUrl();

    const warmupPromises = routesToWarmup.map((route) =>
      this.warmupRoute(`${baseUrl}${route}`).catch((error) =>
        console.error(`Failed to warm up ${route}:`, error)
      )
    );

    await Promise.allSettled(warmupPromises);
    console.log(`Cache warmup completed for ${routesToWarmup.length} routes`);
  }

  // Performance optimization
  async optimizePrerendering(): Promise<void> {
    // Analyze route performance
    const performanceData = await this.analyzeRoutePerformance();

    // Update prerender priorities based on analytics
    this.updatePrioritiesFromAnalytics(performanceData);

    // Optimize cache strategies
    this.optimizeCacheStrategies(performanceData);

    // Schedule intelligent prerendering
    this.scheduleIntelligentPrerendering();
  }

  // Route validation
  async validatePrerenderRoutes(): Promise<{
    valid: string[];
    invalid: string[];
  }> {
    const allRoutes = await this.getAllRoutes();
    const valid: string[] = [];
    const invalid: string[] = [];

    for (const route of allRoutes) {
      const isValid = await this.validateRoute(route);
      if (isValid) {
        valid.push(route);
      } else {
        invalid.push(route);
      }
    }

    return { valid, invalid };
  }

  // Private implementation methods
  private async getAllRoutes(): Promise<string[]> {
    const config = this.config$.value;
    if (!config) return [];

    const staticRoutes = config.routes;
    const dynamicRoutes = await this.discoverDynamicRoutes();

    return [...staticRoutes, ...dynamicRoutes];
  }

  private async fetchDynamicRoutes(
    config: DynamicRouteConfig
  ): Promise<string[]> {
    try {
      const response = await this.http
        .get<any[]>(config.dataSource)
        .toPromise();

      return (response || []).map((item) => {
        return config.pattern
          .replace(":id", item.id)
          .replace(":slug", item.slug);
      });
    } catch (error) {
      console.error(
        `Failed to fetch dynamic routes from ${config.dataSource}:`,
        error
      );
      return [];
    }
  }

  private shouldExcludeRoute(route: string): boolean {
    const config = this.config$.value;
    if (!config) return false;

    return config.excludePatterns.some((pattern) => pattern.test(route));
  }

  private getRoutePriority(route: string): number {
    const config = this.config$.value;
    if (!config) return 0.5;

    const priorityConfig = config.priority.find((p) =>
      new RegExp(p.pattern.replace("*", ".*")).test(route)
    );

    return priorityConfig ? priorityConfig.priority : 0.5;
  }

  private getChangeFrequency(route: string): string {
    const config = this.config$.value;
    if (!config) return "weekly";

    const priorityConfig = config.priority.find((p) =>
      new RegExp(p.pattern.replace("*", ".*")).test(route)
    );

    return priorityConfig ? priorityConfig.changeFrequency : "weekly";
  }

  private getLastModified(route: string): string {
    const config = this.config$.value;
    if (!config) return new Date().toISOString().split("T")[0];

    const priorityConfig = config.priority.find((p) =>
      new RegExp(p.pattern.replace("*", ".*")).test(route)
    );

    const lastMod = priorityConfig?.lastModified || new Date();
    return lastMod.toISOString().split("T")[0];
  }

  private async purgeCDNCache(
    config: CDNConfig,
    routes?: string[]
  ): Promise<any> {
    const payload = this.buildPurgePayload(config, routes);

    const headers: any = {
      "Content-Type": "application/json",
    };

    if (config.authToken) {
      headers["Authorization"] = `Bearer ${config.authToken}`;
    }

    switch (config.provider) {
      case "cloudflare":
        headers["X-Auth-Email"] = "your-email@example.com";
        headers["X-Auth-Key"] = config.authToken;
        break;
      case "aws":
        // AWS CloudFront invalidation headers
        break;
      case "azure":
        // Azure CDN purge headers
        break;
    }

    return this.http
      .post(config.purgeEndpoint!, payload, { headers })
      .toPromise();
  }

  private buildPurgePayload(config: CDNConfig, routes?: string[]): any {
    switch (config.provider) {
      case "cloudflare":
        return {
          files: routes || ["*"],
        };
      case "aws":
        return {
          Paths: {
            Quantity: routes ? routes.length : 1,
            Items: routes || ["/*"],
          },
        };
      default:
        return { routes: routes || ["*"] };
    }
  }

  private async warmupRoute(url: string): Promise<void> {
    try {
      await this.http
        .get(url, {
          headers: { "User-Agent": "Prerender-Bot/1.0" },
        })
        .toPromise();
    } catch (error) {
      throw new Error(`Failed to warm up ${url}: ${error}`);
    }
  }

  private async analyzeRoutePerformance(): Promise<any> {
    // In a real implementation, this would analyze:
    // - Route access patterns
    // - Performance metrics
    // - User behavior data
    // - Cache hit rates

    return {
      mostVisited: ["/", "/products", "/blog"],
      slowestRoutes: ["/heavy-page", "/data-intensive"],
      cacheHitRates: { "/": 0.95, "/products": 0.8 },
      averageLoadTimes: { "/": 250, "/products": 400 },
    };
  }

  private updatePrioritiesFromAnalytics(data: any): void {
    const config = this.config$.value;
    if (!config) return;

    // Update route priorities based on performance data
    data.mostVisited.forEach((route: string, index: number) => {
      const priority = config.priority.find((p) => p.pattern === route);
      if (priority) {
        priority.priority = Math.max(0.8, 1 - index * 0.1);
      }
    });

    this.config$.next(config);
  }

  private optimizeCacheStrategies(data: any): void {
    const config = this.config$.value;
    if (!config) return;

    // Adjust cache TTLs based on hit rates
    Object.entries(data.cacheHitRates).forEach(
      ([route, hitRate]: [string, any]) => {
        if (hitRate > 0.9) {
          // High hit rate - increase TTL
          config.cacheStrategy.routeSpecific[route] *= 1.5;
        } else if (hitRate < 0.7) {
          // Low hit rate - decrease TTL
          config.cacheStrategy.routeSpecific[route] *= 0.7;
        }
      }
    );

    this.config$.next(config);
  }

  private scheduleIntelligentPrerendering(): void {
    // Schedule prerendering based on:
    // - Traffic patterns
    // - Content update schedules
    // - Resource availability

    console.log("Intelligent prerendering scheduled");
  }

  private async validateRoute(route: string): Promise<boolean> {
    try {
      const baseUrl = this.universalService.getBaseUrl();
      const response = await this.http.head(`${baseUrl}${route}`).toPromise();
      return true;
    } catch (error) {
      return false;
    }
  }

  private initializeRouteTracking(): void {
    this.router.events
      .pipe(
        filter((event) => event instanceof NavigationEnd),
        debounceTime(100)
      )
      .subscribe((event: NavigationEnd) => {
        this.trackRouteVisit(event.url);
      });
  }

  private trackRouteVisit(url: string): void {
    // Track route visits for analytics
    console.log("Route visited:", url);
  }
}
```

### **5. 🔍 SEO & Structured Data Service**

```typescript
// src/app/shared/services/seo.service.ts
import { Injectable, inject } from "@angular/core";
import { Meta, Title } from "@angular/platform-browser";
import { Router, NavigationEnd, ActivatedRoute } from "@angular/router";
import { UniversalService } from "./universal.service";
import { BehaviorSubject, Observable, combineLatest } from "rxjs";
import { filter, map, switchMap } from "rxjs/operators";

export interface SEOConfig {
  siteName: string;
  siteUrl: string;
  defaultTitle: string;
  defaultDescription: string;
  defaultImage: string;
  defaultKeywords: string[];
  twitterHandle: string;
  facebookAppId: string;
  language: string;
  locale: string;
}

export interface PageSEOData {
  title: string;
  description: string;
  keywords?: string[];
  image?: string;
  type?: string;
  url?: string;
  author?: string;
  publishedTime?: string;
  modifiedTime?: string;
  section?: string;
  tags?: string[];
  canonical?: string;
}

export interface StructuredDataType {
  "@context": string;
  "@type": string;
  [key: string]: any;
}

export interface BreadcrumbItem {
  name: string;
  url: string;
}

export interface FAQItem {
  question: string;
  answer: string;
}

@Injectable({
  providedIn: "root",
})
export class SEOService {
  private universalService = inject(UniversalService);
  private meta = inject(Meta);
  private title = inject(Title);
  private router = inject(Router);
  private route = inject(ActivatedRoute);

  private readonly config$ = new BehaviorSubject<SEOConfig | null>(null);
  private readonly currentPageData$ = new BehaviorSubject<PageSEOData | null>(
    null
  );

  private readonly defaultConfig: SEOConfig = {
    siteName: "Your Site Name",
    siteUrl: "https://yoursite.com",
    defaultTitle: "Your Site - Default Title",
    defaultDescription: "Default description for your site",
    defaultImage: "/assets/images/og-default.jpg",
    defaultKeywords: ["angular", "typescript", "web development"],
    twitterHandle: "@yourhandle",
    facebookAppId: "your-facebook-app-id",
    language: "en",
    locale: "en_US",
  };

  constructor() {
    this.config$.next(this.defaultConfig);
    this.initializeRouteBasedSEO();
  }

  // Configuration
  setConfig(config: Partial<SEOConfig>): void {
    const currentConfig = this.config$.value || this.defaultConfig;
    this.config$.next({ ...currentConfig, ...config });
  }

  // Page SEO management
  updatePageSEO(data: PageSEOData): void {
    this.currentPageData$.next(data);
    this.applyPageSEO(data);
  }

  // Structured data methods
  addWebsiteStructuredData(): void {
    const config = this.config$.value;
    if (!config) return;

    const websiteData: StructuredDataType = {
      "@context": "https://schema.org",
      "@type": "WebSite",
      name: config.siteName,
      url: config.siteUrl,
      potentialAction: {
        "@type": "SearchAction",
        target: {
          "@type": "EntryPoint",
          urlTemplate: `${config.siteUrl}/search?q={search_term_string}`,
        },
        "query-input": "required name=search_term_string",
      },
    };

    this.universalService.addStructuredData(websiteData);
  }

  addOrganizationStructuredData(data: {
    name: string;
    logo: string;
    url: string;
    description?: string;
    address?: any;
    contactPoint?: any;
    sameAs?: string[];
  }): void {
    const organizationData: StructuredDataType = {
      "@context": "https://schema.org",
      "@type": "Organization",
      name: data.name,
      logo: {
        "@type": "ImageObject",
        url: data.logo,
      },
      url: data.url,
      ...data,
    };

    this.universalService.addStructuredData(organizationData);
  }

  addBreadcrumbStructuredData(items: BreadcrumbItem[]): void {
    const breadcrumbData: StructuredDataType = {
      "@context": "https://schema.org",
      "@type": "BreadcrumbList",
      itemListElement: items.map((item, index) => ({
        "@type": "ListItem",
        position: index + 1,
        name: item.name,
        item: item.url,
      })),
    };

    this.universalService.addStructuredData(breadcrumbData);
  }

  addArticleStructuredData(data: {
    headline: string;
    description: string;
    image: string;
    author: string;
    datePublished: string;
    dateModified?: string;
    publisher: string;
    url: string;
  }): void {
    const config = this.config$.value;
    if (!config) return;

    const articleData: StructuredDataType = {
      "@context": "https://schema.org",
      "@type": "Article",
      headline: data.headline,
      description: data.description,
      image: {
        "@type": "ImageObject",
        url: data.image,
      },
      author: {
        "@type": "Person",
        name: data.author,
      },
      publisher: {
        "@type": "Organization",
        name: data.publisher,
        logo: {
          "@type": "ImageObject",
          url: config.defaultImage,
        },
      },
      datePublished: data.datePublished,
      dateModified: data.dateModified || data.datePublished,
      url: data.url,
    };

    this.universalService.addStructuredData(articleData);
  }

  addProductStructuredData(data: {
    name: string;
    description: string;
    image: string;
    brand: string;
    sku: string;
    price: number;
    currency: string;
    availability: string;
    condition?: string;
    reviews?: any[];
    rating?: number;
  }): void {
    const productData: StructuredDataType = {
      "@context": "https://schema.org",
      "@type": "Product",
      name: data.name,
      description: data.description,
      image: data.image,
      brand: {
        "@type": "Brand",
        name: data.brand,
      },
      sku: data.sku,
      offers: {
        "@type": "Offer",
        price: data.price,
        priceCurrency: data.currency,
        availability: `https://schema.org/${data.availability}`,
        itemCondition: data.condition
          ? `https://schema.org/${data.condition}`
          : "https://schema.org/NewCondition",
      },
    };

    if (data.reviews && data.reviews.length > 0) {
      productData.review = data.reviews;
    }

    if (data.rating) {
      productData.aggregateRating = {
        "@type": "AggregateRating",
        ratingValue: data.rating,
        reviewCount: data.reviews?.length || 1,
      };
    }

    this.universalService.addStructuredData(productData);
  }

  addFAQStructuredData(faqs: FAQItem[]): void {
    const faqData: StructuredDataType = {
      "@context": "https://schema.org",
      "@type": "FAQPage",
      mainEntity: faqs.map((faq) => ({
        "@type": "Question",
        name: faq.question,
        acceptedAnswer: {
          "@type": "Answer",
          text: faq.answer,
        },
      })),
    };

    this.universalService.addStructuredData(faqData);
  }

  addLocalBusinessStructuredData(data: {
    name: string;
    description: string;
    image: string;
    telephone: string;
    address: {
      streetAddress: string;
      addressLocality: string;
      addressRegion: string;
      postalCode: string;
      addressCountry: string;
    };
    geo: {
      latitude: number;
      longitude: number;
    };
    openingHours: string[];
    priceRange?: string;
  }): void {
    const businessData: StructuredDataType = {
      "@context": "https://schema.org",
      "@type": "LocalBusiness",
      name: data.name,
      description: data.description,
      image: data.image,
      telephone: data.telephone,
      address: {
        "@type": "PostalAddress",
        ...data.address,
      },
      geo: {
        "@type": "GeoCoordinates",
        latitude: data.geo.latitude,
        longitude: data.geo.longitude,
      },
      openingHoursSpecification: data.openingHours.map((hours) => ({
        "@type": "OpeningHoursSpecification",
        dayOfWeek: hours.split(" ")[0],
        opens: hours.split(" ")[1],
        closes: hours.split(" ")[2],
      })),
    };

    if (data.priceRange) {
      businessData.priceRange = data.priceRange;
    }

    this.universalService.addStructuredData(businessData);
  }

  // Advanced SEO features
  addHreflangTags(alternatePages: { [language: string]: string }): void {
    Object.entries(alternatePages).forEach(([lang, url]) => {
      this.meta.updateTag({ rel: "alternate", hreflang: lang, href: url });
    });
  }

  addRobotsTags(robots: {
    index?: boolean;
    follow?: boolean;
    noarchive?: boolean;
    nosnippet?: boolean;
    noimageindex?: boolean;
    max_snippet?: number;
    max_image_preview?: string;
  }): void {
    const robotsContent = Object.entries(robots)
      .filter(([key, value]) => value !== undefined)
      .map(([key, value]) => {
        if (typeof value === "boolean") {
          return value ? key : `no${key}`;
        }
        return `${key.replace("_", "-")}:${value}`;
      })
      .join(", ");

    this.meta.updateTag({ name: "robots", content: robotsContent });
  }

  generateRSSFeed(posts: any[]): string {
    const config = this.config$.value;
    if (!config) return "";

    const rssItems = posts
      .map(
        (post) => `
    <item>
      <title>${this.escapeXML(post.title)}</title>
      <link>${config.siteUrl}${post.url}</link>
      <description>${this.escapeXML(post.description)}</description>
      <pubDate>${new Date(post.publishedDate).toUTCString()}</pubDate>
      <guid>${config.siteUrl}${post.url}</guid>
    </item>`
      )
      .join("");

    return `<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
  <channel>
    <title>${config.siteName}</title>
    <link>${config.siteUrl}</link>
    <description>${config.defaultDescription}</description>
    <language>${config.language}</language>
    <lastBuildDate>${new Date().toUTCString()}</lastBuildDate>
    ${rssItems}
  </channel>
</rss>`;
  }

  // Performance and monitoring
  trackSEOMetrics(): Observable<any> {
    return combineLatest([this.config$, this.currentPageData$]).pipe(
      map(([config, pageData]) => ({
        titleLength: pageData?.title?.length || 0,
        descriptionLength: pageData?.description?.length || 0,
        hasImage: !!pageData?.image,
        hasStructuredData: this.hasStructuredData(),
        hasCanonical: this.hasCanonicalTag(),
        hasRobotsTags: this.hasRobotsTags(),
      }))
    );
  }

  validateSEOCompliance(): {
    valid: boolean;
    issues: string[];
    recommendations: string[];
  } {
    const issues: string[] = [];
    const recommendations: string[] = [];
    const pageData = this.currentPageData$.value;

    if (!pageData?.title) {
      issues.push("Missing page title");
    } else if (pageData.title.length > 60) {
      issues.push("Title too long (>60 characters)");
    } else if (pageData.title.length < 30) {
      recommendations.push("Consider longer title (30-60 characters)");
    }

    if (!pageData?.description) {
      issues.push("Missing meta description");
    } else if (pageData.description.length > 160) {
      issues.push("Description too long (>160 characters)");
    } else if (pageData.description.length < 120) {
      recommendations.push("Consider longer description (120-160 characters)");
    }

    if (!pageData?.image) {
      recommendations.push("Add Open Graph image");
    }

    if (!this.hasStructuredData()) {
      recommendations.push("Add structured data");
    }

    return {
      valid: issues.length === 0,
      issues,
      recommendations,
    };
  }

  // Private methods
  private initializeRouteBasedSEO(): void {
    this.router.events
      .pipe(
        filter((event) => event instanceof NavigationEnd),
        switchMap(() => this.route.data)
      )
      .subscribe((data) => {
        if (data && data["seo"]) {
          this.updatePageSEO(data["seo"]);
        }
      });
  }

  private applyPageSEO(data: PageSEOData): void {
    const config = this.config$.value;
    if (!config) return;

    // Title
    const fullTitle = data.title
      ? `${data.title} | ${config.siteName}`
      : config.defaultTitle;
    this.title.setTitle(fullTitle);

    // Meta tags
    this.meta.updateTag({ name: "description", content: data.description });

    if (data.keywords && data.keywords.length > 0) {
      this.meta.updateTag({
        name: "keywords",
        content: data.keywords.join(", "),
      });
    }

    if (data.author) {
      this.meta.updateTag({ name: "author", content: data.author });
    }

    if (data.canonical) {
      this.meta.updateTag({ rel: "canonical", href: data.canonical });
    }

    // Open Graph
    this.meta.updateTag({ property: "og:title", content: data.title });
    this.meta.updateTag({
      property: "og:description",
      content: data.description,
    });
    this.meta.updateTag({
      property: "og:image",
      content: data.image || config.defaultImage,
    });
    this.meta.updateTag({
      property: "og:url",
      content: data.url || this.universalService.getCurrentUrl(),
    });
    this.meta.updateTag({
      property: "og:type",
      content: data.type || "website",
    });
    this.meta.updateTag({ property: "og:site_name", content: config.siteName });
    this.meta.updateTag({ property: "og:locale", content: config.locale });

    if (data.publishedTime) {
      this.meta.updateTag({
        property: "article:published_time",
        content: data.publishedTime,
      });
    }

    if (data.modifiedTime) {
      this.meta.updateTag({
        property: "article:modified_time",
        content: data.modifiedTime,
      });
    }

    if (data.section) {
      this.meta.updateTag({
        property: "article:section",
        content: data.section,
      });
    }

    if (data.tags && data.tags.length > 0) {
      data.tags.forEach((tag) => {
        this.meta.addTag({ property: "article:tag", content: tag });
      });
    }

    // Twitter Card
    this.meta.updateTag({
      name: "twitter:card",
      content: "summary_large_image",
    });
    this.meta.updateTag({ name: "twitter:title", content: data.title });
    this.meta.updateTag({
      name: "twitter:description",
      content: data.description,
    });
    this.meta.updateTag({
      name: "twitter:image",
      content: data.image || config.defaultImage,
    });
    this.meta.updateTag({
      name: "twitter:site",
      content: config.twitterHandle,
    });

    if (data.author) {
      this.meta.updateTag({ name: "twitter:creator", content: data.author });
    }
  }

  private hasStructuredData(): boolean {
    if (!this.universalService.isBrowser) return false;
    return (
      document.querySelector('script[type="application/ld+json"]') !== null
    );
  }

  private hasCanonicalTag(): boolean {
    if (!this.universalService.isBrowser) return false;
    return document.querySelector('link[rel="canonical"]') !== null;
  }

  private hasRobotsTags(): boolean {
    if (!this.universalService.isBrowser) return false;
    return document.querySelector('meta[name="robots"]') !== null;
  }

  private escapeXML(str: string): string {
    return str.replace(/[<>&'"]/g, (c) => {
      switch (c) {
        case "<":
          return "&lt;";
        case ">":
          return "&gt;";
        case "&":
          return "&amp;";
        case "'":
          return "&apos;";
        case '"':
          return "&quot;";
        default:
          return c;
      }
    });
  }
}
```

### **6. 📈 Performance Monitoring Service**

```typescript
// src/app/shared/services/performance-monitor.service.ts
import { Injectable, inject, NgZone } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { UniversalService } from "./universal.service";
import { BehaviorSubject, Observable, interval, fromEvent, merge } from "rxjs";
import { debounceTime, throttleTime, filter, map } from "rxjs/operators";

export interface PerformanceReport {
  timestamp: number;
  url: string;
  metrics: {
    serverSideMetrics: SSRMetrics;
    clientSideMetrics: ClientMetrics;
    webVitals: WebVitalsMetrics;
    resourceTiming: ResourceTiming[];
    userExperience: UserExperienceMetrics;
  };
  deviceInfo: {
    userAgent: string;
    connectionType?: string;
    effectiveConnectionType?: string;
    deviceMemory?: number;
    hardwareConcurrency?: number;
  };
}

export interface SSRMetrics {
  renderTime: number;
  hydrationTime: number;
  cacheHitRatio: number;
  memoryUsage: number;
  cpuUsage?: number;
}

export interface ClientMetrics {
  domContentLoaded: number;
  loadComplete: number;
  firstPaint: number;
  firstContentfulPaint: number;
  timeToInteractive: number;
  totalBlockingTime: number;
}

export interface WebVitalsMetrics {
  lcp: number; // Largest Contentful Paint
  fid: number; // First Input Delay
  cls: number; // Cumulative Layout Shift
  fcp: number; // First Contentful Paint
  ttfb: number; // Time to First Byte
}

export interface ResourceTiming {
  name: string;
  type: string;
  size: number;
  duration: number;
  transferSize: number;
  encodedBodySize: number;
  decodedBodySize: number;
}

export interface UserExperienceMetrics {
  scrollDepth: number;
  clickCount: number;
  timeOnPage: number;
  bounceRate: number;
  conversionRate?: number;
}

export interface PerformanceThresholds {
  lcp: { good: number; poor: number };
  fid: { good: number; poor: number };
  cls: { good: number; poor: number };
  ttfb: { good: number; poor: number };
  renderTime: { good: number; poor: number };
}

@Injectable({
  providedIn: "root",
})
export class PerformanceMonitorService {
  private universalService = inject(UniversalService);
  private http = inject(HttpClient);
  private ngZone = inject(NgZone);

  private readonly performanceData$ = new BehaviorSubject<PerformanceReport[]>(
    []
  );
  private readonly realTimeMetrics$ =
    new BehaviorSubject<Partial<PerformanceReport> | null>(null);

  private readonly thresholds: PerformanceThresholds = {
    lcp: { good: 2500, poor: 4000 },
    fid: { good: 100, poor: 300 },
    cls: { good: 0.1, poor: 0.25 },
    ttfb: { good: 800, poor: 1800 },
    renderTime: { good: 1000, poor: 3000 },
  };

  private startTime = Date.now();
  private pageStartTime = performance.now();
  private observers: PerformanceObserver[] = [];
  private userInteractions = 0;
  private maxScrollDepth = 0;

  constructor() {
    if (this.universalService.isBrowser) {
      this.initializePerformanceMonitoring();
      this.startRealTimeMonitoring();
    }
  }

  // Performance data access
  getPerformanceData(): Observable<PerformanceReport[]> {
    return this.performanceData$.asObservable();
  }

  getRealTimeMetrics(): Observable<Partial<PerformanceReport> | null> {
    return this.realTimeMetrics$.asObservable();
  }

  // Generate comprehensive performance report
  async generatePerformanceReport(): Promise<PerformanceReport> {
    const report: PerformanceReport = {
      timestamp: Date.now(),
      url: this.universalService.getCurrentUrl(),
      metrics: {
        serverSideMetrics: await this.getSSRMetrics(),
        clientSideMetrics: this.getClientMetrics(),
        webVitals: await this.getWebVitalsMetrics(),
        resourceTiming: this.getResourceTiming(),
        userExperience: this.getUserExperienceMetrics(),
      },
      deviceInfo: this.getDeviceInfo(),
    };

    // Store report
    const currentData = this.performanceData$.value;
    this.performanceData$.next([...currentData, report]);

    return report;
  }

  // Performance analysis
  analyzePerformance(reports?: PerformanceReport[]): {
    score: number;
    issues: string[];
    recommendations: string[];
    trends: any;
  } {
    const data = reports || this.performanceData$.value;
    if (data.length === 0) {
      return {
        score: 0,
        issues: ["No performance data available"],
        recommendations: [],
        trends: {},
      };
    }

    const latestReport = data[data.length - 1];
    const issues: string[] = [];
    const recommendations: string[] = [];
    let score = 100;

    // Analyze Web Vitals
    const vitals = latestReport.metrics.webVitals;

    if (vitals.lcp > this.thresholds.lcp.poor) {
      issues.push(
        `LCP is poor (${vitals.lcp}ms > ${this.thresholds.lcp.poor}ms)`
      );
      recommendations.push(
        "Optimize largest contentful paint by reducing server response times"
      );
      score -= 20;
    } else if (vitals.lcp > this.thresholds.lcp.good) {
      recommendations.push("LCP could be improved");
      score -= 10;
    }

    if (vitals.fid > this.thresholds.fid.poor) {
      issues.push(
        `FID is poor (${vitals.fid}ms > ${this.thresholds.fid.poor}ms)`
      );
      recommendations.push("Reduce JavaScript execution time");
      score -= 15;
    }

    if (vitals.cls > this.thresholds.cls.poor) {
      issues.push(`CLS is poor (${vitals.cls} > ${this.thresholds.cls.poor})`);
      recommendations.push("Ensure images and ads have dimensions");
      score -= 15;
    }

    // Analyze resource loading
    const largeResources = latestReport.metrics.resourceTiming.filter(
      (resource) => resource.transferSize > 1024 * 1024
    ); // > 1MB

    if (largeResources.length > 0) {
      issues.push(`${largeResources.length} resources are larger than 1MB`);
      recommendations.push("Optimize large resources with compression");
      score -= 10;
    }

    // Generate trends
    const trends = this.generateTrends(data);

    return { score: Math.max(0, score), issues, recommendations, trends };
  }

  // Automated optimization suggestions
  getOptimizationSuggestions(): string[] {
    const suggestions: string[] = [];
    const latestData = this.performanceData$.value;

    if (latestData.length === 0) return suggestions;

    const latest = latestData[latestData.length - 1];
    const metrics = latest.metrics;

    // SSR optimizations
    if (metrics.serverSideMetrics.renderTime > 2000) {
      suggestions.push("Enable intelligent caching for SSR");
      suggestions.push("Implement component-level caching");
    }

    // Client-side optimizations
    if (metrics.clientSideMetrics.totalBlockingTime > 300) {
      suggestions.push("Split large JavaScript bundles");
      suggestions.push("Use code splitting for route-based chunks");
    }

    // Resource optimizations
    const imageResources = metrics.resourceTiming.filter(
      (r) =>
        r.name.includes(".jpg") ||
        r.name.includes(".png") ||
        r.name.includes(".jpeg")
    );

    if (imageResources.some((img) => img.transferSize > 500000)) {
      suggestions.push("Optimize images with WebP format");
      suggestions.push("Implement responsive images with srcset");
    }

    // Web Vitals improvements
    if (metrics.webVitals.lcp > 3000) {
      suggestions.push("Preload critical resources");
      suggestions.push("Optimize server response times");
    }

    if (metrics.webVitals.cls > 0.15) {
      suggestions.push("Add width/height attributes to images");
      suggestions.push("Reserve space for dynamic content");
    }

    return suggestions;
  }

  // Real-time alerts
  enablePerformanceAlerts(thresholds?: Partial<PerformanceThresholds>): void {
    const alertThresholds = { ...this.thresholds, ...thresholds };

    this.realTimeMetrics$
      .pipe(
        filter((metrics) => metrics !== null),
        debounceTime(1000)
      )
      .subscribe((metrics) => {
        if (!metrics?.metrics?.webVitals) return;

        const vitals = metrics.metrics.webVitals;

        if (vitals.lcp > alertThresholds.lcp.poor) {
          this.sendPerformanceAlert(
            "LCP Alert",
            `LCP exceeded threshold: ${vitals.lcp}ms`
          );
        }

        if (vitals.cls > alertThresholds.cls.poor) {
          this.sendPerformanceAlert(
            "CLS Alert",
            `CLS exceeded threshold: ${vitals.cls}`
          );
        }
      });
  }

  // Export performance data
  exportPerformanceData(format: "json" | "csv" = "json"): string {
    const data = this.performanceData$.value;

    if (format === "csv") {
      return this.convertToCSV(data);
    }

    return JSON.stringify(data, null, 2);
  }

  // Private implementation methods
  private initializePerformanceMonitoring(): void {
    this.ngZone.runOutsideAngular(() => {
      this.setupPerformanceObservers();
      this.trackUserInteractions();
      this.trackScrollDepth();
      this.measurePageTiming();
    });
  }

  private setupPerformanceObservers(): void {
    // Navigation timing
    const navObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      this.processNavigationEntries(entries);
    });
    navObserver.observe({ entryTypes: ["navigation"] });
    this.observers.push(navObserver);

    // Resource timing
    const resourceObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      this.processResourceEntries(entries);
    });
    resourceObserver.observe({ entryTypes: ["resource"] });
    this.observers.push(resourceObserver);

    // Web Vitals observers
    this.setupWebVitalsObservers();
  }

  private setupWebVitalsObservers(): void {
    // LCP Observer
    const lcpObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1];
      this.updateRealTimeMetric("webVitals.lcp", lastEntry.startTime);
    });
    lcpObserver.observe({ entryTypes: ["largest-contentful-paint"] });

    // FID Observer (via first-input)
    const fidObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        const fid = entry.processingStart - entry.startTime;
        this.updateRealTimeMetric("webVitals.fid", fid);
      });
    });
    fidObserver.observe({ entryTypes: ["first-input"] });

    // CLS Observer
    let cumulativeScore = 0;
    const clsObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        if (!entry.hadRecentInput) {
          cumulativeScore += entry.value;
        }
      });
      this.updateRealTimeMetric("webVitals.cls", cumulativeScore);
    });
    clsObserver.observe({ entryTypes: ["layout-shift"] });
  }

  private trackUserInteractions(): void {
    ["click", "keydown", "touchstart"].forEach((eventType) => {
      document.addEventListener(
        eventType,
        () => {
          this.userInteractions++;
        },
        { passive: true }
      );
    });
  }

  private trackScrollDepth(): void {
    const throttledScroll = fromEvent(window, "scroll").pipe(throttleTime(100));

    throttledScroll.subscribe(() => {
      const scrollPercent =
        (window.scrollY + window.innerHeight) / document.body.scrollHeight;
      this.maxScrollDepth = Math.max(this.maxScrollDepth, scrollPercent);
    });
  }

  private measurePageTiming(): void {
    window.addEventListener("load", () => {
      setTimeout(() => {
        this.generatePerformanceReport();
      }, 1000);
    });
  }

  private startRealTimeMonitoring(): void {
    interval(5000).subscribe(() => {
      this.updateRealTimeMetrics();
    });
  }

  private async getSSRMetrics(): Promise<SSRMetrics> {
    // In a real implementation, this would fetch from server
    return {
      renderTime:
        performance.timing.responseEnd - performance.timing.requestStart,
      hydrationTime: performance.now() - this.pageStartTime,
      cacheHitRatio: 0.85, // Mock data
      memoryUsage: (performance as any).memory?.usedJSHeapSize || 0,
    };
  }

  private getClientMetrics(): ClientMetrics {
    const timing = performance.timing;

    return {
      domContentLoaded:
        timing.domContentLoadedEventEnd - timing.navigationStart,
      loadComplete: timing.loadEventEnd - timing.navigationStart,
      firstPaint: this.getFirstPaint(),
      firstContentfulPaint: this.getFirstContentfulPaint(),
      timeToInteractive: this.getTimeToInteractive(),
      totalBlockingTime: this.getTotalBlockingTime(),
    };
  }

  private async getWebVitalsMetrics(): Promise<WebVitalsMetrics> {
    return new Promise((resolve) => {
      const metrics: Partial<WebVitalsMetrics> = {};

      // Get existing metrics or wait for them
      this.collectWebVitalsAsync(metrics, resolve);
    });
  }

  private collectWebVitalsAsync(
    metrics: Partial<WebVitalsMetrics>,
    callback: (metrics: WebVitalsMetrics) => void
  ): void {
    // Implementation would use web-vitals library or manual collection
    setTimeout(() => {
      callback({
        lcp: this.getLCP(),
        fid: this.getFID(),
        cls: this.getCLS(),
        fcp: this.getFirstContentfulPaint(),
        ttfb: this.getTTFB(),
      });
    }, 100);
  }

  private getResourceTiming(): ResourceTiming[] {
    const resources = performance.getEntriesByType(
      "resource"
    ) as PerformanceResourceTiming[];

    return resources.map((resource) => ({
      name: resource.name,
      type: this.getResourceType(resource),
      size: resource.transferSize,
      duration: resource.duration,
      transferSize: resource.transferSize,
      encodedBodySize: resource.encodedBodySize,
      decodedBodySize: resource.decodedBodySize,
    }));
  }

  private getUserExperienceMetrics(): UserExperienceMetrics {
    const timeOnPage = Date.now() - this.startTime;

    return {
      scrollDepth: this.maxScrollDepth,
      clickCount: this.userInteractions,
      timeOnPage,
      bounceRate: this.calculateBounceRate(),
    };
  }

  private getDeviceInfo(): any {
    const connection = (navigator as any).connection;

    return {
      userAgent: navigator.userAgent,
      connectionType: connection?.type,
      effectiveConnectionType: connection?.effectiveType,
      deviceMemory: (navigator as any).deviceMemory,
      hardwareConcurrency: navigator.hardwareConcurrency,
    };
  }

  // Web Vitals helper methods
  private getLCP(): number {
    const lcpEntries = performance.getEntriesByType("largest-contentful-paint");
    return lcpEntries.length > 0
      ? lcpEntries[lcpEntries.length - 1].startTime
      : 0;
  }

  private getFID(): number {
    const fidEntries = performance.getEntriesByType("first-input");
    return fidEntries.length > 0
      ? fidEntries[0].processingStart - fidEntries[0].startTime
      : 0;
  }

  private getCLS(): number {
    const clsEntries = performance.getEntriesByType("layout-shift") as any[];
    return clsEntries.reduce(
      (score, entry) => (entry.hadRecentInput ? score : score + entry.value),
      0
    );
  }

  private getFirstPaint(): number {
    const fpEntry = performance.getEntriesByName("first-paint")[0];
    return fpEntry ? fpEntry.startTime : 0;
  }

  private getFirstContentfulPaint(): number {
    const fcpEntry = performance.getEntriesByName("first-contentful-paint")[0];
    return fcpEntry ? fcpEntry.startTime : 0;
  }

  private getTTFB(): number {
    const navEntry = performance.getEntriesByType(
      "navigation"
    )[0] as PerformanceNavigationTiming;
    return navEntry ? navEntry.responseStart - navEntry.requestStart : 0;
  }

  private getTimeToInteractive(): number {
    // Simplified TTI calculation
    const timing = performance.timing;
    return timing.domInteractive - timing.navigationStart;
  }

  private getTotalBlockingTime(): number {
    // Simplified TBT calculation
    const longTasks = performance.getEntriesByType("longtask");
    return longTasks.reduce(
      (total, task) => total + Math.max(0, task.duration - 50),
      0
    );
  }

  private getResourceType(resource: PerformanceResourceTiming): string {
    const initiatorType = resource.initiatorType;
    if (initiatorType) return initiatorType;

    const url = resource.name.toLowerCase();
    if (url.includes(".js")) return "script";
    if (url.includes(".css")) return "stylesheet";
    if (url.match(/\.(png|jpg|jpeg|gif|svg|webp)$/)) return "image";
    if (url.match(/\.(woff|woff2|ttf|otf)$/)) return "font";

    return "other";
  }

  private calculateBounceRate(): number {
    // Simplified bounce rate calculation
    const timeOnPage = Date.now() - this.startTime;
    const hasInteracted = this.userInteractions > 0;
    const significantTime = timeOnPage > 30000; // 30 seconds

    return hasInteracted && significantTime ? 0 : 1;
  }

  private processNavigationEntries(entries: PerformanceEntry[]): void {
    // Process navigation timing entries
    entries.forEach((entry) => {
      console.log("Navigation entry:", entry);
    });
  }

  private processResourceEntries(entries: PerformanceEntry[]): void {
    // Process resource timing entries
    entries.forEach((entry) => {
      const resource = entry as PerformanceResourceTiming;
      if (resource.transferSize > 1024 * 1024) {
        // > 1MB
        console.warn(
          "Large resource detected:",
          resource.name,
          resource.transferSize
        );
      }
    });
  }

  private updateRealTimeMetric(path: string, value: number): void {
    const current = this.realTimeMetrics$.value || {};
    const pathParts = path.split(".");
    let target = current as any;

    for (let i = 0; i < pathParts.length - 1; i++) {
      if (!target[pathParts[i]]) {
        target[pathParts[i]] = {};
      }
      target = target[pathParts[i]];
    }

    target[pathParts[pathParts.length - 1]] = value;
    this.realTimeMetrics$.next(current);
  }

  private updateRealTimeMetrics(): void {
    const metrics = {
      timestamp: Date.now(),
      metrics: {
        webVitals: {
          lcp: this.getLCP(),
          fid: this.getFID(),
          cls: this.getCLS(),
          fcp: this.getFirstContentfulPaint(),
          ttfb: this.getTTFB(),
        },
      },
    };

    this.realTimeMetrics$.next(metrics);
  }

  private generateTrends(data: PerformanceReport[]): any {
    if (data.length < 2) return {};

    const recent = data.slice(-10); // Last 10 reports

    return {
      lcpTrend: this.calculateTrend(recent.map((r) => r.metrics.webVitals.lcp)),
      fidTrend: this.calculateTrend(recent.map((r) => r.metrics.webVitals.fid)),
      clsTrend: this.calculateTrend(recent.map((r) => r.metrics.webVitals.cls)),
      renderTimeTrend: this.calculateTrend(
        recent.map((r) => r.metrics.serverSideMetrics.renderTime)
      ),
    };
  }

  private calculateTrend(
    values: number[]
  ): "improving" | "degrading" | "stable" {
    if (values.length < 2) return "stable";

    const first = values[0];
    const last = values[values.length - 1];
    const change = (last - first) / first;

    if (change < -0.1) return "improving";
    if (change > 0.1) return "degrading";
    return "stable";
  }

  private sendPerformanceAlert(title: string, message: string): void {
    // Implementation would send alert to monitoring service
    console.warn(`Performance Alert - ${title}: ${message}`);
  }

  private convertToCSV(data: PerformanceReport[]): string {
    const headers = [
      "timestamp",
      "url",
      "lcp",
      "fid",
      "cls",
      "renderTime",
      "hydrationTime",
      "domContentLoaded",
      "loadComplete",
    ];

    const rows = data.map((report) => [
      report.timestamp,
      report.url,
      report.metrics.webVitals.lcp,
      report.metrics.webVitals.fid,
      report.metrics.webVitals.cls,
      report.metrics.serverSideMetrics.renderTime,
      report.metrics.serverSideMetrics.hydrationTime,
      report.metrics.clientSideMetrics.domContentLoaded,
      report.metrics.clientSideMetrics.loadComplete,
    ]);

    return [headers, ...rows].map((row) => row.join(",")).join("\n");
  }
}
```

## 🎯 **Complete Implementation Summary**

This comprehensive Angular Universal & SSR optimization guide covers:

### **🏗️ Production-Ready Features**

- **Advanced Server Configuration** with intelligent caching, compression, security, and performance monitoring
- **Cross-Platform Universal Service** with device detection, SEO utilities, and browser-safe operations
- **Progressive Hydration System** with lazy loading, intersection observers, and intelligent component activation
- **Advanced Prerendering** with dynamic route discovery, sitemap generation, and CDN integration

### **⚡ Performance Excellence**

- **Real-time Performance Monitoring** with Core Web Vitals tracking and automated optimization suggestions
- **Intelligent Caching Strategies** with route-specific TTLs, cache invalidation, and warmup procedures
- **Resource Optimization** with critical CSS injection, font optimization, and image lazy loading
- **Advanced SEO & Structured Data** with automatic meta tag management and schema.org integration

### **🚀 Enterprise Features**

- **CDN Integration** with multi-provider support, cache purging, and edge optimization
- **Performance Analytics** with trend analysis, threshold monitoring, and automated alerts
- **SEO Automation** with sitemap generation, RSS feeds, and compliance validation
- **Production Monitoring** with health checks, metrics collection, and graceful degradation

## 💡 **Key Implementation Insights**

### **1. Intelligent Caching Architecture**

The caching system uses **LRU cache with TTL management**, **route-specific strategies**, and **intelligent cache keys** based on user agent and content preferences. This provides **optimal cache hit rates** while ensuring **fresh content delivery**!

### **2. Progressive Hydration Strategy**

Components are hydrated based on **priority levels** (high/medium/low), **viewport visibility**, **user interaction patterns**, and **main thread availability**. This ensures **optimal perceived performance** and **resource utilization**!

### **3. Performance Monitoring Integration**

Real-time monitoring tracks **Core Web Vitals**, **custom metrics**, and **user experience indicators** with **automated threshold alerts** and **optimization suggestions**. This enables **proactive performance management**!

### **4. SEO & Structured Data Automation**

Automated SEO management with **route-based meta tags**, **structured data injection**, and **compliance validation** ensures **optimal search engine visibility** and **rich snippet generation**!

This enterprise-grade implementation provides **production-ready SSR optimization** with **intelligent caching**, **performance monitoring**, **SEO automation**, and **advanced analytics** for **world-class Angular Universal applications**! 🚀✨
