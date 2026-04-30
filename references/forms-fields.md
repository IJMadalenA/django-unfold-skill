# Forms & Fields Reference

> **Related files:** `SKILL.md` (conditional_fields, list_sections, change_form_datasets),
> `references/actions.md` (actions with intermediate form),
> `references/fields.md` (autocomplete fields, JsonField),
> `references/widgets.md` (ArrayWidget, WysiwygWidget),
> `references/inlines.md` (datasets vs nonrelated comparison)

---

## 1. Conditional Fields

Show/hide fields dynamically based on values of other fields. Uses Alpine.js expressions evaluated client-side.

### Configuration

```python
from unfold.admin import ModelAdmin

class UserAdmin(ModelAdmin):
    conditional_fields = {
        "country": "different_address == true",
        "city": "different_address == true",
        "address": "different_address == true",
    }
```

### Expression Syntax

- Uses standard Alpine.js expressions
- Field names used as variable names
- For compound widgets (e.g., SplitDateTimeWidget), use suffix: `field_name_0`, `field_name_1`
- Comparisons: `==`, `!=`, `>`, `<`, `>=`, `<=`
- Combine with `&&` (and), `||` (or)

### Common mistakes

- **Field name mismatch** — key must match the model field name exactly
- **Wrong expression syntax** — `==` not `=` in Alpine expressions
- **Forgetting `true`/`false` lowercase** — Alpine.js uses lowercase boolean literals

---

## 2. Sections (Expandable Rows)

Add expandable content areas below changelist rows. Two types: `TableSection` (tabular related data) and `TemplateSection` (custom HTML).

```python
from unfold.admin import ModelAdmin
from unfold.sections import TableSection, TemplateSection

class CustomTableSection(TableSection):
    verbose_name = _("Order Items")
    height = 300
    related_name = "items_set"
    fields = ["pk", "name", "price", "custom_field"]

    def custom_field(self, instance):
        return instance.pk


class CardSection(TemplateSection):
    template_name = "myapp/card_template.html"


class OrderAdmin(ModelAdmin):
    list_sections = [CardSection, CustomTableSection]
```

### TableSection Options

| Option | Type | Description |
|--------|------|-------------|
| `verbose_name` | str | Title above the table |
| `related_name` | str | Related model reverse relation name |
| `fields` | list | Field names from the related model |
| `height` | int | Force table height (px). Scroll if content overflows |

### Performance

Expandable rows add N+1 queries. Mitigate with:

```python
class OrderAdmin(ModelAdmin):
    list_per_page = 20

    def get_queryset(self, request):
        return super().get_queryset(request).prefetch_related("items_set")
```

### Multiple Related Tables

```python
class OrderAdmin(ModelAdmin):
    list_sections = [ItemsSection, ShipmentsSection]
```

---

## 3. Datasets

Display a Django admin changelist within another model's changeform page.

```python
from unfold.admin import ModelAdmin
from unfold.datasets import BaseDataset

class OrderDatasetAdmin(ModelAdmin):
    search_fields = ["name"]
    list_display = ["name", "total", "status"]
    list_display_links = ["name"]
    list_per_page = 20

    def get_queryset(self, request):
        obj_id = self.extra_context.get("object")
        if not obj_id:
            return super().get_queryset(request).none()
        return super().get_queryset(request).filter(customer__pk=obj_id)


class CustomerOrderDataset(BaseDataset):
    model = Order
    model_admin = OrderDatasetAdmin
    tab = True  # Display as tab on change form


class CustomerAdmin(ModelAdmin):
    change_form_datasets = [CustomerOrderDataset]
```

### Limitations

- `list_filter` is **not supported** on datasets
- Permissions must be handled manually in `get_queryset()`
- Dataset Admin is NOT registered with `@admin.register`

---

## 4. Crispy Forms

Unfold provides its own crispy-forms template pack: `"unfold_crispy"`.

### Setup

```python
# settings.py
INSTALLED_APPS = [
    "unfold",
    ...
    "crispy_forms",
]

CRISPY_TEMPLATE_PACK = "unfold_crispy"
CRISPY_ALLOWED_TEMPLATE_PACKS = ["unfold_crispy"]
```

```bash
pip install django-crispy-forms
```

### Basic Form

```python
# forms.py
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Layout
from django import forms

class CustomForm(forms.Form):
    name = forms.CharField(max_length=100)

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.layout = Layout("name")
```

```python
# views.py
from django.views.generic import FormView
from unfold.views import UnfoldModelAdminViewMixin

class MyView(UnfoldModelAdminViewMixin, FormView):
    title = "Custom Form"
    form_class = CustomForm
    permission_required = ()
    template_name = "myapp/form.html"
```

```html
{# form.html #}
{% load crispy_forms_tags %}
{% block content %}
    {% crispy form "unfold_crispy" %}
{% endblock %}
```

### Form Helper for Inline Formsets

```python
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit

class YourModelFormHelper(FormHelper):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.template = "unfold_crispy/layout/table_inline_formset.html"
        self.form_id = "your-model-formset"
        self.form_add = True
        self.attrs = {"novalidate": "novalidate"}
        self.add_input(Submit("submit", _("Submit")))
```

### Skip Global CRISPY_TEMPLATE_PACK

If you use crispy-forms for frontend too, avoid the global setting:

```html
{% load crispy_forms_tags %}
{% crispy form "unfold_crispy" %}  {# explicit template pack #}
```

---

## 5. Common Mistakes

- **Conditional fields don't work** → Alpine.js expression uses lowercase `true`/`false`, not Python's `True`/`False`
- **Datasets show no data** → `get_queryset()` must handle the `None` case (new object page)
- **Crispy forms unstyled** → forgot `"unfold_crispy"` template pack parameter
- **Sections have N+1 queries** → add `prefetch_related()` in `get_queryset()`
- **list_filter on datasets** → not supported; use `get_queryset()` instead
- For widgets (ArrayWidget, WysiwygWidget), see `references/widgets.md`
- For autocomplete fields and JsonField, see `references/fields.md`
