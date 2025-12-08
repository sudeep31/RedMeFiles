# 🔄 Angular State Management Patterns: Complete Guide

## 🎯 **Question Overview**

_"How do you implement complex state management in Angular using NgRx, Akita, and other state management libraries?"_

## 🔍 **Understanding State Management**

State management is **crucial for complex Angular applications** where **component communication**, **data consistency**, and **predictable state updates** are essential. Modern Angular offers **multiple approaches** from **built-in signals** to **sophisticated libraries** like **NgRx** and **Akita**.

Advanced state management provides **time-travel debugging**, **middleware support**, **side effects handling**, and **performance optimization**! 📊

## 🏗️ **NgRx Implementation**

### **1. 📦 Store Setup & Configuration**

```typescript
// src/app/store/app.state.ts - Global App State Interface
import { ActionReducerMap, MetaReducer } from "@ngrx/store";
import { environment } from "../../environments/environment";

// Feature state interfaces
import * as fromAuth from "./auth/auth.reducer";
import * as fromProducts from "./products/products.reducer";
import * as fromCart from "./cart/cart.reducer";
import * as fromOrders from "./orders/orders.reducer";
import * as fromUi from "./ui/ui.reducer";

export interface AppState {
  auth: fromAuth.AuthState;
  products: fromProducts.ProductsState;
  cart: fromCart.CartState;
  orders: fromOrders.OrdersState;
  ui: fromUi.UiState;
}

export const reducers: ActionReducerMap<AppState> = {
  auth: fromAuth.authReducer,
  products: fromProducts.productsReducer,
  cart: fromCart.cartReducer,
  orders: fromOrders.ordersReducer,
  ui: fromUi.uiReducer,
};

// Meta-reducers for cross-cutting concerns
export function logger(reducer: any): any {
  return function (state: any, action: any): any {
    console.log("State:", state);
    console.log("Action:", action);
    return reducer(state, action);
  };
}

export function clearStateOnLogout(reducer: any): any {
  return function (state: any, action: any): any {
    if (action.type === "[Auth] Logout Success") {
      // Reset specific parts of state on logout
      const newState = {
        ...state,
        auth: undefined,
        cart: undefined,
        orders: undefined,
        // Keep products and UI state
      };
      return reducer(newState, action);
    }
    return reducer(state, action);
  };
}

export function rehydrateState(reducer: any): any {
  return function (state: any, action: any): any {
    if (action.type === "[App] Hydrate State") {
      // Restore state from localStorage or sessionStorage
      const savedState = localStorage.getItem("app-state");
      if (savedState) {
        try {
          const parsedState = JSON.parse(savedState);
          return { ...state, ...parsedState };
        } catch (error) {
          console.error("Failed to rehydrate state:", error);
        }
      }
    }
    return reducer(state, action);
  };
}

export const metaReducers: MetaReducer<AppState>[] = !environment.production
  ? [logger, clearStateOnLogout, rehydrateState]
  : [clearStateOnLogout, rehydrateState];
```

```typescript
// src/app/store/products/products.actions.ts - Feature Actions
import { createActionGroup, emptyProps, props } from "@ngrx/store";

export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl: string;
  stock: number;
  rating: number;
  reviews: number;
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
}

export interface ProductFilter {
  search?: string;
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  inStock?: boolean;
  rating?: number;
  tags?: string[];
}

export interface ProductsApiResponse {
  products: Product[];
  totalCount: number;
  page: number;
  pageSize: number;
  hasNext: boolean;
  hasPrevious: boolean;
}

export interface ProductCreateRequest {
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl?: string;
  stock: number;
  tags?: string[];
}

// Action Groups for better organization
export const ProductsPageActions = createActionGroup({
  source: "Products Page",
  events: {
    "Enter Products Page": emptyProps(),
    "Load Products": props<{
      filter?: ProductFilter;
      page?: number;
      pageSize?: number;
    }>(),
    "Search Products": props<{ query: string }>(),
    "Filter Products": props<{ filter: ProductFilter }>(),
    "Sort Products": props<{ sortBy: string; sortDirection: "asc" | "desc" }>(),
    "Select Product": props<{ productId: string }>(),
    "Clear Selection": emptyProps(),
    "Add Product To Cart": props<{ productId: string; quantity: number }>(),
    "Toggle Product Favorite": props<{ productId: string }>(),
  },
});

export const ProductsApiActions = createActionGroup({
  source: "Products API",
  events: {
    "Load Products Success": props<{ response: ProductsApiResponse }>(),
    "Load Products Failure": props<{ error: string }>(),
    "Load Product Detail Success": props<{ product: Product }>(),
    "Load Product Detail Failure": props<{ error: string }>(),
    "Create Product Success": props<{ product: Product }>(),
    "Create Product Failure": props<{ error: string }>(),
    "Update Product Success": props<{ product: Product }>(),
    "Update Product Failure": props<{ error: string }>(),
    "Delete Product Success": props<{ productId: string }>(),
    "Delete Product Failure": props<{ error: string }>(),
  },
});

export const ProductsCacheActions = createActionGroup({
  source: "Products Cache",
  events: {
    "Invalidate Cache": emptyProps(),
    "Update Cache Entry": props<{ product: Product }>(),
    "Remove Cache Entry": props<{ productId: string }>(),
    "Preload Products": props<{ productIds: string[] }>(),
  },
});

export const ProductsRealtimeActions = createActionGroup({
  source: "Products Realtime",
  events: {
    "Product Updated": props<{ product: Product }>(),
    "Product Deleted": props<{ productId: string }>(),
    "Stock Updated": props<{ productId: string; stock: number }>(),
    "Price Updated": props<{ productId: string; price: number }>(),
  },
});
```

