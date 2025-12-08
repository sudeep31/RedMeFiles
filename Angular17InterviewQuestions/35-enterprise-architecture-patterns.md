# 🏗️ **Enterprise Architecture Patterns in Angular**

## 🎯 **What You'll Learn**

This guide covers enterprise-grade architectural patterns for building scalable, maintainable Angular applications that can handle complex business requirements, team collaboration, and long-term evolution.

---

## 📚 **The Basics: Enterprise Architecture Fundamentals**

### **🏢 What Makes an Enterprise Application?**

Enterprise applications are characterized by:

- **Complex business domains** with multiple bounded contexts
- **Multiple teams** working on different features simultaneously
- **High scalability requirements** (users, data, features)
- **Long-term maintainability** (5+ year lifespans)
- **Integration needs** with legacy systems and external services
- **Strict security and compliance** requirements

---

## 🧩 **Micro-Frontend Architecture Pattern**

### **🚀 Advanced Micro-Frontend Implementation**

**Problem:** You need to scale development across multiple teams while maintaining independence and deployment flexibility.

```typescript
// src/app/core/micro-frontend/module-federation.config.ts
import { ModuleWithProviders, NgModule, Type } from "@angular/core";
import { Router } from "@angular/router";
import { Observable, BehaviorSubject, fromEvent, merge } from "rxjs";
import { filter, map, shareReplay, distinctUntilChanged } from "rxjs/operators";

export interface MicroFrontendConfig {
  name: string;
  remoteUrl: string;
  exposedModule: string;
  routePath: string;
  preload?: boolean;
  fallbackComponent?: Type<any>;
  healthCheckUrl?: string;
  version?: string;
  dependencies?: string[];
  permissions?: string[];
}

export interface MicroFrontendState {
  id: string;
  status: "loading" | "loaded" | "error" | "offline";
  config: MicroFrontendConfig;
  error?: string;
  loadTime?: number;
  lastHealthCheck?: Date;
}

// 🏭 Micro-Frontend Registry Service
@Injectable({
  providedIn: "root",
})
export class MicroFrontendRegistry {
  private microFrontends = new Map<string, MicroFrontendState>();
  private microFrontendStates$ = new BehaviorSubject<
    Map<string, MicroFrontendState>
  >(new Map());

  private loadedModules = new Map<string, any>();
  private healthCheckInterval = 30000; // 30 seconds
  private healthCheckTimers = new Map<string, any>();

  constructor(
    private router: Router,
    private injector: Injector,
    @Inject("MICRO_FRONTEND_CONFIG") private globalConfig: any
  ) {
    this.initializeGlobalErrorHandling();
    this.initializeCommunicationBus();
  }

  // 📝 REGISTER MICRO-FRONTEND
  registerMicroFrontend(config: MicroFrontendConfig): void {
    const state: MicroFrontendState = {
      id: config.name,
      status: "loading",
      config,
    };

    this.microFrontends.set(config.name, state);
    this.microFrontendStates$.next(new Map(this.microFrontends));

    // Setup health check if URL provided
    if (config.healthCheckUrl) {
      this.setupHealthCheck(config.name, config.healthCheckUrl);
    }

    // Preload if configured
    if (config.preload) {
      this.loadMicroFrontend(config.name);
    }

    console.log(`📝 Registered micro-frontend: ${config.name}`);
  }

  // 🚀 LOAD MICRO-FRONTEND
  async loadMicroFrontend(name: string): Promise<any> {
    const state = this.microFrontends.get(name);
    if (!state) {
      throw new Error(`Micro-frontend ${name} not registered`);
    }

    // Return cached module if already loaded
    if (this.loadedModules.has(name)) {
      return this.loadedModules.get(name);
    }

    const startTime = performance.now();

    try {
      state.status = "loading";
      this.updateState(state);

      // Dynamic import with error handling
      const moduleFactory = await this.dynamicImport(state.config);

      // Validate module dependencies
      await this.validateDependencies(state.config);

      // Initialize module
      const module = await this.initializeModule(moduleFactory, state.config);

      // Cache loaded module
      this.loadedModules.set(name, module);

      // Update state
      state.status = "loaded";
      state.loadTime = performance.now() - startTime;
      this.updateState(state);

      console.log(`🚀 Loaded micro-frontend: ${name} in ${state.loadTime}ms`);
      return module;
    } catch (error) {
      state.status = "error";
      state.error = error.message;
      this.updateState(state);

      console.error(`❌ Failed to load micro-frontend ${name}:`, error);

      // Try to load fallback component
      return this.loadFallbackComponent(state.config);
    }
  }

  // 🧭 SETUP DYNAMIC ROUTING
  setupDynamicRouting(): void {
    this.microFrontendStates$.subscribe((microFrontends) => {
      const routes = Array.from(microFrontends.values())
        .filter((state) => state.status === "loaded")
        .map((state) => ({
          path: state.config.routePath,
          loadChildren: () => this.loadedModules.get(state.id),
          data: {
            microFrontend: state.id,
            permissions: state.config.permissions || [],
          },
        }));

      // Dynamically update router configuration
      this.router.resetConfig([
        ...this.router.config.filter((route) => !route.data?.microFrontend),
        ...routes,
      ]);
    });
  }

  // 🔄 Private Implementation Methods

  private async dynamicImport(config: MicroFrontendConfig): Promise<any> {
    // Support for different loading strategies
    if (config.remoteUrl.startsWith("webpack://")) {
      // Webpack Module Federation
      return this.loadWebpackFederation(config);
    } else if (config.remoteUrl.startsWith("systemjs://")) {
      // SystemJS loading
      return this.loadSystemJS(config);
    } else {
      // Standard ES module import
      return this.loadESModule(config);
    }
  }

  private async loadWebpackFederation(
    config: MicroFrontendConfig
  ): Promise<any> {
    // Load webpack container
    const containerUrl = config.remoteUrl.replace("webpack://", "");

    // Dynamic script injection
    await this.loadScript(`${containerUrl}/remoteEntry.js`);

    // Get container
    const container = (window as any)[config.name];

    if (!container) {
      throw new Error(`Container ${config.name} not found`);
    }

    // Initialize container
    await container.init(__webpack_share_scopes__.default);

    // Get factory
    const factory = await container.get(config.exposedModule);

    return factory();
  }

  private async loadESModule(config: MicroFrontendConfig): Promise<any> {
    const moduleUrl = `${config.remoteUrl}/${config.exposedModule}`;

    // Use dynamic import
    const module = await import(/* webpackIgnore: true */ moduleUrl);

    return module.default || module;
  }

  private async loadSystemJS(config: MicroFrontendConfig): Promise<any> {
    // Load SystemJS if not available
    if (!(window as any).System) {
      await this.loadScript("https://unpkg.com/systemjs/dist/system.js");
    }

    const moduleUrl = config.remoteUrl.replace("systemjs://", "");
    return (window as any).System.import(moduleUrl);
  }

  private loadScript(src: string): Promise<void> {
    return new Promise((resolve, reject) => {
      const script = document.createElement("script");
      script.src = src;
      script.onload = () => resolve();
      script.onerror = () => reject(new Error(`Failed to load script: ${src}`));
      document.head.appendChild(script);
    });
  }

  private async validateDependencies(
    config: MicroFrontendConfig
  ): Promise<void> {
    if (!config.dependencies || config.dependencies.length === 0) {
      return;
    }

    const missingDependencies = [];

    for (const dependency of config.dependencies) {
      const dependentState = this.microFrontends.get(dependency);

      if (!dependentState || dependentState.status !== "loaded") {
        missingDependencies.push(dependency);
      }
    }

    if (missingDependencies.length > 0) {
      throw new Error(
        `Missing dependencies: ${missingDependencies.join(", ")}`
      );
    }
  }

  private async initializeModule(
    moduleFactory: any,
    config: MicroFrontendConfig
  ): Promise<any> {
    // Create module instance
    const moduleRef = moduleFactory();

    // Apply any initialization hooks
    if (moduleRef.init && typeof moduleRef.init === "function") {
      await moduleRef.init({
        injector: this.injector,
        config: this.globalConfig,
        communicationBus: this.communicationBus,
      });
    }

    return moduleRef;
  }

  private loadFallbackComponent(config: MicroFrontendConfig): any {
    if (config.fallbackComponent) {
      return {
        default: config.fallbackComponent,
      };
    }

    // Return default error component
    return {
      default: class DefaultErrorComponent {
        template = `
          <div class="micro-frontend-error">
            <h3>🚫 Service Temporarily Unavailable</h3>
            <p>The ${config.name} service is currently unavailable. Please try again later.</p>
            <button (click)="retry()">Retry</button>
          </div>
        `;

        retry() {
          // Implement retry logic
        }
      },
    };
  }

  private setupHealthCheck(name: string, healthCheckUrl: string): void {
    const checkHealth = async () => {
      try {
        const response = await fetch(healthCheckUrl, {
          method: "GET",
          timeout: 5000,
        });

        const state = this.microFrontends.get(name);
        if (state) {
          state.lastHealthCheck = new Date();

          if (response.ok) {
            if (state.status === "offline") {
              state.status = "loaded";
            }
          } else {
            state.status = "offline";
          }

          this.updateState(state);
        }
      } catch (error) {
        const state = this.microFrontends.get(name);
        if (state) {
          state.status = "offline";
          state.error = "Health check failed";
          this.updateState(state);
        }
      }
    };

    // Initial health check
    checkHealth();

    // Setup periodic health checks
    const timer = setInterval(checkHealth, this.healthCheckInterval);
    this.healthCheckTimers.set(name, timer);
  }

  private updateState(state: MicroFrontendState): void {
    this.microFrontends.set(state.id, state);
    this.microFrontendStates$.next(new Map(this.microFrontends));
  }

  // 📡 CROSS-MICRO-FRONTEND COMMUNICATION
  private communicationBus = new Subject<MicroFrontendMessage>();

  initializeCommunicationBus(): void {
    // Listen for custom events
    fromEvent<CustomEvent>(window, "microfrontend-message")
      .pipe(
        map((event) => event.detail),
        filter((message) => this.validateMessage(message))
      )
      .subscribe((message) => {
        this.communicationBus.next(message);
      });

    // Listen for postMessage events
    fromEvent<MessageEvent>(window, "message")
      .pipe(
        map((event) => event.data),
        filter((data) => data.type === "MICROFRONTEND_MESSAGE"),
        map((data) => data.payload)
      )
      .subscribe((message) => {
        this.communicationBus.next(message);
      });
  }

  // 📤 SEND MESSAGE TO MICRO-FRONTEND
  sendMessage(message: MicroFrontendMessage): void {
    // Validate message
    if (!this.validateMessage(message)) {
      throw new Error("Invalid message format");
    }

    // Emit custom event
    window.dispatchEvent(
      new CustomEvent("microfrontend-message", {
        detail: message,
      })
    );

    // Send to communication bus
    this.communicationBus.next(message);
  }

  // 📥 LISTEN FOR MESSAGES
  listenForMessages(
    targetMicroFrontend?: string
  ): Observable<MicroFrontendMessage> {
    return this.communicationBus.pipe(
      filter(
        (message) =>
          !targetMicroFrontend || message.source === targetMicroFrontend
      ),
      shareReplay(1)
    );
  }

  private validateMessage(message: any): boolean {
    return (
      message &&
      typeof message.type === "string" &&
      typeof message.source === "string" &&
      message.payload !== undefined
    );
  }

  private initializeGlobalErrorHandling(): void {
    // Handle uncaught errors from micro-frontends
    window.addEventListener("error", (event) => {
      const error = event.error;

      // Try to identify which micro-frontend caused the error
      const microFrontend = this.identifyErrorSource(error);

      if (microFrontend) {
        const state = this.microFrontends.get(microFrontend);
        if (state) {
          state.status = "error";
          state.error = error.message;
          this.updateState(state);
        }
      }
    });
  }

  private identifyErrorSource(error: Error): string | null {
    // Analyze stack trace to identify micro-frontend
    const stack = error.stack || "";

    for (const [name, state] of this.microFrontends) {
      if (
        stack.includes(state.config.remoteUrl) ||
        stack.includes(state.config.name)
      ) {
        return name;
      }
    }

    return null;
  }

  // 📊 GET REGISTRY STATE
  getMicroFrontendStates(): Observable<Map<string, MicroFrontendState>> {
    return this.microFrontendStates$.asObservable();
  }

  // 🧹 CLEANUP
  dispose(): void {
    // Clear health check timers
    this.healthCheckTimers.forEach((timer) => clearInterval(timer));
    this.healthCheckTimers.clear();

    // Cleanup loaded modules
    this.loadedModules.clear();

    // Complete subjects
    this.microFrontendStates$.complete();
    this.communicationBus.complete();
  }
}

// 📩 Message Interface
export interface MicroFrontendMessage {
  type: string;
  source: string;
  target?: string;
  payload: any;
  timestamp?: Date;
  correlationId?: string;
}

// 🏗️ Micro-Frontend Module
@NgModule({})
export class MicroFrontendModule {
  static forRoot(config: {
    microFrontends: MicroFrontendConfig[];
    globalConfig?: any;
  }): ModuleWithProviders<MicroFrontendModule> {
    return {
      ngModule: MicroFrontendModule,
      providers: [
        MicroFrontendRegistry,
        {
          provide: "MICRO_FRONTEND_CONFIG",
          useValue: config.globalConfig || {},
        },
        {
          provide: APP_INITIALIZER,
          useFactory: (registry: MicroFrontendRegistry) => {
            return () => {
              // Register all micro-frontends on app start
              config.microFrontends.forEach((mf) => {
                registry.registerMicroFrontend(mf);
              });

              // Setup dynamic routing
              registry.setupDynamicRouting();
            };
          },
          deps: [MicroFrontendRegistry],
          multi: true,
        },
      ],
    };
  }
}
```

