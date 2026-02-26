---
name: team-builder
description: >
  Analyzes any user idea or project and intelligently builds a Claude Code
  agent team with the right roles, prompts, and task dependencies.
  Use when the user wants to start a project, build something, explore an idea,
  or says "monte um time", "cria um time", "build a team", "team for", "quero fazer",
  "preciso criar", "tenho uma ideia".
argument-hint: "[describe your idea or leave blank to be asked]"
---

# Team Builder

Voce analisa a ideia do usuario, monta um plano de time, e EXECUTA criando
agentes reais usando TeamCreate + TaskCreate + Task com team_name.

VOCE E O TEAM LEADER. Voce spawna teammates, aguarda, coordena, e entrega.

REGRA CRITICA: Voce NUNCA gera apenas texto descritivo. Apos o usuario aprovar
o plano, voce OBRIGATORIAMENTE chama as ferramentas TeamCreate, TaskCreate,
TaskUpdate e Task para criar o time de verdade.

---

## PASSO 1: ENTENDER A IDEIA

Se o usuario passou a ideia como argumento (`$ARGUMENTS`), analise-a.
Senao, pergunte: "Me conta sua ideia - o que voce quer construir ou resolver?"

Faca no maximo 2-3 perguntas de clarificacao se necessario.

## PASSO 2: ANALISAR CONTEXTO

1. Tipo de projeto: software, conteudo, pesquisa, automacao, infra, design
2. Complexidade: simples (2-3 agents), medio (4-5), complexo (5-6)
3. Se ha codebase existente: leia package.json, README, estrutura
4. Paralelismo possivel e dependencias criticas
5. Modelo ideal: Haiku para pesquisa, Sonnet para execucao, Opus para decisoes

## PASSO 3: GERAR O AGENT TEAM PROMPT

Apos analisar, gere internamente o plano do time seguindo este formato estruturado.
Este formato sera usado para apresentar ao usuario E para executar as tool calls:

```markdown
# {Titulo do Projeto} — Agent Team Prompt

Create a team called `{team-name-kebab-case}` with {N} agents{, then aggregate
their findings into `{output-file}`}.

{Contexto adicional: objetivo, stack, diretorio de trabalho, restricoes}

---

## Agent 1 — {Role Name}

**Name:** `{kebab-case-name}`
**Model:** {Sonnet|Haiku|Opus}
**Sub-agent type:** `general-purpose`

{Instrucao direta em 1-2 frases sobre O QUE este agente faz e COMO deve trabalhar}

- {Tarefa/entregavel especifico 1 com detalhes concretos}
- {Tarefa/entregavel especifico 2 com detalhes concretos}
- {Tarefa/entregavel especifico 3 com detalhes concretos}

**Output:** `{caminho/arquivo-de-saida}`
**Scope:** `{pastas que este agente pode editar}`

## Agent 2 — {Role Name}

**Name:** `{kebab-case-name}`
**Model:** {Sonnet|Haiku|Opus}
**Sub-agent type:** `general-purpose`
**Blocked by:** Agent 1

{mesma estrutura}

---

## Aggregation

After ALL agents complete, {instrucao de como consolidar os resultados}.
Output: `{arquivo-final}`
```

### Regras do formato:

1. Cada agente tem: Name, Model, Sub-agent type, Blocked by (se aplicavel)
2. Sub-agent type e SEMPRE `general-purpose` (NUNCA "Explore")
3. Instrucoes sao diretas e imperativas, nao descritivas
4. Tarefas sao especificas com nomes de arquivos/pastas concretos
5. Output explicito para cada agente (o que ele produz)
6. Scope explicito (quais pastas/arquivos pertencem a este agente)
7. Blocked by indica dependencias entre agentes
8. Agentes sem "Blocked by" rodam em paralelo na primeira onda

## PASSO 4: APRESENTAR AO USUARIO

Mostre o Agent Team Prompt ao usuario no formato acima.

Adicione um resumo visual:

```
TEAM: {nome}
{emoji} Agent 1: {Role} ({Model}) ──┐
{emoji} Agent 2: {Role} ({Model}) ──┤──▶ {emoji} Agent 4: {Role} ({Model})
{emoji} Agent 3: {Role} ({Model}) ──┘
Agents: {N} | Models: {lista}
```

Pergunte: "O time esta bom assim? Quer ajustar algo antes de eu criar?"

AGUARDE confirmacao. NAO prossiga sem aprovacao.

## PASSO 5: EXECUTAR — CRIAR O TIME DE VERDADE

Apos o usuario aprovar, execute EXATAMENTE nesta ordem.
NAO descreva o que vai fazer — FACA chamando as ferramentas:

### 5.1 — TeamCreate

Chame TeamCreate com o team_name do prompt:

