# Architectural Patterns & Leadership Guide for UI Architects

## Table of Contents

1. [Frontend Architectural Patterns](#frontend-architectural-patterns)
2. [Multi-Tenant Architecture Deep Dive](#multi-tenant-architecture-deep-dive)
3. [Microservices & Micro-Frontend Patterns](#microservices-micro-frontend-patterns)
4. [State Management Patterns](#state-management-patterns)
5. [Component Design Patterns](#component-design-patterns)
6. [Leadership & Team Management](#leadership-team-management)
7. [Agile Pod Management](#agile-pod-management)
8. [Stakeholder Management](#stakeholder-management)
9. [Technical Decision Making](#technical-decision-making)
10. [Project Recovery Strategies](#project-recovery-strategies)

---

## Frontend Architectural Patterns

### Model-View-Controller (MVC) Pattern

**What it is:** MVC is a fundamental architectural pattern that separates an application into three interconnected components, each handling different aspects of the application logic.

**The Problem it Solves:**
Imagine you're building a complex e-commerce website. Without proper separation, you might end up with spaghetti code where user interface logic is mixed with business rules and data handling. This makes the application:

- Hard to maintain and debug
- Difficult to test individual components
- Nearly impossible for teams to work on different parts simultaneously
- Prone to bugs when changes are made in one area affecting others

**Real-World Scenario:**
Think of Netflix's content management system. They need to:

- **Model:** Manage massive amounts of movie data, user preferences, viewing history, and recommendations
- **View:** Present this information across web browsers, mobile apps, smart TVs, and game consoles
- **Controller:** Handle user interactions like searching, playing videos, rating content, and managing profiles

**How MVC Works:**

**Model (Data Layer):**

- Represents the business logic and data
- Handles data validation, storage, and retrieval
- Independent of how data is presented
- Example: User profile data, movie catalog, viewing analytics

**View (Presentation Layer):**

- Responsible for displaying data to users
- Handles user interface elements
- Should be "dumb" - only concerned with presentation
- Example: Login forms, movie carousels, search results display

**Controller (Business Logic Layer):**

- Acts as intermediary between Model and View
- Handles user inputs and updates Model accordingly
- Decides which View to show based on user actions
- Example: Authentication logic, search filtering, recommendation algorithms

**Benefits in Practice:**

- **Team Scalability:** Frontend developers can work on Views while backend developers focus on Models
- **Testability:** Each component can be tested independently
- **Flexibility:** You can change the UI without touching business logic
- **Maintainability:** Bug fixes and feature additions are localized to specific layers

**When to Use MVC:**

- Large applications with complex business logic
- Teams where developers have different skill sets (frontend/backend)
- Applications requiring multiple user interfaces (web, mobile, API)
- Long-term projects where maintainability is crucial

### Model-View-ViewModel (MVVM) Pattern

**What it is:** MVVM is an evolution of MVC that introduces a ViewModel layer to handle the presentation logic and state management, particularly powerful in frameworks with two-way data binding.

**The Problem it Solves:**
Traditional MVC can become cumbersome when dealing with complex user interfaces that require frequent updates. Consider a real-time trading dashboard where:

- Stock prices update every few seconds
- Users can customize their dashboard layout
- Multiple charts and widgets need to stay synchronized
- User interactions should provide immediate feedback

**Real-World Scenario:**
Imagine building Slack's messaging interface. You need:

- Real-time message updates across multiple channels
- User presence indicators that change dynamically
- Message composition with live typing indicators
- Thread conversations with nested replies
- Emoji reactions that update in real-time

**How MVVM Works:**

**Model (Data Layer):**

- Same as MVC - pure data and business logic
- Example: Message data, user profiles, channel information

**View (UI Layer):**

- Declarative UI that automatically updates when ViewModel changes
- No direct manipulation of DOM elements
- Example: Message bubbles, channel lists, user avatars

**ViewModel (Presentation Logic):**

- Maintains the state of the View
- Handles all presentation logic
- Exposes data and commands that the View can bind to
- Example: Current channel state, filtered message lists, typing indicators

**Key Advantages of MVVM:**

- **Two-Way Data Binding:** Changes in UI automatically update ViewModel and vice versa
- **Reactive Programming:** UI responds automatically to data changes
- **Testability:** ViewModel can be tested without any UI dependencies
- **Separation of Concerns:** View is purely declarative, all logic is in ViewModel

**When to Use MVVM:**

- Applications with complex, dynamic user interfaces
- Real-time applications with frequent data updates
- Forms-heavy applications with complex validation
- Applications using reactive frameworks (Angular, Vue.js, React with hooks)

### Component-Based Architecture

**What it is:** A design approach where the user interface is broken down into small, reusable, self-contained components that encapsulate their own state and logic.

**The Problem it Solves:**
Traditional web development often led to:

- Code duplication across different pages
- Inconsistent user interface elements
- Difficulty in maintaining design systems
- Poor developer collaboration on UI features

**Real-World Scenario:**
Consider Airbnb's platform. They need consistent UI elements across:

- Property listing cards (used on search results, favorites, host dashboard)
- User profile components (guest profiles, host profiles, reviews)
- Booking flows (date pickers, guest selectors, payment forms)
- Navigation elements (headers, footers, sidebars)

**Component Architecture Principles:**

**Encapsulation:**

- Each component manages its own state and behavior
- Internal implementation can change without affecting other components
- Example: A date picker component handles calendar logic internally

**Reusability:**

- Components can be used across different parts of the application
- Reduces code duplication and maintenance overhead
- Example: A button component used for forms, navigation, and actions

**Composability:**

- Complex UIs are built by combining simpler components
- Higher-order components can enhance functionality
- Example: A booking form composed of date picker, guest selector, and payment components

**Single Responsibility:**

- Each component has one clear purpose
- Easier to test, debug, and maintain
- Example: A user avatar component only handles displaying user images and status

**Component Design Patterns:**

**Presentational vs Container Components:**

- **Presentational:** Focus on how things look (UI components)
- **Container:** Focus on how things work (data fetching, state management)

**Higher-Order Components:**

- Components that take other components and return enhanced versions
- Used for cross-cutting concerns like authentication, logging, theming

**Render Props Pattern:**

- Components that use functions as children to share code
- Flexible way to share functionality between components

**Benefits:**

- **Developer Experience:** Easier to understand and work with small components
- **Team Collaboration:** Different developers can work on different components
- **Design Consistency:** Shared component library ensures uniform UI
- **Testing:** Isolated components are easier to test
- **Performance:** Can optimize individual components independently

---

## 🏗️ SOLID Principles for Frontend Architecture {#solid-principles}

The SOLID principles, originally defined for object-oriented programming, are equally valuable in frontend architecture. They provide a foundation for building maintainable, scalable, and testable frontend applications.

### **S** - Single Responsibility Principle (SRP)

**What it means:** A component, module, or class should have only one reason to change - it should have only one responsibility.

**The Problem it Solves:**
Components that do too many things become:

- Hard to understand and modify
- Difficult to test in isolation
- Prone to bugs when requirements change
- Impossible to reuse in different contexts

**Real-World Example: E-commerce Product Component**

**❌ Violating SRP - Component doing too much:**

```typescript
// BAD: UserProfileCard does everything
interface UserProfileCardProps {
  userId: string;
}

const UserProfileCard: React.FC<UserProfileCardProps> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);
  const [orders, setOrders] = useState<Order[]>([]);
  const [loading, setLoading] = useState(true);
  const [editing, setEditing] = useState(false);
  const [formData, setFormData] = useState<UserFormData>({});

  // Data fetching responsibility
  useEffect(() => {
    const fetchUserData = async () => {
      try {
        const [userData, orderData] = await Promise.all([
          api.getUser(userId),
          api.getUserOrders(userId),
        ]);
        setUser(userData);
        setOrders(orderData);
      } catch (error) {
        console.error("Failed to fetch user data:", error);
      } finally {
        setLoading(false);
      }
    };
    fetchUserData();
  }, [userId]);

  // Form validation responsibility
  const validateForm = () => {
    return (
      formData.name && formData.email && /\S+@\S+\.\S+/.test(formData.email)
    );
  };

  // API communication responsibility
  const handleSave = async () => {
    if (!validateForm()) return;
    try {
      await api.updateUser(userId, formData);
      setUser((prevUser) => ({ ...prevUser, ...formData }));
      setEditing(false);
    } catch (error) {
      alert("Failed to save user data");
    }
  };

  // Analytics responsibility
  const trackProfileView = () => {
    analytics.track("profile_viewed", {
      userId,
      timestamp: Date.now(),
      source: "profile_card",
    });
  };

  // Rendering responsibility
  if (loading) return <LoadingSpinner />;
  if (!user) return <ErrorMessage message="User not found" />;

  return (
    <div className="user-profile-card">
      {/* Complex rendering logic mixed with business logic */}
      {editing ? (
        <form onSubmit={handleSave}>{/* Form fields */}</form>
      ) : (
        <div>
          {/* Display user info */}
          {/* Display orders */}
          {/* Display analytics */}
        </div>
      )}
    </div>
  );
};
```

**✅ Following SRP - Separated responsibilities:**

```typescript
// GOOD: Each component has a single responsibility

// 1. DATA FETCHING RESPONSIBILITY
const useUserData = (userId: string) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const userData = await userApi.getUser(userId);
        setUser(userData);
        setError(null);
      } catch (err) {
        setError("Failed to fetch user data");
        setUser(null);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  return { user, loading, error };
};

// 2. FORM VALIDATION RESPONSIBILITY
const useUserValidation = () => {
  const validateUser = (userData: UserFormData): ValidationResult => {
    const errors: string[] = [];

    if (!userData.name?.trim()) {
      errors.push("Name is required");
    }

    if (!userData.email?.trim()) {
      errors.push("Email is required");
    } else if (!/\S+@\S+\.\S+/.test(userData.email)) {
      errors.push("Email format is invalid");
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  };

  return { validateUser };
};

// 3. USER ANALYTICS RESPONSIBILITY
const useUserAnalytics = () => {
  const trackProfileView = (userId: string) => {
    analytics.track("profile_viewed", {
      userId,
      timestamp: Date.now(),
      source: "profile_card",
    });
  };

  const trackProfileEdit = (userId: string) => {
    analytics.track("profile_edit_started", {
      userId,
      timestamp: Date.now(),
    });
  };

  return { trackProfileView, trackProfileEdit };
};

// 4. USER FORM RESPONSIBILITY
interface UserFormProps {
  user: User;
  onSave: (data: UserFormData) => void;
  onCancel: () => void;
}

const UserForm: React.FC<UserFormProps> = ({ user, onSave, onCancel }) => {
  const [formData, setFormData] = useState<UserFormData>({
    name: user.name,
    email: user.email,
    bio: user.bio,
  });
  const { validateUser } = useUserValidation();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const validation = validateUser(formData);

    if (validation.isValid) {
      onSave(formData);
    } else {
      // Handle validation errors
      console.error("Validation errors:", validation.errors);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="user-form">
      <div className="form-group">
        <label htmlFor="name">Name</label>
        <input
          id="name"
          type="text"
          value={formData.name}
          onChange={(e) =>
            setFormData((prev) => ({ ...prev, name: e.target.value }))
          }
        />
      </div>

      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={formData.email}
          onChange={(e) =>
            setFormData((prev) => ({ ...prev, email: e.target.value }))
          }
        />
      </div>

      <div className="form-actions">
        <button type="submit">Save</button>
        <button type="button" onClick={onCancel}>
          Cancel
        </button>
      </div>
    </form>
  );
};

// 5. USER DISPLAY RESPONSIBILITY
interface UserDisplayProps {
  user: User;
  onEdit: () => void;
}

const UserDisplay: React.FC<UserDisplayProps> = ({ user, onEdit }) => {
  return (
    <div className="user-display">
      <div className="user-avatar">
        <img src={user.avatarUrl} alt={`${user.name} avatar`} />
      </div>

      <div className="user-info">
        <h2>{user.name}</h2>
        <p>{user.email}</p>
        {user.bio && <p className="user-bio">{user.bio}</p>}
      </div>

      <div className="user-actions">
        <button onClick={onEdit}>Edit Profile</button>
      </div>
    </div>
  );
};

// 6. MAIN COMPONENT - ORCHESTRATION RESPONSIBILITY ONLY
interface UserProfileCardProps {
  userId: string;
}

const UserProfileCard: React.FC<UserProfileCardProps> = ({ userId }) => {
  const { user, loading, error } = useUserData(userId);
  const { trackProfileView, trackProfileEdit } = useUserAnalytics();
  const [editing, setEditing] = useState(false);

  useEffect(() => {
    if (user) {
      trackProfileView(userId);
    }
  }, [user, userId, trackProfileView]);

  const handleEdit = () => {
    setEditing(true);
    trackProfileEdit(userId);
  };

  const handleSave = async (formData: UserFormData) => {
    try {
      await userApi.updateUser(userId, formData);
      setEditing(false);
      // Trigger re-fetch of user data
    } catch (error) {
      console.error("Failed to save user:", error);
    }
  };

  const handleCancel = () => {
    setEditing(false);
  };

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return <ErrorMessage message="User not found" />;

  return (
    <div className="user-profile-card">
      {editing ? (
        <UserForm user={user} onSave={handleSave} onCancel={handleCancel} />
      ) : (
        <UserDisplay user={user} onEdit={handleEdit} />
      )}
    </div>
  );
};
```

**Benefits of Following SRP:**

- **Easier Testing:** Each hook and component can be tested independently
- **Better Reusability:** `useUserData` can be used in other components
- **Easier Maintenance:** Changes to validation logic only affect `useUserValidation`
- **Clearer Code:** Each piece has a clear, single purpose
- **Better Team Collaboration:** Different developers can work on different pieces

### **O** - Open/Closed Principle (OCP)

**What it means:** Software entities should be open for extension but closed for modification. You should be able to add new functionality without changing existing code.

**The Problem it Solves:**
When requirements change, modifying existing code can:

- Introduce bugs in working features
- Break dependent components
- Require extensive retesting
- Create merge conflicts in team environments

**Real-World Example: Notification System**

**❌ Violating OCP - Modifying existing code for new features:**

```typescript
// BAD: Adding new notification types requires modifying existing code
class NotificationService {
  showNotification(type: string, message: string, options?: any) {
    switch (type) {
      case "success":
        return this.showSuccessNotification(message);
      case "error":
        return this.showErrorNotification(message);
      case "warning":
        return this.showWarningNotification(message);
      // Every time we need a new type, we modify this method
      case "info":
        return this.showInfoNotification(message);
      case "toast": // New requirement - had to modify existing code
        return this.showToastNotification(message, options);
      case "modal": // Another new requirement - more modification
        return this.showModalNotification(message, options);
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }

  private showSuccessNotification(message: string) {
    // Implementation
  }

  private showErrorNotification(message: string) {
    // Implementation
  }

  // More methods for each type...
}
```

**✅ Following OCP - Extension without modification:**

```typescript
// GOOD: Open for extension, closed for modification

// 1. DEFINE NOTIFICATION INTERFACE
interface NotificationHandler {
  canHandle(type: string): boolean;
  show(message: string, options?: NotificationOptions): void;
}

interface NotificationOptions {
  duration?: number;
  position?: "top" | "bottom" | "center";
  actions?: NotificationAction[];
  persistent?: boolean;
}

interface NotificationAction {
  label: string;
  handler: () => void;
  style?: "primary" | "secondary" | "danger";
}

// 2. IMPLEMENT BASE NOTIFICATION HANDLERS
class SuccessNotificationHandler implements NotificationHandler {
  canHandle(type: string): boolean {
    return type === "success";
  }

  show(message: string, options: NotificationOptions = {}): void {
    const notification = document.createElement("div");
    notification.className = "notification notification--success";
    notification.innerHTML = `
      <div class="notification__icon">✅</div>
      <div class="notification__message">${message}</div>
    `;

    this.displayNotification(notification, options);
  }

  private displayNotification(
    element: HTMLElement,
    options: NotificationOptions
  ): void {
    document.body.appendChild(element);

    if (!options.persistent) {
      setTimeout(() => {
        element.remove();
      }, options.duration || 3000);
    }
  }
}

class ErrorNotificationHandler implements NotificationHandler {
  canHandle(type: string): boolean {
    return type === "error";
  }

  show(message: string, options: NotificationOptions = {}): void {
    const notification = document.createElement("div");
    notification.className = "notification notification--error";
    notification.innerHTML = `
      <div class="notification__icon">❌</div>
      <div class="notification__message">${message}</div>
      <button class="notification__close" onclick="this.parentElement.remove()">×</button>
    `;

    this.displayNotification(notification, { ...options, persistent: true });
  }

  private displayNotification(
    element: HTMLElement,
    options: NotificationOptions
  ): void {
    document.body.appendChild(element);
  }
}

// 3. EXTENSIBLE NOTIFICATION SERVICE
class NotificationService {
  private handlers: NotificationHandler[] = [];

  constructor() {
    // Register default handlers
    this.registerHandler(new SuccessNotificationHandler());
    this.registerHandler(new ErrorNotificationHandler());
  }

  // Open for extension - add new handlers
  registerHandler(handler: NotificationHandler): void {
    this.handlers.push(handler);
  }

  // Closed for modification - this method never changes
  showNotification(
    type: string,
    message: string,
    options?: NotificationOptions
  ): void {
    const handler = this.handlers.find((h) => h.canHandle(type));

    if (!handler) {
      throw new Error(`No handler registered for notification type: ${type}`);
    }

    handler.show(message, options);
  }
}

// 4. EXTENDING WITHOUT MODIFYING EXISTING CODE
// New requirement: Toast notifications
class ToastNotificationHandler implements NotificationHandler {
  canHandle(type: string): boolean {
    return type === "toast";
  }

  show(message: string, options: NotificationOptions = {}): void {
    const toast = document.createElement("div");
    toast.className = "toast-notification";
    toast.style.cssText = `
      position: fixed;
      bottom: 20px;
      right: 20px;
      background: #333;
      color: white;
      padding: 12px 16px;
      border-radius: 8px;
      transform: translateX(100%);
      transition: transform 0.3s ease;
    `;
    toast.textContent = message;

    document.body.appendChild(toast);

    // Slide in animation
    setTimeout(() => {
      toast.style.transform = "translateX(0)";
    }, 10);

    // Auto remove
    setTimeout(() => {
      toast.style.transform = "translateX(100%)";
      setTimeout(() => toast.remove(), 300);
    }, options.duration || 4000);
  }
}

// New requirement: Modal notifications
class ModalNotificationHandler implements NotificationHandler {
  canHandle(type: string): boolean {
    return type === "modal";
  }

  show(message: string, options: NotificationOptions = {}): void {
    const overlay = document.createElement("div");
    overlay.className = "modal-overlay";
    overlay.style.cssText = `
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.5);
      display: flex;
      align-items: center;
      justify-content: center;
    `;

    const modal = document.createElement("div");
    modal.className = "modal-notification";
    modal.innerHTML = `
      <div class="modal-content">
        <h3>Notification</h3>
        <p>${message}</p>
        <div class="modal-actions">
          ${
            options.actions
              ?.map(
                (action) =>
                  `<button class="btn btn--${action.style || "primary"}" 
                     onclick="this.closest('.modal-overlay').remove(); (${
                       action.handler
                     })()"">
              ${action.label}
            </button>`
              )
              .join("") ||
            '<button class="btn btn--primary" onclick="this.closest(\'.modal-overlay\').remove()">OK</button>'
          }
        </div>
      </div>
    `;

    overlay.appendChild(modal);
    document.body.appendChild(overlay);
  }
}

// 5. USAGE - EXTENDING THE SYSTEM
const notificationService = new NotificationService();

// Extend with new handlers without modifying existing code
notificationService.registerHandler(new ToastNotificationHandler());
notificationService.registerHandler(new ModalNotificationHandler());

// Usage examples
notificationService.showNotification(
  "success",
  "Operation completed successfully!"
);
notificationService.showNotification(
  "error",
  "An error occurred. Please try again."
);
notificationService.showNotification("toast", "Settings saved");
notificationService.showNotification(
  "modal",
  "Are you sure you want to delete this item?",
  {
    actions: [
      {
        label: "Delete",
        style: "danger",
        handler: () => console.log("Deleting..."),
      },
      {
        label: "Cancel",
        style: "secondary",
        handler: () => console.log("Cancelled"),
      },
    ],
  }
);
```

**React Component Example Following OCP:**

```typescript
// Component that's open for extension
interface ButtonProps {
  children: React.ReactNode;
  variant?: string;
  size?: string;
  onClick?: () => void;
  disabled?: boolean;
}

// Base button component - closed for modification
const Button: React.FC<ButtonProps> = ({
  children,
  variant = "primary",
  size = "medium",
  onClick,
  disabled = false,
}) => {
  const buttonClass = `btn btn--${variant} btn--${size}`;

  return (
    <button className={buttonClass} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
};

// Extensions without modifying the base component
const PrimaryButton: React.FC<Omit<ButtonProps, "variant">> = (props) => (
  <Button {...props} variant="primary" />
);

const SecondaryButton: React.FC<Omit<ButtonProps, "variant">> = (props) => (
  <Button {...props} variant="secondary" />
);

const DangerButton: React.FC<Omit<ButtonProps, "variant">> = (props) => (
  <Button {...props} variant="danger" />
);

// New requirement: Loading button (extension, not modification)
interface LoadingButtonProps extends ButtonProps {
  loading?: boolean;
  loadingText?: string;
}

const LoadingButton: React.FC<LoadingButtonProps> = ({
  loading = false,
  loadingText = "Loading...",
  children,
  disabled,
  ...props
}) => (
  <Button {...props} disabled={disabled || loading}>
    {loading ? (
      <>
        <Spinner size="small" />
        {loadingText}
      </>
    ) : (
      children
    )}
  </Button>
);
```

**Benefits of Following OCP:**

- **Safer Changes:** Adding new functionality doesn't risk breaking existing features
- **Easier Testing:** New extensions can be tested independently
- **Better Collaboration:** Multiple developers can add features simultaneously
- **Reduced Regression Risk:** Existing functionality remains untouched
- **Plugin Architecture:** System becomes naturally extensible

### **L** - Liskov Substitution Principle (LSP)

**What it means:** Objects of a superclass should be replaceable with objects of a subclass without breaking the application. Subtypes must be substitutable for their base types.

**The Problem it Solves:**
When subclasses don't properly implement their parent's contract:

- Code that works with the parent class fails with subclasses
- Unexpected behavior occurs during runtime
- Testing becomes unreliable as substitution breaks assumptions
- Polymorphism becomes dangerous instead of helpful

**Real-World Example: Media Player Components**

**❌ Violating LSP - Subclass breaks parent contract:**

```typescript
// BAD: VideoPlayer breaks the contract expected by MediaPlayer
abstract class MediaPlayer {
  abstract play(): void;
  abstract pause(): void;
  abstract stop(): void;
  abstract getDuration(): number; // Returns duration in seconds
  abstract getCurrentTime(): number; // Returns current time in seconds

  // Method that depends on the contract
  getProgress(): number {
    const duration = this.getDuration();
    const currentTime = this.getCurrentTime();
    return duration > 0 ? (currentTime / duration) * 100 : 0;
  }

  canSkipToTime(timeInSeconds: number): boolean {
    return timeInSeconds >= 0 && timeInSeconds <= this.getDuration();
  }
}

class AudioPlayer extends MediaPlayer {
  private audio: HTMLAudioElement;

  constructor(src: string) {
    super();
    this.audio = new Audio(src);
  }

  play(): void {
    this.audio.play();
  }

  pause(): void {
    this.audio.pause();
  }

  stop(): void {
    this.audio.pause();
    this.audio.currentTime = 0;
  }

  getDuration(): number {
    return this.audio.duration || 0; // Correctly returns seconds
  }

  getCurrentTime(): number {
    return this.audio.currentTime || 0; // Correctly returns seconds
  }
}

// VIOLATES LSP: VideoPlayer returns different units
class VideoPlayer extends MediaPlayer {
  private video: HTMLVideoElement;

  constructor(src: string) {
    super();
    this.video = document.createElement("video");
    this.video.src = src;
  }

  play(): void {
    this.video.play();
  }

  pause(): void {
    this.video.pause();
  }

  stop(): void {
    this.video.pause();
    this.video.currentTime = 0;
  }

  // PROBLEM: Returns milliseconds instead of seconds!
  getDuration(): number {
    return (this.video.duration || 0) * 1000; // Returns milliseconds
  }

  // PROBLEM: Returns milliseconds instead of seconds!
  getCurrentTime(): number {
    return (this.video.currentTime || 0) * 1000; // Returns milliseconds
  }
}

// BREAKS when using VideoPlayer
function playMedia(player: MediaPlayer) {
  player.play();

  // This will be wrong for VideoPlayer because it returns milliseconds
  const progress = player.getProgress(); // Wrong calculation!
  console.log(`Progress: ${progress}%`);

  // This will fail for VideoPlayer
  const canSkip = player.canSkipToTime(30); // Expects seconds, but VideoPlayer uses milliseconds
  console.log(`Can skip to 30s: ${canSkip}`);
}

const audioPlayer = new AudioPlayer("song.mp3");
const videoPlayer = new VideoPlayer("movie.mp4");

playMedia(audioPlayer); // Works correctly
playMedia(videoPlayer); // BROKEN! Wrong calculations because of LSP violation
```

**✅ Following LSP - Proper substitution:**

```typescript
// GOOD: All implementations follow the same contract

// 1. WELL-DEFINED CONTRACT WITH CLEAR EXPECTATIONS
abstract class MediaPlayer {
  abstract play(): Promise<void>;
  abstract pause(): void;
  abstract stop(): void;
  abstract getDuration(): number; // ALWAYS returns seconds
  abstract getCurrentTime(): number; // ALWAYS returns seconds
  abstract setCurrentTime(timeInSeconds: number): void;
  abstract getVolume(): number; // ALWAYS returns 0-1
  abstract setVolume(volume: number): void; // ALWAYS accepts 0-1

  // These methods depend on the contract being followed
  getProgress(): number {
    const duration = this.getDuration();
    const currentTime = this.getCurrentTime();
    return duration > 0 ? (currentTime / duration) * 100 : 0;
  }

  canSkipToTime(timeInSeconds: number): boolean {
    return timeInSeconds >= 0 && timeInSeconds <= this.getDuration();
  }

  skipToPercentage(percentage: number): void {
    if (percentage < 0 || percentage > 100) {
      throw new Error("Percentage must be between 0 and 100");
    }

    const duration = this.getDuration();
    const targetTime = (duration * percentage) / 100;
    this.setCurrentTime(targetTime);
  }

  isMuted(): boolean {
    return this.getVolume() === 0;
  }
}

// 2. PROPER IMPLEMENTATIONS THAT FOLLOW THE CONTRACT
class AudioPlayer extends MediaPlayer {
  private audio: HTMLAudioElement;
  private loadingPromise: Promise<void>;

  constructor(src: string) {
    super();
    this.audio = new Audio(src);
    this.loadingPromise = this.initializeAudio();
  }

  private async initializeAudio(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.audio.addEventListener("loadedmetadata", () => resolve());
      this.audio.addEventListener("error", (e) => reject(e));
    });
  }

  async play(): Promise<void> {
    await this.loadingPromise;
    await this.audio.play();
  }

  pause(): void {
    this.audio.pause();
  }

  stop(): void {
    this.audio.pause();
    this.audio.currentTime = 0;
  }

  getDuration(): number {
    return this.audio.duration || 0; // Returns seconds as expected
  }

  getCurrentTime(): number {
    return this.audio.currentTime || 0; // Returns seconds as expected
  }

  setCurrentTime(timeInSeconds: number): void {
    this.audio.currentTime = timeInSeconds;
  }

  getVolume(): number {
    return this.audio.volume; // Returns 0-1 as expected
  }

  setVolume(volume: number): void {
    if (volume < 0 || volume > 1) {
      throw new Error("Volume must be between 0 and 1");
    }
    this.audio.volume = volume;
  }
}

class VideoPlayer extends MediaPlayer {
  private video: HTMLVideoElement;
  private loadingPromise: Promise<void>;

  constructor(src: string, container?: HTMLElement) {
    super();
    this.video = document.createElement("video");
    this.video.src = src;
    this.video.controls = false; // We control programmatically

    if (container) {
      container.appendChild(this.video);
    }

    this.loadingPromise = this.initializeVideo();
  }

  private async initializeVideo(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.video.addEventListener("loadedmetadata", () => resolve());
      this.video.addEventListener("error", (e) => reject(e));
    });
  }

  async play(): Promise<void> {
    await this.loadingPromise;
    await this.video.play();
  }

  pause(): void {
    this.video.pause();
  }

  stop(): void {
    this.video.pause();
    this.video.currentTime = 0;
  }

  getDuration(): number {
    return this.video.duration || 0; // Returns seconds as expected
  }

  getCurrentTime(): number {
    return this.video.currentTime || 0; // Returns seconds as expected
  }

  setCurrentTime(timeInSeconds: number): void {
    this.video.currentTime = timeInSeconds;
  }

  getVolume(): number {
    return this.video.volume; // Returns 0-1 as expected
  }

  setVolume(volume: number): void {
    if (volume < 0 || volume > 1) {
      throw new Error("Volume must be between 0 and 1");
    }
    this.video.volume = volume;
  }

  // Additional video-specific methods (not breaking the contract)
  getVideoElement(): HTMLVideoElement {
    return this.video;
  }

  setPlaybackRate(rate: number): void {
    this.video.playbackRate = rate;
  }
}

// 3. SPECIALIZED IMPLEMENTATIONS STILL FOLLOW THE CONTRACT
class StreamingPlayer extends MediaPlayer {
  private streamUrl: string;
  private currentTimeOffset: number = 0;
  private totalDuration: number = 0;
  private volume: number = 1;
  private isPlaying: boolean = false;

  constructor(streamUrl: string) {
    super();
    this.streamUrl = streamUrl;
    this.initializeStream();
  }

  private async initializeStream(): Promise<void> {
    // Initialize streaming connection
    const metadata = await this.fetchStreamMetadata();
    this.totalDuration = metadata.duration;
  }

  private async fetchStreamMetadata(): Promise<{ duration: number }> {
    // Simulate fetching stream metadata
    return new Promise((resolve) => {
      setTimeout(() => resolve({ duration: 180 }), 100);
    });
  }

  async play(): Promise<void> {
    // Start streaming
    this.isPlaying = true;
    console.log(`Starting stream: ${this.streamUrl}`);
  }

  pause(): void {
    this.isPlaying = false;
    console.log("Pausing stream");
  }

  stop(): void {
    this.isPlaying = false;
    this.currentTimeOffset = 0;
    console.log("Stopping stream");
  }

  getDuration(): number {
    return this.totalDuration; // Returns seconds as expected
  }

  getCurrentTime(): number {
    return this.currentTimeOffset; // Returns seconds as expected
  }

  setCurrentTime(timeInSeconds: number): void {
    this.currentTimeOffset = timeInSeconds;
  }

  getVolume(): number {
    return this.volume; // Returns 0-1 as expected
  }

  setVolume(volume: number): void {
    if (volume < 0 || volume > 1) {
      throw new Error("Volume must be between 0 and 1");
    }
    this.volume = volume;
  }
}

// 4. USAGE - ALL IMPLEMENTATIONS WORK IDENTICALLY
async function createPlaylist(players: MediaPlayer[]): Promise<void> {
  console.log("Creating playlist with", players.length, "items");

  for (const player of players) {
    // All these operations work correctly for ALL implementations
    console.log(`Duration: ${player.getDuration()} seconds`);
    console.log(`Can skip to 30s: ${player.canSkipToTime(30)}`);

    await player.play();

    // Skip to 25%
    player.skipToPercentage(25);
    console.log(`Progress after skip: ${player.getProgress()}%`);

    // Set volume to 50%
    player.setVolume(0.5);
    console.log(`Volume: ${player.getVolume()}`);

    player.pause();
    console.log("---");
  }
}

// Works with any combination of implementations
const players: MediaPlayer[] = [
  new AudioPlayer("song.mp3"),
  new VideoPlayer("movie.mp4"),
  new StreamingPlayer("https://stream.example.com/live"),
];

createPlaylist(players); // All work identically!
```

**React Component Example Following LSP:**

```typescript
// Base form field contract
interface FormFieldProps {
  label: string;
  value: string;
  onChange: (value: string) => void;
  error?: string;
  required?: boolean;
  disabled?: boolean;
}

// Base implementation
const FormField: React.FC<FormFieldProps & { children: React.ReactNode }> = ({
  label,
  error,
  required,
  children,
}) => (
  <div className="form-field">
    <label className="form-field__label">
      {label}
      {required && <span className="required">*</span>}
    </label>
    {children}
    {error && <div className="form-field__error">{error}</div>}
  </div>
);

// All these components can be substituted for FormField
const TextInput: React.FC<FormFieldProps> = ({
  label,
  value,
  onChange,
  error,
  required,
  disabled,
}) => (
  <FormField label={label} error={error} required={required}>
    <input
      type="text"
      value={value}
      onChange={(e) => onChange(e.target.value)}
      disabled={disabled}
      className="form-field__input"
    />
  </FormField>
);

const TextArea: React.FC<FormFieldProps & { rows?: number }> = ({
  label,
  value,
  onChange,
  error,
  required,
  disabled,
  rows = 4,
}) => (
  <FormField label={label} error={error} required={required}>
    <textarea
      value={value}
      onChange={(e) => onChange(e.target.value)}
      disabled={disabled}
      rows={rows}
      className="form-field__textarea"
    />
  </FormField>
);

const SelectInput: React.FC<
  FormFieldProps & { options: { value: string; label: string }[] }
> = ({ label, value, onChange, error, required, disabled, options }) => (
  <FormField label={label} error={error} required={required}>
    <select
      value={value}
      onChange={(e) => onChange(e.target.value)}
      disabled={disabled}
      className="form-field__select"
    >
      <option value="">Select...</option>
      {options.map((option) => (
        <option key={option.value} value={option.value}>
          {option.label}
        </option>
      ))}
    </select>
  </FormField>
);

// Usage - all components are interchangeable
const ContactForm: React.FC = () => {
  const [formData, setFormData] = useState({
    name: "",
    email: "",
    type: "",
    message: "",
  });

  const fields: Array<{
    component: React.ComponentType<any>;
    props: any;
  }> = [
    {
      component: TextInput,
      props: {
        label: "Name",
        value: formData.name,
        onChange: (value: string) =>
          setFormData((prev) => ({ ...prev, name: value })),
        required: true,
      },
    },
    {
      component: TextInput,
      props: {
        label: "Email",
        value: formData.email,
        onChange: (value: string) =>
          setFormData((prev) => ({ ...prev, email: value })),
        required: true,
      },
    },
    {
      component: SelectInput,
      props: {
        label: "Type",
        value: formData.type,
        onChange: (value: string) =>
          setFormData((prev) => ({ ...prev, type: value })),
        options: [
          { value: "general", label: "General Inquiry" },
          { value: "support", label: "Support" },
          { value: "sales", label: "Sales" },
        ],
      },
    },
    {
      component: TextArea,
      props: {
        label: "Message",
        value: formData.message,
        onChange: (value: string) =>
          setFormData((prev) => ({ ...prev, message: value })),
        required: true,
      },
    },
  ];

  return (
    <form>
      {fields.map((field, index) => (
        <field.component key={index} {...field.props} />
      ))}
    </form>
  );
};
```

### **I** - Interface Segregation Principle (ISP)

**What it means:** Clients should not be forced to depend on interfaces they don't use. Create specific, focused interfaces rather than large, monolithic ones.

**The Problem it Solves:**
Large interfaces force implementations to:

- Implement methods they don't need
- Handle dependencies they don't require
- Change when unrelated functionality changes
- Violate the single responsibility principle

**Real-World Example: Document Editor Components**

**❌ Violating ISP - Monolithic interface:**

```typescript
// BAD: Monolithic interface forces all implementations to handle everything
interface DocumentEditor {
  // Basic editing
  insertText(text: string, position: number): void;
  deleteText(start: number, end: number): void;
  replaceText(start: number, end: number, newText: string): void;

  // Formatting
  setBold(start: number, end: number): void;
  setItalic(start: number, end: number): void;
  setFontSize(start: number, end: number, size: number): void;
  setFontColor(start: number, end: number, color: string): void;

  // Advanced formatting
  insertTable(rows: number, cols: number): void;
  insertImage(url: string): void;
  insertLink(text: string, url: string): void;

  // Collaboration
  shareDocument(users: string[]): void;
  addComment(text: string, position: number): void;
  trackChanges(enabled: boolean): void;

  // Export/Import
  exportToPDF(): Blob;
  exportToWord(): Blob;
  importFromHTML(html: string): void;

  // Real-time features
  enableRealTimeSync(): void;
  broadcastChanges(changes: DocumentChange[]): void;
  receiveChanges(changes: DocumentChange[]): void;

  // Spell check
  enableSpellCheck(): void;
  getSuggestions(word: string): string[];
  addToDictionary(word: string): void;
}

// PROBLEMS: Simple implementations are forced to implement everything
class SimpleTextEditor implements DocumentEditor {
  private content: string = "";

  insertText(text: string, position: number): void {
    this.content =
      this.content.slice(0, position) + text + this.content.slice(position);
  }

  deleteText(start: number, end: number): void {
    this.content = this.content.slice(0, start) + this.content.slice(end);
  }

  replaceText(start: number, end: number, newText: string): void {
    this.content =
      this.content.slice(0, start) + newText + this.content.slice(end);
  }

  // FORCED TO IMPLEMENT THINGS IT DOESN'T SUPPORT
  setBold(start: number, end: number): void {
    throw new Error("Bold formatting not supported in simple text editor");
  }

  setItalic(start: number, end: number): void {
    throw new Error("Italic formatting not supported");
  }

  setFontSize(start: number, end: number, size: number): void {
    throw new Error("Font sizing not supported");
  }

  setFontColor(start: number, end: number, color: string): void {
    throw new Error("Color formatting not supported");
  }

  insertTable(rows: number, cols: number): void {
    throw new Error("Tables not supported");
  }

  insertImage(url: string): void {
    throw new Error("Images not supported");
  }

  insertLink(text: string, url: string): void {
    throw new Error("Links not supported");
  }

  shareDocument(users: string[]): void {
    throw new Error("Sharing not supported");
  }

  addComment(text: string, position: number): void {
    throw new Error("Comments not supported");
  }

  trackChanges(enabled: boolean): void {
    throw new Error("Change tracking not supported");
  }

  exportToPDF(): Blob {
    throw new Error("PDF export not supported");
  }

  exportToWord(): Blob {
    throw new Error("Word export not supported");
  }

  importFromHTML(html: string): void {
    throw new Error("HTML import not supported");
  }

  enableRealTimeSync(): void {
    throw new Error("Real-time sync not supported");
  }

  broadcastChanges(changes: DocumentChange[]): void {
    throw new Error("Broadcasting not supported");
  }

  receiveChanges(changes: DocumentChange[]): void {
    throw new Error("Receiving changes not supported");
  }

  enableSpellCheck(): void {
    throw new Error("Spell check not supported");
  }

  getSuggestions(word: string): string[] {
    throw new Error("Spell suggestions not supported");
  }

  addToDictionary(word: string): void {
    throw new Error("Dictionary not supported");
  }
}
```

**✅ Following ISP - Segregated interfaces:**

```typescript
// GOOD: Small, focused interfaces

// 1. CORE EDITING INTERFACE
interface TextEditor {
  insertText(text: string, position: number): void;
  deleteText(start: number, end: number): void;
  replaceText(start: number, end: number, newText: string): void;
  getContent(): string;
  setContent(content: string): void;
}

// 2. FORMATTING CAPABILITIES
interface TextFormatter {
  setBold(start: number, end: number): void;
  setItalic(start: number, end: number): void;
  setFontSize(start: number, end: number, size: number): void;
  setFontColor(start: number, end: number, color: string): void;
  clearFormatting(start: number, end: number): void;
}

// 3. ADVANCED CONTENT INSERTION
interface AdvancedContentInserter {
  insertTable(rows: number, cols: number, position: number): void;
  insertImage(url: string, position: number): void;
  insertLink(text: string, url: string, position: number): void;
}

// 4. COLLABORATION FEATURES
interface CollaborationProvider {
  shareDocument(users: string[]): void;
  addComment(text: string, position: number): void;
  trackChanges(enabled: boolean): void;
  getCollaborators(): string[];
}

// 5. EXPORT/IMPORT CAPABILITIES
interface DocumentExporter {
  exportToPDF(): Blob;
  exportToWord(): Blob;
  exportToHTML(): string;
}

interface DocumentImporter {
  importFromHTML(html: string): void;
  importFromWord(file: File): void;
  importFromText(text: string): void;
}

// 6. REAL-TIME SYNCHRONIZATION
interface RealTimeSync {
  enableRealTimeSync(): void;
  disableRealTimeSync(): void;
  broadcastChanges(changes: DocumentChange[]): void;
  receiveChanges(changes: DocumentChange[]): void;
}

// 7. SPELL CHECKING
interface SpellChecker {
  enableSpellCheck(): void;
  disableSpellCheck(): void;
  getSuggestions(word: string): string[];
  addToDictionary(word: string): void;
  checkDocument(): SpellCheckResult[];
}

// 8. IMPLEMENTATIONS ONLY IMPLEMENT WHAT THEY NEED

// Simple text editor - only basic editing
class SimpleTextEditor implements TextEditor {
  private content: string = "";

  insertText(text: string, position: number): void {
    this.content =
      this.content.slice(0, position) + text + this.content.slice(position);
  }

  deleteText(start: number, end: number): void {
    this.content = this.content.slice(0, start) + this.content.slice(end);
  }

  replaceText(start: number, end: number, newText: string): void {
    this.content =
      this.content.slice(0, start) + newText + this.content.slice(end);
  }

  getContent(): string {
    return this.content;
  }

  setContent(content: string): void {
    this.content = content;
  }
}

// Rich text editor - editing + formatting
class RichTextEditor implements TextEditor, TextFormatter {
  private content: string = "";
  private formatting: Map<string, FormatInfo> = new Map();

  // TextEditor implementation
  insertText(text: string, position: number): void {
    this.content =
      this.content.slice(0, position) + text + this.content.slice(position);
  }

  deleteText(start: number, end: number): void {
    this.content = this.content.slice(0, start) + this.content.slice(end);
    this.updateFormattingAfterDeletion(start, end);
  }

  replaceText(start: number, end: number, newText: string): void {
    this.content =
      this.content.slice(0, start) + newText + this.content.slice(end);
    this.updateFormattingAfterReplacement(start, end, newText.length);
  }

  getContent(): string {
    return this.content;
  }

  setContent(content: string): void {
    this.content = content;
    this.formatting.clear();
  }

  // TextFormatter implementation
  setBold(start: number, end: number): void {
    this.applyFormatting(start, end, { bold: true });
  }

  setItalic(start: number, end: number): void {
    this.applyFormatting(start, end, { italic: true });
  }

  setFontSize(start: number, end: number, size: number): void {
    this.applyFormatting(start, end, { fontSize: size });
  }

  setFontColor(start: number, end: number, color: string): void {
    this.applyFormatting(start, end, { color });
  }

  clearFormatting(start: number, end: number): void {
    for (let i = start; i < end; i++) {
      this.formatting.delete(i.toString());
    }
  }

  private applyFormatting(
    start: number,
    end: number,
    format: Partial<FormatInfo>
  ): void {
    for (let i = start; i < end; i++) {
      const key = i.toString();
      const existing = this.formatting.get(key) || {};
      this.formatting.set(key, { ...existing, ...format });
    }
  }

  private updateFormattingAfterDeletion(start: number, end: number): void {
    // Update formatting positions after deletion
  }

  private updateFormattingAfterReplacement(
    start: number,
    end: number,
    newLength: number
  ): void {
    // Update formatting positions after replacement
  }
}

// Full-featured editor - all interfaces
class FullFeaturedEditor
  implements
    TextEditor,
    TextFormatter,
    AdvancedContentInserter,
    DocumentExporter,
    DocumentImporter
{
  private content: string = "";
  private formatting: Map<string, FormatInfo> = new Map();
  private tables: Table[] = [];
  private images: ImageElement[] = [];
  private links: LinkElement[] = [];

  // TextEditor implementation
  insertText(text: string, position: number): void {
    this.content =
      this.content.slice(0, position) + text + this.content.slice(position);
  }

  deleteText(start: number, end: number): void {
    this.content = this.content.slice(0, start) + this.content.slice(end);
  }

  replaceText(start: number, end: number, newText: string): void {
    this.content =
      this.content.slice(0, start) + newText + this.content.slice(end);
  }

  getContent(): string {
    return this.content;
  }

  setContent(content: string): void {
    this.content = content;
  }

  // TextFormatter implementation
  setBold(start: number, end: number): void {
    this.applyFormatting(start, end, { bold: true });
  }

  setItalic(start: number, end: number): void {
    this.applyFormatting(start, end, { italic: true });
  }

  setFontSize(start: number, end: number, size: number): void {
    this.applyFormatting(start, end, { fontSize: size });
  }

  setFontColor(start: number, end: number, color: string): void {
    this.applyFormatting(start, end, { color });
  }

  clearFormatting(start: number, end: number): void {
    for (let i = start; i < end; i++) {
      this.formatting.delete(i.toString());
    }
  }

  // AdvancedContentInserter implementation
  insertTable(rows: number, cols: number, position: number): void {
    const table = new Table(rows, cols);
    this.tables.push(table);
    this.insertText(`[TABLE:${table.id}]`, position);
  }

  insertImage(url: string, position: number): void {
    const image = new ImageElement(url);
    this.images.push(image);
    this.insertText(`[IMAGE:${image.id}]`, position);
  }

  insertLink(text: string, url: string, position: number): void {
    const link = new LinkElement(text, url);
    this.links.push(link);
    this.insertText(text, position);
  }

  // DocumentExporter implementation
  exportToPDF(): Blob {
    // Convert content to PDF
    return new Blob(["PDF content"], { type: "application/pdf" });
  }

  exportToWord(): Blob {
    // Convert content to Word format
    return new Blob(["Word content"], {
      type: "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
    });
  }

  exportToHTML(): string {
    // Convert content to HTML
    return `<html><body>${this.content}</body></html>`;
  }

  // DocumentImporter implementation
  importFromHTML(html: string): void {
    // Parse HTML and set content
    this.content = this.extractTextFromHTML(html);
  }

  importFromWord(file: File): void {
    // Parse Word document and set content
  }

  importFromText(text: string): void {
    this.setContent(text);
  }

  private applyFormatting(
    start: number,
    end: number,
    format: Partial<FormatInfo>
  ): void {
    // Implementation details
  }

  private extractTextFromHTML(html: string): string {
    // Implementation details
    return html.replace(/<[^>]*>/g, "");
  }
}

// 9. USAGE WITH DEPENDENCY INJECTION
class EditorManager {
  constructor(
    private editor: TextEditor,
    private formatter?: TextFormatter,
    private exporter?: DocumentExporter,
    private collaborationProvider?: CollaborationProvider
  ) {}

  // Only use features that are available
  formatSelection(start: number, end: number): void {
    if (this.formatter) {
      this.formatter.setBold(start, end);
    } else {
      console.log("Formatting not available in this editor");
    }
  }

  exportDocument(): Blob | null {
    if (this.exporter) {
      return this.exporter.exportToPDF();
    } else {
      console.log("Export not available in this editor");
      return null;
    }
  }

  shareWithTeam(users: string[]): void {
    if (this.collaborationProvider) {
      this.collaborationProvider.shareDocument(users);
    } else {
      console.log("Collaboration not available in this editor");
    }
  }
}

// Different configurations for different use cases
const simpleManager = new EditorManager(new SimpleTextEditor());

const richManager = new EditorManager(
  new RichTextEditor(),
  new RichTextEditor() // Same instance implements both interfaces
);

const fullManager = new EditorManager(
  new FullFeaturedEditor(),
  new FullFeaturedEditor(),
  new FullFeaturedEditor(),
  new CollaborationService() // Separate collaboration implementation
);
```

### **D** - Dependency Inversion Principle (DIP)

**What it means:** High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.

**The Problem it Solves:**
When high-level code directly depends on low-level implementations:

- Code becomes tightly coupled and hard to change
- Testing becomes difficult because of concrete dependencies
- Different implementations can't be swapped easily
- Changes in low-level code break high-level code

**Real-World Example: E-commerce Order Processing**

**❌ Violating DIP - High-level code depends on concrete implementations:**

```typescript
// BAD: High-level OrderProcessor directly depends on concrete implementations
class EmailService {
  sendEmail(to: string, subject: string, body: string): void {
    console.log(`Sending email to ${to}: ${subject}`);
    // Direct dependency on specific email provider
    fetch("https://api.sendgrid.com/v3/mail/send", {
      method: "POST",
      headers: { Authorization: "Bearer sg_api_key" },
      body: JSON.stringify({ to, subject, content: body }),
    });
  }
}

class SMSService {
  sendSMS(to: string, message: string): void {
    console.log(`Sending SMS to ${to}: ${message}`);
    // Direct dependency on specific SMS provider
    fetch("https://api.twilio.com/2010-04-01/Accounts/AC123/Messages.json", {
      method: "POST",
      headers: { Authorization: "Basic twilio_auth" },
      body: new URLSearchParams({ To: to, Body: message }),
    });
  }
}

class PaymentProcessor {
  processPayment(amount: number, cardToken: string): boolean {
    console.log(`Processing payment of $${amount}`);
    // Direct dependency on specific payment provider
    const response = fetch("https://api.stripe.com/v1/charges", {
      method: "POST",
      headers: { Authorization: "Bearer sk_stripe_key" },
      body: new URLSearchParams({
        amount: amount.toString(),
        source: cardToken,
      }),
    });
    return true; // Simplified
  }
}

class InventoryManager {
  updateStock(productId: string, quantity: number): void {
    console.log(`Updating stock for ${productId}: ${quantity}`);
    // Direct database dependency
    const connection = mysql.createConnection({
      host: "localhost",
      user: "root",
      password: "password",
      database: "inventory",
    });
    connection.query(
      `UPDATE products SET stock = stock - ${quantity} WHERE id = '${productId}'`
    );
  }
}

// HIGH-LEVEL MODULE WITH MANY CONCRETE DEPENDENCIES
class OrderProcessor {
  private emailService: EmailService;
  private smsService: SMSService;
  private paymentProcessor: PaymentProcessor;
  private inventoryManager: InventoryManager;

  constructor() {
    // PROBLEM: Direct instantiation of concrete classes
    this.emailService = new EmailService();
    this.smsService = new SMSService();
    this.paymentProcessor = new PaymentProcessor();
    this.inventoryManager = new InventoryManager();
  }

  async processOrder(order: Order): Promise<void> {
    try {
      // Process payment
      const paymentSuccess = this.paymentProcessor.processPayment(
        order.total,
        order.paymentToken
      );

      if (!paymentSuccess) {
        throw new Error("Payment failed");
      }

      // Update inventory
      for (const item of order.items) {
        this.inventoryManager.updateStock(item.productId, item.quantity);
      }

      // Send notifications
      this.emailService.sendEmail(
        order.customer.email,
        "Order Confirmation",
        `Your order ${order.id} has been confirmed`
      );

      if (order.customer.phone) {
        this.smsService.sendSMS(
          order.customer.phone,
          `Order ${order.id} confirmed. Total: $${order.total}`
        );
      }
    } catch (error) {
      // Error handling also tightly coupled
      this.emailService.sendEmail(
        order.customer.email,
        "Order Failed",
        `Your order ${order.id} could not be processed`
      );
    }
  }
}

// PROBLEMS:
// 1. Can't test OrderProcessor without real email/SMS/payment services
// 2. Can't switch to different providers without changing OrderProcessor code
// 3. Changes in any concrete service might break OrderProcessor
// 4. Hard to add new notification methods or payment processors
```

**✅ Following DIP - Depend on abstractions:**

```typescript
// GOOD: Define abstractions (interfaces) first

// 1. ABSTRACT INTERFACES (HIGH-LEVEL CONTRACTS)
interface NotificationService {
  send(recipient: string, message: NotificationMessage): Promise<void>;
}

interface PaymentService {
  processPayment(paymentRequest: PaymentRequest): Promise<PaymentResult>;
}

interface InventoryService {
  updateStock(productId: string, quantity: number): Promise<void>;
  checkAvailability(productId: string, quantity: number): Promise<boolean>;
}

interface Logger {
  info(message: string, context?: any): void;
  error(message: string, error?: Error): void;
  warn(message: string, context?: any): void;
}

// 2. DOMAIN MODELS
interface NotificationMessage {
  subject?: string;
  content: string;
  type: "email" | "sms" | "push";
  urgent?: boolean;
}

interface PaymentRequest {
  amount: number;
  currency: string;
  paymentMethod: string;
  customerId?: string;
  metadata?: Record<string, any>;
}

interface PaymentResult {
  success: boolean;
  transactionId?: string;
  error?: string;
}

interface Order {
  id: string;
  customer: Customer;
  items: OrderItem[];
  total: number;
  paymentToken: string;
}

interface Customer {
  id: string;
  email: string;
  phone?: string;
  preferences: CustomerPreferences;
}

interface CustomerPreferences {
  emailNotifications: boolean;
  smsNotifications: boolean;
  pushNotifications: boolean;
}

// 3. HIGH-LEVEL MODULE DEPENDS ON ABSTRACTIONS
class OrderProcessor {
  constructor(
    private paymentService: PaymentService,
    private inventoryService: InventoryService,
    private notificationService: NotificationService,
    private logger: Logger
  ) {
    // Dependencies are injected, not created
  }

  async processOrder(order: Order): Promise<OrderResult> {
    this.logger.info("Starting order processing", { orderId: order.id });

    try {
      // Check inventory availability
      await this.verifyInventory(order);

      // Process payment
      const paymentResult = await this.paymentService.processPayment({
        amount: order.total,
        currency: "USD",
        paymentMethod: order.paymentToken,
        customerId: order.customer.id,
        metadata: { orderId: order.id },
      });

      if (!paymentResult.success) {
        throw new OrderProcessingError("Payment failed", paymentResult.error);
      }

      // Update inventory
      await this.updateInventoryForOrder(order);

      // Send notifications
      await this.sendOrderConfirmation(order, paymentResult.transactionId!);

      this.logger.info("Order processed successfully", {
        orderId: order.id,
        transactionId: paymentResult.transactionId,
      });

      return {
        success: true,
        orderId: order.id,
        transactionId: paymentResult.transactionId!,
      };
    } catch (error) {
      this.logger.error("Order processing failed", error);
      await this.handleOrderFailure(order, error);

      return {
        success: false,
        orderId: order.id,
        error: error.message,
      };
    }
  }

  private async verifyInventory(order: Order): Promise<void> {
    for (const item of order.items) {
      const available = await this.inventoryService.checkAvailability(
        item.productId,
        item.quantity
      );

      if (!available) {
        throw new OrderProcessingError(
          `Insufficient inventory for product ${item.productId}`
        );
      }
    }
  }

  private async updateInventoryForOrder(order: Order): Promise<void> {
    for (const item of order.items) {
      await this.inventoryService.updateStock(item.productId, item.quantity);
    }
  }

  private async sendOrderConfirmation(
    order: Order,
    transactionId: string
  ): Promise<void> {
    const { customer } = order;

    // Send email if customer wants email notifications
    if (customer.preferences.emailNotifications) {
      await this.notificationService.send(customer.email, {
        type: "email",
        subject: `Order Confirmation - ${order.id}`,
        content: `Your order ${order.id} has been confirmed. Transaction ID: ${transactionId}. Total: $${order.total}`,
      });
    }

    // Send SMS if customer has phone and wants SMS notifications
    if (customer.phone && customer.preferences.smsNotifications) {
      await this.notificationService.send(customer.phone, {
        type: "sms",
        content: `Order ${order.id} confirmed. Total: $${order.total}`,
      });
    }
  }

  private async handleOrderFailure(order: Order, error: Error): Promise<void> {
    if (order.customer.preferences.emailNotifications) {
      await this.notificationService.send(order.customer.email, {
        type: "email",
        subject: `Order Failed - ${order.id}`,
        content: `We were unable to process your order ${order.id}. Please try again or contact support.`,
      });
    }
  }
}

// 4. CONCRETE IMPLEMENTATIONS (LOW-LEVEL MODULES)
class EmailNotificationService implements NotificationService {
  constructor(private apiKey: string, private baseUrl: string) {}

  async send(recipient: string, message: NotificationMessage): Promise<void> {
    if (message.type !== "email") {
      throw new Error("This service only handles email notifications");
    }

    const response = await fetch(`${this.baseUrl}/send`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${this.apiKey}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        to: recipient,
        subject: message.subject,
        content: message.content,
      }),
    });

    if (!response.ok) {
      throw new Error(`Failed to send email: ${response.statusText}`);
    }
  }
}

class SMSNotificationService implements NotificationService {
  constructor(private accountSid: string, private authToken: string) {}

  async send(recipient: string, message: NotificationMessage): Promise<void> {
    if (message.type !== "sms") {
      throw new Error("This service only handles SMS notifications");
    }

    const response = await fetch(
      `https://api.twilio.com/2010-04-01/Accounts/${this.accountSid}/Messages.json`,
      {
        method: "POST",
        headers: {
          Authorization: `Basic ${btoa(
            `${this.accountSid}:${this.authToken}`
          )}`,
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: new URLSearchParams({
          To: recipient,
          Body: message.content,
          From: "+1234567890",
        }),
      }
    );

    if (!response.ok) {
      throw new Error(`Failed to send SMS: ${response.statusText}`);
    }
  }
}

// Composite notification service that handles multiple types
class CompositeNotificationService implements NotificationService {
  private services = new Map<string, NotificationService>();

  register(type: string, service: NotificationService): void {
    this.services.set(type, service);
  }

  async send(recipient: string, message: NotificationMessage): Promise<void> {
    const service = this.services.get(message.type);
    if (!service) {
      throw new Error(
        `No service registered for notification type: ${message.type}`
      );
    }

    await service.send(recipient, message);
  }
}

class StripePaymentService implements PaymentService {
  constructor(private secretKey: string) {}

  async processPayment(request: PaymentRequest): Promise<PaymentResult> {
    try {
      const response = await fetch("https://api.stripe.com/v1/charges", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${this.secretKey}`,
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: new URLSearchParams({
          amount: (request.amount * 100).toString(), // Stripe uses cents
          currency: request.currency,
          source: request.paymentMethod,
          metadata: JSON.stringify(request.metadata || {}),
        }),
      });

      const result = await response.json();

      if (response.ok) {
        return {
          success: true,
          transactionId: result.id,
        };
      } else {
        return {
          success: false,
          error: result.error?.message || "Payment failed",
        };
      }
    } catch (error) {
      return {
        success: false,
        error: error.message,
      };
    }
  }
}

class DatabaseInventoryService implements InventoryService {
  constructor(private database: Database) {}

  async updateStock(productId: string, quantity: number): Promise<void> {
    await this.database.query(
      "UPDATE products SET stock = stock - ? WHERE id = ?",
      [quantity, productId]
    );
  }

  async checkAvailability(
    productId: string,
    quantity: number
  ): Promise<boolean> {
    const result = await this.database.query(
      "SELECT stock FROM products WHERE id = ?",
      [productId]
    );

    return result[0]?.stock >= quantity;
  }
}

class ConsoleLogger implements Logger {
  info(message: string, context?: any): void {
    console.log(`[INFO] ${message}`, context || "");
  }

  error(message: string, error?: Error): void {
    console.error(`[ERROR] ${message}`, error || "");
  }

  warn(message: string, context?: any): void {
    console.warn(`[WARN] ${message}`, context || "");
  }
}

// 5. DEPENDENCY INJECTION AND CONFIGURATION
class OrderProcessingFactory {
  static createOrderProcessor(config: OrderProcessingConfig): OrderProcessor {
    // Create concrete implementations
    const emailService = new EmailNotificationService(
      config.email.apiKey,
      config.email.baseUrl
    );

    const smsService = new SMSNotificationService(
      config.sms.accountSid,
      config.sms.authToken
    );

    // Composite notification service
    const notificationService = new CompositeNotificationService();
    notificationService.register("email", emailService);
    notificationService.register("sms", smsService);

    const paymentService = new StripePaymentService(config.stripe.secretKey);
    const inventoryService = new DatabaseInventoryService(config.database);
    const logger = new ConsoleLogger();

    // Inject dependencies into high-level module
    return new OrderProcessor(
      paymentService,
      inventoryService,
      notificationService,
      logger
    );
  }

  // For testing - inject mocks
  static createTestOrderProcessor(
    mockPaymentService: PaymentService,
    mockInventoryService: InventoryService,
    mockNotificationService: NotificationService,
    mockLogger: Logger
  ): OrderProcessor {
    return new OrderProcessor(
      mockPaymentService,
      mockInventoryService,
      mockNotificationService,
      mockLogger
    );
  }
}

// 6. USAGE
const config: OrderProcessingConfig = {
  email: { apiKey: "sg_key", baseUrl: "https://api.sendgrid.com" },
  sms: { accountSid: "AC123", authToken: "auth_token" },
  stripe: { secretKey: "sk_stripe_key" },
  database: databaseConnection,
};

const orderProcessor = OrderProcessingFactory.createOrderProcessor(config);

// Easy to test with mocks
const testProcessor = OrderProcessingFactory.createTestOrderProcessor(
  mockPaymentService,
  mockInventoryService,
  mockNotificationService,
  mockLogger
);
```

**Benefits of Following SOLID Principles:**

- **Maintainability:** Changes are isolated to specific areas
- **Testability:** Each component can be tested independently
- **Flexibility:** Easy to swap implementations and add new features
- **Reusability:** Components can be reused in different contexts
- **Team Collaboration:** Developers can work on different parts independently
- **Debugging:** Problems are easier to locate and fix

---

## 🎯 Essential Design Patterns for Frontend Architecture {#design-patterns}

Design patterns provide proven solutions to common programming problems. In frontend development, these patterns help create maintainable, scalable, and testable applications.

### **👁️ Observer Pattern - Event-Driven Architecture**

**What it solves:** When multiple parts of your application need to react to changes in one component, without creating tight coupling between them.

**Real-World Example: Shopping Cart State Management**

```typescript
// 1. OBSERVER INTERFACE
interface Observer<T> {
  update(data: T): void;
  getId(): string;
}

// 2. SUBJECT INTERFACE
interface Subject<T> {
  subscribe(observer: Observer<T>): void;
  unsubscribe(observerId: string): void;
  notify(data: T): void;
}

// 3. SHOPPING CART IMPLEMENTATION
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
  itemCount: number;
}

class ShoppingCart implements Subject<CartState> {
  private items: CartItem[] = [];
  private observers: Map<string, Observer<CartState>> = new Map();

  subscribe(observer: Observer<CartState>): void {
    this.observers.set(observer.getId(), observer);
    // Immediately notify new observer of current state
    observer.update(this.getState());
  }

  unsubscribe(observerId: string): void {
    this.observers.delete(observerId);
  }

  notify(data: CartState): void {
    this.observers.forEach((observer) => observer.update(data));
  }

  addItem(item: Omit<CartItem, "quantity">): void {
    const existingItem = this.items.find((i) => i.id === item.id);

    if (existingItem) {
      existingItem.quantity += 1;
    } else {
      this.items.push({ ...item, quantity: 1 });
    }

    this.notify(this.getState());
  }

  removeItem(itemId: string): void {
    this.items = this.items.filter((item) => item.id !== itemId);
    this.notify(this.getState());
  }

  updateQuantity(itemId: string, quantity: number): void {
    const item = this.items.find((i) => i.id === itemId);
    if (item) {
      item.quantity = Math.max(0, quantity);
      if (item.quantity === 0) {
        this.removeItem(itemId);
      } else {
        this.notify(this.getState());
      }
    }
  }

  clearCart(): void {
    this.items = [];
    this.notify(this.getState());
  }

  private getState(): CartState {
    const total = this.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
    const itemCount = this.items.reduce((sum, item) => sum + item.quantity, 0);

    return {
      items: [...this.items],
      total,
      itemCount,
    };
  }
}

// 4. OBSERVER IMPLEMENTATIONS
class CartDisplayObserver implements Observer<CartState> {
  constructor(private elementId: string) {}

  getId(): string {
    return `cart-display-${this.elementId}`;
  }

  update(cartState: CartState): void {
    const element = document.getElementById(this.elementId);
    if (element) {
      element.innerHTML = `
        <h3>Shopping Cart (${cartState.itemCount} items)</h3>
        <div class="cart-items">
          ${cartState.items
            .map(
              (item) => `
            <div class="cart-item">
              <span>${item.name}</span>
              <span>$${item.price} x ${item.quantity}</span>
              <span>$${(item.price * item.quantity).toFixed(2)}</span>
            </div>
          `
            )
            .join("")}
        </div>
        <div class="cart-total">
          <strong>Total: $${cartState.total.toFixed(2)}</strong>
        </div>
      `;
    }
  }
}

class CartCounterObserver implements Observer<CartState> {
  constructor(private elementId: string) {}

  getId(): string {
    return `cart-counter-${this.elementId}`;
  }

  update(cartState: CartState): void {
    const element = document.getElementById(this.elementId);
    if (element) {
      element.textContent = cartState.itemCount.toString();
      element.className = `cart-counter ${
        cartState.itemCount > 0 ? "has-items" : ""
      }`;
    }
  }
}

class CartAnalyticsObserver implements Observer<CartState> {
  getId(): string {
    return "cart-analytics";
  }

  update(cartState: CartState): void {
    // Send analytics data
    console.log("Analytics: Cart updated", {
      itemCount: cartState.itemCount,
      totalValue: cartState.total,
      timestamp: Date.now(),
    });

    // Could send to analytics service
    // analytics.track('cart_updated', { ... });
  }
}

// 5. REACT HOOK IMPLEMENTATION
function useCartObserver(): [CartState, ShoppingCart] {
  const [cartState, setCartState] = useState<CartState>({
    items: [],
    total: 0,
    itemCount: 0,
  });

  const cartRef = useRef<ShoppingCart>();
  const observerRef = useRef<Observer<CartState>>();

  useEffect(() => {
    // Create cart instance (could be singleton)
    cartRef.current = new ShoppingCart();

    // Create observer for React component
    observerRef.current = {
      getId: () => "react-cart-observer",
      update: (state: CartState) => setCartState(state),
    };

    // Subscribe to cart updates
    cartRef.current.subscribe(observerRef.current);

    return () => {
      if (cartRef.current && observerRef.current) {
        cartRef.current.unsubscribe(observerRef.current.getId());
      }
    };
  }, []);

  return [cartState, cartRef.current!];
}

// 6. USAGE EXAMPLES
const cart = new ShoppingCart();

// Register multiple observers
const cartDisplay = new CartDisplayObserver("cart-sidebar");
const headerCounter = new CartCounterObserver("header-cart-count");
const analytics = new CartAnalyticsObserver();

cart.subscribe(cartDisplay);
cart.subscribe(headerCounter);
cart.subscribe(analytics);

// Any cart operation automatically updates all observers
cart.addItem({ id: "1", name: "Laptop", price: 999.99 });
cart.addItem({ id: "2", name: "Mouse", price: 29.99 });
cart.updateQuantity("1", 2);
```

### **🎛️ Strategy Pattern - Interchangeable Algorithms**

**What it solves:** When you have multiple ways to perform a task and want to switch between them dynamically.

**Real-World Example: Payment Processing System**

```typescript
// 1. STRATEGY INTERFACE
interface PaymentStrategy {
  processPayment(
    amount: number,
    paymentDetails: PaymentDetails
  ): Promise<PaymentResult>;
  validatePaymentDetails(details: PaymentDetails): ValidationResult;
  getPaymentMethodName(): string;
  getSupportedCurrencies(): string[];
}

// 2. PAYMENT MODELS
interface PaymentDetails {
  type: "credit_card" | "paypal" | "apple_pay" | "crypto";
  data: any;
}

interface PaymentResult {
  success: boolean;
  transactionId?: string;
  error?: string;
  processingFee?: number;
}

interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

// 3. CONCRETE STRATEGIES
class CreditCardStrategy implements PaymentStrategy {
  getPaymentMethodName(): string {
    return "Credit Card";
  }

  getSupportedCurrencies(): string[] {
    return ["USD", "EUR", "GBP", "CAD"];
  }

  validatePaymentDetails(details: PaymentDetails): ValidationResult {
    const errors: string[] = [];
    const { cardNumber, expiryDate, cvv, holderName } = details.data;

    if (!cardNumber || cardNumber.length < 13) {
      errors.push("Invalid card number");
    }

    if (!expiryDate || !/^\d{2}\/\d{2}$/.test(expiryDate)) {
      errors.push("Invalid expiry date format (MM/YY)");
    }

    if (!cvv || cvv.length < 3) {
      errors.push("Invalid CVV");
    }

    if (!holderName || holderName.trim().length < 2) {
      errors.push("Invalid cardholder name");
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  async processPayment(
    amount: number,
    details: PaymentDetails
  ): Promise<PaymentResult> {
    const validation = this.validatePaymentDetails(details);
    if (!validation.isValid) {
      return {
        success: false,
        error: validation.errors.join(", "),
      };
    }

    try {
      // Simulate credit card processing
      const response = await this.callCreditCardAPI(amount, details.data);

      return {
        success: true,
        transactionId: response.transactionId,
        processingFee: amount * 0.029, // 2.9% fee
      };
    } catch (error) {
      return {
        success: false,
        error: "Credit card processing failed",
      };
    }
  }

  private async callCreditCardAPI(amount: number, cardData: any): Promise<any> {
    // Simulate API call
    await new Promise((resolve) => setTimeout(resolve, 1000));
    return {
      transactionId: `cc_${Date.now()}_${Math.random()
        .toString(36)
        .substr(2, 9)}`,
    };
  }
}

class PayPalStrategy implements PaymentStrategy {
  getPaymentMethodName(): string {
    return "PayPal";
  }

  getSupportedCurrencies(): string[] {
    return ["USD", "EUR", "GBP", "CAD", "AUD", "JPY"];
  }

  validatePaymentDetails(details: PaymentDetails): ValidationResult {
    const errors: string[] = [];
    const { email, password } = details.data;

    if (!email || !/\S+@\S+\.\S+/.test(email)) {
      errors.push("Invalid PayPal email");
    }

    if (!password || password.length < 6) {
      errors.push("PayPal password required");
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  async processPayment(
    amount: number,
    details: PaymentDetails
  ): Promise<PaymentResult> {
    const validation = this.validatePaymentDetails(details);
    if (!validation.isValid) {
      return {
        success: false,
        error: validation.errors.join(", "),
      };
    }

    try {
      const response = await this.callPayPalAPI(amount, details.data);

      return {
        success: true,
        transactionId: response.transactionId,
        processingFee: amount * 0.034, // 3.4% fee
      };
    } catch (error) {
      return {
        success: false,
        error: "PayPal processing failed",
      };
    }
  }

  private async callPayPalAPI(amount: number, paypalData: any): Promise<any> {
    await new Promise((resolve) => setTimeout(resolve, 800));
    return {
      transactionId: `pp_${Date.now()}_${Math.random()
        .toString(36)
        .substr(2, 9)}`,
    };
  }
}

class CryptoStrategy implements PaymentStrategy {
  getPaymentMethodName(): string {
    return "Cryptocurrency";
  }

  getSupportedCurrencies(): string[] {
    return ["BTC", "ETH", "LTC", "USD"]; // USD for conversion
  }

  validatePaymentDetails(details: PaymentDetails): ValidationResult {
    const errors: string[] = [];
    const { walletAddress, currency, privateKey } = details.data;

    if (!walletAddress || walletAddress.length < 20) {
      errors.push("Invalid wallet address");
    }

    if (!currency || !this.getSupportedCurrencies().includes(currency)) {
      errors.push("Unsupported cryptocurrency");
    }

    if (!privateKey) {
      errors.push("Private key required for signing");
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  async processPayment(
    amount: number,
    details: PaymentDetails
  ): Promise<PaymentResult> {
    const validation = this.validatePaymentDetails(details);
    if (!validation.isValid) {
      return {
        success: false,
        error: validation.errors.join(", "),
      };
    }

    try {
      const response = await this.processCryptoTransaction(
        amount,
        details.data
      );

      return {
        success: true,
        transactionId: response.txHash,
        processingFee: 0.001, // Fixed network fee
      };
    } catch (error) {
      return {
        success: false,
        error: "Cryptocurrency transaction failed",
      };
    }
  }

  private async processCryptoTransaction(
    amount: number,
    cryptoData: any
  ): Promise<any> {
    await new Promise((resolve) => setTimeout(resolve, 2000)); // Longer for blockchain
    return {
      txHash: `0x${Math.random().toString(16).substr(2, 64)}`,
    };
  }
}

// 4. PAYMENT PROCESSOR CONTEXT
class PaymentProcessor {
  private strategy?: PaymentStrategy;
  private availableStrategies = new Map<string, PaymentStrategy>();

  constructor() {
    // Register available payment strategies
    this.registerStrategy("credit_card", new CreditCardStrategy());
    this.registerStrategy("paypal", new PayPalStrategy());
    this.registerStrategy("crypto", new CryptoStrategy());
  }

  registerStrategy(type: string, strategy: PaymentStrategy): void {
    this.availableStrategies.set(type, strategy);
  }

  setPaymentMethod(type: string): boolean {
    const strategy = this.availableStrategies.get(type);
    if (strategy) {
      this.strategy = strategy;
      return true;
    }
    return false;
  }

  getAvailablePaymentMethods(): Array<{
    type: string;
    name: string;
    currencies: string[];
  }> {
    return Array.from(this.availableStrategies.entries()).map(
      ([type, strategy]) => ({
        type,
        name: strategy.getPaymentMethodName(),
        currencies: strategy.getSupportedCurrencies(),
      })
    );
  }

  async processPayment(
    amount: number,
    currency: string,
    paymentDetails: PaymentDetails
  ): Promise<PaymentResult> {
    if (!this.strategy) {
      return {
        success: false,
        error: "No payment method selected",
      };
    }

    if (!this.strategy.getSupportedCurrencies().includes(currency)) {
      return {
        success: false,
        error: `Currency ${currency} not supported by ${this.strategy.getPaymentMethodName()}`,
      };
    }

    return this.strategy.processPayment(amount, paymentDetails);
  }

  validatePaymentDetails(paymentDetails: PaymentDetails): ValidationResult {
    if (!this.strategy) {
      return {
        isValid: false,
        errors: ["No payment method selected"],
      };
    }

    return this.strategy.validatePaymentDetails(paymentDetails);
  }
}

// 5. REACT IMPLEMENTATION
const PaymentForm: React.FC = () => {
  const [processor] = useState(() => new PaymentProcessor());
  const [selectedMethod, setSelectedMethod] = useState<string>("");
  const [amount, setAmount] = useState<number>(0);
  const [currency, setCurrency] = useState<string>("USD");
  const [paymentDetails, setPaymentDetails] = useState<any>({});
  const [result, setResult] = useState<PaymentResult | null>(null);

  const availableMethods = processor.getAvailablePaymentMethods();

  const handleMethodChange = (method: string) => {
    setSelectedMethod(method);
    processor.setPaymentMethod(method);
    setPaymentDetails({}); // Reset details when method changes
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    const paymentData: PaymentDetails = {
      type: selectedMethod as any,
      data: paymentDetails,
    };

    const result = await processor.processPayment(
      amount,
      currency,
      paymentData
    );
    setResult(result);
  };

  const renderPaymentForm = () => {
    switch (selectedMethod) {
      case "credit_card":
        return (
          <div className="payment-form">
            <input
              type="text"
              placeholder="Card Number"
              value={paymentDetails.cardNumber || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  cardNumber: e.target.value,
                }))
              }
            />
            <input
              type="text"
              placeholder="MM/YY"
              value={paymentDetails.expiryDate || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  expiryDate: e.target.value,
                }))
              }
            />
            <input
              type="text"
              placeholder="CVV"
              value={paymentDetails.cvv || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({ ...prev, cvv: e.target.value }))
              }
            />
            <input
              type="text"
              placeholder="Cardholder Name"
              value={paymentDetails.holderName || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  holderName: e.target.value,
                }))
              }
            />
          </div>
        );

      case "paypal":
        return (
          <div className="payment-form">
            <input
              type="email"
              placeholder="PayPal Email"
              value={paymentDetails.email || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  email: e.target.value,
                }))
              }
            />
            <input
              type="password"
              placeholder="PayPal Password"
              value={paymentDetails.password || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  password: e.target.value,
                }))
              }
            />
          </div>
        );

      case "crypto":
        return (
          <div className="payment-form">
            <input
              type="text"
              placeholder="Wallet Address"
              value={paymentDetails.walletAddress || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  walletAddress: e.target.value,
                }))
              }
            />
            <select
              value={paymentDetails.currency || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  currency: e.target.value,
                }))
              }
            >
              <option value="">Select Cryptocurrency</option>
              <option value="BTC">Bitcoin (BTC)</option>
              <option value="ETH">Ethereum (ETH)</option>
              <option value="LTC">Litecoin (LTC)</option>
            </select>
            <input
              type="password"
              placeholder="Private Key"
              value={paymentDetails.privateKey || ""}
              onChange={(e) =>
                setPaymentDetails((prev) => ({
                  ...prev,
                  privateKey: e.target.value,
                }))
              }
            />
          </div>
        );

      default:
        return <div>Please select a payment method</div>;
    }
  };

  return (
    <div className="payment-processor">
      <h2>Payment Processing</h2>

      <form onSubmit={handleSubmit}>
        <div className="payment-method-selection">
          <h3>Select Payment Method</h3>
          {availableMethods.map((method) => (
            <label key={method.type}>
              <input
                type="radio"
                name="paymentMethod"
                value={method.type}
                checked={selectedMethod === method.type}
                onChange={(e) => handleMethodChange(e.target.value)}
              />
              {method.name} ({method.currencies.join(", ")})
            </label>
          ))}
        </div>

        <div className="amount-section">
          <input
            type="number"
            placeholder="Amount"
            value={amount}
            onChange={(e) => setAmount(Number(e.target.value))}
          />
          <select
            value={currency}
            onChange={(e) => setCurrency(e.target.value)}
          >
            <option value="USD">USD</option>
            <option value="EUR">EUR</option>
            <option value="GBP">GBP</option>
          </select>
        </div>

        {renderPaymentForm()}

        <button type="submit" disabled={!selectedMethod || amount <= 0}>
          Process Payment
        </button>
      </form>

      {result && (
        <div
          className={`payment-result ${result.success ? "success" : "error"}`}
        >
          {result.success ? (
            <div>
              <h4>Payment Successful!</h4>
              <p>Transaction ID: {result.transactionId}</p>
              <p>Processing Fee: ${result.processingFee?.toFixed(2)}</p>
            </div>
          ) : (
            <div>
              <h4>Payment Failed</h4>
              <p>{result.error}</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
};
```

### **🏭 Factory Pattern - Object Creation Management**

**What it solves:** When you need to create objects without specifying their exact classes, or when object creation logic is complex.

**Real-World Example: UI Component Factory**

```typescript
// 1. COMPONENT INTERFACE
interface UIComponent {
  render(): HTMLElement;
  destroy(): void;
  getType(): string;
  addEventListener(event: string, handler: (e: Event) => void): void;
}

// 2. COMPONENT CONFIGURATIONS
interface ComponentConfig {
  type: string;
  props: Record<string, any>;
  children?: ComponentConfig[];
  className?: string;
  id?: string;
}

interface ButtonConfig {
  text: string;
  variant: "primary" | "secondary" | "danger";
  size: "small" | "medium" | "large";
  disabled?: boolean;
  icon?: string;
  onClick?: () => void;
}

interface InputConfig {
  type: "text" | "email" | "password" | "number";
  placeholder?: string;
  value?: string;
  required?: boolean;
  validation?: (value: string) => string | null;
  onChange?: (value: string) => void;
}

interface ModalConfig {
  title: string;
  content: string | ComponentConfig;
  showCloseButton: boolean;
  size: "small" | "medium" | "large";
  onClose?: () => void;
}

// 3. CONCRETE COMPONENT IMPLEMENTATIONS
class Button implements UIComponent {
  private element: HTMLButtonElement;
  private config: ButtonConfig;
  private eventListeners: Map<string, (e: Event) => void> = new Map();

  constructor(config: ButtonConfig, className?: string, id?: string) {
    this.config = config;
    this.element = this.createElement(className, id);
  }

  private createElement(className?: string, id?: string): HTMLButtonElement {
    const button = document.createElement("button");

    if (id) button.id = id;
    if (className) button.className = className;

    button.className += ` btn btn--${this.config.variant} btn--${this.config.size}`;
    button.textContent = this.config.text;
    button.disabled = this.config.disabled || false;

    if (this.config.icon) {
      const icon = document.createElement("span");
      icon.className = `icon icon--${this.config.icon}`;
      button.insertBefore(icon, button.firstChild);
    }

    if (this.config.onClick) {
      button.addEventListener("click", this.config.onClick);
    }

    return button;
  }

  render(): HTMLElement {
    return this.element;
  }

  destroy(): void {
    this.eventListeners.forEach((handler, event) => {
      this.element.removeEventListener(event, handler);
    });
    this.element.remove();
  }

  getType(): string {
    return "button";
  }

  addEventListener(event: string, handler: (e: Event) => void): void {
    this.element.addEventListener(event, handler);
    this.eventListeners.set(event, handler);
  }
}

class Input implements UIComponent {
  private element: HTMLInputElement;
  private config: InputConfig;
  private eventListeners: Map<string, (e: Event) => void> = new Map();

  constructor(config: InputConfig, className?: string, id?: string) {
    this.config = config;
    this.element = this.createElement(className, id);
  }

  private createElement(className?: string, id?: string): HTMLInputElement {
    const input = document.createElement("input");

    if (id) input.id = id;
    if (className) input.className = className;

    input.className += " form-input";
    input.type = this.config.type;
    input.placeholder = this.config.placeholder || "";
    input.value = this.config.value || "";
    input.required = this.config.required || false;

    if (this.config.onChange) {
      const changeHandler = (e: Event) => {
        const target = e.target as HTMLInputElement;
        this.config.onChange!(target.value);
      };
      input.addEventListener("input", changeHandler);
      this.eventListeners.set("input", changeHandler);
    }

    if (this.config.validation) {
      const validationHandler = (e: Event) => {
        const target = e.target as HTMLInputElement;
        const error = this.config.validation!(target.value);
        this.showValidationError(error);
      };
      input.addEventListener("blur", validationHandler);
      this.eventListeners.set("blur", validationHandler);
    }

    return input;
  }

  private showValidationError(error: string | null): void {
    const existingError =
      this.element.parentElement?.querySelector(".validation-error");
    if (existingError) {
      existingError.remove();
    }

    if (error) {
      const errorElement = document.createElement("div");
      errorElement.className = "validation-error";
      errorElement.textContent = error;
      this.element.parentElement?.appendChild(errorElement);
      this.element.classList.add("input--error");
    } else {
      this.element.classList.remove("input--error");
    }
  }

  render(): HTMLElement {
    return this.element;
  }

  destroy(): void {
    this.eventListeners.forEach((handler, event) => {
      this.element.removeEventListener(event, handler);
    });
    this.element.remove();
  }

  getType(): string {
    return "input";
  }

  addEventListener(event: string, handler: (e: Event) => void): void {
    this.element.addEventListener(event, handler);
    this.eventListeners.set(event, handler);
  }
}

class Modal implements UIComponent {
  private element: HTMLDivElement;
  private config: ModalConfig;
  private eventListeners: Map<string, (e: Event) => void> = new Map();

  constructor(config: ModalConfig, className?: string, id?: string) {
    this.config = config;
    this.element = this.createElement(className, id);
  }

  private createElement(className?: string, id?: string): HTMLDivElement {
    const overlay = document.createElement("div");
    overlay.className = `modal-overlay ${className || ""}`;
    if (id) overlay.id = id;

    const modal = document.createElement("div");
    modal.className = `modal modal--${this.config.size}`;

    // Header
    const header = document.createElement("div");
    header.className = "modal__header";

    const title = document.createElement("h3");
    title.className = "modal__title";
    title.textContent = this.config.title;
    header.appendChild(title);

    if (this.config.showCloseButton) {
      const closeButton = document.createElement("button");
      closeButton.className = "modal__close";
      closeButton.innerHTML = "×";
      closeButton.addEventListener("click", () => {
        if (this.config.onClose) {
          this.config.onClose();
        }
        this.destroy();
      });
      header.appendChild(closeButton);
    }

    modal.appendChild(header);

    // Content
    const content = document.createElement("div");
    content.className = "modal__content";

    if (typeof this.config.content === "string") {
      content.innerHTML = this.config.content;
    } else {
      // Recursive component creation
      const contentComponent = UIComponentFactory.create(this.config.content);
      content.appendChild(contentComponent.render());
    }

    modal.appendChild(content);
    overlay.appendChild(modal);

    // Close on overlay click
    overlay.addEventListener("click", (e) => {
      if (e.target === overlay) {
        if (this.config.onClose) {
          this.config.onClose();
        }
        this.destroy();
      }
    });

    return overlay;
  }

  render(): HTMLElement {
    return this.element;
  }

  destroy(): void {
    this.eventListeners.forEach((handler, event) => {
      this.element.removeEventListener(event, handler);
    });
    this.element.remove();
  }

  getType(): string {
    return "modal";
  }

  addEventListener(event: string, handler: (e: Event) => void): void {
    this.element.addEventListener(event, handler);
    this.eventListeners.set(event, handler);
  }
}

// 4. FACTORY IMPLEMENTATION
class UIComponentFactory {
  private static componentRegistry = new Map<
    string,
    new (config: any, className?: string, id?: string) => UIComponent
  >();

  static {
    // Register default components
    UIComponentFactory.register("button", Button);
    UIComponentFactory.register("input", Input);
    UIComponentFactory.register("modal", Modal);
  }

  static register(
    type: string,
    componentClass: new (
      config: any,
      className?: string,
      id?: string
    ) => UIComponent
  ): void {
    UIComponentFactory.componentRegistry.set(type, componentClass);
  }

  static create(config: ComponentConfig): UIComponent {
    const ComponentClass = UIComponentFactory.componentRegistry.get(
      config.type
    );

    if (!ComponentClass) {
      throw new Error(`Unknown component type: ${config.type}`);
    }

    return new ComponentClass(config.props, config.className, config.id);
  }

  static createMultiple(configs: ComponentConfig[]): UIComponent[] {
    return configs.map((config) => UIComponentFactory.create(config));
  }

  static getRegisteredTypes(): string[] {
    return Array.from(UIComponentFactory.componentRegistry.keys());
  }
}

// 5. FORM BUILDER USING FACTORY
class FormBuilder {
  private components: UIComponent[] = [];
  private container: HTMLFormElement;

  constructor(containerId: string) {
    this.container = document.createElement("form");
    this.container.id = containerId;
    this.container.className = "dynamic-form";
  }

  addComponent(config: ComponentConfig): FormBuilder {
    const component = UIComponentFactory.create(config);
    this.components.push(component);

    const wrapper = document.createElement("div");
    wrapper.className = "form-field";
    wrapper.appendChild(component.render());

    this.container.appendChild(wrapper);
    return this;
  }

  addComponents(configs: ComponentConfig[]): FormBuilder {
    configs.forEach((config) => this.addComponent(config));
    return this;
  }

  onSubmit(handler: (formData: FormData) => void): FormBuilder {
    this.container.addEventListener("submit", (e) => {
      e.preventDefault();
      const formData = new FormData(this.container);
      handler(formData);
    });
    return this;
  }

  render(): HTMLFormElement {
    return this.container;
  }

  destroy(): void {
    this.components.forEach((component) => component.destroy());
    this.container.remove();
  }
}

// 6. USAGE EXAMPLES
// Simple component creation
const loginButton = UIComponentFactory.create({
  type: "button",
  props: {
    text: "Login",
    variant: "primary",
    size: "large",
    icon: "user",
    onClick: () => console.log("Login clicked"),
  },
  className: "login-btn",
  id: "main-login-btn",
});

// Complex form creation
const registrationForm = new FormBuilder("registration-form")
  .addComponent({
    type: "input",
    props: {
      type: "text",
      placeholder: "Enter your name",
      required: true,
      validation: (value: string) =>
        value.length < 2 ? "Name must be at least 2 characters" : null,
    },
  })
  .addComponent({
    type: "input",
    props: {
      type: "email",
      placeholder: "Enter your email",
      required: true,
      validation: (value: string) =>
        /\S+@\S+\.\S+/.test(value) ? null : "Invalid email format",
    },
  })
  .addComponent({
    type: "input",
    props: {
      type: "password",
      placeholder: "Enter password",
      required: true,
      validation: (value: string) =>
        value.length < 8 ? "Password must be at least 8 characters" : null,
    },
  })
  .addComponent({
    type: "button",
    props: {
      text: "Register",
      variant: "primary",
      size: "large",
    },
  })
  .onSubmit((formData) => {
    console.log("Registration submitted:", Object.fromEntries(formData));
  });

// Modal with nested components
const confirmModal = UIComponentFactory.create({
  type: "modal",
  props: {
    title: "Confirm Action",
    content: {
      type: "button",
      props: {
        text: "Confirm Delete",
        variant: "danger",
        size: "medium",
        onClick: () => console.log("Confirmed"),
      },
    },
    showCloseButton: true,
    size: "medium",
    onClose: () => console.log("Modal closed"),
  },
});

// Append to DOM
document.body.appendChild(loginButton.render());
document.body.appendChild(registrationForm.render());
document.body.appendChild(confirmModal.render());
```

### **🎨 Decorator Pattern - Component Enhancement**

**What it solves:** When you want to add behavior to objects dynamically without altering their structure or creating extensive inheritance hierarchies.

**Real-World Example: Enhanced Data Table with Features**

```typescript
// 1. CORE COMPONENT INTERFACE
interface DataTable {
  render(): HTMLElement;
  getData(): any[];
  setData(data: any[]): void;
  getColumns(): string[];
  destroy(): void;
}

// 2. BASE DATA TABLE IMPLEMENTATION
class BasicDataTable implements DataTable {
  protected data: any[] = [];
  protected columns: string[] = [];
  protected element: HTMLTableElement;
  protected containerId: string;

  constructor(data: any[], columns: string[], containerId: string) {
    this.data = data;
    this.columns = columns;
    this.containerId = containerId;
    this.element = this.createTable();
  }

  protected createTable(): HTMLTableElement {
    const table = document.createElement("table");
    table.className = "data-table";
    table.id = this.containerId;

    this.renderTable();
    return table;
  }

  protected renderTable(): void {
    this.element.innerHTML = "";

    // Create header
    const thead = document.createElement("thead");
    const headerRow = document.createElement("tr");

    this.columns.forEach((column) => {
      const th = document.createElement("th");
      th.textContent = column;
      th.className = "table-header";
      headerRow.appendChild(th);
    });

    thead.appendChild(headerRow);
    this.element.appendChild(thead);

    // Create body
    const tbody = document.createElement("tbody");

    this.data.forEach((row) => {
      const tr = document.createElement("tr");
      tr.className = "table-row";

      this.columns.forEach((column) => {
        const td = document.createElement("td");
        td.textContent = row[column] || "";
        td.className = "table-cell";
        tr.appendChild(td);
      });

      tbody.appendChild(tr);
    });

    this.element.appendChild(tbody);
  }

  render(): HTMLElement {
    return this.element;
  }

  getData(): any[] {
    return [...this.data];
  }

  setData(data: any[]): void {
    this.data = data;
    this.renderTable();
  }

  getColumns(): string[] {
    return [...this.columns];
  }

  destroy(): void {
    this.element.remove();
  }
}

// 3. ABSTRACT DECORATOR
abstract class DataTableDecorator implements DataTable {
  protected table: DataTable;

  constructor(table: DataTable) {
    this.table = table;
  }

  render(): HTMLElement {
    return this.table.render();
  }

  getData(): any[] {
    return this.table.getData();
  }

  setData(data: any[]): void {
    this.table.setData(data);
  }

  getColumns(): string[] {
    return this.table.getColumns();
  }

  destroy(): void {
    this.table.destroy();
  }
}

// 4. CONCRETE DECORATORS
class SortableTableDecorator extends DataTableDecorator {
  private sortState: { column: string; direction: "asc" | "desc" } | null =
    null;

  constructor(table: DataTable) {
    super(table);
    this.addSortFunctionality();
  }

  private addSortFunctionality(): void {
    const tableElement = this.table.render();
    const headers = tableElement.querySelectorAll("th");

    headers.forEach((header, index) => {
      header.style.cursor = "pointer";
      header.classList.add("sortable-header");

      // Add sort indicator
      const sortIcon = document.createElement("span");
      sortIcon.className = "sort-icon";
      sortIcon.textContent = " ↕️";
      header.appendChild(sortIcon);

      header.addEventListener("click", () => {
        this.sortByColumn(this.getColumns()[index]);
        this.updateSortIndicators(header, headers);
      });
    });
  }

  private sortByColumn(column: string): void {
    const data = this.getData();
    const isCurrentColumn = this.sortState?.column === column;
    const newDirection =
      isCurrentColumn && this.sortState?.direction === "asc" ? "desc" : "asc";

    const sortedData = [...data].sort((a, b) => {
      let aVal = a[column];
      let bVal = b[column];

      // Handle different data types
      if (typeof aVal === "number" && typeof bVal === "number") {
        return newDirection === "asc" ? aVal - bVal : bVal - aVal;
      }

      // String comparison
      aVal = String(aVal).toLowerCase();
      bVal = String(bVal).toLowerCase();

      if (newDirection === "asc") {
        return aVal.localeCompare(bVal);
      } else {
        return bVal.localeCompare(aVal);
      }
    });

    this.sortState = { column, direction: newDirection };
    this.setData(sortedData);
  }

  private updateSortIndicators(
    activeHeader: HTMLElement,
    allHeaders: NodeListOf<Element>
  ): void {
    allHeaders.forEach((header) => {
      const icon = header.querySelector(".sort-icon");
      if (icon) {
        icon.textContent = " ↕️";
      }
    });

    const activeIcon = activeHeader.querySelector(".sort-icon");
    if (activeIcon && this.sortState) {
      activeIcon.textContent = this.sortState.direction === "asc" ? " ↑" : " ↓";
    }
  }
}

class SearchableTableDecorator extends DataTableDecorator {
  private originalData: any[] = [];
  private searchInput: HTMLInputElement;

  constructor(table: DataTable) {
    super(table);
    this.originalData = this.getData();
    this.searchInput = this.createSearchInput();
    this.addSearchFunctionality();
  }

  private createSearchInput(): HTMLInputElement {
    const input = document.createElement("input");
    input.type = "text";
    input.placeholder = "Search table...";
    input.className = "table-search";
    return input;
  }

  private addSearchFunctionality(): void {
    const tableElement = this.table.render();

    // Insert search input before table
    tableElement.parentElement?.insertBefore(this.searchInput, tableElement);

    // Debounced search
    let searchTimeout: NodeJS.Timeout;
    this.searchInput.addEventListener("input", (e) => {
      clearTimeout(searchTimeout);
      searchTimeout = setTimeout(() => {
        const searchTerm = (e.target as HTMLInputElement).value.toLowerCase();
        this.performSearch(searchTerm);
      }, 300);
    });
  }

  private performSearch(searchTerm: string): void {
    if (!searchTerm.trim()) {
      this.setData(this.originalData);
      return;
    }

    const columns = this.getColumns();
    const filteredData = this.originalData.filter((row) => {
      return columns.some((column) => {
        const value = String(row[column] || "").toLowerCase();
        return value.includes(searchTerm);
      });
    });

    this.setData(filteredData);
  }

  setData(data: any[]): void {
    this.originalData = data;
    super.setData(data);
  }

  destroy(): void {
    this.searchInput.remove();
    super.destroy();
  }
}

class PaginatedTableDecorator extends DataTableDecorator {
  private currentPage: number = 1;
  private itemsPerPage: number = 10;
  private allData: any[] = [];
  private paginationControls: HTMLDivElement;

  constructor(table: DataTable, itemsPerPage: number = 10) {
    super(table);
    this.itemsPerPage = itemsPerPage;
    this.allData = this.getData();
    this.paginationControls = this.createPaginationControls();
    this.updateDisplayedData();
  }

  private createPaginationControls(): HTMLDivElement {
    const controls = document.createElement("div");
    controls.className = "pagination-controls";
    return controls;
  }

  private updateDisplayedData(): void {
    const startIndex = (this.currentPage - 1) * this.itemsPerPage;
    const endIndex = startIndex + this.itemsPerPage;
    const pageData = this.allData.slice(startIndex, endIndex);

    super.setData(pageData);
    this.updatePaginationControls();
  }

  private updatePaginationControls(): void {
    const totalPages = Math.ceil(this.allData.length / this.itemsPerPage);

    this.paginationControls.innerHTML = "";

    // Page info
    const pageInfo = document.createElement("span");
    pageInfo.className = "page-info";
    pageInfo.textContent = `Page ${this.currentPage} of ${totalPages} (${this.allData.length} total items)`;
    this.paginationControls.appendChild(pageInfo);

    // Navigation buttons
    const buttonContainer = document.createElement("div");
    buttonContainer.className = "pagination-buttons";

    // Previous button
    const prevButton = document.createElement("button");
    prevButton.textContent = "Previous";
    prevButton.disabled = this.currentPage === 1;
    prevButton.addEventListener("click", () => {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.updateDisplayedData();
      }
    });
    buttonContainer.appendChild(prevButton);

    // Page numbers
    const maxVisiblePages = 5;
    const startPage = Math.max(
      1,
      this.currentPage - Math.floor(maxVisiblePages / 2)
    );
    const endPage = Math.min(totalPages, startPage + maxVisiblePages - 1);

    for (let i = startPage; i <= endPage; i++) {
      const pageButton = document.createElement("button");
      pageButton.textContent = i.toString();
      pageButton.className = i === this.currentPage ? "active" : "";
      pageButton.addEventListener("click", () => {
        this.currentPage = i;
        this.updateDisplayedData();
      });
      buttonContainer.appendChild(pageButton);
    }

    // Next button
    const nextButton = document.createElement("button");
    nextButton.textContent = "Next";
    nextButton.disabled = this.currentPage === totalPages;
    nextButton.addEventListener("click", () => {
      if (this.currentPage < totalPages) {
        this.currentPage++;
        this.updateDisplayedData();
      }
    });
    buttonContainer.appendChild(nextButton);

    this.paginationControls.appendChild(buttonContainer);

    // Append pagination controls after table
    const tableElement = this.table.render();
    if (
      !tableElement.nextElementSibling?.classList.contains(
        "pagination-controls"
      )
    ) {
      tableElement.parentElement?.insertBefore(
        this.paginationControls,
        tableElement.nextSibling
      );
    }
  }

  setData(data: any[]): void {
    this.allData = data;
    this.currentPage = 1;
    this.updateDisplayedData();
  }

  destroy(): void {
    this.paginationControls.remove();
    super.destroy();
  }
}

class ExportableTableDecorator extends DataTableDecorator {
  private exportControls: HTMLDivElement;

  constructor(table: DataTable) {
    super(table);
    this.exportControls = this.createExportControls();
  }

  private createExportControls(): HTMLDivElement {
    const controls = document.createElement("div");
    controls.className = "export-controls";

    const csvButton = document.createElement("button");
    csvButton.textContent = "Export CSV";
    csvButton.className = "export-btn export-csv";
    csvButton.addEventListener("click", () => this.exportToCSV());

    const jsonButton = document.createElement("button");
    jsonButton.textContent = "Export JSON";
    jsonButton.className = "export-btn export-json";
    jsonButton.addEventListener("click", () => this.exportToJSON());

    controls.appendChild(csvButton);
    controls.appendChild(jsonButton);

    // Insert before table
    const tableElement = this.table.render();
    tableElement.parentElement?.insertBefore(controls, tableElement);

    return controls;
  }

  private exportToCSV(): void {
    const data = this.getData();
    const columns = this.getColumns();

    // Create CSV content
    let csvContent = columns.join(",") + "\n";

    data.forEach((row) => {
      const rowData = columns.map((column) => {
        const value = row[column] || "";
        // Escape quotes and wrap in quotes if contains comma
        const escaped = String(value).replace(/"/g, '""');
        return escaped.includes(",") ? `"${escaped}"` : escaped;
      });
      csvContent += rowData.join(",") + "\n";
    });

    this.downloadFile(csvContent, "table-export.csv", "text/csv");
  }

  private exportToJSON(): void {
    const data = this.getData();
    const jsonContent = JSON.stringify(data, null, 2);
    this.downloadFile(jsonContent, "table-export.json", "application/json");
  }

  private downloadFile(
    content: string,
    filename: string,
    mimeType: string
  ): void {
    const blob = new Blob([content], { type: mimeType });
    const url = URL.createObjectURL(blob);

    const link = document.createElement("a");
    link.href = url;
    link.download = filename;
    link.style.display = "none";

    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    URL.revokeObjectURL(url);
  }

  destroy(): void {
    this.exportControls.remove();
    super.destroy();
  }
}

// 5. USAGE EXAMPLES
const sampleData = [
  {
    id: 1,
    name: "John Doe",
    email: "john@example.com",
    age: 30,
    department: "Engineering",
  },
  {
    id: 2,
    name: "Jane Smith",
    email: "jane@example.com",
    age: 28,
    department: "Design",
  },
  {
    id: 3,
    name: "Bob Johnson",
    email: "bob@example.com",
    age: 35,
    department: "Marketing",
  },
  {
    id: 4,
    name: "Alice Brown",
    email: "alice@example.com",
    age: 32,
    department: "Engineering",
  },
  {
    id: 5,
    name: "Charlie Wilson",
    email: "charlie@example.com",
    age: 29,
    department: "Sales",
  },
];

const columns = ["id", "name", "email", "age", "department"];

// Create basic table
let table: DataTable = new BasicDataTable(
  sampleData,
  columns,
  "employee-table"
);

// Apply decorators in any combination
table = new SortableTableDecorator(table);
table = new SearchableTableDecorator(table);
table = new PaginatedTableDecorator(table, 3); // 3 items per page
table = new ExportableTableDecorator(table);

// Render the fully enhanced table
document.body.appendChild(table.render());
```

### **⚡ Command Pattern - Action Encapsulation**

**What it solves:** When you need to encapsulate actions as objects, support undo/redo operations, or queue/log operations.

**Real-World Example: Rich Text Editor with Undo/Redo**

```typescript
// 1. COMMAND INTERFACE
interface Command {
  execute(): void;
  undo(): void;
  getDescription(): string;
}

// 2. EDITOR STATE MODEL
interface EditorState {
  content: string;
  cursorPosition: number;
  selection?: { start: number; end: number };
}

class TextEditor {
  private element: HTMLTextAreaElement;
  private state: EditorState;

  constructor(elementId: string, initialContent: string = "") {
    this.element = document.getElementById(elementId) as HTMLTextAreaElement;
    this.state = {
      content: initialContent,
      cursorPosition: 0,
    };
    this.element.value = initialContent;
  }

  getState(): EditorState {
    return {
      content: this.element.value,
      cursorPosition: this.element.selectionStart,
      selection:
        this.element.selectionStart !== this.element.selectionEnd
          ? {
              start: this.element.selectionStart,
              end: this.element.selectionEnd,
            }
          : undefined,
    };
  }

  setState(state: EditorState): void {
    this.state = { ...state };
    this.element.value = state.content;
    this.element.setSelectionRange(state.cursorPosition, state.cursorPosition);
    this.element.focus();
  }

  insertText(text: string, position: number): void {
    const content = this.element.value;
    const newContent =
      content.slice(0, position) + text + content.slice(position);
    this.element.value = newContent;
    this.element.setSelectionRange(
      position + text.length,
      position + text.length
    );
  }

  deleteText(startPosition: number, endPosition: number): string {
    const content = this.element.value;
    const deletedText = content.slice(startPosition, endPosition);
    const newContent =
      content.slice(0, startPosition) + content.slice(endPosition);
    this.element.value = newContent;
    this.element.setSelectionRange(startPosition, startPosition);
    return deletedText;
  }

  replaceText(
    startPosition: number,
    endPosition: number,
    newText: string
  ): string {
    const content = this.element.value;
    const oldText = content.slice(startPosition, endPosition);
    const newContent =
      content.slice(0, startPosition) + newText + content.slice(endPosition);
    this.element.value = newContent;
    this.element.setSelectionRange(
      startPosition + newText.length,
      startPosition + newText.length
    );
    return oldText;
  }

  getSelectedText(): string {
    return this.element.value.slice(
      this.element.selectionStart,
      this.element.selectionEnd
    );
  }

  getElement(): HTMLTextAreaElement {
    return this.element;
  }
}

// 3. CONCRETE COMMANDS
class InsertTextCommand implements Command {
  private editor: TextEditor;
  private text: string;
  private position: number;
  private previousState: EditorState;

  constructor(editor: TextEditor, text: string, position: number) {
    this.editor = editor;
    this.text = text;
    this.position = position;
    this.previousState = editor.getState();
  }

  execute(): void {
    this.editor.insertText(this.text, this.position);
  }

  undo(): void {
    this.editor.setState(this.previousState);
  }

  getDescription(): string {
    return `Insert "${this.text}"`;
  }
}

class DeleteTextCommand implements Command {
  private editor: TextEditor;
  private startPosition: number;
  private endPosition: number;
  private deletedText: string = "";
  private previousState: EditorState;

  constructor(editor: TextEditor, startPosition: number, endPosition: number) {
    this.editor = editor;
    this.startPosition = startPosition;
    this.endPosition = endPosition;
    this.previousState = editor.getState();
  }

  execute(): void {
    this.deletedText = this.editor.deleteText(
      this.startPosition,
      this.endPosition
    );
  }

  undo(): void {
    this.editor.setState(this.previousState);
  }

  getDescription(): string {
    return `Delete "${this.deletedText}"`;
  }
}

class ReplaceTextCommand implements Command {
  private editor: TextEditor;
  private startPosition: number;
  private endPosition: number;
  private newText: string;
  private oldText: string = "";
  private previousState: EditorState;

  constructor(
    editor: TextEditor,
    startPosition: number,
    endPosition: number,
    newText: string
  ) {
    this.editor = editor;
    this.startPosition = startPosition;
    this.endPosition = endPosition;
    this.newText = newText;
    this.previousState = editor.getState();
  }

  execute(): void {
    this.oldText = this.editor.replaceText(
      this.startPosition,
      this.endPosition,
      this.newText
    );
  }

  undo(): void {
    this.editor.setState(this.previousState);
  }

  getDescription(): string {
    return `Replace "${this.oldText}" with "${this.newText}"`;
  }
}

class FormatTextCommand implements Command {
  private editor: TextEditor;
  private formatter: (text: string) => string;
  private formatName: string;
  private startPosition: number;
  private endPosition: number;
  private previousState: EditorState;

  constructor(
    editor: TextEditor,
    startPosition: number,
    endPosition: number,
    formatter: (text: string) => string,
    formatName: string
  ) {
    this.editor = editor;
    this.startPosition = startPosition;
    this.endPosition = endPosition;
    this.formatter = formatter;
    this.formatName = formatName;
    this.previousState = editor.getState();
  }

  execute(): void {
    const selectedText = this.editor
      .getElement()
      .value.slice(this.startPosition, this.endPosition);
    const formattedText = this.formatter(selectedText);
    this.editor.replaceText(
      this.startPosition,
      this.endPosition,
      formattedText
    );
  }

  undo(): void {
    this.editor.setState(this.previousState);
  }

  getDescription(): string {
    return `Format text: ${this.formatName}`;
  }
}

// 4. COMMAND MANAGER (INVOKER)
class CommandManager {
  private history: Command[] = [];
  private currentPosition: number = -1;
  private maxHistorySize: number = 100;

  executeCommand(command: Command): void {
    // Execute the command
    command.execute();

    // Remove any commands after current position (when redoing is no longer possible)
    this.history = this.history.slice(0, this.currentPosition + 1);

    // Add new command to history
    this.history.push(command);
    this.currentPosition = this.history.length - 1;

    // Limit history size
    if (this.history.length > this.maxHistorySize) {
      this.history = this.history.slice(
        this.history.length - this.maxHistorySize
      );
      this.currentPosition = this.history.length - 1;
    }

    this.notifyHistoryChange();
  }

  undo(): boolean {
    if (this.canUndo()) {
      const command = this.history[this.currentPosition];
      command.undo();
      this.currentPosition--;
      this.notifyHistoryChange();
      return true;
    }
    return false;
  }

  redo(): boolean {
    if (this.canRedo()) {
      this.currentPosition++;
      const command = this.history[this.currentPosition];
      command.execute();
      this.notifyHistoryChange();
      return true;
    }
    return false;
  }

  canUndo(): boolean {
    return this.currentPosition >= 0;
  }

  canRedo(): boolean {
    return this.currentPosition < this.history.length - 1;
  }

  getHistory(): Array<{ command: Command; isActive: boolean }> {
    return this.history.map((command, index) => ({
      command,
      isActive: index <= this.currentPosition,
    }));
  }

  clear(): void {
    this.history = [];
    this.currentPosition = -1;
    this.notifyHistoryChange();
  }

  private notifyHistoryChange(): void {
    // Dispatch custom event for UI updates
    document.dispatchEvent(
      new CustomEvent("commandHistoryChanged", {
        detail: {
          canUndo: this.canUndo(),
          canRedo: this.canRedo(),
          history: this.getHistory(),
        },
      })
    );
  }
}

// 5. RICH TEXT EDITOR WITH COMMANDS
class RichTextEditorWithCommands {
  private editor: TextEditor;
  private commandManager: CommandManager;
  private toolbar: HTMLDivElement;
  private historyPanel: HTMLDivElement;

  constructor(containerId: string) {
    this.commandManager = new CommandManager();
    this.createUI(containerId);
    this.editor = new TextEditor("editor-textarea");
    this.setupEventListeners();
  }

  private createUI(containerId: string): void {
    const container = document.getElementById(containerId);
    if (!container) throw new Error("Container not found");

    container.innerHTML = `
      <div class="rich-editor">
        <div id="editor-toolbar" class="editor-toolbar"></div>
        <textarea id="editor-textarea" class="editor-content" rows="15" cols="80"></textarea>
        <div id="history-panel" class="history-panel"></div>
      </div>
    `;

    this.toolbar = document.getElementById("editor-toolbar") as HTMLDivElement;
    this.historyPanel = document.getElementById(
      "history-panel"
    ) as HTMLDivElement;

    this.createToolbar();
    this.createHistoryPanel();
  }

  private createToolbar(): void {
    const buttons = [
      {
        text: "Bold",
        action: () => this.formatSelection((text) => `**${text}**`, "Bold"),
      },
      {
        text: "Italic",
        action: () => this.formatSelection((text) => `*${text}*`, "Italic"),
      },
      {
        text: "Uppercase",
        action: () =>
          this.formatSelection((text) => text.toUpperCase(), "Uppercase"),
      },
      {
        text: "Lowercase",
        action: () =>
          this.formatSelection((text) => text.toLowerCase(), "Lowercase"),
      },
      {
        text: "Insert Date",
        action: () => this.insertTextAtCursor(new Date().toLocaleDateString()),
      },
      {
        text: "Insert Time",
        action: () => this.insertTextAtCursor(new Date().toLocaleTimeString()),
      },
    ];

    buttons.forEach((button) => {
      const btn = document.createElement("button");
      btn.textContent = button.text;
      btn.className = "toolbar-btn";
      btn.addEventListener("click", button.action);
      this.toolbar.appendChild(btn);
    });

    // Undo/Redo buttons
    const undoBtn = document.createElement("button");
    undoBtn.textContent = "Undo";
    undoBtn.id = "undo-btn";
    undoBtn.className = "toolbar-btn undo-redo-btn";
    undoBtn.disabled = true;
    undoBtn.addEventListener("click", () => this.commandManager.undo());

    const redoBtn = document.createElement("button");
    redoBtn.textContent = "Redo";
    redoBtn.id = "redo-btn";
    redoBtn.className = "toolbar-btn undo-redo-btn";
    redoBtn.disabled = true;
    redoBtn.addEventListener("click", () => this.commandManager.redo());

    this.toolbar.appendChild(undoBtn);
    this.toolbar.appendChild(redoBtn);
  }

  private createHistoryPanel(): void {
    this.historyPanel.innerHTML = `
      <h4>Command History</h4>
      <div id="history-list" class="history-list"></div>
      <button id="clear-history-btn" class="clear-btn">Clear History</button>
    `;

    const clearBtn = document.getElementById("clear-history-btn");
    clearBtn?.addEventListener("click", () => this.commandManager.clear());
  }

  private formatSelection(
    formatter: (text: string) => string,
    formatName: string
  ): void {
    const element = this.editor.getElement();
    const start = element.selectionStart;
    const end = element.selectionEnd;

    if (start === end) {
      alert("Please select text to format");
      return;
    }

    const command = new FormatTextCommand(
      this.editor,
      start,
      end,
      formatter,
      formatName
    );
    this.commandManager.executeCommand(command);
  }

  private insertTextAtCursor(text: string): void {
    const position = this.editor.getElement().selectionStart;
    const command = new InsertTextCommand(this.editor, text, position);
    this.commandManager.executeCommand(command);
  }

  private setupEventListeners(): void {
    // Listen for command history changes
    document.addEventListener("commandHistoryChanged", (e: any) => {
      const { canUndo, canRedo, history } = e.detail;

      // Update undo/redo button states
      const undoBtn = document.getElementById("undo-btn") as HTMLButtonElement;
      const redoBtn = document.getElementById("redo-btn") as HTMLButtonElement;

      undoBtn.disabled = !canUndo;
      redoBtn.disabled = !canRedo;

      // Update history display
      this.updateHistoryDisplay(history);
    });

    // Capture text changes for undo/redo
    const element = this.editor.getElement();
    let lastValue = element.value;
    let timeout: NodeJS.Timeout;

    element.addEventListener("input", () => {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        const currentValue = element.value;
        if (currentValue !== lastValue) {
          // This is a simplified version - in a real implementation,
          // you'd want to capture the exact changes
          lastValue = currentValue;
        }
      }, 500);
    });
  }

  private updateHistoryDisplay(
    history: Array<{ command: Command; isActive: boolean }>
  ): void {
    const historyList = document.getElementById("history-list");
    if (!historyList) return;

    historyList.innerHTML = "";

    history.forEach((item, index) => {
      const historyItem = document.createElement("div");
      historyItem.className = `history-item ${
        item.isActive ? "active" : "inactive"
      }`;
      historyItem.textContent = `${
        index + 1
      }. ${item.command.getDescription()}`;
      historyList.appendChild(historyItem);
    });

    // Scroll to bottom
    historyList.scrollTop = historyList.scrollHeight;
  }
}

// 6. USAGE EXAMPLE
const richEditor = new RichTextEditorWithCommands("editor-container");
```

### **🏢 Facade Pattern - Simplified Interface**

**What it solves:** When you have a complex subsystem with many classes and you want to provide a simple, unified interface.

**Real-World Example: E-commerce Order Processing Facade**

```typescript
// 1. COMPLEX SUBSYSTEM CLASSES
class InventoryService {
  async checkStock(productId: string, quantity: number): Promise<boolean> {
    console.log(
      `Checking stock for product ${productId}, quantity: ${quantity}`
    );
    await this.simulateDelay(300);

    // Simulate stock check logic
    const randomStock = Math.floor(Math.random() * 50) + 10;
    return randomStock >= quantity;
  }

  async reserveItems(productId: string, quantity: number): Promise<string> {
    console.log(`Reserving ${quantity} items of product ${productId}`);
    await this.simulateDelay(200);
    return `reservation_${Date.now()}`;
  }

  async releaseReservation(reservationId: string): Promise<void> {
    console.log(`Releasing reservation ${reservationId}`);
    await this.simulateDelay(100);
  }

  private async simulateDelay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

class PaymentService {
  async validatePaymentMethod(
    paymentInfo: PaymentInfo
  ): Promise<ValidationResult> {
    console.log("Validating payment method");
    await this.simulateDelay(400);

    // Simulate validation
    return {
      isValid: paymentInfo.cardNumber.length >= 16,
      errors: paymentInfo.cardNumber.length < 16 ? ["Invalid card number"] : [],
    };
  }

  async processPayment(
    amount: number,
    paymentInfo: PaymentInfo
  ): Promise<PaymentResult> {
    console.log(`Processing payment of $${amount}`);
    await this.simulateDelay(1000);

    // Simulate payment processing
    const success = Math.random() > 0.1; // 90% success rate

    return {
      success,
      transactionId: success ? `txn_${Date.now()}` : undefined,
      error: success ? undefined : "Payment failed",
    };
  }

  async refundPayment(transactionId: string, amount: number): Promise<boolean> {
    console.log(`Refunding $${amount} for transaction ${transactionId}`);
    await this.simulateDelay(800);
    return true;
  }

  private async simulateDelay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

class ShippingService {
  async calculateShipping(
    address: Address,
    weight: number
  ): Promise<ShippingOption[]> {
    console.log("Calculating shipping options");
    await this.simulateDelay(500);

    return [
      { method: "standard", cost: 9.99, estimatedDays: 5 },
      { method: "express", cost: 19.99, estimatedDays: 2 },
      { method: "overnight", cost: 39.99, estimatedDays: 1 },
    ];
  }

  async scheduleShipment(
    orderId: string,
    address: Address,
    shippingMethod: string
  ): Promise<string> {
    console.log(`Scheduling ${shippingMethod} shipment for order ${orderId}`);
    await this.simulateDelay(300);
    return `tracking_${Date.now()}`;
  }

  async trackShipment(trackingNumber: string): Promise<ShipmentStatus> {
    console.log(`Tracking shipment ${trackingNumber}`);
    await this.simulateDelay(200);

    return {
      status: "in_transit",
      location: "Distribution Center",
      estimatedDelivery: new Date(
        Date.now() + 2 * 24 * 60 * 60 * 1000
      ).toISOString(),
    };
  }

  private async simulateDelay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

class NotificationService {
  async sendOrderConfirmation(
    email: string,
    orderDetails: OrderDetails
  ): Promise<void> {
    console.log(`Sending order confirmation to ${email}`);
    await this.simulateDelay(200);
  }

  async sendPaymentConfirmation(
    email: string,
    paymentDetails: PaymentResult
  ): Promise<void> {
    console.log(`Sending payment confirmation to ${email}`);
    await this.simulateDelay(150);
  }

  async sendShippingNotification(
    email: string,
    trackingNumber: string
  ): Promise<void> {
    console.log(
      `Sending shipping notification to ${email} with tracking ${trackingNumber}`
    );
    await this.simulateDelay(150);
  }

  async sendErrorNotification(email: string, error: string): Promise<void> {
    console.log(`Sending error notification to ${email}: ${error}`);
    await this.simulateDelay(100);
  }

  private async simulateDelay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

class OrderDatabase {
  private orders: Map<string, OrderRecord> = new Map();

  async createOrder(orderData: OrderDetails): Promise<string> {
    console.log("Creating order record");
    await this.simulateDelay(100);

    const orderId = `order_${Date.now()}`;
    const orderRecord: OrderRecord = {
      id: orderId,
      ...orderData,
      status: "created",
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };

    this.orders.set(orderId, orderRecord);
    return orderId;
  }

  async updateOrderStatus(orderId: string, status: string): Promise<void> {
    console.log(`Updating order ${orderId} status to ${status}`);
    await this.simulateDelay(50);

    const order = this.orders.get(orderId);
    if (order) {
      order.status = status;
      order.updatedAt = new Date().toISOString();
    }
  }

  async getOrder(orderId: string): Promise<OrderRecord | undefined> {
    await this.simulateDelay(50);
    return this.orders.get(orderId);
  }

  private async simulateDelay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

// 2. SUPPORTING INTERFACES
interface PaymentInfo {
  cardNumber: string;
  expiryDate: string;
  cvv: string;
  holderName: string;
}

interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

interface PaymentResult {
  success: boolean;
  transactionId?: string;
  error?: string;
}

interface Address {
  street: string;
  city: string;
  state: string;
  zipCode: string;
  country: string;
}

interface ShippingOption {
  method: string;
  cost: number;
  estimatedDays: number;
}

interface ShipmentStatus {
  status: string;
  location: string;
  estimatedDelivery: string;
}

interface OrderItem {
  productId: string;
  productName: string;
  quantity: number;
  price: number;
}

interface OrderDetails {
  customerEmail: string;
  items: OrderItem[];
  shippingAddress: Address;
  billingAddress: Address;
  paymentInfo: PaymentInfo;
  shippingMethod: string;
}

interface OrderRecord extends OrderDetails {
  id: string;
  status: string;
  totalAmount?: number;
  transactionId?: string;
  trackingNumber?: string;
  createdAt: string;
  updatedAt: string;
}

// 3. FACADE IMPLEMENTATION
class ECommerceOrderFacade {
  private inventoryService: InventoryService;
  private paymentService: PaymentService;
  private shippingService: ShippingService;
  private notificationService: NotificationService;
  private orderDatabase: OrderDatabase;

  constructor() {
    this.inventoryService = new InventoryService();
    this.paymentService = new PaymentService();
    this.shippingService = new ShippingService();
    this.notificationService = new NotificationService();
    this.orderDatabase = new OrderDatabase();
  }

  async placeOrder(
    orderDetails: OrderDetails
  ): Promise<{ success: boolean; orderId?: string; error?: string }> {
    let orderId: string | undefined;
    let reservationIds: string[] = [];

    try {
      // 1. Validate and reserve inventory
      console.log("🔍 Step 1: Checking inventory...");
      for (const item of orderDetails.items) {
        const hasStock = await this.inventoryService.checkStock(
          item.productId,
          item.quantity
        );
        if (!hasStock) {
          throw new Error(`Insufficient stock for product ${item.productName}`);
        }

        const reservationId = await this.inventoryService.reserveItems(
          item.productId,
          item.quantity
        );
        reservationIds.push(reservationId);
      }

      // 2. Validate payment method
      console.log("💳 Step 2: Validating payment...");
      const paymentValidation = await this.paymentService.validatePaymentMethod(
        orderDetails.paymentInfo
      );
      if (!paymentValidation.isValid) {
        throw new Error(
          `Payment validation failed: ${paymentValidation.errors.join(", ")}`
        );
      }

      // 3. Calculate shipping
      console.log("📦 Step 3: Calculating shipping...");
      const totalWeight = orderDetails.items.reduce(
        (weight, item) => weight + item.quantity * 1,
        0
      ); // Assume 1 lb per item
      const shippingOptions = await this.shippingService.calculateShipping(
        orderDetails.shippingAddress,
        totalWeight
      );
      const selectedShipping = shippingOptions.find(
        (option) => option.method === orderDetails.shippingMethod
      );

      if (!selectedShipping) {
        throw new Error("Invalid shipping method selected");
      }

      // 4. Calculate total amount
      const itemsTotal = orderDetails.items.reduce(
        (total, item) => total + item.price * item.quantity,
        0
      );
      const totalAmount = itemsTotal + selectedShipping.cost;

      // 5. Process payment
      console.log("💰 Step 4: Processing payment...");
      const paymentResult = await this.paymentService.processPayment(
        totalAmount,
        orderDetails.paymentInfo
      );
      if (!paymentResult.success) {
        throw new Error(paymentResult.error || "Payment processing failed");
      }

      // 6. Create order record
      console.log("📝 Step 5: Creating order record...");
      orderId = await this.orderDatabase.createOrder({
        ...orderDetails,
        totalAmount,
        transactionId: paymentResult.transactionId,
      });

      // 7. Schedule shipment
      console.log("🚚 Step 6: Scheduling shipment...");
      const trackingNumber = await this.shippingService.scheduleShipment(
        orderId,
        orderDetails.shippingAddress,
        orderDetails.shippingMethod
      );

      // 8. Update order with tracking
      await this.orderDatabase.updateOrderStatus(orderId, "confirmed");

      // 9. Send notifications
      console.log("📧 Step 7: Sending notifications...");
      await this.notificationService.sendOrderConfirmation(
        orderDetails.customerEmail,
        orderDetails
      );
      await this.notificationService.sendPaymentConfirmation(
        orderDetails.customerEmail,
        paymentResult
      );
      await this.notificationService.sendShippingNotification(
        orderDetails.customerEmail,
        trackingNumber
      );

      console.log("✅ Order placed successfully!");
      return { success: true, orderId };
    } catch (error) {
      console.error("❌ Order failed:", error);

      // Rollback operations
      await this.rollbackOrder(
        orderId,
        reservationIds,
        orderDetails.customerEmail,
        error as Error
      );

      return { success: false, error: (error as Error).message };
    }
  }

  private async rollbackOrder(
    orderId: string | undefined,
    reservationIds: string[],
    customerEmail: string,
    error: Error
  ): Promise<void> {
    console.log("🔄 Rolling back order...");

    try {
      // Release inventory reservations
      for (const reservationId of reservationIds) {
        await this.inventoryService.releaseReservation(reservationId);
      }

      // Update order status if created
      if (orderId) {
        await this.orderDatabase.updateOrderStatus(orderId, "failed");
      }

      // Send error notification
      await this.notificationService.sendErrorNotification(
        customerEmail,
        error.message
      );

      console.log("✅ Rollback completed");
    } catch (rollbackError) {
      console.error("❌ Rollback failed:", rollbackError);
    }
  }

  async getOrderStatus(orderId: string): Promise<OrderRecord | undefined> {
    return this.orderDatabase.getOrder(orderId);
  }

  async trackShipment(trackingNumber: string): Promise<ShipmentStatus> {
    return this.shippingService.trackShipment(trackingNumber);
  }

  async getShippingOptions(
    address: Address,
    weight: number
  ): Promise<ShippingOption[]> {
    return this.shippingService.calculateShipping(address, weight);
  }
}

// 4. USAGE EXAMPLE
const orderFacade = new ECommerceOrderFacade();

const sampleOrder: OrderDetails = {
  customerEmail: "customer@example.com",
  items: [
    {
      productId: "laptop_001",
      productName: "Gaming Laptop",
      quantity: 1,
      price: 1299.99,
    },
    {
      productId: "mouse_001",
      productName: "Wireless Mouse",
      quantity: 2,
      price: 29.99,
    },
  ],
  shippingAddress: {
    street: "123 Main St",
    city: "Anytown",
    state: "CA",
    zipCode: "12345",
    country: "USA",
  },
  billingAddress: {
    street: "123 Main St",
    city: "Anytown",
    state: "CA",
    zipCode: "12345",
    country: "USA",
  },
  paymentInfo: {
    cardNumber: "1234567890123456",
    expiryDate: "12/25",
    cvv: "123",
    holderName: "John Doe",
  },
  shippingMethod: "express",
};

// Place order using the simple facade interface
orderFacade.placeOrder(sampleOrder).then((result) => {
  if (result.success) {
    console.log(`Order placed successfully! Order ID: ${result.orderId}`);
  } else {
    console.log(`Order failed: ${result.error}`);
  }
});
```

---

## Multi-Tenant Architecture Deep Dive

### Understanding Multi-Tenancy at Scale

**What it Really Means:**
Multi-tenancy isn't just about serving multiple customers from one application. It's about creating a scalable, secure, and customizable platform that can adapt to diverse business needs while maintaining operational efficiency.

**The Business Driver:**
Imagine you're building a project management tool like Asana or Monday.com. You have:

- **Startup teams** (5-10 people) who need basic project tracking
- **Mid-size companies** (100-500 employees) requiring advanced reporting and integrations
- **Enterprise clients** (10,000+ employees) needing custom workflows, SSO, and compliance features

Each client pays different amounts and has vastly different requirements, but maintaining separate applications for each would be:

- Economically unfeasible
- Operationally nightmarish
- Technically unsustainable

### The Three Tenancy Models Explained

#### 1. Shared Everything (Database + Application)

**Real-World Example:** Think of Gmail

- Billions of users share the same application infrastructure
- Your emails are stored in the same database systems as everyone else's
- Google adds a "user_id" to every piece of data to ensure isolation
- All users benefit from the same feature updates simultaneously

**When This Works:**

- **High-volume, low-customization scenarios** (email, social media, basic SaaS tools)
- **Cost-sensitive markets** where keeping prices low is crucial
- **Standardized business processes** where all customers operate similarly

**The Hidden Challenges:**

- **Performance isolation:** One customer's heavy usage can impact others
- **Data security concerns:** Higher risk of data leakage between tenants
- **Compliance complexity:** Difficult to meet varying regulatory requirements
- **Customization limitations:** Hard to provide tenant-specific features

#### 2. Shared Application, Isolated Databases

**Real-World Example:** Think of Shopify

- All merchants use the same Shopify platform
- Each store has its own database instance
- Shared features like payment processing, themes, and apps
- Individual stores can have custom configurations without affecting others

**When This Works:**

- **Regulated industries** where data isolation is mandatory (healthcare, finance)
- **Varying data schemas** where tenants need different data structures
- **Performance-sensitive applications** where database contention is problematic
- **Compliance requirements** that mandate data segregation

**The Trade-offs:**

- **Higher operational costs:** Multiple databases to maintain and backup
- **Complex deployment:** Database migrations become more challenging
- **Feature rollout complexity:** Harder to implement cross-tenant features
- **Resource management:** Need to monitor and scale databases independently

#### 3. Completely Isolated (Single-Tenant)

**Real-World Example:** Think of enterprise Salesforce deployments

- Large enterprises get their own Salesforce instance
- Complete customization of workflows, integrations, and data models
- Dedicated infrastructure with guaranteed performance
- Full control over security, compliance, and operations

**When This Works:**

- **Enterprise clients** with significant revenue per customer
- **Highly regulated industries** (banking, government, healthcare)
- **Complex customization requirements** that can't be standardized
- **Performance guarantees** where SLA requirements are stringent

**The Investment Required:**

- **Significant operational overhead:** Each instance needs individual management
- **Higher customer acquisition cost:** Only viable for high-value customers
- **Complex updates:** Features must be deployed to each instance separately
- **Resource inefficiency:** Lower utilization compared to shared models

### Frontend Implications for UI Architects

#### Theme and Branding Strategies

**The Challenge:**
Every tenant wants their platform to feel like "their own" application. This goes beyond just changing colors - it includes:

- **Brand consistency** across all user touchpoints
- **Custom layouts** that match their business workflows
- **Terminology** that aligns with their industry standards
- **Navigation patterns** that fit their organizational structure

**Strategic Approaches:**

**Design Token System:**
Instead of hardcoding styles, create a token-based system where:

- Colors, fonts, spacing, and borders are all configurable
- Tenants can upload their brand guidelines and automatically generate tokens
- Components automatically adapt to token changes
- Design consistency is maintained even with customization

**Component Theming Strategy:**
Build components with theming in mind:

- Variant props that change component behavior based on tenant configuration
- CSS custom properties that allow runtime theme switching
- Conditional rendering based on tenant feature flags
- Responsive theming for different device types and contexts

**Runtime Customization:**

- Load tenant-specific CSS at application startup
- Dynamic icon and logo replacement
- Configurable dashboard layouts
- Custom terminology and label management

#### State Management Complexity

**The Multi-Tenant State Challenge:**
In a single-tenant application, you might have global state for:

- Current user information
- Application configuration
- Feature flags and permissions

In multi-tenant applications, this becomes:

- Current user + tenant context
- Tenant-specific configuration + global defaults
- Tenant-specific feature flags + global features
- Cross-tenant data isolation

**Strategic Solutions:**

**Context Isolation:**
Every piece of state should be aware of its tenant context:

- API calls automatically include tenant identifiers
- Cached data is isolated by tenant
- User permissions are evaluated within tenant scope
- Session management handles tenant switching

**Hierarchical Configuration:**
Build configuration systems that support:

- Global defaults that apply to all tenants
- Tenant-level overrides for specific needs
- User-level preferences within tenant context
- Role-based configurations within tenant boundaries

### Security Architecture for Multi-Tenancy

**The Security Paradox:**
Multi-tenancy creates a fundamental security challenge: you need to share infrastructure while maintaining complete data isolation. One security vulnerability could potentially expose multiple tenants' data.

**Defense in Depth Strategy:**

**Authentication Layer:**

- Tenant identification happens before user authentication
- User credentials are validated within tenant context
- Session tokens include both user and tenant information
- SSO integrations are tenant-specific

**Authorization Layer:**

- Every data access request validates tenant ownership
- Role-based permissions are scoped to tenant boundaries
- API endpoints enforce tenant isolation at the gateway level
- Database queries automatically include tenant filters

**Data Isolation:**

- Row-level security in databases
- Encrypted data with tenant-specific keys
- Audit logs that track cross-tenant access attempts
- Regular security testing with tenant impersonation

### Performance Considerations

**The Noisy Neighbor Problem in Detail:**
In shared environments, one tenant's behavior can impact others:

- **CPU intensive operations** (large report generation)
- **Memory consumption** (loading massive datasets)
- **Database queries** (complex analytics queries)
- **Network bandwidth** (file uploads/downloads)

**Mitigation Strategies:**

**Resource Quotas and Throttling:**

- API rate limiting per tenant
- Database connection pooling with tenant limits
- File storage quotas with overage policies
- Background job queuing with tenant priorities

**Performance Monitoring:**

- Tenant-specific performance metrics
- Real-time alerting for resource consumption
- Automated scaling based on tenant usage patterns
- Performance degradation detection and mitigation

**Optimization Techniques:**

- Tenant-aware caching strategies
- Database query optimization with tenant-specific indexes
- CDN configurations with tenant-based routing
- Background processing with tenant queuing

### Real-World Decision Framework

**Choosing the Right Model:**

**Start with Shared Everything when:**

- You're validating a new market or product idea
- Customer acquisition cost needs to be very low
- Time to market is critical
- Customer requirements are relatively standardized

**Move to Shared Application/Isolated Data when:**

- You have paying customers with specific data requirements
- Compliance or security requirements demand data isolation
- Performance becomes an issue with shared databases
- Customers need different data schemas or extensive customization

**Consider Single-Tenant when:**

- Individual customer revenue justifies the operational cost
- Regulatory requirements mandate complete isolation
- Customers need extensive customization or integrations
- Performance guarantees are part of your value proposition

**Migration Strategy:**
Most successful SaaS companies start with shared everything and migrate customers to more isolated models as they grow and their requirements become more complex. Plan your architecture to support this evolution rather than trying to build the perfect solution immediately.

**Key Success Factors:**

- **Start simple** and evolve based on actual customer needs
- **Instrument everything** so you can make data-driven decisions
- **Build with migration in mind** to support tenant model evolution
- **Focus on operational excellence** regardless of which model you choose

---

## Microservices & Micro-Frontend Patterns

### The Evolution from Monoliths to Distributed Systems

**Why the Shift Happened:**
Imagine Netflix in 2008 vs 2025. In 2008, they had a monolithic application serving DVD rentals. Today, they have:

- **300+ microservices** handling different aspects (recommendations, billing, streaming, content management)
- **Global distribution** across multiple data centers and cloud regions
- **Thousands of engineers** working on different services simultaneously
- **Continuous deployment** with services updated independently

The monolith couldn't scale to meet these demands.

### Microservices Architecture Deep Dive

**What Microservices Really Are:**
Microservices aren't just "small services." They're a way of organizing teams and technology around business capabilities. Each service:

- **Owns its data** completely (no shared databases)
- **Has a single business purpose** (user management, payment processing, etc.)
- **Can be developed and deployed independently**
- **Communicates through well-defined APIs**

**Real-World Example: Uber's Architecture**
Uber's platform demonstrates microservices at scale:

- **Trip Service:** Manages ride requests and matching
- **Pricing Service:** Handles surge pricing and fare calculations
- **Payment Service:** Processes payments and handles billing
- **Location Service:** Tracks driver and rider locations
- **Notification Service:** Sends push notifications and SMS
- **User Service:** Manages rider and driver profiles

Each service can be updated, scaled, and maintained independently.

**The Hidden Complexities:**

**Service Communication:**
In a monolith, calling another function is simple. In microservices:

- Network calls can fail, timeout, or return errors
- Data consistency across services becomes complex
- Transaction management spans multiple systems
- Performance overhead from network communication

**Data Management:**

- Each service owns its data (no shared databases)
- Data consistency requires careful design
- Reporting across services becomes challenging
- Data migration and schema changes are more complex

**Operational Overhead:**

- Multiple services to deploy, monitor, and debug
- Distributed logging and tracing requirements
- Service discovery and load balancing
- Security policies across service boundaries

### Micro-Frontend Architecture

**The Problem with Monolithic Frontends:**
Even with microservices on the backend, many organizations still have monolithic frontends where:

- **One codebase** serves the entire user interface
- **Single deployment** for all frontend changes
- **Team dependencies** for any UI updates
- **Technology lock-in** to one framework across the entire application

**Real-World Example: Spotify's Approach**
Spotify has dozens of teams working on different features:

- **Music Player Team:** Handles playback, queue management, audio quality
- **Discovery Team:** Manages playlists, recommendations, search
- **Podcast Team:** Handles podcast-specific features and content
- **Social Team:** Manages sharing, following, and social features

Each team can choose their technology stack and deploy independently.

**Micro-Frontend Implementation Strategies:**

**Runtime Integration (Module Federation):**

- Different teams build separate applications
- Applications are combined at runtime in the browser
- Shared dependencies are loaded once and shared
- Teams can use different versions of frameworks

**Build-Time Integration:**

- Components from different teams are combined during build
- Shared component library ensures consistency
- More traditional approach with better performance
- Requires coordination for shared dependency updates

**Server-Side Integration:**

- Different micro-frontends are composed on the server
- Better for SEO and initial load performance
- More complex infrastructure requirements
- Edge-side includes (ESI) or server-side rendering

**Benefits of Micro-Frontends:**

**Team Autonomy:**

- Teams can choose their own technology stack
- Independent deployment and release cycles
- Reduced coordination overhead between teams
- Faster feature development and iteration

**Scalability:**

- Different parts of the application can be scaled independently
- Teams can optimize their specific domain
- Easier to distribute work across multiple teams
- Better fault isolation

**Technology Evolution:**

- Gradual migration from old to new technologies
- Experimentation with new frameworks in isolation
- Legacy code can be maintained while new features use modern tech
- Reduced risk of technology choices

**Challenges and Mitigation:**

**Consistency Challenges:**

- **Solution:** Shared design system and component library
- **Implementation:** Common UI components published as npm packages
- **Governance:** Design system team maintains consistency standards

**Performance Concerns:**

- **Solution:** Careful dependency management and lazy loading
- **Implementation:** Shared vendors bundle for common libraries
- **Monitoring:** Performance budgets for each micro-frontend

**Communication Between Micro-Frontends:**

- **Solution:** Event bus or state management solution
- **Implementation:** Custom events, shared state, or message passing
- **Governance:** Well-defined contracts for cross-team communication

### Domain-Driven Design (DDD) in Frontend Architecture

**Understanding Business Domains:**
Before diving into technical solutions, successful architects understand the business domains:

**Bounded Contexts:**
Each part of your application serves a different business purpose:

- **E-commerce Example:** Product catalog, shopping cart, order management, customer service
- **Banking Example:** Account management, transactions, loans, investments
- **Healthcare Example:** Patient records, scheduling, billing, clinical workflows

**Real-World Application: Amazon's Domain Structure**
Amazon's frontend reflects their business domains:

- **Product Discovery:** Search, categories, recommendations
- **Shopping Experience:** Product pages, cart, checkout
- **Customer Service:** Orders, returns, help center
- **Seller Portal:** Inventory management, analytics, listing tools

Each domain can evolve independently while maintaining the overall user experience.

**Implementing DDD in Frontend Architecture:**

**Feature-Based Organization:**
Instead of organizing by technology (components, services, utils), organize by business domain:

```
/domains
  /product-catalog
    /components
    /services
    /models
  /shopping-cart
    /components
    /services
    /models
  /checkout
    /components
    /services
    /models
```

**Domain Services:**
Each domain has its own services that encapsulate business logic:

- Product service handles catalog operations
- Cart service manages shopping cart state
- Checkout service processes orders

**Cross-Domain Communication:**
Domains communicate through well-defined interfaces:

- Events for loosely coupled communication
- Shared models for common concepts
- APIs that respect domain boundaries

---

## State Management Patterns

### The Evolution of State Management

**From Simple to Complex:**
State management has evolved as applications became more complex:

**Traditional Approach (Early Web):**

- Server rendered pages with embedded state
- Form submissions reload entire pages
- State managed in server sessions
- JavaScript only for simple interactions

**SPA Revolution:**

- Client-side routing and rendering
- AJAX calls for data fetching
- Local component state management
- Browser storage for persistence

**Modern Complexity:**

- Real-time updates from multiple sources
- Optimistic updates with error handling
- Offline capability with sync
- Complex user workflows across multiple screens

### Centralized vs Decentralized State Management

**Centralized State (Redux Pattern):**

**Real-World Example: Trading Platform**
A stock trading platform needs centralized state because:

- **Portfolio data** needs to be consistent across all components
- **Market data** updates affect multiple widgets simultaneously
- **User actions** (buy/sell) impact multiple parts of the UI
- **Real-time updates** must be synchronized across the entire application

**When Centralized State Works:**

- **Complex data relationships** where multiple components need the same data
- **Real-time applications** where data changes frequently
- **Audit requirements** where all state changes need to be tracked
- **Time-travel debugging** for complex workflows

**Decentralized State (Component State + Context):**

**Real-World Example: Blog Platform**
A blogging platform works well with decentralized state:

- **Article editor** manages its own content and autosave state
- **Comment sections** handle their own loading and submission states
- **User profile** manages its own editing state
- **Search functionality** maintains its own query and results state

**When Decentralized State Works:**

- **Independent features** that don't share much data
- **Simple data flows** without complex dependencies
- **Performance critical** applications where centralized store overhead matters
- **Team autonomy** where different teams own different features

### Server State vs Client State

**Understanding the Distinction:**

**Server State Characteristics:**

- **Owned by the server** - your application doesn't control it
- **Can change without your knowledge** - other users or systems modify it
- **Requires synchronization** - keeping client and server in sync
- **Cacheable** - can be stored temporarily for performance

**Client State Characteristics:**

- **Owned by the client** - your application controls it completely
- **Synchronous** - changes are immediate and predictable
- **Local** - doesn't need network communication
- **Temporary** - usually doesn't need persistence

**Real-World Example: Social Media Platform**

**Server State:**

- User posts and comments (other users can add/edit)
- Friend lists and follower counts (change based on other users' actions)
- Notification counts (server generates these)
- User profiles (users can update from different devices)

**Client State:**

- Form input values (draft post content)
- UI state (which modal is open, sidebar collapsed)
- Navigation state (current page, browser history)
- User preferences (theme, language settings stored locally)

### Modern State Management Patterns

**React Query / TanStack Query Pattern:**
Separates server state from client state:

- **Automatic caching** with intelligent invalidation
- **Background refetching** to keep data fresh
- **Optimistic updates** with automatic rollback on errors
- **Request deduplication** to avoid unnecessary network calls

**Real-World Implementation: Project Management Tool**

```typescript
// Server state management with React Query
const useProjectData = (projectId: string) => {
  return useQuery({
    queryKey: ["project", projectId],
    queryFn: () => fetchProject(projectId),
    staleTime: 5 * 60 * 1000, // 5 minutes
    refetchOnWindowFocus: true,
  });
};

// Client state management with Zustand
const useProjectStore = create((set) => ({
  selectedTasks: [],
  viewMode: "list",
  filterCriteria: {},
  setSelectedTasks: (tasks) => set({ selectedTasks: tasks }),
  setViewMode: (mode) => set({ viewMode: mode }),
}));
```

**State Machine Pattern (XState):**
Models application behavior as state machines:

- **Explicit states** make application behavior predictable
- **Controlled transitions** prevent impossible states
- **Side effects** are managed and testable
- **Visualization** of application behavior

**Real-World Example: Form Wizard**
A multi-step form has clear states:

- **Initial:** User hasn't started
- **Step1:** Collecting basic information
- **Step2:** Additional details
- **Validating:** Checking data on server
- **Submitting:** Sending final data
- **Success:** Form completed successfully
- **Error:** Something went wrong

Each state defines what actions are possible and where they lead.

### Performance Optimization in State Management

**The Problem with Naive State Management:**
In large applications, poor state management leads to:

- **Unnecessary re-renders** when unrelated state changes
- **Memory leaks** from subscriptions that aren't cleaned up
- **Stale data** when cache invalidation isn't handled properly
- **Performance degradation** as the application grows

**Optimization Strategies:**

**Selector Optimization:**
Instead of subscribing to entire state objects, subscribe only to the data you need:

```typescript
// Bad - component re-renders when any user data changes
const user = useSelector((state) => state.user);

// Good - component only re-renders when name changes
const userName = useSelector((state) => state.user.name);
```

**Memoization Patterns:**
Prevent expensive computations from running on every render:

```typescript
// Expensive computation that should be memoized
const expensiveValue = useMemo(() => {
  return processLargeDataset(data);
}, [data]);
```

**Virtualization for Large Lists:**
When displaying large datasets, render only visible items:

- **React Window** for simple virtualization
- **React Virtualized** for complex grid layouts
- **Custom virtualization** for specific use cases

**Real-World Example: Email Client**
An email client with thousands of emails should:

- **Virtualize the email list** to render only visible emails
- **Cache email content** to avoid refetching
- **Lazy load attachments** when emails are opened
- **Paginate search results** to limit memory usage

### Testing State Management

**Testing Strategies by Pattern:**

**Component State Testing:**

- Test component behavior with different state values
- Test state transitions from user interactions
- Mock external dependencies that affect state

**Centralized Store Testing:**

- Test reducers/actions in isolation
- Test selectors with different state shapes
- Test middleware and side effects
- Integration tests for complete workflows

**Server State Testing:**

- Mock API responses for different scenarios
- Test error handling and retry logic
- Test cache invalidation strategies
- Test optimistic update behavior

**Real-World Testing Example: Shopping Cart**

```typescript
describe("Shopping Cart", () => {
  // Test individual actions
  it("should add item to cart", () => {
    const state = cartReducer(initialState, addItem(product));
    expect(state.items).toContain(product);
  });

  // Test complex workflows
  it("should handle checkout process", async () => {
    // Setup cart with items
    // Mock payment service
    // Test success and error scenarios
    // Verify state transitions
  });

  // Test UI integration
  it("should update UI when cart changes", () => {
    // Render cart component
    // Dispatch actions
    // Verify UI updates correctly
  });
});
```

---

## Component Design Patterns

### The Psychology of Component Design

**Why Components Matter Beyond Code Organization:**
Components aren't just a technical pattern - they represent how humans think about complex systems. Just as we break down complex problems into smaller, manageable pieces, components allow us to:

- **Reason about complexity** by focusing on one piece at a time
- **Collaborate effectively** by allowing teams to own specific components
- **Maintain consistency** by reusing proven solutions
- **Evolve systems** by replacing parts without affecting the whole

### Container vs Presentational Components

**The Philosophy:**
This pattern separates "how things look" from "how things work," similar to how a theater separates actors (presentational) from directors (container).

**Real-World Example: E-commerce Product Listing**

**Container Component (ProductListContainer):**
Think of this as the "stage manager" that:

- **Fetches product data** from APIs or state management
- **Handles loading states** and error conditions
- **Manages filters and sorting** logic
- **Coordinates with other systems** (analytics, personalization)
- **Handles user interactions** that affect business logic

**Presentational Component (ProductListView):**
Think of this as the "actor" that:

- **Receives props** and renders the UI accordingly
- **Handles UI-only interactions** (hover states, animations)
- **Focuses on accessibility** and user experience
- **Contains no business logic** - purely presentation
- **Can be easily tested** with different prop combinations

**Why This Separation Matters:**

**Team Specialization:**

- **Frontend developers** can focus on user experience and visual design
- **Full-stack developers** can focus on data fetching and business logic
- **Designers** can iterate on presentational components without touching business logic

**Testing Strategy:**

- **Unit tests** for presentational components are fast and focused on UI behavior
- **Integration tests** for container components verify business logic
- **Visual regression tests** can focus on presentational components
- **End-to-end tests** verify the complete interaction

**Reusability:**

- **Presentational components** can be used in different contexts (admin panel, user dashboard)
- **Container components** can swap different presentations (mobile view, desktop view)
- **Storybook development** becomes much easier with pure presentational components

### Higher-Order Components and Composition Patterns

**The Decorator Pattern in React:**
Higher-Order Components (HOCs) are like decorators in object-oriented programming - they add functionality to existing components without modifying them.

**Real-World Example: Authentication and Authorization**

**Before HOCs (Repetitive Code):**
Every protected component needs to:

- Check if user is authenticated
- Redirect to login if not authenticated
- Show loading spinner while checking
- Handle authentication errors
- Manage refresh tokens

**With HOCs (Composition):**

```typescript
const withAuth = (WrappedComponent) => {
  return function AuthenticatedComponent(props) {
    // Authentication logic here
    if (isLoading) return <LoadingSpinner />;
    if (!isAuthenticated) return <Redirect to="/login" />;
    return <WrappedComponent {...props} />;
  };
};

// Usage
const ProtectedDashboard = withAuth(Dashboard);
const ProtectedProfile = withAuth(UserProfile);
```

**Common HOC Patterns:**

**Data Fetching HOC:**

```typescript
const withUserData = withData({
  userProfile: (props) => `/api/users/${props.userId}`,
  userPreferences: (props) => `/api/users/${props.userId}/preferences`,
});
```

**Permission HOC:**

```typescript
const withPermissions = (requiredPermissions) => (Component) => {
  return (props) => {
    const hasPermission = checkPermissions(requiredPermissions);
    if (!hasPermission) return <AccessDenied />;
    return <Component {...props} />;
  };
};
```

**Analytics HOC:**

```typescript
const withAnalytics = (eventName) => (Component) => {
  return (props) => {
    useEffect(() => {
      analytics.track(eventName, { componentProps: props });
    }, []);
    return <Component {...props} />;
  };
};
```

### Render Props and Children as Functions

**The Strategy Pattern in React:**
Render props allow components to share functionality while letting the consumer decide how to render the result.

**Real-World Example: Data Virtualization**

```typescript
const VirtualList = ({ items, itemHeight, children }) => {
  const [visibleItems, setVisibleItems] = useState([]);

  // Complex virtualization logic here

  return (
    <div className="virtual-list">
      {children({ visibleItems, scrollToIndex, isLoading })}
    </div>
  );
};

// Usage - complete control over rendering
<VirtualList items={products} itemHeight={120}>
  {({ visibleItems, scrollToIndex, isLoading }) => (
    <>
      {isLoading && <LoadingSpinner />}
      {visibleItems.map((item) => (
        <ProductCard key={item.id} product={item} />
      ))}
    </>
  )}
</VirtualList>;
```

**Benefits of Render Props:**

- **Flexibility:** Consumer controls exactly how data is rendered
- **Reusability:** Same logic can power different visual representations
- **Testability:** Logic and presentation can be tested separately
- **Composition:** Multiple render prop components can be easily combined

### Compound Components Pattern

**The Facade Pattern for UI:**
Compound components work together as a cohesive unit, similar to how HTML elements like `<select>` and `<option>` work together.

**Real-World Example: Modal Component Family**

```typescript
// Instead of a monolithic modal with many props
<Modal
  isOpen={isOpen}
  title="Edit Profile"
  showCloseButton={true}
  size="large"
  actions={[
    { label: 'Save', onClick: handleSave, variant: 'primary' },
    { label: 'Cancel', onClick: handleCancel, variant: 'secondary' }
  ]}
>
  <ProfileForm />
</Modal>

// Use compound components for flexibility
<Modal isOpen={isOpen}>
  <Modal.Header>
    <Modal.Title>Edit Profile</Modal.Title>
    <Modal.CloseButton />
  </Modal.Header>
  <Modal.Body>
    <ProfileForm />
  </Modal.Body>
  <Modal.Footer>
    <Button variant="primary" onClick={handleSave}>Save</Button>
    <Button variant="secondary" onClick={handleCancel}>Cancel</Button>
  </Modal.Footer>
</Modal>
```

**Why Compound Components Work:**

- **Declarative:** The structure is clear from the JSX
- **Flexible:** Each part can be customized or omitted
- **Maintainable:** Each sub-component has a single responsibility
- **Accessible:** Easier to implement proper ARIA relationships

### Component State Machines

**Modeling Component Behavior:**
Complex components often have intricate state relationships that are hard to manage with simple boolean flags.

**Real-World Example: File Upload Component**
Instead of managing multiple boolean states:

```typescript
const [isUploading, setIsUploading] = useState(false);
const [hasError, setHasError] = useState(false);
const [isComplete, setIsComplete] = useState(false);
const [progress, setProgress] = useState(0);
```

Use a state machine:

```typescript
const uploadStates = {
  idle: {
    on: { START_UPLOAD: "uploading" },
  },
  uploading: {
    on: {
      UPLOAD_SUCCESS: "complete",
      UPLOAD_ERROR: "error",
      UPLOAD_PROGRESS: { target: "uploading", actions: "updateProgress" },
    },
  },
  complete: {
    on: { RESET: "idle" },
  },
  error: {
    on: { RETRY: "uploading", RESET: "idle" },
  },
};
```

**Benefits of State Machines:**

- **Impossible states are impossible:** Can't be uploading and complete simultaneously
- **Predictable behavior:** Clear transitions between states
- **Easy testing:** Each state can be tested independently
- **Self-documenting:** The state machine describes the component's behavior

### Performance Optimization Patterns

**Memoization Strategies:**

**React.memo for Component Memoization:**

```typescript
// Expensive component that should only re-render when props change
const ExpensiveProductCard = React.memo(
  ({ product, onAddToCart }) => {
    // Expensive rendering logic
  },
  (prevProps, nextProps) => {
    // Custom comparison logic
    return (
      prevProps.product.id === nextProps.product.id &&
      prevProps.product.price === nextProps.product.price
    );
  }
);
```

**useMemo for Expensive Computations:**

```typescript
const ProductList = ({ products, filters, sortCriteria }) => {
  const filteredAndSortedProducts = useMemo(() => {
    return products
      .filter(applyFilters(filters))
      .sort(applySorting(sortCriteria));
  }, [products, filters, sortCriteria]);

  return (
    <div>
      {filteredAndSortedProducts.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
};
```

**useCallback for Function Memoization:**

```typescript
const ProductGrid = ({ products }) => {
  const handleProductClick = useCallback(
    (productId) => {
      analytics.track("product_clicked", { productId });
      navigate(`/products/${productId}`);
    },
    [navigate]
  ); // Only recreate if navigate changes

  return (
    <div>
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          onClick={handleProductClick}
        />
      ))}
    </div>
  );
};
```

---

## Leadership & Team Management

### Understanding Technical Leadership vs Management

**The Dual Nature of UI Architect Roles:**
As a UI Architect, you're often caught between two worlds:

- **Technical excellence:** Making the right architectural decisions
- **People leadership:** Guiding teams to implement those decisions

**Technical Leadership Responsibilities:**

**Vision Setting:**

- **Long-term technical strategy:** Where should the frontend architecture evolve over the next 2-3 years?
- **Technology evaluation:** Which frameworks, tools, and patterns should the team adopt?
- **Standards establishment:** What coding standards, review processes, and quality gates ensure consistency?
- **Trade-off communication:** How do you explain technical decisions to non-technical stakeholders?

**Real-World Example: Migration Planning**
Your company has a legacy jQuery application that needs to be modernized. As the technical leader, you need to:

- **Assess current state:** Understand the existing codebase, its limitations, and technical debt
- **Define target architecture:** Choose modern frameworks, state management, and development practices
- **Create migration strategy:** Plan incremental migration to avoid "big bang" rewrites
- **Build team consensus:** Get buy-in from developers, product managers, and executives
- **Monitor progress:** Ensure the migration stays on track and delivers business value

**People Leadership Responsibilities:**

**Mentoring and Development:**

- **Skill assessment:** Understanding each team member's strengths and growth areas
- **Career guidance:** Helping developers advance their careers and take on new challenges
- **Knowledge sharing:** Creating opportunities for team members to learn from each other
- **Feedback delivery:** Providing constructive feedback that helps people improve

**Team Dynamics:**

- **Conflict resolution:** Addressing disagreements before they impact team performance
- **Culture building:** Creating an environment where people want to do their best work
- **Communication facilitation:** Ensuring information flows effectively across the team
- **Recognition:** Celebrating successes and acknowledging individual contributions

### The Art of Influence Without Authority

**Why This Matters:**
UI Architects often need to drive change across multiple teams, departments, and even organizations without having direct management authority over the people involved.

**Building Technical Credibility:**

**Demonstrate Value Through Results:**
Instead of just talking about what should be done, show results:

- **Prototype solutions** that solve real problems teams are facing
- **Measure improvements** and share concrete metrics (performance gains, developer productivity)
- **Document learnings** and share them with the broader organization
- **Be hands-on** when needed to understand the real challenges teams face

**Real-World Example: Design System Adoption**
You want to implement a design system across multiple product teams, but you don't manage those teams:

**Wrong Approach:**

- Send emails about the importance of design consistency
- Schedule mandatory meetings to explain the design system
- Create detailed documentation and expect teams to read it
- Complain when teams don't adopt the system

**Right Approach:**

- **Start with one willing team** and help them implement the design system successfully
- **Measure the impact:** reduced development time, fewer design inconsistencies, improved user experience
- **Share success stories** in team demos and engineering all-hands
- **Make adoption easy:** provide migration guides, pair programming sessions, and ongoing support
- **Address concerns:** listen to feedback and improve the system based on real usage

**Building Relationships:**

**Understanding Motivations:**
Different people are motivated by different things:

- **Developers:** Often motivated by learning new technologies, solving interesting problems, and building quality software
- **Product Managers:** Focused on delivering features that drive business outcomes and user satisfaction
- **Designers:** Want to create great user experiences and see their designs implemented faithfully
- **Executives:** Care about business metrics, competitive advantage, and organizational efficiency

**Tailoring Communication:**

- **For Developers:** Focus on technical benefits, learning opportunities, and code quality improvements
- **For Product Managers:** Emphasize faster delivery, reduced bugs, and better user experiences
- **For Designers:** Highlight design consistency, implementation accuracy, and creative flexibility
- **For Executives:** Present business impact, competitive advantages, and risk mitigation

**Creating Win-Win Scenarios:**

**Finding Common Ground:**
Look for initiatives that benefit everyone:

- **Performance improvements** help users (better experience), developers (better tools), and business (higher conversion)
- **Developer experience improvements** help developers (more productive) and business (faster delivery)
- **Accessibility improvements** help users (inclusive experience), legal (compliance), and brand (reputation)

**Strategic Patience:**

- **Start small** with low-risk, high-impact initiatives
- **Build momentum** through early successes
- **Gradually expand** as trust and credibility grow
- **Persist through setbacks** and learn from failures

### Decision-Making Frameworks

**The Challenge of Technical Decisions:**
Technical decisions often involve trade-offs between competing priorities:

- **Performance vs Development Speed:** Optimized code takes longer to write
- **Flexibility vs Simplicity:** More flexible solutions are often more complex
- **Innovation vs Stability:** New technologies offer benefits but introduce risks
- **Short-term vs Long-term:** Quick fixes vs proper solutions

**Structured Decision-Making Process:**

**1. Define the Problem Clearly:**

- **What exactly are you trying to solve?** Be specific about the problem, not just the symptoms
- **Who is impacted?** Users, developers, business stakeholders?
- **What are the consequences of not solving this?** Technical debt, user experience issues, business risks?

**2. Identify Stakeholders and Their Concerns:**

- **Primary stakeholders:** Who will be directly affected by the decision?
- **Secondary stakeholders:** Who might be indirectly impacted?
- **Decision makers:** Who has the authority to approve or block the decision?

**3. Generate Options:**

- **Brainstorm broadly:** Don't limit yourself to obvious solutions
- **Consider hybrid approaches:** Combinations of different solutions
- **Include "do nothing" option:** Sometimes the status quo is the right choice

**4. Evaluate Options Against Criteria:**

- **Technical criteria:** Performance, maintainability, scalability, security
- **Business criteria:** Cost, time to market, risk, competitive advantage
- **Team criteria:** Skills required, learning curve, developer experience

**Real-World Decision Example: State Management Choice**

**The Problem:**
A growing e-commerce application is struggling with state management. The current approach using local component state and prop drilling is becoming unmaintainable.

**Stakeholders:**

- **Frontend Team:** Wants a solution that's easy to learn and debug
- **Product Team:** Needs features delivered quickly without introducing bugs
- **Performance Team:** Concerned about bundle size and runtime performance
- **Engineering Leadership:** Wants a solution that will scale as the team grows

**Options Considered:**

1. **Redux Toolkit:** Mature, well-documented, great debugging tools
2. **Zustand:** Smaller bundle, simpler API, less boilerplate
3. **React Query + Context:** Specialized solutions for server/client state
4. **Continue with current approach:** Invest in better patterns and tooling

**Evaluation Matrix:**
| Criteria | Redux Toolkit | Zustand | React Query + Context | Current Approach |
|----------|---------------|---------|----------------------|------------------|
| Learning Curve | Medium | Low | Medium | Low |
| Bundle Size | Large | Small | Medium | None |
| Debugging | Excellent | Good | Good | Poor |
| Ecosystem | Excellent | Growing | Excellent | N/A |
| Team Skills | Some experience | None | Some experience | Current |
| Scalability | Excellent | Good | Excellent | Poor |

**Decision Process:**

- **Present options objectively** with pros and cons of each
- **Facilitate team discussion** to understand concerns and preferences
- **Build consensus** around evaluation criteria before discussing solutions
- **Make decision based on criteria,** not personal preferences
- **Document rationale** for future reference and onboarding

**5. Implement with Feedback Loops:**

- **Start with a pilot project** to validate the decision
- **Gather feedback** from team members using the solution
- **Be willing to adjust** if the decision isn't working as expected
- **Document lessons learned** to improve future decisions

### Managing Up: Working with Engineering Leadership

**Understanding Executive Priorities:**
Engineering executives care about different things than individual contributors:

- **Business impact:** How does technical work drive business outcomes?
- **Risk management:** What could go wrong and how are you mitigating risks?
- **Resource allocation:** Are we investing in the right areas?
- **Competitive advantage:** How does our technology help us win in the market?

**Effective Communication Strategies:**

**Executive Summaries:**

- **Start with the business impact:** What problem are you solving and why it matters?
- **Provide clear recommendations:** What should be done and why?
- **Address risks and mitigation:** What could go wrong and how you're handling it?
- **Include timeline and resources:** What's needed to execute the plan?

**Regular Updates:**

- **Consistent format:** Use the same structure for all updates
- **Progress against goals:** Show concrete progress toward stated objectives
- **Escalate blockers early:** Don't wait until deadlines are missed
- **Celebrate wins:** Highlight successes and their impact

**Real-World Example: Quarterly Business Review**
Instead of presenting a list of technical achievements:

```
Q3 Accomplishments:
- Implemented micro-frontends architecture
- Migrated from Redux to Zustand
- Improved test coverage from 60% to 85%
- Updated documentation
```

Present business impact:

```
Q3 Business Impact:
- Reduced time-to-market for new features by 40% through micro-frontend architecture
- Improved developer productivity by 25% through simplified state management
- Reduced production bugs by 50% through improved test coverage
- Accelerated new team member onboarding from 4 weeks to 2 weeks

Q4 Objectives:
- Further reduce time-to-market by implementing automated deployment pipeline
- Expand micro-frontend architecture to mobile applications
- Begin migration to improve performance by 30%
```

---

## Agile Pod Management

### Understanding Pod-Based Organizations

**What Makes Pods Different from Traditional Teams:**
Pods are small, autonomous teams that own a complete business capability, not just a technical function. Think of them as mini-startups within a larger organization.

**Traditional Team Structure:**

- **Frontend Team:** Builds UI components
- **Backend Team:** Creates APIs and services
- **QA Team:** Tests everything
- **Design Team:** Creates mockups and designs
- **Product Team:** Defines requirements

**Pod Structure:**

- **Cross-functional team:** Includes frontend developer, backend developer, designer, product manager, and sometimes QA
- **Business capability ownership:** Responsible for complete user journey (e.g., checkout, user onboarding, search)
- **End-to-end accountability:** From idea to production to maintenance

**Real-World Example: Spotify's Squad Model**
Spotify organizes around autonomous squads (similar to pods):

- **Search Squad:** Owns the entire search experience from UI to algorithms to infrastructure
- **Playlist Squad:** Handles playlist creation, sharing, and discovery features
- **Artist Squad:** Manages artist profiles, analytics, and tools
- **Payment Squad:** Owns subscription management, billing, and payment processing

Each squad can make technical decisions, choose their tools, and deploy independently.

### Pod Composition and Role Definition

**Optimal Pod Size: The "Two Pizza Rule"**
Amazon's Jeff Bezos famously said teams should be small enough to be fed by two pizzas. For UI-focused pods, this typically means 5-8 people:

**Core Roles:**

**UI Architect (Technical Leader):**

- **Responsible for:** Technical vision, architecture decisions, code quality, team mentoring
- **Time allocation:** 40% hands-on coding, 30% architecture and planning, 20% mentoring, 10% stakeholder communication
- **Success metrics:** Team velocity, code quality scores, developer satisfaction, technical debt trends

**Senior Frontend Developer (Technical Expert):**

- **Responsible for:** Complex feature implementation, junior developer mentoring, technical spikes
- **Time allocation:** 60% coding, 20% mentoring, 15% technical planning, 5% documentation
- **Success metrics:** Feature delivery quality, junior developer growth, technical innovation

**Frontend Developers (2-3 people):**

- **Mid-level (2-4 years experience):** Feature implementation, code reviews, testing
- **Junior-level (0-2 years experience):** Simple features, learning, pair programming

**Product Manager (Business Owner):**

- **Responsible for:** Feature prioritization, stakeholder communication, business metrics, user research
- **Time allocation:** 40% planning and prioritization, 30% stakeholder management, 20% user research, 10% data analysis

**UX/UI Designer:**

- **Responsible for:** User experience design, prototyping, user testing, design system maintenance
- **Time allocation:** 50% design work, 20% user research, 20% prototyping, 10% design system contribution

**Optional Roles (depending on pod focus):**

- **QA Engineer:** For complex domains requiring extensive testing
- **DevOps Engineer:** For infrastructure-heavy pods
- **Data Analyst:** For metrics-driven features

### Pod Autonomy and Decision-Making

**The Principle of Subsidiarity:**
Decisions should be made at the lowest possible level where they can be made effectively. This means:

- **Technical decisions** (frameworks, patterns, tools) are made by the pod
- **Business decisions** (feature priorities, user experience) are made by product manager with team input
- **Cross-pod decisions** (shared services, design systems) require coordination
- **Organizational decisions** (hiring, budgets, strategic direction) are made by leadership

**Real-World Decision Framework:**

**Pod-Level Decisions:**

- **Technology choices:** Which state management library to use
- **Implementation approaches:** How to structure components and services
- **Testing strategies:** What types of tests to write and when
- **Code quality standards:** Linting rules, code review processes
- **Sprint planning:** How to break down and estimate work

**Cross-Pod Coordination:**

- **Shared libraries:** Design system components, utility functions
- **API contracts:** Interfaces between different business capabilities
- **Performance standards:** Page load times, bundle size limits
- **Security policies:** Authentication, authorization, data handling

**Organizational Decisions:**

- **Technology platform choices:** React vs Vue vs Angular
- **Infrastructure decisions:** Cloud providers, deployment strategies
- **Hiring and team structure:** Pod composition and growth
- **Budget allocation:** Resource distribution across pods

### Communication Patterns and Ceremonies

**Daily Operations:**

**Pod Standup (Daily, 15 minutes):**

- **Format:** Each person shares yesterday's progress, today's plan, and any blockers
- **Focus:** Coordination within the pod, not status reporting to management
- **Facilitation:** Rotate facilitation among team members to build leadership skills
- **Outcomes:** Clear understanding of pod's daily priorities and immediate blockers

**Real-World Example:**
"Yesterday I finished the user authentication flow and found an issue with our error handling. Today I'm going to pair with Sarah on fixing that and then start on the password reset feature. I'm blocked on getting the API documentation for the new endpoint."

**Sprint Planning (Every 2 weeks, 2-4 hours):**

- **Part 1:** Review sprint goal and available capacity
- **Part 2:** Break down user stories into technical tasks
- **Part 3:** Estimate effort and commit to sprint backlog
- **Outcome:** Clear sprint goal and committed work with realistic estimates

**Sprint Review/Demo (End of sprint, 1 hour):**

- **Audience:** Pod members plus key stakeholders
- **Format:** Demo completed features and gather feedback
- **Focus:** Business value delivered and user experience
- **Outcome:** Stakeholder feedback and input for next sprint planning

**Retrospective (End of sprint, 1 hour):**

- **Format:** What went well, what could be improved, action items
- **Focus:** Process improvement and team dynamics
- **Facilitation:** Different team member each sprint
- **Outcome:** 1-3 concrete action items for the next sprint

**Weekly Coordination:**

**Cross-Pod Sync (Weekly, 30 minutes):**

- **Participants:** Representatives from each pod (usually pod leads)
- **Purpose:** Coordinate cross-pod dependencies and share learnings
- **Format:** Round-robin updates and discussion of blockers
- **Outcome:** Aligned understanding of cross-pod work and resolved dependencies

**Architecture Review (Monthly, 1-2 hours):**

- **Participants:** UI Architects from all pods plus senior engineers
- **Purpose:** Share architectural decisions, discuss patterns, plan improvements
- **Format:** Present recent decisions, discuss challenges, plan shared initiatives
- **Outcome:** Consistent architectural direction and shared learning

### Managing Pod Performance

**Metrics That Matter:**

**Delivery Metrics:**

- **Sprint goal achievement:** Percentage of sprint goals completed
- **Story point velocity:** Consistent delivery over time (not higher is always better)
- **Cycle time:** Time from story start to production deployment
- **Lead time:** Time from idea to user value delivery

**Quality Metrics:**

- **Production incident rate:** Bugs that affect users
- **Code review feedback loops:** Time from PR creation to merge
- **Technical debt trend:** Increasing or decreasing over time
- **Test coverage:** Percentage of code covered by automated tests

**Team Health Metrics:**

- **Team satisfaction:** Regular surveys about workload, collaboration, and growth
- **Knowledge distribution:** How well knowledge is shared across team members
- **Learning and growth:** Individual development goals and progress
- **Retention rate:** Team stability and voluntary turnover

**Real-World Performance Optimization:**

**When Velocity is Inconsistent:**

- **Investigate estimation accuracy:** Are stories being estimated consistently?
- **Look for external dependencies:** Is the pod blocked by other teams?
- **Examine technical debt:** Is poor code quality slowing development?
- **Check team dynamics:** Are communication or collaboration issues affecting productivity?

**When Quality is Suffering:**

- **Review code review process:** Are reviews thorough enough? Too slow?
- **Examine test coverage:** Are the right tests being written?
- **Look at technical practices:** Is pair programming needed? Better documentation?
- **Check workload:** Is the team under too much pressure to deliver?

### Scaling Pod Organizations

**Growing from One Pod to Many:**

**First Pod (Months 1-6):**

- **Focus:** Establish practices, prove the model works
- **Challenges:** Learning new ways of working, building cross-functional skills
- **Success criteria:** Consistent delivery, good team dynamics, stakeholder satisfaction

**Multiple Pods (Months 6-18):**

- **Focus:** Coordination between pods, shared practices, avoiding duplication
- **Challenges:** Communication overhead, inconsistent practices, technical dependencies
- **Solutions:** Regular cross-pod sync, shared tools and standards, clear interfaces

**Scaled Pod Organization (18+ months):**

- **Focus:** Autonomous operation, minimal coordination overhead, consistent culture
- **Challenges:** Maintaining culture, avoiding divergence, coordinating large initiatives
- **Solutions:** Strong shared culture, clear architectural principles, effective leadership structure

**Common Scaling Challenges:**

**Knowledge Silos:**

- **Problem:** Each pod develops specialized knowledge that isn't shared
- **Solution:** Regular knowledge sharing sessions, documentation standards, cross-pod rotation

**Technical Divergence:**

- **Problem:** Pods choose different solutions for similar problems
- **Solution:** Architecture review process, shared component libraries, technology radar

**Communication Overhead:**

- **Problem:** More pods means more coordination complexity
- **Solution:** Clear interfaces, asynchronous communication, minimal dependencies

---

## Stakeholder Management

### Understanding Your Stakeholder Ecosystem

**The Complex Web of Relationships:**
As a UI Architect, you're at the center of a complex web of relationships, each with different priorities, communication styles, and success metrics.

**Primary Stakeholders:**

**Engineering Teams:**

- **Frontend Developers:** Want clear architecture guidance, good developer experience, and opportunities to learn
- **Backend Developers:** Need well-defined APIs, consistent data models, and coordination on features
- **DevOps Engineers:** Care about deployment processes, monitoring, and infrastructure requirements
- **QA Engineers:** Need testable code, clear requirements, and efficient testing processes

**Product Organization:**

- **Product Managers:** Focus on user value, business metrics, and competitive features
- **Product Designers:** Want their designs implemented accurately and efficiently
- **User Researchers:** Need features that actually solve user problems
- **Product Marketing:** Care about features that can be effectively communicated to customers

**Business Leadership:**

- **Engineering Managers:** Balance technical excellence with business delivery
- **VP of Engineering:** Focus on organizational capability and technical risk
- **CEO/CTO:** Care about competitive advantage and business outcomes
- **Customer Success:** Want features that increase customer satisfaction and retention

**External Stakeholders:**

- **Customers:** Want features that solve their problems effectively
- **Partners:** Need stable APIs and integration points
- **Security/Compliance Teams:** Require adherence to security and regulatory standards
- **Legal Teams:** Need intellectual property protection and compliance documentation

### Stakeholder Communication Strategies

**Tailoring Your Message:**

**For Engineers (Technical Depth):**

- **Focus on:** Technical benefits, implementation challenges, learning opportunities
- **Communication style:** Detailed technical discussion, code examples, architecture diagrams
- **Frequency:** Regular (daily standups, weekly reviews)
- **Format:** Technical documentation, code reviews, design sessions

**Example Communication:**
"The new state management approach will reduce boilerplate by 60% and improve debugging through better dev tools. Here's how it handles edge cases we've struggled with, and here's a migration guide for existing components."

**For Product Teams (Business Value):**

- **Focus on:** User impact, feature delivery speed, business outcomes
- **Communication style:** User-focused benefits, delivery timelines, risk mitigation
- **Frequency:** Regular alignment (weekly syncs, sprint planning)
- **Format:** Product planning meetings, feature demos, roadmap reviews

**Example Communication:**
"This architecture change will let us deliver checkout improvements 40% faster, reduce cart abandonment bugs by half, and make it easier to add payment methods that customers are requesting."

**For Executives (Strategic Impact):**

- **Focus on:** Business outcomes, competitive advantage, risk management
- **Communication style:** Executive summaries, clear recommendations, ROI analysis
- **Frequency:** Quarterly reviews, major decision points
- **Format:** Executive presentations, written reports, strategic planning sessions

**Example Communication:**
"Our frontend modernization initiative will reduce time-to-market for new features by 50%, improve customer satisfaction scores by 20%, and position us to enter new markets faster than competitors."

### Managing Competing Priorities

**The Reality of Conflicting Demands:**
Different stakeholders often want incompatible things:

- **Product wants:** More features, faster delivery, unique capabilities
- **Engineering wants:** Technical debt reduction, refactoring, tool improvements
- **Business wants:** Lower costs, reduced risk, proven technologies
- **Users want:** Better performance, easier workflows, fewer bugs

**Framework for Priority Resolution:**

**1. Understand the "Why" Behind Each Request:**

- **Product's perspective:** "We need this feature to compete with [competitor]"
- **Engineering's perspective:** "We need to fix this technical debt before it slows us down"
- **Business perspective:** "We need to reduce our infrastructure costs"
- **User perspective:** "The current workflow is confusing and error-prone"

**2. Find the Underlying Business Objective:**
Often, different requests are actually trying to solve the same underlying problem:

- **Faster feature delivery:** Could be solved by technical debt reduction OR better tools OR process improvements
- **Better user experience:** Could be solved by performance improvements OR design updates OR bug fixes
- **Cost reduction:** Could be achieved through technical efficiency OR process optimization OR tool consolidation

**3. Create Win-Win Solutions:**
Look for solutions that address multiple stakeholder needs:

- **Performance improvements:** Help users (better experience), business (higher conversion), and engineering (better tools)
- **Design system implementation:** Help designers (consistent implementation), developers (reusable components), and product (faster delivery)
- **Automated testing:** Help QA (better coverage), developers (faster feedback), and business (reduced bugs)

**Real-World Example: The Great Mobile Rewrite Debate**

**The Situation:**

- **Product team:** Wants a mobile app to compete with competitors
- **Engineering team:** Wants to rewrite the web app with modern technology
- **Business team:** Wants to minimize costs and risks
- **Customer success:** Reports that users are frustrated with mobile web experience

**The Competing Proposals:**

1. **Native mobile apps** (Product's preference)
2. **Modern web app rewrite** (Engineering's preference)
3. **Incremental improvements** (Business's preference)

**The Win-Win Solution:**
Progressive Web App (PWA) with modern architecture:

- **For Product:** Mobile app-like experience that can be "installed" on phones
- **For Engineering:** Opportunity to modernize architecture without separate codebase
- **For Business:** One codebase to maintain, faster time to market
- **For Users:** Better mobile experience with offline capability

**Implementation Strategy:**

- **Phase 1:** Modernize core web app with PWA capabilities
- **Phase 2:** Add mobile-specific features and optimizations
- **Phase 3:** Evaluate native apps based on real usage data

### Building Long-Term Relationships

**Trust Building Strategies:**

**Consistent Communication:**

- **Regular updates:** Share progress, challenges, and wins consistently
- **Transparent about problems:** Don't hide issues until they become crises
- **Follow through:** Do what you say you're going to do
- **Proactive communication:** Share relevant information before being asked

**Value Delivery:**

- **Quick wins:** Identify and deliver small improvements that provide immediate value
- **Long-term vision:** Show how current work fits into bigger picture
- **Measure impact:** Track and share metrics that matter to stakeholders
- **Continuous improvement:** Regularly evaluate and improve based on feedback

**Professional Growth:**

- **Learn their domain:** Understand business context, user needs, and technical constraints
- **Speak their language:** Adapt communication style to audience
- **Build empathy:** Understand the pressures and challenges each group faces
- **Offer solutions:** Don't just identify problems, propose actionable solutions

**Real-World Relationship Building:**

**With Product Teams:**

- **Attend user research sessions** to understand real user needs
- **Participate in competitive analysis** to understand market pressures
- **Learn business metrics** that drive product decisions
- **Offer technical alternatives** when product requirements seem impossible

**With Business Leadership:**

- **Understand company strategy** and how technology enables it
- **Learn financial basics** (revenue models, cost structures, profitability)
- **Track industry trends** that might affect technology choices
- **Present options** with clear trade-offs and recommendations

**With Engineering Teams:**

- **Stay technical** and continue contributing to code when appropriate
- **Understand team challenges** and work to remove blockers
- **Provide learning opportunities** and career growth support
- **Advocate upward** for team needs and technical investments

---

## Project Recovery Strategies

### Recognizing Projects in Trouble

**Early Warning Signs:**
Projects don't usually fail overnight - they show warning signs that experienced architects learn to recognize:

**Technical Indicators:**

- **Increasing bug rates:** More issues reported in each release
- **Slowing velocity:** Teams consistently missing sprint commitments
- **Growing technical debt:** Quick fixes accumulating faster than they're addressed
- **Performance degradation:** Application getting slower over time
- **Test coverage declining:** Tests not keeping up with new code

**Team Indicators:**

- **High stress levels:** Team members working excessive hours
- **Increased conflicts:** More disagreements about technical approaches
- **Knowledge silos:** Critical knowledge held by only one or two people
- **Burnout symptoms:** Decreased engagement, increased sick days
- **High turnover:** Key team members leaving or expressing dissatisfaction

**Process Indicators:**

- **Missed deadlines:** Consistently failing to meet commitments
- **Scope creep:** Requirements changing frequently without timeline adjustments
- **Poor communication:** Stakeholders surprised by delays or issues
- **Quality shortcuts:** Skipping code reviews, testing, or documentation
- **No retrospectives:** Team not improving processes or learning from mistakes

**Business Indicators:**

- **Stakeholder frustration:** Product managers or executives expressing concerns
- **Customer complaints:** Users reporting issues or requesting missing features
- **Competitive pressure:** Falling behind competitors in key capabilities
- **Budget overruns:** Project costing more than planned without proportional value
- **Lost confidence:** Leadership questioning team's ability to deliver

### Assessment and Diagnosis

**The Project Health Audit:**

**Technical Assessment:**
Conduct a thorough technical review:

- **Code quality analysis:** Use tools like SonarQube to identify technical debt
- **Performance audit:** Measure current performance against targets
- **Architecture review:** Evaluate if current architecture can support business goals
- **Security assessment:** Identify vulnerabilities and compliance gaps
- **Dependency analysis:** Understand external dependencies and risks

**Team Assessment:**
Understand the human factors:

- **Skill gap analysis:** What capabilities does the team need vs what they have?
- **Workload analysis:** Are people overloaded or blocked?
- **Communication patterns:** How effectively is information flowing?
- **Motivation levels:** What's driving or demotivating team members?
- **Knowledge distribution:** How well is knowledge shared across the team?

**Process Assessment:**
Evaluate development practices:

- **Development workflow:** How smooth is the path from idea to production?
- **Quality processes:** Are code review, testing, and deployment processes effective?
- **Project management:** How well are requirements, timelines, and risks managed?
- **Stakeholder engagement:** How effectively are expectations managed?
- **Continuous improvement:** Is the team learning and adapting?

**Real-World Assessment Example:**

**The Situation:** E-commerce platform missing major holiday deadline
**Technical Findings:**

- **Performance:** Page load times increased 300% over 6 months
- **Quality:** 40% increase in production bugs, test coverage dropped to 45%
- **Architecture:** Monolithic architecture couldn't handle traffic spikes
- **Dependencies:** Critical features blocked by third-party API limitations

**Team Findings:**

- **Skills:** Team lacked experience with performance optimization
- **Workload:** Senior developers spending 60% of time on bug fixes
- **Communication:** Frontend and backend teams not coordinating effectively
- **Morale:** Team working 60+ hour weeks, high stress levels

**Process Findings:**

- **Planning:** Requirements changing weekly without timeline adjustments
- **Quality:** Code reviews being skipped to "save time"
- **Testing:** Manual testing only, no automated test suite
- **Deployment:** Manual deployment process taking 6+ hours

### Recovery Planning and Execution

**The Triage Approach:**
Like emergency medicine, project recovery requires triage - addressing the most critical issues first while stabilizing the patient.

**Immediate Stabilization (Week 1-2):**

**Stop the Bleeding:**

- **Halt all non-critical feature work** and focus on stability
- **Implement immediate bug fixes** for critical user-facing issues
- **Add monitoring and alerting** to understand current system behavior
- **Create war room** for coordinated response to production issues
- **Establish clear communication** with stakeholders about the situation

**Quick Wins:**

- **Performance improvements:** Cache static assets, optimize database queries
- **Bug fixes:** Address high-impact, low-complexity issues
- **Monitoring improvements:** Add logging and metrics to understand problems
- **Process improvements:** Implement basic code review for all changes

**Team Stabilization:**

- **Workload management:** Limit overtime and focus on sustainable pace
- **Clear priorities:** Everyone understands what's most important
- **Support:** Bring in additional resources if needed
- **Communication:** Daily standups to coordinate emergency response

**Short-Term Recovery (Weeks 3-8):**

**Technical Debt Reduction:**

- **Prioritized debt paydown:** Focus on debt that's blocking progress
- **Architecture improvements:** Address fundamental design issues
- **Testing infrastructure:** Build automated test suite for critical paths
- **Documentation:** Document critical processes and system components

**Team Building:**

- **Skill development:** Provide training in areas where team is weak
- **Knowledge sharing:** Ensure critical knowledge is distributed
- **Process improvement:** Implement sustainable development practices
- **Morale building:** Celebrate wins and acknowledge hard work

**Stakeholder Alignment:**

- **Expectation reset:** Honest communication about timeline and scope
- **Regular updates:** Transparent progress reporting
- **Success metrics:** Clear definition of what recovery looks like
- **Risk management:** Identify and plan for remaining risks

**Long-Term Sustainability (Months 3-6):**

**Architectural Evolution:**

- **Scalability improvements:** Prepare system for future growth
- **Technology modernization:** Upgrade frameworks and tools
- **Platform thinking:** Build reusable components and services
- **Performance optimization:** Systematic optimization based on data

**Team Development:**

- **Career planning:** Help team members grow their skills
- **Culture building:** Establish practices that prevent future crises
- **Knowledge management:** Create systems for sharing and preserving knowledge
- **Recruitment:** Hire additional talent where needed

**Process Maturation:**

- **Agile practices:** Implement sustainable development methodologies
- **Quality processes:** Automated testing, continuous integration, code quality metrics
- **Risk management:** Regular assessment and mitigation of project risks
- **Continuous improvement:** Regular retrospectives and process evolution

### Real-World Recovery Case Study

**The Crisis: Social Media Platform Performance Collapse**

**Background:**
A social media platform for professionals was experiencing severe performance issues:

- **User growth:** 500% increase in users over 6 months
- **Performance:** Page load times increased from 2 seconds to 30+ seconds
- **Reliability:** Daily outages affecting 10,000+ users
- **Team state:** 12-person team working 80+ hour weeks, 3 senior developers quit

**Week 1-2: Emergency Response**

```
Immediate Actions:
- Implemented CDN for static assets (40% performance improvement)
- Added database read replicas (reduced query load by 60%)
- Created on-call rotation to handle production issues
- Paused all new feature development

Results:
- Page load times reduced to 8-12 seconds
- Outages reduced from daily to 2-3 per week
- Team stress levels decreased with clear priorities
```

**Weeks 3-8: Systematic Recovery**

```
Technical Improvements:
- Implemented database query optimization (50% improvement)
- Added Redis caching layer (30% improvement)
- Migrated to microservices architecture for user service
- Built comprehensive monitoring dashboard

Team Improvements:
- Hired 2 senior developers with scaling experience
- Implemented pair programming for knowledge transfer
- Started weekly architecture review sessions
- Reduced work weeks to sustainable 45-50 hours

Process Improvements:
- Implemented automated testing (coverage increased to 80%)
- Added code review requirements for all changes
- Created incident response playbook
- Established regular stakeholder communication
```

**Months 3-6: Long-term Sustainability**

```
Platform Evolution:
- Completed migration to microservices architecture
- Implemented automated scaling based on load
- Built real-time monitoring and alerting system
- Page load times consistently under 3 seconds

Team Development:
- All team members completed advanced training
- Knowledge documentation reduced single points of failure
- Team satisfaction scores increased from 2.1/5 to 4.2/5
- Zero voluntary turnover in 6 months

Business Impact:
- User engagement increased 40% due to better performance
- Customer support tickets reduced by 60%
- Time-to-market for new features improved by 50%
- Platform supported 10x user growth without performance issues
```

**Key Lessons Learned:**

1. **Act quickly on the obvious issues** while you plan longer-term solutions
2. **Team sustainability is just as important** as technical fixes
3. **Transparent communication** builds trust even during difficult times
4. **Invest in monitoring and tooling** early in the recovery process
5. **Cultural changes** are necessary to prevent recurring crises

**Prevention Strategies:**
Based on this experience, the team implemented preventive measures:

- **Performance budgets:** Automatic alerts when metrics degrade
- **Capacity planning:** Quarterly reviews of system capacity vs projected growth
- **Stress testing:** Regular load testing of critical user flows
- **Technical debt tracking:** Monthly reviews with explicit paydown planning
- **Team health metrics:** Regular surveys and proactive intervention

---

## Advanced POD Management & Leadership Patterns

### 🎯 Advanced POD Management Strategies

#### **POD Maturity Model**

Understanding where your PODs are in their evolution helps tailor your leadership approach:

**Level 1: Forming PODs (Months 1-3)**

- **Characteristics:** Team learning to work cross-functionally, establishing practices
- **Leadership Focus:** Clear structure, defined roles, basic processes
- **Success Metrics:** Completing planned work, establishing team rhythm
- **Common Challenges:** Role confusion, process overhead, communication gaps

**Level 2: Performing PODs (Months 4-12)**

- **Characteristics:** Consistent delivery, good collaboration, some autonomy
- **Leadership Focus:** Coaching, removing impediments, optimizing processes
- **Success Metrics:** Predictable delivery, improving quality, stakeholder satisfaction
- **Common Challenges:** Scaling coordination, maintaining quality under pressure

**Level 3: High-Performing PODs (Year 2+)**

- **Characteristics:** Self-organizing, innovative, driving business outcomes
- **Leadership Focus:** Strategic guidance, capability building, organizational influence
- **Success Metrics:** Business impact, technical innovation, team growth
- **Common Challenges:** Avoiding complacency, maintaining edge, scaling influence

**Real-World Example: POD Evolution at Scale**

```typescript
// POD Maturity Assessment Framework
interface PODMaturityMetrics {
  autonomy: {
    decisionMaking: number; // 1-10 scale
    problemSolving: number;
    processImprovement: number;
  };
  delivery: {
    predictability: number;
    quality: number;
    businessValue: number;
  };
  collaboration: {
    internalCohesion: number;
    externalAlignment: number;
    knowledgeSharing: number;
  };
  innovation: {
    technicalExcellence: number;
    processInnovation: number;
    businessInnovation: number;
  };
}

class PODMaturityTracker {
  private pods: Map<string, PODMaturityMetrics> = new Map();

  assessPOD(podId: string): PODMaturityLevel {
    const metrics = this.pods.get(podId);
    if (!metrics) return PODMaturityLevel.FORMING;

    const averageScore = this.calculateAverageScore(metrics);

    if (averageScore >= 8) return PODMaturityLevel.HIGH_PERFORMING;
    if (averageScore >= 6) return PODMaturityLevel.PERFORMING;
    return PODMaturityLevel.FORMING;
  }

  getGrowthRecommendations(podId: string): GrowthAction[] {
    const metrics = this.pods.get(podId);
    const level = this.assessPOD(podId);

    return this.generateRecommendations(metrics, level);
  }
}
```

#### **POD Leadership Patterns**

**1. Servant Leadership Pattern**

**What it is:** Leaders serve the team by removing obstacles and enabling success
**When to use:** With experienced teams that need autonomy and support
**Implementation:**

```typescript
class ServantLeadershipApproach {
  // Daily focus: What can I do to help the team succeed today?
  dailyServantActions = {
    removeBlockers: async () => {
      const blockers = await this.identifyTeamBlockers();
      return Promise.all(blockers.map((b) => this.resolveBlocker(b)));
    },

    facilitateDecisions: (decision: Decision) => {
      return this.gatherInput(decision)
        .then((input) => this.facilitateConsensus(input))
        .then((consensus) => this.documentDecision(consensus));
    },

    provideContext: (teamMember: TeamMember) => {
      return this.shareBusinessContext()
        .then((context) => this.explainDecisionRationale(context))
        .then(() => this.answerQuestions(teamMember));
    },
  };
}
```

**Real-World Example:**
"I noticed the team is blocked on the API design review. Let me schedule time with the backend architect today so we can move forward. Also, I'll document the decision criteria so future API reviews go faster."

**2. Coach Leadership Pattern**

**What it is:** Developing team members' skills and decision-making capabilities
**When to use:** With growing teams or when developing future leaders
**Implementation:**

```typescript
interface CoachingSession {
  teamMember: string;
  skillArea: string;
  currentLevel: number;
  targetLevel: number;
  actionPlan: Action[];
  followUpDate: Date;
}

class CoachingLeadershipApproach {
  private coachingSessions: CoachingSession[] = [];

  conductSkillAssessment(teamMember: TeamMember): SkillGaps {
    return {
      technical: this.assessTechnicalSkills(teamMember),
      leadership: this.assessLeadershipPotential(teamMember),
      communication: this.assessCommunicationSkills(teamMember),
      problemSolving: this.assessProblemSolving(teamMember),
    };
  }

  createGrowthPlan(teamMember: TeamMember, skillGaps: SkillGaps): GrowthPlan {
    return {
      shortTermGoals: this.defineShortTermObjectives(skillGaps),
      learningResources: this.recommendResources(skillGaps),
      practiceOpportunities: this.identifyPracticeOpportunities(skillGaps),
      mentorshipPairing: this.assignMentor(teamMember, skillGaps),
    };
  }
}
```

**Practical Coaching Techniques:**

- **Socratic Questioning:** "What do you think would happen if we chose approach A vs B?"
- **Pair Leadership:** Have junior members shadow you in stakeholder meetings
- **Progressive Responsibility:** Gradually increase decision-making authority
- **Reflection Sessions:** Regular 1:1s focused on learning and growth

**3. Situational Leadership Pattern**

**What it is:** Adapting leadership style based on team maturity and situation
**When to use:** Always - different team members and situations need different approaches
**Implementation:**

```typescript
enum LeadershipStyle {
  DIRECTING = "directing", // High direction, low support
  COACHING = "coaching", // High direction, high support
  SUPPORTING = "supporting", // Low direction, high support
  DELEGATING = "delegating", // Low direction, low support
}

class SituationalLeadership {
  determineStyle(teamMember: TeamMember, task: Task): LeadershipStyle {
    const competence = this.assessCompetence(teamMember, task);
    const commitment = this.assessCommitment(teamMember, task);

    if (competence === "low" && commitment === "high") {
      return LeadershipStyle.DIRECTING;
    }
    if (competence === "low" && commitment === "low") {
      return LeadershipStyle.COACHING;
    }
    if (competence === "high" && commitment === "low") {
      return LeadershipStyle.SUPPORTING;
    }
    return LeadershipStyle.DELEGATING;
  }
}
```

**Example Applications:**

- **New team member on complex feature:** Directing (clear instructions, frequent check-ins)
- **Experienced developer on new technology:** Coaching (guidance + support)
- **Expert who's lost motivation:** Supporting (encouragement, remove obstacles)
- **High performer on routine task:** Delegating (clear outcome, full autonomy)

#### **POD Conflict Resolution Strategies**

**The Conflict Resolution Framework:**

**1. Technical Disagreements**

```typescript
interface TechnicalConflict {
  participants: string[];
  issue: string;
  proposedSolutions: Solution[];
  businessImpact: string;
  timeConstraints: string;
}

class TechnicalConflictResolver {
  resolveConflict(conflict: TechnicalConflict): Resolution {
    // Step 1: Establish shared criteria
    const criteria = this.establishDecisionCriteria(conflict);

    // Step 2: Objective evaluation
    const evaluation = this.evaluateSolutions(
      conflict.proposedSolutions,
      criteria
    );

    // Step 3: Facilitate discussion
    const consensus = this.facilitateDiscussion(
      conflict.participants,
      evaluation
    );

    return {
      decision: consensus.selectedSolution,
      rationale: consensus.reasoning,
      nextReviewDate: this.scheduleReview(consensus),
    };
  }
}
```

**Real-World Example: State Management Debate**

**The Conflict:** Team split between Redux and Zustand for new features
**Resolution Process:**

```
1. Established Criteria:
   - Learning curve for junior developers
   - Bundle size impact
   - Debugging capabilities
   - Team familiarity
   - Long-term maintainability

2. Objective Evaluation:
   - Created proof-of-concept with both solutions
   - Measured bundle size impact
   - Surveyed team on learning preferences
   - Analyzed debugging workflows

3. Facilitated Discussion:
   - Presented evaluation results objectively
   - Heard concerns from each side
   - Found hybrid approach: Zustand for new features,
     Redux for complex state

4. Outcome:
   - Decision documented with rationale
   - 3-month review scheduled
   - Migration guide created
   - Team training planned
```

**2. Process Disagreements**

**Common Process Conflicts:**

- Code review rigor vs speed
- Testing coverage vs delivery pressure
- Documentation detail vs efficiency
- Meeting frequency vs focus time

**Resolution Approach:**

```typescript
interface ProcessConflict {
  process: string;
  stakeholders: string[];
  currentState: ProcessState;
  proposedChanges: ProcessChange[];
  metrics: ProcessMetrics;
}

class ProcessOptimization {
  optimizeProcess(conflict: ProcessConflict): ProcessSolution {
    // Measure current effectiveness
    const baseline = this.measureCurrentProcess(conflict.process);

    // Pilot proposed changes
    const pilots = this.runPilotPrograms(conflict.proposedChanges);

    // Compare results
    const comparison = this.compareResults(baseline, pilots);

    return this.selectOptimalProcess(comparison);
  }
}
```

#### **Cross-POD Coordination Patterns**

**1. API First Pattern**

**What it is:** PODs define interfaces before implementation
**Benefits:** Parallel development, clear contracts, easier testing

```typescript
// Cross-POD API Contract Definition
interface UserServiceContract {
  version: string;
  endpoints: {
    getUserProfile: {
      input: GetUserProfileRequest;
      output: GetUserProfileResponse;
      errors: UserServiceError[];
    };
    updateUserProfile: {
      input: UpdateUserProfileRequest;
      output: UpdateUserProfileResponse;
      errors: UserServiceError[];
    };
  };
  events: {
    userProfileUpdated: UserProfileUpdatedEvent;
    userDeleted: UserDeletedEvent;
  };
}

// Contract-driven development process
class CrossPODCoordination {
  establishContract(pods: POD[]): ServiceContract {
    // 1. Collaborative design sessions
    const requirements = this.gatherRequirements(pods);

    // 2. Draft contract together
    const draftContract = this.draftContract(requirements);

    // 3. Review and iterate
    const finalContract = this.reviewAndIterate(draftContract, pods);

    // 4. Generate documentation and mocks
    this.generateArtifacts(finalContract);

    return finalContract;
  }
}
```

**2. Event-Driven Coordination Pattern**

**What it is:** PODs coordinate through events rather than direct calls
**Benefits:** Loose coupling, scalability, easier testing

```typescript
interface PODEvent {
  eventType: string;
  source: string;
  data: any;
  timestamp: Date;
  correlationId: string;
}

class EventDrivenCoordination {
  private eventBus: EventBus;

  publishEvent(event: PODEvent): void {
    this.eventBus.publish(event);
  }

  subscribeToEvents(eventTypes: string[], handler: EventHandler): void {
    this.eventBus.subscribe(eventTypes, handler);
  }
}

// Example: E-commerce order processing across PODs
const orderEvents = {
  orderCreated: "order.created",
  paymentProcessed: "payment.processed",
  inventoryReserved: "inventory.reserved",
  shipmentScheduled: "shipment.scheduled",
};

// Each POD handles relevant events
class CheckoutPOD {
  handleOrderCreated(event: PODEvent) {
    // Process payment
    this.processPayment(event.data.orderId);
  }
}

class InventoryPOD {
  handlePaymentProcessed(event: PODEvent) {
    // Reserve inventory
    this.reserveInventory(event.data.orderId);
  }
}
```

**3. Shared Services Pattern**

**What it is:** Common capabilities provided as shared services
**Benefits:** Consistency, reduced duplication, specialized expertise

```typescript
// Shared service examples
interface SharedServices {
  authentication: AuthenticationService;
  logging: LoggingService;
  analytics: AnalyticsService;
  notifications: NotificationService;
  configuration: ConfigurationService;
}

class SharedServiceGovernance {
  private services: Map<string, SharedService> = new Map();

  registerService(service: SharedService): void {
    this.validateServiceContract(service);
    this.ensureServiceQuality(service);
    this.publishServiceDocumentation(service);
    this.services.set(service.name, service);
  }

  evolveService(serviceName: string, newVersion: ServiceVersion): void {
    const migrationPlan = this.createMigrationPlan(serviceName, newVersion);
    this.coordinateWithConsumers(migrationPlan);
    this.executeMigration(migrationPlan);
  }
}
```

#### **POD Performance Optimization**

**1. Flow Metrics Pattern**

**What it is:** Measuring work flow rather than just activity
**Benefits:** Focus on value delivery, identify bottlenecks

```typescript
interface FlowMetrics {
  leadTime: number; // Idea to production
  cycleTime: number; // Start work to done
  workInProgress: number; // Active items
  throughput: number; // Items completed per period
  flowEfficiency: number; // Active time / total time
}

class FlowMetricsTracker {
  private workItems: WorkItem[] = [];

  calculateFlowMetrics(timeframe: Timeframe): FlowMetrics {
    const completedItems = this.getCompletedItems(timeframe);

    return {
      leadTime: this.calculateAverageLeadTime(completedItems),
      cycleTime: this.calculateAverageCycleTime(completedItems),
      workInProgress: this.getCurrentWIP(),
      throughput: completedItems.length / timeframe.weeks,
      flowEfficiency: this.calculateFlowEfficiency(completedItems),
    };
  }

  identifyBottlenecks(): Bottleneck[] {
    const stateDistribution = this.analyzeStateDistribution();
    return stateDistribution
      .filter((state) => state.averageTime > this.threshold)
      .map((state) => ({
        stage: state.name,
        averageTime: state.averageTime,
        improvementOpportunities: this.suggestImprovements(state),
      }));
  }
}
```

**2. Continuous Improvement Pattern**

**What it is:** Regular, systematic process improvements
**Benefits:** Sustainable performance gains, team engagement

```typescript
interface ImprovementExperiment {
  hypothesis: string;
  metrics: string[];
  duration: number;
  expectedImpact: number;
  actualImpact?: number;
  learnings: string[];
  decision: "adopt" | "adapt" | "abandon";
}

class ContinuousImprovement {
  private experiments: ImprovementExperiment[] = [];

  proposeExperiment(improvement: ImprovementIdea): ImprovementExperiment {
    return {
      hypothesis: improvement.description,
      metrics: this.defineSuccessMetrics(improvement),
      duration: this.estimateDuration(improvement),
      expectedImpact: improvement.estimatedBenefit,
    };
  }

  runExperiment(experiment: ImprovementExperiment): ExperimentResults {
    const baseline = this.measureBaseline(experiment.metrics);

    // Implement change for limited time
    this.implementChange(experiment);

    // Measure results
    const results = this.measureResults(
      experiment.metrics,
      experiment.duration
    );

    return this.analyzeResults(baseline, results);
  }
}
```

**Example Improvements:**

- **Pair Programming Experiment:** Does pair programming on complex features reduce bugs?
- **Async Code Review:** Can we maintain quality while reducing review latency?
- **Focused Time Blocks:** Does protecting 2-hour focus blocks improve throughput?
- **Cross-POD Shadowing:** Does shadowing other PODs improve collaboration?

### 🎯 Leadership Anti-Patterns to Avoid

**1. The Hero Architect**

- **Pattern:** Solving all technical problems personally
- **Problem:** Creates dependency, doesn't develop team
- **Solution:** Coach others to solve problems, step back gradually

**2. The Perfectionist**

- **Pattern:** Requiring perfection before any release
- **Problem:** Paralysis, missed opportunities, team frustration
- **Solution:** Define "good enough" criteria, iterate

**3. The Ivory Tower**

- **Pattern:** Making decisions without understanding ground reality
- **Problem:** Impractical solutions, team disconnection
- **Solution:** Stay close to the code, regular team work

**4. The Micromanager**

- **Pattern:** Controlling every technical decision
- **Problem:** Reduces team autonomy and growth
- **Solution:** Define boundaries, delegate within them

---

## 🎯 UI Architect Leadership Interview Guide

### **Comprehensive Interview Framework for UI Architect Leadership Roles**

This guide covers both technical architecture competence and leadership capabilities essential for senior UI architect positions.

#### **Interview Structure Overview**

**Phase 1: Technical Foundation (45 minutes)**

- Architectural design and decision-making
- System design and scalability
- Technology evaluation and adoption

**Phase 2: Leadership & Management (45 minutes)**

- Team leadership and POD management
- Stakeholder communication and influence
- Conflict resolution and decision-making

**Phase 3: Strategic Thinking (30 minutes)**

- Vision and roadmap development
- Business alignment and value delivery
- Innovation and continuous improvement

---

### **Phase 1: Technical Foundation Questions**

#### **Architectural Design & Decision Making**

**Q1: System Design Scenario**
_"Design the frontend architecture for a multi-tenant SaaS platform that serves both small businesses (< 100 users) and enterprises (10,000+ users). Walk me through your approach."_

**What to look for:**

- Systematic thinking and structured approach
- Consideration of scalability, performance, and maintainability
- Discussion of trade-offs and alternatives
- Understanding of multi-tenancy challenges

**Strong Answer Indicators:**

```typescript
// Example response structure
interface ArchitecturalApproach {
  analysis: {
    requirements: BusinessRequirement[];
    constraints: TechnicalConstraint[];
    assumptions: Assumption[];
  };

  design: {
    architecturalPattern: string; // Micro-frontends, modular monolith, etc.
    stateManagement: StateStrategy;
    routing: RoutingStrategy;
    dataFetching: DataStrategy;
    authentication: AuthStrategy;
    multiTenancy: TenancyStrategy;
  };

  scalingStrategy: {
    caching: CachingStrategy;
    codesplitting: SplittingStrategy;
    performance: PerformanceStrategy;
    monitoring: MonitoringStrategy;
  };

  evolution: {
    migrationPath: MigrationPlan;
    technicalDebtManagement: DebtStrategy;
    futureConsiderations: EvolutionPlan;
  };
}
```

**Q2: Technology Evaluation**
_"Your team is currently using Redux for state management, but some developers want to switch to Zustand or React Query. How would you approach this decision?"_

**What to look for:**

- Structured decision-making process
- Consideration of multiple stakeholder perspectives
- Understanding of migration complexity
- Balance between innovation and stability

**Strong Answer Framework:**

1. **Understand the Problem:** Why is change being considered?
2. **Stakeholder Analysis:** Who is affected and how?
3. **Evaluation Criteria:** Performance, learning curve, maintainability, etc.
4. **Proof of Concept:** Practical evaluation with real scenarios
5. **Migration Strategy:** How to transition safely
6. **Decision Documentation:** Rationale for future reference

**Q3: Legacy System Modernization**
_"You've inherited a large jQuery-based application that needs to be modernized. The business wants new features but the current codebase is hard to maintain. What's your approach?"_

**What to look for:**

- Understanding of strangler pattern and incremental migration
- Risk assessment and mitigation
- Business value delivery during transition
- Team capability assessment

**Expected Discussion Points:**

- Assessment methodology for current system
- Incremental migration strategies (strangler pattern, micro-frontends)
- Risk mitigation approaches
- Team training and capability building
- Measurement of progress and success

#### **System Design & Scalability**

**Q4: Performance Under Load**
_"Your e-commerce application is experiencing slow performance during peak shopping periods. Walk me through your diagnostic and optimization approach."_

**What to look for:**

- Systematic debugging methodology
- Understanding of performance bottlenecks
- Both short-term fixes and long-term solutions
- Measurement and monitoring practices

**Strong Answer Structure:**

```typescript
interface PerformanceOptimization {
  diagnosis: {
    dataGathering: MetricsCollection;
    bottleneckIdentification: PerformanceAnalysis;
    userImpactAssessment: UserExperienceMetrics;
  };

  shortTermFixes: {
    quickWins: OptimizationAction[];
    impactEstimation: PerformanceGain[];
  };

  longTermStrategy: {
    architecturalChanges: ArchitectureImprovement[];
    infrastructureUpgrades: InfrastructureChange[];
    processImprovements: ProcessOptimization[];
  };

  monitoring: {
    metrics: PerformanceMetric[];
    alerting: AlertingStrategy;
    continuousOptimization: OptimizationProcess;
  };
}
```

### **Phase 2: Leadership & Management Questions**

#### **Team Leadership & POD Management**

**Q5: Building High-Performing Teams**
_"You're leading a newly formed POD with a mix of senior and junior developers. How do you build them into a high-performing team?"_

**What to look for:**

- Understanding of team development stages
- Concrete strategies for skill development
- Approaches to building team culture
- Methods for measuring team health

**Strong Answer Components:**

```typescript
interface TeamBuildingStrategy {
  assessment: {
    individualSkills: SkillAssessment[];
    teamDynamics: TeamAnalysis;
    workingStyles: CommunicationPreferences[];
  };

  development: {
    skillGrowthPlans: LearningPlan[];
    mentorshipPrograms: MentorshipStrategy;
    knowledgeSharing: KnowledgeTransfer;
    crossTraining: SkillDiversification;
  };

  cultureBuilding: {
    sharedValues: TeamValues;
    workingAgreements: TeamNorms;
    celebrationRituals: SuccessRecognition;
    communicationPatterns: CollaborationFramework;
  };

  measurement: {
    teamHealthMetrics: HealthIndicator[];
    performanceMetrics: DeliveryMetrics;
    satisfactionTracking: FeedbackMechanism;
  };
}
```

**Q6: Conflict Resolution**
_"Two senior developers in your POD strongly disagree about architectural approach for a critical feature. The deadline is approaching and tensions are rising. How do you handle this?"_

**What to look for:**

- Structured conflict resolution approach
- Understanding of different perspectives
- Focus on objective criteria over personal preferences
- Leadership in decision-making when consensus isn't possible

**Expected Process:**

1. **Immediate De-escalation:** Separate discussion from emotion
2. **Understand Positions:** What each person is actually advocating for
3. **Identify Underlying Interests:** Why they believe their approach is better
4. **Establish Decision Criteria:** Objective measures for evaluation
5. **Facilitate Evaluation:** Help team assess options against criteria
6. **Make Decision:** Lead decision if consensus isn't reached
7. **Document and Move Forward:** Clear communication of rationale

**Q7: Remote/Distributed Team Management**
_"Your POD is distributed across three time zones. How do you maintain team cohesion and effective collaboration?"_

**What to look for:**

- Understanding of distributed team challenges
- Concrete strategies for asynchronous collaboration
- Tools and processes for distributed work
- Cultural awareness and inclusion

#### **Stakeholder Communication & Influence**

**Q8: Managing Up**
_"Your engineering director wants to rewrite the entire frontend in a new framework, but you believe incremental improvement would be better for the business. How do you handle this disagreement?"_

**What to look for:**

- Ability to present technical recommendations in business terms
- Understanding of different stakeholder perspectives
- Influence without authority
- Constructive disagreement with leadership

**Strong Approach:**

- **Understand the Director's Concerns:** What problem are they trying to solve?
- **Present Business Case:** ROI, risk analysis, timeline comparison
- **Propose Alternative:** Incremental approach with measurable milestones
- **Suggest Pilot Program:** Prove value with limited scope
- **Maintain Relationship:** Respectful disagreement, collaborative solution

**Q9: Cross-Functional Alignment**
_"Product managers are pressuring for faster feature delivery, while QA is concerned about quality. How do you balance these competing demands?"_

**What to look for:**

- Understanding of different functional priorities
- Ability to find win-win solutions
- Facilitation of cross-functional discussions
- Systems thinking about overall optimization

### **Phase 3: Strategic Thinking Questions**

#### **Vision & Roadmap Development**

**Q10: Long-term Technical Strategy**
_"Create a 3-year technical vision for a growing startup's frontend that expects to scale from 10,000 to 1 million users."_

**What to look for:**

- Strategic thinking and planning
- Understanding of scaling challenges
- Balance between current needs and future requirements
- Consideration of team growth and capabilities

**Expected Framework:**

```typescript
interface TechnicalVision {
  currentState: {
    userBase: number;
    teamSize: number;
    technicalCapabilities: Capability[];
    businessConstraints: Constraint[];
  };

  futureState: {
    targetScale: ScaleRequirements;
    requiredCapabilities: Capability[];
    organizationalStructure: TeamStructure;
  };

  roadmap: {
    year1: {
      priorities: StrategicPriority[];
      investments: TechnicalInvestment[];
      risks: Risk[];
    };
    year2: {
      // Similar structure
    };
    year3: {
      // Similar structure
    };
  };

  successMetrics: {
    technical: TechnicalMetric[];
    business: BusinessMetric[];
    team: TeamMetric[];
  };
}
```

#### **Innovation & Continuous Improvement**

**Q11: Technology Innovation Balance**
_"How do you balance innovation and experimentation with stability and delivery pressures?"_

**What to look for:**

- Understanding of innovation frameworks
- Risk management in technology adoption
- Practical approaches to experimentation
- Measurement of innovation impact

**Q12: Organizational Influence**
_"You want to implement a design system across multiple teams, but you don't have direct authority over those teams. How do you drive adoption?"_

**What to look for:**

- Influence without authority
- Change management understanding
- Coalition building
- Value demonstration

---

### **Evaluation Rubric**

#### **Technical Competence (40%)**

**Excellent (4/4):**

- Demonstrates deep architectural understanding
- Considers multiple solutions with clear trade-offs
- Shows experience with scale and complexity
- Thinks systematically about technical debt and evolution

**Good (3/4):**

- Solid technical foundation
- Understands architectural principles
- Can design reasonable solutions
- Considers some trade-offs and constraints

**Developing (2/4):**

- Basic technical knowledge
- Limited architectural thinking
- Solutions lack depth or consideration of alternatives
- Minimal understanding of scale or complexity

**Inadequate (1/4):**

- Weak technical foundation
- Cannot design coherent solutions
- No understanding of architectural principles
- Lacks practical experience

#### **Leadership & Communication (35%)**

**Excellent (4/4):**

- Clear communication adapted to audience
- Demonstrates successful team leadership experience
- Shows emotional intelligence and conflict resolution skills
- Can influence and align stakeholders

**Good (3/4):**

- Good communication skills
- Some leadership experience
- Understands team dynamics
- Can work with stakeholders

**Developing (2/4):**

- Basic communication skills
- Limited leadership experience
- Some understanding of team challenges
- Struggles with stakeholder management

**Inadequate (1/4):**

- Poor communication
- No leadership experience
- Cannot handle team challenges
- Cannot work effectively with stakeholders

#### **Strategic Thinking (25%)**

**Excellent (4/4):**

- Thinks strategically about technology and business alignment
- Can create coherent long-term vision
- Understands innovation management
- Shows business acumen

**Good (3/4):**

- Some strategic thinking capability
- Can plan for future needs
- Understands business context
- Some innovation experience

**Developing (2/4):**

- Limited strategic perspective
- Focuses mainly on short-term needs
- Basic business understanding
- Little innovation experience

**Inadequate (1/4):**

- No strategic thinking
- Cannot plan beyond immediate needs
- No business context understanding
- No innovation experience

---

### **Red Flags to Watch For**

**Technical Red Flags:**

- Cannot explain architectural decisions clearly
- Focuses only on latest technologies without understanding trade-offs
- No experience with scale or complexity
- Cannot discuss technical debt management

**Leadership Red Flags:**

- Blames team members for failures
- Cannot give examples of difficult conversations or conflicts
- Shows no evidence of developing others
- Talks only about individual contributions, not team success

**Communication Red Flags:**

- Cannot adapt explanation to different audiences
- Becomes defensive when questioned
- Cannot acknowledge mistakes or limitations
- Interrupts or dismisses interviewer concerns

**Strategic Red Flags:**

- Cannot connect technical decisions to business value
- No long-term thinking or planning experience
- Cannot discuss innovation or change management
- Focuses only on technical aspects, ignoring organizational impact

---

### **Follow-up Questions for Deeper Assessment**

**For Technical Depth:**

- "What would you do differently if you designed this again?"
- "How would this solution change if requirements doubled?"
- "What monitoring would you implement for this system?"
- "How would you migrate users from the old to new system?"

**For Leadership Insight:**

- "Tell me about a time when you had to make an unpopular decision"
- "How do you handle a team member who consistently misses deadlines?"
- "Describe a situation where you changed your mind after team feedback"
- "How do you know if your team is happy and productive?"

**For Strategic Understanding:**

- "How do you prioritize technical debt against new features?"
- "What's your approach to evaluating new technologies?"
- "How do you measure the success of architectural changes?"
- "How do you align technical roadmap with business strategy?"

This comprehensive interview guide provides a framework for assessing both the technical depth and leadership capability essential for senior UI architect roles. The key is to look for evidence of real experience, thoughtful decision-making, and the ability to balance technical excellence with business value delivery.
