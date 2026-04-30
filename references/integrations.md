# Integrations Reference — Third-Party Packages

> **Related files:** `references/installation.md` (INSTALLED_APPS setup),
> `../SKILL.md` (Rule 2: INSTALLED_APPS order, Common Pitfalls)

---

## Overview

All integrations follow the same pattern:
1. Add the `unfold.contrib.*` app to `INSTALLED_APPS` (before `django.contrib.admin`)
2. Unregister the third-party model from default admin
3. Re-register with `unfold.admin.ModelAdmin`

---

## 1. django-celery-beat

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.celery",   # <-- add
    "django.contrib.admin",
]
```

No additional re-registration needed — `unfold.contrib.celery` handles this automatically.

---

## 2. django-import-export

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.import_export",  # <-- add
    "django.contrib.admin",
    "import_export",
]
```

Use `ImportExportModelAdmin` from Unfold:

```python
from unfold.contrib.import_export.admin import ImportExportModelAdmin

@admin.register(Product)
class ProductAdmin(ImportExportModelAdmin, ModelAdmin):
    pass
```

@display decorator works fully with import-export fields.

---

## 3. django-guardian

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.guardian",  # <-- add
    "django.contrib.admin",
    "guardian",
]
```

Then unregister and re-register:

```python
from django.contrib.auth.models import User, Group
from guardian.admin import GuardedModelAdmin

admin.site.unregister(User)
admin.site.unregister(Group)

@admin.register(User)
class UserAdmin(GuardedModelAdmin, ModelAdmin):
    pass
```

---

## 4. django-simple-history

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.simple_history",  # <-- add
    "django.contrib.admin",
    "simple_history",
]
```

Add `SimpleHistoryAdmin` to see history in Unfold-styled UI.

---

## 5. django-constance

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.constance",  # <-- add
    "django.contrib.admin",
    "constance",
]
```

Unfold provides styled constance config pages automatically.

---

## 6. django-location-field

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.location_field",  # <-- add
    "django.contrib.admin",
    "location_field",
]
```

Enables styled map picker widget in Unfold forms.

---

## 7. django-modeltranslation

No Unfold contrib package. Configure in `UNFOLD` settings:

```python
UNFOLD = {
    "EXTENSIONS": {
        "modeltranslation": {
            "flags": {
                "en": "🇬🇧",
                "es": "🇪🇸",
                "fr": "🇫🇷",
            },
        },
    },
}
```

---

## 8. django-money

No Unfold contrib package. Unfold styles money fields automatically when used in readonly_fields or list_display.

---

## 9. djangoql

No Unfold contrib package. Works alongside Unfold without extra configuration.

```python
# With djangoql installed, add to ModelAdmin:
class OrderAdmin(ModelAdmin):
    search_fields = ["id", "customer__name"]  # djangoql enables cross-model search
```

---

## 10. django-json-widget

No Unfold contrib package. Install and use directly — Unfold renders JSON widgets with default styling.

---

## Common Mistakes

- **Third-party admin unstyled** → forgot to add `unfold.contrib.*` or re-register with `ModelAdmin`
- **django-guardian + User model** → must unregister User and Group before re-registering
- **django-celery-beat tasks not styled** → add `"unfold.contrib.celery"` before `"django.contrib.admin"`
- **import-export buttons unstyled** → use `ImportExportModelAdmin` from Unfold, not from django-import-export
