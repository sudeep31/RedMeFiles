# 📊 Measuring Angular Application Performance

## 🎯 **Question Overview**

_"How do you measure the performance of an Angular application?"_

## 🔍 **Understanding Performance Measurement**

Performance measurement in Angular involves tracking various metrics that impact user experience, from initial load times to runtime responsiveness. Let's explore comprehensive strategies for measuring and optimizing Angular app performance! 📈

## 🛠️ **Built-in Performance Measurement Tools**

### **1. 📈 Angular Performance API**

```typescript
// Performance service for comprehensive metrics collection
@Injectable({
  providedIn: "root",
})
export class AngularPerformanceService {
  private performanceEntries: PerformanceEntry[] = [];
  private customMetrics = new Map<string, number>();

  constructor() {
    this.initializePerformanceMonitoring();
  }

  private initializePerformanceMonitoring() {
    // Monitor navigation timing
    window.addEventListener("load", () => {
      this.captureNavigationMetrics();
    });

    // Monitor resource loading
    this.monitorResourceLoading();

    // Monitor change detection cycles
    this.monitorChangeDetection();
  }

  // Navigation Timing API metrics
  captureNavigationMetrics() {
    const navigation = performance.getEntriesByType(
      "navigation"
    )[0] as PerformanceNavigationTiming;

    const metrics = {
      // Time to First Byte (TTFB)
      ttfb: navigation.responseStart - navigation.requestStart,

      // DOM Content Loaded
      domContentLoaded:
        navigation.domContentLoadedEventEnd - navigation.navigationStart,

      // Full page load
      loadComplete: navigation.loadEventEnd - navigation.navigationStart,

      // DNS lookup time
      dnsLookup: navigation.domainLookupEnd - navigation.domainLookupStart,

      // TCP connection time
      tcpConnection: navigation.connectEnd - navigation.connectStart,

      // Server response time
      serverResponse: navigation.responseEnd - navigation.responseStart,

      // DOM processing time
      domProcessing: navigation.domComplete - navigation.domLoading,
    };

    console.table(metrics);
    this.storeMetrics("navigation", metrics);
    return metrics;
  }

  // Resource loading performance
  monitorResourceLoading() {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        if (entry.entryType === "resource") {
          const resource = entry as PerformanceResourceTiming;

          // Track slow resources
          if (resource.duration > 1000) {
            console.warn(
              `🐌 Slow resource: ${resource.name} took ${resource.duration}ms`
            );
          }

          this.analyzeResourceTiming(resource);
        }
      });
    });

    observer.observe({ entryTypes: ["resource"] });
  }

  private analyzeResourceTiming(resource: PerformanceResourceTiming) {
    const analysis = {
      name: resource.name,
      duration: resource.duration,
      size: resource.transferSize,
      cached: resource.transferSize === 0 && resource.decodedBodySize > 0,
      initiatorType: resource.initiatorType,
      // Calculate resource efficiency
      efficiency: resource.decodedBodySize / resource.duration,
    };

    // Store resource metrics by type
    const resourceType = this.getResourceType(resource.name);
    this.storeResourceMetric(resourceType, analysis);
  }

  private getResourceType(url: string): string {
    if (url.match(/\.(js|ts)$/i)) return "javascript";
    if (url.match(/\.(css|scss|sass)$/i)) return "stylesheet";
    if (url.match(/\.(jpg|jpeg|png|gif|webp|svg)$/i)) return "image";
    if (url.match(/\.(woff|woff2|ttf|otf)$/i)) return "font";
    return "other";
  }

  // Angular-specific performance monitoring
  monitorChangeDetection() {
    let changeDetectionCount = 0;
    let totalChangeDetectionTime = 0;

    // Hook into NgZone for change detection monitoring
    const originalRun = Zone.current.run;
    Zone.current.run = function (
      fn: Function,
      applyThis?: any,
      applyArgs?: any[]
    ) {
      const start = performance.now();
      changeDetectionCount++;

      const result = originalRun.call(this, fn, applyThis, applyArgs);

      const duration = performance.now() - start;
      totalChangeDetectionTime += duration;

      if (duration > 16.67) {
        // Slower than 60fps
        console.warn(
          `🔄 Slow change detection cycle: ${duration.toFixed(2)}ms`
        );
      }

      return result;
    };

    // Report change detection metrics periodically
    setInterval(() => {
      if (changeDetectionCount > 0) {
        const avgTime = totalChangeDetectionTime / changeDetectionCount;
        console.log(`📊 Change Detection Stats:`, {
          cycles: changeDetectionCount,
          averageTime: `${avgTime.toFixed(2)}ms`,
          totalTime: `${totalChangeDetectionTime.toFixed(2)}ms`,
        });

        // Reset counters
        changeDetectionCount = 0;
        totalChangeDetectionTime = 0;
      }
    }, 10000); // Report every 10 seconds
  }

  // Core Web Vitals measurement
  measureCoreWebVitals() {
    return new Promise<CoreWebVitals>((resolve) => {
      const metrics: Partial<CoreWebVitals> = {};

      // Largest Contentful Paint (LCP)
      new PerformanceObserver((list) => {
        const entries = list.getEntries();
        const lastEntry = entries[entries.length - 1];
        metrics.lcp = lastEntry.startTime;
        console.log(`🖼️ LCP: ${metrics.lcp}ms`);
      }).observe({ entryTypes: ["largest-contentful-paint"] });

      // First Input Delay (FID)
      new PerformanceObserver((list) => {
        list.getEntries().forEach((entry: any) => {
          metrics.fid = entry.processingStart - entry.startTime;
          console.log(`⚡ FID: ${metrics.fid}ms`);
        });
      }).observe({ entryTypes: ["first-input"] });

      // Cumulative Layout Shift (CLS)
      let clsValue = 0;
      new PerformanceObserver((list) => {
        list.getEntries().forEach((entry: any) => {
          if (!entry.hadRecentInput) {
            clsValue += entry.value;
          }
        });
        metrics.cls = clsValue;
        console.log(`📐 CLS: ${metrics.cls}`);
      }).observe({ entryTypes: ["layout-shift"] });

      // First Contentful Paint (FCP)
      new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          metrics.fcp = entry.startTime;
          console.log(`🎨 FCP: ${metrics.fcp}ms`);
        });
      }).observe({ entryTypes: ["paint"] });

      // Return metrics after a delay to capture all values
      setTimeout(() => {
        resolve(metrics as CoreWebVitals);
      }, 5000);
    });
  }

  // Custom performance markers
  startMeasure(name: string) {
    performance.mark(`${name}-start`);
  }

  endMeasure(name: string) {
    performance.mark(`${name}-end`);
    performance.measure(name, `${name}-start`, `${name}-end`);

    const measure = performance.getEntriesByName(name, "measure")[0];
    this.customMetrics.set(name, measure.duration);

    console.log(`⏱️ ${name}: ${measure.duration.toFixed(2)}ms`);
    return measure.duration;
  }

  // Bundle size analysis
  analyzeBundleSize() {
    const scripts = Array.from(document.querySelectorAll("script[src]"));
    const stylesheets = Array.from(
      document.querySelectorAll('link[rel="stylesheet"]')
    );

    const bundleAnalysis = {
      scripts: scripts.map((script) => ({
        src: script.getAttribute("src"),
        size: this.getResourceSize(script.getAttribute("src") || ""),
      })),
      stylesheets: stylesheets.map((link) => ({
        href: link.getAttribute("href"),
        size: this.getResourceSize(link.getAttribute("href") || ""),
      })),
    };

    console.table(bundleAnalysis);
    return bundleAnalysis;
  }

  private getResourceSize(url: string): number {
    const resource = performance.getEntriesByName(
      url
    )[0] as PerformanceResourceTiming;
    return resource ? resource.transferSize : 0;
  }

  // Memory usage tracking
  trackMemoryUsage() {
    if ("memory" in performance) {
      const memory = (performance as any).memory;
      const memoryInfo = {
        used: `${(memory.usedJSHeapSize / 1024 / 1024).toFixed(2)} MB`,
        total: `${(memory.totalJSHeapSize / 1024 / 1024).toFixed(2)} MB`,
        limit: `${(memory.jsHeapSizeLimit / 1024 / 1024).toFixed(2)} MB`,
        usage: `${(
          (memory.usedJSHeapSize / memory.jsHeapSizeLimit) *
          100
        ).toFixed(1)}%`,
      };

      console.log("🧠 Memory Usage:", memoryInfo);
      return memoryInfo;
    }

    console.warn("Memory API not available");
    return null;
  }

  // Generate performance report
  generateReport(): PerformanceReport {
    const coreWebVitals = this.measureCoreWebVitals();
    const memoryInfo = this.trackMemoryUsage();
    const bundleInfo = this.analyzeBundleSize();

    return {
      timestamp: new Date().toISOString(),
      navigationMetrics: this.captureNavigationMetrics(),
      coreWebVitals,
      memoryUsage: memoryInfo,
      bundleAnalysis: bundleInfo,
      customMetrics: Object.fromEntries(this.customMetrics),
      recommendations: this.generateRecommendations(),
    };
  }

  private generateRecommendations(): string[] {
    const recommendations: string[] = [];
    const navigationMetrics = this.captureNavigationMetrics();

    if (navigationMetrics.ttfb > 200) {
      recommendations.push(
        "🚀 Consider server-side rendering or caching to improve TTFB"
      );
    }

    if (navigationMetrics.domContentLoaded > 1500) {
      recommendations.push(
        "📦 Consider code splitting to reduce initial bundle size"
      );
    }

    const memoryInfo = this.trackMemoryUsage();
    if (memoryInfo && parseFloat(memoryInfo.usage) > 80) {
      recommendations.push(
        "🧠 High memory usage detected - check for memory leaks"
      );
    }

    return recommendations;
  }

  private storeMetrics(type: string, metrics: any) {
    // Store in local storage or send to analytics
    localStorage.setItem(
      `performance_${type}`,
      JSON.stringify({
        ...metrics,
        timestamp: Date.now(),
      })
    );
  }

  private storeResourceMetric(type: string, metric: any) {
    const existing = JSON.parse(
      localStorage.getItem(`resources_${type}`) || "[]"
    );
    existing.push(metric);
    localStorage.setItem(`resources_${type}`, JSON.stringify(existing));
  }
}

interface CoreWebVitals {
  lcp: number; // Largest Contentful Paint
  fid: number; // First Input Delay
  cls: number; // Cumulative Layout Shift
  fcp: number; // First Contentful Paint
}

interface PerformanceReport {
  timestamp: string;
  navigationMetrics: any;
  coreWebVitals: Promise<CoreWebVitals>;
  memoryUsage: any;
  bundleAnalysis: any;
  customMetrics: Record<string, number>;
  recommendations: string[];
}
```

