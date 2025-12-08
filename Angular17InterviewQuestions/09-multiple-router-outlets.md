# 🛣️ Multiple Router Outlets in Angular: Advanced Routing

## 🎯 **Question Overview**

_"When should we use multiple router outlets?"_

## 🔍 **Understanding Router Outlets**

Router outlets in Angular are **placeholders** where routed components are displayed. While most applications use a single primary outlet, **multiple router outlets** allow you to display different components simultaneously in different areas of your application.

Think of multiple outlets as having **multiple TV screens** in your living room - each can show different content at the same time! 📺

## 🎯 **When to Use Multiple Router Outlets**

### **1. 🏢 Complex Application Layouts**

When your application has multiple distinct content areas that need independent navigation:

```typescript
// App layout with multiple areas
@Component({
  selector: "app-layout",
  template: `
    <div class="app-layout">
      <!-- Header with navigation -->
      <header class="header">
        <nav>
          <a routerLink="/dashboard">Dashboard</a>
          <a routerLink="/users">Users</a>
          <a routerLink="/settings">Settings</a>
        </nav>
      </header>

      <div class="content-area">
        <!-- Main content outlet -->
        <main class="main-content">
          <router-outlet></router-outlet>
        </main>

        <!-- Secondary sidebar outlet -->
        <aside class="sidebar">
          <router-outlet name="sidebar"></router-outlet>
        </aside>

        <!-- Modal/popup outlet -->
        <div class="modal-area">
          <router-outlet name="modal"></router-outlet>
        </div>
      </div>

      <!-- Footer outlet for contextual actions -->
      <footer class="footer">
        <router-outlet name="footer"></router-outlet>
      </footer>
    </div>
  `,
  styles: [
    `
      .app-layout {
        display: grid;
        grid-template-rows: auto 1fr auto;
        height: 100vh;
      }

      .content-area {
        display: grid;
        grid-template-columns: 1fr 300px;
        gap: 20px;
      }

      .modal-area {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
        z-index: 1000;
      }
    `,
  ],
  standalone: true,
  imports: [RouterOutlet, RouterLink],
})
export class AppLayoutComponent {}
```

### **2. 📊 Dashboard Applications**

For dashboards where you need independent panels:

```typescript
// Routes configuration for dashboard
const routes: Routes = [
  {
    path: "dashboard",
    component: DashboardLayoutComponent,
    children: [
      // Main dashboard content
      {
        path: "",
        component: DashboardHomeComponent,
      },
      {
        path: "analytics",
        component: AnalyticsComponent,
      },

      // Sidebar content
      {
        path: "",
        component: DashboardSidebarComponent,
        outlet: "sidebar",
      },
      {
        path: "notifications",
        component: NotificationPanelComponent,
        outlet: "sidebar",
      },
      {
        path: "quick-actions",
        component: QuickActionsComponent,
        outlet: "sidebar",
      },

      // Footer actions
      {
        path: "",
        component: DashboardFooterComponent,
        outlet: "footer",
      },
    ],
  },
];

// Navigation with multiple outlets
@Component({
  selector: "app-dashboard",
  template: `
    <div class="dashboard-nav">
      <button
        [routerLink]="[
          { outlets: { primary: 'analytics', sidebar: 'notifications' } }
        ]"
      >
        Analytics + Notifications
      </button>
      <button
        [routerLink]="[
          { outlets: { primary: null, sidebar: 'quick-actions' } }
        ]"
      >
        Home + Quick Actions
      </button>
    </div>
  `,
})
export class DashboardNavComponent {}
```

### **3. 🎭 Modal and Popup Management**

For handling modals through routing:

```typescript
// Modal routing configuration
const routes: Routes = [
  {
    path: "users",
    component: UserListComponent,
    children: [
      // Modal outlets for user actions
      {
        path: "create",
        component: CreateUserModalComponent,
        outlet: "modal",
      },
      {
        path: "edit/:id",
        component: EditUserModalComponent,
        outlet: "modal",
      },
      {
        path: "delete/:id",
        component: DeleteConfirmModalComponent,
        outlet: "modal",
      },
    ],
  },
];

// User list with modal outlet
@Component({
  selector: "app-user-list",
  template: `
    <div class="user-list">
      <div class="header">
        <h2>Users</h2>
        <button [routerLink]="[{ outlets: { modal: 'create' } }]">
          Add User
        </button>
      </div>

      <table>
        <tr *ngFor="let user of users">
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>
            <button [routerLink]="[{ outlets: { modal: ['edit', user.id] } }]">
              Edit
            </button>
            <button
              [routerLink]="[{ outlets: { modal: ['delete', user.id] } }]"
            >
              Delete
            </button>
          </td>
        </tr>
      </table>
    </div>

    <!-- Modal outlet -->
    <router-outlet name="modal"></router-outlet>
  `,
  standalone: true,
  imports: [CommonModule, RouterOutlet, RouterLink],
})
export class UserListComponent {
  users: User[] = [];
}
```

