# 📱 Progressive Web App Features in Angular: Complete Guide

## 🎯 **Question Overview**

_"How do you implement Progressive Web App (PWA) features in Angular applications?"_

## 🔍 **Understanding PWA in Angular**

Progressive Web Apps are **web applications** that use **modern web capabilities** to provide a **native app-like experience**. Angular provides **built-in PWA support** through **service workers**, **app shell architecture**, and **offline capabilities**.

Modern PWAs offer **installability**, **offline functionality**, **push notifications**, and **background sync**! 📲

## 🏗️ **PWA Setup & Configuration**

### **1. 🚀 Adding PWA Support**

```bash
# Add PWA support to existing Angular app
ng add @angular/pwa

# What this command does:
# - Adds @angular/service-worker dependency
# - Creates ngsw-config.json for service worker config
# - Updates angular.json with service worker build options
# - Adds manifest.webmanifest for app metadata
# - Creates icon files for various sizes
# - Updates index.html with manifest and theme-color meta tags
```

```json
// angular.json - Service Worker Configuration
{
  "projects": {
    "my-pwa-app": {
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "serviceWorker": true,
            "ngswConfigPath": "ngsw-config.json"
          }
        }
      }
    }
  }
}
```

```json
// ngsw-config.json - Service Worker Configuration
{
  "$schema": "./node_modules/@angular/service-worker/config/schema.json",
  "index": "/index.html",
  "assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",
      "resources": {
        "files": [
          "/favicon.ico",
          "/index.html",
          "/manifest.webmanifest",
          "/*.css",
          "/*.js"
        ]
      }
    },
    {
      "name": "assets",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "resources": {
        "files": [
          "/assets/**",
          "/*.(eot|svg|cur|jpg|png|webp|gif|otf|ttf|woff|woff2|ani)"
        ]
      }
    }
  ],
  "dataGroups": [
    {
      "name": "api-cache",
      "urls": [
        "https://api.example.com/users/**",
        "https://api.example.com/products/**"
      ],
      "cacheConfig": {
        "strategy": "performance",
        "maxSize": 100,
        "maxAge": "3d",
        "timeout": "10s"
      }
    },
    {
      "name": "api-freshness",
      "urls": [
        "https://api.example.com/notifications/**",
        "https://api.example.com/realtime/**"
      ],
      "cacheConfig": {
        "strategy": "freshness",
        "maxSize": 50,
        "maxAge": "1h",
        "timeout": "5s"
      }
    }
  ],
  "navigationUrls": ["/**", "!/**/*.*", "!/**/*__*", "!/**/*__*/**"]
}
```

### **2. 📱 App Manifest Configuration**

```json
// src/manifest.webmanifest - PWA Manifest
{
  "name": "Modern E-Commerce PWA",
  "short_name": "ECommerce",
  "description": "A modern e-commerce progressive web application built with Angular",
  "theme_color": "#1976d2",
  "background_color": "#fafafa",
  "display": "standalone",
  "scope": "./",
  "start_url": "./",
  "orientation": "portrait-primary",
  "categories": ["shopping", "business"],
  "lang": "en-US",
  "dir": "ltr",
  "icons": [
    {
      "src": "assets/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ],
  "shortcuts": [
    {
      "name": "View Products",
      "short_name": "Products",
      "description": "Browse our product catalog",
      "url": "/products",
      "icons": [
        {
          "src": "assets/icons/products-96x96.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "My Orders",
      "short_name": "Orders",
      "description": "View your order history",
      "url": "/orders",
      "icons": [
        {
          "src": "assets/icons/orders-96x96.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Shopping Cart",
      "short_name": "Cart",
      "description": "View items in your cart",
      "url": "/cart",
      "icons": [
        {
          "src": "assets/icons/cart-96x96.png",
          "sizes": "96x96"
        }
      ]
    }
  ],
  "screenshots": [
    {
      "src": "assets/screenshots/desktop-home.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide",
      "label": "Desktop Home Page"
    },
    {
      "src": "assets/screenshots/mobile-products.png",
      "sizes": "390x844",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "Mobile Products Page"
    }
  ],
  "protocol_handlers": [
    {
      "protocol": "web+shopping",
      "url": "/product?id=%s"
    }
  ],
  "prefer_related_applications": false
}
```

