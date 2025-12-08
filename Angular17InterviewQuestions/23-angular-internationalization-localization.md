# 🌐 Angular Internationalization (i18n) & Localization: Complete Guide

## 🎯 **Question Overview**

_"How do you implement comprehensive internationalization and localization in Angular applications with dynamic language switching, RTL support, and locale-specific formatting?"_

## 🔍 **Understanding Angular i18n**

Angular's **internationalization (i18n)** provides **robust support** for **multiple languages**, **locales**, **cultural formatting**, and **accessibility**. Modern Angular offers **both build-time** and **runtime** approaches with **advanced features** like **lazy loading translations**, **ICU expressions**, and **dynamic locale switching**! 🌍

## 🏗️ **Build-time i18n Implementation**

### **1. 📦 Setup & Configuration**

```typescript
// angular.json - Build Configuration for Multiple Locales
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "my-app": {
      "projectType": "application",
      "i18n": {
        "sourceLocale": "en-US",
        "locales": {
          "es": {
            "translation": "src/locale/messages.es.xlf",
            "baseHref": "/es/"
          },
          "fr": {
            "translation": "src/locale/messages.fr.xlf",
            "baseHref": "/fr/"
          },
          "de": {
            "translation": "src/locale/messages.de.xlf",
            "baseHref": "/de/"
          },
          "ar": {
            "translation": "src/locale/messages.ar.xlf",
            "baseHref": "/ar/"
          },
          "zh": {
            "translation": "src/locale/messages.zh.xlf",
            "baseHref": "/zh/"
          },
          "ja": {
            "translation": "src/locale/messages.ja.xlf",
            "baseHref": "/ja/"
          }
        }
      },
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "localize": true,
            "aot": true,
            "outputPath": "dist/my-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",
            "assets": [
              "src/favicon.ico",
              "src/assets",
              {
                "glob": "**/*",
                "input": "src/locale",
                "output": "/locale/"
              }
            ]
          },
          "configurations": {
            "production": {
              "localize": true,
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                }
              ]
            },
            "en": {
              "aot": true,
              "outputPath": "dist/my-app/en",
              "i18nFile": "src/locale/messages.en.xlf",
              "i18nFormat": "xlf",
              "i18nLocale": "en"
            },
            "es": {
              "aot": true,
              "outputPath": "dist/my-app/es",
              "i18nFile": "src/locale/messages.es.xlf",
              "i18nFormat": "xlf",
              "i18nLocale": "es",
              "baseHref": "/es/"
            },
            "fr": {
              "aot": true,
              "outputPath": "dist/my-app/fr",
              "i18nFile": "src/locale/messages.fr.xlf",
              "i18nFormat": "xlf",
              "i18nLocale": "fr",
              "baseHref": "/fr/"
            }
          }
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "options": {},
          "configurations": {
            "production": {
              "browserTarget": "my-app:build:production"
            },
            "en": {
              "browserTarget": "my-app:build:en"
            },
            "es": {
              "browserTarget": "my-app:build:es"
            },
            "fr": {
              "browserTarget": "my-app:build:fr"
            }
          }
        }
      }
    }
  }
}
```

