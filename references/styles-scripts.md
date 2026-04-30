# Styles & Scripts Reference — Loading Custom CSS/JS, Tailwind

> **Related files:** `references/configuration.md` (STYLES, SCRIPTS keys in UNFOLD dict),
> `references/theming.md` (COLORS, BORDER_RADIUS — alternative to custom CSS),
> `references/dashboard-components.md` (custom dashboard templates needing styles)

---

## 1. Loading Custom Styles and Scripts

Add global CSS and JS to every admin page via `UNFOLD["STYLES"]` and `UNFOLD["SCRIPTS"]`.

```python
# settings.py
from django.templatetags.static import static

UNFOLD = {
    "STYLES": [
        lambda request: static("css/admin-custom.css"),
    ],
    "SCRIPTS": [
        lambda request: static("js/admin-custom.js"),
    ],
}
```

The lambdas receive the `request` object, so you can serve different files per user/environment:

```python
lambda request: static("css/dark.css") if request.user.is_superuser else static("css/light.css")
```

**Production note:** Always run `python manage.py collectstatic` after adding new static files.

---

## 2. Customizing Tailwind Stylesheet

### Unfold ≥ 0.56 — Recommended: Plain CSS or Tailwind 4

For Unfold 0.56+, writing plain CSS loaded via `UNFOLD["STYLES"]` is the simplest approach. If you need Tailwind utility classes in custom templates:

```bash
npm i tailwindcss @tailwindcss/cli
```

Create a CSS file:

```css
/* styles.css */
@import 'tailwindcss';

.my-widget { @apply bg-primary-600 text-white rounded; }
```

Compile and load:

```bash
npx @tailwindcss/cli -i styles.css -o myapp/static/css/admin.css --minify
```

```python
UNFOLD = {
    "STYLES": [lambda request: static("css/admin.css")],
}
```

Add to `package.json` for convenience:

```json
{
  "scripts": {
    "tailwind:watch": "npx @tailwindcss/cli -i styles.css -o myapp/static/css/admin.css --minify --watch",
    "tailwind:build": "npx @tailwindcss/cli -i styles.css -o myapp/static/css/admin.css --minify"
  }
}
```

### Unfold < 0.56 — Tailwind 3

```javascript
// tailwind.config.js
module.exports = {
    darkMode: "class",
    content: ["./myapp/**/*.{html,py,js}"],
    theme: {
        extend: {
            colors: {
                primary: {
                    50:  "rgb(var(--color-primary-50) / <alpha-value>)",
                    // ... 100 through 950
                },
                base: {
                    50:  "rgb(var(--color-base-50) / <alpha-value>)",
                    // ... 100 through 950
                },
                font: {
                    "subtle-light": "rgb(var(--color-font-subtle-light) / <alpha-value>)",
                    "default-light": "rgb(var(--color-font-default-light) / <alpha-value>)",
                    "important-light": "rgb(var(--color-font-important-light) / <alpha-value>)",
                    "subtle-dark": "rgb(var(--color-font-subtle-dark) / <alpha-value>)",
                    "default-dark": "rgb(var(--color-font-default-dark) / <alpha-value>)",
                    "important-dark": "rgb(var(--color-font-important-dark) / <alpha-value>)",
                },
            },
        },
    },
};
```

```bash
npx tailwindcss -i styles.css -o myapp/static/css/admin.css --minify --watch
```

---

## 3. When to Use What

| Approach | When to use |
|----------|------------|
| `UNFOLD["COLORS"]` | Simple color/brand changes — no custom CSS needed |
| `UNFOLD["STYLES"]` + plain CSS | Most custom dashboard/components — simplest, no build step |
| Tailwind 4 via `@import 'tailwindcss'` | Need Tailwind utility classes in custom templates (Unfold ≥ 0.56) |
| Tailwind 3 config | Need Tailwind utility classes (Unfold < 0.56) |

---

## 4. Common Mistakes

- **Styles not appearing in production** → forgot `collectstatic`
- **Tailwind 3 classes not working** → Unfold ≥ 0.56 uses Tailwind 4 — write plain CSS instead
- **`@apply` directives failing** → ensure you're using the right Tailwind version syntax
- **Custom styles conflicting with Unfold** → scope selectors under a parent class or `:where()` to avoid specificity battles

---

→ For the complete UNFOLD settings dict, see `references/configuration.md`
→ For color palettes and theming (no CSS needed), see `references/theming.md`
