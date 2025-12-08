# 🌊 Advanced RxJS Patterns in Angular: Expert Implementation Guide

## 🎯 **Question Overview**

_"How do you implement advanced RxJS patterns like error recovery, backpressure handling, custom operators, and complex data flow management in Angular applications?"_

## 🔍 **Understanding Advanced RxJS Patterns**

Advanced RxJS patterns enable **robust data flow management**, **sophisticated error handling**, **performance optimization**, and **complex asynchronous orchestration**. Modern Angular applications require **custom operators**, **backpressure management**, **error recovery strategies**, and **memory leak prevention**! 🚀

## 🛠️ **Custom Operators Implementation**

### **1. 🎯 Advanced Error Recovery Operators**

```typescript
// src/app/shared/operators/error-recovery.operators.ts
import { Observable, timer, throwError, EMPTY, of } from "rxjs";
import {
  retryWhen,
  mergeMap,
  finalize,
  tap,
  catchError,
  switchMap,
  delay,
  take,
  concat,
  scan,
} from "rxjs/operators";

export interface RetryConfig {
  maxRetries: number;
  baseDelay: number;
  maxDelay: number;
  exponentialBackoff: boolean;
  jitter: boolean;
  retryCondition?: (error: any, retryCount: number) => boolean;
  onRetry?: (error: any, retryCount: number) => void;
}

export interface CircuitBreakerConfig {
  failureThreshold: number;
  recoveryTimeout: number;
  monitoringPeriod: number;
  onStateChange?: (state: "closed" | "open" | "half-open") => void;
}

// Advanced retry with exponential backoff and jitter
export function retryWithBackoff<T>(config: RetryConfig) {
  return (source: Observable<T>) => {
    return source.pipe(
      retryWhen((errors) =>
        errors.pipe(
          scan((retryCount, error) => {
            // Check if we should retry this error
            if (
              config.retryCondition &&
              !config.retryCondition(error, retryCount + 1)
            ) {
              throw error;
            }

            // Check max retries
            if (retryCount >= config.maxRetries) {
              throw error;
            }

            return retryCount + 1;
          }, 0),
          mergeMap((retryCount, index) => {
            // Calculate delay with exponential backoff
            let delay = config.baseDelay;

            if (config.exponentialBackoff) {
              delay = Math.min(
                config.baseDelay * Math.pow(2, retryCount - 1),
                config.maxDelay
              );
            }

            // Add jitter to prevent thundering herd
            if (config.jitter) {
              delay = delay + Math.random() * delay * 0.1;
            }

            // Call retry callback
            if (config.onRetry) {
              const error = errors.pipe(take(1));
              error.subscribe((err) => config.onRetry!(err, retryCount));
            }

            console.log(`Retry attempt ${retryCount} in ${delay}ms`);
            return timer(delay);
          })
        )
      )
    );
  };
}

// Circuit breaker pattern implementation
export function circuitBreaker<T>(config: CircuitBreakerConfig) {
  let state: "closed" | "open" | "half-open" = "closed";
  let failureCount = 0;
  let lastFailureTime = 0;
  let successCount = 0;

  return (source: Observable<T>) => {
    return new Observable<T>((subscriber) => {
      const now = Date.now();

      // Check if circuit should transition from open to half-open
      if (state === "open" && now - lastFailureTime >= config.recoveryTimeout) {
        state = "half-open";
        failureCount = 0;
        successCount = 0;
        config.onStateChange?.("half-open");
        console.log("Circuit breaker: OPEN -> HALF-OPEN");
      }

      // If circuit is open, immediately fail
      if (state === "open") {
        subscriber.error(new Error("Circuit breaker is OPEN"));
        return;
      }

      // Subscribe to source
      const subscription = source.subscribe({
        next: (value) => {
          // Success in half-open state
          if (state === "half-open") {
            successCount++;
            if (successCount >= 3) {
              // Require 3 successes to close
              state = "closed";
              failureCount = 0;
              config.onStateChange?.("closed");
              console.log("Circuit breaker: HALF-OPEN -> CLOSED");
            }
          } else if (state === "closed") {
            // Reset failure count on success
            failureCount = 0;
          }

          subscriber.next(value);
        },
        error: (error) => {
          failureCount++;
          lastFailureTime = now;

          if (state === "closed" && failureCount >= config.failureThreshold) {
            state = "open";
            config.onStateChange?.("open");
            console.log("Circuit breaker: CLOSED -> OPEN");
          } else if (state === "half-open") {
            state = "open";
            config.onStateChange?.("open");
            console.log("Circuit breaker: HALF-OPEN -> OPEN");
          }

          subscriber.error(error);
        },
        complete: () => {
          subscriber.complete();
        },
      });

      return () => subscription.unsubscribe();
    });
  };
}

// Fallback with multiple strategies
export function fallback<T>(...fallbackSources: Observable<T>[]) {
  return (source: Observable<T>) => {
    return source.pipe(
      catchError((error) => {
        console.log("Primary source failed, trying fallbacks:", error.message);

        if (fallbackSources.length === 0) {
          return throwError(error);
        }

        // Try fallbacks in sequence
        return fallbackSources.reduce((acc, fallbackSource, index) => {
          return acc.pipe(
            catchError((fallbackError) => {
              console.log(
                `Fallback ${index + 1} failed:`,
                fallbackError.message
              );

              if (index === fallbackSources.length - 1) {
                // Last fallback failed, throw original error
                return throwError(error);
              }

              return throwError(fallbackError);
            })
          );
        }, fallbackSources[0]);
      })
    );
  };
}

// Timeout with custom error message
export function timeoutWithError<T>(timeout: number, errorMessage?: string) {
  return (source: Observable<T>) => {
    return new Observable<T>((subscriber) => {
      let timedOut = false;

      const timeoutId = setTimeout(() => {
        timedOut = true;
        subscriber.error(
          new Error(errorMessage || `Operation timed out after ${timeout}ms`)
        );
      }, timeout);

      const subscription = source.subscribe({
        next: (value) => {
          if (!timedOut) {
            subscriber.next(value);
          }
        },
        error: (error) => {
          if (!timedOut) {
            clearTimeout(timeoutId);
            subscriber.error(error);
          }
        },
        complete: () => {
          if (!timedOut) {
            clearTimeout(timeoutId);
            subscriber.complete();
          }
        },
      });

      return () => {
        clearTimeout(timeoutId);
        subscription.unsubscribe();
      };
    });
  };
}
```

### **2. 🎛️ Backpressure and Flow Control Operators**