```typescript
// src/app/app.module.ts - i18n Module Configuration
import { NgModule, LOCALE_ID, DEFAULT_CURRENCY_CODE } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { registerLocaleData } from "@angular/common";

// Import locale data for different regions
import localeEs from "@angular/common/locales/es";
import localeFr from "@angular/common/locales/fr";
import localeDe from "@angular/common/locales/de";
import localeAr from "@angular/common/locales/ar";
import localeZh from "@angular/common/locales/zh";
import localeJa from "@angular/common/locales/ja";

import { AppComponent } from "./app.component";
import { I18nService } from "./services/i18n.service";

// Register locale data
registerLocaleData(localeEs, "es");
registerLocaleData(localeFr, "fr");
registerLocaleData(localeDe, "de");
registerLocaleData(localeAr, "ar");
registerLocaleData(localeZh, "zh");
registerLocaleData(localeJa, "ja");

// Determine locale from browser or URL
function getLocale(): string {
  const supportedLocales = ["en-US", "es", "fr", "de", "ar", "zh", "ja"];

  // Check URL first
  const urlLocale = window.location.pathname.split("/")[1];
  if (supportedLocales.includes(urlLocale)) {
    return urlLocale;
  }

  // Check localStorage
  const savedLocale = localStorage.getItem("preferred-locale");
  if (savedLocale && supportedLocales.includes(savedLocale)) {
    return savedLocale;
  }

  // Check browser locale
  const browserLocale = navigator.language;
  const shortLocale = browserLocale.split("-")[0];

  if (supportedLocales.includes(browserLocale)) {
    return browserLocale;
  }

  if (supportedLocales.includes(shortLocale)) {
    return shortLocale;
  }

  return "en-US"; // Default fallback
}

// Get currency based on locale
function getCurrency(locale: string): string {
  const currencyMap: { [key: string]: string } = {
    "en-US": "USD",
    es: "EUR",
    fr: "EUR",
    de: "EUR",
    ar: "SAR",
    zh: "CNY",
    ja: "JPY",
  };

  return currencyMap[locale] || "USD";
}

const currentLocale = getLocale();

@NgModule({
  declarations: [
    AppComponent,
    // ... other components
  ],
  imports: [
    BrowserModule,
    // ... other modules
  ],
  providers: [
    {
      provide: LOCALE_ID,
      useValue: currentLocale,
    },
    {
      provide: DEFAULT_CURRENCY_CODE,
      useValue: getCurrency(currentLocale),
    },
    I18nService,
    // ... other providers
  ],
  bootstrap: [AppComponent],
})
export class AppModule {
  constructor() {
    console.log(`Application started with locale: ${currentLocale}`);
  }
}
```

### **2. 🎯 Advanced Translation Markup**

