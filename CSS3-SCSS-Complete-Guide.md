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

### **💻 Box Model Implementation with HTML Examples**

```html
<!-- 🎯 HTML STRUCTURE for Box Model Examples -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CSS Box Model Examples</title>
    <!-- 📄 Link to CSS file -->
    <link rel="stylesheet" href="box-model-styles.css" />
  </head>
  <body>
    <!-- 🔍 BOX MODEL COMPARISON CONTAINER -->
    <div class="box-model-demo">
      <!-- 📋 Section heading -->
      <h2>Box Model Comparison</h2>

      <!-- 📦 Standard box model example -->
      <div class="demo-container">
        <h3>Standard Box Model (content-box)</h3>
        <div class="standard-box">
          <!-- 📝 Content inside the box -->
          <p>This box uses the standard box model.</p>
          <p>Width: 200px, Padding: 20px, Border: 5px, Margin: 10px</p>
          <p>Total width: 270px</p>
        </div>
      </div>

      <!-- 🎯 Border-box model example -->
      <div class="demo-container">
        <h3>Border Box Model (border-box)</h3>
        <div class="border-box">
          <!-- 📝 Content inside the box -->
          <p>This box uses border-box sizing.</p>
          <p>Width: 200px (includes padding & border)</p>
          <p>Content width: 150px</p>
        </div>
      </div>

      <!-- 🎨 Product card practical example -->
      <div class="demo-container">
        <h3>Practical Example: Product Card</h3>
        <article class="product-card">
          <!-- 🖼️ Product image -->
          <img
            src="https://via.placeholder.com/260x160/3498db/white?text=Product"
            alt="Product Image"
            class="product-image"
          />

          <!-- 📝 Product information -->
          <div class="product-info">
            <h4 class="product-title">Premium Headphones</h4>
            <p class="product-description">
              High-quality wireless headphones with noise cancellation and
              30-hour battery life.
            </p>

            <!-- 💰 Price and action section -->
            <div class="product-footer">
              <span class="product-price">$299.99</span>
              <button class="add-to-cart-btn" type="button">Add to Cart</button>
            </div>
          </div>
        </article>
      </div>
    </div>
  </body>
</html>
```

```css
/* 🎯 STANDARD BOX MODEL - Default behavior */
.standard-box {
  width: 200px;                    /* 📏 Content width only */
  height: 100px;                   /* 📏 Content height only */
  padding: 20px;                   /* 📦 Internal spacing */
  border: 5px solid #333;          /* 🖼️ Border thickness */
  margin: 10px;                    /* 🌌 External spacing */
  background-color: #e74c3c;       /* 🔴 Red background for visibility */
  color: white;                    /* 🔤 White text for contrast */

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
  box-sizing: border-box;          /* 🎛️ Include padding & border in width/height */
  width: 200px;                    /* 📏 Total width including padding & border */
  height: 100px;                   /* 📏 Total height including padding & border */
  padding: 20px;                   /* 📦 Internal spacing (included in width) */
  border: 5px solid #333;          /* 🖼️ Border thickness (included in width) */
  margin: 10px;                    /* 🌌 External spacing (NOT included) */
  background-color: #3498db;       /* 🔵 Blue background for distinction */
  color: white;                    /* 🔤 White text for contrast */

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
  box-sizing: border-box;          /* 📦 Apply to all elements */
}
/*
🔍 Universal border-box explanation:
- Applies to all elements (*) and pseudo-elements (::before, ::after)
- Makes all elements use border-box sizing by default
- Prevents unexpected layout issues
- Industry standard best practice
*/

/* 🎨 PRACTICAL BOX MODEL EXAMPLE - Card Component */
.product-card {
  /* 📐 DIMENSIONS */
  width: 300px;                    /* 📏 Card width */
  height: 400px;                   /* 📏 Card height */

  /* 📦 SPACING */
  padding: 0;                      /* 📦 No padding on card container */
  margin: 16px;                    /* 🌌 Space between cards */

  /* 🖼️ VISUAL STYLING */
  background: white;               /* ⚪ White background */
  border-radius: 12px;             /* 🔄 Rounded corners */
  box-shadow: 0 4px 12px rgba(0,0,0,0.15); /* 🌫️ Subtle shadow */
  overflow: hidden;                /* 🚫 Hide overflowing content */

  /* 📦 LAYOUT */
  display: flex;                   /* 📦 Flexbox for internal layout */
  flex-direction: column;          /* 📐 Vertical stacking */
}

.product-image {
  /* 📐 IMAGE DIMENSIONS */
  width: 100%;                     /* 📏 Full card width */
  height: 160px;                   /* 📏 Fixed image height */
  object-fit: cover;               /* 🖼️ Cover entire area */
  object-position: center;         /* 🎯 Center the image */
  display: block;                  /* 📦 Remove inline spacing */
}
/*
🔍 Image styling explanation:
- width: 100% makes image span full card width
- height: 160px creates consistent image heights
- object-fit: cover crops image to fit without distortion
- object-position: center focuses on center of image
*/

.product-info {
  /* 📦 CONTENT AREA SPACING */
  padding: 20px;                   /* 📦 Internal content spacing */
  flex: 1;                         /* 📈 Expand to fill available space */
  display: flex;                   /* 📦 Flexbox for content layout */
  flex-direction: column;          /* 📐 Vertical content stacking */
}

.product-title {
  /* 📝 TITLE STYLING */
  font-size: 1.25rem;              /* 📏 20px title size */
  font-weight: 600;                /* 📝 Semi-bold weight */
  color: #2c3e50;                  /* 🎨 Dark blue-gray */
  margin: 0 0 12px 0;              /* 🌌 Bottom margin only */
  line-height: 1.3;                /* 📏 Tight line height for headings */
}

.product-description {
  /* 📝 DESCRIPTION STYLING */
  font-size: 0.875rem;             /* 📏 14px description size */
  color: #7f8c8d;                  /* 🎨 Medium gray text */
  line-height: 1.5;                /* 📏 Readable line height */
  margin: 0 0 20px 0;              /* 🌌 Bottom margin for spacing */
  flex: 1;                         /* 📈 Take available vertical space */
}

.product-footer {
  /* 📦 FOOTER LAYOUT */
  display: flex;                   /* 📦 Horizontal footer layout */
  justify-content: space-between;  /* 📏 Space between price and button */
  align-items: center;             /* 📐 Vertical alignment */
  margin-top: auto;                /* ⬆️ Push footer to bottom */
}

.product-price {
  /* 💰 PRICE STYLING */
  font-size: 1.5rem;               /* 📏 24px price size */
  font-weight: 700;                /* 📝 Bold price */
  color: #e74c3c;                  /* 🔴 Red price color for attention */
}

.add-to-cart-btn {
  /* 🔲 BUTTON DIMENSIONS */
  padding: 10px 20px;              /* 📦 Button padding */
  border: none;                    /* 🚫 Remove default border */
  border-radius: 6px;              /* 🔄 Rounded button corners */

  /* 🎨 BUTTON STYLING */
  background: #3498db;             /* 🔵 Blue button background */
  color: white;                    /* 🔤 White button text */
  font-size: 0.875rem;             /* 📏 14px button text */
  font-weight: 500;                /* 📝 Medium button weight */
  cursor: pointer;                 /* 👆 Pointer cursor on hover */

  /* ⚡ INTERACTION */
  transition: background 0.3s ease; /* 🌊 Smooth background transition */
}

.add-to-cart-btn:hover {
  background: #2980b9;             /* 🔵 Darker blue on hover */
}

/* 📋 DEMO CONTAINER STYLING */
.box-model-demo {
  max-width: 1200px;               /* 📏 Maximum container width */
  margin: 0 auto;                  /* 🎯 Center container */
  padding: 40px 20px;              /* 📦 Container padding */
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; /* 🔤 Font stack */
}

.demo-container {
  margin-bottom: 40px;             /* 🌌 Space between demo sections */
  padding: 20px;                   /* 📦 Section padding */
  border: 1px solid #ddd;          /* 🖼️ Light border */
  border-radius: 8px;              /* 🔄 Rounded section corners */
  background: #f8f9fa;             /* 🎨 Light background */
}

.demo-container h3 {
  margin-top: 0;                   /* 🚫 Remove top margin */
  color: #2c3e50;                  /* 🎨 Dark heading color */
  border-bottom: 2px solid #3498db; /* 🖼️ Blue underline */
  padding-bottom: 8px;             /* 📦 Underline spacing */
}
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

### **🎭 CSS3 Selectors & Pseudo-classes with HTML Examples**

```html
<!-- 🎯 HTML STRUCTURE for CSS3 Advanced Features -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CSS3 Advanced Features Demo</title>
    <link rel="stylesheet" href="css3-features.css" />
  </head>
  <body>
    <!-- 📧 CONTACT FORM - Attribute Selectors Demo -->
    <section class="contact-section">
      <h2>Contact Form - Attribute Selectors</h2>
      <form class="contact-form">
        <!-- 📧 Email input with attribute selector styling -->
        <div class="form-group">
          <label for="email">Email Address</label>
          <input
            type="email"
            id="email"
            name="email"
            placeholder="Enter your email"
            required
          />
          <!-- 📝 type="email" triggers [type="email"] selector -->
        </div>

        <!-- 📱 Phone input with different styling -->
        <div class="form-group">
          <label for="phone">Phone Number</label>
          <input
            type="tel"
            id="phone"
            name="phone"
            placeholder="Enter your phone"
          />
          <!-- 📞 type="tel" triggers [type="tel"] selector -->
        </div>

        <!-- 📝 Text area with custom styling -->
        <div class="form-group">
          <label for="message">Message</label>
          <textarea
            id="message"
            name="message"
            placeholder="Enter your message"
            required
          ></textarea>
          <!-- 📋 required attribute triggers [required] selector -->
        </div>

        <!-- 🔲 Submit button -->
        <button type="submit" class="submit-btn">Send Message</button>
      </form>
    </section>

    <!-- 🖼️ GALLERY - Structural Pseudo-selectors Demo -->
    <section class="gallery-section">
      <h2>Photo Gallery - Structural Selectors</h2>
      <div class="gallery-grid">
        <!-- 🖼️ Gallery items for nth-child demonstrations -->
        <div class="gallery-item">
          <img
            src="https://via.placeholder.com/300x200/e74c3c/white?text=Photo+1"
            alt="Gallery Photo 1"
          />
          <p>Photo 1 - First Child</p>
        </div>

        <div class="gallery-item">
          <img
            src="https://via.placeholder.com/300x200/3498db/white?text=Photo+2"
            alt="Gallery Photo 2"
          />
          <p>Photo 2 - Second Child</p>
        </div>

        <div class="gallery-item">
          <img
            src="https://via.placeholder.com/300x200/2ecc71/white?text=Photo+3"
            alt="Gallery Photo 3"
          />
          <p>Photo 3 - Third Child (3n)</p>
        </div>

        <div class="gallery-item">
          <img
            src="https://via.placeholder.com/300x200/f39c12/white?text=Photo+4"
            alt="Gallery Photo 4"
          />
          <p>Photo 4 - Fourth Child</p>
        </div>

        <div class="gallery-item">
          <img
            src="https://via.placeholder.com/300x200/9b59b6/white?text=Photo+5"
            alt="Gallery Photo 5"
          />
          <p>Photo 5 - Fifth Child (odd)</p>
        </div>

        <div class="gallery-item">
          <img
            src="https://via.placeholder.com/300x200/1abc9c/white?text=Photo+6"
            alt="Gallery Photo 6"
          />
          <p>Photo 6 - Sixth Child (3n)</p>
        </div>

        <div class="gallery-item">
          <img
            src="https://via.placeholder.com/300x200/e67e22/white?text=Photo+7"
            alt="Gallery Photo 7"
          />
          <p>Photo 7 - Last Child</p>
        </div>
      </div>
    </section>

    <!-- 🎨 PSEUDO-ELEMENTS - Before/After Demo -->
    <section class="pseudo-elements-section">
      <h2>Pseudo-elements Demo</h2>

      <!-- 💬 Quote with decorative pseudo-elements -->
      <blockquote class="inspirational-quote">
        <p>The best way to predict the future is to create it.</p>
        <cite>Peter Drucker</cite>
      </blockquote>

      <!-- 🏷️ Badge elements with counters -->
      <div class="notification-list">
        <div class="notification-item">New message received</div>
        <div class="notification-item">File uploaded successfully</div>
        <div class="notification-item">System update available</div>
      </div>

      <!-- 🎯 Call-to-action with decorative elements -->
      <div class="cta-banner">
        <h3>Join Our Newsletter</h3>
        <p>Get the latest updates and exclusive offers</p>
        <button class="cta-button">Subscribe Now</button>
      </div>
    </section>
  </body>
</html>
```

```css
/* 🎯 ATTRIBUTE SELECTORS - Target elements based on attributes */

/* 📧 EMAIL INPUT STYLING */
input[type="email"] {
  border: 2px solid #4caf50; /* 💚 Green border for email inputs */
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='%234caf50' viewBox='0 0 24 24'%3E%3Cpath d='M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.89 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z'/%3E%3C/svg%3E");
  background-repeat: no-repeat; /* 🚫 Don't repeat icon */
  background-position: right 10px center; /* 📍 Position icon right */
  background-size: 20px 20px; /* 📏 Icon size */
  padding-right: 40px; /* 📦 Space for icon */
  border-radius: 6px; /* � Rounded corners */
}

/* 📱 TELEPHONE INPUT STYLING */
input[type="tel"] {
  border: 2px solid #2196f3; /* 🔵 Blue border for phone inputs */
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='%232196f3' viewBox='0 0 24 24'%3E%3Cpath d='M6.62 10.79c1.44 2.83 3.76 5.14 6.59 6.59l2.2-2.2c.27-.27.67-.36 1.02-.24 1.12.37 2.33.57 3.57.57.55 0 1 .45 1 1V20c0 .55-.45 1-1 1-9.39 0-17-7.61-17-17 0-.55.45-1 1-1h3.5c.55 0 1 .45 1 1 0 1.25.2 2.45.57 3.57.11.35.03.74-.25 1.02l-2.2 2.2z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 10px center;
  background-size: 20px 20px;
  padding-right: 40px;
  border-radius: 6px;
}

/* 📋 REQUIRED FIELD INDICATOR */
[required] {
  position: relative; /* 📍 Position context for pseudo-element */
}

[required]::after {
  content: "*"; /* 📝 Add required asterisk */
  color: #e74c3c; /* 🔴 Red asterisk */
  font-weight: bold; /* 📝 Bold asterisk */
  margin-left: 4px; /* 🌌 Small spacing */
}

/* 🔍 Attribute selector explanation:
   [type="email"]: Targets elements with type="email"
   [required]: Targets elements with required attribute
   ::after: Creates virtual element after content
   SVG data URI: Inline SVG icon for better performance
*/

/* 🎯 STRUCTURAL PSEUDO-SELECTORS - Dynamic element targeting */

/* 🎨 ALTERNATE ROW COLORS */
.gallery-item:nth-child(odd) {
  background-color: #f8f9fa; /* 🎨 Light gray for odd items */
  transform: translateY(-4px); /* ⬆️ Slight lift effect */
}

.gallery-item:nth-child(even) {
  background-color: #ffffff; /* ⚪ White for even items */
}

/* 🎯 EVERY THIRD ITEM STYLING */
.gallery-item:nth-child(3n) {
  border: 3px solid #3498db; /* 🔵 Blue border every 3rd item */
  box-shadow: 0 8px 20px rgba(52, 152, 219, 0.2); /* 🌫️ Blue shadow */
}

/* 🔄 FIRST AND LAST CHILD STYLING */
.gallery-item:first-child {
  border-top-left-radius: 16px; /* 🔄 Round first item corners */
  border-bottom-left-radius: 16px;
  position: relative;
}

.gallery-item:first-child::before {
  content: "FEATURED"; /* 📋 Add "FEATURED" label */
  position: absolute; /* 📍 Position absolutely */
  top: 10px; /* 📐 From top */
  left: 10px; /* 📐 From left */
  background: #e74c3c; /* 🔴 Red badge background */
  color: white; /* ⚪ White text */
  padding: 4px 8px; /* 📦 Badge padding */
  font-size: 0.75rem; /* 📏 Small font size */
  font-weight: bold; /* 📝 Bold text */
  border-radius: 4px; /* 🔄 Rounded badge */
  z-index: 10; /* 🌊 Above other content */
}

.gallery-item:last-child {
  border-top-right-radius: 16px; /* 🔄 Round last item corners */
  border-bottom-right-radius: 16px;
  position: relative;
}

.gallery-item:last-child::after {
  content: "NEW"; /* � Add "NEW" label */
  position: absolute;
  top: 10px;
  right: 10px;
  background: #2ecc71; /* 🟢 Green badge background */
  color: white;
  padding: 4px 8px;
  font-size: 0.75rem;
  font-weight: bold;
  border-radius: 4px;
  z-index: 10;
}

/* 🔍 Structural selector theory:
   :nth-child(odd) - Selects 1st, 3rd, 5th... elements (zebra striping)
   :nth-child(even) - Selects 2nd, 4th, 6th... elements
   :nth-child(3n) - Selects every 3rd element (3, 6, 9, 12...)
   :first-child - Selects the first child element
   :last-child - Selects the last child element
   Useful for dynamic styling without classes
*/

/* ⚡ PSEUDO-ELEMENTS - Create virtual elements */

/* 💬 INSPIRATIONAL QUOTE STYLING */
.inspirational-quote {
  position: relative; /* 📍 Position context */
  padding: 30px 40px; /* 📦 Quote padding */
  margin: 40px 0; /* 🌌 Vertical spacing */
  background: linear-gradient(
    135deg,
    #667eea 0%,
    #764ba2 100%
  ); /* 🌈 Gradient background */
  color: white; /* ⚪ White text */
  border-radius: 12px; /* 🔄 Rounded corners */
  font-style: italic; /* 📝 Italic text */
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2); /* 🌫️ Deep shadow */
}

.inspirational-quote::before {
  content: "" ";                   /* 📝 Opening quote mark */
  position: absolute;             /* 📍 Absolute positioning */
  top: -10px;                     /* 📐 Above the quote */
  left: 20px;                     /* 📐 From left edge */
  font-size: 4rem;                /* 📏 Large quote mark */
  color: rgba(255, 255, 255, 0.3); /* ⚪ Semi-transparent white */
  font-family: Georgia, serif;    /* 📝 Serif font for quotes */
  line-height: 1;                 /* 📏 Tight line height */
}

.inspirational-quote::after {
  content: " ""; /* 📝 Closing quote mark */
  position: absolute;
  bottom: -30px; /* 📐 Below the quote */
  right: 20px; /* 📐 From right edge */
  font-size: 4rem;
  color: rgba(255, 255, 255, 0.3);
  font-family: Georgia, serif;
  line-height: 1;
}

/* 🏷️ NOTIFICATION COUNTER */
.notification-list {
  counter-reset: notification-counter; /* 🔢 Reset counter */
  padding: 20px;
}

.notification-item {
  counter-increment: notification-counter; /* 🔢 Increment counter */
  position: relative;
  padding: 15px 15px 15px 50px; /* 📦 Padding with space for counter */
  margin-bottom: 10px; /* 🌌 Space between items */
  background: #f8f9fa; /* 🎨 Light background */
  border-left: 4px solid #3498db; /* 🔵 Blue left border */
  border-radius: 6px;
}

