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
import {
  Component,
  afterRenderEffect,
  signal,
  ViewChild,
  ElementRef,
} from "@angular/core";

@Component({
  selector: "app-chart-component", // 🏷️ Component selector for template usage
  template: `
    <div #chartContainer class="chart-container">
      <!-- 📊 Chart container with template reference -->
      <canvas
        #chartCanvas
        width="800"
        height="400"
        [attr.aria-label]="
          'Chart displaying ' + chartData().length + ' data points'
        "
      >
      </canvas>

      <!-- 🎮 Chart controls -->
      <div class="chart-controls">
        <button
          (click)="addDataPoint()"
          [disabled]="isUpdating()"
          class="btn-add"
        >
          {{ isUpdating() ? "Updating..." : "Add Data Point" }}
        </button>
        <button
          (click)="removeDataPoint()"
          [disabled]="chartData().length <= 1"
          class="btn-remove"
        >
          Remove Last Point
        </button>
        <button (click)="resetChart()" class="btn-reset">Reset Chart</button>
      </div>

      <!-- 📊 Data display -->
      <div class="data-summary">
        <p>Data Points: {{ chartData().length }}</p>
        <p>Max Value: {{ maxValue() }}</p>
        <p>Average: {{ averageValue() | number : "1.2-2" }}</p>
      </div>
    </div>
  `,
  styles: [
    `
      .chart-container {
        padding: 20px; /* 🎨 Container spacing */
        border: 1px solid #ddd; /* 🖼️ Visual boundary */
        border-radius: 8px; /* 🎨 Rounded corners */
      }

      canvas {
        border: 1px solid #ccc; /* 📊 Canvas outline */
        display: block; /* 🎨 Block display for proper sizing */
        margin-bottom: 16px; /* 📏 Space below chart */
      }

      .chart-controls {
        display: flex; /* 🔄 Horizontal layout */
        gap: 12px; /* 📏 Space between buttons */
        margin-bottom: 16px; /* 📏 Space below controls */
      }

      .data-summary {
        background: #f5f5f5; /* 🎨 Light background */
        padding: 12px; /* 📏 Internal spacing */
        border-radius: 4px; /* 🎨 Slight rounding */
      }
    `,
  ],
})
export class ChartComponent {
  // 📊 SIGNAL-BASED STATE MANAGEMENT - Reactive data storage
  private chartData = signal([1, 2, 3, 4, 5]); // 📈 Initial chart data points
  private isUpdating = signal(false); // ⏳ Update state tracking
  private chartInstance: any = null; // 📊 Chart.js instance reference

  // 🔗 TEMPLATE REFERENCES - Direct DOM element access
  @ViewChild("chartContainer") chartContainer!: ElementRef<HTMLDivElement>; // 📦 Container element
  @ViewChild("chartCanvas") chartCanvas!: ElementRef<HTMLCanvasElement>; // 🎨 Canvas element

  // 📊 COMPUTED SIGNALS - Derived values from chartData
  readonly maxValue = computed(() => {
    const data = this.chartData(); // 📈 Get current data
    return data.length > 0 ? Math.max(...data) : 0; // 🔢 Find maximum value
  });

  readonly averageValue = computed(() => {
    const data = this.chartData(); // 📈 Get current data
    return data.length > 0
      ? data.reduce((sum, val) => sum + val, 0) / data.length
      : 0; // 📊 Calculate average
  });

  constructor() {
    // 🔄 AFTER RENDER EFFECT - Executes after every render cycle
    afterRenderEffect(() => {
      console.log("🔄 afterRenderEffect triggered - DOM is fully updated"); // 📝 Log effect execution

      // ✅ SAFE DOM ACCESS - Guaranteed DOM availability
      this.updateChart(); // 📊 Update chart with latest data

      // 🎯 PERFORMANCE LOGGING - Track render performance
      console.log(
        "📊 Chart updated with data points:",
        this.chartData().length
      );
    });

    // 🎯 EFFECT FOR DATA CHANGES - React to signal updates
    effect(() => {
      const currentData = this.chartData(); // 📈 Access reactive data
      console.log("📈 Chart data changed:", currentData); // 📝 Log data changes

      // 🔄 TRIGGER ACCESSIBILITY UPDATES
      this.updateAccessibilityInfo(currentData);
    });

    // ⚡ EFFECT FOR UPDATE STATE - Track updating status
    effect(() => {
      const updating = this.isUpdating(); // ⏳ Get update status
      if (updating) {
        console.log("⏳ Chart update started..."); // 📝 Log update start
      } else {
        console.log("✅ Chart update completed"); // 📝 Log update completion
      }
    });
  }

  // 📊 CHART UPDATE METHOD - Core chart rendering logic
  private updateChart(): void {
    // 🚫 GUARD CLAUSE - Ensure canvas availability
    if (!this.chartCanvas?.nativeElement) {
      console.warn("⚠️ Canvas element not available yet"); // ⚠️ Log warning
      return; // 🚪 Exit early if canvas not ready
    }

    const canvas = this.chartCanvas.nativeElement; // 🎨 Get canvas element
    const ctx = canvas.getContext("2d"); // 🖼️ Get 2D rendering context

    if (!ctx) {
      console.error("❌ Unable to get canvas 2D context"); // ❌ Log context error
      return; // 🚪 Exit if context unavailable
    }

    // 🧹 CLEAR EXISTING CHART - Reset canvas for new drawing
    ctx.clearRect(0, 0, canvas.width, canvas.height); // 🗑️ Clear entire canvas

    const data = this.chartData(); // 📈 Get current chart data
    const maxValue = this.maxValue(); // 🔢 Get maximum value for scaling

    // 📏 CHART DIMENSIONS - Calculate drawing area
    const padding = 40; // 📏 Chart padding from edges
    const chartWidth = canvas.width - 2 * padding; // 📐 Available chart width
    const chartHeight = canvas.height - 2 * padding; // 📐 Available chart height
    const barWidth = chartWidth / data.length; // 📊 Width of each bar

    // 🎨 DRAWING CONFIGURATION - Set visual properties
    ctx.fillStyle = "#4CAF50"; // 🎨 Bar fill color (green)
    ctx.strokeStyle = "#2E7D32"; // 🖊️ Bar border color (dark green)
    ctx.lineWidth = 2; // 📏 Border thickness

    // 📊 DRAW CHART BARS - Render each data point
    data.forEach((value, index) => {
      const barHeight = maxValue > 0 ? (value / maxValue) * chartHeight : 0; // 📏 Scale bar height
      const x = padding + index * barWidth; // 📐 X position of bar
      const y = canvas.height - padding - barHeight; // 📐 Y position of bar (bottom-up)

      // 🎨 DRAW BAR - Fill and stroke rectangle
      ctx.fillRect(x + 2, y, barWidth - 4, barHeight); // 📊 Fill bar with slight padding
      ctx.strokeRect(x + 2, y, barWidth - 4, barHeight); // 🖊️ Add border to bar

      // 🔤 DRAW VALUE LABELS - Show data values
      ctx.fillStyle = "#333"; // 🎨 Text color (dark gray)
      ctx.font = "14px Arial"; // 🔤 Font configuration
      ctx.textAlign = "center"; // 📐 Center-align text
      ctx.fillText(
        value.toString(), // 🔢 Convert number to string
        x + barWidth / 2, // 📐 Center horizontally in bar
        y - 5 // 📐 Position above bar
      );

      ctx.fillStyle = "#4CAF50"; // 🎨 Reset fill color for next bar
    });

    // 📐 DRAW AXES - Add coordinate system
    ctx.strokeStyle = "#666"; // 🎨 Axis color (gray)
    ctx.lineWidth = 1; // 📏 Axis thickness

    // 📐 X-axis
    ctx.beginPath(); // 🎨 Start new path
    ctx.moveTo(padding, canvas.height - padding); // 📍 Move to start point
    ctx.lineTo(canvas.width - padding, canvas.height - padding); // 📏 Draw horizontal line
    ctx.stroke(); // 🖊️ Apply stroke

    // 📐 Y-axis
    ctx.beginPath(); // 🎨 Start new path
    ctx.moveTo(padding, padding); // 📍 Move to start point
    ctx.lineTo(padding, canvas.height - padding); // 📏 Draw vertical line
    ctx.stroke(); // 🖊️ Apply stroke

    console.log("🎨 Chart rendering completed successfully"); // ✅ Log completion
  }