### **2. 🎛️ Component Performance Monitoring**

```typescript
// Advanced component performance decorator
export function PerformanceMonitor(options: PerformanceMonitorOptions = {}) {
  return function <T extends { new (...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      private performanceData = {
        initTime: 0,
        renderCount: 0,
        totalRenderTime: 0,
        slowRenders: 0,
        memorySnapshots: [],
      };

      constructor(...args: any[]) {
        const initStart = performance.now();
        super(...args);
        this.performanceData.initTime = performance.now() - initStart;

        console.log(
          `🏗️ ${
            constructor.name
          } initialized in ${this.performanceData.initTime.toFixed(2)}ms`
        );
      }

      ngOnInit() {
        const renderStart = performance.now();

        if (super.ngOnInit) {
          super.ngOnInit();
        }

        const renderTime = performance.now() - renderStart;
        this.recordRender(renderTime);
      }

      ngDoCheck() {
        const checkStart = performance.now();

        if (super.ngDoCheck) {
          super.ngDoCheck();
        }

        const checkTime = performance.now() - checkStart;
        this.recordRender(checkTime);

        // Track memory if enabled
        if (options.trackMemory) {
          this.trackMemoryUsage();
        }
      }

      ngOnDestroy() {
        if (super.ngOnDestroy) {
          super.ngOnDestroy();
        }

        this.reportPerformanceData();
      }

      private recordRender(duration: number) {
        this.performanceData.renderCount++;
        this.performanceData.totalRenderTime += duration;

        // Track slow renders
        const threshold = options.slowRenderThreshold || 16.67; // 60fps
        if (duration > threshold) {
          this.performanceData.slowRenders++;

          if (options.logSlowRenders !== false) {
            console.warn(
              `🐌 Slow render in ${constructor.name}: ${duration.toFixed(2)}ms`
            );
          }
        }
      }

      private trackMemoryUsage() {
        if ("memory" in performance) {
          const memory = (performance as any).memory;
          this.performanceData.memorySnapshots.push({
            timestamp: Date.now(),
            used: memory.usedJSHeapSize,
            total: memory.totalJSHeapSize,
          });
        }
      }

      private reportPerformanceData() {
        const avgRenderTime =
          this.performanceData.totalRenderTime /
          this.performanceData.renderCount;
        const slowRenderPercentage =
          (this.performanceData.slowRenders /
            this.performanceData.renderCount) *
          100;

        const report = {
          component: constructor.name,
          initTime: `${this.performanceData.initTime.toFixed(2)}ms`,
          totalRenders: this.performanceData.renderCount,
          averageRenderTime: `${avgRenderTime.toFixed(2)}ms`,
          slowRenders: this.performanceData.slowRenders,
          slowRenderPercentage: `${slowRenderPercentage.toFixed(1)}%`,
          memorySnapshots: this.performanceData.memorySnapshots.length,
        };

        if (options.generateReport !== false) {
          console.table(report);
        }

        // Send to analytics if configured
        if (options.analyticsCallback) {
          options.analyticsCallback(report);
        }
      }
    };
  };
}

interface PerformanceMonitorOptions {
  slowRenderThreshold?: number;
  trackMemory?: boolean;
  logSlowRenders?: boolean;
  generateReport?: boolean;
  analyticsCallback?: (data: any) => void;
}

// Usage example
@PerformanceMonitor({
  slowRenderThreshold: 10,
  trackMemory: true,
  analyticsCallback: (data) => {
    // Send performance data to analytics service
    console.log("📊 Sending performance data to analytics:", data);
  },
})
@Component({
  selector: "app-monitored-component",
  template: `
    <div>
      <h3>Monitored Component</h3>
      <p>This component tracks its performance automatically</p>
    </div>
  `,
})
export class MonitoredComponent implements OnInit, DoCheck, OnDestroy {
  ngOnInit() {}
  ngDoCheck() {}
  ngOnDestroy() {}
}
```

