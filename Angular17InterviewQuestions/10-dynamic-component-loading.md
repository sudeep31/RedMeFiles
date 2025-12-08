# 🔧 Dynamic Component Loading from String Names in Angular

## 🎯 **Question Overview**

_"How can we load component dynamically from a string name?"_

## 🔍 **Understanding Dynamic Component Loading**

Dynamic component loading allows you to **create and render components at runtime** based on string identifiers, user selections, or data from APIs. This is incredibly powerful for building flexible, plugin-based, or data-driven UIs.

Think of it like a **factory** that can create any component you need, just by giving it the component's name! 🏭

## 🛠️ **Modern Approach: Standalone Components (Angular 14+)**

### **1. 🚀 Component Registry Pattern**

The cleanest approach is to create a component registry:

```typescript
// component-registry.service.ts
import { Injectable, Type } from "@angular/core";

// Import all dynamic components
import { UserProfileComponent } from "./components/user-profile.component";
import { DashboardWidgetComponent } from "./components/dashboard-widget.component";
import { ChartComponent } from "./components/chart.component";
import { TableComponent } from "./components/table.component";
import { FormComponent } from "./components/form.component";

@Injectable({
  providedIn: "root",
})
export class ComponentRegistryService {
  private componentMap = new Map<string, Type<any>>([
    ["user-profile", UserProfileComponent],
    ["dashboard-widget", DashboardWidgetComponent],
    ["chart", ChartComponent],
    ["table", TableComponent],
    ["form", FormComponent],
  ]);

  getComponent(name: string): Type<any> | null {
    return this.componentMap.get(name.toLowerCase()) || null;
  }

  getAvailableComponents(): string[] {
    return Array.from(this.componentMap.keys());
  }

  registerComponent(name: string, component: Type<any>): void {
    this.componentMap.set(name.toLowerCase(), component);
  }

  hasComponent(name: string): boolean {
    return this.componentMap.has(name.toLowerCase());
  }
}
```

### **2. 🎯 Dynamic Component Loader Service**

```typescript
// dynamic-component-loader.service.ts
import {
  Injectable,
  ViewContainerRef,
  ComponentRef,
  EnvironmentInjector,
  createComponent,
  Type,
} from "@angular/core";
import { ComponentRegistryService } from "./component-registry.service";

export interface DynamicComponentData {
  componentName: string;
  inputs?: { [key: string]: any };
  outputs?: { [key: string]: (event: any) => void };
}

@Injectable({
  providedIn: "root",
})
export class DynamicComponentLoaderService {
  constructor(
    private componentRegistry: ComponentRegistryService,
    private injector: EnvironmentInjector
  ) {}

  loadComponent(
    viewContainer: ViewContainerRef,
    componentData: DynamicComponentData
  ): ComponentRef<any> | null {
    const componentType = this.componentRegistry.getComponent(
      componentData.componentName
    );

    if (!componentType) {
      console.error(
        `Component '${componentData.componentName}' not found in registry`
      );
      return null;
    }

    // Clear previous component
    viewContainer.clear();

    // Create component dynamically
    const componentRef = createComponent(componentType, {
      environmentInjector: this.injector,
      hostElement: viewContainer.element.nativeElement,
    });

    // Set inputs
    if (componentData.inputs) {
      Object.entries(componentData.inputs).forEach(([key, value]) => {
        if (componentRef.instance.hasOwnProperty(key)) {
          componentRef.instance[key] = value;
        }
      });
    }

    // Subscribe to outputs
    if (componentData.outputs) {
      Object.entries(componentData.outputs).forEach(([key, handler]) => {
        if (
          componentRef.instance[key] &&
          typeof componentRef.instance[key].subscribe === "function"
        ) {
          componentRef.instance[key].subscribe(handler);
        }
      });
    }

    // Attach to view
    viewContainer.insert(componentRef.hostView);

    // Trigger change detection
    componentRef.changeDetectorRef.detectChanges();

    return componentRef;
  }

  loadMultipleComponents(
    viewContainer: ViewContainerRef,
    components: DynamicComponentData[]
  ): ComponentRef<any>[] {
    viewContainer.clear();

    return components
      .map((componentData) => this.loadComponent(viewContainer, componentData))
      .filter((ref) => ref !== null) as ComponentRef<any>[];
  }
}
```