```typescript
// src/app/store/products/products.reducer.ts - Feature Reducer
import { createReducer, on } from "@ngrx/store";
import { EntityState, EntityAdapter, createEntityAdapter } from "@ngrx/entity";

import { Product, ProductFilter } from "./products.actions";
import {
  ProductsPageActions,
  ProductsApiActions,
  ProductsCacheActions,
  ProductsRealtimeActions,
} from "./products.actions";

// Entity adapter for normalized state
export const productAdapter: EntityAdapter<Product> =
  createEntityAdapter<Product>({
    selectId: (product: Product) => product.id,
    sortComparer: (a: Product, b: Product) => a.name.localeCompare(b.name),
  });

export interface ProductsState extends EntityState<Product> {
  // Loading states
  loading: boolean;
  loadingDetail: boolean;
  creating: boolean;
  updating: boolean;
  deleting: boolean;

  // Data state
  selectedProductId: string | null;
  filter: ProductFilter;
  sortBy: string;
  sortDirection: "asc" | "desc";

  // Pagination
  currentPage: number;
  pageSize: number;
  totalCount: number;
  hasNext: boolean;
  hasPrevious: boolean;

  // Cache management
  lastUpdated: { [productId: string]: number };
  cacheExpiry: number; // milliseconds

  // Error handling
  error: string | null;
  apiErrors: { [operation: string]: string };

  // UI state
  searchQuery: string;
  favoriteIds: string[];
  recentlyViewed: string[];

  // Performance optimizations
  prefetchedIds: string[];
  optimisticUpdates: { [productId: string]: Partial<Product> };
}

export const initialState: ProductsState = productAdapter.getInitialState({
  loading: false,
  loadingDetail: false,
  creating: false,
  updating: false,
  deleting: false,

  selectedProductId: null,
  filter: {},
  sortBy: "name",
  sortDirection: "asc" as const,

  currentPage: 1,
  pageSize: 20,
  totalCount: 0,
  hasNext: false,
  hasPrevious: false,

  lastUpdated: {},
  cacheExpiry: 5 * 60 * 1000, // 5 minutes

  error: null,
  apiErrors: {},

  searchQuery: "",
  favoriteIds: [],
  recentlyViewed: [],

  prefetchedIds: [],
  optimisticUpdates: {},
});

export const productsReducer = createReducer(
  initialState,

  // Page Actions
  on(ProductsPageActions.enterProductsPage, (state) => ({
    ...state,
    error: null,
  })),

  on(ProductsPageActions.loadProducts, (state, { filter, page, pageSize }) => ({
    ...state,
    loading: true,
    error: null,
    filter: filter ? { ...state.filter, ...filter } : state.filter,
    currentPage: page || state.currentPage,
    pageSize: pageSize || state.pageSize,
  })),

  on(ProductsPageActions.searchProducts, (state, { query }) => ({
    ...state,
    searchQuery: query,
    filter: { ...state.filter, search: query },
    currentPage: 1, // Reset to first page on search
  })),

  on(ProductsPageActions.filterProducts, (state, { filter }) => ({
    ...state,
    filter: { ...state.filter, ...filter },
    currentPage: 1, // Reset to first page on filter
  })),

  on(ProductsPageActions.sortProducts, (state, { sortBy, sortDirection }) => ({
    ...state,
    sortBy,
    sortDirection,
  })),

  on(ProductsPageActions.selectProduct, (state, { productId }) => ({
    ...state,
    selectedProductId: productId,
    recentlyViewed: [
      productId,
      ...state.recentlyViewed.filter((id) => id !== productId),
    ].slice(0, 10), // Keep only last 10
  })),

  on(ProductsPageActions.clearSelection, (state) => ({
    ...state,
    selectedProductId: null,
  })),

  on(ProductsPageActions.toggleProductFavorite, (state, { productId }) => {
    const isFavorite = state.favoriteIds.includes(productId);
    return {
      ...state,
      favoriteIds: isFavorite
        ? state.favoriteIds.filter((id) => id !== productId)
        : [...state.favoriteIds, productId],
    };
  }),

  // API Success Actions
  on(ProductsApiActions.loadProductsSuccess, (state, { response }) =>
    productAdapter.setMany(response.products, {
      ...state,
      loading: false,
      error: null,
      totalCount: response.totalCount,
      currentPage: response.page,
      pageSize: response.pageSize,
      hasNext: response.hasNext,
      hasPrevious: response.hasPrevious,
      lastUpdated: {
        ...state.lastUpdated,
        ...response.products.reduce(
          (acc, product) => ({
            ...acc,
            [product.id]: Date.now(),
          }),
          {}
        ),
      },
    })
  ),

  on(ProductsApiActions.loadProductDetailSuccess, (state, { product }) =>
    productAdapter.upsertOne(product, {
      ...state,
      loadingDetail: false,
      selectedProductId: product.id,
      lastUpdated: {
        ...state.lastUpdated,
        [product.id]: Date.now(),
      },
    })
  ),

  on(ProductsApiActions.createProductSuccess, (state, { product }) =>
    productAdapter.addOne(product, {
      ...state,
      creating: false,
      error: null,
      totalCount: state.totalCount + 1,
    })
  ),

  on(ProductsApiActions.updateProductSuccess, (state, { product }) =>
    productAdapter.updateOne(
      { id: product.id, changes: product },
      {
        ...state,
        updating: false,
        error: null,
        optimisticUpdates: {
          ...state.optimisticUpdates,
          [product.id]: undefined, // Clear optimistic update
        },
        lastUpdated: {
          ...state.lastUpdated,
          [product.id]: Date.now(),
        },
      }
    )
  ),

  on(ProductsApiActions.deleteProductSuccess, (state, { productId }) =>
    productAdapter.removeOne(productId, {
      ...state,
      deleting: false,
      error: null,
      selectedProductId:
        state.selectedProductId === productId ? null : state.selectedProductId,
      favoriteIds: state.favoriteIds.filter((id) => id !== productId),
      recentlyViewed: state.recentlyViewed.filter((id) => id !== productId),
      totalCount: Math.max(0, state.totalCount - 1),
    })
  ),

  // API Failure Actions
  on(
    ProductsApiActions.loadProductsFailure,
    ProductsApiActions.loadProductDetailFailure,
    ProductsApiActions.createProductFailure,
    ProductsApiActions.updateProductFailure,
    ProductsApiActions.deleteProductFailure,
    (state, { error }) => ({
      ...state,
      loading: false,
      loadingDetail: false,
      creating: false,
      updating: false,
      deleting: false,
      error,
    })
  ),

  // Cache Actions
  on(ProductsCacheActions.invalidateCache, (state) => ({
    ...state,
    lastUpdated: {},
  })),

  on(ProductsCacheActions.updateCacheEntry, (state, { product }) =>
    productAdapter.upsertOne(product, {
      ...state,
      lastUpdated: {
        ...state.lastUpdated,
        [product.id]: Date.now(),
      },
    })
  ),

  on(ProductsCacheActions.removeCacheEntry, (state, { productId }) =>
    productAdapter.removeOne(productId, {
      ...state,
      lastUpdated: {
        ...state.lastUpdated,
        [productId]: undefined,
      },
    })
  ),

  on(ProductsCacheActions.preloadProducts, (state, { productIds }) => ({
    ...state,
    prefetchedIds: [...new Set([...state.prefetchedIds, ...productIds])],
  })),

  // Realtime Actions
  on(ProductsRealtimeActions.productUpdated, (state, { product }) =>
    productAdapter.upsertOne(product, {
      ...state,
      lastUpdated: {
        ...state.lastUpdated,
        [product.id]: Date.now(),
      },
    })
  ),

  on(ProductsRealtimeActions.productDeleted, (state, { productId }) =>
    productAdapter.removeOne(productId, {
      ...state,
      selectedProductId:
        state.selectedProductId === productId ? null : state.selectedProductId,
      favoriteIds: state.favoriteIds.filter((id) => id !== productId),
      recentlyViewed: state.recentlyViewed.filter((id) => id !== productId),
    })
  ),

  on(ProductsRealtimeActions.stockUpdated, (state, { productId, stock }) => {
    const product = state.entities[productId];
    if (product) {
      return productAdapter.updateOne(
        { id: productId, changes: { stock } },
        state
      );
    }
    return state;
  }),

  on(ProductsRealtimeActions.priceUpdated, (state, { productId, price }) => {
    const product = state.entities[productId];
    if (product) {
      return productAdapter.updateOne(
        { id: productId, changes: { price } },
        state
      );
    }
    return state;
  })
);

// Export entity selectors
export const {
  selectIds: selectProductIds,
  selectEntities: selectProductEntities,
  selectAll: selectAllProducts,
  selectTotal: selectProductsTotal,
} = productAdapter.getSelectors();
```

