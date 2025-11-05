# 🚀 Angular 19 DX Enhancements - Developer Experience Guide

## 📋 Table of Contents

1. [New Lifecycle Hooks](#new-lifecycle-hooks)
2. [Material Design V18](#material-design-v18)
3. [Enhanced DevTools](#enhanced-devtools)
4. [Standalone APIs](#standalone-apis)

---

## 🔄 New Lifecycle Hooks {#new-lifecycle-hooks}

Angular 19 introduces several new lifecycle hooks that provide better control over component and directive behavior, especially with the new signal-based architecture.

### **afterRenderEffect() Hook**

This new hook allows you to run side effects after the component has been rendered to the DOM, similar to `useEffect` in React but specifically for post-render operations.

```typescript
import { Component, afterRenderEffect, signal } from "@angular/core";

@Component({
  selector: "app-chart-component",
  template: `
    <div #chartContainer class="chart-container">
      <canvas #chartCanvas></canvas>
    </div>
  `,
})
export class ChartComponent {
  private chartData = signal([1, 2, 3, 4, 5]);

  constructor() {
    // Runs after every render cycle
    afterRenderEffect(() => {
      // Safe to access DOM elements here
      this.updateChart();
    });
  }

  private updateChart() {
    // DOM is guaranteed to be updated here
    console.log("Chart rendered with data:", this.chartData());
    // Initialize or update chart library
    this.initializeChartJS();
  }

  updateData(newData: number[]) {
    this.chartData.set(newData);
    // afterRenderEffect will automatically trigger after this update
  }
}
```

### **afterNextRender() Hook**

Execute code after the next render cycle - useful for one-time DOM operations.

```typescript
import {
  Component,
  afterNextRender,
  ViewChild,
  ElementRef,
} from "@angular/core";

@Component({
  selector: "app-focus-input",
  template: `
    <input #inputElement type="text" placeholder="Auto-focus input" />
    <button (click)="resetAndFocus()">Reset & Focus</button>
  `,
})
export class FocusInputComponent {
  @ViewChild("inputElement") inputElement!: ElementRef<HTMLInputElement>;

  constructor() {
    // Runs only after the next render
    afterNextRender(() => {
      // Focus the input after initial render
      this.inputElement.nativeElement.focus();
    });
  }

  resetAndFocus() {
    this.inputElement.nativeElement.value = "";

    // Schedule focus for after next render
    afterNextRender(() => {
      this.inputElement.nativeElement.focus();
    });
  }
}
```

### **Lifecycle with Signals Integration**

New hooks work seamlessly with Angular's signal-based reactivity:

```typescript
import {
  Component,
  signal,
  computed,
  effect,
  afterRenderEffect,
} from "@angular/core";

@Component({
  selector: "app-reactive-component",
  template: `
    <div class="metrics-dashboard">
      <div class="counter">Count: {{ count() }}</div>
      <div class="doubled">Doubled: {{ doubled() }}</div>
      <div class="status" [class]="statusClass()">{{ status() }}</div>

      <button (click)="increment()">Increment</button>
      <button (click)="reset()">Reset</button>
    </div>
  `,
})
export class ReactiveComponent {
  // Signals
  count = signal(0);
  status = signal("idle");

  // Computed signals
  doubled = computed(() => this.count() * 2);
  statusClass = computed(() => {
    const currentStatus = this.status();
    return {
      "status-idle": currentStatus === "idle",
      "status-active": currentStatus === "active",
      "status-complete": currentStatus === "complete",
    };
  });

  constructor() {
    // Effect runs when signals change
    effect(() => {
      console.log("Count changed to:", this.count());

      // Update status based on count
      if (this.count() === 0) {
        this.status.set("idle");
      } else if (this.count() < 10) {
        this.status.set("active");
      } else {
        this.status.set("complete");
      }
    });

    // Runs after each render when DOM is updated
    afterRenderEffect(() => {
      // DOM operations after signals update the view
      this.updateProgressBar();
      this.logDOMState();
    });
  }

  increment() {
    this.count.update((current) => current + 1);
  }

  reset() {
    this.count.set(0);
  }

  private updateProgressBar() {
    // Safe DOM manipulation after render
    const progressBar = document.querySelector(".progress-bar");
    if (progressBar) {
      const percentage = Math.min(this.count() * 10, 100);
      progressBar.setAttribute("style", `width: ${percentage}%`);
    }
  }

  private logDOMState() {
    console.log(
      "DOM updated - Count in DOM:",
      document.querySelector(".counter")?.textContent
    );
  }
}
```

---

## 🎨 Material Design V18 {#material-design-v18}

Angular Material Design V18 introduces Material Design 3 (Material You) with enhanced theming, new components, and improved accessibility.

### **Material Design 3 Theming**

The new theming system provides more flexible color schemes and typography:

```typescript
// theme.scss - Material Design 3 Theme Configuration
@use '@angular/material' as mat;

// Define your color palette based on Material Design 3
$primary-palette: mat.define-palette(mat.$azure-palette, 500);
$accent-palette: mat.define-palette(mat.$rose-palette, 200);
$warn-palette: mat.define-palette(mat.$red-palette);

// Create Material Design 3 theme
$theme: mat.define-theme((
  color: (
    theme-type: light,
    primary: $primary-palette,
    tertiary: $accent-palette,
  ),
  typography: (
    brand-family: 'Inter, sans-serif',
    plain-family: 'Roboto, sans-serif',
  ),
  density: (
    scale: 0,
  )
));

// Apply the theme
@include mat.all-component-themes($theme);

// Material Design 3 specific mixins
@include mat.system-level-colors($theme);
@include mat.system-level-typography($theme);
```

### **New Material Components**

Enhanced components with Material Design 3 principles:

```typescript
// Enhanced Material Components Example
@Component({
  selector: "app-material-showcase",
  template: `
    <div class="material-showcase">
      <!-- Enhanced Cards with Material Design 3 -->
      <mat-card class="product-card" appearance="elevated">
        <mat-card-header>
          <div mat-card-avatar class="product-avatar">
            <mat-icon>shopping_bag</mat-icon>
          </div>
          <mat-card-title>Premium Product</mat-card-title>
          <mat-card-subtitle>Latest in Material Design 3</mat-card-subtitle>
        </mat-card-header>

        <img mat-card-image src="product-image.jpg" alt="Product" />

        <mat-card-content>
          <p>Enhanced card design with better elevation and surface colors.</p>

          <!-- New Chip components -->
          <mat-chip-set aria-label="Product tags">
            <mat-chip selected>Premium</mat-chip>
            <mat-chip>Featured</mat-chip>
            <mat-chip disabled>Limited</mat-chip>
          </mat-chip-set>
        </mat-card-content>

        <mat-card-actions>
          <!-- Enhanced buttons with Material Design 3 -->
          <button mat-button color="primary">LEARN MORE</button>
          <button mat-raised-button color="accent">ADD TO CART</button>
        </mat-card-actions>
      </mat-card>

      <!-- Enhanced Form Components -->
      <form class="material-form">
        <mat-form-field appearance="outline">
          <mat-label>Product Name</mat-label>
          <input matInput placeholder="Enter product name" />
          <mat-icon matSuffix>edit</mat-icon>
          <mat-hint>Choose a descriptive name</mat-hint>
        </mat-form-field>

        <!-- New Select with enhanced UX -->
        <mat-form-field appearance="fill">
          <mat-label>Category</mat-label>
          <mat-select>
            <mat-option value="electronics">Electronics</mat-option>
            <mat-option value="clothing">Clothing</mat-option>
            <mat-option value="books">Books</mat-option>
          </mat-select>
        </mat-form-field>

        <!-- Enhanced Date Picker -->
        <mat-form-field appearance="outline">
          <mat-label>Launch Date</mat-label>
          <input matInput [matDatepicker]="picker" />
          <mat-datepicker-toggle
            matIconSuffix
            [for]="picker"
          ></mat-datepicker-toggle>
          <mat-datepicker #picker></mat-datepicker>
        </mat-form-field>
      </form>

      <!-- Enhanced Navigation Components -->
      <mat-tab-group animationDuration="300ms" dynamicHeight>
        <mat-tab label="Overview">
          <div class="tab-content">
            <h3>Product Overview</h3>
            <p>
              Enhanced tabs with smoother animations and better accessibility.
            </p>
          </div>
        </mat-tab>

        <mat-tab label="Specifications">
          <div class="tab-content">
            <h3>Technical Specifications</h3>
            <mat-list>
              <mat-list-item>
                <mat-icon matListItemIcon>memory</mat-icon>
                <div matListItemTitle>Memory: 16GB RAM</div>
                <div matListItemLine>High-performance memory</div>
              </mat-list-item>

              <mat-list-item>
                <mat-icon matListItemIcon>storage</mat-icon>
                <div matListItemTitle>Storage: 512GB SSD</div>
                <div matListItemLine>Fast solid-state drive</div>
              </mat-list-item>
            </mat-list>
          </div>
        </mat-tab>

        <mat-tab label="Reviews" [disabled]="true">
          <div class="tab-content">
            <p>Coming soon...</p>
          </div>
        </mat-tab>
      </mat-tab-group>
    </div>
  `,
  styles: [
    `
      .material-showcase {
        padding: 24px;
        display: grid;
        gap: 24px;
        max-width: 800px;
      }

      .product-card {
        max-width: 400px;
      }

      .product-avatar {
        background-color: var(--mat-sys-primary-container);
        color: var(--mat-sys-on-primary-container);
      }

      .material-form {
        display: flex;
        flex-direction: column;
        gap: 16px;
      }

      .tab-content {
        padding: 16px;
        min-height: 200px;
      }
    `,
  ],
})
export class MaterialShowcaseComponent {}
```

### **Enhanced Accessibility Features**

Material Design V18 includes improved accessibility:

```typescript
// Accessibility-Enhanced Component
@Component({
  selector: "app-accessible-table",
  template: `
    <div class="accessible-table-container">
      <!-- Enhanced Table with better accessibility -->
      <table
        mat-table
        [dataSource]="dataSource"
        class="mat-elevation-z2"
        role="table"
        aria-label="Product inventory table"
      >
        <!-- Selection Column -->
        <ng-container matColumnDef="select">
          <th mat-header-cell *matHeaderCellDef>
            <mat-checkbox
              (change)="$event ? toggleAllRows() : null"
              [checked]="selection.hasValue() && isAllSelected()"
              [indeterminate]="selection.hasValue() && !isAllSelected()"
              [aria-label]="checkboxLabel()"
            >
            </mat-checkbox>
          </th>
          <td mat-cell *matCellDef="let row">
            <mat-checkbox
              (click)="$event.stopPropagation()"
              (change)="$event ? selection.toggle(row) : null"
              [checked]="selection.isSelected(row)"
              [aria-label]="checkboxLabel(row)"
            >
            </mat-checkbox>
          </td>
        </ng-container>

        <!-- Product Name Column -->
        <ng-container matColumnDef="name">
          <th
            mat-header-cell
            *matHeaderCellDef
            mat-sort-header
            aria-label="Sort by product name"
          >
            Product Name
          </th>
          <td mat-cell *matCellDef="let element">
            <div class="product-name-cell">
              <span>{{ element.name }}</span>
              <mat-icon
                class="status-icon"
                [attr.aria-label]="getStatusLabel(element.status)"
                [class]="'status-' + element.status"
              >
                {{ getStatusIcon(element.status) }}
              </mat-icon>
            </div>
          </td>
        </ng-container>

        <!-- Price Column -->
        <ng-container matColumnDef="price">
          <th mat-header-cell *matHeaderCellDef mat-sort-header>Price</th>
          <td mat-cell *matCellDef="let element">
            {{ element.price | currency : "USD" : "symbol" : "1.2-2" }}
          </td>
        </ng-container>

        <!-- Actions Column -->
        <ng-container matColumnDef="actions">
          <th mat-header-cell *matHeaderCellDef>Actions</th>
          <td mat-cell *matCellDef="let element">
            <button
              mat-icon-button
              [matMenuTriggerFor]="actionMenu"
              aria-label="More actions for {{ element.name }}"
            >
              <mat-icon>more_vert</mat-icon>
            </button>

            <mat-menu #actionMenu="matMenu">
              <button mat-menu-item (click)="editProduct(element)">
                <mat-icon>edit</mat-icon>
                <span>Edit</span>
              </button>
              <button mat-menu-item (click)="duplicateProduct(element)">
                <mat-icon>content_copy</mat-icon>
                <span>Duplicate</span>
              </button>
              <button
                mat-menu-item
                (click)="deleteProduct(element)"
                class="delete-action"
              >
                <mat-icon>delete</mat-icon>
                <span>Delete</span>
              </button>
            </mat-menu>
          </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr
          mat-row
          *matRowDef="let row; columns: displayedColumns"
          (click)="selection.toggle(row)"
          [class.selected-row]="selection.isSelected(row)"
          role="row"
          [attr.aria-selected]="selection.isSelected(row)"
        ></tr>
      </table>

      <!-- Enhanced Paginator with better accessibility -->
      <mat-paginator
        [pageSizeOptions]="[5, 10, 20]"
        showFirstLastButtons
        aria-label="Select page of products"
      >
      </mat-paginator>
    </div>
  `,
  styles: [
    `
      .accessible-table-container {
        width: 100%;
        margin: 16px 0;
      }

      .product-name-cell {
        display: flex;
        align-items: center;
        gap: 8px;
      }

      .status-icon.status-active {
        color: var(--mat-sys-success);
      }

      .status-icon.status-inactive {
        color: var(--mat-sys-error);
      }

      .selected-row {
        background-color: var(--mat-sys-primary-container);
      }

      .delete-action {
        color: var(--mat-sys-error);
      }
    `,
  ],
})
export class AccessibleTableComponent {
  displayedColumns: string[] = ["select", "name", "price", "actions"];
  dataSource = new MatTableDataSource(PRODUCT_DATA);
  selection = new SelectionModel<Product>(true, []);

  constructor() {
    // Enhanced keyboard navigation
    this.setupKeyboardNavigation();
  }

  isAllSelected() {
    const numSelected = this.selection.selected.length;
    const numRows = this.dataSource.data.length;
    return numSelected === numRows;
  }

  toggleAllRows() {
    if (this.isAllSelected()) {
      this.selection.clear();
      return;
    }
    this.selection.select(...this.dataSource.data);
  }

  checkboxLabel(row?: Product): string {
    if (!row) {
      return `${this.isAllSelected() ? "deselect" : "select"} all`;
    }
    return `${this.selection.isSelected(row) ? "deselect" : "select"} row ${
      row.name
    }`;
  }

  getStatusLabel(status: string): string {
    const labels = {
      active: "Product is active and available",
      inactive: "Product is inactive or out of stock",
      pending: "Product status is pending review",
    };
    return labels[status] || "Unknown status";
  }

  getStatusIcon(status: string): string {
    const icons = {
      active: "check_circle",
      inactive: "cancel",
      pending: "schedule",
    };
    return icons[status] || "help";
  }

  private setupKeyboardNavigation() {
    // Enhanced keyboard navigation for better accessibility
    // Implementation details for custom keyboard shortcuts
  }
}
```

---

## 🛠️ Enhanced DevTools {#enhanced-devtools}

Angular 19 includes significantly improved developer tools with better debugging capabilities, performance profiling, and signal inspection.

### **Signal Debugging in DevTools**

New DevTools provide deep insights into signal-based state management:

```typescript
// Component optimized for DevTools debugging
@Component({
  selector: "app-debug-signals",
  template: `
    <div class="debug-dashboard">
      <h2>Signal Debugging Demo</h2>

      <!-- Signal values displayed -->
      <div class="signal-display">
        <p>User Count: {{ userCount() }}</p>
        <p>Active Users: {{ activeUsers() }}</p>
        <p>Completion Rate: {{ completionRate() }}%</p>
        <p>Status: {{ status() }}</p>
      </div>

      <!-- Control buttons -->
      <div class="controls">
        <button (click)="addUser()">Add User</button>
        <button (click)="removeUser()">Remove User</button>
        <button (click)="toggleUserStatus()">Toggle Status</button>
        <button (click)="simulateActivity()">Simulate Activity</button>
      </div>

      <!-- Debug information -->
      <div class="debug-info" *ngIf="debugMode()">
        <h3>Debug Information</h3>
        <pre>{{ getDebugInfo() | json }}</pre>
      </div>
    </div>
  `,
  // DevTools will show this component in the component tree
  // with signal inspection capabilities
})
export class DebugSignalsComponent {
  // These signals will be visible in Angular DevTools
  userCount = signal(0);
  activeUserCount = signal(0);
  debugMode = signal(false);

  // Computed signals show dependency graphs in DevTools
  activeUsers = computed(() => {
    console.log("🔄 Computing active users"); // DevTools captures this
    return Math.min(this.activeUserCount(), this.userCount());
  });

  completionRate = computed(() => {
    console.log("🔄 Computing completion rate"); // DevTools captures this
    const total = this.userCount();
    const active = this.activeUsers();
    return total > 0 ? Math.round((active / total) * 100) : 0;
  });

  status = computed(() => {
    const rate = this.completionRate();
    if (rate >= 80) return "Excellent";
    if (rate >= 60) return "Good";
    if (rate >= 40) return "Average";
    return "Needs Improvement";
  });

  constructor() {
    // Effects are tracked in DevTools with execution timeline
    effect(() => {
      console.log("👥 User count changed:", this.userCount());
      // DevTools shows when this effect runs and what triggered it
    });

    effect(() => {
      console.log("📊 Completion rate updated:", this.completionRate());
      // DevTools shows the signal dependency chain
    });

    // DevTools can show effect cleanup and re-execution
    effect((onCleanup) => {
      const subscription = this.setupRealtimeUpdates();

      onCleanup(() => {
        subscription.unsubscribe();
        console.log("🧹 Cleaned up realtime updates");
      });
    });
  }

  addUser() {
    this.userCount.update((count) => count + 1);
    // DevTools shows signal update and propagation
  }

  removeUser() {
    this.userCount.update((count) => Math.max(0, count - 1));
    this.activeUserCount.update((count) =>
      Math.max(0, Math.min(count, this.userCount()))
    );
  }

  toggleUserStatus() {
    this.activeUserCount.update((count) =>
      count === this.userCount() ? Math.floor(count * 0.7) : this.userCount()
    );
  }

  simulateActivity() {
    // DevTools can track this sequence of signal updates
    const interval = setInterval(() => {
      this.activeUserCount.update((count) =>
        Math.min(this.userCount(), count + Math.floor(Math.random() * 3))
      );
    }, 500);

    setTimeout(() => clearInterval(interval), 3000);
  }

  getDebugInfo() {
    return {
      userCount: this.userCount(),
      activeUserCount: this.activeUserCount(),
      activeUsers: this.activeUsers(),
      completionRate: this.completionRate(),
      status: this.status(),
      timestamp: new Date().toISOString(),
    };
  }

  private setupRealtimeUpdates() {
    // Simulate real-time updates for DevTools debugging
    return {
      unsubscribe: () => console.log("Unsubscribed from updates"),
    };
  }
}
```

### **Performance Profiling Integration**

Enhanced performance profiling with detailed insights:

```typescript
// Performance monitoring component
@Component({
  selector: "app-performance-monitor",
  template: `
    <div class="performance-monitor">
      <h2>Performance Monitoring</h2>

      <!-- Performance metrics displayed -->
      <div class="metrics-grid">
        <div class="metric-card">
          <h3>Render Time</h3>
          <span class="metric-value">{{ renderTime() }}ms</span>
        </div>

        <div class="metric-card">
          <h3>Signal Updates</h3>
          <span class="metric-value">{{ signalUpdates() }}</span>
        </div>

        <div class="metric-card">
          <h3>Effect Executions</h3>
          <span class="metric-value">{{ effectExecutions() }}</span>
        </div>

        <div class="metric-card">
          <h3>Component Updates</h3>
          <span class="metric-value">{{ componentUpdates() }}</span>
        </div>
      </div>

      <!-- Performance test controls -->
      <div class="controls">
        <button (click)="runPerformanceTest()">Run Performance Test</button>
        <button (click)="triggerHeavyOperation()">
          Trigger Heavy Operation
        </button>
        <button (click)="resetMetrics()">Reset Metrics</button>
      </div>

      <!-- Performance chart -->
      <div class="performance-chart" #chartContainer></div>
    </div>
  `,
})
export class PerformanceMonitorComponent implements OnInit, AfterViewInit {
  // Signals for performance metrics (visible in DevTools)
  renderTime = signal(0);
  signalUpdates = signal(0);
  effectExecutions = signal(0);
  componentUpdates = signal(0);

  private performanceObserver?: PerformanceObserver;

  constructor() {
    // DevTools can track the performance impact of these effects
    effect(() => {
      // This effect execution is tracked in DevTools
      this.effectExecutions.update((count) => count + 1);
    });

    // Performance tracking effect
    effect(() => {
      const updateCount = this.componentUpdates();
      if (updateCount > 0) {
        console.log("📊 Component updated", updateCount, "times");
        // DevTools shows performance impact of frequent updates
      }
    });
  }

  ngOnInit() {
    this.setupPerformanceObserver();
  }

  ngAfterViewInit() {
    // DevTools shows lifecycle timing
    console.log("🔧 Component fully initialized");
  }

  runPerformanceTest() {
    console.log("🚀 Starting performance test");
    const startTime = performance.now();

    // Simulate heavy operations that DevTools can profile
    for (let i = 0; i < 1000; i++) {
      this.signalUpdates.update((count) => count + 1);
    }

    const endTime = performance.now();
    this.renderTime.set(Math.round(endTime - startTime));

    console.log("✅ Performance test completed");
  }

  triggerHeavyOperation() {
    // DevTools can profile this operation
    performance.mark("heavy-operation-start");

    // Simulate complex computation
    const result = this.heavyComputation();

    performance.mark("heavy-operation-end");
    performance.measure(
      "heavy-operation",
      "heavy-operation-start",
      "heavy-operation-end"
    );

    this.componentUpdates.update((count) => count + 1);

    console.log("💪 Heavy operation completed:", result);
  }

  resetMetrics() {
    this.renderTime.set(0);
    this.signalUpdates.set(0);
    this.effectExecutions.set(0);
    this.componentUpdates.set(0);

    console.log("🔄 Metrics reset");
  }

  private setupPerformanceObserver() {
    if ("PerformanceObserver" in window) {
      this.performanceObserver = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.name === "heavy-operation") {
            console.log("⏱️ Heavy operation took:", entry.duration, "ms");
            // DevTools integration for custom performance marks
          }
        }
      });

      this.performanceObserver.observe({ entryTypes: ["measure"] });
    }
  }

  private heavyComputation(): number {
    // Simulate CPU-intensive task
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += Math.sqrt(i);
    }
    return result;
  }
}
```

### **DevTools Component Inspector**

Enhanced component inspection with signal state visualization:

```typescript
// Component designed for DevTools inspection
@Component({
  selector: "app-inspectable-component",
  template: `
    <div class="component-inspector-demo">
      <!-- DevTools shows complete component state -->
      <div class="state-display">
        <h3>Component State (Visible in DevTools)</h3>
        <ul>
          <li>Loading: {{ isLoading() }}</li>
          <li>Error: {{ error() || "None" }}</li>
          <li>Data Count: {{ data().length }}</li>
          <li>Selected Items: {{ selectedItems().length }}</li>
        </ul>
      </div>

      <!-- DevTools can inspect event handlers -->
      <div class="interaction-controls">
        <button (click)="loadData()" [disabled]="isLoading()">
          {{ isLoading() ? "Loading..." : "Load Data" }}
        </button>

        <button (click)="clearData()" [disabled]="data().length === 0">
          Clear Data
        </button>

        <button (click)="simulateError()">Simulate Error</button>
      </div>

      <!-- DevTools shows directive and component relationships -->
      <div class="data-list" *ngIf="data().length > 0">
        <div
          *ngFor="let item of data(); trackBy: trackByFn; let i = index"
          class="data-item"
          [class.selected]="isSelected(item)"
          (click)="toggleSelection(item)"
        >
          <span>{{ item.name }}</span>
          <span class="item-index">{{ i }}</span>
        </div>
      </div>

      <!-- Error display -->
      <div class="error-display" *ngIf="error()">
        <mat-icon>error</mat-icon>
        <span>{{ error() }}</span>
        <button (click)="clearError()">Clear Error</button>
      </div>
    </div>
  `,
  // DevTools provides detailed component metadata
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class InspectableComponent {
  // All signals are inspectable in DevTools
  isLoading = signal(false);
  error = signal<string | null>(null);
  data = signal<DataItem[]>([]);
  selectedItems = signal<DataItem[]>([]);

  // Computed signals show dependency graphs
  hasData = computed(() => this.data().length > 0);
  selectionCount = computed(() => this.selectedItems().length);
  isAllSelected = computed(
    () => this.hasData() && this.selectionCount() === this.data().length
  );

  constructor(private dataService: DataService) {
    // Effects are tracked with their dependencies
    effect(() => {
      console.log("📊 Data changed:", this.data().length, "items");
      // DevTools shows what triggered this effect
    });

    effect(() => {
      const errorState = this.error();
      if (errorState) {
        console.error("❌ Error occurred:", errorState);
        // DevTools highlights error states
      }
    });
  }

  async loadData() {
    this.isLoading.set(true);
    this.error.set(null);

    try {
      // DevTools can track async operations
      const result = await this.dataService.fetchData();
      this.data.set(result);
      console.log("✅ Data loaded successfully");
    } catch (err) {
      this.error.set(err instanceof Error ? err.message : "Unknown error");
      console.error("❌ Failed to load data:", err);
    } finally {
      this.isLoading.set(false);
    }
  }

  clearData() {
    this.data.set([]);
    this.selectedItems.set([]);
    console.log("🗑️ Data cleared");
  }

  simulateError() {
    this.error.set("Simulated error for DevTools debugging");
    console.error("🚫 Simulated error triggered");
  }

  clearError() {
    this.error.set(null);
    console.log("✅ Error cleared");
  }

  toggleSelection(item: DataItem) {
    const selected = this.selectedItems();
    const index = selected.findIndex((s) => s.id === item.id);

    if (index >= 0) {
      // Remove from selection
      this.selectedItems.update((items) =>
        items.filter((s) => s.id !== item.id)
      );
    } else {
      // Add to selection
      this.selectedItems.update((items) => [...items, item]);
    }

    console.log(
      "🎯 Selection changed:",
      this.selectionCount(),
      "items selected"
    );
  }

  isSelected(item: DataItem): boolean {
    return this.selectedItems().some((s) => s.id === item.id);
  }

  trackByFn(index: number, item: DataItem): any {
    return item.id; // DevTools can optimize change detection tracking
  }
}

interface DataItem {
  id: number;
  name: string;
  value: any;
}
```

---

## 🏗️ Standalone APIs {#standalone-apis}

Angular 19 enhances standalone APIs with improved bootstrapping, better tree-shaking, and simplified application architecture.

### **Enhanced Standalone Bootstrapping**

Simplified application setup with better performance:

```typescript
// main.ts - Enhanced standalone bootstrapping
import { bootstrapApplication } from "@angular/platform-browser";
import { provideRouter } from "@angular/router";
import { provideHttpClient, withInterceptors } from "@angular/common/http";
import { provideAnimations } from "@angular/platform-browser/animations";
import { provideServiceWorker } from "@angular/service-worker";
import { importProvidersFrom } from "@angular/core";

// Enhanced standalone app component
import { AppComponent } from "./app/app.component";
import { routes } from "./app/app.routes";

// Enhanced provider configuration
bootstrapApplication(AppComponent, {
  providers: [
    // Router with enhanced configuration
    provideRouter(
      routes,
      withEnabledBlockingInitialNavigation(),
      withInMemoryScrolling({
        scrollPositionRestoration: "enabled",
        anchorScrolling: "enabled",
      }),
      withRouterConfig({
        onSameUrlNavigation: "reload",
      })
    ),

    // HTTP client with enhanced features
    provideHttpClient(
      withInterceptors([
        authInterceptor,
        errorInterceptor,
        loadingInterceptor,
        cacheInterceptor,
      ]),
      withFetch() // Use fetch API for better performance
    ),

    // Animations with enhanced performance
    provideAnimations(),

    // Service Worker with enhanced caching
    provideServiceWorker("ngsw-worker.js", {
      enabled: environment.production,
      registrationStrategy: "registerWhenStable:30000",
    }),

    // Enhanced Material providers
    importProvidersFrom([
      MatDialogModule,
      MatSnackBarModule,
      MatDatepickerModule,
    ]),

    // Custom standalone providers
    provideAppConfig(),
    provideAnalytics(),
    provideErrorHandling(),
  ],
}).catch((err) => console.error(err));
```

### **Standalone Component Architecture**

Complete standalone component with all dependencies:

```typescript
// Enhanced standalone component
@Component({
  selector: "app-product-catalog",
  standalone: true,
  imports: [
    // Angular common modules
    CommonModule,
    ReactiveFormsModule,
    RouterModule,

    // Material Design modules
    MatCardModule,
    MatButtonModule,
    MatIconModule,
    MatInputModule,
    MatSelectModule,
    MatChipsModule,
    MatPaginatorModule,
    MatProgressSpinnerModule,
    MatSnackBarModule,

    // Custom standalone components
    ProductCardComponent,
    FilterBarComponent,
    SearchBoxComponent,
    LoadingSpinnerComponent,

    // Custom standalone directives
    LazyLoadDirective,
    InfiniteScrollDirective,

    // Custom standalone pipes
    CurrencyFormatterPipe,
    HighlightSearchPipe,
  ],
  providers: [
    // Component-level services
    ProductService,
    CartService,
    AnalyticsService,

    // Component-level configurations
    {
      provide: PAGINATION_CONFIG,
      useValue: {
        pageSize: 24,
        pageSizeOptions: [12, 24, 48, 96],
      },
    },
  ],
  template: `
    <div class="product-catalog">
      <!-- Enhanced search with standalone components -->
      <app-search-box
        [placeholder]="'Search products...'"
        (searchQuery)="onSearch($event)"
        (suggestions)="onSuggestions($event)"
      >
      </app-search-box>

      <!-- Standalone filter bar -->
      <app-filter-bar
        [categories]="categories()"
        [priceRange]="priceRange()"
        [selectedFilters]="selectedFilters()"
        (filtersChanged)="onFiltersChanged($event)"
      >
      </app-filter-bar>

      <!-- Product grid with enhanced features -->
      <div
        class="product-grid"
        appInfiniteScroll
        (scrolled)="loadMoreProducts()"
      >
        @for (product of products(); track product.id) {
        <app-product-card
          [product]="product"
          [isLoading]="loadingStates().get(product.id)"
          appLazyLoad
          (addToCart)="addToCart($event)"
          (addToWishlist)="addToWishlist($event)"
          (productClick)="navigateToProduct($event)"
        >
        </app-product-card>
        } @empty {
        <div class="empty-state">
          <mat-icon>inventory_2</mat-icon>
          <h3>No products found</h3>
          <p>Try adjusting your search criteria</p>
          <button mat-raised-button color="primary" (click)="clearFilters()">
            Clear Filters
          </button>
        </div>
        }
      </div>

      <!-- Enhanced pagination -->
      <mat-paginator
        [length]="totalProducts()"
        [pageSize]="pageSize()"
        [pageSizeOptions]="pageSizeOptions"
        (page)="onPageChange($event)"
        showFirstLastButtons
      >
      </mat-paginator>

      <!-- Loading spinner -->
      @if (isLoading()) {
      <app-loading-spinner [message]="'Loading products...'">
      </app-loading-spinner>
      }
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ProductCatalogComponent {
  // Enhanced signal-based state
  products = signal<Product[]>([]);
  categories = signal<Category[]>([]);
  selectedFilters = signal<ProductFilters>({});
  isLoading = signal(false);
  loadingStates = signal(new Map<number, boolean>());

  // Computed properties
  totalProducts = computed(() => this.productService.getTotalCount());
  pageSize = computed(() => this.selectedFilters().pageSize || 24);
  priceRange = computed(() => {
    const products = this.products();
    if (products.length === 0) return { min: 0, max: 0 };

    const prices = products.map((p) => p.price);
    return {
      min: Math.min(...prices),
      max: Math.max(...prices),
    };
  });

  // Configuration
  pageSizeOptions = [12, 24, 48, 96];

  constructor(
    private productService: ProductService,
    private cartService: CartService,
    private router: Router,
    private snackBar: MatSnackBar,
    @Inject(PAGINATION_CONFIG) private paginationConfig: PaginationConfig
  ) {
    // Initialize component
    this.loadInitialData();
  }

  async loadInitialData() {
    this.isLoading.set(true);

    try {
      const [products, categories] = await Promise.all([
        this.productService.getProducts(),
        this.productService.getCategories(),
      ]);

      this.products.set(products);
      this.categories.set(categories);
    } catch (error) {
      this.handleError("Failed to load products", error);
    } finally {
      this.isLoading.set(false);
    }
  }

  onSearch(query: string) {
    this.selectedFilters.update((filters) => ({
      ...filters,
      searchQuery: query,
      page: 0, // Reset to first page
    }));

    this.loadProducts();
  }

  onFiltersChanged(filters: ProductFilters) {
    this.selectedFilters.set({
      ...filters,
      page: 0, // Reset to first page
    });

    this.loadProducts();
  }

  async addToCart(product: Product) {
    // Set loading state for specific product
    this.loadingStates.update((states) =>
      new Map(states).set(product.id, true)
    );

    try {
      await this.cartService.addToCart({
        productId: product.id,
        quantity: 1,
      });

      this.snackBar.open(`${product.name} added to cart!`, "View Cart", {
        duration: 3000,
      });
    } catch (error) {
      this.handleError("Failed to add to cart", error);
    } finally {
      this.loadingStates.update((states) => {
        const newStates = new Map(states);
        newStates.delete(product.id);
        return newStates;
      });
    }
  }

  async addToWishlist(product: Product) {
    try {
      await this.productService.addToWishlist(product.id);
      this.snackBar.open(`${product.name} added to wishlist!`, "", {
        duration: 2000,
      });
    } catch (error) {
      this.handleError("Failed to add to wishlist", error);
    }
  }

  navigateToProduct(product: Product) {
    this.router.navigate(["/products", product.slug]);
  }

  onPageChange(event: PageEvent) {
    this.selectedFilters.update((filters) => ({
      ...filters,
      page: event.pageIndex,
      pageSize: event.pageSize,
    }));

    this.loadProducts();
  }

  clearFilters() {
    this.selectedFilters.set({});
    this.loadProducts();
  }

  async loadMoreProducts() {
    if (this.isLoading()) return;

    const currentPage = this.selectedFilters().page || 0;
    this.selectedFilters.update((filters) => ({
      ...filters,
      page: currentPage + 1,
    }));

    await this.loadProducts(true); // Append mode
  }

  private async loadProducts(append = false) {
    this.isLoading.set(true);

    try {
      const products = await this.productService.getProducts(
        this.selectedFilters()
      );

      if (append) {
        this.products.update((current) => [...current, ...products]);
      } else {
        this.products.set(products);
      }
    } catch (error) {
      this.handleError("Failed to load products", error);
    } finally {
      this.isLoading.set(false);
    }
  }

  private handleError(message: string, error: any) {
    console.error(message, error);
    this.snackBar.open(message, "Close", {
      duration: 5000,
      panelClass: ["error-snackbar"],
    });
  }
}

// Configuration token for dependency injection
export const PAGINATION_CONFIG = new InjectionToken<PaginationConfig>(
  "PAGINATION_CONFIG"
);

interface PaginationConfig {
  pageSize: number;
  pageSizeOptions: number[];
}
```

### **Standalone Services and Providers**

Enhanced service architecture with improved tree-shaking:

```typescript
// Enhanced standalone service
@Injectable({
  providedIn: "root",
})
export class EnhancedProductService {
  private baseUrl = "/api/v1/products";

  constructor(
    private http: HttpClient,
    @Inject(APP_CONFIG) private config: AppConfig
  ) {}

  // Enhanced with better type safety and error handling
  getProducts(filters: ProductFilters = {}): Observable<Product[]> {
    const params = this.buildHttpParams(filters);

    return this.http
      .get<ApiResponse<Product[]>>(`${this.baseUrl}`, { params })
      .pipe(
        map((response) => response.data),
        catchError(this.handleError("getProducts"))
      );
  }

  getProduct(id: number): Observable<Product> {
    return this.http.get<ApiResponse<Product>>(`${this.baseUrl}/${id}`).pipe(
      map((response) => response.data),
      catchError(this.handleError("getProduct"))
    );
  }

  private buildHttpParams(filters: ProductFilters): HttpParams {
    let params = new HttpParams();

    Object.entries(filters).forEach(([key, value]) => {
      if (value !== null && value !== undefined) {
        params = params.set(key, value.toString());
      }
    });

    return params;
  }

  private handleError<T>(operation = "operation") {
    return (error: any): Observable<T> => {
      console.error(`${operation} failed:`, error);
      throw error;
    };
  }
}

// Enhanced provider functions
export function provideAppConfig(): EnvironmentProviders {
  return makeEnvironmentProviders([
    {
      provide: APP_CONFIG,
      useValue: {
        apiUrl: environment.apiUrl,
        version: environment.version,
        features: environment.features,
      },
    },
  ]);
}

export function provideAnalytics(): EnvironmentProviders {
  return makeEnvironmentProviders([
    {
      provide: ANALYTICS_CONFIG,
      useValue: {
        trackingId: environment.analyticsTrackingId,
        debug: !environment.production,
      },
    },
    AnalyticsService,
  ]);
}

export function provideErrorHandling(): EnvironmentProviders {
  return makeEnvironmentProviders([
    {
      provide: ErrorHandler,
      useClass: GlobalErrorHandler,
    },
  ]);
}

// Configuration tokens
export const APP_CONFIG = new InjectionToken<AppConfig>("APP_CONFIG");
export const ANALYTICS_CONFIG = new InjectionToken<AnalyticsConfig>(
  "ANALYTICS_CONFIG"
);
```

These Angular 19 DX enhancements significantly improve the developer experience by providing:

- **Better debugging** with signal inspection and effect tracking
- **Enhanced tooling** with improved DevTools and performance profiling
- **Simplified architecture** with standalone components and improved bootstrapping
- **Better accessibility** with Material Design 3 and enhanced components
- **Improved performance** with optimized change detection and tree-shaking

Each enhancement works together to create a more productive and enjoyable development experience while maintaining high performance and accessibility standards.