```typescript
// src/app/shared/operators/backpressure.operators.ts
import {
  Observable,
  Subject,
  BehaviorSubject,
  timer,
  EMPTY,
  merge,
} from "rxjs";
import {
  mergeMap,
  concatMap,
  exhaustMap,
  switchMap,
  buffer,
  bufferTime,
  bufferCount,
  debounceTime,
  throttleTime,
  sample,
  auditTime,
  distinctUntilChanged,
  filter,
  scan,
  share,
  tap,
  take,
  skip,
} from "rxjs/operators";

export interface BackpressureConfig {
  strategy: "drop" | "buffer" | "throttle" | "sample";
  bufferSize?: number;
  timeWindow?: number;
  dropThreshold?: number;
}

export interface RateLimitConfig {
  maxCalls: number;
  timeWindow: number;
  strategy: "sliding" | "fixed";
  onLimitExceeded?: (droppedCount: number) => void;
}

// Advanced backpressure handling
export function handleBackpressure<T>(config: BackpressureConfig) {
  return (source: Observable<T>) => {
    switch (config.strategy) {
      case "drop":
        return source.pipe(
          scan(
            (acc: { count: number; last: T | null }, current: T) => {
              if (acc.count < (config.dropThreshold || 100)) {
                return { count: acc.count + 1, last: current };
              } else {
                console.warn("Backpressure: Dropping value due to overflow");
                return acc; // Drop the value
              }
            },
            { count: 0, last: null }
          ),
          filter((state) => state.last !== null),
          map((state) => state.last!)
        );

      case "buffer":
        if (config.timeWindow) {
          return source.pipe(
            bufferTime(config.timeWindow, null, config.bufferSize),
            filter((buffer) => buffer.length > 0),
            mergeMap((buffer) => buffer) // Flatten the buffer
          );
        } else {
          return source.pipe(
            bufferCount(config.bufferSize || 10),
            mergeMap((buffer) => buffer)
          );
        }

      case "throttle":
        return source.pipe(throttleTime(config.timeWindow || 1000));

      case "sample":
        return source.pipe(sample(timer(0, config.timeWindow || 1000)));

      default:
        return source;
    }
  };
}

// Rate limiting operator
export function rateLimit<T>(config: RateLimitConfig) {
  return (source: Observable<T>) => {
    if (config.strategy === "sliding") {
      return slidingWindowRateLimit<T>(config)(source);
    } else {
      return fixedWindowRateLimit<T>(config)(source);
    }
  };
}

// Sliding window rate limiting
function slidingWindowRateLimit<T>(config: RateLimitConfig) {
  let callTimes: number[] = [];

  return (source: Observable<T>) => {
    return source.pipe(
      filter(() => {
        const now = Date.now();

        // Remove calls outside the time window
        callTimes = callTimes.filter((time) => now - time < config.timeWindow);

        // Check if we can make another call
        if (callTimes.length < config.maxCalls) {
          callTimes.push(now);
          return true;
        } else {
          config.onLimitExceeded?.(1);
          console.warn("Rate limit exceeded (sliding window)");
          return false;
        }
      })
    );
  };
}

// Fixed window rate limiting
function fixedWindowRateLimit<T>(config: RateLimitConfig) {
  let windowStart = Date.now();
  let callCount = 0;

  return (source: Observable<T>) => {
    return source.pipe(
      filter(() => {
        const now = Date.now();

        // Reset window if time has passed
        if (now - windowStart >= config.timeWindow) {
          windowStart = now;
          callCount = 0;
        }

        // Check if we can make another call
        if (callCount < config.maxCalls) {
          callCount++;
          return true;
        } else {
          config.onLimitExceeded?.(1);
          console.warn("Rate limit exceeded (fixed window)");
          return false;
        }
      })
    );
  };
}

// Adaptive batching based on load
export function adaptiveBatch<T>(
  minBatchSize: number = 1,
  maxBatchSize: number = 10,
  maxWaitTime: number = 1000
) {
  return (source: Observable<T>) => {
    let currentBatchSize = minBatchSize;
    let processingTime = 0;

    return source.pipe(
      buffer(
        merge(
          timer(maxWaitTime), // Maximum wait time
          source.pipe(
            scan((count, _) => count + 1, 0),
            filter((count) => count >= currentBatchSize)
          )
        )
      ),
      filter((batch) => batch.length > 0),
      tap((batch) => {
        const start = performance.now();

        // Simulate processing (in real scenario, this would be actual processing)
        return timer(0).pipe(
          tap(() => {
            processingTime = performance.now() - start;

            // Adapt batch size based on processing time
            if (processingTime < 50 && currentBatchSize < maxBatchSize) {
              currentBatchSize = Math.min(currentBatchSize + 1, maxBatchSize);
              console.log(`Increased batch size to ${currentBatchSize}`);
            } else if (
              processingTime > 200 &&
              currentBatchSize > minBatchSize
            ) {
              currentBatchSize = Math.max(currentBatchSize - 1, minBatchSize);
              console.log(`Decreased batch size to ${currentBatchSize}`);
            }
          })
        );
      }),
      mergeMap((batch) => batch) // Flatten the batch
    );
  };
}

// Load balancing across multiple processors
export function loadBalance<T, R>(
  processors: Array<(item: T) => Observable<R>>,
  strategy: "round-robin" | "least-busy" | "random" = "round-robin"
) {
  let currentIndex = 0;
  const busyProcessors = new Map<number, boolean>();

  return (source: Observable<T>) => {
    return source.pipe(
      mergeMap((item) => {
        let selectedIndex: number;

        switch (strategy) {
          case "round-robin":
            selectedIndex = currentIndex % processors.length;
            currentIndex++;
            break;

          case "least-busy":
            selectedIndex = 0;
            for (let i = 0; i < processors.length; i++) {
              if (!busyProcessors.get(i)) {
                selectedIndex = i;
                break;
              }
            }
            break;

          case "random":
            selectedIndex = Math.floor(Math.random() * processors.length);
            break;

          default:
            selectedIndex = 0;
        }

        // Mark processor as busy
        busyProcessors.set(selectedIndex, true);

        return processors[selectedIndex](item).pipe(
          finalize(() => {
            // Mark processor as available
            busyProcessors.set(selectedIndex, false);
          })
        );
      })
    );
  };
}
```

### **3. 🎨 Data Transformation and Caching Operators**

```typescript
// src/app/shared/operators/data-transformation.operators.ts
import {
  Observable,
  BehaviorSubject,
  timer,
  of,
  EMPTY,
  throwError,
} from "rxjs";
import {
  map,
  tap,
  mergeMap,
  switchMap,
  distinctUntilChanged,
  filter,
  scan,
  pairwise,
  startWith,
  shareReplay,
  catchError,
} from "rxjs/operators";

export interface CacheConfig {
  ttl: number; // Time to live in milliseconds
  maxSize: number;
  strategy: "lru" | "fifo" | "lfu";
  keySelector?: (value: any) => string;
}

export interface TransformConfig {
  debounceMs?: number;
  distinctBy?: (value: any) => any;
  validator?: (value: any) => boolean;
  transformer?: (value: any) => any;
}

// Smart caching operator with TTL and eviction strategies
export function smartCache<T>(config: CacheConfig) {
  const cache = new Map<
    string,
    { value: T; timestamp: number; accessCount: number }
  >();
  const accessOrder: string[] = []; // For LRU

  return (source: Observable<T>) => {
    return source.pipe(
      mergeMap((value) => {
        const key = config.keySelector
          ? config.keySelector(value)
          : JSON.stringify(value);
        const now = Date.now();
        const cached = cache.get(key);

        // Check if cached value is still valid
        if (cached && now - cached.timestamp < config.ttl) {
          // Update access for LRU and LFU
          cached.accessCount++;
          updateAccessOrder(key);
          console.log(`Cache hit for key: ${key}`);
          return of(cached.value);
        }

        // Cache miss or expired - process and cache the value
        return of(value).pipe(
          tap((processedValue) => {
            // Evict if cache is full
            if (cache.size >= config.maxSize) {
              evictFromCache();
            }

            // Add to cache
            cache.set(key, {
              value: processedValue,
              timestamp: now,
              accessCount: 1,
            });

            updateAccessOrder(key);
            console.log(`Cached value for key: ${key}`);
          })
        );
      })
    );

    function updateAccessOrder(key: string): void {
      if (config.strategy === "lru") {
        const index = accessOrder.indexOf(key);
        if (index > -1) {
          accessOrder.splice(index, 1);
        }
        accessOrder.push(key);
      }
    }

    function evictFromCache(): void {
      let keyToEvict: string;

      switch (config.strategy) {
        case "lru":
          keyToEvict = accessOrder.shift()!;
          break;

        case "fifo":
          keyToEvict = cache.keys().next().value;
          break;

        case "lfu":
          let minAccess = Infinity;
          keyToEvict = "";
          cache.forEach((entry, key) => {
            if (entry.accessCount < minAccess) {
              minAccess = entry.accessCount;
              keyToEvict = key;
            }
          });
          break;

        default:
          keyToEvict = cache.keys().next().value;
      }

      cache.delete(keyToEvict);
      console.log(`Evicted cache entry: ${keyToEvict}`);
    }
  };
}

// Delta detection operator
export function detectDeltas<T>(keySelector?: (value: T) => string) {
  return (source: Observable<T[]>) => {
    return source.pipe(
      startWith([]),
      pairwise(),
      map(([previous, current]) => {
        const getKey =
          keySelector || ((item: any) => item.id || JSON.stringify(item));

        const previousMap = new Map(
          previous.map((item) => [getKey(item), item])
        );
        const currentMap = new Map(current.map((item) => [getKey(item), item]));

        const added: T[] = [];
        const removed: T[] = [];
        const updated: T[] = [];

        // Find added and updated items
        current.forEach((item) => {
          const key = getKey(item);
          const previousItem = previousMap.get(key);

          if (!previousItem) {
            added.push(item);
          } else if (JSON.stringify(previousItem) !== JSON.stringify(item)) {
            updated.push(item);
          }
        });

        // Find removed items
        previous.forEach((item) => {
          const key = getKey(item);
          if (!currentMap.has(key)) {
            removed.push(item);
          }
        });

        return { added, removed, updated, current };
      })
    );
  };
}

// Smart data transformation with validation
export function transformData<T, R>(
  transformFn: (value: T) => R,
  config: TransformConfig = {}
) {
  return (source: Observable<T>) => {
    let stream = source;

    // Apply debouncing if configured
    if (config.debounceMs) {
      stream = stream.pipe(debounceTime(config.debounceMs));
    }

    // Apply validation if configured
    if (config.validator) {
      stream = stream.pipe(filter(config.validator));
    }

    // Apply distinctness check if configured
    if (config.distinctBy) {
      stream = stream.pipe(distinctUntilChanged(config.distinctBy));
    }

    // Apply transformation
    return stream.pipe(
      map((value) => {
        try {
          return transformFn(value);
        } catch (error) {
          console.error("Transformation failed:", error);
          throw error;
        }
      }),
      catchError((error) => {
        console.error("Data transformation error:", error);
        return throwError(() => error);
      })
    );
  };
}

// Aggregate multiple streams with conflict resolution
export function aggregateStreams<T>(
  conflictResolver: (values: T[]) => T = (values) => values[values.length - 1]
) {
  return (...sources: Observable<T>[]) => {
    const latestValues = new Map<number, T>();

    return new Observable<T>((subscriber) => {
      const subscriptions = sources.map((source, index) => {
        return source.subscribe({
          next: (value) => {
            latestValues.set(index, value);

            // If we have values from all sources, resolve conflicts and emit
            if (latestValues.size === sources.length) {
              const values = Array.from(latestValues.values());
              const resolvedValue = conflictResolver(values);
              subscriber.next(resolvedValue);
            }
          },
          error: (error) => subscriber.error(error),
          complete: () => {
            // Complete only when all sources complete
            latestValues.delete(index);
            if (latestValues.size === 0) {
              subscriber.complete();
            }
          },
        });
      });

      return () => {
        subscriptions.forEach((sub) => sub.unsubscribe());
      };
    });
  };
}

// Conditional operator chaining
export function conditionalPipe<T>(
  condition: (value: T) => boolean,
  trueOperator: (source: Observable<T>) => Observable<T>,
  falseOperator?: (source: Observable<T>) => Observable<T>
) {
  return (source: Observable<T>) => {
    return source.pipe(
      mergeMap((value) => {
        if (condition(value)) {
          return trueOperator(of(value));
        } else if (falseOperator) {
          return falseOperator(of(value));
        } else {
          return of(value);
        }
      })
    );
  };
}
```