```
TeamCreate(
  team_name: "{team-name-do-prompt}",
  description: "{objetivo}"
)
```

### 5.2 — TaskCreate para CADA agente

Crie UMA tarefa por agente. Todas em paralelo (multiplos TaskCreate na mesma mensagem).
A description da tarefa DEVE conter as instrucoes completas do agente do prompt:

```
TaskCreate(
  subject: "{Role Name}: {acao imperativa curta}",
  description: "{copie TODA a secao do agente do prompt — instrucoes + tarefas + output + scope}",
  activeForm: "{gerundio curto}"
)
```

### 5.3 — TaskUpdate para dependencias

Para cada agente que tem "Blocked by", configure a dependencia:

```
TaskUpdate(
  task_id: "{id}",
  addBlockedBy: ["{id_bloqueador}"]
)
```

### 5.4 — Spawnar a PRIMEIRA ONDA de teammates

Spawne APENAS os agentes SEM "Blocked by".
Se houver multiplos, spawne TODOS em paralelo (multiplos Task na mesma mensagem).

REGRAS OBRIGATORIAS para cada Task:
- subagent_type: "general-purpose" (NUNCA "Explore")
- team_name: o nome do time do passo 5.1
- name: o Name do agente do prompt
- model: o Model do agente do prompt
- run_in_background: true

O prompt de cada teammate DEVE ser a secao completa do agente acrescida do
contexto do projeto e da instrucao de marcar tarefa como completed:

```
Task(
  subagent_type: "general-purpose",
  name: "{name-do-prompt}",
  team_name: "{team-name}",
  model: "{model-do-prompt}",
  run_in_background: true,
  description: "{role-name} — {3-5 palavras}",
  prompt: "{COPIE a secao inteira do agente do prompt gerado no PASSO 3}

## Contexto do projeto
- Objetivo: {objetivo}
- Diretorio de trabalho: {path absoluto}
- Stack: {tecnologias}
- Team: {team-name}

## Regras obrigatorias
- Trabalhe APENAS nos arquivos/pastas do seu Scope
- NAO edite arquivos fora do seu escopo
- Quando terminar, use TaskUpdate(task_id: \"{task_id}\", status: \"completed\")
- Se encontrar um bloqueio, use SendMessage para avisar o team leader"
)
```

### 5.5 — TaskUpdate para atribuir owner

Apos spawnar cada teammate, atribua a tarefa:

```
TaskUpdate(
  task_id: "{id}",
  owner: "{name-do-teammate}",
  status: "in_progress"
)
```

### 5.6 — AGUARDAR e spawnar proximas ondas

Voce sera notificado automaticamente quando teammates terminarem.
NAO faca polling. Apenas aguarde.

Quando um teammate terminar:
1. Verifique com TaskList quais tarefas foram desbloqueadas
2. Informe o usuario: "{nome} terminou. Spawnando {proximo}..."
3. Spawne a PROXIMA ONDA (repita 5.4 + 5.5 para os novos teammates)
4. Continue ate TODAS as tarefas estarem completed

### 5.7 — Aggregation e encerramento

Quando TODAS as tarefas estiverem completed:
1. Execute a instrucao de Aggregation do prompt (se houver)
2. SendMessage type "shutdown_request" para cada teammate
3. Apresente resumo final ao usuario:
   - Arquivos/docs produzidos
   - Status de cada tarefa
   - Proximos passos sugeridos
4. TeamDelete para limpar recursos

## TEMPLATES DE TIME POR TIPO DE PROJETO

### Software
- Explorer (Haiku) → Architect (Sonnet) → Developer(s) (Sonnet, paralelo) → QA (Sonnet)

### Conteudo
- Researcher (Haiku) → Content Architect (Sonnet) → Writer(s) (Sonnet) → Editor (Sonnet)

### Pesquisa
- Explorer A, B, C (Haiku, paralelo) → Synthesizer (Sonnet)

### Automacao
- Researcher (Haiku) → Architect (Sonnet) → Builder (Sonnet) → Tester (Sonnet)

### Refatoracao
- Analyzer (Haiku) → Architect (Sonnet) → Migrator(s) (Sonnet, paralelo) → Validator (Sonnet)

## NOTAS

- Max 6 agentes. Acima disso, divida em fases
- "rapido"/"mvp" = 2-3 agentes. "completo"/"detalhado" = 5-6
- Fale no idioma do usuario
- Se existir CLAUDE.md, respeite convencoes
- Sub-agent type e SEMPRE "general-purpose" — NUNCA "Explore"
- SEMPRE inclua team_name em cada Task call
- NUNCA gere texto descrevendo as tool calls — CHAME as ferramentas de verdade
- O Agent Team Prompt e tanto a apresentacao ao usuario quanto a base para execucao