## 🔄 **Service Worker Management**

### **1. 📡 Service Worker Service**

```typescript
// src/app/services/pwa.service.ts - PWA Management Service
import { Injectable, signal, inject } from "@angular/core";
import { SwUpdate, SwPush, VersionReadyEvent } from "@angular/service-worker";
import { filter, map } from "rxjs/operators";
import { MatSnackBar } from "@angular/material/snack-bar";
import { Platform } from "@angular/cdk/platform";

export interface PWAInstallPrompt {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: "accepted" | "dismissed" }>;
}

export interface UpdateInfo {
  available: boolean;
  current: string;
  available_version: string;
}

export interface PushSubscriptionData {
  endpoint: string;
  keys: {
    p256dh: string;
    auth: string;
  };
}

@Injectable({
  providedIn: "root",
})
export class PWAService {
  private swUpdate = inject(SwUpdate);
  private swPush = inject(SwPush);
  private snackBar = inject(MatSnackBar);
  private platform = inject(Platform);

  // PWA State Signals
  readonly isOnline = signal(navigator.onLine);
  readonly isInstalled = signal(this.checkIfInstalled());
  readonly updateAvailable = signal<UpdateInfo | null>(null);
  readonly installPromptEvent = signal<PWAInstallPrompt | null>(null);
  readonly pushNotificationsEnabled = signal(false);

  // PWA Capabilities
  readonly isPWASupported = signal(this.checkPWASupport());
  readonly canInstall = signal(false);
  readonly hasNotificationSupport = signal("Notification" in window);
  readonly hasBackgroundSyncSupport = signal(
    "serviceWorker" in navigator &&
      "sync" in window.ServiceWorkerRegistration.prototype
  );

  constructor() {
    this.initializePWAFeatures();
    this.setupEventListeners();
  }

  private initializePWAFeatures(): void {
    // Check for update availability
    if (this.swUpdate.isEnabled) {
      this.checkForUpdates();
      this.setupUpdateNotifications();
    }

    // Setup push notifications
    if (this.swPush.isEnabled) {
      this.setupPushNotifications();
    }

    // Setup install prompt handling
    this.setupInstallPrompt();

    // Monitor online/offline status
    this.setupNetworkStatusMonitoring();
  }

  private setupEventListeners(): void {
    // Listen for app updates
    this.swUpdate.versionUpdates
      .pipe(
        filter((evt): evt is VersionReadyEvent => evt.type === "VERSION_READY")
      )
      .subscribe((evt) => {
        this.updateAvailable.set({
          available: true,
          current: evt.currentVersion.hash,
          available_version: evt.latestVersion.hash,
        });

        this.showUpdateNotification();
      });

    // Listen for push messages
    this.swPush.messages.subscribe((message) => {
      this.handlePushMessage(message);
    });

    // Listen for notification clicks
    this.swPush.notificationClicks.subscribe((click) => {
      this.handleNotificationClick(click);
    });
  }

  // Update Management
  async checkForUpdates(): Promise<boolean> {
    if (!this.swUpdate.isEnabled) return false;

    try {
      const updateFound = await this.swUpdate.checkForUpdate();
      return updateFound;
    } catch (error) {
      console.error("Error checking for updates:", error);
      return false;
    }
  }

  async applyUpdate(): Promise<void> {
    if (!this.swUpdate.isEnabled) return;

    try {
      await this.swUpdate.activateUpdate();
      document.location.reload();
    } catch (error) {
      console.error("Error applying update:", error);
      throw error;
    }
  }

  private showUpdateNotification(): void {
    const snackBarRef = this.snackBar.open(
      "A new version is available!",
      "Update Now",
      {
        duration: 0,
        horizontalPosition: "center",
        verticalPosition: "bottom",
      }
    );

    snackBarRef.onAction().subscribe(() => {
      this.applyUpdate();
    });
  }

  // Installation Management
  async promptInstall(): Promise<boolean> {
    const promptEvent = this.installPromptEvent();
    if (!promptEvent) return false;

    try {
      await promptEvent.prompt();
      const choice = await promptEvent.userChoice;

      if (choice.outcome === "accepted") {
        this.isInstalled.set(true);
        this.installPromptEvent.set(null);
        return true;
      }

      return false;
    } catch (error) {
      console.error("Error during install prompt:", error);
      return false;
    }
  }

  private setupInstallPrompt(): void {
    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      this.installPromptEvent.set(e as any);
      this.canInstall.set(true);
    });

    window.addEventListener("appinstalled", () => {
      this.isInstalled.set(true);
      this.installPromptEvent.set(null);
      this.canInstall.set(false);
    });
  }

  private checkIfInstalled(): boolean {
    return (
      window.matchMedia("(display-mode: standalone)").matches ||
      window.matchMedia("(display-mode: fullscreen)").matches ||
      (window as any).navigator?.standalone === true
    );
  }

  // Push Notifications
  async requestPushSubscription(): Promise<PushSubscriptionData | null> {
    if (!this.swPush.isEnabled) return null;

    try {
      const permission = await Notification.requestPermission();
      if (permission !== "granted") return null;

      const subscription = await this.swPush.requestSubscription({
        serverPublicKey: this.getVAPIDPublicKey(),
      });

      const subscriptionData: PushSubscriptionData = {
        endpoint: subscription.endpoint,
        keys: {
          p256dh: this.arrayBufferToBase64(subscription.getKey("p256dh")!),
          auth: this.arrayBufferToBase64(subscription.getKey("auth")!),
        },
      };

      this.pushNotificationsEnabled.set(true);
      return subscriptionData;
    } catch (error) {
      console.error("Error subscribing to push notifications:", error);
      return null;
    }
  }

  async unsubscribeFromPush(): Promise<boolean> {
    if (!this.swPush.isEnabled) return false;

    try {
      await this.swPush.unsubscribe();
      this.pushNotificationsEnabled.set(false);
      return true;
    } catch (error) {
      console.error("Error unsubscribing from push notifications:", error);
      return false;
    }
  }

  private setupPushNotifications(): void {
    // Check if already subscribed
    this.swPush.subscription.subscribe((subscription) => {
      this.pushNotificationsEnabled.set(!!subscription);
    });
  }

  private handlePushMessage(message: any): void {
    console.log("Received push message:", message);

    // Handle different types of push messages
    if (message.type === "order-update") {
      this.handleOrderUpdate(message.data);
    } else if (message.type === "promotion") {
      this.handlePromotion(message.data);
    }
  }

  private handleNotificationClick(click: any): void {
    console.log("Notification clicked:", click);

    // Navigate to relevant page based on notification action
    if (click.action === "view-order") {
      window.open(`/orders/${click.notification.data.orderId}`, "_blank");
    }
  }

  // Network Status
  private setupNetworkStatusMonitoring(): void {
    window.addEventListener("online", () => {
      this.isOnline.set(true);
      this.syncWhenOnline();
    });

    window.addEventListener("offline", () => {
      this.isOnline.set(false);
    });
  }

  private async syncWhenOnline(): Promise<void> {
    if (
      "serviceWorker" in navigator &&
      "sync" in window.ServiceWorkerRegistration.prototype
    ) {
      try {
        const registration = await navigator.serviceWorker.ready;
        await registration.sync.register("background-sync");
      } catch (error) {
        console.error("Background sync registration failed:", error);
      }
    }
  }

  // Utility Methods
  private checkPWASupport(): boolean {
    return (
      this.platform.isBrowser &&
      "serviceWorker" in navigator &&
      "PushManager" in window &&
      "Notification" in window
    );
  }

  private getVAPIDPublicKey(): string {
    // Replace with your actual VAPID public key
    return "BEl62iUYgUivxIkv69yViEuiBIa40HI80NM9VqwJBe_HTpSHyLjKiE1VBNi1k4DhQq8QcSmb5oDgqJ-6H4qmGM8";
  }

  private arrayBufferToBase64(buffer: ArrayBuffer): string {
    const bytes = new Uint8Array(buffer);
    const binary = bytes.reduce(
      (acc, byte) => acc + String.fromCharCode(byte),
      ""
    );
    return btoa(binary);
  }

  private handleOrderUpdate(data: any): void {
    // Handle order update notification
    this.snackBar.open(
      `Order #${data.orderId} has been ${data.status}`,
      "View Order",
      { duration: 5000 }
    );
  }

  private handlePromotion(data: any): void {
    // Handle promotional notification
    this.snackBar.open(data.message, "View Offer", { duration: 10000 });
  }

  // Public API
  getInstallationStatus() {
    return {
      isInstalled: this.isInstalled(),
      canInstall: this.canInstall(),
      isPWASupported: this.isPWASupported(),
    };
  }

  getNetworkStatus() {
    return {
      isOnline: this.isOnline(),
      hasBackgroundSync: this.hasBackgroundSyncSupport(),
    };
  }

  getNotificationStatus() {
    return {
      isEnabled: this.pushNotificationsEnabled(),
      isSupported: this.hasNotificationSupport(),
    };
  }
}
```

### **2. 🎛️ PWA Control Component**

```typescript
// src/app/components/pwa-controls/pwa-controls.component.ts
import { Component, inject, signal } from "@angular/core";
import { CommonModule } from "@angular/common";
import { MatButtonModule } from "@angular/material/button";
import { MatIconModule } from "@angular/material/icon";
import { MatBadgeModule } from "@angular/material/badge";
import { MatTooltipModule } from "@angular/material/tooltip";
import { MatSnackBar } from "@angular/material/snack-bar";