### **3. 🔧 Bundle Analysis Service**

```typescript
// Service for analyzing bundle size and dependencies
@Injectable({
  providedIn: "root",
})
export class BundleAnalysisService {
  private bundleData = {
    chunks: [],
    dependencies: [],
    duplicates: [],
    unusedExports: [],
  };

  analyzeBundleComposition() {
    const scripts = Array.from(document.querySelectorAll("script[src]"));
    const analysis = {
      mainBundle: this.analyzeMainBundle(scripts),
      vendorBundle: this.analyzeVendorBundle(scripts),
      lazyChunks: this.analyzeLazyChunks(scripts),
      totalSize: 0,
    };

    analysis.totalSize =
      analysis.mainBundle.size +
      analysis.vendorBundle.size +
      analysis.lazyChunks.reduce((sum, chunk) => sum + chunk.size, 0);

    console.table(analysis);
    return analysis;
  }

  private analyzeMainBundle(scripts: Element[]) {
    const mainScript = scripts.find((script) =>
      script.getAttribute("src")?.includes("main")
    );

    if (mainScript) {
      const src = mainScript.getAttribute("src")!;
      const resource = performance.getEntriesByName(
        src
      )[0] as PerformanceResourceTiming;

      return {
        name: "main",
        src,
        size: resource?.transferSize || 0,
        loadTime: resource?.duration || 0,
      };
    }

    return { name: "main", src: "", size: 0, loadTime: 0 };
  }

  private analyzeVendorBundle(scripts: Element[]) {
    const vendorScript = scripts.find(
      (script) =>
        script.getAttribute("src")?.includes("vendor") ||
        script.getAttribute("src")?.includes("polyfills")
    );

    if (vendorScript) {
      const src = vendorScript.getAttribute("src")!;
      const resource = performance.getEntriesByName(
        src
      )[0] as PerformanceResourceTiming;

      return {
        name: "vendor",
        src,
        size: resource?.transferSize || 0,
        loadTime: resource?.duration || 0,
      };
    }

    return { name: "vendor", src: "", size: 0, loadTime: 0 };
  }

  private analyzeLazyChunks(scripts: Element[]) {
    return scripts
      .filter((script) => {
        const src = script.getAttribute("src");
        return src && /\d+\..*\.js$/.test(src); // Lazy chunk pattern
      })
      .map((script) => {
        const src = script.getAttribute("src")!;
        const resource = performance.getEntriesByName(
          src
        )[0] as PerformanceResourceTiming;

        return {
          src,
          size: resource?.transferSize || 0,
          loadTime: resource?.duration || 0,
        };
      });
  }

  // Tree-shaking analysis (simulated)
  analyzeTreeShaking() {
    const analysis = {
      deadCodeEliminated: true,
      unusedModules: [
        // This would be populated by actual bundle analysis
        { module: "unused-utility", size: "2.3kB" },
        { module: "old-component", size: "5.7kB" },
      ],
      sideEffects: [
        { module: "@angular/platform-browser", reason: "DOM manipulation" },
        { module: "zone.js", reason: "Global patching" },
      ],
      recommendations: [
        "Consider using barrel exports carefully to avoid unused imports",
        "Review and remove unused dependencies",
        "Use dynamic imports for heavy libraries",
      ],
    };

    console.log("🌳 Tree Shaking Analysis:", analysis);
    return analysis;
  }

  // Performance budget checking
  checkPerformanceBudgets(budgets: PerformanceBudgets) {
    const currentMetrics = this.analyzeBundleComposition();
    const results = {
      mainBundle: this.checkBudget(
        currentMetrics.mainBundle.size,
        budgets.mainBundle
      ),
      vendorBundle: this.checkBudget(
        currentMetrics.vendorBundle.size,
        budgets.vendorBundle
      ),
      totalSize: this.checkBudget(currentMetrics.totalSize, budgets.totalSize),
      lazyChunks: currentMetrics.lazyChunks.map((chunk) =>
        this.checkBudget(chunk.size, budgets.lazyChunk)
      ),
    };

    const violations = this.getBudgetViolations(results);
    if (violations.length > 0) {
      console.warn("💸 Performance budget violations:", violations);
    } else {
      console.log("✅ All performance budgets met!");
    }

    return results;
  }

  private checkBudget(actual: number, budget: number) {
    const percentage = (actual / budget) * 100;
    return {
      actual,
      budget,
      percentage: percentage.toFixed(1),
      status: actual <= budget ? "pass" : "fail",
      overBy: actual > budget ? actual - budget : 0,
    };
  }

  private getBudgetViolations(results: any) {
    const violations = [];

    for (const [key, result] of Object.entries(results)) {
      if (Array.isArray(result)) {
        result.forEach((r: any, index) => {
          if (r.status === "fail") {
            violations.push(`${key}[${index}]: ${r.overBy} bytes over budget`);
          }
        });
      } else if ((result as any).status === "fail") {
        violations.push(`${key}: ${(result as any).overBy} bytes over budget`);
      }
    }

    return violations;
  }
}

interface PerformanceBudgets {
  mainBundle: number; // bytes
  vendorBundle: number; // bytes
  lazyChunk: number; // bytes
  totalSize: number; // bytes
}
```