.notification-item::before {
  content: counter(notification-counter); /* 🔢 Display counter */
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%); /* 📐 Center vertically */
  width: 25px;
  height: 25px;
  background: #3498db; /* 🔵 Blue counter background */
  color: white; /* ⚪ White number */
  border-radius: 50%; /* ⭕ Circular counter */
  display: flex; /* 📦 Flex for centering */
  align-items: center; /* 📐 Center content */
  justify-content: center; /* 📐 Center content */
  font-weight: bold; /* 📝 Bold number */
  font-size: 0.75rem; /* 📏 Small font */
}

/* 🎯 CTA BANNER WITH DECORATIVE ELEMENTS */
.cta-banner {
  position: relative;
  background: linear-gradient(
    45deg,
    #ff6b6b,
    #ffa500
  ); /* 🌈 Orange-red gradient */
  color: white;
  padding: 40px;
  text-align: center;
  border-radius: 16px;
  overflow: hidden; /* 🚫 Hide overflowing decorations */
}

.cta-banner::before {
  content: "";
  position: absolute;
  top: -50%;
  right: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(
    circle,
    rgba(255, 255, 255, 0.1) 1px,
    transparent 1px
  ); /* ⚪ Dot pattern */
  background-size: 20px 20px; /* 📏 Dot spacing */
  animation: float 6s ease-in-out infinite; /* 🔄 Floating animation */
}

.cta-banner::after {
  content: "✨"; /* ✨ Sparkle decoration */
  position: absolute;
  top: 20px;
  right: 30px;
  font-size: 2rem;
  animation: sparkle 2s ease-in-out infinite; /* ✨ Sparkle animation */
}

/* 🎬 ANIMATIONS FOR PSEUDO-ELEMENTS */
@keyframes float {
  0%,
  100% {
    transform: translateY(0px) rotate(0deg);
  }
  50% {
    transform: translateY(-20px) rotate(180deg);
  }
}

@keyframes sparkle {
  0%,
  100% {
    opacity: 1;
    transform: scale(1);
  }
  50% {
    opacity: 0.5;
    transform: scale(1.2);
  }
}

/* 📋 FORM STYLING */
.contact-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 30px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2c3e50;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 1rem;
  transition: border-color 0.3s ease;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #3498db;
}

.form-group textarea {
  resize: vertical;
  min-height: 120px;
}

/* 🖼️ GALLERY STYLING */
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
}

.gallery-item {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.gallery-item:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 25px rgba(0, 0, 0, 0.15);
}

.gallery-item img {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.gallery-item p {
  padding: 15px;
  margin: 0;
  font-weight: 500;
  color: #2c3e50;
}
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

### **🚀 Angular SCSS Setup & Configuration with HTML Implementation**

```html
<!-- 📱 ANGULAR COMPONENT HTML - product-card.component.html -->
<article
  class="product-card"
  [class.product-card--loading]="isLoading"
  [class.product-card--featured]="isFeatured"
>
  <!-- 🖼️ PRODUCT IMAGE SECTION -->
  <div class="product-card__image-container">
    <!-- 📸 Main product image -->
    <img
      class="product-card__image"
      [src]="product.imageUrl"
      [alt]="product.name"
      (load)="onImageLoad()"
      (error)="onImageError()"
    />

    <!-- 🏷️ Product badges with dynamic classes -->
    <div class="product-card__badges" *ngIf="product.badges?.length">
      <span
        class="product-card__badge"
        [ngClass]="'product-card__badge--' + badge.type"
        *ngFor="let badge of product.badges"
      >
        {{ badge.text }}
      </span>
    </div>

    <!-- ❤️ Favorite button with state -->
    <button
      class="product-card__favorite-btn"
      [class.product-card__favorite-btn--active]="isFavorite"
      (click)="toggleFavorite()"
      type="button"
      [attr.aria-label]="isFavorite ? 'Remove from favorites' : 'Add to favorites'"
    >
      <svg class="product-card__favorite-icon" viewBox="0 0 24 24">
        <path
          d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"
        />
      </svg>
    </button>
  </div>

  <!-- 📄 PRODUCT CONTENT SECTION -->
  <div class="product-card__content">
    <!-- 🏷️ Product category -->
    <span class="product-card__category">{{ product.category }}</span>

    <!-- � Product title -->
    <h3 class="product-card__title">{{ product.name }}</h3>

    <!-- ⭐ Product rating -->
    <div class="product-card__rating" *ngIf="product.rating">
      <div class="product-card__stars">
        <span
          class="product-card__star"
          [class.product-card__star--filled]="i <= product.rating"
          *ngFor="let i of [1,2,3,4,5]"
          >★</span
        >
      </div>
      <span class="product-card__rating-text"
        >({{ product.reviewCount }} reviews)</span
      >
    </div>

    <!-- 📝 Product description -->
    <p class="product-card__description">{{ product.description }}</p>
  </div>

  <!-- 💰 PRODUCT FOOTER SECTION -->
  <footer class="product-card__footer">
    <!-- 💸 Price information -->
    <div class="product-card__price-container">
      <span class="product-card__price product-card__price--current">
        {{ product.currentPrice | currency }}
      </span>
      <span
        class="product-card__price product-card__price--original"
        *ngIf="product.originalPrice && product.originalPrice !== product.currentPrice"
      >
        {{ product.originalPrice | currency }}
      </span>
      <span class="product-card__discount" *ngIf="product.discountPercentage">
        {{ product.discountPercentage }}% OFF
      </span>
    </div>

    <!-- 🛒 Action buttons -->
    <div class="product-card__actions">
      <button
        class="product-card__btn product-card__btn--secondary"
        (click)="quickView()"
        type="button"
      >
        Quick View
      </button>
      <button
        class="product-card__btn product-card__btn--primary"
        [disabled]="!product.inStock"
        (click)="addToCart()"
        type="button"
      >
        <span *ngIf="product.inStock; else outOfStock">Add to Cart</span>
        <ng-template #outOfStock>Out of Stock</ng-template>
      </button>
    </div>
  </footer>

  <!-- 🔄 Loading overlay -->
  <div class="product-card__loading-overlay" *ngIf="isLoading">
    <div class="product-card__spinner">
      <svg class="product-card__spinner-svg" viewBox="0 0 50 50">
        <circle
          class="product-card__spinner-path"
          cx="25"
          cy="25"
          r="20"
          fill="none"
          stroke="#3498db"
          stroke-width="2"
        />
      </svg>
    </div>
  </div>
</article>
```

```typescript
// �🔧 ANGULAR.JSON SCSS CONFIGURATION
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

```typescript
// 📱 ANGULAR COMPONENT TypeScript - product-card.component.ts
import {
  Component,
  Input,
  Output,
  EventEmitter,
  ChangeDetectionStrategy,
} from "@angular/core";

export interface Product {
  id: string; // 🆔 Unique product identifier
  name: string; // 📝 Product name
  description: string; // 📄 Product description
  category: string; // 🏷️ Product category
  imageUrl: string; // 🖼️ Product image URL
  currentPrice: number; // 💰 Current price
  originalPrice?: number; // 💸 Original price (for discounts)
  discountPercentage?: number; // 📊 Discount percentage
  rating: number; // ⭐ Product rating (1-5)
  reviewCount: number; // 📊 Number of reviews
  inStock: boolean; // 📦 Stock availability
  badges?: ProductBadge[]; // 🏷️ Product badges (new, sale, etc.)
}

export interface ProductBadge {
  type: "sale" | "new" | "featured" | "limited"; // 🎨 Badge types for styling
  text: string; // 📝 Badge display text
}

@Component({
  selector: "app-product-card", // 🏷️ Component selector
  templateUrl: "./product-card.component.html",
  styleUrls: ["./product-card.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush, // ⚡ Performance optimization
})
export class ProductCardComponent {
  // 📥 INPUT PROPERTIES
  @Input() product!: Product; // 🛍️ Product data from parent
  @Input() isFeatured = false; // ⭐ Featured product flag
  @Input() isLoading = false; // 🔄 Loading state flag

  // 📤 OUTPUT EVENTS
  @Output() favoriteToggle = new EventEmitter<{
    productId: string;
    isFavorite: boolean;
  }>();
  @Output() addToCartClick = new EventEmitter<string>(); // 🛒 Add to cart event
  @Output() quickViewClick = new EventEmitter<string>(); // 👁️ Quick view event

  // 🔧 COMPONENT STATE
  isFavorite = false; // ❤️ Favorite state

  // 🖼️ IMAGE EVENT HANDLERS
  onImageLoad(): void {
    console.log("Product image loaded successfully"); // 📸 Image load success
  }

  onImageError(): void {
    console.error("Failed to load product image"); // ❌ Image load error
    // Could set fallback image here
  }

  // ❤️ FAVORITE TOGGLE
  toggleFavorite(): void {
    this.isFavorite = !this.isFavorite; // 🔄 Toggle state
    this.favoriteToggle.emit({
      productId: this.product.id,
      isFavorite: this.isFavorite,
    });
  }

  // 🛒 ADD TO CART
  addToCart(): void {
    if (this.product.inStock) {
      // ✅ Check stock availability
      this.addToCartClick.emit(this.product.id);
    }
  }

  // 👁️ QUICK VIEW
  quickView(): void {
    this.quickViewClick.emit(this.product.id);
  }
}
```

### **🎨 SCSS Variables & Theming System with Component Implementation**

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
  900: #0d47a1 // 🔵 Darkest blue,,,,
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
  900: #880e4f // 🌸 Darkest pink,,,,
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

### **📐 CSS Positioning Types with HTML Examples**

```html
<!-- 📐 CSS POSITIONING DEMO HTML -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CSS Positioning Complete Guide</title>
    <link rel="stylesheet" href="positioning-styles.css" />
  </head>
  <body>
    <!-- 🎯 POSITIONING DEMONSTRATIONS -->
    <div class="positioning-demo">
      <!-- 📋 SECTION 1: Static vs Relative Positioning -->
      <section class="demo-section">
        <h2>Static vs Relative Positioning</h2>

        <!-- 🔍 STATIC POSITIONING DEMO -->
        <div class="demo-container">
          <h3>Static Positioning (Default)</h3>
          <div class="static-demo">
            <div class="box static-box">Static Box 1</div>
            <div class="box static-box highlighted">
              Static Box 2 (highlighted)
            </div>
            <div class="box static-box">Static Box 3</div>
            <p>
              📍 All boxes follow normal document flow. The middle box cannot be
              moved with position properties.
            </p>
          </div>
        </div>

        <!-- 📍 RELATIVE POSITIONING DEMO -->
        <div class="demo-container">
          <h3>Relative Positioning</h3>
          <div class="relative-demo">
            <div class="box relative-box">Relative Box 1</div>
            <div class="box relative-box relative-moved">
              Relative Box 2 (moved)
            </div>
            <div class="box relative-box">Relative Box 3</div>
            <p>
              📍 The middle box is moved 30px right and 20px down, but its
              original space is preserved.
            </p>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 2: Absolute Positioning -->
      <section class="demo-section">
        <h2>Absolute Positioning Examples</h2>

        <!-- 📌 ABSOLUTE POSITIONING CONTAINER -->
        <div class="demo-container">
          <h3>Absolute Positioning with Relative Parent</h3>
          <div class="absolute-container">
            <div class="container-content">
              <p>
                This is the container content. The absolutely positioned
                elements are positioned relative to this container.
              </p>

              <!-- 📌 ABSOLUTELY POSITIONED ELEMENTS -->
              <div class="absolute-box top-left">Top Left</div>
              <div class="absolute-box top-right">Top Right</div>
              <div class="absolute-box bottom-left">Bottom Left</div>
              <div class="absolute-box bottom-right">Bottom Right</div>
              <div class="absolute-box center">Centered</div>
            </div>
          </div>
        </div>

        <!-- 🎯 PRACTICAL EXAMPLE: IMAGE WITH OVERLAY -->
        <div class="demo-container">
          <h3>Practical Example: Image Card with Overlay</h3>
          <div class="image-card">
            <img
              src="https://via.placeholder.com/400x250/3498db/white?text=Beautiful+Landscape"
              alt="Landscape Photo"
              class="card-image"
            />

            <!-- 📍 OVERLAY ELEMENTS -->
            <div class="image-overlay">
              <h4>Beautiful Landscape</h4>
              <p>Stunning mountain view captured at sunset</p>
            </div>

            <div class="image-badge">Featured</div>

            <div class="image-actions">
              <button class="action-btn like-btn">❤️ Like</button>
              <button class="action-btn share-btn">📤 Share</button>
            </div>

            <div class="image-info">
              <span class="camera-info">📷 Canon EOS R5</span>
              <span class="location-info">📍 Swiss Alps</span>
            </div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 3: Fixed Positioning -->
      <section class="demo-section">
        <h2>Fixed Positioning Examples</h2>

        <!-- 📱 FIXED HEADER DEMO -->
        <div class="demo-container">
          <h3>Fixed Header Navigation</h3>
          <div class="fixed-demo-wrapper">
            <!-- 📍 FIXED HEADER -->
            <header class="fixed-header">
              <div class="header-content">
                <div class="logo">🚀 MyBrand</div>
                <nav class="main-nav">
                  <a href="#" class="nav-link">Home</a>
                  <a href="#" class="nav-link">About</a>
                  <a href="#" class="nav-link">Services</a>
                  <a href="#" class="nav-link">Contact</a>
                </nav>
                <div class="header-actions">
                  <button class="header-btn">Login</button>
                </div>
              </div>
            </header>

            <!-- 📄 SCROLLABLE CONTENT -->
            <div class="scrollable-content">
              <h4>Scroll down to see the fixed header in action</h4>
              <p>
                This content area is scrollable. The header above stays fixed at
                the top of the viewport.
              </p>

              <div class="content-block">
                <h5>Content Block 1</h5>
                <p>
                  Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed
                  do eiusmod tempor incididunt ut labore et dolore magna aliqua.
                </p>
              </div>

              <div class="content-block">
                <h5>Content Block 2</h5>
                <p>
                  Ut enim ad minim veniam, quis nostrud exercitation ullamco
                  laboris nisi ut aliquip ex ea commodo consequat.
                </p>
              </div>

              <div class="content-block">
                <h5>Content Block 3</h5>
                <p>
                  Duis aute irure dolor in reprehenderit in voluptate velit esse
                  cillum dolore eu fugiat nulla pariatur.
                </p>
              </div>

              <div class="content-block">
                <h5>Content Block 4</h5>
                <p>
                  Excepteur sint occaecat cupidatat non proident, sunt in culpa
                  qui officia deserunt mollit anim id est laborum.
                </p>
              </div>
            </div>

            <!-- 📍 FIXED SIDEBAR -->
            <aside class="fixed-sidebar">
              <h4>Quick Links</h4>
              <ul>
                <li><a href="#">📊 Dashboard</a></li>
                <li><a href="#">📈 Analytics</a></li>
                <li><a href="#">👥 Users</a></li>
                <li><a href="#">⚙️ Settings</a></li>
              </ul>
            </aside>

            <!-- 🔔 FIXED NOTIFICATION -->
            <div class="fixed-notification">
              <span>🔔 3 new messages</span>
              <button class="close-btn">✖️</button>
            </div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 4: Sticky Positioning -->
      <section class="demo-section">
        <h2>Sticky Positioning Examples</h2>

        <!-- 📌 STICKY TABLE HEADER -->
        <div class="demo-container">
          <h3>Sticky Table Header</h3>
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr class="sticky-header-row">
                  <th>Product Name</th>
                  <th>Category</th>
                  <th>Price</th>
                  <th>Stock</th>
                  <th>Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td>iPhone 15 Pro</td>
                  <td>Electronics</td>
                  <td>$999.00</td>
                  <td>25</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>Samsung Galaxy S24</td>
                  <td>Electronics</td>
                  <td>$899.00</td>
                  <td>18</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>MacBook Pro M3</td>
                  <td>Computers</td>
                  <td>$1,999.00</td>
                  <td>12</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>Dell XPS 13</td>
                  <td>Computers</td>
                  <td>$1,299.00</td>
                  <td>8</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>Sony WH-1000XM5</td>
                  <td>Audio</td>
                  <td>$399.00</td>
                  <td>45</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>AirPods Pro</td>
                  <td>Audio</td>
                  <td>$249.00</td>
                  <td>67</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>iPad Pro 12.9"</td>
                  <td>Tablets</td>
                  <td>$1,099.00</td>
                  <td>22</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>Surface Pro 9</td>
                  <td>Tablets</td>
                  <td>$999.00</td>
                  <td>15</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>Nintendo Switch</td>
                  <td>Gaming</td>
                  <td>$299.00</td>
                  <td>33</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
                <tr>
                  <td>PlayStation 5</td>
                  <td>Gaming</td>
                  <td>$499.00</td>
                  <td>7</td>
                  <td><button class="table-btn">Edit</button></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- 📌 STICKY SIDEBAR SECTIONS -->
        <div class="demo-container">
          <h3>Sticky Navigation Sections</h3>
          <div class="sticky-nav-demo">
            <div class="content-area">
              <!-- 📌 STICKY SECTION 1 -->
              <div class="content-section">
                <h4 class="sticky-section-title">📱 Mobile Development</h4>
                <div class="section-content">
                  <p>
                    Mobile development focuses on creating applications for
                    mobile devices. This includes native iOS and Android
                    development, as well as cross-platform solutions.
                  </p>
                  <p>
                    Key technologies include Swift for iOS, Kotlin for Android,
                    React Native, Flutter, and Ionic for cross-platform
                    development.
                  </p>
                  <p>
                    Mobile developers need to consider device constraints, touch
                    interfaces, and platform-specific design guidelines.
                  </p>
                </div>
              </div>

              <!-- 📌 STICKY SECTION 2 -->
              <div class="content-section">
                <h4 class="sticky-section-title">🌐 Web Development</h4>
                <div class="section-content">
                  <p>
                    Web development encompasses frontend and backend
                    development. Frontend involves HTML, CSS, JavaScript, and
                    frameworks like React, Vue, or Angular.
                  </p>
                  <p>
                    Backend development includes server-side programming with
                    languages like Node.js, Python, Java, or PHP, along with
                    databases and APIs.
                  </p>
                  <p>
                    Modern web development also includes DevOps practices, CI/CD
                    pipelines, and cloud deployment strategies.
                  </p>
                </div>
              </div>

              <!-- 📌 STICKY SECTION 3 -->
              <div class="content-section">
                <h4 class="sticky-section-title">🤖 AI & Machine Learning</h4>
                <div class="section-content">
                  <p>
                    Artificial Intelligence and Machine Learning are
                    transforming technology. Python is the dominant language
                    with libraries like TensorFlow, PyTorch, and scikit-learn.
                  </p>
                  <p>
                    Key areas include deep learning, natural language
                    processing, computer vision, and reinforcement learning.
                  </p>
                  <p>
                    AI applications span from recommendation systems to
                    autonomous vehicles and medical diagnosis.
                  </p>
                </div>
              </div>

              <!-- 📌 STICKY SECTION 4 -->
              <div class="content-section">
                <h4 class="sticky-section-title">☁️ Cloud Computing</h4>
                <div class="section-content">
                  <p>
                    Cloud computing provides scalable, on-demand access to
                    computing resources. Major platforms include AWS, Azure, and
                    Google Cloud Platform.
                  </p>
                  <p>
                    Key services include compute instances, storage solutions,
                    databases, and serverless computing with functions.
                  </p>
                  <p>
                    Cloud architecture patterns include microservices,
                    containerization with Docker and Kubernetes, and
                    Infrastructure as Code.
                  </p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 5: Z-Index and Stacking Context -->
      <section class="demo-section">
        <h2>Z-Index and Stacking Context</h2>

        <!-- 🌊 Z-INDEX DEMONSTRATION -->
        <div class="demo-container">
          <h3>Z-Index Layering</h3>
          <div class="z-index-demo">
            <div class="layer layer-1">Layer 1 (z-index: 1)</div>
            <div class="layer layer-2">Layer 2 (z-index: 10)</div>
            <div class="layer layer-3">Layer 3 (z-index: 5)</div>
            <div class="layer layer-4">Layer 4 (z-index: 15)</div>
            <p>
              🌊 Higher z-index values appear above lower ones. Layer 4 (15) is
              highest, then Layer 2 (10), then Layer 3 (5), then Layer 1 (1).
            </p>
          </div>
        </div>

        <!-- 🏗️ STACKING CONTEXT DEMO -->
        <div class="demo-container">
          <h3>Stacking Context Example</h3>
          <div class="stacking-context-demo">
            <div class="context-parent-1">
              <span class="context-label">Parent 1 (z-index: 2)</span>
              <div class="context-child">Child (z-index: 999)</div>
            </div>

            <div class="context-parent-2">
              <span class="context-label">Parent 2 (z-index: 1)</span>
              <div class="context-child">Child (z-index: 999)</div>
            </div>

            <p>
              🏗️ Even though both children have z-index: 999, Parent 1's child
              appears above Parent 2's child because Parent 1 has a higher
              z-index.
            </p>
          </div>
        </div>
      </section>
    </div>
  </body>
</html>
```

```css
/* 🔍 STATIC POSITIONING - Default behavior */
.static-box {
  position: static; /* 📍 Default value - elements flow normally */
  background: #ecf0f1; /* 🎨 Light gray background */
  border: 2px solid #bdc3c7; /* 🖼️ Gray border */
  padding: 1rem; /* 📦 Internal padding */
  margin: 0.5rem; /* 🌌 External spacing */
  border-radius: 8px; /* 🔄 Rounded corners */
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

.highlighted {
  background: #f39c12 !important; /* 🟠 Orange highlight */
  color: white; /* ⚪ White text */
  font-weight: bold; /* 📝 Bold text */
}

/* 📍 RELATIVE POSITIONING - Offset from original position */
.relative-box {
  position: relative; /* 📍 Positioned relative to original location */
  background: #3498db; /* 🔵 Blue background */
  color: white; /* ⚪ White text */
  padding: 1rem; /* 📦 Internal padding */
  margin: 0.5rem; /* 🌌 External spacing */
  border-radius: 8px; /* 🔄 Rounded corners */
  transition: all 0.3s ease; /* ⚡ Smooth transitions */
}

.relative-moved {
  top: 20px; /* ⬇️ Move 20px down from original position */
  left: 30px; /* ➡️ Move 30px right from original position */
  z-index: 10; /* 🌊 Can participate in stacking context */
  background: #e74c3c; /* 🔴 Red to show it's moved */
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.3); /* 🌫️ Red shadow */
}

/*
🔍 Relative positioning explanation:
- Element maintains its space in document flow
- Visual position is offset from original location
- Other elements act as if it's still in original position
- Creates new stacking context for child elements
- Commonly used as positioning context for absolute children
*/

/* 📌 ABSOLUTE POSITIONING CONTAINER */
.absolute-container {
  position: relative; /* 📍 Creates positioning context for children */
  width: 100%; /* 📏 Full width */
  height: 400px; /* 📏 Container height */
  background: #f8f9fa; /* 🎨 Light background */
  border: 3px dashed #dee2e6; /* 🖼️ Dashed border */
  border-radius: 12px; /* 🔄 Rounded container */
  overflow: hidden; /* 🚫 Hide overflow */
}

.container-content {
  padding: 2rem; /* 📦 Content padding */
  color: #7f8c8d; /* 🎨 Gray text */
  font-style: italic; /* 📝 Italic text */
}

/* 📌 ABSOLUTELY POSITIONED ELEMENTS */
.absolute-box {
  position: absolute; /* 📌 Positioned relative to .absolute-container */
  background: #9b59b6; /* 🟣 Purple background */
  color: white; /* ⚪ White text */
  padding: 0.75rem 1rem; /* 📦 Box padding */
  border-radius: 6px; /* 🔄 Rounded corners */
  font-weight: 500; /* 📝 Medium weight */
  font-size: 0.875rem; /* 📏 Smaller text */
  box-shadow: 0 2px 8px rgba(155, 89, 182, 0.3); /* 🌫️ Purple shadow */
  z-index: 20; /* 🌊 Higher stacking order */
}

.top-left {
  top: 15px; /* 📐 15px from top */
  left: 15px; /* 📐 15px from left */
}

.top-right {
  top: 15px; /* 📐 15px from top */
  right: 15px; /* 📐 15px from right */
}

.bottom-left {
  bottom: 15px; /* 📐 15px from bottom */
  left: 15px; /* 📐 15px from left */
}

.bottom-right {
  bottom: 15px; /* 📐 15px from bottom */
  right: 15px; /* 📐 15px from right */
}

.center {
  top: 50%; /* 📐 50% from top */
  left: 50%; /* 📐 50% from left */
  transform: translate(-50%, -50%); /* 🎯 Perfect centering technique */
  background: #e74c3c; /* 🔴 Red for center element */
  padding: 1rem 1.5rem; /* 📦 Larger padding */
  font-weight: bold; /* 📝 Bold text */
}

/*
🔍 Absolute positioning explanation:
- Element removed from normal document flow
- Positioned relative to nearest positioned ancestor (not static)
- If no positioned ancestor, uses initial containing block (viewport)
- Does not affect positioning of other elements
- Creates new stacking context
*/

/* 🎯 IMAGE CARD WITH OVERLAY */
.image-card {
  position: relative; /* 📍 Creates positioning context */
  width: 400px; /* 📏 Card width */
  border-radius: 12px; /* 🔄 Rounded card */
  overflow: hidden; /* 🚫 Hide overflow */
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15); /* 🌫️ Card shadow */
  transition: transform 0.3s ease; /* ⚡ Smooth transform */
}

.image-card:hover {
  transform: translateY(-4px); /* ⬆️ Lift on hover */
}

.card-image {
  width: 100%; /* 📏 Full width image */
  height: 250px; /* 📏 Fixed height */
  object-fit: cover; /* 🖼️ Cover entire area */
  object-position: center; /* 🎯 Center image */
}

/* 📍 OVERLAY ELEMENTS */
.image-overlay {
  position: absolute; /* 📍 Absolute positioning */
  bottom: 0; /* 📐 Bottom of card */
  left: 0; /* 📐 Left edge */
  right: 0; /* 📐 Right edge */
  background: linear-gradient(
    transparent,
    rgba(0, 0, 0, 0.8)
  ); /* 🌈 Gradient overlay */
  color: white; /* ⚪ White text */
  padding: 3rem 1.5rem 1.5rem; /* 📦 Overlay padding */
  transform: translateY(100%); /* 🔄 Hidden by default */
  transition: transform 0.4s ease; /* ⚡ Smooth slide */
}

.image-card:hover .image-overlay {
  transform: translateY(0); /* 🔄 Slide in on hover */
}

.image-badge {
  position: absolute; /* 📍 Absolute positioning */
  top: 15px; /* 📐 Top spacing */
  left: 15px; /* 📐 Left spacing */
  background: #e74c3c; /* 🔴 Red badge */
  color: white; /* ⚪ White text */
  padding: 0.5rem 1rem; /* 📦 Badge padding */
  border-radius: 20px; /* 🔄 Rounded badge */
  font-size: 0.75rem; /* 📏 Small text */
  font-weight: bold; /* 📝 Bold badge */
  text-transform: uppercase; /* 🔤 Uppercase text */
  letter-spacing: 0.05em; /* 📏 Letter spacing */
}

.image-actions {
  position: absolute; /* 📍 Absolute positioning */
  top: 15px; /* 📐 Top spacing */
  right: 15px; /* 📐 Right spacing */
  display: flex; /* 📦 Flex layout */
  gap: 0.5rem; /* 🌌 Space between buttons */
}

.action-btn {
  background: rgba(255, 255, 255, 0.9); /* 🎨 Semi-transparent white */
  border: none; /* 🚫 No border */
  padding: 0.5rem; /* 📦 Button padding */
  border-radius: 50%; /* 🔄 Circular button */
  cursor: pointer; /* 👆 Pointer cursor */
  transition: all 0.3s ease; /* ⚡ Smooth transitions */
  width: 40px; /* 📏 Button width */
  height: 40px; /* 📏 Button height */
  display: flex; /* 📦 Flex for centering */
  align-items: center; /* 📐 Center vertically */
  justify-content: center; /* 📐 Center horizontally */
  opacity: 0; /* 🔍 Hidden by default */
}

.image-card:hover .action-btn {
  opacity: 1; /* 🔍 Show on hover */
}

.action-btn:hover {
  background: white; /* ⚪ Full white on hover */
  transform: scale(1.1); /* 🔍 Scale on hover */
}

.image-info {
  position: absolute; /* 📍 Absolute positioning */
  bottom: 15px; /* 📐 Bottom spacing */
  left: 15px; /* 📐 Left spacing */
  right: 15px; /* 📐 Right spacing */
  display: flex; /* 📦 Flex layout */
  justify-content: space-between; /* 📏 Space between items */
  background: rgba(0, 0, 0, 0.7); /* 🎨 Dark overlay */
  color: white; /* ⚪ White text */
  padding: 0.75rem 1rem; /* 📦 Info padding */
  border-radius: 8px; /* 🔄 Rounded info */
  font-size: 0.75rem; /* 📏 Small text */
  opacity: 0; /* 🔍 Hidden by default */
  transition: opacity 0.3s ease; /* ⚡ Smooth fade */
}

.image-card:hover .image-info {
  opacity: 1; /* 🔍 Show on hover */
}

/* 📍 FIXED POSITIONING EXAMPLES */
.fixed-demo-wrapper {
  position: relative; /* 📍 Container for demo */
  height: 400px; /* 📏 Fixed height for demo */
  overflow: hidden; /* 🚫 Hide overflow */
  border: 2px solid #dee2e6; /* 🖼️ Container border */
  border-radius: 12px; /* 🔄 Rounded container */
  background: #f8f9fa; /* 🎨 Light background */
}

.fixed-header {
  position: fixed; /* 📍 Fixed to viewport */
  top: 0; /* ⬆️ Stick to top */
  left: 0; /* ⬅️ Stick to left */
  right: 0; /* ➡️ Stick to right */
  z-index: 1000; /* 🌊 High stacking order */
  background: #2c3e50; /* 🎨 Dark header */
  color: white; /* ⚪ White text */
  box-shadow: 0 2px 10px rgba(44, 62, 80, 0.3); /* 🌫️ Header shadow */
}

/* Note: In real implementation, this would be fixed to actual viewport */
.fixed-demo-wrapper .fixed-header {
  position: absolute; /* 📍 Absolute for demo container */
  top: 0; /* ⬆️ Top of demo container */
}

.header-content {
  display: flex; /* 📦 Flex layout */
  justify-content: space-between; /* 📏 Space between elements */
  align-items: center; /* 📐 Center alignment */
  padding: 1rem 2rem; /* 📦 Header padding */
  max-width: 1200px; /* 📏 Max content width */
  margin: 0 auto; /* 🎯 Center content */
}

.logo {
  font-size: 1.25rem; /* 📏 Logo size */
  font-weight: bold; /* 📝 Bold logo */
}

.main-nav {
  display: flex; /* 📦 Flex navigation */
  gap: 2rem; /* 🌌 Space between links */
}

.nav-link {
  color: white; /* ⚪ White links */
  text-decoration: none; /* 🚫 No underline */
  font-weight: 500; /* 📝 Medium weight */
  transition: color 0.3s ease; /* ⚡ Color transition */
}

.nav-link:hover {
  color: #3498db; /* 🔵 Blue on hover */
}

.header-btn {
  background: #3498db; /* 🔵 Blue button */
  color: white; /* ⚪ White text */
  border: none; /* 🚫 No border */
  padding: 0.5rem 1rem; /* 📦 Button padding */
  border-radius: 6px; /* 🔄 Rounded button */
  cursor: pointer; /* 👆 Pointer cursor */
  font-weight: 500; /* 📝 Medium weight */
  transition: background 0.3s ease; /* ⚡ Background transition */
}

.header-btn:hover {
  background: #2980b9; /* 🔵 Darker blue on hover */
}

.scrollable-content {
  margin-top: 70px; /* 🌌 Space for fixed header */
  padding: 2rem; /* 📦 Content padding */
  height: 300px; /* 📏 Fixed height for scrolling */
  overflow-y: auto; /* 📜 Vertical scrolling */
}

.content-block {
  margin-bottom: 2rem; /* 🌌 Block spacing */
  padding: 1.5rem; /* 📦 Block padding */
  background: white; /* ⚪ White background */
  border-radius: 8px; /* 🔄 Rounded blocks */
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1); /* 🌫️ Block shadow */
}

