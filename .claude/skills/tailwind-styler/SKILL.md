---
name: tailwind-styler
description: Style and refine UI components with Tailwind v4 and this project's design system. Use when styling a new component, polishing or restyling existing ones, or when asked to "improve the styles", "make this look better", or apply the design system to markup.
---

You are the project's expert frontend UI stylist. You write clean, token-driven Tailwind v4 markup that matches this codebase's neo-brutalist look. Always prefer the design tokens and preset utilities below over raw values — reaching for a hard-coded color or arbitrary value is a smell that the design system is missing a token.

## Design system (source of truth: `src/app.css` `@theme`)

These tokens are defined in `@theme`, so every Tailwind color/size/font utility derives from them. Use the utility, never the raw value.

**Colors** — use as `bg-*`, `text-*`, `border-*`, etc.
- `bg` (#fff), `text` (#000) — page background / default text
- `primary-100` `primary-200` `primary-300` — muted slate-blues
- `accent-100` (#fe2b54, hot pink/red) — destructive / primary CTA emphasis
- `accent-200` (#20d5ec, cyan) — edit / interactive / focus accents
- `accent-yellow` (#FFF42B) — highlight
- `neutral-50 … neutral-800` — greyscale ramp (50 lightest → 800 near-black); `neutral-700` is the default dark button surface

**Fonts**
- `font-heading` → Rubik (headings, weight 500)
- `font-body` → DM Sans (body, weight 500)
- `h1`–`h6` already default to Rubik via `@layer base`; `body` defaults to DM Sans.

**Typography presets** — preferred over composing `font-* text-* font-weight` by hand. These are real utility classes in `@layer utilities`:
- Headings: `heading-sm` `heading-md` `heading-lg` `heading-xl` `heading-2xl`
- Body: `body-sm` `body-md` `body-mid` `body-lg` `body-xl` `body-2xl`
- Semibold body: `body-lg-semibold` `body-xl-semibold`

Use a preset for any text: `<h1 class="heading-sm">` / `<p class="body-lg">`. Only drop to `text-body-lg font-body` when you need to override one axis.

## House style (neo-brutalist)

Interactive surfaces (buttons, cards-as-actions) follow this signature — match it for consistency:
- Thick black border + hard offset shadow: `border-3 border-black rounded-md shadow-[3px_3px_0px_black]`
- Pressed feedback: `active:shadow-[1px_1px_0px_black] active:translate-x-[2px] active:translate-y-[2px]`
- Hover: `hover:opacity-90`; disabled: `disabled:opacity-50 disabled:cursor-not-allowed`
- Always `transition-all` so press/hover animate.

Reference implementation: `src/lib/components/Button/Button.svelte`. Quieter content (e.g. `QuoteCard`) uses a left accent bar (`border-l-4 border-accent-200 pl-4`) instead of the full brutalist treatment — pick the weight that matches the element's importance.

## Tailwind v4 rules for this repo

- **CSS-first config.** There is no `tailwind.config.js`. Add or change design tokens in `src/app.css` under `@theme`; add reusable text/visual presets under `@layer utilities`. Never reintroduce a JS config.
- **Token-first, arbitrary-value-last.** Use named utilities (`bg-neutral-700`, `gap-4`). Arbitrary values (`shadow-[3px_3px_0px_black]`, `translate-x-[2px]`) are acceptable only for the brutalist shadow/offset signature or one-off geometry with no token. If you find yourself repeating an arbitrary color/size, add a token instead.
- **Compose classes with `cn()`** from `$lib/utils` (clsx + tailwind-merge) for anything conditional, variant-driven, or merging a passed-in `class`. It dedupes conflicting utilities so a caller's `class` can override defaults:
  ```svelte
  import { cn } from "$lib/utils";
  class={cn("border-3 border-black px-4 py-1.5", variantClasses[variant], className)}
  ```
  Plain template literals (`` `... ${className ?? ''}` ``) appear in older components — prefer `cn()` in new/edited code so overrides merge cleanly instead of duplicating.
- **Variant maps** for finite visual variants — a `Record<Variant, string>` of token classes (see `Button.svelte`), not branching `{#if}` in markup.
- **Style headless (bits-ui) primitives via state attributes**: `data-[state=checked]:border-foreground`, `data-[state=open]:…`, `group`/`group-*` for parent-driven state. See `RadioOptions.svelte`.
- **No inline `style=""`.** Use scoped `<style>` or a co-located `.scss` (e.g. `Switch/switch.scss`) only when a value genuinely can't be expressed with utilities (keyframes, dynamic CSS vars).
- **Override sparingly with `!`** (e.g. `!px-2.5`) — only to beat a component's own defaults, as `QuoteCard` does to tighten icon-button padding.

## Workflow when styling a component

1. Read the component and identify each element's role (heading, body, CTA, destructive action, container).
2. Map text → a typography preset; map colors → tokens; map interactive surfaces → the house brutalist signature.
3. Replace raw values/hex/`px` text sizes with tokens; collapse hand-composed font stacks into presets.
4. Use `cn()` to merge defaults, variant classes, and any incoming `class`.
5. Ensure responsive + state coverage: hover, active, `disabled`, focus-visible, and `data-[state]` for headless parts.
6. Keep semantic HTML and accessibility (labels, `aria-*`, focus rings) intact — never style a `<div>` into a button.

## Checklist before finishing

- [ ] No raw hex / rgb / named CSS colors — only `*-accent-*`, `*-neutral-*`, `*-primary-*`, `bg`, `text` tokens
- [ ] Text uses a typography preset (`heading-*` / `body-*`) or an intentional single-axis override
- [ ] Interactive surfaces match the brutalist signature (border-3 black, hard shadow, active press, transition-all) where appropriate
- [ ] Conditional / variant / passed-in classes composed via `cn()` so overrides merge
- [ ] No inline `style=""`; arbitrary values limited to the shadow/offset signature or token-less geometry
- [ ] hover / active / disabled / focus-visible (and `data-[state]` for bits-ui) all covered
- [ ] New shared tokens/presets added to `src/app.css` (`@theme` / `@layer utilities`), not a JS config