## 🔍 **Third-Party Performance Tools Integration**

### **1. 🎯 Lighthouse Integration**

```typescript
// Service for running Lighthouse programmatically
@Injectable({
  providedIn: "root",
})
export class LighthouseService {
  async runLighthouseAudit(): Promise<LighthouseReport> {
    // Note: This requires lighthouse to be available in the environment
    try {
      const lighthouse = await import("lighthouse");
      const chromeLauncher = await import("chrome-launcher");

      const chrome = await chromeLauncher.launch({
        chromeFlags: ["--headless"],
      });

      const options = {
        logLevel: "info",
        output: "json",
        onlyCategories: ["performance"],
        port: chrome.port,
      };

      const runnerResult = await lighthouse.lighthouse(
        window.location.href,
        options
      );
      await chrome.kill();

      const report = runnerResult?.report;
      const scores = JSON.parse(report).categories;

      return {
        performanceScore: scores.performance.score * 100,
        firstContentfulPaint: this.extractMetric(
          report,
          "first-contentful-paint"
        ),
        largestContentfulPaint: this.extractMetric(
          report,
          "largest-contentful-paint"
        ),
        speedIndex: this.extractMetric(report, "speed-index"),
        cumulativeLayoutShift: this.extractMetric(
          report,
          "cumulative-layout-shift"
        ),
        opportunities: this.extractOpportunities(report),
      };
    } catch (error) {
      console.error("Failed to run Lighthouse audit:", error);
      throw error;
    }
  }

  private extractMetric(report: string, metricId: string) {
    const parsed = JSON.parse(report);
    const audit = parsed.audits[metricId];
    return {
      score: audit.score,
      displayValue: audit.displayValue,
      numericValue: audit.numericValue,
    };
  }

  private extractOpportunities(report: string) {
    const parsed = JSON.parse(report);
    return Object.values(parsed.audits)
      .filter((audit: any) => audit.details?.type === "opportunity")
      .map((audit: any) => ({
        id: audit.id,
        title: audit.title,
        description: audit.description,
        score: audit.score,
        displayValue: audit.displayValue,
      }));
  }

  // Client-side performance API equivalent
  measureWebVitals(): Promise<WebVitalsMetrics> {
    return new Promise((resolve) => {
      const metrics: Partial<WebVitalsMetrics> = {};

      // Use web-vitals library if available
      import("web-vitals")
        .then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
          getCLS((metric) => {
            metrics.cls = metric.value;
          });

          getFID((metric) => {
            metrics.fid = metric.value;
          });

          getFCP((metric) => {
            metrics.fcp = metric.value;
          });

          getLCP((metric) => {
            metrics.lcp = metric.value;
          });

          getTTFB((metric) => {
            metrics.ttfb = metric.value;
          });

          // Resolve after collecting metrics
          setTimeout(() => resolve(metrics as WebVitalsMetrics), 3000);
        })
        .catch(() => {
          // Fallback to manual measurement
          resolve(this.measureWebVitalsManually());
        });
    });
  }

  private measureWebVitalsManually(): WebVitalsMetrics {
    // Manual implementation using Performance API
    const navigation = performance.getEntriesByType(
      "navigation"
    )[0] as PerformanceNavigationTiming;

    return {
      fcp: this.getFirstContentfulPaint(),
      lcp: this.getLargestContentfulPaint(),
      fid: 0, // Requires user interaction
      cls: this.getCumulativeLayoutShift(),
      ttfb: navigation.responseStart - navigation.requestStart,
    };
  }

  private getFirstContentfulPaint(): number {
    const paintEntries = performance.getEntriesByType("paint");
    const fcpEntry = paintEntries.find(
      (entry) => entry.name === "first-contentful-paint"
    );
    return fcpEntry?.startTime || 0;
  }

  private getLargestContentfulPaint(): number {
    return new Promise<number>((resolve) => {
      let lcp = 0;

      new PerformanceObserver((list) => {
        const entries = list.getEntries();
        const lastEntry = entries[entries.length - 1];
        lcp = lastEntry.startTime;
      }).observe({ entryTypes: ["largest-contentful-paint"] });

      setTimeout(() => resolve(lcp), 2000);
    }) as any;
  }

  private getCumulativeLayoutShift(): number {
    let clsValue = 0;

    new PerformanceObserver((list) => {
      list.getEntries().forEach((entry: any) => {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      });
    }).observe({ entryTypes: ["layout-shift"] });

    return clsValue;
  }
}

interface LighthouseReport {
  performanceScore: number;
  firstContentfulPaint: any;
  largestContentfulPaint: any;
  speedIndex: any;
  cumulativeLayoutShift: any;
  opportunities: any[];
}

interface WebVitalsMetrics {
  fcp: number; // First Contentful Paint
  lcp: number; // Largest Contentful Paint
  fid: number; // First Input Delay
  cls: number; // Cumulative Layout Shift
  ttfb: number; // Time to First Byte
}
```

