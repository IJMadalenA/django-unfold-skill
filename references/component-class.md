# Component Class Reference — Python-Based Components

> **Related files:** `references/dashboard-components.md` (template-based components: card, chart, table, etc.),
> `references/configuration.md` (DASHBOARD_CALLBACK in UNFOLD dict),
> `SKILL.md` (dashboard setup overview)

---

## Overview

Unfold's Python component classes let you encapsulate data preparation logic in a reusable class. They are **optional** — you can pass data directly to templates via `DASHBOARD_CALLBACK`. Component classes are useful when:

- The same data logic is needed in multiple places
- You want clean separation between data logic and templates
- Components need their own methods, queries, or permissions

---

## 1. Basic Component Class

```python
# admin.py or components.py
from unfold.components import BaseComponent, register_component
from myapp.models import Order
from django.db.models import Sum


@register_component
class RevenueCard(BaseComponent):
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["revenue"] = (
            Order.objects.aggregate(total=Sum("total"))["total"] or 0
        )
        return context
```

### In Template

```django
{% load unfold %}

{% component "unfold/components/card.html" with component_class="RevenueCard" %}
  {% component "unfold/components/title.html" %}{{ revenue }}{% endcomponent %}
{% endcomponent %}
```

Unfold instantiates `RevenueCard`, calls `get_context_data()`, and injects the returned context into the component template.

---

## 2. Component Class with Parameters

The `component_class` parameter can also be followed by additional keyword arguments that are passed to the component:

```django
{% component "unfold/components/card.html" with component_class="OrderStatsCard" period="monthly" %}
{% endcomponent %}
```

```python
@register_component
class OrderStatsCard(BaseComponent):
    def get_context_data(self, period="monthly", **kwargs):
        context = super().get_context_data(**kwargs)
        if period == "monthly":
            context["stats"] = self.get_monthly_stats()
        else:
            context["stats"] = self.get_yearly_stats()
        return context

    def get_monthly_stats(self):
        from django.db.models import Count
        return Order.objects.filter(
            created_at__month=__import__("datetime").date.today().month
        ).aggregate(count=Count("id"))

    def get_yearly_stats(self):
        from django.db.models import Count
        return Order.objects.filter(
            created_at__year=__import__("datetime").date.today().year
        ).aggregate(count=Count("id"))
```

---

## 3. When to Use Component Class vs DASHBOARD_CALLBACK

| Approach | Best for |
|----------|----------|
| `DASHBOARD_CALLBACK` | Simple data injection into one dashboard template |
| Component class | Reusable widgets used across multiple pages/dashboards |
| Inline in template | One-off display with no data processing |

For dashboards, you'll often use both: `DASHBOARD_CALLBACK` for the page-level KPI values, and component classes for reusable chart/table widgets.

---

## 4. Common Mistakes

- **Component not rendering** → decorator `@register_component` is required — without it, Unfold can't find the class
- **Context data not appearing in template** → `get_context_data()` MUST call `super().get_context_data(**kwargs)` first
- **Component works locally but not on another page** → component classes are registered globally; ensure the module is imported (e.g., in `apps.py` ready() or in `admin.py`)

---

→ For all available template-based components (card, chart, table, tracker, etc.), see `references/dashboard-components.md`
→ For the DASHBOARD_CALLBACK setting, see `references/configuration.md`
