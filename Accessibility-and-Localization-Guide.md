# 🌐 Complete Guide to Accessibility and Localization with HTML5, CSS3, and Angular

## 📋 Table of Contents

1. [Introduction to Accessibility & Localization](#introduction)
2. [HTML5 Accessibility Features](#html5-accessibility)
3. [CSS3 for Accessible Design](#css3-accessibility)
4. [Angular Accessibility Implementation](#angular-accessibility)
5. [Localization (i18n) Fundamentals](#localization-fundamentals)
6. [Angular Internationalization](#angular-i18n)
7. [Advanced Accessibility Patterns](#advanced-patterns)
8. [Testing & Tools](#testing-tools)
9. [Best Practices & Guidelines](#best-practices)

---

## 🎯 Introduction to Accessibility & Localization {#introduction}

### What is Web Accessibility?

Web accessibility ensures that websites and applications are usable by people with disabilities, including:

- **Visual impairments** (blindness, low vision, color blindness)
- **Hearing impairments** (deafness, hard of hearing)
- **Motor impairments** (limited fine motor control, paralysis)
- **Cognitive impairments** (dyslexia, ADHD, autism)

### What is Localization?

Localization (l10n) is the process of adapting software for different languages, regions, and cultures, including:

- **Language translation**
- **Cultural adaptation**
- **Date/time formats**
- **Number formats**
- **Currency formats**
- **Text direction (LTR/RTL)**

### Why They Matter Together

```typescript
// Example: A truly inclusive application considers both accessibility and localization
interface InclusiveApp {
  accessibility: {
    screenReaderSupport: boolean;
    keyboardNavigation: boolean;
    colorContrast: string; // 'AA' | 'AAA'
    focusManagement: boolean;
  };
  localization: {
    languages: string[]; // ['en', 'es', 'ar', 'zh']
    rtlSupport: boolean;
    culturalAdaptation: boolean;
    dateTimeFormats: Record<string, string>;
  };
}
```

---

## 🏗️ HTML5 Accessibility Features {#html5-accessibility}

### Semantic HTML Elements

HTML5 introduced semantic elements that provide meaning and structure, making content more accessible to screen readers and other assistive technologies.

#### Basic Semantic Structure

```html
<!DOCTYPE html>
<html lang="en">
  <!-- Always specify language for screen readers -->
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Accessible Page Title - Company Name</title>
    <!-- Title should be descriptive and unique for each page -->
  </head>
  <body>
    <!-- Main navigation landmark -->
    <nav role="navigation" aria-label="Main navigation">
      <ul>
        <li><a href="#main" class="skip-link">Skip to main content</a></li>
        <li><a href="/" aria-current="page">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>

    <!-- Main content landmark -->
    <main id="main" role="main">
      <!-- Page header section -->
      <header>
        <h1>Page Title</h1>
        <!-- Only one h1 per page -->
        <p>Brief description of page content</p>
      </header>

      <!-- Main content sections -->
      <section aria-labelledby="section1-heading">
        <h2 id="section1-heading">Section Title</h2>
        <p>Section content...</p>
      </section>

      <!-- Article for standalone content -->
      <article aria-labelledby="article-heading">
        <h2 id="article-heading">Article Title</h2>
        <p>Article content that could stand alone...</p>
      </article>

      <!-- Aside for supplementary content -->
      <aside aria-label="Related information">
        <h3>Related Links</h3>
        <ul>
          <li><a href="/related-1">Related Article 1</a></li>
          <li><a href="/related-2">Related Article 2</a></li>
        </ul>
      </aside>
    </main>

    <!-- Footer landmark -->
    <footer role="contentinfo">
      <p>&copy; 2025 Company Name. All rights reserved.</p>
    </footer>
  </body>
</html>
```

#### Form Accessibility

```html
<!-- Accessible form with proper labeling and validation -->
<form novalidate>
  <!-- Use novalidate to provide custom validation messages -->
  <fieldset>
    <legend>Personal Information</legend>
    <!-- Legend describes the group -->

    <!-- Text input with explicit label -->
    <div class="form-group">
      <label for="first-name">
        First Name <span aria-label="required">*</span>
      </label>
      <input
        type="text"
        id="first-name"
        name="firstName"
        required
        aria-describedby="first-name-error first-name-help"
        aria-invalid="false"
        autocomplete="given-name"
      />
      <div id="first-name-help" class="help-text">
        Enter your legal first name as it appears on official documents
      </div>
      <div
        id="first-name-error"
        class="error-message"
        role="alert"
        aria-live="polite"
      >
        <!-- Error message will be announced by screen readers when populated -->
      </div>
    </div>

    <!-- Email input with validation -->
    <div class="form-group">
      <label for="email">
        Email Address <span aria-label="required">*</span>
      </label>
      <input
        type="email"
        id="email"
        name="email"
        required
        aria-describedby="email-error email-help"
        autocomplete="email"
        pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
      />
      <div id="email-help" class="help-text">
        We'll use this to send you important updates
      </div>
      <div
        id="email-error"
        class="error-message"
        role="alert"
        aria-live="polite"
      ></div>
    </div>

    <!-- Select dropdown with proper labeling -->
    <div class="form-group">
      <label for="country">Country</label>
      <select id="country" name="country" aria-describedby="country-help">
        <option value="">Choose a country</option>
        <option value="US">United States</option>
        <option value="CA">Canada</option>
        <option value="UK">United Kingdom</option>
        <option value="AU">Australia</option>
      </select>
      <div id="country-help" class="help-text">
        Select your country of residence
      </div>
    </div>

    <!-- Checkbox with proper association -->
    <div class="form-group">
      <input
        type="checkbox"
        id="newsletter"
        name="newsletter"
        aria-describedby="newsletter-help"
      />
      <label for="newsletter"> Subscribe to our newsletter </label>
      <div id="newsletter-help" class="help-text">
        Get monthly updates about new features and tips
      </div>
    </div>

    <!-- Radio button group -->
    <fieldset>
      <legend>Preferred contact method</legend>
      <div class="radio-group">
        <input
          type="radio"
          id="contact-email"
          name="contactMethod"
          value="email"
        />
        <label for="contact-email">Email</label>

        <input
          type="radio"
          id="contact-phone"
          name="contactMethod"
          value="phone"
        />
        <label for="contact-phone">Phone</label>

        <input
          type="radio"
          id="contact-mail"
          name="contactMethod"
          value="mail"
        />
        <label for="contact-mail">Postal Mail</label>
      </div>
    </fieldset>
  </fieldset>

  <!-- Form submission buttons -->
  <div class="form-actions">
    <button type="submit" class="btn-primary">Submit Application</button>
    <button type="button" class="btn-secondary" onclick="resetForm()">
      Clear Form
    </button>
  </div>
</form>
```

#### Accessible Tables

```html
<!-- Complex data table with proper headers and scope -->
<table role="table" aria-label="Quarterly Sales Report">
  <caption>
    Quarterly Sales Report for 2025
    <details>
      <summary>Table Description</summary>
      <p>
        This table shows sales data across different regions and quarters.
        Navigate using arrow keys or Tab. Headers identify each cell's context.
      </p>
    </details>
  </caption>

  <thead>
    <tr>
      <th scope="col" id="region-header">Region</th>
      <th scope="col" id="q1-header">Q1 2025</th>
      <th scope="col" id="q2-header">Q2 2025</th>
      <th scope="col" id="q3-header">Q3 2025</th>
      <th scope="col" id="total-header">Total</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <th scope="row" id="north-america">North America</th>
      <td headers="north-america q1-header">$125,000</td>
      <td headers="north-america q2-header">$134,000</td>
      <td headers="north-america q3-header">$142,000</td>
      <td headers="north-america total-header"><strong>$401,000</strong></td>
    </tr>
    <tr>
      <th scope="row" id="europe">Europe</th>
      <td headers="europe q1-header">$98,000</td>
      <td headers="europe q2-header">$105,000</td>
      <td headers="europe q3-header">$112,000</td>
      <td headers="europe total-header"><strong>$315,000</strong></td>
    </tr>
    <tr>
      <th scope="row" id="asia-pacific">Asia Pacific</th>
      <td headers="asia-pacific q1-header">$87,000</td>
      <td headers="asia-pacific q2-header">$94,000</td>
      <td headers="asia-pacific q3-header">$101,000</td>
      <td headers="asia-pacific total-header"><strong>$282,000</strong></td>
    </tr>
  </tbody>

  <tfoot>
    <tr>
      <th scope="row">Total All Regions</th>
      <td><strong>$310,000</strong></td>
      <td><strong>$333,000</strong></td>
      <td><strong>$355,000</strong></td>
      <td><strong>$998,000</strong></td>
    </tr>
  </tfoot>
</table>
```

### ARIA (Accessible Rich Internet Applications) Attributes

#### Common ARIA Attributes

```html
<!-- ARIA Landmarks for navigation -->
<div role="banner">Header content</div>
<div role="navigation" aria-label="Primary">Navigation menu</div>
<div role="main">Main content</div>
<div role="complementary">Sidebar content</div>
<div role="contentinfo">Footer content</div>

<!-- ARIA Labels and Descriptions -->
<button aria-label="Close dialog">×</button>
<input aria-describedby="password-help" type="password" />
<div id="password-help">Password must be at least 8 characters</div>

<!-- ARIA States -->
<button aria-pressed="false">Toggle Button</button>
<div aria-expanded="false">Collapsible content</div>
<input aria-invalid="true" aria-describedby="error-msg" />

<!-- ARIA Live Regions for dynamic content -->
<div aria-live="polite" id="status-message">
  <!-- Status updates will be announced -->
</div>
<div aria-live="assertive" id="error-alerts">
  <!-- Critical errors will interrupt screen reader -->
</div>
```

#### Custom ARIA Components

```html
<!-- Accessible Custom Dropdown -->
<div
  class="dropdown"
  role="combobox"
  aria-expanded="false"
  aria-haspopup="listbox"
>
  <button
    id="dropdown-button"
    aria-labelledby="dropdown-label"
    aria-controls="dropdown-list"
    class="dropdown-toggle"
  >
    Select an option
    <span aria-hidden="true">▼</span>
    <!-- Decorative icon hidden from screen readers -->
  </button>

  <label id="dropdown-label" class="dropdown-label">
    Choose your preferred language
  </label>

  <ul
    id="dropdown-list"
    role="listbox"
    aria-labelledby="dropdown-label"
    class="dropdown-list hidden"
  >
    <li role="option" aria-selected="false" tabindex="-1">English</li>
    <li role="option" aria-selected="false" tabindex="-1">Spanish</li>
    <li role="option" aria-selected="false" tabindex="-1">French</li>
    <li role="option" aria-selected="true" tabindex="0">German</li>
  </ul>
</div>

<!-- Accessible Tab Panel -->
<div class="tab-container">
  <div role="tablist" aria-label="Settings categories">
    <button
      role="tab"
      aria-selected="true"
      aria-controls="panel-general"
      id="tab-general"
      tabindex="0"
    >
      General
    </button>
    <button
      role="tab"
      aria-selected="false"
      aria-controls="panel-privacy"
      id="tab-privacy"
      tabindex="-1"
    >
      Privacy
    </button>
    <button
      role="tab"
      aria-selected="false"
      aria-controls="panel-security"
      id="tab-security"
      tabindex="-1"
    >
      Security
    </button>
  </div>

  <div
    role="tabpanel"
    id="panel-general"
    aria-labelledby="tab-general"
    tabindex="0"
  >
    <h3>General Settings</h3>
    <p>Configure your general preferences here.</p>
    <!-- General settings content -->
  </div>

  <div
    role="tabpanel"
    id="panel-privacy"
    aria-labelledby="tab-privacy"
    tabindex="0"
    hidden
  >
    <h3>Privacy Settings</h3>
    <p>Manage your privacy preferences.</p>
    <!-- Privacy settings content -->
  </div>

  <div
    role="tabpanel"
    id="panel-security"
    aria-labelledby="tab-security"
    tabindex="0"
    hidden
  >
    <h3>Security Settings</h3>
    <p>Configure security options.</p>
    <!-- Security settings content -->
  </div>
</div>

<!-- Accessible Modal Dialog -->
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
  class="modal"
  id="confirm-modal"
>
  <div class="modal-content">
    <header class="modal-header">
      <h2 id="modal-title">Confirm Action</h2>
      <button
        aria-label="Close dialog"
        class="modal-close"
        onclick="closeModal()"
      >
        ×
      </button>
    </header>

    <div class="modal-body">
      <p id="modal-description">
        Are you sure you want to delete this item? This action cannot be undone.
      </p>
    </div>

    <footer class="modal-footer">
      <button class="btn-danger" onclick="confirmDelete()">Delete</button>
      <button class="btn-secondary" onclick="closeModal()">Cancel</button>
    </footer>
  </div>
</div>
```

---

## 🎨 CSS3 for Accessible Design {#css3-accessibility}

### Color and Contrast

Proper color contrast is essential for users with visual impairments. WCAG 2.1 requires minimum contrast ratios.

#### Contrast Requirements

```css
/* WCAG AA Compliance - Minimum contrast ratios:
   - Normal text: 4.5:1
   - Large text (18pt+ or 14pt+ bold): 3:1
   - UI components and graphics: 3:1
*/

:root {
  /* Color palette with accessible contrast ratios */
  --primary-color: #1976d2; /* Blue - 4.51:1 contrast on white */
  --primary-dark: #0d47a1; /* Dark blue - 8.59:1 contrast on white */
  --secondary-color: #388e3c; /* Green - 4.54:1 contrast on white */
  --error-color: #d32f2f; /* Red - 5.04:1 contrast on white */
  --warning-color: #f57c00; /* Orange - 3.05:1 contrast on white */
  --text-primary: #212121; /* Almost black - 16.10:1 contrast on white */
  --text-secondary: #757575; /* Gray - 4.61:1 contrast on white */
  --background-light: #ffffff;
  --background-dark: #121212;

  /* Focus indicators */
  --focus-color: #005fcc;
  --focus-outline: 2px solid var(--focus-color);
  --focus-shadow: 0 0 0 3px rgba(0, 95, 204, 0.3);
}

/* High contrast theme for users who need it */
@media (prefers-contrast: high) {
  :root {
    --primary-color: #000080; /* Higher contrast blue */
    --text-primary: #000000; /* Pure black */
    --text-secondary: #333333; /* Darker gray */
    --border-color: #000000;
  }
}

/* Accessible button styles with proper contrast */
.btn {
  /* Base button styles */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.75rem 1.5rem;
  border: 2px solid transparent;
  border-radius: 4px;
  font-family: inherit;
  font-size: 1rem;
  font-weight: 500;
  line-height: 1.2;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease-in-out;

  /* Focus management */
  outline: none; /* Remove default outline */
}

/* Primary button with accessible contrast */
.btn-primary {
  background-color: var(--primary-color);
  color: white; /* 4.51:1 contrast ratio */
  border-color: var(--primary-color);
}

.btn-primary:hover {
  background-color: var(--primary-dark);
  border-color: var(--primary-dark);
  /* Hover maintains contrast ratio */
}

.btn-primary:focus {
  /* Visible focus indicator for keyboard users */
  outline: var(--focus-outline);
  outline-offset: 2px;
  box-shadow: var(--focus-shadow);
}

.btn-primary:active {
  background-color: #0d47a1;
  transform: translateY(1px); /* Subtle feedback */
}

/* Disabled state should still be distinguishable */
.btn-primary:disabled {
  background-color: #cccccc; /* 2.84:1 contrast - clearly disabled */
  color: #666666;
  border-color: #cccccc;
  cursor: not-allowed;
  opacity: 0.6;
}

/* Error button with high contrast */
.btn-error {
  background-color: var(--error-color);
  color: white; /* 5.04:1 contrast ratio */
  border-color: var(--error-color);
}

.btn-error:hover {
  background-color: #b71c1c;
}

.btn-error:focus {
  outline: 2px solid var(--error-color);
  outline-offset: 2px;
  box-shadow: 0 0 0 3px rgba(211, 47, 47, 0.3);
}
```

#### Color-Independent Design

```css
/* Design that doesn't rely solely on color to convey information */

/* Status indicators using multiple visual cues */
.status {
  display: inline-flex;
  align-items: center;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-weight: 500;
  font-size: 0.875rem;
}

/* Success status - green color + checkmark icon + text */
.status-success {
  background-color: #e8f5e9;
  color: #2e7d32;
  border: 1px solid #4caf50;
}

.status-success::before {
  content: "✓ "; /* Checkmark provides visual confirmation */
  font-weight: bold;
  margin-right: 0.25rem;
}

/* Error status - red color + X icon + text */
.status-error {
  background-color: #ffebee;
  color: #c62828;
  border: 1px solid #f44336;
}

.status-error::before {
  content: "✗ "; /* X mark provides visual indication */
  font-weight: bold;
  margin-right: 0.25rem;
}

/* Warning status - orange color + exclamation + text */
.status-warning {
  background-color: #fff3e0;
  color: #ef6c00;
  border: 1px solid #ff9800;
}

.status-warning::before {
  content: "⚠ "; /* Warning symbol */
  font-weight: bold;
  margin-right: 0.25rem;
}

/* Required field indicators - not just red asterisk */
.form-label.required {
  position: relative;
}

.form-label.required::after {
  content: " (required)"; /* Text indicator */
  color: var(--error-color);
  font-size: 0.875rem;
  font-weight: normal;
}

/* Alternative: using both color and symbol */
.form-label.required-symbol::after {
  content: " *";
  color: var(--error-color);
  font-weight: bold;
  margin-left: 0.25rem;
}

/* Form validation - multiple indicators */
.input-error {
  border: 2px solid var(--error-color);
  background-color: #ffebee;
  /* Adding background pattern for additional indication */
  background-image: linear-gradient(
    45deg,
    transparent 40%,
    rgba(211, 47, 47, 0.1) 40%,
    rgba(211, 47, 47, 0.1) 60%,
    transparent 60%
  );
  background-size: 20px 20px;
}

.input-success {
  border: 2px solid var(--secondary-color);
  background-color: #e8f5e9;
}

/* Links that are distinguishable without color */
a {
  color: var(--primary-color);
  text-decoration: underline; /* Always underline links */
  text-underline-offset: 2px;
  text-decoration-thickness: 1px;
}

a:hover {
  text-decoration-thickness: 2px;
  text-underline-offset: 3px;
}

a:focus {
  outline: var(--focus-outline);
  outline-offset: 2px;
  background-color: rgba(25, 118, 210, 0.1);
}

/* Charts and graphs - patterns instead of just colors */
.chart-bar {
  transition: all 0.3s ease;
}

.chart-bar.series-1 {
  fill: var(--primary-color);
  stroke: #0d47a1;
  stroke-width: 1;
}

.chart-bar.series-2 {
  fill: var(--secondary-color);
  stroke: #2e7d32;
  stroke-width: 1;
  /* Adding pattern for colorblind users */
  fill-opacity: 0.8;
  stroke-dasharray: 3, 2;
}

.chart-bar.series-3 {
  fill: var(--warning-color);
  stroke: #ef6c00;
  stroke-width: 1;
  /* Different pattern */
  stroke-dasharray: 5, 5;
}
```

### Responsive Typography

```css
/* Accessible typography that scales properly */

/* Responsive font sizes using clamp() for better readability */
:root {
  /* Base font size - 16px minimum for accessibility */
  --font-size-xs: clamp(0.75rem, 0.7rem + 0.25vw, 0.875rem); /* 12-14px */
  --font-size-sm: clamp(0.875rem, 0.8rem + 0.375vw, 1rem); /* 14-16px */
  --font-size-base: clamp(1rem, 0.9rem + 0.5vw, 1.125rem); /* 16-18px */
  --font-size-lg: clamp(1.125rem, 1rem + 0.625vw, 1.25rem); /* 18-20px */
  --font-size-xl: clamp(1.25rem, 1.1rem + 0.75vw, 1.5rem); /* 20-24px */
  --font-size-2xl: clamp(1.5rem, 1.3rem + 1vw, 2rem); /* 24-32px */
  --font-size-3xl: clamp(2rem, 1.7rem + 1.5vw, 3rem); /* 32-48px */

  /* Line heights for optimal readability */
  --line-height-tight: 1.25;
  --line-height-normal: 1.5; /* WCAG recommends 1.5 for body text */
  --line-height-relaxed: 1.75;

  /* Letter spacing for improved readability */
  --letter-spacing-tight: -0.025em;
  --letter-spacing-normal: 0;
  --letter-spacing-wide: 0.025em;
}

/* Base typography styles */
html {
  font-size: 100%; /* Respect user's browser font size settings */
  scroll-behavior: smooth; /* Smooth scrolling for better UX */
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    "Helvetica Neue", Arial, sans-serif;
  font-size: var(--font-size-base);
  line-height: var(--line-height-normal);
  color: var(--text-primary);
  background-color: var(--background-light);

  /* Improve text rendering */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

/* Headings with proper hierarchy and spacing */
h1,
h2,
h3,
h4,
h5,
h6 {
  font-weight: 600;
  line-height: var(--line-height-tight);
  letter-spacing: var(--letter-spacing-tight);
  margin-top: 0;
  margin-bottom: 0.5em;
  color: var(--text-primary);
}

h1 {
  font-size: var(--font-size-3xl);
  margin-bottom: 0.75em;
}

h2 {
  font-size: var(--font-size-2xl);
  margin-top: 1.5em;
}

h3 {
  font-size: var(--font-size-xl);
  margin-top: 1.25em;
}

h4 {
  font-size: var(--font-size-lg);
  margin-top: 1em;
}

h5 {
  font-size: var(--font-size-base);
  margin-top: 1em;
}

h6 {
  font-size: var(--font-size-sm);
  margin-top: 1em;
  text-transform: uppercase;
  letter-spacing: var(--letter-spacing-wide);
}

/* Paragraph spacing for readability */
p {
  margin-top: 0;
  margin-bottom: 1em;
  max-width: 65ch; /* Optimal line length for reading */
}

/* Large text for better readability */
.text-large {
  font-size: var(--font-size-lg);
  line-height: var(--line-height-relaxed);
}

/* Small text that's still readable */
.text-small {
  font-size: var(--font-size-sm);
  line-height: var(--line-height-normal);
}

/* High contrast text for important information */
.text-high-contrast {
  color: var(--text-primary);
  font-weight: 600;
}

/* User preferences for reduced motion */
@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }

  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* User preferences for increased contrast */
@media (prefers-contrast: high) {
  :root {
    --text-primary: #000000;
    --text-secondary: #333333;
    --primary-color: #0000ee;
    --error-color: #cc0000;
  }

  .btn {
    border-width: 3px;
  }
}

/* Dark mode support */
@media (prefers-color-scheme: dark) {
  :root {
    --text-primary: #ffffff;
    --text-secondary: #cccccc;
    --background-light: var(--background-dark);
    --primary-color: #64b5f6;
    --secondary-color: #81c784;
    --error-color: #f28b82;
    --warning-color: #ffab40;
  }

  /* Adjust focus indicators for dark mode */
  .btn:focus {
    outline-color: #64b5f6;
    box-shadow: 0 0 0 3px rgba(100, 181, 246, 0.3);
  }
}
```

### Focus Management and Keyboard Navigation

```css
/* Comprehensive focus management for keyboard accessibility */

/* Global focus styles - visible and consistent */
*:focus {
  outline: 2px solid var(--focus-color);
  outline-offset: 2px;
}

/* Remove focus outline for mouse users, keep for keyboard users */
*:focus:not(:focus-visible) {
  outline: none;
}

*:focus-visible {
  outline: 2px solid var(--focus-color);
  outline-offset: 2px;
}

/* Skip links - hidden but accessible */
.skip-link {
  position: absolute;
  top: -40px;
  left: 6px;
  background: var(--background-light);
  color: var(--text-primary);
  padding: 8px 16px;
  text-decoration: none;
  border: 2px solid var(--primary-color);
  border-radius: 4px;
  z-index: 1000;
  transition: top 0.3s ease;
}

.skip-link:focus {
  top: 6px; /* Slide down when focused */
  outline: none; /* Skip link has its own styling */
}

/* Focus trap for modals and overlays */
.focus-trap {
  position: relative;
}

.focus-trap::before,
.focus-trap::after {
  content: "";
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Interactive elements - proper sizing for touch */
button,
input,
select,
textarea,
a[href],
[tabindex]:not([tabindex="-1"]) {
  /* Minimum touch target size: 44x44px */
  min-height: 44px;
  min-width: 44px;

  /* Or use padding to achieve target size */
  padding: 0.75rem 1rem;
}

/* Navigation menu - keyboard accessible */
.nav-menu {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
}

.nav-menu li {
  position: relative;
}

.nav-menu a {
  display: block;
  padding: 1rem 1.5rem;
  text-decoration: none;
  color: var(--text-primary);
  border: 2px solid transparent;
  transition: all 0.2s ease;
}

.nav-menu a:hover {
  background-color: rgba(25, 118, 210, 0.1);
  color: var(--primary-color);
}

.nav-menu a:focus {
  outline: none;
  border-color: var(--focus-color);
  background-color: rgba(25, 118, 210, 0.1);
}

/* Dropdown menu - keyboard navigation */
.nav-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  background: var(--background-light);
  border: 1px solid #ccc;
  border-radius: 4px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  opacity: 0;
  visibility: hidden;
  transform: translateY(-8px);
  transition: all 0.2s ease;
  z-index: 1000;
}

.nav-menu li:hover .nav-dropdown,
.nav-menu li:focus-within .nav-dropdown {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
}

/* Form elements - accessible focus styles */
input,
textarea,
select {
  border: 2px solid #ccc;
  border-radius: 4px;
  padding: 0.75rem 1rem;
  font-size: var(--font-size-base);
  line-height: 1.5;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

input:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: var(--focus-color);
  box-shadow: 0 0 0 3px rgba(0, 95, 204, 0.2);
}

/* Invalid form elements */
input:invalid,
textarea:invalid,
select:invalid {
  border-color: var(--error-color);
}

input:invalid:focus,
textarea:invalid:focus,
select:invalid:focus {
  box-shadow: 0 0 0 3px rgba(211, 47, 47, 0.2);
}

/* Custom checkbox and radio buttons - keyboard accessible */
.custom-checkbox,
.custom-radio {
  position: relative;
  display: inline-flex;
  align-items: center;
  cursor: pointer;
  user-select: none;
}

.custom-checkbox input,
.custom-radio input {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.custom-checkbox .checkmark,
.custom-radio .checkmark {
  width: 20px;
  height: 20px;
  border: 2px solid #ccc;
  border-radius: 3px;
  margin-right: 0.5rem;
  position: relative;
  transition: all 0.2s ease;
}

.custom-radio .checkmark {
  border-radius: 50%;
}

/* Focus styles for custom controls */
.custom-checkbox input:focus + .checkmark,
.custom-radio input:focus + .checkmark {
  border-color: var(--focus-color);
  box-shadow: 0 0 0 3px rgba(0, 95, 204, 0.2);
}

/* Checked state */
.custom-checkbox input:checked + .checkmark,
.custom-radio input:checked + .checkmark {
  background-color: var(--primary-color);
  border-color: var(--primary-color);
}

/* Checkmark icon */
.custom-checkbox input:checked + .checkmark::after {
  content: "✓";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 14px;
  font-weight: bold;
}

.custom-radio input:checked + .checkmark::after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: white;
}
```

---

## ⚡ Angular Accessibility Implementation {#angular-accessibility}

### Angular CDK Accessibility Features

Angular provides the CDK (Component Dev Kit) with built-in accessibility features.

#### Setting Up Angular CDK A11y

```bash
# Install Angular CDK
npm install @angular/cdk

# In your module
import { A11yModule } from '@angular/cdk/a11y';

@NgModule({
  imports: [A11yModule],
  // ...
})
export class AppModule { }
```

#### Focus Management with Angular CDK

```typescript
// focus-management.component.ts
import { Component, ElementRef, ViewChild, AfterViewInit } from "@angular/core";
import { FocusMonitor, FocusOrigin } from "@angular/cdk/a11y";
import { LiveAnnouncer } from "@angular/cdk/a11y";

@Component({
  selector: "app-focus-management",
  template: `
    <div class="focus-demo">
      <h2>Focus Management Demo</h2>

      <!-- Monitored button shows focus origin -->
      <button #monitoredButton class="btn-primary" (click)="onButtonClick()">
        Monitored Button
        <span class="focus-origin" *ngIf="focusOrigin">
          ({{ focusOrigin }})
        </span>
      </button>

      <!-- Focus trap example -->
      <div
        class="focus-trap-demo"
        cdkTrapFocus
        [cdkTrapFocusAutoCapture]="trapFocusEnabled"
      >
        <h3>Focus Trap Area</h3>
        <p>Focus is trapped within this area when enabled.</p>
        <button (click)="toggleFocusTrap()">
          {{ trapFocusEnabled ? "Disable" : "Enable" }} Focus Trap
        </button>
        <input placeholder="Input 1" />
        <input placeholder="Input 2" />
        <button>Another Button</button>
      </div>

      <!-- Live announcer example -->
      <div class="live-announcer-demo">
        <h3>Live Announcer</h3>
        <button (click)="announceMessage('polite')">Announce Politely</button>
        <button (click)="announceMessage('assertive')">
          Announce Assertively
        </button>
        <input
          [(ngModel)]="customMessage"
          placeholder="Custom message to announce"
        />
        <button (click)="announceCustomMessage()">Announce Custom</button>
      </div>
    </div>
  `,
  styles: [
    `
      .focus-demo {
        padding: 2rem;
        max-width: 800px;
      }

      .focus-trap-demo {
        border: 2px dashed #ccc;
        padding: 1rem;
        margin: 1rem 0;
        border-radius: 4px;
      }

      .focus-trap-demo.cdk-trap-focus-active {
        border-color: #1976d2;
        background-color: #e3f2fd;
      }

      .live-announcer-demo {
        margin-top: 2rem;
      }

      .focus-origin {
        font-size: 0.8em;
        color: #666;
        font-style: italic;
      }

      button,
      input {
        margin: 0.5rem;
        padding: 0.5rem 1rem;
      }
    `,
  ],
})
export class FocusManagementComponent implements AfterViewInit {
  @ViewChild("monitoredButton") monitoredButton!: ElementRef<HTMLButtonElement>;

  focusOrigin: FocusOrigin = null;
  trapFocusEnabled = false;
  customMessage = "";

  constructor(
    private focusMonitor: FocusMonitor,
    private liveAnnouncer: LiveAnnouncer
  ) {}

  ngAfterViewInit() {
    // Monitor focus on the button to detect how it was focused
    this.focusMonitor.monitor(this.monitoredButton).subscribe((origin) => {
      this.focusOrigin = origin;

      // Origin can be: 'mouse', 'keyboard', 'touch', 'program', or null
      console.log("Button focused via:", origin);
    });
  }

  ngOnDestroy() {
    // Clean up focus monitoring
    this.focusMonitor.stopMonitoring(this.monitoredButton);
  }

  onButtonClick() {
    console.log("Button clicked!");
  }

  toggleFocusTrap() {
    this.trapFocusEnabled = !this.trapFocusEnabled;

    // Announce the state change
    this.liveAnnouncer.announce(
      `Focus trap ${this.trapFocusEnabled ? "enabled" : "disabled"}`,
      "polite"
    );
  }

  announceMessage(politeness: "polite" | "assertive") {
    const message =
      politeness === "polite"
        ? "This is a polite announcement"
        : "This is an assertive announcement!";

    this.liveAnnouncer.announce(message, politeness);
  }

  announceCustomMessage() {
    if (this.customMessage.trim()) {
      this.liveAnnouncer.announce(this.customMessage, "polite");
      this.customMessage = ""; // Clear after announcing
    }
  }
}
```

This is Part 1 of the Angular accessibility section. Due to length constraints, I'll continue with more Angular examples in the next part. Would you like me to continue with the localization fundamentals section or add more Angular accessibility patterns?

---

## 🌍 Localization (i18n) Fundamentals {#localization-fundamentals}

### Understanding Internationalization vs Localization

```typescript
// Internationalization (i18n) - Building support for multiple languages
interface I18nConfig {
  defaultLocale: string;
  supportedLocales: string[];
  fallbackLocale: string;
  rtlLocales: string[];
  dateFormats: Record<string, string>;
  numberFormats: Record<string, Intl.NumberFormatOptions>;
}

// Example configuration for a global application
const i18nConfig: I18nConfig = {
  defaultLocale: "en-US",
  supportedLocales: [
    "en-US",
    "es-ES",
    "fr-FR",
    "de-DE",
    "ar-SA",
    "zh-CN",
    "ja-JP",
  ],
  fallbackLocale: "en-US",
  rtlLocales: ["ar-SA", "he-IL", "fa-IR"],
  dateFormats: {
    "en-US": "MM/dd/yyyy",
    "es-ES": "dd/MM/yyyy",
    "de-DE": "dd.MM.yyyy",
    "ar-SA": "dd/MM/yyyy",
    "zh-CN": "yyyy/MM/dd",
  },
  numberFormats: {
    "en-US": { style: "decimal", minimumFractionDigits: 2 },
    "de-DE": { style: "decimal", minimumFractionDigits: 2 },
    "ar-SA": {
      style: "decimal",
      minimumFractionDigits: 2,
      numberingSystem: "arab",
    },
  },
};
```

### HTML5 Language and Direction Support

```html
<!-- Document language and direction -->
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <!-- Always specify language and direction -->
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Multilingual Application</title>

    <!-- Language-specific stylesheets -->
    <link rel="stylesheet" href="styles/main.css" />
    <link rel="stylesheet" href="styles/ltr.css" data-dir="ltr" />
    <link rel="stylesheet" href="styles/rtl.css" data-dir="rtl" disabled />
  </head>
  <body>
    <!-- Language switcher with proper labels -->
    <nav
      class="language-switcher"
      role="navigation"
      aria-label="Language selection"
    >
      <label for="language-select" class="sr-only">Select language</label>
      <select id="language-select" onchange="changeLanguage(this.value)">
        <option value="en-US" selected>English (US)</option>
        <option value="es-ES">Español (España)</option>
        <option value="fr-FR">Français (France)</option>
        <option value="de-DE">Deutsch (Deutschland)</option>
        <option value="ar-SA">العربية (السعودية)</option>
        <option value="zh-CN">中文 (简体)</option>
        <option value="ja-JP">日本語</option>
      </select>
    </nav>

    <main>
      <!-- Content with language-specific attributes -->
      <article lang="en" dir="ltr">
        <h1>Welcome to Our Platform</h1>
        <p>This content is in English and reads left-to-right.</p>

        <!-- Date and time with locale-specific formatting -->
        <time datetime="2025-11-05T14:30:00Z" data-locale="en-US">
          November 5, 2025 at 2:30 PM
        </time>

        <!-- Numbers and currency with proper formatting -->
        <p
          class="price"
          data-amount="1234.56"
          data-currency="USD"
          data-locale="en-US"
        >
          $1,234.56
        </p>
      </article>

      <!-- Mixed content with different languages -->
      <section class="testimonials">
        <h2>Customer Testimonials</h2>

        <blockquote lang="es" dir="ltr">
          <p>
            "Esta aplicación es increíble. Me ha ayudado mucho en mi trabajo
            diario."
          </p>
          <cite>María González, España</cite>
        </blockquote>

        <blockquote lang="ar" dir="rtl">
          <p>"هذا التطبيق رائع جداً ويساعدني كثيراً في عملي اليومي."</p>
          <cite>أحمد محمد، السعودية</cite>
        </blockquote>

        <blockquote lang="zh" dir="ltr">
          <p>"这个应用程序真的很棒，对我的日常工作很有帮助。"</p>
          <cite>李伟，中国</cite>
        </blockquote>
      </section>

      <!-- Form with localized validation messages -->
      <form class="contact-form" data-locale="en-US">
        <h2>Contact Us</h2>

        <div class="form-group">
          <label for="full-name" data-i18n="label.fullName">Full Name</label>
          <input
            type="text"
            id="full-name"
            name="fullName"
            required
            data-validation-required="This field is required"
            data-validation-required-es="Este campo es obligatorio"
            data-validation-required-ar="هذا الحقل مطلوب"
            data-validation-required-zh="此字段为必填项"
            placeholder="Enter your full name"
            data-placeholder-es="Ingrese su nombre completo"
            data-placeholder-ar="أدخل اسمك الكامل"
            data-placeholder-zh="请输入您的全名"
          />
          <div class="error-message" role="alert" aria-live="polite"></div>
        </div>

        <div class="form-group">
          <label for="email" data-i18n="label.email">Email Address</label>
          <input
            type="email"
            id="email"
            name="email"
            required
            data-validation-email="Please enter a valid email address"
            data-validation-email-es="Por favor ingrese una dirección de correo válida"
            data-validation-email-ar="يرجى إدخال عنوان بريد إلكتروني صحيح"
            data-validation-email-zh="请输入有效的电子邮件地址"
            placeholder="your.email@example.com"
          />
          <div class="error-message" role="alert" aria-live="polite"></div>
        </div>

        <div class="form-group">
          <label for="message" data-i18n="label.message">Message</label>
          <textarea
            id="message"
            name="message"
            rows="5"
            required
            placeholder="Enter your message here"
            data-placeholder-es="Ingrese su mensaje aquí"
            data-placeholder-ar="أدخل رسالتك هنا"
            data-placeholder-zh="在此输入您的信息"
          ></textarea>
          <div class="error-message" role="alert" aria-live="polite"></div>
        </div>

        <button type="submit" data-i18n="button.submit">Send Message</button>
      </form>
    </main>

    <!-- Footer with locale information -->
    <footer>
      <p>
        <span data-i18n="footer.currentLocale">Current locale:</span>
        <span id="current-locale" lang="en">English (United States)</span>
      </p>
      <p>
        <span data-i18n="footer.lastUpdated">Last updated:</span>
        <time id="last-updated" datetime="2025-11-05T14:30:00Z"></time>
      </p>
    </footer>
  </body>
</html>
```

### CSS for RTL/LTR Support

```css
/* CSS logical properties for bidirectional support */
:root {
  /* Logical properties work automatically with direction changes */
  --spacing-inline-start: 1rem;
  --spacing-inline-end: 1rem;
  --spacing-block-start: 1rem;
  --spacing-block-end: 1rem;

  /* Direction-specific values */
  --border-start-width: 3px;
  --border-end-width: 0;
}

/* Base styles using logical properties */
.content {
  /* Use logical properties instead of left/right */
  margin-inline-start: var(--spacing-inline-start);
  margin-inline-end: var(--spacing-inline-end);
  margin-block-start: var(--spacing-block-start);
  margin-block-end: var(--spacing-block-end);

  /* Logical padding */
  padding-inline: 2rem;
  padding-block: 1rem;

  /* Logical borders */
  border-inline-start: var(--border-start-width) solid #1976d2;
  border-inline-end: var(--border-end-width) solid #1976d2;

  /* Text alignment that respects direction */
  text-align: start; /* 'start' works for both LTR and RTL */
}

/* Navigation with logical properties */
.nav-menu {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
  gap: 1rem;
}

.nav-menu li {
  /* Logical margins for proper spacing */
  margin-inline-end: 1rem;
}

.nav-menu a {
  display: block;
  padding-inline: 1rem;
  padding-block: 0.5rem;
  text-decoration: none;

  /* Border radius that works with direction */
  border-start-start-radius: 4px;
  border-start-end-radius: 4px;
}

/* Form layouts with logical properties */
.form-group {
  margin-block-end: 1.5rem;
}

.form-group label {
  display: block;
  margin-block-end: 0.5rem;
  font-weight: 600;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding-inline: 1rem;
  padding-block: 0.75rem;
  border: 2px solid #ccc;
  border-radius: 4px;

  /* Text direction inheritance */
  direction: inherit;
  text-align: inherit;
}

/* Specific styles for RTL languages */
[dir="rtl"] {
  /* RTL-specific adjustments */
  font-family: "Segoe UI", Tahoma, Arial, sans-serif; /* Fonts that support Arabic */
}

[dir="rtl"] .nav-menu {
  /* Reverse flex direction for RTL */
  flex-direction: row-reverse;
}

[dir="rtl"] .form-group input[type="email"],
[dir="rtl"] .form-group input[type="url"] {
  /* Email and URLs should remain LTR even in RTL context */
  direction: ltr;
  text-align: left;
}

/* Language-specific typography */
[lang="ar"] {
  font-family: "Noto Sans Arabic", "Arabic UI Display", "Geeza Pro", sans-serif;
  line-height: 1.8; /* Arabic text needs more line height */
  font-size: 1.1em; /* Slightly larger for better readability */
}

[lang="zh"] {
  font-family: "Noto Sans SC", "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", sans-serif;
  line-height: 1.7;
}

[lang="ja"] {
  font-family: "Noto Sans JP", "Hiragino Kaku Gothic ProN", "Meiryo", sans-serif;
  line-height: 1.7;
}

[lang="ko"] {
  font-family: "Noto Sans KR", "Malgun Gothic", "Apple SD Gothic Neo",
    sans-serif;
  line-height: 1.6;
}

/* Responsive typography for different scripts */
@media (max-width: 768px) {
  [lang="ar"] {
    font-size: 1.15em; /* Larger on mobile for Arabic */
  }

  [lang="zh"],
  [lang="ja"] {
    font-size: 1.05em; /* Slightly larger for CJK characters */
  }
}

/* Components that need direction-specific styling */
.breadcrumb {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
}

.breadcrumb li {
  display: flex;
  align-items: center;
}

.breadcrumb li:not(:last-child)::after {
  content: "/";
  margin-inline: 0.5rem;
  color: #666;
}

/* RTL breadcrumb separator */
[dir="rtl"] .breadcrumb li:not(:last-child)::after {
  content: "\\";
}

/* Card layouts with logical properties */
.card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 1.5rem;
  margin-block-end: 1rem;

  /* Shadow that works with direction */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.card-header {
  border-block-end: 1px solid #e0e0e0;
  margin-block-end: 1rem;
  padding-block-end: 1rem;
}

.card-actions {
  display: flex;
  gap: 1rem;
  margin-block-start: 1rem;
  justify-content: flex-end;
}

[dir="rtl"] .card-actions {
  justify-content: flex-start;
}

/* Tables with RTL support */
.data-table {
  width: 100%;
  border-collapse: collapse;
  direction: ltr; /* Keep table structure LTR */
}

.data-table th,
.data-table td {
  padding-inline: 1rem;
  padding-block: 0.75rem;
  border-block-end: 1px solid #e0e0e0;
  text-align: start;
}

.data-table th {
  background-color: #f5f5f5;
  font-weight: 600;
}

/* RTL-specific table adjustments */
[dir="rtl"] .data-table {
  direction: rtl;
}

[dir="rtl"] .data-table th:first-child,
[dir="rtl"] .data-table td:first-child {
  text-align: end;
}

/* Animation that respects direction */
@keyframes slideInStart {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideInEnd {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.slide-in-start {
  animation: slideInStart 0.3s ease-out;
}

.slide-in-end {
  animation: slideInEnd 0.3s ease-out;
}

/* RTL animations */
[dir="rtl"] .slide-in-start {
  animation: slideInEnd 0.3s ease-out;
}

[dir="rtl"] .slide-in-end {
  animation: slideInStart 0.3s ease-out;
}

/* Language switcher styling */
.language-switcher {
  position: relative;
  margin-block-end: 1rem;
}

.language-switcher select {
  padding-inline: 1rem;
  padding-block: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: white;
  font-size: 0.9rem;

  /* Arrow positioning for different directions */
  background-image: url("data:image/svg+xml;utf8,<svg fill='black' height='24' viewBox='0 0 24 24' width='24' xmlns='http://www.w3.org/2000/svg'><path d='M7 10l5 5 5-5z'/></svg>");
  background-repeat: no-repeat;
  background-position: right 0.7rem top 50%;
  background-size: 0.65rem auto;
  padding-inline-end: 2rem;
}

[dir="rtl"] .language-switcher select {
  background-position: left 0.7rem top 50%;
  padding-inline-end: 1rem;
  padding-inline-start: 2rem;
}
```

---

## 🌐 Angular Internationalization {#angular-i18n}

### Setting Up Angular i18n

```bash
# Add Angular i18n package
ng add @angular/localize

# Generate translation source file
ng extract-i18n

# Build for specific locales
ng build --localize

# Serve with specific locale
ng serve --configuration=es
```

#### Angular i18n Configuration

```typescript
// angular.json - Configuration for multiple locales
{
  "projects": {
    "your-app": {
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
          "ar": {
            "translation": "src/locale/messages.ar.xlf",
            "baseHref": "/ar/"
          },
          "zh": {
            "translation": "src/locale/messages.zh.xlf",
            "baseHref": "/zh/"
          }
        }
      },
      "architect": {
        "build": {
          "configurations": {
            "es": {
              "aot": true,
              "outputPath": "dist/es/",
              "i18nFile": "src/locale/messages.es.xlf",
              "i18nFormat": "xlf",
              "i18nLocale": "es"
            },
            "fr": {
              "aot": true,
              "outputPath": "dist/fr/",
              "i18nFile": "src/locale/messages.fr.xlf",
              "i18nFormat": "xlf",
              "i18nLocale": "fr"
            },
            "ar": {
              "aot": true,
              "outputPath": "dist/ar/",
              "i18nFile": "src/locale/messages.ar.xlf",
              "i18nFormat": "xlf",
              "i18nLocale": "ar"
            }
          }
        },
        "serve": {
          "configurations": {
            "es": {
              "browserTarget": "your-app:build:es"
            },
            "fr": {
              "browserTarget": "your-app:build:fr"
            },
            "ar": {
              "browserTarget": "your-app:build:ar"
            }
          }
        }
      }
    }
  }
}
```

#### Internationalization in Components

```typescript
// internationalized.component.ts
import { Component, OnInit, LOCALE_ID, Inject } from "@angular/core";
import { formatDate, formatCurrency, formatNumber } from "@angular/common";
import { TranslateService } from "@ngx-translate/core";

@Component({
  selector: "app-internationalized",
  template: `
    <div
      class="i18n-demo"
      [attr.lang]="currentLocale"
      [attr.dir]="textDirection"
    >
      <header class="demo-header">
        <h1 i18n="@@page.title">International Application</h1>
        <p i18n="@@page.description">
          This application supports multiple languages and regions
        </p>

        <!-- Language switcher -->
        <div class="language-switcher">
          <label for="locale-select" i18n="@@label.selectLanguage">
            Select Language:
          </label>
          <select
            id="locale-select"
            [(ngModel)]="selectedLocale"
            (change)="changeLocale($event)"
          >
            <option value="en-US">English (United States)</option>
            <option value="es-ES">Español (España)</option>
            <option value="fr-FR">Français (France)</option>
            <option value="de-DE">Deutsch (Deutschland)</option>
            <option value="ar-SA">العربية (السعودية)</option>
            <option value="zh-CN">中文 (简体)</option>
            <option value="ja-JP">日本語</option>
          </select>
        </div>
      </header>

      <main class="demo-content">
        <!-- Simple text internationalization -->
        <section class="text-demo">
          <h2 i18n="@@section.basicText">Basic Text Translation</h2>
          <p i18n="@@welcome.message">
            Welcome to our international platform! We're glad you're here.
          </p>

          <!-- Text with HTML markup -->
          <p i18n="@@rich.text">
            You can also include <strong>formatting</strong> and
            <a href="/help">links</a> in your translated text.
          </p>

          <!-- Conditional text based on user type -->
          <ng-container *ngIf="isAdmin; else regularUser">
            <p i18n="@@admin.welcome">Welcome back, Administrator!</p>
          </ng-container>
          <ng-template #regularUser>
            <p i18n="@@user.welcome">Welcome back!</p>
          </ng-template>
        </section>

        <!-- Pluralization examples -->
        <section class="plural-demo">
          <h2 i18n="@@section.pluralization">Pluralization</h2>

          <div class="counter-demo">
            <button (click)="decrementCount()" [disabled]="itemCount <= 0">
              -
            </button>
            <span class="count">{{ itemCount }}</span>
            <button (click)="incrementCount()">+</button>
          </div>

          <!-- ICU expressions for pluralization -->
          <p i18n="@@items.count">
            {itemCount, plural, =0 {No items} =1 {One item} other
            {{{itemCount}} items} } in your cart
          </p>

          <!-- Complex pluralization with gender -->
          <p i18n="@@notification.message">
            {unreadCount, plural, =0 {No new notifications} =1 {You have one new
            notification} other {You have {{ unreadCount }} new notifications} }
          </p>
        </section>

        <!-- Date and time formatting -->
        <section class="datetime-demo">
          <h2 i18n="@@section.datetime">Date and Time Formatting</h2>

          <div class="datetime-examples">
            <h3 i18n="@@current.date">Current Date and Time:</h3>

            <!-- Short date format -->
            <p>
              <span i18n="@@label.shortDate">Short Date:</span>
              {{ currentDate | date : "short" : currentLocale }}
            </p>

            <!-- Medium date format -->
            <p>
              <span i18n="@@label.mediumDate">Medium Date:</span>
              {{ currentDate | date : "medium" : currentLocale }}
            </p>

            <!-- Full date format -->
            <p>
              <span i18n="@@label.fullDate">Full Date:</span>
              {{ currentDate | date : "full" : currentLocale }}
            </p>

            <!-- Custom date format -->
            <p>
              <span i18n="@@label.customDate">Custom Format:</span>
              {{ currentDate | date : "EEEE, MMMM d, y" : currentLocale }}
            </p>

            <!-- Relative time -->
            <p>
              <span i18n="@@label.lastUpdated">Last Updated:</span>
              {{ getRelativeTime(lastUpdated) }}
            </p>
          </div>
        </section>

        <!-- Number and currency formatting -->
        <section class="number-demo">
          <h2 i18n="@@section.numbers">Number and Currency Formatting</h2>

          <div class="number-examples">
            <!-- Decimal numbers -->
            <h3 i18n="@@label.decimalNumbers">Decimal Numbers:</h3>
            <p>
              <span i18n="@@label.simpleNumber">Simple Number:</span>
              {{ sampleNumber | number : "1.2-2" : currentLocale }}
            </p>

            <p>
              <span i18n="@@label.largeNumber">Large Number:</span>
              {{ largeNumber | number : "1.0-0" : currentLocale }}
            </p>

            <!-- Percentage -->
            <h3 i18n="@@label.percentages">Percentages:</h3>
            <p>
              <span i18n="@@label.completion">Completion Rate:</span>
              {{ completionRate | percent : "1.1-1" : currentLocale }}
            </p>

            <!-- Currency -->
            <h3 i18n="@@label.currency">Currency:</h3>
            <p>
              <span i18n="@@label.price">Price:</span>
              {{
                productPrice
                  | currency : currencyCode : "symbol" : "1.2-2" : currentLocale
              }}
            </p>

            <p>
              <span i18n="@@label.total">Total:</span>
              {{
                totalAmount
                  | currency
                    : currencyCode
                    : "symbol-narrow"
                    : "1.2-2"
                    : currentLocale
              }}
            </p>

            <!-- Multiple currencies -->
            <div class="currency-examples">
              <h4 i18n="@@label.multiCurrency">Multiple Currencies:</h4>
              <ul>
                <li>
                  USD:
                  {{
                    baseAmount
                      | currency : "USD" : "symbol" : "1.2-2" : currentLocale
                  }}
                </li>
                <li>
                  EUR:
                  {{
                    convertCurrency(baseAmount, "EUR")
                      | currency : "EUR" : "symbol" : "1.2-2" : currentLocale
                  }}
                </li>
                <li>
                  JPY:
                  {{
                    convertCurrency(baseAmount, "JPY")
                      | currency : "JPY" : "symbol" : "1.0-0" : currentLocale
                  }}
                </li>
                <li>
                  SAR:
                  {{
                    convertCurrency(baseAmount, "SAR")
                      | currency : "SAR" : "symbol" : "1.2-2" : currentLocale
                  }}
                </li>
              </ul>
            </div>
          </div>
        </section>

        <!-- Form validation with i18n -->
        <section class="form-demo">
          <h2 i18n="@@section.forms">Internationalized Forms</h2>

          <form [formGroup]="internationalForm" (ngSubmit)="onSubmit()">
            <!-- Name field with localized validation -->
            <div class="form-group">
              <label for="name" i18n="@@label.name">Name *</label>
              <input
                id="name"
                type="text"
                formControlName="name"
                [placeholder]="getLocalizedPlaceholder('name')"
                [attr.aria-describedby]="getAriaDescribedBy('name')"
                [class.error]="isFieldInvalid('name')"
              />

              <div
                id="name-error"
                class="error-message"
                role="alert"
                *ngIf="isFieldInvalid('name')"
              >
                {{ getFieldErrorMessage("name") }}
              </div>
            </div>

            <!-- Email with localized format validation -->
            <div class="form-group">
              <label for="email" i18n="@@label.email">Email Address *</label>
              <input
                id="email"
                type="email"
                formControlName="email"
                [placeholder]="getLocalizedPlaceholder('email')"
                [class.error]="isFieldInvalid('email')"
              />

              <div
                id="email-error"
                class="error-message"
                role="alert"
                *ngIf="isFieldInvalid('email')"
              >
                {{ getFieldErrorMessage("email") }}
              </div>
            </div>

            <!-- Phone number with regional formatting -->
            <div class="form-group">
              <label for="phone" i18n="@@label.phone">Phone Number</label>
              <input
                id="phone"
                type="tel"
                formControlName="phone"
                [placeholder]="getLocalizedPhonePlaceholder()"
                [pattern]="getPhonePattern()"
                [class.error]="isFieldInvalid('phone')"
              />

              <div class="help-text">
                {{ getPhoneFormatHelp() }}
              </div>

              <div
                id="phone-error"
                class="error-message"
                role="alert"
                *ngIf="isFieldInvalid('phone')"
              >
                {{ getFieldErrorMessage("phone") }}
              </div>
            </div>

            <!-- Country/Region selector -->
            <div class="form-group">
              <label for="country" i18n="@@label.country">Country/Region</label>
              <select id="country" formControlName="country">
                <option value="" i18n="@@option.selectCountry">
                  Select Country
                </option>
                <option
                  *ngFor="let country of countries"
                  [value]="country.code"
                >
                  {{ getLocalizedCountryName(country.code) }}
                </option>
              </select>
            </div>

            <!-- Submit button -->
            <button
              type="submit"
              class="btn-primary"
              [disabled]="internationalForm.invalid"
              i18n="@@button.submit"
            >
              Submit Form
            </button>
          </form>
        </section>

        <!-- RTL/LTR text direction demo -->
        <section class="direction-demo" *ngIf="isRTL">
          <h2 i18n="@@section.textDirection">Text Direction</h2>
          <p i18n="@@rtl.explanation">
            This section demonstrates right-to-left text layout for Arabic and
            Hebrew languages.
          </p>

          <!-- Mixed direction content -->
          <div class="mixed-content">
            <p dir="auto">
              <span i18n="@@mixed.text.1"
                >This text adapts to the content direction:</span
              >
              English text مختلط مع نص عربي and back to English.
            </p>

            <!-- Email addresses and URLs remain LTR -->
            <p>
              <span i18n="@@contact.email">Contact us at:</span>
              <span dir="ltr">support@example.com</span>
            </p>
          </div>
        </section>
      </main>

      <!-- Status and debug information -->
      <footer class="demo-footer">
        <div class="locale-info">
          <h3 i18n="@@section.localeInfo">Locale Information</h3>
          <ul>
            <li>
              <span i18n="@@info.currentLocale">Current Locale:</span>
              {{ currentLocale }}
            </li>
            <li>
              <span i18n="@@info.textDirection">Text Direction:</span>
              {{ textDirection }}
            </li>
            <li>
              <span i18n="@@info.dateFormat">Date Format:</span>
              {{ getDateFormatExample() }}
            </li>
            <li>
              <span i18n="@@info.numberFormat">Number Format:</span>
              {{ getNumberFormatExample() }}
            </li>
          </ul>
        </div>
      </footer>
    </div>
  `,
  styles: [
    `
      .i18n-demo {
        max-width: 1200px;
        margin: 0 auto;
        padding: 2rem;
        font-family: inherit;
      }

      .demo-header {
        border-bottom: 2px solid #e0e0e0;
        margin-bottom: 2rem;
        padding-bottom: 1rem;
      }

      .language-switcher {
        margin-top: 1rem;
      }

      .language-switcher label {
        display: block;
        margin-bottom: 0.5rem;
        font-weight: 600;
      }

      .language-switcher select {
        padding: 0.5rem 1rem;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 1rem;
        min-width: 250px;
      }

      .demo-content section {
        margin-bottom: 3rem;
        padding: 1.5rem;
        border: 1px solid #e0e0e0;
        border-radius: 8px;
        background-color: #fafafa;
      }

      .demo-content h2 {
        margin-top: 0;
        color: #1976d2;
        border-bottom: 1px solid #e0e0e0;
        padding-bottom: 0.5rem;
      }

      .counter-demo {
        display: flex;
        align-items: center;
        gap: 1rem;
        margin: 1rem 0;
      }

      .counter-demo button {
        width: 40px;
        height: 40px;
        border: 1px solid #ccc;
        border-radius: 4px;
        background: white;
        font-size: 1.2rem;
        cursor: pointer;
      }

      .counter-demo .count {
        font-size: 1.5rem;
        font-weight: bold;
        min-width: 40px;
        text-align: center;
      }

      .datetime-examples,
      .number-examples {
        display: grid;
        gap: 1rem;
      }

      .datetime-examples p,
      .number-examples p {
        margin: 0.5rem 0;
        padding: 0.5rem;
        background: white;
        border-radius: 4px;
        border-left: 3px solid #1976d2;
      }

      .currency-examples ul {
        list-style: none;
        padding: 0;
      }

      .currency-examples li {
        padding: 0.5rem;
        margin: 0.25rem 0;
        background: white;
        border-radius: 4px;
      }

      .form-demo form {
        background: white;
        padding: 1.5rem;
        border-radius: 8px;
      }

      .form-group {
        margin-bottom: 1.5rem;
      }

      .form-group label {
        display: block;
        margin-bottom: 0.5rem;
        font-weight: 600;
      }

      .form-group input,
      .form-group select {
        width: 100%;
        padding: 0.75rem 1rem;
        border: 2px solid #ccc;
        border-radius: 4px;
        font-size: 1rem;
      }

      .form-group input.error,
      .form-group select.error {
        border-color: #f44336;
        background-color: #ffebee;
      }

      .error-message {
        color: #f44336;
        font-size: 0.875rem;
        margin-top: 0.25rem;
      }

      .help-text {
        font-size: 0.875rem;
        color: #666;
        margin-top: 0.25rem;
      }

      .btn-primary {
        background-color: #1976d2;
        color: white;
        border: none;
        padding: 0.75rem 2rem;
        border-radius: 4px;
        font-size: 1rem;
        cursor: pointer;
        transition: background-color 0.2s;
      }

      .btn-primary:hover:not(:disabled) {
        background-color: #1565c0;
      }

      .btn-primary:disabled {
        background-color: #cccccc;
        cursor: not-allowed;
      }

      .mixed-content {
        background: white;
        padding: 1rem;
        border-radius: 4px;
        border: 1px solid #e0e0e0;
      }

      .demo-footer {
        margin-top: 3rem;
        padding-top: 2rem;
        border-top: 2px solid #e0e0e0;
      }

      .locale-info ul {
        list-style: none;
        padding: 0;
      }

      .locale-info li {
        padding: 0.5rem 0;
        border-bottom: 1px solid #e0e0e0;
      }

      .locale-info li:last-child {
        border-bottom: none;
      }

      /* RTL-specific styles */
      [dir="rtl"] .counter-demo {
        flex-direction: row-reverse;
      }

      [dir="rtl"] .form-group input,
      [dir="rtl"] .form-group select {
        text-align: right;
      }

      [dir="rtl"] .form-group input[type="email"],
      [dir="rtl"] .form-group input[type="url"] {
        text-align: left;
        direction: ltr;
      }
    `,
  ],
})
export class InternationalizedComponent implements OnInit {
  currentLocale: string;
  selectedLocale: string;
  textDirection: "ltr" | "rtl" = "ltr";
  isRTL: boolean = false;
  isAdmin: boolean = false;

  // Demo data
  itemCount: number = 3;
  unreadCount: number = 5;
  currentDate: Date = new Date();
  lastUpdated: Date = new Date(Date.now() - 2 * 60 * 60 * 1000); // 2 hours ago

  // Number demo data
  sampleNumber: number = 1234.56;
  largeNumber: number = 1234567;
  completionRate: number = 0.847;
  productPrice: number = 299.99;
  totalAmount: number = 1247.89;
  baseAmount: number = 100;

  currencyCode: string = "USD";

  // Form
  internationalForm!: FormGroup;

  // Countries list for demo
  countries = [
    { code: "US", name: "United States" },
    { code: "ES", name: "Spain" },
    { code: "FR", name: "France" },
    { code: "DE", name: "Germany" },
    { code: "SA", name: "Saudi Arabia" },
    { code: "CN", name: "China" },
    { code: "JP", name: "Japan" },
  ];

  constructor(
    @Inject(LOCALE_ID) private localeId: string,
    private translateService: TranslateService,
    private fb: FormBuilder
  ) {
    this.currentLocale = localeId;
    this.selectedLocale = localeId;
    this.updateLocaleSettings();
  }

  ngOnInit() {
    this.initializeForm();
    this.updateCurrencyByLocale();
  }

  initializeForm() {
    this.internationalForm = this.fb.group({
      name: ["", [Validators.required, Validators.minLength(2)]],
      email: ["", [Validators.required, Validators.email]],
      phone: ["", [this.phoneValidator.bind(this)]],
      country: ["", Validators.required],
    });
  }

  updateLocaleSettings() {
    // RTL languages
    const rtlLocales = ["ar", "he", "fa", "ur"];
    this.isRTL = rtlLocales.some((locale) =>
      this.currentLocale.startsWith(locale)
    );
    this.textDirection = this.isRTL ? "rtl" : "ltr";

    // Update document direction
    document.documentElement.setAttribute("dir", this.textDirection);
    document.documentElement.setAttribute("lang", this.currentLocale);
  }

  updateCurrencyByLocale() {
    const currencyMap: { [key: string]: string } = {
      "en-US": "USD",
      "es-ES": "EUR",
      "fr-FR": "EUR",
      "de-DE": "EUR",
      "ar-SA": "SAR",
      "zh-CN": "CNY",
      "ja-JP": "JPY",
    };

    this.currencyCode = currencyMap[this.currentLocale] || "USD";
  }

  changeLocale(event: Event) {
    const select = event.target as HTMLSelectElement;
    this.selectedLocale = select.value;

    // In a real application, you would navigate to the new locale
    // or reload the page with the new locale
    console.log("Changing locale to:", this.selectedLocale);

    // For demo purposes, update current locale
    this.currentLocale = this.selectedLocale;
    this.updateLocaleSettings();
    this.updateCurrencyByLocale();
  }

  incrementCount() {
    this.itemCount++;
  }

  decrementCount() {
    if (this.itemCount > 0) {
      this.itemCount--;
    }
  }

  getRelativeTime(date: Date): string {
    const now = new Date();
    const diffInSeconds = Math.floor((now.getTime() - date.getTime()) / 1000);

    if (diffInSeconds < 60) {
      return this.translateService.instant("time.justNow");
    } else if (diffInSeconds < 3600) {
      const minutes = Math.floor(diffInSeconds / 60);
      return this.translateService.instant("time.minutesAgo", { minutes });
    } else if (diffInSeconds < 86400) {
      const hours = Math.floor(diffInSeconds / 3600);
      return this.translateService.instant("time.hoursAgo", { hours });
    } else {
      return formatDate(date, "short", this.currentLocale);
    }
  }

  convertCurrency(amount: number, targetCurrency: string): number {
    // Mock currency conversion - in real app, use actual exchange rates
    const exchangeRates: { [key: string]: number } = {
      USD: 1,
      EUR: 0.85,
      JPY: 110,
      SAR: 3.75,
      CNY: 6.45,
    };

    return amount * (exchangeRates[targetCurrency] || 1);
  }

  // Form helper methods
  isFieldInvalid(fieldName: string): boolean {
    const field = this.internationalForm.get(fieldName);
    return !!(field && field.invalid && field.touched);
  }

  getAriaDescribedBy(fieldName: string): string {
    const parts = [`${fieldName}-help`];
    if (this.isFieldInvalid(fieldName)) {
      parts.push(`${fieldName}-error`);
    }
    return parts.join(" ");
  }

  getLocalizedPlaceholder(fieldName: string): string {
    const placeholders: { [key: string]: { [locale: string]: string } } = {
      name: {
        "en-US": "Enter your full name",
        "es-ES": "Ingrese su nombre completo",
        "fr-FR": "Entrez votre nom complet",
        "de-DE": "Geben Sie Ihren vollständigen Namen ein",
        "ar-SA": "أدخل اسمك الكامل",
        "zh-CN": "请输入您的全名",
        "ja-JP": "フルネームを入力してください",
      },
      email: {
        "en-US": "your.email@example.com",
        "es-ES": "su.correo@ejemplo.com",
        "fr-FR": "votre.email@exemple.com",
        "de-DE": "ihre.email@beispiel.com",
        "ar-SA": "your.email@example.com", // Keep LTR for email
        "zh-CN": "your.email@example.com",
        "ja-JP": "your.email@example.com",
      },
    };

    return (
      placeholders[fieldName]?.[this.currentLocale] ||
      placeholders[fieldName]?.["en-US"] ||
      ""
    );
  }

  getLocalizedPhonePlaceholder(): string {
    const phonePlaceholders: { [locale: string]: string } = {
      "en-US": "+1 (555) 123-4567",
      "es-ES": "+34 912 345 678",
      "fr-FR": "+33 1 23 45 67 89",
      "de-DE": "+49 30 12345678",
      "ar-SA": "+966 11 123 4567",
      "zh-CN": "+86 138 0013 8000",
      "ja-JP": "+81 90-1234-5678",
    };

    return phonePlaceholders[this.currentLocale] || phonePlaceholders["en-US"];
  }

  getPhonePattern(): string {
    const phonePatterns: { [locale: string]: string } = {
      "en-US": "^\\+1\\s\\(\\d{3}\\)\\s\\d{3}-\\d{4}$",
      "es-ES": "^\\+34\\s\\d{3}\\s\\d{3}\\s\\d{3}$",
      "fr-FR": "^\\+33\\s\\d{1}\\s\\d{2}\\s\\d{2}\\s\\d{2}\\s\\d{2}$",
      "de-DE": "^\\+49\\s\\d{2}\\s\\d{8}$",
      "ar-SA": "^\\+966\\s\\d{2}\\s\\d{3}\\s\\d{4}$",
      "zh-CN": "^\\+86\\s\\d{3}\\s\\d{4}\\s\\d{4}$",
      "ja-JP": "^\\+81\\s\\d{2,3}-\\d{4}-\\d{4}$",
    };

    return phonePatterns[this.currentLocale] || phonePatterns["en-US"];
  }

  getPhoneFormatHelp(): string {
    const helpTexts: { [locale: string]: string } = {
      "en-US": "Format: +1 (555) 123-4567",
      "es-ES": "Formato: +34 912 345 678",
      "fr-FR": "Format: +33 1 23 45 67 89",
      "de-DE": "Format: +49 30 12345678",
      "ar-SA": "التنسيق: +966 11 123 4567",
      "zh-CN": "格式：+86 138 0013 8000",
      "ja-JP": "形式：+81 90-1234-5678",
    };

    return helpTexts[this.currentLocale] || helpTexts["en-US"];
  }

  phoneValidator(control: AbstractControl): ValidationErrors | null {
    if (!control.value) return null;

    const pattern = this.getPhonePattern();
    const regex = new RegExp(pattern);

    return regex.test(control.value) ? null : { phone: true };
  }

  getLocalizedCountryName(countryCode: string): string {
    // In a real app, use a proper i18n library for country names
    const countryNames: { [locale: string]: { [code: string]: string } } = {
      "en-US": {
        US: "United States",
        ES: "Spain",
        FR: "France",
        DE: "Germany",
        SA: "Saudi Arabia",
        CN: "China",
        JP: "Japan",
      },
      "es-ES": {
        US: "Estados Unidos",
        ES: "España",
        FR: "Francia",
        DE: "Alemania",
        SA: "Arabia Saudí",
        CN: "China",
        JP: "Japón",
      },
      "ar-SA": {
        US: "الولايات المتحدة",
        ES: "إسبانيا",
        FR: "فرنسا",
        DE: "ألمانيا",
        SA: "المملكة العربية السعودية",
        CN: "الصين",
        JP: "اليابان",
      },
      // Add more locales as needed
    };

    return (
      countryNames[this.currentLocale]?.[countryCode] ||
      countryNames["en-US"]?.[countryCode] ||
      countryCode
    );
  }

  getFieldErrorMessage(fieldName: string): string {
    const field = this.internationalForm.get(fieldName);
    if (!field || !field.errors) return "";

    const errorMessages: {
      [locale: string]: { [field: string]: { [error: string]: string } };
    } = {
      "en-US": {
        name: {
          required: "Name is required",
          minlength: "Name must be at least 2 characters",
        },
        email: {
          required: "Email is required",
          email: "Please enter a valid email address",
        },
        phone: {
          phone: "Please enter a valid phone number",
        },
        country: {
          required: "Please select a country",
        },
      },
      "es-ES": {
        name: {
          required: "El nombre es obligatorio",
          minlength: "El nombre debe tener al menos 2 caracteres",
        },
        email: {
          required: "El correo electrónico es obligatorio",
          email: "Por favor ingrese una dirección de correo válida",
        },
        phone: {
          phone: "Por favor ingrese un número de teléfono válido",
        },
        country: {
          required: "Por favor seleccione un país",
        },
      },
      "ar-SA": {
        name: {
          required: "الاسم مطلوب",
          minlength: "يجب أن يكون الاسم حرفين على الأقل",
        },
        email: {
          required: "البريد الإلكتروني مطلوب",
          email: "يرجى إدخال عنوان بريد إلكتروني صحيح",
        },
        phone: {
          phone: "يرجى إدخال رقم هاتف صحيح",
        },
        country: {
          required: "يرجى تحديد بلد",
        },
      },
    };

    const firstError = Object.keys(field.errors)[0];
    const localeMessages =
      errorMessages[this.currentLocale] || errorMessages["en-US"];

    return (
      localeMessages[fieldName]?.[firstError] ||
      errorMessages["en-US"][fieldName]?.[firstError] ||
      "Invalid input"
    );
  }

  getDateFormatExample(): string {
    return formatDate(new Date(), "short", this.currentLocale);
  }

  getNumberFormatExample(): string {
    return formatNumber(1234.56, this.currentLocale, "1.2-2");
  }

  onSubmit() {
    if (this.internationalForm.valid) {
      console.log("Form submitted:", this.internationalForm.value);
      // Process form submission
    } else {
      console.log("Form is invalid");
      // Mark all fields as touched to show validation errors
      Object.keys(this.internationalForm.controls).forEach((key) => {
        this.internationalForm.get(key)?.markAsTouched();
      });
    }
  }
}
```

This completes a comprehensive section on Angular internationalization with detailed examples. The guide now covers HTML5 accessibility, CSS3 for accessible design, Angular accessibility implementation, localization fundamentals, and Angular i18n. Would you like me to continue with additional sections like advanced accessibility patterns, testing tools, or best practices?
