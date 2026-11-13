# Reference examples

Files under `components/ui` come from `npx shadcn@latest add` and keep `cn` and `cva`. The Button example below is the shape of a composition that has no registry component. Do not use it to replace `components/ui/button.tsx`.

## Button (variants)

```tsx
import { tv, type VariantProps } from 'tailwind-variants'
import { twMerge } from 'tailwind-merge'
import type { ComponentProps } from 'react'

export const buttonVariants = tv({
  base: [
    'inline-flex cursor-pointer items-center justify-center font-medium rounded-lg border transition-colors',
    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring',
    'data-[disabled]:pointer-events-none data-[disabled]:opacity-50',
  ],
  variants: {
    variant: {
      primary: 'border-primary bg-primary text-primary-foreground hover:bg-primary-hover',
      secondary: 'border-border bg-secondary text-secondary-foreground hover:bg-muted',
      ghost: 'border-transparent bg-transparent text-muted-foreground hover:text-foreground',
      destructive: 'border-destructive bg-destructive text-destructive-foreground hover:bg-destructive/90',
    },
    size: {
      sm: 'h-6 px-2 gap-1.5 text-xs [&_svg]:size-3',
      md: 'h-7 px-3 gap-2 text-sm [&_svg]:size-3.5',
      lg: 'h-9 px-4 gap-2.5 text-base [&_svg]:size-4',
    },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
})

export interface ButtonProps
  extends ComponentProps<'button'>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, disabled, children, ...props }: ButtonProps) {
  return (
    <button
      type="button"
      data-slot="button"
      data-disabled={disabled ? '' : undefined}
      className={twMerge(buttonVariants({ variant, size }), className)}
      disabled={disabled}
      {...props}
    >
      {children}
    </button>
  )
}
```

## Card (compound components)

```tsx
import { twMerge } from 'tailwind-merge'
import type { ComponentProps } from 'react'

export interface CardProps extends ComponentProps<'div'> {}

export function Card({ className, ...props }: CardProps) {
  return (
    <div
      data-slot="card"
      className={twMerge('bg-surface flex flex-col gap-6 rounded-xl border border-border p-6 shadow-sm', className)}
      {...props}
    />
  )
}

export function CardHeader({ className, ...props }: ComponentProps<'div'>) {
  return <div data-slot="card-header" className={twMerge('flex flex-col gap-1.5', className)} {...props} />
}

export function CardTitle({ className, ...props }: ComponentProps<'h3'>) {
  return <h3 data-slot="card-title" className={twMerge('text-lg font-semibold', className)} {...props} />
}

export function CardContent({ className, ...props }: ComponentProps<'div'>) {
  return <div data-slot="card-content" className={className} {...props} />
}
```

## Base UI anatomy

`@base-ui/react` exports each primitive from its own path. Alias it as `XPrimitive` so the wrapper can keep the public name. Compose another element with `render`, not `asChild`.

```tsx
// Dialog
import { Dialog as DialogPrimitive } from '@base-ui/react/dialog'
<DialogPrimitive.Root>
  <DialogPrimitive.Trigger render={<Button />}>Open</DialogPrimitive.Trigger>
  <DialogPrimitive.Portal>
    <DialogPrimitive.Backdrop />
    <DialogPrimitive.Popup>
      <DialogPrimitive.Title />
      <DialogPrimitive.Description />
      <DialogPrimitive.Close />
    </DialogPrimitive.Popup>
  </DialogPrimitive.Portal>
</DialogPrimitive.Root>

// Tabs
import { Tabs as TabsPrimitive } from '@base-ui/react/tabs'
<TabsPrimitive.Root defaultValue="a">
  <TabsPrimitive.List><TabsPrimitive.Tab value="a" /></TabsPrimitive.List>
  <TabsPrimitive.Panel value="a" />
</TabsPrimitive.Root>

// Menu
import { Menu as MenuPrimitive } from '@base-ui/react/menu'
<MenuPrimitive.Root>
  <MenuPrimitive.Trigger render={<Button variant="ghost" />}>Open</MenuPrimitive.Trigger>
  <MenuPrimitive.Portal>
    <MenuPrimitive.Positioner>
      <MenuPrimitive.Popup>
        <MenuPrimitive.Item />
      </MenuPrimitive.Popup>
    </MenuPrimitive.Positioner>
  </MenuPrimitive.Portal>
</MenuPrimitive.Root>
```

Wrapping and styling a Base UI part:

```tsx
'use client'

import { Tabs as TabsPrimitive } from '@base-ui/react/tabs'
import type { ComponentProps } from 'react'
import { twMerge } from 'tailwind-merge'

export function TabsTrigger({ className, ...props }: ComponentProps<typeof TabsPrimitive.Tab>) {
  return (
    <TabsPrimitive.Tab
      data-slot="tabs-trigger"
      className={twMerge(
        'min-h-10 shrink-0 touch-manipulation whitespace-nowrap text-muted-foreground',
        'data-active:text-foreground data-disabled:opacity-50',
        'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring',
        className,
      )}
      {...props}
    />
  )
}
```

