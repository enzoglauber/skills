# skills

Coleção pessoal de Agent Skills do Cursor. Este repositório fica em `~/.cursor/skills`, então todas as skills ficam disponíveis em **qualquer projeto** aberto no Cursor, sem precisar copiar nada para dentro do repositório do projeto.

## Skills disponíveis

| Skill                                  | O que faz                                                                                        | Quando é ativada                                                                            |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| [`design-to-react`](./design-to-react) | Converte um design (screenshot, frame do Figma ou do Pencil `.pen`) em componentes React prontos | Ao anexar um design e pedir para implementar, ou ao criar/refatorar componentes React de UI |

### design-to-react

Atua como um tech lead de front-end: analisa o design, lê o `package.json` e o `@theme` do projeto, mapeia as cores para tokens do tema e gera um componente (ou família de componentes compostos) por arquivo, terminando com um checklist de qualidade.

- **Stack gerada**: React 19 (sem `forwardRef`), TypeScript strict, Tailwind CSS v4 com `@theme`, Base UI (`@base-ui/react`), `tailwind-variants`, `tailwind-merge` e Lucide/Phosphor Icons.
- **Padrões impostos**: arquivos em kebab-case, só named exports, sem barrel files, variantes com `tv()`, `twMerge()` com o `className` do consumidor por último, `data-slot` em cada parte, estados via `data-[state]:`, nenhuma cor hardcoded, `focus-visible` e `aria-label` em botões só com ícone.
- **Next.js**: adiciona `'use client'` em componentes interativos.
- **Origem**: adaptada do artigo [Design para React com IA](https://www.rocketseat.com.br/blog/artigos/post/design-para-react-ia-prompt), de Diego Fernandes (Rocketseat).

Dependências que o projeto precisa ter:

```bash
npm install tailwind-variants tailwind-merge @base-ui/react lucide-react
```

Documentação completa em [`design-to-react/README.md`](./design-to-react/README.md).

## Instalação

```bash
git clone https://github.com/enzoglauber/skills.git ~/.cursor/skills
```

No WSL, o caminho é o `~/.cursor/skills` do Linux (ex.: `\\wsl.localhost\Ubuntu-26.04\home\<usuario>\.cursor\skills` visto pelo Windows). Para atualizar, basta `git pull` dentro da pasta. O Cursor detecta as skills automaticamente; se uma skill nova não aparecer, recarregue a janela.

## Como usar uma skill

- **Automático**: o agente lê o campo `description` do `SKILL.md` e aplica a skill quando o pedido combina com ela (ex.: anexar um print e pedir "implementa esse componente").
- **Manual**: digite `/<nome-da-skill>` no chat (ex.: `/design-to-react`).

## Estrutura

Cada skill é uma pasta com o mesmo nome do campo `name` do frontmatter:

```
skills/
├── README.md                 # este arquivo
└── design-to-react/
    ├── SKILL.md              # instruções para o agente (obrigatório)
    ├── reference.md          # exemplos carregados sob demanda
    └── README.md             # documentação para humanos
```

## Convenções

- **`SKILL.md` em inglês e enxuto**: é carregado no contexto do agente, então menos texto significa menos tokens. Exemplos longos vão para arquivos separados (ex.: `reference.md`) e são referenciados por link, para o agente só lê-los quando precisar.
- **`README.md` em português**: documentação para humanos, com origem, motivação e FAQ.
- **Frontmatter obrigatório** no `SKILL.md`:

  ```yaml
  ---
  name: nome-da-skill # kebab-case, igual ao nome da pasta
  description: O que faz. Use when ... # o "quando usar" decide a ativação automática
  ---
  ```

## Adicionando uma nova skill

1. Crie a pasta `nome-da-skill/` com um `SKILL.md` seguindo o frontmatter acima (ou peça ao agente usando a skill nativa `create-skill`).
2. Escreva uma `description` específica, com gatilhos claros ("Use when..."), para a ativação automática funcionar bem.
3. Adicione um `README.md` na pasta e uma linha na tabela [Skills disponíveis](#skills-disponíveis) deste arquivo.
4. Teste num projeto real, pedindo algo que deveria disparar a skill, e ajuste a `description` se ela não for acionada.
