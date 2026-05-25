---
name: svelte-component
description: Scaffold a new Svelte 5 component following project conventions (runes, named exports, arrow functions, TypeScript, ES6+). Use when asked to create a new component or when a component should be extracted.
---

When creating a Svelte component, follow this template and all rules below.

## Template

```svelte
<script lang="ts">
    // 1. Third-party imports
    // 2. Internal imports ($lib/...)

    // Props interface — always explicit
    interface Props {
        label: string;
        disabled?: boolean;
        onclick?: () => void;
        children?: import('svelte').Snippet;
    }

    const { label, disabled = false, onclick, children }: Props = $props();

    // Reactive state — $state for mutable, $derived for computed
    let count = $state(0);
    let doubled = $derived(count * 2);

    // Named arrow functions only — no function declarations
    const handleClick = () => {
        count++;
        onclick?.();
    };

    // Side effects — return a cleanup fn when needed
    $effect(() => {
        // setup
        return () => {
            // teardown
        };
    });
</script>

<div class="component-root">
    <button {disabled} onclick={handleClick}>
        {label}
    </button>

    {@render children?.()}
</div>
```

## Rules

**TypeScript**
- Always `<script lang="ts">`.
- Define a named `interface Props` for every component, even if it has no props yet.
- Destructure props directly from `$props()` with inline defaults.

**Functions**
- Arrow functions only: `const doThing = () => { }` — never `function doThing() {}`.
- Named exports only — no default exports from `.ts` utility files used alongside the component.

**Variables**
- Never `var`. Use `const` when the binding won't be reassigned, `let` otherwise.
- ES6+ always: destructuring, spread (`{ ...rest }`), optional chaining (`foo?.bar`), template literals.

**Svelte 5 runes**
- `$state()` for mutable reactive values.
- `$derived()` for values computed from state — never recompute in template expressions.
- `$effect()` for side effects; always return a cleanup function if setup allocates resources.
- `$props()` for component props — destructure at the top of `<script>`.
- Pass-through props use rest (`...rest`) and spread onto the root element.

**Snippets / slots**
- Use `{@render children?.()}` for default slot content.
- Typed as `import('svelte').Snippet` in the Props interface.

**Styles**
- Scoped `<style>` block or a co-located `.scss` file imported in `<script>`.
- Never inline `style=""` attributes.

## Checklist before finishing

- [ ] `interface Props` declared and destructured from `$props()`
- [ ] All functions are arrow functions
- [ ] No `var` anywhere
- [ ] Reactive state uses `$state` / `$derived` — no plain `let` for derived values
- [ ] `$effect` cleanup returned if any listeners/intervals/timeouts are registered
- [ ] Component is placed under `src/lib/components/<ComponentName>/` with its own directory