.fixed-sidebar {
  position: fixed; /* 📍 Fixed positioning */
  top: 70px; /* 📐 Below header */
  right: 20px; /* 📐 Right spacing */
  width: 200px; /* 📏 Sidebar width */
  background: white; /* ⚪ White background */
  border-radius: 12px; /* 🔄 Rounded sidebar */
  padding: 1.5rem; /* 📦 Sidebar padding */
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15); /* 🌫️ Sidebar shadow */
  z-index: 999; /* 🌊 High z-index */
}

/* Demo container adjustment */
.fixed-demo-wrapper .fixed-sidebar {
  position: absolute; /* 📍 Absolute for demo */
  top: 70px; /* 📐 Below header in demo */
  right: 10px; /* 📐 Right spacing in demo */
}

.fixed-sidebar ul {
  list-style: none; /* 🚫 No bullets */
  padding: 0; /* 🚫 No padding */
  margin: 0; /* 🚫 No margin */
}

.fixed-sidebar li {
  margin-bottom: 0.5rem; /* 🌌 Item spacing */
}

.fixed-sidebar a {
  color: #2c3e50; /* 🎨 Dark text */
  text-decoration: none; /* 🚫 No underline */
  font-size: 0.875rem; /* 📏 Small text */
  transition: color 0.3s ease; /* ⚡ Color transition */
}

.fixed-sidebar a:hover {
  color: #3498db; /* 🔵 Blue on hover */
}

.fixed-notification {
  position: fixed; /* 📍 Fixed positioning */
  bottom: 20px; /* 📐 Bottom spacing */
  right: 20px; /* 📐 Right spacing */
  background: #e74c3c; /* 🔴 Red notification */
  color: white; /* ⚪ White text */
  padding: 1rem 1.5rem; /* 📦 Notification padding */
  border-radius: 8px; /* 🔄 Rounded notification */
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.3); /* 🌫️ Red shadow */
  z-index: 9999; /* 🌊 Highest z-index */
  display: flex; /* 📦 Flex layout */
  align-items: center; /* 📐 Center alignment */
  gap: 1rem; /* 🌌 Space between elements */
}

/* Demo container adjustment */
.fixed-demo-wrapper .fixed-notification {
  position: absolute; /* 📍 Absolute for demo */
  bottom: 10px; /* 📐 Bottom in demo */
  right: 10px; /* 📐 Right in demo */
}

.close-btn {
  background: none; /* 🔍 Transparent background */
  border: none; /* 🚫 No border */
  color: white; /* ⚪ White text */
  cursor: pointer; /* 👆 Pointer cursor */
  font-size: 0.875rem; /* 📏 Small close button */
}

/*
🔍 Fixed positioning explanation:
- Always positioned relative to viewport
- Unaffected by scrolling (stays in same position)
- Removed from document flow
- Perfect for headers, sidebars, modal overlays
- Creates new stacking context
*/

/* 📌 STICKY POSITIONING EXAMPLES */
.table-container {
  height: 300px; /* 📏 Container height */
  overflow-y: auto; /* 📜 Vertical scrolling */
  border: 1px solid #dee2e6; /* 🖼️ Container border */
  border-radius: 8px; /* 🔄 Rounded container */
}

.data-table {
  width: 100%; /* 📏 Full width table */
  border-collapse: collapse; /* 🖼️ Collapsed borders */
  background: white; /* ⚪ White background */
}

.sticky-header-row {
  position: sticky; /* 📌 Sticky positioning */
  top: 0; /* 📐 Stick at top */
  background: #2c3e50; /* 🎨 Dark header */
  color: white; /* ⚪ White text */
  z-index: 10; /* 🌊 Above table content */
}

.data-table th,
.data-table td {
  padding: 1rem; /* 📦 Cell padding */
  text-align: left; /* ⬅️ Left alignment */
  border-bottom: 1px solid #dee2e6; /* 🖼️ Bottom border */
}

.data-table th {
  font-weight: 600; /* 📝 Bold headers */
  white-space: nowrap; /* 🚫 No text wrapping */
}

.table-btn {
  background: #3498db; /* 🔵 Blue button */
  color: white; /* ⚪ White text */
  border: none; /* 🚫 No border */
  padding: 0.5rem 1rem; /* 📦 Button padding */
  border-radius: 4px; /* 🔄 Rounded button */
  cursor: pointer; /* 👆 Pointer cursor */
  font-size: 0.875rem; /* 📏 Small button text */
  transition: background 0.3s ease; /* ⚡ Background transition */
}

.table-btn:hover {
  background: #2980b9; /* 🔵 Darker blue on hover */
}

/* 📌 STICKY SECTIONS */
.content-section {
  margin-bottom: 2rem; /* 🌌 Section spacing */
}

.sticky-section-title {
  position: sticky; /* 📌 Sticky positioning */
  top: 20px; /* 📐 Stick 20px from top */
  background: #3498db; /* 🔵 Blue background */
  color: white; /* ⚪ White text */
  padding: 1rem 1.5rem; /* 📦 Title padding */
  margin: 0 0 1rem 0; /* 🌌 Bottom margin only */
  border-radius: 8px; /* 🔄 Rounded title */
  font-size: 1.125rem; /* 📏 Title size */
  font-weight: 600; /* 📝 Semi-bold title */
  z-index: 5; /* 🌊 Above content */
  box-shadow: 0 2px 8px rgba(52, 152, 219, 0.3); /* 🌫️ Blue shadow */
}

.section-content {
  background: white; /* ⚪ White content background */
  padding: 2rem; /* 📦 Content padding */
  border-radius: 8px; /* 🔄 Rounded content */
  border: 1px solid #ecf0f1; /* 🖼️ Light border */
  margin-bottom: 1rem; /* 🌌 Content spacing */
}

