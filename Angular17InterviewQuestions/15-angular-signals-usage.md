# 🎯 Mastering Angular Signals: Complete Usage Guide

## 🎯 **Question Overview**

_"What are Angular Signals and how do you use them effectively in modern applications?"_

## 🧠 **Understanding Angular Signals**

Signals are a **reactive primitive** introduced in Angular 16+ that provide a new way to handle state management and change detection. They offer **fine-grained reactivity**, **automatic dependency tracking**, and **better performance** than traditional approaches.

Think of Signals as **smart variables** that know when they've changed and can automatically update anything that depends on them! 📡

## 🔍 **Why Signals Matter**

### **Traditional Problems:**

- **Manual change detection** can be error-prone
- **Zone.js overhead** impacts performance
- **Hard to track dependencies** between values
- **Complex state synchronization** in large apps

### **Signals Solutions:**

- **Automatic dependency tracking** 🎯
- **Fine-grained updates** ⚡
- **Better performance** 🚀
- **Simplified state management** 🧩
- **Zoneless compatibility** 🌐

## 🛠️ **Core Signal Types**

### **1. 📊 Basic Signals**

```typescript
// basic-signals.component.ts
import { signal, computed, effect } from "@angular/core";

@Component({
  selector: "app-basic-signals",
  template: `
    <div class="signals-demo">
      <h2>🎯 Basic Signals Demo</h2>

      <!-- Simple signal display -->
      <div class="signal-section">
        <h3>📈 Counter Signal</h3>
        <p>
          Count: <span class="value">{{ count() }}</span>
        </p>
        <div class="controls">
          <button (click)="increment()">+1</button>
          <button (click)="decrement()">-1</button>
          <button (click)="reset()">Reset</button>
          <button (click)="setRandom()">Random</button>
        </div>
      </div>

      <!-- String signal -->
      <div class="signal-section">
        <h3>📝 Message Signal</h3>
        <p>
          Message: <span class="value">{{ message() }}</span>
        </p>
        <input
          [value]="message()"
          (input)="updateMessage($event)"
          placeholder="Type a message"
        />
        <button (click)="clearMessage()">Clear</button>
      </div>

      <!-- Boolean signal -->
      <div class="signal-section">
        <h3>🔘 Toggle Signal</h3>
        <p>
          Active:
          <span [class]="isActive() ? 'active' : 'inactive'">
            {{ isActive() ? "✅ ON" : "❌ OFF" }}
          </span>
        </p>
        <button (click)="toggle()">Toggle</button>
      </div>

      <!-- Array signal -->
      <div class="signal-section">
        <h3>📋 Items Signal</h3>
        <p>Items count: {{ items().length }}</p>
        <ul>
          <li *ngFor="let item of items(); trackBy: trackByIndex">
            {{ item }}
            <button (click)="removeItem(item)" class="remove-btn">×</button>
          </li>
        </ul>
        <div class="add-item">
          <input
            [(ngModel)]="newItem"
            placeholder="New item"
            (keyup.enter)="addItem()"
          />
          <button (click)="addItem()">Add Item</button>
        </div>
      </div>

      <!-- Object signal -->
      <div class="signal-section">
        <h3>👤 User Signal</h3>
        <div class="user-card">
          <p>Name: {{ user().name }}</p>
          <p>Email: {{ user().email }}</p>
          <p>Age: {{ user().age }}</p>
          <button (click)="updateUser()">Update User</button>
          <button (click)="incrementAge()">Birthday!</button>
        </div>
      </div>
    </div>
  `,
  styles: [
    `
      .signals-demo {
        padding: 20px;
        max-width: 800px;
        margin: 0 auto;
      }

      .signal-section {
        margin: 30px 0;
        padding: 20px;
        border: 1px solid #ddd;
        border-radius: 8px;
        background: #f9f9f9;
      }

      .signal-section h3 {
        margin-top: 0;
        color: #333;
      }

      .value {
        font-weight: bold;
        color: #007acc;
        font-size: 1.2em;
      }

      .controls {
        margin: 15px 0;
      }

      .controls button,
      .add-item button {
        margin: 5px;
        padding: 8px 16px;
        border: none;
        border-radius: 4px;
        background: #007acc;
        color: white;
        cursor: pointer;
      }

      .controls button:hover {
        background: #005999;
      }

      .active {
        color: #4caf50;
        font-weight: bold;
      }

      .inactive {
        color: #f44336;
        font-weight: bold;
      }

      input {
        padding: 8px;
        margin: 5px;
        border: 1px solid #ddd;
        border-radius: 4px;
        width: 200px;
      }

      ul {
        list-style: none;
        padding: 0;
      }

      li {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 8px;
        margin: 5px 0;
        background: white;
        border-radius: 4px;
        border: 1px solid #eee;
      }

      .remove-btn {
        background: #f44336 !important;
        color: white;
        border: none;
        border-radius: 50%;
        width: 25px;
        height: 25px;
        cursor: pointer;
        font-size: 14px;
      }

      .add-item {
        display: flex;
        align-items: center;
        margin-top: 15px;
      }

      .user-card {
        background: white;
        padding: 15px;
        border-radius: 8px;
        border: 1px solid #ddd;
      }

      .user-card p {
        margin: 8px 0;
      }

      .user-card button {
        margin: 5px;
        padding: 8px 16px;
        border: none;
        border-radius: 4px;
        background: #4caf50;
        color: white;
        cursor: pointer;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule, FormsModule],
})
export class BasicSignalsComponent {
  // 📊 Primitive signals
  count = signal(0);
  message = signal("Hello Signals!");
  isActive = signal(true);

