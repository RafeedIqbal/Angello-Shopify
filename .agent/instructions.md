# Angello Fragrances — Shopify Theme Development Guide

## Project Overview

This is a **Shopify Online Store 2.0** theme for **Angello Fragrances** (`angello-fragrances.myshopify.com`). It is based on Shopify's **Dawn** theme using the **"Refresh"** preset. The theme uses **Liquid** templating, **JSON templates**, vanilla CSS, and vanilla JavaScript (Web Components).

---

## Tech Stack

| Layer        | Technology                                                 |
| ------------ | ---------------------------------------------------------- |
| Templating   | Liquid (`.liquid` files)                                   |
| Templates    | JSON-based Online Store 2.0 templates (`.json`)            |
| Styling      | Vanilla CSS (no preprocessor, no Tailwind)                 |
| JavaScript   | Vanilla JS with Web Components (Custom Elements)           |
| Fonts        | **Archivo** (headings, weight 700), **Questrial** (body)   |
| Dev Server   | Shopify CLI (`shopify theme dev`)                          |
| Version Ctrl | Git                                                        |

---

## Directory Structure

```
Angello-Shopify/
├── assets/          # CSS, JS, SVGs, and static assets (flat, no subdirectories)
├── config/          # Theme settings (settings_schema.json, settings_data.json)
├── layout/          # Layout wrappers (theme.liquid, password.liquid)
├── locales/         # Translation files (en.default.json + 25 languages)
├── sections/        # Reusable section files (.liquid) and section groups (.json)
├── snippets/        # Reusable partial templates (rendered via {% render %})
└── templates/       # Page-level JSON templates and customers/ subdirectory
```

### Key Conventions

- **`assets/`** is flat — all CSS, JS, SVGs live at the root level.  
  - CSS files follow naming: `base.css`, `component-*.css`, `section-*.css`, `template-*.css`
  - JS files are standalone modules loaded via `<script src="{{ 'file.js' | asset_url }}" defer>`
- **`sections/`** contain both `.liquid` section files and `.json` section group files (e.g. `header-group.json`, `footer-group.json`)
- **`templates/`** use JSON format — they define which sections appear on each page type and their settings. Do NOT put HTML/Liquid in template files (except `gift_card.liquid`).
- **`snippets/`** are included via `{% render 'snippet-name' %}` — never use `{% include %}` (deprecated).

---

## Color Schemes

The theme uses 5 color schemes defined in `config/settings_data.json`. Reference them in Liquid/CSS via `color-{{ scheme.id }}` class:

| Scheme     | Background | Text      | Button    | Usage               |
| ---------- | ---------- | --------- | --------- | ------------------- |
| `scheme-1` | `#eff0f5`  | `#0e1b4d` | `#4770db` | Default / primary   |
| `scheme-2` | `#FFFFFF`  | `#0e1b4d` | `#0e1b4d` | White / clean       |
| `scheme-3` | `#0e1b4d`  | `#FFFFFF` | `#FFFFFF` | Dark / inverted     |
| `scheme-4` | `#4770db`  | `#FFFFFF` | `#FFFFFF` | Accent blue / sales |
| `scheme-5` | `#E32402`  | `#FFFFFF` | `#FFFFFF` | Alert red           |

---

## Design Tokens (CSS Custom Properties)

All design tokens are set in `layout/theme.liquid` as CSS custom properties on `:root`. Key tokens:

```css
/* Typography */
--font-body-family      /* Questrial */
--font-heading-family   /* Archivo */
--font-body-scale       /* 1.05 */
--font-heading-scale    /* derived from heading_scale / body_scale */

/* Layout */
--page-width            /* 120rem (1200px) */
--spacing-sections-desktop
--grid-desktop-vertical-spacing
--grid-desktop-horizontal-spacing

/* Components */
--buttons-radius: 40px          /* Very rounded buttons */
--inputs-radius: 26px
--card-corner-radius: 18px
--media-radius: 20px
--popup-corner-radius: 18px
```

---

## Development Workflow

### Starting the Dev Server

```bash
shopify theme dev --store=angello-fragrances.myshopify.com
```

