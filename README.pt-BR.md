# SDD Workflow — Manual do Desenvolvedor

Um workflow estruturado para desenvolvimento assistido por IA utilizando **Spec Driven Development (SDD)** e a metodologia **Research → Plan → Implement (RPI)**.

Projetado para IDEs agênticas com acesso ao filesystem (Claude Code, Antigravity, OpenCode, Codex CLI, entre outras), onde o agente atua como pair programmer: controlado, rastreável e orientado por spec.

---

## Sumário

1. [Conceitos Fundamentais](#1-conceitos-fundamentais)
2. [Pré-requisitos](#2-pré-requisitos)
3. [Estrutura de Diretórios](#3-estrutura-de-diretórios)
4. [Configuração Inicial](#4-configuração-inicial)
5. [O Ciclo RPI](#5-o-ciclo-rpi)
6. [Sistema de Skills](#6-sistema-de-skills)
7. [Prompts Utilitários](#7-prompts-utilitários)
8. [Regras de Comportamento](#8-regras-de-comportamento)
9. [Boas Práticas](#9-boas-práticas)

---

## 1. Conceitos Fundamentais

### Spec Driven Development (SDD)

Toda feature começa com uma spec. Nenhum código é escrito sem um `SPEC.md` e um `TASK.md`. Isso evita scope creep, preserva decisões arquiteturais e torna o desenvolvimento assistido por IA auditável e reproduzível.

### Research → Plan → Implement (RPI)

Cada feature passa por três fases isoladas. Cada fase roda em sua própria sessão do agente e produz um único artefato que se torna o contexto exclusivo da próxima fase.

```
RESEARCH → PLAN → IMPLEMENT
```

| Fase | Entrada | Saída |
|---|---|---|
| Research | Requisitos (PRD, descrição ou código existente) | `RESEARCH.md` |
| Plan | `RESEARCH.md` | `SPEC.md` + `TASK.md` + `PROGRESS.md` + `TEST.md` |
| Implement | `SPEC.md` + `TASK.md` + `PROGRESS.md` | Código + `TASK.md` atualizado + `PROGRESS.md` atualizado |

### Por que sessões isoladas?

Cada sessão começa do zero. Passar apenas o artefato da fase anterior — em vez de todo o histórico da conversa — mantém o contexto enxuto, reduz gasto de tokens e evita context rot ao longo de ciclos de desenvolvimento longos.

---

## 2. Pré-requisitos

- Uma IDE agêntica com **acesso de leitura e escrita ao filesystem** (Claude Code, Antigravity, OpenCode, Codex CLI, Cursor, Windsurf ou equivalente)
- O agente deve ser capaz de ler e escrever arquivos dentro do diretório do projeto
- Nenhuma linguagem ou framework específico é necessário — o workflow é agnóstico de stack

---

## 3. Estrutura de Diretórios

```
agents/
  AGENTS.md                   ← Papel do agente e definição do ciclo SDD
  RULES.md                    ← Regras de processo obrigatórias (git, tarefas, decisões)
  PROJECT.md                  ← Stack, arquitetura e regras específicas do projeto
  DECISIONS.md                ← Log de decisões arquiteturais (somente append)
  SETUP.md                    ← Guia de configuração inicial (deletar após o setup)
  prompts/
    rpi-research.md           ← Fase 1: prompt de Research
    rpi-plan.md               ← Fase 2: prompt de Plan
    rpi-implement.md          ← Fase 3: prompt de Implement
    task-create.md            ← Utilitário: criar uma tarefa manualmente
    skill-call.md             ← Utilitário: invocar uma skill específica
    conventional-commit.md    ← Utilitário: gerar mensagem de commit convencional
    pr-template.md            ← Utilitário: gerar descrição de pull request
  skills/
    [um arquivo .md por capacidade]
  specs/
    001-nome-da-feature/
      RESEARCH.md
      SPEC.md
      TASK.md
      PROGRESS.md
      DECISIONS.md
      TEST.md
```

Um diretório `docs/` opcional na raiz do projeto (fora de `agents/`) pode ser usado para
centralizar documentos de referência que servem como insumo para o workflow:

```
docs/                         ← opcional, na raiz do projeto
  prd.md                      ← requisitos do produto
  roadmap.md                  ← priorização de features
  architecture.md             ← decisões arquiteturais de alto nível
  business-rules.md           ← regras de negócio do domínio
  api-contracts/              ← referências de APIs externas
  rfcs/                       ← propostas arquiteturais
  research/                   ← spikes técnicos e avaliações
```

Esses documentos não fazem parte do workflow em si — são consultados como insumo
durante as fases de Research e Plan quando relevantes.

---

### Arquivos lidos em toda sessão

`AGENTS.md`, `PROJECT.md`, `RULES.md`

> Mantenha esses arquivos densos e escaneáveis. Cada token conta.

### Arquivos lidos por tarefa (sob demanda)

`SPEC.md`, `TASK.md`, `PROGRESS.md` e as skills relevantes de `agents/skills/`

---

## 4. Configuração Inicial

### Passo 1 — Copie o template

Copie **apenas o diretório `agents/`** para a raiz do seu projeto. Nenhum outro arquivo
deste repositório é necessário — o `README.md` e demais arquivos da raiz ficam para trás.

### Passo 2 — Execute o setup

Abra uma nova sessão do agente e instrua-o a seguir o arquivo de setup:

> *"Leia e siga as instruções em `agents/SETUP.md`"*

O agente lerá o arquivo diretamente do filesystem e configurará o workflow automaticamente com base no que você fornecer. Não é necessário copiar ou colar seu conteúdo.

O arquivo de setup instrui o agente a produzir três saídas: `PROJECT.md`, `DECISIONS.md` e os arquivos de skill em `agents/skills/`, usando como fonte (em ordem de prioridade) o PRD fornecido, os arquivos do projeto existente, ou fazendo perguntas quando faltar informação crítica.

---

### O que fornecer junto à instrução de setup

| Cenário | O que enviar |
|---|---|
| Projeto novo + PRD | Referencie o caminho do arquivo, descreva os requisitos inline ou anexe o documento |
| Projeto existente | Apenas envie a instrução — o agente escaneia o projeto |
| Projeto existente + PRD | Forneça o PRD junto — o agente combina as duas fontes |

> O agente aceita requisitos em qualquer forma: um caminho de arquivo (`docs/prd.md`),
> uma descrição inline no próprio prompt ou um documento anexado. Use o que for mais conveniente.

### Passo 3 — Revise e delete o SETUP.md

Após o agente gerar os arquivos, revise e confirme:

- [ ] `PROJECT.md` não tem placeholders `[FILL: ...]` restantes
- [ ] `DECISIONS.md` reflete decisões existentes (projetos existentes) ou está vazio (projetos novos)
- [ ] Ao menos um arquivo de skill existe em `agents/skills/`
- [ ] `AGENTS.md` e `RULES.md` estão sem modificações

Então delete `agents/SETUP.md`.

---

## 5. O Ciclo RPI

Cada feature segue o mesmo ciclo de três fases, sempre em sessões isoladas.

### Fase 1 — Research

**Objetivo:** entender o escopo. Produzir `RESEARCH.md`.

1. Crie a pasta da spec: `agents/specs/NNN-nome-da-feature/`
2. Abra uma nova sessão do agente
3. Instrua o agente: *"Leia e siga `agents/prompts/rpi-research.md`"*
4. Anexe ou descreva seus requisitos
5. Revise o `RESEARCH.md` gerado e aprove antes de prosseguir

> Se houver perguntas em aberto no `RESEARCH.md`, responda-as antes de avançar para o Plan.

### Fase 2 — Plan

**Objetivo:** definir o que será construído e como. Produzir `SPEC.md`, `TASK.md`, `PROGRESS.md` e `TEST.md`.

1. Abra uma nova sessão do agente
2. Instrua o agente: *"Leia e siga `agents/prompts/rpi-plan.md`"*
3. O agente lê o `RESEARCH.md` e produz os artefatos do plano
4. Revise todos os arquivos antes de prosseguir

> `TASK.md` é o backlog de implementação. Cada `[ ]` é uma tarefa atômica.

### Fase 3 — Implement

**Objetivo:** implementar uma tarefa por vez. Produzir código e atualizar `TASK.md` e `PROGRESS.md`.

1. Abra uma nova sessão do agente
2. Instrua o agente: *"Leia e siga `agents/prompts/rpi-implement.md`"*
3. O agente identifica o primeiro `[ ]` não marcado no `TASK.md` e implementa
4. Revise o output e confirme antes de o agente avançar para a próxima tarefa

> O agente para após cada tarefa e aguarda sua confirmação. Isso é intencional.

### Pasta da spec após um ciclo completo

```
agents/specs/001-nome-da-feature/
  RESEARCH.md     ← entendimento comprimido da feature
  SPEC.md         ← requisitos funcionais e regras de negócio
  TASK.md         ← checklist de tarefas (todos [x] quando concluído)
  PROGRESS.md     ← histórico de execução e observações
  DECISIONS.md    ← decisões arquiteturais locais da feature
  TEST.md         ← casos de teste de aceitação (dado/quando/então)
```

---

## 6. Sistema de Skills

Skills são arquivos de instrução focados que o agente carrega **apenas quando uma tarefa exige aquela capacidade específica**. Elas codificam os padrões, anti-padrões e convenções do seu projeto para que o agente produza código consistente sem precisar re-explicar o mesmo contexto a cada sessão.

### Como o agente decide quais skills carregar

Antes de implementar uma tarefa, o agente escaneia `agents/skills/` e lê a seção **"When to use this skill"** de cada arquivo. Se a seção corresponder à tarefa atual, a skill é carregada no contexto.

> A qualidade da detecção de skills depende diretamente de quão específica e precisa está a seção "When to use".

### Forçando uma skill para uma tarefa específica

Use `agents/prompts/skill-call.md` quando quiser garantir que uma skill seja aplicada, independentemente da detecção automática.

### Estrutura de um arquivo de skill

```markdown
# Skill: [Nome da Capacidade]

## When to use this skill
[Tipos de tarefa que devem carregar esta skill]

## Patterns
[Abordagem correta com exemplos de código]

## Anti-patterns
[O que NÃO fazer, com motivo]

## Checklist
- [ ] Itens a verificar antes de finalizar a tarefa
```

> Os arquivos de skill são escritos em inglês para economizar tokens — eles são lidos pelo agente a cada sessão relevante.

### Skills comuns a criar

| Arquivo | Criar quando... |
|---|---|
| `ui-components.md` | O projeto tem uma camada de UI |
| `data-access.md` | O projeto usa banco de dados ou camada de persistência |
| `auth.md` | O projeto tem autenticação ou autorização |
| `validation.md` | O projeto valida entrada de usuário ou dados externos |
| `api-integration.md` | O projeto consome APIs externas |
| `error-handling.md` | O projeto tem formato de erro ou estratégia de logging definidos |

---

## 7. Prompts Utilitários

Esses prompts são usados manualmente pelo desenvolvedor — não fazem parte do ciclo RPI.

| Prompt | Quando usar |
|---|---|
| `task-create.md` | Adicionar uma nova tarefa a um `TASK.md` existente sem rodar o Plan completo |
| `skill-call.md` | Forçar uma skill específica para uma tarefa |
| `conventional-commit.md` | Gerar mensagem de commit convencional após a implementação |
| `pr-template.md` | Gerar descrição de pull request a partir da spec e das tarefas concluídas |

### Como usar os prompts utilitários

Abra uma nova sessão do agente e instrua-o a ler o arquivo de prompt diretamente:
> *"Leia e siga `agents/prompts/conventional-commit.md`"*

Substitua o nome do arquivo pelo prompt desejado. Substitua os `<PLACEHOLDERS>` mencionados no arquivo pelos valores relevantes.

---

## 8. Regras de Comportamento

Estas regras são definidas em `AGENTS.md` e `RULES.md` e se aplicam a toda sessão:

| Regra | Descrição |
|---|---|
| R01 | Nenhum código sem `SPEC.md` e `TASK.md` |
| R02 | Uma tarefa por sessão — nunca implementar múltiplas tarefas de uma vez |
| R03 | Atualizar `PROGRESS.md` a cada tarefa concluída |
| R04 | Marcar `[x]` no `TASK.md` quando uma tarefa for concluída |
| R05 | Registrar decisões arquiteturais no `DECISIONS.md` |
| R06 | Nunca executar comandos git (commit, push, branch, tag) |
| R07 | Preferir código simples e legível — sem abstrações desnecessárias |
| R08 | Nenhuma dependência nova sem justificativa no `DECISIONS.md` |
| R09 | Nenhum comentário auto-gerado ou JSDoc sem solicitação explícita |
| R11 | Nenhuma refatoração fora do escopo da tarefa atual |
| R12 | Não antecipar tarefas futuras |
| R13 | Seguir KISS, DRY, Clean Code e SOLID |

---

## 9. Boas Práticas

**Mantenha o PROJECT.md enxuto**
Ele é lido em toda sessão. Cada linha desnecessária custa tokens. Use tabelas, evite parágrafos.

**Uma spec por feature, uma tarefa por sessão**
Resista à tentação de agrupar. Tarefas pequenas e atômicas produzem output mais previsível e revisável.

**Revise antes de confirmar**
O agente para após cada tarefa e aguarda sua aprovação. Use esse checkpoint — revise o diff antes de confirmar a próxima tarefa.

**Skills em vez de instruções repetidas**
Se você se pega explicando o mesmo padrão em múltiplas sessões, ele pertence a um arquivo de skill.

**Decisões são contexto permanente**
Cada entrada no `DECISIONS.md` impede o agente de questionar ou re-decidir algo já definido. Mantenha-o atualizado.

**Não modifique AGENTS.md ou RULES.md**
Regras específicas do stack pertencem ao `PROJECT.md` na seção Stack Rules. Modificar os arquivos centrais quebra o contrato do workflow.