  // 🎯 USER INTERACTION METHODS - Handle user actions

  addDataPoint(): void {
    this.isUpdating.set(true); // ⏳ Set updating state

    // 🎲 GENERATE RANDOM DATA - Create new data point
    const newValue = Math.floor(Math.random() * 10) + 1; // 🔢 Random number 1-10

    // 📈 UPDATE SIGNAL - Add new data point
    this.chartData.update((currentData) => {
      const newData = [...currentData, newValue]; // 📊 Create new array with added value
      console.log("➕ Added new data point:", newValue); // 📝 Log addition
      return newData;
    });

    // ⏱️ SIMULATE ASYNC OPERATION - Mimic real data processing
    setTimeout(() => {
      this.isUpdating.set(false); // ✅ Clear updating state
      console.log("✅ Data point addition completed"); // 📝 Log completion
    }, 500);
  }

  removeDataPoint(): void {
    // 🚫 GUARD CLAUSE - Prevent removing from empty array
    if (this.chartData().length <= 1) {
      console.warn("⚠️ Cannot remove data point - minimum one point required"); // ⚠️ Log warning
      return; // 🚪 Exit early
    }

    this.isUpdating.set(true); // ⏳ Set updating state

    // 📉 UPDATE SIGNAL - Remove last data point
    this.chartData.update((currentData) => {
      const newData = currentData.slice(0, -1); // ✂️ Remove last element
      console.log("➖ Removed last data point, remaining:", newData.length); // 📝 Log removal
      return newData;
    });

    // ⏱️ SIMULATE ASYNC OPERATION
    setTimeout(() => {
      this.isUpdating.set(false); // ✅ Clear updating state
      console.log("✅ Data point removal completed"); // 📝 Log completion
    }, 300);
  }

  resetChart(): void {
    this.isUpdating.set(true); // ⏳ Set updating state

    // 🔄 RESET TO INITIAL STATE
    this.chartData.set([1, 2, 3, 4, 5]); // 📊 Reset to initial data
    console.log("🔄 Chart reset to initial state"); // 📝 Log reset

    // ⏱️ SIMULATE ASYNC OPERATION
    setTimeout(() => {
      this.isUpdating.set(false); // ✅ Clear updating state
      console.log("✅ Chart reset completed"); // 📝 Log completion
    }, 400);
  }

  // ♿ ACCESSIBILITY HELPER - Update screen reader information
  private updateAccessibilityInfo(data: number[]): void {
    const canvas = this.chartCanvas?.nativeElement; // 🎨 Get canvas reference
    if (canvas) {
      const description = `Chart displaying ${
        data.length
      } data points ranging from ${Math.min(...data)} to ${Math.max(...data)}`; // 📝 Create description
      canvas.setAttribute("aria-label", description); // ♿ Set accessibility label
      console.log("♿ Updated accessibility information:", description); // 📝 Log accessibility update
    }
  }

  // 🔧 LIFECYCLE CLEANUP - Cleanup chart instance if needed
  ngOnDestroy(): void {
    if (this.chartInstance) {
      this.chartInstance.destroy(); // 🧹 Cleanup chart instance
      console.log("🧹 Chart instance destroyed"); // 📝 Log cleanup
    }
  }

  // 🎯 UTILITY METHODS - Helper functions

  updateData(newData: number[]): void {
    // ✅ INPUT VALIDATION - Ensure valid data
    if (!Array.isArray(newData) || newData.length === 0) {
      console.error("❌ Invalid data provided to updateData"); // ❌ Log validation error
      return; // 🚪 Exit early
    }

    console.log("🔄 Updating chart data externally:", newData); // 📝 Log external update
    this.chartData.set(newData); // 📊 Update signal with new data

    // 🔄 afterRenderEffect will automatically trigger after this update
    console.log(
      "✅ External data update completed - afterRenderEffect will handle rendering"
    ); // 📝 Log completion
  }

  // 📊 PUBLIC GETTERS - Expose reactive state
  getCurrentData(): readonly number[] {
    return this.chartData(); // 📈 Return current data (readonly)
  }

  isChartUpdating(): boolean {
    return this.isUpdating(); // ⏳ Return current updating state
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
  signal,
  computed,
  effect,
  OnInit,
  OnDestroy,
} from "@angular/core";

@Component({
  selector: "app-focus-input", // 🏷️ Component selector for template usage
  template: `
    <div class="focus-demo-container">
      <h3>🎯 Focus Management Demo</h3>

      <!-- 🎮 Primary input with auto-focus -->
      <div class="input-group">
        <label for="mainInput" class="input-label">
          Main Input (Auto-focused on load)
        </label>
        <input
          #inputElement
          id="mainInput"
          type="text"
          placeholder="Auto-focus input"
          class="main-input"
          [class.has-content]="hasContent()"
          (input)="onInputChange($event)"
          (blur)="onInputBlur()"
          (focus)="onInputFocus()"
        />
        <small
          class="character-count"
          [class.warning]="isNearLimit()"
          [class.error]="isOverLimit()"
        >
          {{ inputValue().length }}/{{ maxLength }} characters
        </small>
      </div>

      <!-- 🎮 Control buttons with focus management -->
      <div class="button-group">
        <button
          (click)="resetAndFocus()"
          class="btn-primary"
          [disabled]="isProcessing()"
        >
          {{ isProcessing() ? "Processing..." : "Reset & Focus" }}
        </button>

        <button (click)="clearAndFocusNext()" class="btn-secondary">
          Clear & Focus Next
        </button>

        <button (click)="selectAllText()" [disabled]="!hasContent()">
          Select All Text
        </button>

        <button (click)="focusWithDelay()" class="btn-info">
          Focus with Delay
        </button>
      </div>

      <!-- 🎯 Additional inputs for focus testing -->
      <div class="input-group">
        <label for="secondInput" class="input-label"> Second Input </label>
        <input
          #secondInput
          id="secondInput"
          type="text"
          placeholder="Second input for focus testing"
          class="secondary-input"
          (focus)="onSecondInputFocus()"
          (blur)="onSecondInputBlur()"
        />
      </div>

      <div class="input-group">
        <label for="emailInput" class="input-label"> Email Input </label>
        <input
          #emailInput
          id="emailInput"
          type="email"
          placeholder="email@example.com"
          class="email-input"
          [class.invalid]="emailValue() && !isValidEmail()"
          (input)="onEmailChange($event)"
          (blur)="validateEmail()"
        />
        <small
          class="validation-message"
          *ngIf="emailValue() && !isValidEmail()"
        >
          Please enter a valid email address
        </small>
      </div>

      <!-- 📊 Focus state indicators -->
      <div class="state-indicators">
        <div class="indicator" [class.active]="currentFocus() === 'main'">
          🎯 Main Input
          {{ currentFocus() === "main" ? "Focused" : "Unfocused" }}
        </div>
        <div class="indicator" [class.active]="currentFocus() === 'second'">
          🎯 Second Input
          {{ currentFocus() === "second" ? "Focused" : "Unfocused" }}
        </div>
        <div class="indicator" [class.active]="currentFocus() === 'email'">
          📧 Email Input
          {{ currentFocus() === "email" ? "Focused" : "Unfocused" }}
        </div>
      </div>

      <!-- 📈 Statistics -->
      <div class="statistics">
        <p>Focus Events: {{ focusEventCount() }}</p>
        <p>Render Cycles: {{ renderCycleCount() }}</p>
        <p>Last Focus Time: {{ lastFocusTime() || "Never" }}</p>
      </div>
    </div>
  `,
  styles: [
    `
      .focus-demo-container {
        max-width: 600px; /* 📏 Container width limit */
        margin: 20px auto; /* 📐 Center container */
        padding: 24px; /* 📏 Internal spacing */
        border: 1px solid #ddd; /* 🖼️ Container border */
        border-radius: 8px; /* 🎨 Rounded corners */
        background: #fafafa; /* 🎨 Light background */
      }

      .input-group {
        margin-bottom: 20px; /* 📏 Space between groups */
      }

      .input-label {
        display: block; /* 🔄 Block display for proper spacing */
        margin-bottom: 6px; /* 📏 Space below label */
        font-weight: 500; /* 🔤 Semi-bold text */
        color: #333; /* 🎨 Dark text color */
      }

      input {
        width: 100%; /* 📐 Full width */
        padding: 12px; /* 📏 Internal spacing */
        border: 2px solid #ddd; /* 🖼️ Input border */
        border-radius: 4px; /* 🎨 Slight rounding */
        font-size: 16px; /* 🔤 Font size */
        transition: all 0.2s ease; /* 🎨 Smooth transitions */
      }

      input:focus {
        border-color: #007bff; /* 🎨 Blue focus border */
        box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1); /* ✨ Focus shadow */
        outline: none; /* 🚫 Remove default outline */
      }

      input.has-content {
        border-color: #28a745; /* 🎨 Green border with content */
      }

      input.invalid {
        border-color: #dc3545; /* 🎨 Red border for invalid */
      }

      .character-count {
        display: block; /* 🔄 Block display */
        margin-top: 4px; /* 📏 Space above */
        font-size: 12px; /* 🔤 Small font */
        color: #666; /* 🎨 Gray text */
      }

      .character-count.warning {
        color: #ffc107; /* 🎨 Warning yellow */
      }

      .character-count.error {
        color: #dc3545; /* 🎨 Error red */
      }

      .button-group {
        display: flex; /* 🔄 Flex layout */
        gap: 8px; /* 📏 Space between buttons */
        margin-bottom: 20px; /* 📏 Space below group */
        flex-wrap: wrap; /* 🔄 Wrap on small screens */
      }

      button {
        padding: 8px 16px; /* 📏 Button spacing */
        border: none; /* 🚫 Remove default border */
        border-radius: 4px; /* 🎨 Rounded corners */
        font-size: 14px; /* 🔤 Font size */
        cursor: pointer; /* 👆 Pointer cursor */
        transition: all 0.2s ease; /* 🎨 Smooth transitions */
      }

      .btn-primary {
        background: #007bff; /* 🎨 Blue background */
        color: white; /* 🎨 White text */
      }

      .btn-primary:hover:not(:disabled) {
        background: #0056b3; /* 🎨 Darker blue on hover */
      }

      .btn-secondary {
        background: #6c757d; /* 🎨 Gray background */
        color: white; /* 🎨 White text */
      }

      .btn-info {
        background: #17a2b8; /* 🎨 Teal background */
        color: white; /* 🎨 White text */
      }

      button:disabled {
        opacity: 0.5; /* 🎨 Reduced opacity */
        cursor: not-allowed; /* 🚫 Not allowed cursor */
      }

      .state-indicators {
        display: flex; /* 🔄 Flex layout */
        flex-direction: column; /* 🔄 Vertical layout */
        gap: 8px; /* 📏 Space between indicators */
        margin-bottom: 16px; /* 📏 Space below */
      }

      .indicator {
        padding: 8px; /* 📏 Internal spacing */
        border-radius: 4px; /* 🎨 Rounded corners */
        background: #f8f9fa; /* 🎨 Light background */
        border: 1px solid #dee2e6; /* 🖼️ Light border */
      }

      .indicator.active {
        background: #d4edda; /* 🎨 Green background for active */
        border-color: #c3e6cb; /* 🖼️ Green border */
        color: #155724; /* 🎨 Dark green text */
      }

      .statistics {
        background: #e9ecef; /* 🎨 Light gray background */
        padding: 16px; /* 📏 Internal spacing */
        border-radius: 4px; /* 🎨 Rounded corners */
      }

      .statistics p {
        margin: 4px 0; /* 📏 Small margin */
        font-size: 14px; /* 🔤 Small font */
      }
    `,
  ],
})
export class FocusInputComponent implements OnInit, OnDestroy {
  // 🔗 TEMPLATE REFERENCES - Direct DOM element access
  @ViewChild("inputElement") inputElement!: ElementRef<HTMLInputElement>; // 🎯 Main input element
  @ViewChild("secondInput") secondInput!: ElementRef<HTMLInputElement>; // 🎯 Second input element
  @ViewChild("emailInput") emailInput!: ElementRef<HTMLInputElement>; // 📧 Email input element