### **3. 🎨 Dynamic Component Host**

```typescript
// dynamic-component-host.component.ts
@Component({
  selector: "app-dynamic-host",
  template: `
    <div class="dynamic-host">
      <div class="controls">
        <select
          [(ngModel)]="selectedComponent"
          (change)="loadSelectedComponent()"
        >
          <option value="">Select a component...</option>
          <option *ngFor="let comp of availableComponents" [value]="comp">
            {{ comp | titlecase }}
          </option>
        </select>

        <button (click)="clearComponent()">Clear</button>
        <button (click)="loadRandomComponent()">Random</button>
      </div>

      <div class="component-inputs" *ngIf="selectedComponent">
        <h4>Component Inputs:</h4>
        <div *ngFor="let input of getComponentInputs(selectedComponent)">
          <label>{{ input.name }}:</label>
          <input
            [(ngModel)]="input.value"
            [type]="input.type"
            (input)="updateComponentInput(input.name, input.value)"
          />
        </div>
      </div>

      <!-- Dynamic component container -->
      <div class="component-container" #dynamicContainer>
        <!-- Components will be loaded here -->
      </div>

      <div class="component-info" *ngIf="currentComponentRef">
        <h4>Loaded Component Info:</h4>
        <p>Type: {{ currentComponentRef.componentType.name }}</p>
        <p>Instance: {{ currentComponentRef.instance.constructor.name }}</p>
        <button (click)="destroyComponent()">Destroy Component</button>
      </div>
    </div>
  `,
  styles: [
    `
      .dynamic-host {
        padding: 20px;
        border: 1px solid #ddd;
        border-radius: 8px;
      }

      .controls {
        display: flex;
        gap: 10px;
        margin-bottom: 20px;
        align-items: center;
      }

      .controls select {
        flex: 1;
        padding: 8px;
      }

      .controls button {
        padding: 8px 16px;
        background: #007acc;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      }

      .component-inputs {
        margin-bottom: 20px;
        padding: 15px;
        background: #f5f5f5;
        border-radius: 4px;
      }

      .component-inputs div {
        display: flex;
        align-items: center;
        gap: 10px;
        margin-bottom: 10px;
      }

      .component-inputs label {
        min-width: 100px;
        font-weight: bold;
      }

      .component-inputs input {
        flex: 1;
        padding: 4px 8px;
      }

      .component-container {
        min-height: 200px;
        border: 2px dashed #ddd;
        border-radius: 4px;
        padding: 20px;
        margin: 20px 0;
      }

      .component-info {
        margin-top: 20px;
        padding: 15px;
        background: #e8f5e8;
        border-radius: 4px;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule, FormsModule],
})
export class DynamicComponentHostComponent implements OnInit, OnDestroy {
  @ViewChild("dynamicContainer", { read: ViewContainerRef })
  dynamicContainer!: ViewContainerRef;

  selectedComponent = "";
  availableComponents: string[] = [];
  currentComponentRef: ComponentRef<any> | null = null;

  // Component input definitions for demo
  private componentInputs = new Map([
    [
      "user-profile",
      [
        { name: "userId", type: "text", value: "123" },
        { name: "showAvatar", type: "checkbox", value: true },
      ],
    ],
    [
      "chart",
      [
        { name: "chartType", type: "text", value: "bar" },
        { name: "data", type: "text", value: "[1,2,3,4,5]" },
        { name: "width", type: "number", value: 400 },
        { name: "height", type: "number", value: 300 },
      ],
    ],
    [
      "table",
      [
        { name: "columns", type: "text", value: "name,age,email" },
        { name: "sortable", type: "checkbox", value: true },
      ],
    ],
  ]);

  constructor(
    private componentLoader: DynamicComponentLoaderService,
    private componentRegistry: ComponentRegistryService
  ) {}

  ngOnInit() {
    this.availableComponents = this.componentRegistry.getAvailableComponents();
  }

  ngOnDestroy() {
    this.destroyComponent();
  }