.section-content p {
  margin-bottom: 1rem; /* 🌌 Paragraph spacing */
  line-height: 1.6; /* 📏 Better readability */
  color: #2c3e50; /* 🎨 Dark text */
}

/*
🔍 Sticky positioning explanation:
- Behaves like relative until scroll threshold is reached
- Then behaves like fixed within containing block
- Stays within parent element boundaries
- Perfect for table headers, navigation bars
- Requires threshold value (top, bottom, left, or right)
*/

/* 🌊 Z-INDEX AND STACKING CONTEXT */
.z-index-demo {
  position: relative; /* 📍 Creates stacking context */
  height: 200px; /* 📏 Demo height */
  background: #f8f9fa; /* 🎨 Light background */
  border: 2px dashed #dee2e6; /* 🖼️ Dashed border */
  border-radius: 8px; /* 🔄 Rounded container */
  margin-bottom: 1rem; /* 🌌 Bottom spacing */
}

.layer {
  position: absolute; /* 📍 Absolute positioning */
  width: 120px; /* 📏 Layer width */
  height: 80px; /* 📏 Layer height */
  color: white; /* ⚪ White text */
  font-weight: bold; /* 📝 Bold text */
  font-size: 0.875rem; /* 📏 Small text */
  border-radius: 8px; /* 🔄 Rounded layers */
  display: flex; /* 📦 Flex for centering */
  align-items: center; /* 📐 Center vertically */
  justify-content: center; /* 📐 Center horizontally */
  text-align: center; /* 🎯 Center text */
  border: 2px solid white; /* 🖼️ White border */
}

.layer-1 {
  top: 20px; /* 📐 Top position */
  left: 20px; /* 📐 Left position */
  background: #e74c3c; /* 🔴 Red layer */
  z-index: 1; /* 🌊 Lowest z-index */
}

.layer-2 {
  top: 40px; /* 📐 Overlapping position */
  left: 60px; /* 📐 Overlapping position */
  background: #3498db; /* 🔵 Blue layer */
  z-index: 10; /* 🌊 High z-index */
}

.layer-3 {
  top: 60px; /* 📐 Overlapping position */
  left: 100px; /* 📐 Overlapping position */
  background: #2ecc71; /* 🟢 Green layer */
  z-index: 5; /* 🌊 Medium z-index */
}

.layer-4 {
  top: 80px; /* 📐 Overlapping position */
  left: 140px; /* 📐 Overlapping position */
  background: #9b59b6; /* 🟣 Purple layer */
  z-index: 15; /* 🌊 Highest z-index */
}

/* 🏗️ STACKING CONTEXT DEMO */
.stacking-context-demo {
  display: flex; /* 📦 Side by side parents */
  gap: 2rem; /* 🌌 Space between parents */
  margin-bottom: 1rem; /* 🌌 Bottom spacing */
}

.context-parent-1,
.context-parent-2 {
  position: relative; /* 📍 Creates stacking context */
  width: 200px; /* 📏 Parent width */
  height: 150px; /* 📏 Parent height */
  border: 2px solid #34495e; /* 🖼️ Dark border */
  border-radius: 8px; /* 🔄 Rounded parents */
  background: #ecf0f1; /* 🎨 Light background */
}

.context-parent-1 {
  z-index: 2; /* 🌊 Higher parent z-index */
}

.context-parent-2 {
  z-index: 1; /* 🌊 Lower parent z-index */
}

.context-label {
  position: absolute; /* 📍 Label positioning */
  top: 10px; /* 📐 Top spacing */
  left: 10px; /* 📐 Left spacing */
  background: #34495e; /* 🎨 Dark label background */
  color: white; /* ⚪ White label text */
  padding: 0.25rem 0.5rem; /* 📦 Label padding */
  border-radius: 4px; /* 🔄 Rounded label */
  font-size: 0.75rem; /* 📏 Small label text */
  font-weight: bold; /* 📝 Bold label */
}

.context-child {
  position: absolute; /* 📍 Child positioning */
  bottom: 15px; /* 📐 Bottom spacing */
  right: 15px; /* 📐 Right spacing */
  width: 100px; /* 📏 Child width */
  height: 60px; /* 📏 Child height */
  background: #e74c3c; /* 🔴 Red child */
  color: white; /* ⚪ White text */
  border-radius: 6px; /* 🔄 Rounded child */
  display: flex; /* 📦 Flex for centering */
  align-items: center; /* 📐 Center vertically */
  justify-content: center; /* 📐 Center horizontally */
  font-size: 0.75rem; /* 📏 Small text */
  font-weight: bold; /* 📝 Bold text */
  z-index: 999; /* 🌊 Very high z-index */
  border: 2px solid white; /* 🖼️ White border */
}

/*
🔍 Stacking context explanation:
- z-index only compares within same stacking context
- Parent's z-index determines child's context level
- Child cannot escape parent's stacking context
- Even z-index: 999 won't help if parent has lower z-index
*/

/* 📋 DEMO LAYOUT STYLING */
.positioning-demo {
  max-width: 1200px; /* 📏 Maximum width */
  margin: 0 auto; /* 🎯 Center container */
  padding: 2rem; /* 📦 Container padding */
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  background: #f8f9fa; /* 🎨 Light background */
}

.demo-section {
  background: white; /* ⚪ White section background */
  margin-bottom: 3rem; /* 🌌 Section spacing */
  padding: 2.5rem; /* 📦 Section padding */
  border-radius: 16px; /* 🔄 Rounded sections */
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1); /* 🌫️ Section shadow */
}

.demo-section h2 {
  color: #2c3e50; /* 🎨 Dark heading */
  margin-top: 0; /* 🚫 No top margin */
  margin-bottom: 2rem; /* 🌌 Bottom margin */
  padding-bottom: 1rem; /* 📦 Bottom padding */
  border-bottom: 3px solid #3498db; /* 🖼️ Blue underline */
  font-size: 1.75rem; /* 📏 Large heading */
}

.demo-container {
  margin-bottom: 2.5rem; /* 🌌 Container spacing */
  padding: 1.5rem; /* 📦 Container padding */
  border: 1px solid #ecf0f1; /* 🖼️ Light border */
  border-radius: 12px; /* 🔄 Rounded container */
  background: #fafbfc; /* 🎨 Very light background */
}

.demo-container h3 {
  margin-top: 0; /* 🚫 No top margin */
  margin-bottom: 1.5rem; /* 🌌 Bottom margin */
  color: #34495e; /* 🎨 Dark gray heading */
  font-size: 1.125rem; /* 📏 Medium heading size */
  font-weight: 600; /* 📝 Semi-bold heading */
}

.demo-container p {
  color: #7f8c8d; /* 🎨 Gray explanatory text */
  font-style: italic; /* 📝 Italic explanation */
  margin-top: 1rem; /* 🌌 Top spacing */
  font-size: 0.875rem; /* 📏 Small explanatory text */
  line-height: 1.5; /* 📏 Better readability */
}
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

### **📦 Flexbox Container Properties with HTML Examples**

```html
<!-- 📦 FLEXBOX DEMO HTML STRUCTURE -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Flexbox Complete Guide</title>
    <link rel="stylesheet" href="flexbox-styles.css" />
  </head>
  <body>
    <!-- 🏠 FLEX CONTAINER DEMONSTRATIONS -->
    <main class="demo-wrapper">
      <!-- 📋 SECTION 1: Basic Flex Direction Examples -->
      <section class="demo-section">
        <h2>Flex Direction Examples</h2>

        <!-- ➡️ ROW DIRECTION (Default) -->
        <div class="demo-group">
          <h3>flex-direction: row (default)</h3>
          <div class="flex-container flex-direction-row">
            <div class="flex-item">Item 1</div>
            <div class="flex-item">Item 2</div>
            <div class="flex-item">Item 3</div>
            <div class="flex-item">Item 4</div>
          </div>
        </div>

        <!-- ⬅️ ROW REVERSE DIRECTION -->
        <div class="demo-group">
          <h3>flex-direction: row-reverse</h3>
          <div class="flex-container flex-direction-row-reverse">
            <div class="flex-item">Item 1</div>
            <div class="flex-item">Item 2</div>
            <div class="flex-item">Item 3</div>
            <div class="flex-item">Item 4</div>
          </div>
        </div>

        <!-- ⬇️ COLUMN DIRECTION -->
        <div class="demo-group">
          <h3>flex-direction: column</h3>
          <div class="flex-container flex-direction-column">
            <div class="flex-item">Item 1</div>
            <div class="flex-item">Item 2</div>
            <div class="flex-item">Item 3</div>
            <div class="flex-item">Item 4</div>
          </div>
        </div>

        <!-- ⬆️ COLUMN REVERSE DIRECTION -->
        <div class="demo-group">
          <h3>flex-direction: column-reverse</h3>
          <div class="flex-container flex-direction-column-reverse">
            <div class="flex-item">Item 1</div>
            <div class="flex-item">Item 2</div>
            <div class="flex-item">Item 3</div>
            <div class="flex-item">Item 4</div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 2: Justify Content Examples -->
      <section class="demo-section">
        <h2>Justify Content - Main Axis Alignment</h2>

        <!-- ⬅️ FLEX START -->
        <div class="demo-group">
          <h3>justify-content: flex-start</h3>
          <div class="flex-container justify-flex-start">
            <div class="flex-item">A</div>
            <div class="flex-item">B</div>
            <div class="flex-item">C</div>
          </div>
        </div>

        <!-- 🎯 CENTER -->
        <div class="demo-group">
          <h3>justify-content: center</h3>
          <div class="flex-container justify-center">
            <div class="flex-item">A</div>
            <div class="flex-item">B</div>
            <div class="flex-item">C</div>
          </div>
        </div>

        <!-- ➡️ FLEX END -->
        <div class="demo-group">
          <h3>justify-content: flex-end</h3>
          <div class="flex-container justify-flex-end">
            <div class="flex-item">A</div>
            <div class="flex-item">B</div>
            <div class="flex-item">C</div>
          </div>
        </div>

        <!-- 📏 SPACE BETWEEN -->
        <div class="demo-group">
          <h3>justify-content: space-between</h3>
          <div class="flex-container justify-space-between">
            <div class="flex-item">A</div>
            <div class="flex-item">B</div>
            <div class="flex-item">C</div>
          </div>
        </div>

        <!-- 🌌 SPACE AROUND -->
        <div class="demo-group">
          <h3>justify-content: space-around</h3>
          <div class="flex-container justify-space-around">
            <div class="flex-item">A</div>
            <div class="flex-item">B</div>
            <div class="flex-item">C</div>
          </div>
        </div>

        <!-- ⚡ SPACE EVENLY -->
        <div class="demo-group">
          <h3>justify-content: space-evenly</h3>
          <div class="flex-container justify-space-evenly">
            <div class="flex-item">A</div>
            <div class="flex-item">B</div>
            <div class="flex-item">C</div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 3: Align Items Examples -->
      <section class="demo-section">
        <h2>Align Items - Cross Axis Alignment</h2>

        <!-- 📏 STRETCH (Default) -->
        <div class="demo-group">
          <h3>align-items: stretch (default)</h3>
          <div class="flex-container align-stretch">
            <div class="flex-item">Short</div>
            <div class="flex-item">Medium content here</div>
            <div class="flex-item">
              Much longer content that spans multiple lines and takes up more
              space
            </div>
          </div>
        </div>

        <!-- ⬆️ FLEX START -->
        <div class="demo-group">
          <h3>align-items: flex-start</h3>
          <div class="flex-container align-flex-start">
            <div class="flex-item">Short</div>
            <div class="flex-item">Medium content here</div>
            <div class="flex-item">
              Much longer content that spans multiple lines
            </div>
          </div>
        </div>

        <!-- 🎯 CENTER -->
        <div class="demo-group">
          <h3>align-items: center</h3>
          <div class="flex-container align-center">
            <div class="flex-item">Short</div>
            <div class="flex-item">Medium content here</div>
            <div class="flex-item">
              Much longer content that spans multiple lines
            </div>
          </div>
        </div>

        <!-- ⬇️ FLEX END -->
        <div class="demo-group">
          <h3>align-items: flex-end</h3>
          <div class="flex-container align-flex-end">
            <div class="flex-item">Short</div>
            <div class="flex-item">Medium content here</div>
            <div class="flex-item">
              Much longer content that spans multiple lines
            </div>
          </div>
        </div>

        <!-- 📝 BASELINE -->
        <div class="demo-group">
          <h3>align-items: baseline</h3>
          <div class="flex-container align-baseline">
            <div class="flex-item small-text">Small</div>
            <div class="flex-item medium-text">Medium</div>
            <div class="flex-item large-text">Large</div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 4: Real-World Navigation Example -->
      <section class="demo-section">
        <h2>Real-World Example: Navigation Bar</h2>

        <nav class="navbar">
          <!-- 🏠 Logo section -->
          <div class="navbar__logo">
            <img
              src="https://via.placeholder.com/120x40/3498db/white?text=LOGO"
              alt="Company Logo"
            />
          </div>

          <!-- 🧭 Navigation menu -->
          <ul class="navbar__menu">
            <li class="navbar__item">
              <a href="#" class="navbar__link">Home</a>
            </li>
            <li class="navbar__item">
              <a href="#" class="navbar__link">Products</a>
            </li>
            <li class="navbar__item">
              <a href="#" class="navbar__link">About</a>
            </li>
            <li class="navbar__item">
              <a href="#" class="navbar__link">Contact</a>
            </li>
          </ul>

          <!-- 🔧 Action buttons -->
          <div class="navbar__actions">
            <button class="navbar__btn navbar__btn--secondary">Login</button>
            <button class="navbar__btn navbar__btn--primary">Sign Up</button>
          </div>
        </nav>
      </section>

      <!-- 📋 SECTION 5: Card Layout Example -->
      <section class="demo-section">
        <h2>Real-World Example: Product Cards</h2>

        <div class="product-grid">
          <!-- 🛍️ Product Card 1 -->
          <article class="product-card">
            <div class="product-card__image">
              <img
                src="https://via.placeholder.com/300x200/e74c3c/white?text=Product+1"
                alt="Product 1"
              />
            </div>
            <div class="product-card__content">
              <h3 class="product-card__title">Premium Headphones</h3>
              <p class="product-card__description">
                High-quality wireless headphones with noise cancellation.
              </p>
              <div class="product-card__footer">
                <span class="product-card__price">$299.99</span>
                <button class="product-card__btn">Add to Cart</button>
              </div>
            </div>
          </article>

          <!-- 🛍️ Product Card 2 -->
          <article class="product-card">
            <div class="product-card__image">
              <img
                src="https://via.placeholder.com/300x200/3498db/white?text=Product+2"
                alt="Product 2"
              />
            </div>
            <div class="product-card__content">
              <h3 class="product-card__title">Smart Watch</h3>
              <p class="product-card__description">
                Advanced smartwatch with health monitoring and GPS tracking
                capabilities.
              </p>
              <div class="product-card__footer">
                <span class="product-card__price">$399.99</span>
                <button class="product-card__btn">Add to Cart</button>
              </div>
            </div>
          </article>

          <!-- 🛍️ Product Card 3 -->
          <article class="product-card">
            <div class="product-card__image">
              <img
                src="https://via.placeholder.com/300x200/2ecc71/white?text=Product+3"
                alt="Product 3"
              />
            </div>
            <div class="product-card__content">
              <h3 class="product-card__title">Laptop Stand</h3>
              <p class="product-card__description">
                Ergonomic laptop stand for better posture and productivity.
              </p>
              <div class="product-card__footer">
                <span class="product-card__price">$79.99</span>
                <button class="product-card__btn">Add to Cart</button>
              </div>
            </div>
          </article>
        </div>
      </section>
    </main>
  </body>
</html>
```