## 🏗️ **Complex Data Flow Management**

### **1. 📊 Advanced Stream Orchestration Service**

```typescript
// src/app/shared/services/stream-orchestrator.service.ts
import { Injectable, inject, OnDestroy } from "@angular/core";
import {
  Observable,
  Subject,
  BehaviorSubject,
  merge,
  combineLatest,
  forkJoin,
  of,
  EMPTY,
  timer,
  interval,
} from "rxjs";
import {
  switchMap,
  mergeMap,
  concatMap,
  exhaustMap,
  takeUntil,
  share,
  shareReplay,
  startWith,
  scan,
  distinctUntilChanged,
  filter,
  map,
  tap,
  catchError,
  retry,
  timeout,
} from "rxjs/operators";

import {
  retryWithBackoff,
  circuitBreaker,
} from "../operators/error-recovery.operators";
import {
  handleBackpressure,
  rateLimit,
} from "../operators/backpressure.operators";
import {
  smartCache,
  detectDeltas,
} from "../operators/data-transformation.operators";

export interface StreamConfig {
  id: string;
  source: Observable<any>;
  backpressure?: any;
  rateLimit?: any;
  cache?: any;
  errorRecovery?: any;
  dependencies?: string[];
}

export interface StreamMetrics {
  id: string;
  eventsProcessed: number;
  errorsCount: number;
  averageLatency: number;
  lastActivity: Date;
  isActive: boolean;
}

@Injectable({
  providedIn: "root",
})
export class StreamOrchestratorService implements OnDestroy {
  private readonly destroy$ = new Subject<void>();
  private streams = new Map<string, Observable<any>>();
  private metrics = new Map<string, StreamMetrics>();
  private dependencies = new Map<string, string[]>();

  // Signal-based reactive state
  private readonly _activeStreams = new BehaviorSubject<string[]>([]);
  private readonly _streamMetrics = new BehaviorSubject<StreamMetrics[]>([]);

  readonly activeStreams$ = this._activeStreams.asObservable();
  readonly streamMetrics$ = this._streamMetrics.asObservable();

  constructor() {
    // Start metrics collection
    this.startMetricsCollection();
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  // Register and configure streams
  registerStream<T>(config: StreamConfig): Observable<T> {
    let stream = config.source;

    // Apply backpressure handling
    if (config.backpressure) {
      stream = stream.pipe(handleBackpressure(config.backpressure));
    }

    // Apply rate limiting
    if (config.rateLimit) {
      stream = stream.pipe(rateLimit(config.rateLimit));
    }

    // Apply caching
    if (config.cache) {
      stream = stream.pipe(smartCache(config.cache));
    }

    // Apply error recovery
    if (config.errorRecovery) {
      if (config.errorRecovery.retryConfig) {
        stream = stream.pipe(
          retryWithBackoff(config.errorRecovery.retryConfig)
        );
      }

      if (config.errorRecovery.circuitBreakerConfig) {
        stream = stream.pipe(
          circuitBreaker(config.errorRecovery.circuitBreakerConfig)
        );
      }
    }

    // Add metrics tracking
    stream = stream.pipe(
      tap(() => this.updateStreamMetrics(config.id, "event")),
      catchError((error) => {
        this.updateStreamMetrics(config.id, "error");
        throw error;
      }),
      share()
    );

    // Store stream and dependencies
    this.streams.set(config.id, stream);
    if (config.dependencies) {
      this.dependencies.set(config.id, config.dependencies);
    }

    // Initialize metrics
    this.initializeStreamMetrics(config.id);

    // Update active streams list
    this.updateActiveStreamsList();

    return stream;
  }

  // Get stream by ID
  getStream<T>(id: string): Observable<T> | undefined {
    return this.streams.get(id);
  }

  // Combine multiple streams with dependency resolution
  combineStreams<T>(
    streamIds: string[],
    combinator?: (...values: any[]) => T
  ): Observable<T> {
    // Resolve dependencies
    const resolvedIds = this.resolveDependencies(streamIds);
    const streams = resolvedIds
      .map((id) => this.streams.get(id))
      .filter(Boolean) as Observable<any>[];

    if (streams.length === 0) {
      return EMPTY;
    }

    if (combinator) {
      return combineLatest(streams).pipe(
        map((values) => combinator(...values)),
        distinctUntilChanged()
      );
    } else {
      return combineLatest(streams) as Observable<T>;
    }
  }

  // Merge multiple streams with prioritization
  mergeStreamsWithPriority<T>(
    streamConfigs: Array<{ id: string; priority: number }>
  ): Observable<T> {
    // Sort by priority (higher number = higher priority)
    const sortedConfigs = streamConfigs.sort((a, b) => b.priority - a.priority);

    const prioritizedStreams = sortedConfigs
      .map((config) => {
        const stream = this.streams.get(config.id);
        if (!stream) return EMPTY;

        // Add priority metadata
        return stream.pipe(
          map((value) => ({
            value,
            priority: config.priority,
            streamId: config.id,
          }))
        );
      })
      .filter((stream) => stream !== EMPTY);

    return merge(...prioritizedStreams).pipe(
      scan((acc: any, current: any) => {
        // Emit only if priority is higher or equal
        if (!acc || current.priority >= acc.priority) {
          return current;
        }
        return acc;
      }, null),
      filter(Boolean),
      map((item) => item.value),
      distinctUntilChanged()
    );
  }

  // Create data pipeline with stages
  createPipeline<T, R>(
    sourceStreamId: string,
    stages: Array<{
      name: string;
      operator: (source: Observable<any>) => Observable<any>;
      errorHandler?: (error: any, stage: string) => Observable<any>;
    }>
  ): Observable<R> {
    const sourceStream = this.streams.get(sourceStreamId);
    if (!sourceStream) {
      throw new Error(`Source stream '${sourceStreamId}' not found`);
    }

    let pipeline: Observable<any> = sourceStream;

    stages.forEach((stage, index) => {
      const stageName = `${sourceStreamId}_${stage.name}_${index}`;

      pipeline = pipeline.pipe(
        switchMap((value) => {
          const stageStart = performance.now();

          return of(value).pipe(
            stage.operator,
            tap(() => {
              const stageDuration = performance.now() - stageStart;
              console.log(
                `Pipeline stage '${
                  stage.name
                }' completed in ${stageDuration.toFixed(2)}ms`
              );
            }),
            catchError((error) => {
              console.error(`Pipeline stage '${stage.name}' failed:`, error);

              if (stage.errorHandler) {
                return stage.errorHandler(error, stage.name);
              } else {
                throw error;
              }
            })
          );
        })
      );
    });

    // Register the pipeline as a new stream
    const pipelineId = `${sourceStreamId}_pipeline`;
    this.streams.set(pipelineId, pipeline);
    this.initializeStreamMetrics(pipelineId);

    return pipeline;
  }

  // Fan-out pattern: distribute data to multiple processors
  fanOut<T, R>(
    sourceStreamId: string,
    processors: Array<{
      id: string;
      processor: (value: T) => Observable<R>;
      filter?: (value: T) => boolean;
    }>
  ): Map<string, Observable<R>> {
    const sourceStream = this.streams.get(sourceStreamId);
    if (!sourceStream) {
      throw new Error(`Source stream '${sourceStreamId}' not found`);
    }

    const results = new Map<string, Observable<R>>();

    processors.forEach((config) => {
      let processedStream: Observable<R> = sourceStream.pipe(
        filter(config.filter || (() => true)),
        mergeMap((value) => config.processor(value)),
        share()
      );

      // Register each fan-out stream
      const fanOutId = `${sourceStreamId}_fanout_${config.id}`;
      this.streams.set(fanOutId, processedStream);
      this.initializeStreamMetrics(fanOutId);

      results.set(config.id, processedStream);
    });

    return results;
  }

  // Fan-in pattern: aggregate results from multiple streams
  fanIn<T, R>(
    streamIds: string[],
    aggregator: (values: T[]) => R,
    strategy: "wait-for-all" | "first-wins" | "latest-wins" = "latest-wins"
  ): Observable<R> {
    const streams = streamIds
      .map((id) => this.streams.get(id))
      .filter(Boolean) as Observable<T>[];

    if (streams.length === 0) {
      return EMPTY;
    }

    let aggregatedStream: Observable<R>;

    switch (strategy) {
      case "wait-for-all":
        aggregatedStream = forkJoin(streams).pipe(
          map((values) => aggregator(values))
        );
        break;

      case "first-wins":
        aggregatedStream = merge(...streams).pipe(
          take(1),
          map((value) => aggregator([value]))
        );
        break;

      case "latest-wins":
      default:
        aggregatedStream = combineLatest(streams).pipe(
          map((values) => aggregator(values)),
          distinctUntilChanged()
        );
        break;
    }

    // Register aggregated stream
    const fanInId = `fanin_${streamIds.join("_")}`;
    this.streams.set(fanInId, aggregatedStream);
    this.initializeStreamMetrics(fanInId);

    return aggregatedStream;
  }

  // Stream lifecycle management
  pauseStream(id: string): void {
    // Implementation would depend on specific requirements
    // Could use a subject to control the stream flow
    console.log(`Pausing stream: ${id}`);
  }

  resumeStream(id: string): void {
    console.log(`Resuming stream: ${id}`);
  }

  removeStream(id: string): void {
    this.streams.delete(id);
    this.metrics.delete(id);
    this.dependencies.delete(id);
    this.updateActiveStreamsList();
  }

  // Performance monitoring
  getStreamMetrics(id: string): StreamMetrics | undefined {
    return this.metrics.get(id);
  }

  getAllMetrics(): StreamMetrics[] {
    return Array.from(this.metrics.values());
  }

  // Private helper methods
  private resolveDependencies(streamIds: string[]): string[] {
    const resolved = new Set<string>();
    const visiting = new Set<string>();

    const visit = (id: string): void => {
      if (resolved.has(id)) return;
      if (visiting.has(id)) {
        throw new Error(`Circular dependency detected: ${id}`);
      }

      visiting.add(id);

      const deps = this.dependencies.get(id) || [];
      deps.forEach((dep) => visit(dep));

      visiting.delete(id);
      resolved.add(id);
    };

    streamIds.forEach((id) => visit(id));
    return Array.from(resolved);
  }

  private initializeStreamMetrics(id: string): void {
    this.metrics.set(id, {
      id,
      eventsProcessed: 0,
      errorsCount: 0,
      averageLatency: 0,
      lastActivity: new Date(),
      isActive: true,
    });
  }

  private updateStreamMetrics(id: string, type: "event" | "error"): void {
    const metrics = this.metrics.get(id);
    if (!metrics) return;

    if (type === "event") {
      metrics.eventsProcessed++;
    } else {
      metrics.errorsCount++;
    }

    metrics.lastActivity = new Date();
    this.metrics.set(id, metrics);
  }

  private updateActiveStreamsList(): void {
    const activeIds = Array.from(this.streams.keys());
    this._activeStreams.next(activeIds);
  }

  private startMetricsCollection(): void {
    interval(5000)
      .pipe(takeUntil(this.destroy$))
      .subscribe(() => {
        const allMetrics = Array.from(this.metrics.values());
        this._streamMetrics.next(allMetrics);
      });
  }
}
```

### **2. 🎯 Advanced Use Case: Real-time Data Dashboard**

```typescript
// src/app/features/dashboard/services/dashboard-streams.service.ts
import { Injectable, inject } from "@angular/core";
import { Observable, combineLatest, timer } from "rxjs";
import { map, startWith, share } from "rxjs/operators";