  loadSelectedComponent() {
    if (!this.selectedComponent) return;

    this.loadComponent(this.selectedComponent);
  }

  loadComponent(componentName: string) {
    const inputs = this.getComponentInputsAsObject(componentName);
    const outputs = {
      click: (event: any) => console.log("Component clicked:", event),
      change: (event: any) => console.log("Component changed:", event),
    };

    this.currentComponentRef = this.componentLoader.loadComponent(
      this.dynamicContainer,
      {
        componentName,
        inputs,
        outputs,
      }
    );
  }

  loadRandomComponent() {
    const randomComponent =
      this.availableComponents[
        Math.floor(Math.random() * this.availableComponents.length)
      ];
    this.selectedComponent = randomComponent;
    this.loadSelectedComponent();
  }

  clearComponent() {
    this.selectedComponent = "";
    this.destroyComponent();
  }

  destroyComponent() {
    if (this.currentComponentRef) {
      this.currentComponentRef.destroy();
      this.currentComponentRef = null;
    }
    this.dynamicContainer?.clear();
  }

  getComponentInputs(componentName: string) {
    return this.componentInputs.get(componentName) || [];
  }

  getComponentInputsAsObject(componentName: string): { [key: string]: any } {
    const inputs = this.getComponentInputs(componentName);
    const result: { [key: string]: any } = {};

    inputs.forEach((input) => {
      let value = input.value;
      if (input.type === "number") value = Number(value);
      if (input.type === "checkbox") value = Boolean(value);
      if (input.name === "data" && typeof value === "string") {
        try {
          value = JSON.parse(value);
        } catch {}
      }
      result[input.name] = value;
    });

    return result;
  }

  updateComponentInput(inputName: string, value: any) {
    if (
      this.currentComponentRef &&
      this.currentComponentRef.instance.hasOwnProperty(inputName)
    ) {
      this.currentComponentRef.instance[inputName] = value;
      this.currentComponentRef.changeDetectorRef.detectChanges();
    }
  }
}
```

## 🔄 **Legacy NgModule Approach**

For applications still using NgModules:

```typescript
// module-based-loader.service.ts
import {
  Injectable,
  ComponentFactory,
  ComponentFactoryResolver,
  ViewContainerRef,
  ComponentRef,
  Type,
} from "@angular/core";

@Injectable({
  providedIn: "root",
})
export class ModuleBasedLoaderService {
  constructor(private componentFactoryResolver: ComponentFactoryResolver) {}

  loadComponent(
    viewContainer: ViewContainerRef,
    componentType: Type<any>,
    inputs?: { [key: string]: any }
  ): ComponentRef<any> {
    // Create component factory
    const componentFactory: ComponentFactory<any> =
      this.componentFactoryResolver.resolveComponentFactory(componentType);

    // Clear container
    viewContainer.clear();

    // Create component
    const componentRef: ComponentRef<any> =
      viewContainer.createComponent(componentFactory);

    // Set inputs
    if (inputs) {
      Object.entries(inputs).forEach(([key, value]) => {
        if (componentRef.instance.hasOwnProperty(key)) {
          componentRef.instance[key] = value;
        }
      });
    }

    return componentRef;
  }
}

// Component must be declared in entryComponents (Angular < 9) or module
@NgModule({
  declarations: [
    UserProfileComponent,
    DashboardWidgetComponent,
    // ... other components
  ],
  entryComponents: [
    // No longer needed in Angular 9+ with Ivy
    UserProfileComponent,
    DashboardWidgetComponent,
    // ... other components
  ],
})
export class DynamicComponentsModule {}
```

## 🔀 **Advanced Patterns**

### **1. 📦 Lazy Loading Dynamic Components**

```typescript
// lazy-component-loader.service.ts
@Injectable({
  providedIn: "root",
})
export class LazyComponentLoaderService {
  private componentCache = new Map<string, Type<any>>();

  constructor(private injector: EnvironmentInjector) {}

