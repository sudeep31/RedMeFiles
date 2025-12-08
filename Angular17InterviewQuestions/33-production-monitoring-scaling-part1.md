# 🚀 **Production Monitoring & Server Scaling for Angular Apps**

## 🎯 **What You'll Learn**

Master **production-grade monitoring and scaling** for Angular applications! Transform your app from a **single-server setup** to a **globally distributed, self-healing system** that scales automatically and alerts you before problems occur! 📊

---

## 📚 **The Production Challenge (Understanding the Stakes)**

### **Why Production Monitoring Matters? 🔍**

Think of your Angular app in production like a **busy restaurant chain**:

- 🏪 **Single Location** (basic setup) = One server, hope it doesn't crash
- 🌍 **Global Chain** (enterprise setup) = Multiple locations, backup plans, real-time management

**Production Realities:**

- 💥 **Outages Cost Money** - Every minute down = lost revenue
- 🐛 **Silent Failures** - Errors users never report
- 📈 **Traffic Spikes** - Black Friday, viral content, marketing campaigns
- 🌍 **Global Users** - Different time zones, network conditions
- 🔒 **Security Threats** - Constant monitoring needed

---

## 📊 **Frontend Monitoring & Error Tracking**

### **1. 🛡️ Comprehensive Error Monitoring Service**

