# 🔍 Deep Dive into Angular Change Detection

## 🎯 **Question Overview**

_"Explain the change detection mechanism in Angular and how it's triggered?"_

## 🧠 **Understanding Change Detection**

Change detection is Angular's mechanism for synchronizing the component tree with the underlying data model. It's the process that determines when the UI needs to be updated based on changes in application state.

Think of it as Angular's way of asking: _"Has anything changed that would affect what the user sees?"_ 🤔

## ⚙️ **How Change Detection Works**

### **1. 🌳 The Change Detection Tree**

```typescript
// Change detection flows from root to leaf components
@Component({
  selector: "app-root",
  template: `
    <app-header [user]="currentUser"></app-header>
    <app-main [data]="appData"></app-main>
    <app-footer></app-footer>
  `,
})
export class AppComponent {
  currentUser = { name: "John Doe", id: 1 };
  appData = { items: [], loading: false };

  // When this method is called, change detection runs
  updateUser(newUser: User) {
    this.currentUser = newUser; // Triggers change detection
  }
}

// Each component has its own change detector
@Component({
  selector: "app-header",
  template: `
    <h1>Welcome {{ user.name }}!</h1>
    <span>User ID: {{ user.id }}</span>
  `,
  // OnPush strategy - only checks when inputs change
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class HeaderComponent {
  @Input() user!: User;
}
```

### **2. 🔄 Change Detection Cycle**

```typescript
// Understanding the change detection process
@Injectable()
export class ChangeDetectionExplainer {
  explainDetectionCycle() {
    /*
    Change Detection Cycle Steps:
    
    1. 🎯 Trigger Event
       - User interaction (click, input)
       - HTTP response
       - Timer (setTimeout, setInterval)
       - Promises and Observables
    
    2. 🌊 Zone.js Patches
       - Monkey patches browser APIs
       - Intercepts async operations
       - Triggers change detection automatically
    
    3. 🔍 Dirty Checking
       - Compare current values with previous values
       - Check all components from root to leaves
       - Update DOM if changes detected
    
    4. ✨ DOM Updates
       - Apply changes to the actual DOM
       - Update text nodes, attributes, styles
       - Add/remove elements as needed
    */
  }
}

// Example of change detection in action
@Component({
  selector: "app-counter",
  template: `
    <div>
      <h2>Counter: {{ count }}</h2>
      <p>Last Updated: {{ lastUpdated | date : "short" }}</p>
      <button (click)="increment()">+1</button>
      <button (click)="decrement()">-1</button>
      <button (click)="reset()">Reset</button>
    </div>

    <!-- This will show change detection runs -->
    <div class="debug-info">
      <p>Detection runs: {{ detectionCount }}</p>
      <p>Component checked: {{ componentChecked }}</p>
    </div>
  `,
})
export class CounterComponent implements DoCheck {
  count = 0;
  lastUpdated = new Date();
  detectionCount = 0;
  componentChecked = false;

  // This runs on every change detection cycle
  ngDoCheck() {
    this.detectionCount++;
    this.componentChecked = !this.componentChecked;
    console.log("🔍 Change detection ran for CounterComponent");
  }

  increment() {
    this.count++;
    this.lastUpdated = new Date();
    console.log("➕ Increment triggered change detection");
  }

  decrement() {
    this.count--;
    this.lastUpdated = new Date();
    console.log("➖ Decrement triggered change detection");
  }

  reset() {
    this.count = 0;
    this.lastUpdated = new Date();
    console.log("🔄 Reset triggered change detection");
  }
}
```

### **3. 🎯 Change Detection Triggers**