```typescript
// src/app/components/product-details/product-details.component.ts
import { Component, Input, OnInit, inject, LOCALE_ID } from '@angular/core';
import { formatCurrency, formatDate, formatNumber } from '@angular/common';

@Component({
  selector: 'app-product-details',
  template: `
    <div class="product-details">
      <!-- Basic Translation with Custom ID -->
      <h1 i18n="@@product.title">Product Details</h1>

      <!-- Translation with Description -->
      <h2 i18n="Product name header|Header showing the product name">
        {{ product.name }}
      </h2>

      <!-- ICU Expressions for Pluralization -->
      <p i18n="@@product.reviews.count">
        {reviewCount, plural,
          =0 {No reviews yet}
          =1 {One review}
          other {{{reviewCount}} reviews}
        }
      </p>

      <!-- ICU with Select for Gender/Category -->
      <p i18n="@@product.availability">
        {product.category, select,
          electronics {This electronic device is}
          clothing {This clothing item is}
          books {This book is}
          other {This product is}
        }
        {product.stock, plural,
          =0 {out of stock}
          =1 {the last one in stock}
          other {available with {{product.stock}} items in stock}
        }
      </p>

      <!-- Complex ICU with Nested Expressions -->
      <div i18n="@@shipping.info" class="shipping-info">
        {product.weight, plural,
          =0 {Digital product - no shipping required}
          other {
            Shipping cost: {shippingZone, select,
              domestic {Free shipping within the country}
              international {
                {product.weight, plural,
                  =1 {$5 for lightweight items}
                  other {${{calculateShipping(product.weight)}} based on weight}
                }
              }
              other {Contact us for shipping quote}
            }
          }
        }
      </div>

      <!-- Translation with Interpolated Values -->
      <div class="price-section">
        <span i18n="@@product.price.label">Price:</span>
        <strong class="price">
          {{ formatPrice(product.price) }}
        </strong>

        <!-- Discount Information with ICU -->
        <div *ngIf="product.discount > 0"
             i18n="@@product.discount.info"
             class="discount-info">
          Save {product.discount, plural,
            =1 {one dollar}
            other {{{formatPrice(product.discount)}}
          }} ({calculateDiscountPercentage()}% off)
        </div>
      </div>

      <!-- Date Formatting with Locale -->
      <div class="product-metadata">
        <p i18n="@@product.added.date">
          Added on: {{ formatLocalDate(product.createdDate) }}
        </p>
        <p i18n="@@product.updated.date">
          Last updated: {{ formatRelativeDate(product.updatedDate) }}
        </p>
      </div>

      <!-- Rating with Locale-specific Number Formatting -->
      <div class="rating-section">
        <span i18n="@@product.rating.label">Rating:</span>
        <div class="stars">
          {{ formatNumber(product.rating, '1.1-1') }} / 5
        </div>
        <span i18n="@@product.rating.based.on">
          Based on {{ formatNumber(product.reviewCount) }}
          {product.reviewCount, plural, =1 {review} other {reviews}}
        </span>
      </div>

      <!-- Conditional Content Based on Locale -->
      <div class="locale-specific-content">
        <ng-container [ngSwitch]="currentLocale">
          <!-- US-specific content -->
          <div *ngSwitchCase="'en-US'">
            <p i18n="@@us.specific.warranty">
              2-year manufacturer warranty included
            </p>
            <p i18n="@@us.specific.returns">
              30-day return policy - no questions asked
            </p>
          </div>

          <!-- European content -->
          <div *ngSwitchCase="'es'">
            <p i18n="@@eu.specific.warranty">
              Garantía del fabricante de 2 años incluida
            </p>
            <p i18n="@@eu.specific.returns">
              Política de devolución de 14 días según la ley europea
            </p>
          </div>

          <!-- Default content -->
          <div *ngSwitchDefault>
            <p i18n="@@default.warranty">
              Warranty terms vary by region
            </p>
          </div>
        </ng-container>
      </div>

      <!-- Action Buttons with Context -->
      <div class="action-buttons">
        <button
          type="button"
          [disabled]="product.stock === 0"
          i18n="@@product.add.to.cart.button|Button to add product to shopping cart"
          class="add-to-cart-btn">
          {product.stock, plural,
            =0 {Out of Stock}
            other {Add to Cart}
          }
        </button>

        <button
          type="button"
          i18n="@@product.add.to.wishlist.button|Button to add product to wishlist"
          class="wishlist-btn">
          Add to Wishlist
        </button>

        <button
          type="button"
          i18n="@@product.share.button|Button to share product with others"
          class="share-btn">
          Share
        </button>
      </div>

      <!-- Form with Validation Messages -->
      <form class="review-form" *ngIf="showReviewForm">
        <h3 i18n="@@review.form.title">Leave a Review</h3>

        <div class="form-group">
          <label i18n="@@review.rating.label">Rating (1-5 stars):</label>
          <select [(ngModel)]="newReview.rating" name="rating" required>
            <option value="" i18n="@@review.rating.placeholder">Select rating</option>
            <option value="1" i18n="@@review.rating.1star">1 star - Poor</option>
            <option value="2" i18n="@@review.rating.2star">2 stars - Fair</option>
            <option value="3" i18n="@@review.rating.3star">3 stars - Good</option>
            <option value="4" i18n="@@review.rating.4star">4 stars - Very Good</option>
            <option value="5" i18n="@@review.rating.5star">5 stars - Excellent</option>
          </select>
        </div>

        <div class="form-group">
          <label i18n="@@review.comment.label">Your Review:</label>
          <textarea
            [(ngModel)]="newReview.comment"
            name="comment"
            required
            minlength="10"
            maxlength="500"
            i18n-placeholder="@@review.comment.placeholder"
            placeholder="Share your experience with this product...">
          </textarea>

          <div class="character-count">
            <span i18n="@@review.character.count">
              {{ newReview.comment?.length || 0 }} / 500 characters
            </span>
          </div>
        </div>

        <button
          type="submit"
          [disabled]="!isReviewValid()"
          i18n="@@review.submit.button"
          class="submit-review-btn">
          Submit Review
        </button>
      </form>

      <!-- Error Messages with i18n -->
      <div *ngIf="errorMessage" class="error-message">
        <ng-container [ngSwitch]="errorType">
          <p *ngSwitchCase="'network'" i18n="@@error.network">
            Unable to connect to the server. Please check your internet connection.
          </p>
          <p *ngSwitchCase="'not-found'" i18n="@@error.product.not.found">
            This product is no longer available.
          </p>
          <p *ngSwitchCase="'permission'" i18n="@@error.permission.denied">
            You don't have permission to view this product.
          </p>
          <p *ngSwitchDefault i18n="@@error.generic">
            An unexpected error occurred. Please try again later.
          </p>
        </ng-container>
      </div>
    </div>
  `,
  styleUrls: ['./product-details.component.scss']
})
export class ProductDetailsComponent implements OnInit {
  @Input() product: any;

  currentLocale = inject(LOCALE_ID);

  reviewCount = 0;
  shippingZone = 'domestic';
  showReviewForm = false;
  newReview = { rating: '', comment: '' };
  errorMessage = '';
  errorType = '';

  ngOnInit() {
    this.reviewCount = this.product?.reviewCount || 0;
  }

  formatPrice(price: number): string {
    return formatCurrency(price, this.currentLocale, '$');
  }

  formatNumber(value: number, format?: string): string {
    return formatNumber(value, this.currentLocale, format);
  }

  formatLocalDate(date: Date): string {
    return formatDate(date, 'medium', this.currentLocale);
  }

  formatRelativeDate(date: Date): string {
    const now = new Date();
    const diff = now.getTime() - date.getTime();
    const days = Math.floor(diff / (1000 * 60 * 60 * 24));

    // This would ideally use Intl.RelativeTimeFormat
    // but for demonstration, we'll use simple logic
    if (days === 0) return 'Today';
    if (days === 1) return 'Yesterday';
    if (days < 7) return `${days} days ago`;

    return this.formatLocalDate(date);
  }

  calculateDiscountPercentage(): number {
    return Math.round((this.product.discount / this.product.price) * 100);
  }

  calculateShipping(weight: number): number {
    return Math.ceil(weight * 2.5); // Simple calculation
  }

  isReviewValid(): boolean {
    return this.newReview.rating !== '' &&
           this.newReview.comment?.length >= 10;
  }
}
```