### **2. 🎯 Advanced Selectors**

```typescript
// src/app/store/products/products.selectors.ts - Memoized Selectors
import { createSelector, createFeatureSelector } from "@ngrx/store";
import {
  ProductsState,
  selectAllProducts,
  selectProductEntities,
} from "./products.reducer";

// Feature selector
export const selectProductsState =
  createFeatureSelector<ProductsState>("products");

// Basic selectors
export const selectProductsLoading = createSelector(
  selectProductsState,
  (state) => state.loading
);

export const selectProductsError = createSelector(
  selectProductsState,
  (state) => state.error
);

export const selectSelectedProductId = createSelector(
  selectProductsState,
  (state) => state.selectedProductId
);

export const selectCurrentFilter = createSelector(
  selectProductsState,
  (state) => state.filter
);

export const selectPaginationInfo = createSelector(
  selectProductsState,
  (state) => ({
    currentPage: state.currentPage,
    pageSize: state.pageSize,
    totalCount: state.totalCount,
    hasNext: state.hasNext,
    hasPrevious: state.hasPrevious,
  })
);

// Computed selectors
export const selectFilteredProducts = createSelector(
  selectAllProducts,
  selectCurrentFilter,
  (products, filter) => {
    if (!filter || Object.keys(filter).length === 0) {
      return products;
    }

    return products.filter((product) => {
      // Search filter
      if (filter.search) {
        const searchTerm = filter.search.toLowerCase();
        const matchesSearch =
          product.name.toLowerCase().includes(searchTerm) ||
          product.description.toLowerCase().includes(searchTerm) ||
          product.tags.some((tag) => tag.toLowerCase().includes(searchTerm));

        if (!matchesSearch) return false;
      }

      // Category filter
      if (filter.category && product.category !== filter.category) {
        return false;
      }

      // Price range filter
      if (filter.minPrice && product.price < filter.minPrice) {
        return false;
      }

      if (filter.maxPrice && product.price > filter.maxPrice) {
        return false;
      }

      // Stock filter
      if (filter.inStock && product.stock <= 0) {
        return false;
      }

      // Rating filter
      if (filter.rating && product.rating < filter.rating) {
        return false;
      }

      // Tags filter
      if (filter.tags && filter.tags.length > 0) {
        const hasMatchingTag = filter.tags.some((tag) =>
          product.tags.includes(tag)
        );
        if (!hasMatchingTag) return false;
      }

      return true;
    });
  }
);

export const selectSortedProducts = createSelector(
  selectFilteredProducts,
  selectProductsState,
  (products, state) => {
    const { sortBy, sortDirection } = state;

    return [...products].sort((a, b) => {
      let comparison = 0;

      switch (sortBy) {
        case "name":
          comparison = a.name.localeCompare(b.name);
          break;
        case "price":
          comparison = a.price - b.price;
          break;
        case "rating":
          comparison = a.rating - b.rating;
          break;
        case "stock":
          comparison = a.stock - b.stock;
          break;
        case "created":
          comparison =
            new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime();
          break;
        default:
          return 0;
      }

      return sortDirection === "asc" ? comparison : -comparison;
    });
  }
);

export const selectPaginatedProducts = createSelector(
  selectSortedProducts,
  selectPaginationInfo,
  (products, pagination) => {
    const start = (pagination.currentPage - 1) * pagination.pageSize;
    const end = start + pagination.pageSize;
    return products.slice(start, end);
  }
);

export const selectSelectedProduct = createSelector(
  selectProductEntities,
  selectSelectedProductId,
  (entities, selectedId) => (selectedId ? entities[selectedId] : null)
);

export const selectFavoriteProducts = createSelector(
  selectAllProducts,
  selectProductsState,
  (products, state) =>
    products.filter((product) => state.favoriteIds.includes(product.id))
);

export const selectRecentlyViewedProducts = createSelector(
  selectProductEntities,
  selectProductsState,
  (entities, state) =>
    state.recentlyViewed
      .map((id) => entities[id])
      .filter((product) => product !== undefined)
);

// Performance selectors
export const selectProductsLoadingState = createSelector(
  selectProductsState,
  (state) => ({
    loading: state.loading,
    loadingDetail: state.loadingDetail,
    creating: state.creating,
    updating: state.updating,
    deleting: state.deleting,
  })
);

export const selectCacheStatus = createSelector(
  selectProductsState,
  (state) => ({
    lastUpdated: state.lastUpdated,
    cacheExpiry: state.cacheExpiry,
    prefetchedIds: state.prefetchedIds,
  })
);

// Category-specific selectors
export const selectProductsByCategory = createSelector(
  selectAllProducts,
  (products) => {
    return products.reduce((acc, product) => {
      const category = product.category;
      if (!acc[category]) {
        acc[category] = [];
      }
      acc[category].push(product);
      return acc;
    }, {} as { [category: string]: Product[] });
  }
);

export const selectProductCategories = createSelector(
  selectAllProducts,
  (products) => [...new Set(products.map((p) => p.category))].sort()
);

// Statistics selectors
export const selectProductsStats = createSelector(
  selectAllProducts,
  (products) => {
    if (products.length === 0) {
      return {
        total: 0,
        averagePrice: 0,
        averageRating: 0,
        totalStock: 0,
        outOfStock: 0,
        lowStock: 0,
      };
    }

    const total = products.length;
    const averagePrice = products.reduce((sum, p) => sum + p.price, 0) / total;
    const averageRating =
      products.reduce((sum, p) => sum + p.rating, 0) / total;
    const totalStock = products.reduce((sum, p) => sum + p.stock, 0);
    const outOfStock = products.filter((p) => p.stock === 0).length;
    const lowStock = products.filter((p) => p.stock > 0 && p.stock < 10).length;

    return {
      total,
      averagePrice: Math.round(averagePrice * 100) / 100,
      averageRating: Math.round(averageRating * 10) / 10,
      totalStock,
      outOfStock,
      lowStock,
    };
  }
);

// Search selectors
export const selectSearchResults = createSelector(
  selectProductsState,
  selectFilteredProducts,
  (state, products) => ({
    query: state.searchQuery,
    results: products,
    totalResults: products.length,
    hasResults: products.length > 0,
  })
);

// Complex business logic selectors
export const selectRecommendedProducts = createSelector(
  selectAllProducts,
  selectSelectedProduct,
  selectFavoriteProducts,
  (allProducts, selectedProduct, favoriteProducts) => {
    if (!selectedProduct) return [];

    // Simple recommendation algorithm
    const recommendations = allProducts
      .filter((p) => p.id !== selectedProduct.id)
      .filter(
        (p) =>
          p.category === selectedProduct.category ||
          p.tags.some((tag) => selectedProduct.tags.includes(tag))
      )
      .sort((a, b) => b.rating - a.rating)
      .slice(0, 5);

    return recommendations;
  }
);
```