  // 📊 REACTIVE STATE MANAGEMENT - Signal-based state
  private inputValue = signal(""); // 📝 Main input value
  private emailValue = signal(""); // 📧 Email input value
  private currentFocus = signal<"main" | "second" | "email" | null>(null); // 🎯 Currently focused input
  private focusEventCount = signal(0); // 📊 Count of focus events
  private renderCycleCount = signal(0); // 📊 Count of render cycles
  private lastFocusTime = signal<string>(""); // ⏰ Last focus timestamp
  private isProcessing = signal(false); // ⏳ Processing state

  // 🔢 CONFIGURATION CONSTANTS
  private readonly maxLength = 100; // 📏 Maximum input length

  // 📊 COMPUTED SIGNALS - Derived reactive values
  readonly hasContent = computed(() => this.inputValue().trim().length > 0); // ✅ Has input content
  readonly isNearLimit = computed(
    () => this.inputValue().length > this.maxLength * 0.8
  ); // ⚠️ Near character limit
  readonly isOverLimit = computed(
    () => this.inputValue().length > this.maxLength
  ); // ❌ Over character limit

  constructor() {
    console.log(
      "🏗️ FocusInputComponent constructor - setting up lifecycle hooks"
    ); // 📝 Log constructor

    // 🔄 AFTER NEXT RENDER HOOK - Executes after the NEXT render cycle only
    afterNextRender(() => {
      console.log(
        "🎯 afterNextRender triggered - DOM is ready for initial focus"
      ); // 📝 Log hook execution

      // ✅ SAFE DOM ACCESS - Guaranteed element availability
      if (this.inputElement?.nativeElement) {
        this.inputElement.nativeElement.focus(); // 🎯 Set initial focus
        console.log("✨ Initial focus set on main input"); // 📝 Log focus action

        // 📊 UPDATE STATE
        this.currentFocus.set("main"); // 🎯 Set focus state
        this.updateFocusStats(); // 📊 Update statistics
      } else {
        console.warn("⚠️ Input element not available during afterNextRender"); // ⚠️ Log warning
      }
    });

    // 🎯 EFFECT FOR FOCUS TRACKING - Monitor focus changes
    effect(() => {
      const focus = this.currentFocus(); // 🎯 Get current focus state
      console.log("🔍 Focus changed to:", focus || "none"); // 📝 Log focus changes

      // 📊 INCREMENT RENDER COUNTER
      this.renderCycleCount.update((count) => count + 1);
    });

    // 📝 EFFECT FOR INPUT VALUE TRACKING
    effect(() => {
      const value = this.inputValue(); // 📝 Get input value
      console.log("📝 Input value changed:", value.length, "characters"); // 📝 Log value changes

      // ⚠️ VALIDATION LOGGING
      if (this.isOverLimit()) {
        console.warn("⚠️ Input value exceeds maximum length"); // ⚠️ Log limit warning
      }
    });

    // 📧 EFFECT FOR EMAIL VALIDATION
    effect(() => {
      const email = this.emailValue(); // 📧 Get email value
      if (email) {
        const isValid = this.isValidEmail(); // ✅ Check validity
        console.log(
          "📧 Email validation:",
          email,
          "-",
          isValid ? "Valid" : "Invalid"
        ); // 📝 Log validation
      }
    });
  }

  ngOnInit(): void {
    console.log("🚀 Component initialized - ready for user interaction"); // 📝 Log initialization
  }

  ngOnDestroy(): void {
    console.log("🧹 Component destroyed - cleanup completed"); // 📝 Log destruction
  }

  // 🎯 USER INTERACTION METHODS - Handle user actions

  resetAndFocus(): void {
    console.log("🔄 Reset and focus requested"); // 📝 Log action start
    this.isProcessing.set(true); // ⏳ Set processing state

    // 🗑️ CLEAR INPUT VALUE
    if (this.inputElement?.nativeElement) {
      this.inputElement.nativeElement.value = ""; // 🗑️ Clear DOM value
      this.inputValue.set(""); // 🗑️ Clear signal value
      console.log("🗑️ Input cleared"); // 📝 Log clear action
    }

    // ⏱️ SIMULATE PROCESSING TIME
    setTimeout(() => {
      // 🔄 AFTER NEXT RENDER - Schedule focus for after DOM update
      afterNextRender(() => {
        if (this.inputElement?.nativeElement) {
          this.inputElement.nativeElement.focus(); // 🎯 Set focus after clear
          this.currentFocus.set("main"); // 🎯 Update focus state
          this.updateFocusStats(); // 📊 Update statistics
          console.log("✨ Focus restored after reset"); // 📝 Log focus restoration
        }
      });

      this.isProcessing.set(false); // ✅ Clear processing state
      console.log("✅ Reset and focus completed"); // 📝 Log completion
    }, 800);
  }