This starts a local proxy server with hot reload. Ensure you have the [Shopify CLI](https://shopify.dev/docs/api/shopify-cli) installed and are authenticated.

### Deploying

```bash
# Push to the connected theme
shopify theme push

# Push to a specific theme by ID
shopify theme push --theme <THEME_ID>

# Deploy as a new unpublished theme
shopify theme push --unpublished
```

### Checking for Errors

```bash
# Run Shopify's theme linter
shopify theme check
```

---

## Shopify Liquid Patterns

### Section Schema

Every section file must end with a `{% schema %}` JSON block that defines:
- `name` — display name in the theme editor
- `settings` — array of setting input definitions
- `blocks` — optional repeatable content blocks
- `presets` — defines if/how the section appears in the theme editor's "Add section" menu

### Rendering Assets

```liquid
<!-- CSS -->
{{ 'component-card.css' | asset_url | stylesheet_tag }}

<!-- JS (always defer) -->
<script src="{{ 'product-form.js' | asset_url }}" defer="defer"></script>

<!-- Lazy-load CSS (print trick) -->
<link rel="stylesheet" href="{{ 'component-price.css' | asset_url }}" media="print" onload="this.media='all'">
```

### Including Snippets

```liquid
{%- render 'card-product', card_product: product, show_vendor: section.settings.show_vendor -%}
```

### Translation Strings

```liquid
{{ 'products.product.add_to_cart' | t }}
```

Translation keys are defined in `locales/en.default.json`.

---

## JavaScript Architecture

The theme uses **Web Components** (Custom Elements) extensively. Key patterns:

```javascript
// Typical component definition in assets/*.js
class ProductForm extends HTMLElement {
  constructor() {
    super();
    // initialization
  }
  // methods...
}
customElements.define('product-form', ProductForm);
```

### Key JS Files

| File                | Purpose                                          |
| ------------------- | ------------------------------------------------ |
| `global.js`         | Core utilities, base classes, shared components   |
| `pubsub.js`         | Publish/subscribe event system                    |
| `constants.js`      | Shared constants (e.g., `ON_CHANGE_DEBOUNCE_TIMER`) |
| `product-info.js`   | Product page variant switching & dynamic updates  |
| `product-form.js`   | Add-to-cart form handling                         |
| `cart.js`           | Cart page logic                                  |
| `cart-drawer.js`    | Slide-out cart drawer                             |
| `facets.js`         | Collection filtering & sorting                   |

### Event Communication

Components communicate via a pub/sub system (`pubsub.js`). Common events:

```javascript
// Publishing
publish(PUB_SUB_EVENTS.cartUpdate, { source: 'product-form', cartData: data });

// Subscribing
subscribe(PUB_SUB_EVENTS.cartUpdate, (event) => { /* handle */ });
```

---

## Important Rules

### DO

- ✅ Use `{% render %}` for snippets (never `{% include %}`)
- ✅ Use `defer` on all `<script>` tags
- ✅ Follow the existing CSS naming conventions (`component-*`, `section-*`, `template-*`)
- ✅ Add new section settings to the `{% schema %}` block
- ✅ Use CSS custom properties from `:root` instead of hardcoding values
- ✅ Use the translation system (`| t` filter) for user-facing strings
- ✅ Use Online Store 2.0 JSON templates — sections should be reusable
- ✅ Place all assets in the flat `assets/` directory (no subdirectories allowed)
- ✅ Follow Web Component patterns when adding new interactive JS elements
- ✅ Use `color-{{ scheme.id }}` classes to apply color schemes to sections

### DON'T

- ❌ Edit `config/settings_data.json` directly — it's auto-generated by the theme editor
- ❌ Put Liquid/HTML in JSON template files (only in sections and snippets)
- ❌ Use `{% include %}` (deprecated, breaks scope isolation)
- ❌ Create subdirectories inside `assets/` (Shopify doesn't support it)
- ❌ Use inline `<style>` blocks for section-specific styles (use `assets/` CSS files)
- ❌ Use npm, Webpack, or any build tools — this is a vanilla Shopify theme
- ❌ Hardcode color values — always use CSS custom properties from the color scheme system
- ❌ Use `document.write()` or synchronous script loading

---

## Cart Configuration

- **Cart type**: Drawer (slide-out panel, not a separate page)
- Cart drawer logic is in `assets/cart-drawer.js` and styled by `assets/component-cart-drawer.css`
- Cart sections: `sections/cart-drawer.liquid` (empty stub), `snippets/cart-drawer.liquid` (actual implementation)

---

## Theme Editor Section Groups

The header and footer are defined as **section groups** (Online Store 2.0):

- `sections/header-group.json` — controls which sections appear in the header area
- `sections/footer-group.json` — controls which sections appear in the footer area

These are rendered in `layout/theme.liquid` via:
```liquid
{% sections 'header-group' %}
{% sections 'footer-group' %}
```

---

## File Naming Conventions

| Pattern                      | Example                        | Description                       |
| ---------------------------- | ------------------------------ | --------------------------------- |
| `base.css`                   | Core reset and global styles   | Loaded on every page              |
| `component-*.css`            | `component-card.css`           | Styles for reusable UI components |
| `section-*.css`              | `section-image-banner.css`     | Styles for specific sections      |
| `template-*.css`             | `template-collection.css`      | Styles for page templates         |
| `icon-*.svg`                 | `icon-cart.svg`                | SVG icon assets                   |
| `main-*.liquid` (sections)   | `main-product.liquid`          | Primary section for a page type   |
| `card-*.liquid` (snippets)   | `card-product.liquid`          | Card-style snippet components     |

---

## Quick Reference: Common Tasks

### Add a new section
1. Create `sections/my-section.liquid` with HTML/Liquid and a `{% schema %}` block
2. Create `assets/section-my-section.css` for styles
3. Optionally create `assets/my-section.js` for interactivity
4. Add a `presets` entry in the schema to make it available in the theme editor

### Add a new snippet
1. Create `snippets/my-snippet.liquid`
2. Render it from a section: `{% render 'my-snippet', param: value %}`

### Modify theme settings
1. Edit `config/settings_schema.json` to add new setting definitions
2. Access in Liquid via `{{ settings.your_setting_name }}`

### Add a new page template
1. Create `templates/page.my-template.json`
2. Define which sections it uses in the JSON structure
3. Assign it to pages in the Shopify admin
