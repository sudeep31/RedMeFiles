# 🔌 **Third-Party Library Integration in Angular**

## 🎯 **What You'll Learn**

This comprehensive guide covers advanced techniques for integrating third-party libraries into Angular applications, from simple utility libraries to complex UI frameworks, with proper TypeScript support and Angular best practices.

---

## 📚 **The Basics: Third-Party Integration Fundamentals**

### **🤔 Why Integrate Third-Party Libraries?**

**Common Use Cases:**

- 📊 **Data Visualization** - Chart.js, D3.js, Highcharts
- 🗓️ **Date/Time** - Moment.js, date-fns, Day.js
- 🎨 **UI Components** - jQuery plugins, vanilla JS components
- 📱 **Analytics** - Google Analytics, Mixpanel
- 🎯 **Utilities** - Lodash, Underscore.js
- 🌍 **Maps** - Google Maps, Leaflet, Mapbox

**Integration Challenges:**

- ⚠️ **TypeScript Compatibility**
- ⚠️ **Angular Change Detection**
- ⚠️ **Lifecycle Management**
- ⚠️ **Bundle Size Impact**
- ⚠️ **Memory Leaks**

---

## 🛠️ **Installation Methods**

### **📦 Method 1: NPM Package Installation**

```bash
# 📦 Install library with types
npm install chart.js @types/chart.js

# 📦 Install without types (need to create own)
npm install some-library
npm install --save-dev @types/some-library

# 📦 Install with peer dependencies
npm install library-name
npm install peer-dependency-1 peer-dependency-2
```

### **📜 Method 2: CDN Integration**

```html
<!-- index.html - CDN approach -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Angular App</title>

    <!-- 🎨 CSS Libraries -->
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css"
    />

    <!-- 📊 JavaScript Libraries -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY"></script>
  </head>
  <body>
    <app-root></app-root>
  </body>
</html>
```

### **🔧 Method 3: Manual Integration**

```bash
# 📂 Download and place in assets
src/assets/libs/
├── custom-library.js
├── custom-library.css
└── custom-library.d.ts
```

---

## 💉 **Dependency Injection Strategies**

### **🎯 Method 1: Service Wrapper Pattern**

```typescript
// src/app/services/chart.service.ts
import { Injectable, Inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";

// 🎯 Chart.js interface (if no types available)
declare var Chart: any;

export interface ChartData {
  labels: string[];
  datasets: Array<{
    label: string;
    data: number[];
    backgroundColor?: string | string[];
    borderColor?: string | string[];
  }>;
}

export interface ChartOptions {
  responsive?: boolean;
  maintainAspectRatio?: boolean;
  plugins?: any;
  scales?: any;
}

@Injectable({
  providedIn: "root",
})
export class ChartService {
  private isLibraryLoaded = false;
  private chartInstances = new Map<string, any>();

  constructor(@Inject(PLATFORM_ID) private platformId: Object) {
    this.initializeLibrary();
  }

  // 📊 CREATE CHART
  async createChart(
    canvasId: string,
    type: "line" | "bar" | "pie" | "doughnut",
    data: ChartData,
    options: ChartOptions = {}
  ): Promise<any> {
    if (!isPlatformBrowser(this.platformId)) {
      console.warn("Chart creation skipped: not in browser environment");
      return null;
    }

    // Wait for library to load
    await this.ensureLibraryLoaded();

    const canvas = document.getElementById(canvasId) as HTMLCanvasElement;
    if (!canvas) {
      throw new Error(`Canvas element with id '${canvasId}' not found`);
    }

    // Destroy existing chart if present
    this.destroyChart(canvasId);

    // Create new chart
    const chart = new Chart(canvas.getContext("2d"), {
      type,
      data,
      options: {
        responsive: true,
        maintainAspectRatio: false,
        ...options,
      },
    });

    // Store reference for cleanup
    this.chartInstances.set(canvasId, chart);

    console.log(`📊 Chart created successfully: ${canvasId}`);
    return chart;
  }

  // 🔄 UPDATE CHART
  updateChart(canvasId: string, data: ChartData): void {
    const chart = this.chartInstances.get(canvasId);

    if (chart) {
      chart.data = data;
      chart.update("active");
      console.log(`🔄 Chart updated: ${canvasId}`);
    } else {
      console.warn(`Chart not found for update: ${canvasId}`);
    }
  }

  // 🗑️ DESTROY CHART
  destroyChart(canvasId: string): void {
    const chart = this.chartInstances.get(canvasId);

    if (chart) {
      chart.destroy();
      this.chartInstances.delete(canvasId);
      console.log(`🗑️ Chart destroyed: ${canvasId}`);
    }
  }

  // 🧹 CLEANUP ALL CHARTS
  destroyAllCharts(): void {
    this.chartInstances.forEach((chart, id) => {
      chart.destroy();
      console.log(`🧹 Cleanup chart: ${id}`);
    });
    this.chartInstances.clear();
  }

  // 📈 GET CHART INSTANCE
  getChart(canvasId: string): any {
    return this.chartInstances.get(canvasId);
  }

  // 💾 EXPORT CHART AS IMAGE
  exportChart(canvasId: string, format: "png" | "jpeg" = "png"): string | null {
    const chart = this.chartInstances.get(canvasId);

    if (chart) {
      return chart.toBase64Image(`image/${format}`);
    }

    return null;
  }

  // 🔧 Private Methods

  private async initializeLibrary(): Promise<void> {
    if (!isPlatformBrowser(this.platformId)) {
      return;
    }

    // Check if Chart.js is already loaded
    if (typeof Chart !== "undefined") {
      this.isLibraryLoaded = true;
      return;
    }

    // Dynamic import for Chart.js
    try {
      await import("chart.js/auto");
      this.isLibraryLoaded = true;
      console.log("📊 Chart.js loaded successfully");
    } catch (error) {
      console.error("Failed to load Chart.js:", error);
    }
  }

  private async ensureLibraryLoaded(): Promise<void> {
    if (this.isLibraryLoaded) {
      return;
    }

    // Wait for library to load with timeout
    return new Promise((resolve, reject) => {
      let attempts = 0;
      const maxAttempts = 50; // 5 seconds max

      const checkInterval = setInterval(() => {
        attempts++;

        if (this.isLibraryLoaded || typeof Chart !== "undefined") {
          clearInterval(checkInterval);
          this.isLibraryLoaded = true;
          resolve();
        } else if (attempts >= maxAttempts) {
          clearInterval(checkInterval);
          reject(new Error("Chart.js failed to load within timeout"));
        }
      }, 100);
    });
  }
}
```