### **3. ⚡ Effects for Side Effects**

```typescript
// src/app/store/products/products.effects.ts - Side Effects Management
import { Injectable, inject } from "@angular/core";
import { Actions, createEffect, ofType } from "@ngrx/effects";
import { Store } from "@ngrx/store";
import { of, timer, EMPTY } from "rxjs";
import {
  map,
  catchError,
  switchMap,
  mergeMap,
  debounceTime,
  distinctUntilChanged,
  withLatestFrom,
  tap,
  filter,
  exhaustMap,
  concatMap,
} from "rxjs/operators";

import { ProductsService } from "../../services/products.service";
import { NotificationService } from "../../services/notification.service";
import { CacheService } from "../../services/cache.service";
import { AnalyticsService } from "../../services/analytics.service";

import {
  ProductsPageActions,
  ProductsApiActions,
  ProductsCacheActions,
} from "./products.actions";
import {
  selectCurrentFilter,
  selectPaginationInfo,
  selectCacheStatus,
} from "./products.selectors";

@Injectable()
export class ProductsEffects {
  private actions$ = inject(Actions);
  private store = inject(Store);
  private productsService = inject(ProductsService);
  private notificationService = inject(NotificationService);
  private cacheService = inject(CacheService);
  private analytics = inject(AnalyticsService);

  // Load products with caching and error handling
  loadProducts$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsPageActions.loadProducts),
      withLatestFrom(
        this.store.select(selectCurrentFilter),
        this.store.select(selectPaginationInfo),
        this.store.select(selectCacheStatus)
      ),
      switchMap(([action, filter, pagination, cacheStatus]) => {
        // Check cache first
        const cacheKey = this.generateCacheKey(filter, pagination);
        const cachedData = this.cacheService.get(cacheKey);

        if (
          cachedData &&
          this.isCacheValid(cachedData.timestamp, cacheStatus.cacheExpiry)
        ) {
          return of(
            ProductsApiActions.loadProductsSuccess({
              response: cachedData.data,
            })
          );
        }

        // Load from API
        return this.productsService
          .getProducts({
            ...filter,
            page: pagination.currentPage,
            pageSize: pagination.pageSize,
          })
          .pipe(
            map((response) => {
              // Cache the response
              this.cacheService.set(cacheKey, {
                data: response,
                timestamp: Date.now(),
              });

              return ProductsApiActions.loadProductsSuccess({ response });
            }),
            catchError((error) => {
              // Log error for monitoring
              this.analytics.trackError("load_products_failed", error);

              return of(
                ProductsApiActions.loadProductsFailure({
                  error: this.formatError(error),
                })
              );
            })
          );
      })
    )
  );

  // Search with debouncing
  searchProducts$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsPageActions.searchProducts),
      debounceTime(300),
      distinctUntilChanged((prev, curr) => prev.query === curr.query),
      map(({ query }) =>
        ProductsPageActions.loadProducts({
          filter: { search: query },
          page: 1,
        })
      )
    )
  );

  // Load product detail with optimistic loading
  loadProductDetail$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsPageActions.selectProduct),
      mergeMap(({ productId }) =>
        this.productsService.getProductById(productId).pipe(
          map((product) =>
            ProductsApiActions.loadProductDetailSuccess({ product })
          ),
          catchError((error) =>
            of(
              ProductsApiActions.loadProductDetailFailure({
                error: this.formatError(error),
              })
            )
          )
        )
      )
    )
  );

  // Create product with optimistic updates
  createProduct$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsPageActions.createProduct),
      exhaustMap(({ productData }) =>
        this.productsService.createProduct(productData).pipe(
          map((product) => {
            this.notificationService.showSuccess(
              "Product created successfully"
            );
            this.analytics.track("product_created", { productId: product.id });

            return ProductsApiActions.createProductSuccess({ product });
          }),
          catchError((error) => {
            this.notificationService.showError("Failed to create product");
            this.analytics.trackError("create_product_failed", error);

            return of(
              ProductsApiActions.createProductFailure({
                error: this.formatError(error),
              })
            );
          })
        )
      )
    )
  );

  // Update product with conflict resolution
  updateProduct$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsPageActions.updateProduct),
      concatMap(({ productId, changes }) =>
        this.productsService.updateProduct(productId, changes).pipe(
          map((product) => {
            this.notificationService.showSuccess(
              "Product updated successfully"
            );
            this.analytics.track("product_updated", { productId });

            // Invalidate related cache entries
            this.store.dispatch(
              ProductsCacheActions.updateCacheEntry({ product })
            );

            return ProductsApiActions.updateProductSuccess({ product });
          }),
          catchError((error) => {
            this.notificationService.showError("Failed to update product");
            this.analytics.trackError("update_product_failed", error);

            return of(
              ProductsApiActions.updateProductFailure({
                error: this.formatError(error),
              })
            );
          })
        )
      )
    )
  );

  // Delete product with confirmation
  deleteProduct$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsPageActions.deleteProduct),
      filter(({ confirmed }) => confirmed === true),
      mergeMap(({ productId }) =>
        this.productsService.deleteProduct(productId).pipe(
          map(() => {
            this.notificationService.showSuccess(
              "Product deleted successfully"
            );
            this.analytics.track("product_deleted", { productId });

            // Clear cache entries
            this.store.dispatch(
              ProductsCacheActions.removeCacheEntry({ productId })
            );

            return ProductsApiActions.deleteProductSuccess({ productId });
          }),
          catchError((error) => {
            this.notificationService.showError("Failed to delete product");
            this.analytics.trackError("delete_product_failed", error);

            return of(
              ProductsApiActions.deleteProductFailure({
                error: this.formatError(error),
              })
            );
          })
        )
      )
    )
  );

  // Cache invalidation
  invalidateCache$ = createEffect(
    () =>
      this.actions$.pipe(
        ofType(
          ProductsApiActions.createProductSuccess,
          ProductsApiActions.updateProductSuccess,
          ProductsApiActions.deleteProductSuccess
        ),
        tap(() => {
          // Clear all products cache when data is modified
          this.cacheService.clearPattern("products_*");
        })
      ),
    { dispatch: false }
  );

  // Preload related products
  preloadRelatedProducts$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsApiActions.loadProductDetailSuccess),
      switchMap(({ product }) => {
        // Preload products from the same category
        return this.productsService
          .getProducts({
            category: product.category,
            pageSize: 5,
          })
          .pipe(
            map((response) =>
              ProductsCacheActions.preloadProducts({
                productIds: response.products.map((p) => p.id),
              })
            ),
            catchError(() => EMPTY)
          );
      })
    )
  );

  // Analytics tracking
  trackProductViews$ = createEffect(
    () =>
      this.actions$.pipe(
        ofType(ProductsPageActions.selectProduct),
        tap(({ productId }) => {
          this.analytics.track("product_viewed", { productId });
        })
      ),
    { dispatch: false }
  );

  trackProductInteractions$ = createEffect(
    () =>
      this.actions$.pipe(
        ofType(
          ProductsPageActions.addProductToCart,
          ProductsPageActions.toggleProductFavorite
        ),
        tap((action) => {
          if (action.type === "[Products Page] Add Product To Cart") {
            this.analytics.track("product_added_to_cart", {
              productId: action.productId,
              quantity: action.quantity,
            });
          } else if (
            action.type === "[Products Page] Toggle Product Favorite"
          ) {
            this.analytics.track("product_favorited", {
              productId: action.productId,
            });
          }
        })
      ),
    { dispatch: false }
  );

  // Auto-refresh data periodically
  autoRefresh$ = createEffect(() =>
    timer(0, 5 * 60 * 1000).pipe(
      // Every 5 minutes
      switchMap(() =>
        this.store.select(selectCurrentFilter).pipe(
          withLatestFrom(this.store.select(selectPaginationInfo)),
          map(([filter, pagination]) =>
            ProductsPageActions.loadProducts({ filter, ...pagination })
          )
        )
      )
    )
  );

  // Helper methods
  private generateCacheKey(filter: any, pagination: any): string {
    return `products_${JSON.stringify({ filter, pagination })}`;
  }

  private isCacheValid(timestamp: number, expiry: number): boolean {
    return Date.now() - timestamp < expiry;
  }

  private formatError(error: any): string {
    if (error.error?.message) {
      return error.error.message;
    }

    if (error.message) {
      return error.message;
    }

    return "An unexpected error occurred";
  }
}
```

