# design-to-react

Skill pessoal do Cursor (disponível em todos os projetos) que converte um design (screenshot, frame do Figma ou frame do Pencil `.pen`) em componentes React prontos para produção.

## Origem

Baseado no artigo [Design para React com IA: o prompt exato para acelerar seu front-end](https://www.rocketseat.com.br/blog/artigos/post/design-para-react-ia-prompt), de Diego Fernandes (Rocketseat). O prompt do artigo foi transformado em regras de skill, com um passo extra para respeitar a stack e os tokens já existentes no projeto.

## Arquivos

| Arquivo        | Para quem           | Conteúdo                                                                |
| -------------- | ------------------- | ----------------------------------------------------------------------- |
| `SKILL.md`     | Agente              | Workflow, regras, tokens e checklist (em inglês para economizar tokens) |
| `reference.md` | Agente, sob demanda | Exemplos completos: Button, Card, Radix e `@theme`                      |
| `README.md`    | Humanos             | Esta documentação                                                       |

## Como usar

- **Automático**: anexe o print do design no chat e peça algo como "implementa esse componente". O agente aplica o skill sozinho.
- **Manual**: digite `/design-to-react` no chat e anexe o design.

## Stack gerada

- React 19 (sem `forwardRef`)
- TypeScript strict
- Tailwind CSS v4 com `@theme` e CSS variables
- shadcn/ui sobre Radix (pacote unificado `radix-ui`) para componentes headless
- `tailwind-variants` para variantes e `tailwind-merge` para merge de classes
- Lucide React ou Phosphor Icons

Dependências:

```bash
npm install tailwind-variants tailwind-merge radix-ui lucide-react
```

Componentes interativos entram pela CLI (`npx shadcn@latest add <componente>`) e são adaptados em seguida: `cn()`/`cva` viram `twMerge`/`tv()`, exports nomeados e tokens do projeto.

## Padrões impostos

- Arquivos em kebab-case, só named exports, sem barrel files em pastas internas
- `tv()` para variantes e `twMerge()` com o `className` do consumidor
- `data-slot` em cada parte e estados via `data-[state]:`
- Só cores do tema (nada hardcoded)
- `focus-visible` em elementos interativos e `aria-label` em botões só com ícone
- `{...props}` sempre por último

## Por que funciona

1. **Stack atualizada**: React 19 e Tailwind v4 com variáveis CSS nativas.
2. **Manutenibilidade**: `tailwind-variants` + `tailwind-merge` evitam briga de especificidade e facilitam customização.
3. **Acessibilidade**: estados via data-attributes, `focus-visible` e `aria-label` obrigatórios.

## FAQ

- **Serve para Next.js?** Sim (App Router). Componentes interativos recebem `'use client'`.
- **O código é acessível?** O skill força as boas práticas, mas valide o resultado com o Axe DevTools.
- **E se meu projeto usar outros nomes de tokens?** O agente lê o `@theme` do projeto e usa os nomes existentes; se faltar algum, ele sugere adicionar.

## Diferenças em relação ao artigo

O artigo usa Base UI; esta skill usa shadcn/ui sobre Radix. Os imports no `reference.md` vêm do pacote unificado (`import { Dialog as DialogPrimitive } from 'radix-ui'`), e cada parte é embrulhada numa função própria com `data-slot`, estilizada pelos atributos do Radix (`data-[state=open]`, `data-disabled`...).

A seção **Responsive & touch** do `SKILL.md` também não existe no artigo: ela veio de um bug real em que um grid sem `grid-cols-1` alargava a página no celular, gerava scroll horizontal e tirava o Dialog do centro.

<!-- \\wsl.localhost\Ubuntu-26.04\home\enzoglauber\.cursor\skills\design-to-react -->
