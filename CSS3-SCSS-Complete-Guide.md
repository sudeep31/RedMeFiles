# 🎨 CSS3 & SCSS Complete Guide

## For Senior Frontend Developers & Angular Architects

## 📋 Table of Contents

1. [CSS Box Model Fundamentals](#css-box-model)
2. [CSS3 Advanced Features](#css3-features)
3. [SCSS/Sass Integration with Angular](#scss-angular)
4. [Positioning & Layout Systems](#positioning-layout)
5. [Transforms & Transitions](#transforms-transitions)
6. [Flexbox Complete Guide](#flexbox-guide)
7. [CSS Grid Advanced Layouts](#css-grid-guide)
8. [Real-World Scenarios & Best Practices](#real-world-scenarios)
9. [Performance Optimization](#performance-optimization)
10. [Tricky Interview Questions](#interview-questions)

---

## 📦 CSS Box Model Fundamentals {#css-box-model}

Understanding the CSS Box Model is crucial for precise layout control. Every HTML element is essentially a rectangular box with four key areas.

### **🔍 Box Model Diagram**

```
┌─────────────────────────────────────┐
│               MARGIN                │ ← Transparent area outside border
│  ┌───────────────────────────────┐  │
│  │            BORDER             │  │ ← Visible boundary of element
│  │  ┌─────────────────────────┐  │  │
│  │  │        PADDING          │  │  │ ← Space between content & border
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │     CONTENT       │  │  │  │ ← Actual content area
│  │  │  │    (width ×       │  │  │  │
│  │  │  │     height)       │  │  │  │
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### **💻 Box Model Implementation**

```css
/* 🎯 STANDARD BOX MODEL - Default behavior */
.standard-box {
  width: 200px; /* 📏 Content width only */
  height: 100px; /* 📏 Content height only */
  padding: 20px; /* 📦 Internal spacing */
  border: 5px solid #333; /* 🖼️ Border thickness */
  margin: 10px; /* 🌌 External spacing */

  /* 📊 TOTAL DIMENSIONS CALCULATION:
     Total Width = width + (padding × 2) + (border × 2) + (margin × 2)
                 = 200px + (20px × 2) + (5px × 2) + (10px × 2)
                 = 200px + 40px + 10px + 20px = 270px
     
     Total Height = height + (padding × 2) + (border × 2) + (margin × 2)
                  = 100px + (20px × 2) + (5px × 2) + (10px × 2)
                  = 100px + 40px + 10px + 20px = 170px
  */
}

/* 🎯 BORDER-BOX MODEL - Modern approach */
.border-box {
  box-sizing: border-box; /* 🎛️ Include padding & border in width/height */
  width: 200px; /* 📏 Total width including padding & border */
  height: 100px; /* 📏 Total height including padding & border */
  padding: 20px; /* 📦 Internal spacing (included in width) */
  border: 5px solid #333; /* 🖼️ Border thickness (included in width) */
  margin: 10px; /* 🌌 External spacing (NOT included) */

  /* 📊 BORDER-BOX CALCULATION:
     Content Width = width - (padding × 2) - (border × 2)
                   = 200px - (20px × 2) - (5px × 2)
                   = 200px - 40px - 10px = 150px
     
     Total Width = width + (margin × 2)
                 = 200px + (10px × 2) = 220px
  */
}

/* 🌟 UNIVERSAL BORDER-BOX - Best Practice */
*,
*::before,
*::after {
  box-sizing: border-box; /* 📦 Apply to all elements */
}

/* 🎨 PRACTICAL BOX MODEL EXAMPLE - Card Component */
.product-card {
  /* 📐 DIMENSIONS */
  width: 300px; /* 📏 Card width */
  height: 400px; /* 📏 Card height */

  /* 📦 SPACING */
  padding: 20px; /* 📦 Internal content spacing */
  margin: 16px; /* 🌌 Space between cards */

  /* 🖼️ VISUAL STYLING */
  border: 1px solid #e0e0e0; /* 🔲 Subtle border */
  border-radius: 8px; /* 🔄 Rounded corners */
  background-color: #ffffff; /* 🎨 Background color */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1); /* 🌫️ Depth shadow */

  /* 🔄 INTERACTION */
  transition: transform 0.2s ease, box-shadow 0.2s ease; /* ⚡ Smooth hover */
  cursor: pointer; /* 👆 Indicate clickability */
}

.product-card:hover {
  transform: translateY(-4px); /* 🔺 Lift effect */
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15); /* 🌫️ Enhanced shadow */
}

/* 📱 RESPONSIVE BOX MODEL - Mobile-first approach */
.responsive-container {
  width: 100%; /* 📱 Full width on mobile */
  max-width: 1200px; /* 🖥️ Maximum width for large screens */
  padding: 16px; /* 📦 Base padding */
  margin: 0 auto; /* 🎯 Center horizontally */

  /* 📊 MEDIA QUERIES - Progressive enhancement */
  @media (min-width: 768px) {
    /* 📱➡️💻 Tablet breakpoint */
    padding: 24px; /* 📦 Increased padding */
  }

  @media (min-width: 1024px) {
    /* 💻 Desktop breakpoint */
    padding: 32px; /* 📦 Maximum padding */
  }
}
```

### **🧠 Box Model Theory Explanation**

**Line-by-line breakdown:**

1. **`width: 200px`** - Sets the content area width (excluding padding, border, margin)
2. **`padding: 20px`** - Creates internal space between content and border (20px on all sides)
3. **`border: 5px solid #333`** - Creates a 5px thick solid border around the padding area
4. **`margin: 10px`** - Creates external space around the border (separation from other elements)
5. **`box-sizing: border-box`** - Changes calculation so width includes content + padding + border

**🎯 When to use each approach:**

- **Standard box model**: Legacy systems, specific calculation needs
- **Border-box model**: Modern development (recommended default)
- **Universal border-box**: Production applications for consistent behavior

---

## 🎨 CSS3 Advanced Features {#css3-features}

CSS3 introduced powerful features that enable complex layouts, animations, and visual effects without JavaScript.

### **🎭 CSS3 Selectors & Pseudo-classes**

```css
/* 🎯 ATTRIBUTE SELECTORS - Target elements based on attributes */
input[type="email"] {
  border-color: #4caf50; /* 💚 Green border for email inputs */
  background-image: url("email-icon.svg"); /* 📧 Email icon */
  background-repeat: no-repeat;
  background-position: right 10px center;
  padding-right: 40px; /* 📦 Space for icon */
}

/* 🔍 Line-by-line explanation:
   - input[type="email"]: Targets only email input fields
   - border-color: Changes border to green for visual feedback
   - background-image: Adds email icon for better UX
   - background-position: Positions icon on the right side
   - padding-right: Prevents text overlap with icon
*/

/* 🎯 STRUCTURAL PSEUDO-SELECTORS - Dynamic element targeting */
.gallery-item:nth-child(odd) {
  background-color: #f5f5f5; /* 🎨 Alternate row colors */
}

.gallery-item:nth-child(3n) {
  margin-right: 0; /* 🌌 Remove margin from every 3rd item */
}

.gallery-item:first-child {
  border-top-left-radius: 12px; /* 🔄 Round first item corner */
  border-bottom-left-radius: 12px;
}

.gallery-item:last-child {
  border-top-right-radius: 12px; /* 🔄 Round last item corner */
  border-bottom-right-radius: 12px;
}

/* 🔍 Theory explanation:
   :nth-child(odd) - Selects 1st, 3rd, 5th... elements
   :nth-child(3n) - Selects every 3rd element (3, 6, 9...)
   :first-child - Selects the first child element
   :last-child - Selects the last child element
*/

/* ⚡ PSEUDO-ELEMENTS - Create virtual elements */
.quote-text::before {
  content: '"'; /* 📝 Opening quote mark */
  font-size: 2em; /* 📏 Large quote size */
  color: #2196f3; /* 🎨 Blue accent color */
  position: absolute; /* 📍 Absolute positioning */
  top: -10px; /* ⬆️ Move up slightly */
  left: -20px; /* ⬅️ Move left for overlap */
  font-family: "Georgia", serif; /* 🖋️ Serif font for elegance */
}

.quote-text::after {
  content: '"'; /* 📝 Closing quote mark */
  font-size: 2em; /* 📏 Large quote size */
  color: #2196f3; /* 🎨 Blue accent color */
  position: absolute; /* 📍 Absolute positioning */
  bottom: -20px; /* ⬇️ Move down */
  right: -10px; /* ➡️ Move right */
  font-family: "Georgia", serif; /* 🖋️ Serif font for elegance */
}

/* 🔍 Pseudo-elements explanation:
   ::before - Creates virtual element before content
   ::after - Creates virtual element after content
   content: - Required property to display pseudo-elements
   position: absolute - Allows precise positioning
*/
```

### **🌈 CSS3 Advanced Properties**

```css
/* 🎨 GRADIENTS - Modern background effects */
.gradient-button {
  /* 🌈 LINEAR GRADIENT - Smooth color transition */
  background: linear-gradient(
    45deg,
    /* 📐 45-degree angle */ #ff6b6b 0%,
    /* 🔴 Start color at 0% */ #4ecdc4 50%,
    /* 🟢 Middle color at 50% */ #45b7d1 100% /* 🔵 End color at 100% */
  );

  /* 🎭 Alternative gradient techniques */
  background: radial-gradient(
    ellipse at center,
    /* 🎯 Elliptical shape from center */ #ff9a56 0%,
    /* 🟠 Inner color */ #ff6b6b 100% /* 🔴 Outer color */
  );

  /* 🔍 Gradient explanation:
     linear-gradient: Creates straight-line color transition
     45deg: Angle of gradient direction (0deg = top to bottom)
     Color stops: Define colors at specific positions (0% to 100%)
     radial-gradient: Creates circular/elliptical color transition
  */

  padding: 12px 24px; /* 📦 Button padding */
  border: none; /* 🚫 Remove default border */
  border-radius: 6px; /* 🔄 Rounded corners */
  color: white; /* 🎨 White text for contrast */
  font-weight: 600; /* 📝 Semi-bold text */
  cursor: pointer; /* 👆 Pointer cursor on hover */
  transition: transform 0.2s ease; /* ⚡ Smooth animations */
}

/* 🌫️ BOX SHADOW - Depth and elevation */
.elevated-card {
  /* 🌫️ MULTIPLE SHADOW LAYERS - Realistic depth */
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), /* 📏 Subtle close shadow */ 0 1px
      2px rgba(0, 0, 0, 0.24),
    /* 📏 Secondary shadow */ 0 8px 24px rgba(0, 0, 0, 0.1); /* 📏 Large depth shadow */

  /* 🔍 Box-shadow syntax:
     offset-x: Horizontal shadow offset
     offset-y: Vertical shadow offset  
     blur-radius: Shadow blur amount
     spread-radius: Shadow size expansion
     color: Shadow color (rgba for transparency)
  */

  /* 🎭 INSET SHADOW - Inner depth effect */
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1); /* 🕳️ Inner shadow */
}

/* 🔄 BORDER RADIUS - Advanced corner shaping */
.modern-container {
  /* 🎯 INDIVIDUAL CORNER CONTROL */
  border-top-left-radius: 20px; /* ↖️ Top-left corner */
  border-top-right-radius: 20px; /* ↗️ Top-right corner */
  border-bottom-left-radius: 4px; /* ↙️ Bottom-left corner */
  border-bottom-right-radius: 4px; /* ↘️ Bottom-right corner */

  /* 🎭 ELLIPTICAL CORNERS - Advanced shaping */
  border-radius: 20px / 10px; /* 🔄 Horizontal/Vertical radius */

  /* 🔍 Border-radius explanation:
     Single value: Same radius for all corners
     Two values: horizontal/vertical radius (elliptical)
     Four values: top-left, top-right, bottom-right, bottom-left
  */
}

/* 🎨 TEXT EFFECTS - Advanced typography */
.stylized-heading {
  /* 🌫️ TEXT SHADOW - Depth for text */
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3), /* 📏 Main shadow */ 0 0 8px rgba(255, 255, 255, 0.8); /* ✨ Glow effect */

  /* 🎨 TEXT STROKE - Outline effect */
  -webkit-text-stroke: 2px #333; /* 🖊️ Text outline */
  -webkit-text-fill-color: transparent; /* 🔍 Hollow text */

  /* 🔤 TEXT TRANSFORM - Case modification */
  text-transform: uppercase; /* 🔠 Convert to uppercase */
  letter-spacing: 2px; /* 📏 Space between letters */

  /* 🔍 Text effects explanation:
     text-shadow: Creates shadow behind text
     -webkit-text-stroke: Adds outline to text
     text-transform: Changes text case
     letter-spacing: Adjusts space between characters
  */
}
```

### **📱 CSS3 Media Queries & Responsive Design**

```css
/* 📱 MOBILE FIRST APPROACH - Start with smallest screens */
.responsive-grid {
  display: grid; /* 📊 Use CSS Grid */
  grid-template-columns: 1fr; /* 📱 Single column on mobile */
  gap: 16px; /* 🌌 Space between grid items */
  padding: 16px; /* 📦 Container padding */

  /* 🔍 Mobile-first explanation:
     Start with mobile styles as base
     Use min-width media queries to enhance for larger screens
     1fr means "1 fraction" - takes available space
  */
}

/* 📱➡️💻 TABLET BREAKPOINT - Medium screens */
@media (min-width: 768px) {
  .responsive-grid {
    grid-template-columns: repeat(2, 1fr); /* 📊 Two columns */
    gap: 24px; /* 🌌 Increased spacing */
    padding: 24px; /* 📦 More padding */

    /* 🔍 Tablet enhancement:
       repeat(2, 1fr): Creates 2 equal-width columns
       Increased spacing for better visual hierarchy
    */
  }
}

/* 💻 DESKTOP BREAKPOINT - Large screens */
@media (min-width: 1024px) {
  .responsive-grid {
    grid-template-columns: repeat(3, 1fr); /* 📊 Three columns */
    gap: 32px; /* 🌌 Maximum spacing */
    padding: 32px; /* 📦 Maximum padding */
    max-width: 1200px; /* 📏 Limit maximum width */
    margin: 0 auto; /* 🎯 Center horizontally */
  }
}

/* 🖥️ LARGE DESKTOP BREAKPOINT - Extra large screens */
@media (min-width: 1440px) {
  .responsive-grid {
    grid-template-columns: repeat(4, 1fr); /* 📊 Four columns */
    gap: 40px; /* 🌌 Extra spacing */
  }
}

/* 🎯 ADVANCED MEDIA QUERY CONDITIONS */
@media (min-width: 768px) and (max-width: 1023px) {
  /* 📱💻 Tablet-only styles */
  .tablet-specific {
    font-size: 1.1em; /* 📝 Slightly larger text */
  }
}

@media (orientation: landscape) {
  /* 📱🔄 Landscape mode adjustments */
  .landscape-header {
    height: 60px; /* 📏 Reduced height in landscape */
  }
}

@media (prefers-reduced-motion: reduce) {
  /* ♿ Accessibility - Reduced motion preference */
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* 🔍 Advanced media query explanation:
   and: Combines multiple conditions
   orientation: landscape/portrait detection
   prefers-reduced-motion: Accessibility for motion sensitivity
   !important: Overrides other styles (use sparingly)
*/
```

This is Part 1 of the comprehensive CSS3 & SCSS guide. The content covers:

1. **CSS Box Model Fundamentals** - Complete understanding with calculations
2. **CSS3 Advanced Features** - Selectors, pseudo-classes, gradients, shadows
3. **Responsive Design** - Mobile-first approach with detailed explanations

Each code block includes:

- **Line-by-line explanations** in comments
- **Theory sections** explaining the concepts
- **Real-world examples** with practical applications
- **Best practices** for production use

---

## 🎯 SCSS/Sass Integration with Angular {#scss-angular}

SCSS (Sassy CSS) is a CSS preprocessor that adds powerful features like variables, nesting, mixins, and functions. Angular has built-in SCSS support for component-level and global styling.

### **🚀 Angular SCSS Setup & Configuration**

```typescript
// 🔧 ANGULAR.JSON SCSS CONFIGURATION
{
  "projects": {
    "your-app": {
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/your-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",
            "inlineStyleLanguage": "scss",  // 🎨 Enable SCSS globally
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"              // 📁 Global SCSS entry point
            ],
            "stylePreprocessorOptions": {    // 🔧 SCSS compiler options
              "includePaths": [
                "src/app/shared/styles",     // 📂 Shared styles directory
                "src/assets/scss",           // 📂 SCSS assets directory
                "node_modules"               // 📦 Node modules for libraries
              ]
            }
          }
        }
      }
    }
  }
}

/* 🔍 Configuration explanation:
   inlineStyleLanguage: Sets SCSS as default for component styles
   styles array: Defines global SCSS entry points
   includePaths: Directories for @import statements
   stylePreprocessorOptions: SCSS compiler settings
*/
```

### **🎨 SCSS Variables & Theming System**

```scss
// 📁 src/app/shared/styles/_variables.scss
// 🎨 COLOR PALETTE - Semantic color naming
$primary-colors: (
  50: #e3f2fd,
  // 🔵 Lightest blue
  100: #bbdefb,
  // 🔵 Very light blue
  200: #90caf9,
  // 🔵 Light blue
  300: #64b5f6,
  // 🔵 Medium light blue
  400: #42a5f5,
  // 🔵 Medium blue
  500: #2196f3,
  // 🔵 Primary blue
  600: #1e88e5,
  // 🔵 Medium dark blue
  700: #1976d2,
  // 🔵 Dark blue
  800: #1565c0,
  // 🔵 Very dark blue
  900: #0d47a1 // 🔵 Darkest blue,,
) !default;

// 🔍 SCSS Map explanation:
// Maps store key-value pairs for organized data
// !default ensures variables can be overridden
// Semantic naming follows Material Design principles

$accent-colors: (
  50: #fce4ec,
  // 🌸 Lightest pink
  500: #e91e63,
  // 🌸 Primary pink
  900: #880e4f // 🌸 Darkest pink,,
) !default;

// 🎯 SEMANTIC COLOR VARIABLES - Easy to remember names
$color-primary: map-get($primary-colors, 500) !default; // 🔵 #2196f3
$color-primary-dark: map-get($primary-colors, 700) !default; // 🔵 #1976d2
$color-primary-light: map-get($primary-colors, 100) !default; // 🔵 #bbdefb
$color-accent: map-get($accent-colors, 500) !default; // 🌸 #e91e63

// 🎨 GRAYSCALE PALETTE - Neutral colors
$color-black: #000000 !default; // ⚫ Pure black
$color-white: #ffffff !default; // ⚪ Pure white
$color-gray-50: #fafafa !default; // 🔘 Lightest gray
$color-gray-100: #f5f5f5 !default; // 🔘 Very light gray
$color-gray-300: #e0e0e0 !default; // 🔘 Light gray
$color-gray-500: #9e9e9e !default; // 🔘 Medium gray
$color-gray-700: #616161 !default; // 🔘 Dark gray
$color-gray-900: #212121 !default; // 🔘 Very dark gray

// 🚦 STATUS COLORS - User feedback colors
$color-success: #4caf50 !default; // ✅ Green for success
$color-warning: #ff9800 !default; // ⚠️ Orange for warnings
$color-error: #f44336 !default; // ❌ Red for errors
$color-info: #2196f3 !default; // ℹ️ Blue for information

// 📏 SPACING SYSTEM - Consistent spacing scale
$spacing-unit: 8px !default; // 📐 Base spacing unit (8px grid)
$spacing-xs: $spacing-unit * 0.5 !default; // 4px
$spacing-sm: $spacing-unit * 1 !default; // 8px
$spacing-md: $spacing-unit * 2 !default; // 16px
$spacing-lg: $spacing-unit * 3 !default; // 24px
$spacing-xl: $spacing-unit * 4 !default; // 32px
$spacing-xxl: $spacing-unit * 6 !default; // 48px

/* 🔍 Spacing system explanation:
   8px grid system provides visual harmony
   Consistent spacing improves user experience
   Mathematical progression (0.5x, 1x, 2x, 3x, 4x, 6x)
   Easy to remember and implement
*/

// 🔤 TYPOGRAPHY SCALE - Modular type scale
$font-family-primary: "Roboto", "Helvetica Neue", Arial, sans-serif !default;
$font-family-mono: "Roboto Mono", "Courier New", monospace !default;

// 📝 Font sizes following modular scale (1.25 ratio)
$font-size-xs: 0.75rem !default; // 12px - Small text
$font-size-sm: 0.875rem !default; // 14px - Body small
$font-size-base: 1rem !default; // 16px - Body text
$font-size-lg: 1.125rem !default; // 18px - Large text
$font-size-xl: 1.25rem !default; // 20px - Subheadings
$font-size-xxl: 1.5rem !default; // 24px - Headings
$font-size-xxxl: 2rem !default; // 32px - Large headings

// 📐 Line heights for optimal readability
$line-height-tight: 1.25 !default; // 🔗 For headings
$line-height-normal: 1.5 !default; // 📄 For body text
$line-height-loose: 1.75 !default; // 📖 For large text blocks

// 🎯 BREAKPOINTS - Responsive design breakpoints
$breakpoint-xs: 0 !default; // 📱 Extra small devices
$breakpoint-sm: 576px !default; // 📱 Small devices
$breakpoint-md: 768px !default; // 📱💻 Medium devices (tablets)
$breakpoint-lg: 992px !default; // 💻 Large devices (desktops)
$breakpoint-xl: 1200px !default; // 🖥️ Extra large devices
$breakpoint-xxl: 1400px !default; // 🖥️ Extra extra large devices

// 🌊 Z-INDEX SCALE - Layering system
$z-index-dropdown: 1000 !default; // 📋 Dropdown menus
$z-index-sticky: 1020 !default; // 📌 Sticky elements
$z-index-fixed: 1030 !default; // 📍 Fixed elements
$z-index-modal-backdrop: 1040 !default; // 🌫️ Modal backdrop
$z-index-modal: 1050 !default; // 🗨️ Modal dialog
$z-index-popover: 1060 !default; // 💭 Popover content
$z-index-tooltip: 1070 !default; // 💬 Tooltip content
$z-index-toast: 1080 !default; // 🍞 Toast notifications

/* 🔍 Z-index system explanation:
   Organized layering prevents stacking conflicts
   10-point increments allow for intermediate values
   Semantic naming makes purpose clear
   Consistent across entire application
*/

// 🎭 ANIMATION & TRANSITIONS - Smooth interactions
$transition-speed-fast: 0.15s !default; // ⚡ Fast transitions
$transition-speed-normal: 0.25s !default; // 🔄 Normal transitions
$transition-speed-slow: 0.4s !default; // 🐌 Slow transitions

$transition-easing-ease: ease !default; // 📈 Standard easing
$transition-easing-ease-in: ease-in !default; // 📈 Accelerating
$transition-easing-ease-out: ease-out !default; // 📉 Decelerating
$transition-easing-ease-in-out: ease-in-out !default; // 📊 Smooth curve

// 🎯 Common transition combinations
$transition-all: all $transition-speed-normal $transition-easing-ease !default;
$transition-color: color $transition-speed-fast $transition-easing-ease !default;
$transition-transform: transform $transition-speed-normal
  $transition-easing-ease-out !default;
```

### **🧩 SCSS Mixins - Reusable Code Blocks**

```scss
// 📁 src/app/shared/styles/_mixins.scss

// 🎯 BUTTON MIXIN - Consistent button styling
@mixin button-base(
  $bg-color: $color-primary,
  $text-color: $color-white,
  $padding: $spacing-md $spacing-lg,
  $border-radius: 4px
) {
  // 🎨 VISUAL STYLING
  background-color: $bg-color; // 🎨 Background color parameter
  color: $text-color; // 🔤 Text color parameter
  border: none; // 🚫 Remove default border
  border-radius: $border-radius; // 🔄 Rounded corners
  padding: $padding; // 📦 Internal spacing

  // 📝 TYPOGRAPHY
  font-family: $font-family-primary; // 🔤 Primary font
  font-size: $font-size-base; // 📏 Base font size
  font-weight: 500; // 📝 Medium weight
  line-height: $line-height-normal; // 📏 Readable line height
  text-decoration: none; // 🚫 No underline
  text-align: center; // 🎯 Center text

  // 🖱️ INTERACTION
  cursor: pointer; // 👆 Pointer cursor
  user-select: none; // 🚫 Prevent text selection
  display: inline-block; // 📦 Inline block for sizing
  vertical-align: middle; // 📐 Vertical alignment

  // ⚡ TRANSITIONS
  transition: background-color $transition-speed-normal $transition-easing-ease,
    transform $transition-speed-fast $transition-easing-ease-out,
    box-shadow $transition-speed-normal $transition-easing-ease;

  // 🎭 HOVER STATE
  &:hover {
    background-color: darken($bg-color, 8%); // 🌑 Darker on hover
    transform: translateY(-1px); // ⬆️ Subtle lift effect
    box-shadow: 0 4px 12px rgba($bg-color, 0.3); // 🌫️ Elevated shadow
  }

  // 👆 ACTIVE STATE
  &:active {
    transform: translateY(0); // ⬇️ Press down effect
    box-shadow: 0 2px 6px rgba($bg-color, 0.3); // 🌫️ Reduced shadow
  }

  // 🚫 DISABLED STATE
  &:disabled {
    background-color: $color-gray-300; // 🔘 Gray background
    color: $color-gray-500; // 🔘 Gray text
    cursor: not-allowed; // 🚫 Not allowed cursor
    transform: none; // 🚫 No hover effects
    box-shadow: none; // 🚫 No shadow
  }
}

/* 🔍 Mixin explanation:
   @mixin: Defines reusable code block
   Parameters with defaults allow customization
   & refers to parent selector (the element using the mixin)
   darken() is SCSS function to darken colors
   Comprehensive state handling (hover, active, disabled)
*/

// 📱 RESPONSIVE MIXIN - Media query helper
@mixin respond-to($breakpoint) {
  // 📱 PHONE PORTRAIT
  @if $breakpoint == phone {
    @media (max-width: #{$breakpoint-sm - 1px}) {
      @content; // 📝 Content passed to mixin
    }
  }

  // 📱 TABLET PORTRAIT
  @else if $breakpoint == tablet {
    @media (min-width: #{$breakpoint-md}) and (max-width: #{$breakpoint-lg - 1px}) {
      @content;
    }
  }

  // 💻 DESKTOP
  @else if $breakpoint == desktop {
    @media (min-width: #{$breakpoint-lg}) {
      @content;
    }
  }

  // 🖥️ LARGE DESKTOP
  @else if $breakpoint == large-desktop {
    @media (min-width: #{$breakpoint-xl}) {
      @content;
    }
  }

  // 🎯 CUSTOM BREAKPOINT
  @else {
    @media (min-width: #{$breakpoint}) {
      @content;
    }
  }
}

/* 🔍 Responsive mixin explanation:
   @content: Inserts styles passed to the mixin
   #{}: SCSS interpolation for dynamic values
   Conditional logic with @if/@else
   Named breakpoints for easier maintenance
*/

// 🎨 CARD MIXIN - Consistent card styling
@mixin card(
  $padding: $spacing-lg,
  $border-radius: 8px,
  $shadow: true,
  $hover-effect: true
) {
  // 🎨 BASIC STYLING
  background-color: $color-white; // ⚪ White background
  border-radius: $border-radius; // 🔄 Rounded corners
  padding: $padding; // 📦 Internal spacing
  border: 1px solid $color-gray-100; // 🖼️ Subtle border

  // 🌫️ CONDITIONAL SHADOW
  @if $shadow {
    box-shadow: 0 2px 8px rgba($color-black, 0.1); // 🌫️ Subtle depth
  }

  // 🎭 CONDITIONAL HOVER EFFECT
  @if $hover-effect {
    transition: $transition-all; // ⚡ Smooth transitions

    &:hover {
      transform: translateY(-2px); // ⬆️ Lift on hover
      box-shadow: 0 8px 24px rgba($color-black, 0.15); // 🌫️ Enhanced shadow
    }
  }
}

/* 🔍 Card mixin explanation:
   Boolean parameters control optional features
   @if conditionals add features only when requested
   Consistent styling across all card components
   Hover effects enhance user experience
*/

// 🔤 TEXT TRUNCATION MIXIN - Handle overflow text
@mixin text-truncate($lines: 1) {
  // 📝 SINGLE LINE TRUNCATION
  @if $lines == 1 {
    overflow: hidden; // 🚫 Hide overflow
    text-overflow: ellipsis; // ... Ellipsis for cut text
    white-space: nowrap; // 🚫 Prevent line breaks
  }

  // 📝 MULTI-LINE TRUNCATION
  @else {
    display: -webkit-box; // 📦 Webkit flexbox
    -webkit-line-clamp: $lines; // 📏 Number of lines
    -webkit-box-orient: vertical; // 📐 Vertical orientation
    overflow: hidden; // 🚫 Hide overflow
    text-overflow: ellipsis; // ... Ellipsis for cut text
  }
}

/* 🔍 Text truncation explanation:
   Single vs multi-line truncation handling
   Webkit-specific properties for multi-line
   Ellipsis indicates truncated content
   Responsive text overflow solution
*/

// 🎯 FLEXBOX CENTERING MIXIN - Perfect centering
@mixin flex-center($direction: row) {
  display: flex; // 📦 Flexbox layout
  align-items: center; // 📐 Vertical centering
  justify-content: center; // 📐 Horizontal centering
  flex-direction: $direction; // 🔄 Layout direction
}

/* 🔍 Flexbox centering explanation:
   Most reliable centering method
   Works for any content type
   Flexible direction parameter
   No margin/padding calculations needed
*/
```

### **🧩 SCSS Functions - Dynamic Value Generation**

```scss
// 📁 src/app/shared/styles/_functions.scss

// 🎨 COLOR FUNCTIONS - Dynamic color manipulation
@function color-variant($color, $variant: 500) {
  // 🔍 Function explanation:
  // Returns specific shade from color map
  // Fallback to base color if variant not found

  @if type-of($color) == "map" {
    @return map-get($color, $variant); // 📊 Get from color map
  } @else {
    @return $color; // 🔄 Return original color
  }
}

// 🌟 EXAMPLE USAGE:
// $button-color: color-variant($primary-colors, 600); // Returns #1e88e5

// 📏 SPACING FUNCTION - Calculate consistent spacing
@function spacing($multiplier: 1) {
  // 🔍 Function explanation:
  // Multiplies base spacing unit by given factor
  // Ensures consistent spacing throughout app

  @return $spacing-unit * $multiplier; // 📐 Calculate spacing
}

// 🌟 EXAMPLE USAGE:
// margin: spacing(2);    // Returns 16px (8px × 2)
// padding: spacing(1.5); // Returns 12px (8px × 1.5)

// 📱 RESPONSIVE VALUE FUNCTION - Fluid scaling
@function fluid-size(
  $min-size,
  $max-size,
  $min-width: $breakpoint-sm,
  $max-width: $breakpoint-xl
) {
  // 🔍 Function explanation:
  // Creates CSS calc() for fluid scaling between breakpoints
  // Smoothly scales values based on viewport width

  $slope: ($max-size - $min-size) / ($max-width - $min-width);
  $intercept: $min-size - $slope * $min-width;

  @return calc(#{$intercept}px + #{$slope * 100}vw);
}

// 🌟 EXAMPLE USAGE:
// font-size: fluid-size(16, 24); // Scales from 16px to 24px across breakpoints

// 🎯 Z-INDEX FUNCTION - Organized layering
@function z-index($layer) {
  // 🔍 Function explanation:
  // Returns z-index value from organized map
  // Prevents stacking context conflicts

  $z-indexes: (
    "dropdown": $z-index-dropdown,
    "modal": $z-index-modal,
    "tooltip": $z-index-tooltip,
    "toast": $z-index-toast,
  );

  @return map-get($z-indexes, $layer);
}

// 🌟 EXAMPLE USAGE:
// z-index: z-index('modal');    // Returns 1050
// z-index: z-index('tooltip');  // Returns 1070
```

### **🏗️ Angular Component SCSS Architecture**

```scss
// 📁 src/app/components/product-card/product-card.component.scss

// 🔗 IMPORT SHARED STYLES
@import "shared/styles/variables";
@import "shared/styles/mixins";
@import "shared/styles/functions";

// 🎯 COMPONENT-SPECIFIC VARIABLES
$card-width: 300px; // 📏 Fixed card width
$card-height: 400px; // 📏 Fixed card height
$card-border-radius: 12px; // 🔄 Rounded corners
$card-transition-duration: 0.3s; // ⚡ Animation speed

// 🏠 HOST ELEMENT STYLING
:host {
  // 🔍 :host explanation:
  // Targets the component's host element
  // Equivalent to styling the component tag itself

  display: block; // 📦 Block-level element
  width: $card-width; // 📏 Component width
  margin: $spacing-md; // 🌌 External spacing

  // 📱 RESPONSIVE HOST STYLING
  @include respond-to(tablet) {
    width: calc(50% - #{$spacing-md}); // 📱💻 Half width on tablets
  }

  @include respond-to(phone) {
    width: calc(100% - #{$spacing-md}); // 📱 Full width on phones
    margin: $spacing-sm; // 🌌 Reduced margin
  }
}

// 🎨 COMPONENT STYLING
.product-card {
  // 🔧 APPLY CARD MIXIN
  @include card(
    $padding: $spacing-lg,
    // 📦 Internal spacing
    $border-radius: $card-border-radius,
    // 🔄 Corner rounding
    $shadow: true,
    // 🌫️ Enable shadow
    $hover-effect: true // 🎭 Enable hover effect
  );

  height: $card-height; // 📏 Fixed height
  display: flex; // 📦 Flexbox layout
  flex-direction: column; // 📐 Vertical stacking
  overflow: hidden; // 🚫 Hide overflow content

  // 🔍 Component structure explanation:
  // Flexbox allows flexible content arrangement
  // Fixed height maintains consistent grid
  // Overflow hidden prevents content spillage

  &__image {
    // 🔗 BEM NAMING CONVENTION
    // Block: product-card
    // Element: image
    // Modifier: (none in this case)

    width: 100%; // 📏 Full width
    height: 200px; // 📏 Fixed height
    object-fit: cover; // 🖼️ Crop to fit
    object-position: center; // 🎯 Center crop
    border-radius: $card-border-radius $card-border-radius 0 0; // 🔄 Top corners only

    // 🔍 Image styling explanation:
    // object-fit: cover maintains aspect ratio while filling space
    // object-position: center focuses on center of image
    // Border-radius only on top to match card design
  }

  &__content {
    padding: $spacing-lg; // 📦 Content padding
    flex: 1; // 📈 Grow to fill space
    display: flex; // 📦 Nested flexbox
    flex-direction: column; // 📐 Vertical stacking
    justify-content: space-between; // 📐 Space distribution

    // 🔍 Content area explanation:
    // flex: 1 makes content area expand to fill available space
    // Nested flexbox allows precise content positioning
    // justify-content: space-between pushes footer to bottom
  }

  &__title {
    @include text-truncate(2); // 📝 Truncate after 2 lines

    font-size: $font-size-lg; // 📏 Large title text
    font-weight: 600; // 📝 Semi-bold weight
    color: $color-gray-900; // 🎨 Dark gray text
    margin-bottom: $spacing-sm; // 🌌 Bottom spacing
    line-height: $line-height-tight; // 📏 Tight line spacing

    // 🔍 Title styling explanation:
    // Text truncation prevents layout breaking
    // Semantic font sizes maintain hierarchy
    // Margin creates visual separation
  }

  &__description {
    @include text-truncate(3); // 📝 Truncate after 3 lines

    font-size: $font-size-sm; // 📏 Small description text
    color: $color-gray-700; // 🎨 Medium gray text
    line-height: $line-height-normal; // 📏 Normal line spacing
    margin-bottom: $spacing-md; // 🌌 Bottom spacing
    flex: 1; // 📈 Expand to fill space

    // 🔍 Description styling explanation:
    // Smaller font size for secondary content
    // flex: 1 allows description to take available space
    // Multi-line truncation maintains card height
  }

  &__footer {
    display: flex; // 📦 Flexbox layout
    justify-content: space-between; // 📐 Space between items
    align-items: center; // 📐 Vertical centering
    margin-top: auto; // ⬆️ Push to bottom

    // 🔍 Footer explanation:
    // margin-top: auto pushes footer to bottom of flex container
    // Flexbox distributes price and button across width
    // align-items centers content vertically
  }

  &__price {
    font-size: $font-size-xl; // 📏 Large price text
    font-weight: 700; // 📝 Bold weight
    color: $color-primary; // 🎨 Primary brand color

    // 🔍 Price styling explanation:
    // Prominent styling draws attention to price
    // Primary color reinforces brand
    // Bold weight emphasizes importance
  }

  &__button {
    @include button-base(
      $bg-color: $color-primary,
      // 🎨 Primary button color
      $text-color: $color-white,
      // 🔤 White text
      $padding: $spacing-sm $spacing-md,
      // 📦 Compact padding
      $border-radius: 6px // 🔄 Rounded button
    );

    font-size: $font-size-sm; // 📏 Small button text
    min-width: 100px; // 📏 Minimum button width

    // 🔍 Button styling explanation:
    // Mixin provides consistent button behavior
    // Smaller size fits card proportions
    // min-width prevents button from being too narrow
  }

  // 🎭 LOADING STATE
  &--loading {
    .product-card__image {
      background: linear-gradient(
        90deg,
        $color-gray-100 25%,
        // 🔘 Light gray
        $color-gray-50 50%,
        // 🔘 Very light gray
        $color-gray-100 75% // 🔘 Light gray
      );
      background-size: 200% 100%; // 📏 Double width for animation
      animation: loading-shimmer 1.5s infinite; // 🔄 Shimmer effect
    }

    .product-card__title,
    .product-card__description {
      background: $color-gray-200; // 🔘 Gray placeholder
      border-radius: 4px; // 🔄 Rounded placeholder
      color: transparent; // 🔍 Hide text
    }
  }

  // 🎭 ERROR STATE
  &--error {
    border: 2px solid $color-error; // ❌ Red border for error

    .product-card__image {
      background: $color-gray-100; // 🔘 Gray background
      display: flex; // 📦 Flexbox centering
      align-items: center; // 📐 Vertical center
      justify-content: center; // 📐 Horizontal center

      &::before {
        content: "⚠️"; // ⚠️ Error icon
        font-size: 2rem; // 📏 Large icon
      }
    }
  }
}

// 🔄 KEYFRAME ANIMATIONS
@keyframes loading-shimmer {
  0% {
    background-position: -200% 0; // ⬅️ Start off-screen left
  }
  100% {
    background-position: 200% 0; // ➡️ End off-screen right
  }
}

/* 🔍 Animation explanation:
   Keyframes define animation steps from 0% to 100%
   background-position creates sliding shimmer effect
   Infinite iteration creates continuous loading indication
   CSS animations are more performant than JavaScript
*/

// 📱 RESPONSIVE COMPONENT DESIGN
@include respond-to(tablet) {
  .product-card {
    &__image {
      height: 160px; // 📏 Smaller image on tablets
    }

    &__content {
      padding: $spacing-md; // 📦 Reduced padding
    }

    &__title {
      font-size: $font-size-base; // 📏 Smaller title
    }
  }
}

@include respond-to(phone) {
  .product-card {
    height: auto; // 📏 Auto height on mobile

    &__image {
      height: 200px; // 📏 Maintain image prominence
    }

    &__footer {
      flex-direction: column; // 📐 Stack vertically
      align-items: stretch; // 📐 Full width items
      gap: $spacing-sm; // 🌌 Space between items
    }

    &__button {
      width: 100%; // 📏 Full-width button
    }
  }
}

/* 🔍 Responsive design explanation:
   Mobile-first approach with progressive enhancement
   Maintain visual hierarchy across breakpoints
   Stack elements vertically on small screens
   Full-width buttons improve touch targets
*/
```

This is Part 2 covering:

1. **SCSS/Angular Integration** - Complete setup and configuration
2. **SCSS Variables & Theming** - Comprehensive design system
3. **SCSS Mixins & Functions** - Reusable code patterns
4. **Angular Component SCSS** - Real-world component styling

Would you like me to continue with Part 3 covering positioning systems, transforms, and transitions?

---

## 🎯 CSS Positioning & Layout Systems {#positioning-layouts}

CSS positioning is fundamental to creating complex layouts. Understanding the positioning context and stacking order is crucial for modern web development.

### **📐 CSS Positioning Types - Complete Understanding**

```css
/* 🔍 STATIC POSITIONING - Default behavior */
.element-static {
  position: static; /* 📍 Default value - elements flow normally */
  /* top, right, bottom, left have NO EFFECT */
  /* z-index has NO EFFECT */
}

/*
🔍 Static positioning explanation:
- Elements follow normal document flow
- Position properties (top, right, bottom, left) ignored
- Cannot create stacking contexts
- Most common positioning type
- Block elements stack vertically, inline elements horizontally
*/

/* 📍 RELATIVE POSITIONING - Offset from original position */
.element-relative {
  position: relative; /* 📍 Positioned relative to its original location */
  top: 20px; /* ⬇️ Move 20px down from original position */
  left: 30px; /* ➡️ Move 30px right from original position */
  z-index: 10; /* 🌊 Can participate in stacking context */
}

/*
🔍 Relative positioning explanation:
- Element maintains its space in document flow
- Visual position is offset from original location
- Other elements act as if it's still in original position
- Creates new stacking context for child elements
- Commonly used as positioning context for absolute children
*/

/* 📌 ABSOLUTE POSITIONING - Positioned relative to nearest positioned ancestor */
.container {
  position: relative; /* 📍 Creates positioning context for children */
  width: 500px; /* 📏 Container width */
  height: 300px; /* 📏 Container height */
}

.element-absolute {
  position: absolute; /* 📌 Positioned relative to .container */
  top: 50%; /* 📐 50% from top of container */
  left: 50%; /* 📐 50% from left of container */
  transform: translate(-50%, -50%); /* 🎯 Perfect centering technique */
  z-index: 20; /* 🌊 Higher stacking order */
}

/*
🔍 Absolute positioning explanation:
- Element removed from normal document flow
- Positioned relative to nearest positioned ancestor (not static)
- If no positioned ancestor, uses initial containing block (viewport)
- Does not affect positioning of other elements
- Creates new stacking context
*/

/* 📍 FIXED POSITIONING - Positioned relative to viewport */
.element-fixed {
  position: fixed; /* 📍 Fixed to viewport */
  top: 0; /* ⬆️ Stick to top of viewport */
  right: 0; /* ➡️ Stick to right of viewport */
  width: 300px; /* 📏 Fixed width */
  height: 100vh; /* 📏 Full viewport height */
  z-index: 1000; /* 🌊 High stacking order */
  background: rgba(0, 0, 0, 0.8); /* 🎨 Semi-transparent background */
}

/*
🔍 Fixed positioning explanation:
- Always positioned relative to viewport
- Unaffected by scrolling (stays in same position)
- Removed from document flow
- Perfect for headers, sidebars, modal overlays
- Creates new stacking context
*/

/* 📌 STICKY POSITIONING - Hybrid behavior */
.element-sticky {
  position: sticky; /* 📌 Sticky positioning */
  top: 20px; /* 📐 Stick when 20px from top */
  background: #fff; /* 🎨 Background for visibility */
  z-index: 100; /* 🌊 Above other content */
  padding: 10px; /* 📦 Internal spacing */
}

/*
🔍 Sticky positioning explanation:
- Behaves like relative until scroll threshold is reached
- Then behaves like fixed within containing block
- Stays within parent element boundaries
- Perfect for table headers, navigation bars
- Requires threshold value (top, bottom, left, or right)
*/
```

### **🎯 Advanced Positioning Techniques**

```css
/* 🎯 PERFECT CENTERING - Multiple methods */

/* METHOD 1: Absolute + Transform (works with unknown dimensions) */
.center-absolute {
  position: absolute;
  top: 50%; /* 📐 50% from top */
  left: 50%; /* 📐 50% from left */
  transform: translate(-50%, -50%); /* 🔄 Offset by half of element size */
}

/*
🔍 Absolute centering explanation:
- 50% positions element's top-left corner at center
- translate(-50%, -50%) moves element back by half its dimensions
- Works regardless of element size
- Most flexible centering method
*/

/* METHOD 2: Absolute + Margin Auto (requires known dimensions) */
.center-margin {
  position: absolute;
  top: 0; /* ⬆️ Top boundary */
  right: 0; /* ➡️ Right boundary */
  bottom: 0; /* ⬇️ Bottom boundary */
  left: 0; /* ⬅️ Left boundary */
  width: 300px; /* 📏 Known width required */
  height: 200px; /* 📏 Known height required */
  margin: auto; /* 🎯 Auto margin centers */
}

/*
🔍 Margin auto centering explanation:
- All position values set to 0 stretches element
- Fixed dimensions constrain the stretching
- Auto margins center the constrained element
- Requires known dimensions
*/

/* METHOD 3: Flexbox Centering (modern approach) */
.center-flexbox {
  display: flex; /* 📦 Flex container */
  justify-content: center; /* 📐 Horizontal centering */
  align-items: center; /* 📐 Vertical centering */
  height: 100vh; /* 📏 Full viewport height */
}

/*
🔍 Flexbox centering explanation:
- Most intuitive centering method
- Works with any content size
- No positioning required
- Responsive by default
*/

/* 🌊 Z-INDEX AND STACKING CONTEXTS */
.stacking-context-parent {
  position: relative; /* 📍 Creates positioning context */
  z-index: 1; /* 🌊 Low stacking order */
}

.stacking-context-child {
  position: absolute; /* 📌 Child positioning */
  z-index: 999; /* 🌊 High z-index BUT... */
  /*
  🚨 IMPORTANT: This child cannot appear above elements
  outside its stacking context, even with higher z-index
  */
}

.outside-element {
  position: relative; /* 📍 Different stacking context */
  z-index: 2; /* 🌊 Higher than parent context */
  /*
  ✅ This element will appear above the child with z-index: 999
  because it's in a higher stacking context
  */
}

/*
🔍 Stacking context explanation:
- Created by certain CSS properties (position + z-index, opacity, transform, etc.)
- Z-index only compares elements within the same stacking context
- Child elements cannot escape their parent's stacking context
- Understanding this prevents z-index frustrations
*/

/* 📐 RESPONSIVE POSITIONING */
.responsive-positioning {
  position: fixed; /* 📍 Fixed positioning */
  bottom: 20px; /* ⬇️ Distance from bottom */
  right: 20px; /* ➡️ Distance from right */

  /* 📱 MOBILE ADJUSTMENTS */
  @media (max-width: 768px) {
    position: absolute; /* 📌 Change to absolute on mobile */
    bottom: 10px; /* ⬇️ Reduced spacing */
    right: 10px; /* ➡️ Reduced spacing */
  }
}

/*
🔍 Responsive positioning explanation:
- Fixed positioning can cause issues on mobile
- Touch interfaces may need different positioning
- Consider viewport height changes on mobile
- Test across devices for optimal UX
*/
```

### **🎨 CSS Transforms - Shape and Position Manipulation**

```css
/* 🔄 2D TRANSFORMS - Basic transformations */

/* ↔️ TRANSLATE - Move elements */
.transform-translate {
  transform: translate(50px, 100px); /* ➡️⬇️ Move right 50px, down 100px */
  /* Alternative: translateX(50px) translateY(100px) */
}

/*
🔍 Translate explanation:
- Moves element without affecting other elements
- Values can be px, %, em, rem, etc.
- Percentage values relative to element's own size
- Does not trigger layout recalculation (performant)
*/

/* 🔄 SCALE - Resize elements */
.transform-scale {
  transform: scale(1.5); /* 📈 Scale 1.5x (150% of original size) */
  /* Alternative: scale(2, 0.5) - 2x width, 0.5x height */
  /* Alternative: scaleX(2) scaleY(0.5) - separate axis control */
}

/*
🔍 Scale explanation:
- 1 = original size, 2 = double size, 0.5 = half size
- Scales from element's center by default
- Can specify different values for X and Y axes
- Use transform-origin to change scaling point
*/

/* 🔄 ROTATE - Spin elements */
.transform-rotate {
  transform: rotate(45deg); /* 🔄 Rotate 45 degrees clockwise */
  /* Negative values rotate counter-clockwise */
  /* Units: deg, rad, grad, turn */
}

/*
🔍 Rotate explanation:
- Positive values = clockwise rotation
- Negative values = counter-clockwise rotation
- 360deg = 1turn = full rotation
- Rotates around element's center by default
*/

/* ↔️ SKEW - Distort elements */
.transform-skew {
  transform: skew(15deg, 10deg); /* 📐 Skew X-axis 15°, Y-axis 10° */
  /* Alternative: skewX(15deg) skewY(10deg) */
}

/*
🔍 Skew explanation:
- Creates parallelogram effect
- First value = X-axis skew, second = Y-axis skew
- Positive values skew towards upper-left
- Rarely used but useful for creative effects
*/

/* 🔗 COMBINING TRANSFORMS - Multiple effects */
.transform-combined {
  transform: translate(50px, 100px) /* ➡️⬇️ First: move position */ rotate(
      30deg
    )
    /* 🔄 Second: rotate */ scale(1.2); /* 📈 Third: scale up */

  /* 
  ⚡ PERFORMANCE TIP: Combine transforms in single property
  Order matters! Applied right-to-left in function list
  */
}

/*
🔍 Transform combination explanation:
- Order of operations matters
- Functions applied from right to left
- More efficient than separate transform properties
- Common pattern: translate → rotate → scale
*/

/* 📍 TRANSFORM ORIGIN - Change transformation point */
.transform-origin {
  transform-origin: top left; /* 📍 Transform from top-left corner */
  transform: rotate(45deg); /* 🔄 Rotate around top-left */

  /* Other values: center (default), top, bottom, left, right */
  /* Precise values: 25% 75%, 100px 50px */
}

/*
🔍 Transform origin explanation:
- Default origin is center (50% 50%)
- Keywords: top, right, bottom, left, center
- Percentage values: X% Y% from top-left
- Pixel values: precise positioning
- Affects all transform functions
*/

/* 🎯 3D TRANSFORMS - Advanced transformations */
.transform-3d {
  /* 🌐 3D CONTEXT SETUP */
  perspective: 1000px; /* 👁️ Viewing distance for 3D */
  transform-style: preserve-3d; /* 🌍 Enable 3D space for children */
}

.transform-3d-child {
  transform: translateZ(100px) /* ➡️ Move towards viewer */ rotateX(30deg)
    /* 🔄 Rotate around X-axis */ rotateY(45deg); /* 🔄 Rotate around Y-axis */

  /* Additional 3D functions:
     translateZ(), translate3d(x, y, z)
     rotateZ(), rotate3d(x, y, z, angle)
     scaleZ(), scale3d(x, y, z)
  */
}

/*
🔍 3D transforms explanation:
- Requires perspective on parent element
- Z-axis points towards/away from viewer
- transform-style: preserve-3d maintains 3D context
- More complex but creates realistic effects
*/

/* 🎭 TRANSFORM ANIMATIONS - Smooth transitions */
.transform-animated {
  transition: transform 0.3s ease-in-out; /* ⚡ Smooth transform changes */
  transform: scale(1); /* 📏 Initial state */
}

.transform-animated:hover {
  transform: scale(1.1) rotate(5deg); /* 🎭 Hover state */
}

/*
🔍 Transform animations explanation:
- Transforms are hardware accelerated (smooth performance)
- Use transitions for smooth state changes
- Combine with pseudo-classes for interactions
- GPU acceleration makes transforms very performant
*/
```

### **⚡ CSS Transitions - Smooth State Changes**

```css
/* 🔄 BASIC TRANSITIONS - Property changes over time */
.transition-basic {
  background-color: #3498db; /* 🎨 Initial background */
  padding: 20px; /* 📦 Initial padding */
  border-radius: 4px; /* 🔄 Initial border radius */

  /* ⚡ TRANSITION SHORTHAND */
  transition: all 0.3s ease-in-out;
  /*
  🔍 Shorthand breakdown:
  - all: transition all animatable properties
  - 0.3s: duration of transition
  - ease-in-out: timing function (starts slow, speeds up, slows down)
  */
}

.transition-basic:hover {
  background-color: #e74c3c; /* 🎨 New background on hover */
  padding: 25px; /* 📦 Increased padding */
  border-radius: 8px; /* 🔄 More rounded corners */
  transform: translateY(-2px); /* ⬆️ Slight lift effect */
}

/*
🔍 Basic transition explanation:
- Automatically animates changes between CSS states
- Works with most numeric CSS properties
- Triggered by state changes (:hover, :focus, class changes)
- Hardware accelerated for smooth performance
*/

/* 🎯 SPECIFIC PROPERTY TRANSITIONS - Precise control */
.transition-specific {
  background-color: #2ecc71; /* 🎨 Green background */
  transform: scale(1); /* 📏 Normal size */
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1); /* 🌫️ Subtle shadow */

  /* 🔧 INDIVIDUAL PROPERTY CONTROL */
  transition-property: background-color, transform, box-shadow;
  transition-duration: 0.2s, 0.3s, 0.4s; /* ⏱️ Different durations */
  transition-timing-function: ease, ease-out, ease-in-out;
  transition-delay: 0s, 0.1s, 0.2s; /* ⏰ Staggered delays */
}

.transition-specific:hover {
  background-color: #27ae60; /* 🎨 Darker green */
  transform: scale(1.05); /* 📈 Slight scale up */
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15); /* 🌫️ Enhanced shadow */
}

/*
🔍 Specific transitions explanation:
- Comma-separated values apply to respective properties
- Different durations create choreographed effects
- Delays create sequential animations
- More control than 'all' keyword
*/

/* 📈 TIMING FUNCTIONS - Animation curves */
.timing-functions-demo {
  width: 100px;
  height: 100px;
  background: #9b59b6;
  transition-duration: 1s; /* ⏱️ Long duration to see effect */
  transition-property: transform;
}

/* 🚀 LINEAR - Constant speed */
.timing-linear {
  transition-timing-function: linear;
  /* Speed remains constant throughout animation */
}

/* 📈 EASE - Default curve */
.timing-ease {
  transition-timing-function: ease;
  /* Starts slow, speeds up, then slows down */
}

/* 🚀 EASE-IN - Accelerating */
.timing-ease-in {
  transition-timing-function: ease-in;
  /* Starts slow, speeds up */
}

/* 🛑 EASE-OUT - Decelerating */
.timing-ease-out {
  transition-timing-function: ease-out;
  /* Starts fast, slows down */
}

/* 🎢 EASE-IN-OUT - Smooth curve */
.timing-ease-in-out {
  transition-timing-function: ease-in-out;
  /* Most natural feeling animation */
}

/* 🎯 CUBIC-BEZIER - Custom curves */
.timing-custom {
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
  /* Creates bounce effect - values can exceed 0-1 range */
}

/*
🔍 Timing functions explanation:
- Control the acceleration curve of transitions
- linear: constant speed (robotic feeling)
- ease: most common, natural feeling
- ease-in: good for exits
- ease-out: good for entrances
- ease-in-out: smooth, professional
- cubic-bezier: unlimited customization
*/

/* ⏰ TRANSITION DELAYS - Staggered effects */
.staggered-container {
  display: flex;
  gap: 10px;
}

.staggered-item {
  width: 50px;
  height: 50px;
  background: #e67e22;
  transition: transform 0.3s ease-out;
}

/* 🎭 STAGGERED HOVER EFFECTS */
.staggered-item:nth-child(1) {
  transition-delay: 0s;
} /* 🥇 First */
.staggered-item:nth-child(2) {
  transition-delay: 0.1s;
} /* 🥈 Second */
.staggered-item:nth-child(3) {
  transition-delay: 0.2s;
} /* 🥉 Third */
.staggered-item:nth-child(4) {
  transition-delay: 0.3s;
} /* 4️⃣ Fourth */

.staggered-container:hover .staggered-item {
  transform: translateY(-20px); /* ⬆️ Lift effect on container hover */
}

/*
🔍 Staggered effects explanation:
- Creates wave-like animation sequences
- nth-child() selector targets specific items
- Container hover triggers all items with delays
- Professional animation technique
*/

/* 🎪 ADVANCED TRANSITION TECHNIQUES */

/* 🌊 SMOOTH STATE MANAGEMENT */
.button-advanced {
  background: linear-gradient(
    45deg,
    #3498db,
    #2980b9
  ); /* 🌈 Gradient background */
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;

  /* 🔧 MULTI-PROPERTY TRANSITIONS */
  transition: transform 0.2s ease-out, box-shadow 0.2s ease-out,
    background 0.3s ease;
}

.button-advanced::before {
  content: "";
  position: absolute;
  top: 0;
  left: -100%; /* ⬅️ Start off-screen */
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.2),
    transparent
  );
  transition: left 0.5s ease-out; /* ➡️ Sweep animation */
}

.button-advanced:hover {
  transform: translateY(-2px) scale(1.02); /* ⬆️📈 Lift and scale */
  box-shadow: 0 8px 25px rgba(52, 152, 219, 0.3); /* 🌫️ Enhanced shadow */
}

.button-advanced:hover::before {
  left: 100%; /* ➡️ Sweep across */
}

.button-advanced:active {
  transform: translateY(0) scale(1); /* ⬇️ Press down effect */
  transition-duration: 0.1s; /* ⚡ Faster active response */
}

/*
🔍 Advanced transitions explanation:
- Multiple transitions for different properties
- Pseudo-elements for additional effects
- State-specific transition durations
- Layered animation effects
- Hardware acceleration optimizations
*/

/* 🎭 TRANSITION BEST PRACTICES */
.transition-optimized {
  /* ✅ PERFORMANCE OPTIMIZATIONS */
  will-change: transform; /* 🚀 GPU acceleration hint */
  transition: transform 0.3s ease-out;

  /* 🎯 TARGET SPECIFIC PROPERTIES */
  /* Avoid: transition: all (impacts performance) */
  /* Prefer: transition: transform, opacity */
}

.transition-optimized:hover {
  transform: translateY(-5px); /* ⬆️ Hardware accelerated */
  /* Avoid animating: width, height, top, left (causes layout) */
  /* Prefer animating: transform, opacity (composited) */
}

/*
🔍 Performance best practices:
- Use transform and opacity for animations
- Avoid animating layout properties (width, height, margin)
- will-change property hints to browser for optimization
- GPU acceleration for smooth 60fps animations
- Test on lower-end devices
*/
```

---

## 🧩 Flexbox Complete Guide {#flexbox-guide}

Flexbox is a one-dimensional layout method that provides efficient arrangement of items in a container, even when their size is unknown or dynamic.

### **📦 Flexbox Container Properties - Parent Element Control**

```css
/* 🏠 FLEX CONTAINER - Parent element setup */
.flex-container {
  display: flex; /* 📦 Enable flexbox layout */
  /* Alternative: display: inline-flex; for inline flex containers */
}

/*
🔍 Display flex explanation:
- Creates a flex formatting context
- Direct children become flex items
- Establishes main axis and cross axis
- Block-level flex container by default
- inline-flex creates inline-level flex container
*/

/* ➡️ FLEX DIRECTION - Main axis direction */
.flex-direction-examples {
  /* 🔄 HORIZONTAL LAYOUTS */
  display: flex;
  flex-direction: row; /* ➡️ Left to right (default) */
  /* flex-direction: row-reverse; */ /* ⬅️ Right to left */

  /* 🔄 VERTICAL LAYOUTS */
  /* flex-direction: column; */ /* ⬇️ Top to bottom */
  /* flex-direction: column-reverse; */ /* ⬆️ Bottom to top */
}

/*
🔍 Flex direction explanation:
- Defines the main axis direction
- row: horizontal main axis (default)
- column: vertical main axis
- reverse versions flip the direction
- Cross axis is perpendicular to main axis
*/

/* 📦 FLEX WRAP - Item wrapping behavior */
.flex-wrap-examples {
  display: flex;
  flex-wrap: nowrap; /* 🚫 No wrapping (default) */
  /* flex-wrap: wrap; */ /* ✅ Wrap to new lines */
  /* flex-wrap: wrap-reverse; */ /* ✅ Wrap in reverse order */

  /* 🔗 SHORTHAND */
  /* flex-flow: row wrap; */ /* direction + wrap combined */
}

/*
🔍 Flex wrap explanation:
- Controls whether items wrap to new lines
- nowrap: all items on one line (may overflow)
- wrap: items wrap to new lines as needed
- wrap-reverse: wrap but in reverse order
- flex-flow combines direction and wrap
*/

/* 📐 JUSTIFY CONTENT - Main axis alignment */
.justify-content-examples {
  display: flex;
  height: 200px; /* 📏 Container height for visibility */
  border: 2px solid #ddd; /* 🖼️ Visual boundary */

  /* 🎯 MAIN AXIS ALIGNMENT OPTIONS */
  justify-content: flex-start; /* ⬅️ Align to start (default) */
  /* justify-content: flex-end; */ /* ➡️ Align to end */
  /* justify-content: center; */ /* 🎯 Center alignment */
  /* justify-content: space-between; */ /* 📏 Space between items */
  /* justify-content: space-around; */ /* 🌌 Space around items */
  /* justify-content: space-evenly; */ /* ⚡ Even space distribution */
}

/*
🔍 Justify content explanation:
- Controls alignment along main axis
- flex-start: items at start of container
- flex-end: items at end of container
- center: items centered in container
- space-between: equal space between items
- space-around: equal space around each item
- space-evenly: equal space everywhere
*/

/* 📐 ALIGN ITEMS - Cross axis alignment */
.align-items-examples {
  display: flex;
  height: 200px; /* 📏 Height needed to see cross-axis alignment */

  /* 🎯 CROSS AXIS ALIGNMENT OPTIONS */
  align-items: stretch; /* 📏 Stretch to container height (default) */
  /* align-items: flex-start; */ /* ⬆️ Align to cross-axis start */
  /* align-items: flex-end; */ /* ⬇️ Align to cross-axis end */
  /* align-items: center; */ /* 🎯 Center on cross-axis */
  /* align-items: baseline; */ /* 📝 Align to text baseline */
}

/*
🔍 Align items explanation:
- Controls alignment along cross axis
- stretch: items fill container height (default)
- flex-start: items at start of cross axis
- flex-end: items at end of cross axis
- center: items centered on cross axis
- baseline: items aligned by text baseline
*/

/* 📦 ALIGN CONTENT - Multiple line alignment */
.align-content-examples {
  display: flex;
  flex-wrap: wrap; /* ✅ Enable wrapping for multiple lines */
  height: 300px; /* 📏 Tall container for multiple lines */

  /* 🎯 MULTI-LINE ALIGNMENT */
  align-content: stretch; /* 📏 Stretch lines (default) */
  /* align-content: flex-start; */ /* ⬆️ Lines at start */
  /* align-content: flex-end; */ /* ⬇️ Lines at end */
  /* align-content: center; */ /* 🎯 Lines centered */
  /* align-content: space-between; */ /* 📏 Space between lines */
  /* align-content: space-around; */ /* 🌌 Space around lines */
  /* align-content: space-evenly; */ /* ⚡ Even space between lines */
}

/*
🔍 Align content explanation:
- Only affects multi-line flex containers
- Controls how wrapped lines are distributed
- Similar to justify-content but for cross axis
- Has no effect with single line of items
*/

/* 🌌 GAP - Spacing between items */
.flex-gap {
  display: flex;
  gap: 20px; /* 🌌 Equal gap between all items */
  /* row-gap: 20px; */ /* 🌌 Gap between rows only */
  /* column-gap: 30px; */ /* 🌌 Gap between columns only */
}

/*
🔍 Gap explanation:
- Modern property for spacing between flex items
- Replaces margin-based spacing techniques
- More predictable than margins
- Works with both row and column directions
*/
```

### **🎯 Flexbox Item Properties - Child Element Control**

```css
/* 🔢 FLEX ORDER - Change visual order */
.flex-items-container {
  display: flex;
}

.flex-item-1 {
  order: 2;
} /* 2️⃣ Appears second */
.flex-item-2 {
  order: 1;
} /* 1️⃣ Appears first */
.flex-item-3 {
  order: 3;
} /* 3️⃣ Appears third */
/* Default order value is 0 */

/*
🔍 Order explanation:
- Changes visual order without affecting HTML structure
- Lower values appear first
- Default order is 0
- Useful for responsive reordering
- Affects visual layout only, not tab order
*/

/* 📈 FLEX GROW - Expansion behavior */
.flex-grow-examples {
  display: flex;
  width: 600px; /* 📏 Fixed container width */
}

.grow-item-1 {
  flex-grow: 1; /* 📈 Takes 1 fraction of extra space */
  background: #e74c3c; /* 🔴 Red background for visibility */
}

.grow-item-2 {
  flex-grow: 2; /* 📈 Takes 2 fractions of extra space */
  background: #3498db; /* 🔵 Blue background */
}

.grow-item-3 {
  flex-grow: 1; /* 📈 Takes 1 fraction of extra space */
  background: #2ecc71; /* 🟢 Green background */
}

/*
🔍 Flex grow explanation:
- Defines how much an item should grow
- Value represents proportion of available space
- 0 = don't grow (default)
- Items with higher values get more space
- Calculated after fixed widths are allocated
*/

/* 📉 FLEX SHRINK - Contraction behavior */
.flex-shrink-examples {
  display: flex;
  width: 200px; /* 📏 Small container forces shrinking */
}

.shrink-item {
  width: 100px; /* 📏 Initial width */
  flex-shrink: 1; /* 📉 Default shrink behavior */
}

.no-shrink-item {
  width: 100px; /* 📏 Initial width */
  flex-shrink: 0; /* 🚫 Never shrink */
  background: #f39c12; /* 🟡 Orange background */
}

/*
🔍 Flex shrink explanation:
- Defines how much an item should shrink
- 1 = normal shrinking (default)
- 0 = never shrink
- Higher values shrink more
- Prevents overflow in tight spaces
*/

/* 📏 FLEX BASIS - Initial size */
.flex-basis-examples {
  display: flex;
}

.basis-item-1 {
  flex-basis: 200px; /* 📏 Initial size before grow/shrink */
  background: #9b59b6; /* 🟣 Purple background */
}

.basis-item-2 {
  flex-basis: auto; /* 📏 Based on content size (default) */
  background: #1abc9c; /* 🟢 Teal background */
}

.basis-item-3 {
  flex-basis: 0; /* 📏 Zero basis, only grow/shrink */
  flex-grow: 1; /* 📈 Will grow to fill space */
  background: #e67e22; /* 🟠 Orange background */
}

/*
🔍 Flex basis explanation:
- Sets initial size before grow/shrink
- auto: uses content size or width/height
- 0: ignores content size, uses only grow/shrink
- Length values: specific initial size
- Percentage values: relative to container
*/

/* 🔗 FLEX SHORTHAND - Combined properties */
.flex-shorthand-examples {
  display: flex;
}

.flex-equal {
  flex: 1; /* 🔗 Equal distribution */
  /* Equivalent to: flex: 1 1 0; */
  /* grow: 1, shrink: 1, basis: 0 */
}

.flex-auto {
  flex: auto; /* 🔗 Content-based sizing */
  /* Equivalent to: flex: 1 1 auto; */
  /* grow: 1, shrink: 1, basis: auto */
}

.flex-none {
  flex: none; /* 🔗 Fixed sizing */
  /* Equivalent to: flex: 0 0 auto; */
  /* grow: 0, shrink: 0, basis: auto */
}

.flex-custom {
  flex: 2 1 300px; /* 🔗 Custom values */
  /* grow: 2, shrink: 1, basis: 300px */
}

/*
🔍 Flex shorthand explanation:
- Combines grow, shrink, and basis
- flex: 1 = equal distribution
- flex: auto = content-based with flexibility
- flex: none = fixed size, no flexibility
- Always prefer shorthand over individual properties
*/

/* 🎯 ALIGN SELF - Individual cross-axis alignment */
.align-self-container {
  display: flex;
  align-items: flex-start; /* 📐 Default alignment for all items */
  height: 200px; /* 📏 Container height */
}

.align-self-center {
  align-self: center; /* 🎯 Override container alignment */
  background: #e74c3c; /* 🔴 Red background */
}

.align-self-end {
  align-self: flex-end; /* ⬇️ Align to bottom */
  background: #3498db; /* 🔵 Blue background */
}

.align-self-stretch {
  align-self: stretch; /* 📏 Stretch full height */
  background: #2ecc71; /* 🟢 Green background */
}

/*
🔍 Align self explanation:
- Overrides align-items for individual items
- Same values as align-items
- Allows fine-tuned control per item
- Useful for exceptions in layouts
*/
```

### **🏗️ Real-World Flexbox Layouts**

```css
/* 🎯 NAVIGATION BAR - Horizontal layout with space distribution */
.navbar {
  display: flex; /* 📦 Flex container */
  justify-content: space-between; /* 📏 Logo left, menu right */
  align-items: center; /* 📐 Vertical centering */
  padding: 1rem 2rem; /* 📦 Internal spacing */
  background: #2c3e50; /* 🎨 Dark background */
  color: white; /* 🔤 White text */
}

.navbar__logo {
  font-size: 1.5rem; /* 📏 Larger logo text */
  font-weight: bold; /* 📝 Bold logo */
}

.navbar__menu {
  display: flex; /* 📦 Horizontal menu items */
  gap: 2rem; /* 🌌 Space between menu items */
  list-style: none; /* 🚫 Remove bullet points */
  margin: 0; /* 🚫 Remove default margin */
  padding: 0; /* 🚫 Remove default padding */
}

.navbar__item {
  padding: 0.5rem 1rem; /* 📦 Clickable area */
  border-radius: 4px; /* 🔄 Rounded corners */
  transition: background 0.3s ease; /* ⚡ Smooth hover effect */
}

.navbar__item:hover {
  background: rgba(255, 255, 255, 0.1); /* 🎭 Subtle hover effect */
}

/*
🔍 Navigation explanation:
- space-between: pushes logo left, menu right
- align-items: center vertically aligns all content
- Gap property creates consistent spacing
- Flexible design adapts to content changes
*/

/* 🃏 CARD LAYOUT - Flexible card grid */
.card-container {
  display: flex; /* 📦 Flex container */
  flex-wrap: wrap; /* ✅ Allow wrapping */
  gap: 2rem; /* 🌌 Consistent spacing */
  padding: 2rem; /* 📦 Container padding */
}

.card {
  flex: 1 1 300px; /* 📏 Flexible cards, min 300px */
  /* grow: 1, shrink: 1, basis: 300px */
  display: flex; /* 📦 Nested flex for card content */
  flex-direction: column; /* 📐 Vertical card layout */
  background: white; /* 🎨 White background */
  border-radius: 8px; /* 🔄 Rounded corners */
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1); /* 🌫️ Subtle shadow */
  overflow: hidden; /* 🚫 Contain content */
  transition: transform 0.3s ease; /* ⚡ Smooth hover animation */
}

.card:hover {
  transform: translateY(-4px); /* ⬆️ Lift effect on hover */
}

.card__image {
  width: 100%; /* 📏 Full width image */
  height: 200px; /* 📏 Fixed height */
  object-fit: cover; /* 🖼️ Crop to fit */
  object-position: center; /* 🎯 Center crop */
}

.card__content {
  padding: 1.5rem; /* 📦 Content padding */
  flex: 1; /* 📈 Expand to fill available space */
  display: flex; /* 📦 Nested flex for content */
  flex-direction: column; /* 📐 Vertical content layout */
}

.card__title {
  font-size: 1.25rem; /* 📏 Larger title */
  font-weight: 600; /* 📝 Semi-bold */
  margin-bottom: 0.75rem; /* 🌌 Bottom spacing */
  color: #2c3e50; /* 🎨 Dark text color */
}

.card__description {
  color: #7f8c8d; /* 🎨 Muted text color */
  line-height: 1.6; /* 📏 Readable line height */
  flex: 1; /* 📈 Take available space */
  margin-bottom: 1rem; /* 🌌 Bottom spacing */
}

.card__footer {
  margin-top: auto; /* ⬆️ Push to bottom */
  display: flex; /* 📦 Horizontal footer layout */
  justify-content: space-between; /* 📏 Space between elements */
  align-items: center; /* 📐 Vertical alignment */
}

.card__price {
  font-size: 1.125rem; /* 📏 Larger price text */
  font-weight: 700; /* 📝 Bold price */
  color: #e74c3c; /* 🔴 Attention-grabbing red */
}

.card__button {
  background: #3498db; /* 🔵 Blue button */
  color: white; /* 🔤 White text */
  border: none; /* 🚫 No border */
  padding: 0.5rem 1rem; /* 📦 Button padding */
  border-radius: 4px; /* 🔄 Rounded button */
  cursor: pointer; /* 👆 Pointer cursor */
  transition: background 0.3s ease; /* ⚡ Smooth color transition */
}

.card__button:hover {
  background: #2980b9; /* 🔵 Darker blue on hover */
}

/*
🔍 Card layout explanation:
- flex: 1 1 300px creates responsive card sizing
- Cards wrap when container is too narrow
- Nested flex in cards enables flexible content layout
- margin-top: auto pushes footer to bottom
- Object-fit ensures images maintain aspect ratio
*/

/* 📱 RESPONSIVE SIDEBAR LAYOUT */
.layout {
  display: flex; /* 📦 Main layout container */
  min-height: 100vh; /* 📏 Full viewport height */
}

.sidebar {
  flex: 0 0 250px; /* 📏 Fixed sidebar width */
  /* grow: 0, shrink: 0, basis: 250px */
  background: #34495e; /* 🎨 Dark sidebar */
  color: white; /* 🔤 White text */
  padding: 2rem; /* 📦 Sidebar padding */
  transition: transform 0.3s ease; /* ⚡ Smooth slide animation */
}

.main-content {
  flex: 1; /* 📈 Take remaining space */
  padding: 2rem; /* 📦 Content padding */
  background: #ecf0f1; /* 🎨 Light background */
}

/* 📱 MOBILE RESPONSIVE */
@media (max-width: 768px) {
  .layout {
    flex-direction: column; /* 📐 Stack vertically on mobile */
  }

  .sidebar {
    flex: none; /* 🚫 Remove flex behavior */
    width: 100%; /* 📏 Full width on mobile */
    transform: translateX(-100%); /* ⬅️ Hide sidebar by default */
    position: fixed; /* 📍 Fixed positioning */
    height: 100vh; /* 📏 Full height */
    z-index: 1000; /* 🌊 Above other content */
  }

  .sidebar.open {
    transform: translateX(0); /* ➡️ Show sidebar when open */
  }

  .main-content {
    width: 100%; /* 📏 Full width content */
    padding: 1rem; /* 📦 Reduced padding on mobile */
  }
}

/*
🔍 Responsive sidebar explanation:
- Fixed sidebar width on desktop with flex: 0 0 250px
- Main content takes remaining space with flex: 1
- Mobile: sidebar becomes overlay with transform
- Smooth transitions for better user experience
*/
```

---

## 🔲 CSS Grid Advanced Layouts {#css-grid-guide}

CSS Grid is a two-dimensional layout system that provides precise control over both rows and columns simultaneously.

### **🏗️ Grid Container Properties - Parent Element Control**

```css
/* 📊 BASIC GRID SETUP */
.grid-container {
  display: grid; /* 🔲 Enable grid layout */
  /* Alternative: display: inline-grid; for inline grid containers */
}

/*
🔍 Display grid explanation:
- Creates a grid formatting context
- Direct children become grid items
- Establishes implicit grid with auto-sized rows/columns
- Block-level grid container by default
*/

/* 📏 GRID TEMPLATE COLUMNS - Column structure */
.grid-columns {
  display: grid;

  /* 🔢 FIXED COLUMN WIDTHS */
  grid-template-columns: 200px 300px 100px; /* 📏 3 columns with specific widths */

  /* 📈 FRACTIONAL UNITS (fr) */
  /* grid-template-columns: 1fr 2fr 1fr; */ /* 📊 Proportional columns (1:2:1 ratio) */

  /* 🔄 REPEAT FUNCTION */
  /* grid-template-columns: repeat(4, 1fr); */ /* 📊 4 equal columns */
  /* grid-template-columns: repeat(3, minmax(200px, 1fr)); */ /* 📊 3 responsive columns */

  /* 🎯 MIXED UNITS */
  /* grid-template-columns: 250px 1fr auto; */ /* 📊 Fixed, flexible, content-based */
}

/*
🔍 Grid template columns explanation:
- Defines column sizes and count
- fr unit represents fraction of available space
- repeat() reduces repetition
- minmax() creates flexible boundaries
- auto sizes based on content
*/

/* 📏 GRID TEMPLATE ROWS - Row structure */
.grid-rows {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 📊 3 equal columns */

  /* 🔢 FIXED ROW HEIGHTS */
  grid-template-rows: 100px 200px auto; /* 📏 3 rows with specific heights */

  /* 📈 FRACTIONAL ROWS */
  /* grid-template-rows: 1fr 2fr 1fr; */ /* 📊 Proportional rows */

  /* 🔄 REPEAT WITH ROWS */
  /* grid-template-rows: repeat(4, 150px); */ /* 📏 4 rows, 150px each */
}

/*
🔍 Grid template rows explanation:
- Similar to columns but for row sizing
- auto adjusts to content height
- fr units distribute available vertical space
- Explicit rows vs implicit rows
*/

/* 🌌 GRID GAP - Spacing between grid items */
.grid-gap {
  display: grid;
  grid-template-columns: repeat(3, 1fr);

  gap: 20px; /* 🌌 Equal gap for rows and columns */
  /* row-gap: 15px; */ /* 🌌 Gap between rows only */
  /* column-gap: 25px; */ /* 🌌 Gap between columns only */

  /* 🔗 LEGACY SYNTAX */
  /* grid-gap: 20px; */ /* 🌌 Older browsers (still works) */
  /* grid-row-gap: 15px; */ /* 🌌 Legacy row gap */
  /* grid-column-gap: 25px; */ /* 🌌 Legacy column gap */
}

/*
🔍 Grid gap explanation:
- Creates gutters between grid items
- Doesn't add space around grid edges
- More predictable than margin-based spacing
- gap is modern syntax, grid-gap for older browsers
*/

/* 📍 GRID AREAS - Named grid regions */
.grid-areas {
  display: grid;
  grid-template-columns: 1fr 3fr 1fr; /* 📊 3 columns */
  grid-template-rows: auto 1fr auto; /* 📏 3 rows */
  grid-template-areas:
    "header  header  header" /* 📋 Header spans full width */
    "sidebar content ads" /* 📊 Main content area */
    "footer  footer  footer"; /* 📋 Footer spans full width */
  gap: 1rem; /* 🌌 Consistent spacing */
  min-height: 100vh; /* 📏 Full viewport height */
}

/* 🎯 ASSIGNING AREAS TO GRID ITEMS */
.header {
  grid-area: header;
} /* 📋 Header area */
.sidebar {
  grid-area: sidebar;
} /* 🗂️ Sidebar area */
.content {
  grid-area: content;
} /* 📄 Main content area */
.ads {
  grid-area: ads;
} /* 📢 Advertisement area */
.footer {
  grid-area: footer;
} /* 📋 Footer area */

/*
🔍 Grid areas explanation:
- Creates named regions for easy item placement
- Visual representation of layout structure
- Each area name must form rectangular region
- Dot (.) represents empty grid cells
- More intuitive than line-based positioning
*/

/* 📐 GRID ALIGNMENT - Container-level alignment */
.grid-alignment {
  display: grid;
  grid-template-columns: repeat(3, 200px); /* 📏 Fixed column widths */
  grid-template-rows: repeat(3, 100px); /* 📏 Fixed row heights */
  width: 800px; /* 📏 Container wider than grid */
  height: 400px; /* 📏 Container taller than grid */

  /* 🎯 HORIZONTAL ALIGNMENT (justify-content) */
  justify-content: center; /* 🎯 Center grid horizontally */
  /* justify-content: start; */ /* ⬅️ Align to start (default) */
  /* justify-content: end; */ /* ➡️ Align to end */
  /* justify-content: space-between; */ /* 📏 Space between columns */
  /* justify-content: space-around; */ /* 🌌 Space around columns */
  /* justify-content: space-evenly; */ /* ⚡ Even space distribution */

  /* 🎯 VERTICAL ALIGNMENT (align-content) */
  align-content: center; /* 🎯 Center grid vertically */
  /* align-content: start; */ /* ⬆️ Align to top (default) */
  /* align-content: end; */ /* ⬇️ Align to bottom */
  /* align-content: space-between; */ /* 📏 Space between rows */
  /* align-content: space-around; */ /* 🌌 Space around rows */
  /* align-content: space-evenly; */ /* ⚡ Even space distribution */
}

/*
🔍 Grid alignment explanation:
- justify-content: aligns entire grid horizontally
- align-content: aligns entire grid vertically
- Only works when grid is smaller than container
- Similar to flexbox alignment but for 2D grid
*/
```

This completes Part 4 covering:

1. **Complete Flexbox System** - Container and item properties with real-world examples
2. **Advanced Flexbox Layouts** - Navigation, cards, responsive sidebars
3. **CSS Grid Fundamentals** - Grid container setup, columns, rows, gaps, and areas
4. **Grid Alignment Systems** - Container-level positioning and spacing

Would you like me to continue with Part 5 covering advanced Grid techniques, responsive patterns, and layout combinations?
