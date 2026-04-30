# Widgets Reference — ArrayWidget, WysiwygWidget

> **Related files:** `references/configuration.md` (formfield_overrides context),
> `references/fields.md` (autocomplete fields, JsonField),
> `SKILL.md` (ModelAdmin options: formfield_overrides)

**Prerequisite:** Add `"unfold.contrib.forms"` to `INSTALLED_APPS`.

```python
INSTALLED_APPS = [
    "unfold",
    "unfold.contrib.forms",  # required for widgets below
    "django.contrib.admin",
]
```

---

## 1. WysiwygWidget — Rich Text Editor

Replaces plain `<textarea>` with the [Trix editor](https://trix-editor.org/) — a clean, open-source WYSIWYG editor.

### Basic Usage

```python
from django.db import models
from unfold.admin import ModelAdmin
from unfold.contrib.forms.widgets import WysiwygWidget


@admin.register(Article)
class ArticleAdmin(ModelAdmin):
    formfield_overrides = {
        models.TextField: {
            "widget": WysiwygWidget,
        },
    }
```

All `TextField` fields will now use the Trix rich text editor.

### Notes

- Trix does **not** include built-in file upload. To add images, upload them through your media system first, then insert the URL.
- Works best for admin users who need basic formatting (bold, italic, lists, links).
- No additional JS/CSS to load — Unfold handles it automatically when the widget is used.

---

## 2. ArrayWidget — PostgreSQL ArrayField Input

Renders `ArrayField` values as a list of input fields with add/remove controls. When `choices` are provided, switches to a dropdown list interface.

### Basic Usage

```python
from django.contrib.postgres.fields import ArrayField
from unfold.admin import ModelAdmin
from unfold.contrib.forms.widgets import ArrayWidget


@admin.register(Product)
class ProductAdmin(ModelAdmin):
    formfield_overrides = {
        ArrayField: {
            "widget": ArrayWidget,
        },
    }
```

### With Choices (Dropdown Mode)

When the widget has `choices`, it renders a dropdown list instead of text inputs:

```python
from django.db.models import TextChoices
from django.utils.translation import gettext_lazy as _


class SizeChoices(TextChoices):
    SMALL = "S", _("Small")
    MEDIUM = "M", _("Medium")
    LARGE = "L", _("Large")
    XL = "XL", _("Extra Large")


@admin.register(Product)
class ProductAdmin(ModelAdmin):
    formfield_overrides = {
        ArrayField: {
            "widget": ArrayWidget,
        },
    }

    def get_form(self, request, obj=None, change=False, **kwargs):
        form = super().get_form(request, obj, change, **kwargs)
        form.base_fields["sizes"].widget = ArrayWidget(choices=SizeChoices)
        return form
```

The widget does **not** auto-detect choices from the field definition — set them explicitly in `get_form()`.

---

## 3. Common Mistakes

- **Widget not rendering** → forgot `"unfold.contrib.forms"` in `INSTALLED_APPS`
- **ArrayWidget showing text inputs instead of dropdown** → choices must be passed explicitly via `get_form()`
- **WysiwygWidget not saving HTML** → ensure the field is a `TextField`, not a `CharField` (which may truncate)
- **Both widgets conflicting** → order of `formfield_overrides` doesn't matter, each field type gets its own widget

---

→ For custom form fields (autocomplete, conditional fields), see `references/fields.md`
→ For form-level customization (crispy forms, datasets), see `references/forms-fields.md`
