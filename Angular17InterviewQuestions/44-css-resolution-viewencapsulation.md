# 🎨 **Angular CSS Resolution & ViewEncapsulation: Complete Guide**

## 🎯 **What You'll Learn**

Master Angular's CSS encapsulation strategies, style isolation mechanisms, and advanced styling patterns with practical examples for enterprise applications.

---

## 📚 **The Basics: CSS Encapsulation Fundamentals**

### **🤔 What is ViewEncapsulation?**

**ViewEncapsulation** determines how Angular applies styles to components. Think of it as different **"bubble" strategies** for your styles:

- 🔒 **Emulated** (Default) - Creates a fake shadow DOM with unique attributes
- 🌐 **None** - No encapsulation, styles apply globally
- 👥 **ShadowDOM** - True browser shadow DOM encapsulation

```typescript
// Basic ViewEncapsulation example
import { Component, ViewEncapsulation } from "@angular/core";

@Component({
  selector: "app-example",
  template: `<div class="container">Content</div>`,
  styles: [
    `
      .container {
        background: blue;
        color: white;
      }
    `,
  ],
  encapsulation: ViewEncapsulation.Emulated, // Default
})
export class ExampleComponent {}
```

**How it works in the DOM:**

```html
<!-- ViewEncapsulation.Emulated (Default) -->
<app-example _ngcontent-abc-123>
  <div class="container" _ngcontent-abc-123>Content</div>
</app-example>

<!-- Generated CSS -->
<style>
  .container[_ngcontent-abc-123] {
    background: blue;
    color: white;
  }
</style>

<!-- ViewEncapsulation.None -->
<app-example>
  <div class="container">Content</div>
</app-example>

<!-- Generated CSS (Global) -->
<style>
  .container {
    background: blue;
    color: white;
  }
</style>

<!-- ViewEncapsulation.ShadowDOM -->
<app-example>
  #shadow-root
  <style>
    .container {
      background: blue;
      color: white;
    }
  </style>
  <div class="container">Content</div>
</app-example>
```

---

## 🔒 **ViewEncapsulation.Emulated (Default)**

### **🛡️ Safe Encapsulation with Performance**

```typescript
// src/app/components/user-card/user-card.component.ts
import { Component, Input, ViewEncapsulation } from "@angular/core";

@Component({
  selector: "app-user-card",
  encapsulation: ViewEncapsulation.Emulated, // Explicit (default)
  template: `
    <div class="card">
      <div class="card-header">
        <img [src]="user?.avatar" [alt]="user?.name" class="avatar" />
        <div class="user-info">
          <h3 class="name">{{ user?.name }}</h3>
          <p class="email">{{ user?.email }}</p>
        </div>
      </div>

      <div class="card-body">
        <div class="stats">
          <div class="stat">
            <span class="label">Status</span>
            <span class="value" [class]="user?.status">{{ user?.status }}</span>
          </div>
          <div class="stat">
            <span class="label">Department</span>
            <span class="value">{{ user?.department }}</span>
          </div>
        </div>
      </div>

      <div class="card-actions">
        <button class="btn btn-primary">Edit</button>
        <button class="btn btn-secondary">View</button>
      </div>
    </div>
  `,
  styles: [
    `
      /* 🎯 These styles are SCOPED to this component */
      .card {
        background: white;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        overflow: hidden;
        transition: transform 0.2s ease;
      }

      .card:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
      }

      .card-header {
        display: flex;
        align-items: center;
        padding: 20px;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
      }

      .avatar {
        width: 60px;
        height: 60px;
        border-radius: 50%;
        margin-right: 15px;
        border: 3px solid rgba(255, 255, 255, 0.2);
      }

      .user-info {
        flex: 1;
      }

      .name {
        margin: 0 0 5px 0;
        font-size: 1.25rem;
        font-weight: 600;
      }

      .email {
        margin: 0;
        opacity: 0.9;
        font-size: 0.875rem;
      }

      .card-body {
        padding: 20px;
      }

      .stats {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 15px;
      }

      .stat {
        text-align: center;
        padding: 15px;
        background: #f8f9fa;
        border-radius: 6px;
      }

      .stat .label {
        display: block;
        font-size: 0.75rem;
        color: #666;
        text-transform: uppercase;
        letter-spacing: 0.5px;
        margin-bottom: 5px;
      }

      .stat .value {
        display: block;
        font-weight: 600;
        font-size: 1rem;
      }

      .stat .value.active {
        color: #28a745;
      }

      .stat .value.inactive {
        color: #dc3545;
      }

      .card-actions {
        padding: 20px;
        background: #f8f9fa;
        display: flex;
        gap: 10px;
        justify-content: flex-end;
      }

      .btn {
        padding: 8px 16px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-weight: 500;
        transition: all 0.2s ease;
      }

      .btn-primary {
        background: #007bff;
        color: white;
      }

      .btn-primary:hover {
        background: #0056b3;
        transform: translateY(-1px);
      }

      .btn-secondary {
        background: #6c757d;
        color: white;
      }

      .btn-secondary:hover {
        background: #545b62;
        transform: translateY(-1px);
      }

      /* 📱 Responsive design */
      @media (max-width: 768px) {
        .card-header {
          flex-direction: column;
          text-align: center;
        }

        .avatar {
          margin-right: 0;
          margin-bottom: 15px;
        }

        .stats {
          grid-template-columns: 1fr;
        }

        .card-actions {
          flex-direction: column;
        }
      }
    `,
  ],
})
export class UserCardComponent {
  @Input() user?: {
    name: string;
    email: string;
    avatar: string;
    status: "active" | "inactive";
    department: string;
  };
}
```