  // 📋 Array signal
  items = signal<string[]>(["Item 1", "Item 2", "Item 3"]);

  // 👤 Object signal
  user = signal({
    name: "John Doe",
    email: "john@example.com",
    age: 30,
  });

  // Form input for new items
  newItem = "";

  // Counter operations
  increment() {
    this.count.update((value) => value + 1);
  }

  decrement() {
    this.count.update((value) => value - 1);
  }

  reset() {
    this.count.set(0);
  }

  setRandom() {
    this.count.set(Math.floor(Math.random() * 100));
  }

  // Message operations
  updateMessage(event: Event) {
    const target = event.target as HTMLInputElement;
    this.message.set(target.value);
  }

  clearMessage() {
    this.message.set("");
  }

  // Toggle operations
  toggle() {
    this.isActive.update((value) => !value);
  }

  // Array operations
  addItem() {
    if (this.newItem.trim()) {
      this.items.update((items) => [...items, this.newItem.trim()]);
      this.newItem = "";
    }
  }

  removeItem(itemToRemove: string) {
    this.items.update((items) => items.filter((item) => item !== itemToRemove));
  }

  trackByIndex(index: number): number {
    return index;
  }

  // Object operations
  updateUser() {
    this.user.update((user) => ({
      ...user,
      name: "Jane Smith",
      email: "jane@example.com",
    }));
  }

