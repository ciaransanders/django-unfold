# Custom Branch (`custom`) — Rules & Feature Inventory

This branch is a fork of upstream **django-unfold** with personal customizations by
Ciaran Sanders on top of `main`. When merging/rebasing `main` into `custom`, the
features below MUST be preserved. All are unique to `custom` and not in `main`.

Diff baseline: `git diff main...custom`.

## 1. Python API additions

### `list_filter_buttons_top` ModelAdmin option
- `src/unfold/admin.py`: new `list_filter_buttons_top = False` flag on `ModelAdmin`.
- Documented in `docs/configuration/modeladmin.md`.
- Renders the filter Apply/Clear action buttons at the **top** of the list filter
  panel instead of the bottom (`unfold/helpers/change_list_filter.html` +
  `change_list_filter_actions.html` conditionally include the actions block based on
  this flag and adjust padding/margins).

### `UnfoldRelatedFieldWidgetWrapper` + related-objects lookup ("magnifying glass")
- `src/unfold/widgets.py`: new `UnfoldRelatedFieldWidgetWrapper(RelatedFieldWidgetWrapper)`.
  - Adds a search/lookup (magnifying glass) link that opens the related model's
    changelist. Overrides `get_related_url` / `get_context` to inject `related_url`.
  - Supports `limit_choices_to` (incl. multi-value list filters) by URL-encoding the
    limits onto the changelist URL as query filters.
- `src/unfold/mixins/base_model_admin.py`: `formfield_for_dbfield` now swaps in
  `UnfoldRelatedFieldWidgetWrapper` and passes through the related admin's
  add/change/delete/view permissions. If the related model has **no registered admin**,
  the wrapper is omitted.
- `src/unfold/templates/unfold/widgets/related_widget_wrapper.html`: renders the
  `related_url` magnifying-glass (`material-symbols-outlined: search`) link.
- `src/unfold/static/admin/js/admin/RelatedObjectLookups.js`:
  `dismissRelatedLookupPopup` extended to handle `<select>` (select2 autocomplete)
  targets — injects a new `Option` and dispatches `change`; now also receives/uses the
  chosen item's display name.

> **Removed:** a `fieldset_has_required` template filter used to live here (it drew a red
> `*` on fieldset tabs containing required fields). Its template usage in
> `fieldsets_tabs.html` was already lost in an earlier `main` merge, leaving the filter
> orphaned, so it was deleted. If required-field tab markers are wanted again, both the
> filter and its `{% if fieldset|fieldset_has_required %}*{% endif %}` usage must be
> re-added.

## 2. Template / layout changes

- **`base_simple.html`**: added an overridable `{% block content_classes %}` around the
  `#content` container classes.
- **Form error placement**: the non-field/error note block was moved OUT of
  `admin/change_form.html` and INTO `admin/base.html` so errors render **above the inline
  tabs** on all admin pages.
- **Capitalized labels**:
  - `admin/filter.html`: filter title uses `|title`.
  - `unfold/helpers/form_label.html`: field label uses `|title`.
- **`app_list.html`**: model links get `pl-4` left padding.
- **Tab UX**: `fieldsets_tabs.html`, `tab_items.html`, `theme_switch.html` get
  `select-none hover:cursor-pointer`. Tab anchors had their `href="#..."` fragments
  **removed** so switching tabs no longer appends a URL fragment.
- **`tab_items.html`**: removed the old warning indicator (superseded by upstream's
  error-count badge).
- **`admin/change_list_results.html`**: removed the `whitespace-nowrap` Tailwind class
  from the `#result_list` `<table>` so changelist result cells wrap instead of forcing a
  single line.

## 3. CSS (`src/unfold/styles.css`)

A dedicated `/* Custom Styles - Ciaran Sanders */` block adds:
- Changelist link styling: `#result_list tbody tr th a` / `td a` → primary color.
- `.thermaline-link` utility class → primary-colored link styling.
- Readonly field distinction (non-compact mode):
  `td[class*="field-"] input[readonly]` gets muted `base-100/base-800` bg + default cursor.
- Changeform readonly links: `form .readonly a` → primary color.
- Autocomplete min width: `.tabular-table .select2.select2-container { min-w-80 }`.
- Minor: `x-cloak` rule changed `@apply hidden!` → `@apply !hidden`.

`src/unfold/static/unfold/css/styles.css` (compiled Tailwind output) is a build artifact
reflecting the above — regenerate via the project build rather than hand-editing.

## Merge / rebase watch-list (`main` → `custom`)

1. **`select2.init.js` — already reconciled to `main`.** This branch previously carried a
   whitespace-only reformat of that file; meanwhile `main` added real features
   (`"use strict"`, `closeOnSelect`, and a `formset:added` re-init listener). The file has
   now been replaced with `main`'s current version, so it is byte-identical to `main` and
   will merge with zero conflict. Do **not** reintroduce the reformat.
2. **Warning-icon tab feature** was added then removed here; upstream's error-count badge
   is the replacement — don't reintroduce the warning icon.
3. Compiled `static/unfold/css/styles.css` will regenerate on build; resolve conflicts in
   the **source** `src/unfold/styles.css`, not the compiled file.