**Generated Output Example:**

```html
<!-- In the DOM -->
<app-user-card _ngcontent-abc-123="">
  <div class="card" _ngcontent-abc-123="">
    <div class="card-header" _ngcontent-abc-123="">
      <!-- Content with scoped attributes -->
    </div>
  </div>
</app-user-card>
```

```css
/* Generated CSS with unique attributes */
.card[_ngcontent-abc-123] {
  background: white;
  border-radius: 8px;
  /* ... */
}

.card[_ngcontent-abc-123]:hover {
  transform: translateY(-2px);
  /* ... */
}
```

---

## 🌐 **ViewEncapsulation.None (Global Styles)**

### **🌍 When You Need Global Impact**

```typescript
// src/app/components/global-modal/global-modal.component.ts
import { Component, ViewEncapsulation } from "@angular/core";

@Component({
  selector: "app-global-modal",
  encapsulation: ViewEncapsulation.None, // No encapsulation
  template: `
    <div class="modal-overlay" *ngIf="isOpen">
      <div class="modal-container">
        <div class="modal-header">
          <h2 class="modal-title">Global Modal</h2>
          <button class="modal-close" (click)="close()">×</button>
        </div>

        <div class="modal-body">
          <ng-content></ng-content>
        </div>

        <div class="modal-footer">
          <ng-content select="[slot=footer]"></ng-content>
        </div>
      </div>
    </div>
  `,
  styles: [
    `
      /* ⚠️ These styles are GLOBAL - affect entire application */

      /* Modal overlay covers entire viewport */
      .modal-overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background: rgba(0, 0, 0, 0.5);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 9999;
        backdrop-filter: blur(2px);
        animation: fadeIn 0.3s ease;
      }

      .modal-container {
        background: white;
        border-radius: 12px;
        box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
        max-width: 90vw;
        max-height: 90vh;
        overflow: hidden;
        animation: slideIn 0.3s ease;
      }

      .modal-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 24px;
        border-bottom: 1px solid #e9ecef;
        background: #f8f9fa;
      }

      .modal-title {
        margin: 0;
        font-size: 1.5rem;
        font-weight: 600;
        color: #333;
      }

      .modal-close {
        background: none;
        border: none;
        font-size: 2rem;
        color: #666;
        cursor: pointer;
        padding: 0;
        width: 40px;
        height: 40px;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        transition: all 0.2s ease;
      }

      .modal-close:hover {
        background: rgba(0, 0, 0, 0.1);
        color: #333;
      }

      .modal-body {
        padding: 32px 24px;
        max-height: 60vh;
        overflow-y: auto;
      }

      .modal-footer {
        padding: 20px 24px;
        border-top: 1px solid #e9ecef;
        background: #f8f9fa;
        display: flex;
        justify-content: flex-end;
        gap: 12px;
      }

      /* 🎬 Animations */
      @keyframes fadeIn {
        from {
          opacity: 0;
        }
        to {
          opacity: 1;
        }
      }

      @keyframes slideIn {
        from {
          opacity: 0;
          transform: scale(0.9) translateY(-20px);
        }
        to {
          opacity: 1;
          transform: scale(1) translateY(0);
        }
      }

      /* 🛡️ Prevent body scroll when modal is open */
      body.modal-open {
        overflow: hidden;
      }

      /* 📱 Mobile responsive */
      @media (max-width: 768px) {
        .modal-container {
          margin: 20px;
          max-width: calc(100vw - 40px);
          max-height: calc(100vh - 40px);
        }

        .modal-header,
        .modal-body,
        .modal-footer {
          padding: 16px;
        }

        .modal-title {
          font-size: 1.25rem;
        }
      }

      /* 🎯 Global utility classes for modal content */
      .modal-section {
        margin-bottom: 24px;
      }

      .modal-section:last-child {
        margin-bottom: 0;
      }

      .modal-text {
        line-height: 1.6;
        color: #555;
      }

      .modal-highlight {
        background: linear-gradient(120deg, #a8edea 0%, #fed6e3 100%);
        padding: 2px 6px;
        border-radius: 4px;
      }

      /* 🔘 Modal-specific button styles */
      .modal-btn {
        padding: 10px 20px;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-weight: 600;
        transition: all 0.2s ease;
        min-width: 100px;
      }

      .modal-btn-primary {
        background: #007bff;
        color: white;
      }

      .modal-btn-primary:hover {
        background: #0056b3;
        transform: translateY(-1px);
        box-shadow: 0 4px 12px rgba(0, 123, 255, 0.3);
      }

      .modal-btn-secondary {
        background: #6c757d;
        color: white;
      }

      .modal-btn-secondary:hover {
        background: #545b62;
        transform: translateY(-1px);
      }

      .modal-btn-danger {
        background: #dc3545;
        color: white;
      }

      .modal-btn-danger:hover {
        background: #c82333;
        transform: translateY(-1px);
        box-shadow: 0 4px 12px rgba(220, 53, 69, 0.3);
      }
    `,
  ],
})
export class GlobalModalComponent {
  isOpen = false;

  open(): void {
    this.isOpen = true;
    document.body.classList.add("modal-open");
  }

  close(): void {
    this.isOpen = false;
    document.body.classList.remove("modal-open");
  }
}

// Usage in other components
@Component({
  selector: "app-modal-demo",
  template: `
    <div class="demo-container">
      <button class="demo-btn" (click)="openModal()">Open Global Modal</button>

      <app-global-modal #globalModal>
        <div class="modal-section">
          <h3>Modal Content</h3>
          <p class="modal-text">
            This modal uses
            <span class="modal-highlight">ViewEncapsulation.None</span>
            so its styles are applied globally and can affect the entire
            application.
          </p>
        </div>

        <div class="modal-section">
          <h4>Features:</h4>
          <ul>
            <li>Global z-index management</li>
            <li>Backdrop blur effect</li>
            <li>Smooth animations</li>
            <li>Mobile responsive</li>
            <li>Body scroll prevention</li>
          </ul>
        </div>

        <div slot="footer">
          <button
            class="modal-btn modal-btn-secondary"
            (click)="globalModal.close()"
          >
            Cancel
          </button>
          <button
            class="modal-btn modal-btn-primary"
            (click)="handleConfirm(globalModal)"
          >
            Confirm
          </button>
        </div>
      </app-global-modal>
    </div>
  `,
  styles: [
    `
      .demo-container {
        padding: 40px;
        text-align: center;
      }

      .demo-btn {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        border: none;
        padding: 12px 24px;
        border-radius: 8px;
        cursor: pointer;
        font-weight: 600;
        transition: transform 0.2s ease;
      }

      .demo-btn:hover {
        transform: translateY(-2px);
      }
    `,
  ],
})
export class ModalDemoComponent {
  openModal(): void {
    // Access modal via ViewChild or template reference
  }

  handleConfirm(modal: GlobalModalComponent): void {
    console.log("✅ Modal confirmed!");
    modal.close();
  }
}
```

**⚠️ Important Considerations for ViewEncapsulation.None:**

```typescript
// Best practices for global styles
@Component({
  encapsulation: ViewEncapsulation.None,
  // 🎯 Use specific class prefixes to avoid conflicts
  styles: [
    `
      /* ✅ Good: Prefixed classes */
      .my-component-container {
      }
      .my-component-header {
      }

      /* ❌ Bad: Generic classes that might conflict */
      .container {
      }
      .header {
      }

      /* 🎯 Or scope to component selector */
      app-my-component .container {
      }
      app-my-component .header {
      }
    `,
  ],
})
export class MyComponent {}
```

---

## 👥 **ViewEncapsulation.ShadowDOM (True Isolation)**

### **🔒 Browser-Native Encapsulation**

```typescript
// src/app/components/isolated-widget/isolated-widget.component.ts
import { Component, ViewEncapsulation, Input } from "@angular/core";

@Component({
  selector: "app-isolated-widget",
  encapsulation: ViewEncapsulation.ShadowDOM, // True Shadow DOM
  template: `
    <div class="widget">
      <div class="widget-header">
        <h3 class="widget-title">{{ title }}</h3>
        <div class="widget-actions">
          <button class="action-btn" (click)="refresh()">🔄</button>
          <button class="action-btn" (click)="minimize()">➖</button>
          <button class="action-btn" (click)="close()">✖</button>
        </div>
      </div>

      <div class="widget-content" [class.minimized]="isMinimized">
        <div class="content-wrapper">
          <ng-content></ng-content>
        </div>
      </div>
    </div>
  `,
  styles: [
    `
      /* 🛡️ These styles are COMPLETELY isolated in Shadow DOM */

      :host {
        display: block;
        margin: 10px;
        font-family: "Segoe UI", system-ui, sans-serif;
      }

      .widget {
        background: white;
        border: 1px solid #e1e5e9;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        overflow: hidden;
        transition: all 0.3s ease;
      }

      .widget:hover {
        box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
      }

      .widget-header {
        background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 100%);
        color: white;
        padding: 12px 16px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        user-select: none;
      }

      .widget-title {
        margin: 0;
        font-size: 1rem;
        font-weight: 600;
      }

      .widget-actions {
        display: flex;
        gap: 8px;
      }

      .action-btn {
        background: rgba(255, 255, 255, 0.2);
        border: none;
        color: white;
        width: 24px;
        height: 24px;
        border-radius: 4px;
        cursor: pointer;
        font-size: 12px;
        display: flex;
        align-items: center;
        justify-content: center;
        transition: background 0.2s ease;
      }

      .action-btn:hover {
        background: rgba(255, 255, 255, 0.3);
      }

      .widget-content {
        max-height: 300px;
        overflow: hidden;
        transition: max-height 0.3s ease;
      }

      .widget-content.minimized {
        max-height: 0;
      }

      .content-wrapper {
        padding: 16px;
      }

      /* 🎨 Custom scrollbar for Shadow DOM */
      .content-wrapper::-webkit-scrollbar {
        width: 6px;
      }

      .content-wrapper::-webkit-scrollbar-track {
        background: #f1f1f1;
        border-radius: 3px;
      }

      .content-wrapper::-webkit-scrollbar-thumb {
        background: #c1c1c1;
        border-radius: 3px;
      }

      .content-wrapper::-webkit-scrollbar-thumb:hover {
        background: #a8a8a8;
      }

      /* 🎯 Reset any external styles that might interfere */
      * {
        box-sizing: border-box;
      }

      /* 📱 Responsive behavior */
      @media (max-width: 768px) {
        .widget-header {
          padding: 10px 12px;
        }

        .widget-title {
          font-size: 0.875rem;
        }

        .action-btn {
          width: 28px;
          height: 28px;
          font-size: 14px;
        }
      }

      /* 🎨 Animation utilities */
      @keyframes slideDown {
        from {
          opacity: 0;
          transform: translateY(-10px);
        }
        to {
          opacity: 1;
          transform: translateY(0);
        }
      }

      .content-wrapper {
        animation: slideDown 0.3s ease;
      }

      /* 🔧 Utility classes for content */
      .highlight {
        background: linear-gradient(120deg, #a8edea 0%, #fed6e3 100%);
        padding: 2px 6px;
        border-radius: 4px;
        font-weight: 500;
      }

      .success {
        color: #28a745;
        font-weight: 600;
      }

      .warning {
        color: #ffc107;
        font-weight: 600;
      }

      .error {
        color: #dc3545;
        font-weight: 600;
      }

      .code {
        background: #f8f9fa;
        padding: 2px 6px;
        border-radius: 3px;
        font-family: "Consolas", "Monaco", monospace;
        font-size: 0.875rem;
        border: 1px solid #e9ecef;
      }
    `,
  ],
})
export class IsolatedWidgetComponent {
  @Input() title = "Widget";

  isMinimized = false;

  refresh(): void {
    console.log("🔄 Widget refreshed");
  }

  minimize(): void {
    this.isMinimized = !this.isMinimized;
    console.log(
      `${this.isMinimized ? "➖" : "⬆️"} Widget ${
        this.isMinimized ? "minimized" : "expanded"
      }`
    );
  }

  close(): void {
    console.log("✖ Widget closed");
    // Emit close event or handle removal
  }
}

// Usage example with multiple widgets
@Component({
  selector: "app-widget-dashboard",
  template: `
    <div class="dashboard">
      <h1>Widget Dashboard</h1>

      <div class="widgets-grid">
        <app-isolated-widget title="User Stats">
          <div class="stats-content">
            <p>Total Users: <span class="highlight">1,247</span></p>
            <p>Active Today: <span class="success">892</span></p>
            <p>Pending: <span class="warning">23</span></p>
            <p>Issues: <span class="error">3</span></p>
          </div>
        </app-isolated-widget>

        <app-isolated-widget title="System Status">
          <div class="status-content">
            <p>CPU Usage: <span class="code">45%</span></p>
            <p>Memory: <span class="code">2.1GB / 8GB</span></p>
            <p>Disk: <span class="code">156GB / 500GB</span></p>
            <p>Network: <span class="success">Optimal</span></p>
          </div>
        </app-isolated-widget>

        <app-isolated-widget title="Recent Activity">
          <div class="activity-content">
            <div class="activity-item">
              <span class="timestamp">2:30 PM</span>
              <span class="action"
                >User login: <span class="code">john@example.com</span></span
              >
            </div>
            <div class="activity-item">
              <span class="timestamp">2:25 PM</span>
              <span class="action"
                >File uploaded: <span class="code">report.pdf</span></span
              >
            </div>
            <div class="activity-item">
              <span class="timestamp">2:20 PM</span>
              <span class="action"
                >Database backup: <span class="success">Completed</span></span
              >
            </div>
          </div>
        </app-isolated-widget>
      </div>
    </div>
  `,
  styles: [
    `
      .dashboard {
        padding: 20px;
        background: #f5f6fa;
        min-height: 100vh;
      }

      .dashboard h1 {
        color: #333;
        margin-bottom: 30px;
        text-align: center;
      }

      .widgets-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 20px;
        max-width: 1200px;
        margin: 0 auto;
      }

      /* 🎨 Content styling for widgets */
      .stats-content p,
      .status-content p {
        margin: 8px 0;
        font-size: 14px;
      }

      .activity-content {
        font-size: 14px;
      }

      .activity-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 8px 0;
        border-bottom: 1px solid #eee;
      }

      .activity-item:last-child {
        border-bottom: none;
      }

      .timestamp {
        color: #666;
        font-size: 12px;
        white-space: nowrap;
      }

      .action {
        flex: 1;
        margin-left: 10px;
        color: #333;
      }
    `,
  ],
})
export class WidgetDashboardComponent {}
```