### **4. 📱 Master-Detail Interfaces**

For applications with master-detail patterns:

```typescript
// Master-detail routing
const routes: Routes = [
  {
    path: "products",
    component: ProductMasterDetailComponent,
    children: [
      // Master list
      {
        path: "",
        component: ProductListComponent,
      },
      {
        path: "category/:categoryId",
        component: ProductListComponent,
      },

      // Detail panel
      {
        path: "details/:id",
        component: ProductDetailComponent,
        outlet: "detail",
      },
      {
        path: "edit/:id",
        component: ProductEditComponent,
        outlet: "detail",
      },
    ],
  },
];

@Component({
  selector: "app-product-master-detail",
  template: `
    <div class="master-detail-layout">
      <!-- Master panel -->
      <div class="master-panel">
        <router-outlet></router-outlet>
      </div>

      <!-- Detail panel -->
      <div class="detail-panel" [class.hidden]="!hasDetailRoute">
        <div class="detail-header">
          <button (click)="closeDetail()" class="close-btn">×</button>
        </div>
        <router-outlet name="detail"></router-outlet>
      </div>
    </div>
  `,
  styles: [
    `
      .master-detail-layout {
        display: grid;
        grid-template-columns: 1fr 1fr;
        height: 100vh;
        gap: 20px;
      }

      .detail-panel.hidden {
        display: none;
      }

      @media (max-width: 768px) {
        .master-detail-layout {
          grid-template-columns: 1fr;
        }

        .detail-panel:not(.hidden) + .master-panel {
          display: none;
        }
      }
    `,
  ],
  standalone: true,
  imports: [RouterOutlet],
})
export class ProductMasterDetailComponent {
  hasDetailRoute = false;

  constructor(private router: Router) {
    this.router.events
      .pipe(filter((event) => event instanceof NavigationEnd))
      .subscribe(() => {
        this.hasDetailRoute = this.router.url.includes("(detail:");
      });
  }

  closeDetail() {
    this.router.navigate([{ outlets: { detail: null } }], {
      relativeTo: this.activatedRoute,
    });
  }
}
```

## 🔧 **Advanced Multiple Outlet Patterns**

### **1. 🎪 Tabbed Interface with Independent Navigation**

```typescript
// Tabbed interface with separate routing
const routes: Routes = [
  {
    path: "workspace",
    component: WorkspaceComponent,
    children: [
      // Tab 1: Code editor
      {
        path: "file/:fileName",
        component: CodeEditorComponent,
        outlet: "editor",
      },

      // Tab 2: File explorer
      {
        path: "folder/:path",
        component: FileExplorerComponent,
        outlet: "explorer",
      },

      // Tab 3: Terminal
      {
        path: "terminal/:sessionId",
        component: TerminalComponent,
        outlet: "terminal",
      },

      // Status bar
      {
        path: "",
        component: StatusBarComponent,
        outlet: "status",
      },
    ],
  },
];

@Component({
  selector: "app-workspace",
  template: `
    <div class="workspace">
      <!-- Tab navigation -->
      <div class="tab-nav">
        <button
          [routerLink]="[{ outlets: { editor: ['file', 'main.ts'] } }]"
          routerLinkActive="active"
        >
          Code Editor
        </button>
        <button
          [routerLink]="[{ outlets: { explorer: ['folder', 'src'] } }]"
          routerLinkActive="active"
        >
          File Explorer
        </button>
        <button
          [routerLink]="[{ outlets: { terminal: ['terminal', 'main'] } }]"
          routerLinkActive="active"
        >
          Terminal
        </button>
      </div>

      <!-- Content areas -->
      <div class="content-grid">
        <div class="editor-area">
          <router-outlet name="editor"></router-outlet>
        </div>
        <div class="explorer-area">
          <router-outlet name="explorer"></router-outlet>
        </div>
        <div class="terminal-area">
          <router-outlet name="terminal"></router-outlet>
        </div>
      </div>

      <!-- Status bar -->
      <div class="status-bar">
        <router-outlet name="status"></router-outlet>
      </div>
    </div>
  `,
  styles: [
    `
      .workspace {
        display: grid;
        grid-template-rows: auto 1fr auto;
        height: 100vh;
      }

      .content-grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        grid-template-rows: 1fr 200px;
        gap: 10px;
      }

      .editor-area {
        grid-column: 1 / -1;
      }
      .explorer-area {
        grid-row: 2;
      }
      .terminal-area {
        grid-row: 2;
      }
    `,
  ],
})
export class WorkspaceComponent {}
```