## 🔄 **Alternative: Signal-based State (Angular 17+)**

### **1. 🎯 Signal Store Implementation**

```typescript
// src/app/store/signal-products.store.ts - Modern Signal-based Store
import { Injectable, computed, signal, effect, inject } from "@angular/core";
import { rxMethod } from "@ngrx/signals/rxjs-interop";
import {
  pipe,
  tap,
  switchMap,
  map,
  catchError,
  of,
  debounceTime,
  distinctUntilChanged,
} from "rxjs";

import { Product, ProductFilter } from "./products.actions";
import { ProductsService } from "../services/products.service";

export interface ProductsSignalState {
  products: Product[];
  loading: boolean;
  error: string | null;
  filter: ProductFilter;
  selectedId: string | null;
  page: number;
  pageSize: number;
  totalCount: number;
  favoriteIds: string[];
}

@Injectable({
  providedIn: "root",
})
export class ProductsSignalStore {
  private productsService = inject(ProductsService);

  // Private signals for state management
  private readonly _products = signal<Product[]>([]);
  private readonly _loading = signal(false);
  private readonly _error = signal<string | null>(null);
  private readonly _filter = signal<ProductFilter>({});
  private readonly _selectedId = signal<string | null>(null);
  private readonly _page = signal(1);
  private readonly _pageSize = signal(20);
  private readonly _totalCount = signal(0);
  private readonly _favoriteIds = signal<string[]>([]);

  // Public readonly selectors (computed signals)
  readonly products = this._products.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly error = this._error.asReadonly();
  readonly filter = this._filter.asReadonly();
  readonly selectedId = this._selectedId.asReadonly();
  readonly page = this._page.asReadonly();
  readonly pageSize = this._pageSize.asReadonly();
  readonly totalCount = this._totalCount.asReadonly();
  readonly favoriteIds = this._favoriteIds.asReadonly();

  // Computed derived state
  readonly filteredProducts = computed(() => {
    const products = this._products();
    const filter = this._filter();

    if (!filter || Object.keys(filter).length === 0) {
      return products;
    }

    return products.filter((product) => {
      if (filter.search) {
        const searchTerm = filter.search.toLowerCase();
        if (
          !product.name.toLowerCase().includes(searchTerm) &&
          !product.description.toLowerCase().includes(searchTerm)
        ) {
          return false;
        }
      }

      if (filter.category && product.category !== filter.category) {
        return false;
      }

      if (filter.minPrice && product.price < filter.minPrice) {
        return false;
      }

      if (filter.maxPrice && product.price > filter.maxPrice) {
        return false;
      }

      if (filter.inStock && product.stock <= 0) {
        return false;
      }

      return true;
    });
  });

  readonly selectedProduct = computed(() => {
    const products = this._products();
    const selectedId = this._selectedId();
    return selectedId
      ? products.find((p) => p.id === selectedId) || null
      : null;
  });

  readonly favoriteProducts = computed(() => {
    const products = this._products();
    const favoriteIds = this._favoriteIds();
    return products.filter((p) => favoriteIds.includes(p.id));
  });

  readonly paginatedProducts = computed(() => {
    const filtered = this.filteredProducts();
    const page = this._page();
    const pageSize = this._pageSize();

    const start = (page - 1) * pageSize;
    const end = start + pageSize;

    return filtered.slice(start, end);
  });

  readonly paginationInfo = computed(() => ({
    currentPage: this._page(),
    pageSize: this._pageSize(),
    totalCount: this._totalCount(),
    totalPages: Math.ceil(this._totalCount() / this._pageSize()),
    hasNext: this._page() < Math.ceil(this._totalCount() / this._pageSize()),
    hasPrevious: this._page() > 1,
  }));

  readonly isProductFavorite = computed(() => {
    const selectedId = this._selectedId();
    const favoriteIds = this._favoriteIds();
    return selectedId ? favoriteIds.includes(selectedId) : false;
  });

  // Effects for side effects
  constructor() {
    // Auto-persist favorites to localStorage
    effect(() => {
      const favoriteIds = this._favoriteIds();
      localStorage.setItem("favorite-products", JSON.stringify(favoriteIds));
    });

    // Load favorites on init
    this.loadFavorites();
  }

  // Reactive methods using rxMethod
  readonly loadProducts = rxMethod<{ filter?: ProductFilter; page?: number }>(
    pipe(
      debounceTime(300),
      distinctUntilChanged(),
      tap(() => {
        this._loading.set(true);
        this._error.set(null);
      }),
      switchMap(({ filter = {}, page = 1 }) => {
        // Update filter and page
        this._filter.set(filter);
        this._page.set(page);

        return this.productsService
          .getProducts({
            ...filter,
            page,
            pageSize: this._pageSize(),
          })
          .pipe(
            map((response) => {
              this._products.set(response.products);
              this._totalCount.set(response.totalCount);
              this._loading.set(false);
              return response;
            }),
            catchError((error) => {
              this._error.set(this.formatError(error));
              this._loading.set(false);
              return of(null);
            })
          );
      })
    )
  );

  readonly searchProducts = rxMethod<string>(
    pipe(
      debounceTime(500),
      distinctUntilChanged(),
      tap((query) => {
        const currentFilter = this._filter();
        this.loadProducts({
          filter: { ...currentFilter, search: query },
          page: 1,
        });
      })
    )
  );

  // Imperative methods for direct state updates
  selectProduct(productId: string): void {
    this._selectedId.set(productId);
  }

  clearSelection(): void {
    this._selectedId.set(null);
  }

  updateFilter(filter: ProductFilter): void {
    const currentFilter = this._filter();
    this.loadProducts({
      filter: { ...currentFilter, ...filter },
      page: 1,
    });
  }

  changePage(page: number): void {
    if (page < 1 || page > this.paginationInfo().totalPages) {
      return;
    }

    this._page.set(page);
    this.loadProducts({
      filter: this._filter(),
      page,
    });
  }

  changePageSize(pageSize: number): void {
    this._pageSize.set(pageSize);
    this._page.set(1);
    this.loadProducts({
      filter: this._filter(),
      page: 1,
    });
  }

  toggleFavorite(productId: string): void {
    const currentFavorites = this._favoriteIds();
    const isFavorite = currentFavorites.includes(productId);

    if (isFavorite) {
      this._favoriteIds.set(currentFavorites.filter((id) => id !== productId));
    } else {
      this._favoriteIds.set([...currentFavorites, productId]);
    }
  }

  addProduct(product: Product): void {
    this._products.update((products) => [...products, product]);
    this._totalCount.update((count) => count + 1);
  }

  updateProduct(updatedProduct: Product): void {
    this._products.update((products) =>
      products.map((p) => (p.id === updatedProduct.id ? updatedProduct : p))
    );
  }

  removeProduct(productId: string): void {
    this._products.update((products) =>
      products.filter((p) => p.id !== productId)
    );
    this._totalCount.update((count) => Math.max(0, count - 1));

    if (this._selectedId() === productId) {
      this._selectedId.set(null);
    }
  }

  clearError(): void {
    this._error.set(null);
  }

  reset(): void {
    this._products.set([]);
    this._loading.set(false);
    this._error.set(null);
    this._filter.set({});
    this._selectedId.set(null);
    this._page.set(1);
    this._totalCount.set(0);
  }

  // Private helpers
  private loadFavorites(): void {
    try {
      const stored = localStorage.getItem("favorite-products");
      if (stored) {
        const favoriteIds = JSON.parse(stored);
        this._favoriteIds.set(favoriteIds);
      }
    } catch (error) {
      console.warn("Failed to load favorite products:", error);
    }
  }

  private formatError(error: any): string {
    return error?.message || "An unexpected error occurred";
  }
}
```