**Shadow DOM Benefits:**

- ✅ **True Isolation** - Styles cannot leak in or out
- ✅ **CSS Reset Safe** - External resets don't affect component
- ✅ **Third-party Safe** - No conflicts with external libraries
- ✅ **Predictable** - Styles work exactly as written

**Shadow DOM Limitations:**

- ❌ **Limited Browser Support** - Older browsers may not support
- ❌ **Styling Challenges** - Harder to apply global themes
- ❌ **DevTools Complexity** - Debugging requires Shadow DOM inspection
- ❌ **Performance** - Slight overhead compared to emulated

---

## 🎭 **Advanced CSS Strategies**

### **🔗 CSS Custom Properties (CSS Variables) Bridge**

```typescript
// src/app/components/themeable-card/themeable-card.component.ts
import { Component, ViewEncapsulation, Input } from "@angular/core";

@Component({
  selector: "app-themeable-card",
  encapsulation: ViewEncapsulation.Emulated, // or ShadowDOM
  template: `
    <div class="card" [attr.data-theme]="theme">
      <div class="card-header">
        <h3 class="card-title">{{ title }}</h3>
        <div class="card-badge" *ngIf="badge">{{ badge }}</div>
      </div>

      <div class="card-content">
        <ng-content></ng-content>
      </div>

      <div class="card-footer" *ngIf="showFooter">
        <ng-content select="[slot=footer]"></ng-content>
      </div>
    </div>
  `,
  styles: [
    `
      /* 🎨 CSS Custom Properties for theming */
      :host {
        --card-bg: var(--theme-card-bg, #ffffff);
        --card-border: var(--theme-card-border, #e1e5e9);
        --card-shadow: var(--theme-card-shadow, 0 2px 8px rgba(0, 0, 0, 0.1));
        --card-radius: var(--theme-card-radius, 8px);

        --header-bg: var(--theme-header-bg, #f8f9fa);
        --header-color: var(--theme-header-color, #333);

        --title-size: var(--theme-title-size, 1.25rem);
        --title-weight: var(--theme-title-weight, 600);

        --content-padding: var(--theme-content-padding, 20px);
        --footer-bg: var(--theme-footer-bg, #f8f9fa);

        display: block;
        margin: 10px;
      }

      /* 🌓 Dark theme overrides */
      :host([data-theme="dark"]) {
        --card-bg: #2d3748;
        --card-border: #4a5568;
        --card-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);

        --header-bg: #1a202c;
        --header-color: #e2e8f0;

        --footer-bg: #1a202c;
      }

      /* 🟢 Success theme */
      :host([data-theme="success"]) {
        --card-border: #28a745;
        --header-bg: linear-gradient(135deg, #28a745, #20c997);
        --header-color: white;
        --card-shadow: 0 2px 8px rgba(40, 167, 69, 0.2);
      }

      /* 🔴 Danger theme */
      :host([data-theme="danger"]) {
        --card-border: #dc3545;
        --header-bg: linear-gradient(135deg, #dc3545, #fd7e14);
        --header-color: white;
        --card-shadow: 0 2px 8px rgba(220, 53, 69, 0.2);
      }

      /* 🔵 Primary theme */
      :host([data-theme="primary"]) {
        --card-border: #007bff;
        --header-bg: linear-gradient(135deg, #007bff, #6f42c1);
        --header-color: white;
        --card-shadow: 0 2px 8px rgba(0, 123, 255, 0.2);
      }

      .card {
        background: var(--card-bg);
        border: 1px solid var(--card-border);
        border-radius: var(--card-radius);
        box-shadow: var(--card-shadow);
        overflow: hidden;
        transition: all 0.3s ease;
      }

      .card:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
      }

      .card-header {
        background: var(--header-bg);
        color: var(--header-color);
        padding: 16px var(--content-padding);
        display: flex;
        justify-content: space-between;
        align-items: center;
      }

      .card-title {
        margin: 0;
        font-size: var(--title-size);
        font-weight: var(--title-weight);
      }

      .card-badge {
        background: rgba(255, 255, 255, 0.2);
        color: inherit;
        padding: 4px 8px;
        border-radius: 12px;
        font-size: 0.75rem;
        font-weight: 600;
      }

      .card-content {
        padding: var(--content-padding);
        line-height: 1.6;
      }

      .card-footer {
        background: var(--footer-bg);
        padding: 12px var(--content-padding);
        border-top: 1px solid var(--card-border);
      }

      /* 📱 Responsive adjustments */
      @media (max-width: 768px) {
        :host {
          --content-padding: 16px;
          --title-size: 1.125rem;
        }

        .card-header {
          flex-direction: column;
          gap: 8px;
          text-align: center;
        }
      }
    `,
  ],
})
export class ThemeableCardComponent {
  @Input() title = "";
  @Input() badge = "";
  @Input() theme: "default" | "dark" | "success" | "danger" | "primary" =
    "default";
  @Input() showFooter = false;
}

// Global theme service
@Injectable({ providedIn: "root" })
export class ThemeService {
  private currentTheme = new BehaviorSubject<"light" | "dark">("light");

  theme$ = this.currentTheme.asObservable();

  constructor() {
    this.initializeTheme();
  }

  setTheme(theme: "light" | "dark"): void {
    this.currentTheme.next(theme);
    document.documentElement.setAttribute("data-theme", theme);

    // Update CSS custom properties globally
    if (theme === "dark") {
      document.documentElement.style.setProperty("--theme-card-bg", "#2d3748");
      document.documentElement.style.setProperty(
        "--theme-card-border",
        "#4a5568"
      );
      document.documentElement.style.setProperty(
        "--theme-header-color",
        "#e2e8f0"
      );
    } else {
      document.documentElement.style.setProperty("--theme-card-bg", "#ffffff");
      document.documentElement.style.setProperty(
        "--theme-card-border",
        "#e1e5e9"
      );
      document.documentElement.style.setProperty(
        "--theme-header-color",
        "#333"
      );
    }

    localStorage.setItem("app-theme", theme);
  }

  getCurrentTheme(): "light" | "dark" {
    return this.currentTheme.value;
  }

  toggleTheme(): void {
    const newTheme = this.currentTheme.value === "light" ? "dark" : "light";
    this.setTheme(newTheme);
  }

  private initializeTheme(): void {
    const savedTheme = localStorage.getItem("app-theme") as "light" | "dark";
    const systemPrefersDark = window.matchMedia(
      "(prefers-color-scheme: dark)"
    ).matches;

    const initialTheme = savedTheme || (systemPrefersDark ? "dark" : "light");
    this.setTheme(initialTheme);

    // Listen for system theme changes
    window
      .matchMedia("(prefers-color-scheme: dark)")
      .addEventListener("change", (e) => {
        if (!localStorage.getItem("app-theme")) {
          this.setTheme(e.matches ? "dark" : "light");
        }
      });
  }
}
```

---

## 🎉 **Summary: CSS Resolution Mastery**

### **✅ What We've Covered:**

🔒 **ViewEncapsulation.Emulated** - Safe, performance-optimized encapsulation  
🌐 **ViewEncapsulation.None** - Global styles for special cases  
👥 **ViewEncapsulation.ShadowDOM** - True browser-native isolation  
🎨 **CSS Custom Properties** - Dynamic theming across encapsulation boundaries  
🔧 **Advanced Strategies** - Production-ready styling patterns

### **🎯 Best Practices:**

- ✅ **Use Emulated by default** - Best balance of isolation and performance
- ✅ **Use None sparingly** - Only for truly global components (modals, overlays)
- ✅ **Use ShadowDOM carefully** - When complete isolation is critical
- ✅ **CSS Variables for theming** - Bridge encapsulation boundaries
- ✅ **Component-specific prefixes** - Avoid naming conflicts
- ✅ **Mobile-first responsive** - Design for all screen sizes

**You're now ready to master CSS encapsulation in Angular!** 🚀
