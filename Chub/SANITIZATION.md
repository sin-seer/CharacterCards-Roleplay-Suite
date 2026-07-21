# Chub.ai Sanitization and Styling Conventions

This repository contains HTML/CSS intended to render inside chub.ai and, in some cases, JanitorAI. Those hosts sanitize user-authored markup before it reaches the shared page. Treat the sanitizer as part of the runtime, not as an optional deployment detail.

## Constructs that are stripped or blocked

Avoid these constructs in deployable Chub snippets:

- CSS `@import` rules.
- CSS `url()` values and legacy `expression()` values. In particular, do not depend on `url()` for card backgrounds, hero art, fonts, or decorative images.
- `<script>` tags.
- Inline event-handler attributes such as `onclick` and `onerror`.
- `javascript:` URLs.
- `id` attributes. Chub may strip them, so interactions must not depend on `id`, fragment targets, or `for`/`id` pairs.

This list documents behavior observed by the project. Host sanitizers can change, so every deployable file should be rechecked after substantial edits.

## Recommended CSS delivery

The primary delivery pattern is a self-contained inline `<style>` block placed directly above the card markup. The complete component stylesheet travels with the card and does not depend on a second request surviving the sanitizer.

For hosts that explicitly permit external stylesheets, `<link rel="stylesheet" href="...">` may be used as a fallback. It is not the Chub-first path.

This replaces the previous `@import` workflow. The old pattern imported a repository-hosted stylesheet from inside a `<style>` tag, but Chub blocks or removes that construct. Inline CSS removes the blocked hop and keeps the deployed artifact deterministic.

## Repository scoping conventions

Chub renders cards inside a shared host document. Generic selectors can collide with the platform or with another card, so this repository scopes reusable systems with unique class prefixes:

- `srv-` for the generic character-card component library.
- `bb-` for branching/book-style card components.
- `ss-` for Sin Seers layouts.

Keep new component selectors inside the appropriate prefix unless a selector intentionally targets host chrome.

### Double-class self-compounding

Selectors such as `.srv-wrapper.srv-wrapper` deliberately repeat the same class. The repetition raises specificity without introducing an `id` or coupling the component to a host-specific ancestor. Use this technique in override layers where Chub-injected styles otherwise win.

### Per-card themes

Cards set CSS custom properties inline on the root wrapper, for example:

```html
<div class="srv-wrapper" style="
  --theme-hex:#6E7BA8;
  --theme-rgb:110,123,168;
  --secondary-hex:#7FA98C;
  --secondary-rgb:127,169,140;
">
```

This keeps one component library reusable across many cards while allowing each card to carry its own palette. Inline custom properties are preferred over generating a new stylesheet per character.

## HOST PLATFORM HIDES

Several stylesheets include a section labelled `HOST PLATFORM HIDES`. It suppresses JanitorAI/Chub Ant Design chrome that competes with the custom card, including ribbons, lock-icon rows, generated spacer elements, typography headers, and message-markdown wrappers or siblings that duplicate the card presentation.

These selectors are intentionally host-aware and often use `:has()` plus `!important`. Keep them grouped and clearly labelled. When the host DOM changes, update this section rather than scattering new Ant Design overrides through component rules.

The deployable safe template retains this layer. It also hides host siblings around `.srv-wrapper` only where the existing convention already does so; broad host selectors should be treated carefully because they can remove future platform controls.

## Readability override layer

The `srv-` stylesheet ends with a high-specificity readability override layer. It is applied last in the cascade so text color, shadows, centered prose, chips, alternate-path colors, and other legibility fixes override host-injected base styles.

Preserve that ordering. Base component rules come first, scoped/self-compounded duplicates follow where needed, and the readability layer is appended after them. Later feature layers may extend it, but must not accidentally move host-resilience rules earlier in the cascade.

## Images without CSS `url()`

Supply card-owned images through markup-level `<img>` elements. The safe template uses an absolutely positioned `.srv-bg-image` for the card background and normal `<img>` elements for avatars, choices, galleries, and showcase/lightbox art.

Host-owned hero areas cannot always accept injected markup. The safe template therefore styles the hero container without requiring a CSS image URL. A host-provided hero may still display; otherwise the container falls back to gradients and the card palette.

## Pre-deployment audit

Before pasting a card into Chub, verify that the deployable snippet contains no `@import`, `expression()`, `<script>`, inline event handler, `javascript:` URL, or required `id`. Search for `url(` as well and move any required image into an `<img>` element. Confirm that mobile media queries, reduced-motion rules, host hides, and the final readability override are still present.