### **3. 🌍 RTL Support & Directionality**

```typescript
// src/app/services/directionality.service.ts - RTL/LTR Service
import { Injectable, inject, LOCALE_ID } from "@angular/core";
import { BehaviorSubject, Observable } from "rxjs";

export type TextDirection = "ltr" | "rtl";

export interface DirectionalityConfig {
  locale: string;
  direction: TextDirection;
  language: string;
  region?: string;
  isRTL: boolean;
  textAlign: "left" | "right";
  marginStart: "margin-left" | "margin-right";
  marginEnd: "margin-right" | "margin-left";
  borderStart: "border-left" | "border-right";
  borderEnd: "border-right" | "border-left";
}

@Injectable({
  providedIn: "root",
})
export class DirectionalityService {
  private locale = inject(LOCALE_ID);

  // RTL languages
  private readonly RTL_LOCALES = [
    "ar", // Arabic
    "he", // Hebrew
    "fa", // Persian/Farsi
    "ur", // Urdu
    "ps", // Pashto
    "sd", // Sindhi
    "ug", // Uighur
    "yi", // Yiddish
  ];

  private readonly _config = new BehaviorSubject<DirectionalityConfig>(
    this.createDirectionalityConfig(this.locale)
  );

  readonly config$ = this._config.asObservable();
  readonly isRTL$ = new BehaviorSubject(this.isLocaleRTL(this.locale));

  get currentConfig(): DirectionalityConfig {
    return this._config.value;
  }

  get isRTL(): boolean {
    return this.isRTL$.value;
  }

  get direction(): TextDirection {
    return this.isRTL ? "rtl" : "ltr";
  }

  setLocale(locale: string): void {
    const config = this.createDirectionalityConfig(locale);
    this._config.next(config);
    this.isRTL$.next(config.isRTL);

    // Update document direction
    document.documentElement.dir = config.direction;
    document.documentElement.lang = config.language;

    // Update CSS custom properties for dynamic styling
    document.documentElement.style.setProperty(
      "--text-direction",
      config.direction
    );
    document.documentElement.style.setProperty(
      "--text-align-start",
      config.textAlign
    );
    document.documentElement.style.setProperty(
      "--text-align-end",
      config.isRTL ? "left" : "right"
    );
  }

  private createDirectionalityConfig(locale: string): DirectionalityConfig {
    const isRTL = this.isLocaleRTL(locale);
    const language = locale.split("-")[0];
    const region = locale.split("-")[1];

    return {
      locale,
      direction: isRTL ? "rtl" : "ltr",
      language,
      region,
      isRTL,
      textAlign: isRTL ? "right" : "left",
      marginStart: isRTL ? "margin-right" : "margin-left",
      marginEnd: isRTL ? "margin-left" : "margin-right",
      borderStart: isRTL ? "border-right" : "border-left",
      borderEnd: isRTL ? "border-left" : "border-right",
    };
  }

  private isLocaleRTL(locale: string): boolean {
    const language = locale.split("-")[0].toLowerCase();
    return this.RTL_LOCALES.includes(language);
  }

  // Utility methods for dynamic styling
  getMarginStart(value: string): { [key: string]: string } {
    const config = this.currentConfig;
    return { [config.marginStart]: value };
  }

  getMarginEnd(value: string): { [key: string]: string } {
    const config = this.currentConfig;
    return { [config.marginEnd]: value };
  }

  getBorderStart(value: string): { [key: string]: string } {
    const config = this.currentConfig;
    return { [config.borderStart]: value };
  }

  getBorderEnd(value: string): { [key: string]: string } {
    const config = this.currentConfig;
    return { [config.borderEnd]: value };
  }

  getTextAlign(alignment: "start" | "end" = "start"): string {
    const config = this.currentConfig;
    if (alignment === "start") {
      return config.textAlign;
    } else {
      return config.isRTL ? "left" : "right";
    }
  }
}
```

