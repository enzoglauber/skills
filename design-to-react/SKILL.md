---
name: design-to-react
description: Converts UI designs (screenshots, Figma frames, Pencil .pen frames) into production-ready React 19 components using TypeScript strict, Tailwind CSS v4, Base UI, tailwind-variants and tailwind-merge. Use when the user attaches a design/screenshot, asks to implement a layout or component from Figma/.pen, or to create/refactor React UI components following this stack.
---

# Design to React

Convert the attached design into React components that follow the rules below. Act as a tech lead: rules first, code second.

## Workflow

1. **Analyze the design**: list components, variants (`variant`, `size`...), interactive states (hover, focus, disabled, selected, open) and the colors/spacing/radii used.
2. **Check the project**: read `package.json` and the global CSS (`@theme` block). Reuse existing components, tokens and the icon library already installed. If a dependency of the stack is missing, tell the user which to install.
3. **Map colors to theme tokens**: never hardcode hex/rgb or palette classes like `bg-zinc-900`. If a needed token is missing, propose adding it to `@theme`.
4. **Generate code**: one component (or compound family) per file.
5. **Run the checklist** at the end and fix anything that fails.

## Stack

- React 19 (no `forwardRef`; `ref` is a regular prop)
- TypeScript strict
- Tailwind CSS v4 with `@theme` and CSS variables
- Base UI (`@base-ui/react`) for headless/interactive components
- `tailwind-variants` (`tv`) for variants
- `tailwind-merge` (`twMerge`) for class merging
- Lucide React or Phosphor Icons

## Naming

- Files in kebab-case: `user-card.tsx`, `use-modal.ts`
- Named exports only, never `export default`
- No barrel files (`index.ts`) for internal folders

## Component rules

```tsx
// Variants with tv(), exported as `<name>Variants`
export const buttonVariants = tv({ base: [...], variants: {...}, defaultVariants: {...} })

// Always merge the consumer className last
className={twMerge(buttonVariants({ variant, size }), className)}

// Identify every part with data-slot
<div data-slot="card-header">

// States via data-attributes, styled with data-[state]:
data-disabled={disabled ? '' : undefined}
className="data-[disabled]:opacity-50 data-[selected]:bg-primary"

// Focus visible on every interactive element
'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring'

// Icon sizing
<Check className="size-4" />
'[&_svg]:size-3.5' // inside variants

// Icon-only buttons need aria-label
<button type="button" aria-label="Close"><X className="size-4" /></button>

// Spread props last
{...props}
```

- Native `<button>` defaults to `type="button"`.
- Compound components: export each part (`Card`, `CardHeader`, `CardTitle`, `CardContent`) from the same file.

## TypeScript

```tsx
import type { ComponentProps } from 'react'
import { tv, type VariantProps } from 'tailwind-variants'

export interface ButtonProps
  extends ComponentProps<'button'>,
    VariantProps<typeof buttonVariants> {}
```

- Extend `ComponentProps<'element'>` (+ `VariantProps` when there are variants).
- Use `import type` for types.
- No `React.FC`, no `any`.

## Theme tokens

```
bg-surface, bg-surface-raised       -> backgrounds
bg-primary, bg-secondary, bg-muted  -> actions/states
bg-destructive                      -> errors/danger
text-foreground                     -> main text
text-foreground-subtle              -> secondary text
text-muted-foreground               -> disabled text
text-primary-foreground             -> text on primary
border-border, border-input         -> default borders
border-primary, border-destructive  -> highlighted borders
ring-ring                           -> focus ring
```

Prefer the project's existing token names when they differ.

## Base UI

Use Base UI for Dialog, Tabs, Select, Menu, Popover, Tooltip, etc. Style its parts with `className` + `data-[state]` attributes it exposes. Anatomy examples in [reference.md](reference.md).

## Next.js

Add `'use client'` at the top of components that use state, effects, event handlers or Base UI interactive parts.

## Checklist

- [ ] File in kebab-case
- [ ] Named export
- [ ] `ComponentProps<'element'>` + `VariantProps`
- [ ] Variants with `tv()`, classes with `twMerge()`
- [ ] `data-slot` on every part
- [ ] States via `data-[state]:`
- [ ] Theme colors only (nothing hardcoded)
- [ ] `focus-visible` on interactive elements
- [ ] `aria-label` on icon-only buttons
- [ ] `{...props}` last

## Additional resources

- Full examples (Button, Card, Base UI, `@theme`): [reference.md](reference.md)
