# Decorators Reference — @display and @action

> **Related files:** `references/actions.md` (usage in actions_list/row/detail/submitline),
> `../SKILL.md` (Quick Reference: @display decorator usage)

---

## @display Decorator

**Always use `unfold.decorators.display`**, not Django's built-in `django.contrib.admin.decorators.display`.

```python
from unfold.decorators import display
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `description` | str | — | Column header text |
| `ordering` | str | — | Field to order by when sorting this column |
| `label` | bool/dict | False | `True` = default colors; `dict` = value-to-color mapping |
| `header` | bool | False | Two-line cell (return tuple of 2-3 elements) |
| `dropdown` | bool | False | Interactive dropdown menu in cell |
| `boolean` | bool | False | Render as on/off icon |

### label — Colored Status Badges

```python
@display(
    description="Status",
    ordering="status",
    label=True,  # default colors
)
def show_status(self, obj):
    return obj.status
```

**Custom color mapping (dict):**

```python
@display(
    description="Status",
    ordering="status",
    label={
        "PENDING":   "warning",   # orange
        "ACTIVE":    "info",      # blue
        "COMPLETED": "success",   # green
        "FAILED":    "danger",    # red
    },
)
def show_status(self, obj):
    return obj.status
```

**Available colors:** `success` (green), `info` (blue), `warning` (orange), `danger` (red)

### header — Two-Line Cells

```python
@display(header=True)
def show_customer(self, obj):
    return obj.full_name, obj.email   # tuple: (main, subtitle)
```

**With optional third argument** (initials circle or image):

```python
@display(header=True)
def show_customer(self, obj):
    return [
        "First main heading",
        "Smaller additional description",
        "AB",  # initials displayed in circle
        # OR image instead of initials:
        {
            "path": "some/path/picture.jpg",
            "squared": True,
            "borderless": True,
            "width": 64,
            "height": 48,
        },
    ]
```

### dropdown — Interactive Dropdown in Cell

```python
@display(description="Actions", dropdown=True)
def show_dropdown(self, obj):
    return {
        "title": "Custom title",
        "striped": True,
        "height": 200,
        "width": 240,
        "items": [
            {"title": "First", "link": "#"},
            {"title": "Second", "link": "#"},
        ],
    }
```

**Custom template dropdown:**
```python
@display(description="Actions", dropdown=True)
def show_dropdown(self, obj):
    return {
        "title": "Actions",
        "content": "template content or rendered HTML",
    }
```

---

## @action Decorator

**Always use `unfold.decorators.action`**, not Django's built-in.

```python
from unfold.decorators import action
from unfold.enums import ActionVariant
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `description` | str | required | Button label text |
| `icon` | str | None | Material Symbols icon name |
| `variant` | ActionVariant | DEFAULT | Button color style |
| `url_path` | str | None | Custom URL path segment |
| `attrs` | dict | None | Extra HTML attributes on `<a>` |
| `permissions` | list | None | Permission checks |

### Action Variants

```python
ActionVariant.DEFAULT  # default gray
ActionVariant.PRIMARY  # blue
ActionVariant.SUCCESS  # green
ActionVariant.INFO     # light blue
ActionVariant.WARNING  # orange
ActionVariant.DANGER   # red
```

### Permissions

```python
@action(
    description="Approve",
    permissions=["approve_order"],           # calls has_approve_order_permission()
    # OR:
    permissions=["orders.approve_order"],    # Django permission string
)
def approve_order(self, request, object_id):
    pass

def has_approve_order_permission(self, request, obj=None):
    return request.user.has_perm("orders.approve_order")
```

### HTML Attributes

```python
@action(
    description="View Receipt",
    attrs={"target": "_blank"},
)
def view_receipt(self, request, object_id):
    ...
```

---

## Common Mistakes

- **Using Django's built-in `@display` or `@action`** — Unfold's decorators support extra params (label, header, icon, variant, etc.)
- **Forgetting `@action(description=...)`** — required for all action types
- **Using wrong import path** — `unfold.decorators.display` and `unfold.decorators.action`, not `unfold.admin.display`
