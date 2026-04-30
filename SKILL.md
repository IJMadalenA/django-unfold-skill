---
name: django-unfold
description: >-
  Expert skill for building, customizing, and debugging Django admin panels with django-unfold.
  Use whenever the user mentions django-unfold, UnfoldModelAdmin, Unfold admin theme,
  django admin customization with Tailwind, or any modern admin UI features (colored badges,
  sidebar navigation, dashboard components, charts, dialog actions, custom filters, tabs,
  inlines with drag-and-drop, WYSIWYG editors, autocomplete fields, or third-party integrations
  like django-import-export, django-celery-beat, django-guardian). Also use when the user reports
  unstyled Django admin pages, missing styles, or asks to "make the admin look better" in a project
  that uses or should use django-unfold. Make sure to also use this skill when the user asks to
  set up custom pages, build admin dashboards, create sortable changelists, or add command palette
  search, even if they don't explicitly mention Unfold by name — if their project has `django-unfold`
  in dependencies, this is the right skill.
compatibility:
  - python
  - django
  - django-unfold
---

# Django Unfold — Modern Admin for Django

## Overview

Django Unfold is a drop-in enhancement of `django.contrib.admin` built on Tailwind CSS. It's fully compatible with all native Django admin patterns while adding rich UI components and developer features.

**Current version: 0.90.x** — requires Django 5.0+

## Quick Navigation

| Task | Start Here |
|------|------------|
| Installation / first setup | `references/installation.md` |
| UNFOLD settings dict | `references/configuration.md` |
| Colors / sidebar / login page | `references/theming.md` |
| @display / @action decorators | `references/decorators.md` |
| Actions (list, row, detail, dialog) | `references/actions.md` |
| Filters (text, date, dropdown, numeric) | `references/filters.md` |
| Tabs (changelist, changeform, fieldsets) | `references/tabs.md` |
| Inlines (sortable, paginated, nested) | `references/inlines.md` |
| Dashboard & components | `references/dashboard-components.md` |
| Component classes (Python) | `references/component-class.md` |
| Forms, sections, datasets, crispy | `references/forms-fields.md` |
| Custom fields (autocomplete, JSON) | `references/fields.md` |
| Widgets (WYSIWYG, ArrayWidget) | `references/widgets.md` |
| Custom pages / sites / sortable changelist | `references/advanced.md` |
| Custom CSS/JS / Tailwind | `references/styles-scripts.md` |
| Third-party integrations | `references/integrations.md` |

---

## Critical Rules

### Rule 1: ModelAdmin inheritance

Every admin class **must** inherit from `unfold.admin.ModelAdmin`:

```python
from unfold.admin import ModelAdmin  # NOT django.contrib.admin

@admin.register(Product)
class ProductAdmin(ModelAdmin):
    pass
```

Using `django.contrib.admin.ModelAdmin` silently drops all Unfold styling.

### Rule 2: INSTALLED_APPS order

`"unfold"` must appear **before** `"django.contrib.admin"`:

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.filters",           # advanced filters
    "unfold.contrib.forms",             # ArrayWidget, WysiwygWidget, crispy
    "unfold.contrib.inlines",           # NonrelatedInline, nested inlines
    "unfold.contrib.import_export",
    "unfold.contrib.simple_history",
    "unfold.contrib.guardian",
    "unfold.contrib.constance",
    "unfold.contrib.location_field",
    "django.contrib.admin",             # MUST be after unfold
]
```

### Rule 3: Unfold inline base classes

```python
from unfold.admin import StackedInline, TabularInline  # NOT django.contrib.admin
```

### Rule 4: User & Group — must re-register

Built-in User/Group models come unstyled unless re-registered with Unfold's `ModelAdmin`. See `references/installation.md`.

---

## ModelAdmin Quick Reference

```python
from unfold.admin import ModelAdmin

class MyModelAdmin(ModelAdmin):
    # ── Changelist ─────────────────────────────────────────
    list_fullwidth = False
    list_filter_sheet = True           # bottom sheet (False = sidebar)
    list_filter_submit = False
    list_horizontal_scrollbar_top = False
    list_disable_select_all = False

    # ── Change form ────────────────────────────────────────
    compressed_fields = True
    warn_unsaved_form = True
    show_add_link = True
    change_form_show_cancel_button = False

    # ── Actions ────────────────────────────────────────────
    actions_list = []                  # header buttons
    actions_row = []                   # per-row buttons
    actions_detail = []                # change form header
    actions_submit_line = []           # near Save button
    actions_list_hide_default = False  # hide Django's default dropdown

    # ── Custom templates ───────────────────────────────────
    change_form_before_template = "myapp/before_form.html"
    change_form_after_template = "myapp/after_form.html"
    change_form_outer_before_template = "myapp/outer_before.html"
    change_form_outer_after_template = "myapp/outer_after.html"

    # ── Readonly preprocess ────────────────────────────────
    readonly_preprocess_fields = {
        "html_field": "html.unescape",
        "text_field": lambda c: c.strip(),
    }

    # ── Widgets ────────────────────────────────────────────
    formfield_overrides = {
        models.TextField: {"widget": WysiwygWidget},
        ArrayField: {"widget": ArrayWidget},
    }
```

---

## @display Decorator — Quick Examples

```python
from unfold.decorators import display

class OrderAdmin(ModelAdmin):
    list_display = ["show_customer", "show_status", "show_priority"]

    @display(header=True)
    def show_customer(self, obj):
        return obj.full_name, obj.email        # (main, subtitle)

    @display(
        description="Status",
        ordering="status",
        label={
            "PENDING":   "warning",
            "ACTIVE":    "info",
            "COMPLETED": "success",
            "FAILED":    "danger",
        },
    )
    def show_status(self, obj):
        return obj.status

    @display(description="VIP", label=True)
    def show_priority(self, obj):
        return obj.is_vip
```

---

## Common Mistakes (From Baseline Testing)

- **Unstyled admin** → forgot to inherit from `unfold.admin.ModelAdmin`
- **Hex colors silently fail** → Unfold uses OKLCH (generate at https://oklch.com or https://unfoldadmin.com/colors/)
- **User/Group unstyled** → must unregister and re-register (see `references/installation.md`)
- **Missing styles in production** → run `python manage.py collectstatic`
- **Tailwind conflict** → Unfold ≥ 0.56 uses Tailwind 4 — don't load Tailwind 3 CSS; use `UNFOLD["COLORS"]` instead
- **Django 4.2 + Unfold ≥ 0.90** → incompatible; use Unfold < 0.90
- **Third-party admin unstyled** → unregister and re-register with `unfold.admin.ModelAdmin`
- **Custom pages not in sidebar** → add manually to `UNFOLD["SIDEBAR"]["navigation"]`
- **list_filter on Datasets** → not supported; filter via `get_queryset()` instead
- **Dialog actions not working** → must return `HttpResponse` with `HX-Redirect` header

---

## Red Flags — STOP and Read the Reference

| "I'll just use hex colors" | No. OKLCH required → `references/theming.md` |
| "django.contrib.admin.ModelAdmin is fine" | No. Loses all styling → `references/installation.md` |
| "INSTALLED_APPS order doesn't matter" | It does → `references/installation.md` |
| "User/Group just work" | They don't → `references/installation.md` |
| "I'll skip the dialog form response" | Dialog actions need `HX-Redirect` → `references/actions.md` |
| "My custom page isn't showing" | Add to sidebar manually → `references/advanced.md` |
