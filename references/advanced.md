# Advanced Reference — Custom Pages, Sites, Changelist, Command Palette, Multi-Language

> **Related files:** `references/configuration.md` (full UNFOLD dict context),
> `references/theming.md` (sidebar navigation needing manual custom page entries),
> `references/dashboard-components.md` (custom pages using same component system)

---

## 1. Custom Pages

Create class-based views with Unfold styling.

### View Definition

```python
from django.urls import path
from django.views.generic import TemplateView
from unfold.admin import ModelAdmin
from unfold.views import UnfoldModelAdminViewMixin

class MyCustomView(UnfoldModelAdminViewMixin, TemplateView):
    title = "Custom Page Title"
    permission_required = ()  # tuple of permissions
    template_name = "myapp/custom_page.html"


@admin.register(MyModel)
class MyModelAdmin(ModelAdmin):
    def get_urls(self):
        custom_view = self.admin_site.admin_view(
            MyClassBasedView.as_view(model_admin=self)
        )
        return super().get_urls() + [
            path("custom-url/", custom_view, name="custom_name"),
        ]
```

### Template

```django
{% extends "unfold/layouts/base_simple.html" %}
{% load i18n unfold %}

{% block content %}
    {% tab_list "my_tab_key" %}
    <h1>{% trans "Custom page" %}</h1>
{% endblock %}
```

### Add to Sidebar Navigation

Custom pages are NOT added to sidebar automatically. Add manually:

```python
UNFOLD = {
    "SIDEBAR": {
        "navigation": [
            {
                "title": _("Custom"),
                "items": [
                    {
                        "title": _("My Custom Page"),
                        "icon": "settings",
                        "link": reverse_lazy("admin:custom_name"),
                    },
                ],
            },
        ],
    },
}
```

---

## 2. Custom Sites

Run multiple admin sites with Unfold.

```python
# admin_sites.py
from django.contrib.admin import AdminSite
from unfold.admin import ModelAdmin

class CustomAdminSite(AdminSite):
    site_header = "Custom Site"

custom_admin_site = CustomAdminSite(name="custom_admin")


@admin.register(MyModel, site=custom_admin_site)
class MyModelAdmin(ModelAdmin):
    pass
```

```python
# urls.py
from .admin_sites import custom_admin_site

urlpatterns = [
    path("custom-admin/", custom_admin_site.urls),
]
```

---

## 3. Sortable Changelist

Enable drag-and-drop reordering on the changelist.

### Model Setup

```python
from django.db import models

class Slide(models.Model):
    title = models.CharField(max_length=100)
    sort = models.PositiveIntegerField(default=0)

    class Meta:
        ordering = ["sort"]
```

### Admin Setup

```python
from unfold.admin import ModelAdmin

class SlideAdmin(ModelAdmin):
    ordering = ["sort"]
    list_display = ["title", "sort"]
    list_editable = ["sort"]  # optional: inline editing
```

The drag handle appears automatically when the model has an ordering Meta option with a numeric field.

---

## 4. Command Palette

Unfold includes a command palette (Cmd+K / Ctrl+K) for quick navigation and search across models.

### Enable Command Search in Sidebar

```python
UNFOLD = {
    "SIDEBAR": {
        "show_search": True,
        "command_search": True,  # replaces sidebar search with command palette
    },
}
```

The command palette searches across:
- All registered model names
- All sidebar navigation items
- Custom pages (if added to sidebar)

---

## 5. Multi-Language

### Language Switcher

Enable the language switcher in the admin header:

```python
UNFOLD = {
    "SITE_TITLE": "My Admin",
}
```

Django's `LocaleMiddleware` and `i18n_patterns` work automatically with Unfold.

### Internationalized URLs

```python
# urls.py
from django.conf.urls.i18n import i18n_patterns

urlpatterns = i18n_patterns(
    path("admin/", admin.site.urls),
)
```

### Model Translation Flags

```python
UNFOLD = {
    "EXTENSIONS": {
        "modeltranslation": {
            "flags": {
                "en": "🇬🇧",
                "es": "🇪🇸",
                "fr": "🇫🇷",
                "de": "🇩🇪",
            },
        },
    },
}
```

### Translatable Strings

Use Django's `gettext_lazy` for settings:

```python
from django.utils.translation import gettext_lazy as _

UNFOLD = {
    "SITE_TITLE": _("My Admin"),
    "SIDEBAR": {
        "navigation": [
            {
                "title": _("Dashboard"),
                "items": [
                    {"title": _("Orders"), "link": reverse_lazy("admin:shop_order_changelist")},
                ],
            },
        ],
    },
}
```

---

## 6. Paginator

### Infinite Paginator

For very large datasets:

```python
# settings.py
UNFOLD = {
    "PAGINATOR": "unfold.paginator.InfinitePaginator",
}
```

```python
# admin.py
class LargeDatasetAdmin(ModelAdmin):
    show_full_result_count = False  # recommended with infinite paginator
```

---

## 7. Development / Contributing

### Starting the Testing Server

The Unfold repository has a testing server at `tests/server`:
```bash
cd tests/server
uv run -- python manage.py runserver
```

### Running Tests
```bash
uv run -- pytest .
```

### Compiling Tailwind (for Unfold source development)
```bash
npm install
npm run tailwind:watch   # watch for changes
npm run tailwind:build   # one-time build
```

### Pre-commit
```bash
pip install pre-commit
pre-commit install
pre-commit install --hook-type commit-msg
```

---

## Common Mistakes

- **Custom page returns 404** → URL not registered in `get_urls()` or wrong path
- **Custom page not styled** → template must extend `unfold/layouts/base_simple.html`, not Django's default
- **Sortable changelist not working** → model must have `ordering = ["numeric_field"]` in Meta
- **Command palette missing items** → items must be in `UNFOLD["SIDEBAR"]["navigation"]`
- **Language switcher not appearing** → `USE_I18N = True` and `LocaleMiddleware` in MIDDLEWARE