import { PWAService } from "../../services/pwa.service";

@Component({
  selector: "app-pwa-controls",
  standalone: true,
  imports: [
    CommonModule,
    MatButtonModule,
    MatIconModule,
    MatBadgeModule,
    MatTooltipModule,
  ],
  template: `
    <div class="pwa-controls">
      <!-- Install App Button -->
      <button
        *ngIf="pwaService.canInstall() && !pwaService.isInstalled()"
        mat-raised-button
        color="primary"
        (click)="installApp()"
        matTooltip="Install this app for a better experience"
        class="install-button"
      >
        <mat-icon>download</mat-icon>
        Install App
      </button>

      <!-- Update Available Badge -->
      <button
        *ngIf="pwaService.updateAvailable()"
        mat-raised-button
        color="accent"
        (click)="applyUpdate()"
        matTooltip="Update available - click to install"
        class="update-button"
      >
        <mat-icon matBadge="!" matBadgeColor="warn">system_update</mat-icon>
        Update Available
      </button>

      <!-- Offline Indicator -->
      <div
        *ngIf="!pwaService.isOnline()"
        class="offline-indicator"
        matTooltip="You are currently offline"
      >
        <mat-icon>cloud_off</mat-icon>
        Offline Mode
      </div>

      <!-- Notification Toggle -->
      <button
        mat-icon-button
        [color]="pwaService.pushNotificationsEnabled() ? 'primary' : 'basic'"
        (click)="toggleNotifications()"
        [matTooltip]="getNotificationTooltip()"
        class="notification-button"
      >
        <mat-icon>
          {{
            pwaService.pushNotificationsEnabled()
              ? "notifications"
              : "notifications_off"
          }}
        </mat-icon>
      </button>

      <!-- PWA Info -->
      <button
        *ngIf="pwaService.isInstalled()"
        mat-icon-button
        matTooltip="Running as installed app"
        class="pwa-info"
        disabled
      >
        <mat-icon color="primary">phone_android</mat-icon>
      </button>
    </div>
  `,
  styleUrls: ["./pwa-controls.component.scss"],
})
export class PwaControlsComponent {
  pwaService = inject(PWAService);
  snackBar = inject(MatSnackBar);