import { StreamOrchestratorService } from "../../../shared/services/stream-orchestrator.service";
import { WebSocketService } from "../../../shared/services/websocket.service";
import { HttpClient } from "@angular/common/http";

export interface DashboardData {
  metrics: {
    activeUsers: number;
    revenue: number;
    orders: number;
    performance: number;
  };
  charts: {
    salesTrend: number[];
    userActivity: number[];
    systemHealth: number[];
  };
  alerts: Array<{
    id: string;
    type: "warning" | "error" | "info";
    message: string;
    timestamp: Date;
  }>;
}

@Injectable({
  providedIn: "root",
})
export class DashboardStreamsService {
  private streamOrchestrator = inject(StreamOrchestratorService);
  private websocketService = inject(WebSocketService);
  private http = inject(HttpClient);

  constructor() {
    this.setupDashboardStreams();
  }

  private setupDashboardStreams(): void {
    // Real-time metrics stream
    this.streamOrchestrator.registerStream({
      id: "real-time-metrics",
      source: this.websocketService.connect("ws://localhost:8080/metrics"),
      backpressure: {
        strategy: "throttle",
        timeWindow: 1000, // Throttle to 1 update per second
      },
      cache: {
        ttl: 5000, // 5 seconds TTL
        maxSize: 100,
        strategy: "lru",
      },
    });

    // Historical data stream
    this.streamOrchestrator.registerStream({
      id: "historical-data",
      source: timer(0, 30000).pipe(
        // Every 30 seconds
        switchMap(() => this.http.get("/api/dashboard/historical"))
      ),
      errorRecovery: {
        retryConfig: {
          maxRetries: 3,
          baseDelay: 1000,
          maxDelay: 5000,
          exponentialBackoff: true,
          jitter: true,
        },
      },
    });

    // User activity stream
    this.streamOrchestrator.registerStream({
      id: "user-activity",
      source: this.websocketService.connect(
        "ws://localhost:8080/user-activity"
      ),
      backpressure: {
        strategy: "buffer",
        timeWindow: 2000,
        bufferSize: 50,
      },
    });

    // System alerts stream
    this.streamOrchestrator.registerStream({
      id: "system-alerts",
      source: this.websocketService.connect("ws://localhost:8080/alerts"),
      rateLimit: {
        maxCalls: 10,
        timeWindow: 60000, // 10 alerts per minute max
        strategy: "sliding",
      },
    });

    // Performance metrics stream
    this.streamOrchestrator.registerStream({
      id: "performance-metrics",
      source: timer(0, 5000).pipe(
        // Every 5 seconds
        switchMap(() => this.http.get("/api/performance"))
      ),
      dependencies: ["real-time-metrics"],
      errorRecovery: {
        circuitBreakerConfig: {
          failureThreshold: 5,
          recoveryTimeout: 30000,
          monitoringPeriod: 60000,
        },
      },
    });
  }