```typescript
// All the ways change detection can be triggered
@Component({
  selector: "app-trigger-examples",
  template: `
    <div class="triggers-demo">
      <h2>Change Detection Triggers</h2>

      <!-- 1. DOM Events -->
      <button (click)="onButtonClick()">Click Me (DOM Event)</button>

      <!-- 2. Input Changes -->
      <input (input)="onInputChange($event)" placeholder="Type something" />

      <!-- 3. Form Events -->
      <form (submit)="onSubmit($event)">
        <input type="text" [(ngModel)]="formValue" />
        <button type="submit">Submit</button>
      </form>

      <!-- 4. HTTP Responses -->
      <button (click)="loadData()">Load Data (HTTP)</button>

      <!-- 5. Timers -->
      <button (click)="startTimer()">Start Timer</button>

      <!-- 6. Promises -->
      <button (click)="triggerPromise()">Trigger Promise</button>

      <!-- 7. Observables -->
      <button (click)="triggerObservable()">Trigger Observable</button>

      <div class="results">
        <p>Current time: {{ currentTime | date : "medium" }}</p>
        <p>Form value: {{ formValue }}</p>
        <p>HTTP data: {{ httpData | json }}</p>
        <p>Timer count: {{ timerCount }}</p>
      </div>
    </div>
  `,
})
export class TriggerExamplesComponent implements OnInit, OnDestroy {
  currentTime = new Date();
  formValue = "";
  httpData: any = null;
  timerCount = 0;
  private subscription = new Subscription();

  constructor(
    private http: HttpClient,
    private cdr: ChangeDetectorRef,
    private zone: NgZone
  ) {}

  ngOnInit() {
    // Update time every second (triggers change detection)
    this.subscription.add(
      timer(0, 1000).subscribe(() => {
        this.currentTime = new Date();
        console.log("⏰ Timer triggered change detection");
      })
    );
  }

  ngOnDestroy() {
    this.subscription.unsubscribe();
  }

  // 1. DOM Event Trigger
  onButtonClick() {
    console.log("🖱️ Button click triggered change detection");
    this.currentTime = new Date();
  }

  // 2. Input Change Trigger
  onInputChange(event: Event) {
    const value = (event.target as HTMLInputElement).value;
    console.log("⌨️ Input change triggered change detection:", value);
  }

  // 3. Form Submit Trigger
  onSubmit(event: Event) {
    event.preventDefault();
    console.log("📝 Form submit triggered change detection");
    this.formValue = "";
  }

  // 4. HTTP Response Trigger
  loadData() {
    console.log("🌐 Starting HTTP request...");

    this.http
      .get("https://jsonplaceholder.typicode.com/users/1")
      .subscribe((data) => {
        this.httpData = data;
        console.log("📡 HTTP response triggered change detection");
      });
  }

  // 5. Timer Trigger
  startTimer() {
    console.log("⏲️ Starting timer...");

    setTimeout(() => {
      this.timerCount++;
      console.log("⏰ setTimeout triggered change detection");
    }, 1000);
  }

  // 6. Promise Trigger
  triggerPromise() {
    console.log("🤝 Creating promise...");

    Promise.resolve("Promise resolved!").then((result) => {
      this.httpData = { message: result };
      console.log("✅ Promise resolution triggered change detection");
    });
  }

  // 7. Observable Trigger
  triggerObservable() {
    console.log("🔄 Creating observable...");

    of("Observable emitted!")
      .pipe(delay(500))
      .subscribe((result) => {
        this.httpData = { message: result };
        console.log("📊 Observable emission triggered change detection");
      });
  }

  // Manual trigger (outside Zone.js)
  manualTrigger() {
    // This won't trigger change detection automatically
    this.zone.runOutsideAngular(() => {
      setTimeout(() => {
        this.timerCount++;
        console.log("🚫 Outside zone - no auto detection");

        // Manually trigger detection
        this.cdr.detectChanges();
        console.log("🔧 Manually triggered change detection");
      }, 1000);
    });
  }
}
```

## 🚀 **Change Detection Strategies**

### **1. 🔄 Default Strategy**

```typescript
// Default change detection checks every component
@Component({
  selector: "app-default-strategy",
  template: `
    <h3>Default Strategy Component</h3>
    <p>Name: {{ person.name }}</p>
    <p>Age: {{ person.age }}</p>
    <p>Checked: {{ checksCount }} times</p>

    <app-child [data]="childData" (dataChange)="onChildDataChange($event)">
    </app-child>
  `,
  // Default strategy - checks on every change detection cycle
  changeDetection: ChangeDetectionStrategy.Default,
})
export class DefaultStrategyComponent implements DoCheck {
  person = { name: "Alice", age: 25 };
  childData = { value: "initial" };
  checksCount = 0;

  ngDoCheck() {
    this.checksCount++;
    console.log(
      `🔍 Default strategy component checked ${this.checksCount} times`
    );
  }

  onChildDataChange(newData: any) {
    this.childData = newData;
    console.log("📥 Child data changed:", newData);
  }

  // Any method that changes data triggers change detection
  updatePerson() {
    this.person.age++; // This will be detected
    console.log("👤 Person updated");
  }

  // Even if we don't change anything, this still triggers a check
  doNothing() {
    console.log("🤷 Did nothing, but change detection still runs");
  }
}
```

