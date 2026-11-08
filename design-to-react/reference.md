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

## Radix anatomy

The unified `radix-ui` package exports each primitive as a namespace (`Dialog`, `Tabs`...). Alias it as `XPrimitive` so the wrapper can keep the public name.

```tsx
// Dialog
import { Dialog as DialogPrimitive } from 'radix-ui'
<DialogPrimitive.Root>
  <DialogPrimitive.Trigger asChild><Button /></DialogPrimitive.Trigger>
  <DialogPrimitive.Portal>
    <DialogPrimitive.Overlay />
    <DialogPrimitive.Content>
      <DialogPrimitive.Title />
      <DialogPrimitive.Description />
      <DialogPrimitive.Close />
    </DialogPrimitive.Content>
  </DialogPrimitive.Portal>
</DialogPrimitive.Root>

// Tabs
import { Tabs as TabsPrimitive } from 'radix-ui'
<TabsPrimitive.Root defaultValue="a">
  <TabsPrimitive.List><TabsPrimitive.Trigger value="a" /></TabsPrimitive.List>
  <TabsPrimitive.Content value="a" />
</TabsPrimitive.Root>

// Select
import { Select as SelectPrimitive } from 'radix-ui'
<SelectPrimitive.Root>
  <SelectPrimitive.Trigger><SelectPrimitive.Value placeholder="Selecione" /></SelectPrimitive.Trigger>
  <SelectPrimitive.Portal>
    <SelectPrimitive.Content position="popper">
      <SelectPrimitive.Viewport>
        <SelectPrimitive.Item value="a"><SelectPrimitive.ItemText /></SelectPrimitive.Item>
      </SelectPrimitive.Viewport>
    </SelectPrimitive.Content>
  </SelectPrimitive.Portal>
</SelectPrimitive.Root>

// DropdownMenu
import { DropdownMenu as DropdownMenuPrimitive } from 'radix-ui'
<DropdownMenuPrimitive.Root>
  <DropdownMenuPrimitive.Trigger />
  <DropdownMenuPrimitive.Portal>
    <DropdownMenuPrimitive.Content sideOffset={4}>
      <DropdownMenuPrimitive.Item />
    </DropdownMenuPrimitive.Content>
  </DropdownMenuPrimitive.Portal>
</DropdownMenuPrimitive.Root>
```

Wrapping and styling a Radix part:

```tsx
'use client'

import { Tabs as TabsPrimitive } from 'radix-ui'
import type { ComponentProps } from 'react'
import { twMerge } from 'tailwind-merge'

export function TabsTrigger({ className, ...props }: ComponentProps<typeof TabsPrimitive.Trigger>) {
  return (
    <TabsPrimitive.Trigger
      data-slot="tabs-trigger"
      className={twMerge(
        'min-h-10 shrink-0 touch-manipulation whitespace-nowrap text-muted-foreground',
        'data-[state=active]:text-foreground data-disabled:opacity-50',
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