---

## 🎯 **Domain-Driven Design (DDD) Architecture**

### **🏛️ Advanced DDD Implementation with Angular**

**Problem:** You need to organize complex business logic across multiple domains while maintaining clear boundaries and reducing coupling.

```typescript
// src/app/core/domain/domain-architecture.ts
import { Injectable, InjectionToken, Inject } from "@angular/core";
import { Observable, BehaviorSubject, Subject, combineLatest } from "rxjs";
import {
  map,
  filter,
  distinctUntilChanged,
  shareReplay,
  switchMap,
} from "rxjs/operators";

// 🏗️ DOMAIN ENTITY BASE CLASS
export abstract class DomainEntity {
  constructor(
    public readonly id: string,
    protected readonly domainEvents: DomainEvent[] = []
  ) {}

  // 📤 RAISE DOMAIN EVENT
  protected raiseDomainEvent(event: DomainEvent): void {
    event.aggregateId = this.id;
    event.timestamp = new Date();
    this.domainEvents.push(event);
  }

  // 📥 GET UNCOMMITTED EVENTS
  getUncommittedEvents(): DomainEvent[] {
    return [...this.domainEvents];
  }

  // ✅ MARK EVENTS AS COMMITTED
  markEventsAsCommitted(): void {
    this.domainEvents.length = 0;
  }
}

// 🎯 VALUE OBJECT BASE CLASS
export abstract class ValueObject {
  abstract equals(other: ValueObject): boolean;

  protected static areEqual(left: any, right: any): boolean {
    if (left === right) return true;
    if (left == null || right == null) return false;

    if (typeof left === "object" && typeof right === "object") {
      return JSON.stringify(left) === JSON.stringify(right);
    }

    return false;
  }
}

// 🏢 AGGREGATE ROOT BASE CLASS
export abstract class AggregateRoot extends DomainEntity {
  private version = 0;

  constructor(id: string) {
    super(id);
  }

  // 🔄 APPLY EVENT
  protected applyEvent(event: DomainEvent): void {
    this.raiseDomainEvent(event);
    this.version++;
    this.when(event);
  }

  // 🎬 EVENT HANDLER (to be implemented by aggregates)
  protected abstract when(event: DomainEvent): void;

  // 📊 GET VERSION
  getVersion(): number {
    return this.version;
  }
}

// 📨 DOMAIN EVENT INTERFACE
export interface DomainEvent {
  eventType: string;
  aggregateId: string;
  timestamp: Date;
  version?: number;
  data: any;
  correlationId?: string;
  userId?: string;
}

// 🏪 REPOSITORY PATTERN
export interface Repository<T extends AggregateRoot> {
  getById(id: string): Observable<T | null>;
  save(aggregate: T): Observable<void>;
  delete(id: string): Observable<void>;
  findBy(criteria: any): Observable<T[]>;
}

// 🎯 DOMAIN SERVICE PATTERN
export abstract class DomainService {
  constructor(protected readonly name: string) {}
}

// 📋 SPECIFICATION PATTERN
export interface Specification<T> {
  isSatisfiedBy(candidate: T): boolean;
  and(other: Specification<T>): Specification<T>;
  or(other: Specification<T>): Specification<T>;
  not(): Specification<T>;
}

export abstract class CompositeSpecification<T> implements Specification<T> {
  abstract isSatisfiedBy(candidate: T): boolean;

  and(other: Specification<T>): Specification<T> {
    return new AndSpecification(this, other);
  }

  or(other: Specification<T>): Specification<T> {
    return new OrSpecification(this, other);
  }

  not(): Specification<T> {
    return new NotSpecification(this);
  }
}

// Implementation classes for specifications
class AndSpecification<T> extends CompositeSpecification<T> {
  constructor(private left: Specification<T>, private right: Specification<T>) {
    super();
  }

  isSatisfiedBy(candidate: T): boolean {
    return (
      this.left.isSatisfiedBy(candidate) && this.right.isSatisfiedBy(candidate)
    );
  }
}

class OrSpecification<T> extends CompositeSpecification<T> {
  constructor(private left: Specification<T>, private right: Specification<T>) {
    super();
  }

  isSatisfiedBy(candidate: T): boolean {
    return (
      this.left.isSatisfiedBy(candidate) || this.right.isSatisfiedBy(candidate)
    );
  }
}

class NotSpecification<T> extends CompositeSpecification<T> {
  constructor(private spec: Specification<T>) {
    super();
  }

  isSatisfiedBy(candidate: T): boolean {
    return !this.spec.isSatisfiedBy(candidate);
  }
}

// 🚌 DOMAIN EVENT BUS
@Injectable({
  providedIn: "root",
})
export class DomainEventBus {
  private events$ = new Subject<DomainEvent>();
  private handlers = new Map<string, Array<(event: DomainEvent) => void>>();

  // 📤 PUBLISH EVENT
  publish(event: DomainEvent): void {
    console.log(`📤 Publishing domain event: ${event.eventType}`);

    // Add metadata
    if (!event.timestamp) {
      event.timestamp = new Date();
    }

    if (!event.correlationId) {
      event.correlationId = this.generateCorrelationId();
    }

    // Emit to subscribers
    this.events$.next(event);

    // Call registered handlers
    const handlers = this.handlers.get(event.eventType) || [];
    handlers.forEach((handler) => {
      try {
        handler(event);
      } catch (error) {
        console.error(`Error handling event ${event.eventType}:`, error);
      }
    });
  }

  // 📥 SUBSCRIBE TO EVENTS
  subscribe<T extends DomainEvent>(
    eventType: string,
    handler: (event: T) => void
  ): void {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }

    this.handlers.get(eventType)!.push(handler as any);
  }

  // 🎧 LISTEN TO ALL EVENTS
  getAllEvents(): Observable<DomainEvent> {
    return this.events$.asObservable();
  }

  // 🎯 LISTEN TO SPECIFIC EVENT TYPE
  getEventsOfType<T extends DomainEvent>(eventType: string): Observable<T> {
    return this.events$.pipe(
      filter((event) => event.eventType === eventType),
      map((event) => event as T)
    );
  }

  private generateCorrelationId(): string {
    return `${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// 🏢 BOUNDED CONTEXT MANAGER
export interface BoundedContextConfig {
  name: string;
  aggregates: Array<new (...args: any[]) => AggregateRoot>;
  services: Array<new (...args: any[]) => DomainService>;
  repositories: Array<any>;
  eventHandlers: Array<any>;
  policies: Array<any>;
}

@Injectable({
  providedIn: "root",
})
export class BoundedContextRegistry {
  private contexts = new Map<string, BoundedContextConfig>();
  private contextStates$ = new BehaviorSubject<Map<string, any>>(new Map());

  // 📝 REGISTER BOUNDED CONTEXT
  registerBoundedContext(config: BoundedContextConfig): void {
    this.validateContextConfig(config);
    this.contexts.set(config.name, config);

    console.log(`📝 Registered bounded context: ${config.name}`);

    // Initialize context services
    this.initializeContext(config);
  }

  // 🎯 GET CONTEXT
  getContext(name: string): BoundedContextConfig | null {
    return this.contexts.get(name) || null;
  }

  // 📋 LIST CONTEXTS
  getAllContexts(): Map<string, BoundedContextConfig> {
    return new Map(this.contexts);
  }

  private validateContextConfig(config: BoundedContextConfig): void {
    if (!config.name || config.name.trim() === "") {
      throw new Error("Bounded context must have a name");
    }

    if (this.contexts.has(config.name)) {
      throw new Error(`Bounded context '${config.name}' already exists`);
    }
  }

  private initializeContext(config: BoundedContextConfig): void {
    // Initialize services, repositories, and event handlers
    // This would be handled by the dependency injection system

    const contextState = {
      name: config.name,
      status: "initialized",
      aggregateCount: config.aggregates.length,
      serviceCount: config.services.length,
      repositoryCount: config.repositories.length,
      initializedAt: new Date(),
    };

    const currentStates = this.contextStates$.value;
    currentStates.set(config.name, contextState);
    this.contextStates$.next(currentStates);
  }

  // 📊 GET CONTEXT STATES
  getContextStates(): Observable<Map<string, any>> {
    return this.contextStates$.asObservable();
  }
}

// 🎬 COMMAND PATTERN
export interface Command {
  commandType: string;
  aggregateId: string;
  data: any;
  userId?: string;
  timestamp?: Date;
}

export interface CommandHandler<T extends Command> {
  handle(command: T): Observable<void>;
}

// 🚌 COMMAND BUS
@Injectable({
  providedIn: "root",
})
export class CommandBus {
  private handlers = new Map<string, CommandHandler<any>>();

  // 📝 REGISTER HANDLER
  registerHandler<T extends Command>(
    commandType: string,
    handler: CommandHandler<T>
  ): void {
    if (this.handlers.has(commandType)) {
      throw new Error(`Handler for command ${commandType} already registered`);
    }

    this.handlers.set(commandType, handler);
    console.log(`📝 Registered command handler for: ${commandType}`);
  }

  // 📤 SEND COMMAND
  send<T extends Command>(command: T): Observable<void> {
    const handler = this.handlers.get(command.commandType);

    if (!handler) {
      throw new Error(
        `No handler registered for command: ${command.commandType}`
      );
    }

    // Add metadata
    if (!command.timestamp) {
      command.timestamp = new Date();
    }

    console.log(`📤 Sending command: ${command.commandType}`);

    return handler.handle(command);
  }
}

// 🔍 QUERY PATTERN
export interface Query {
  queryType: string;
  parameters: any;
  userId?: string;
}

export interface QueryHandler<T extends Query, R> {
  handle(query: T): Observable<R>;
}

// 🚌 QUERY BUS
@Injectable({
  providedIn: "root",
})
export class QueryBus {
  private handlers = new Map<string, QueryHandler<any, any>>();

  // 📝 REGISTER HANDLER
  registerHandler<T extends Query, R>(
    queryType: string,
    handler: QueryHandler<T, R>
  ): void {
    if (this.handlers.has(queryType)) {
      throw new Error(`Handler for query ${queryType} already registered`);
    }

    this.handlers.set(queryType, handler);
    console.log(`📝 Registered query handler for: ${queryType}`);
  }

  // 📥 EXECUTE QUERY
  execute<T extends Query, R>(query: T): Observable<R> {
    const handler = this.handlers.get(query.queryType);

    if (!handler) {
      throw new Error(`No handler registered for query: ${query.queryType}`);
    }

    console.log(`📥 Executing query: ${query.queryType}`);

    return handler.handle(query);
  }
}
```

This is **Part 1** of the Enterprise Architecture Patterns guide. Would you like me to continue with **Part 2** covering:

- 🏗️ **Clean Architecture Implementation**
- 🔧 **CQRS Pattern with Event Sourcing**
- 🌐 **Hexagonal Architecture (Ports & Adapters)**
- 🚀 **Scalable Module Federation Strategies**

Should I proceed with Part 2? 🚀