### **2. 🎮 Component Integration**

```typescript
// src/app/components/products/products.component.ts - Using Signal Store
import { Component, computed, effect, inject, signal } from "@angular/core";
import { CommonModule } from "@angular/common";
import { FormsModule } from "@angular/forms";

import { ProductsSignalStore } from "../../store/signal-products.store";

@Component({
  selector: "app-products",
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div class="products-page">
      <!-- Search & Filters -->
      <div class="filters">
        <input
          type="text"
          placeholder="Search products..."
          [value]="searchQuery()"
          (input)="onSearchInput($event)"
          class="search-input"
        />

        <select
          [value]="selectedCategory()"
          (change)="onCategoryChange($event)"
          class="category-filter"
        >
          <option value="">All Categories</option>
          <option *ngFor="let category of categories()" [value]="category">
            {{ category }}
          </option>
        </select>

        <div class="price-filter">
          <label>
            Min Price:
            <input
              type="number"
              [value]="minPrice()"
              (input)="onMinPriceChange($event)"
            />
          </label>
          <label>
            Max Price:
            <input
              type="number"
              [value]="maxPrice()"
              (input)="onMaxPriceChange($event)"
            />
          </label>
        </div>

        <label class="stock-filter">
          <input
            type="checkbox"
            [checked]="inStockOnly()"
            (change)="onStockFilterChange($event)"
          />
          In Stock Only
        </label>
      </div>

      <!-- Loading State -->
      <div *ngIf="store.loading()" class="loading">
        <div class="spinner"></div>
        <p>Loading products...</p>
      </div>

      <!-- Error State -->
      <div *ngIf="store.error()" class="error">
        <p>{{ store.error() }}</p>
        <button (click)="retry()" class="retry-button">Try Again</button>
      </div>

      <!-- Products Grid -->
      <div *ngIf="!store.loading() && !store.error()" class="products-grid">
        <div
          *ngFor="
            let product of store.paginatedProducts();
            trackBy: trackByProduct
          "
          class="product-card"
          [class.selected]="product.id === store.selectedId()"
        >
          <img
            [src]="product.imageUrl"
            [alt]="product.name"
            class="product-image"
          />

          <div class="product-info">
            <h3>{{ product.name }}</h3>
            <p class="description">{{ product.description }}</p>
            <div class="price">\${{ product.price }}</div>
            <div class="rating">★ {{ product.rating }}/5</div>
            <div class="stock">{{ product.stock }} in stock</div>
          </div>

          <div class="product-actions">
            <button
              (click)="selectProduct(product.id)"
              [class.active]="product.id === store.selectedId()"
              class="select-btn"
            >
              {{ product.id === store.selectedId() ? "Selected" : "Select" }}
            </button>

            <button
              (click)="toggleFavorite(product.id)"
              [class.favorite]="store.favoriteIds().includes(product.id)"
              class="favorite-btn"
            >
              {{ store.favoriteIds().includes(product.id) ? "♥" : "♡" }}
            </button>
          </div>
        </div>
      </div>

      <!-- Pagination -->
      <div *ngIf="store.paginationInfo().totalPages > 1" class="pagination">
        <button
          (click)="changePage(store.paginationInfo().currentPage - 1)"
          [disabled]="!store.paginationInfo().hasPrevious"
          class="page-btn"
        >
          Previous
        </button>

        <span class="page-info">
          Page {{ store.paginationInfo().currentPage }} of
          {{ store.paginationInfo().totalPages }}
        </span>

        <button
          (click)="changePage(store.paginationInfo().currentPage + 1)"
          [disabled]="!store.paginationInfo().hasNext"
          class="page-btn"
        >
          Next
        </button>
      </div>

      <!-- Selected Product Details -->
      <div *ngIf="store.selectedProduct()" class="selected-product-details">
        <h2>{{ store.selectedProduct()!.name }}</h2>
        <p>{{ store.selectedProduct()!.description }}</p>
        <div class="details-grid">
          <div>Price: \${{ store.selectedProduct()!.price }}</div>
          <div>Category: {{ store.selectedProduct()!.category }}</div>
          <div>Stock: {{ store.selectedProduct()!.stock }}</div>
          <div>Rating: {{ store.selectedProduct()!.rating }}/5</div>
        </div>
      </div>

      <!-- Favorites Summary -->
      <div
        *ngIf="store.favoriteProducts().length > 0"
        class="favorites-summary"
      >
        <h3>Favorites ({{ store.favoriteProducts().length }})</h3>
        <div class="favorite-items">
          <span
            *ngFor="let fav of store.favoriteProducts()"
            class="favorite-item"
          >
            {{ fav.name }}
          </span>
        </div>
      </div>
    </div>
  `,
  styleUrls: ["./products.component.scss"],
})
export class ProductsComponent {
  store = inject(ProductsSignalStore);

