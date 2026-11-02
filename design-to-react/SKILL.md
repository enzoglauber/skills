---
name: design-to-react
description: Converts UI designs (screenshots, Figma frames, Pencil .pen frames) into production-ready React 19 components using TypeScript strict, Tailwind CSS v4, shadcn/ui on Radix, tailwind-variants and tailwind-merge. Use when the user attaches a design/screenshot, asks to implement a layout or component from Figma/.pen, or to create/refactor React UI components following this stack.
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
- shadcn/ui on Radix (`radix-ui` unified package) for headless/interactive components
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

## Radix (via shadcn/ui)

Use Radix primitives for Dialog, Sheet, Tabs, Select, Accordion, DropdownMenu, Popover, Tooltip, etc. Import from the unified package: `import { Dialog as DialogPrimitive } from "radix-ui"`.

- Add components with `npx shadcn@latest add <component>`, then adapt the generated file: swap `cn()`/`cva` for `twMerge`/`tv()`, use named exports and `import type`, apply the project tokens and translate labels (e.g. `aria-label="Fechar"`). Never let `@/lib/utils` or `class-variance-authority` back in.
- Wrap each part in its own exported function with `data-slot`, extending `ComponentProps<typeof XPrimitive.Part>`.
- Use `asChild` (Radix `Slot`) to render a primitive as your own component (`<DialogTrigger asChild><Button /></DialogTrigger>`).
- Style states with the attributes Radix exposes: `data-[state=open]`, `data-[state=closed]`, `data-[state=active]`, `data-[state=checked]`, `data-disabled`, `data-highlighted`, `data-[orientation=vertical]`, `data-[side=top]`, `aria-expanded`.
- Positioned content uses Radix CSS variables: `max-h-(--radix-select-content-available-height)`, `origin-(--radix-popover-content-transform-origin)`.

Anatomy examples in [reference.md](reference.md).

## Responsive & touch (mobile-first)

A single element wider than the viewport makes the whole page scroll sideways on mobile and breaks every `fixed` overlay: mobile browsers grow the layout viewport to the content width, so a centered Dialog (`fixed inset-0 m-auto`) is centered on the wider page, not on the screen. Treat horizontal overflow as a bug, never hide it with `overflow-x-hidden` on `html`/`body`.

```tsx
// Grids: always declare the mobile column. Implicit columns grow to the min-content of the widest child
'grid grid-cols-1 gap-6 md:grid-cols-2'
'md:grid-cols-[minmax(0,1fr)_auto]' // custom tracks: minmax(0,1fr), never bare 1fr next to wide content

// Flex/grid children holding wide content (tabs, tables, code, long words) need min-w-0
'flex min-w-0 flex-col'

// Rows that can exceed the screen (tabs, chips, toolbars) scroll inside themselves
'flex max-w-full min-w-0 overflow-x-auto overscroll-x-contain touch-pan-x [scrollbar-width:none] [&::-webkit-scrollbar]:hidden'
'shrink-0 whitespace-nowrap' // on each item of that row

// Fixed-size widgets (calendar, media, cards) are fluid on mobile and fixed from sm up
'w-full sm:w-fit'
'min-w-0 sm:min-w-(--cell-size)'
'aspect-square w-full sm:h-90 sm:w-80' // skeleton matches the real widget at every breakpoint

// Padding scales with the screen
'p-5 sm:p-6 md:p-8'
```

- Order the DOM for mobile reading order (fields → date → submit). Rearrange desktop with grid placement (`md:col-start-2 md:row-span-2`), not by duplicating markup.
- Touch targets: at least 24×24px (WCAG 2.2 AA), aim for 40px+ on tabs, chips and icon buttons. Text-like buttons (`variant="link"`) get `min-h-6`. A card link can cover the whole card with `after:absolute after:inset-0`.
- Add `touch-manipulation` to tappable items inside scrollable rows.
- Tailwind v4 `hover:` only applies under `@media (hover: hover)`, so touch devices never show hover. Never put essential feedback only on hover; use `active:`, `focus-visible:` or `data-[state]`.
- Overlays: `max-w-[calc(100%-2rem)]`, `max-h-[calc(100dvh-2rem)] overflow-y-auto`, smaller padding on mobile (`p-6 sm:p-8`).

**Verify** at 320, 360, 390, 768, 1024 and 1280px. `document.documentElement.scrollWidth` must equal `clientWidth`. To find the culprit, list elements whose `getBoundingClientRect().right` exceeds the viewport and that have no ancestor with `overflow-x` set to `auto`, `scroll`, `hidden` or `clip`. Open a Dialog at the smallest width and check that the left and right margins are equal.

## Next.js

Add `'use client'` at the top of components that use state, effects, event handlers or Radix interactive parts.

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
- [ ] Every grid declares `grid-cols-1` for mobile; custom tracks use `minmax(0,1fr)`
- [ ] Wide rows (tabs, chips) scroll inside themselves; their parents have `min-w-0`
- [ ] Fixed-size widgets are fluid below `sm`
- [ ] No horizontal page scroll from 320px up (`scrollWidth === clientWidth`); Dialog centered at 320px
- [ ] Touch targets ≥ 24px; no essential feedback that depends only on `hover:`
- [ ] DOM order matches the mobile reading order

## Additional resources

- Full examples (Button, Card, Radix, `@theme`): [reference.md](reference.md)