  private isProcessing = signal(false);

  async installApp(): Promise<void> {
    if (this.isProcessing()) return;

    this.isProcessing.set(true);

    try {
      const success = await this.pwaService.promptInstall();

      if (success) {
        this.snackBar.open("App installed successfully!", "Great!", {
          duration: 3000,
        });
      } else {
        this.snackBar.open("Installation cancelled", "OK", { duration: 2000 });
      }
    } catch (error) {
      this.snackBar.open("Installation failed. Please try again.", "OK", {
        duration: 3000,
      });
    } finally {
      this.isProcessing.set(false);
    }
  }

  async applyUpdate(): Promise<void> {
    if (this.isProcessing()) return;

    this.isProcessing.set(true);

    try {
      await this.pwaService.applyUpdate();
      // App will reload automatically
    } catch (error) {
      this.snackBar.open("Update failed. Please refresh the page.", "Refresh", {
        duration: 5000,
        action: () => window.location.reload(),
      });
    } finally {
      this.isProcessing.set(false);
    }
  }

  async toggleNotifications(): Promise<void> {
    if (this.isProcessing()) return;

    this.isProcessing.set(true);

    try {
      const isEnabled = this.pwaService.pushNotificationsEnabled();

      if (isEnabled) {
        const success = await this.pwaService.unsubscribeFromPush();
        if (success) {
          this.snackBar.open("Notifications disabled", "OK", {
            duration: 2000,
          });
        }
      } else {
        const subscription = await this.pwaService.requestPushSubscription();
        if (subscription) {
          // Send subscription to your server
          await this.sendSubscriptionToServer(subscription);
          this.snackBar.open("Notifications enabled!", "Great!", {
            duration: 3000,
          });
        } else {
          this.snackBar.open("Notification permission denied", "OK", {
            duration: 3000,
          });
        }
      }
    } catch (error) {
      this.snackBar.open("Failed to toggle notifications", "OK", {
        duration: 3000,
      });
    } finally {
      this.isProcessing.set(false);
    }
  }