  incrementAge() {
    this.user.update((user) => ({
      ...user,
      age: user.age + 1,
    }));
  }
}
```

### **2. 🧮 Computed Signals**

```typescript
// computed-signals.component.ts
@Component({
  selector: "app-computed-signals",
  template: `
    <div class="computed-demo">
      <h2>🧮 Computed Signals Demo</h2>

      <!-- Input signals -->
      <div class="input-section">
        <h3>📥 Input Values</h3>
        <div class="input-group">
          <label>First Number:</label>
          <input
            type="number"
            [value]="firstNumber()"
            (input)="setFirstNumber($event)"
          />
        </div>
        <div class="input-group">
          <label>Second Number:</label>
          <input
            type="number"
            [value]="secondNumber()"
            (input)="setSecondNumber($event)"
          />
        </div>
        <div class="input-group">
          <label>Items:</label>
          <input
            [value]="newItemText"
            (input)="newItemText = $any($event.target).value"
            placeholder="Add item"
          />
          <button (click)="addNumberItem()">Add</button>
        </div>
      </div>

      <!-- Computed results -->
      <div class="computed-section">
        <h3>🧮 Computed Results</h3>

        <!-- Basic arithmetic -->
        <div class="result-card">
          <h4>➕ Basic Math</h4>
          <p>
            Sum: <span class="value">{{ sum() }}</span>
          </p>
          <p>
            Product: <span class="value">{{ product() }}</span>
          </p>
          <p>
            Average: <span class="value">{{ average() }}</span>
          </p>
          <p>
            Max: <span class="value">{{ maximum() }}</span>
          </p>
        </div>

        <!-- String computations -->
        <div class="result-card">
          <h4>📝 String Operations</h4>
          <p>
            Concatenation: <span class="value">{{ concatenated() }}</span>
          </p>
          <p>
            Description: <span class="value">{{ description() }}</span>
          </p>
        </div>

        <!-- Array computations -->
        <div class="result-card">
          <h4>📊 Array Statistics</h4>
          <p>Numbers: {{ numbers() | json }}</p>
          <p>
            Count: <span class="value">{{ count() }}</span>
          </p>
          <p>
            Sum: <span class="value">{{ arraySum() }}</span>
          </p>
          <p>
            Average: <span class="value">{{ arrayAverage() }}</span>
          </p>
          <p>Even Numbers: {{ evenNumbers() | json }}</p>
          <p>Sorted: {{ sortedNumbers() | json }}</p>
        </div>

        <!-- Complex computations -->
        <div class="result-card">
          <h4>🎯 Complex Calculations</h4>
          <p>
            Is Perfect Square:
            <span class="value">{{ isPerfectSquare() ? "Yes" : "No" }}</span>
          </p>
          <p>
            Factorial (sum): <span class="value">{{ factorial() }}</span>
          </p>
          <p>
            Fibonacci (first):
            <span class="value">{{ fibonacciSequence() | json }}</span>
          </p>
        </div>

        <!-- Conditional computations -->
        <div class="result-card">
          <h4>🔀 Conditional Logic</h4>
          <p>
            Status: <span [class]="statusClass()">{{ status() }}</span>
          </p>
          <p>
            Grade: <span [class]="gradeClass()">{{ grade() }}</span>
          </p>
          <p>
            Category: <span class="value">{{ category() }}</span>
          </p>
        </div>
      </div>
    </div>
  `,
  styles: [
    `
      .computed-demo {
        padding: 20px;
        max-width: 1000px;
        margin: 0 auto;
      }

      .input-section,
      .computed-section {
        margin: 30px 0;
      }

      .input-group {
        display: flex;
        align-items: center;
        margin: 10px 0;
        gap: 10px;
      }

      .input-group label {
        min-width: 120px;
        font-weight: bold;
      }

      .input-group input {
        padding: 8px;
        border: 1px solid #ddd;
        border-radius: 4px;
        width: 150px;
      }

      .input-group button {
        padding: 8px 16px;
        background: #007acc;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      }

      .computed-section {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 20px;
      }

      .result-card {
        background: white;
        border: 1px solid #ddd;
        border-radius: 8px;
        padding: 20px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      }

      .result-card h4 {
        margin-top: 0;
        color: #333;
        border-bottom: 1px solid #eee;
        padding-bottom: 10px;
      }

      .value {
        font-weight: bold;
        color: #007acc;
      }

      .status-good {
        color: #4caf50;
        font-weight: bold;
      }

      .status-warning {
        color: #ff9800;
        font-weight: bold;
      }

      .status-error {
        color: #f44336;
        font-weight: bold;
      }

      .grade-a {
        color: #4caf50;
        font-weight: bold;
      }

      .grade-b {
        color: #8bc34a;
        font-weight: bold;
      }

      .grade-c {
        color: #ff9800;
        font-weight: bold;
      }

      .grade-d {
        color: #f44336;
        font-weight: bold;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule],
})
export class ComputedSignalsComponent {
  // Input signals
  firstNumber = signal(10);
  secondNumber = signal(5);
  numbers = signal<number[]>([1, 2, 3, 4, 5]);

  // Form input
  newItemText = "";

  // Basic arithmetic computations
  sum = computed(() => this.firstNumber() + this.secondNumber());
  product = computed(() => this.firstNumber() * this.secondNumber());
  average = computed(() => (this.firstNumber() + this.secondNumber()) / 2);
  maximum = computed(() => Math.max(this.firstNumber(), this.secondNumber()));

  // String computations
  concatenated = computed(() => `${this.firstNumber()}-${this.secondNumber()}`);
  description = computed(() => {
    const first = this.firstNumber();
    const second = this.secondNumber();
    const sum = this.sum();
    return `Adding ${first} and ${second} gives us ${sum}`;
  });

  // Array computations
  count = computed(() => this.numbers().length);
  arraySum = computed(() => this.numbers().reduce((sum, num) => sum + num, 0));
  arrayAverage = computed(() => {
    const nums = this.numbers();
    return nums.length > 0 ? this.arraySum() / nums.length : 0;
  });

  evenNumbers = computed(() => this.numbers().filter((num) => num % 2 === 0));
  sortedNumbers = computed(() => [...this.numbers()].sort((a, b) => a - b));

  // Complex computations
  isPerfectSquare = computed(() => {
    const sum = this.sum();
    const sqrt = Math.sqrt(sum);
    return sqrt === Math.floor(sqrt);
  });

  factorial = computed(() => {
    const sum = this.sum();
    if (sum < 0 || sum > 10) return "N/A"; // Prevent large calculations

    let result = 1;
    for (let i = 2; i <= sum; i++) {
      result *= i;
    }
    return result;
  });

  fibonacciSequence = computed(() => {
    const first = this.firstNumber();
    const length = Math.min(Math.max(first, 1), 10); // Limit to prevent large arrays

    const sequence = [1];
    if (length > 1) sequence.push(1);

    for (let i = 2; i < length; i++) {
      sequence.push(sequence[i - 1] + sequence[i - 2]);
    }

    return sequence;
  });

  // Conditional computations
  status = computed(() => {
    const sum = this.sum();
    if (sum < 10) return "Low";
    if (sum < 50) return "Medium";
    if (sum < 100) return "High";
    return "Very High";
  });

  statusClass = computed(() => {
    const status = this.status();
    switch (status) {
      case "Low":
        return "status-error";
      case "Medium":
        return "status-warning";
      case "High":
        return "status-good";
      case "Very High":
        return "status-good";
      default:
        return "";
    }
  });

  grade = computed(() => {
    const average = this.average();
    if (average >= 90) return "A";
    if (average >= 80) return "B";
    if (average >= 70) return "C";
    return "D";
  });

  gradeClass = computed(() => `grade-${this.grade().toLowerCase()}`);

  category = computed(() => {
    const count = this.count();
    const sum = this.arraySum();

    if (count === 0) return "Empty";
    if (sum < 0) return "Negative";
    if (sum === 0) return "Zero";
    if (sum < count * 5) return "Low Average";
    if (sum < count * 10) return "Medium Average";
    return "High Average";
  });

  // Input handlers
  setFirstNumber(event: Event) {
    const target = event.target as HTMLInputElement;
    const value = parseFloat(target.value) || 0;
    this.firstNumber.set(value);
  }

  setSecondNumber(event: Event) {
    const target = event.target as HTMLInputElement;
    const value = parseFloat(target.value) || 0;
    this.secondNumber.set(value);
  }

  addNumberItem() {
    const value = parseFloat(this.newItemText);
    if (!isNaN(value)) {
      this.numbers.update((nums) => [...nums, value]);
      this.newItemText = "";
    }
  }
}
```

### **3. ⚡ Effect Signals**

```typescript
// effect-signals.component.ts
@Component({
  selector: "app-effect-signals",
  template: `
    <div class="effects-demo">
      <h2>⚡ Effect Signals Demo</h2>

      <!-- Controls -->
      <div class="controls-section">
        <h3>🎛️ Controls</h3>
        <div class="control-group">
          <label>User Name:</label>
          <input
            [value]="userName()"
            (input)="setUserName($event)"
            placeholder="Enter name"
          />
        </div>

        <div class="control-group">
          <label>Theme:</label>
          <select [value]="theme()" (change)="setTheme($event)">
            <option value="light">Light</option>
            <option value="dark">Dark</option>
            <option value="auto">Auto</option>
          </select>
        </div>

        <div class="control-group">
          <label>Counter:</label>
          <span class="counter-display">{{ count() }}</span>
          <button (click)="increment()">+</button>
          <button (click)="decrement()">-</button>
        </div>

        <div class="control-group">
          <label>Auto Increment:</label>
          <button
            (click)="toggleAutoIncrement()"
            [class.active]="autoIncrement()"
          >
            {{ autoIncrement() ? "Stop" : "Start" }}
          </button>
        </div>
      </div>

      <!-- Effect Logs -->
      <div class="logs-section">
        <h3>📋 Effect Logs</h3>
        <div class="log-container">
          <div
            *ngFor="let log of logs(); trackBy: trackByIndex"
            class="log-entry"
            [class]="log.type"
          >
            <span class="timestamp">{{ log.timestamp | date : "short" }}</span>
            <span class="message">{{ log.message }}</span>
          </div>
        </div>
        <button (click)="clearLogs()" class="clear-btn">Clear Logs</button>
      </div>

      <!-- Analytics -->
      <div class="analytics-section">
        <h3>📊 Analytics</h3>
        <div class="analytics-grid">
          <div class="analytics-card">
            <h4>User Activity</h4>
            <p>Name changes: {{ nameChangeCount() }}</p>
            <p>Theme changes: {{ themeChangeCount() }}</p>
            <p>Total clicks: {{ clickCount() }}</p>
          </div>

          <div class="analytics-card">
            <h4>Counter Stats</h4>
            <p>Current: {{ count() }}</p>
            <p>Max reached: {{ maxCount() }}</p>
            <p>Changes: {{ counterChangeCount() }}</p>
          </div>

          <div class="analytics-card">
            <h4>Session Info</h4>
            <p>Duration: {{ sessionDuration() }} seconds</p>
            <p>Auto increment cycles: {{ autoIncrementCycles() }}</p>
          </div>
        </div>
      </div>

      <!-- Derived Data -->
      <div class="derived-section">
        <h3>🔄 Derived Data</h3>
        <div class="derived-card">
          <h4>User Profile</h4>
          <p>Name: {{ userProfile().name }}</p>
          <p>Display Name: {{ userProfile().displayName }}</p>
          <p>Theme: {{ userProfile().theme }}</p>
          <p>Activity Level: {{ userProfile().activityLevel }}</p>
          <p>Last Active: {{ userProfile().lastActive | date : "medium" }}</p>
        </div>
      </div>
    </div>
  `,
  styles: [
    `
      .effects-demo {
        padding: 20px;
        max-width: 1200px;
        margin: 0 auto;
      }

      .controls-section,
      .logs-section,
      .analytics-section,
      .derived-section {
        margin: 30px 0;
        padding: 20px;
        border: 1px solid #ddd;
        border-radius: 8px;
        background: white;
      }

      .control-group {
        display: flex;
        align-items: center;
        margin: 15px 0;
        gap: 15px;
      }

      .control-group label {
        min-width: 120px;
        font-weight: bold;
      }

      .control-group input,
      .control-group select {
        padding: 8px;
        border: 1px solid #ddd;
        border-radius: 4px;
      }

      .control-group button {
        padding: 8px 16px;
        border: none;
        border-radius: 4px;
        background: #007acc;
        color: white;
        cursor: pointer;
        margin: 0 5px;
      }

      .control-group button.active {
        background: #f44336;
      }

      .counter-display {
        font-weight: bold;
        font-size: 1.2em;
        color: #007acc;
        margin: 0 15px;
      }

      .log-container {
        background: #f5f5f5;
        padding: 15px;
        border-radius: 4px;
        max-height: 300px;
        overflow-y: auto;
        margin-bottom: 15px;
      }

      .log-entry {
        display: flex;
        margin: 5px 0;
        padding: 8px;
        border-radius: 4px;
        background: white;
        border-left: 3px solid #ddd;
      }

      .log-entry.info {
        border-left-color: #2196f3;
      }

      .log-entry.success {
        border-left-color: #4caf50;
      }

      .log-entry.warning {
        border-left-color: #ff9800;
      }

      .log-entry.error {
        border-left-color: #f44336;
      }

      .timestamp {
        font-size: 0.8em;
        color: #666;
        margin-right: 15px;
        min-width: 120px;
      }

      .message {
        flex: 1;
      }

      .clear-btn {
        padding: 8px 16px;
        background: #ff9800;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      }

      .analytics-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 20px;
      }

      .analytics-card,
      .derived-card {
        background: #f9f9f9;
        padding: 15px;
        border-radius: 8px;
        border: 1px solid #eee;
      }

      .analytics-card h4,
      .derived-card h4 {
        margin-top: 0;
        color: #333;
        border-bottom: 1px solid #ddd;
        padding-bottom: 8px;
      }

      .analytics-card p,
      .derived-card p {
        margin: 8px 0;
        display: flex;
        justify-content: space-between;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule],
})
export class EffectSignalsComponent implements OnInit, OnDestroy {
  // Primary signals
  userName = signal("");
  theme = signal<"light" | "dark" | "auto">("light");
  count = signal(0);
  autoIncrement = signal(false);

  // Analytics signals
  logs = signal<LogEntry[]>([]);
  nameChangeCount = signal(0);
  themeChangeCount = signal(0);
  clickCount = signal(0);
  counterChangeCount = signal(0);
  maxCount = signal(0);
  sessionStartTime = signal(Date.now());
  autoIncrementCycles = signal(0);

  // Computed signals
  sessionDuration = computed(() =>
    Math.floor((Date.now() - this.sessionStartTime()) / 1000)
  );

  userProfile = computed(() => ({
    name: this.userName() || "Anonymous",
    displayName: this.userName() || "Guest User",
    theme: this.theme(),
    activityLevel: this.getActivityLevel(),
    lastActive: new Date(),
  }));

  private autoIncrementInterval?: number;
  private effects: EffectRef[] = [];

  ngOnInit() {
    this.setupEffects();
  }

  ngOnDestroy() {
    // Cleanup effects
    this.effects.forEach((effectRef) => {
      effectRef.destroy();
    });

    // Clear interval
    if (this.autoIncrementInterval) {
      clearInterval(this.autoIncrementInterval);
    }
  }

  private setupEffects() {
    // Effect 1: Log user name changes
    const nameEffect = effect(() => {
      const name = this.userName();
      if (name) {
        this.addLog(`User name changed to: ${name}`, "info");
        this.nameChangeCount.update((count) => count + 1);
      }
    });
    this.effects.push(nameEffect);

    // Effect 2: Log theme changes and apply to document
    const themeEffect = effect(() => {
      const currentTheme = this.theme();
      this.addLog(`Theme changed to: ${currentTheme}`, "info");
      this.themeChangeCount.update((count) => count + 1);

      // Apply theme to document
      document.body.className = `theme-${currentTheme}`;
    });
    this.effects.push(themeEffect);

    // Effect 3: Track counter changes and max value
    const counterEffect = effect(() => {
      const currentCount = this.count();
      const currentMax = this.maxCount();

      this.counterChangeCount.update((count) => count + 1);

      if (currentCount > currentMax) {
        this.maxCount.set(currentCount);
        this.addLog(`New max count reached: ${currentCount}`, "success");
      }

      // Log significant milestones
      if (currentCount % 10 === 0 && currentCount !== 0) {
        this.addLog(`Counter milestone: ${currentCount}`, "warning");
      }
    });
    this.effects.push(counterEffect);

    // Effect 4: Handle auto increment
    const autoIncrementEffect = effect(() => {
      const shouldAutoIncrement = this.autoIncrement();

      if (shouldAutoIncrement) {
        this.addLog("Auto increment started", "success");
        this.autoIncrementInterval = window.setInterval(() => {
          this.count.update((count) => count + 1);
          this.autoIncrementCycles.update((cycles) => cycles + 1);
        }, 500);
      } else {
        this.addLog("Auto increment stopped", "warning");
        if (this.autoIncrementInterval) {
          clearInterval(this.autoIncrementInterval);
          this.autoIncrementInterval = undefined;
        }
      }
    });
    this.effects.push(autoIncrementEffect);

    // Effect 5: User activity tracking
    const activityEffect = effect(() => {
      const name = this.userName();
      const theme = this.theme();
      const count = this.count();

      // Track overall user activity
      const activityLevel = this.getActivityLevel();

      if (activityLevel === "High") {
        this.addLog("High user activity detected!", "info");
      }
    });
    this.effects.push(activityEffect);

    // Effect 6: Session duration tracking
    const sessionEffect = effect(() => {
      const duration = this.sessionDuration();

      // Log session milestones
      if (duration > 0 && duration % 30 === 0) {
        // Every 30 seconds
        this.addLog(`Session duration: ${duration} seconds`, "info");
      }
    });
    this.effects.push(sessionEffect);

    // Effect 7: Local storage persistence
    const persistenceEffect = effect(() => {
      const userData = {
        userName: this.userName(),
        theme: this.theme(),
        count: this.count(),
      };

      localStorage.setItem("angular-signals-demo", JSON.stringify(userData));
    });
    this.effects.push(persistenceEffect);
  }

  // Event handlers
  setUserName(event: Event) {
    const target = event.target as HTMLInputElement;
    this.userName.set(target.value);
    this.clickCount.update((count) => count + 1);
  }

  setTheme(event: Event) {
    const target = event.target as HTMLSelectElement;
    this.theme.set(target.value as any);
    this.clickCount.update((count) => count + 1);
  }

  increment() {
    this.count.update((count) => count + 1);
    this.clickCount.update((count) => count + 1);
  }

  decrement() {
    this.count.update((count) => count - 1);
    this.clickCount.update((count) => count + 1);
  }

  toggleAutoIncrement() {
    this.autoIncrement.update((value) => !value);
    this.clickCount.update((count) => count + 1);
  }

  clearLogs() {
    this.logs.set([]);
    this.addLog("Logs cleared", "warning");
  }

  trackByIndex(index: number): number {
    return index;
  }

  // Helper methods
  private addLog(
    message: string,
    type: "info" | "success" | "warning" | "error" = "info"
  ) {
    const log: LogEntry = {
      id: Date.now(),
      message,
      type,
      timestamp: new Date(),
    };

    this.logs.update((logs) => [...logs, log]);

    // Limit logs to prevent memory issues
    if (this.logs().length > 50) {
      this.logs.update((logs) => logs.slice(-25));
    }
  }

  private getActivityLevel(): "Low" | "Medium" | "High" {
    const totalChanges =
      this.nameChangeCount() +
      this.themeChangeCount() +
      this.counterChangeCount();
    const clicksPerMinute =
      this.clickCount() / Math.max(this.sessionDuration() / 60, 1);

    if (totalChanges > 20 || clicksPerMinute > 10) return "High";
    if (totalChanges > 10 || clicksPerMinute > 5) return "Medium";
    return "Low";
  }
}

interface LogEntry {
  id: number;
  message: string;
  type: "info" | "success" | "warning" | "error";
  timestamp: Date;
}
```

## 🌐 **Advanced Signal Patterns**

### **1. 🔄 Signal-Based State Management**

```typescript
// advanced-state.service.ts - Enterprise signal patterns
@Injectable({
  providedIn: "root",
})
export class AdvancedStateService {
  // Core application state
  private readonly _users = signal<User[]>([]);
  private readonly _selectedUser = signal<User | null>(null);
  private readonly _loading = signal(false);
  private readonly _error = signal<string | null>(null);

  // Filters and search
  private readonly _searchTerm = signal("");
  private readonly _filterStatus = signal<UserStatus | "all">("all");
  private readonly _sortBy = signal<"name" | "email" | "lastActive">("name");
  private readonly _sortDirection = signal<"asc" | "desc">("asc");

  // Readonly access to state
  readonly users = this._users.asReadonly();
  readonly selectedUser = this._selectedUser.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly error = this._error.asReadonly();

  // Filter signals
  readonly searchTerm = this._searchTerm.asReadonly();
  readonly filterStatus = this._filterStatus.asReadonly();
  readonly sortBy = this._sortBy.asReadonly();
  readonly sortDirection = this._sortDirection.asReadonly();

  // Computed state
  readonly filteredUsers = computed(() => {
    const users = this._users();
    const search = this._searchTerm().toLowerCase();
    const status = this._filterStatus();

    return users.filter((user) => {
      // Search filter
      const matchesSearch =
        !search ||
        user.name.toLowerCase().includes(search) ||
        user.email.toLowerCase().includes(search);

      // Status filter
      const matchesStatus = status === "all" || user.status === status;

      return matchesSearch && matchesStatus;
    });
  });

  readonly sortedUsers = computed(() => {
    const users = this.filteredUsers();
    const sortBy = this._sortBy();
    const direction = this._sortDirection();

    return [...users].sort((a, b) => {
      let comparison = 0;

      switch (sortBy) {
        case "name":
          comparison = a.name.localeCompare(b.name);
          break;
        case "email":
          comparison = a.email.localeCompare(b.email);
          break;
        case "lastActive":
          comparison = a.lastActive.getTime() - b.lastActive.getTime();
          break;
      }

      return direction === "asc" ? comparison : -comparison;
    });
  });

  readonly userStats = computed(() => {
    const users = this._users();
    return {
      total: users.length,
      active: users.filter((u) => u.status === "active").length,
      inactive: users.filter((u) => u.status === "inactive").length,
      pending: users.filter((u) => u.status === "pending").length,
      averageAge:
        users.length > 0
          ? users.reduce((sum, u) => sum + u.age, 0) / users.length
          : 0,
    };
  });

  // Actions
  async loadUsers(): Promise<void> {
    this._loading.set(true);
    this._error.set(null);

    try {
      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 1000));

      const mockUsers: User[] = [
        {
          id: "1",
          name: "Alice Johnson",
          email: "alice@example.com",
          age: 28,
          status: "active",
          lastActive: new Date(),
          preferences: { theme: "light", notifications: true },
        },
        {
          id: "2",
          name: "Bob Smith",
          email: "bob@example.com",
          age: 35,
          status: "inactive",
          lastActive: new Date(Date.now() - 86400000),
          preferences: { theme: "dark", notifications: false },
        },
        // ... more users
      ];

      this._users.set(mockUsers);
    } catch (error) {
      this._error.set("Failed to load users");
      console.error("Error loading users:", error);
    } finally {
      this._loading.set(false);
    }
  }

  selectUser(user: User | null): void {
    this._selectedUser.set(user);
  }

  updateUser(userId: string, updates: Partial<User>): void {
    this._users.update((users) =>
      users.map((user) =>
        user.id === userId
          ? { ...user, ...updates, lastActive: new Date() }
          : user
      )
    );
  }

  deleteUser(userId: string): void {
    this._users.update((users) => users.filter((user) => user.id !== userId));

    // Clear selection if deleted user was selected
    if (this._selectedUser()?.id === userId) {
      this._selectedUser.set(null);
    }
  }

  addUser(userData: Omit<User, "id" | "lastActive">): void {
    const newUser: User = {
      ...userData,
      id: Date.now().toString(),
      lastActive: new Date(),
    };

    this._users.update((users) => [...users, newUser]);
  }

  // Filter actions
  setSearchTerm(term: string): void {
    this._searchTerm.set(term);
  }

  setFilterStatus(status: UserStatus | "all"): void {
    this._filterStatus.set(status);
  }

  setSorting(
    sortBy: "name" | "email" | "lastActive",
    direction?: "asc" | "desc"
  ): void {
    this._sortBy.set(sortBy);
    if (direction) {
      this._sortDirection.set(direction);
    } else {
      // Toggle direction if same field
      this._sortDirection.update((current) =>
        current === "asc" ? "desc" : "asc"
      );
    }
  }

  clearFilters(): void {
    this._searchTerm.set("");
    this._filterStatus.set("all");
    this._sortBy.set("name");
    this._sortDirection.set("asc");
  }

  // Bulk operations
  bulkUpdateStatus(userIds: string[], status: UserStatus): void {
    this._users.update((users) =>
      users.map((user) =>
        userIds.includes(user.id)
          ? { ...user, status, lastActive: new Date() }
          : user
      )
    );
  }

  bulkDelete(userIds: string[]): void {
    this._users.update((users) =>
      users.filter((user) => !userIds.includes(user.id))
    );

    // Clear selection if deleted
    if (userIds.includes(this._selectedUser()?.id || "")) {
      this._selectedUser.set(null);
    }
  }
}