### **2. 🎮 Gaming Interface with Multiple HUDs**

```typescript
// Gaming interface with multiple HUD elements
const routes: Routes = [
  {
    path: "game",
    component: GameLayoutComponent,
    children: [
      // Main game area
      {
        path: "level/:levelId",
        component: GameCanvasComponent,
      },

      // HUD elements
      {
        path: "",
        component: PlayerStatsComponent,
        outlet: "stats",
      },
      {
        path: "",
        component: MinimapComponent,
        outlet: "minimap",
      },
      {
        path: "",
        component: InventoryComponent,
        outlet: "inventory",
      },
      {
        path: "",
        component: ChatComponent,
        outlet: "chat",
      },

      // Overlay elements
      {
        path: "menu",
        component: GameMenuComponent,
        outlet: "overlay",
      },
      {
        path: "pause",
        component: PauseScreenComponent,
        outlet: "overlay",
      },
    ],
  },
];

@Component({
  selector: "app-game-layout",
  template: `
    <div class="game-interface">
      <!-- Main game area -->
      <div class="game-canvas">
        <router-outlet></router-outlet>
      </div>

      <!-- HUD overlays -->
      <div class="hud-overlay">
        <div class="top-left hud-element">
          <router-outlet name="stats"></router-outlet>
        </div>

        <div class="top-right hud-element">
          <router-outlet name="minimap"></router-outlet>
        </div>

        <div class="bottom-left hud-element">
          <router-outlet name="inventory"></router-outlet>
        </div>

        <div class="bottom-right hud-element">
          <router-outlet name="chat"></router-outlet>
        </div>
      </div>

      <!-- Full-screen overlays -->
      <div class="overlay-area">
        <router-outlet name="overlay"></router-outlet>
      </div>
    </div>
  `,
  styles: [
    `
      .game-interface {
        position: relative;
        width: 100vw;
        height: 100vh;
        overflow: hidden;
      }

      .game-canvas {
        width: 100%;
        height: 100%;
      }

      .hud-overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
      }

      .hud-element {
        position: absolute;
        pointer-events: auto;
      }

      .top-left {
        top: 20px;
        left: 20px;
      }
      .top-right {
        top: 20px;
        right: 20px;
      }
      .bottom-left {
        bottom: 20px;
        left: 20px;
      }
      .bottom-right {
        bottom: 20px;
        right: 20px;
      }

      .overlay-area {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 1000;
      }
    `,
  ],
})
export class GameLayoutComponent {}
```

## 🛠️ **Navigation with Multiple Outlets**

### **1. 📍 Programmatic Navigation**