### **🎨 Component Usage Example**

```typescript
// src/app/components/dashboard/dashboard.component.ts
import {
  Component,
  OnInit,
  OnDestroy,
  AfterViewInit,
  ViewChild,
  ElementRef,
} from "@angular/core";
import { ChartService, ChartData } from "../../services/chart.service";

@Component({
  selector: "app-dashboard",
  template: `
    <div class="dashboard">
      <h2>Sales Dashboard</h2>

      <!-- 📊 Chart Container -->
      <div class="chart-container">
        <canvas #salesChart id="salesChart" width="400" height="200"> </canvas>
      </div>

      <!-- 🎮 Chart Controls -->
      <div class="chart-controls">
        <button
          class="btn btn-primary"
          (click)="updateChartData()"
          [disabled]="loading"
        >
          {{ loading ? "Updating..." : "Update Data" }}
        </button>

        <button class="btn btn-secondary" (click)="exportChart()">
          Export PNG
        </button>

        <select
          class="form-select"
          [(ngModel)]="chartType"
          (change)="changeChartType()"
        >
          <option value="line">Line Chart</option>
          <option value="bar">Bar Chart</option>
          <option value="pie">Pie Chart</option>
        </select>
      </div>

      <!-- 📈 Chart Statistics -->
      <div class="chart-stats" *ngIf="chartData">
        <div class="stat">
          <span class="stat-label">Total Sales:</span>
          <span class="stat-value">{{ getTotalSales() | currency }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">Average:</span>
          <span class="stat-value">{{ getAverageSales() | currency }}</span>
        </div>
      </div>
    </div>
  `,
  styles: [
    `
      .dashboard {
        padding: 20px;
      }

      .chart-container {
        position: relative;
        width: 100%;
        height: 400px;
        margin: 20px 0;
        background: white;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        padding: 20px;
      }

      .chart-controls {
        display: flex;
        gap: 12px;
        margin: 20px 0;
        align-items: center;
        flex-wrap: wrap;
      }

      .chart-stats {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 16px;
        margin-top: 20px;
      }

      .stat {
        background: #f8f9fa;
        padding: 16px;
        border-radius: 6px;
        border-left: 4px solid #007bff;
      }

      .stat-label {
        display: block;
        font-weight: 500;
        color: #6c757d;
        margin-bottom: 4px;
      }

      .stat-value {
        display: block;
        font-size: 1.25rem;
        font-weight: 600;
        color: #2c3e50;
      }

      .btn {
        padding: 8px 16px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
        transition: all 0.2s;
      }

      .btn-primary {
        background: #007bff;
        color: white;
      }

      .btn-primary:hover {
        background: #0056b3;
      }

      .btn-primary:disabled {
        opacity: 0.6;
        cursor: not-allowed;
      }

      .btn-secondary {
        background: #6c757d;
        color: white;
      }

      .btn-secondary:hover {
        background: #545b62;
      }

      .form-select {
        padding: 8px 12px;
        border: 1px solid #ddd;
        border-radius: 4px;
        background: white;
      }
    `,
  ],
})
export class DashboardComponent implements OnInit, AfterViewInit, OnDestroy {
  @ViewChild("salesChart") salesChartRef!: ElementRef;

  chartType: "line" | "bar" | "pie" = "line";
  loading = false;
  chartData: ChartData = {
    labels: ["January", "February", "March", "April", "May", "June"],
    datasets: [
      {
        label: "Sales 2024",
        data: [12000, 19000, 8000, 15000, 22000, 18000],
        backgroundColor: [
          "rgba(54, 162, 235, 0.8)",
          "rgba(255, 99, 132, 0.8)",
          "rgba(255, 205, 86, 0.8)",
          "rgba(75, 192, 192, 0.8)",
          "rgba(153, 102, 255, 0.8)",
          "rgba(255, 159, 64, 0.8)",
        ],
        borderColor: [
          "rgba(54, 162, 235, 1)",
          "rgba(255, 99, 132, 1)",
          "rgba(255, 205, 86, 1)",
          "rgba(75, 192, 192, 1)",
          "rgba(153, 102, 255, 1)",
          "rgba(255, 159, 64, 1)",
        ],
        borderWidth: 2,
      },
    ],
  };

  constructor(private chartService: ChartService) {}

  ngOnInit(): void {
    console.log("🚀 Dashboard component initialized");
  }

  async ngAfterViewInit(): Promise<void> {
    // Wait a tick for the view to be fully rendered
    setTimeout(() => {
      this.createChart();
    }, 0);
  }

  ngOnDestroy(): void {
    // 🧹 Cleanup charts to prevent memory leaks
    this.chartService.destroyChart("salesChart");
    console.log("🧹 Dashboard component cleanup completed");
  }

  async createChart(): Promise<void> {
    try {
      await this.chartService.createChart(
        "salesChart",
        this.chartType,
        this.chartData,
        {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            title: {
              display: true,
              text: "Monthly Sales Report",
            },
            legend: {
              display: this.chartType !== "pie",
              position: "top",
            },
          },
          scales:
            this.chartType === "pie"
              ? {}
              : {
                  y: {
                    beginAtZero: true,
                    ticks: {
                      callback: function (value: any) {
                        return "$" + value.toLocaleString();
                      },
                    },
                  },
                },
        }
      );
    } catch (error) {
      console.error("Failed to create chart:", error);
    }
  }

  async changeChartType(): Promise<void> {
    this.chartService.destroyChart("salesChart");
    await this.createChart();
  }

  updateChartData(): void {
    this.loading = true;

    // Simulate API call
    setTimeout(() => {
      // Generate random data
      this.chartData = {
        ...this.chartData,
        datasets: [
          {
            ...this.chartData.datasets[0],
            data: this.generateRandomData(),
          },
        ],
      };

      this.chartService.updateChart("salesChart", this.chartData);
      this.loading = false;
    }, 1000);
  }

  exportChart(): void {
    const imageData = this.chartService.exportChart("salesChart", "png");

    if (imageData) {
      // Create download link
      const link = document.createElement("a");
      link.download = `sales-chart-${
        new Date().toISOString().split("T")[0]
      }.png`;
      link.href = imageData;
      link.click();
    }
  }

  getTotalSales(): number {
    return this.chartData.datasets[0].data.reduce(
      (sum, value) => sum + value,
      0
    );
  }

  getAverageSales(): number {
    const data = this.chartData.datasets[0].data;
    return data.reduce((sum, value) => sum + value, 0) / data.length;
  }

  private generateRandomData(): number[] {
    return Array.from(
      { length: 6 },
      () => Math.floor(Math.random() * 25000) + 5000
    );
  }
}
```

---

## 🎯 **Advanced Integration Patterns**

### **🏭 Method 2: Token-Based Injection**

```typescript
// src/app/tokens/third-party.tokens.ts
import { InjectionToken } from "@angular/core";

// 🎯 Create injection tokens for libraries
export const GOOGLE_MAPS_API_KEY = new InjectionToken<string>(
  "GoogleMapsApiKey"
);
export const ANALYTICS_CONFIG = new InjectionToken<any>("AnalyticsConfig");
export const LIBRARY_CONFIG = new InjectionToken<any>("LibraryConfig");

// 🌍 Google Maps integration
export interface GoogleMapsConfig {
  apiKey: string;
  libraries?: string[];
  language?: string;
  region?: string;
}

export const GOOGLE_MAPS_CONFIG = new InjectionToken<GoogleMapsConfig>(
  "GoogleMapsConfig"
);
```

```typescript
// src/app/services/google-maps.service.ts
import { Injectable, Inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";
import {
  GOOGLE_MAPS_CONFIG,
  GoogleMapsConfig,
} from "../tokens/third-party.tokens";

declare var google: any;

@Injectable({
  providedIn: "root",
})
export class GoogleMapsService {
  private isLoaded = false;
  private mapInstances = new Map<string, any>();

  constructor(
    @Inject(GOOGLE_MAPS_CONFIG) private config: GoogleMapsConfig,
    @Inject(PLATFORM_ID) private platformId: Object
  ) {}

  // 🗺️ LOAD GOOGLE MAPS API
  async loadGoogleMaps(): Promise<void> {
    if (!isPlatformBrowser(this.platformId)) {
      return;
    }

    if (this.isLoaded || typeof google !== "undefined") {
      return;
    }

    return new Promise((resolve, reject) => {
      const script = document.createElement("script");
      script.src = this.buildGoogleMapsUrl();
      script.async = true;
      script.defer = true;

      script.onload = () => {
        this.isLoaded = true;
        resolve();
      };

      script.onerror = () => {
        reject(new Error("Failed to load Google Maps API"));
      };

      document.head.appendChild(script);
    });
  }

  // 🗺️ CREATE MAP
  async createMap(
    containerId: string,
    center: { lat: number; lng: number },
    zoom: number = 10,
    options: any = {}
  ): Promise<any> {
    await this.loadGoogleMaps();

    const container = document.getElementById(containerId);
    if (!container) {
      throw new Error(`Container with id '${containerId}' not found`);
    }

    const map = new google.maps.Map(container, {
      center,
      zoom,
      ...options,
    });

    this.mapInstances.set(containerId, map);
    return map;
  }

  // 📍 ADD MARKER
  addMarker(
    mapId: string,
    position: { lat: number; lng: number },
    options: any = {}
  ): any {
    const map = this.mapInstances.get(mapId);

    if (!map) {
      throw new Error(`Map with id '${mapId}' not found`);
    }

    return new google.maps.Marker({
      position,
      map,
      ...options,
    });
  }

  private buildGoogleMapsUrl(): string {
    const baseUrl = "https://maps.googleapis.com/maps/api/js";
    const params = new URLSearchParams();

    params.append("key", this.config.apiKey);

    if (this.config.libraries) {
      params.append("libraries", this.config.libraries.join(","));
    }

    if (this.config.language) {
      params.append("language", this.config.language);
    }

    if (this.config.region) {
      params.append("region", this.config.region);
    }

    return `${baseUrl}?${params.toString()}`;
  }
}
```

### **🎯 Provider Configuration**

```typescript
// src/app/app.module.ts
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";

import { AppComponent } from "./app.component";
import { DashboardComponent } from "./components/dashboard/dashboard.component";

// 🔌 Import tokens and services
import {
  GOOGLE_MAPS_CONFIG,
  ANALYTICS_CONFIG,
  GoogleMapsConfig,
} from "./tokens/third-party.tokens";

import { ChartService } from "./services/chart.service";
import { GoogleMapsService } from "./services/google-maps.service";

// 📊 Environment-based configuration
import { environment } from "../environments/environment";

@NgModule({
  declarations: [AppComponent, DashboardComponent],
  imports: [BrowserModule],
  providers: [
    // 🗺️ Google Maps Configuration
    {
      provide: GOOGLE_MAPS_CONFIG,
      useValue: {
        apiKey: environment.googleMapsApiKey,
        libraries: ["places", "geometry"],
        language: "en",
        region: "US",
      } as GoogleMapsConfig,
    },

    // 📊 Analytics Configuration
    {
      provide: ANALYTICS_CONFIG,
      useValue: {
        trackingId: environment.googleAnalyticsId,
        enableTracing: !environment.production,
        anonymizeIp: true,
      },
    },

    // 🔧 Services
    ChartService,
    GoogleMapsService,
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

---

## 📱 **Real-World Integration Examples**

### **🎨 jQuery Plugin Integration**

```typescript
// src/app/directives/datepicker.directive.ts
import {
  Directive,
  ElementRef,
  Input,
  Output,
  EventEmitter,
  OnInit,
  OnDestroy,
  Inject,
  PLATFORM_ID,
} from "@angular/core";
import { isPlatformBrowser } from "@angular/common";

// 📅 jQuery & Datepicker types
declare var $: any;

@Directive({
  selector: "[appDatepicker]",
})
export class DatepickerDirective implements OnInit, OnDestroy {
  @Input() dateFormat = "yyyy-mm-dd";
  @Input() minDate?: Date;
  @Input() maxDate?: Date;
  @Input() disabled = false;

  @Output() dateChange = new EventEmitter<Date>();
  @Output() dateSelect = new EventEmitter<Date>();

  private $element: any;
  private isInitialized = false;

  constructor(
    private elementRef: ElementRef,
    @Inject(PLATFORM_ID) private platformId: Object
  ) {}

  ngOnInit(): void {
    if (isPlatformBrowser(this.platformId)) {
      this.initializeDatepicker();
    }
  }

  ngOnDestroy(): void {
    if (this.isInitialized && this.$element) {
      this.$element.datepicker("destroy");
    }
  }

  private async initializeDatepicker(): Promise<void> {
    try {
      // Ensure jQuery and datepicker are loaded
      await this.ensureLibrariesLoaded();

      this.$element = $(this.elementRef.nativeElement);

      const options = {
        dateFormat: this.dateFormat,
        minDate: this.minDate,
        maxDate: this.maxDate,
        disabled: this.disabled,
        onSelect: (dateText: string, inst: any) => {
          const date = new Date(dateText);
          this.dateSelect.emit(date);
        },
        onChangeMonthYear: (year: number, month: number, inst: any) => {
          // Handle month/year changes if needed
        },
      };

      this.$element.datepicker(options);
      this.isInitialized = true;

      // Listen for changes
      this.$element.on("change", (event: any) => {
        const value = event.target.value;
        if (value) {
          this.dateChange.emit(new Date(value));
        }
      });
    } catch (error) {
      console.error("Failed to initialize datepicker:", error);
    }
  }

  private async ensureLibrariesLoaded(): Promise<void> {
    // Check if jQuery is loaded
    if (typeof $ === "undefined") {
      await this.loadScript("https://code.jquery.com/jquery-3.6.0.min.js");
    }

    // Check if datepicker is loaded
    if (!$.fn.datepicker) {
      await this.loadScript(
        "https://code.jquery.com/ui/1.13.0/jquery-ui.min.js"
      );
      await this.loadStylesheet(
        "https://code.jquery.com/ui/1.13.0/themes/ui-lightness/jquery-ui.css"
      );
    }
  }

  private loadScript(src: string): Promise<void> {
    return new Promise((resolve, reject) => {
      if (document.querySelector(`script[src="${src}"]`)) {
        resolve();
        return;
      }

      const script = document.createElement("script");
      script.src = src;
      script.onload = () => resolve();
      script.onerror = () => reject(new Error(`Failed to load script: ${src}`));
      document.head.appendChild(script);
    });
  }

  private loadStylesheet(href: string): Promise<void> {
    return new Promise((resolve) => {
      if (document.querySelector(`link[href="${href}"]`)) {
        resolve();
        return;
      }

      const link = document.createElement("link");
      link.rel = "stylesheet";
      link.href = href;
      link.onload = () => resolve();
      document.head.appendChild(link);
    });
  }
}
```

---

## 🎉 **Summary: Third-Party Integration Mastery**

### **✅ What We've Covered:**

🔌 **Integration Methods** - NPM, CDN, and manual approaches  
💉 **Dependency Injection** - Service wrappers and token-based injection  
📊 **Chart.js Integration** - Complete service with lifecycle management  
🗺️ **Google Maps Integration** - API loading and configuration  
📅 **jQuery Plugin Integration** - Directive-based approach

### **🎯 Best Practices:**

- ✅ **Platform Checks** - Use `isPlatformBrowser` for SSR compatibility
- ✅ **Lazy Loading** - Load libraries only when needed
- ✅ **Memory Management** - Proper cleanup to prevent leaks
- ✅ **Error Handling** - Graceful fallbacks for library failures
- ✅ **TypeScript Support** - Create types or use `@types` packages
- ✅ **Bundle Optimization** - Consider library impact on app size

**You're now ready to integrate any third-party library into Angular!** 🚀