### **2. ⚡ OnPush Strategy**

```typescript
// OnPush strategy - only checks when inputs change or events occur
@Component({
  selector: "app-onpush-strategy",
  template: `
    <h3>OnPush Strategy Component</h3>
    <p>Input data: {{ inputData | json }}</p>
    <p>Internal count: {{ internalCount }}</p>
    <p>Checks: {{ checksCount }}</p>

    <button (click)="incrementInternal()">Increment Internal</button>
    <button (click)="forceUpdate()">Force Update</button>

    <div class="children">
      <app-onpush-child
        *ngFor="let item of items"
        [item]="item"
        (itemChange)="onItemChange($event)"
      >
      </app-onpush-child>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OnPushStrategyComponent implements DoCheck, OnInit {
  @Input() inputData: any = {};

  internalCount = 0;
  checksCount = 0;
  items = [
    { id: 1, name: "Item 1", value: 10 },
    { id: 2, name: "Item 2", value: 20 },
  ];

  constructor(private cdr: ChangeDetectorRef) {}

  ngOnInit() {
    // Demonstrate that OnPush doesn't check automatically
    setInterval(() => {
      console.log("⏰ Timer tick - OnPush component NOT checked automatically");
    }, 2000);
  }

  ngDoCheck() {
    this.checksCount++;
    console.log(`⚡ OnPush component checked ${this.checksCount} times`);
  }

  // This won't trigger change detection because it's internal state
  incrementInternal() {
    this.internalCount++;
    console.log("📈 Internal count incremented, but UI might not update");

    // Need to manually trigger detection
    this.cdr.markForCheck();
  }

  // Events still trigger change detection in OnPush
  onItemChange(updatedItem: any) {
    const index = this.items.findIndex((item) => item.id === updatedItem.id);
    if (index >= 0) {
      // Create new array to ensure change detection
      this.items = [
        ...this.items.slice(0, index),
        updatedItem,
        ...this.items.slice(index + 1),
      ];
    }
    console.log("🔄 Item changed via event");
  }

  // Force change detection
  forceUpdate() {
    this.cdr.detectChanges();
    console.log("🔧 Forced change detection");
  }

  // Async operation that needs manual detection
  loadDataAsync() {
    // This won't automatically trigger change detection in OnPush
    setTimeout(() => {
      this.internalCount += 10;
      console.log("📡 Async data loaded");

      // Mark for check to trigger detection
      this.cdr.markForCheck();
    }, 1000);
  }
}

