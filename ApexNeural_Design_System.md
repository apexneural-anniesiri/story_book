# ApexNeural Design System
## Complete Design System Documentation

**Brand Name:** ApexNeural  
**Last Updated:** January 2026  
**Version:** 1.0

---

## Table of Contents

1. [Color Palette](#color-palette)
2. [Typography](#typography)
3. [Spacing System](#spacing-system)
4. [Components](#components)
5. [Alerts](#alerts)
6. [Icons](#icons)

---

## Color Palette

### Theme Overview

The ApexNeural Design System uses a carefully curated color palette designed for both light and dark themes, ensuring accessibility and visual consistency across all digital products.

### Primary Colors

#### Primary - Light Theme
- **Hex:** #E64C66
- **RGB:** 230, 76, 102
- **HSL:** 348°, 76%, 60%
- **Usage:** Primary actions, main call-to-action buttons, key UI elements
- **Accessibility:** Use with white text (contrast ratio: 4.5:1)

#### Primary - Dark Theme
- **Hex:** #931F33
- **RGB:** 147, 31, 51
- **HSL:** 348°, 65%, 35%
- **Usage:** Primary actions in dark environments, emphasis states
- **Accessibility:** Use with white text (contrast ratio: 7.1:1)

### Secondary Colors

#### Secondary - Light Theme
- **Hex:** #F4F4F4
- **RGB:** 244, 244, 244
- **HSL:** 0°, 0%, 96%
- **Usage:** Backgrounds, subtle elements, surfaces
- **Accessibility:** Use with dark text (contrast ratio: 14:1)

#### Secondary - Dark Theme
- **Hex:** #221E1C
- **RGB:** 34, 30, 28
- **HSL:** 20°, 9%, 12%
- **Usage:** Dark backgrounds, primary text background
- **Accessibility:** Use with white/light text (contrast ratio: 10.5:1)

### Primary Color Gradient Scale

A complete 11-step gradient scale from white to black for versatile color application:

| Level | Hex Code | RGB Values | Usage |
|-------|----------|-----------|-------|
| 0 | #F9E1E5 | 249, 225, 229 | Lightest backgrounds, disabled states |
| 0.5 | #EFCED3 | 239, 206, 211 | Light backgrounds, subtle hover states |
| 1 | #E5BAC1 | 229, 186, 193 | Light background variants |
| 1.5 | #DAA7B0 | 218, 167, 176 | Subtle interactive states |
| 2 | #D0939E | 208, 147, 158 | Light interactive states |
| 3 | #BC6D7A | 188, 109, 122 | Hover states for primary |
| 4 | #A74657 | 167, 70, 87 | Alternative primary |
| 5 | #931F33 | 147, 31, 51 | **Primary Dark Color** |
| 6 | #761929 | 118, 25, 41 | Active states, emphasis |
| 7 | #58131F | 88, 19, 31 | Dark emphasis |
| 8 | #3B0C14 | 59, 12, 20 | Very dark states |
| 8.5 | #2C090F | 44, 9, 15 | Near black, text |
| 9 | #1D060A | 29, 6, 10 | Almost black, borders |
| 9.5 | #0F0305 | 15, 3, 5 | Darkest backgrounds |
| 10 | #000000 | 0, 0, 0 | Pure black |

### Semantic Colors

Semantic colors provide immediate visual feedback for different states and messages. Each color is carefully chosen for accessibility and clarity.

#### Success Color
- **Hex:** #10B981
- **RGB:** 16, 185, 129
- **HSL:** 160°, 84%, 39%
- **Usage:** Positive confirmations, success messages, completed states
- **Example:** "Form submitted successfully", checkmarks, positive indicators
- **WCAG Contrast:** 4.5:1+ with white background

#### Warning Color
- **Hex:** #F59E0B
- **RGB:** 245, 158, 11
- **HSL:** 38°, 92%, 50%
- **Usage:** Caution alerts, warnings, review needed states
- **Example:** "Please review this information carefully", warning icons
- **WCAG Contrast:** 3:1+ with dark text, 4.5:1+ with white text

#### Error Color
- **Hex:** #EF4444
- **RGB:** 239, 68, 68
- **HSL:** 0°, 91%, 60%
- **Usage:** Error states, failures, destructive actions
- **Example:** "Something went wrong", form validation errors
- **WCAG Contrast:** 3:1+ with white background

#### Info Color
- **Hex:** #3B82F6
- **RGB:** 59, 130, 246
- **HSL:** 217°, 91%, 60%
- **Usage:** Informational messages, tips, helpful content
- **Example:** "This is an informational message", helpful hints
- **WCAG Contrast:** 4.5:1+ with white background

### Usage Guidelines

#### When to Use Each Color

**Primary Color (#931F33 / #E64C66)**
- Main action buttons
- Links and interactive elements
- Key visual hierarchy elements
- Focus indicators
- Selected states

**Secondary Color (#F4F4F4 / #221E1C)**
- Default backgrounds
- Neutral surfaces
- Container backgrounds
- Disabled states

**Success Color (#10B981)**
- Positive confirmations
- Success messages
- Checkmarks and validated states
- Green indicators

**Warning Color (#F59E0B)**
- Non-critical alerts
- Items requiring attention
- Cautionary messages
- Yellow indicators

**Error Color (#EF4444)**
- Error messages
- Invalid form states
- Destructive actions
- Red indicators

**Info Color (#3B82F6)**
- Informational messages
- Helpful tips
- Blue indicators
- Educational content

---

## Typography

### Font Families

#### Playfair Display
- **Usage:** Display styles, large headings (H1-H2)
- **Classification:** Serif, elegant
- **Weight:** Medium to Bold
- **Best For:** Headlines, Display text, brand statements

#### DM Sans
- **Usage:** Body text, smaller headings (H3-H6)
- **Classification:** Geometric sans-serif, modern
- **Weight:** Regular to Semibold
- **Best For:** Body copy, subheadings, readable content

#### JetBrains Mono
- **Usage:** Labels, tags, code
- **Classification:** Monospace, technical
- **Weight:** Medium
- **Best For:** Code snippets, technical labels, captions

### Type Scale

A complete type hierarchy with specific sizes, weights, and line heights for every style:

#### Display
- **Font:** Playfair Display
- **Size:** 72px
- **Weight:** Bold (700)
- **Line Height:** 1.1 (79px)
- **Letter Spacing:** 0
- **Usage:** Large promotional text, main page titles
- **Example:** Hero sections, main brand statements

#### Heading 1 (H1)
- **Font:** Playfair Display
- **Size:** 48px
- **Weight:** Medium (500)
- **Line Height:** 1.2 (58px)
- **Letter Spacing:** 0
- **Usage:** Primary page heading
- **Example:** Page titles, major section headers

#### Heading 2 (H2)
- **Font:** Playfair Display
- **Size:** 36px
- **Weight:** Medium (500)
- **Line Height:** 1.25 (45px)
- **Letter Spacing:** 0
- **Usage:** Secondary headings, major sections
- **Example:** Section headers, large content blocks

#### Heading 3 (H3)
- **Font:** DM Sans
- **Size:** 28px
- **Weight:** Semibold (600)
- **Line Height:** 1.3 (36px)
- **Letter Spacing:** 0
- **Usage:** Tertiary headings, subsections
- **Example:** Card titles, subsection headers

#### Heading 4 (H4)
- **Font:** DM Sans
- **Size:** 24px
- **Weight:** Semibold (600)
- **Line Height:** 1.35 (32px)
- **Letter Spacing:** 0
- **Usage:** Component headings
- **Example:** Modal titles, form section headers

#### Heading 5 (H5)
- **Font:** DM Sans
- **Size:** 20px
- **Weight:** Medium (500)
- **Line Height:** 1.4 (28px)
- **Letter Spacing:** 0
- **Usage:** Smaller headings
- **Example:** Input labels, component titles

#### Heading 6 (H6)
- **Font:** DM Sans
- **Size:** 18px
- **Weight:** Semibold (600)
- **Line Height:** 1.4 (25px)
- **Letter Spacing:** 0
- **Usage:** Sub-headings, emphasis text
- **Example:** Emphasized labels, secondary headers

#### Body Large
- **Font:** DM Sans
- **Size:** 20px
- **Weight:** Regular (400)
- **Line Height:** 1.6 (32px)
- **Letter Spacing:** 0
- **Usage:** Large body text, introductions
- **Example:** Article introductions, emphasized body copy

#### Body Regular
- **Font:** DM Sans
- **Size:** 18px
- **Weight:** Regular (400)
- **Line Height:** 1.6 (29px)
- **Letter Spacing:** 0
- **Usage:** Primary body text
- **Example:** Paragraph text, standard content

#### Body Small
- **Font:** DM Sans
- **Size:** 14px
- **Weight:** Regular (400)
- **Line Height:** 1.5 (21px)
- **Letter Spacing:** 0
- **Usage:** Secondary body text, supporting content
- **Example:** Secondary descriptions, helper text

#### Label / Tag
- **Font:** JetBrains Mono
- **Size:** 14px
- **Weight:** Medium (500)
- **Line Height:** 1.4 (20px)
- **Letter Spacing:** 0
- **Usage:** Labels, tags, badges
- **Example:** Form labels, tag elements, badges

#### Caption
- **Font:** JetBrains Mono
- **Size:** 14px
- **Weight:** Medium (500)
- **Line Height:** 1.4 (20px)
- **Letter Spacing:** 0
- **Usage:** Small text, footnotes
- **Example:** Image captions, small labels, footnotes

#### Code / Mono
- **Font:** JetBrains Mono
- **Size:** 14px
- **Weight:** Regular (400)
- **Line Height:** 1.5 (21px)
- **Letter Spacing:** 0
- **Usage:** Code blocks, technical content
- **Example:** Code snippets, terminal output, technical documentation

### Typography Guidelines

#### Hierarchy Best Practices
1. **Use consistent sizing** - Follow the defined type scale
2. **Limit font families** - Use Playfair Display for headings, DM Sans for body
3. **Maintain line length** - 50-75 characters for optimal readability
4. **Adequate spacing** - Use line heights as defined for readability
5. **Visual weight** - Use weight variation for emphasis, not just size

#### Responsive Typography
All typography scales should adapt responsively:
- **Desktop:** Use sizes as specified
- **Tablet (640-1024px):** Reduce heading sizes by 10-15%
- **Mobile (< 640px):** Reduce heading sizes by 20-25%

#### Color Application
- **Primary Text:** Use secondary dark color (#221E1C)
- **Secondary Text:** Use neutral gray (70% opacity)
- **Disabled Text:** Use gray (50% opacity)
- **On Primary:** Use white or light text

---

## Spacing System

### Base Unit

**Base Spacing Unit:** 4px

All spacing values are multiples of 4px for consistency and flexibility.

### Spacing Scale

A comprehensive spacing scale for all layout needs:

| Token | Size | Usage |
|-------|------|-------|
| --space-4 | 4px | Tight spacing, micro interactions |
| --space-8 | 8px | Icon spacing, small gaps |
| --space-12 | 12px | Component padding, form elements |
| --space-16 | 16px | Default spacing, standard padding |
| --space-24 | 24px | Card padding, section spacing |
| --space-32 | 32px | Large gaps, prominent spacing |
| --space-48 | 48px | Section spacing, major separators |
| --space-64 | 64px | Large sections, major layout gaps |

### Spacing Guidelines

#### Component Padding
- **Small components** (badges, tags): 4-8px
- **Standard components** (buttons, inputs): 12px vertical, 16px horizontal
- **Large components** (cards, modals): 24px
- **Content areas:** 24-32px

#### Margin and Gaps
- **Tight spacing:** 4px (icon spacing, visual connections)
- **Small gaps:** 8-12px (form elements, related items)
- **Default spacing:** 16px (most spacing decisions)
- **Large gaps:** 32-48px (section separation)
- **Extra large gaps:** 64px (major layout sections)

#### Grid and Layout
- **Gutter width:** 16px (space between columns)
- **Container padding:** 16px (mobile), 24px (tablet), 32px (desktop)
- **Section margin:** 48-64px (vertical spacing between sections)
- **Max-width:** 1280px (standard content width)

---

## Components

### Buttons

#### Overview
Buttons are primary interactive elements that trigger actions. ApexNeural provides multiple button styles, sizes, and states for different use cases.

### Button Variants

#### Primary Button
- **Background Color:** #931F33 (primary)
- **Text Color:** White
- **Corner Radius:** 8px
- **Padding:** 12px vertical, 16px horizontal
- **Font:** DM Sans, 16px, Regular
- **Usage:** Main actions, primary call-to-action
- **Hover State:** Background darkens to #761929
- **Active State:** Background darkens to #58131F
- **Disabled State:** Opacity 50%, cursor disabled

**Example HTML:**
```html
<button class="btn btn-primary">Click Me</button>
```

#### Primary Button - Disabled
- **Background Color:** #931F33 (opacity: 50%)
- **Text Color:** White (opacity: 50%)
- **Corner Radius:** 8px
- **Cursor:** Not allowed
- **Interaction:** No hover effect
- **Usage:** When action is unavailable

**Example HTML:**
```html
<button class="btn btn-primary" disabled>Disabled Action</button>
```

#### Secondary Button
- **Background Color:** Transparent or #F4F4F4
- **Border:** 1px solid #931F33
- **Text Color:** #931F33
- **Corner Radius:** 8px
- **Padding:** 12px vertical, 16px horizontal
- **Font:** DM Sans, 16px, Regular
- **Usage:** Alternative actions, secondary options
- **Hover State:** Light background (#F9E1E5)
- **Active State:** Background darker
- **Disabled State:** Opacity 50%

**Example HTML:**
```html
<button class="btn btn-secondary">Secondary Action</button>
```

#### Secondary Button - Disabled
- **Background Color:** #F4F4F4
- **Border Color:** #D0939E (opacity: 50%)
- **Text Color:** #931F33 (opacity: 50%)
- **Cursor:** Not allowed
- **Interaction:** No hover effect

**Example HTML:**
```html
<button class="btn btn-secondary" disabled>Disabled</button>
```

#### Loading Button
- **State:** Shows loading indicator (spinner)
- **Icon:** Animated spinner inside button
- **Text:** Maintains button label or shows "Loading..."
- **Disabled:** Cannot be clicked while loading
- **Duration:** Until action completes

**Example HTML:**
```html
<button class="btn btn-primary is-loading">
  <span class="spinner"></span>
  Processing...
</button>
```

### Button Sizes

#### Small Button (36px)
- **Height:** 36px
- **Padding:** 8px vertical, 12px horizontal
- **Font Size:** 14px
- **Usage:** Compact spaces, density
- **Example:** List actions, form submissions

**Example HTML:**
```html
<button class="btn btn-primary btn-sm">Small Button</button>
```

#### Medium Button (44px)
- **Height:** 44px
- **Padding:** 12px vertical, 16px horizontal
- **Font Size:** 16px
- **Usage:** Standard, default size
- **Example:** Primary actions, general buttons
- **Note:** Meet minimum touch target (44x44px)

**Example HTML:**
```html
<button class="btn btn-primary btn-md">Medium Button</button>
```

#### Large Button (52px)
- **Height:** 52px
- **Padding:** 16px vertical, 20px horizontal
- **Font Size:** 18px
- **Usage:** Prominent actions, featured buttons
- **Example:** Hero section CTA, important actions

**Example HTML:**
```html
<button class="btn btn-primary btn-lg">Large Button</button>
```

### Button with Icons

#### Icon Left Button
- **Layout:** Icon on left, text on right
- **Icon Size:** 20px
- **Icon Spacing:** 8px from text
- **Alignment:** Centered vertically
- **Example:** Download, Save, Edit actions

**Example HTML:**
```html
<button class="btn btn-primary btn-with-icon">
  <icon class="icon-left" size="20"></icon>
  Icon Left Button Test
</button>
```

#### Icon Right Button
- **Layout:** Text on left, icon on right
- **Icon Size:** 20px
- **Icon Spacing:** 8px from text
- **Alignment:** Centered vertically
- **Example:** Open, External link, Arrow indicators

**Example HTML:**
```html
<button class="btn btn-primary btn-with-icon">
  Icon Right Button Test
  <icon class="icon-right" size="20"></icon>
</button>
```

#### Icon Only Button
- **Width/Height:** Match size (36px, 44px, 52px)
- **Icon Size:** 20-24px
- **Padding:** Centered
- **Accessibility:** Requires aria-label
- **Usage:** Compact space, simple actions

**Example HTML:**
```html
<button class="btn btn-primary btn-icon" aria-label="Save">
  <icon size="24"></icon>
</button>
```

### Button States

#### Default State
- **Appearance:** Base styling as defined
- **Cursor:** pointer
- **Interaction:** Ready for click

#### Hover State
- **Background Color:** Slightly darker (10-15% darker)
- **Transition:** 200ms ease
- **Cursor:** pointer
- **Accessibility:** Should be keyboard accessible

#### Active/Pressed State
- **Background Color:** 20-25% darker
- **Visual Feedback:** Subtle inset or scale effect
- **Duration:** Momentary, until release

#### Focus State
- **Outline:** 2px solid primary color
- **Outline Offset:** 2px
- **Visibility:** Must meet WCAG AA contrast
- **Accessibility:** Essential for keyboard navigation

#### Disabled State
- **Opacity:** 50%
- **Background Color:** Unchanged but dimmed
- **Cursor:** not-allowed
- **Interaction:** No hover effects, not clickable
- **Accessibility:** Marked with disabled attribute

#### Loading State
- **Content:** Replace icon or show spinner
- **Interaction:** Disabled during loading
- **Feedback:** Clear loading indication
- **Timeout:** Show loading for action duration

### Toggle Buttons

#### Overview
Toggle buttons allow users to switch between two states (on/off, selected/unselected).

#### States
- **Off/Unselected:** Base styling, lower emphasis
- **On/Selected:** Primary color background, higher emphasis
- **Hover:** Slight background change
- **Disabled:** 50% opacity

#### Usage
- Feature toggles
- Filter selections
- Mode switching
- Settings

---

## Alerts

### Alert Components

Alerts provide important messages and feedback to users. ApexNeural offers four alert types with distinct styling for quick recognition.

### Alert Types

#### Info Alert
- **Icon:** Information icon (i)
- **Background Color:** Light blue (#E0F2FE or similar)
- **Border Color:** #3B82F6
- **Text Color:** Dark gray
- **Text:** "This is an informational message"
- **Usage:** General information, helpful tips, notices
- **Dismissible:** Optional close button
- **Icon Color:** #3B82F6

**Example HTML:**
```html
<div class="alert alert-info">
  <icon class="alert-icon">info</icon>
  <span>This is an informational message</span>
  <button class="alert-close" aria-label="Close">×</button>
</div>
```

#### Success Alert
- **Icon:** Checkmark icon
- **Background Color:** Light green (#ECFDF5 or similar)
- **Border Color:** #10B981
- **Text Color:** Dark gray
- **Text:** "Your action was completed successfully"
- **Usage:** Confirmations, successful operations, positive feedback
- **Dismissible:** Optional close button
- **Icon Color:** #10B981

**Example HTML:**
```html
<div class="alert alert-success">
  <icon class="alert-icon">check</icon>
  <span>Your action was completed successfully</span>
  <button class="alert-close" aria-label="Close">×</button>
</div>
```

#### Warning Alert
- **Icon:** Warning/Alert triangle icon
- **Background Color:** Light amber (#FFFBEB or similar)
- **Border Color:** #F59E0B
- **Text Color:** Dark gray
- **Text:** "Please review this information carefully"
- **Usage:** Cautions, important notices, items needing attention
- **Dismissible:** Optional close button
- **Icon Color:** #F59E0B

**Example HTML:**
```html
<div class="alert alert-warning">
  <icon class="alert-icon">alert-triangle</icon>
  <span>Please review this information carefully</span>
  <button class="alert-close" aria-label="Close">×</button>
</div>
```

#### Error Alert
- **Icon:** X or error icon
- **Background Color:** Light red (#FEF2F2 or similar)
- **Border Color:** #EF4444
- **Text Color:** Dark gray
- **Text:** "Something went wrong, please try again"
- **Usage:** Errors, failures, critical issues
- **Dismissible:** Optional close button (usually persistent)
- **Icon Color:** #EF4444

**Example HTML:**
```html
<div class="alert alert-error">
  <icon class="alert-icon">x-circle</icon>
  <span>Something went wrong, please try again</span>
  <button class="alert-close" aria-label="Close">×</button>
</div>
```

### Alert Anatomy

#### Structure
```
┌─────────────────────────────────────┐
│ [Icon]  Message Text        [×]     │
└─────────────────────────────────────┘
```

#### Components
- **Icon:** 20-24px, colored to match type
- **Message Text:** Body Regular (18px), left-aligned
- **Close Button:** Optional, 24-32px, right-aligned
- **Background:** Color-coded by type
- **Border:** Left border or top border (2-4px)
- **Padding:** 12-16px
- **Margin:** 16px bottom (space for stacking)
- **Border Radius:** 8px

### Alert Usage Guidelines

#### When to Use Each Alert Type

**Info Alert**
- Helpful tips and suggestions
- General information notices
- Educational content
- Informational updates

**Success Alert**
- Form submission confirmations
- Action completions
- Positive operations
- Validation success

**Warning Alert**
- Non-critical issues requiring attention
- Pending actions
- Important notices
- Cautions and alerts

**Error Alert**
- Failed operations
- Validation errors
- Critical issues
- System errors

#### Best Practices
1. **Be clear and concise** - Brief, actionable messages
2. **Use appropriate type** - Match alert type to message severity
3. **Provide next steps** - Include what user should do
4. **Auto-dismiss when appropriate** - Success/info after 4-6 seconds
5. **Allow manual dismiss** - Provide close button option
6. **Accessibility** - Use role="alert" for dynamic alerts
7. **Multiple alerts** - Stack vertically, max 3-4 visible

---

## Icons

### Icon Library

ApexNeural uses a consistent, modern icon set designed for clarity and scalability across all applications.

### Icon Sizing

The following standard sizes ensure consistency and proper scaling:

#### Extra Small - 16px
- **Usage:** Inline icons, tight spaces
- **Example:** Badge icons, small indicators
- **Line Weight:** 2px

#### Small - 20px
- **Usage:** Form fields, small buttons
- **Example:** Input icons, action icons
- **Line Weight:** 2px

#### Medium - 24px (Default)
- **Usage:** Buttons, navigation
- **Example:** Button icons, toolbar icons, navigation items
- **Line Weight:** 2px

#### Large - 32px
- **Usage:** Featured content, emphasis
- **Example:** Card icons, feature highlights
- **Line Weight:** 2px

#### Extra Large - 48px
- **Usage:** Promotional, hero sections
- **Example:** Large feature icons, promotional graphics
- **Line Weight:** 2px

### Icon Properties

#### Line Weight
- **Standard:** 2px stroke width
- **Consistency:** All icons use consistent weight
- **Fill:** Icons can be outline, solid, or filled based on context

#### Style
- **Design Language:** Modern, minimal, geometric
- **Stroke Type:** Rounded caps and joins for softness
- **Visual Harmony:** Consistent with brand aesthetic
- **Scalability:** Works at all sizes from 16px to 48px+

#### Color Application
- **Primary Iconography:** Use primary color (#931F33)
- **Secondary:** Use secondary color or neutral gray
- **Semantic:** Use appropriate semantic colors (success, warning, error, info)
- **Disabled:** 50% opacity or gray color
- **On Primary:** Use white or light colors

### Icon Spacing

#### Icon with Text
- **Horizontal spacing:** 8px between icon and text
- **Vertical alignment:** Centered with text
- **Icon-first or text-first:** Both acceptable

#### Standalone Icons
- **Padding area:** 8-12px padding within container
- **Touch target:** Ensure 44x44px minimum when interactive
- **Alignment:** Center within container

### Icon Grid

All icons are designed on a 24x24px grid for medium size icons, scaling proportionally:
- **16px icons:** Designed on 16x16px grid
- **24px icons:** Designed on 24x24px grid
- **32px icons:** Designed on 32x32px grid
- **48px icons:** Designed on 48x48px grid

### Icon Categories

Common icon categories used throughout ApexNeural:

#### Navigation Icons
- Home, Back, Forward, Menu, Close, Expand, Collapse

#### Action Icons
- Add, Edit, Delete, Save, Cancel, Refresh, Share

#### Status Icons
- Check, X, Alert, Info, Help, Warning

#### Media Icons
- Download, Upload, Image, Video, File, Folder

#### Social Icons
- Facebook, Twitter, LinkedIn, Instagram, GitHub

#### UI Icons
- Settings, Profile, Notification, Search, Filter, Sort

### Icon Accessibility

#### Standalone Icons
- **Decorative:** Use aria-hidden="true"
- **Meaningful:** Provide aria-label or alt text
- **Interactive:** Ensure 44x44px touch target
- **Color:** Don't rely solely on color for information

#### Icon Buttons
- **Label:** Always include aria-label
- **Tooltip:** Show on hover/focus for clarity
- **Focus:** Visible focus indicator required
- **Keyboard:** Fully keyboard accessible

---

## Design Tokens

### Color Tokens

```css
/* Primary Colors */
--color-primary: #931F33;
--color-primary-light: #E64C66;
--color-primary-gradient-0: #F9E1E5;
--color-primary-gradient-1: #EFCED3;
--color-primary-gradient-2: #E5BAC1;
--color-primary-gradient-3: #DAA7B0;
--color-primary-gradient-4: #D0939E;
--color-primary-gradient-5: #BC6D7A;
--color-primary-gradient-6: #A74657;
--color-primary-gradient-7: #58131F;
--color-primary-gradient-8: #3B0C14;
--color-primary-gradient-9: #1D060A;

/* Secondary Colors */
--color-secondary-light: #F4F4F4;
--color-secondary-dark: #221E1C;

/* Semantic Colors */
--color-success: #10B981;
--color-warning: #F59E0B;
--color-error: #EF4444;
--color-info: #3B82F6;

/* Neutral/Gray Scale */
--color-gray-50: #F9FAFB;
--color-gray-100: #F3F4F6;
--color-gray-200: #E5E7EB;
--color-gray-300: #D1D5DB;
--color-gray-400: #9CA3AF;
--color-gray-500: #6B7280;
--color-gray-600: #4B5563;
--color-gray-700: #374151;
--color-gray-800: #1F2937;
--color-gray-900: #111827;

/* Text Colors */
--color-text-primary: #221E1C;
--color-text-secondary: rgba(34, 30, 28, 0.7);
--color-text-disabled: rgba(34, 30, 28, 0.5);
--color-text-inverse: #FFFFFF;

/* Background Colors */
--color-bg-primary: #FFFFFF;
--color-bg-secondary: #F4F4F4;
--color-bg-overlay: rgba(0, 0, 0, 0.5);

/* Border Colors */
--color-border-primary: #D0939E;
--color-border-secondary: #E5BAC1;
--color-border-light: #F9E1E5;
```

### Typography Tokens

```css
/* Font Families */
--font-display: 'Playfair Display', serif;
--font-body: 'DM Sans', sans-serif;
--font-mono: 'JetBrains Mono', monospace;

/* Display */
--font-size-display: 72px;
--font-weight-display: 700;
--line-height-display: 1.1;

/* Heading 1 */
--font-size-h1: 48px;
--font-weight-h1: 500;
--line-height-h1: 1.2;

/* Heading 2 */
--font-size-h2: 36px;
--font-weight-h2: 500;
--line-height-h2: 1.25;

/* Heading 3 */
--font-size-h3: 28px;
--font-weight-h3: 600;
--line-height-h3: 1.3;

/* Heading 4 */
--font-size-h4: 24px;
--font-weight-h4: 600;
--line-height-h4: 1.35;

/* Heading 5 */
--font-size-h5: 20px;
--font-weight-h5: 500;
--line-height-h5: 1.4;

/* Heading 6 */
--font-size-h6: 18px;
--font-weight-h6: 600;
--line-height-h6: 1.4;

/* Body Large */
--font-size-body-lg: 20px;
--font-weight-body-lg: 400;
--line-height-body-lg: 1.6;

/* Body Regular */
--font-size-body: 18px;
--font-weight-body: 400;
--line-height-body: 1.6;

/* Body Small */
--font-size-body-sm: 14px;
--font-weight-body-sm: 400;
--line-height-body-sm: 1.5;

/* Label / Tag */
--font-size-label: 14px;
--font-weight-label: 500;
--line-height-label: 1.4;

/* Caption */
--font-size-caption: 14px;
--font-weight-caption: 500;
--line-height-caption: 1.4;

/* Code / Mono */
--font-size-code: 14px;
--font-weight-code: 400;
--line-height-code: 1.5;
```

### Spacing Tokens

```css
--space-4: 4px;
--space-8: 8px;
--space-12: 12px;
--space-16: 16px;
--space-24: 24px;
--space-32: 32px;
--space-48: 48px;
--space-64: 64px;

/* Component Spacing */
--padding-button-sm: 8px 12px;
--padding-button-md: 12px 16px;
--padding-button-lg: 16px 20px;

--padding-card: 24px;
--padding-input: 12px 16px;
--padding-small: 8px;

--margin-section: 64px 0;
--margin-component: 16px 0;
--margin-small: 8px 0;

--gap-default: 16px;
--gap-large: 32px;
--gap-small: 8px;
```

### Shadow Tokens

```css
--shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
--shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
--shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
--shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.1);
--shadow-2xl: 0 25px 50px rgba(0, 0, 0, 0.25);
```

### Border Radius Tokens

```css
--radius-none: 0;
--radius-sm: 4px;
--radius-md: 8px;
--radius-lg: 12px;
--radius-xl: 16px;
--radius-full: 9999px;
```

### Transition Tokens

```css
--transition-fast: 100ms ease;
--transition-normal: 200ms ease;
--transition-slow: 300ms ease;
--transition-bounce: cubic-bezier(0.68, -0.55, 0.265, 1.55);
```

---

## Best Practices

### Color Usage
1. **Accessibility First** - Always verify contrast ratios
2. **Semantic Colors** - Use appropriate colors for their meaning
3. **Consistency** - Use the defined palette, avoid ad-hoc colors
4. **Dark Mode** - Test colors in both light and dark themes

### Typography Usage
1. **Hierarchy** - Use size and weight for visual hierarchy
2. **Readability** - Maintain adequate line heights and lengths
3. **Consistency** - Use defined font families and sizes
4. **Legibility** - Ensure sufficient contrast with background

### Spacing Usage
1. **Consistency** - Always use defined spacing values
2. **Rhythm** - Create visual rhythm with consistent spacing
3. **Breathing Room** - Don't crowd content
4. **Relationships** - Tighter spacing for related items

### Component Usage
1. **States** - Always include all necessary states
2. **Accessibility** - Include focus states and labels
3. **Feedback** - Provide clear interaction feedback
4. **Consistency** - Maintain visual and behavioral consistency

---

## Implementation Resources

### CSS Framework Integration

The ApexNeural Design System is optimized for:
- **Bootstrap 5+**
- **Tailwind CSS**
- **Material Design**
- **Custom CSS**

### Design File Access
- **Figma:** [Link to design system file]
- **Sketch:** [Link to design system file]
- **Adobe XD:** [Link to design system file]

### Developer Documentation
- **Component Library:** [Link to storybook or component docs]
- **Design Tokens:** [Link to token documentation]
- **Code Examples:** [Link to code examples]

---

## Version History

### Version 1.0 (Current)
- Initial design system release
- Core colors, typography, spacing defined
- Button, alert, icon components specified
- Complete accessibility guidelines

---

## Support and Feedback

For questions, feedback, or to request new components:
- **Design System Slack:** #design-system
- **Email:** design-system@apexneural.com
- **Issue Tracker:** [Link to issue tracker]

---

*ApexNeural Design System © 2026. All rights reserved.*