  clearAndFocusNext(): void {
    console.log("🔄 Clear and focus next requested"); // 📝 Log action

    // 🗑️ CLEAR CURRENT INPUT
    if (this.inputElement?.nativeElement) {
      this.inputElement.nativeElement.value = ""; // 🗑️ Clear value
      this.inputValue.set(""); // 📊 Update signal
    }

    // 🔄 AFTER NEXT RENDER - Focus second input after DOM update
    afterNextRender(() => {
      if (this.secondInput?.nativeElement) {
        this.secondInput.nativeElement.focus(); // 🎯 Focus second input
        this.currentFocus.set("second"); // 🎯 Update focus state
        this.updateFocusStats(); // 📊 Update statistics
        console.log("🎯 Focused moved to second input"); // 📝 Log focus change
      }
    });
  }

  selectAllText(): void {
    // ✅ GUARD CLAUSE - Ensure element and content exist
    if (!this.inputElement?.nativeElement || !this.hasContent()) {
      console.warn("⚠️ Cannot select text - no element or content"); // ⚠️ Log warning
      return; // 🚪 Exit early
    }

    const input = this.inputElement.nativeElement; // 🎯 Get input element

    // 🔄 AFTER NEXT RENDER - Ensure DOM is stable before selection
    afterNextRender(() => {
      input.select(); // 🎯 Select all text
      input.setSelectionRange(0, input.value.length); // 📏 Set selection range
      console.log("✨ All text selected"); // 📝 Log selection

      // 📊 UPDATE FOCUS STATE
      this.currentFocus.set("main");
      this.updateFocusStats();
    });
  }

  focusWithDelay(): void {
    console.log("⏱️ Focus with delay requested"); // 📝 Log delayed focus start

    // ⏱️ DELAYED FOCUS - Focus after specific delay
    setTimeout(() => {
      // 🔄 AFTER NEXT RENDER - Ensure clean render cycle
      afterNextRender(() => {
        if (this.emailInput?.nativeElement) {
          this.emailInput.nativeElement.focus(); // 🎯 Focus email input
          this.currentFocus.set("email"); // 🎯 Update state
          this.updateFocusStats(); // 📊 Update statistics
          console.log("🎯 Delayed focus applied to email input"); // 📝 Log delayed focus
        }
      });
    }, 1500);

    console.log("⏰ Delayed focus scheduled for 1.5 seconds"); // 📝 Log schedule
  }

  // 📝 INPUT EVENT HANDLERS - Handle user input

  onInputChange(event: Event): void {
    const target = event.target as HTMLInputElement; // 🎯 Get input element
    const value = target.value; // 📝 Extract value

    // ✂️ ENFORCE LENGTH LIMIT
    if (value.length > this.maxLength) {
      const truncated = value.substring(0, this.maxLength); // ✂️ Truncate excess
      target.value = truncated; // 🔄 Update DOM
      this.inputValue.set(truncated); // 📊 Update signal
      console.warn(`✂️ Input truncated to ${this.maxLength} characters`); // ⚠️ Log truncation
    } else {
      this.inputValue.set(value); // 📊 Update signal with full value
    }

    console.log("📝 Input changed:", this.inputValue().length, "characters"); // 📝 Log change
  }

  onEmailChange(event: Event): void {
    const target = event.target as HTMLInputElement; // 📧 Get email input
    this.emailValue.set(target.value); // 📊 Update email signal
    console.log("📧 Email input changed:", target.value); // 📝 Log email change
  }

  // 🎯 FOCUS EVENT HANDLERS - Track focus state

  onInputFocus(): void {
    console.log("🎯 Main input focused"); // 📝 Log focus
    this.currentFocus.set("main"); // 🎯 Update focus state
    this.updateFocusStats(); // 📊 Update statistics
  }

  onInputBlur(): void {
    console.log("🌫️ Main input blurred"); // 📝 Log blur
    // Note: We don't set currentFocus to null immediately as another element might gain focus
  }

  onSecondInputFocus(): void {
    console.log("🎯 Second input focused"); // 📝 Log focus
    this.currentFocus.set("second"); // 🎯 Update focus state
    this.updateFocusStats(); // 📊 Update statistics
  }

  onSecondInputBlur(): void {
    console.log("🌫️ Second input blurred"); // 📝 Log blur
  }

  // 📧 EMAIL VALIDATION METHODS

  validateEmail(): void {
    const email = this.emailValue(); // 📧 Get email value
    if (email && !this.isValidEmail()) {
      console.warn("⚠️ Invalid email format entered"); // ⚠️ Log validation warning
    }
  }

  isValidEmail(): boolean {
    const email = this.emailValue(); // 📧 Get email value
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/; // 📝 Email validation regex
    return emailRegex.test(email); // ✅ Test against pattern
  }

  // 📊 UTILITY METHODS - Helper functions

  private updateFocusStats(): void {
    this.focusEventCount.update((count) => count + 1); // 📊 Increment focus count
    this.lastFocusTime.set(new Date().toLocaleTimeString()); // ⏰ Update timestamp
    console.log("📊 Focus statistics updated:", this.focusEventCount()); // 📝 Log stats update
  }

  // 📊 PUBLIC GETTERS - Expose reactive state (for template)