type UserStatus = "active" | "inactive" | "pending";

interface User {
  id: string;
  name: string;
  email: string;
  age: number;
  status: UserStatus;
  lastActive: Date;
  preferences: {
    theme: "light" | "dark";
    notifications: boolean;
  };
}
```

### **2. 🧪 Signal Testing Utilities**

```typescript
// signal-testing.spec.ts - Comprehensive testing patterns
describe("Signals Testing", () => {
  describe("Basic Signal Tests", () => {
    it("should update signal values", () => {
      const count = signal(0);

      expect(count()).toBe(0);

      count.set(5);
      expect(count()).toBe(5);

      count.update((val) => val + 3);
      expect(count()).toBe(8);
    });

    it("should work with object signals", () => {
      const user = signal({ name: "John", age: 30 });

      expect(user().name).toBe("John");
      expect(user().age).toBe(30);

      user.update((u) => ({ ...u, age: 31 }));
      expect(user().age).toBe(31);
      expect(user().name).toBe("John"); // Unchanged
    });
  });

  describe("Computed Signal Tests", () => {
    it("should automatically update when dependencies change", () => {
      const firstName = signal("John");
      const lastName = signal("Doe");
      const fullName = computed(() => `${firstName()} ${lastName()}`);

      expect(fullName()).toBe("John Doe");

      firstName.set("Jane");
      expect(fullName()).toBe("Jane Doe");

      lastName.set("Smith");
      expect(fullName()).toBe("Jane Smith");
    });

    it("should handle complex computations", () => {
      const numbers = signal([1, 2, 3, 4, 5]);
      const sum = computed(() => numbers().reduce((a, b) => a + b, 0));
      const average = computed(() => sum() / numbers().length);

      expect(sum()).toBe(15);
      expect(average()).toBe(3);

      numbers.update((nums) => [...nums, 6]);
      expect(sum()).toBe(21);
      expect(average()).toBe(3.5);
    });
  });

  describe("Effect Signal Tests", () => {
    it("should run effects when dependencies change", fakeAsync(() => {
      const counter = signal(0);
      const effectCallCount = signal(0);

      const effectRef = effect(() => {
        counter(); // Access signal to create dependency
        effectCallCount.update((count) => count + 1);
      });

      tick(); // Allow effect to run
      expect(effectCallCount()).toBe(1);

      counter.set(1);
      tick();
      expect(effectCallCount()).toBe(2);

      counter.set(2);
      tick();
      expect(effectCallCount()).toBe(3);

      effectRef.destroy();
    }));

    it("should handle side effects properly", fakeAsync(() => {
      const user = signal<User | null>(null);
      const logs: string[] = [];

      const effectRef = effect(() => {
        const currentUser = user();
        if (currentUser) {
          logs.push(`User logged in: ${currentUser.name}`);
        } else {
          logs.push("User logged out");
        }
      });

      tick();
      expect(logs).toEqual(["User logged out"]);

      user.set({ id: "1", name: "John", email: "john@example.com" } as User);
      tick();
      expect(logs).toEqual(["User logged out", "User logged in: John"]);

      user.set(null);
      tick();
      expect(logs).toEqual([
        "User logged out",
        "User logged in: John",
        "User logged out",
      ]);

      effectRef.destroy();
    }));
  });

  describe("Advanced Signal Patterns", () => {
    let stateService: AdvancedStateService;

    beforeEach(() => {
      TestBed.configureTestingModule({});
      stateService = TestBed.inject(AdvancedStateService);
    });

    it("should filter users correctly", () => {
      // Setup test data
      const testUsers: User[] = [
        {
          id: "1",
          name: "Alice Johnson",
          email: "alice@example.com",
          age: 28,
          status: "active",
          lastActive: new Date(),
          preferences: { theme: "light", notifications: true },
        },
        {
          id: "2",
          name: "Bob Smith",
          email: "bob@example.com",
          age: 35,
          status: "inactive",
          lastActive: new Date(),
          preferences: { theme: "dark", notifications: false },
        },
      ];

      // Simulate loading users
      stateService["_users"].set(testUsers);

      // Test initial state
      expect(stateService.sortedUsers().length).toBe(2);

      // Test search filter
      stateService.setSearchTerm("alice");
      expect(stateService.sortedUsers().length).toBe(1);
      expect(stateService.sortedUsers()[0].name).toBe("Alice Johnson");

      // Test status filter
      stateService.setSearchTerm("");
      stateService.setFilterStatus("active");
      expect(stateService.sortedUsers().length).toBe(1);
      expect(stateService.sortedUsers()[0].status).toBe("active");
    });

    it("should compute user statistics correctly", () => {
      const testUsers: User[] = [
        {
          id: "1",
          name: "User 1",
          email: "user1@example.com",
          age: 20,
          status: "active",
          lastActive: new Date(),
          preferences: { theme: "light", notifications: true },
        },
        {
          id: "2",
          name: "User 2",
          email: "user2@example.com",
          age: 30,
          status: "inactive",
          lastActive: new Date(),
          preferences: { theme: "dark", notifications: false },
        },
        {
          id: "3",
          name: "User 3",
          email: "user3@example.com",
          age: 40,
          status: "pending",
          lastActive: new Date(),
          preferences: { theme: "light", notifications: true },
        },
      ];

      stateService["_users"].set(testUsers);

      const stats = stateService.userStats();
      expect(stats.total).toBe(3);
      expect(stats.active).toBe(1);
      expect(stats.inactive).toBe(1);
      expect(stats.pending).toBe(1);
      expect(stats.averageAge).toBe(30);
    });
  });
});
```

## 📊 **Angular Version Comparison**

| Feature                  | Angular 16 | Angular 17           | Angular 19        |
| ------------------------ | ---------- | -------------------- | ----------------- |
| **Basic Signals**        | Stable     | Enhanced             | Optimized         |
| **Computed Signals**     | Available  | Improved performance | Auto-optimization |
| **Effect Signals**       | Basic      | Enhanced cleanup     | Advanced patterns |
| **Zoneless Integration** | Limited    | Good                 | Full integration  |
| **Performance**          | Baseline   | 15% improvement      | 30% improvement   |

## 🎯 **Key Takeaways**

### **🚀 Signal Advantages:**

1. **Fine-grained Reactivity** - Only update what actually changed
2. **Automatic Dependency Tracking** - No manual subscriptions needed
3. **Better Performance** - More efficient than traditional change detection
4. **Simpler State Management** - Less boilerplate, clearer data flow
5. **Zoneless Ready** - Perfect for modern Angular applications

### **📋 Best Practices:**

1. **Use signals for all reactive state** in components and services
2. **Prefer computed over effects** for derived values
3. **Keep effects focused and simple** - avoid complex logic
4. **Use readonly signals** for public APIs
5. **Test signal interactions** thoroughly

### **🚨 Common Pitfalls:**

- Forgetting to call signals as functions: `count` instead of `count()`
- Creating circular dependencies between computed signals
- Overusing effects instead of computed signals
- Not cleaning up effects in components
- Mixing signals with traditional observables unnecessarily

Angular Signals represent the future of reactive programming in Angular - embrace them for cleaner, more performant applications! ✨