  // Local UI state
  searchQuery = signal("");
  selectedCategory = signal("");
  minPrice = signal<number | null>(null);
  maxPrice = signal<number | null>(null);
  inStockOnly = signal(false);

  // Computed categories from products
  categories = computed(() => {
    const products = this.store.products();
    return [...new Set(products.map((p) => p.category))].sort();
  });

  constructor() {
    // Load initial data
    this.loadProducts();

    // Auto-apply filters when they change
    effect(() => {
      const filter = {
        search: this.searchQuery() || undefined,
        category: this.selectedCategory() || undefined,
        minPrice: this.minPrice() || undefined,
        maxPrice: this.maxPrice() || undefined,
        inStock: this.inStockOnly() || undefined,
      };

      // Remove undefined values
      const cleanFilter = Object.fromEntries(
        Object.entries(filter).filter(([_, v]) => v !== undefined)
      );

      if (Object.keys(cleanFilter).length > 0) {
        this.store.updateFilter(cleanFilter);
      }
    });
  }

  onSearchInput(event: Event): void {
    const target = event.target as HTMLInputElement;
    this.searchQuery.set(target.value);
  }

  onCategoryChange(event: Event): void {
    const target = event.target as HTMLSelectElement;
    this.selectedCategory.set(target.value);
  }

