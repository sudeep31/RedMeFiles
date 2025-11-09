# 🎯 Angular Content Projection, Directives & Pipes - Complete Guide

> **🚀 Comprehensive guide covering Angular Content Projection, Custom Directives, Custom Pipes, Built-in Features, and Dynamic Content Loading for Angular 15 vs Angular 20**

---

## 📋 Table of Contents

1. [🎨 Content Projection](#content-projection)

   - [Basic Projection](#basic-projection)
   - [Multi-Slot Projection](#multi-slot-projection)
   - [Conditional Projection](#conditional-projection)
   - [Angular 15 vs 20 Differences](#projection-differences)

2. [📐 Custom Directives](#custom-directives)

   - [Attribute Directives](#attribute-directives)
   - [Structural Directives](#structural-directives)
   - [Directive Composition](#directive-composition)
   - [Advanced Patterns](#directive-patterns)

3. [🔧 Custom Pipes](#custom-pipes)

   - [Transform Pipes](#transform-pipes)
   - [Async Pipes](#async-pipes)
   - [Pure vs Impure](#pipe-types)
   - [Performance Optimization](#pipe-performance)

4. [🛠️ Built-in Pipes & Directives](#built-in-features)

   - [Common Pipes](#common-pipes)
   - [Structural Directives](#structural-directives-builtin)
   - [Attribute Directives](#attribute-directives-builtin)
   - [Angular 15 vs 20 Updates](#builtin-updates)

5. [⚡ Dynamic Content & Component Loading](#dynamic-loading)

   - [Dynamic Components](#dynamic-components)
   - [ViewContainerRef](#view-container-ref)
   - [Lazy Loading](#lazy-loading)
   - [Module Federation](#module-federation)

6. [🚀 Performance & Best Practices](#performance)
7. [📝 Interview Questions](#interview-questions)

---

## 🎨 Content Projection {#content-projection}

Content projection allows you to create flexible, reusable components by projecting external content into predefined slots within your component's template.

### 🔹 Basic Content Projection {#basic-projection}

The simplest form of content projection using `<ng-content>`.

```typescript
// Basic Card Component with detailed explanations
@Component({
  selector: "app-basic-card", // 📝 Component selector - defines HTML tag name
  standalone: true, // ✨ Angular 20 feature - makes component self-contained without module
  template: `
    <div class="card">
      <!-- 📄 Static header section - always rendered -->
      <div class="card-header">
        <h3>Card Title</h3>
        <!-- 🏷️ Fixed title - could be made dynamic with @Input() -->
      </div>

      <!-- 🎯 CONTENT PROJECTION ZONE - This is where external content gets inserted -->
      <div class="card-body">
        <!-- 
          🔥 <ng-content></ng-content> - THE MAGIC HAPPENS HERE!
          - This is a placeholder for external content
          - Angular replaces this with content passed between component tags
          - Content maintains its original context (CSS, data binding, etc.)
          - No select attribute = accepts ALL content passed to component
        -->
        <ng-content></ng-content>
      </div>

      <!-- ⚡ Static footer section - always rendered -->
      <div class="card-footer">
        <button class="btn-primary">Action</button>
        <!-- 🖱️ Static action button -->
      </div>
    </div>
  `,
  styles: [
    `
      /* 🎨 Card Container Styles */
      .card {
        border: 1px solid #ddd; /* 🖼️ Light gray border for visual separation */
        border-radius: 8px; /* 🔘 Rounded corners for modern appearance */
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* 🌊 Subtle shadow for depth */
        margin: 16px 0; /* 📏 Vertical spacing between cards */
      }

      /* 📦 Common padding for all card sections */
      .card-header,
      .card-body,
      .card-footer {
        padding: 16px; /* 📐 Consistent internal spacing */
      }

      /* 🔝 Header styling */
      .card-header {
        background-color: #f8f9fa; /* 🎨 Light gray background */
        border-bottom: 1px solid #ddd; /* 📏 Visual separation from body */
      }

      /* 🔻 Footer styling */
      .card-footer {
        background-color: #f8f9fa; /* 🎨 Matching header background */
        border-top: 1px solid #ddd; /* 📏 Visual separation from body */
      }

      /* 🖱️ Primary button styling */
      .btn-primary {
        background: #007bff; /* 🔵 Bootstrap-like blue background */
        color: white; /* ⚪ White text for contrast */
        padding: 8px 16px; /* 📐 Button internal spacing */
        border: none; /* 🚫 Remove default border */
        border-radius: 4px; /* 🔘 Rounded button corners */
        cursor: pointer; /* 👆 Pointer cursor on hover */
      }
    `,
  ],
})
export class BasicCardComponent {
  constructor() {
    // 📝 Constructor runs when component instance is created
    // 🔍 Useful for logging component lifecycle events
    console.log("🎨 Basic Card Component initialized");
  }
}

// Usage Example
@Component({
  selector: "app-basic-projection-demo",
  standalone: true,
  imports: [BasicCardComponent],
  template: `
    <div class="demo-container">
      <h2>Basic Content Projection Demo</h2>

      <!-- 
        🎯 PROJECTION EXAMPLE 1: Text Content
        - Everything between <app-basic-card> tags gets projected
        - Content replaces <ng-content></ng-content> in BasicCardComponent
        - Original styling and Angular features are preserved
      -->
      <app-basic-card>
        <!-- 📝 This paragraph will appear inside the card body -->
        <p>This content is <strong>projected</strong> into the card!</p>
        <!-- 📋 This list will also be projected -->
        <ul>
          <li>Flexible content inclusion</li>
          <!-- ✨ Bullet point 1 -->
          <li>Maintains styling context</li>
          <!-- ✨ Bullet point 2 -->
          <li>Component remains reusable</li>
          <!-- ✨ Bullet point 3 -->
        </ul>
      </app-basic-card>

      <!-- 
        🎯 PROJECTION EXAMPLE 2: Rich Content with Images
        - Demonstrates that ANY HTML content can be projected
        - Images, divs, custom styling all work seamlessly
      -->
      <app-basic-card>
        <div class="custom-content">
          <!-- 🖼️ Image element projected into card -->
          <img src="/api/placeholder/200/150" alt="Projected Image" />
          <!-- 📝 Text content alongside image -->
          <p>Images, forms, and any content can be projected!</p>
        </div>
      </app-basic-card>

      <!-- 
        🎯 PROJECTION EXAMPLE 3: Interactive Content (Forms)
        - Shows that interactive elements work perfectly
        - Event handling, form controls, validation all preserved
        - Parent component's context maintained
      -->
      <app-basic-card>
        <form class="projected-form">
          <!-- 📝 Text input with placeholder -->
          <input type="text" placeholder="Enter your name" class="form-input" />
          <!-- 📧 Email input with validation -->
          <input
            type="email"
            placeholder="Enter your email"
            class="form-input"
          />
          <!-- 🚀 Submit button with form handling -->
          <button type="submit" class="form-submit">Submit</button>
        </form>
      </app-basic-card>
    </div>
  `,
  styles: [
    `
      /* 🏗️ MAIN CONTAINER STYLING */
      .demo-container {
        max-width: 800px; /* 📏 Limit maximum width for readability */
        margin: 0 auto; /* 🎯 Center container horizontally */
        padding: 20px; /* 📐 Add internal spacing around content */
      }

      /* 🖼️ CUSTOM CONTENT STYLING (for image projection example) */
      .custom-content {
        text-align: center; /* 🎯 Center-align image and text */
      }

      /* 🖼️ IMAGE RESPONSIVENESS */
      .custom-content img {
        max-width: 100%; /* 📱 Ensure image doesn't overflow container */
        height: auto; /* 📐 Maintain aspect ratio */
        border-radius: 4px; /* 🔘 Rounded image corners */
      }

      /* 📝 FORM LAYOUT (for interactive content example) */
      .projected-form {
        display: flex; /* 🔄 Use flexbox for layout */
        flex-direction: column; /* ⬇️ Stack form elements vertically */
        gap: 12px; /* 📏 Space between form elements */
      }

      /* 📝 INPUT FIELD STYLING */
      .form-input {
        padding: 10px; /* 📐 Internal padding for comfort */
        border: 1px solid #ddd; /* 🖼️ Light border */
        border-radius: 4px; /* 🔘 Rounded corners */
        font-size: 14px; /* 📏 Readable font size */
      }

      /* 🚀 SUBMIT BUTTON STYLING */
      .form-submit {
        background: #28a745; /* 🟢 Green background for success action */
        color: white; /* ⚪ White text for contrast */
        padding: 10px; /* 📐 Button internal spacing */
        border: none; /* 🚫 Remove default border */
        border-radius: 4px; /* 🔘 Rounded button corners */
        cursor: pointer; /* 👆 Pointer cursor on hover */
      }
    `,
  ],
})
export class BasicProjectionDemoComponent {
  constructor() {
    // 📝 Constructor executes when component instance is created
    // 🔍 Perfect place for initialization logging
    // ⚡ Runs before ngOnInit lifecycle hook
    console.log("🎯 Basic projection demo initialized");
  }
}
```

### 🔸 Multi-Slot Content Projection {#multi-slot-projection}

Using named slots to project content into specific locations within a component.

```typescript
// Advanced Card with Named Slots - Multi-Slot Content Projection
@Component({
  selector: "app-multi-slot-card", // 🏷️ Component selector for HTML usage
  standalone: true, // ✨ Self-contained component (Angular 20 feature)
  template: `
    <div class="advanced-card">

      <!--
        🎯 HEADER SECTION WITH NAMED SLOT
        - Uses select="[slot=header]" to target specific content
        - Provides fallback content when no header is projected
        - Multiple slots can exist in same component
      -->
      <div class="card-header">
        <ng-content select="[slot=header]">
          <!-- 🔄 FALLBACK CONTENT: Shows when no header slot content provided -->
          <h3>Default Header</h3>
        </ng-content>

        <!--
          🎮 HEADER ACTIONS SLOT
          - Separate slot for action buttons in header
          - No fallback content = empty if nothing projected
        -->
        <div class="header-actions">
          <ng-content select="[slot=header-actions]"></ng-content>
        </div>
      </div>

      <!--
        🎯 MAIN CONTENT SLOT
        - Primary content area with fallback
        - Most flexible slot for main component content
      -->
      <div class="card-content">
        <ng-content select="[slot=content]">
          <!-- 🔄 FALLBACK: Default message when no content provided -->
          <p>Default content goes here...</p>
        </ng-content>
      </div>

      <!--
        🎯 CONDITIONAL SIDEBAR SLOT
        - Only renders if hasSidebar property is true
        - Demonstrates combining slots with Angular directives
        - *ngIf prevents rendering unused DOM elements
      -->
      <div class="card-sidebar" *ngIf="hasSidebar">
        <ng-content select="[slot=sidebar]"></ng-content>
      </div>

      <!--
        🎯 FOOTER WITH MULTIPLE SUB-SLOTS
        - Complex layout with three separate projection areas
        - Each footer section can have different content
        - Demonstrates fine-grained content control
      -->
      <div class="card-footer">
        <!-- 📍 Left footer section -->
        <div class="footer-left">
          <ng-content select="[slot=footer-left]"></ng-content>
        </div>

        <!-- 📍 Center footer section -->
        <div class="footer-center">
          <ng-content select="[slot=footer-center]"></ng-content>
        </div>

        <!-- 📍 Right footer section with fallback button -->
        <div class="footer-right">
          <ng-content select="[slot=footer-right]">
            <!-- 🔄 Default action button when no content projected -->
            <button class="default-action">Default Action</button>
          </ng-content>
        </div>
      </div>

      <!--
        🎯 FLOATING/OVERLAY CONTENT SLOT
        - Positioned absolutely for overlays, notifications, etc.
        - No fallback content = invisible when unused
      -->
      <div class="floating-content">
        <ng-content select="[slot=floating]"></ng-content>
      </div>
    </div>
  `,
  styles: [
    `
      .advanced-card {
        position: relative;
        border: 1px solid #e0e0e0;
        border-radius: 12px;
        overflow: hidden;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        background: white;
        margin: 20px 0;
      }

      .card-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 20px;
      }

      .header-actions {
        display: flex;
        gap: 8px;
      }

      .card-content {
        flex: 1;
        padding: 24px;
        min-height: 200px;
      }

      .card-sidebar {
        background: #f8f9fa;
        border-left: 1px solid #e0e0e0;
        padding: 20px;
        min-width: 200px;
      }

      .card-footer {
        display: flex;
        justify-content: space-between;
        align-items: center;
        background: #f8f9fa;
        border-top: 1px solid #e0e0e0;
        padding: 16px 20px;
      }

      .footer-left,
      .footer-center,
      .footer-right {
        display: flex;
        gap: 8px;
      }

      .floating-content {
        position: absolute;
        top: 16px;
        right: 16px;
        z-index: 10;
      }

      .default-action {
        background: #007bff;
        color: white;
        padding: 8px 16px;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-size: 14px;
      }

      /* Responsive layout */
      @media (min-width: 768px) {
        .advanced-card {
          display: grid;
          grid-template-areas:
            "header header"
            "content sidebar"
            "footer footer";
          grid-template-columns: 1fr 200px;
          grid-template-rows: auto 1fr auto;
        }

        .card-header {
          grid-area: header;
        }
        .card-content {
          grid-area: content;
        }
        .card-sidebar {
          grid-area: sidebar;
        }
        .card-footer {
          grid-area: footer;
        }
      }
    `,
  ],
})
// Component class with reactive properties
export class MultiSlotCardComponent {
  @Input() hasSidebar = false; // 🎛️ Control sidebar visibility declaratively

  constructor() {
    // 🚀 Component lifecycle - log initialization for debugging
    console.log("🎨 Multi-slot card component initialized");
  }
}

// Complex Usage Example with Full Slot Utilization
@Component({
  selector: "app-multi-slot-demo", // 🏷️ Component selector for template usage
  standalone: true, // ✨ Self-contained component (Angular 20 feature)
  imports: [MultiSlotCardComponent, CommonModule], // 📦 Import dependencies
  template: `
    <div class="demo-container">
      <!-- 📋 Demo section title -->
      <h2>Multi-Slot Content Projection Demo</h2>

      <!--
        🎯 COMPLETE MULTI-SLOT PROJECTION EXAMPLE
        - Demonstrates all available slots in action
        - Shows how different content types work with named slots
        - [hasSidebar]="true" enables conditional sidebar rendering
      -->
      <app-multi-slot-card [hasSidebar]="true">

        <!--
          🎯 HEADER SLOT CONTENT
          - Uses slot="header" attribute to target header ng-content
          - Can contain any HTML elements or components
        -->
        <div slot="header">
          <h2>🎯 Project Dashboard</h2>
          <p>Real-time project insights</p>
        </div>

        <!--
          🎮 HEADER ACTIONS SLOT
          - Targets header-actions named slot
          - Perfect for toolbar buttons and controls
        -->
        <div slot="header-actions">
          <button class="icon-btn" title="Refresh">🔄</button>
          <button class="icon-btn" title="Settings">⚙️</button>
          <button class="icon-btn" title="Export">📊</button>
        </div>

        <!--
          📋 MAIN CONTENT SLOT
          - Primary content area with complex nested components
          - Demonstrates rich content projection capabilities
        -->
        <div slot="content">
          <div class="dashboard-content">

            <!-- 📊 Metrics grid with data binding -->
            <div class="metrics-grid">
              <div class="metric-card">
                <h4>Total Tasks</h4>
                <!-- 🔗 Data binding from component property -->
                <span class="metric-value">{{ totalTasks }}</span>
              </div>

              <div class="metric-card">
                <h4>Completed</h4>
                <!-- 🎨 CSS class binding based on status -->
                <span class="metric-value completed">{{ completedTasks }}</span>
              </div>

              <div class="metric-card">
                <h4>In Progress</h4>
                <span class="metric-value progress">{{ inProgressTasks }}</span>
              </div>

              <div class="metric-card">
                <h4>Overdue</h4>
                <span class="metric-value overdue">{{ overdueTasks }}</span>
              </div>
            </div>

            <!-- 📈 Chart container with template reference -->
            <div class="chart-container">
              <!-- 🎯 Template reference variable for canvas manipulation -->
              <canvas #chartCanvas width="400" height="200"></canvas>
            </div>

            <!-- 📋 Activity list with structural directive -->
            <div class="recent-activity">
              <h4>Recent Activity</h4>
              <ul class="activity-list">
                <!--
                  🔄 *ngFor structural directive for list rendering
                  - Iterates over recentActivities array
                  - Creates li element for each activity item
                -->
                <li
                  *ngFor="let activity of recentActivities"
                  class="activity-item"
                >
                  <!-- 📍 Property binding for dynamic content -->
                  <span class="activity-icon">{{ activity.icon }}</span>
                  <span class="activity-text">{{ activity.text }}</span>
                  <span class="activity-time">{{ activity.time }}</span>
                </li>
              </ul>
            </div>
          </div>
        </div>

        <!--
          📌 SIDEBAR SLOT CONTENT
          - Only renders when [hasSidebar]="true" due to *ngIf directive
          - Contains independent functionality and components
        -->
        <div slot="sidebar">
          <div class="sidebar-content">
            <h4>Quick Actions</h4>

            <!-- 🎮 Action buttons with event binding -->
            <div class="quick-actions">
              <!-- 🖱️ Event binding (click) to component methods -->
              <button class="quick-action" (click)="addTask()">
                ➕ Add Task
              </button>
              <button class="quick-action" (click)="viewReports()">
                📊 View Reports
              </button>
              <button class="quick-action" (click)="manageTeam()">
                👥 Manage Team
              </button>
            </div>

            <h4>Team Online</h4>
            <!-- 👥 Team members list with dynamic styling -->
            <div class="team-status">
              <!-- 🔄 Loop through team members array -->
              <div *ngFor="let member of teamMembers" class="team-member">
                <!--
                  🎨 Dynamic style binding for avatar colors
                  - [style.background-color] binds to member.color property
                -->
                <div class="avatar" [style.background-color]="member.color">
                  {{ member.initials }}
                </div>
                <span class="member-name">{{ member.name }}</span>
                <!--
                  🎯 Dynamic CSS class binding
                  - [class] binds member.status as CSS class
                -->
                <span class="status" [class]="member.status">{{
                  member.status
                }}</span>
              </div>
            </div>
          </div>
        </div>

        <!--
          🦶 FOOTER SLOTS - Multiple sections for layout control
          Each footer slot has specific purpose and positioning
        -->

        <!-- 📍 Left footer section for metadata -->
        <div slot="footer-left">
          <!-- 📅 Pipe usage for date formatting -->
          <span class="footer-info"
            >Last updated: {{ lastUpdated | date : "short" }}</span
          >
        </div>

        <!-- 📍 Center footer for progress indicators -->
        <div slot="footer-center">
          <div class="progress-indicator">
            <span>Project Progress:</span>
            <!--
              📊 Progress bar with dynamic width binding
              - [style.width.%] binds percentage value with unit
            -->
            <div class="progress-bar">
              <div
                class="progress-fill"
                [style.width.%]="projectProgress"
              ></div>
            </div>
            <span>{{ projectProgress }}%</span>
          </div>
        </div>

        <!-- 📍 Right footer for primary actions -->
        <div slot="footer-right">
          <!-- 🎮 Action buttons with event handlers -->
          <button class="action-btn primary" (click)="saveProject()">
            💾 Save Project
          </button>
          <button class="action-btn secondary" (click)="shareProject()">
            🔗 Share
          </button>
        </div>

        <!--
          🎈 FLOATING CONTENT SLOT
          - Positioned absolutely for overlays
          - *ngIf conditional rendering for notifications
        -->
        <div slot="floating">
          <div *ngIf="hasNotifications" class="notification-badge">
            {{ notificationCount }}
          </div>
        </div>
      </app-multi-slot-card>

      <!--
        🎯 MINIMAL USAGE EXAMPLE
        - Demonstrates partial slot usage
        - Unused slots fall back to default content
        - Shows flexibility of multi-slot system
      -->
      <app-multi-slot-card>
        <h3 slot="header">Simple Card</h3>
        <p slot="content">
          This card uses only basic slots, demonstrating the flexibility of
          multi-slot projection. Unused slots fall back to defaults.
        </p>
        <!-- 📅 Footer with pipe transformation -->
        <span slot="footer-left">Created: {{ creationDate | date }}</span>
      </app-multi-slot-card>
    </div>
  `,
      </app-multi-slot-card>
    </div>
  `,
  styles: [
    `
      .demo-container {
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
      }

      .icon-btn {
        background: rgba(255, 255, 255, 0.2);
        border: 1px solid rgba(255, 255, 255, 0.3);
        color: white;
        padding: 8px;
        border-radius: 6px;
        cursor: pointer;
        font-size: 16px;
        transition: background 0.2s;
      }

  styles: [`
    /* 🎨 DEMO CONTAINER LAYOUT */
    .demo-container {
      max-width: 1200px; /* 📏 Constrain maximum width for readability */
      margin: 0 auto; /* 📍 Center container horizontally */
      padding: 20px; /* 📦 External padding for breathing room */
    }

    /* 🎮 ICON BUTTON STYLES IN HEADER ACTIONS */
    .icon-btn {
      background: rgba(255, 255, 255, 0.2); /* 🎨 Semi-transparent white */
      color: white; /* 📝 White icon/text for contrast */
      border: none; /* 🚫 Remove default button styling */
      padding: 8px; /* 📦 Square padding for icon buttons */
      border-radius: 4px; /* ⭕ Subtle rounded corners */
      cursor: pointer; /* 👆 Pointer cursor for interaction */
      font-size: 14px; /* 📏 Consistent icon size */
      transition: background 0.2s ease; /* ✨ Smooth hover animation */
    }

    /* 🎮 ICON BUTTON HOVER STATE */
    .icon-btn:hover {
      background: rgba(255, 255, 255, 0.3); /* 🔆 Brighter on hover */
    }

    /* 📋 DASHBOARD CONTENT CONTAINER */
    .dashboard-content {
      display: flex; /* 🔄 Flexbox layout for sections */
      flex-direction: column; /* ⬇️ Vertical stacking of dashboard sections */
      gap: 24px; /* 📏 Consistent spacing between sections */
    }

    /* 📊 METRICS GRID LAYOUT */
    .metrics-grid {
      display: grid; /* 📐 CSS Grid for responsive metric cards */
      /* 📏 Auto-fit: creates responsive columns, min 150px, max equal width */
      grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 16px; /* 📏 Space between metric cards */
    }

    /* 📊 INDIVIDUAL METRIC CARD STYLING */
    .metric-card {
      background: #f8f9fa; /* 🎨 Light gray background for metrics */
      padding: 16px; /* 📦 Internal padding for card content */
      border-radius: 8px; /* ⭕ Rounded corners for modern look */
      text-align: center; /* 📍 Center-align text and numbers */
      border: 1px solid #e9ecef; /* 📏 Subtle border for definition */
    }

    /* 📊 METRIC CARD TITLE STYLING */
    .metric-card h4 {
      margin: 0 0 8px 0; /* 📦 Remove default margins, add bottom spacing */
      color: #6c757d; /* 🎨 Muted gray for labels */
      font-size: 12px; /* 📏 Small font for labels */
      text-transform: uppercase; /* 🔤 Uppercase for emphasis */
      font-weight: 600; /* 💪 Semi-bold for readability */
    }

    /* 📊 METRIC VALUE DISPLAY */
    .metric-value {
      font-size: 28px; /* 📏 Large font for emphasis on numbers */
      font-weight: bold; /* 💪 Bold for prominence */
      color: #343a40; /* 🎨 Dark gray for high contrast */
    }

    /* 📊 STATUS-SPECIFIC METRIC COLORS */
    .metric-value.completed {
      color: #28a745; /* 🟢 Green for completed tasks */
    }
    .metric-value.progress {
      color: #ffc107; /* 🟡 Yellow/orange for in-progress */
    }
    .metric-value.overdue {
      color: #dc3545; /* 🔴 Red for overdue/urgent items */
    }

    /* 📈 CHART CONTAINER STYLING */
    .chart-container {
      background: #f8f9fa; /* 🎨 Light background for chart area */
      border-radius: 8px; /* ⭕ Consistent rounded corners */
      padding: 20px; /* 📦 Padding around chart content */
      text-align: center; /* 📍 Center-align chart elements */
    }

    /* 📋 ACTIVITY LIST RESET */
    .activity-list {
      list-style: none; /* 🚫 Remove default bullet points */
      padding: 0; /* 📦 Remove default padding */
      margin: 0; /* 📦 Remove default margin */
    }

    /* 📋 INDIVIDUAL ACTIVITY ITEM LAYOUT */
    .activity-item {
      display: flex; /* 🔄 Horizontal layout for activity components */
      align-items: center; /* ⚖️ Vertically center content */
      gap: 12px; /* 📏 Space between icon, text, and time */
      padding: 8px 0; /* 📦 Vertical padding for touch targets */
      /* 📏 Bottom border separator between items */
      border-bottom: 1px solid #e9ecef;
    }

    /* 📋 ACTIVITY ICON STYLING */
    .activity-icon {
      font-size: 18px; /* 📏 Consistent icon size */
    }

    /* 📋 ACTIVITY TEXT CONTENT */
    .activity-text {
      flex: 1; /* 📏 Take up remaining space */
      font-size: 14px; /* 📏 Readable text size */
    }

    /* 📋 ACTIVITY TIMESTAMP */
    .activity-time {
      font-size: 12px; /* 📏 Smaller font for timestamps */
      color: #6c757d; /* 🎨 Muted color for secondary information */
    }

    /* 📌 SIDEBAR CONTENT HEADING */
    .sidebar-content h4 {
      margin: 0 0 16px 0; /* 📦 Bottom margin for section separation */
      color: #495057; /* 🎨 Slightly darker gray for headings */
      font-size: 16px; /* 📏 Larger than body text for hierarchy */
    }

    /* 🎮 QUICK ACTIONS CONTAINER */
    .quick-actions {
      display: flex; /* 🔄 Flex container for action buttons */
      flex-direction: column; /* ⬇️ Stack buttons vertically */
      gap: 8px; /* 📏 Space between action buttons */
      margin-bottom: 24px; /* 📦 Space before next sidebar section */
    }

    /* 🎮 INDIVIDUAL QUICK ACTION BUTTON */
    .quick-action {
      background: #e9ecef; /* 🎨 Light gray background */
      border: none; /* 🚫 Remove default button border */
      padding: 10px 12px; /* 📦 Button internal spacing */
      border-radius: 6px; /* ⭕ Rounded button corners */
      cursor: pointer; /* 👆 Pointer cursor for interaction */
      font-size: 14px; /* 📏 Readable button text size */
      text-align: left; /* 📍 Left-align button text with icons */
      transition: background 0.2s ease; /* ✨ Smooth hover animation */
    }

    /* 🎮 QUICK ACTION HOVER STATE */
    .quick-action:hover {
      background: #dee2e6; /* 🔆 Slightly darker gray on hover */
    }

    /* 👥 TEAM STATUS CONTAINER */
    .team-status {
      display: flex; /* 🔄 Flex layout for team members */
      flex-direction: column; /* ⬇️ Stack team members vertically */
      gap: 12px; /* 📏 Space between team member items */
    }

    /* 👥 INDIVIDUAL TEAM MEMBER LAYOUT */
    .team-member {
      display: flex; /* 🔄 Horizontal layout for member info */
      align-items: center; /* ⚖️ Vertically center avatar and text */
      gap: 8px; /* 📏 Space between avatar, name, and status */
    }

    /* 👤 TEAM MEMBER AVATAR STYLING */
    .avatar {
      width: 32px; /* 📏 Fixed avatar width */
      height: 32px; /* 📏 Fixed avatar height */
      border-radius: 50%; /* ⭕ Circular avatar shape */
      /* 📍 Center avatar initials */
      display: flex;
      align-items: center;
      justify-content: center;
      color: white; /* 📝 White text for contrast on colored background */
      font-weight: bold; /* 💪 Bold initials for readability */
      font-size: 12px; /* 📏 Small font to fit in circle */
    }

    /* 👤 TEAM MEMBER NAME */
    .member-name {
      flex: 1; /* 📏 Take up available space */
      font-size: 14px; /* 📏 Readable name size */
      font-weight: 500; /* 💪 Medium weight for names */
    }

    /* 🔆 TEAM MEMBER STATUS INDICATOR */
    .status {
      font-size: 10px; /* 📏 Small status text */
      padding: 2px 6px; /* 📦 Tight padding for status badge */
      border-radius: 10px; /* ⭕ Pill-shaped status indicator */
      font-weight: 600; /* 💪 Bold status text */
      text-transform: uppercase; /* 🔤 Uppercase for emphasis */
    }

    /* 🟢 ONLINE STATUS STYLING */
    .status.online {
      background: #d4edda; /* 🎨 Light green background */
      color: #155724; /* 🎨 Dark green text */
    }

    /* 🟡 AWAY STATUS STYLING */
    .status.away {
      background: #fff3cd; /* 🎨 Light yellow background */
      color: #856404; /* 🎨 Dark yellow text */
    }

    /* 🔴 OFFLINE STATUS STYLING */
    .status.offline {
      background: #f8d7da; /* 🎨 Light red background */
      color: #721c24; /* 🎨 Dark red text */
    }

    /* 📊 PROGRESS INDICATOR CONTAINER */
    .progress-indicator {
      display: flex; /* 🔄 Horizontal layout for progress elements */
      align-items: center; /* ⚖️ Vertically center progress components */
      gap: 8px; /* 📏 Space between label, bar, and percentage */
      font-size: 12px; /* 📏 Small font for progress text */
    }

    /* 📊 PROGRESS BAR BACKGROUND */
    .progress-bar {
      background: #e9ecef; /* 🎨 Light gray background for empty progress */
      height: 8px; /* 📏 Thin progress bar */
      width: 100px; /* 📏 Fixed width for consistent layout */
      border-radius: 4px; /* ⭕ Rounded progress bar ends */
      overflow: hidden; /* 🚫 Hide progress fill overflow */
    }

    /* 📊 PROGRESS BAR FILL */
    .progress-fill {
      background: linear-gradient(90deg, #28a745, #20c997); /* 🌈 Green gradient fill */
      height: 100%; /* 📏 Full height of progress bar */
      transition: width 0.3s ease; /* ✨ Smooth width animation */
      border-radius: 4px; /* ⭕ Maintain rounded corners */
    }

    /* 🎮 ACTION BUTTON BASE STYLES */
    .action-btn {
      border: none; /* 🚫 Remove default button border */
      padding: 8px 16px; /* 📦 Button internal spacing */
      border-radius: 6px; /* ⭕ Rounded button corners */
      cursor: pointer; /* 👆 Pointer cursor for interaction */
      font-weight: 500; /* 💪 Medium weight for button text */
      font-size: 14px; /* 📏 Readable button text size */
      transition: all 0.2s ease; /* ✨ Smooth hover animations */
    }

    /* 🎮 PRIMARY ACTION BUTTON */
    .action-btn.primary {
      background: #007bff; /* 🔵 Blue background for primary actions */
      color: white; /* 📝 White text for contrast */
    }

    /* 🎮 PRIMARY BUTTON HOVER STATE */
    .action-btn.primary:hover {
      background: #0056b3; /* 🔵 Darker blue on hover */
    }

    /* 🎮 SECONDARY ACTION BUTTON */
    .action-btn.secondary {
      background: #6c757d; /* 🎨 Gray background for secondary actions */
      color: white; /* 📝 White text for contrast */
    }

    /* 🎮 SECONDARY BUTTON HOVER STATE */
    .action-btn.secondary:hover {
      background: #545b62; /* 🎨 Darker gray on hover */
    }

    /* 🔔 NOTIFICATION BADGE STYLING */
    .notification-badge {
      background: #dc3545; /* 🔴 Red background for urgency */
      color: white; /* 📝 White text for contrast */
      border-radius: 50%; /* ⭕ Circular notification badge */
      width: 24px; /* 📏 Fixed badge width */
      height: 24px; /* 📏 Fixed badge height */
      /* 📍 Center notification count */
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 12px; /* 📏 Small font for count number */
      font-weight: bold; /* 💪 Bold for emphasis */
      /* 💫 Subtle shadow for depth */
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    }

    /* 📄 FOOTER INFORMATION TEXT */
    .footer-info {
      font-size: 12px; /* 📏 Small font for footer metadata */
      color: #6c757d; /* 🎨 Muted color for secondary information */
    }
  `],
        cursor: pointer;
        text-align: left;
        font-size: 14px;
        transition: background 0.2s;
      }

      .quick-action:hover {
        background: #dee2e6;
      }

      .team-member {
        display: flex;
        align-items: center;
        gap: 8px;
        margin-bottom: 8px;
      }

      .avatar {
        width: 32px;
        height: 32px;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        color: white;
        font-weight: bold;
        font-size: 12px;
      }

      .member-name {
        flex: 1;
        font-size: 14px;
      }

      .status {
        font-size: 12px;
        padding: 2px 6px;
        border-radius: 4px;
        text-transform: capitalize;
      }

      .status.online {
        background: #d4edda;
        color: #155724;
      }

      .status.away {
        background: #fff3cd;
        color: #856404;
      }

      .status.offline {
        background: #f8d7da;
        color: #721c24;
      }

      .footer-info {
        font-size: 12px;
        color: #6c757d;
      }

      .progress-indicator {
        display: flex;
        align-items: center;
        gap: 8px;
        font-size: 14px;
      }

      .progress-bar {
        width: 100px;
        height: 8px;
        background: #e9ecef;
        border-radius: 4px;
        overflow: hidden;
      }

      .progress-fill {
        height: 100%;
        background: linear-gradient(90deg, #28a745, #20c997);
        transition: width 0.3s ease;
      }

      .action-btn {
        padding: 8px 16px;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-size: 14px;
        font-weight: 500;
        transition: all 0.2s;
      }

      .action-btn.primary {
        background: #007bff;
        color: white;
      }

      .action-btn.primary:hover {
        background: #0056b3;
      }

      .action-btn.secondary {
        background: #6c757d;
        color: white;
      }

      .action-btn.secondary:hover {
        background: #545b62;
      }

      .notification-badge {
        background: #dc3545;
        color: white;
        width: 24px;
        height: 24px;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        font-weight: bold;
      }
    `,
  ],
})
export class MultiSlotDemoComponent {
  // 📊 DASHBOARD METRICS - Core data properties for project tracking
  totalTasks = 156; // 🔢 Total number of tasks in the project
  completedTasks = 89; // ✅ Number of successfully completed tasks
  inProgressTasks = 45; // 🔄 Tasks currently being worked on
  overdueTasks = 22; // ⚠️ Tasks that have exceeded their deadline
  projectProgress = 67; // 📈 Overall project completion percentage (0-100)

  // 📅 TIMESTAMP PROPERTIES - For tracking and display
  lastUpdated = new Date(); // 🕒 When the project was last modified
  creationDate = new Date("2024-01-15"); // 📅 Initial project creation date

  // 🔔 NOTIFICATION STATE - User alert management
  hasNotifications = true; // 🔔 Boolean flag to show/hide notification badge
  notificationCount = 3; // 🔢 Number of unread notifications to display

  // 📋 RECENT ACTIVITIES - Activity feed data structure
  recentActivities = [
    {
      icon: "✅", // 📍 Emoji icon representing the activity type
      text: 'Task "Update documentation" completed', // 📝 Human-readable activity description
      time: "2 min ago", // 🕒 Relative timestamp for user-friendly display
    },
    {
      icon: "👥",
      text: "John joined the project",
      time: "15 min ago"
    },
    {
      icon: "📝",
      text: 'New comment on "Fix login bug"',
      time: "1 hour ago"
    },
    {
      icon: "🔄",
      text: "Deployment to staging completed",
      time: "2 hours ago",
    },
  ];

  // 👥 TEAM MEMBERS - Array of team member objects with visual and status data
  teamMembers = [
    {
      name: "Alice Johnson", // 📝 Full name for display
      initials: "AJ", // 🔤 Two-letter initials for avatar
      color: "#e74c3c", // 🎨 Hex color for avatar background (red)
      status: "online", // 🟢 Current availability status
    },
    {
      name: "Bob Smith",
      initials: "BS",
      color: "#3498db", // 🔵 Blue avatar background
      status: "away" // 🟡 Away status indicator
    },
    {
      name: "Carol Davis",
      initials: "CD",
      color: "#2ecc71", // 🟢 Green avatar background
      status: "online"
    },
    {
      name: "David Wilson",
      initials: "DW",
      color: "#f39c12", // 🟠 Orange avatar background
      status: "offline", // 🔴 Offline status indicator
    },
  ];

  constructor() {
    // 🚀 Component initialization - Log component startup for debugging
    console.log("🎯 Multi-slot demo initialized");
  }

  // 🎮 ACTION METHODS - Event handlers for user interactions

  /**
   * ➕ Add Task Action Handler
   * Increments total task count when user adds a new task
   * In production: would open task creation form/modal
   */
  addTask(): void {
    console.log("➕ Adding new task");
    this.totalTasks++; // 🔢 Increment total count for immediate UI feedback
    // 💡 Production: this.taskService.createTask() or open modal
  }

  /**
   * 📊 View Reports Action Handler
   * Opens the reports/analytics view for the project
   * In production: would navigate to reports page or dashboard
   */
  viewReports(): void {
    console.log("📊 Opening reports view");
    // 💡 Production: this.router.navigate(['/reports']) or modal
  }

  /**
   * 👥 Manage Team Action Handler
   * Opens team management interface for adding/removing members
   * In production: would navigate to team settings page
   */
  manageTeam(): void {
    console.log("👥 Opening team management");
    // 💡 Production: this.router.navigate(['/team']) or modal
  }

  /**
   * 💾 Save Project Action Handler
   * Persists current project state and updates timestamp
   * Updates lastUpdated for immediate UI feedback
   */
  saveProject(): void {
    console.log("💾 Saving project");
    this.lastUpdated = new Date(); // 🕒 Update timestamp for user feedback
    // 💡 Production: this.projectService.save(this.projectData)
  }

  /**
   * 🔗 Share Project Action Handler
   * Opens sharing interface for project collaboration
   * In production: would generate shareable links or send invites
   */
  shareProject(): void {
    console.log("🔗 Sharing project");
    // 💡 Production: this.shareService.generateLink() or modal
  }
}
```

### 🔸 Conditional Content Projection {#conditional-projection}

Advanced projection techniques using `ng-template`, conditional projection, and dynamic content.

```typescript
// Conditional Projection Component
@Component({
  selector: "app-conditional-card",
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="conditional-card" [class]="cardVariant">
      <!-- 🎯 Conditional header projection -->
      <div class="card-header" *ngIf="showHeader">
        <ng-content select="[slot=header]">
          <!-- Default header template -->
          <ng-container
            *ngTemplateOutlet="defaultHeaderTemplate"
          ></ng-container>
        </ng-content>
      </div>

      <!-- 🎯 Main content with conditional sidebar -->
      <div class="card-main" [class.has-sidebar]="showSidebar">
        <div class="card-content">
          <ng-content select="[slot=content]">
            <ng-container *ngTemplateOutlet="emptyStateTemplate"></ng-container>
          </ng-content>
        </div>

        <!-- Conditional sidebar -->
        <div class="card-sidebar" *ngIf="showSidebar">
          <ng-content select="[slot=sidebar]">
            <ng-container
              *ngTemplateOutlet="defaultSidebarTemplate"
            ></ng-container>
          </ng-content>
        </div>
      </div>

      <!-- 🎯 Dynamic footer based on state -->
      <div class="card-footer" *ngIf="showFooter">
        <ng-container [ngSwitch]="footerType">
          <!-- Actions footer -->
          <div *ngSwitchCase="'actions'" class="footer-actions">
            <ng-content select="[slot=footer-actions]">
              <button class="btn btn-primary" (click)="onDefaultAction()">
                Default Action
              </button>
            </ng-content>
          </div>

          <!-- Info footer -->
          <div *ngSwitchCase="'info'" class="footer-info">
            <ng-content select="[slot=footer-info]">
              <span>No additional information provided</span>
            </ng-content>
          </div>

          <!-- Navigation footer -->
          <div *ngSwitchCase="'navigation'" class="footer-navigation">
            <ng-content select="[slot=footer-nav]">
              <div class="nav-buttons">
                <button class="btn btn-secondary" (click)="onPrevious()">
                  ← Previous
                </button>
                <button class="btn btn-secondary" (click)="onNext()">
                  Next →
                </button>
              </div>
            </ng-content>
          </div>

          <!-- Default footer -->
          <div *ngSwitchDefault class="footer-default">
            <ng-content select="[slot=footer]">
              <ng-container
                *ngTemplateOutlet="defaultFooterTemplate"
              ></ng-container>
            </ng-content>
          </div>
        </ng-container>
      </div>

      <!-- 🎯 Overlay content (conditionally shown) -->
      <div class="card-overlay" *ngIf="showOverlay" @fadeInOut>
        <ng-content select="[slot=overlay]">
          <ng-container *ngTemplateOutlet="loadingTemplate"></ng-container>
        </ng-content>
      </div>
    </div>

    <!-- 🎯 Default templates -->
    <ng-template #defaultHeaderTemplate>
      <h3>{{ defaultTitle }}</h3>
      <p>{{ defaultSubtitle }}</p>
    </ng-template>

    <ng-template #emptyStateTemplate>
      <div class="empty-state">
        <div class="empty-icon">📭</div>
        <h4>No Content Available</h4>
        <p>Please add content to this card or check back later.</p>
        <button class="btn btn-primary" (click)="onAddContent()">
          Add Content
        </button>
      </div>
    </ng-template>

    <ng-template #defaultSidebarTemplate>
      <div class="default-sidebar">
        <h4>Quick Links</h4>
        <ul>
          <li>
            <a href="#" (click)="onQuickLink('docs')">📚 Documentation</a>
          </li>
          <li><a href="#" (click)="onQuickLink('help')">❓ Help</a></li>
          <li><a href="#" (click)="onQuickLink('settings')">⚙️ Settings</a></li>
        </ul>
      </div>
    </ng-template>

    <ng-template #defaultFooterTemplate>
      <div class="default-footer">
        <span>Created: {{ createdDate | date : "mediumDate" }}</span>
        <span>Modified: {{ modifiedDate | date : "mediumDate" }}</span>
      </div>
    </ng-template>

    <ng-template #loadingTemplate>
      <div class="loading-content">
        <div class="spinner"></div>
        <p>{{ loadingMessage }}</p>
      </div>
    </ng-template>
  `,
  styles: [
    `
      .conditional-card {
        position: relative;
        border: 1px solid #e0e0e0;
        border-radius: 8px;
        background: white;
        overflow: hidden;
        margin: 16px 0;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        transition: all 0.3s ease;
      }

      .conditional-card.primary {
        border-color: #007bff;
      }

      .conditional-card.success {
        border-color: #28a745;
      }

      .conditional-card.warning {
        border-color: #ffc107;
      }

      .conditional-card.danger {
        border-color: #dc3545;
      }

      .card-header {
        background: linear-gradient(135deg, #f8f9fa, #e9ecef);
        padding: 20px;
        border-bottom: 1px solid #e0e0e0;
      }

      .card-main {
        display: flex;
        min-height: 200px;
      }

      .card-main.has-sidebar {
        display: grid;
        grid-template-columns: 1fr 250px;
      }

      .card-content {
        padding: 24px;
        flex: 1;
      }

      .card-sidebar {
        background: #f8f9fa;
        border-left: 1px solid #e0e0e0;
        padding: 20px;
      }

      .card-footer {
        border-top: 1px solid #e0e0e0;
        background: #f8f9fa;
      }

      .footer-actions,
      .footer-info,
      .footer-navigation,
      .footer-default {
        padding: 16px 20px;
      }

      .footer-actions {
        display: flex;
        justify-content: flex-end;
        gap: 8px;
      }

      .footer-info {
        color: #6c757d;
        font-size: 14px;
      }

      .footer-navigation {
        display: flex;
        justify-content: space-between;
        align-items: center;
      }

      .nav-buttons {
        display: flex;
        gap: 8px;
      }

      .footer-default {
        display: flex;
        justify-content: space-between;
        font-size: 12px;
        color: #6c757d;
      }

      .card-overlay {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: rgba(255, 255, 255, 0.95);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 10;
      }

      .empty-state {
        text-align: center;
        color: #6c757d;
        padding: 40px;
      }

      .empty-icon {
        font-size: 48px;
        margin-bottom: 16px;
      }

      .empty-state h4 {
        color: #495057;
        margin: 16px 0 8px 0;
      }

      .empty-state p {
        margin-bottom: 24px;
      }

      .default-sidebar h4 {
        margin: 0 0 16px 0;
        color: #495057;
      }

      .default-sidebar ul {
        list-style: none;
        padding: 0;
        margin: 0;
      }

      .default-sidebar li {
        margin-bottom: 8px;
      }

      .default-sidebar a {
        color: #007bff;
        text-decoration: none;
        font-size: 14px;
        display: flex;
        align-items: center;
        gap: 8px;
        padding: 4px 0;
      }

      .default-sidebar a:hover {
        text-decoration: underline;
      }

      .loading-content {
        text-align: center;
        color: #6c757d;
      }

      .spinner {
        width: 40px;
        height: 40px;
        border: 4px solid #e9ecef;
        border-left: 4px solid #007bff;
        border-radius: 50%;
        animation: spin 1s linear infinite;
        margin: 0 auto 16px;
      }

      .btn {
        padding: 8px 16px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
        font-weight: 500;
        transition: all 0.2s;
        text-decoration: none;
        display: inline-block;
      }

      .btn-primary {
        background: #007bff;
        color: white;
      }

      .btn-primary:hover {
        background: #0056b3;
      }

      .btn-secondary {
        background: #6c757d;
        color: white;
      }

      .btn-secondary:hover {
        background: #545b62;
      }

      @keyframes spin {
        to {
          transform: rotate(360deg);
        }
      }
    `,
  ],
  animations: [
    trigger("fadeInOut", [
      transition(":enter", [
        style({ opacity: 0 }),
        animate("300ms ease-in", style({ opacity: 1 })),
      ]),
      transition(":leave", [animate("300ms ease-out", style({ opacity: 0 }))]),
    ]),
  ],
})
export class ConditionalCardComponent {
  @Input() showHeader = true;
  @Input() showSidebar = false;
  @Input() showFooter = true;
  @Input() showOverlay = false;
  @Input() footerType: "actions" | "info" | "navigation" | "default" =
    "default";
  @Input() cardVariant: "primary" | "success" | "warning" | "danger" | "" = "";
  @Input() defaultTitle = "Default Card Title";
  @Input() defaultSubtitle = "This is a default subtitle";
  @Input() loadingMessage = "Loading content...";

  createdDate = new Date("2024-01-15");
  modifiedDate = new Date();

  onDefaultAction(): void {
    console.log("🎯 Default action triggered");
  }

  onPrevious(): void {
    console.log("← Previous action triggered");
  }

  onNext(): void {
    console.log("→ Next action triggered");
  }

  onAddContent(): void {
    console.log("➕ Add content action triggered");
  }

  onQuickLink(type: string): void {
    console.log(`🔗 Quick link clicked: ${type}`);
  }
}
```

### 🔹 Angular 15 vs 20 Content Projection Differences {#projection-differences}

**Key improvements and new features in Angular 20 content projection:**

| Feature                   | Angular 15                 | Angular 20                              |
| ------------------------- | -------------------------- | --------------------------------------- |
| **Standalone Components** | Optional                   | Default and optimized                   |
| **Signals Integration**   | Not available              | Native support for reactive projections |
| **Performance**           | Manual OnPush optimization | Automatic signal-based optimization     |
| **Template Safety**       | Runtime errors             | Compile-time validation                 |
| **Lazy Projection**       | Manual implementation      | Built-in lazy content loading           |

```typescript
// Angular 15 approach
@Component({
  selector: "app-legacy-projection",
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="legacy-card">
      <ng-content select="[slot=header]"></ng-content>
      <ng-content></ng-content>
      <ng-content select="[slot=footer]"></ng-content>
    </div>
  `,
})
export class LegacyProjectionComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();

  @Input() set data(value: any) {
    // Manual change detection trigger
    this.cdr.markForCheck();
  }

  constructor(private cdr: ChangeDetectorRef) {}

  ngOnInit(): void {
    // Manual subscription management
    someObservable$.pipe(takeUntil(this.destroy$)).subscribe(() => {
      this.cdr.markForCheck();
    });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Angular 20 approach with signals
@Component({
  selector: "app-modern-projection",
  standalone: true,
  template: `
    <div class="modern-card">
      <!-- 🆕 Reactive content projection -->
      <ng-content select="[slot=header]" *ngIf="showHeader()"></ng-content>

      <!-- 🆕 Conditional projection with signals -->
      <ng-content *ngIf="hasContent(); else emptyState"></ng-content>

      <ng-template #emptyState>
        <div class="empty-state">{{ emptyMessage() }}</div>
      </ng-template>

      <!-- 🆕 Lazy-loaded footer -->
      <ng-content
        select="[slot=footer]"
        *ngIf="shouldShowFooter()"
        @lazyLoad
      ></ng-content>
    </div>
  `,
})
export class ModernProjectionComponent {
  // 🆕 Signals for reactive state
  showHeader = signal(true);
  hasContent = signal(false);
  emptyMessage = signal("No content available");

  // 🆕 Computed properties
  shouldShowFooter = computed(() => this.hasContent() && this.showHeader());

  // 🆕 Automatic change detection with signals
  @Input() set data(value: any) {
    this.hasContent.set(!!value);
  }
}

// Advanced Angular 20 projection with lazy loading
@Component({
  selector: "app-lazy-projection",
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="lazy-card">
      <!-- 🆕 Immediate content -->
      <ng-content select="[priority=high]"></ng-content>

      <!-- 🆕 Lazy-loaded content -->
      <div class="lazy-content" *ngIf="shouldLoadLazy()" @fadeIn>
        <ng-content select="[priority=low]"></ng-content>
      </div>

      <!-- 🆕 On-demand content -->
      <div class="on-demand-content">
        <button
          *ngIf="!showOnDemand()"
          (click)="loadOnDemand()"
          class="load-more-btn"
        >
          Load More Content
        </button>

        <ng-content
          *ngIf="showOnDemand()"
          select="[priority=ondemand]"
        ></ng-content>
      </div>
    </div>
  `,
  animations: [
    trigger("fadeIn", [
      transition(":enter", [
        style({ opacity: 0, transform: "translateY(10px)" }),
        animate(
          "300ms ease-out",
          style({ opacity: 1, transform: "translateY(0)" })
        ),
      ]),
    ]),
  ],
})
export class LazyProjectionComponent implements AfterViewInit {
  shouldLoadLazy = signal(false);
  showOnDemand = signal(false);

  ngAfterViewInit(): void {
    // 🆕 Intersection Observer for lazy loading
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            this.shouldLoadLazy.set(true);
            observer.disconnect();
          }
        });
      },
      { threshold: 0.1 }
    );

    // Observe the component for lazy loading trigger
    observer.observe(document.querySelector(".lazy-card")!);
  }

  loadOnDemand(): void {
    this.showOnDemand.set(true);
    console.log("🔄 Loading on-demand content");
  }
}
```

---

## 📐 Custom Directives {#custom-directives}

Custom directives extend HTML elements with additional behavior, styling, and functionality. Angular 20 brings enhanced directive capabilities with better performance and easier composition patterns.

### 🔹 Attribute Directives {#attribute-directives}

Attribute directives modify the appearance or behavior of existing elements without changing the DOM structure.

```typescript
// Advanced Highlight Directive with Animation - Attribute Directive Example
@Directive({
  selector: "[appHighlight]", // 🏷️ Attribute selector - use as <div appHighlight>
  standalone: true, // ✨ Self-contained directive (Angular 20 feature)
})
export class HighlightDirective implements OnInit, OnDestroy {
  // 📥 INPUT PROPERTIES - Configuration from parent component
  @Input() appHighlight: string = "#ffeb3b"; // 🎨 Highlight color (yellow default)
  @Input() highlightDuration: number = 300; // ⏱️ Animation duration in milliseconds
  @Input() highlightIntensity: "light" | "medium" | "strong" = "medium"; // 🔆 Intensity level
  @Input() highlightTrigger: "hover" | "click" | "focus" | "auto" = "hover"; // 🎮 Trigger method

  // 📤 OUTPUT EVENTS - Communicate state changes to parent
  @Output() highlighted = new EventEmitter<{
    color: string; // 🎨 The color that was applied
    element: HTMLElement; // 🎯 Reference to the DOM element
  }>();
  @Output() unhighlighted = new EventEmitter<HTMLElement>(); // 🔄 When highlight is removed

  // 🏠 PRIVATE STATE MANAGEMENT
  private originalBackground?: string; // 💾 Store original background to restore later
  private isHighlighted = false; // 🔍 Track current highlight state
  private animationFrameId?: number; // 🎬 For smooth animations with requestAnimationFrame

  constructor(
    private el: ElementRef<HTMLElement>, // 🎯 Reference to the host DOM element
    private renderer: Renderer2 // 🎨 Safe DOM manipulation service
  ) {}

  ngOnInit(): void {
    console.log("🎨 Highlight directive initialized");
    this.setupInitialState(); // 🚀 Configure initial element state
    this.setupEventListeners(); // 👂 Set up global event listeners
  }

  ngOnDestroy(): void {
    // 🧹 CLEANUP - Prevent memory leaks
    if (this.animationFrameId) {
      cancelAnimationFrame(this.animationFrameId); // ❌ Cancel pending animations
    }
    console.log("🧹 Highlight directive destroyed");
  }

  // 🖱️ HOST LISTENERS - Respond to events on the host element

  @HostListener("mouseenter") // 🖱️ When mouse enters the element
  onMouseEnter(): void {
    if (this.highlightTrigger === "hover") {
      this.highlight(); // ✨ Apply highlight on hover
    }
  }

  @HostListener("mouseleave") // 🖱️ When mouse leaves the element
  onMouseLeave(): void {
    if (this.highlightTrigger === "hover") {
      this.removeHighlight(); // 🔄 Remove highlight when mouse leaves
    }
  }

  @HostListener("click") // 🖱️ When element is clicked
  onClick(): void {
    if (this.highlightTrigger === "click") {
      this.toggleHighlight(); // 🔄 Toggle highlight state on click
    }
  }

  @HostListener("focus") // ⌨️ When element receives keyboard focus
  onFocus(): void {
    if (this.highlightTrigger === "focus") {
      this.highlight(); // ✨ Highlight on focus for accessibility
    }
  }

  @HostListener("blur") // ⌨️ When element loses keyboard focus
  onBlur(): void {
    if (this.highlightTrigger === "focus") {
      this.removeHighlight(); // 🔄 Remove highlight when focus lost
    }
  }

  /**
   * 🚀 INITIAL SETUP METHOD
   * Configures the element's initial state and styling
   */
  private setupInitialState(): void {
    const element = this.el.nativeElement; // 🎯 Get the actual DOM element

    // 💾 Store original background color for restoration later
    this.originalBackground = getComputedStyle(element).backgroundColor;

    // 🎨 Apply CSS transition for smooth color changes
    this.renderer.setStyle(
      element,
      "transition", // 🎬 CSS transition property
      `background-color ${this.highlightDuration}ms ease-in-out` // ⏱️ Smooth transition
    );

    // ♿ ACCESSIBILITY: Make element focusable if focus trigger is used
    if (
      this.highlightTrigger === "focus" &&
      !element.hasAttribute("tabindex") // 🔍 Check if already focusable
    ) {
      this.renderer.setAttribute(element, "tabindex", "0"); // ⌨️ Make focusable
    }

    // 🚀 AUTO-TRIGGER: Apply highlight immediately if auto mode
    if (this.highlightTrigger === "auto") {
      this.highlight();
    }
  }

  /**
   * 👂 GLOBAL EVENT LISTENERS SETUP
   * Sets up listeners for window events that affect the directive
   */
  private setupEventListeners(): void {
    // 📱 RESPONSIVE: Update highlight on window resize
    this.renderer.listen("window", "resize", () => {
      if (this.isHighlighted) {
        this.updateHighlight(); // 🔄 Recalculate highlight on resize
      }
    });
  }

  /**
   * ✨ APPLY HIGHLIGHT METHOD
   * Applies highlight effect to the element
   */
  private highlight(): void {
    if (this.isHighlighted) return; // 🚫 Prevent double-highlighting

    const color = this.getIntensityColor(); // 🎨 Get color based on intensity
    this.applyHighlight(color); // 🖌️ Apply the calculated color

    // 🏷️ UPDATE STATE
    this.isHighlighted = true;

    // 📡 EMIT EVENT - Notify parent component
    this.highlighted.emit({
      color,
      element: this.el.nativeElement,
    });

    console.log("✨ Element highlighted:", color);
  }

  /**
   * 🔄 REMOVE HIGHLIGHT METHOD
   * Restores element to original state
   */
  private removeHighlight(): void {
    if (!this.isHighlighted) return; // 🚫 Nothing to remove

    // 🔄 Restore original background color
    this.applyHighlight(this.originalBackground || "");

    // 🏷️ UPDATE STATE
    this.isHighlighted = false;

    // 📡 EMIT EVENT - Notify parent about removal
    this.unhighlighted.emit(this.el.nativeElement);

    console.log("🔄 Highlight removed");
  }

  /**
   * 🔄 TOGGLE HIGHLIGHT METHOD
   * Switches between highlighted and normal state
   */
  private toggleHighlight(): void {
    if (this.isHighlighted) {
      this.removeHighlight();
    } else {
      this.highlight();
    }
  }

  /**
   * 🖌️ APPLY HIGHLIGHT COLOR METHOD
   * Uses requestAnimationFrame for smooth DOM updates
   */
  private applyHighlight(color: string): void {
    // 🎬 Use requestAnimationFrame for smooth rendering
    this.animationFrameId = requestAnimationFrame(() => {
      this.renderer.setStyle(this.el.nativeElement, "backgroundColor", color);
    });
  }

  /**
   * 🔄 UPDATE EXISTING HIGHLIGHT
   * Recalculates and reapplies highlight (useful for responsive updates)
   */
  private updateHighlight(): void {
    if (this.isHighlighted) {
      const color = this.getIntensityColor(); // 🎨 Recalculate color
      this.applyHighlight(color); // 🖌️ Reapply with new color
    }
  }

  /**
   * 🎨 COLOR INTENSITY CALCULATION
   * Adjusts the base highlight color based on intensity setting
   * @returns {string} Calculated color value
   */
  private getIntensityColor(): string {
    const color = this.appHighlight; // 🎨 Base color from input

    // 🔆 INTENSITY ADJUSTMENT
    switch (this.highlightIntensity) {
      case "light":
        return this.adjustColorBrightness(color, 0.7); // 🔆 Make lighter (70% brighter)
      case "strong":
        return this.adjustColorBrightness(color, -0.2); // 🔅 Make darker (20% darker)
      default:
        return color; // 🎨 Use original color for medium intensity
    }
  }

  /**
   * 🌈 COLOR BRIGHTNESS ADJUSTMENT UTILITY
   * Mathematically adjusts color brightness
   * @param {string} color - Hex color code
   * @param {number} factor - Brightness factor (-1 to 1)
   * @returns {string} Adjusted RGB color string
   */
  private adjustColorBrightness(color: string, factor: number): string {
    // 🔍 CHECK FOR HEX COLOR FORMAT
    if (color.startsWith("#")) {
      const hex = color.replace("#", ""); // 🧹 Remove hash symbol

      // 🔢 CONVERT HEX TO RGB VALUES
      const r = parseInt(hex.substr(0, 2), 16); // 🔴 Red component (0-255)
      const g = parseInt(hex.substr(2, 2), 16); // 🟢 Green component (0-255)
      const b = parseInt(hex.substr(4, 2), 16); // 🔵 Blue component (0-255)

      // 🧮 BRIGHTNESS ADJUSTMENT FUNCTION
      const adjust = (val: number) => {
        // ➕ Positive factor: brighten by adding to max (255)
        // ➖ Negative factor: darken by multiplying
        const adjusted =
          factor > 0 ? val + (255 - val) * factor : val * (1 + factor);
        // 📏 Clamp value between 0 and 255
        return Math.max(0, Math.min(255, Math.round(adjusted)));
      };

      // 🎨 APPLY ADJUSTMENT TO EACH COLOR COMPONENT
      const newR = adjust(r);
      const newG = adjust(g);
      const newB = adjust(b);

      // 🔄 RETURN AS RGB STRING
      return `rgb(${newR}, ${newG}, ${newB})`;
    }

    // 🔄 RETURN ORIGINAL COLOR IF NOT HEX FORMAT
    return color;
  }
}

// Smart Tooltip Directive with Positioning - Advanced Attribute Directive
@Directive({
  selector: "[appTooltip]", // 🏷️ Attribute selector for tooltip functionality
  standalone: true, // ✨ Self-contained directive (Angular 20 feature)
})
export class TooltipDirective implements OnInit, OnDestroy, AfterViewInit {
  // 📥 INPUT PROPERTIES - Tooltip configuration options
  @Input() appTooltip: string = ""; // 📝 Tooltip text content
  @Input() tooltipPosition: "top" | "bottom" | "left" | "right" | "auto" =
    "auto"; // 📍 Position preference
  @Input() tooltipDelay: number = 500; // ⏱️ Show delay in milliseconds
  @Input() tooltipTheme: "dark" | "light" | "info" | "warning" | "error" =
    "dark"; // 🎨 Visual theme
  @Input() tooltipMaxWidth: string = "250px"; // 📏 Maximum tooltip width
  @Input() tooltipOffset: number = 8; // 📐 Distance from host element in pixels
  @Input() tooltipShowOnTouch: boolean = true; // 📱 Show on mobile touch events

  // 🏠 PRIVATE STATE MANAGEMENT
  private tooltipElement?: HTMLElement; // 📋 Reference to tooltip DOM element
  private showTimeout?: number; // ⏱️ Timeout ID for delayed show
  private hideTimeout?: number; // ⏱️ Timeout ID for delayed hide
  private isVisible = false; // 👁️ Track tooltip visibility state
  private resizeObserver?: ResizeObserver; // 📱 Observer for responsive positioning

  constructor(
    private el: ElementRef<HTMLElement>, // 🎯 Host element reference
    private renderer: Renderer2 // 🎨 Safe DOM manipulation service
  ) {}

  ngOnInit(): void {
    console.log("📝 Advanced tooltip directive initialized");
    this.setupAccessibility(); // ♿ Configure accessibility attributes
  }

  ngAfterViewInit(): void {
    this.setupResizeObserver(); // 📱 Set up responsive behavior
  }

  ngOnDestroy(): void {
    // 🧹 CLEANUP - Prevent memory leaks
    this.clearTimeouts(); // ⏱️ Clear any pending timeouts
    this.destroyTooltip(); // 🗑️ Remove tooltip from DOM
    this.resizeObserver?.disconnect(); // 📱 Disconnect resize observer
    console.log("🧹 Tooltip directive destroyed");
  }

  // 🖱️ HOST LISTENERS - Mouse and touch event handlers

  @HostListener("mouseenter") // 🖱️ When mouse enters element
  onMouseEnter(): void {
    this.clearTimeouts(); // ⏱️ Clear any pending hide timeout
    // ⏱️ Show tooltip after delay for better UX
    this.showTimeout = window.setTimeout(() => {
      this.showTooltip();
    }, this.tooltipDelay);
  }

  @HostListener("mouseleave") // 🖱️ When mouse leaves element
  onMouseLeave(): void {
    this.clearTimeouts(); // ⏱️ Clear show timeout if mouse leaves quickly
    this.hideTooltip(); // 🙈 Hide tooltip immediately on mouse leave
  }

  @HostListener("touchstart") // 📱 Touch event for mobile devices
  onTouchStart(): void {
    if (this.tooltipShowOnTouch) {
      this.clearTimeouts();
      if (this.isVisible) {
        this.hideTooltip(); // 🔄 Toggle behavior on touch
      } else {
        this.showTooltip(); // 👁️ Show tooltip on touch
      }
    }
  }

  @HostListener("focus") // ⌨️ Keyboard focus for accessibility
  onFocus(): void {
    this.clearTimeouts();
    // 📝 Show immediately on focus (accessibility requirement)
    this.showTooltip();
  }

  @HostListener("blur") // ⌨️ When element loses focus
  onBlur(): void {
    this.clearTimeouts();
    this.hideTooltip(); // 🙈 Hide when focus is lost
  }

  /**
   * ♿ ACCESSIBILITY SETUP METHOD
   * Configures ARIA attributes for screen readers
   */
  private setupAccessibility(): void {
    const element = this.el.nativeElement;

    // 🏷️ Add ARIA label for screen readers
    this.renderer.setAttribute(element, "aria-label", this.appTooltip);

    // 📝 Indicate that element has a description
    this.renderer.setAttribute(
      element,
      "aria-describedby",
      `tooltip-${this.generateId()}`
    );

    // ⌨️ Make element focusable if it's not already
    if (
      !element.hasAttribute("tabindex") &&
      element.tagName !== "BUTTON" &&
      element.tagName !== "A"
    ) {
      this.renderer.setAttribute(element, "tabindex", "0");
    }
  }

  /**
   * 📱 RESIZE OBSERVER SETUP
   * Handles responsive repositioning when viewport changes
   */
  private setupResizeObserver(): void {
    // 📱 Modern browser feature detection
    if ("ResizeObserver" in window) {
      this.resizeObserver = new ResizeObserver(() => {
        if (this.isVisible && this.tooltipElement) {
          this.updatePosition(); // 📐 Recalculate position on resize
        }
      });

      // 👀 Observe the host element for size changes
      this.resizeObserver.observe(this.el.nativeElement);
    }
  }

  /**
   * 👁️ SHOW TOOLTIP METHOD
   * Creates and displays the tooltip with proper positioning
   */
  private showTooltip(): void {
    if (this.isVisible || !this.appTooltip.trim()) return; // 🚫 Prevent double-show or empty tooltip

    this.createTooltip(); // 🏗️ Create tooltip DOM element
    this.positionTooltip(); // 📐 Calculate and apply position
    this.animateIn(); // 🎬 Smooth entrance animation

    this.isVisible = true;
    console.log("👁️ Tooltip shown:", this.appTooltip);
  }

  /**
   * 🙈 HIDE TOOLTIP METHOD
   * Smoothly hides and removes tooltip from DOM
   */
  private hideTooltip(): void {
    if (!this.isVisible) return; // 🚫 Nothing to hide

    this.animateOut(() => {
      this.destroyTooltip(); // 🗑️ Remove from DOM after animation
    });

    this.isVisible = false;
    console.log("🙈 Tooltip hidden");
  }

  /**
   * 🏗️ CREATE TOOLTIP DOM ELEMENT
   * Builds the tooltip element with styling and content
   */
  private createTooltip(): void {
    // 🏗️ Create tooltip container element
    this.tooltipElement = this.renderer.createElement("div");

    // 🏷️ Add CSS classes for styling
    this.renderer.addClass(this.tooltipElement, "app-tooltip");
    this.renderer.addClass(
      this.tooltipElement,
      `app-tooltip--${this.tooltipTheme}`
    );

    // 🆔 Set unique ID for ARIA references
    this.renderer.setAttribute(
      this.tooltipElement,
      "id",
      `tooltip-${this.generateId()}`
    );

    // ♿ Set ARIA role for screen readers
    this.renderer.setAttribute(this.tooltipElement, "role", "tooltip");

    // 📝 Set tooltip content
    this.renderer.setProperty(
      this.tooltipElement,
      "textContent",
      this.appTooltip
    );

    // 🎨 Apply styling
    this.applyTooltipStyles();

    // 🌐 Add to document body for absolute positioning
    this.renderer.appendChild(document.body, this.tooltipElement);
  }

  /**
   * 🎨 APPLY TOOLTIP STYLING
   * Sets CSS styles for appearance and behavior
   */
  private applyTooltipStyles(): void {
    if (!this.tooltipElement) return;

    const styles = {
      position: "absolute", // 📐 Absolute positioning for precise placement
      zIndex: "9999", // 📚 Ensure tooltip appears above other elements
      maxWidth: this.tooltipMaxWidth, // 📏 Constrain width for readability
      padding: "8px 12px", // 📦 Internal spacing for content
      borderRadius: "4px", // ⭕ Rounded corners for modern look
      fontSize: "14px", // 📏 Readable font size
      lineHeight: "1.4", // 📏 Comfortable line spacing
      wordWrap: "break-word", // 📝 Handle long words gracefully
      pointerEvents: "none", // 🖱️ Don't interfere with mouse events
      opacity: "0", // 🌫️ Start hidden for smooth animation
      transform: "scale(0.8)", // 🎬 Start scaled down for animation
      transition: "all 0.2s ease-in-out", // ✨ Smooth transitions
    };

    // 🎨 Apply theme-specific colors
    const themeStyles = this.getThemeStyles();
    Object.assign(styles, themeStyles);

    // 🖌️ Apply all styles to tooltip element
    Object.entries(styles).forEach(([property, value]) => {
      this.renderer.setStyle(this.tooltipElement, property, value);
    });
  }

  /**
   * 🎨 GET THEME-SPECIFIC STYLES
   * Returns color scheme based on selected theme
   * @returns {object} CSS style properties for the theme
   */
  private getThemeStyles(): { [key: string]: string } {
    switch (this.tooltipTheme) {
      case "light":
        return {
          backgroundColor: "#ffffff", // ⚪ White background
          color: "#333333", // ⚫ Dark text for contrast
          border: "1px solid #e0e0e0", // 📏 Light border for definition
          boxShadow: "0 2px 8px rgba(0, 0, 0, 0.15)", // 💫 Subtle shadow
        };

      case "info":
        return {
          backgroundColor: "#3498db", // 🔵 Blue background
          color: "#ffffff", // ⚪ White text
          boxShadow: "0 2px 8px rgba(52, 152, 219, 0.3)", // 💫 Blue-tinted shadow
        };

      case "warning":
        return {
          backgroundColor: "#f39c12", // 🟠 Orange background
          color: "#ffffff", // ⚪ White text
          boxShadow: "0 2px 8px rgba(243, 156, 18, 0.3)", // 💫 Orange-tinted shadow
        };

      case "error":
        return {
          backgroundColor: "#e74c3c", // 🔴 Red background
          color: "#ffffff", // ⚪ White text
          boxShadow: "0 2px 8px rgba(231, 76, 60, 0.3)", // 💫 Red-tinted shadow
        };

      default: // "dark" theme
        return {
          backgroundColor: "#2c3e50", // ⚫ Dark background
          color: "#ffffff", // ⚪ White text
          boxShadow: "0 2px 8px rgba(0, 0, 0, 0.2)", // 💫 Dark shadow
        };
    }
  }

  /**
   * 🆔 GENERATE UNIQUE ID
   * Creates unique identifier for ARIA references
   * @returns {string} Unique ID string
   */
  private generateId(): string {
    return Math.random().toString(36).substr(2, 9); // 🎲 Random alphanumeric string
  }

  ngOnDestroy(): void {
    this.cleanup();
    console.log("📝 Tooltip directive destroyed");
  }

  @HostListener("mouseenter")
  onMouseEnter(): void {
    this.scheduleShow();
  }

  @HostListener("mouseleave")
  onMouseLeave(): void {
    this.scheduleHide();
  }

  @HostListener("focus")
  onFocus(): void {
    this.scheduleShow();
  }

  @HostListener("blur")
  onBlur(): void {
    this.scheduleHide();
  }

  @HostListener("touchstart")
  onTouchStart(): void {
    if (this.tooltipShowOnTouch) {
      this.scheduleShow();
    }
  }

  @HostListener("touchend")
  onTouchEnd(): void {
    if (this.tooltipShowOnTouch) {
      this.scheduleHide(1000); // Longer delay for touch
    }
  }

  @HostListener("window:scroll")
  @HostListener("window:resize")
  onWindowEvents(): void {
    if (this.isVisible) {
      this.repositionTooltip();
    }
  }

  private setupAccessibility(): void {
    const element = this.el.nativeElement;

    if (
      !element.hasAttribute("tabindex") &&
      !["button", "input", "select", "textarea", "a"].includes(
        element.tagName.toLowerCase()
      )
    ) {
      this.renderer.setAttribute(element, "tabindex", "0");
    }

    this.renderer.setAttribute(
      element,
      "aria-describedby",
      this.getTooltipId()
    );
  }

  private setupResizeObserver(): void {
    if ("ResizeObserver" in window) {
      this.resizeObserver = new ResizeObserver(() => {
        if (this.isVisible) {
          this.repositionTooltip();
        }
      });
      this.resizeObserver.observe(this.el.nativeElement);
    }
  }

  private scheduleShow(): void {
    this.clearTimeouts();
    this.showTimeout = window.setTimeout(() => {
      this.showTooltip();
    }, this.tooltipDelay);
  }

  private scheduleHide(delay = 100): void {
    this.clearTimeouts();
    this.hideTimeout = window.setTimeout(() => {
      this.hideTooltip();
    }, delay);
  }

  private showTooltip(): void {
    if (!this.appTooltip || this.isVisible) return;

    this.createTooltip();
    this.positionTooltip();
    this.animateIn();

    this.isVisible = true;
    console.log("💭 Tooltip shown");
  }

  private hideTooltip(): void {
    if (!this.isVisible) return;

    this.animateOut(() => {
      this.removeTooltip();
    });

    this.isVisible = false;
    console.log("💭 Tooltip hidden");
  }

  private createTooltip(): void {
    this.tooltipElement = this.renderer.createElement("div");

    this.renderer.setAttribute(this.tooltipElement, "id", this.getTooltipId());
    this.renderer.setAttribute(this.tooltipElement, "role", "tooltip");
    this.renderer.setProperty(
      this.tooltipElement,
      "textContent",
      this.appTooltip
    );

    this.applyTooltipStyles();
    this.renderer.appendChild(document.body, this.tooltipElement);
  }

  private applyTooltipStyles(): void {
    if (!this.tooltipElement) return;

    const baseStyles = {
      position: "absolute",
      zIndex: "10000",
      padding: "8px 12px",
      borderRadius: "6px",
      fontSize: "14px",
      fontWeight: "400",
      lineHeight: "1.4",
      maxWidth: this.tooltipMaxWidth,
      wordWrap: "break-word",
      opacity: "0",
      transform: "scale(0.8)",
      transition: "all 0.2s ease-in-out",
      pointerEvents: "none",
      whiteSpace: "pre-wrap",
    };

    Object.entries(baseStyles).forEach(([prop, value]) => {
      this.renderer.setStyle(this.tooltipElement, prop, value);
    });

    this.applyThemeStyles();
  }

  private applyThemeStyles(): void {
    if (!this.tooltipElement) return;

    const themes = {
      dark: {
        backgroundColor: "rgba(33, 37, 41, 0.95)",
        color: "#fff",
        boxShadow: "0 4px 12px rgba(0, 0, 0, 0.3)",
      },
      light: {
        backgroundColor: "rgba(255, 255, 255, 0.95)",
        color: "#333",
        border: "1px solid #e9ecef",
        boxShadow: "0 4px 12px rgba(0, 0, 0, 0.15)",
      },
      info: {
        backgroundColor: "rgba(23, 162, 184, 0.95)",
        color: "#fff",
        boxShadow: "0 4px 12px rgba(23, 162, 184, 0.3)",
      },
      warning: {
        backgroundColor: "rgba(255, 193, 7, 0.95)",
        color: "#212529",
        boxShadow: "0 4px 12px rgba(255, 193, 7, 0.3)",
      },
      error: {
        backgroundColor: "rgba(220, 53, 69, 0.95)",
        color: "#fff",
        boxShadow: "0 4px 12px rgba(220, 53, 69, 0.3)",
      },
    };

    const themeStyles = themes[this.tooltipTheme];
    Object.entries(themeStyles).forEach(([prop, value]) => {
      this.renderer.setStyle(this.tooltipElement, prop, value);
    });
  }

  private positionTooltip(): void {
    if (!this.tooltipElement) return;

    const hostRect = this.el.nativeElement.getBoundingClientRect();
    const tooltipRect = this.tooltipElement.getBoundingClientRect();
    const viewport = {
      width: window.innerWidth,
      height: window.innerHeight,
      scrollX: window.pageXOffset,
      scrollY: window.pageYOffset,
    };

    let position = this.tooltipPosition;

    // Auto-detect best position if set to 'auto'
    if (position === "auto") {
      position = this.getBestPosition(hostRect, tooltipRect, viewport);
    }

    const coords = this.calculatePosition(
      position,
      hostRect,
      tooltipRect,
      viewport
    );

    this.renderer.setStyle(this.tooltipElement, "top", `${coords.top}px`);
    this.renderer.setStyle(this.tooltipElement, "left", `${coords.left}px`);

    // Add position class for potential arrow styling
    this.renderer.addClass(this.tooltipElement, `tooltip-${position}`);
  }

  private getBestPosition(
    hostRect: DOMRect,
    tooltipRect: DOMRect,
    viewport: any
  ): "top" | "bottom" | "left" | "right" {
    const space = {
      top: hostRect.top,
      bottom: viewport.height - hostRect.bottom,
      left: hostRect.left,
      right: viewport.width - hostRect.right,
    };

    const needed = {
      top: tooltipRect.height + this.tooltipOffset,
      bottom: tooltipRect.height + this.tooltipOffset,
      left: tooltipRect.width + this.tooltipOffset,
      right: tooltipRect.width + this.tooltipOffset,
    };

    // Prefer top/bottom over left/right
    if (space.top >= needed.top) return "top";
    if (space.bottom >= needed.bottom) return "bottom";
    if (space.right >= needed.right) return "right";
    if (space.left >= needed.left) return "left";

    // Fallback to position with most space
    const maxSpace = Math.max(space.top, space.bottom, space.left, space.right);
    return Object.keys(space).find(
      (key) => space[key as keyof typeof space] === maxSpace
    ) as any;
  }

  private calculatePosition(
    position: "top" | "bottom" | "left" | "right",
    hostRect: DOMRect,
    tooltipRect: DOMRect,
    viewport: any
  ): { top: number; left: number } {
    let top = 0;
    let left = 0;

    switch (position) {
      case "top":
        top =
          hostRect.top -
          tooltipRect.height -
          this.tooltipOffset +
          viewport.scrollY;
        left =
          hostRect.left +
          (hostRect.width - tooltipRect.width) / 2 +
          viewport.scrollX;
        break;

      case "bottom":
        top = hostRect.bottom + this.tooltipOffset + viewport.scrollY;
        left =
          hostRect.left +
          (hostRect.width - tooltipRect.width) / 2 +
          viewport.scrollX;
        break;

      case "left":
        top =
          hostRect.top +
          (hostRect.height - tooltipRect.height) / 2 +
          viewport.scrollY;
        left =
          hostRect.left -
          tooltipRect.width -
          this.tooltipOffset +
          viewport.scrollX;
        break;

      case "right":
        top =
          hostRect.top +
          (hostRect.height - tooltipRect.height) / 2 +
          viewport.scrollY;
        left = hostRect.right + this.tooltipOffset + viewport.scrollX;
        break;
    }

    // Constrain to viewport
    left = Math.max(
      8 + viewport.scrollX,
      Math.min(left, viewport.width - tooltipRect.width - 8 + viewport.scrollX)
    );
    top = Math.max(
      8 + viewport.scrollY,
      Math.min(top, viewport.height - tooltipRect.height - 8 + viewport.scrollY)
    );

    return { top, left };
  }

  private repositionTooltip(): void {
    this.positionTooltip();
  }

  private animateIn(): void {
    if (!this.tooltipElement) return;

    requestAnimationFrame(() => {
      this.renderer.setStyle(this.tooltipElement, "opacity", "1");
      this.renderer.setStyle(this.tooltipElement, "transform", "scale(1)");
    });
  }

  private animateOut(callback: () => void): void {
    if (!this.tooltipElement) return;

    this.renderer.setStyle(this.tooltipElement, "opacity", "0");
    this.renderer.setStyle(this.tooltipElement, "transform", "scale(0.8)");

    setTimeout(callback, 200);
  }

  private removeTooltip(): void {
    if (this.tooltipElement) {
      this.renderer.removeChild(document.body, this.tooltipElement);
      this.tooltipElement = undefined;
    }
  }

  private clearTimeouts(): void {
    if (this.showTimeout) {
      clearTimeout(this.showTimeout);
      this.showTimeout = undefined;
    }

    if (this.hideTimeout) {
      clearTimeout(this.hideTimeout);
      this.hideTimeout = undefined;
    }
  }

  private cleanup(): void {
    this.clearTimeouts();
    this.hideTooltip();

    if (this.resizeObserver) {
      this.resizeObserver.disconnect();
    }
  }

  private getTooltipId(): string {
    return `tooltip-${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

---

## 🔧 Custom Pipes {#custom-pipes}

Custom pipes transform data in templates, providing reusable data formatting and processing capabilities.

### 🔹 Transform Pipes {#transform-pipes}

Custom pipes that transform data for display purposes.

```typescript
// Advanced Text Transform Pipe - Data Transformation for Display
@Pipe({
  name: "textTransform", // 🏷️ Pipe name used in templates: {{ text | textTransform }}
  standalone: true, // ✨ Self-contained pipe (Angular 20 feature)
  pure: true, // 🔄 Pure pipe - only re-executes when input changes (performance optimization)
})
export class TextTransformPipe implements PipeTransform {
  /**
   * 🔄 TRANSFORM METHOD - Main pipe logic
   * Transforms text based on specified type and options
   * @param {string} value - Input text to transform
   * @param {string} type - Transformation type (uppercase, lowercase, etc.)
   * @param {object} options - Optional configuration object
   * @returns {string} Transformed text
   */
  transform(
    value: string, // 📝 Input text value
    type:
      | "uppercase" // 📤 CONVERT TO UPPERCASE
      | "lowercase" // 📥 convert to lowercase
      | "capitalize" // 📝 Capitalize first letter only
      | "title" // 📚 Title Case Every Word
      | "camel" // 🐪 camelCase transformation
      | "snake" // 🐍 snake_case transformation
      | "kebab" = "capitalize", // 🍖 kebab-case transformation (default: capitalize)
    options?: {
      preserveCase?: boolean; // 🔒 Keep original casing where applicable
      locale?: string; // 🌍 Locale for proper case transformations
      stripHtml?: boolean; // 🧹 Remove HTML tags from input
      truncate?: number; // ✂️ Maximum character length
      ellipsis?: string; // 📝 Text to append when truncating ("..." default)
    }
  ): string {
    // 🚫 VALIDATION: Handle null/undefined input
    if (!value) return "";

    // 📝 CONVERT TO STRING: Ensure we're working with string data
    let result = value.toString();

    // 🧹 HTML STRIPPING: Remove HTML tags if requested
    if (options?.stripHtml) {
      result = result.replace(/<[^>]*>/g, ""); // 🧹 Regex to remove all HTML tags
    }

    // 🔄 TRANSFORMATION LOGIC: Apply requested transformation type
    switch (type) {
      case "uppercase":
        // 🌍 LOCALE-AWARE UPPERCASE: Use locale if provided
        result = options?.locale
          ? result.toLocaleUpperCase(options.locale) // 🌍 Locale-specific uppercase
          : result.toUpperCase(); // 📤 Standard uppercase
        break;

      case "lowercase":
        // 🌍 LOCALE-AWARE LOWERCASE: Use locale if provided
        result = options?.locale
          ? result.toLocaleLowerCase(options.locale) // 🌍 Locale-specific lowercase
          : result.toLowerCase(); // 📥 Standard lowercase
        break;

      case "capitalize":
        // 📝 CAPITALIZE FIRST LETTER ONLY
        result =
          result.charAt(0).toUpperCase() + // 📤 First character uppercase
          (options?.preserveCase
            ? result.slice(1) // 🔒 Keep rest of string as-is
            : result.slice(1).toLowerCase()); // 📥 Make rest lowercase
        break;

      case "title":
        // 📚 TITLE CASE: Capitalize Each Word
        result = result.replace(
          /\w\S*/g, // 🔍 Regex: match words (word char + non-space chars)
          (txt) =>
            txt.charAt(0).toUpperCase() + // 📤 Capitalize first letter
            (options?.preserveCase
              ? txt.substr(1) // 🔒 Preserve rest of word casing
              : txt.substr(1).toLowerCase()) // 📥 Lowercase rest of word
        );
        break;

      case "camel":
        // 🐪 CAMEL CASE TRANSFORMATION
        result = result
          .replace(
            /(?:^\w|[A-Z]|\b\w)/g,
            (word, index) =>
              index === 0 ? word.toLowerCase() : word.toUpperCase() // 📝 First word lowercase, rest uppercase
          )
          .replace(/\s+/g, ""); // 🧹 Remove all spaces
        break;

      case "snake":
        // 🐍 SNAKE_CASE TRANSFORMATION
        result = result
          .replace(/\W+/g, " ") // 🔄 Replace non-word chars with spaces
          .split(/ |\B(?=[A-Z])/) // ✂️ Split on spaces and before capital letters
          .map((word) => word.toLowerCase()) // 📥 Lowercase each word
          .join("_"); // 🔗 Join with underscores
        break;

      case "kebab":
        // 🍖 KEBAB-CASE TRANSFORMATION
        result = result
          .replace(/\W+/g, " ") // 🔄 Replace non-word chars with spaces
          .split(/ |\B(?=[A-Z])/) // ✂️ Split on spaces and before capital letters
          .map((word) => word.toLowerCase()) // 📥 Lowercase each word
          .join("-"); // 🔗 Join with hyphens
        break;
    }

    // ✂️ TRUNCATION: Apply length limit if specified
    if (options?.truncate && result.length > options.truncate) {
      const ellipsis = options.ellipsis || "..."; // 📝 Default ellipsis
      result =
        result.substring(0, options.truncate - ellipsis.length) + ellipsis;
      // ✂️ Cut text and append ellipsis
    }

    return result; // 📤 Return transformed text
  }
}

// Advanced Number Format Pipe - International Number Formatting
@Pipe({
  name: "numberFormat", // 🏷️ Pipe name: {{ 1234.56 | numberFormat:'currency' }}
  standalone: true, // ✨ Self-contained pipe (Angular 20 feature)
  pure: true, // 🔄 Pure pipe for performance optimization
})
export class NumberFormatPipe implements PipeTransform {
  /**
   * 🔢 TRANSFORM METHOD - Format numbers with international standards
   * Uses Intl.NumberFormat API for proper localization
   * @param {number | string} value - Number to format
   * @param {string} format - Format type (currency, percent, etc.)
   * @param {object} options - Intl.NumberFormat options
   * @returns {string} Formatted number string
   */
  transform(
    value: number | string, // 🔢 Input number (string numbers are parsed)
    format:
      | "currency" // 💰 Currency formatting ($1,234.56)
      | "percent" // 📊 Percentage formatting (50%)
      | "decimal" // 🔢 Decimal formatting (1,234.56)
      | "scientific" // 🧪 Scientific notation (1.23E+3)
      | "compact" // 📱 Compact notation (1.2K)
      | "ordinal" = "decimal", // 🥇 Ordinal numbers (1st, 2nd, 3rd) - default: decimal
    options?: {
      locale?: string; // 🌍 Locale for formatting (en-US, de-DE, etc.)
      currency?: string; // 💰 Currency code (USD, EUR, etc.)
      currencyDisplay?: "symbol" | "code" | "name"; // 💰 Currency display style ($, USD, Dollar)
      minimumFractionDigits?: number; // 📏 Minimum decimal places
      maximumFractionDigits?: number; // 📏 Maximum decimal places
      minimumIntegerDigits?: number; // 📏 Minimum integer digits (zero padding)
      useGrouping?: boolean; // 🔢 Use thousand separators (1,234 vs 1234)
      notation?: "standard" | "scientific" | "engineering" | "compact"; // 📝 Number notation style
      compactDisplay?: "short" | "long"; // 📱 Compact display style (1K vs 1 thousand)
      signDisplay?: "auto" | "never" | "always" | "exceptZero"; // ➕➖ Sign display rules
    }
  ): string {
    // 🔢 NUMBER PARSING: Convert string to number if needed
    const numValue = typeof value === "string" ? parseFloat(value) : value;

    // 🚫 VALIDATION: Handle invalid numbers
    if (isNaN(numValue)) return value.toString();

    // 🌍 LOCALE SETUP: Default to US English
    const locale = options?.locale || "en-US";

    // ⚙️ BASE OPTIONS: Common Intl.NumberFormat options
    const baseOptions: Intl.NumberFormatOptions = {
      minimumFractionDigits: options?.minimumFractionDigits, // 📏 Min decimal places
      maximumFractionDigits: options?.maximumFractionDigits, // 📏 Max decimal places
      minimumIntegerDigits: options?.minimumIntegerDigits, // 📏 Min integer digits
      useGrouping: options?.useGrouping ?? true, // 🔢 Grouping enabled by default
      notation: options?.notation, // 📝 Notation style
      signDisplay: options?.signDisplay, // ➕➖ Sign display
    };

    try {
      // 🔄 FORMAT SWITCH: Apply specific formatting based on format type
      switch (format) {
        case "currency":
          // 💰 CURRENCY FORMATTING
          return new Intl.NumberFormat(locale, {
            ...baseOptions, // 📋 Spread base options
            style: "currency", // 💰 Currency style
            currency: options?.currency || "USD", // 💵 Default to US Dollar
            currencyDisplay: options?.currencyDisplay || "symbol", // 💰 Show symbol ($)
          }).format(numValue);

        case "percent":
          // 📊 PERCENTAGE FORMATTING
          return new Intl.NumberFormat(locale, {
            ...baseOptions, // 📋 Spread base options
            style: "percent", // 📊 Percentage style
          }).format(numValue);

        case "scientific":
          // 🧪 SCIENTIFIC NOTATION
          return new Intl.NumberFormat(locale, {
            ...baseOptions, // 📋 Spread base options
            notation: "scientific", // 🧪 Force scientific notation
          }).format(numValue);

        case "compact":
          return new Intl.NumberFormat(locale, {
            ...baseOptions,
            notation: "compact",
            compactDisplay: options?.compactDisplay || "short",
          }).format(numValue);

        case "ordinal":
          // English ordinals (1st, 2nd, 3rd, etc.)
          const suffix = ["th", "st", "nd", "rd"];
          const remainder = numValue % 100;
          const ordinalSuffix =
            suffix[(remainder - 20) % 10] || suffix[remainder] || suffix[0];
          return numValue + ordinalSuffix;

        case "decimal":
        default:
          return new Intl.NumberFormat(locale, baseOptions).format(numValue);
      }
    } catch (error) {
      console.warn("NumberFormatPipe error:", error);
      return value.toString();
    }
  }
}

// Advanced Date Format Pipe
@Pipe({
  name: "dateFormat",
  standalone: true,
  pure: true,
})
export class DateFormatPipe implements PipeTransform {
  transform(
    value: Date | string | number,
    format:
      | "short"
      | "medium"
      | "long"
      | "full"
      | "relative"
      | "custom" = "medium",
    options?: {
      locale?: string;
      timezone?: string;
      customFormat?: string;
      relativeTo?: Date;
      includeTime?: boolean;
      use24Hour?: boolean;
    }
  ): string {
    if (!value) return "";

    const date = new Date(value);
    if (isNaN(date.getTime())) return value.toString();

    const locale = options?.locale || "en-US";
    const timezone =
      options?.timezone || Intl.DateTimeFormat().resolvedOptions().timeZone;

    try {
      switch (format) {
        case "relative":
          return this.formatRelative(date, options?.relativeTo, locale);

        case "custom":
          return this.formatCustom(
            date,
            options?.customFormat || "yyyy-MM-dd",
            locale
          );

        case "short":
          return new Intl.DateTimeFormat(locale, {
            dateStyle: "short",
            timeStyle: options?.includeTime ? "short" : undefined,
            timeZone: timezone,
            hour12: !options?.use24Hour,
          }).format(date);

        case "medium":
          return new Intl.DateTimeFormat(locale, {
            dateStyle: "medium",
            timeStyle: options?.includeTime ? "medium" : undefined,
            timeZone: timezone,
            hour12: !options?.use24Hour,
          }).format(date);

        case "long":
          return new Intl.DateTimeFormat(locale, {
            dateStyle: "long",
            timeStyle: options?.includeTime ? "long" : undefined,
            timeZone: timezone,
            hour12: !options?.use24Hour,
          }).format(date);

        case "full":
          return new Intl.DateTimeFormat(locale, {
            dateStyle: "full",
            timeStyle: options?.includeTime ? "full" : undefined,
            timeZone: timezone,
            hour12: !options?.use24Hour,
          }).format(date);

        default:
          return date.toISOString();
      }
    } catch (error) {
      console.warn("DateFormatPipe error:", error);
      return date.toISOString();
    }
  }

  private formatRelative(
    date: Date,
    relativeTo?: Date,
    locale = "en-US"
  ): string {
    const now = relativeTo || new Date();
    const diffInSeconds = (now.getTime() - date.getTime()) / 1000;
    const diffInMinutes = diffInSeconds / 60;
    const diffInHours = diffInMinutes / 60;
    const diffInDays = diffInHours / 24;

    if ("Intl" in window && "RelativeTimeFormat" in Intl) {
      const rtf = new Intl.RelativeTimeFormat(locale, { numeric: "auto" });

      if (Math.abs(diffInSeconds) < 60) {
        return rtf.format(-Math.round(diffInSeconds), "second");
      } else if (Math.abs(diffInMinutes) < 60) {
        return rtf.format(-Math.round(diffInMinutes), "minute");
      } else if (Math.abs(diffInHours) < 24) {
        return rtf.format(-Math.round(diffInHours), "hour");
      } else if (Math.abs(diffInDays) < 30) {
        return rtf.format(-Math.round(diffInDays), "day");
      } else {
        return rtf.format(-Math.round(diffInDays / 30), "month");
      }
    }

    // Fallback for older browsers
    return this.formatRelativeFallback(diffInSeconds);
  }

  private formatRelativeFallback(diffInSeconds: number): string {
    const absSeconds = Math.abs(diffInSeconds);
    const isPast = diffInSeconds > 0;

    if (absSeconds < 60) {
      return isPast ? "just now" : "in a moment";
    } else if (absSeconds < 3600) {
      const minutes = Math.round(absSeconds / 60);
      return isPast ? `${minutes}m ago` : `in ${minutes}m`;
    } else if (absSeconds < 86400) {
      const hours = Math.round(absSeconds / 3600);
      return isPast ? `${hours}h ago` : `in ${hours}h`;
    } else {
      const days = Math.round(absSeconds / 86400);
      return isPast ? `${days}d ago` : `in ${days}d`;
    }
  }

  private formatCustom(date: Date, format: string, locale: string): string {
    // Simple custom format implementation
    const tokens: { [key: string]: string } = {
      yyyy: date.getFullYear().toString(),
      yy: date.getFullYear().toString().slice(-2),
      MM: (date.getMonth() + 1).toString().padStart(2, "0"),
      M: (date.getMonth() + 1).toString(),
      dd: date.getDate().toString().padStart(2, "0"),
      d: date.getDate().toString(),
      HH: date.getHours().toString().padStart(2, "0"),
      H: date.getHours().toString(),
      hh: (date.getHours() % 12 || 12).toString().padStart(2, "0"),
      h: (date.getHours() % 12 || 12).toString(),
      mm: date.getMinutes().toString().padStart(2, "0"),
      m: date.getMinutes().toString(),
      ss: date.getSeconds().toString().padStart(2, "0"),
      s: date.getSeconds().toString(),
      a: date.getHours() >= 12 ? "PM" : "AM",
      A: date.getHours() >= 12 ? "PM" : "AM",
    };

    let result = format;
    Object.keys(tokens).forEach((token) => {
      result = result.replace(new RegExp(token, "g"), tokens[token]);
    });

    return result;
  }
}

// File Size Pipe
@Pipe({
  name: "fileSize",
  standalone: true,
  pure: true,
})
export class FileSizePipe implements PipeTransform {
  transform(
    bytes: number,
    precision: number = 1,
    unit: "binary" | "decimal" = "binary"
  ): string {
    if (bytes === 0) return "0 Bytes";
    if (!bytes || isNaN(bytes)) return "";

    const base = unit === "binary" ? 1024 : 1000;
    const sizes =
      unit === "binary"
        ? ["Bytes", "KiB", "MiB", "GiB", "TiB", "PiB"]
        : ["Bytes", "KB", "MB", "GB", "TB", "PB"];

    const index = Math.floor(Math.log(bytes) / Math.log(base));
    const size = bytes / Math.pow(base, index);

    return `${size.toFixed(precision)} ${sizes[index]}`;
  }
}

// Safe HTML Pipe
@Pipe({
  name: "safeHtml",
  standalone: true,
  pure: true,
})
export class SafeHtmlPipe implements PipeTransform {
  constructor(private sanitizer: DomSanitizer) {}

  transform(
    value: string,
    type: "html" | "style" | "script" | "url" | "resourceUrl" = "html"
  ): SafeHtml {
    if (!value) return "";

    switch (type) {
      case "html":
        return this.sanitizer.bypassSecurityTrustHtml(value);
      case "style":
        return this.sanitizer.bypassSecurityTrustStyle(value);
      case "script":
        return this.sanitizer.bypassSecurityTrustScript(value);
      case "url":
        return this.sanitizer.bypassSecurityTrustUrl(value);
      case "resourceUrl":
        return this.sanitizer.bypassSecurityTrustResourceUrl(value);
      default:
        return this.sanitizer.bypassSecurityTrustHtml(value);
    }
  }
}
```

### 🔹 Async Pipes {#async-pipes}

Advanced async pipes that handle observables and promises with enhanced features.

```typescript
// Enhanced Async Pipe with Loading States
@Pipe({
  name: "asyncEnhanced",
  standalone: true,
  pure: false,
})
export class AsyncEnhancedPipe implements PipeTransform, OnDestroy {
  private subscription?: Subscription;
  private currentValue: any = null;
  private isLoading = true;
  private error: any = null;
  private hasValue = false;

  transform(
    observable: Observable<any> | Promise<any> | null | undefined,
    options?: {
      loadingValue?: any;
      errorValue?: any;
      retryCount?: number;
      retryDelay?: number;
      timeout?: number;
    }
  ): { value: any; loading: boolean; error: any; hasValue: boolean } {
    if (!observable) {
      return {
        value: null,
        loading: false,
        error: null,
        hasValue: false,
      };
    }

    // Unsubscribe from previous subscription
    this.unsubscribe();

    const obs$ = this.normalizeToObservable(observable);

    // Apply timeout if specified
    const timeoutObs$ = options?.timeout
      ? obs$.pipe(timeout(options.timeout))
      : obs$;

    // Apply retry logic if specified
    const retryObs$ = options?.retryCount
      ? timeoutObs$.pipe(
          retry({
            count: options.retryCount,
            delay: options.retryDelay || 1000,
          })
        )
      : timeoutObs$;

    // Reset state
    this.isLoading = true;
    this.error = null;
    this.hasValue = false;

    this.subscription = retryObs$.subscribe({
      next: (value) => {
        this.currentValue = value;
        this.isLoading = false;
        this.hasValue = true;
        this.error = null;
      },
      error: (error) => {
        this.error = error;
        this.isLoading = false;
        this.currentValue = options?.errorValue ?? null;
        console.error("AsyncEnhanced pipe error:", error);
      },
    });

    return {
      value: this.hasValue ? this.currentValue : options?.loadingValue ?? null,
      loading: this.isLoading,
      error: this.error,
      hasValue: this.hasValue,
    };
  }

  ngOnDestroy(): void {
    this.unsubscribe();
  }

  private normalizeToObservable(
    input: Observable<any> | Promise<any>
  ): Observable<any> {
    if (input instanceof Promise) {
      return from(input);
    }
    return input as Observable<any>;
  }

  private unsubscribe(): void {
    if (this.subscription) {
      this.subscription.unsubscribe();
      this.subscription = undefined;
    }
  }
}

// Debounced Async Pipe
@Pipe({
  name: "asyncDebounced",
  standalone: true,
  pure: false,
})
export class AsyncDebouncedPipe implements PipeTransform, OnDestroy {
  private subscription?: Subscription;
  private currentValue: any = null;
  private pendingValue: any = null;

  transform(
    observable: Observable<any> | null | undefined,
    debounceTime: number = 300,
    distinctUntilChanged: boolean = true
  ): any {
    if (!observable) return this.currentValue;

    this.unsubscribe();

    let processedObs$ = observable.pipe(debounceTime(debounceTime));

    if (distinctUntilChanged) {
      processedObs$ = processedObs$.pipe(distinctUntilChanged());
    }

    this.subscription = processedObs$.subscribe((value) => {
      this.currentValue = value;
    });

    return this.currentValue;
  }

  ngOnDestroy(): void {
    this.unsubscribe();
  }

  private unsubscribe(): void {
    if (this.subscription) {
      this.subscription.unsubscribe();
      this.subscription = undefined;
    }
  }
}

// Cached Async Pipe
@Pipe({
  name: "asyncCached",
  standalone: true,
  pure: false,
})
export class AsyncCachedPipe implements PipeTransform, OnDestroy {
  private static cache = new Map<string, { value: any; timestamp: number }>();
  private subscription?: Subscription;
  private currentValue: any = null;
  private cacheKey?: string;

  transform(
    observable: Observable<any> | null | undefined,
    cacheKey: string,
    ttl: number = 300000 // 5 minutes default
  ): any {
    if (!observable || !cacheKey) return this.currentValue;

    this.cacheKey = cacheKey;

    // Check cache first
    const cached = AsyncCachedPipe.cache.get(cacheKey);
    const now = Date.now();

    if (cached && now - cached.timestamp < ttl) {
      this.currentValue = cached.value;
      return this.currentValue;
    }

    // Remove expired cache entry
    if (cached && now - cached.timestamp >= ttl) {
      AsyncCachedPipe.cache.delete(cacheKey);
    }

    this.unsubscribe();

    this.subscription = observable
      .pipe(
        take(1) // Only take the first emission for caching
      )
      .subscribe((value) => {
        this.currentValue = value;

        // Cache the value
        AsyncCachedPipe.cache.set(cacheKey, {
          value,
          timestamp: now,
        });
      });

    return this.currentValue;
  }

  ngOnDestroy(): void {
    this.unsubscribe();
  }

  private unsubscribe(): void {
    if (this.subscription) {
      this.subscription.unsubscribe();
      this.subscription = undefined;
    }
  }

  // Static method to clear cache
  static clearCache(key?: string): void {
    if (key) {
      AsyncCachedPipe.cache.delete(key);
    } else {
      AsyncCachedPipe.cache.clear();
    }
  }
}
```

### 🔹 Pure vs Impure Pipes {#pipe-types}

Understanding the difference between pure and impure pipes and when to use each.

```typescript
// Pure Pipe Example - Only recalculates when input changes
@Pipe({
  name: "pureMath",
  standalone: true,
  pure: true, // This is the default
})
export class PureMathPipe implements PipeTransform {
  private calculationCount = 0;

  transform(
    value: number,
    operation: "square" | "cube" | "sqrt" = "square"
  ): number {
    this.calculationCount++;
    console.log(`🔢 Pure pipe calculation #${this.calculationCount}`);

    switch (operation) {
      case "square":
        return Math.pow(value, 2);
      case "cube":
        return Math.pow(value, 3);
      case "sqrt":
        return Math.sqrt(value);
      default:
        return value;
    }
  }
}

// Impure Pipe Example - Recalculates on every change detection
@Pipe({
  name: "impureClock",
  standalone: true,
  pure: false, // Explicitly set to false
})
export class ImpureClockPipe implements PipeTransform {
  transform(format: "time" | "date" | "datetime" = "time"): string {
    const now = new Date();

    switch (format) {
      case "time":
        return now.toLocaleTimeString();
      case "date":
        return now.toLocaleDateString();
      case "datetime":
        return now.toLocaleString();
      default:
        return now.toISOString();
    }
  }
}

// Pure Pipe with Object Input (demonstrates common pitfall)
@Pipe({
  name: "objectFilter",
  standalone: true,
  pure: true,
})
export class ObjectFilterPipe implements PipeTransform {
  transform<T>(items: T[], filterFn: (item: T) => boolean): T[] {
    if (!items || !filterFn) return items;

    console.log("🔍 Object filter called");
    return items.filter(filterFn);
  }
}

// Optimized Pure Pipe with Memoization
@Pipe({
  name: "memoizedFilter",
  standalone: true,
  pure: true,
})
export class MemoizedFilterPipe implements PipeTransform {
  private cache = new Map<string, any>();

  transform<T>(
    items: T[],
    property: keyof T,
    searchTerm: string,
    caseSensitive: boolean = false
  ): T[] {
    if (!items || !searchTerm) return items;

    // Create cache key based on inputs
    const cacheKey = JSON.stringify({
      itemsHash: this.hashArray(items),
      property,
      searchTerm: caseSensitive ? searchTerm : searchTerm.toLowerCase(),
      caseSensitive,
    });

    // Check cache
    if (this.cache.has(cacheKey)) {
      console.log("📦 Using cached filter result");
      return this.cache.get(cacheKey);
    }

    console.log("🔍 Computing filter result");

    const filterValue = caseSensitive ? searchTerm : searchTerm.toLowerCase();

    const result = items.filter((item) => {
      const itemValue = String(item[property]);
      const compareValue = caseSensitive ? itemValue : itemValue.toLowerCase();
      return compareValue.includes(filterValue);
    });

    // Cache result
    this.cache.set(cacheKey, result);

    // Clean up old cache entries (keep last 10)
    if (this.cache.size > 10) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    return result;
  }

  private hashArray<T>(arr: T[]): string {
    return arr.length + "_" + JSON.stringify(arr.slice(0, 3)); // Quick hash
  }
}

// Performance Comparison Component
@Component({
  selector: "app-pipe-performance",
  standalone: true,
  imports: [
    CommonModule,
    PureMathPipe,
    ImpureClockPipe,
    ObjectFilterPipe,
    MemoizedFilterPipe,
  ],
  template: `
    <div class="pipe-performance">
      <h3>Pipe Performance Comparison</h3>

      <div class="performance-section">
        <h4>Pure vs Impure Behavior</h4>

        <!-- Pure pipe - only recalculates when input changes -->
        <div class="test-case">
          <p>Pure Math Pipe (value: {{ mathValue }}):</p>
          <p>Square: {{ mathValue | pureMath : "square" }}</p>
          <p>Cube: {{ mathValue | pureMath : "cube" }}</p>
          <button (click)="changeMathValue()">Change Value</button>
          <button (click)="triggerChangeDetection()">Trigger CD</button>
        </div>

        <!-- Impure pipe - recalculates on every change detection -->
        <div class="test-case">
          <p>Impure Clock Pipe (recalculates every CD cycle):</p>
          <p>Current time: {{ "time" | impureClock }}</p>
          <p>Change detection cycles: {{ cdCycles }}</p>
          <button (click)="triggerChangeDetection()">Trigger CD</button>
        </div>
      </div>

      <div class="performance-section">
        <h4>Filter Performance</h4>

        <!-- Object filter with function (causes issues with pure pipes) -->
        <div class="test-case">
          <p>Object Filter (creates new function each time):</p>
          <input
            [(ngModel)]="filterTerm"
            placeholder="Filter items..."
            (input)="onFilterChange()"
          />

          <!-- ❌ This will cause issues with pure pipes -->
          <div class="filtered-items">
            <div
              *ngFor="let item of items | objectFilter : getFilterFn()"
              class="item"
            >
              {{ item.name }} - {{ item.category }}
            </div>
          </div>
        </div>

        <!-- Memoized filter with proper caching -->
        <div class="test-case">
          <p>Memoized Filter (optimized caching):</p>
          <input
            [(ngModel)]="memoizedFilterTerm"
            placeholder="Filter items..."
          />

          <div class="filtered-items">
            <div
              *ngFor="
                let item of items | memoizedFilter : 'name' : memoizedFilterTerm
              "
              class="item"
            >
              {{ item.name }} - {{ item.category }}
            </div>
          </div>
        </div>
      </div>

      <div class="performance-metrics">
        <h4>Performance Metrics</h4>
        <p>Total change detection cycles: {{ cdCycles }}</p>
        <p>Filter computations: {{ filterComputations }}</p>
        <p>Math computations: {{ mathComputations }}</p>
      </div>
    </div>
  `,
  styles: [
    `
      .pipe-performance {
        padding: 20px;
        max-width: 800px;
      }

      .performance-section {
        margin: 20px 0;
        padding: 15px;
        border: 1px solid #ddd;
        border-radius: 8px;
      }

      .test-case {
        margin: 15px 0;
        padding: 10px;
        background: #f8f9fa;
        border-radius: 4px;
      }

      .filtered-items {
        max-height: 200px;
        overflow-y: auto;
        margin: 10px 0;
      }

      .item {
        padding: 5px;
        margin: 2px 0;
        background: white;
        border-radius: 3px;
      }

      .performance-metrics {
        background: #e9ecef;
        padding: 15px;
        border-radius: 8px;
        margin-top: 20px;
      }

      button {
        margin: 5px;
        padding: 8px 12px;
        background: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      }

      input {
        padding: 8px;
        border: 1px solid #ddd;
        border-radius: 4px;
        margin: 5px;
      }
    `,
  ],
})
export class PipePerformanceComponent implements DoCheck {
  mathValue = 5;
  filterTerm = "";
  memoizedFilterTerm = "";
  cdCycles = 0;
  filterComputations = 0;
  mathComputations = 0;

  items = [
    { name: "Apple iPhone", category: "Electronics" },
    { name: "Samsung Galaxy", category: "Electronics" },
    { name: "MacBook Pro", category: "Computers" },
    { name: "Dell XPS", category: "Computers" },
    { name: "Sony Headphones", category: "Audio" },
    { name: "Bose Speaker", category: "Audio" },
  ];

  ngDoCheck(): void {
    this.cdCycles++;
  }

  changeMathValue(): void {
    this.mathValue = Math.floor(Math.random() * 20) + 1;
  }

  triggerChangeDetection(): void {
    // This method just triggers change detection
    // The view will update and pipes will be called
  }

  onFilterChange(): void {
    this.filterComputations++;
  }

  // ❌ This creates a new function reference every time
  // causing pure pipes to recalculate unnecessarily
  getFilterFn() {
    return (item: any) => {
      return item.name.toLowerCase().includes(this.filterTerm.toLowerCase());
    };
  }
}
```

---

## 🛠️ Built-in Pipes & Directives {#built-in-features}

Angular provides a comprehensive set of built-in pipes and directives that cover most common use cases.

### 🔹 Common Pipes {#common-pipes}

Built-in pipes for data transformation with Angular 15 vs 20 enhancements.

```typescript
// Comprehensive Built-in Pipes Demo
@Component({
  selector: "app-builtin-pipes-demo",
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div class="builtin-pipes-demo">
      <h3>Built-in Pipes Comprehensive Demo</h3>

      <!-- 🔹 Text Transformation Pipes -->
      <section class="pipe-section">
        <h4>Text Transformation</h4>

        <div class="pipe-example">
          <label>Original text:</label>
          <input [(ngModel)]="sampleText" class="input-field" />

          <div class="transformations">
            <p><strong>Uppercase:</strong> {{ sampleText | uppercase }}</p>
            <p><strong>Lowercase:</strong> {{ sampleText | lowercase }}</p>
            <p><strong>Title Case:</strong> {{ sampleText | titlecase }}</p>

            <!-- Angular 20 enhanced -->
            <p>
              <strong>Slice (0-10):</strong> {{ sampleText | slice : 0 : 10 }}
            </p>
            <p><strong>Slice (5-):</strong> {{ sampleText | slice : 5 }}</p>
          </div>
        </div>
      </section>

      <!-- 🔹 Number Formatting Pipes -->
      <section class="pipe-section">
        <h4>Number Formatting</h4>

        <div class="pipe-example">
          <label>Number value:</label>
          <input type="number" [(ngModel)]="numberValue" class="input-field" />

          <div class="transformations">
            <p>
              <strong>Decimal (1.2-2):</strong>
              {{ numberValue | number : "1.2-2" }}
            </p>
            <p>
              <strong>Currency (USD):</strong>
              {{ numberValue | currency : "USD" : "symbol" : "1.2-2" }}
            </p>
            <p>
              <strong>Currency (EUR):</strong>
              {{ numberValue | currency : "EUR" : "symbol-narrow" : "1.2-2" }}
            </p>
            <p>
              <strong>Percent:</strong>
              {{ numberValue / 100 | percent : "1.1-2" }}
            </p>

            <!-- Angular 20 enhanced currency formatting -->
            <p>
              <strong>Currency (code):</strong>
              {{ numberValue | currency : "USD" : "code" }}
            </p>
            <p>
              <strong>Currency (name):</strong>
              {{
                numberValue
                  | currency : "USD" : "symbol-narrow" : "1.2-2" : "en-US"
              }}
            </p>
          </div>
        </div>
      </section>

      <!-- 🔹 Date Formatting Pipes -->
      <section class="pipe-section">
        <h4>Date Formatting</h4>

        <div class="pipe-example">
          <label>Date value:</label>
          <input
            type="datetime-local"
            [(ngModel)]="dateValue"
            class="input-field"
          />

          <div class="transformations">
            <p><strong>Short Date:</strong> {{ dateValue | date : "short" }}</p>
            <p>
              <strong>Medium Date:</strong> {{ dateValue | date : "medium" }}
            </p>
            <p><strong>Long Date:</strong> {{ dateValue | date : "long" }}</p>
            <p><strong>Full Date:</strong> {{ dateValue | date : "full" }}</p>
            <p>
              <strong>Custom (yyyy-MM-dd):</strong>
              {{ dateValue | date : "yyyy-MM-dd" }}
            </p>
            <p>
              <strong>Custom (HH:mm:ss):</strong>
              {{ dateValue | date : "HH:mm:ss" }}
            </p>
            <p>
              <strong>With Timezone:</strong>
              {{ dateValue | date : "medium" : "UTC" }}
            </p>

            <!-- Angular 20 enhanced date formatting -->
            <p>
              <strong>Relative Time:</strong>
              {{ dateValue | date : "relative" }}
            </p>
            <p>
              <strong>ISO String:</strong>
              {{ dateValue | date : "yyyy-MM-ddTHH:mm:ss.SSSZ" }}
            </p>
          </div>
        </div>
      </section>

      <!-- 🔹 Array Transformation Pipes -->
      <section class="pipe-section">
        <h4>Array Transformation</h4>

        <div class="pipe-example">
          <div class="array-controls">
            <button (click)="addRandomItem()">Add Random Item</button>
            <button (click)="shuffleArray()">Shuffle Array</button>
            <button (click)="sortArray()">Sort Array</button>
          </div>

          <div class="transformations">
            <p><strong>Original Array:</strong> {{ arrayItems }}</p>
            <p><strong>JSON Pipe:</strong> {{ arrayItems | json }}</p>
            <p>
              <strong>Slice (0-3):</strong>
              {{ arrayItems | slice : 0 : 3 | json }}
            </p>
            <p>
              <strong>Slice (-3):</strong> {{ arrayItems | slice : -3 | json }}
            </p>

            <!-- KeyValue pipe for objects -->
            <div class="keyvalue-demo">
              <h5>KeyValue Pipe Demo</h5>
              <div
                *ngFor="let item of objectData | keyvalue : sortByKey"
                class="key-value-item"
              >
                <strong>{{ item.key }}:</strong> {{ item.value }}
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- 🔹 Async Pipe -->
      <section class="pipe-section">
        <h4>Async Pipe</h4>

        <div class="pipe-example">
          <div class="async-controls">
            <button (click)="startTimer()">Start Timer</button>
            <button (click)="stopTimer()">Stop Timer</button>
            <button (click)="loadAsyncData()">Load Async Data</button>
          </div>

          <div class="transformations">
            <p><strong>Timer Value:</strong> {{ timer$ | async }}</p>
            <p><strong>Async Data:</strong> {{ asyncData$ | async | json }}</p>
            <p><strong>Promise Data:</strong> {{ promiseData | async }}</p>

            <!-- Angular 20 enhanced async handling -->
            <div
              class="async-state"
              *ngIf="asyncData$ | async as data; else loading"
            >
              <h5>Loaded Data:</h5>
              <pre>{{ data | json }}</pre>
            </div>

            <ng-template #loading>
              <div class="loading">Loading async data...</div>
            </ng-template>
          </div>
        </div>
      </section>
    </div>
  `,
  styles: [
    `
      .builtin-pipes-demo {
        padding: 20px;
        max-width: 1000px;
        margin: 0 auto;
      }

      .pipe-section {
        margin: 30px 0;
        padding: 20px;
        border: 1px solid #e0e0e0;
        border-radius: 8px;
        background: #f8f9fa;
      }

      .pipe-section h4 {
        color: #495057;
        margin-bottom: 15px;
        border-bottom: 2px solid #007bff;
        padding-bottom: 5px;
      }

      .pipe-example {
        background: white;
        padding: 15px;
        border-radius: 6px;
        border: 1px solid #dee2e6;
      }

      .input-field {
        width: 100%;
        padding: 8px 12px;
        border: 1px solid #ced4da;
        border-radius: 4px;
        margin: 5px 0 15px 0;
        font-size: 14px;
      }

      .transformations {
        margin-top: 15px;
      }

      .transformations p {
        margin: 8px 0;
        padding: 8px;
        background: #f1f3f4;
        border-radius: 4px;
        font-family: "Courier New", monospace;
        font-size: 13px;
      }

      .array-controls,
      .async-controls {
        margin-bottom: 15px;
      }

      .array-controls button,
      .async-controls button {
        margin: 5px;
        padding: 8px 12px;
        background: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
      }

      .array-controls button:hover,
      .async-controls button:hover {
        background: #0056b3;
      }

      .keyvalue-demo {
        margin-top: 15px;
        padding: 15px;
        background: #e9ecef;
        border-radius: 6px;
      }

      .key-value-item {
        padding: 5px 0;
        border-bottom: 1px solid #dee2e6;
      }

      .key-value-item:last-child {
        border-bottom: none;
      }

      .async-state {
        background: #d4edda;
        border: 1px solid #c3e6cb;
        border-radius: 4px;
        padding: 15px;
        margin-top: 10px;
      }

      .loading {
        background: #fff3cd;
        border: 1px solid #ffeaa7;
        border-radius: 4px;
        padding: 15px;
        text-align: center;
        color: #856404;
      }

      pre {
        background: #f8f9fa;
        padding: 10px;
        border-radius: 4px;
        overflow-x: auto;
        font-size: 12px;
      }
    `,
  ],
})
export class BuiltinPipesDemoComponent implements OnInit, OnDestroy {
  // 📝 SAMPLE DATA PROPERTIES - Demonstration data for various pipe types

  sampleText = "Angular Content Projection and Pipes Guide"; // 📄 Text for string transformation demos
  numberValue = 1234.567; // 🔢 Number for numeric formatting demonstrations
  dateValue = new Date().toISOString().slice(0, 16); // 📅 Current date for datetime-local input
  arrayItems = ["Angular", "React", "Vue", "Svelte"]; // 📋 Array for array pipe demonstrations

  // 📋 OBJECT DATA - Key-value pipe demonstration
  objectData = {
    framework: "Angular", // 🏗️ Framework name
    version: "20.0", // 🔢 Version number
    year: 2024, // 📅 Release year
    features: ["Signals", "Standalone", "SSR"], // ✨ Key features array
    popularity: "High", // 📊 Popularity rating
  };

  // 🔄 ASYNC DATA PROPERTIES - For async pipe demonstrations
  timer$?: Observable<number>; // ⏱️ Timer observable for counting
  asyncData$?: Observable<any>; // 📡 Generic async data observable
  promiseData?: Promise<any>; // 🤝 Promise-based async data

  // 📦 SUBSCRIPTION MANAGEMENT
  private timerSubscription?: Subscription; // ⏱️ Timer subscription for cleanup

  ngOnInit(): void {
    console.log("🛠️ Built-in pipes demo initialized");
    this.loadAsyncData(); // 🚀 Initialize async data sources
  }

  ngOnDestroy(): void {
    // 🧹 CLEANUP - Prevent memory leaks
    this.stopTimer(); // ⏱️ Stop timer and clean up subscriptions
  }

  /**
   * ➕ ADD RANDOM ITEM TO ARRAY
   * Demonstrates array manipulation for pipe updates
   * Updates arrayItems with a random framework name
   */
  addRandomItem(): void {
    // 📋 Available framework options for random selection
    const frameworks = [
      "Ember", "Backbone", "Knockout", "Preact", "Lit", "Qwik",
    ];

    // 🎲 Random selection algorithm
    const randomFramework = frameworks[Math.floor(Math.random() * frameworks.length)];

    // 📋 Immutable array update (creates new array for Angular change detection)
    this.arrayItems = [...this.arrayItems, randomFramework];

    console.log("➕ Added framework:", randomFramework);
  }

  /**
   * 🔀 SHUFFLE ARRAY METHOD
   * Randomizes array order using Fisher-Yates shuffle algorithm
   * Demonstrates array pipe reactivity to data changes
   */
  shuffleArray(): void {
    // 📋 Create copy to avoid mutating original array
    const shuffled = [...this.arrayItems];

    // 🔀 FISHER-YATES SHUFFLE ALGORITHM
    for (let i = shuffled.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1)); // 🎲 Random index
      [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]; // 🔄 Swap elements
    }

    // 📋 Update array to trigger change detection
    this.arrayItems = shuffled;
    console.log("🔀 Array shuffled:", shuffled);
  }

  /**
   * 📊 SORT ARRAY METHOD
   * Sorts array alphabetically to demonstrate sorted display
   */
  sortArray(): void {
    // 📋 Sort alphabetically and create new array reference
    this.arrayItems = [...this.arrayItems].sort();
    console.log("📊 Array sorted alphabetically");
  }

  /**
   * 📊 SORT BY KEY FUNCTION
   * Custom comparator for KeyValue pipe sorting
   * @param {KeyValue<string, any>} a - First key-value pair
   * @param {KeyValue<string, any>} b - Second key-value pair
   * @returns {number} Sort order (-1, 0, 1)
   */
  sortByKey = (a: KeyValue<string, any>, b: KeyValue<string, any>): number => {
    // 📝 Alphabetical sorting by key name
    return a.key.localeCompare(b.key);
  };

  /**
   * 🚀 LOAD ASYNC DATA METHOD
   * Initializes various async data sources for pipe demonstrations
   */
  private loadAsyncData(): void {
    // ⏱️ TIMER OBSERVABLE - Counts up every second
    this.timer$ = timer(0, 1000).pipe(
      map(count => count + 1), // 🔢 Start counting from 1
      take(60) // ⏱️ Stop after 60 seconds
    );

    // 📡 SIMULATED API DATA - Mock HTTP request with delay
    this.asyncData$ = of({
      status: 'success', // ✅ Success status
      data: {
        message: 'Data loaded successfully!', // 📝 Success message
        timestamp: new Date().toISOString(), // 📅 Load timestamp
        items: ['Item 1', 'Item 2', 'Item 3'] // 📋 Sample data items
      }
    }).pipe(
      delay(2000) // ⏱️ 2-second delay to simulate network latency
    );

    // 🤝 PROMISE DATA - Promise-based async example
    this.promiseData = new Promise(resolve => {
      // ⏱️ Simulate async operation with setTimeout
      setTimeout(() => {
        resolve({
          type: 'promise', // 🏷️ Data source type
          value: 'Promise resolved successfully!', // ✅ Success message
          resolvedAt: new Date().toISOString() // 📅 Resolution timestamp
        });
      }, 1500); // ⏱️ 1.5-second delay
    });

    console.log("📡 Async data sources initialized");
  }

  /**
   * ⏱️ START TIMER METHOD
   * Starts the demonstration timer for async pipe examples
   */
  startTimer(): void {
    if (this.timer$) {
      console.log("⏱️ Timer already running");
      return;
    }

    // 🚀 Create new timer observable
    this.timer$ = timer(0, 1000).pipe(
      map(count => count + 1),
      take(30) // ⏱️ 30-second duration for demo
    );

    console.log("⏱️ Timer started");
  }

  /**
   * ⏹️ STOP TIMER METHOD
   * Stops the timer and cleans up subscriptions
   */
  stopTimer(): void {
    // 🧹 Clean up timer subscription to prevent memory leaks
    if (this.timerSubscription) {
      this.timerSubscription.unsubscribe();
      this.timerSubscription = undefined;
    }

    // 🔄 Reset timer observable
    this.timer$ = undefined;
    console.log("⏹️ Timer stopped and cleaned up");
  }

  /**
   * 🔄 RELOAD ASYNC DATA METHOD
   * Reloads all async data sources for testing pipe reactivity
   */
  reloadAsyncData(): void {
    console.log("🔄 Reloading async data sources");
    this.loadAsyncData(); // 🚀 Reinitialize all async data
  }

  /**
   * 🎲 RANDOMIZE OBJECT DATA METHOD
   * Updates object data to demonstrate KeyValue pipe reactivity
   */
  randomizeObjectData(): void {
    // 🎲 Random data updates
    const versions = ['18.0', '19.0', '20.0', '21.0'];
    const features = [
      ['Signals', 'Standalone'],
      ['SSR', 'Hydration'],
      ['Control Flow', 'Defer'],
      ['Standalone', 'Signals', 'SSR']
    ];
    const popularityLevels = ['High', 'Very High', 'Excellent', 'Outstanding'];

    // 🔄 Update object data with random values
    this.objectData = {
      ...this.objectData, // 📋 Preserve existing properties
      version: versions[Math.floor(Math.random() * versions.length)],
      features: features[Math.floor(Math.random() * features.length)],
      popularity: popularityLevels[Math.floor(Math.random() * popularityLevels.length)],
      lastUpdated: new Date().toISOString() // 📅 Add timestamp
    };

    console.log("🎲 Object data randomized:", this.objectData);
  }
}
      const j = Math.floor(Math.random() * (i + 1));
      [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    }
    this.arrayItems = shuffled;
  }

  sortArray(): void {
    this.arrayItems = [...this.arrayItems].sort();
  }

  startTimer(): void {
    this.stopTimer(); // Stop any existing timer
    this.timer$ = interval(1000).pipe(map((count) => count + 1));
  }

  stopTimer(): void {
    if (this.timerSubscription) {
      this.timerSubscription.unsubscribe();
    }
    this.timer$ = undefined;
  }

  loadAsyncData(): void {
    // Simulate API call with Observable
    this.asyncData$ = timer(2000).pipe(
      map(() => ({
        id: Math.random().toString(36),
        timestamp: new Date().toISOString(),
        data: "Simulated API response",
        status: "success",
      }))
    );

    // Simulate API call with Promise
    this.promiseData = new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          promiseId: Math.random().toString(36),
          message: "Promise resolved successfully",
          delay: 1500,
        });
      }, 1500);
    });
  }

  sortByKey = (a: KeyValue<string, any>, b: KeyValue<string, any>) => {
    return a.key.localeCompare(b.key);
  };
}
```

<function_calls>
<invoke name="manage_todo_list">
<parameter name="operation">write