### **2. 📊 Real User Monitoring (RUM)**

```typescript
// Real User Monitoring service for production metrics
@Injectable({
  providedIn: "root",
})
export class RealUserMonitoringService {
  private sessionId = this.generateSessionId();
  private pageLoadStartTime = Date.now();
  private userInteractions: UserInteraction[] = [];

  constructor(@Inject(DOCUMENT) private document: Document) {
    this.initializeRUM();
  }

  private initializeRUM() {
    // Track page visibility changes
    this.trackPageVisibility();

    // Track user interactions
    this.trackUserInteractions();

    // Track errors
    this.trackErrors();

    // Send data periodically
    this.scheduleDataSending();
  }

  private trackPageVisibility() {
    let startTime = Date.now();

    const handleVisibilityChange = () => {
      if (this.document.hidden) {
        // Page became hidden
        const visibleTime = Date.now() - startTime;
        this.sendMetric("page-visibility", {
          sessionId: this.sessionId,
          visibleTime,
          timestamp: Date.now(),
        });
      } else {
        // Page became visible
        startTime = Date.now();
      }
    };

    this.document.addEventListener("visibilitychange", handleVisibilityChange);
  }

  private trackUserInteractions() {
    const interactionTypes = ["click", "scroll", "keypress", "input"];

    interactionTypes.forEach((type) => {
      this.document.addEventListener(
        type,
        (event) => {
          const interaction: UserInteraction = {
            type,
            timestamp: Date.now(),
            target: this.getElementSelector(event.target as Element),
            sessionId: this.sessionId,
          };

          this.userInteractions.push(interaction);

          // Limit stored interactions to prevent memory issues
          if (this.userInteractions.length > 100) {
            this.userInteractions = this.userInteractions.slice(-50);
          }
        },
        { passive: true }
      );
    });
  }

  private trackErrors() {
    // JavaScript errors
    window.addEventListener("error", (event) => {
      this.sendMetric("javascript-error", {
        sessionId: this.sessionId,
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        stack: event.error?.stack,
        timestamp: Date.now(),
        url: window.location.href,
      });
    });

    // Unhandled promise rejections
    window.addEventListener("unhandledrejection", (event) => {
      this.sendMetric("unhandled-rejection", {
        sessionId: this.sessionId,
        reason: event.reason,
        timestamp: Date.now(),
        url: window.location.href,
      });
    });

    // Resource loading errors
    this.document.addEventListener(
      "error",
      (event) => {
        if (event.target !== window) {
          const target = event.target as Element;
          this.sendMetric("resource-error", {
            sessionId: this.sessionId,
            tagName: target.tagName,
            src: target.getAttribute("src") || target.getAttribute("href"),
            timestamp: Date.now(),
          });
        }
      },
      true
    );
  }

  // Track custom performance metrics
  trackCustomMetric(
    name: string,
    value: number,
    labels: Record<string, string> = {}
  ) {
    this.sendMetric("custom-metric", {
      sessionId: this.sessionId,
      name,
      value,
      labels,
      timestamp: Date.now(),
      url: window.location.href,
    });
  }

  // Track route changes
  trackRouteChange(from: string, to: string) {
    const routeChangeTime = Date.now();

    this.sendMetric("route-change", {
      sessionId: this.sessionId,
      fromRoute: from,
      toRoute: to,
      changeTime: routeChangeTime,
      timestamp: Date.now(),
    });

    // Measure route transition time
    setTimeout(() => {
      const transitionTime = Date.now() - routeChangeTime;
      this.trackCustomMetric("route-transition-time", transitionTime, {
        fromRoute: from,
        toRoute: to,
      });
    }, 100);
  }

  // Track component performance
  trackComponentPerformance(
    componentName: string,
    operation: string,
    duration: number
  ) {
    this.sendMetric("component-performance", {
      sessionId: this.sessionId,
      componentName,
      operation,
      duration,
      timestamp: Date.now(),
      url: window.location.href,
    });
  }

  // Generate session report
  generateSessionReport(): SessionReport {
    const sessionDuration = Date.now() - this.pageLoadStartTime;
    const performanceMetrics = this.getPerformanceMetrics();

    return {
      sessionId: this.sessionId,
      duration: sessionDuration,
      interactions: this.userInteractions.length,
      url: window.location.href,
      userAgent: navigator.userAgent,
      timestamp: Date.now(),
      performanceMetrics,
      deviceInfo: this.getDeviceInfo(),
    };
  }

  private getPerformanceMetrics() {
    const navigation = performance.getEntriesByType(
      "navigation"
    )[0] as PerformanceNavigationTiming;

    return {
      domContentLoaded:
        navigation.domContentLoadedEventEnd - navigation.navigationStart,
      loadComplete: navigation.loadEventEnd - navigation.navigationStart,
      firstPaint: this.getFirstPaint(),
      firstContentfulPaint: this.getFirstContentfulPaint(),
      memoryUsage: this.getMemoryUsage(),
    };
  }

  private getFirstPaint(): number {
    const paintEntries = performance.getEntriesByType("paint");
    const fpEntry = paintEntries.find((entry) => entry.name === "first-paint");
    return fpEntry?.startTime || 0;
  }

  private getFirstContentfulPaint(): number {
    const paintEntries = performance.getEntriesByType("paint");
    const fcpEntry = paintEntries.find(
      (entry) => entry.name === "first-contentful-paint"
    );
    return fcpEntry?.startTime || 0;
  }

  private getMemoryUsage() {
    if ("memory" in performance) {
      const memory = (performance as any).memory;
      return {
        used: memory.usedJSHeapSize,
        total: memory.totalJSHeapSize,
        limit: memory.jsHeapSizeLimit,
      };
    }
    return null;
  }

  private getDeviceInfo() {
    return {
      userAgent: navigator.userAgent,
      platform: navigator.platform,
      language: navigator.language,
      cookieEnabled: navigator.cookieEnabled,
      onLine: navigator.onLine,
      screenResolution: `${screen.width}x${screen.height}`,
      viewport: `${window.innerWidth}x${window.innerHeight}`,
      devicePixelRatio: window.devicePixelRatio,
    };
  }

  private scheduleDataSending() {
    // Send data every 30 seconds
    setInterval(() => {
      if (this.userInteractions.length > 0) {
        const report = this.generateSessionReport();
        this.sendReport(report);
      }
    }, 30000);

    // Send data before page unload
    window.addEventListener("beforeunload", () => {
      const report = this.generateSessionReport();
      this.sendReport(report, true); // Use sendBeacon for reliability
    });
  }

  private sendMetric(type: string, data: any) {
    // Send individual metrics to analytics endpoint
    if (navigator.sendBeacon) {
      navigator.sendBeacon("/api/metrics", JSON.stringify({ type, data }));
    } else {
      // Fallback for older browsers
      fetch("/api/metrics", {
        method: "POST",
        body: JSON.stringify({ type, data }),
        headers: { "Content-Type": "application/json" },
        keepalive: true,
      }).catch((error) => {
        console.warn("Failed to send metric:", error);
      });
    }
  }

  private sendReport(report: SessionReport, useBeacon = false) {
    const payload = JSON.stringify(report);

    if (useBeacon && navigator.sendBeacon) {
      navigator.sendBeacon("/api/rum-report", payload);
    } else {
      fetch("/api/rum-report", {
        method: "POST",
        body: payload,
        headers: { "Content-Type": "application/json" },
        keepalive: true,
      }).catch((error) => {
        console.warn("Failed to send RUM report:", error);
      });
    }
  }

  private generateSessionId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }

  private getElementSelector(element: Element): string {
    if (!element) return "";

    if (element.id) {
      return `#${element.id}`;
    }

    if (element.className) {
      return `.${element.className.split(" ")[0]}`;
    }

    return element.tagName.toLowerCase();
  }
}