@Component({
  selector: "app-onpush-child",
  template: `
    <div class="child-item">
      <span>{{ item.name }}: {{ item.value }}</span>
      <button (click)="increment()">+1</button>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OnPushChildComponent {
  @Input() item!: any;
  @Output() itemChange = new EventEmitter<any>();

  increment() {
    const updatedItem = {
      ...this.item,
      value: this.item.value + 1,
    };
    this.itemChange.emit(updatedItem);
  }
}
```

### **3. 🎛️ Manual Change Detection Control**

```typescript
// Complete control over change detection
@Component({
  selector: "app-manual-control",
  template: `
    <h3>Manual Change Detection Control</h3>

    <div class="controls">
      <button (click)="enableAutoDetection()">Enable Auto Detection</button>
      <button (click)="disableAutoDetection()">Disable Auto Detection</button>
      <button (click)="detectChangesOnce()">Detect Changes Once</button>
      <button (click)="reattachDetector()">Reattach Detector</button>
    </div>

    <div class="status">
      <p>
        Auto Detection: {{ isAutoDetectionEnabled ? "Enabled" : "Disabled" }}
      </p>
      <p>Manual Updates: {{ manualUpdateCount }}</p>
      <p>Current Time: {{ currentTime | date : "medium" }}</p>
    </div>

    <div class="data">
      <h4>Reactive Data (with async pipe):</h4>
      <p>{{ reactiveData$ | async | json }}</p>

      <h4>Manual Data:</h4>
      <p>{{ manualData | json }}</p>
    </div>
  `,
})
export class ManualControlComponent implements OnInit, OnDestroy {
  isAutoDetectionEnabled = true;
  manualUpdateCount = 0;
  currentTime = new Date();
  manualData = { value: 0, lastUpdate: new Date() };

  reactiveData$ = timer(0, 1000).pipe(
    map((tick) => ({
      tick,
      timestamp: new Date(),
      message: `Reactive update #${tick}`,
    }))
  );

  private timeUpdateSubscription?: Subscription;

  constructor(private cdr: ChangeDetectorRef) {}

  ngOnInit() {
    // Start time updates
    this.startTimeUpdates();
  }

  ngOnDestroy() {
    this.timeUpdateSubscription?.unsubscribe();
  }

  private startTimeUpdates() {
    this.timeUpdateSubscription = timer(0, 1000).subscribe(() => {
      this.currentTime = new Date();
      this.manualData = {
        value: this.manualData.value + 1,
        lastUpdate: new Date(),
      };
    });
  }

  enableAutoDetection() {
    this.cdr.reattach();
    this.isAutoDetectionEnabled = true;
    console.log("✅ Auto detection enabled");
  }

  disableAutoDetection() {
    this.cdr.detach();
    this.isAutoDetectionEnabled = false;
    console.log("❌ Auto detection disabled");
  }

  detectChangesOnce() {
    this.manualUpdateCount++;
    this.cdr.detectChanges();
    console.log("🔧 Manual change detection triggered");
  }

  reattachDetector() {
    this.cdr.reattach();
    this.isAutoDetectionEnabled = true;
    console.log("🔗 Change detector reattached");
  }

  // This method won't update the UI if auto-detection is disabled
  updateManualData() {
    this.manualData = {
      value: Math.floor(Math.random() * 1000),
      lastUpdate: new Date(),
    };
    console.log("📝 Manual data updated (might not reflect in UI)");
  }
}
```

## 🔬 **Advanced Change Detection Patterns**

### **1. 🎯 Immutable Data Patterns**

```typescript
// Using immutable patterns for optimal OnPush performance
@Injectable()
export class ImmutableStateService {
  private _state = new BehaviorSubject({
    users: [],
    loading: false,
    error: null,
  });

  state$ = this._state.asObservable();

  // Always return new objects for OnPush components
  addUser(user: User) {
    const currentState = this._state.value;

    // Create new state object - triggers OnPush detection
    const newState = {
      ...currentState,
      users: [...currentState.users, user], // New array reference
    };

    this._state.next(newState);
  }

  updateUser(userId: string, updates: Partial<User>) {
    const currentState = this._state.value;

    const newState = {
      ...currentState,
      users: currentState.users.map((user) =>
        user.id === userId
          ? { ...user, ...updates } // New user object
          : user
      ),
    };

    this._state.next(newState);
  }

  removeUser(userId: string) {
    const currentState = this._state.value;

    const newState = {
      ...currentState,
      users: currentState.users.filter((user) => user.id !== userId),
    };

    this._state.next(newState);
  }
}