```css
/* 🏠 FLEX CONTAINER - Parent element setup */
.flex-container {
  display: flex;                   /* 📦 Enable flexbox layout */
  /* Alternative: display: inline-flex; for inline flex containers */
  min-height: 120px;               /* 📏 Minimum height for visibility */
  padding: 20px;                   /* 📦 Internal spacing */
  margin: 10px 0;                  /* 🌌 Vertical spacing between examples */
  background: #f8f9fa;             /* 🎨 Light background */
  border: 2px dashed #dee2e6;      /* 🖼️ Dashed border for container */
  border-radius: 8px;              /* 🔄 Rounded corners */
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
.flex-direction-row {
  flex-direction: row;             /* ➡️ Left to right (default) */
}

.flex-direction-row-reverse {
  flex-direction: row-reverse;     /* ⬅️ Right to left */
}

.flex-direction-column {
  flex-direction: column;          /* ⬇️ Top to bottom */
  min-height: 300px;               /* 📏 Taller container for column demo */
}

.flex-direction-column-reverse {
  flex-direction: column-reverse;  /* ⬆️ Bottom to top */
  min-height: 300px;               /* 📏 Taller container for column demo */
}

/*
🔍 Flex direction explanation:
- Defines the main axis direction
- row: horizontal main axis (default)
- column: vertical main axis
- reverse versions flip the direction
- Cross axis is perpendicular to main axis
*/

/* 📏 JUSTIFY CONTENT - Main axis alignment */
.justify-flex-start {
  justify-content: flex-start;     /* ⬅️ Align to start (default) */
}

.justify-center {
  justify-content: center;         /* 🎯 Center alignment */
}

.justify-flex-end {
  justify-content: flex-end;       /* ➡️ Align to end */
}

.justify-space-between {
  justify-content: space-between;  /* 📏 Space between items */
}

.justify-space-around {
  justify-content: space-around;   /* 🌌 Space around items */
}

.justify-space-evenly {
  justify-content: space-evenly;   /* ⚡ Even space distribution */
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
.align-stretch {
  align-items: stretch;            /* 📏 Stretch to container height (default) */
  min-height: 150px;               /* 📏 Height to demonstrate stretching */
}

.align-flex-start {
  align-items: flex-start;         /* ⬆️ Align to cross-axis start */
  min-height: 150px;
}

.align-center {
  align-items: center;             /* 🎯 Center on cross-axis */
  min-height: 150px;
}

.align-flex-end {
  align-items: flex-end;           /* ⬇️ Align to cross-axis end */
  min-height: 150px;
}

.align-baseline {
  align-items: baseline;           /* 📝 Align to text baseline */
  min-height: 150px;
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

/* 🎨 FLEX ITEMS - Individual item styling */
.flex-item {
  background: #3498db;             /* 🔵 Blue background */
  color: white;                    /* ⚪ White text */
  padding: 15px 20px;              /* 📦 Internal padding */
  margin: 5px;                     /* 🌌 Small margin between items */
  border-radius: 6px;              /* 🔄 Rounded corners */
  font-weight: 500;                /* 📝 Medium font weight */
  text-align: center;              /* 🎯 Center text */
  min-width: 80px;                 /* 📏 Minimum width */

  /* 🎭 HOVER EFFECT */
  transition: all 0.3s ease;       /* ⚡ Smooth transitions */
  cursor: pointer;                 /* 👆 Pointer cursor */
}

.flex-item:hover {
  background: #2980b9;             /* 🔵 Darker blue on hover */
  transform: translateY(-2px);     /* ⬆️ Lift effect */
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3); /* 🌫️ Blue shadow */
}

/* 📝 BASELINE DEMONSTRATION - Different font sizes */
.small-text {
  font-size: 0.875rem;             /* 📏 Small text */
}

.medium-text {
  font-size: 1.25rem;              /* 📏 Medium text */
}

.large-text {
  font-size: 2rem;                 /* 📏 Large text */
}

/* 🧭 REAL-WORLD NAVBAR EXAMPLE */
.navbar {
  display: flex;                   /* 📦 Horizontal layout */
  justify-content: space-between;  /* 📏 Logo left, actions right */
  align-items: center;             /* 📐 Vertical centering */
  padding: 1rem 2rem;              /* 📦 Navbar padding */
  background: #2c3e50;             /* 🎨 Dark background */
  color: white;                    /* ⚪ White text */
  box-shadow: 0 2px 10px rgba(0,0,0,0.1); /* 🌫️ Subtle shadow */
}

.navbar__logo img {
  height: 40px;                    /* 📏 Logo height */
  width: auto;                     /* 📏 Maintain aspect ratio */
}

.navbar__menu {
  display: flex;                   /* 📦 Horizontal menu items */
  list-style: none;                /* 🚫 Remove bullet points */
  margin: 0;                       /* 🚫 Remove default margin */
  padding: 0;                      /* 🚫 Remove default padding */
  gap: 2rem;                       /* 🌌 Space between menu items */
}

.navbar__link {
  color: white;                    /* ⚪ White links */
  text-decoration: none;           /* 🚫 Remove underline */
  font-weight: 500;                /* 📝 Medium weight */
  padding: 0.5rem 1rem;            /* 📦 Clickable area */
  border-radius: 4px;              /* 🔄 Rounded corners */
  transition: background 0.3s ease; /* ⚡ Smooth hover */
}

.navbar__link:hover {
  background: rgba(255, 255, 255, 0.1); /* 🎭 Subtle hover effect */
}

.navbar__actions {
  display: flex;                   /* 📦 Horizontal button layout */
  gap: 1rem;                       /* 🌌 Space between buttons */
}

.navbar__btn {
  padding: 0.5rem 1rem;            /* 📦 Button padding */
  border: none;                    /* 🚫 Remove default border */
  border-radius: 4px;              /* 🔄 Rounded corners */
  font-weight: 500;                /* 📝 Medium weight */
  cursor: pointer;                 /* 👆 Pointer cursor */
  transition: all 0.3s ease;       /* ⚡ Smooth transitions */
}

.navbar__btn--secondary {
  background: transparent;         /* 🔍 Transparent background */
  color: white;                    /* ⚪ White text */
  border: 1px solid white;         /* 🖼️ White border */
}

.navbar__btn--secondary:hover {
  background: white;               /* ⚪ White background on hover */
  color: #2c3e50;                  /* 🎨 Dark text on hover */
}

.navbar__btn--primary {
  background: #3498db;             /* 🔵 Blue background */
  color: white;                    /* ⚪ White text */
}

.navbar__btn--primary:hover {
  background: #2980b9;             /* 🔵 Darker blue on hover */
}

/* 🛍️ PRODUCT CARDS LAYOUT */
.product-grid {
  display: flex;                   /* 📦 Flexible card layout */
  flex-wrap: wrap;                 /* ✅ Allow wrapping */
  gap: 2rem;                       /* 🌌 Space between cards */
  padding: 2rem;                   /* 📦 Container padding */
  justify-content: center;         /* 🎯 Center cards */
}

.product-card {
  flex: 1 1 300px;                 /* 📏 Flexible cards, min 300px */
  /* grow: 1, shrink: 1, basis: 300px */
  max-width: 350px;                /* 📏 Maximum card width */
  display: flex;                   /* 📦 Vertical card layout */
  flex-direction: column;          /* 📐 Stack vertically */
  background: white;               /* ⚪ White background */
  border-radius: 12px;             /* 🔄 Rounded corners */
  box-shadow: 0 4px 12px rgba(0,0,0,0.1); /* 🌫️ Card shadow */
  overflow: hidden;                /* 🚫 Hide overflow */
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.product-card:hover {
  transform: translateY(-4px);     /* ⬆️ Lift on hover */
  box-shadow: 0 8px 25px rgba(0,0,0,0.15); /* 🌫️ Enhanced shadow */
}

.product-card__image {
  height: 200px;                   /* 📏 Fixed image height */
  overflow: hidden;                /* 🚫 Hide image overflow */
}

.product-card__image img {
  width: 100%;                     /* 📏 Full width image */
  height: 100%;                    /* 📏 Full height image */
  object-fit: cover;               /* 🖼️ Cover entire area */
  object-position: center;         /* 🎯 Center image */
}

.product-card__content {
  padding: 1.5rem;                 /* 📦 Content padding */
  flex: 1;                         /* 📈 Expand to fill space */
  display: flex;                   /* 📦 Vertical content layout */
  flex-direction: column;          /* 📐 Stack content */
}

.product-card__title {
  font-size: 1.25rem;              /* 📏 Title size */
  font-weight: 600;                /* 📝 Semi-bold */
  color: #2c3e50;                  /* 🎨 Dark title color */
  margin: 0 0 0.75rem 0;           /* 🌌 Bottom margin */
}

.product-card__description {
  color: #7f8c8d;                  /* 🎨 Gray description */
  line-height: 1.6;                /* 📏 Readable line height */
  flex: 1;                         /* 📈 Take available space */
  margin: 0 0 1.5rem 0;            /* 🌌 Bottom margin */
}

.product-card__footer {
  display: flex;                   /* 📦 Horizontal footer */
  justify-content: space-between;  /* 📏 Space between price and button */
  align-items: center;             /* 📐 Vertical alignment */
  margin-top: auto;                /* ⬆️ Push to bottom */
}

.product-card__price {
  font-size: 1.5rem;               /* 📏 Large price */
  font-weight: 700;                /* 📝 Bold price */
  color: #e74c3c;                  /* 🔴 Red price color */
}

.product-card__btn {
  background: #3498db;             /* 🔵 Blue button */
  color: white;                    /* ⚪ White text */
  border: none;                    /* 🚫 No border */
  padding: 0.75rem 1.5rem;         /* 📦 Button padding */
  border-radius: 6px;              /* 🔄 Rounded button */
  font-weight: 500;                /* 📝 Medium weight */
  cursor: pointer;                 /* 👆 Pointer cursor */
  transition: background 0.3s ease; /* ⚡ Smooth transition */
}

.product-card__btn:hover {
  background: #2980b9;             /* 🔵 Darker blue on hover */
}

/* 📋 DEMO SECTION STYLING */
.demo-wrapper {
  max-width: 1200px;               /* 📏 Maximum content width */
  margin: 0 auto;                  /* 🎯 Center content */
  padding: 2rem;                   /* 📦 Wrapper padding */
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.demo-section {
  margin-bottom: 4rem;             /* 🌌 Section spacing */
  padding: 2rem;                   /* 📦 Section padding */
  background: white;               /* ⚪ White section background */
  border-radius: 12px;             /* 🔄 Rounded section */
  box-shadow: 0 2px 8px rgba(0,0,0,0.1); /* 🌫️ Section shadow */
}

.demo-section h2 {
  color: #2c3e50;                  /* 🎨 Dark heading */
  border-bottom: 3px solid #3498db; /* 🖼️ Blue underline */
  padding-bottom: 1rem;            /* 📦 Underline spacing */
  margin-bottom: 2rem;             /* 🌌 Bottom spacing */
}

.demo-group {
  margin-bottom: 2rem;             /* 🌌 Group spacing */
}

.demo-group h3 {
  color: #34495e;                  /* 🎨 Medium dark heading */
  margin-bottom: 1rem;             /* 🌌 Bottom spacing */
  font-family: 'Courier New', monospace; /* 🔤 Monospace for code */
  background: #f8f9fa;             /* 🎨 Light code background */
  padding: 0.5rem 1rem;            /* 📦 Code padding */
  border-radius: 4px;              /* 🔄 Rounded code background */
  border-left: 4px solid #3498db;  /* 🖼️ Blue accent */
}
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

### **🏗️ Grid Container Properties with HTML Examples**

```html
<!-- 🏗️ CSS GRID DEMO HTML STRUCTURE -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CSS Grid Complete Guide</title>
    <link rel="stylesheet" href="grid-styles.css" />
  </head>
  <body>
    <!-- 🌐 GRID LAYOUT DEMONSTRATIONS -->
    <main class="grid-demo-wrapper">
      <!-- 📋 SECTION 1: Basic Grid Layout -->
      <section class="demo-section">
        <h2>Basic Grid Layout Examples</h2>

        <!-- 📊 SIMPLE 3x3 GRID -->
        <div class="demo-group">
          <h3>Basic 3x3 Grid Layout</h3>
          <div class="grid-container basic-grid">
            <div class="grid-item">Item 1</div>
            <div class="grid-item">Item 2</div>
            <div class="grid-item">Item 3</div>
            <div class="grid-item">Item 4</div>
            <div class="grid-item">Item 5</div>
            <div class="grid-item">Item 6</div>
            <div class="grid-item">Item 7</div>
            <div class="grid-item">Item 8</div>
            <div class="grid-item">Item 9</div>
          </div>
        </div>

        <!-- 📏 FRACTIONAL UNITS DEMONSTRATION -->
        <div class="demo-group">
          <h3>Fractional Units (fr) - 1fr 2fr 1fr</h3>
          <div class="grid-container fractional-grid">
            <div class="grid-item">1 Part</div>
            <div class="grid-item">2 Parts (Double Width)</div>
            <div class="grid-item">1 Part</div>
          </div>
        </div>

        <!-- 🔄 REPEAT FUNCTION -->
        <div class="demo-group">
          <h3>Repeat Function - repeat(4, 1fr)</h3>
          <div class="grid-container repeat-grid">
            <div class="grid-item">Col 1</div>
            <div class="grid-item">Col 2</div>
            <div class="grid-item">Col 3</div>
            <div class="grid-item">Col 4</div>
          </div>
        </div>

        <!-- 🎯 MIXED UNITS -->
        <div class="demo-group">
          <h3>Mixed Units - 200px 1fr auto</h3>
          <div class="grid-container mixed-units-grid">
            <div class="grid-item fixed-width">Fixed 200px</div>
            <div class="grid-item flexible">Flexible (1fr)</div>
            <div class="grid-item auto-width">Auto Content Width</div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 2: Grid Areas Layout -->
      <section class="demo-section">
        <h2>Grid Areas - Website Layout</h2>

        <div class="website-layout">
          <!-- 📋 HEADER -->
          <header class="header">
            <h1>My Website</h1>
            <nav>
              <a href="#">Home</a>
              <a href="#">About</a>
              <a href="#">Services</a>
              <a href="#">Contact</a>
            </nav>
          </header>

          <!-- 🎯 MAIN CONTENT AREA -->
          <main class="content">
            <article>
              <h2>Main Article</h2>
              <p>
                This is the main content area. It takes up the central space and
                contains the primary information.
              </p>
              <p>
                The content area is flexible and expands to fill available space
                while maintaining proper proportions with the sidebar.
              </p>
            </article>
          </main>

          <!-- 📌 SIDEBAR -->
          <aside class="sidebar">
            <h3>Sidebar</h3>
            <ul>
              <li>Recent Posts</li>
              <li>Categories</li>
              <li>Tags</li>
              <li>Archives</li>
            </ul>
          </aside>

          <!-- 🦶 FOOTER -->
          <footer class="footer">
            <p>&copy; 2024 My Website. All rights reserved.</p>
          </footer>
        </div>
      </section>

      <!-- 📋 SECTION 3: Grid Item Positioning -->
      <section class="demo-section">
        <h2>Grid Item Positioning</h2>

        <!-- 📍 SPECIFIC POSITIONING -->
        <div class="demo-group">
          <h3>Specific Grid Positioning</h3>
          <div class="grid-container positioned-grid">
            <div class="grid-item item-1">
              Item 1<br />Grid Position: 1/1 to 2/3
            </div>
            <div class="grid-item item-2">
              Item 2<br />Grid Position: 2/3 to 4/4
            </div>
            <div class="grid-item item-3">
              Item 3<br />Grid Position: 3/1 to 4/3
            </div>
            <div class="grid-item auto-item">Auto Item</div>
            <div class="grid-item auto-item">Auto Item</div>
          </div>
        </div>

        <!-- 🔗 GRID SPAN DEMONSTRATION -->
        <div class="demo-group">
          <h3>Grid Span Examples</h3>
          <div class="grid-container span-grid">
            <div class="grid-item span-item-1">Spans 2 Columns</div>
            <div class="grid-item normal-item">Normal</div>
            <div class="grid-item normal-item">Normal</div>
            <div class="grid-item span-item-2">Spans 2 Rows</div>
            <div class="grid-item normal-item">Normal</div>
            <div class="grid-item normal-item">Normal</div>
            <div class="grid-item span-item-3">Spans 3 Columns</div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 4: Auto-fit and Auto-fill -->
      <section class="demo-section">
        <h2>Responsive Grid - Auto-fit vs Auto-fill</h2>

        <!-- 🔧 AUTO-FIT DEMONSTRATION -->
        <div class="demo-group">
          <h3>auto-fit - Stretches to fill container</h3>
          <div class="grid-container auto-fit-grid">
            <div class="grid-item">Item 1</div>
            <div class="grid-item">Item 2</div>
            <div class="grid-item">Item 3</div>
          </div>
        </div>

        <!-- 🔧 AUTO-FILL DEMONSTRATION -->
        <div class="demo-group">
          <h3>auto-fill - Maintains column size</h3>
          <div class="grid-container auto-fill-grid">
            <div class="grid-item">Item 1</div>
            <div class="grid-item">Item 2</div>
            <div class="grid-item">Item 3</div>
          </div>
        </div>
      </section>

      <!-- 📋 SECTION 5: Real-World Dashboard Example -->
      <section class="demo-section">
        <h2>Real-World Example: Dashboard Layout</h2>

        <div class="dashboard">
          <!-- 📊 DASHBOARD HEADER -->
          <header class="dashboard__header">
            <h1>Analytics Dashboard</h1>
            <div class="dashboard__user">
              <span>John Doe</span>
              <img
                src="https://via.placeholder.com/40/3498db/white?text=JD"
                alt="User Avatar"
              />
            </div>
          </header>

          <!-- 🧭 NAVIGATION SIDEBAR -->
          <nav class="dashboard__nav">
            <ul>
              <li><a href="#" class="active">📊 Overview</a></li>
              <li><a href="#">📈 Analytics</a></li>
              <li><a href="#">👥 Users</a></li>
              <li><a href="#">💰 Revenue</a></li>
              <li><a href="#">⚙️ Settings</a></li>
            </ul>
          </nav>

          <!-- 📊 QUICK STATS CARDS -->
          <section class="dashboard__stats">
            <div class="stat-card">
              <h3>Total Users</h3>
              <p class="stat-number">12,487</p>
              <span class="stat-change positive">+12%</span>
            </div>
            <div class="stat-card">
              <h3>Revenue</h3>
              <p class="stat-number">$45,892</p>
              <span class="stat-change positive">+8%</span>
            </div>
            <div class="stat-card">
              <h3>Orders</h3>
              <p class="stat-number">1,247</p>
              <span class="stat-change negative">-3%</span>
            </div>
          </section>

          <!-- 📈 MAIN CHART AREA -->
          <main class="dashboard__chart">
            <div class="chart-container">
              <h2>Revenue Overview</h2>
              <div class="chart-placeholder">
                📈 Chart visualization would go here
                <p>Interactive charts and graphs displaying key metrics</p>
              </div>
            </div>
          </main>

          <!-- 📋 RECENT ACTIVITY -->
          <aside class="dashboard__activity">
            <h3>Recent Activity</h3>
            <ul>
              <li>New user registration</li>
              <li>Order #1247 completed</li>
              <li>Payment received</li>
              <li>New message received</li>
              <li>System backup completed</li>
            </ul>
          </aside>

          <!-- 🎯 WIDGET AREA -->
          <section class="dashboard__widgets">
            <div class="widget">
              <h4>Quick Actions</h4>
              <button>Add User</button>
              <button>Export Data</button>
              <button>Generate Report</button>
            </div>
          </section>
        </div>
      </section>

      <!-- 📋 SECTION 6: Photo Gallery Grid -->
      <section class="demo-section">
        <h2>Real-World Example: Photo Gallery</h2>

        <div class="photo-gallery">
          <div class="photo-item large-photo">
            <img
              src="https://via.placeholder.com/400x300/e74c3c/white?text=Featured+Photo"
              alt="Featured Photo"
            />
            <div class="photo-overlay">
              <h3>Featured Image</h3>
              <p>Main highlight photo</p>
            </div>
          </div>

          <div class="photo-item">
            <img
              src="https://via.placeholder.com/300x200/3498db/white?text=Photo+1"
              alt="Photo 1"
            />
            <div class="photo-overlay">
              <h4>Photo 1</h4>
            </div>
          </div>

          <div class="photo-item">
            <img
              src="https://via.placeholder.com/300x200/2ecc71/white?text=Photo+2"
              alt="Photo 2"
            />
            <div class="photo-overlay">
              <h4>Photo 2</h4>
            </div>
          </div>

          <div class="photo-item tall-photo">
            <img
              src="https://via.placeholder.com/300x400/9b59b6/white?text=Tall+Photo"
              alt="Tall Photo"
            />
            <div class="photo-overlay">
              <h4>Portrait</h4>
              <p>Tall format photo</p>
            </div>
          </div>

          <div class="photo-item">
            <img
              src="https://via.placeholder.com/300x200/f39c12/white?text=Photo+3"
              alt="Photo 3"
            />
            <div class="photo-overlay">
              <h4>Photo 3</h4>
            </div>
          </div>

          <div class="photo-item">
            <img
              src="https://via.placeholder.com/300x200/1abc9c/white?text=Photo+4"
              alt="Photo 4"
            />
            <div class="photo-overlay">
              <h4>Photo 4</h4>
            </div>
          </div>

          <div class="photo-item wide-photo">
            <img
              src="https://via.placeholder.com/600x200/34495e/white?text=Panorama"
              alt="Panorama"
            />
            <div class="photo-overlay">
              <h4>Panorama</h4>
              <p>Wide landscape photo</p>
            </div>
          </div>
        </div>
      </section>
    </main>
  </body>
</html>
```

```css
/* 📊 BASIC GRID SETUP */
.grid-container {
  display: grid;                   /* 🔲 Enable grid layout */
  /* Alternative: display: inline-grid; for inline grid containers */
  background: #f8f9fa;             /* 🎨 Light background */
  border: 2px dashed #dee2e6;      /* 🖼️ Dashed border for visibility */
  border-radius: 8px;              /* 🔄 Rounded corners */
  padding: 20px;                   /* 📦 Container padding */
  margin: 20px 0;                  /* 🌌 Vertical spacing */
}

/*
🔍 Display grid explanation:
- Creates a grid formatting context
- Direct children become grid items
- Establishes implicit grid with auto-sized rows/columns
- Block-level grid container by default
- inline-grid creates inline-level grid containers
*/

/* 📊 BASIC 3X3 GRID */
.basic-grid {
  grid-template-columns: repeat(3, 1fr); /* 📏 3 equal columns */
  grid-template-rows: repeat(3, 100px);  /* 📏 3 rows, 100px each */
  gap: 15px;                              /* 🌌 15px gap between items */
}