interface UserInteraction {
  type: string;
  timestamp: number;
  target: string;
  sessionId: string;
}

interface SessionReport {
  sessionId: string;
  duration: number;
  interactions: number;
  url: string;
  userAgent: string;
  timestamp: number;
  performanceMetrics: any;
  deviceInfo: any;
}
```

## 📊 **Performance Dashboard Component**

```typescript
// Component for displaying comprehensive performance metrics
@Component({
  selector: "app-performance-dashboard",
  template: `
    <div class="performance-dashboard">
      <h2>🚀 Angular Performance Dashboard</h2>

      <!-- Core Web Vitals -->
      <div class="metrics-section">
        <h3>🎯 Core Web Vitals</h3>
        <div class="metrics-grid">
          <div
            class="metric-card"
            [class.good]="isGoodLCP()"
            [class.poor]="isPoorLCP()"
          >
            <h4>LCP</h4>
            <span class="value">{{ webVitals.lcp | number : "1.2-2" }}ms</span>
            <span class="label">Largest Contentful Paint</span>
          </div>

          <div
            class="metric-card"
            [class.good]="isGoodFID()"
            [class.poor]="isPoorFID()"
          >
            <h4>FID</h4>
            <span class="value">{{ webVitals.fid | number : "1.2-2" }}ms</span>
            <span class="label">First Input Delay</span>
          </div>

          <div
            class="metric-card"
            [class.good]="isGoodCLS()"
            [class.poor]="isPoorCLS()"
          >
            <h4>CLS</h4>
            <span class="value">{{ webVitals.cls | number : "1.3-3" }}</span>
            <span class="label">Cumulative Layout Shift</span>
          </div>
        </div>
      </div>

      <!-- Bundle Analysis -->
      <div class="metrics-section">
        <h3>📦 Bundle Analysis</h3>
        <div class="bundle-chart">
          <div
            *ngFor="let bundle of bundleData"
            class="bundle-bar"
            [style.width.%]="getBundlePercentage(bundle.size)"
            [attr.title]="
              bundle.name +
              ': ' +
              (bundle.size / 1024 | number : '1.1-1') +
              'KB'
            "
          >
            {{ bundle.name }}
          </div>
        </div>
      </div>

      <!-- Performance Timeline -->
      <div class="metrics-section">
        <h3>⏱️ Performance Timeline</h3>
        <div class="timeline">
          <div
            *ngFor="let event of performanceTimeline"
            class="timeline-event"
            [style.left.%]="getTimelinePosition(event.timestamp)"
          >
            <div class="event-marker" [class]="event.type"></div>
            <span class="event-label">{{ event.name }}</span>
          </div>
        </div>
      </div>

      <!-- Memory Usage -->
      <div class="metrics-section" *ngIf="memoryInfo">
        <h3>🧠 Memory Usage</h3>
        <div class="memory-chart">
          <div class="memory-bar">
            <div
              class="memory-used"
              [style.width.%]="getMemoryUsagePercentage()"
            ></div>
          </div>
          <p>
            {{ memoryInfo.used }} / {{ memoryInfo.total }} ({{
              memoryInfo.usage
            }})
          </p>
        </div>
      </div>

      <!-- Recommendations -->
      <div class="metrics-section" *ngIf="recommendations.length > 0">
        <h3>💡 Performance Recommendations</h3>
        <ul class="recommendations-list">
          <li *ngFor="let recommendation of recommendations">
            {{ recommendation }}
          </li>
        </ul>
      </div>

      <!-- Actions -->
      <div class="actions">
        <button (click)="refreshMetrics()" class="btn primary">
          🔄 Refresh Metrics
        </button>
        <button (click)="runFullAnalysis()" class="btn secondary">
          🔍 Full Analysis
        </button>
        <button (click)="exportReport()" class="btn secondary">
          📄 Export Report
        </button>
      </div>
    </div>
  `,
  styles: [
    `
      .performance-dashboard {
        padding: 20px;
        background: #f5f5f5;
      }

      .metrics-section {
        background: white;
        margin: 20px 0;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      }

      .metrics-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 20px;
        margin-top: 15px;
      }

      .metric-card {
        padding: 20px;
        border-radius: 8px;
        text-align: center;
        border: 2px solid #ddd;
      }

      .metric-card.good {
        border-color: #4caf50;
        background-color: #f1f8e9;
      }

      .metric-card.poor {
        border-color: #f44336;
        background-color: #ffebee;
      }

      .metric-card h4 {
        margin: 0;
        font-size: 1.2em;
      }

      .metric-card .value {
        display: block;
        font-size: 2em;
        font-weight: bold;
        margin: 10px 0;
      }

      .metric-card .label {
        font-size: 0.9em;
        color: #666;
      }

      .bundle-chart {
        margin-top: 15px;
      }

      .bundle-bar {
        height: 30px;
        margin: 5px 0;
        background: linear-gradient(90deg, #007acc, #00a8ff);
        border-radius: 4px;
        display: flex;
        align-items: center;
        padding: 0 10px;
        color: white;
        font-weight: bold;
      }

      .timeline {
        position: relative;
        height: 60px;
        background: #e0e0e0;
        border-radius: 4px;
        margin-top: 15px;
      }

      .timeline-event {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
      }

      .event-marker {
        width: 12px;
        height: 12px;
        border-radius: 50%;
        background: #007acc;
      }

      .event-marker.critical {
        background: #f44336;
      }

      .event-marker.warning {
        background: #ff9800;
      }

      .event-label {
        position: absolute;
        top: -30px;
        left: 50%;
        transform: translateX(-50%);
        font-size: 0.8em;
        white-space: nowrap;
      }

      .memory-chart {
        margin-top: 15px;
      }

      .memory-bar {
        height: 20px;
        background: #e0e0e0;
        border-radius: 10px;
        overflow: hidden;
      }

      .memory-used {
        height: 100%;
        background: linear-gradient(90deg, #4caf50, #8bc34a);
      }

      .recommendations-list {
        margin-top: 15px;
      }

      .recommendations-list li {
        margin: 10px 0;
        padding: 10px;
        background: #fff3e0;
        border-left: 4px solid #ff9800;
      }

      .actions {
        display: flex;
        gap: 10px;
        margin-top: 20px;
      }

      .btn {
        padding: 10px 20px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-weight: bold;
      }

      .btn.primary {
        background: #007acc;
        color: white;
      }

      .btn.secondary {
        background: #6c757d;
        color: white;
      }

      .btn:hover {
        opacity: 0.9;
      }
    `,
  ],
})
export class PerformanceDashboardComponent implements OnInit {
  webVitals = { lcp: 0, fid: 0, cls: 0, fcp: 0, ttfb: 0 };
  bundleData: any[] = [];
  performanceTimeline: any[] = [];
  memoryInfo: any = null;
  recommendations: string[] = [];