  // Combined dashboard data stream
  getDashboardData(): Observable<DashboardData> {
    return this.streamOrchestrator
      .combineStreams(
        [
          "real-time-metrics",
          "historical-data",
          "user-activity",
          "system-alerts",
        ],
        (realTimeMetrics, historicalData, userActivity, systemAlerts) => {
          return {
            metrics: {
              activeUsers: realTimeMetrics?.activeUsers || 0,
              revenue: realTimeMetrics?.revenue || 0,
              orders: realTimeMetrics?.orders || 0,
              performance: realTimeMetrics?.performance || 0,
            },
            charts: {
              salesTrend: historicalData?.salesTrend || [],
              userActivity: userActivity?.trend || [],
              systemHealth: historicalData?.systemHealth || [],
            },
            alerts: systemAlerts?.alerts || [],
          };
        }
      )
      .pipe(
        startWith({
          metrics: { activeUsers: 0, revenue: 0, orders: 0, performance: 0 },
          charts: { salesTrend: [], userActivity: [], systemHealth: [] },
          alerts: [],
        }),
        share()
      );
  }

  // Specialized streams for different dashboard sections
  getMetricsStream(): Observable<any> {
    return (
      this.streamOrchestrator.getStream("real-time-metrics") || new Observable()
    );
  }

  getAlertsStream(): Observable<any> {
    return (
      this.streamOrchestrator.getStream("system-alerts") || new Observable()
    );
  }

  // Performance monitoring for the dashboard itself
  getDashboardPerformanceMetrics(): Observable<any> {
    return this.streamOrchestrator.streamMetrics$.pipe(
      map((metrics) => {
        const dashboardMetrics = metrics.filter(
          (m) =>
            m.id.includes("dashboard") ||
            m.id.includes("real-time") ||
            m.id.includes("historical")
        );

        return {
          totalEventsProcessed: dashboardMetrics.reduce(
            (sum, m) => sum + m.eventsProcessed,
            0
          ),
          totalErrors: dashboardMetrics.reduce(
            (sum, m) => sum + m.errorsCount,
            0
          ),
          averageLatency:
            dashboardMetrics.reduce((sum, m) => sum + m.averageLatency, 0) /
            dashboardMetrics.length,
          activeStreams: dashboardMetrics.filter((m) => m.isActive).length,
        };
      })
    );
  }
}
```

### **3. 🧠 Memory Leak Prevention & Resource Management**

```typescript
// src/app/shared/operators/memory-management.operators.ts
import {
  Observable,
  Subject,
  BehaviorSubject,
  timer,
  EMPTY,
  fromEvent,
  merge,
} from "rxjs";
import {
  takeUntil,
  finalize,
  tap,
  share,
  shareReplay,
  switchMap,
  exhaustMap,
  debounceTime,
  distinctUntilChanged,
  filter,
  map,
  scan,
  startWith,
  combineLatest,
} from "rxjs/operators";
import { Injectable, OnDestroy, inject, NgZone } from "@angular/core";

export interface MemoryTracker {
  subscriptions: Set<string>;
  observers: Map<string, number>;
  totalMemoryUsage: number;
  activeStreams: number;
}

export interface LeakDetectionConfig {
  maxSubscriptions: number;
  maxObservers: number;
  memoryThreshold: number;
  checkInterval: number;
  onLeakDetected?: (leak: MemoryLeak) => void;
}

export interface MemoryLeak {
  type: "subscription" | "observer" | "memory";
  count: number;
  threshold: number;
  streamId?: string;
  timestamp: Date;
}

// Memory-safe subscription operator
export function safeSubscription<T>(streamId: string) {
  return (source: Observable<T>) => {
    return new Observable<T>((subscriber) => {
      const subscriptionId = `${streamId}_${Date.now()}_${Math.random()}`;

      // Track subscription
      MemoryManager.getInstance().trackSubscription(subscriptionId);

      const subscription = source.subscribe({
        next: (value) => {
          try {
            subscriber.next(value);
          } catch (error) {
            console.error(`Error in stream ${streamId}:`, error);
            subscriber.error(error);
          }
        },
        error: (error) => {
          MemoryManager.getInstance().untrackSubscription(subscriptionId);
          subscriber.error(error);
        },
        complete: () => {
          MemoryManager.getInstance().untrackSubscription(subscriptionId);
          subscriber.complete();
        },
      });

      // Return cleanup function
      return () => {
        MemoryManager.getInstance().untrackSubscription(subscriptionId);
        subscription.unsubscribe();
      };
    });
  };
}

// Auto-cleanup operator for long-running streams
export function autoCleanup<T>(
  maxLifetime: number = 300000, // 5 minutes default
  inactivityTimeout: number = 60000 // 1 minute default
) {
  return (source: Observable<T>) => {
    let lastActivity = Date.now();
    let isActive = false;

    return new Observable<T>((subscriber) => {
      const startTime = Date.now();

      // Lifetime timer
      const lifetimeTimer = setTimeout(() => {
        console.log("Stream auto-cleanup: Maximum lifetime reached");
        subscriber.complete();
      }, maxLifetime);

      // Inactivity timer
      let inactivityTimer: NodeJS.Timeout;

      const resetInactivityTimer = () => {
        if (inactivityTimer) {
          clearTimeout(inactivityTimer);
        }

        inactivityTimer = setTimeout(() => {
          if (!isActive) {
            console.log("Stream auto-cleanup: Inactivity timeout reached");
            subscriber.complete();
          }
        }, inactivityTimeout);
      };

      resetInactivityTimer();

      const subscription = source.subscribe({
        next: (value) => {
          lastActivity = Date.now();
          isActive = true;
          resetInactivityTimer();
          subscriber.next(value);

          // Reset active flag after processing
          setTimeout(() => {
            isActive = false;
          }, 100);
        },
        error: (error) => {
          clearTimeout(lifetimeTimer);
          clearTimeout(inactivityTimer);
          subscriber.error(error);
        },
        complete: () => {
          clearTimeout(lifetimeTimer);
          clearTimeout(inactivityTimer);
          subscriber.complete();
        },
      });

      return () => {
        clearTimeout(lifetimeTimer);
        clearTimeout(inactivityTimer);
        subscription.unsubscribe();
      };
    });
  };
}

// Bounded replay operator
export function boundedShareReplay<T>(
  bufferSize: number = 1,
  windowTime?: number,
  scheduler?: any
) {
  return (source: Observable<T>) => {
    return source.pipe(
      shareReplay({
        bufferSize,
        windowTime,
        refCount: true, // Important: automatically cleanup when no subscribers
      })
    );
  };
}

// Memory-conscious buffer operator
export function memoryAwareBuffer<T>(
  maxBufferSize: number = 1000,
  maxMemoryMB: number = 50
) {
  return (source: Observable<T>) => {
    let buffer: T[] = [];
    let currentMemoryUsage = 0;

    return new Observable<T[]>((subscriber) => {
      const subscription = source.subscribe({
        next: (value) => {
          // Estimate memory usage (rough calculation)
          const itemSize = JSON.stringify(value).length * 2; // chars to bytes
          currentMemoryUsage += itemSize;

          buffer.push(value);

          // Check memory limits
          if (
            buffer.length >= maxBufferSize ||
            currentMemoryUsage >= maxMemoryMB * 1024 * 1024
          ) {
            subscriber.next([...buffer]);

            // Clear buffer and reset memory tracking
            buffer = [];
            currentMemoryUsage = 0;
          }
        },
        error: (error) => subscriber.error(error),
        complete: () => {
          if (buffer.length > 0) {
            subscriber.next([...buffer]);
          }
          subscriber.complete();
        },
      });

      return () => {
        buffer = [];
        currentMemoryUsage = 0;
        subscription.unsubscribe();
      };
    });
  };
}