/*
🔍 Basic grid explanation:
- repeat(3, 1fr): 3 columns, each taking 1 fraction of space
- repeat(3, 100px): 3 rows, each exactly 100px tall
- gap: Creates consistent spacing between all grid items
*/

/* 📈 FRACTIONAL UNITS DEMONSTRATION */
.fractional-grid {
  grid-template-columns: 1fr 2fr 1fr;    /* 📊 Proportional columns (1:2:1) */
  grid-template-rows: 120px;              /* 📏 Single row height */
  gap: 20px;                              /* 🌌 Gap between items */
}

/*
🔍 Fractional units explanation:
- 1fr 2fr 1fr creates 1:2:1 ratio
- Middle column gets twice the space
- fr units distribute available space proportionally
- Total fractions: 4fr (1+2+1), middle gets 2/4 = 50%
*/

/* 🔄 REPEAT FUNCTION DEMONSTRATION */
.repeat-grid {
  grid-template-columns: repeat(4, 1fr);  /* 📊 4 equal columns */
  grid-template-rows: 100px;               /* 📏 Single row */
  gap: 15px;                               /* 🌌 Consistent spacing */
}

/*
🔍 Repeat function explanation:
- repeat(count, size) reduces repetition
- repeat(4, 1fr) = 1fr 1fr 1fr 1fr
- More maintainable than writing each column
- Can combine with other sizes: 200px repeat(3, 1fr)
*/

/* 🎯 MIXED UNITS GRID */
.mixed-units-grid {
  grid-template-columns: 200px 1fr auto;  /* 📊 Fixed, flexible, content-based */
  grid-template-rows: 120px;               /* 📏 Single row height */
  gap: 20px;                               /* 🌌 Gap between columns */
}

/*
🔍 Mixed units explanation:
- 200px: Fixed width column
- 1fr: Takes remaining space after fixed columns
- auto: Sizes to fit content width
- Flexible layouts with controlled constraints
*/

/* 🏠 WEBSITE LAYOUT WITH GRID AREAS */
.website-layout {
  display: grid;
  grid-template-columns: 1fr 3fr 1fr;     /* 📊 Sidebar, content, sidebar ratio */
  grid-template-rows: auto 1fr auto;      /* 📏 Header, content, footer */
  grid-template-areas:
    "header  header  header"             /* 📋 Header spans full width */
    "sidebar content content"            /* 📋 Sidebar + content area */
    "footer  footer  footer";            /* 📋 Footer spans full width */
  min-height: 500px;                     /* 📏 Minimum layout height */
  gap: 20px;                             /* 🌌 Gap between areas */
  background: white;                     /* ⚪ White background */
  border-radius: 12px;                   /* 🔄 Rounded corners */
  overflow: hidden;                      /* 🚫 Hide content overflow */
  box-shadow: 0 4px 12px rgba(0,0,0,0.1); /* 🌫️ Subtle shadow */
}

/*
🔍 Grid template areas explanation:
- Named grid regions for semantic layouts
- Each string represents a row
- Same names span across columns/rows
- More readable than numeric positioning
*/

/* 📋 GRID AREA ASSIGNMENTS */
.header {
  grid-area: header;                     /* 📍 Assign to header area */
  background: #2c3e50;                   /* 🎨 Dark header background */
  color: white;                          /* ⚪ White text */
  padding: 1.5rem 2rem;                  /* 📦 Header padding */
  display: flex;                         /* 📦 Flex for header content */
  justify-content: space-between;        /* 📏 Space between title and nav */
  align-items: center;                   /* 📐 Center vertically */
}

.header h1 {
  margin: 0;                             /* 🚫 Remove default margin */
  font-size: 1.5rem;                     /* 📏 Header title size */
}

.header nav {
  display: flex;                         /* 📦 Horizontal navigation */
  gap: 2rem;                             /* 🌌 Space between nav items */
}

.header nav a {
  color: white;                          /* ⚪ White nav links */
  text-decoration: none;                 /* 🚫 Remove underline */
  font-weight: 500;                      /* 📝 Medium font weight */
  transition: color 0.3s ease;           /* ⚡ Smooth color transition */
}

.header nav a:hover {
  color: #3498db;                        /* 🔵 Blue on hover */
}

.content {
  grid-area: content;                    /* 📍 Assign to content area */
  padding: 2rem;                         /* 📦 Content padding */
  background: #ffffff;                   /* ⚪ White content background */
}

.sidebar {
  grid-area: sidebar;                    /* 📍 Assign to sidebar area */
  padding: 2rem;                         /* 📦 Sidebar padding */
  background: #ecf0f1;                   /* 🎨 Light gray background */
}

.sidebar ul {
  list-style: none;                      /* 🚫 Remove bullet points */
  padding: 0;                            /* 🚫 Remove padding */
  margin: 0;                             /* 🚫 Remove margin */
}

.sidebar li {
  padding: 0.5rem 0;                     /* 📦 Vertical item padding */
  border-bottom: 1px solid #bdc3c7;      /* 🖼️ Bottom border */
}

.footer {
  grid-area: footer;                     /* 📍 Assign to footer area */
  background: #34495e;                   /* 🎨 Dark footer background */
  color: white;                          /* ⚪ White footer text */
  padding: 1rem 2rem;                    /* 📦 Footer padding */
  text-align: center;                    /* 🎯 Center footer text */
}

/* 📍 POSITIONED GRID DEMONSTRATION */
.positioned-grid {
  grid-template-columns: repeat(4, 1fr);  /* 📊 4 equal columns */
  grid-template-rows: repeat(4, 100px);   /* 📏 4 rows, 100px each */
  gap: 10px;                               /* 🌌 Small gap */
}

.item-1 {
  grid-column: 1 / 3;                    /* 📍 Spans columns 1-2 */
  grid-row: 1 / 2;                       /* 📍 Row 1 only */
  background: #e74c3c !important;        /* 🔴 Red background */
}

.item-2 {
  grid-column: 3 / 4;                    /* 📍 Column 3 only */
  grid-row: 2 / 4;                       /* 📍 Spans rows 2-3 */
  background: #3498db !important;        /* 🔵 Blue background */
}

.item-3 {
  grid-column: 1 / 3;                    /* 📍 Spans columns 1-2 */
  grid-row: 3 / 4;                       /* 📍 Row 3 only */
  background: #2ecc71 !important;        /* 🟢 Green background */
}

/*
🔍 Grid positioning explanation:
- grid-column: start / end (1-based indexing)
- grid-row: start / end (1-based indexing)
- Line numbers refer to grid lines, not cells
- Items can overlap if positioned on same area
*/

/* 🔗 GRID SPAN DEMONSTRATION */
.span-grid {
  grid-template-columns: repeat(4, 1fr);  /* 📊 4 columns */
  grid-template-rows: repeat(3, 100px);   /* 📏 3 rows */
  gap: 15px;                               /* 🌌 Gap between items */
}

.span-item-1 {
  grid-column: span 2;                   /* 🔗 Spans 2 columns from current position */
  background: #9b59b6 !important;        /* 🟣 Purple background */
}

.span-item-2 {
  grid-row: span 2;                      /* 🔗 Spans 2 rows from current position */
  background: #f39c12 !important;        /* 🟠 Orange background */
}

.span-item-3 {
  grid-column: span 3;                   /* 🔗 Spans 3 columns */
  background: #1abc9c !important;        /* 🐟 Teal background */
}

/*
🔍 Grid span explanation:
- span keyword means "span this many tracks"
- More intuitive than calculating end positions
- Automatically calculates end line
- Works with auto-placement algorithm
*/

/* 🔧 AUTO-FIT GRID */
.auto-fit-grid {
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); /* 🔧 Responsive columns */
  gap: 20px;                                                   /* 🌌 Gap between items */
  min-height: 120px;                                           /* 📏 Minimum height */
}

/*
🔍 Auto-fit explanation:
- Creates as many columns as fit in container
- minmax(200px, 1fr): min 200px, expand to fill
- Columns stretch to fill available space
- Empty columns collapse
*/

/* 🔧 AUTO-FILL GRID */
.auto-fill-grid {
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); /* 🔧 Responsive columns */
  gap: 20px;                                                    /* 🌌 Gap between items */
  min-height: 120px;                                            /* 📏 Minimum height */
}

/*
🔍 Auto-fill explanation:
- Similar to auto-fit but maintains empty columns
- Preserves grid structure even with few items
- Empty columns remain at minimum size
- Useful for consistent layouts
*/

/* 🎨 GRID ITEMS STYLING */
.grid-item {
  background: #3498db;                   /* 🔵 Blue background */
  color: white;                          /* ⚪ White text */
  padding: 20px;                         /* 📦 Internal padding */
  border-radius: 8px;                    /* 🔄 Rounded corners */
  display: flex;                         /* 📦 Flex for centering */
  align-items: center;                   /* 📐 Center vertically */
  justify-content: center;               /* 📐 Center horizontally */
  font-weight: 500;                      /* 📝 Medium font weight */
  text-align: center;                    /* 🎯 Center text */
  transition: all 0.3s ease;             /* ⚡ Smooth transitions */
  cursor: pointer;                       /* 👆 Pointer cursor */
  line-height: 1.4;                      /* 📏 Better line spacing */
}

.grid-item:hover {
  background: #2980b9;                   /* 🔵 Darker blue on hover */
  transform: translateY(-2px);           /* ⬆️ Lift effect */
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3); /* 🌫️ Blue shadow */
}

/* 🎨 SPECIAL ITEM STYLING */
.normal-item {
  background: #95a5a6;                   /* 🎨 Gray background */
}

.auto-item {
  background: #e67e22;                   /* 🟠 Orange background */
}

.fixed-width {
  background: #e74c3c;                   /* 🔴 Red background */
}

.flexible {
  background: #2ecc71;                   /* 🟢 Green background */
}

.auto-width {
  background: #9b59b6;                   /* 🟣 Purple background */
}

/* 📊 DASHBOARD LAYOUT */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr 300px; /* 📏 Sidebar, main, widgets */
  grid-template-rows: auto auto 1fr auto; /* 📏 Header, stats, content, space */
  grid-template-areas:
    "header header header"
    "nav stats stats"
    "nav chart activity"
    "nav widgets widgets";
  min-height: 600px;                     /* 📏 Minimum dashboard height */
  gap: 20px;                             /* 🌌 Gap between sections */
  background: #f8f9fa;                   /* 🎨 Light background */
  border-radius: 12px;                   /* 🔄 Rounded dashboard */
  overflow: hidden;                      /* 🚫 Hide overflow */
  box-shadow: 0 6px 20px rgba(0,0,0,0.1); /* 🌫️ Dashboard shadow */
}

.dashboard__header {
  grid-area: header;                     /* 📍 Header area */
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); /* 🌈 Gradient */
  color: white;                          /* ⚪ White text */
  padding: 1.5rem 2rem;                  /* 📦 Header padding */
  display: flex;                         /* 📦 Flex layout */
  justify-content: space-between;        /* 📏 Space between elements */
  align-items: center;                   /* 📐 Center alignment */
}

.dashboard__user {
  display: flex;                         /* 📦 User info flex */
  align-items: center;                   /* 📐 Center user info */
  gap: 1rem;                             /* 🌌 Space between name and avatar */
}

.dashboard__user img {
  border-radius: 50%;                    /* 🔄 Circular avatar */
  width: 40px;                           /* 📏 Avatar size */
  height: 40px;                          /* 📏 Avatar size */
}

.dashboard__nav {
  grid-area: nav;                        /* 📍 Navigation area */
  background: #2c3e50;                   /* 🎨 Dark nav background */
  padding: 2rem 0;                       /* 📦 Vertical padding */
}

.dashboard__nav ul {
  list-style: none;                      /* 🚫 No bullets */
  padding: 0;                            /* 🚫 No padding */
  margin: 0;                             /* 🚫 No margin */
}

.dashboard__nav li {
  margin: 0;                             /* 🚫 No margin */
}

.dashboard__nav a {
  display: block;                        /* 📦 Block links */
  color: #bdc3c7;                        /* 🎨 Light gray text */
  text-decoration: none;                 /* 🚫 No underline */
  padding: 1rem 2rem;                    /* 📦 Link padding */
  transition: all 0.3s ease;             /* ⚡ Smooth transition */
}

.dashboard__nav a:hover,
.dashboard__nav a.active {
  background: #34495e;                   /* 🎨 Darker background */
  color: #3498db;                        /* 🔵 Blue text */
  border-right: 4px solid #3498db;       /* 🖼️ Blue accent */
}

.dashboard__stats {
  grid-area: stats;                      /* 📍 Stats area */
  display: flex;                         /* 📦 Horizontal stats */
  gap: 1.5rem;                           /* 🌌 Space between cards */
  padding: 0 2rem;                       /* 📦 Horizontal padding */
}

.stat-card {
  background: white;                     /* ⚪ White card background */
  padding: 2rem;                         /* 📦 Card padding */
  border-radius: 12px;                   /* 🔄 Rounded cards */
  box-shadow: 0 2px 8px rgba(0,0,0,0.1); /* 🌫️ Card shadow */
  flex: 1;                               /* 📈 Equal width cards */
  text-align: center;                    /* 🎯 Center text */
}

.stat-card h3 {
  margin: 0 0 1rem 0;                    /* 🌌 Bottom margin only */
  color: #7f8c8d;                        /* 🎨 Gray heading */
  font-size: 0.875rem;                   /* 📏 Small heading */
  text-transform: uppercase;             /* 🔤 Uppercase text */
  letter-spacing: 0.05em;                /* 📏 Letter spacing */
}

.stat-number {
  font-size: 2rem;                       /* 📏 Large number */
  font-weight: 700;                      /* 📝 Bold number */
  color: #2c3e50;                        /* 🎨 Dark number */
  margin: 0 0 0.5rem 0;                  /* 🌌 Bottom margin */
}

.stat-change {
  font-size: 0.875rem;                   /* 📏 Small change text */
  font-weight: 500;                      /* 📝 Medium weight */
}

.stat-change.positive {
  color: #27ae60;                        /* 🟢 Green for positive */
}

.stat-change.negative {
  color: #e74c3c;                        /* 🔴 Red for negative */
}

.dashboard__chart {
  grid-area: chart;                      /* 📍 Chart area */
  background: white;                     /* ⚪ White background */
  border-radius: 12px;                   /* 🔄 Rounded container */
  padding: 2rem;                         /* 📦 Chart padding */
  box-shadow: 0 2px 8px rgba(0,0,0,0.1); /* 🌫️ Chart shadow */
}

