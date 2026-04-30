# Fields Reference — Autocomplete Fields, JsonField

> **Related files:** `references/filters.md` (AutocompleteFilter for changelist),
> `references/widgets.md` (ArrayWidget, WysiwygWidget for form inputs),
> `references/forms-fields.md` (conditional fields, crispy forms)

---

## 1. Autocomplete Fields

Add search-as-you-type autocomplete to custom `ModelChoiceField` and `ModelMultipleChoiceField` in admin forms.

### How It Works

1. Create a view subclassing `BaseAutocompleteView` — returns JSON with matching options.
2. Register the view as a URL in your `ModelAdmin` via `custom_urls`.
3. Use `UnfoldAdminAutocompleteModelChoiceField` or `UnfoldAdminMultipleAutocompleteModelChoiceField` in your form, pointing to the URL.

### Step-by-Step Example

```python
from django import forms
from django.contrib import admin
from django.utils.translation import gettext_lazy as _

from unfold.admin import ModelAdmin
from unfold.views import BaseAutocompleteView
from unfold.fields import (
    UnfoldAdminAutocompleteModelChoiceField,
    UnfoldAdminMultipleAutocompleteModelChoiceField,
)

from .models import MyModel


# Step 1: Create the autocomplete view
class MyAutocompleteView(BaseAutocompleteView):
    model = MyModel

    def dispatch(self, request, *args, **kwargs):
        # Permission checks here
        return super().dispatch(request, *args, **kwargs)

    def get_queryset(self):
        term = self.request.GET.get("term", "")

        if term == "":
            return super().get_queryset()

        return super().get_queryset().filter(my_field__icontains=term)


class MyForm(forms.Form):
    # Step 3a: Single-select autocomplete
    one_object = UnfoldAdminAutocompleteModelChoiceField(
        label=_("Object - Single value"),
        queryset=MyModel.objects.all(),
        url_path="admin:custom_autocomplete_path_name",
    )

    # Step 3b: Multi-select autocomplete
    multiple_objects = UnfoldAdminMultipleAutocompleteModelChoiceField(
        label=_("Objects - Multiple values"),
        queryset=MyModel.objects.all(),
        url_path="admin:custom_autocomplete_path_name",
    )


@admin.register(MyModel)
class MyModelAdmin(ModelAdmin):
    # Step 2: Register the view URL
    custom_urls = (
        (
            "autocomplete-url-path",
            "custom_autocomplete_path_name",
            MyAutocompleteView.as_view(),
        ),
    )
```

### SQL Query Optimization for Large Tables

The `ModelChoiceField` loads the full queryset on GET requests. For large tables, override the queryset per request method:

```python
class MyForm(forms.Form):
    one_object = UnfoldAdminAutocompleteModelChoiceField(
        label=_("Object - Single value"),
        queryset=MyModel.objects.none(),         # start empty
        url_path="admin:custom_autocomplete_path_name",
    )

    def __init__(self, request, *args, **kwargs):
        if request.method == "POST":
            # Only validate against submitted IDs
            selected_ids = request.POST.getlist("one_object")
            self.fields["one_object"].queryset = MyModel.objects.filter(
                pk__in=selected_ids
            )
        super().__init__(request, *args, **kwargs)
```

---

## 2. JsonField — Syntax Highlighting

Unfold renders `JSONField` values with syntax highlighting when in `readonly_fields` and `pygments` is installed.

### Basic Usage

```python
@admin.register(MyModel)
class MyModelAdmin(ModelAdmin):
    readonly_fields = ["data"]
```

### With Pretty Printing

```python
import json

class PrettyJSONEncoder(json.JSONEncoder):
    def __init__(self, *args, indent, sort_keys, **kwargs):
        super().__init__(*args, indent=4, sort_keys=True, **kwargs)

class MyModel(models.Model):
    data = models.JSONField(null=True, blank=True, encoder=PrettyJSONEncoder)
```

### Syntax Highlighting

```bash
pip install pygments
```

With `pygments` installed, Unfold automatically adds syntax highlighting to JSON rendered in readonly fields.

---

## 3. Common Mistakes

- **Autocomplete returns no results** → the related model's admin needs `search_fields` for the autocomplete filter; for field autocomplete, check `get_queryset()` filtering logic
- **Autocomplete too slow** → override queryset to use `.none()` on GET; only load full queryset for POST validation
- **JsonField raw text instead of formatted** → not in `readonly_fields`, or `pygments` not installed
- **custom_urls not working** → must be a tuple of tuples, each with `(url_path, name, view)` — not a list

---

→ For autocomplete **filter** (changelist sidebar), see `references/filters.md`
→ For widgets (ArrayWidget, WysiwygWidget), see `references/widgets.md`
→ For conditional fields, sections, datasets, see `references/forms-fields.md`