// Singleton Memory Manager
export class MemoryManager {
  private static instance: MemoryManager;
  private tracker: MemoryTracker = {
    subscriptions: new Set(),
    observers: new Map(),
    totalMemoryUsage: 0,
    activeStreams: 0,
  };

  private leakDetectionConfig: LeakDetectionConfig = {
    maxSubscriptions: 1000,
    maxObservers: 100,
    memoryThreshold: 100 * 1024 * 1024, // 100MB
    checkInterval: 30000, // 30 seconds
  };

  private monitoringInterval?: NodeJS.Timeout;

  private constructor() {
    this.startMonitoring();
  }

  static getInstance(): MemoryManager {
    if (!MemoryManager.instance) {
      MemoryManager.instance = new MemoryManager();
    }
    return MemoryManager.instance;
  }

  trackSubscription(id: string): void {
    this.tracker.subscriptions.add(id);
    this.updateMemoryUsage();
  }

  untrackSubscription(id: string): void {
    this.tracker.subscriptions.delete(id);
    this.updateMemoryUsage();
  }

  trackObserver(streamId: string): void {
    const count = this.tracker.observers.get(streamId) || 0;
    this.tracker.observers.set(streamId, count + 1);
    this.updateMemoryUsage();
  }

  untrackObserver(streamId: string): void {
    const count = this.tracker.observers.get(streamId) || 0;
    if (count <= 1) {
      this.tracker.observers.delete(streamId);
    } else {
      this.tracker.observers.set(streamId, count - 1);
    }
    this.updateMemoryUsage();
  }

  getMemoryStats(): MemoryTracker {
    return { ...this.tracker };
  }

  configureLeakDetection(config: Partial<LeakDetectionConfig>): void {
    this.leakDetectionConfig = { ...this.leakDetectionConfig, ...config };
  }

  private updateMemoryUsage(): void {
    // Estimate memory usage
    this.tracker.totalMemoryUsage =
      this.tracker.subscriptions.size * 1024 + // Rough estimate per subscription
      Array.from(this.tracker.observers.values()).reduce(
        (sum, count) => sum + count * 512,
        0
      );

    this.tracker.activeStreams = this.tracker.observers.size;
  }

  private startMonitoring(): void {
    this.monitoringInterval = setInterval(() => {
      this.checkForLeaks();
    }, this.leakDetectionConfig.checkInterval);
  }

  private checkForLeaks(): void {
    const config = this.leakDetectionConfig;

    // Check subscription leaks
    if (this.tracker.subscriptions.size > config.maxSubscriptions) {
      const leak: MemoryLeak = {
        type: "subscription",
        count: this.tracker.subscriptions.size,
        threshold: config.maxSubscriptions,
        timestamp: new Date(),
      };

      console.warn("Memory leak detected: Too many subscriptions", leak);
      config.onLeakDetected?.(leak);
    }

    // Check observer leaks
    const totalObservers = Array.from(this.tracker.observers.values()).reduce(
      (sum, count) => sum + count,
      0
    );

    if (totalObservers > config.maxObservers) {
      const leak: MemoryLeak = {
        type: "observer",
        count: totalObservers,
        threshold: config.maxObservers,
        timestamp: new Date(),
      };

      console.warn("Memory leak detected: Too many observers", leak);
      config.onLeakDetected?.(leak);
    }

    // Check memory usage
    if (this.tracker.totalMemoryUsage > config.memoryThreshold) {
      const leak: MemoryLeak = {
        type: "memory",
        count: this.tracker.totalMemoryUsage,
        threshold: config.memoryThreshold,
        timestamp: new Date(),
      };

      console.warn("Memory leak detected: High memory usage", leak);
      config.onLeakDetected?.(leak);
    }
  }

  cleanup(): void {
    if (this.monitoringInterval) {
      clearInterval(this.monitoringInterval);
    }
    this.tracker.subscriptions.clear();
    this.tracker.observers.clear();
    this.tracker.totalMemoryUsage = 0;
    this.tracker.activeStreams = 0;
  }
}
```

### **4. 🌐 Advanced WebSocket Integration**

```typescript
// src/app/shared/services/advanced-websocket.service.ts
import { Injectable, inject, NgZone } from "@angular/core";
import {
  Observable,
  Subject,
  BehaviorSubject,
  timer,
  fromEvent,
  merge,
  NEVER,
  throwError,
} from "rxjs";
import {
  switchMap,
  retry,
  retryWhen,
  delay,
  tap,
  filter,
  map,
  share,
  takeUntil,
  catchError,
  startWith,
  scan,
  distinctUntilChanged,
  debounceTime,
} from "rxjs/operators";
import { retryWithBackoff } from "../operators/error-recovery.operators";

export interface WebSocketConfig {
  url: string;
  protocols?: string[];
  heartbeatInterval?: number;
  reconnectInterval?: number;
  maxReconnectAttempts?: number;
  messageBuffer?: boolean;
  compression?: boolean;
  auth?: {
    token?: string;
    method?: "header" | "query" | "message";
  };
}

export interface WebSocketMessage {
  id?: string;
  type: string;
  payload: any;
  timestamp: number;
  priority?: "high" | "medium" | "low";
}

export interface ConnectionState {
  status:
    | "connecting"
    | "connected"
    | "disconnected"
    | "reconnecting"
    | "error";
  lastConnected?: Date;
  reconnectAttempts: number;
  latency?: number;
  messagesSent: number;
  messagesReceived: number;
}

@Injectable({
  providedIn: "root",
})
export class AdvancedWebSocketService {
  private ngZone = inject(NgZone);

  private connections = new Map<
    string,
    {
      socket: WebSocket | null;
      config: WebSocketConfig;
      state$: BehaviorSubject<ConnectionState>;
      messages$: Subject<WebSocketMessage>;
      messageBuffer: WebSocketMessage[];
      heartbeatInterval?: NodeJS.Timeout;
      reconnectTimeout?: NodeJS.Timeout;
    }
  >();

  // Create and manage WebSocket connection
  connect(
    connectionId: string,
    config: WebSocketConfig
  ): Observable<WebSocketMessage> {
    if (this.connections.has(connectionId)) {
      const connection = this.connections.get(connectionId)!;
      return connection.messages$.asObservable();
    }

    const state$ = new BehaviorSubject<ConnectionState>({
      status: "connecting",
      reconnectAttempts: 0,
      messagesSent: 0,
      messagesReceived: 0,
    });

    const messages$ = new Subject<WebSocketMessage>();
    const messageBuffer: WebSocketMessage[] = [];

    const connection = {
      socket: null,
      config,
      state$,
      messages$,
      messageBuffer,
    };

    this.connections.set(connectionId, connection);

    // Start connection process
    this.establishConnection(connectionId);

    return messages$.asObservable().pipe(share());
  }

  // Send message through WebSocket
  send(
    connectionId: string,
    message: Omit<WebSocketMessage, "timestamp">
  ): Observable<boolean> {
    return new Observable((subscriber) => {
      const connection = this.connections.get(connectionId);

      if (!connection) {
        subscriber.error(new Error(`Connection ${connectionId} not found`));
        return;
      }

      const fullMessage: WebSocketMessage = {
        ...message,
        id: message.id || this.generateMessageId(),
        timestamp: Date.now(),
      };

      if (connection.socket?.readyState === WebSocket.OPEN) {
        try {
          this.sendMessage(connectionId, fullMessage);
          subscriber.next(true);
          subscriber.complete();
        } catch (error) {
          subscriber.error(error);
        }
      } else if (connection.config.messageBuffer) {
        // Buffer message for later sending
        connection.messageBuffer.push(fullMessage);
        subscriber.next(false); // Indicates message was buffered
        subscriber.complete();
      } else {
        subscriber.error(
          new Error("Connection not open and buffering disabled")
        );
      }
    });
  }

  // Get connection state
  getConnectionState(connectionId: string): Observable<ConnectionState> {
    const connection = this.connections.get(connectionId);
    if (!connection) {
      return throwError(
        () => new Error(`Connection ${connectionId} not found`)
      );
    }
    return connection.state$.asObservable();
  }