@Component({
  selector: "app-immutable-demo",
  template: `
    <h3>Immutable State Demo</h3>

    <div *ngIf="state$ | async as state">
      <div class="user-list">
        <app-user-card
          *ngFor="let user of state.users; trackBy: trackByUserId"
          [user]="user"
          (userUpdate)="onUserUpdate($event)"
          (userDelete)="onUserDelete($event)"
        >
        </app-user-card>
      </div>

      <button (click)="addRandomUser()">Add Random User</button>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ImmutableDemoComponent {
  state$ = this.stateService.state$;

  constructor(private stateService: ImmutableStateService) {}

  trackByUserId(index: number, user: User): string {
    return user.id;
  }

  onUserUpdate(event: { userId: string; updates: Partial<User> }) {
    this.stateService.updateUser(event.userId, event.updates);
  }

  onUserDelete(userId: string) {
    this.stateService.removeUser(userId);
  }

  addRandomUser() {
    const newUser: User = {
      id: Date.now().toString(),
      name: `User ${Math.floor(Math.random() * 1000)}`,
      email: `user${Math.floor(Math.random() * 1000)}@example.com`,
      active: true,
    };

    this.stateService.addUser(newUser);
  }
}
```

### **2. 🧩 Zoneless Change Detection (Angular 19+)**

```typescript
// Modern zoneless change detection with Signals
@Component({
  selector: "app-zoneless-demo",
  template: `
    <h3>Zoneless Change Detection</h3>

    <div class="signals-demo">
      <h4>Signals (Auto-tracked)</h4>
      <p>Count: {{ count() }}</p>
      <p>Double: {{ doubleCount() }}</p>
      <p>Message: {{ message() }}</p>

      <button (click)="increment()">Increment</button>
      <button (click)="updateMessage()">Update Message</button>
    </div>

    <div class="computed-demo">
      <h4>Computed Values</h4>
      <p>Total: {{ total() }}</p>
      <p>Average: {{ average() }}</p>

      <button (click)="addValue()">Add Random Value</button>
      <button (click)="clearValues()">Clear Values</button>
    </div>

    <div class="effect-demo">
      <h4>Side Effects</h4>
      <p>Log entries: {{ logEntries().length }}</p>
      <ul>
        <li *ngFor="let entry of logEntries()">{{ entry }}</li>
      </ul>
    </div>
  `,
})
export class ZonelessDemoComponent implements OnInit {
  // Signals automatically track dependencies
  count = signal(0);
  message = signal("Hello Signals!");
  values = signal<number[]>([]);
  logEntries = signal<string[]>([]);

  // Computed signals automatically update when dependencies change
  doubleCount = computed(() => this.count() * 2);
  total = computed(() => this.values().reduce((sum, val) => sum + val, 0));
  average = computed(() => {
    const vals = this.values();
    return vals.length > 0 ? this.total() / vals.length : 0;
  });

  constructor() {
    // Effects run automatically when signals change
    effect(() => {
      const currentCount = this.count();
      const currentMessage = this.message();

      console.log(
        `🔄 Effect triggered - Count: ${currentCount}, Message: ${currentMessage}`
      );

      // Update log entries (this creates a new signal update)
      const newEntry = `Count: ${currentCount} at ${new Date().toLocaleTimeString()}`;
      this.logEntries.update((entries) => [...entries, newEntry]);
    });
  }

  ngOnInit() {
    // Zoneless operations don't need Zone.js
    this.startZonelessTimer();
  }

  increment() {
    // Signal updates automatically trigger change detection
    this.count.update((val) => val + 1);
  }

  updateMessage() {
    const messages = [
      "Hello Angular!",
      "Zoneless is awesome!",
      "Signals rock!",
      "Change detection evolved!",
    ];

    const randomMessage = messages[Math.floor(Math.random() * messages.length)];
    this.message.set(randomMessage);
  }

  addValue() {
    const newValue = Math.floor(Math.random() * 100);
    this.values.update((vals) => [...vals, newValue]);
  }

  clearValues() {
    this.values.set([]);
    this.logEntries.set([]);
  }

  private startZonelessTimer() {
    // This doesn't trigger zone-based change detection
    setInterval(() => {
      console.log("⏰ Zoneless timer tick");
      // Only signal updates trigger UI updates
    }, 5000);
  }
}
```

## 🛠️ **Change Detection Debugging Tools**

### **1. 🔍 Change Detection Profiler**

```typescript
// Advanced debugging service for change detection
@Injectable({
  providedIn: "root",
})
export class ChangeDetectionProfiler {
  private detectionCycles = 0;
  private componentChecks = new Map<string, number>();
  private performanceMarks: PerformanceMark[] = [];

  startProfiling() {
    console.log("🔬 Change Detection Profiling Started");
    this.detectionCycles = 0;
    this.componentChecks.clear();
    this.performanceMarks = [];

    // Hook into NgZone to track cycles
    const zone = Zone.current;
    const originalRun = zone.run;

    zone.run = function (fn, applyThis, applyArgs) {
      performance.mark("cd-cycle-start");

      const result = originalRun.call(this, fn, applyThis, applyArgs);

      performance.mark("cd-cycle-end");
      performance.measure("cd-cycle", "cd-cycle-start", "cd-cycle-end");

      return result;
    };
  }

  recordComponentCheck(componentName: string) {
    const count = this.componentChecks.get(componentName) || 0;
    this.componentChecks.set(componentName, count + 1);
  }

  recordDetectionCycle() {
    this.detectionCycles++;
    console.log(`🔄 Change Detection Cycle #${this.detectionCycles}`);
  }

  getProfileReport(): ChangeDetectionReport {
    const measures = performance
      .getEntriesByType("measure")
      .filter((entry) => entry.name === "cd-cycle") as PerformanceMeasure[];

    return {
      totalCycles: this.detectionCycles,
      componentChecks: Object.fromEntries(this.componentChecks),
      averageCycleTime:
        measures.length > 0
          ? measures.reduce((sum, m) => sum + m.duration, 0) / measures.length
          : 0,
      slowestCycle:
        measures.length > 0 ? Math.max(...measures.map((m) => m.duration)) : 0,
      totalTime: measures.reduce((sum, m) => sum + m.duration, 0),
    };
  }

  logReport() {
    const report = this.getProfileReport();
    console.table({
      "Total Cycles": report.totalCycles,
      "Average Cycle Time (ms)": report.averageCycleTime.toFixed(2),
      "Slowest Cycle (ms)": report.slowestCycle.toFixed(2),
      "Total Time (ms)": report.totalTime.toFixed(2),
    });

    console.log("📊 Component Check Counts:", report.componentChecks);
  }
}

