# 🧩 **Advanced Angular Coding Challenges & Algorithms - Part 3**

## 🌳 **Tree & Graph Algorithms for Hierarchical Data**

### **Challenge 6: 🏗️ Advanced Tree Data Structure with Lazy Loading**

**Problem:** Implement a sophisticated tree component that handles millions of nodes with lazy loading, virtual scrolling, drag-and-drop, and real-time updates.

```typescript
// src/app/shared/components/advanced-tree/tree-data.service.ts
import { Injectable } from "@angular/core";
import { BehaviorSubject, Observable, Subject, merge } from "rxjs";
import {
  map,
  filter,
  debounceTime,
  distinctUntilChanged,
  switchMap,
  catchError,
  shareReplay,
  tap,
} from "rxjs/operators";

export interface TreeNode<T = any> {
  id: string;
  parentId: string | null;
  data: T;
  children?: TreeNode<T>[];
  hasChildren: boolean;
  isLoading: boolean;
  isExpanded: boolean;
  isSelected: boolean;
  level: number;
  path: string;
  isVisible: boolean;
  checkboxState: "checked" | "unchecked" | "indeterminate";
  metadata: {
    childCount: number;
    loadedChildCount: number;
    lastLoaded?: Date;
    isVirtual: boolean;
    estimatedHeight: number;
    actualHeight?: number;
    position: number;
  };
}

export interface TreeViewState {
  expandedNodes: Set<string>;
  selectedNodes: Set<string>;
  checkedNodes: Set<string>;
  loadedNodes: Set<string>;
  searchQuery: string;
  filteredNodes: Set<string>;
  sortConfig?: {
    field: string;
    direction: "asc" | "desc";
  };
}

export interface TreeOperationResult {
  success: boolean;
  affectedNodes: string[];
  error?: string;
  timestamp: Date;
}

export interface DragDropOperation {
  draggedNodeId: string;
  targetNodeId: string;
  operation: "move-into" | "move-before" | "move-after";
  originalParentId: string;
  newParentId: string;
}

@Injectable()
export class TreeDataService<T = any> {
  private nodes$ = new BehaviorSubject<Map<string, TreeNode<T>>>(new Map());
  private viewState$ = new BehaviorSubject<TreeViewState>({
    expandedNodes: new Set(),
    selectedNodes: new Set(),
    checkedNodes: new Set(),
    loadedNodes: new Set(),
    searchQuery: "",
    filteredNodes: new Set(),
  });

  private nodeCache = new Map<string, TreeNode<T>>();
  private childrenCache = new Map<string, TreeNode<T>[]>();
  private searchIndex = new Map<string, Set<string>>(); // searchTerm -> nodeIds
  private operationQueue: Array<() => Promise<void>> = [];
  private isProcessingOperations = false;

  private readonly BATCH_SIZE = 100;
  private readonly VIRTUAL_THRESHOLD = 1000;

  // 🌳 INITIALIZE TREE
  initializeTree(
    rootNodes: TreeNode<T>[] | (() => Observable<TreeNode<T>[]>),
    options: {
      enableVirtualScrolling?: boolean;
      enableSearch?: boolean;
      enableDragDrop?: boolean;
      enableCheckboxes?: boolean;
      lazyLoadThreshold?: number;
    } = {}
  ): Observable<TreeNode<T>[]> {
    console.log("🌳 Initializing advanced tree...");

    // Load initial nodes
    const nodes$ =
      typeof rootNodes === "function"
        ? rootNodes()
        : new BehaviorSubject(rootNodes).asObservable();

    return nodes$.pipe(
      tap((nodes) => this.processInitialNodes(nodes, options)),
      switchMap(() => this.getVisibleNodes()),
      shareReplay(1)
    );
  }

  // 📂 EXPAND NODE
  async expandNode(
    nodeId: string,
    loadChildren = true
  ): Promise<TreeOperationResult> {
    return this.queueOperation(async () => {
      const node = this.nodeCache.get(nodeId);
      if (!node) {
        throw new Error(`Node ${nodeId} not found`);
      }

      const viewState = this.viewState$.value;
      viewState.expandedNodes.add(nodeId);

      node.isExpanded = true;

      if (
        loadChildren &&
        node.hasChildren &&
        !viewState.loadedNodes.has(nodeId)
      ) {
        await this.loadChildren(nodeId);
        viewState.loadedNodes.add(nodeId);
      }

      this.updateNodeVisibility();
      this.viewState$.next(viewState);
      this.emitNodesUpdate();

      return {
        success: true,
        affectedNodes: [nodeId],
        timestamp: new Date(),
      };
    });
  }

  // 📁 COLLAPSE NODE
  async collapseNode(nodeId: string): Promise<TreeOperationResult> {
    return this.queueOperation(async () => {
      const node = this.nodeCache.get(nodeId);
      if (!node) {
        throw new Error(`Node ${nodeId} not found`);
      }

      const viewState = this.viewState$.value;
      viewState.expandedNodes.delete(nodeId);

      node.isExpanded = false;

      // Hide all descendant nodes
      const descendants = this.getDescendants(nodeId);
      descendants.forEach((descendant) => {
        descendant.isVisible = false;
      });

      this.updateNodeVisibility();
      this.viewState$.next(viewState);
      this.emitNodesUpdate();

      return {
        success: true,
        affectedNodes: [nodeId, ...descendants.map((n) => n.id)],
        timestamp: new Date(),
      };
    });
  }

  // 🎯 SELECT NODE
  async selectNode(
    nodeId: string,
    multiSelect = false
  ): Promise<TreeOperationResult> {
    return this.queueOperation(async () => {
      const viewState = this.viewState$.value;

      if (!multiSelect) {
        // Clear previous selections
        viewState.selectedNodes.clear();
        this.nodeCache.forEach((node) => (node.isSelected = false));
      }

      const node = this.nodeCache.get(nodeId);
      if (node) {
        node.isSelected = true;
        viewState.selectedNodes.add(nodeId);
      }

      this.viewState$.next(viewState);
      this.emitNodesUpdate();

      return {
        success: true,
        affectedNodes: [nodeId],
        timestamp: new Date(),
      };
    });
  }

  // ☑️ CHECK NODE (with smart propagation)
  async checkNode(
    nodeId: string,
    checked: boolean
  ): Promise<TreeOperationResult> {
    return this.queueOperation(async () => {
      const affectedNodes: string[] = [];

      // Update the node itself
      const node = this.nodeCache.get(nodeId);
      if (node) {
        node.checkboxState = checked ? "checked" : "unchecked";
        affectedNodes.push(nodeId);

        const viewState = this.viewState$.value;
        if (checked) {
          viewState.checkedNodes.add(nodeId);
        } else {
          viewState.checkedNodes.delete(nodeId);
        }

        // Propagate down to children
        await this.propagateCheckStateDown(nodeId, checked, affectedNodes);

        // Propagate up to parents
        await this.propagateCheckStateUp(nodeId, affectedNodes);

        this.viewState$.next(viewState);
      }

      this.emitNodesUpdate();

      return {
        success: true,
        affectedNodes,
        timestamp: new Date(),
      };
    });
  }

  // 🔍 SEARCH NODES
  async searchNodes(query: string): Promise<TreeOperationResult> {
    return this.queueOperation(async () => {
      const viewState = this.viewState$.value;
      viewState.searchQuery = query;
      viewState.filteredNodes.clear();

      if (!query.trim()) {
        // Show all nodes when search is cleared
        this.nodeCache.forEach((node) => {
          node.isVisible = this.isNodeVisible(node);
        });
      } else {
        // Perform search
        const matchingNodeIds = await this.performSearch(query);
        matchingNodeIds.forEach((nodeId) =>
          viewState.filteredNodes.add(nodeId)
        );

        // Update visibility based on search results
        this.updateNodeVisibilityForSearch(matchingNodeIds);
      }

      this.viewState$.next(viewState);
      this.emitNodesUpdate();

      return {
        success: true,
        affectedNodes: Array.from(viewState.filteredNodes),
        timestamp: new Date(),
      };
    });
  }

  // 🔀 MOVE NODE (drag and drop)
  async moveNode(operation: DragDropOperation): Promise<TreeOperationResult> {
    return this.queueOperation(async () => {
      const { draggedNodeId, targetNodeId, operation: moveType } = operation;

      const draggedNode = this.nodeCache.get(draggedNodeId);
      const targetNode = this.nodeCache.get(targetNodeId);

      if (!draggedNode || !targetNode) {
        throw new Error("Invalid drag and drop operation");
      }

      // Validate move operation
      if (!this.isValidMove(draggedNode, targetNode, moveType)) {
        throw new Error("Invalid move: would create circular reference");
      }

      const affectedNodes = await this.performNodeMove(
        draggedNode,
        targetNode,
        moveType
      );

      // Update tree structure
      this.rebuildTreeStructure();
      this.updateNodeVisibility();
      this.emitNodesUpdate();

      return {
        success: true,
        affectedNodes,
        timestamp: new Date(),
      };
    });
  }

  // 📊 GET TREE STATISTICS
  getTreeStatistics(): {
    totalNodes: number;
    visibleNodes: number;
    expandedNodes: number;
    selectedNodes: number;
    checkedNodes: number;
    loadedNodes: number;
    maxDepth: number;
    avgChildrenPerNode: number;
    memoryUsage: number;
  } {
    const viewState = this.viewState$.value;
    const visibleNodes = Array.from(this.nodeCache.values()).filter(
      (n) => n.isVisible
    );

    return {
      totalNodes: this.nodeCache.size,
      visibleNodes: visibleNodes.length,
      expandedNodes: viewState.expandedNodes.size,
      selectedNodes: viewState.selectedNodes.size,
      checkedNodes: viewState.checkedNodes.size,
      loadedNodes: viewState.loadedNodes.size,
      maxDepth: this.calculateMaxDepth(),
      avgChildrenPerNode: this.calculateAverageChildren(),
      memoryUsage: this.calculateMemoryUsage(),
    };
  }

  // 👁️ GET VISIBLE NODES (for virtual scrolling)
  getVisibleNodes(): Observable<TreeNode<T>[]> {
    return this.nodes$.pipe(
      map((nodesMap) => Array.from(nodesMap.values())),
      map((nodes) => nodes.filter((node) => node.isVisible)),
      map((nodes) => this.sortNodes(nodes)),
      distinctUntilChanged(
        (prev, curr) =>
          prev.length === curr.length &&
          prev.every((node, index) => node.id === curr[index]?.id)
      )
    );
  }

  // 🔄 Private Methods

  private processInitialNodes(nodes: TreeNode<T>[], options: any): void {
    // Build initial node cache and hierarchy
    nodes.forEach((node) => this.processNode(node, null, 0));

    // Build search index if search is enabled
    if (options.enableSearch) {
      this.buildSearchIndex();
    }

    // Initialize visibility
    this.updateNodeVisibility();

    console.log(`✅ Processed ${nodes.length} initial tree nodes`);
  }

  private processNode(
    node: TreeNode<T>,
    parent: TreeNode<T> | null,
    level: number
  ): void {
    // Enhance node with required properties
    const enhancedNode: TreeNode<T> = {
      ...node,
      level,
      path: parent ? `${parent.path}/${node.id}` : node.id,
      isVisible: level === 0, // Only root nodes visible initially
      isLoading: false,
      isExpanded: false,
      isSelected: false,
      checkboxState: "unchecked",
      metadata: {
        childCount: node.children?.length || 0,
        loadedChildCount: node.children?.length || 0,
        isVirtual: false,
        estimatedHeight: 32,
        position: 0,
        ...node.metadata,
      },
    };

    // Add to cache
    this.nodeCache.set(node.id, enhancedNode);

    // Process children recursively
    if (node.children) {
      node.children.forEach((child) =>
        this.processNode(child, enhancedNode, level + 1)
      );
    }
  }

  private async loadChildren(nodeId: string): Promise<TreeNode<T>[]> {
    const node = this.nodeCache.get(nodeId);
    if (!node) return [];

    // Check if children are already cached
    if (this.childrenCache.has(nodeId)) {
      return this.childrenCache.get(nodeId)!;
    }

    node.isLoading = true;
    this.emitNodesUpdate();

    try {
      // Simulate lazy loading (replace with actual API call)
      const children = await this.fetchChildrenFromAPI(nodeId);

      // Process and cache children
      children.forEach((child) =>
        this.processNode(child, node, node.level + 1)
      );
      this.childrenCache.set(nodeId, children);

      node.isLoading = false;
      node.metadata.loadedChildCount = children.length;
      node.metadata.lastLoaded = new Date();

      this.emitNodesUpdate();
      return children;
    } catch (error) {
      node.isLoading = false;
      console.error(`Failed to load children for node ${nodeId}:`, error);
      throw error;
    }
  }

  private async fetchChildrenFromAPI(nodeId: string): Promise<TreeNode<T>[]> {
    // Mock API call with delay
    await new Promise((resolve) => setTimeout(resolve, 500));

    // Return mock children
    return Array.from({ length: 10 }, (_, index) => ({
      id: `${nodeId}_child_${index}`,
      parentId: nodeId,
      data: { name: `Child ${index} of ${nodeId}` } as T,
      hasChildren: Math.random() > 0.7,
      isLoading: false,
      isExpanded: false,
      isSelected: false,
      level: 0, // Will be set by processNode
      path: "", // Will be set by processNode
      isVisible: false,
      checkboxState: "unchecked" as const,
      metadata: {
        childCount: 0,
        loadedChildCount: 0,
        isVirtual: false,
        estimatedHeight: 32,
        position: 0,
      },
    }));
  }

  private updateNodeVisibility(): void {
    this.nodeCache.forEach((node) => {
      node.isVisible = this.isNodeVisible(node);
    });
  }

  private isNodeVisible(node: TreeNode<T>): boolean {
    // Root nodes are always visible
    if (node.level === 0) {
      return true;
    }

    // Check if all ancestors are expanded
    let current = node;
    while (current.parentId) {
      const parent = this.nodeCache.get(current.parentId);
      if (!parent || !parent.isExpanded) {
        return false;
      }
      current = parent;
    }

    // If there's a search query, check if node matches
    const viewState = this.viewState$.value;
    if (viewState.searchQuery && viewState.filteredNodes.size > 0) {
      return viewState.filteredNodes.has(node.id);
    }

    return true;
  }

  private async propagateCheckStateDown(
    nodeId: string,
    checked: boolean,
    affectedNodes: string[]
  ): Promise<void> {
    const children = this.getDirectChildren(nodeId);

    for (const child of children) {
      if (child.checkboxState !== (checked ? "checked" : "unchecked")) {
        child.checkboxState = checked ? "checked" : "unchecked";
        affectedNodes.push(child.id);

        const viewState = this.viewState$.value;
        if (checked) {
          viewState.checkedNodes.add(child.id);
        } else {
          viewState.checkedNodes.delete(child.id);
        }

        // Recursively propagate to grandchildren
        await this.propagateCheckStateDown(child.id, checked, affectedNodes);
      }
    }
  }

  private async propagateCheckStateUp(
    nodeId: string,
    affectedNodes: string[]
  ): Promise<void> {
    const node = this.nodeCache.get(nodeId);
    if (!node || !node.parentId) return;

    const parent = this.nodeCache.get(node.parentId);
    if (!parent) return;

    const siblings = this.getDirectChildren(node.parentId);
    const checkedSiblings = siblings.filter(
      (s) => s.checkboxState === "checked"
    );
    const uncheckedSiblings = siblings.filter(
      (s) => s.checkboxState === "unchecked"
    );

    let newParentState: "checked" | "unchecked" | "indeterminate";

    if (checkedSiblings.length === siblings.length) {
      newParentState = "checked";
    } else if (uncheckedSiblings.length === siblings.length) {
      newParentState = "unchecked";
    } else {
      newParentState = "indeterminate";
    }

    if (parent.checkboxState !== newParentState) {
      parent.checkboxState = newParentState;
      affectedNodes.push(parent.id);

      const viewState = this.viewState$.value;
      if (newParentState === "checked") {
        viewState.checkedNodes.add(parent.id);
      } else {
        viewState.checkedNodes.delete(parent.id);
      }

      // Continue propagating up
      await this.propagateCheckStateUp(parent.id, affectedNodes);
    }
  }

  private getDirectChildren(nodeId: string): TreeNode<T>[] {
    return Array.from(this.nodeCache.values()).filter(
      (node) => node.parentId === nodeId
    );
  }

  private getDescendants(nodeId: string): TreeNode<T>[] {
    const descendants: TreeNode<T>[] = [];
    const children = this.getDirectChildren(nodeId);

    for (const child of children) {
      descendants.push(child);
      descendants.push(...this.getDescendants(child.id));
    }

    return descendants;
  }

  private buildSearchIndex(): void {
    this.searchIndex.clear();

    this.nodeCache.forEach((node) => {
      const searchableText = this.extractSearchableText(node);
      const words = searchableText.toLowerCase().split(/\s+/);

      words.forEach((word) => {
        if (!this.searchIndex.has(word)) {
          this.searchIndex.set(word, new Set());
        }
        this.searchIndex.get(word)!.add(node.id);
      });
    });
  }

  private extractSearchableText(node: TreeNode<T>): string {
    // Extract searchable text from node data
    const data = node.data as any;
    let searchText = "";

    if (typeof data === "string") {
      searchText = data;
    } else if (data && typeof data === "object") {
      // Common searchable properties
      const searchableProps = [
        "name",
        "title",
        "label",
        "description",
        "text",
        "value",
      ];
      searchableProps.forEach((prop) => {
        if (data[prop] && typeof data[prop] === "string") {
          searchText += " " + data[prop];
        }
      });
    }

    return searchText.trim();
  }

  private async performSearch(query: string): Promise<string[]> {
    const matchingNodeIds = new Set<string>();
    const queryWords = query.toLowerCase().split(/\s+/);

    // Search using index
    queryWords.forEach((word) => {
      // Exact matches
      if (this.searchIndex.has(word)) {
        this.searchIndex
          .get(word)!
          .forEach((nodeId) => matchingNodeIds.add(nodeId));
      }

      // Partial matches
      this.searchIndex.forEach((nodeIds, indexWord) => {
        if (indexWord.includes(word)) {
          nodeIds.forEach((nodeId) => matchingNodeIds.add(nodeId));
        }
      });
    });

    return Array.from(matchingNodeIds);
  }

  private updateNodeVisibilityForSearch(matchingNodeIds: string[]): void {
    // Hide all nodes first
    this.nodeCache.forEach((node) => (node.isVisible = false));

    // Show matching nodes and their ancestors
    matchingNodeIds.forEach((nodeId) => {
      this.showNodeAndAncestors(nodeId);
    });
  }

  private showNodeAndAncestors(nodeId: string): void {
    const node = this.nodeCache.get(nodeId);
    if (!node) return;

    node.isVisible = true;

    // Expand and show all ancestors
    if (node.parentId) {
      const parent = this.nodeCache.get(node.parentId);
      if (parent) {
        parent.isExpanded = true;
        parent.isVisible = true;
        this.viewState$.value.expandedNodes.add(parent.id);
        this.showNodeAndAncestors(parent.id);
      }
    }
  }

  private isValidMove(
    draggedNode: TreeNode<T>,
    targetNode: TreeNode<T>,
    operation: string
  ): boolean {
    // Cannot move node into itself or its descendants
    if (draggedNode.id === targetNode.id) {
      return false;
    }

    // Check if target is a descendant of dragged node
    const descendants = this.getDescendants(draggedNode.id);
    if (descendants.some((d) => d.id === targetNode.id)) {
      return false;
    }

    return true;
  }

  private async performNodeMove(
    draggedNode: TreeNode<T>,
    targetNode: TreeNode<T>,
    operation: string
  ): Promise<string[]> {
    const affectedNodes: string[] = [draggedNode.id, targetNode.id];

    // Remove from current parent
    if (draggedNode.parentId) {
      affectedNodes.push(draggedNode.parentId);
    }

    switch (operation) {
      case "move-into":
        draggedNode.parentId = targetNode.id;
        draggedNode.level = targetNode.level + 1;
        affectedNodes.push(targetNode.id);
        break;

      case "move-before":
      case "move-after":
        draggedNode.parentId = targetNode.parentId;
        draggedNode.level = targetNode.level;
        if (targetNode.parentId) {
          affectedNodes.push(targetNode.parentId);
        }
        break;
    }

    // Update paths for dragged node and its descendants
    this.updateNodePaths(draggedNode);

    return affectedNodes;
  }

  private updateNodePaths(node: TreeNode<T>): void {
    const parent = node.parentId ? this.nodeCache.get(node.parentId) : null;
    node.path = parent ? `${parent.path}/${node.id}` : node.id;

    // Update descendants
    const children = this.getDirectChildren(node.id);
    children.forEach((child) => this.updateNodePaths(child));
  }

  private rebuildTreeStructure(): void {
    // Recalculate positions and update tree structure
    const rootNodes = Array.from(this.nodeCache.values()).filter(
      (n) => n.level === 0
    );
    let position = 0;

    rootNodes.forEach((root) => {
      position = this.updateNodePositions(root, position);
    });
  }

  private updateNodePositions(
    node: TreeNode<T>,
    startPosition: number
  ): number {
    node.metadata.position = startPosition;
    let position = startPosition + 1;

    if (node.isExpanded) {
      const children = this.getDirectChildren(node.id).sort((a, b) =>
        a.data && b.data ? String(a.data).localeCompare(String(b.data)) : 0
      );

      children.forEach((child) => {
        position = this.updateNodePositions(child, position);
      });
    }

    return position;
  }

  private sortNodes(nodes: TreeNode<T>[]): TreeNode<T>[] {
    const viewState = this.viewState$.value;
    if (!viewState.sortConfig) {
      return nodes;
    }

    return nodes.sort((a, b) => {
      const { field, direction } = viewState.sortConfig!;
      const aValue = this.getFieldValue(a, field);
      const bValue = this.getFieldValue(b, field);

      const comparison = String(aValue).localeCompare(String(bValue));
      return direction === "asc" ? comparison : -comparison;
    });
  }

  private getFieldValue(node: TreeNode<T>, field: string): any {
    const data = node.data as any;
    return data && typeof data === "object" ? data[field] : data;
  }

  private async queueOperation(
    operation: () => Promise<TreeOperationResult>
  ): Promise<TreeOperationResult> {
    return new Promise((resolve, reject) => {
      this.operationQueue.push(async () => {
        try {
          const result = await operation();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      this.processOperationQueue();
    });
  }

  private async processOperationQueue(): Promise<void> {
    if (this.isProcessingOperations || this.operationQueue.length === 0) {
      return;
    }

    this.isProcessingOperations = true;

    while (this.operationQueue.length > 0) {
      const operation = this.operationQueue.shift()!;
      await operation();
    }

    this.isProcessingOperations = false;
  }

  private emitNodesUpdate(): void {
    this.nodes$.next(new Map(this.nodeCache));
  }

  private calculateMaxDepth(): number {
    let maxDepth = 0;
    this.nodeCache.forEach((node) => {
      maxDepth = Math.max(maxDepth, node.level);
    });
    return maxDepth;
  }

  private calculateAverageChildren(): number {
    const nodesWithChildren = Array.from(this.nodeCache.values()).filter(
      (node) => node.metadata.childCount > 0
    );

    if (nodesWithChildren.length === 0) return 0;

    const totalChildren = nodesWithChildren.reduce(
      (sum, node) => sum + node.metadata.childCount,
      0
    );

    return totalChildren / nodesWithChildren.length;
  }

  private calculateMemoryUsage(): number {
    // Rough estimation of memory usage
    const nodeSize = 500; // Estimated bytes per node
    const cacheSize = this.childrenCache.size * 100; // Estimated cache overhead
    const indexSize = this.searchIndex.size * 50; // Estimated index size

    return (this.nodeCache.size * nodeSize + cacheSize + indexSize) / 1024; // KB
  }
}
```