  onMinPriceChange(event: Event): void {
    const target = event.target as HTMLInputElement;
    const value = target.value ? parseFloat(target.value) : null;
    this.minPrice.set(value);
  }

  onMaxPriceChange(event: Event): void {
    const target = event.target as HTMLInputElement;
    const value = target.value ? parseFloat(target.value) : null;
    this.maxPrice.set(value);
  }

  onStockFilterChange(event: Event): void {
    const target = event.target as HTMLInputElement;
    this.inStockOnly.set(target.checked);
  }

  selectProduct(productId: string): void {
    this.store.selectProduct(productId);
  }

  toggleFavorite(productId: string): void {
    this.store.toggleFavorite(productId);
  }

  changePage(page: number): void {
    this.store.changePage(page);
  }

  loadProducts(): void {
    this.store.loadProducts({ page: 1 });
  }

  retry(): void {
    this.store.clearError();
    this.loadProducts();
  }

  trackByProduct(index: number, product: any): string {
    return product.id;
  }
}
```

## 📊 **State Management Comparison**

| Feature            | NgRx      | Signals   | Akita    | Custom Service |
| ------------------ | --------- | --------- | -------- | -------------- |
| **Learning Curve** | Steep     | Moderate  | Moderate | Easy           |
| **Boilerplate**    | High      | Low       | Medium   | Very Low       |
| **DevTools**       | Excellent | Basic     | Good     | None           |
| **Time Travel**    | Yes       | No        | Yes      | No             |
| **Performance**    | Excellent | Excellent | Good     | Variable       |
| **Type Safety**    | Excellent | Excellent | Good     | Variable       |

## 🎯 **Part 1 Summary**

This covers:

- **🏗️ NgRx Setup** - Complete store architecture with actions, reducers, and selectors
- **⚡ Effects Management** - Side effects with caching, error handling, and analytics
- **🎯 Signal Store** - Modern Angular 17+ signal-based state management
- **🎮 Component Integration** - Practical usage patterns and reactive UI

**Coming in Part 2:**

- Akita implementation
- Custom state management patterns
- Performance optimization strategies
- Testing state management code