```typescript
@Injectable({
  providedIn: "root",
})
export class MultiOutletNavigationService {
  constructor(private router: Router) {}

  // Navigate to multiple outlets simultaneously
  navigateToMultipleOutlets(outlets: {
    [name: string]: any;
  }): Promise<boolean> {
    return this.router.navigate([{ outlets }]);
  }

  // Open modal while preserving current route
  openModal(modalRoute: string): Promise<boolean> {
    return this.router.navigate([{ outlets: { modal: modalRoute } }], {
      queryParamsHandling: "preserve",
    });
  }

  // Close specific outlet
  closeOutlet(outletName: string): Promise<boolean> {
    return this.router.navigate([{ outlets: { [outletName]: null } }]);
  }

  // Navigate with complex outlet combinations
  navigateToDashboard(
    mainView: string,
    sidebarView?: string,
    modalView?: string
  ): Promise<boolean> {
    const outlets: any = { primary: mainView };

    if (sidebarView) outlets.sidebar = sidebarView;
    if (modalView) outlets.modal = modalView;
    else outlets.modal = null; // Explicitly close modal

    return this.router.navigate([{ outlets }]);
  }

  // Get current outlet states
  getCurrentOutletStates(): { [outlet: string]: string } {
    const urlTree = this.router.parseUrl(this.router.url);
    const outlets: { [outlet: string]: string } = {};

    // Primary outlet
    if (urlTree.root.children["primary"]) {
      outlets["primary"] = urlTree.root.children["primary"].segments
        .map((segment) => segment.path)
        .join("/");
    }

    // Named outlets
    Object.keys(urlTree.root.children).forEach((outletName) => {
      if (outletName !== "primary") {
        outlets[outletName] = urlTree.root.children[outletName].segments
          .map((segment) => segment.path)
          .join("/");
      }
    });

    return outlets;
  }
}
```

### **2. 🎯 Smart Navigation Component**

```typescript
@Component({
  selector: "app-smart-navigation",
  template: `
    <div class="navigation-controls">
      <div class="outlet-controls">
        <h3>Main Content</h3>
        <button
          *ngFor="let option of mainOptions"
          (click)="setMainContent(option.route)"
          [class.active]="currentOutlets['primary'] === option.route"
        >
          {{ option.label }}
        </button>
      </div>

      <div class="outlet-controls">
        <h3>Sidebar</h3>
        <button
          *ngFor="let option of sidebarOptions"
          (click)="setSidebar(option.route)"
          [class.active]="currentOutlets['sidebar'] === option.route"
        >
          {{ option.label }}
        </button>
        <button
          (click)="closeSidebar()"
          [class.active]="!currentOutlets['sidebar']"
        >
          Close Sidebar
        </button>
      </div>

      <div class="outlet-controls">
        <h3>Modals</h3>
        <button
          *ngFor="let option of modalOptions"
          (click)="openModal(option.route)"
        >
          {{ option.label }}
        </button>
        <button
          (click)="closeModal()"
          [class.active]="!currentOutlets['modal']"
        >
          Close Modal
        </button>
      </div>

      <div class="quick-combinations">
        <h3>Quick Combinations</h3>
        <button (click)="setDashboardView()">Dashboard + Stats</button>
        <button (click)="setAnalyticsView()">Analytics + Charts</button>
        <button (click)="setSettingsView()">Settings + Help</button>
      </div>

      <div class="current-state">
        <h3>Current State</h3>
        <pre>{{ currentOutlets | json }}</pre>
      </div>
    </div>
  `,
  styles: [
    `
      .navigation-controls {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 20px;
        padding: 20px;
      }

      .outlet-controls h3 {
        margin-bottom: 10px;
        color: #333;
      }

      button {
        display: block;
        width: 100%;
        margin-bottom: 5px;
        padding: 8px 12px;
        border: 1px solid #ddd;
        background: white;
        cursor: pointer;
      }

      button:hover {
        background: #f5f5f5;
      }

      button.active {
        background: #007acc;
        color: white;
      }

      .current-state pre {
        background: #f5f5f5;
        padding: 10px;
        border-radius: 4px;
        font-size: 12px;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule, JsonPipe],
})
export class SmartNavigationComponent implements OnInit, OnDestroy {
  currentOutlets: { [outlet: string]: string } = {};

  mainOptions = [
    { route: "dashboard", label: "Dashboard" },
    { route: "analytics", label: "Analytics" },
    { route: "users", label: "Users" },
    { route: "settings", label: "Settings" },
  ];

  sidebarOptions = [
    { route: "stats", label: "Statistics" },
    { route: "notifications", label: "Notifications" },
    { route: "help", label: "Help" },
    { route: "charts", label: "Charts" },
  ];

  modalOptions = [
    { route: "create-user", label: "Create User" },
    { route: "upload", label: "Upload File" },
    { route: "settings", label: "Quick Settings" },
  ];

  private destroy$ = new Subject<void>();

  constructor(
    private router: Router,
    private navService: MultiOutletNavigationService
  ) {}