```scss
/* src/styles/directionality.scss - RTL/LTR Styles */

// Mixins for directional styling
@mixin margin-start($value) {
  [dir="ltr"] & {
    margin-left: $value;
  }

  [dir="rtl"] & {
    margin-right: $value;
  }
}

@mixin margin-end($value) {
  [dir="ltr"] & {
    margin-right: $value;
  }

  [dir="rtl"] & {
    margin-left: $value;
  }
}

@mixin padding-start($value) {
  [dir="ltr"] & {
    padding-left: $value;
  }

  [dir="rtl"] & {
    padding-right: $value;
  }
}

@mixin padding-end($value) {
  [dir="ltr"] & {
    padding-right: $value;
  }

  [dir="rtl"] & {
    padding-left: $value;
  }
}

@mixin border-start($value) {
  [dir="ltr"] & {
    border-left: $value;
  }

  [dir="rtl"] & {
    border-right: $value;
  }
}

@mixin border-end($value) {
  [dir="ltr"] & {
    border-right: $value;
  }

  [dir="rtl"] & {
    border-left: $value;
  }
}

@mixin text-align-start() {
  [dir="ltr"] & {
    text-align: left;
  }

  [dir="rtl"] & {
    text-align: right;
  }
}

@mixin text-align-end() {
  [dir="ltr"] & {
    text-align: right;
  }

  [dir="rtl"] & {
    text-align: left;
  }
}

// Base directional styles
:root {
  --text-direction: ltr;
  --text-align-start: left;
  --text-align-end: right;
}

[dir="rtl"] {
  --text-direction: rtl;
  --text-align-start: right;
  --text-align-end: left;
}

// Global RTL adjustments
body {
  direction: var(--text-direction);
  text-align: var(--text-align-start);
}

// Navigation adjustments
.navigation {
  ul {
    li {
      @include margin-end(1rem);

      &:last-child {
        margin: 0;
      }
    }
  }

  .menu-icon {
    [dir="ltr"] & {
      float: right;
    }

    [dir="rtl"] & {
      float: left;
    }
  }
}

// Form adjustments
.form-group {
  label {
    @include text-align-start();
    display: block;
    @include margin-end(0.5rem);
  }

  input,
  select,
  textarea {
    @include text-align-start();
  }

  .form-help {
    @include text-align-start();
    font-size: 0.875rem;
    color: #666;
    @include margin-start(0.25rem);
  }
}

// Card layouts
.card {
  .card-image {
    [dir="ltr"] & {
      float: left;
    }

    [dir="rtl"] & {
      float: right;
    }
  }

  .card-content {
    @include padding-start(1rem);
  }

  .card-actions {
    @include text-align-end();

    .btn {
      @include margin-start(0.5rem);

      &:first-child {
        margin: 0;
      }
    }
  }
}

// Table adjustments
table {
  th,
  td {
    &:first-child {
      @include text-align-start();
    }

    &.numeric {
      @include text-align-end();
    }
  }

  .sort-indicator {
    [dir="ltr"] & {
      margin-left: 0.25rem;
    }

    [dir="rtl"] & {
      margin-right: 0.25rem;
    }
  }
}

// Modal and dialog adjustments
.modal {
  .modal-header {
    .close-btn {
      [dir="ltr"] & {
        position: absolute;
        top: 1rem;
        right: 1rem;
      }

      [dir="rtl"] & {
        position: absolute;
        top: 1rem;
        left: 1rem;
      }
    }
  }
}

// Tooltip adjustments
.tooltip {
  &.tooltip-start {
    [dir="ltr"] & {
      left: 0;
      transform: translateX(-100%);
    }

    [dir="rtl"] & {
      right: 0;
      transform: translateX(100%);
    }
  }

  &.tooltip-end {
    [dir="ltr"] & {
      right: 0;
      transform: translateX(100%);
    }

    [dir="rtl"] & {
      left: 0;
      transform: translateX(-100%);
    }
  }
}

// Animation adjustments for RTL
@keyframes slideInStart {
  from {
    [dir="ltr"] & {
      transform: translateX(-100%);
    }

    [dir="rtl"] & {
      transform: translateX(100%);
    }
  }

  to {
    transform: translateX(0);
  }
}

@keyframes slideInEnd {
  from {
    [dir="ltr"] & {
      transform: translateX(100%);
    }

    [dir="rtl"] & {
      transform: translateX(-100%);
    }
  }

  to {
    transform: translateX(0);
  }
}

// Sidebar adjustments
.sidebar {
  [dir="ltr"] & {
    border-right: 1px solid #ddd;
  }

  [dir="rtl"] & {
    border-left: 1px solid #ddd;
  }

  .sidebar-item {
    @include padding-start(1rem);
    @include border-start(3px solid transparent);

    &.active {
      @include border-start(3px solid #007bff);
    }

    .icon {
      @include margin-end(0.5rem);
    }
  }
}

// Breadcrumb adjustments
.breadcrumb {
  .breadcrumb-item {
    &:not(:last-child) {
      &::after {
        content: "/";
        @include margin-start(0.5rem);
        @include margin-end(0.5rem);

        [dir="rtl"] & {
          content: "\\";
        }
      }
    }
  }
}

// Progress indicators
.progress {
  [dir="rtl"] & {
    transform: scaleX(-1);
  }

  .progress-bar {
    [dir="rtl"] & {
      transform: scaleX(-1);
    }
  }
}
```

