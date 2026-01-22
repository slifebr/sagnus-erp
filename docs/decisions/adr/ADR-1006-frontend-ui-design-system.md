# ADR-1006 — UI / Design System do Frontend

- **Status:** Accepted
- **Data:** 2026-01-22
- **Escopo:** Frontend (Sagnus Shell + telas ERP)

## Contexto

O Sagnus ERP terá muitas telas de cadastro/consulta, com componentes repetidos:
- Formulários densos (campos, labels, validação, mensagens)
- Layout padrão (shell, menu, tabs)
- Diálogos (confirmar, pesquisar, selecionar)
- Tabelas e grids (o grid “pesado” é tratado no ADR-1007)
- Padrões de acessibilidade e navegação por teclado

Dependência excessiva de uma biblioteca “mega UI” pode acelerar no começo, mas costuma:
- limitar personalização do ERP
- criar lock-in forte
- dificultar upgrades
- aumentar risco de “componente Frankenstein” (estilo Delphi/Forms)

Precisamos de um design system controlado, consistente e evolutivo.

## Decisão

### 1) Fonte da verdade
- Criar um pacote interno: `packages/ui`
- Esse pacote será a **fonte da verdade** para componentes e tokens visuais.

### 2) Base técnica
- **Tailwind CSS** para utilitários e consistência rápida
- **shadcn/ui** como base de componentes (copiados para o projeto e customizáveis)
- Ícones: `lucide-react` (padrão)
- Tipografia/spacing definidos por tokens internos (documentados no `packages/ui`)

> Observação: shadcn/ui não é um “framework fechado”; são componentes que você controla no repositório.

### 3) Regras de uso
- Features/telas **não devem** importar diretamente componentes “crus” de terceiros quando já existir equivalente em `packages/ui`.
- Qualquer novo componente de uso geral entra em `packages/ui` com:
  - variações suportadas
  - estados (disabled, loading, error)
  - padrões de acessibilidade (aria, foco)
- Componentes específicos de uma tela/feature ficam dentro da feature.

### 4) Estilo e consistência do ERP
- Layout com densidade configurável (compact/normal), pensando em UX de ERP.
- Padrões obrigatórios:
  - foco visível (keyboard-first)
  - validação com mensagem por campo e painel global (estilo Forms)
  - botões e diálogos com ações consistentes (Salvar/Cancelar/Confirmar)

### 5) Integração com Grid
- Componentes de formulário/inputs do design system devem ter “modo grid-editor”
  (ex.: input compacto, sem margem, com validação inline), para integração com ADR-1007.

## Alternativas consideradas

### A) PrimeReact (ou similar “mega UI”)
- **Prós:** velocidade inicial (muitos componentes prontos).
- **Contras:** lock-in, tema/customizações complexas, risco de dependência excessiva e upgrades dolorosos.
- **Decisão:** não adotado como base do sistema.

### B) Material UI (MUI)
- **Prós:** maturidade e consistência.
- **Contras:** estilo/material pode conflitar com densidade/estética de ERP clássico; customização pode ficar pesada; possível lock-in.
- **Decisão:** não adotado como base, pode ser reavaliado caso haja necessidade específica com ADR novo.

### C) CSS “na unha” sem design system
- **Prós:** controle total.
- **Contras:** alto custo, inconsistência e retrabalho.
- **Decisão:** não adotado.

## Consequências

### Benefícios
- Consistência visual e de comportamento em todo o ERP
- Menos dependência de vendor e maior controle de evolução
- Base sólida para UX “Forms-like” (teclado, foco, validação)

### Custos/Riscos
- Exige disciplina: tudo reutilizável deve ir para `packages/ui`
- Custo inicial para montar tokens e componentes base
- Necessidade de documentação mínima (Storybook opcional no futuro)

## Plano de adoção

1. Criar `packages/ui` com:
   - Button, Input, Select, Checkbox, Dialog, Tabs, Toast
   - FormField (label + hint + error)
2. Definir tokens (spacing, radius, typography) e modo compact.
3. Criar exemplos de uso no Shell (login + layout).
4. Documentar padrões no README do `packages/ui`.
