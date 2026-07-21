# Chub.ai Sanitization and Styling Conventions

This repository contains HTML and CSS intended to render inside chub.ai and, in some cases, JanitorAI. Those hosts sanitize user-authored markup before it reaches the shared page. Treat sanitization as part of the runtime.

## Constructs known to be stripped or blocked

Avoid relying on these constructs in deployable Chub snippets:

- CSS `@import` rules.
- CSS `url()` and legacy `expression()` values. Do not depend on them for fonts, backgrounds, hero art, logos, or decorative images.
- `<script>` tags.
- Inline event handlers such as `onclick` and `onerror`.
- `javascript:` URLs.
- `id` attributes. Chub may remove them, so interactions must not depend on fragment targets, `for`/`id` pairs, or selectors that require an ID.

This list records behavior observed by the project. Host sanitizers can change, so recheck deployed cards after substantial host updates.

## Recommended CSS delivery

The primary delivery pattern is a self-contained inline `<style>` block immediately above the card markup. The full component stylesheet travels with the card and does not depend on a second request surviving sanitization.

For hosts that explicitly permit external stylesheets, `<link rel="stylesheet" href="...">` is a fallback. It is not the Chub-first path and should be tested on the target host.

This replaces the previous pattern of placing a repository-hosted stylesheet inside an `@import`. Chub blocks or removes that hop. Inline CSS makes the deployed artifact deterministic and fixes Moriae's previous state, where `Moriae-UI.html` shipped without any CSS delivery mechanism.

External font stylesheets follow the same rule. The safe template uses system and websafe stacks. A permitted host may load fonts through a `<link>` fallback, but the template must remain readable when that request is absent.

## Repository scoping conventions

Chub renders cards inside a shared host document. Generic selectors can collide with host styles or another card, so reusable systems use unique prefixes:

- `srv-` for the generic character-card system.
- `bb-` for book, branching, and Moriae-derived components.
- `bbx-` for `bb-` media and presentation assets such as background, hero, logo, divider, and support images.
- `ss-` for Sin Seers-specific layouts.

Keep component selectors within the appropriate prefix unless a selector intentionally targets host chrome.

### Double-class self-compounding

Selectors such as `.srv-wrapper.srv-wrapper`, `.bb-page.bb-page`, and `.bb-container.bb-container` repeat a class to raise specificity without introducing an `id` or coupling the component to a fragile host ancestor. Use this in override layers where Chub-injected styles otherwise win.

### Inline per-card theming

Set CSS custom properties on the card's root wrapper:

```html
<div class="bb-page bb-page" style="
  --bb-accent:#ff7a1a;
  --bb-accent-rgb:255,122,26;
  --bb-secondary:#4e008a;
  --bb-secondary-rgb:78,0,138;
">
```

This keeps the component library reusable while each card carries its own palette. Inline custom properties are preferred over generating a new stylesheet for every character.

## HOST PLATFORM HIDES

Stylesheets may contain a clearly labelled `HOST PLATFORM HIDES` section. It suppresses JanitorAI/Chub Ant Design chrome that competes with the custom card, including:

- ribbon labels and ribbon wrappers;
- lock-icon rows;
- generated spacer elements;
- duplicated typography/header rows;
- message-markdown wrappers and adjacent presentation scaffolding;
- selected platform controls that the existing layout replaces.

These selectors are intentionally host-aware and commonly use `:has()` plus `!important`. Keep them grouped instead of scattering Ant Design overrides through component rules. When the host DOM changes, repair this layer in one place.

## Readability override layer

The final section of a deployable stylesheet is a high-specificity readability override layer. It is appended last so card text colors, shadows, table cells, headings, labels, and muted copy defeat host-injected base styles.

The expected order is:

1. variables and system-font defaults;
2. host-platform hides;
3. base component styles;
4. interactions, animations, and responsive rules;
5. self-compounded readability overrides.

Do not move the readability layer earlier in the cascade.

## Images without CSS `url()`

Deliver card-owned art through markup-level `<img>` elements:

- `.bbx-bg-image` for the fixed page background;
- `.bbx-hero-image` for the hero;
- `.bbx-title-logo` for the title/logo formerly injected through `.bb-title::after`;
- `.bb-mid-image` for dividers;
- `.bb-quest-img` for scenario art;
- `.bbx-support-image` and normal support-row images.

Moriae's former `--bb-hero-fallback` and CSS title-logo background are intentionally absent from the safe template. The mobile layout uses the same markup hero image rather than creating a pseudo-element background.

## Pre-deployment audit

Before pasting a card into Chub:

1. Confirm the deployable snippet contains no active `@import`, `expression()`, `<script>`, inline event handler, `javascript:` URL, or required `id`.
2. Search the stylesheet for `url(` and move card-owned imagery into `<img>` markup.
3. Confirm the leading Google-Fonts import from `Moriae-Ui.css` has not returned.
4. Verify the host-platform hides, keyframe animations, mobile breakpoints, reduced-motion behavior, and final readability override layer are still present.
5. Open the companion preview file locally, then test the deployable snippet in Chub because host sanitization cannot be fully simulated in a standalone browser.