  ngOnInit() {
    // Monitor route changes
    this.router.events
      .pipe(
        filter((event) => event instanceof NavigationEnd),
        takeUntil(this.destroy$)
      )
      .subscribe(() => {
        this.currentOutlets = this.navService.getCurrentOutletStates();
      });

    // Initial state
    this.currentOutlets = this.navService.getCurrentOutletStates();
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }

  setMainContent(route: string) {
    this.navService.navigateToMultipleOutlets({
      primary: route,
      sidebar: this.currentOutlets["sidebar"] || null,
      modal: null, // Close any open modal
    });
  }

  setSidebar(route: string) {
    this.navService.navigateToMultipleOutlets({
      primary: this.currentOutlets["primary"] || "dashboard",
      sidebar: route,
    });
  }

  closeSidebar() {
    this.navService.closeOutlet("sidebar");
  }

  openModal(route: string) {
    this.navService.openModal(route);
  }

  closeModal() {
    this.navService.closeOutlet("modal");
  }

  // Quick combination methods
  setDashboardView() {
    this.navService.navigateToDashboard("dashboard", "stats");
  }

  setAnalyticsView() {
    this.navService.navigateToDashboard("analytics", "charts");
  }

  setSettingsView() {
    this.navService.navigateToDashboard("settings", "help");
  }
}
```

## 🚨 **Common Pitfalls & Best Practices**

### **❌ Common Mistakes**

1. **Over-complicating simple interfaces**

```typescript
// ❌ Bad - Using outlets for simple component communication
const routes: Routes = [
  {
    path: "simple",
    children: [
      { path: "header", component: HeaderComponent, outlet: "header" },
      { path: "content", component: ContentComponent, outlet: "content" },
      { path: "footer", component: FooterComponent, outlet: "footer" },
    ],
  },
];
// This should just be a regular layout component!
```

2. **Not handling outlet state properly**

```typescript
// ❌ Bad - Not cleaning up outlets
closeModal() {
  this.router.navigate(['/']); // Doesn't clear the modal outlet!
}
```

### **✅ Best Practices**

1. **Use outlets for truly independent areas**

```typescript
// ✅ Good - Outlets for independent functionality
const routes: Routes = [
  {
    path: "workspace",
    children: [
      { path: "file/:id", component: FileEditorComponent },
      { path: "terminal", component: TerminalComponent, outlet: "terminal" },
      { path: "debug/:id", component: DebuggerComponent, outlet: "debugger" },
    ],
  },
];
```

2. **Properly manage outlet lifecycle**

```typescript
// ✅ Good - Explicit outlet management
@Component({})
export class OutletManagerComponent {
  closeAllSecondaryOutlets() {
    this.router.navigate([
      {
        outlets: {
          sidebar: null,
          modal: null,
          footer: null,
        },
      },
    ]);
  }
}
```

## 📊 **Performance Considerations**

### **Bundle Splitting by Outlet**

```typescript
// Lazy load different outlet components
const routes: Routes = [
  {
    path: "app",
    children: [
      {
        path: "editor",
        loadComponent: () =>
          import("./editor/editor.component").then((m) => m.EditorComponent),
        outlet: "editor",
      },
      {
        path: "terminal",
        loadComponent: () =>
          import("./terminal/terminal.component").then(
            (m) => m.TerminalComponent
          ),
        outlet: "terminal",
      },
    ],
  },
];
```

## 🎯 **Key Takeaways**

### **Use Multiple Router Outlets When You Need:**

1. **🏢 Complex layouts** with independent navigation areas
2. **📊 Dashboard interfaces** with multiple panels
3. **🎭 Modal management** through routing
4. **📱 Master-detail interfaces** that need URL state
5. **🎮 Multi-panel applications** like IDEs or games
6. **📺 Media applications** with multiple video/content streams

### **Avoid Multiple Outlets When:**

- ❌ Simple parent-child component relationships
- ❌ Basic layout components
- ❌ Temporary UI state that doesn't need URLs
- ❌ Simple modal dialogs

### **Best Practices:**

1. **Plan your outlet strategy** early in development
2. **Use descriptive outlet names** (not just 'aux', 'secondary')
3. **Handle outlet cleanup** properly
4. **Consider mobile responsive** behavior
5. **Test complex navigation flows** thoroughly
6. **Document outlet usage** for your team

Multiple router outlets are powerful for complex applications but should be used judiciously based on your actual navigation requirements! 🚀
