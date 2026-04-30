# Skill: Django Unfold (para Claude / OpenCode)

> Un skill diseñado para Claude y agentes compatibles que proporciona conocimiento profundo y actualizado sobre [django-unfold](https://github.com/unfoldadmin/django-unfold), el tema moderno de administración para Django basado en Tailwind CSS.

---

## ¿Qué es este proyecto?

Este repositorio contiene un **skill de agente IA** que permite a Claude (y agentes compatibles con MCP) generar código, depurar problemas y configurar paneles de administración de Django utilizando `django-unfold` de forma precisa y alineada con la documentación oficial.

Cubre la versión **0.90.x** de django-unfold (requiere Django 5.0+).

---

## ¿Qué es django-unfold?

[Django Unfold](https://unfoldadmin.com/docs/) es un **tema moderno de administración para Django** construido sobre Tailwind CSS. Es una mejora directa (*drop-in*) de `django.contrib.admin` — totalmente compatible con todos los patrones nativos de Django admin, pero con:

- Componentes UI ricos (tarjetas, gráficos, tablas, trackers)
- Navegación por pestañas en formularios y listados
- Acciones con diálogos de confirmación modales
- Filtros avanzados (texto, fecha, desplegable, numérico, autocompletado)
- Inlines ordenables con drag-and-drop
- Widgets WYSIWYG y ArrayWidget
- Campos condicionales con Alpine.js
- Dashboards personalizables
- Paleta de colores OKLCH
- Integraciones con paquetes de terceros

---

## Estructura del skill

El skill utiliza una arquitectura de **divulgación progresiva** en tres niveles:

### Nivel 1: Metadata (siempre en contexto)
- `SKILL.md` — nombre, descripción y compatibilidad

### Nivel 2: Cuerpo del skill (cuando se activa)
- `SKILL.md` — reglas críticas, referencias rápidas, errores comunes, red flags

### Nivel 3: Recursos empaquetados (bajo demanda)
📁 `references/` — 15 archivos de referencia especializados:

| Archivo | Contenido |
|---------|-----------|
| `installation.md` | Instalación, INSTALLED_APPS, re-registro de User/Group, collectstatic |
| `configuration.md` | Diccionario UNFOLD completo, callbacks, branding |
| `theming.md` | Paleta OKLCH, sidebar, iconos, login page, Tailwind |
| `actions.md` | actions_list, row, detail, submit_line, **dialog actions**, permisos |
| `filters.md` | Filtros: texto, fecha, desplegable, numérico, autocompletado, checkbox/radio |
| `tabs.md` | Changelist tabs, changeform tabs, fieldset tabs, inline tabs, dynamic tabs |
| `inlines.md` | StackedInline, TabularInline, sortable, paginated, nested, nonrelated |
| `decorators.md` | `@display` (header, label, dropdown) y `@action` (variant, permisos) |
| `dashboard-components.md` | Componentes de template: card, chart, table, tracker, progress, etc. |
| `component-class.md` | Clases Python con `BaseComponent` y `register_component` |
| `forms-fields.md` | Campos condicionales, secciones expandibles, datasets, crispy forms |
| `fields.md` | Autocomplete fields (`BaseAutocompleteView`) y JsonField con syntax highlighting |
| `widgets.md` | WysiwygWidget (Trix) y ArrayWidget |
| `styles-scripts.md` | Carga CSS/JS, personalización con Tailwind 4 vs 3 |
| `advanced.md` | Páginas custom, custom sites, sortable changelist, command palette, multi-language, paginator |
| `integrations.md` | django-import-export, celery, guardian, simple-history, constance, modeltranslation, etc. |

---

## Cómo usar este skill

### En Claude Code / OpenCode
1. Coloca este repositorio en el directorio de skills de tu agente (ej. `.agents/skills/`)
2. El skill se activa automáticamente cuando Claude detecta referencias a django-unfold, admin customization, o cualquiera de los triggers definidos en la descripción

### En Claude.ai
- Empaqueta el skill con el script `package_skill.py` (si está disponible)
- Instálalo en tu entorno de Claude.ai

---

## Qué cubre el skill

- **✅ Instalación y configuración inicial** — desde `INSTALLED_APPS` hasta `collectstatic`
- **✅ Herencia correcta** — `unfold.admin.ModelAdmin`, `StackedInline`, `TabularInline`
- **✅ Personalización visual** — colores OKLCH, sidebar, login page, favicons, logos
- **✅ Dashboards** — templates, componentes, gráficos con Chart.js, tablas, KPIs
- **✅ Decoradores** — `@display` con badges coloreados, encabezados de dos líneas; `@action` con variantes y permisos
- **✅ Acciones con diálogos** — modales de confirmación y formularios custom intermedios
- **✅ Filtros avanzados** — todos los tipos de filtro de `unfold.contrib.filters`
- **✅ Tabs** — en changelist, changeform, fieldsets e inlines
- **✅ Inlines** — ordenables, paginados, anidados, nonrelated
- **✅ Widgets** — WysiwygWidget (Trix), ArrayWidget con choices
- **✅ Campos** — autocomplete fields, JsonField con syntax highlighting
- **✅ Formularios** — campos condicionales (Alpine.js), secciones expandibles, datasets, crispy forms
- **✅ Páginas custom** — vistas con `UnfoldModelAdminViewMixin`, navegación en sidebar
- **✅ Styling** — CSS/JS custom, integración con Tailwind 4
- **✅ Integraciones** — 10 paquetes de terceros con configuración Unfold
- **✅ Errores comunes** — red flags y errores detectados en testing de baseline

---

## Desarrollo del skill

Este skill fue creado y refinado siguiendo el proceso del **Skill Creator**:

1. Investigación de la documentación oficial (unfoldadmin.com/docs y GitHub docs)
2. Identificación de gaps respecto a la documentación original
3. Creación de archivos de referencia especializados
4. Validación de cross-references entre archivos
5. Casos de prueba en `evals/evals.json`

### Dependencias
- Python 3.10+
- Para empaquetar: `python -m scripts.package_skill <path>` (del skill-creator)

---

## Licencia

MIT License — véase `LICENSE.txt`

---

## Créditos

- [django-unfold](https://github.com/unfoldadmin/django-unfold) creado por unfoldadmin.com
- Este skill es una contribución comunitaria independiente, mantenida para el ecosistema de agentes IA