  async loadComponentLazily(
    componentName: string,
    viewContainer: ViewContainerRef
  ): Promise<ComponentRef<any> | null> {
    // Check cache first
    let componentType = this.componentCache.get(componentName);

    if (!componentType) {
      componentType = await this.importComponent(componentName);
      if (componentType) {
        this.componentCache.set(componentName, componentType);
      }
    }

    if (!componentType) {
      console.error(`Failed to load component: ${componentName}`);
      return null;
    }

    return this.createComponent(componentType, viewContainer);
  }

  private async importComponent(
    componentName: string
  ): Promise<Type<any> | null> {
    try {
      switch (componentName.toLowerCase()) {
        case "user-profile":
          const userModule = await import(
            "./components/user-profile/user-profile.component"
          );
          return userModule.UserProfileComponent;

        case "dashboard-widget":
          const dashboardModule = await import(
            "./components/dashboard-widget/dashboard-widget.component"
          );
          return dashboardModule.DashboardWidgetComponent;

        case "chart":
          const chartModule = await import(
            "./components/chart/chart.component"
          );
          return chartModule.ChartComponent;

        case "data-grid":
          const gridModule = await import(
            "./components/data-grid/data-grid.component"
          );
          return gridModule.DataGridComponent;

        default:
          // Try dynamic import based on naming convention
          const dynamicModule = await import(
            `./components/${componentName}/${componentName}.component`
          );
          const componentClassName =
            this.toPascalCase(componentName) + "Component";
          return dynamicModule[componentClassName];
      }
    } catch (error) {
      console.error(`Failed to import component ${componentName}:`, error);
      return null;
    }
  }

  private createComponent(
    componentType: Type<any>,
    viewContainer: ViewContainerRef
  ): ComponentRef<any> {
    viewContainer.clear();

    const componentRef = createComponent(componentType, {
      environmentInjector: this.injector,
    });

    viewContainer.insert(componentRef.hostView);
    return componentRef;
  }

  private toPascalCase(str: string): string {
    return str
      .split("-")
      .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
      .join("");
  }
}
```

### **2. 🔧 Configuration-Driven Component Loading**

```typescript
// config-driven-loader.service.ts
export interface ComponentConfig {
  name: string;
  component: string;
  inputs?: { [key: string]: any };
  outputs?: { [key: string]: string };
  lazy?: boolean;
  permissions?: string[];
}

@Injectable({
  providedIn: "root",
})
export class ConfigDrivenLoaderService {
  constructor(
    private componentLoader: DynamicComponentLoaderService,
    private lazyLoader: LazyComponentLoaderService,
    private authService: AuthService
  ) {}

  async loadComponentFromConfig(
    config: ComponentConfig,
    viewContainer: ViewContainerRef
  ): Promise<ComponentRef<any> | null> {
    // Check permissions
    if (config.permissions && !this.hasPermissions(config.permissions)) {
      console.warn(`Insufficient permissions for component: ${config.name}`);
      return null;
    }

    // Load component based on configuration
    if (config.lazy) {
      return this.lazyLoader.loadComponentLazily(
        config.component,
        viewContainer
      );
    } else {
      return this.componentLoader.loadComponent(viewContainer, {
        componentName: config.component,
        inputs: config.inputs,
        outputs: this.createOutputHandlers(config.outputs),
      });
    }
  }

  async loadDashboardFromConfig(
    dashboardConfig: ComponentConfig[],
    viewContainer: ViewContainerRef
  ): Promise<ComponentRef<any>[]> {
    const components: ComponentRef<any>[] = [];

    for (const config of dashboardConfig) {
      const componentRef = await this.loadComponentFromConfig(
        config,
        viewContainer
      );
      if (componentRef) {
        components.push(componentRef);
      }
    }

    return components;
  }

  private hasPermissions(requiredPermissions: string[]): boolean {
    const userPermissions = this.authService.getUserPermissions();
    return requiredPermissions.every((permission) =>
      userPermissions.includes(permission)
    );
  }