  inputValue = this.inputValue.asReadonly(); // 📝 Readonly input value
  emailValue = this.emailValue.asReadonly(); // 📧 Readonly email value
  currentFocus = this.currentFocus.asReadonly(); // 🎯 Readonly focus state
  focusEventCount = this.focusEventCount.asReadonly(); // 📊 Readonly event count
  renderCycleCount = this.renderCycleCount.asReadonly(); // 📊 Readonly render count
  lastFocusTime = this.lastFocusTime.asReadonly(); // ⏰ Readonly last focus time
  isProcessing = this.isProcessing.asReadonly(); // ⏳ Readonly processing state
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
  afterNextRender,
  ViewChild,
  ElementRef,
  OnInit,
  OnDestroy,
} from "@angular/core";

@Component({
  selector: "app-reactive-component", // 🏷️ Component selector for template usage
  template: `
    <div class="metrics-dashboard">
      <h2>🔄 Reactive Lifecycle Demo</h2>

      <!-- 📊 Main metrics display -->
      <div class="metrics-grid">
        <div class="metric-card" [class.highlight]="count() > 0">
          <h3>📊 Count</h3>
          <span class="metric-value">{{ count() }}</span>
        </div>

        <div class="metric-card" [class.highlight]="doubled() > 10">
          <h3>✖️ Doubled</h3>
          <span class="metric-value">{{ doubled() }}</span>
        </div>

        <div class="metric-card" [class]="'status-' + status()">
          <h3>🎯 Status</h3>
          <span class="status-label">{{ status().toUpperCase() }}</span>
        </div>

        <div class="metric-card">
          <h3>🔄 Effects</h3>
          <span class="metric-value">{{ effectExecutions() }}</span>
        </div>
      </div>

      <!-- 🎮 Interactive controls -->
      <div class="control-panel">
        <button
          (click)="increment()"
          class="btn-primary"
          [disabled]="isProcessing()"
        >
          {{ isProcessing() ? "Processing..." : "➕ Increment" }}
        </button>

        <button
          (click)="decrement()"
          class="btn-secondary"
          [disabled]="count() <= 0"
        >
          ➖ Decrement
        </button>

        <button (click)="reset()" class="btn-warning">🔄 Reset</button>

        <button (click)="batchUpdate()" class="btn-info">
          🚀 Batch Update
        </button>

        <button (click)="simulateAsync()" class="btn-success">
          ⏱️ Async Update
        </button>
      </div>

      <!-- 📈 Progress visualization -->
      <div class="progress-section">
        <h3>Progress Visualization</h3>
        <div class="progress-container">
          <div
            class="progress-bar"
            #progressBar
            [style.width.%]="progressPercentage()"
          ></div>
          <span class="progress-text">{{ progressPercentage() }}%</span>
        </div>
      </div>

      <!-- 🎯 Status visualization with dynamic classes -->
      <div
        class="status-visualization"
        [ngClass]="statusClass()"
        [attr.aria-label]="'Current status: ' + status()"
      >
        <div class="status-icon">{{ getStatusEmoji() }}</div>
        <div class="status-description">{{ getStatusDescription() }}</div>
      </div>

      <!-- 📋 Activity log -->
      <div class="activity-log">
        <h3>📋 Activity Log</h3>
        <div class="log-container" #logContainer>
          <div
            *ngFor="let entry of activityLog(); let i = index"
            class="log-entry"
            [class.latest]="i === activityLog().length - 1"
          >
            <span class="log-time">{{ entry.timestamp }}</span>
            <span class="log-action">{{ entry.action }}</span>
            <span class="log-value">{{ entry.value }}</span>
          </div>
        </div>
      </div>

      <!-- 📊 Render cycle information -->
      <div class="debug-info">
        <h4>🔍 Debug Information</h4>
        <p>Render Cycles: {{ renderCycles() }}</p>
        <p>DOM Updates: {{ domUpdates() }}</p>
        <p>Last Updated: {{ lastUpdateTime() }}</p>
        <p>Performance Score: {{ performanceScore() }}</p>
      </div>
    </div>
  `,
  styles: [
    `
      .metrics-dashboard {
        max-width: 1000px; /* 📏 Container width limit */
        margin: 20px auto; /* 📐 Center container */
        padding: 24px; /* 📏 Internal spacing */
        font-family: "Segoe UI", sans-serif; /* 🔤 Modern font */
      }

      .metrics-grid {
        display: grid; /* 🎛️ Grid layout */
        grid-template-columns: repeat(
          auto-fit,
          minmax(200px, 1fr)
        ); /* 📐 Responsive columns */
        gap: 16px; /* 📏 Space between cards */
        margin-bottom: 24px; /* 📏 Space below grid */
      }

      .metric-card {
        background: #f8f9fa; /* 🎨 Light background */
        border: 2px solid #dee2e6; /* 🖼️ Border */
        border-radius: 8px; /* 🎨 Rounded corners */
        padding: 16px; /* 📏 Internal spacing */
        text-align: center; /* 📐 Center text */
        transition: all 0.3s ease; /* 🎨 Smooth transitions */
      }

      .metric-card.highlight {
        border-color: #007bff; /* 🎨 Blue border for highlight */
        background: #e3f2fd; /* 🎨 Light blue background */
        transform: scale(1.02); /* 🎨 Slight scale effect */
      }

      .metric-card.status-idle {
        background: #f1f3f4; /* 🎨 Gray for idle */
        border-color: #9aa0a6; /* 🖼️ Gray border */
      }

      .metric-card.status-active {
        background: #fff3cd; /* 🎨 Yellow for active */
        border-color: #ffc107; /* 🖼️ Yellow border */
      }

      .metric-card.status-complete {
        background: #d4edda; /* 🎨 Green for complete */
        border-color: #28a745; /* 🖼️ Green border */
      }

      .metric-value {
        font-size: 2rem; /* 🔤 Large metric display */
        font-weight: bold; /* 🔤 Bold text */
        color: #495057; /* 🎨 Dark gray */
        display: block; /* 🔄 Block display */
      }

      .status-label {
        font-size: 1.2rem; /* 🔤 Medium font size */
        font-weight: 500; /* 🔤 Medium weight */
      }

      .control-panel {
        display: flex; /* 🔄 Flex layout */
        gap: 12px; /* 📏 Space between buttons */
        margin-bottom: 24px; /* 📏 Space below panel */
        flex-wrap: wrap; /* 🔄 Wrap on small screens */
      }

      button {
        padding: 10px 16px; /* 📏 Button spacing */
        border: none; /* 🚫 Remove default border */
        border-radius: 6px; /* 🎨 Rounded corners */
        font-size: 14px; /* 🔤 Font size */
        font-weight: 500; /* 🔤 Medium weight */
        cursor: pointer; /* 👆 Pointer cursor */
        transition: all 0.2s ease; /* 🎨 Smooth transitions */
      }

      .btn-primary {
        background: #007bff; /* 🎨 Blue background */
        color: white; /* 🎨 White text */
      }

      .btn-primary:hover:not(:disabled) {
        background: #0056b3; /* 🎨 Darker blue on hover */
      }

      .btn-secondary {
        background: #6c757d; /* 🎨 Gray background */
        color: white; /* 🎨 White text */
      }

      .btn-warning {
        background: #ffc107; /* 🎨 Yellow background */
        color: #212529; /* 🎨 Dark text */
      }

      .btn-info {
        background: #17a2b8; /* 🎨 Teal background */
        color: white; /* 🎨 White text */
      }

      .btn-success {
        background: #28a745; /* 🎨 Green background */
        color: white; /* 🎨 White text */
      }

      button:disabled {
        opacity: 0.5; /* 🎨 Reduced opacity */
        cursor: not-allowed; /* 🚫 Not allowed cursor */
      }

      .progress-section {
        margin-bottom: 24px; /* 📏 Space below section */
      }

      .progress-container {
        position: relative; /* 📐 Relative positioning */
        background: #e9ecef; /* 🎨 Light gray background */
        border-radius: 20px; /* 🎨 Rounded progress bar */
        height: 30px; /* 📏 Progress bar height */
        overflow: hidden; /* 🚫 Hide overflow */
      }

      .progress-bar {
        background: linear-gradient(
          90deg,
          #007bff,
          #28a745
        ); /* 🌈 Gradient background */
        height: 100%; /* 📏 Full height */
        border-radius: 20px; /* 🎨 Rounded corners */
        transition: width 0.3s ease; /* 🎨 Smooth width transition */
        min-width: 0; /* 📏 Minimum width */
      }

      .progress-text {
        position: absolute; /* 📐 Absolute positioning */
        top: 50%; /* 📐 Vertical center */
        left: 50%; /* 📐 Horizontal center */
        transform: translate(-50%, -50%); /* 📐 Perfect centering */
        font-weight: bold; /* 🔤 Bold text */
        color: #495057; /* 🎨 Dark gray */
      }

      .status-visualization {
        background: #f8f9fa; /* 🎨 Light background */
        border-radius: 8px; /* 🎨 Rounded corners */
        padding: 20px; /* 📏 Internal spacing */
        text-align: center; /* 📐 Center alignment */
        margin-bottom: 24px; /* 📏 Space below */
      }

      .status-icon {
        font-size: 3rem; /* 🔤 Large icon */
        margin-bottom: 10px; /* 📏 Space below icon */
      }

      .status-description {
        font-size: 1.1rem; /* 🔤 Medium font size */
        color: #495057; /* 🎨 Dark gray */
      }

      .activity-log {
        margin-bottom: 24px; /* 📏 Space below log */
      }

      .log-container {
        background: #f8f9fa; /* 🎨 Light background */
        border-radius: 6px; /* 🎨 Rounded corners */
        padding: 16px; /* 📏 Internal spacing */
        max-height: 200px; /* 📏 Maximum height */
        overflow-y: auto; /* 📜 Vertical scroll */
      }

      .log-entry {
        display: flex; /* 🔄 Flex layout */
        justify-content: space-between; /* 📐 Space between items */
        padding: 4px 0; /* 📏 Vertical padding */
        border-bottom: 1px solid #dee2e6; /* 🖼️ Bottom border */
        font-family: monospace; /* 🔤 Monospace font */
        font-size: 12px; /* 🔤 Small font */
      }

      .log-entry.latest {
        background: #e3f2fd; /* 🎨 Highlight latest entry */
        border-radius: 4px; /* 🎨 Rounded corners */
        padding: 6px 8px; /* 📏 Extra padding */
      }

      .log-time {
        color: #6c757d; /* 🎨 Gray timestamp */
      }

      .log-action {
        font-weight: bold; /* 🔤 Bold action */
      }

      .log-value {
        color: #007bff; /* 🎨 Blue value */
      }

      .debug-info {
        background: #343a40; /* 🎨 Dark background */
        color: white; /* 🎨 White text */
        padding: 16px; /* 📏 Internal spacing */
        border-radius: 6px; /* 🎨 Rounded corners */
        font-family: monospace; /* 🔤 Monospace font */
        font-size: 12px; /* 🔤 Small font */
      }

      .debug-info h4 {
        margin-top: 0; /* 📏 Remove top margin */
        margin-bottom: 12px; /* 📏 Space below heading */
      }

      .debug-info p {
        margin: 4px 0; /* 📏 Small margin */
      }
    `,
  ],
})
export class ReactiveComponent implements OnInit, OnDestroy {
  // 📊 REACTIVE STATE MANAGEMENT - Core application signals
  count = signal(0); // 📊 Primary counter value
  status = signal<"idle" | "active" | "complete">("idle"); // 🎯 Current component status
  effectExecutions = signal(0); // 📊 Count of effect executions
  renderCycles = signal(0); // 📊 Count of render cycles
  domUpdates = signal(0); // 📊 Count of DOM updates
  isProcessing = signal(false); // ⏳ Processing state
  activityLog = signal<ActivityEntry[]>([]); // 📋 Activity history
  lastUpdateTime = signal<string>("Never"); // ⏰ Last update timestamp