### **4. 📱 Responsive i18n Component**

```typescript
// src/app/components/language-switcher/language-switcher.component.ts
import { Component, OnInit, inject } from "@angular/core";
import { Router } from "@angular/router";
import { DOCUMENT } from "@angular/common";

import { DirectionalityService } from "../../services/directionality.service";
import { I18nService } from "../../services/i18n.service";

export interface Language {
  code: string;
  name: string;
  nativeName: string;
  flag: string;
  isRTL: boolean;
  region?: string;
}

@Component({
  selector: "app-language-switcher",
  template: `
    <div class="language-switcher" [class.rtl]="directionalityService.isRTL">
      <!-- Desktop Dropdown -->
      <div class="desktop-switcher" [class.hidden]="isMobile">
        <button
          class="current-language-btn"
          (click)="toggleDropdown()"
          [attr.aria-expanded]="isDropdownOpen"
          [attr.aria-haspopup]="true"
          i18n-aria-label="@@language.switcher.aria.label"
          aria-label="Change language"
        >
          <img
            [src]="currentLanguage.flag"
            [alt]="currentLanguage.name"
            class="flag-icon"
          />

          <span class="language-name">
            {{ currentLanguage.nativeName }}
          </span>

          <svg
            class="dropdown-arrow"
            [class.rotated]="isDropdownOpen"
            width="12"
            height="12"
            viewBox="0 0 12 12"
          >
            <path d="M2 4l4 4 4-4" stroke="currentColor" fill="none" />
          </svg>
        </button>

        <ul
          class="language-dropdown"
          [class.open]="isDropdownOpen"
          role="menu"
          [attr.aria-hidden]="!isDropdownOpen"
        >
          <li
            *ngFor="
              let language of availableLanguages;
              trackBy: trackByLanguage
            "
            role="none"
          >
            <button
              class="language-option"
              [class.current]="language.code === currentLanguage.code"
              [class.rtl-lang]="language.isRTL"
              (click)="switchLanguage(language)"
              role="menuitem"
              [attr.aria-current]="
                language.code === currentLanguage.code ? 'true' : null
              "
            >
              <img
                [src]="language.flag"
                [alt]="language.name"
                class="flag-icon"
              />

              <div class="language-info">
                <div class="language-native">{{ language.nativeName }}</div>
                <div class="language-english">{{ language.name }}</div>
              </div>

              <svg
                *ngIf="language.code === currentLanguage.code"
                class="check-icon"
                width="16"
                height="16"
                viewBox="0 0 16 16"
              >
                <path
                  d="M13 4L6 11 3 8"
                  stroke="currentColor"
                  fill="none"
                  stroke-width="2"
                />
              </svg>
            </button>
          </li>
        </ul>
      </div>

      <!-- Mobile Modal -->
      <div class="mobile-switcher" [class.hidden]="!isMobile">
        <button
          class="mobile-language-btn"
          (click)="openMobileModal()"
          i18n-aria-label="@@language.switcher.mobile.aria.label"
          aria-label="Change language"
        >
          <img
            [src]="currentLanguage.flag"
            [alt]="currentLanguage.name"
            class="flag-icon"
          />

          <span class="sr-only">{{ currentLanguage.nativeName }}</span>
        </button>
      </div>

      <!-- Mobile Modal Overlay -->
      <div
        class="modal-overlay"
        [class.open]="isMobileModalOpen"
        (click)="closeMobileModal()"
      >
        <div
          class="mobile-modal"
          [class.open]="isMobileModalOpen"
          (click)="$event.stopPropagation()"
        >
          <div class="modal-header">
            <h2 i18n="@@language.switcher.modal.title">Choose Language</h2>

            <button
              class="close-btn"
              (click)="closeMobileModal()"
              i18n-aria-label="@@modal.close.aria.label"
              aria-label="Close language selection"
            >
              <svg width="24" height="24" viewBox="0 0 24 24">
                <path
                  d="M18 6L6 18M6 6l12 12"
                  stroke="currentColor"
                  stroke-width="2"
                />
              </svg>
            </button>
          </div>

          <div class="modal-content">
            <div class="current-language-display">
              <span i18n="@@language.current.label">Current:</span>
              <div class="current-lang">
                <img
                  [src]="currentLanguage.flag"
                  [alt]="currentLanguage.name"
                  class="flag-icon"
                />
                <span>{{ currentLanguage.nativeName }}</span>
              </div>
            </div>

            <div class="language-grid">
              <button
                *ngFor="
                  let language of availableLanguages;
                  trackBy: trackByLanguage
                "
                class="language-card"
                [class.current]="language.code === currentLanguage.code"
                [class.rtl-lang]="language.isRTL"
                (click)="switchLanguage(language)"
              >
                <img
                  [src]="language.flag"
                  [alt]="language.name"
                  class="flag-icon"
                />

                <div class="language-info">
                  <div class="language-native">{{ language.nativeName }}</div>
                  <div class="language-english">{{ language.name }}</div>
                  <div *ngIf="language.region" class="language-region">
                    {{ language.region }}
                  </div>
                </div>

                <div
                  class="selection-indicator"
                  [class.selected]="language.code === currentLanguage.code"
                >
                  <svg width="20" height="20" viewBox="0 0 20 20">
                    <circle
                      cx="10"
                      cy="10"
                      r="9"
                      stroke="currentColor"
                      fill="none"
                    />
                    <circle
                      *ngIf="language.code === currentLanguage.code"
                      cx="10"
                      cy="10"
                      r="5"
                      fill="currentColor"
                    />
                  </svg>
                </div>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  `,
  styleUrls: ["./language-switcher.component.scss"],
})
export class LanguageSwitcherComponent implements OnInit {
  private router = inject(Router);
  private document = inject(DOCUMENT);

  directionalityService = inject(DirectionalityService);
  i18nService = inject(I18nService);

  isDropdownOpen = false;
  isMobileModalOpen = false;
  isMobile = false;

  availableLanguages: Language[] = [
    {
      code: "en-US",
      name: "English",
      nativeName: "English",
      flag: "/assets/flags/us.svg",
      isRTL: false,
      region: "United States",
    },
    {
      code: "es",
      name: "Spanish",
      nativeName: "Español",
      flag: "/assets/flags/es.svg",
      isRTL: false,
      region: "Spain",
    },
    {
      code: "fr",
      name: "French",
      nativeName: "Français",
      flag: "/assets/flags/fr.svg",
      isRTL: false,
      region: "France",
    },
    {
      code: "de",
      name: "German",
      nativeName: "Deutsch",
      flag: "/assets/flags/de.svg",
      isRTL: false,
      region: "Germany",
    },
    {
      code: "ar",
      name: "Arabic",
      nativeName: "العربية",
      flag: "/assets/flags/sa.svg",
      isRTL: true,
      region: "Saudi Arabia",
    },
    {
      code: "zh",
      name: "Chinese",
      nativeName: "中文",
      flag: "/assets/flags/cn.svg",
      isRTL: false,
      region: "China",
    },
    {
      code: "ja",
      name: "Japanese",
      nativeName: "日本語",
      flag: "/assets/flags/jp.svg",
      isRTL: false,
      region: "Japan",
    },
  ];

  currentLanguage: Language;

  constructor() {
    // Detect mobile
    this.checkIfMobile();
    this.document.defaultView?.addEventListener("resize", () => {
      this.checkIfMobile();
    });
  }

  ngOnInit() {
    // Set initial current language based on URL or saved preference
    const currentLocale = this.getCurrentLocale();
    this.currentLanguage =
      this.availableLanguages.find((lang) => lang.code === currentLocale) ||
      this.availableLanguages[0];

    // Close dropdown when clicking outside
    this.document.addEventListener("click", (event) => {
      if (
        !event.target ||
        !(event.target as Element).closest(".language-switcher")
      ) {
        this.isDropdownOpen = false;
      }
    });
  }

  switchLanguage(language: Language): void {
    if (language.code === this.currentLanguage.code) {
      this.closeDropdowns();
      return;
    }

    this.currentLanguage = language;

    // Update directionality
    this.directionalityService.setLocale(language.code);

    // Save preference
    localStorage.setItem("preferred-locale", language.code);

    // Navigate to new locale URL
    this.navigateToLocale(language.code);

    // Close any open dropdowns/modals
    this.closeDropdowns();
  }

  toggleDropdown(): void {
    this.isDropdownOpen = !this.isDropdownOpen;
  }

  openMobileModal(): void {
    this.isMobileModalOpen = true;
    // Prevent body scroll
    this.document.body.style.overflow = "hidden";
  }

  closeMobileModal(): void {
    this.isMobileModalOpen = false;
    // Restore body scroll
    this.document.body.style.overflow = "";
  }

  private closeDropdowns(): void {
    this.isDropdownOpen = false;
    this.closeMobileModal();
  }

  private getCurrentLocale(): string {
    // Get locale from URL path
    const pathSegments = this.document.location.pathname.split("/");
    const possibleLocale = pathSegments[1];

    if (this.availableLanguages.some((lang) => lang.code === possibleLocale)) {
      return possibleLocale;
    }

    // Fallback to default
    return "en-US";
  }

  private navigateToLocale(locale: string): void {
    const currentPath = this.document.location.pathname;
    const pathSegments = currentPath.split("/");

    // Remove current locale from path if present
    if (this.availableLanguages.some((lang) => lang.code === pathSegments[1])) {
      pathSegments.splice(1, 1);
    }

    // Add new locale
    const newPath =
      locale === "en-US"
        ? pathSegments.join("/")
        : `/${locale}${pathSegments.join("/")}`;

    // Navigate to new URL
    this.document.location.href = newPath;
  }

  private checkIfMobile(): void {
    this.isMobile = this.document.defaultView!.innerWidth < 768;
  }

  trackByLanguage(index: number, language: Language): string {
    return language.code;
  }
}
```

## 🎯 **Part 1 Summary**

This comprehensive guide covers:

- **🏗️ Build-time i18n** - Complete Angular.json configuration and setup
- **🎯 Advanced Translation Markup** - ICU expressions, pluralization, and complex formatting
- **🌍 RTL Support** - Complete directionality service and responsive styling
- **📱 Language Switcher** - Responsive component with mobile support

**Coming in Part 2:**

- Runtime i18n implementation
- Custom translation pipes
- Dynamic content translation
- Performance optimization and lazy loading