  // Disconnect and cleanup
  disconnect(connectionId: string): void {
    const connection = this.connections.get(connectionId);
    if (!connection) return;

    // Clean up timers
    if (connection.heartbeatInterval) {
      clearInterval(connection.heartbeatInterval);
    }
    if (connection.reconnectTimeout) {
      clearTimeout(connection.reconnectTimeout);
    }

    // Close socket
    if (connection.socket) {
      connection.socket.close(1000, "Normal closure");
    }

    // Complete subjects
    connection.messages$.complete();
    connection.state$.complete();

    // Remove from connections
    this.connections.delete(connectionId);
  }

  // Get all active connections
  getActiveConnections(): string[] {
    return Array.from(this.connections.keys()).filter((id) => {
      const connection = this.connections.get(id);
      return connection?.socket?.readyState === WebSocket.OPEN;
    });
  }

  // Broadcast message to multiple connections
  broadcast(
    connectionIds: string[],
    message: Omit<WebSocketMessage, "timestamp">
  ): Observable<{ [connectionId: string]: boolean }> {
    const results: { [connectionId: string]: boolean } = {};

    return new Observable((subscriber) => {
      const sendPromises = connectionIds.map((id) =>
        this.send(id, message)
          .toPromise()
          .then(
            (result) => {
              results[id] = result || false;
            },
            (error) => {
              results[id] = false;
            }
          )
      );

      Promise.all(sendPromises).then(() => {
        subscriber.next(results);
        subscriber.complete();
      });
    });
  }

  // Private implementation methods
  private establishConnection(connectionId: string): void {
    const connection = this.connections.get(connectionId);
    if (!connection) return;

    const config = connection.config;

    this.ngZone.runOutsideAngular(() => {
      try {
        // Add auth to URL if specified
        let url = config.url;
        if (config.auth?.method === "query" && config.auth.token) {
          const separator = url.includes("?") ? "&" : "?";
          url += `${separator}token=${config.auth.token}`;
        }

        const socket = new WebSocket(url, config.protocols);
        connection.socket = socket;

        // Set up event handlers
        socket.onopen = () => this.onOpen(connectionId);
        socket.onclose = (event) => this.onClose(connectionId, event);
        socket.onerror = (error) => this.onError(connectionId, error);
        socket.onmessage = (event) => this.onMessage(connectionId, event);

        // Update connection state
        connection.state$.next({
          ...connection.state$.value,
          status: "connecting",
        });
      } catch (error) {
        this.onError(connectionId, error);
      }
    });
  }

  private onOpen(connectionId: string): void {
    const connection = this.connections.get(connectionId);
    if (!connection) return;

    console.log(`WebSocket connected: ${connectionId}`);

    // Update state
    connection.state$.next({
      ...connection.state$.value,
      status: "connected",
      lastConnected: new Date(),
      reconnectAttempts: 0,
    });

    // Send auth message if required
    if (
      connection.config.auth?.method === "message" &&
      connection.config.auth.token
    ) {
      this.sendMessage(connectionId, {
        id: this.generateMessageId(),
        type: "auth",
        payload: { token: connection.config.auth.token },
        timestamp: Date.now(),
      });
    }

    // Send buffered messages
    if (connection.messageBuffer.length > 0) {
      connection.messageBuffer.forEach((message) => {
        this.sendMessage(connectionId, message);
      });
      connection.messageBuffer.length = 0; // Clear buffer
    }

    // Start heartbeat
    this.startHeartbeat(connectionId);
  }

  private onClose(connectionId: string, event: CloseEvent): void {
    const connection = this.connections.get(connectionId);
    if (!connection) return;

    console.log(`WebSocket closed: ${connectionId}`, event.code, event.reason);

    // Stop heartbeat
    if (connection.heartbeatInterval) {
      clearInterval(connection.heartbeatInterval);
    }

    // Update state
    const currentState = connection.state$.value;

    if (
      event.code !== 1000 &&
      currentState.reconnectAttempts <
        (connection.config.maxReconnectAttempts || 5)
    ) {
      // Attempt reconnection
      connection.state$.next({
        ...currentState,
        status: "reconnecting",
        reconnectAttempts: currentState.reconnectAttempts + 1,
      });

      const reconnectDelay = Math.min(
        1000 * Math.pow(2, currentState.reconnectAttempts),
        30000 // Max 30 seconds
      );

      connection.reconnectTimeout = setTimeout(() => {
        this.establishConnection(connectionId);
      }, reconnectDelay);
    } else {
      // Final disconnection
      connection.state$.next({
        ...currentState,
        status: "disconnected",
      });
    }
  }

  private onError(connectionId: string, error: any): void {
    const connection = this.connections.get(connectionId);
    if (!connection) return;

    console.error(`WebSocket error: ${connectionId}`, error);

    connection.state$.next({
      ...connection.state$.value,
      status: "error",
    });
  }

  private onMessage(connectionId: string, event: MessageEvent): void {
    const connection = this.connections.get(connectionId);
    if (!connection) return;

    try {
      let message: WebSocketMessage;

      if (typeof event.data === "string") {
        message = JSON.parse(event.data);
      } else {
        // Handle binary data if needed
        message = {
          id: this.generateMessageId(),
          type: "binary",
          payload: event.data,
          timestamp: Date.now(),
        };
      }

      // Calculate latency if this is a pong message
      if (message.type === "pong" && message.payload?.timestamp) {
        const latency = Date.now() - message.payload.timestamp;
        connection.state$.next({
          ...connection.state$.value,
          latency,
        });
      }

      // Update message count
      const currentState = connection.state$.value;
      connection.state$.next({
        ...currentState,
        messagesReceived: currentState.messagesReceived + 1,
      });

      // Emit message
      this.ngZone.run(() => {
        connection.messages$.next(message);
      });
    } catch (error) {
      console.error(
        `Failed to parse WebSocket message: ${connectionId}`,
        error
      );
    }
  }

  private sendMessage(connectionId: string, message: WebSocketMessage): void {
    const connection = this.connections.get(connectionId);
    if (!connection || !connection.socket) return;

    try {
      const data = JSON.stringify(message);
      connection.socket.send(data);

      // Update sent count
      const currentState = connection.state$.value;
      connection.state$.next({
        ...currentState,
        messagesSent: currentState.messagesSent + 1,
      });
    } catch (error) {
      console.error(`Failed to send WebSocket message: ${connectionId}`, error);
      throw error;
    }
  }

  private startHeartbeat(connectionId: string): void {
    const connection = this.connections.get(connectionId);
    if (!connection || !connection.config.heartbeatInterval) return;

    connection.heartbeatInterval = setInterval(() => {
      if (connection.socket?.readyState === WebSocket.OPEN) {
        this.sendMessage(connectionId, {
          id: this.generateMessageId(),
          type: "ping",
          payload: { timestamp: Date.now() },
          timestamp: Date.now(),
        });
      }
    }, connection.config.heartbeatInterval);
  }