  private async sendSubscriptionToServer(subscription: any): Promise<void> {
    // Implement server subscription logic
    console.log("Sending subscription to server:", subscription);
  }

  getNotificationTooltip(): string {
    if (!this.pwaService.hasNotificationSupport()) {
      return "Notifications not supported";
    }

    return this.pwaService.pushNotificationsEnabled()
      ? "Disable notifications"
      : "Enable notifications";
  }
}
```

## 🗄️ **Offline Functionality**

### **1. 📦 Offline Storage Service**

```typescript
// src/app/services/offline-storage.service.ts
import { Injectable, signal } from "@angular/core";

export interface OfflineAction {
  id: string;
  type: string;
  data: any;
  timestamp: number;
  attempts: number;
  maxAttempts: number;
}

export interface OfflineData<T> {
  data: T;
  timestamp: number;
  expiry?: number;
}

@Injectable({
  providedIn: "root",
})
export class OfflineStorageService {
  private readonly DB_NAME = "pwa-offline-db";
  private readonly DB_VERSION = 1;
  private db: IDBDatabase | null = null;

  // Offline state
  readonly isOffline = signal(!navigator.onLine);
  readonly pendingActions = signal<OfflineAction[]>([]);
  readonly syncStatus = signal<"idle" | "syncing" | "error">("idle");

  constructor() {
    this.initializeDB();
    this.setupNetworkMonitoring();
    this.loadPendingActions();
  }

