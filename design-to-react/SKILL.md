---
name: design-to-react
description: Converts UI designs (screenshots, Figma frames, Pencil .pen frames) into production-ready React 19 components using TypeScript strict, Tailwind CSS v4, and shadcn/ui on Base UI. Verifies Tailwind and shadcn are installed before writing UI, then adds registry components with the shadcn CLI. Use when the user attaches a design/screenshot, asks to implement a layout or component from Figma/.pen, or to create/refactor React UI components following this stack.
---

# Design to React

Convert the attached design into React components that follow the rules below. Act as a tech lead: rules first, code second.

## Workflow

1. **Analyze the design**: list components, variants (`variant`, `size`...), interactive states (hover, focus, disabled, selected, open) and the colors/spacing/radii used.
2. **Run the preflight below**. Do not write or edit UI until Tailwind and shadcn are installed and the registry components the screen needs are on disk.
3. **Map colors to theme tokens**: never hardcode hex/rgb or palette classes like `bg-zinc-900`. If a needed token is missing, add it to `@theme`.
4. **Compose the screen** from the shadcn files in `components/ui`. One product component (or compound family) per file. Do not hand-roll a second copy of a registry primitive.
5. **Run the checklist** at the end and fix anything that fails.

## Preflight

Run this before any component work. If something is missing, install it and continue. Do not stop to ask the user to install it.

### Tailwind

`package.json` must include `tailwindcss` v4 and the integration this repo already uses (`@tailwindcss/postcss` or `@tailwindcss/turbopack`). The global CSS must start with `@import "tailwindcss"`. If either is missing, install the packages and add the import before continuing.

### shadcn

All of these must exist:

- `components.json` with `"style": "base-nova"` unless the repo already chose another style
- `src/lib/utils.ts` exporting `cn`
- dependencies: `@base-ui/react`, `class-variance-authority`, `tailwind-merge`, `lucide-react`, plus this repo’s `cn` (do not add unused `clsx` if `cn` already merges classes)

If `components.json` or `cn` is missing, run:

```bash
npx shadcn@latest init -d --base base
```

`init` rewrites CSS variables. After it finishes, put the project's existing palette back. Do not replace a designed theme with the CLI palette. Alias any shadcn token the project does not have (`accent`, `accent-foreground`, `input`, `card`, `card-foreground`, `popover`, `popover-foreground`, `radius`) to an existing token.

### Registry components

For every interactive primitive the design needs, the file `src/components/ui/<name>.tsx` must be the one the shadcn registry generated (`cn` from `@/lib/utils`, `cva` where the registry uses it, `data-slot`, Base UI anatomy). A hand-written lookalike does not count. If the file is missing or was not added by the CLI, run:

```bash
npx shadcn@latest add <name> --yes --overwrite
```

Typical set: `button`, `input`, `label`, `select`, `checkbox`, `radio-group`, `switch`, `slider`, `tabs`, `toggle`, `toggle-group`, `dialog`, `alert-dialog`, `sheet`, `dropdown-menu`, `tooltip`, `popover`, `calendar`, `table`, `pagination`, `breadcrumb`, `badge`, `card`, `separator`, `skeleton`, `sonner`.

Only after those files exist, change classes and variants so they match the design (color, radius, height). Keep `cn`, `cva`, `data-slot`, the Base UI parts, and the export names the CLI generated.

`tailwind-variants` is for a composition that has no registry component. It does not replace a file under `components/ui`.

## Stack