  // 🔗 TEMPLATE REFERENCES - Direct DOM element access
  @ViewChild("progressBar") progressBar!: ElementRef<HTMLDivElement>; // 📊 Progress bar element
  @ViewChild("logContainer") logContainer!: ElementRef<HTMLDivElement>; // 📋 Log container element

  // 📊 COMPUTED SIGNALS - Derived reactive values
  doubled = computed(() => {
    const currentCount = this.count(); // 📊 Get current count
    console.log("🔄 Computing doubled value for count:", currentCount); // 📝 Log computation
    return currentCount * 2; // ✖️ Return doubled value
  });

  statusClass = computed(() => {
    const currentStatus = this.status(); // 🎯 Get current status
    console.log("🎨 Computing status classes for:", currentStatus); // 📝 Log class computation

    return {
      "status-idle": currentStatus === "idle", // 🎨 Idle state class
      "status-active": currentStatus === "active", // 🎨 Active state class
      "status-complete": currentStatus === "complete", // 🎨 Complete state class
    };
  });

  progressPercentage = computed(() => {
    const current = this.count(); // 📊 Get current count
    const max = 20; // 🔢 Maximum value for 100%
    const percentage = Math.min((current / max) * 100, 100); // 📊 Calculate percentage with cap
    console.log("📊 Computing progress percentage:", percentage); // 📝 Log percentage calculation
    return Math.round(percentage); // 🔢 Round to integer
  });

  performanceScore = computed(() => {
    const renders = this.renderCycles(); // 📊 Get render count
    const effects = this.effectExecutions(); // 📊 Get effect count
    const updates = this.domUpdates(); // 📊 Get DOM update count

    // 📊 CALCULATE PERFORMANCE SCORE - Higher is better
    const baseScore = 100;
    const renderPenalty = renders * 0.5; // 📉 Penalty for excessive renders
    const effectPenalty = effects * 0.3; // 📉 Penalty for excessive effects
    const updatePenalty = updates * 0.2; // 📉 Penalty for excessive DOM updates

    const score = Math.max(
      0,
      baseScore - renderPenalty - effectPenalty - updatePenalty
    ); // 📊 Calculate final score
    return Math.round(score);
  });

  constructor() {
    console.log(
      "🏗️ ReactiveComponent constructor - initializing lifecycle hooks"
    ); // 📝 Log constructor

    // 🔄 EFFECT FOR COUNT CHANGES - Reactive state management
    effect(() => {
      const currentCount = this.count(); // 📊 Access reactive count
      console.log("📊 Count effect triggered - new value:", currentCount); // 📝 Log count change

      // 📊 INCREMENT EFFECT COUNTER
      this.effectExecutions.update((count) => count + 1);

      // 📋 LOG ACTIVITY
      this.logActivity("COUNT_CHANGED", currentCount);

      // 🎯 UPDATE STATUS BASED ON COUNT - Business logic in effects
      if (currentCount === 0) {
        this.status.set("idle"); // 🎯 Set idle status
        console.log("🎯 Status updated to idle"); // 📝 Log status change
      } else if (currentCount < 10) {
        this.status.set("active"); // 🎯 Set active status
        console.log("🎯 Status updated to active"); // 📝 Log status change
      } else {
        this.status.set("complete"); // 🎯 Set complete status
        console.log("🎯 Status updated to complete"); // 📝 Log status change
      }

      // ⏰ UPDATE TIMESTAMP
      this.lastUpdateTime.set(new Date().toLocaleTimeString());
    });

    // 🔄 EFFECT FOR STATUS CHANGES - Monitor status transitions
    effect(() => {
      const currentStatus = this.status(); // 🎯 Access reactive status
      console.log("🎯 Status effect triggered - new status:", currentStatus); // 📝 Log status change

      // 📋 LOG STATUS ACTIVITY
      this.logActivity("STATUS_CHANGED", currentStatus);
    });

    // 🔄 AFTER RENDER EFFECT - Execute after every render cycle
    afterRenderEffect(() => {
      console.log("🔄 afterRenderEffect triggered - DOM fully updated"); // 📝 Log render effect

      // 📊 INCREMENT RENDER COUNTER
      this.renderCycles.update((count) => count + 1);

      // 🎨 UPDATE PROGRESS BAR - Safe DOM manipulation after render
      this.updateProgressBar();

      // 📋 UPDATE DOM STATE LOGGING
      this.logDOMState();

      // 📜 SCROLL LOG CONTAINER - Keep latest entries visible
      this.scrollLogToBottom();

      // 📊 INCREMENT DOM UPDATE COUNTER
      this.domUpdates.update((count) => count + 1);
    });

    // 🔄 AFTER NEXT RENDER - One-time initialization after first render
    afterNextRender(() => {
      console.log("🚀 afterNextRender triggered - initial DOM setup"); // 📝 Log initial render

      // 🎯 INITIAL FOCUS SETUP
      this.setupInitialState();

      // 📋 LOG INITIAL ACTIVITY
      this.logActivity("COMPONENT_INITIALIZED", "Ready");
    });
  }

  ngOnInit(): void {
    console.log("🚀 Component initialized - ready for user interaction"); // 📝 Log initialization
  }

  ngOnDestroy(): void {
    console.log("🧹 Component destroyed - cleanup completed"); // 📝 Log destruction
  }

  // 🎯 USER INTERACTION METHODS - Handle user actions

  increment(): void {
    console.log("➕ Increment requested"); // 📝 Log increment request
    this.isProcessing.set(true); // ⏳ Set processing state

    // ⏱️ SIMULATE ASYNC PROCESSING
    setTimeout(() => {
      this.count.update((current) => {
        const newValue = current + 1; // ➕ Increment count
        console.log("📈 Count incremented to:", newValue); // 📝 Log new value
        return newValue;
      });

      this.isProcessing.set(false); // ✅ Clear processing state
      console.log("✅ Increment completed"); // 📝 Log completion
    }, 300);
  }

  decrement(): void {
    // 🚫 GUARD CLAUSE - Prevent negative values
    if (this.count() <= 0) {
      console.warn("⚠️ Cannot decrement below zero"); // ⚠️ Log warning
      return; // 🚪 Exit early
    }

    console.log("➖ Decrement requested"); // 📝 Log decrement request

    this.count.update((current) => {
      const newValue = current - 1; // ➖ Decrement count
      console.log("📉 Count decremented to:", newValue); // 📝 Log new value
      return newValue;
    });
  }

  reset(): void {
    console.log("🔄 Reset requested"); // 📝 Log reset request

    // 🔄 RESET ALL STATE
    this.count.set(0); // 📊 Reset count
    this.effectExecutions.set(0); // 📊 Reset effect counter
    this.renderCycles.set(0); // 📊 Reset render counter
    this.domUpdates.set(0); // 📊 Reset DOM update counter
    this.activityLog.set([]); // 📋 Clear activity log
    this.lastUpdateTime.set("Reset"); // ⏰ Update timestamp

    console.log("✅ Component state reset to initial values"); // 📝 Log reset completion
  }

  batchUpdate(): void {
    console.log("🚀 Batch update requested"); // 📝 Log batch update start
    this.isProcessing.set(true); // ⏳ Set processing state

    // 🚀 BATCH MULTIPLE UPDATES - Demonstrate efficient signal updates
    setTimeout(() => {
      const updates = [2, 4, 6, 8, 10]; // 📊 Predefined update values

      updates.forEach((value, index) => {
        setTimeout(() => {
          this.count.set(value); // 📊 Set specific value
          console.log(`🔄 Batch update ${index + 1}/5: ${value}`); // 📝 Log batch progress

          // ✅ COMPLETE PROCESSING ON LAST UPDATE
          if (index === updates.length - 1) {
            this.isProcessing.set(false); // ✅ Clear processing state
            console.log("✅ Batch update completed"); // 📝 Log batch completion
          }
        }, index * 200); // ⏱️ Staggered updates
      });
    }, 100);
  }