  constructor(
    private performanceService: AngularPerformanceService,
    private bundleAnalysisService: BundleAnalysisService,
    private rumService: RealUserMonitoringService
  ) {}

  async ngOnInit() {
    await this.loadAllMetrics();
  }

  private async loadAllMetrics() {
    try {
      // Load Core Web Vitals
      this.webVitals = await this.performanceService.measureCoreWebVitals();

      // Load bundle analysis
      const bundleAnalysis =
        this.bundleAnalysisService.analyzeBundleComposition();
      this.bundleData = [
        { name: "Main", size: bundleAnalysis.mainBundle.size },
        { name: "Vendor", size: bundleAnalysis.vendorBundle.size },
        ...bundleAnalysis.lazyChunks.map((chunk, i) => ({
          name: `Chunk ${i + 1}`,
          size: chunk.size,
        })),
      ];

      // Load memory info
      this.memoryInfo = this.performanceService.trackMemoryUsage();

      // Generate timeline
      this.performanceTimeline = this.generatePerformanceTimeline();

      // Generate recommendations
      this.recommendations = this.generateRecommendations();
    } catch (error) {
      console.error("Failed to load performance metrics:", error);
    }
  }

  // Core Web Vitals thresholds
  isGoodLCP(): boolean {
    return this.webVitals.lcp <= 2500;
  }
  isPoorLCP(): boolean {
    return this.webVitals.lcp > 4000;
  }