---

## 🎨 **Data Transformation & Normalization Algorithms**

### **Challenge 7: 🔄 Advanced Data Pipeline with Real-time Transformation**

**Problem:** Create a sophisticated data processing pipeline that handles complex transformations, validation, caching, and real-time updates with conflict resolution.

```typescript
// src/app/core/services/data-pipeline.service.ts
import { Injectable } from "@angular/core";
import {
  Observable,
  Subject,
  BehaviorSubject,
  combineLatest,
  merge,
  EMPTY,
} from "rxjs";
import {
  map,
  filter,
  debounceTime,
  distinctUntilChanged,
  switchMap,
  mergeMap,
  concatMap,
  catchError,
  retry,
  shareReplay,
  scan,
  tap,
  bufferTime,
  groupBy,
  reduce,
} from "rxjs/operators";

export interface DataTransformationRule<TInput, TOutput> {
  id: string;
  name: string;
  priority: number;
  condition: (data: TInput) => boolean;
  transform: (data: TInput) => TOutput | Promise<TOutput>;
  validate?: (data: TOutput) => boolean | Promise<boolean>;
  errorHandler?: (error: any, data: TInput) => TOutput | null;
  cache?: {
    enabled: boolean;
    ttl: number; // Time to live in milliseconds
    key: (data: TInput) => string;
  };
}

export interface PipelineStage<TInput, TOutput> {
  id: string;
  name: string;
  type: "transform" | "validate" | "filter" | "aggregate" | "branch";
  rules: DataTransformationRule<TInput, TOutput>[];
  parallel?: boolean;
  optional?: boolean;
  retry?: {
    maxAttempts: number;
    delay: number;
  };
}

export interface PipelineMetrics {
  totalProcessed: number;
  successfulTransformations: number;
  failedTransformations: number;
  averageProcessingTime: number;
  cacheHitRate: number;
  throughputPerSecond: number;
  stageMetrics: Map<
    string,
    {
      processed: number;
      failed: number;
      avgTime: number;
    }
  >;
}

export interface ConflictResolution<T> {
  strategy: "latest-wins" | "merge" | "queue" | "custom";
  merger?: (existing: T, incoming: T) => T;
  validator?: (resolved: T) => boolean;
}

export interface DataItem<T = any> {
  id: string;
  data: T;
  metadata: {
    timestamp: Date;
    version: number;
    source: string;
    transformationHistory: string[];
    validationResults: ValidationResult[];
    cacheInfo?: {
      cached: boolean;
      cacheKey: string;
      expiry: Date;
    };
  };
}

export interface ValidationResult {
  ruleId: string;
  passed: boolean;
  message?: string;
  timestamp: Date;
}

@Injectable({
  providedIn: "root",
})
export class DataPipelineService {
  private pipelines = new Map<string, PipelineStage<any, any>[]>();
  private dataCache = new Map<string, { data: any; expiry: Date }>();
  private conflictResolvers = new Map<string, ConflictResolution<any>>();
  private metrics = new Map<string, PipelineMetrics>();

  private dataInput$ = new Subject<DataItem<any>>();
  private dataOutput$ = new Subject<DataItem<any>>();
  private processingErrors$ = new Subject<{
    error: any;
    data: DataItem<any>;
    stage: string;
  }>();

  private isProcessing = false;
  private processingQueue: DataItem<any>[] = [];
  private readonly QUEUE_BATCH_SIZE = 50;

  // 🏗️ REGISTER PIPELINE
  registerPipeline<TInput, TOutput>(
    pipelineId: string,
    stages: PipelineStage<TInput, TOutput>[],
    conflictResolution?: ConflictResolution<TOutput>
  ): void {
    // Validate pipeline stages
    this.validatePipelineStages(stages);

    // Sort stages by priority
    const sortedStages = stages.sort(
      (a, b) => (a.rules[0]?.priority || 0) - (b.rules[0]?.priority || 0)
    );

    this.pipelines.set(pipelineId, sortedStages);

    if (conflictResolution) {
      this.conflictResolvers.set(pipelineId, conflictResolution);
    }

    // Initialize metrics
    this.metrics.set(pipelineId, {
      totalProcessed: 0,
      successfulTransformations: 0,
      failedTransformations: 0,
      averageProcessingTime: 0,
      cacheHitRate: 0,
      throughputPerSecond: 0,
      stageMetrics: new Map(),
    });

    console.log(
      `🏗️ Registered data pipeline: ${pipelineId} with ${stages.length} stages`
    );
  }

  // 📤 PROCESS DATA
  processData<TInput, TOutput>(
    pipelineId: string,
    data: TInput,
    metadata?: Partial<DataItem<TInput>["metadata"]>
  ): Observable<DataItem<TOutput>> {
    const dataItem: DataItem<TInput> = {
      id: this.generateDataId(),
      data,
      metadata: {
        timestamp: new Date(),
        version: 1,
        source: "user-input",
        transformationHistory: [],
        validationResults: [],
        ...metadata,
      },
    };

    // Add to processing queue
    this.processingQueue.push(dataItem);
    this.processQueue();

    // Return observable for this specific data item
    return this.dataOutput$.pipe(
      filter((result) => result.id === dataItem.id),
      map((result) => result as DataItem<TOutput>)
    );
  }

  // 🔄 PROCESS STREAM
  processDataStream<TInput, TOutput>(
    pipelineId: string,
    dataStream: Observable<TInput>,
    options: {
      batchSize?: number;
      debounceMs?: number;
      enableConflictResolution?: boolean;
    } = {}
  ): Observable<DataItem<TOutput>> {
    const {
      batchSize = 10,
      debounceMs = 100,
      enableConflictResolution = false,
    } = options;

    return dataStream.pipe(
      // Group by batch size and time window
      bufferTime(debounceMs, null, batchSize),
      filter((batch) => batch.length > 0),

      // Convert to data items
      map((batch) =>
        batch.map((data) => ({
          id: this.generateDataId(),
          data,
          metadata: {
            timestamp: new Date(),
            version: 1,
            source: "stream-input",
            transformationHistory: [],
            validationResults: [],
          },
        }))
      ),

      // Process batch through pipeline
      concatMap((batch) =>
        this.processBatch(pipelineId, batch, enableConflictResolution)
      ),

      // Flatten results
      mergeMap((results) => results)
    );
  }

  // 📊 GET PIPELINE METRICS
  getPipelineMetrics(pipelineId: string): Observable<PipelineMetrics> {
    return new BehaviorSubject(
      this.metrics.get(pipelineId) || this.createEmptyMetrics()
    ).asObservable();
  }

  // ⚠️ GET PROCESSING ERRORS
  getProcessingErrors(): Observable<{
    error: any;
    data: DataItem<any>;
    stage: string;
  }> {
    return this.processingErrors$.asObservable();
  }

  // 🧹 CLEAR CACHE
  clearCache(pattern?: string): void {
    if (pattern) {
      const regex = new RegExp(pattern);
      for (const [key] of this.dataCache) {
        if (regex.test(key)) {
          this.dataCache.delete(key);
        }
      }
    } else {
      this.dataCache.clear();
    }

    console.log(`🧹 Cleared cache${pattern ? ` for pattern: ${pattern}` : ""}`);
  }

  // 🔄 Private Methods

  private async processQueue(): Promise<void> {
    if (this.isProcessing || this.processingQueue.length === 0) {
      return;
    }

    this.isProcessing = true;

    while (this.processingQueue.length > 0) {
      const batch = this.processingQueue.splice(0, this.QUEUE_BATCH_SIZE);
      await this.processBatchInQueue(batch);
    }

    this.isProcessing = false;
  }

  private async processBatchInQueue(batch: DataItem<any>[]): Promise<void> {
    // Process items in parallel with controlled concurrency
    const processingPromises = batch.map((item) => this.processDataItem(item));

    try {
      await Promise.allSettled(processingPromises);
    } catch (error) {
      console.error("Batch processing error:", error);
    }
  }

  private async processDataItem(dataItem: DataItem<any>): Promise<void> {
    try {
      // Find applicable pipeline (in real implementation, this would be more sophisticated)
      const pipelineEntry = Array.from(this.pipelines.entries())[0];
      if (!pipelineEntry) {
        throw new Error("No pipeline registered");
      }

      const [pipelineId, stages] = pipelineEntry;
      const startTime = performance.now();

      // Process through all stages
      let currentData = dataItem;

      for (const stage of stages) {
        currentData = await this.processStage(currentData, stage, pipelineId);
      }

      // Update metrics
      const processingTime = performance.now() - startTime;
      this.updateMetrics(pipelineId, true, processingTime);

      // Emit result
      this.dataOutput$.next(currentData);
    } catch (error) {
      console.error("Data processing error:", error);
      this.updateMetrics("unknown", false, 0);
      this.processingErrors$.next({ error, data: dataItem, stage: "unknown" });
    }
  }

  private async processStage<TInput, TOutput>(
    dataItem: DataItem<TInput>,
    stage: PipelineStage<TInput, TOutput>,
    pipelineId: string
  ): Promise<DataItem<TOutput>> {
    const stageStartTime = performance.now();

    try {
      // Apply all rules in the stage
      let result = dataItem;

      if (stage.parallel && stage.rules.length > 1) {
        // Process rules in parallel
        const rulePromises = stage.rules.map((rule) =>
          this.applyRule(result, rule)
        );
        const ruleResults = await Promise.allSettled(rulePromises);

        // Merge results (simplified - in practice, you'd need more sophisticated merging)
        result = ruleResults
          .filter(
            (r): r is PromiseFulfilledResult<DataItem<TOutput>> =>
              r.status === "fulfilled"
          )
          .reduce((acc, r) => r.value, result as any);
      } else {
        // Process rules sequentially
        for (const rule of stage.rules) {
          if (rule.condition(result.data)) {
            result = await this.applyRule(result, rule);
          }
        }
      }

      // Update stage metrics
      const stageTime = performance.now() - stageStartTime;
      this.updateStageMetrics(pipelineId, stage.id, true, stageTime);

      // Add to transformation history
      result.metadata.transformationHistory.push(stage.id);

      return result as DataItem<TOutput>;
    } catch (error) {
      if (!stage.optional) {
        throw error;
      }

      console.warn(`Optional stage ${stage.id} failed:`, error);
      const stageTime = performance.now() - stageStartTime;
      this.updateStageMetrics(pipelineId, stage.id, false, stageTime);

      return dataItem as any;
    }
  }

  private async applyRule<TInput, TOutput>(
    dataItem: DataItem<TInput>,
    rule: DataTransformationRule<TInput, TOutput>
  ): Promise<DataItem<TOutput>> {
    try {
      // Check cache first
      if (rule.cache?.enabled) {
        const cacheKey = rule.cache.key(dataItem.data);
        const cached = this.getFromCache(cacheKey);

        if (cached) {
          return {
            ...dataItem,
            data: cached,
            metadata: {
              ...dataItem.metadata,
              cacheInfo: {
                cached: true,
                cacheKey,
                expiry: new Date(Date.now() + (rule.cache.ttl || 300000)),
              },
            },
          } as DataItem<TOutput>;
        }
      }

      // Apply transformation
      const transformedData = await rule.transform(dataItem.data);

      // Validate if validator is provided
      if (rule.validate) {
        const isValid = await rule.validate(transformedData);

        if (!isValid) {
          throw new Error(`Validation failed for rule ${rule.id}`);
        }
      }

      // Cache result if caching is enabled
      if (rule.cache?.enabled) {
        const cacheKey = rule.cache.key(dataItem.data);
        this.setCache(cacheKey, transformedData, rule.cache.ttl);
      }

      // Create transformed data item
      const transformedItem: DataItem<TOutput> = {
        ...dataItem,
        data: transformedData,
        metadata: {
          ...dataItem.metadata,
          version: dataItem.metadata.version + 1,
          validationResults: [
            ...dataItem.metadata.validationResults,
            {
              ruleId: rule.id,
              passed: true,
              timestamp: new Date(),
            },
          ],
        },
      };

      return transformedItem;
    } catch (error) {
      // Handle error with error handler if provided
      if (rule.errorHandler) {
        const fallbackData = rule.errorHandler(error, dataItem.data);

        if (fallbackData !== null) {
          return {
            ...dataItem,
            data: fallbackData,
            metadata: {
              ...dataItem.metadata,
              validationResults: [
                ...dataItem.metadata.validationResults,
                {
                  ruleId: rule.id,
                  passed: false,
                  message: error.message,
                  timestamp: new Date(),
                },
              ],
            },
          } as DataItem<TOutput>;
        }
      }

      throw error;
    }
  }

  private async processBatch<TInput, TOutput>(
    pipelineId: string,
    batch: DataItem<TInput>[],
    enableConflictResolution: boolean
  ): Promise<DataItem<TOutput>[]> {
    const results: DataItem<TOutput>[] = [];

    // Process each item in the batch
    for (const item of batch) {
      try {
        const processed = await this.processDataItem(item);

        if (enableConflictResolution) {
          const resolved = await this.resolveConflicts(
            pipelineId,
            processed as DataItem<TOutput>
          );
          results.push(resolved);
        } else {
          results.push(processed as DataItem<TOutput>);
        }
      } catch (error) {
        console.error("Item processing error:", error);
        this.processingErrors$.next({
          error,
          data: item,
          stage: "batch-processing",
        });
      }
    }

    return results;
  }

  private async resolveConflicts<T>(
    pipelineId: string,
    newData: DataItem<T>
  ): Promise<DataItem<T>> {
    const resolver = this.conflictResolvers.get(pipelineId);
    if (!resolver) {
      return newData;
    }

    // In a real implementation, you'd check for existing data with the same ID
    // and apply conflict resolution strategies

    switch (resolver.strategy) {
      case "latest-wins":
        return newData;

      case "merge":
        if (resolver.merger) {
          // Get existing data and merge
          const existingData = this.getExistingData(newData.id);
          if (existingData) {
            const merged = resolver.merger(existingData.data, newData.data);
            return {
              ...newData,
              data: merged,
              metadata: {
                ...newData.metadata,
                version:
                  Math.max(
                    existingData.metadata.version,
                    newData.metadata.version
                  ) + 1,
              },
            };
          }
        }
        return newData;

      case "queue":
        // Queue for manual resolution
        return newData;

      case "custom":
        // Apply custom resolution logic
        return newData;

      default:
        return newData;
    }
  }

  // Helper methods...

  private validatePipelineStages(stages: PipelineStage<any, any>[]): void {
    // Validate stage configuration
    stages.forEach((stage) => {
      if (
        !stage.id ||
        !stage.name ||
        !stage.rules ||
        stage.rules.length === 0
      ) {
        throw new Error(`Invalid stage configuration: ${stage.id}`);
      }
    });
  }

  private getFromCache(key: string): any {
    const cached = this.dataCache.get(key);
    if (cached && cached.expiry > new Date()) {
      return cached.data;
    }

    // Remove expired cache
    if (cached) {
      this.dataCache.delete(key);
    }

    return null;
  }

  private setCache(key: string, data: any, ttl: number): void {
    const expiry = new Date(Date.now() + ttl);
    this.dataCache.set(key, { data, expiry });
  }

  private updateMetrics(
    pipelineId: string,
    success: boolean,
    processingTime: number
  ): void {
    const metrics = this.metrics.get(pipelineId);
    if (!metrics) return;

    metrics.totalProcessed++;
    if (success) {
      metrics.successfulTransformations++;
    } else {
      metrics.failedTransformations++;
    }

    // Update average processing time
    const prevAvg = metrics.averageProcessingTime;
    const count = metrics.totalProcessed;
    metrics.averageProcessingTime =
      (prevAvg * (count - 1) + processingTime) / count;

    this.metrics.set(pipelineId, metrics);
  }

  private updateStageMetrics(
    pipelineId: string,
    stageId: string,
    success: boolean,
    processingTime: number
  ): void {
    const metrics = this.metrics.get(pipelineId);
    if (!metrics) return;

    if (!metrics.stageMetrics.has(stageId)) {
      metrics.stageMetrics.set(stageId, {
        processed: 0,
        failed: 0,
        avgTime: 0,
      });
    }

    const stageMetrics = metrics.stageMetrics.get(stageId)!;
    stageMetrics.processed++;

    if (!success) {
      stageMetrics.failed++;
    }

    // Update average time
    const prevAvg = stageMetrics.avgTime;
    const count = stageMetrics.processed;
    stageMetrics.avgTime = (prevAvg * (count - 1) + processingTime) / count;
  }

  private createEmptyMetrics(): PipelineMetrics {
    return {
      totalProcessed: 0,
      successfulTransformations: 0,
      failedTransformations: 0,
      averageProcessingTime: 0,
      cacheHitRate: 0,
      throughputPerSecond: 0,
      stageMetrics: new Map(),
    };
  }

  private generateDataId(): string {
    return `data_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private getExistingData(id: string): DataItem<any> | null {
    // In practice, this would query your data store
    return null;
  }
}
```

This is **Part 3** of the Coding Challenges guide. Would you like me to continue with **Part 4** covering real-time synchronization algorithms, advanced caching strategies, and WebSocket optimization patterns?