  simulateAsync(): void {
    console.log("⏱️ Async simulation requested"); // 📝 Log async start
    this.isProcessing.set(true); // ⏳ Set processing state

    // 🎲 SIMULATE ASYNC OPERATION - Mock API call or heavy computation
    const simulateApiCall = () => {
      return new Promise<number>((resolve) => {
        const delay = Math.random() * 2000 + 1000; // 🎲 Random delay 1-3 seconds
        const newValue = Math.floor(Math.random() * 20) + 1; // 🎲 Random value 1-20

        setTimeout(() => {
          console.log(
            `⏱️ Async operation completed after ${Math.round(delay)}ms`
          ); // 📝 Log completion
          resolve(newValue);
        }, delay);
      });
    };

    simulateApiCall().then((newValue) => {
      this.count.set(newValue); // 📊 Set new value from async operation
      this.isProcessing.set(false); // ✅ Clear processing state
      console.log("✅ Async simulation completed with value:", newValue); // 📝 Log async completion
    });
  }

  // 🔧 UTILITY METHODS - Helper functions

  private updateProgressBar(): void {
    // ✅ SAFE DOM ACCESS - afterRenderEffect guarantees availability
    if (this.progressBar?.nativeElement) {
      const percentage = this.progressPercentage(); // 📊 Get progress percentage
      const element = this.progressBar.nativeElement; // 🎯 Get DOM element

      // 🎨 APPLY VISUAL UPDATES
      element.style.width = `${percentage}%`; // 📏 Set width
      element.setAttribute("aria-valuenow", percentage.toString()); // ♿ Accessibility
      element.setAttribute("aria-valuemin", "0"); // ♿ Minimum value
      element.setAttribute("aria-valuemax", "100"); // ♿ Maximum value

      console.log("🎨 Progress bar updated to:", percentage + "%"); // 📝 Log progress update
    }
  }

  private logDOMState(): void {
    // 📊 LOG DOM STATE - Verify DOM synchronization with signals
    const countElements = document.querySelectorAll(".metric-value"); // 🔍 Find metric displays
    if (countElements.length > 0) {
      const domCount = countElements[0].textContent; // 📊 Get displayed count
      const signalCount = this.count(); // 📊 Get signal count
      console.log(
        "📊 DOM-Signal sync check - DOM:",
        domCount,
        "Signal:",
        signalCount
      ); // 📝 Log sync check
    }
  }

  private scrollLogToBottom(): void {
    // 📜 AUTO-SCROLL LOG - Keep latest entries visible
    if (this.logContainer?.nativeElement) {
      const container = this.logContainer.nativeElement; // 📋 Get log container
      container.scrollTop = container.scrollHeight; // 📜 Scroll to bottom
    }
  }

  private setupInitialState(): void {
    console.log("🎯 Setting up initial component state"); // 📝 Log initial setup

    // 🎯 INITIAL ACCESSIBILITY SETUP
    if (this.progressBar?.nativeElement) {
      this.progressBar.nativeElement.setAttribute("role", "progressbar"); // ♿ Set ARIA role
      this.progressBar.nativeElement.setAttribute(
        "aria-label",
        "Count Progress"
      ); // ♿ Set label
    }
  }

  private logActivity(action: string, value: any): void {
    const entry: ActivityEntry = {
      timestamp: new Date().toLocaleTimeString(), // ⏰ Current time
      action: action, // 📝 Action description
      value: value.toString(), // 📊 Action value
    };

    // 📋 ADD TO LOG - Maintain activity history
    this.activityLog.update((log) => {
      const newLog = [...log, entry]; // 📋 Create new log array
      // 🗑️ LIMIT LOG SIZE - Keep only last 20 entries
      return newLog.length > 20 ? newLog.slice(-20) : newLog;
    });
  }

  // 🎨 UI HELPER METHODS - Template utility functions

  getStatusEmoji(): string {
    const status = this.status(); // 🎯 Get current status
    const emojis = {
      idle: "😴", // 😴 Idle state
      active: "⚡", // ⚡ Active state
      complete: "🎉", // 🎉 Complete state
    };
    return emojis[status] || "❓"; // ❓ Unknown status fallback
  }

  getStatusDescription(): string {
    const status = this.status(); // 🎯 Get current status
    const descriptions = {
      idle: "Component is in idle state", // 😴 Idle description
      active: "Component is actively processing", // ⚡ Active description
      complete: "Component has completed its task", // 🎉 Complete description
    };
    return descriptions[status] || "Unknown status"; // ❓ Unknown status fallback
  }
}

// 📋 TYPE DEFINITIONS - Strong typing for activity logging
interface ActivityEntry {
  timestamp: string; // ⏰ When the activity occurred
  action: string; // 📝 What action was performed
  value: string; // 📊 The value associated with the action
}
```

---

## 🎨 Material Design V18 {#material-design-v18}

Angular Material Design V18 introduces Material Design 3 (Material You) with enhanced theming, new components, and improved accessibility.

### **Material Design 3 Theming**

The new theming system provides more flexible color schemes and typography:

```scss
// theme.scss - Material Design 3 Theme Configuration
@use "@angular/material" as mat;

// 🎨 MATERIAL DESIGN 3 CORE - Import the new M3 core styles
@include mat.core();

// 🎨 COLOR PALETTE DEFINITIONS - Define brand-specific color schemes
$primary-palette: mat.define-palette(
  mat.$azure-palette,
  500
); // 🔵 Primary brand color (Azure blue)
$accent-palette: mat.define-palette(
  mat.$rose-palette,
  200
); // 🌸 Accent color (Rose pink)
$warn-palette: mat.define-palette(
  mat.$red-palette
); // 🔴 Warning/error color (Red)

// 🌈 CUSTOM BRAND COLORS - Additional brand colors for Material You
$custom-colors: (
  // 🎨 SURFACE COLORS - Background and surface variations
  "surface-dim": #f1f3f4,
  // 🌫️ Dimmed surface color
  "surface-bright": #ffffff,
  // ✨ Bright surface color
  "surface-container": #f8f9fa,
  // 📦 Container surface color
  "surface-container-low": #f1f3f4,
  // 📦 Low container surface
  "surface-container-high": #e8eaed,

  // 📦 High container surface
  // 🎯 ACCENT COLORS - Secondary brand colors
  "secondary": #5f6368,
  // 🔘 Secondary text/elements
  "tertiary": #1a73e8,
  // 🔹 Tertiary accent
  "quaternary": #34a853,

  // 🟢 Success/positive actions
  // 🎭 STATE COLORS - Interactive states
  "hover-overlay": rgba(0, 0, 0, 0.04),
  // 👆 Hover state overlay
  "pressed-overlay": rgba(0, 0, 0, 0.1),
  // 👇 Pressed state overlay
  "focus-overlay": rgba(26, 115, 232, 0.12),
  // 🎯 Focus state overlay
  "disabled-overlay": rgba(0, 0, 0, 0.38),
  // 🚫 Disabled state overlay
);

// 🌓 LIGHT THEME CONFIGURATION - Primary theme definition
$theme: mat.define-theme(
  (
    color: (
      theme-type: light,
      // 🌞 Light theme variant
      primary: $primary-palette,
      // 🔵 Primary color scheme
      tertiary: $accent-palette,
      // 🌸 Tertiary color scheme
      use-system-variables: true,
      // 🔧 Enable CSS custom properties
    ),
    typography: (
      brand-family: "Inter, system-ui, sans-serif",
      // 🔤 Modern brand font
      plain-family: "Roboto, Arial, sans-serif",
      // 🔤 Readable body font
      use-system-variables: true,
      // 🔧 Enable CSS font variables
    ),
    density: (
      scale: 0,
      // 📏 Standard density (0 = default)
    ),
  )
);

// 🌃 DARK THEME CONFIGURATION - Alternative dark mode theme
$dark-theme: mat.define-theme(
  (
    color: (
      theme-type: dark,
      // 🌙 Dark theme variant
      primary: $primary-palette,
      // 🔵 Same primary colors
      tertiary: $accent-palette,
      // 🌸 Same tertiary colors
      use-system-variables: true,
      // 🔧 Enable CSS custom properties
    ),
    typography: (
      brand-family: "Inter, system-ui, sans-serif",
      // 🔤 Consistent fonts
      plain-family: "Roboto, Arial, sans-serif",
      // 🔤 Consistent fonts
      use-system-variables: true,
      // 🔧 Enable CSS font variables
    ),
    density: (
      scale: 0,
      // 📏 Consistent density
    ),
  )
);

// 🎯 APPLY BASE THEME - Set default theme for all components
@include mat.all-component-themes($theme);

// 🎨 MATERIAL DESIGN 3 ENHANCEMENTS - Apply M3 specific features
@include mat.system-level-colors($theme); // 🌈 System color tokens
@include mat.system-level-typography($theme); // 🔤 System typography tokens

// 🌃 DARK THEME SELECTOR - Auto dark mode support
@media (prefers-color-scheme: dark) {
  .mat-app-background {
    @include mat.all-component-colors(
      $dark-theme
    ); // 🌙 Apply dark theme colors
  }
}

// 🎯 MANUAL DARK MODE CLASS - User-controlled dark mode
.dark-theme {
  @include mat.all-component-colors($dark-theme); // 🌙 Dark theme override

  // 🔧 CUSTOM DARK MODE VARIABLES - Additional dark mode customizations
  --mat-sys-surface: #121212; // 🌑 Dark surface
  --mat-sys-on-surface: #e8eaed; // 🌕 Light text on dark
  --mat-sys-surface-container: #1e1e1e; // 📦 Dark container
}

// 🎨 HIGH CONTRAST MODE - Accessibility enhancement
@media (prefers-contrast: high) {
  .mat-mdc-button {
    --mat-mdc-button-persistent-ripple-color: currentColor; // 🌊 High contrast ripples
    border: 2px solid currentColor !important; // 🖼️ Strong borders
  }

  .mat-mdc-form-field {
    --mat-form-field-container-text-color: #000000; // 🔤 High contrast text
    --mat-form-field-disabled-input-text-color: #666666; // 🔤 Disabled text
  }
}

// 🎯 REDUCED MOTION - Respect user motion preferences
@media (prefers-reduced-motion: reduce) {
  .mat-mdc-tab-group {
    --mat-tab-animation-duration: 0ms !important; // 🚫 Disable tab animations
  }

  .mat-mdc-button {
    --mat-mdc-button-state-layer-color: transparent; // 🚫 Disable ripple animations
  }
}

// 🎨 CUSTOM COMPONENT THEMING - Brand-specific customizations

// 📊 Enhanced Card Styling
.mat-mdc-card {
  --mat-card-container-color: var(
    --mat-sys-surface-container
  ); // 📦 Container background
  --mat-card-container-shape: 16px; // 🔘 Rounded corners

  // ✨ ELEVATION ENHANCEMENT - Better shadow system
  &.elevated {
    box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.12), // 📏 Small shadow
      0px 1px 2px rgba(0, 0, 0, 0.24); // 📏 Medium shadow
  }