- React 19 (no `forwardRef`; `ref` is a regular prop)
- TypeScript strict
- Tailwind CSS v4 with `@theme` and CSS variables
- shadcn/ui on Base UI (`@base-ui/react`, style `base-nova`). Interactive primitives come from `npx shadcn@latest add` and stay in that shape (`cn`, `cva`, `data-slot`)
- `class-variance-authority` and `clsx`, as the registry components require
- `tailwind-merge`, used by `cn` and by compositions outside `components/ui`
- `tailwind-variants` only for compositions that have no shadcn component
- Lucide React for every UI icon (`Menu`, `Search`, `Send`, `X`). Size with `className="size-4"` (or `size-3.5` / `size-5` to match the design). Do not paste inline `<svg>` for those icons.
- If Lucide has no mark (brand logos such as Instagram or X), use `@phosphor-icons/react` (`InstagramLogoIcon`, `XLogoIcon`). Do not hand-draw the path or wrap it in `createLucideIcon`. Flags, photos, and generated illustrations stay as they are.
- Motion (`motion`, imported from `motion/react`) for animation
- `react-hook-form` for forms

```tsx
import { InstagramLogoIcon, XLogoIcon } from '@phosphor-icons/react'
import { Menu, Send } from 'lucide-react'

<Menu className="size-5" />
<Send className="size-4" />
<XLogoIcon className="size-4" />
<InstagramLogoIcon className="size-4" />
```

## Naming

- Files in kebab-case: `user-card.tsx`, `use-modal.ts`
- Named exports only, never `export default`
- No barrel files (`index.ts`) for internal folders

## Component rules

Registry files under `components/ui` keep the `cva` / `cn` the CLI wrote. The `tv()` pattern below is only for a composition that has no shadcn component.

```tsx
// Variants with tv(), exported as `<name>Variants`, only outside components/ui
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
- One component per file. Never declare a component inside another component, and never add a second component to the same file. Compound parts of a single component (`Card`, `CardHeader`, `CardTitle`, `CardContent`) are the exception and stay together.

## Motion

Entrances, reveals, and transitions use Motion. Import from `motion/react`. Do not add CSS `@keyframes` for component motion.

```tsx
import { MotionConfig, motion } from 'motion/react'

<MotionConfig reducedMotion="user">
  <motion.div initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }} />
</MotionConfig>
```

Wrap the animated tree in `MotionConfig` with `reducedMotion="user"`.

## Forms

A screen does not own the form. The screen receives copy and renders one specialized form component (`LoginForm`). That file is the only place with `useForm`, field errors, and submit. The request lives in a hook (`useLogin`) that returns `{ login, pending }` and a failure code, so the component never parses a response, a status, or a token. The screen's TypeScript stays the layout.

Forms use `react-hook-form`, typed with the DTO the API already exports. The submit handler receives those values. Do not read `FormData` and force a type with `String()`, `Number()`, or `as`. Set `noValidate` so the browser does not replace the copy.

```tsx
const { login, pending } = useLogin()
const {
  register,
  handleSubmit,
  setError,
  clearErrors,
  formState: { errors },
} = useForm<SignInDto>({ defaultValues: { email: '', password: '' } })

async function onSubmit(values: SignInDto) {
  clearErrors('root')
  const failure = await login(values)
  if (!failure) {
    router.push('/')
    return
  }
  setError('root', { message: failure === 'invalid' ? copy.failed : copy.unavailable })
}

<motion.form noValidate onSubmit={handleSubmit(onSubmit)} variants={riseGroup}>
```

Copy is a prop, typed from the message module: `type LoginCopy = (typeof auth)['Login']`. `{...register(...)}` is last on the control.

The form is a `motion.form` with the group's variants. Each field is its own `motion.div` with the child variant, so the fields rise in order. The screen wraps that tree in `MotionConfig reducedMotion="user"`. A message that mounts later animates on its own (`initial` / `animate`), because it is not in the stagger.

A field's validation message always sits directly under that field, inside the same group. Email never renders the password message, and one shared `message` is never parked after the last control.

```tsx
const emailErrorId = useId()

<motion.div variants={rise} className="flex flex-col gap-2">
  <Label htmlFor="email">{copy.email}</Label>
  <Input
    id="email"
    aria-invalid={errors.email ? true : undefined}
    aria-describedby={errors.email ? emailErrorId : undefined}
    {...register('email', { required: copy.invalid })}
  />
  {errors.email?.message ? (
    <motion.p
      id={emailErrorId}
      role="alert"
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      className="text-caption text-danger-foreground"
    >
      {errors.email.message}
    </motion.p>
  ) : null}