  private generateMessageId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### **5. 🧪 Advanced Testing Patterns**

```typescript
// src/app/shared/testing/rxjs-test-helpers.ts
import { TestScheduler, VirtualTimeScheduler } from "rxjs/testing";
import { Observable, Subject, BehaviorSubject, timer } from "rxjs";
import { TestBed } from "@angular/core/testing";
import { cold, hot, getTestScheduler } from "jasmine-marbles";

export interface StreamTestCase<T> {
  name: string;
  input: string;
  expected: string;
  values?: { [key: string]: T };
  error?: any;
}

export interface AsyncTestContext {
  scheduler: TestScheduler;
  expectObservable: (observable: Observable<any>) => any;
  expectSubscriptions: (subscriptions: string) => any;
  flush: () => void;
  hot: (marbles: string, values?: any, error?: any) => Observable<any>;
  cold: (marbles: string, values?: any, error?: any) => Observable<any>;
}

// Advanced RxJS Testing Utility
export class RxJSTestRunner {
  private scheduler!: TestScheduler;

  createTestContext(): AsyncTestContext {
    this.scheduler = new TestScheduler((actual, expected) => {
      expect(actual).toEqual(expected);
    });

    return {
      scheduler: this.scheduler,
      expectObservable: (obs) => this.scheduler.expectObservable(obs),
      expectSubscriptions: (subs) => this.scheduler.expectSubscriptions(subs),
      flush: () => this.scheduler.flush(),
      hot: (marbles, values, error) =>
        this.scheduler.createHotObservable(marbles, values, error),
      cold: (marbles, values, error) =>
        this.scheduler.createColdObservable(marbles, values, error),
    };
  }

  // Test custom operators
  testOperator<T, R>(
    operator: (source: Observable<T>) => Observable<R>,
    testCases: StreamTestCase<T | R>[]
  ): void {
    testCases.forEach((testCase) => {
      it(testCase.name, () => {
        const context = this.createTestContext();

        const source$ = context.cold(testCase.input, testCase.values);
        const result$ = source$.pipe(operator);

        if (testCase.error) {
          context
            .expectObservable(result$)
            .toBe(testCase.expected, testCase.values, testCase.error);
        } else {
          context
            .expectObservable(result$)
            .toBe(testCase.expected, testCase.values);
        }

        context.flush();
      });
    });
  }

  // Test stream orchestration
  testStreamOrchestration(
    orchestratorFn: (streams: {
      [key: string]: Observable<any>;
    }) => Observable<any>,
    inputStreams: { [key: string]: string },
    expectedOutput: string,
    values?: { [key: string]: any }
  ): void {
    const context = this.createTestContext();

    const streams: { [key: string]: Observable<any> } = {};
    Object.entries(inputStreams).forEach(([key, marbles]) => {
      streams[key] = context.cold(marbles, values);
    });

    const result$ = orchestratorFn(streams);

    context.expectObservable(result$).toBe(expectedOutput, values);
    context.flush();
  }

  // Test memory leaks
  testMemoryLeaks(
    streamFactory: () => Observable<any>,
    subscriptionCount: number = 1000
  ): Promise<boolean> {
    return new Promise((resolve) => {
      const subscriptions: any[] = [];
      const initialMemory = this.getMemoryUsage();

      // Create many subscriptions
      for (let i = 0; i < subscriptionCount; i++) {
        const subscription = streamFactory().subscribe();
        subscriptions.push(subscription);
      }

      const peakMemory = this.getMemoryUsage();

      // Clean up subscriptions
      subscriptions.forEach((sub) => sub.unsubscribe());

      // Force garbage collection if available
      if (global.gc) {
        global.gc();
      }

      setTimeout(() => {
        const finalMemory = this.getMemoryUsage();
        const memoryLeak =
          finalMemory > initialMemory + (peakMemory - initialMemory) * 0.1;

        console.log("Memory Test Results:", {
          initial: initialMemory,
          peak: peakMemory,
          final: finalMemory,
          leakDetected: memoryLeak,
        });

        resolve(!memoryLeak);
      }, 100);
    });
  }

  // Performance testing
  testPerformance(
    streamFactory: () => Observable<any>,
    expectedMaxTime: number = 100
  ): Promise<number> {
    return new Promise((resolve, reject) => {
      const startTime = performance.now();
      let subscriptionCount = 0;

      const subscription = streamFactory().subscribe({
        next: () => {
          subscriptionCount++;
        },
        complete: () => {
          const endTime = performance.now();
          const duration = endTime - startTime;

          console.log(
            `Performance Test: ${duration}ms for ${subscriptionCount} emissions`
          );

          if (duration <= expectedMaxTime) {
            resolve(duration);
          } else {
            reject(
              new Error(
                `Performance test failed: ${duration}ms > ${expectedMaxTime}ms`
              )
            );
          }
        },
        error: reject,
      });

      // Cleanup after timeout
      setTimeout(() => {
        subscription.unsubscribe();
        reject(new Error("Performance test timed out"));
      }, expectedMaxTime * 10);
    });
  }

  private getMemoryUsage(): number {
    if (typeof performance !== "undefined" && (performance as any).memory) {
      return (performance as any).memory.usedJSHeapSize;
    }
    return 0;
  }
}

// Marble testing helpers
export const MarbleHelpers = {
  // Create marble patterns for common scenarios
  createDelayedEmission: (delay: number, value: string = "a"): string => {
    return "-".repeat(delay) + value + "|";
  },

  createRepeatedEmissions: (
    count: number,
    interval: number = 1,
    value: string = "a"
  ): string => {
    return Array(count).fill(value).join("-".repeat(interval)) + "|";
  },

  createErrorPattern: (delay: number): string => {
    return "-".repeat(delay) + "#";
  },

  createInfiniteStream: (interval: number = 1, value: string = "a"): string => {
    return (value + "-".repeat(interval)).repeat(100);
  },

  // Complex patterns
  createBackpressurePattern: (): string => {
    return "a-a-a-aaa-a-a-|"; // Burst followed by normal rate
  },

  createJitterPattern: (): string => {
    return "a--a-a---a-a-|"; // Irregular timing
  },
};

// Example test implementation
describe("Advanced RxJS Patterns", () => {
  let testRunner: RxJSTestRunner;

  beforeEach(() => {
    testRunner = new RxJSTestRunner();
  });

  describe("retryWithBackoff operator", () => {
    testRunner.testOperator(
      (source$) =>
        source$.pipe(
          retryWithBackoff({
            maxRetries: 2,
            baseDelay: 10,
            maxDelay: 100,
            exponentialBackoff: true,
            jitter: false,
          })
        ),
      [
        {
          name: "should retry on error with exponential backoff",
          input: "--#",
          expected: "-- 10ms --(--#)",
          error: new Error("Test error"),
        },
        {
          name: "should emit successful values",
          input: "--a--b--|",
          expected: "--a--b--|",
          values: { a: 1, b: 2 },
        },
      ]
    );
  });

  describe("Memory leak testing", () => {
    it("should not leak memory with proper cleanup", async () => {
      const noMemoryLeak = await testRunner.testMemoryLeaks(
        () => timer(0, 1).pipe(take(100)),
        100
      );

      expect(noMemoryLeak).toBe(true);
    });
  });

  describe("Performance testing", () => {
    it("should process 1000 items within 100ms", async () => {
      const duration = await testRunner.testPerformance(
        () => range(0, 1000),
        100
      );

      expect(duration).toBeLessThan(100);
    });
  });
});
```

## 🎯 **Complete Implementation Summary**

This comprehensive Advanced RxJS Patterns guide covers:

### **🛠️ Production-Ready Features**

- **Error Recovery** with exponential backoff, circuit breakers, and intelligent fallback strategies
- **Backpressure Management** with adaptive batching, rate limiting, and load balancing
- **Memory Management** with leak detection, auto-cleanup, and resource tracking
- **WebSocket Integration** with reconnection logic, heartbeat monitoring, and message buffering

### **⚡ Performance Excellence**

- **Smart Caching** with TTL, LRU/FIFO/LFU eviction strategies
- **Stream Orchestration** with dependency resolution and priority-based merging
- **Resource Optimization** with bounded replay and memory-conscious operators
- **Advanced Testing** with marble diagrams, performance benchmarks, and leak detection

### **🚀 Enterprise Patterns**

- **Fan-out/Fan-in** patterns for complex data distribution
- **Pipeline Architecture** with stage-based processing and error handling
- **Connection Management** with health monitoring and automatic recovery
- **Metrics Collection** with real-time performance tracking

## 💡 **Key Implementation Insights**

### **1. Error Recovery Strategy**

The retry operators implement **exponential backoff with jitter** to prevent thundering herd problems while **circuit breakers provide fast failure** during sustained outages, ensuring **optimal user experience** and **system stability**.

### **2. Memory Management Architecture**

**Automatic cleanup operators** combined with **leak detection monitoring** prevent memory accumulation in long-running applications, while **bounded operators** ensure **predictable resource usage** even under heavy load.

### **3. WebSocket Resilience**

**Smart reconnection logic** with **message buffering** ensures **seamless user experience** during network interruptions, while **heartbeat monitoring** provides **real-time connection health** information.

### **4. Testing Excellence**

**Marble testing** combined with **performance benchmarks** and **memory leak detection** ensures **production-ready reliability** and **optimal performance** characteristics.

This enterprise-grade RxJS implementation provides **robust data flow management**, **exceptional error handling**, **memory safety**, and **production-ready resilience** for **world-class Angular applications**! 🎯✨