  private async initializeDB(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.DB_NAME, this.DB_VERSION);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };

      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;

        // Create object stores
        if (!db.objectStoreNames.contains("cached-data")) {
          const cacheStore = db.createObjectStore("cached-data", {
            keyPath: "key",
          });
          cacheStore.createIndex("timestamp", "timestamp");
          cacheStore.createIndex("expiry", "expiry");
        }

        if (!db.objectStoreNames.contains("pending-actions")) {
          const actionStore = db.createObjectStore("pending-actions", {
            keyPath: "id",
          });
          actionStore.createIndex("type", "type");
          actionStore.createIndex("timestamp", "timestamp");
        }

        if (!db.objectStoreNames.contains("offline-forms")) {
          const formStore = db.createObjectStore("offline-forms", {
            keyPath: "id",
          });
          formStore.createIndex("timestamp", "timestamp");
        }
      };
    });
  }

  // Data Caching
  async cacheData<T>(
    key: string,
    data: T,
    expiryMinutes?: number
  ): Promise<void> {
    if (!this.db) return;

    const expiry = expiryMinutes
      ? Date.now() + expiryMinutes * 60 * 1000
      : undefined;

    const cacheEntry: OfflineData<T> = {
      data,
      timestamp: Date.now(),
      expiry,
    };

    const transaction = this.db.transaction(["cached-data"], "readwrite");
    const store = transaction.objectStore("cached-data");

    await this.promisifyIDBRequest(store.put({ key, ...cacheEntry }));
  }

  async getCachedData<T>(key: string): Promise<T | null> {
    if (!this.db) return null;

    const transaction = this.db.transaction(["cached-data"], "readonly");
    const store = transaction.objectStore("cached-data");

    const result = await this.promisifyIDBRequest(store.get(key));

    if (!result) return null;

    // Check if expired
    if (result.expiry && Date.now() > result.expiry) {
      await this.deleteCachedData(key);
      return null;
    }

    return result.data;
  }

  async deleteCachedData(key: string): Promise<void> {
    if (!this.db) return;

    const transaction = this.db.transaction(["cached-data"], "readwrite");
    const store = transaction.objectStore("cached-data");

    await this.promisifyIDBRequest(store.delete(key));
  }

  async clearExpiredCache(): Promise<void> {
    if (!this.db) return;

    const transaction = this.db.transaction(["cached-data"], "readwrite");
    const store = transaction.objectStore("cached-data");
    const index = store.index("expiry");

    const now = Date.now();
    const range = IDBKeyRange.upperBound(now);

    const cursor = await this.promisifyIDBRequest(index.openCursor(range));

    const deletePromises: Promise<void>[] = [];

    if (cursor) {
      do {
        deletePromises.push(
          this.promisifyIDBRequest(store.delete(cursor.primaryKey))
        );
      } while (cursor.continue());
    }

    await Promise.all(deletePromises);
  }

  // Offline Actions Queue
  async queueAction(type: string, data: any, maxAttempts = 3): Promise<string> {
    const action: OfflineAction = {
      id: this.generateId(),
      type,
      data,
      timestamp: Date.now(),
      attempts: 0,
      maxAttempts,
    };

    if (this.db) {
      const transaction = this.db.transaction(["pending-actions"], "readwrite");
      const store = transaction.objectStore("pending-actions");

      await this.promisifyIDBRequest(store.add(action));
    }

    this.updatePendingActions();
    return action.id;
  }

  async removeAction(actionId: string): Promise<void> {
    if (!this.db) return;

    const transaction = this.db.transaction(["pending-actions"], "readwrite");
    const store = transaction.objectStore("pending-actions");

    await this.promisifyIDBRequest(store.delete(actionId));
    this.updatePendingActions();
  }

  async syncPendingActions(): Promise<void> {
    if (this.isOffline() || this.syncStatus() === "syncing") return;

    this.syncStatus.set("syncing");

    try {
      const actions = await this.getAllPendingActions();

      for (const action of actions) {
        try {
          await this.executeAction(action);
          await this.removeAction(action.id);
        } catch (error) {
          action.attempts++;

          if (action.attempts >= action.maxAttempts) {
            await this.removeAction(action.id);
            console.error(
              `Max attempts reached for action ${action.id}:`,
              error
            );
          } else {
            await this.updateAction(action);
          }
        }
      }

      this.syncStatus.set("idle");
    } catch (error) {
      this.syncStatus.set("error");
      console.error("Sync failed:", error);
    }
  }

  private async getAllPendingActions(): Promise<OfflineAction[]> {
    if (!this.db) return [];

    const transaction = this.db.transaction(["pending-actions"], "readonly");
    const store = transaction.objectStore("pending-actions");

    return await this.promisifyIDBRequest(store.getAll());
  }

  private async updateAction(action: OfflineAction): Promise<void> {
    if (!this.db) return;

    const transaction = this.db.transaction(["pending-actions"], "readwrite");
    const store = transaction.objectStore("pending-actions");

    await this.promisifyIDBRequest(store.put(action));
  }

  private async executeAction(action: OfflineAction): Promise<void> {
    // Implement action execution based on type
    switch (action.type) {
      case "create-user":
        return await this.executeCreateUser(action.data);
      case "update-profile":
        return await this.executeUpdateProfile(action.data);
      case "delete-item":
        return await this.executeDeleteItem(action.data);
      default:
        throw new Error(`Unknown action type: ${action.type}`);
    }
  }

  // Form Data Storage
  async saveFormData(formId: string, formData: any): Promise<void> {
    if (!this.db) return;

    const formEntry = {
      id: formId,
      data: formData,
      timestamp: Date.now(),
    };

    const transaction = this.db.transaction(["offline-forms"], "readwrite");
    const store = transaction.objectStore("offline-forms");

    await this.promisifyIDBRequest(store.put(formEntry));
  }

  async getFormData(formId: string): Promise<any | null> {
    if (!this.db) return null;

    const transaction = this.db.transaction(["offline-forms"], "readonly");
    const store = transaction.objectStore("offline-forms");

    const result = await this.promisifyIDBRequest(store.get(formId));
    return result?.data || null;
  }

  async deleteFormData(formId: string): Promise<void> {
    if (!this.db) return;

    const transaction = this.db.transaction(["offline-forms"], "readwrite");
    const store = transaction.objectStore("offline-forms");

    await this.promisifyIDBRequest(store.delete(formId));
  }

  // Network Monitoring
  private setupNetworkMonitoring(): void {
    window.addEventListener("online", () => {
      this.isOffline.set(false);
      this.syncPendingActions();
    });

    window.addEventListener("offline", () => {
      this.isOffline.set(true);
    });
  }

  private async loadPendingActions(): Promise<void> {
    this.updatePendingActions();
  }

  private async updatePendingActions(): Promise<void> {
    const actions = await this.getAllPendingActions();
    this.pendingActions.set(actions);
  }

  // Utility Methods
  private promisifyIDBRequest<T>(request: IDBRequest<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  // Action Executors (implement based on your API)
  private async executeCreateUser(userData: any): Promise<void> {
    // Implementation depends on your API structure
    const response = await fetch("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
  }

  private async executeUpdateProfile(profileData: any): Promise<void> {
    const response = await fetch(`/api/users/${profileData.id}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(profileData),
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
  }

  private async executeDeleteItem(itemData: any): Promise<void> {
    const response = await fetch(`/api/items/${itemData.id}`, {
      method: "DELETE",
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
  }

  // Public API
  getStorageStats(): Promise<{ usage: number; quota: number }> {
    if ("storage" in navigator && "estimate" in navigator.storage) {
      return navigator.storage.estimate().then((estimate) => ({
        usage: estimate.usage || 0,
        quota: estimate.quota || 0,
      }));
    }

    return Promise.resolve({ usage: 0, quota: 0 });
  }

  async clearAllData(): Promise<void> {
    if (!this.db) return;

    const storeNames = ["cached-data", "pending-actions", "offline-forms"];

    for (const storeName of storeNames) {
      const transaction = this.db.transaction([storeName], "readwrite");
      const store = transaction.objectStore(storeName);
      await this.promisifyIDBRequest(store.clear());
    }

    this.pendingActions.set([]);
  }
}
```

## 🎯 **Complete PWA Implementation**

I've created the first part of the comprehensive PWA guide covering:

- **🚀 PWA Setup** - Angular CLI integration and configuration
- **📱 App Manifest** - Complete manifest with shortcuts and protocols
- **🔄 Service Worker** - Advanced PWA service with signals and updates
- **🎛️ PWA Controls** - UI component for installation and notifications
- **🗄️ Offline Storage** - IndexedDB integration with action queuing

**Coming in Part 2:**

- Background sync implementation
- Push notifications server integration
- App shell architecture
- Performance optimization strategies
- PWA testing and deployment