</motion.div>
```

`setError('email', { message })` when the API rejects that field. `errors.root` is the form failure (credentials, network). Render it once, under the submit control. Do not drop it into a field, and do not mark every field `aria-invalid` for it.

Full shape in [reference.md](reference.md#specialized-form).

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

## shadcn components

Interactive UI is the registry component, not a hand-written Base UI wrapper.

- Import it from `@/components/ui/<name>`: `Button`, `Input`, `Select`, `Dialog`, `Tabs`, `DropdownMenu`, and the rest of the preflight set.
- The file keeps `import { cn } from '@/lib/utils'` and `cva` when the CLI emitted them. Do not rewrite it to `tv()` / `twMerge()`, and do not delete `@/lib/utils`.
- Adjust only what the design changes: token classes (`bg-primary`, `rounded-full`, heights) and extra variants the screen needs. Translate visible labels and `aria-label` (for example `aria-label="Fechar"`).
- Compose with the `render` prop (`<DropdownMenuTrigger render={<Button variant="ghost" />}>Open</DropdownMenuTrigger>`). Do not use `asChild`.
- Style states with the attributes Base UI exposes: `data-open`, `data-closed`, `data-active`, `data-pressed`, `data-checked`, `data-disabled`, `data-highlighted`, `data-[side=top]`, `aria-expanded`.
- Positioned content uses Base UI CSS variables: `max-h-(--available-height)`, `w-(--anchor-width)`, `origin-(--transform-origin)`.

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

Add `'use client'` at the top of components that use state, effects, event handlers or Base UI interactive parts.

## Checklist

- [ ] Preflight passed: Tailwind v4 imported, `components.json` and `cn` present, registry files installed with `shadcn add` before the screen was written
- [ ] Interactive primitives imported from `@/components/ui`, still using `cn` / `cva` / `data-slot` as the CLI generated them
- [ ] File in kebab-case
- [ ] Named export
- [ ] `ComponentProps<'element'>` + `VariantProps` on compositions outside `components/ui`
- [ ] `tv()` / `twMerge()` only where no shadcn component exists
- [ ] `data-slot` on every part
- [ ] States via Base UI data attributes (`data-open`, `data-active`, `data-pressed`)
- [ ] Theme colors only (nothing hardcoded)
- [ ] `focus-visible` on interactive elements
- [ ] `aria-label` on icon-only buttons
- [ ] UI icons come from `lucide-react`; brand marks Lucide dropped come from `@phosphor-icons/react`; no inline `<svg>` for those icons
- [ ] `{...props}` last
- [ ] Every grid declares `grid-cols-1` for mobile; custom tracks use `minmax(0,1fr)`
- [ ] Wide rows (tabs, chips) scroll inside themselves; their parents have `min-w-0`
- [ ] Fixed-size widgets are fluid below `sm`
- [ ] No horizontal page scroll from 320px up (`scrollWidth === clientWidth`); Dialog centered at 320px
- [ ] Touch targets ≥ 24px; no essential feedback that depends only on `hover:`
- [ ] DOM order matches the mobile reading order
- [ ] No component is declared inside another component or shares its file (compound parts excepted)
- [ ] Animation comes from Motion, inside `MotionConfig reducedMotion="user"`
- [ ] The screen renders a specialized form; `useForm` and field errors stay in that file, and the request stays in a hook
- [ ] Forms use react-hook-form typed with the API DTO; submitted values are used as typed, with no `String()`, `Number()`, or `as` cast
- [ ] Each field error renders under that field, with `aria-invalid` and `aria-describedby` pointing at that message; `errors.root` sits under the submit control

## Additional resources

- Full examples (Button, Card, Base UI, `@theme`): [reference.md](reference.md)