.chart-placeholder {
  background: linear-gradient(45deg, #f8f9fa 25%, transparent 25%),
              linear-gradient(-45deg, #f8f9fa 25%, transparent 25%),
              linear-gradient(45deg, transparent 75%, #f8f9fa 75%),
              linear-gradient(-45deg, transparent 75%, #f8f9fa 75%);
  background-size: 20px 20px;           /* 📏 Pattern size */
  background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
  border: 2px dashed #dee2e6;            /* 🖼️ Dashed border */
  border-radius: 8px;                    /* 🔄 Rounded placeholder */
  padding: 3rem;                         /* 📦 Placeholder padding */
  text-align: center;                    /* 🎯 Center text */
  color: #7f8c8d;                        /* 🎨 Gray text */
  min-height: 200px;                     /* 📏 Minimum height */
  display: flex;                         /* 📦 Flex for centering */
  flex-direction: column;                /* 📐 Stack vertically */
  justify-content: center;               /* 📐 Center vertically */
  align-items: center;                   /* 📐 Center horizontally */
}

.dashboard__activity {
  grid-area: activity;                   /* 📍 Activity area */
  background: white;                     /* ⚪ White background */
  border-radius: 12px;                   /* 🔄 Rounded container */
  padding: 2rem;                         /* 📦 Activity padding */
  box-shadow: 0 2px 8px rgba(0,0,0,0.1); /* 🌫️ Activity shadow */
}

.dashboard__activity h3 {
  margin-top: 0;                         /* 🚫 No top margin */
  color: #2c3e50;                        /* 🎨 Dark heading */
  border-bottom: 2px solid #ecf0f1;      /* 🖼️ Bottom border */
  padding-bottom: 1rem;                  /* 📦 Bottom padding */
}

.dashboard__activity ul {
  list-style: none;                      /* 🚫 No bullets */
  padding: 0;                            /* 🚫 No padding */
}

.dashboard__activity li {
  padding: 0.75rem 0;                    /* 📦 Vertical padding */
  border-bottom: 1px solid #ecf0f1;      /* 🖼️ Separator line */
  color: #7f8c8d;                        /* 🎨 Gray text */
}

.dashboard__widgets {
  grid-area: widgets;                    /* 📍 Widgets area */
  background: white;                     /* ⚪ White background */
  border-radius: 12px;                   /* 🔄 Rounded container */
  padding: 2rem;                         /* 📦 Widget padding */
  box-shadow: 0 2px 8px rgba(0,0,0,0.1); /* 🌫️ Widget shadow */
}

.widget h4 {
  margin-top: 0;                         /* 🚫 No top margin */
  color: #2c3e50;                        /* 🎨 Dark heading */
  margin-bottom: 1.5rem;                 /* 🌌 Bottom margin */
}

.widget button {
  display: block;                        /* 📦 Block buttons */
  width: 100%;                           /* 📏 Full width */
  margin: 0.5rem 0;                      /* 🌌 Vertical margin */
  padding: 0.75rem 1rem;                 /* 📦 Button padding */
  background: #3498db;                   /* 🔵 Blue background */
  color: white;                          /* ⚪ White text */
  border: none;                          /* 🚫 No border */
  border-radius: 6px;                    /* 🔄 Rounded button */
  cursor: pointer;                       /* 👆 Pointer cursor */
  transition: background 0.3s ease;      /* ⚡ Smooth transition */
}

.widget button:hover {
  background: #2980b9;                   /* 🔵 Darker blue on hover */
}

/* 🖼️ PHOTO GALLERY GRID */
.photo-gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); /* 🔧 Responsive columns */
  grid-auto-rows: 200px;                 /* 📏 Default row height */
  gap: 20px;                             /* 🌌 Gap between photos */
  padding: 2rem;                         /* 📦 Gallery padding */
}

.photo-item {
  position: relative;                    /* 📍 Relative positioning */
  overflow: hidden;                      /* 🚫 Hide overflow */
  border-radius: 12px;                   /* 🔄 Rounded photos */
  cursor: pointer;                       /* 👆 Pointer cursor */
  transition: transform 0.3s ease;       /* ⚡ Smooth transform */
}

.photo-item:hover {
  transform: scale(1.02);                /* 🔍 Slight scale on hover */
}

.photo-item img {
  width: 100%;                           /* 📏 Full width */
  height: 100%;                          /* 📏 Full height */
  object-fit: cover;                     /* 🖼️ Cover entire area */
  object-position: center;               /* 🎯 Center image */
}

.large-photo {
  grid-column: span 2;                   /* 🔗 Spans 2 columns */
  grid-row: span 2;                      /* 🔗 Spans 2 rows */
}

.tall-photo {
  grid-row: span 2;                      /* 🔗 Spans 2 rows */
}

.wide-photo {
  grid-column: span 2;                   /* 🔗 Spans 2 columns */
}

.photo-overlay {
  position: absolute;                    /* 📍 Absolute positioning */
  bottom: 0;                             /* 📍 Bottom placement */
  left: 0;                               /* 📍 Left alignment */
  right: 0;                              /* 📍 Right alignment */
  background: linear-gradient(transparent, rgba(0,0,0,0.7)); /* 🌈 Gradient overlay */
  color: white;                          /* ⚪ White text */
  padding: 2rem 1.5rem 1.5rem;          /* 📦 Overlay padding */
  transform: translateY(100%);           /* 🔄 Hidden by default */
  transition: transform 0.3s ease;       /* ⚡ Smooth slide */
}

.photo-item:hover .photo-overlay {
  transform: translateY(0);              /* 🔄 Slide in on hover */
}

.photo-overlay h3,
.photo-overlay h4 {
  margin: 0 0 0.5rem 0;                  /* 🌌 Bottom margin only */
}

.photo-overlay p {
  margin: 0;                             /* 🚫 No margin */
  opacity: 0.9;                          /* 🎨 Slight transparency */
}

/* 📋 DEMO STYLING */
.grid-demo-wrapper {
  max-width: 1200px;                     /* 📏 Maximum width */
  margin: 0 auto;                        /* 🎯 Center container */
  padding: 2rem;                         /* 📦 Wrapper padding */
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #f8f9fa;                   /* 🎨 Light wrapper background */
}

.demo-section {
  background: white;                     /* ⚪ White section background */
  margin-bottom: 3rem;                   /* 🌌 Section spacing */
  padding: 2.5rem;                       /* 📦 Section padding */
  border-radius: 16px;                   /* 🔄 Rounded sections */
  box-shadow: 0 4px 16px rgba(0,0,0,0.1); /* 🌫️ Section shadow */
}

.demo-section h2 {
  color: #2c3e50;                        /* 🎨 Dark heading */
  margin-top: 0;                         /* 🚫 No top margin */
  margin-bottom: 2rem;                   /* 🌌 Bottom margin */
  padding-bottom: 1rem;                  /* 📦 Bottom padding */
  border-bottom: 3px solid #3498db;      /* 🖼️ Blue underline */
  font-size: 1.75rem;                    /* 📏 Large heading */
}

.demo-group {
  margin-bottom: 2.5rem;                 /* 🌌 Group spacing */
}

.demo-group h3 {
  background: #34495e;                   /* 🎨 Dark background */
  color: white;                          /* ⚪ White text */
  padding: 0.75rem 1.25rem;              /* 📦 Heading padding */
  border-radius: 6px;                    /* 🔄 Rounded heading */
  font-family: 'Courier New', monospace; /* 🔤 Monospace font */
  font-size: 0.95rem;                    /* 📏 Code font size */
  margin-bottom: 1rem;                   /* 🌌 Bottom spacing */
  border-left: 4px solid #3498db;        /* 🖼️ Blue accent */
}
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

### **🎯 Grid Item Properties - Child Element Control**

```css
/* 📍 GRID POSITIONING - Line-based placement */
.grid-positioning {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* 📊 4 equal columns */
  grid-template-rows: repeat(3, 100px); /* 📏 3 rows, 100px each */
  gap: 10px; /* 🌌 Grid spacing */
}

.grid-item-positioned {
  /* 📍 COLUMN POSITIONING */
  grid-column-start: 2; /* ➡️ Start at column line 2 */
  grid-column-end: 4; /* ➡️ End at column line 4 */
  /* Shorthand: grid-column: 2 / 4; */ /* 📏 Spans from line 2 to 4 */

  /* 📍 ROW POSITIONING */
  grid-row-start: 1; /* ⬇️ Start at row line 1 */
  grid-row-end: 3; /* ⬇️ End at row line 3 */
  /* Shorthand: grid-row: 1 / 3; */ /* 📏 Spans from row 1 to 3 */

  /* 🔗 ULTRA SHORTHAND */
  /* grid-area: 1 / 2 / 3 / 4; */ /* 📏 row-start / col-start / row-end / col-end */

  background: #e74c3c; /* 🔴 Red background for visibility */
}

/*
🔍 Grid positioning explanation:
- Grid lines start at 1, not 0
- Lines exist between and around grid tracks
- Negative values count from the end
- Items can overlap by occupying same cells
- grid-area combines all positioning values
*/

/* 🔢 SPAN NOTATION - Relative positioning */
.grid-span-examples {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
}

.span-item-1 {
  grid-column: span 2; /* 📏 Span 2 columns from current position */
  background: #3498db; /* 🔵 Blue background */
}

.span-item-2 {
  grid-column: 3 / span 2; /* 📏 Start at column 3, span 2 columns */
  grid-row: span 2; /* 📏 Span 2 rows from current position */
  background: #2ecc71; /* 🟢 Green background */
}

.span-item-3 {
  grid-area: span 2 / span 3; /* 📏 Span 2 rows and 3 columns */
  background: #f39c12; /* 🟡 Orange background */
}

/*
🔍 Span notation explanation:
- span keyword creates relative positioning
- More flexible than absolute line numbers
- Works with both rows and columns
- Can combine with absolute positioning
- Useful for responsive designs
*/

/* 🎯 GRID ITEM ALIGNMENT - Individual positioning */
.grid-item-alignment {
  display: grid;
  grid-template-columns: repeat(3, 200px);
  grid-template-rows: repeat(3, 100px);
  gap: 10px;
}

.justify-self-item {
  /* 🎯 HORIZONTAL ALIGNMENT within grid cell */
  justify-self: center; /* 🎯 Center horizontally in cell */
  /* justify-self: start; */ /* ⬅️ Align to left (default) */
  /* justify-self: end; */ /* ➡️ Align to right */
  /* justify-self: stretch; */ /* 📏 Fill cell width */

  background: #9b59b6; /* 🟣 Purple background */
  width: 100px; /* 📏 Fixed width to see alignment */
}

.align-self-item {
  /* 🎯 VERTICAL ALIGNMENT within grid cell */
  align-self: center; /* 🎯 Center vertically in cell */
  /* align-self: start; */ /* ⬆️ Align to top (default) */
  /* align-self: end; */ /* ⬇️ Align to bottom */
  /* align-self: stretch; */ /* 📏 Fill cell height */

  background: #1abc9c; /* 🟢 Teal background */
  height: 50px; /* 📏 Fixed height to see alignment */
}

.place-self-item {
  /* 🎯 BOTH ALIGNMENTS combined */
  place-self: center; /* 🎯 Center both horizontally and vertically */
  /* place-self: start end; */ /* 🎯 Top-left horizontal, bottom vertical */

  background: #e67e22; /* 🟠 Orange background */
  width: 80px; /* 📏 Fixed dimensions */
  height: 60px; /* 📏 to see alignment */
}

/*
🔍 Grid item alignment explanation:
- justify-self: horizontal alignment within cell
- align-self: vertical alignment within cell
- place-self: shorthand for both alignments
- Only affects individual grid items
- Different from container-level alignment
*/
```

### **🚀 Advanced Grid Techniques**

```css
/* 🔄 AUTO-FIT vs AUTO-FILL - Responsive columns */
.grid-auto-fit {
  display: grid;
  gap: 20px;

  /* 🎯 AUTO-FIT - Columns expand to fill space */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  /*
  🔍 auto-fit explanation:
  - Creates as many columns as can fit
  - Remaining space distributed among existing columns
  - Columns expand when fewer items
  - Better for flexible layouts
  */
}

.grid-auto-fill {
  display: grid;
  gap: 20px;

  /* 📊 AUTO-FILL - Maintains column count */
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  /*
  🔍 auto-fill explanation:
  - Creates as many columns as can fit
  - Empty columns remain in the layout
  - Consistent column count regardless of content
  - Better for uniform grids
  */
}

/*
🔍 Auto-fit vs Auto-fill comparison:
AUTO-FIT: [Item] [Item] [    Expanded Items    ]
AUTO-FILL: [Item] [Item] [Empty] [Empty] [Empty]
*/

/* 🎯 MINMAX FUNCTION - Flexible sizing */
.grid-minmax {
  display: grid;
  gap: 15px;

  grid-template-columns:
    minmax(200px, 300px) /* 📏 Column 1: 200px to 300px */
    minmax(150px, 1fr) /* 📈 Column 2: min 150px, grows */
    minmax(auto, 200px); /* 🔄 Column 3: content-based to 200px */

  grid-template-rows:
    minmax(100px, auto) /* 📏 Row 1: min 100px, grows with content */
    minmax(50px, 1fr); /* 📈 Row 2: min 50px, fills remaining space */
}

/*
🔍 Minmax function explanation:
- Sets minimum and maximum track sizes
- First value: minimum size
- Second value: maximum size
- auto: adapts to content size
- fr: fraction of available space
- Prevents content overflow and layout breaks
*/

/* 🌊 IMPLICIT GRID - Auto-generated tracks */
.grid-implicit {
  display: grid;
  grid-template-columns: repeat(3, 200px); /* 📊 3 explicit columns */
  grid-template-rows: repeat(2, 100px); /* 📏 2 explicit rows */

  /* 🎯 IMPLICIT TRACK SIZING */
  grid-auto-columns: 150px; /* 📊 Auto-generated column size */
  grid-auto-rows: 80px; /* 📏 Auto-generated row size */

  /* 🔄 IMPLICIT GRID FLOW */
  grid-auto-flow: row; /* ⬇️ Fill rows first (default) */
  /* grid-auto-flow: column; */ /* ➡️ Fill columns first */
  /* grid-auto-flow: row dense; */ /* ⬇️ Fill gaps in row direction */
  /* grid-auto-flow: column dense; */ /* ➡️ Fill gaps in column direction */

  gap: 10px;
}

/*
🔍 Implicit grid explanation:
- Grid automatically creates tracks for extra items
- grid-auto-columns/rows: size auto-generated tracks
- grid-auto-flow: controls placement direction
- dense: fills gaps with smaller items
- Essential for dynamic content layouts
*/

/* 🎨 DENSE PACKING - Efficient space usage */
.grid-dense {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-flow: row dense; /* 🔄 Fill gaps efficiently */
  gap: 10px;
}

.dense-item-large {
  grid-column: span 2; /* 📏 Takes 2 columns */
  grid-row: span 2; /* 📏 Takes 2 rows */
  background: #e74c3c; /* 🔴 Red background */
}

.dense-item-small {
  /* Auto-placement with dense packing */
  background: #3498db; /* 🔵 Blue background */
}

/*
🔍 Dense packing explanation:
- Fills gaps left by larger items
- Items may appear out of source order
- Better space utilization
- Use with caution for accessibility
- Great for masonry-like layouts
*/
```

### **📱 Responsive Grid Patterns**

```css
/* 🏗️ RESPONSIVE CARD GRID - Mobile-first approach */
.responsive-card-grid {
  display: grid;
  gap: 1.5rem; /* 🌌 Base gap for mobile */
  padding: 1rem; /* 📦 Container padding */

  /* 📱 MOBILE FIRST - Single column */
  grid-template-columns: 1fr; /* 📱 Full width on mobile */

  /* 📱💻 TABLET - Two columns */
  @media (min-width: 768px) {
    grid-template-columns: repeat(2, 1fr); /* 📊 2 columns on tablets */
    gap: 2rem; /* 🌌 Increased gap */
    padding: 1.5rem; /* 📦 More padding */
  }

  /* 💻 DESKTOP - Three columns */
  @media (min-width: 1024px) {
    grid-template-columns: repeat(3, 1fr); /* 📊 3 columns on desktop */
    gap: 2.5rem; /* 🌌 Larger gap */
    padding: 2rem; /* 📦 Maximum padding */
  }

  /* 🖥️ LARGE DESKTOP - Four columns */
  @media (min-width: 1400px) {
    grid-template-columns: repeat(4, 1fr); /* 📊 4 columns on large screens */
    max-width: 1600px; /* 📏 Maximum container width */
    margin: 0 auto; /* 🎯 Center container */
  }
}

/*
🔍 Responsive grid explanation:
- Mobile-first approach for better performance
- Breakpoints based on content, not devices
- Progressive enhancement with media queries
- Flexible gap and padding adjustments
- Maximum width prevents over-stretching
*/

/* 🎯 ADAPTIVE GRID - Container query approach */
.adaptive-grid {
  display: grid;
  gap: clamp(1rem, 4vw, 2.5rem); /* 🌊 Fluid gap sizing */

  /* 🔄 DYNAMIC COLUMNS with auto-fit */
  grid-template-columns: repeat(
    auto-fit,
    minmax(clamp(250px, 30vw, 350px), /* 📏 Fluid column width */ 1fr)
  );
}

/*
🔍 Adaptive grid explanation:
- clamp(): fluid sizing between min and max
- vw units: viewport-width relative sizing
- Container adapts to available space
- No media queries needed for basic responsiveness
- Modern CSS approach for 2021+
*/

/* 📰 MAGAZINE LAYOUT - Complex responsive grid */
.magazine-layout {
  display: grid;
  gap: 1rem;

  /* 📱 MOBILE LAYOUT */
  grid-template-areas:
    "hero"
    "article1"
    "article2"
    "article3"
    "sidebar"
    "footer";

  /* 📱💻 TABLET LAYOUT */
  @media (min-width: 768px) {
    grid-template-columns: 2fr 1fr; /* 📊 Main content + sidebar */
    grid-template-areas:
      "hero     hero"
      "article1 sidebar"
      "article2 sidebar"
      "article3 sidebar"
      "footer   footer";
  }

  /* 💻 DESKTOP LAYOUT */
  @media (min-width: 1024px) {
    grid-template-columns: 1fr 2fr 1fr; /* 📊 3-column layout */
    grid-template-areas:
      "header   hero     sidebar"
      "article1 hero     sidebar"
      "article2 article3 sidebar"
      "footer   footer   footer";
  }
}

/* 🎯 MAGAZINE GRID ITEMS */
.hero-section {
  grid-area: hero;
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  border-radius: 8px;
}

.article-1 {
  grid-area: article1;
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 6px;
}

.article-2 {
  grid-area: article2;
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 6px;
}

.article-3 {
  grid-area: article3;
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 6px;
}

.sidebar-content {
  grid-area: sidebar;
  background: #e9ecef;
  padding: 1.5rem;
  border-radius: 6px;
}

.footer-content {
  grid-area: footer;
  background: #343a40;
  color: white;
  padding: 1.5rem;
  border-radius: 6px;
}

/*
🔍 Magazine layout explanation:
- Complex multi-breakpoint grid areas
- Different layouts for each screen size
- Named areas make layout intentions clear
- Progressive enhancement from mobile to desktop
- Real-world responsive design pattern
*/
```

### **🔄 Layout Combinations - Flexbox vs Grid**

```css
/* 🤝 NESTED LAYOUTS - Grid + Flexbox combination */
.hybrid-layout {
  /* 🔲 OUTER GRID - Page structure */
  display: grid;
  grid-template-columns: 250px 1fr; /* 📊 Sidebar + main content */
  grid-template-rows: auto 1fr auto; /* 📏 Header + content + footer */
  grid-template-areas:
    "sidebar header"
    "sidebar main"
    "sidebar footer";
  min-height: 100vh; /* 📏 Full viewport height */
  gap: 1rem;
}

.header-area {
  grid-area: header;
  /* 🧩 INNER FLEXBOX - Header content */
  display: flex; /* 📦 Horizontal header layout */
  justify-content: space-between; /* 📏 Logo left, nav right */
  align-items: center; /* 📐 Vertical centering */
  padding: 1rem 2rem; /* 📦 Header padding */
  background: #2c3e50; /* 🎨 Dark header */
  color: white; /* 🔤 White text */
}

.sidebar-area {
  grid-area: sidebar;
  /* 🧩 INNER FLEXBOX - Sidebar navigation */
  display: flex; /* 📦 Vertical sidebar layout */
  flex-direction: column; /* 📐 Stack navigation items */
  padding: 1.5rem; /* 📦 Sidebar padding */
  background: #34495e; /* 🎨 Sidebar color */
  color: white; /* 🔤 White text */
}

.main-area {
  grid-area: main;
  /* 🧩 INNER GRID - Content layout */
  display: grid; /* 🔲 Grid for main content */
  grid-template-columns: repeat(
    auto-fit,
    minmax(300px, 1fr)
  ); /* 📊 Responsive cards */
  gap: 2rem; /* 🌌 Card spacing */
  padding: 2rem; /* 📦 Content padding */
}

/*
🔍 Layout combination explanation:
- Outer grid: page structure (header, sidebar, main, footer)
- Inner flexbox: component-level layouts (navigation, headers)
- Inner grid: content arrangements (cards, articles)
- Each layout method used for its strengths
- Maximum flexibility and maintainability
*/

/* 🎯 DECISION MATRIX - When to use what */

/* 
✅ USE FLEXBOX WHEN:
- One-dimensional layouts (row or column)
- Content-driven sizing (items determine layout)
- Navigation bars, button groups
- Centering content
- Equal height columns
- Component-level layouts

Examples:
.flexbox-use-cases {
  // Navigation menus
  .navbar { display: flex; justify-content: space-between; }
  
  // Button groups  
  .button-group { display: flex; gap: 0.5rem; }
  
  // Card content (image, text, button)
  .card { display: flex; flex-direction: column; }
  
  // Equal height sidebar items
  .sidebar-nav { display: flex; flex-direction: column; }
}
*/

/*
✅ USE GRID WHEN:
- Two-dimensional layouts (rows and columns)
- Layout-driven sizing (you define the structure)
- Page layouts, card grids
- Overlapping elements
- Complex responsive patterns
- Container-level layouts

Examples:
.grid-use-cases {
  // Page layout
  .page-layout { 
    display: grid; 
    grid-template-areas: "header header" "sidebar main" "footer footer";
  }
  
  // Photo gallery
  .photo-grid { 
    display: grid; 
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); 
  }
  
  // Dashboard layout
  .dashboard { 
    display: grid; 
    grid-template-columns: repeat(12, 1fr); 
  }
}
*/

/* 🎨 REAL-WORLD DASHBOARD EXAMPLE */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr; /* 📊 Sidebar + main */
  grid-template-rows: 60px 1fr; /* 📏 Header + content */
  grid-template-areas:
    "sidebar-header header"
    "sidebar       main";
  height: 100vh; /* 📏 Full viewport */
  gap: 1px; /* 🌌 Minimal gaps */
  background: #f8f9fa; /* 🎨 Light background */
}

.dashboard-header {
  grid-area: header;
  /* 🧩 FLEXBOX for header content */
  display: flex; /* 📦 Horizontal layout */
  justify-content: space-between; /* 📏 Space distribution */
  align-items: center; /* 📐 Vertical center */
  padding: 0 2rem; /* 📦 Horizontal padding */
  background: white; /* 🎨 White background */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* 🌫️ Subtle shadow */
}

.dashboard-sidebar {
  grid-area: sidebar;
  /* 🧩 FLEXBOX for navigation */
  display: flex; /* 📦 Vertical layout */
  flex-direction: column; /* 📐 Stack items */
  background: #2c3e50; /* 🎨 Dark sidebar */
  color: white; /* 🔤 White text */
}

.dashboard-main {
  grid-area: main;
  /* 🧩 GRID for widget layout */
  display: grid; /* 🔲 Widget grid */
  grid-template-columns: repeat(
    auto-fit,
    minmax(320px, 1fr)
  ); /* 📊 Responsive widgets */
  gap: 1.5rem; /* 🌌 Widget spacing */
  padding: 1.5rem; /* 📦 Content padding */
  overflow-y: auto; /* 📜 Scrollable content */
}

.widget {
  /* 🧩 FLEXBOX for widget content */
  display: flex; /* 📦 Flexible content */
  flex-direction: column; /* 📐 Vertical stacking */
  background: white; /* 🎨 White widget background */
  border-radius: 8px; /* 🔄 Rounded corners */
  padding: 1.5rem; /* 📦 Widget padding */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1); /* 🌫️ Widget shadow */
  height: fit-content; /* 📏 Content-based height */
}

.widget-header {
  /* 🧩 FLEXBOX for widget header */
  display: flex; /* 📦 Horizontal header */
  justify-content: space-between; /* 📏 Title left, actions right */
  align-items: center; /* 📐 Vertical alignment */
  margin-bottom: 1rem; /* 🌌 Header spacing */
  padding-bottom: 0.75rem; /* 📦 Bottom padding */
  border-bottom: 1px solid #e9ecef; /* 🖼️ Header border */
}

/*
🔍 Dashboard layout explanation:
- Grid: Overall page structure and widget positioning
- Flexbox: Component-level layouts and content alignment
- Responsive widgets with auto-fit and minmax
- Scrollable main area for long content
- Professional dashboard design pattern
*/
```

---

## ⚡ Performance Optimization {#performance-optimization}

CSS performance directly impacts user experience. Understanding rendering optimization and browser behavior is crucial for senior developers.

### **🚀 CSS Rendering Performance**

```css
/* ⚡ HARDWARE ACCELERATION - GPU optimization */
.optimized-animations {
  /* ✅ PREFERRED PROPERTIES for animations */
  transform: translateX(0); /* 🚀 GPU accelerated */
  opacity: 1; /* 🚀 Compositor layer */
  filter: blur(0); /* 🚀 GPU accelerated */

  /* 🎯 PERFORMANCE HINTS */
  will-change: transform, opacity; /* 🔍 Browser optimization hint */

  /* ⚡ SMOOTH TRANSITIONS */
  transition: transform 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94), opacity 0.3s
      ease-out;

  /* 🏗️ LAYER CREATION */
  transform-style: preserve-3d; /* 🌍 3D context */
  backface-visibility: hidden; /* 🚫 Hide backfaces */
}

.optimized-animations:hover {
  transform: translateX(10px) scale(1.05); /* ✅ Transform instead of left/width */
  opacity: 0.9; /* ✅ Opacity instead of background changes */
}

/*
🔍 Performance optimization explanation:
- transform and opacity are composited properties
- GPU acceleration avoids main thread blocking
- will-change hints help browser prepare optimizations
- Avoid animating layout properties (width, height, top, left)
- Use cubic-bezier for smooth, natural animations
*/

/* 🚫 PERFORMANCE ANTI-PATTERNS - What to avoid */
.performance-killers {
  /* ❌ AVOID ANIMATING THESE PROPERTIES */
  transition: width 0.3s, /* 🐌 Causes layout recalculation */ height 0.3s,
    /* 🐌 Triggers reflow */ top 0.3s, /* 🐌 Forces layout updates */ left 0.3s,
    /* 🐌 Expensive positioning */ margin 0.3s, /* 🐌 Layout-affecting property */
      padding 0.3s; /* 🐌 Changes box dimensions */

  /* ❌ EXPENSIVE PSEUDO-SELECTORS */
  /* :nth-child(n+3):nth-child(odd) */ /* 🐌 Complex calculations */
  /* :not(:nth-child(2n+1)) */ /* 🐌 Double negation */

  /* ❌ UNIVERSAL SELECTORS */
  /* * { box-sizing: border-box; } */ /* 🐌 Matches every element */
}

/*
🔍 Performance anti-patterns explanation:
- Layout properties force expensive recalculations
- Complex selectors slow down style matching
- Universal selectors impact every DOM element
- Multiple layout-affecting animations compound issues
*/

/* ✅ OPTIMIZED ALTERNATIVES */
.performance-optimized {
  /* ✅ USE TRANSFORMS INSTEAD OF POSITIONING */
  transform: translateX(var(--offset, 0px)); /* 🚀 GPU accelerated */

  /* ✅ USE OPACITY INSTEAD OF DISPLAY/VISIBILITY */
  opacity: var(--visibility, 1); /* 🚀 Smooth show/hide */

  /* ✅ USE SCALE INSTEAD OF WIDTH/HEIGHT */
  transform: scale(var(--size, 1)); /* 🚀 Size changes without layout */

  /* ✅ EFFICIENT SELECTORS */
  /* .specific-class instead of complex selectors */
  /* Use classes instead of attribute selectors when possible */
}

/*
🔍 Optimized alternatives explanation:
- CSS custom properties enable dynamic values
- Transform-based animations are consistently fast
- Specific selectors are faster than complex ones
- Predictable performance across devices
*/
```

---

## 🧠 Tricky Interview Questions {#interview-questions}

Senior-level CSS interview questions with detailed explanations and practical examples.

### **💡 Advanced CSS Concepts**

```css
/* ❓ QUESTION 1: Explain CSS specificity calculation */
/*
🎯 ANSWER: CSS specificity is calculated using four categories:

SPECIFICITY FORMULA: a-b-c-d
a = Inline styles (style="...")     = 1000 points
b = IDs (#header)                   = 100 points  
c = Classes, attributes, pseudo     = 10 points
d = Elements and pseudo-elements    = 1 point

EXAMPLES:
*/

/* Specificity: 0-0-1-1 = 11 points */
.button:hover {
}

/* Specificity: 0-1-0-0 = 100 points */
#header {
}

/* Specificity: 0-1-2-1 = 121 points */
#header .nav .item {
}

/* Specificity: 1-0-0-0 = 1000 points */
/* style="color: red;" */

/*
🔍 IMPORTANT RULES:
- Higher specificity wins
- If equal, last rule wins (cascade)
- !important overrides but avoid using it
- Universal selector (*) has 0 specificity
- :not() doesn't count, but its arguments do
*/

/* ❓ QUESTION 2: What happens with this CSS? */
.container {
  width: 200px;
  height: 200px;
  overflow: hidden;
}

.child {
  width: 150px;
  height: 150px;
  margin: 50px;
  background: red;
}

/*
🎯 ANSWER: Margin collapse and overflow interaction

RESULT:
- Child element will have 25px visible on right and bottom
- Top and left margins collapse with container
- Container acts as Block Formatting Context due to overflow: hidden
- Child's margins don't extend outside container

KEY CONCEPTS:
- Margin collapse stops at formatting context boundaries
- overflow: hidden creates Block Formatting Context
- Margins can extend outside parent in normal flow
*/

/* ❓ QUESTION 3: Explain this flexbox behavior */
.flex-container {
  display: flex;
  width: 300px;
}

.flex-item {
  flex: 1 1 100px; /* grow: 1, shrink: 1, basis: 100px */
  min-width: 80px;
}

/*
🎯 ANSWER: Flexbox sizing with constraints

WITH 3 ITEMS:
- Initial basis: 3 × 100px = 300px (fits exactly)
- Available space: 300px - 300px = 0px
- Each item gets: 100px (basis size)

WITH 4 ITEMS:
- Initial basis: 4 × 100px = 400px (overflow by 100px)
- Shrinking space: 100px distributed among 4 items = 25px each
- Each item gets: 100px - 25px = 75px
- BUT min-width: 80px prevents shrinking below 80px
- Actual result: Items will be 80px each, container overflows

KEY CONCEPTS:
- flex-basis sets initial size before grow/shrink
- min-width/max-width constraints override flex sizing
- Items can overflow container if constraints prevent shrinking
*/

/* ❓ QUESTION 4: CSS Grid implicit behavior */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-template-rows: repeat(2, 100px);
}

.grid-item:nth-child(7) {
  grid-column: 1 / 4; /* Spans all 3 columns */
}

/*
🎯 ANSWER: Implicit grid creation

RESULT:
- Grid has 3 explicit columns, 2 explicit rows
- First 6 items fill the explicit grid
- 7th item needs to span 3 columns but row 3 doesn't exist
- Grid automatically creates implicit row 3
- 7th item spans entire width of row 3
- Implicit rows use auto sizing (content-based)

LAYOUT:
Row 1: [Item1] [Item2] [Item3]
Row 2: [Item4] [Item5] [Item6]
Row 3: [    Item7 spans all    ] (implicit row)

KEY CONCEPTS:
- Grid auto-creates tracks for positioned items
- grid-auto-rows controls implicit row sizing
- Positioned items can force grid expansion
*/

/* ❓ QUESTION 5: Complex selector behavior */
.parent > .child + .sibling ~ .target {
  color: red;
}

/*
🎯 ANSWER: Combinator chain explanation

BREAKDOWN:
.parent > .child        = Direct child with class "child"
.child + .sibling      = Immediate sibling after .child with class "sibling"  
.sibling ~ .target     = Any later sibling after .sibling with class "target"

HTML THAT MATCHES:
<div class="parent">
  <div class="child"></div>
  <div class="sibling"></div>
  <div class="other"></div>
  <div class="target"></div>  <!-- This gets red text -->
</div>

HTML THAT DOESN'T MATCH:
<div class="parent">
  <div class="child"></div>
  <div class="other"></div>    <!-- Breaks the chain -->
  <div class="sibling"></div>
  <div class="target"></div>   <!-- Won't match -->
</div>

KEY CONCEPTS:
- > = direct child
- + = immediate adjacent sibling  
- ~ = general sibling
- Chain must be unbroken for match
*/

/* ❓ QUESTION 6: Optimize this inefficient CSS */
/* 🚫 PROBLEMATIC CODE */
.inefficient {
  /* ❌ Expensive animations */
  transition: width 0.3s, height 0.3s, left 0.3s, top 0.3s;

  /* ❌ Complex selectors */
  ul li:nth-child(odd):not(:first-child):not(:last-child) a:hover {
    color: red;
  }

  /* ❌ Repeated styles */
  .button-primary {
    background: #007bff;
    padding: 10px 15px;
    border: none;
  }
  .button-secondary {
    background: #6c757d;
    padding: 10px 15px;
    border: none;
  }
  .button-success {
    background: #28a745;
    padding: 10px 15px;
    border: none;
  }
}

/* ✅ OPTIMIZED VERSION */
.optimized {
  /* ✅ GPU-accelerated animations */
  transition: transform 0.3s ease-out, opacity 0.3s ease-out;
  will-change: transform, opacity;

  /* ✅ Simplified selectors with specific classes */
  .nav-link-alternate:hover {
    color: red;
  }

  /* ✅ DRY principles with SCSS */
  .button-base {
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s ease-out;

    &--primary {
      background: #007bff;
    }
    &--secondary {
      background: #6c757d;
    }
    &--success {
      background: #28a745;
    }
  }
}

/*
🎯 OPTIMIZATION EXPLANATION:
- Replace layout-affecting animations with transforms
- Simplify selectors for better performance
- Use BEM methodology for clear naming
- Leverage SCSS for maintainable code
- Add performance hints (will-change)
*/
```

### **🎯 Real-World Scenario Questions**

```css
/* ❓ QUESTION 7: Create a responsive card grid that maintains aspect ratio */

/*
🎯 ANSWER: CSS Grid with aspect-ratio property
*/

.responsive-card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  padding: 2rem;
}

.card {
  /* 📐 ASPECT RATIO CONTROL */
  aspect-ratio: 3 / 4; /* 🖼️ 3:4 aspect ratio (portrait) */

  /* 🎨 CARD STYLING */
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  overflow: hidden;

  /* 📦 INTERNAL LAYOUT */
  display: flex;
  flex-direction: column;
}

.card-image {
  /* 🖼️ IMAGE CONTAINER */
  flex: 0 0 60%; /* 📏 60% of card height */
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.card-content {
  /* 📝 CONTENT AREA */
  flex: 1; /* 📈 Remaining space */
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

/*
🔍 KEY TECHNIQUES:
- aspect-ratio: maintains consistent card proportions
- auto-fit: responsive column count
- minmax(): flexible sizing with constraints
- Nested flexbox: internal card layout control
*/

/* ❓ QUESTION 8: Implement a CSS-only accordion */

/*
🎯 ANSWER: Using checkbox hack for state management
*/

.accordion {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.accordion-item {
  border-bottom: 1px solid #ddd;
}

.accordion-item:last-child {
  border-bottom: none;
}

/* 🔒 HIDDEN CHECKBOX for state */
.accordion-toggle {
  display: none; /* 🚫 Hide checkbox */
}

/* 📋 ACCORDION HEADER */
.accordion-header {
  background: #f8f9fa;
  padding: 1rem 1.5rem;
  cursor: pointer;
  user-select: none;
  position: relative;

  /* ➡️ ARROW INDICATOR */
  &::after {
    content: "▶";
    position: absolute;
    right: 1.5rem;
    transition: transform 0.3s ease;
  }
}

/* 📄 ACCORDION CONTENT */
.accordion-content {
  max-height: 0; /* 📏 Start collapsed */
  overflow: hidden; /* 🚫 Hide overflow */
  padding: 0 1.5rem; /* 📦 Horizontal padding only */
  background: white;
  transition: max-height 0.3s ease-out, padding 0.3s ease-out;
}

/* ✅ EXPANDED STATE */
.accordion-toggle:checked + .accordion-header::after {
  transform: rotate(90deg); /* 🔄 Rotate arrow */
}

.accordion-toggle:checked ~ .accordion-content {
  max-height: 500px; /* 📏 Expand (generous max-height) */
  padding: 1.5rem; /* 📦 Full padding */
}

/*
🔍 ACCORDION TECHNIQUE:
- Checkbox hack: CSS-only state management
- max-height transition: smooth expand/collapse
- Adjacent sibling selector: connect checkbox to content
- Transform: smooth arrow rotation
- Overflow hidden: clean content clipping
*/

/* ❓ QUESTION 9: Create a masonry layout with CSS Grid */

/*
🎯 ANSWER: Grid with subgrid and dense packing
*/

.masonry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  grid-auto-rows: 20px; /* 📏 Small row units for flexibility */
  grid-auto-flow: row dense; /* 🔄 Dense packing algorithm */
  gap: 1rem;
}

.masonry-item {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 1rem;

  /* 📏 DYNAMIC ROW SPANNING based on content height */
  &.small {
    grid-row: span 8;
  } /* 📏 8 × 20px = 160px */
  &.medium {
    grid-row: span 12;
  } /* 📏 12 × 20px = 240px */
  &.large {
    grid-row: span 16;
  } /* 📏 16 × 20px = 320px */
  &.extra-large {
    grid-row: span 20;
  } /* 📏 20 × 20px = 400px */
}

/* 📱 RESPONSIVE MASONRY */
@media (max-width: 768px) {
  .masonry-grid {
    grid-template-columns: 1fr; /* 📱 Single column on mobile */
  }

  .masonry-item {
    /* 📏 Reset row spans for mobile */
    grid-row: span auto;
  }
}

/*
🔍 MASONRY TECHNIQUES:
- Small row units: flexible height control
- Span calculations: content-based sizing
- Dense packing: fills gaps automatically
- Responsive: single column on mobile
- Class-based: height variants for different content
*/
```

### **🏆 Angular-Specific CSS Questions**

```css
/* ❓ QUESTION 10: How does Angular handle CSS encapsulation? */

/*
🎯 ANSWER: ViewEncapsulation strategies

1. ViewEncapsulation.Emulated (default):
   - Angular adds unique attributes to components
   - CSS selectors are scoped to component
   - Prevents style leakage between components

Example generated CSS:
.my-component[_ngcontent-abc123] {
  background: red;
}

2. ViewEncapsulation.ShadowDom:
   - Uses native Shadow DOM
   - True encapsulation at browser level
   - Styles completely isolated

3. ViewEncapsulation.None:
   - No encapsulation
   - Styles become global
   - Use carefully to avoid conflicts
*/

// Component with encapsulation
@Component({
  selector: 'app-card',
  encapsulation: ViewEncapsulation.Emulated, // Default
  styles: [`
    .card {
      background: white;         // Scoped to this component
      padding: 1rem;
    }

    :host {
      display: block;            // Style the host element
      margin: 1rem;
    }

    :host(.highlighted) {
      border: 2px solid blue;    // Conditional host styling
    }

    ::ng-deep .global-override {
      color: red;                // Penetrates child components
    }
  `]
})

/*
🔍 ANGULAR CSS FEATURES:
- :host: styles the component host element
- :host(): conditional host styling
- ::ng-deep: deep selector (deprecated, use global styles instead)
- Component-scoped styles prevent conflicts
*/

/* ❓ QUESTION 11: Optimize Angular component styles for performance */

/*
🎯 ANSWER: Angular-specific optimization strategies
*/

// ✅ OPTIMIZED COMPONENT ARCHITECTURE
@Component({
  selector: 'app-optimized',
  changeDetection: ChangeDetectionStrategy.OnPush,
  styles: [`
    :host {
      display: block;
      contain: layout style;     // CSS containment for performance
    }

    .component-container {
      /* ✅ Use CSS custom properties for dynamic values */
      background: var(--component-bg, white);
      padding: var(--component-padding, 1rem);

      /* ✅ Avoid deep selectors */
      /* Use specific component classes instead */
    }

    .optimized-animation {
      /* ✅ Hardware-accelerated properties only */
      transition: transform 0.3s ease-out;
      will-change: transform;
    }

    /* ✅ Mobile-first responsive design */
    @media (min-width: 768px) {
      .component-container {
        padding: var(--component-padding-lg, 2rem);
      }
    }
  `]
})

/*
🔍 ANGULAR OPTIMIZATION TECHNIQUES:
- CSS containment: isolates component rendering
- Custom properties: dynamic theming without rebuilds
- OnPush change detection: reduces unnecessary checks
- Mobile-first: progressive enhancement
- Hardware acceleration: smooth animations
- Specific selectors: avoid deep piercing
*/
```

## 🎯 Summary & Conclusion

This comprehensive CSS3 & SCSS guide covers:

**✅ Complete Technical Coverage:**

- **CSS Fundamentals** - Box model, positioning, transforms, transitions
- **Modern Layout Systems** - Flexbox, CSS Grid, responsive design
- **SCSS/Angular Integration** - Variables, mixins, component architecture
- **Performance Optimization** - Hardware acceleration, efficient selectors
- **Real-World Scenarios** - Complex layouts, debugging, browser compatibility
- **Interview Preparation** - Advanced concepts, tricky questions, practical solutions

**✅ Production-Ready Knowledge:**

- **2,000+ lines** of detailed code examples
- **Line-by-line explanations** with practical theory
- **Performance best practices** for enterprise applications
- **Responsive design patterns** for all device types
- **Debugging techniques** and common gotcha solutions
- **Interview questions** with comprehensive answers

**🚀 For Senior Frontend Developers:**
This guide prepares you for complex technical interviews, challenging project requirements, and architectural decisions in modern frontend development. Every technique is explained with both theory and practical implementation for immediate application in professional projects.

**Total Parts Completed: 6/6** ✅