## Tailwind v4 `@theme` with the tokens

```css
@import 'tailwindcss';

@theme {
  --color-surface: oklch(0.99 0 0);
  --color-surface-raised: oklch(1 0 0);

  --color-foreground: oklch(0.21 0.01 260);
  --color-foreground-subtle: oklch(0.45 0.01 260);
  --color-muted-foreground: oklch(0.6 0.01 260);

  --color-primary: oklch(0.55 0.2 265);
  --color-primary-hover: oklch(0.5 0.2 265);
  --color-primary-foreground: oklch(0.99 0 0);

  --color-secondary: oklch(0.96 0.005 260);
  --color-secondary-foreground: oklch(0.21 0.01 260);
  --color-muted: oklch(0.94 0.005 260);

  --color-destructive: oklch(0.58 0.22 27);
  --color-destructive-foreground: oklch(0.99 0 0);

  --color-border: oklch(0.91 0.005 260);
  --color-input: oklch(0.91 0.005 260);
  --color-ring: oklch(0.55 0.2 265);
}
```

Every `--color-<name>` becomes `bg-<name>`, `text-<name>`, `border-<name>`, `ring-<name>`. For dark mode, override the variables under a `.dark` selector or `@media (prefers-color-scheme: dark)`.

## Specialized form

The screen only places the form. `useForm`, the field messages, and submit stay in the form file. The hook owns the request and returns a failure code.

```tsx
export function LoginScreen({ copy }: { copy: LoginCopy }) {
  return (
    <MotionConfig reducedMotion="user">
      <LoginForm copy={copy} />
    </MotionConfig>
  )
}
```

```tsx
export function useLogin() {
  const [pending, setPending] = useState(false)

  async function login(values: SignInDto): Promise<'invalid' | 'unavailable' | null> {
    setPending(true)
    try {
      const result = await authControllerLogin({ body: values })
      if (!result.data) return result.response?.status === 401 ? 'invalid' : 'unavailable'
      setAccessToken(result.data.access_token)
      return null
    } catch {
      return 'unavailable'
    } finally {
      setPending(false)
    }
  }

  return { login, pending }
}
```

```tsx
export function LoginForm({ copy }: { copy: LoginCopy }) {
  const emailErrorId = useId()
  const passwordErrorId = useId()
  const formErrorId = useId()
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
    if (!failure) return
    setError('root', { message: failure === 'invalid' ? copy.failed : copy.unavailable })
  }

  return (
    <motion.form noValidate onSubmit={handleSubmit(onSubmit)} variants={riseGroup} className="flex flex-col gap-4">
      <motion.div variants={rise} className="flex flex-col gap-2">
        <Label htmlFor="email">{copy.email}</Label>
        <Input
          id="email"
          aria-invalid={errors.email ? true : undefined}
          aria-describedby={errors.email ? emailErrorId : undefined}
          {...register('email', { required: copy.invalid })}
        />
        {errors.email?.message ? (
          <motion.p id={emailErrorId} role="alert" initial={{ opacity: 0, y: 8 }} animate={{ opacity: 1, y: 0 }}>
            {errors.email.message}
          </motion.p>
        ) : null}
      </motion.div>

      <motion.div variants={rise} className="flex flex-col gap-2">
        <Label htmlFor="password">{copy.password}</Label>
        <PasswordInput
          id="password"
          aria-invalid={errors.password ? true : undefined}
          aria-describedby={errors.password ? passwordErrorId : undefined}
          {...register('password', { required: copy.invalid })}
        />
        {errors.password?.message ? (
          <motion.p id={passwordErrorId} role="alert" initial={{ opacity: 0, y: 8 }} animate={{ opacity: 1, y: 0 }}>
            {errors.password.message}
          </motion.p>
        ) : null}
      </motion.div>

      <Button type="submit" loading={pending}>{copy.submit}</Button>
      {errors.root?.message ? (
        <motion.p id={formErrorId} role="alert" initial={{ opacity: 0, y: 8 }} animate={{ opacity: 1, y: 0 }}>
          {errors.root.message}
        </motion.p>
      ) : null}
    </motion.form>
  )
}
```

The password message is inside the password group, under the control. The email message stays in the email group. `errors.root` is the only message that follows the submit control.

## Icons (Lucide)

```tsx
import { InstagramLogoIcon, XLogoIcon } from '@phosphor-icons/react'
import { Menu, Send } from 'lucide-react'

<Menu className="size-5" />
<Send className="size-4" />
<XLogoIcon className="size-4" />
<InstagramLogoIcon className="size-4" />
```