  isGoodFID(): boolean {
    return this.webVitals.fid <= 100;
  }
  isPoorFID(): boolean {
    return this.webVitals.fid > 300;
  }

  isGoodCLS(): boolean {
    return this.webVitals.cls <= 0.1;
  }
  isPoorCLS(): boolean {
    return this.webVitals.cls > 0.25;
  }

  getBundlePercentage(size: number): number {
    const total = this.bundleData.reduce((sum, bundle) => sum + bundle.size, 0);
    return (size / total) * 100;
  }

  getTimelinePosition(timestamp: number): number {
    // Calculate position based on navigation timing
    const navigation = performance.getEntriesByType(
      "navigation"
    )[0] as PerformanceNavigationTiming;
    const totalTime = navigation.loadEventEnd - navigation.navigationStart;
    return ((timestamp - navigation.navigationStart) / totalTime) * 100;
  }

  getMemoryUsagePercentage(): number {
    if (!this.memoryInfo) return 0;
    const used = parseFloat(this.memoryInfo.used.replace(" MB", ""));
    const total = parseFloat(this.memoryInfo.total.replace(" MB", ""));
    return (used / total) * 100;
  }

  private generatePerformanceTimeline(): any[] {
    const navigation = performance.getEntriesByType(
      "navigation"
    )[0] as PerformanceNavigationTiming;

    return [
      {
        name: "DNS",
        timestamp: navigation.domainLookupStart,
        type: "info",
      },
      {
        name: "Connect",
        timestamp: navigation.connectStart,
        type: "info",
      },
      {
        name: "Response",
        timestamp: navigation.responseStart,
        type: "info",
      },
      {
        name: "DOM Ready",
        timestamp: navigation.domContentLoadedEventEnd,
        type: "success",
      },
      {
        name: "Load Complete",
        timestamp: navigation.loadEventEnd,
        type: "success",
      },
    ];
  }

  private generateRecommendations(): string[] {
    const recommendations = [];

    if (this.webVitals.lcp > 2500) {
      recommendations.push(
        "🖼️ Optimize images and implement lazy loading to improve LCP"
      );
    }

    if (this.webVitals.fid > 100) {
      recommendations.push(
        "⚡ Reduce JavaScript execution time to improve FID"
      );
    }

    if (this.webVitals.cls > 0.1) {
      recommendations.push(
        "📐 Add size attributes to images and reserve space for dynamic content"
      );
    }

    const totalBundleSize = this.bundleData.reduce(
      (sum, bundle) => sum + bundle.size,
      0
    );
    if (totalBundleSize > 1024 * 1024) {
      // 1MB
      recommendations.push("📦 Consider code splitting to reduce bundle size");
    }

    if (this.memoryInfo) {
      const usage = parseFloat(this.memoryInfo.usage.replace("%", ""));
      if (usage > 80) {
        recommendations.push(
          "🧠 High memory usage detected - check for memory leaks"
        );
      }
    }

    return recommendations;
  }

  async refreshMetrics() {
    console.log("🔄 Refreshing performance metrics...");
    await this.loadAllMetrics();
  }

  async runFullAnalysis() {
    console.log("🔍 Running full performance analysis...");
    // Trigger comprehensive analysis
    const report = this.performanceService.generateReport();
    console.log("📊 Full Performance Report:", report);
  }

  exportReport() {
    const report = {
      timestamp: new Date().toISOString(),
      webVitals: this.webVitals,
      bundleData: this.bundleData,
      memoryInfo: this.memoryInfo,
      recommendations: this.recommendations,
    };

    const blob = new Blob([JSON.stringify(report, null, 2)], {
      type: "application/json",
    });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = `performance-report-${Date.now()}.json`;
    link.click();

    URL.revokeObjectURL(url);
    console.log("📄 Performance report exported");
  }
}
```

## 📊 **Angular Version Comparison**

| Feature               | Angular 15         | Angular 17         | Angular 19             |
| --------------------- | ------------------ | ------------------ | ---------------------- |
| **Performance API**   | Basic support      | Enhanced metrics   | Advanced profiling     |
| **Bundle Analysis**   | Manual tools       | Built-in analysis  | Comprehensive insights |
| **Core Web Vitals**   | Manual measurement | Integrated support | Auto-optimization      |
| **RUM Integration**   | Third-party only   | Framework support  | Native RUM APIs        |
| **Development Tools** | Basic profiler     | Enhanced DevTools  | Advanced debugging     |

## 🎯 **Key Takeaways**

### **📊 Essential Performance Metrics:**

1. **Core Web Vitals** - LCP, FID, CLS for user experience
2. **Bundle Size Analysis** - Monitor and optimize application size
3. **Memory Usage** - Track and prevent memory leaks
4. **Change Detection** - Measure and optimize render cycles
5. **Network Performance** - Monitor resource loading times

### **🛠️ Best Practices:**

1. **Implement Comprehensive Monitoring** - Track all aspects of performance
2. **Use Real User Monitoring** - Gather data from actual users
3. **Set Performance Budgets** - Define and enforce size limits
4. **Regular Performance Audits** - Schedule periodic assessments
5. **Automate Performance Testing** - Include in CI/CD pipelines

### **🚨 Common Pitfalls:**

- Only measuring in development environment
- Ignoring real user metrics
- Not setting performance budgets
- Overlooking memory leaks
- Missing long-term trend analysis

Comprehensive performance measurement is essential for maintaining a fast and responsive Angular application! 🚀