  private createOutputHandlers(outputs?: { [key: string]: string }): {
    [key: string]: (event: any) => void;
  } {
    const handlers: { [key: string]: (event: any) => void } = {};

    if (outputs) {
      Object.entries(outputs).forEach(([eventName, handlerName]) => {
        handlers[eventName] = (event: any) => {
          // You could implement a handler registry here
          console.log(`Event ${eventName} handled by ${handlerName}:`, event);
        };
      });
    }

    return handlers;
  }
}
```

### **3. 📊 Data-Driven Dynamic Dashboard**

```typescript
// dashboard.component.ts
@Component({
  selector: "app-dynamic-dashboard",
  template: `
    <div class="dashboard">
      <div class="dashboard-header">
        <h1>Dynamic Dashboard</h1>
        <button (click)="refreshDashboard()">Refresh</button>
        <button (click)="editMode = !editMode">
          {{ editMode ? "Save" : "Edit" }}
        </button>
      </div>

      <div class="dashboard-grid" [class.edit-mode]="editMode">
        <div
          *ngFor="let widget of widgets; track: widget.id"
          class="widget-container"
          [style.grid-area]="widget.gridArea"
        >
          <div class="widget-header" *ngIf="editMode">
            <span>{{ widget.title }}</span>
            <button (click)="removeWidget(widget.id)">×</button>
          </div>

          <div #widgetContainer></div>
        </div>
      </div>

      <div class="add-widget" *ngIf="editMode">
        <select [(ngModel)]="newWidgetType">
          <option value="">Select widget type...</option>
          <option *ngFor="let type of availableWidgetTypes" [value]="type">
            {{ type | titlecase }}
          </option>
        </select>
        <button (click)="addWidget()" [disabled]="!newWidgetType">
          Add Widget
        </button>
      </div>
    </div>
  `,
  styles: [
    `
      .dashboard {
        padding: 20px;
      }

      .dashboard-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20px;
      }

      .dashboard-grid {
        display: grid;
        grid-template-columns: repeat(12, 1fr);
        grid-auto-rows: 200px;
        gap: 20px;
        min-height: 400px;
      }

      .widget-container {
        background: white;
        border: 1px solid #ddd;
        border-radius: 8px;
        padding: 10px;
        position: relative;
      }

      .edit-mode .widget-container {
        border: 2px dashed #007acc;
      }

      .widget-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 10px;
        padding-bottom: 10px;
        border-bottom: 1px solid #eee;
      }

      .add-widget {
        margin-top: 20px;
        display: flex;
        gap: 10px;
        align-items: center;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule, FormsModule],
})
export class DynamicDashboardComponent implements OnInit, OnDestroy {
  @ViewChildren("widgetContainer", { read: ViewContainerRef })
  widgetContainers!: QueryList<ViewContainerRef>;

  widgets: DashboardWidget[] = [];
  availableWidgetTypes = ["chart", "table", "stats", "weather", "news"];
  newWidgetType = "";
  editMode = false;

  private widgetComponents = new Map<string, ComponentRef<any>>();

  constructor(
    private configLoader: ConfigDrivenLoaderService,
    private dashboardService: DashboardService
  ) {}

  async ngOnInit() {
    this.widgets = await this.dashboardService.getUserDashboardConfig();
    setTimeout(() => this.loadAllWidgets(), 0);
  }

  ngOnDestroy() {
    this.destroyAllWidgets();
  }

  async loadAllWidgets() {
    const containers = this.widgetContainers.toArray();

    for (let i = 0; i < this.widgets.length; i++) {
      const widget = this.widgets[i];
      const container = containers[i];

      if (container) {
        await this.loadWidget(widget, container);
      }
    }
  }

  private async loadWidget(
    widget: DashboardWidget,
    container: ViewContainerRef
  ) {
    const config: ComponentConfig = {
      name: widget.title,
      component: widget.type,
      inputs: widget.config,
      lazy: true,
    };

    const componentRef = await this.configLoader.loadComponentFromConfig(
      config,
      container
    );

    if (componentRef) {
      this.widgetComponents.set(widget.id, componentRef);
    }
  }

  async addWidget() {
    if (!this.newWidgetType) return;

    const newWidget: DashboardWidget = {
      id: Date.now().toString(),
      type: this.newWidgetType,
      title:
        this.newWidgetType.charAt(0).toUpperCase() +
        this.newWidgetType.slice(1),
      gridArea: `auto / auto / span 1 / span 3`,
      config: {},
    };

    this.widgets.push(newWidget);
    this.newWidgetType = "";

    // Wait for view to update, then load the new widget
    setTimeout(() => {
      const containers = this.widgetContainers.toArray();
      const lastContainer = containers[containers.length - 1];
      if (lastContainer) {
        this.loadWidget(newWidget, lastContainer);
      }
    }, 0);
  }

  removeWidget(widgetId: string) {
    // Destroy component
    const componentRef = this.widgetComponents.get(widgetId);
    if (componentRef) {
      componentRef.destroy();
      this.widgetComponents.delete(widgetId);
    }

    // Remove from widgets array
    this.widgets = this.widgets.filter((w) => w.id !== widgetId);
  }

  async refreshDashboard() {
    this.destroyAllWidgets();
    this.widgets = await this.dashboardService.getUserDashboardConfig();
    setTimeout(() => this.loadAllWidgets(), 0);
  }

  private destroyAllWidgets() {
    this.widgetComponents.forEach((componentRef) => {
      componentRef.destroy();
    });
    this.widgetComponents.clear();
  }
}

interface DashboardWidget {
  id: string;
  type: string;
  title: string;
  gridArea: string;
  config: { [key: string]: any };
}
```

## 🚨 **Common Pitfalls & Best Practices**

### **❌ Common Mistakes**

1. **Memory leaks from not destroying components**

```typescript
// ❌ Bad - Not cleaning up components
loadComponent(name: string) {
  const component = this.createComponent(name);
  // Component created but never destroyed!
}
```

2. **Not handling component not found**

```typescript
// ❌ Bad - No error handling
loadComponent(name: string) {
  const componentType = this.registry.get(name);
  return this.createComponent(componentType); // Might be undefined!
}
```

### **✅ Best Practices**

1. **Proper component lifecycle management**

```typescript
// ✅ Good - Proper cleanup
@Component({})
export class ComponentHost implements OnDestroy {
  private loadedComponents = new Set<ComponentRef<any>>();

  loadComponent(name: string) {
    const ref = this.componentLoader.load(name);
    if (ref) {
      this.loadedComponents.add(ref);
    }
    return ref;
  }

  ngOnDestroy() {
    this.loadedComponents.forEach((ref) => ref.destroy());
    this.loadedComponents.clear();
  }
}
```

2. **Type safety with interfaces**

```typescript
// ✅ Good - Type-safe dynamic loading
interface DynamicComponent {
  title: string;
  initialize?(data: any): void;
  destroy?(): void;
}

export class TypeSafeLoader {
  loadComponent<T extends DynamicComponent>(
    name: string
  ): ComponentRef<T> | null {
    // Implementation with proper typing
  }
}
```

## 📊 **Angular Version Comparison**

| Feature                | Angular 15                         | Angular 17-19                  |
| ---------------------- | ---------------------------------- | ------------------------------ |
| **Component Creation** | ViewContainerRef.createComponent() | createComponent() function     |
| **Standalone Support** | Limited                            | Full support with better APIs  |
| **Bundle Splitting**   | Manual lazy loading                | Enhanced with standalone       |
| **Type Safety**        | Good with careful typing           | Improved with better inference |
| **Performance**        | ComponentFactory overhead          | Direct component creation      |

## 🎯 **Key Takeaways**

### **When to Use Dynamic Component Loading:**

1. **🔌 Plugin architectures** where components are loaded at runtime
2. **📊 Dynamic dashboards** with user-configurable widgets
3. **🎨 CMS systems** where content types are dynamic
4. **📱 Micro-frontend** architectures
5. **⚡ A/B testing** with different component variations

### **Best Practices:**

1. **Create a component registry** for organized component management
2. **Handle errors gracefully** when components don't exist
3. **Manage component lifecycle** properly to avoid memory leaks
4. **Use lazy loading** for better performance
5. **Implement proper typing** for type safety
6. **Consider security implications** when loading components dynamically

### **Performance Considerations:**

- Use **lazy loading** for large components
- Implement **component caching** to avoid repeated imports
- **Destroy components** when no longer needed
- Consider **bundle splitting** strategies
- Monitor **memory usage** in dynamic scenarios

Dynamic component loading is powerful but should be used thoughtfully with proper architecture and cleanup! 🚀