```typescript
// src/app/core/services/error-monitoring.service.ts
import { Injectable, ErrorHandler } from "@angular/core";
import { HttpErrorResponse } from "@angular/common/http";
import { Observable, Subject, BehaviorSubject, timer } from "rxjs";
import {
  filter,
  debounceTime,
  groupBy,
  mergeMap,
  bufferTime,
} from "rxjs/operators";

export interface ErrorContext {
  userId?: string;
  sessionId: string;
  userAgent: string;
  url: string;
  timestamp: Date;
  buildVersion: string;
  environment: string;
  feature?: string;
  additionalData?: Record<string, any>;
}

export interface MonitoredError {
  id: string;
  type: "javascript" | "http" | "angular" | "custom" | "performance";
  severity: "low" | "medium" | "high" | "critical";
  message: string;
  stack?: string;
  context: ErrorContext;
  fingerprint: string; // For grouping similar errors
  occurrences: number;
  firstSeen: Date;
  lastSeen: Date;
}

@Injectable({
  providedIn: "root",
})
export class ErrorMonitoringService implements ErrorHandler {
  private errors$ = new Subject<MonitoredError>();
  private errorBuffer: MonitoredError[] = [];
  private sessionId = this.generateSessionId();
  private userId: string | undefined;

  // 📊 Real-time metrics
  public readonly errorCount$ = new BehaviorSubject<number>(0);
  public readonly criticalErrors$ = new BehaviorSubject<MonitoredError[]>([]);
  public readonly errorRate$ = new BehaviorSubject<number>(0);

  private config = {
    enableConsoleLogging: true,
    enableRemoteLogging: true,
    batchSize: 10,
    batchTimeout: 5000, // 5 seconds
    maxRetries: 3,
    enablePerformanceMonitoring: true,
    samplingRate: 1.0, // Log 100% of errors (reduce for high-traffic apps)
    endpoints: {
      errors: "/api/monitoring/errors",
      performance: "/api/monitoring/performance",
      userFeedback: "/api/monitoring/feedback",
    },
  };

  constructor() {
    this.initializeErrorTracking();
    this.setupErrorBuffering();
    this.setupPerformanceMonitoring();
    this.setupUnhandledRejectionTracking();
  }

  // 🚨 Angular ErrorHandler Implementation
  handleError(error: any): void {
    this.logError(error, {
      type: "angular",
      severity: this.determineSeverity(error),
      feature: this.getCurrentFeature(),
    });
  }

  // 📝 LOG ERROR (Main Method)
  logError(
    error: Error | HttpErrorResponse | string,
    options: {
      type?: MonitoredError["type"];
      severity?: MonitoredError["severity"];
      feature?: string;
      additionalData?: Record<string, any>;
    } = {}
  ): void {
    try {
      // Sample errors based on configuration
      if (Math.random() > this.config.samplingRate) {
        return;
      }

      const monitoredError = this.createMonitoredError(error, options);

      // Add to buffer for batch processing
      this.errorBuffer.push(monitoredError);
      this.errors$.next(monitoredError);

      // Update real-time metrics
      this.updateMetrics(monitoredError);

      // Log to console in development
      if (this.config.enableConsoleLogging && !this.isProduction()) {
        console.group(
          `🚨 ${monitoredError.type.toUpperCase()} Error - ${monitoredError.severity.toUpperCase()}`
        );
        console.error("Message:", monitoredError.message);
        console.error("Stack:", monitoredError.stack);
        console.error("Context:", monitoredError.context);
        console.error("Full Error:", error);
        console.groupEnd();
      }

      // Handle critical errors immediately
      if (monitoredError.severity === "critical") {
        this.handleCriticalError(monitoredError);
      }
    } catch (loggingError) {
      // Prevent infinite loops in error logging
      console.error("Failed to log error:", loggingError);
    }
  }

  // 🌐 LOG HTTP ERROR
  logHttpError(error: HttpErrorResponse, requestContext?: any): void {
    this.logError(error, {
      type: "http",
      severity: this.determineHttpErrorSeverity(error),
      additionalData: {
        status: error.status,
        statusText: error.statusText,
        url: error.url,
        requestContext,
      },
    });
  }

  // ⚡ LOG PERFORMANCE ISSUE
  logPerformanceIssue(metric: string, value: number, threshold: number): void {
    this.logError(`Performance issue: ${metric}`, {
      type: "performance",
      severity: value > threshold * 2 ? "high" : "medium",
      additionalData: {
        metric,
        value,
        threshold,
        userAgent: navigator.userAgent,
        connectionType: this.getConnectionType(),
      },
    });
  }

  // 🎯 LOG CUSTOM ERROR
  logCustomError(
    message: string,
    severity: MonitoredError["severity"] = "medium",
    additionalData?: Record<string, any>
  ): void {
    this.logError(new Error(message), {
      type: "custom",
      severity,
      additionalData,
    });
  }

  // 👤 SET USER CONTEXT
  setUserContext(userId: string, additionalData?: Record<string, any>): void {
    this.userId = userId;
    console.log(`📊 User context set: ${userId}`);

    // Log user session start
    this.logCustomError("User session started", "low", {
      userId,
      sessionId: this.sessionId,
      ...additionalData,
    });
  }

  // 📊 GET ERROR STATISTICS
  getErrorStatistics(): Observable<{
    totalErrors: number;
    errorsByType: Record<string, number>;
    errorsBySeverity: Record<string, number>;
    errorRate: number;
    topErrors: MonitoredError[];
  }> {
    return new Observable((observer) => {
      const stats = this.calculateErrorStatistics();
      observer.next(stats);
      observer.complete();
    });
  }

  // 🔄 Private Methods

  private initializeErrorTracking(): void {
    // Track global JavaScript errors
    window.addEventListener("error", (event) => {
      this.logError(event.error || event.message, {
        type: "javascript",
        severity: "high",
        additionalData: {
          filename: event.filename,
          lineno: event.lineno,
          colno: event.colno,
        },
      });
    });

    // Track global promise rejections
    window.addEventListener("unhandledrejection", (event) => {
      this.logError(event.reason, {
        type: "javascript",
        severity: "high",
        additionalData: {
          promiseRejection: true,
        },
      });
    });

    console.log("🛡️ Error monitoring initialized");
  }

  private setupErrorBuffering(): void {
    // Batch errors for efficient sending
    this.errors$
      .pipe(
        bufferTime(this.config.batchTimeout, null, this.config.batchSize),
        filter((errors) => errors.length > 0)
      )
      .subscribe((errors) => {
        if (this.config.enableRemoteLogging) {
          this.sendErrorsToServer(errors);
        }
      });
  }

  private setupPerformanceMonitoring(): void {
    if (!this.config.enablePerformanceMonitoring) return;

    // Monitor Core Web Vitals
    this.monitorCoreWebVitals();

    // Monitor resource loading
    this.monitorResourceLoading();

    // Monitor memory usage
    this.monitorMemoryUsage();
  }

  private setupUnhandledRejectionTracking(): void {
    // Track unhandled promise rejections
    window.addEventListener("unhandledrejection", (event) => {
      this.logError(event.reason, {
        type: "javascript",
        severity: "high",
        feature: "promise-rejection",
      });
    });
  }

  private createMonitoredError(
    error: Error | HttpErrorResponse | string,
    options: any
  ): MonitoredError {
    const errorMessage = this.extractErrorMessage(error);
    const stack = error instanceof Error ? error.stack : undefined;

    return {
      id: this.generateErrorId(),
      type: options.type || "javascript",
      severity: options.severity || "medium",
      message: errorMessage,
      stack,
      context: this.createErrorContext(options.feature, options.additionalData),
      fingerprint: this.generateFingerprint(errorMessage, stack),
      occurrences: 1,
      firstSeen: new Date(),
      lastSeen: new Date(),
    };
  }

  private createErrorContext(
    feature?: string,
    additionalData?: Record<string, any>
  ): ErrorContext {
    return {
      userId: this.userId,
      sessionId: this.sessionId,
      userAgent: navigator.userAgent,
      url: window.location.href,
      timestamp: new Date(),
      buildVersion: this.getBuildVersion(),
      environment: this.getEnvironment(),
      feature,
      additionalData,
    };
  }

  private determineSeverity(error: any): MonitoredError["severity"] {
    // HTTP errors
    if (error instanceof HttpErrorResponse) {
      return this.determineHttpErrorSeverity(error);
    }

    // JavaScript errors
    if (error instanceof Error) {
      const message = error.message?.toLowerCase() || "";

      if (message.includes("network") || message.includes("timeout")) {
        return "high";
      }

      if (message.includes("permission") || message.includes("unauthorized")) {
        return "critical";
      }

      if (message.includes("validation") || message.includes("format")) {
        return "medium";
      }
    }

    return "medium";
  }

  private determineHttpErrorSeverity(
    error: HttpErrorResponse
  ): MonitoredError["severity"] {
    if (error.status >= 500) return "critical";
    if (error.status >= 400) return "high";
    if (error.status >= 300) return "medium";
    return "low";
  }

  private updateMetrics(error: MonitoredError): void {
    // Update error count
    this.errorCount$.next(this.errorCount$.value + 1);

    // Update critical errors list
    if (error.severity === "critical") {
      const criticalErrors = this.criticalErrors$.value;
      criticalErrors.unshift(error);

      // Keep only last 10 critical errors
      if (criticalErrors.length > 10) {
        criticalErrors.pop();
      }

      this.criticalErrors$.next(criticalErrors);
    }

    // Calculate error rate (errors per minute)
    this.calculateErrorRate();
  }

  private calculateErrorRate(): void {
    // Calculate errors in the last minute
    const oneMinuteAgo = new Date(Date.now() - 60000);
    const recentErrors = this.errorBuffer.filter(
      (error) => error.lastSeen > oneMinuteAgo
    );

    this.errorRate$.next(recentErrors.length);
  }

  private handleCriticalError(error: MonitoredError): void {
    console.error("🚨 CRITICAL ERROR DETECTED:", error);

    // Send immediately (don't wait for batch)
    if (this.config.enableRemoteLogging) {
      this.sendErrorsToServer([error], true);
    }

    // Could trigger alerts, notifications, etc.
    this.triggerCriticalErrorAlert(error);
  }

  private async sendErrorsToServer(
    errors: MonitoredError[],
    immediate = false
  ): Promise<void> {
    try {
      const response = await fetch(this.config.endpoints.errors, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          errors,
          metadata: {
            sessionId: this.sessionId,
            userId: this.userId,
            timestamp: new Date().toISOString(),
            immediate,
          },
        }),
      });

      if (!response.ok) {
        throw new Error(`Failed to send errors: ${response.statusText}`);
      }

      console.log(`📤 Sent ${errors.length} errors to server`);
    } catch (error) {
      console.error("Failed to send errors to server:", error);

      // Store errors locally for retry
      this.storeErrorsLocally(errors);
    }
  }

  private triggerCriticalErrorAlert(error: MonitoredError): void {
    // This could integrate with notification services
    console.log("🚨 CRITICAL ERROR ALERT TRIGGERED:", error.message);

    // Example: Show user-friendly error message
    // Example: Send to monitoring service like Sentry, DataDog, etc.
    // Example: Trigger PagerDuty alert for on-call engineers
  }

  // 🔧 Utility Methods

  private extractErrorMessage(error: any): string {
    if (typeof error === "string") return error;
    if (error instanceof HttpErrorResponse)
      return `HTTP ${error.status}: ${error.message}`;
    if (error instanceof Error) return error.message;
    return "Unknown error occurred";
  }

  private generateErrorId(): string {
    return `error_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private generateSessionId(): string {
    return `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private generateFingerprint(message: string, stack?: string): string {
    // Create a unique fingerprint for grouping similar errors
    const content = `${message}_${stack?.split("\n")[0] || "no-stack"}`;
    return btoa(content).substr(0, 16);
  }

  private getCurrentFeature(): string {
    // Extract feature name from current route
    const path = window.location.pathname;
    const segments = path.split("/").filter((s) => s);
    return segments[0] || "unknown";
  }

  private getBuildVersion(): string {
    // Get build version from environment or package.json
    return (window as any).__BUILD_VERSION__ || "1.0.0";
  }

  private getEnvironment(): string {
    return (
      (window as any).__ENVIRONMENT__ ||
      (this.isProduction() ? "production" : "development")
    );
  }

  private isProduction(): boolean {
    return !!(window as any).__PRODUCTION__;
  }

  private getConnectionType(): string {
    const connection = (navigator as any).connection;
    return connection?.effectiveType || "unknown";
  }

  private monitorCoreWebVitals(): void {
    // Monitor First Contentful Paint, Largest Contentful Paint, etc.
    // This would integrate with Web Vitals library
    if ("PerformanceObserver" in window) {
      // Monitor LCP
      const lcpObserver = new PerformanceObserver((list) => {
        const entries = list.getEntries();
        const lcp = entries[entries.length - 1] as PerformanceEntry;

        if (lcp.startTime > 2500) {
          // LCP threshold
          this.logPerformanceIssue("LCP", lcp.startTime, 2500);
        }
      });

      lcpObserver.observe({ entryTypes: ["largest-contentful-paint"] });
    }
  }

  private monitorResourceLoading(): void {
    // Monitor slow-loading resources
    if ("PerformanceObserver" in window) {
      const resourceObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry: PerformanceEntry) => {
          const duration = entry.duration;

          if (duration > 3000) {
            // 3 second threshold
            this.logPerformanceIssue("Slow Resource", duration, 3000);
          }
        });
      });

      resourceObserver.observe({ entryTypes: ["resource"] });
    }
  }

  private monitorMemoryUsage(): void {
    // Monitor memory usage (if supported)
    if ("memory" in performance) {
      setInterval(() => {
        const memory = (performance as any).memory;
        const usedMB = memory.usedJSHeapSize / 1048576; // Convert to MB

        if (usedMB > 100) {
          // 100MB threshold
          this.logPerformanceIssue("High Memory Usage", usedMB, 100);
        }
      }, 30000); // Check every 30 seconds
    }
  }

  private calculateErrorStatistics(): any {
    const totalErrors = this.errorBuffer.length;
    const errorsByType: Record<string, number> = {};
    const errorsBySeverity: Record<string, number> = {};

    this.errorBuffer.forEach((error) => {
      errorsByType[error.type] = (errorsByType[error.type] || 0) + 1;
      errorsBySeverity[error.severity] =
        (errorsBySeverity[error.severity] || 0) + 1;
    });

    const topErrors = this.errorBuffer
      .sort((a, b) => b.occurrences - a.occurrences)
      .slice(0, 10);

    return {
      totalErrors,
      errorsByType,
      errorsBySeverity,
      errorRate: this.errorRate$.value,
      topErrors,
    };
  }

  private storeErrorsLocally(errors: MonitoredError[]): void {
    // Store errors in localStorage for retry
    try {
      const stored = JSON.parse(localStorage.getItem("pending_errors") || "[]");
      stored.push(...errors);
      localStorage.setItem("pending_errors", JSON.stringify(stored));
    } catch (error) {
      console.error("Failed to store errors locally:", error);
    }
  }
}
```

### **2. 🎛️ Performance Monitoring Dashboard Component**

```typescript
// src/app/features/monitoring/components/monitoring-dashboard.component.ts
import { Component, OnInit, OnDestroy } from "@angular/core";
import { Observable, combineLatest, timer } from "rxjs";
import { map, startWith, takeUntil } from "rxjs/operators";

import { BaseComponent } from "../../../shared/components/base-component";
import {
  ErrorMonitoringService,
  MonitoredError,
} from "../../../core/services/error-monitoring.service";
import { PerformanceMonitoringService } from "../../../core/services/performance-monitoring.service";
import { MemoryCleanupService } from "../../../core/services/memory-cleanup.service";

interface DashboardMetrics {
  errorCount: number;
  errorRate: number;
  criticalErrors: MonitoredError[];
  averageResponseTime: number;
  memoryUsage: number;
  activeUsers: number;
  serverHealth: "healthy" | "warning" | "critical";
}

@Component({
  selector: "app-monitoring-dashboard",
  template: `
    <div class="monitoring-dashboard">
      <div class="dashboard-header">
        <h2>🚀 Production Monitoring Dashboard</h2>
        <div class="last-updated">
          Last updated: {{ lastUpdated | date : "medium" }}
        </div>
      </div>

      <div *ngIf="metrics$ | async as metrics" class="metrics-grid">
        <!-- Error Metrics -->
        <div class="metric-card error-metrics">
          <div class="metric-header">
            <h3>🚨 Error Tracking</h3>
            <span
              class="status-indicator"
              [class.warning]="metrics.errorRate > 5"
              [class.critical]="metrics.errorRate > 10"
            ></span>
          </div>
          <div class="metric-value">{{ metrics.errorCount }}</div>
          <div class="metric-label">Total Errors</div>
          <div class="metric-secondary">Rate: {{ metrics.errorRate }}/min</div>
        </div>

        <!-- Performance Metrics -->
        <div class="metric-card performance-metrics">
          <div class="metric-header">
            <h3>⚡ Performance</h3>
            <span
              class="status-indicator"
              [class.warning]="metrics.averageResponseTime > 1000"
              [class.critical]="metrics.averageResponseTime > 2000"
            ></span>
          </div>
          <div class="metric-value">{{ metrics.averageResponseTime }}ms</div>
          <div class="metric-label">Avg Response Time</div>
        </div>

        <!-- Memory Metrics -->
        <div class="metric-card memory-metrics">
          <div class="metric-header">
            <h3>💾 Memory Usage</h3>
            <span
              class="status-indicator"
              [class.warning]="metrics.memoryUsage > 80"
              [class.critical]="metrics.memoryUsage > 90"
            ></span>
          </div>
          <div class="metric-value">{{ metrics.memoryUsage }}%</div>
          <div class="metric-label">Memory Utilization</div>
        </div>

        <!-- Server Health -->
        <div class="metric-card server-health">
          <div class="metric-header">
            <h3>🖥️ Server Status</h3>
            <span
              class="status-indicator"
              [class]="metrics.serverHealth"
            ></span>
          </div>
          <div class="metric-value">{{ metrics.activeUsers }}</div>
          <div class="metric-label">Active Users</div>
          <div class="metric-secondary">
            Status: {{ metrics.serverHealth | titlecase }}
          </div>
        </div>
      </div>

      <!-- Critical Errors Section -->
      <div class="critical-errors-section" *ngIf="metrics$ | async as metrics">
        <h3>🚨 Critical Errors</h3>
        <div class="error-list">
          <div
            *ngFor="
              let error of metrics.criticalErrors;
              trackBy: trackByErrorId
            "
            class="error-item"
          >
            <div class="error-info">
              <div class="error-message">{{ error.message }}</div>
              <div class="error-meta">
                {{ error.type }} • {{ error.lastSeen | date : "short" }}
              </div>
            </div>
            <div class="error-actions">
              <button
                type="button"
                class="btn btn-sm"
                (click)="viewErrorDetails(error)"
              >
                Details
              </button>
            </div>
          </div>

          <div *ngIf="metrics.criticalErrors.length === 0" class="no-errors">
            ✅ No critical errors detected
          </div>
        </div>
      </div>

      <!-- Real-time Charts -->
      <div class="charts-section">
        <div class="chart-container">
          <h3>📊 Error Rate Over Time</h3>
          <canvas #errorChart></canvas>
        </div>

        <div class="chart-container">
          <h3>⚡ Performance Trends</h3>
          <canvas #performanceChart></canvas>
        </div>
      </div>
    </div>
  `,
  styleUrls: ["./monitoring-dashboard.component.scss"],
})
export class MonitoringDashboardComponent
  extends BaseComponent
  implements OnInit
{
  metrics$: Observable<DashboardMetrics>;
  lastUpdated = new Date();

  constructor(
    memoryCleanupService: MemoryCleanupService,
    private errorMonitoringService: ErrorMonitoringService,
    private performanceMonitoringService: PerformanceMonitoringService
  ) {
    super(memoryCleanupService);
  }

  ngOnInit(): void {
    this.setupDashboardMetrics();
    this.setupAutoRefresh();
  }

  private setupDashboardMetrics(): void {
    // Combine all monitoring streams
    this.metrics$ = combineLatest([
      this.errorMonitoringService.errorCount$,
      this.errorMonitoringService.errorRate$,
      this.errorMonitoringService.criticalErrors$,
      this.performanceMonitoringService.averageResponseTime$,
      this.performanceMonitoringService.memoryUsage$,
      this.performanceMonitoringService.activeUsers$,
      this.performanceMonitoringService.serverHealth$,
    ]).pipe(
      map(
        ([
          errorCount,
          errorRate,
          criticalErrors,
          responseTime,
          memoryUsage,
          activeUsers,
          serverHealth,
        ]) => ({
          errorCount,
          errorRate,
          criticalErrors,
          averageResponseTime: responseTime,
          memoryUsage,
          activeUsers,
          serverHealth,
        })
      ),
      takeUntil(this.destroy$)
    );
  }

  private setupAutoRefresh(): void {
    // Refresh every 30 seconds
    this.safeSubscribe(timer(0, 30000)).subscribe(() => {
      this.lastUpdated = new Date();
      this.refreshMetrics();
    });
  }

  private refreshMetrics(): void {
    // Trigger refresh of all monitoring services
    this.performanceMonitoringService.refreshMetrics();
  }

  // 🎯 Event Handlers
  viewErrorDetails(error: MonitoredError): void {
    console.log("Viewing error details:", error);
    // Navigate to error details page or open modal
  }

  trackByErrorId(index: number, error: MonitoredError): string {
    return error.id;
  }
}
```

---

## 📈 **Server Infrastructure & Scaling**

### **1. 🌐 Load Balancing & Auto-Scaling Setup**

```typescript
// infrastructure/nginx/load-balancer.conf
# High-Performance Load Balancer Configuration

upstream angular_app {
    # Load balancing strategy
    least_conn;

    # Backend servers
    server app1.company.com:3000 max_fails=3 fail_timeout=30s;
    server app2.company.com:3000 max_fails=3 fail_timeout=30s;
    server app3.company.com:3000 max_fails=3 fail_timeout=30s;

    # Health checks
    keepalive 32;
}

# Main server block
server {
    listen 80;
    listen 443 ssl http2;
    server_name app.company.com;

    # SSL Configuration
    ssl_certificate /etc/ssl/certs/app.company.com.crt;
    ssl_certificate_key /etc/ssl/private/app.company.com.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types
        application/javascript
        application/json
        application/xml
        text/css
        text/javascript
        text/plain
        text/xml;

    # Browser caching for static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Vary Accept-Encoding;

        # CORS for assets
        add_header Access-Control-Allow-Origin "*";
    }

    # API requests
    location /api/ {
        proxy_pass http://angular_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeout settings
        proxy_connect_timeout 10s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # Rate limiting
        limit_req zone=api burst=20 nodelay;
    }

    # Angular app (serve from CDN for better performance)
    location / {
        # First try to serve from local cache, then from backend
        try_files $uri $uri/ @backend;

        # Cache HTML files for short period
        location ~* \.html$ {
            expires 10m;
            add_header Cache-Control "public, no-transform";
        }

        # Cache manifest and service worker
        location ~* \.(manifest\.json|sw\.js)$ {
            expires 10m;
            add_header Cache-Control "public, no-cache";
        }
    }

    # Backend fallback
    location @backend {
        proxy_pass http://angular_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}

# Rate limiting zones
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;

    # Connection limiting
    limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
    limit_conn conn_limit_per_ip 20;
}
```

### **2. 🐳 Docker & Kubernetes Deployment**

```yaml
# k8s/angular-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: angular-app
  labels:
    app: angular-app
    version: v1.0.0
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: angular-app
  template:
    metadata:
      labels:
        app: angular-app
        version: v1.0.0
    spec:
      containers:
        - name: angular-app
          image: your-registry/angular-app:1.0.0
          ports:
            - containerPort: 3000

          # Resource limits
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"

          # Environment variables
          env:
            - name: NODE_ENV
              value: "production"
            - name: API_BASE_URL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: api-base-url
            - name: MONITORING_ENDPOINT
              valueFrom:
                secretKeyRef:
                  name: monitoring-secrets
                  key: endpoint

          # Health checks
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3

          # Volume mounts
          volumeMounts:
            - name: app-logs
              mountPath: /app/logs
            - name: app-config
              mountPath: /app/config

      # Volumes
      volumes:
        - name: app-logs
          emptyDir: {}
        - name: app-config
          configMap:
            name: app-config

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: angular-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: angular-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: angular-app-service
  labels:
    app: angular-app
spec:
  selector:
    app: angular-app
  ports:
    - name: http
      port: 80
      targetPort: 3000
      protocol: TCP
  type: ClusterIP

---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: angular-app-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
spec:
  tls:
    - hosts:
        - app.company.com
      secretName: angular-app-tls
  rules:
    - host: app.company.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: angular-app-service
                port:
                  number: 80
```

---

## 🔍 **Advanced Monitoring & Alerting**

### **1. 📊 Comprehensive Monitoring Stack**

```typescript
// src/app/core/services/metrics-collection.service.ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { interval, BehaviorSubject, Observable } from "rxjs";
import { switchMap, catchError } from "rxjs/operators";

export interface SystemMetrics {
  timestamp: Date;
  application: {
    errorRate: number;
    responseTime: number;
    throughput: number;
    userSessions: number;
    memoryUsage: number;
    cpuUsage: number;
  };
  infrastructure: {
    serverLoad: number;
    databaseConnections: number;
    queueLength: number;
    diskUsage: number;
    networkLatency: number;
  };
  business: {
    activeUsers: number;
    conversionRate: number;
    revenue: number;
    customerSatisfaction: number;
  };
}

export interface AlertRule {
  id: string;
  name: string;
  metric: string;
  threshold: number;
  operator: "gt" | "lt" | "eq";
  severity: "info" | "warning" | "critical";
  duration: number; // seconds
  enabled: boolean;
  channels: ("email" | "slack" | "pagerduty" | "webhook")[];
}

@Injectable({
  providedIn: "root",
})
export class MetricsCollectionService {
  private metrics$ = new BehaviorSubject<SystemMetrics | null>(null);
  private alerts$ = new BehaviorSubject<any[]>([]);

  private alertRules: AlertRule[] = [
    {
      id: "high-error-rate",
      name: "High Error Rate",
      metric: "application.errorRate",
      threshold: 5, // 5% error rate
      operator: "gt",
      severity: "critical",
      duration: 60, // 1 minute
      enabled: true,
      channels: ["email", "slack", "pagerduty"],
    },
    {
      id: "slow-response-time",
      name: "Slow Response Time",
      metric: "application.responseTime",
      threshold: 2000, // 2 seconds
      operator: "gt",
      severity: "warning",
      duration: 120, // 2 minutes
      enabled: true,
      channels: ["slack"],
    },
    {
      id: "high-memory-usage",
      name: "High Memory Usage",
      metric: "application.memoryUsage",
      threshold: 85, // 85%
      operator: "gt",
      severity: "warning",
      duration: 300, // 5 minutes
      enabled: true,
      channels: ["email", "slack"],
    },
    {
      id: "low-conversion-rate",
      name: "Low Conversion Rate",
      metric: "business.conversionRate",
      threshold: 2, // 2%
      operator: "lt",
      severity: "warning",
      duration: 900, // 15 minutes
      enabled: true,
      channels: ["email"],
    },
  ];

  constructor(private http: HttpClient) {
    this.startMetricsCollection();
    this.startAlertMonitoring();
  }

  // 📊 GET CURRENT METRICS
  getCurrentMetrics(): Observable<SystemMetrics | null> {
    return this.metrics$.asObservable();
  }

  // 🚨 GET ACTIVE ALERTS
  getActiveAlerts(): Observable<any[]> {
    return this.alerts$.asObservable();
  }

  // 📈 GET HISTORICAL METRICS
  getHistoricalMetrics(
    startTime: Date,
    endTime: Date,
    interval: "1m" | "5m" | "15m" | "1h" | "1d" = "5m"
  ): Observable<SystemMetrics[]> {
    return this.http.get<SystemMetrics[]>("/api/metrics/historical", {
      params: {
        start: startTime.toISOString(),
        end: endTime.toISOString(),
        interval,
      },
    });
  }

  // ⚙️ CONFIGURE ALERT RULES
  updateAlertRule(rule: AlertRule): void {
    const index = this.alertRules.findIndex((r) => r.id === rule.id);
    if (index >= 0) {
      this.alertRules[index] = rule;
    } else {
      this.alertRules.push(rule);
    }

    // Save to backend
    this.http.put(`/api/alerts/rules/${rule.id}`, rule).subscribe({
      next: () => console.log(`✅ Alert rule updated: ${rule.name}`),
      error: (error) => console.error("❌ Failed to update alert rule:", error),
    });
  }

  // 🔄 Private Methods

  private startMetricsCollection(): void {
    // Collect metrics every 30 seconds
    interval(30000)
      .pipe(
        switchMap(() => this.collectMetrics()),
        catchError((error) => {
          console.error("❌ Failed to collect metrics:", error);
          return [];
        })
      )
      .subscribe((metrics) => {
        if (metrics) {
          this.metrics$.next(metrics);
          this.checkAlerts(metrics);
        }
      });
  }

  private collectMetrics(): Observable<SystemMetrics> {
    return new Observable((observer) => {
      // Collect application metrics
      const applicationMetrics = this.collectApplicationMetrics();

      // Get infrastructure metrics from monitoring service
      this.http.get<any>("/api/monitoring/infrastructure").subscribe({
        next: (infrastructure) => {
          // Get business metrics
          this.http.get<any>("/api/analytics/business-metrics").subscribe({
            next: (business) => {
              const metrics: SystemMetrics = {
                timestamp: new Date(),
                application: applicationMetrics,
                infrastructure,
                business,
              };

              observer.next(metrics);
              observer.complete();
            },
            error: (error) => {
              console.error("Failed to get business metrics:", error);
              observer.error(error);
            },
          });
        },
        error: (error) => {
          console.error("Failed to get infrastructure metrics:", error);
          observer.error(error);
        },
      });
    });
  }

  private collectApplicationMetrics(): SystemMetrics["application"] {
    // Collect client-side metrics
    const memory = (performance as any).memory;
    const navigation = performance.getEntriesByType(
      "navigation"
    )[0] as PerformanceNavigationTiming;

    return {
      errorRate: this.calculateErrorRate(),
      responseTime: navigation
        ? navigation.loadEventEnd - navigation.loadEventStart
        : 0,
      throughput: this.calculateThroughput(),
      userSessions: this.getActiveSessionCount(),
      memoryUsage: memory
        ? (memory.usedJSHeapSize / memory.jsHeapSizeLimit) * 100
        : 0,
      cpuUsage: 0, // Would need Web Workers to calculate CPU usage
    };
  }

  private calculateErrorRate(): number {
    // Calculate error rate from error monitoring service
    const errorCount = this.getRecentErrorCount();
    const totalRequests = this.getRecentRequestCount();
    return totalRequests > 0 ? (errorCount / totalRequests) * 100 : 0;
  }

  private calculateThroughput(): number {
    // Calculate requests per second
    const requests = this.getRecentRequestCount();
    return requests / 60; // Per minute to per second
  }

  private getActiveSessionCount(): number {
    // Get from session tracking
    return parseInt(sessionStorage.getItem("activeUsers") || "1");
  }

  private getRecentErrorCount(): number {
    // Integration with error monitoring service
    return 0; // Placeholder
  }

  private getRecentRequestCount(): number {
    // Track API requests
    return 0; // Placeholder
  }

  private startAlertMonitoring(): void {
    // Check alerts every minute
    interval(60000).subscribe(() => {
      const currentMetrics = this.metrics$.value;
      if (currentMetrics) {
        this.checkAlerts(currentMetrics);
      }
    });
  }

  private checkAlerts(metrics: SystemMetrics): void {
    this.alertRules
      .filter((rule) => rule.enabled)
      .forEach((rule) => {
        const metricValue = this.getMetricValue(metrics, rule.metric);
        const isAlertTriggered = this.evaluateAlertCondition(metricValue, rule);

        if (isAlertTriggered) {
          this.triggerAlert(rule, metricValue, metrics);
        }
      });
  }

  private getMetricValue(metrics: SystemMetrics, metricPath: string): number {
    // Navigate nested object path (e.g., 'application.errorRate')
    return (
      metricPath.split(".").reduce((obj, key) => obj?.[key], metrics as any) ||
      0
    );
  }

  private evaluateAlertCondition(value: number, rule: AlertRule): boolean {
    switch (rule.operator) {
      case "gt":
        return value > rule.threshold;
      case "lt":
        return value < rule.threshold;
      case "eq":
        return value === rule.threshold;
      default:
        return false;
    }
  }

  private triggerAlert(
    rule: AlertRule,
    value: number,
    metrics: SystemMetrics
  ): void {
    const alert = {
      id: `alert_${Date.now()}`,
      ruleId: rule.id,
      name: rule.name,
      severity: rule.severity,
      message: `${rule.name}: ${rule.metric} is ${value} (threshold: ${rule.threshold})`,
      timestamp: new Date(),
      value,
      threshold: rule.threshold,
      metrics,
    };

    console.warn(`🚨 ALERT TRIGGERED: ${alert.message}`);

    // Add to active alerts
    const currentAlerts = this.alerts$.value;
    currentAlerts.unshift(alert);

    // Keep only last 50 alerts
    if (currentAlerts.length > 50) {
      currentAlerts.splice(50);
    }

    this.alerts$.next(currentAlerts);

    // Send alert through configured channels
    this.sendAlert(alert, rule);
  }

  private sendAlert(alert: any, rule: AlertRule): void {
    rule.channels.forEach((channel) => {
      switch (channel) {
        case "email":
          this.sendEmailAlert(alert);
          break;
        case "slack":
          this.sendSlackAlert(alert);
          break;
        case "pagerduty":
          this.sendPagerDutyAlert(alert);
          break;
        case "webhook":
          this.sendWebhookAlert(alert);
          break;
      }
    });
  }

  private sendEmailAlert(alert: any): void {
    this.http
      .post("/api/alerts/email", {
        subject: `🚨 Alert: ${alert.name}`,
        message: alert.message,
        severity: alert.severity,
        timestamp: alert.timestamp,
      })
      .subscribe({
        next: () => console.log("✅ Email alert sent"),
        error: (error) =>
          console.error("❌ Failed to send email alert:", error),
      });
  }

  private sendSlackAlert(alert: any): void {
    this.http
      .post("/api/alerts/slack", {
        channel: "#alerts",
        message: `${this.getSeverityEmoji(alert.severity)} *${alert.name}*\n${
          alert.message
        }`,
        severity: alert.severity,
      })
      .subscribe({
        next: () => console.log("✅ Slack alert sent"),
        error: (error) =>
          console.error("❌ Failed to send Slack alert:", error),
      });
  }

  private sendPagerDutyAlert(alert: any): void {
    if (alert.severity === "critical") {
      this.http
        .post("/api/alerts/pagerduty", {
          incident_key: alert.id,
          description: alert.message,
          severity: alert.severity,
        })
        .subscribe({
          next: () => console.log("✅ PagerDuty alert sent"),
          error: (error) =>
            console.error("❌ Failed to send PagerDuty alert:", error),
        });
    }
  }

  private sendWebhookAlert(alert: any): void {
    this.http.post("/api/alerts/webhook", alert).subscribe({
      next: () => console.log("✅ Webhook alert sent"),
      error: (error) =>
        console.error("❌ Failed to send webhook alert:", error),
    });
  }

  private getSeverityEmoji(severity: string): string {
    switch (severity) {
      case "critical":
        return "🚨";
      case "warning":
        return "⚠️";
      case "info":
        return "ℹ️";
      default:
        return "📊";
    }
  }
}
```

This is **Part 1** of the Production Monitoring guide. Would you like me to continue with **Part 2** covering CDN optimization, database scaling, and advanced deployment strategies?