  // 🎯 HOVER STATE - Interactive feedback
  &:hover {
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.12), // 📏 Larger shadow on hover
      0px 2px 4px rgba(0, 0, 0, 0.08); // 📏 Subtle depth
    transform: translateY(-2px); // ⬆️ Slight lift effect
    transition: all 0.2s ease; // 🎨 Smooth transition
  }
}

// 🎮 Enhanced Button Styling
.mat-mdc-button,
.mat-mdc-raised-button,
.mat-mdc-outlined-button {
  --mat-mdc-button-container-shape: 20px; // 🔘 More rounded buttons
  --mat-mdc-button-horizontal-padding: 24px; // 📏 Generous padding

  // 🎯 FOCUS ENHANCEMENT - Better focus indicators
  &:focus-visible {
    outline: 3px solid var(--mat-sys-primary); // 🎯 Strong focus outline
    outline-offset: 2px; // 📏 Offset from button
  }
}

// 📝 Enhanced Form Field Styling
.mat-mdc-form-field {
  --mat-form-field-container-shape: 12px; // 🔘 Rounded form fields

  // 🎯 FOCUS STATE - Enhanced focus styling
  &.mat-focused {
    .mat-mdc-form-field-outline-thick {
      --mat-form-field-outline-color: var(
        --mat-sys-primary
      ); // 🔵 Primary color focus
      --mat-form-field-outline-width: 3px; // 📏 Thicker focus outline
    }
  }

  // ❌ ERROR STATE - Clear error indication
  &.mat-form-field-invalid {
    .mat-mdc-form-field-outline-thick {
      --mat-form-field-outline-color: var(
        --mat-sys-error
      ); // 🔴 Error color outline
    }
  }
}

// 📊 Enhanced Chip Styling
.mat-mdc-chip-set {
  gap: 8px; // 📏 Space between chips

  .mat-mdc-chip {
    --mat-chip-container-shape: 16px; // 🔘 Rounded chips
    --mat-chip-with-avatar-leading-space: 4px; // 📏 Avatar spacing

    // ✅ SELECTED STATE - Clear selection indication
    &.mat-mdc-chip-selected {
      --mat-chip-selected-container-color: var(
        --mat-sys-primary-container
      ); // 🎨 Selection background
      --mat-chip-selected-label-text-color: var(
        --mat-sys-on-primary-container
      ); // 🔤 Selection text
    }
  }
}

// 📑 Enhanced Tab Styling
.mat-mdc-tab-group {
  --mat-tab-header-label-text-color: var(
    --mat-sys-on-surface
  ); // 🔤 Tab text color
  --mat-tab-header-active-label-text-color: var(
    --mat-sys-primary
  ); // 🔤 Active tab color
  --mat-tab-header-active-ripple-color: var(
    --mat-sys-primary
  ); // 🌊 Active ripple color

  // 🎯 FOCUS ENHANCEMENT - Better keyboard navigation
  .mat-mdc-tab:focus-visible {
    outline: 2px solid var(--mat-sys-primary); // 🎯 Focus outline
    outline-offset: -2px; // 📏 Inset outline
    border-radius: 4px; // 🔘 Rounded focus
  }
}

// 📊 DATA TABLE ENHANCEMENTS - Better table theming
.mat-mdc-table {
  --mat-table-row-item-container-color: var(
    --mat-sys-surface
  ); // 📋 Row background
  --mat-table-header-container-color: var(
    --mat-sys-surface-variant
  ); // 📋 Header background

  // 🎯 HOVER STATE - Row highlighting
  .mat-mdc-row:hover {
    background-color: var(--mat-sys-surface-container); // 🎨 Hover background
  }

  // ✅ SELECTED STATE - Clear selection
  .mat-mdc-row.selected {
    background-color: var(
      --mat-sys-primary-container
    ); // 🎨 Selection background
    color: var(--mat-sys-on-primary-container); // 🔤 Selection text
  }
}

// 🔧 UTILITY CLASSES - Helper classes for consistent theming

.surface-container {
  background-color: var(--mat-sys-surface-container); // 📦 Standard container
  color: var(--mat-sys-on-surface); // 🔤 Container text
  border-radius: 12px; // 🔘 Container shape
}

.primary-container {
  background-color: var(--mat-sys-primary-container); // 🎨 Primary container
  color: var(--mat-sys-on-primary-container); // 🔤 Primary container text
  border-radius: 16px; // 🔘 Primary shape
}

.surface-variant {
  background-color: var(--mat-sys-surface-variant); // 🎨 Variant surface
  color: var(--mat-sys-on-surface-variant); // 🔤 Variant text
}

// 📱 RESPONSIVE BREAKPOINTS - Mobile-first theming
@media (max-width: 768px) {
  .mat-mdc-button {
    --mat-mdc-button-horizontal-padding: 16px; // 📏 Smaller mobile padding
    font-size: 14px; // 🔤 Smaller mobile text
  }

  .mat-mdc-form-field {
    width: 100%; // 📐 Full width on mobile
  }
}

/* 📋 MATERIAL DESIGN 3 BENEFITS:

✅ ENHANCED ACCESSIBILITY:
- Better contrast ratios for WCAG compliance
- Improved focus indicators for keyboard navigation
- High contrast mode support
- Reduced motion preferences

✅ BRAND CONSISTENCY:
- System-level color tokens for consistent theming
- CSS custom properties for easy customization
- Dark mode support with automatic detection
- Responsive design considerations

✅ PERFORMANCE OPTIMIZATIONS:
- CSS custom properties reduce bundle size
- Reduced redundancy in style definitions
- Better tree-shaking of unused styles
- Improved runtime performance

✅ DEVELOPER EXPERIENCE:
- Type-safe theme configuration
- Better IDE support with IntelliSense
- Clearer documentation and examples
- Easier migration path from M2 to M3
*/
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