interface ChangeDetectionReport {
  totalCycles: number;
  componentChecks: Record<string, number>;
  averageCycleTime: number;
  slowestCycle: number;
  totalTime: number;
}
```

### **2. 🎯 Component Performance Monitor**

```typescript
// Decorator to monitor component performance
export function MonitorChangeDetection(componentName?: string) {
  return function (constructor: any) {
    const originalNgDoCheck = constructor.prototype.ngDoCheck;

    constructor.prototype.ngDoCheck = function () {
      const name = componentName || constructor.name;
      const start = performance.now();

      if (originalNgDoCheck) {
        originalNgDoCheck.call(this);
      }

      const duration = performance.now() - start;

      if (duration > 1) {
        // Log slow checks
        console.warn(
          `⚠️ Slow change detection in ${name}: ${duration.toFixed(2)}ms`
        );
      }

      // Record check
      const profiler = inject(ChangeDetectionProfiler);
      profiler.recordComponentCheck(name);
    };
  };
}

// Usage
@MonitorChangeDetection("UserListComponent")
@Component({
  selector: "app-user-list",
  template: `...`,
})
export class UserListComponent implements DoCheck {
  ngDoCheck() {
    // Original logic here
  }
}
```

## 📊 **Angular Version Comparison**

| Feature             | Angular 15    | Angular 17   | Angular 19    |
| ------------------- | ------------- | ------------ | ------------- |
| **Zone.js**         | Required      | Required     | Optional      |
| **Signals**         | Not available | Stable       | Enhanced      |
| **Zoneless CD**     | Not available | Experimental | Stable        |
| **OnPush Strategy** | Available     | Available    | Available     |
| **Manual Control**  | Full support  | Full support | Enhanced APIs |

## 🎯 **Key Takeaways**

### **🔄 Change Detection Fundamentals:**

1. **Zone.js Integration** - Automatically patches async operations
2. **Tree-based Checking** - Flows from root to leaf components
3. **Dirty Checking** - Compares current vs previous values
4. **Strategy Options** - Default vs OnPush optimization

### **⚡ Performance Optimization:**

1. **Use OnPush Strategy** for components with infrequent updates
2. **Implement Immutable Patterns** for predictable change tracking
3. **Leverage Signals** (Angular 17+) for automatic dependency tracking
4. **Consider Zoneless** (Angular 19+) for modern applications

### **🛠️ Best Practices:**

1. **Profile Change Detection** to identify bottlenecks
2. **Use TrackBy Functions** for ngFor optimization
3. **Minimize DoCheck Logic** to avoid performance issues
4. **Prefer Reactive Patterns** over manual change detection

### **🚨 Common Pitfalls:**

- Mutating objects directly (breaks OnPush)
- Heavy computations in templates
- Missing trackBy in large lists
- Overusing manual change detection
- Not understanding async pipe behavior

Understanding change detection is crucial for building performant Angular applications! 🚀
