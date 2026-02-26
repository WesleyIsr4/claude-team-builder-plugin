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

## PASSO 3: APRESENTAR O PLANO

Mostre ao usuario:
- Diagrama visual do time com dependencias
- Tabela com: Papel, Modelo, Responsabilidade, Entregaveis
- Numero de agentes e modelos

Pergunte: "O time esta bom assim? Quer ajustar algo antes de eu criar?"

AGUARDE confirmacao. NAO prossiga sem aprovacao.

## PASSO 4: EXECUTAR — CRIAR O TIME DE VERDADE

Apos o usuario aprovar, execute EXATAMENTE nesta ordem:

### 4.1 — TeamCreate

Chame a ferramenta TeamCreate AGORA:

```
TeamCreate(
  team_name: "{nome-kebab-case}",
  description: "{objetivo do time}"
)
```

### 4.2 — TaskCreate para CADA agente

Chame TaskCreate uma vez para CADA agente do time.
A description da tarefa deve ser DETALHADA — e o que o teammate le para saber o que fazer.

Crie todas as tarefas em paralelo (multiplos TaskCreate numa unica mensagem).

```
TaskCreate(
  subject: "{acao imperativa curta}",
  description: "{descricao completa do trabalho — contexto, escopo, arquivos, criterios}",
  activeForm: "{gerundio curto}"
)
```

### 4.3 — TaskUpdate para dependencias

Para CADA tarefa que depende de outra, chame TaskUpdate:

```
TaskUpdate(
  task_id: "{id}",
  addBlockedBy: ["{id_bloqueador}"]
)
```

### 4.4 — Task para spawnar a PRIMEIRA ONDA de teammates

Spawne APENAS os agentes cujas tarefas NAO tem blockedBy.

REGRAS OBRIGATORIAS para cada Task:
- subagent_type DEVE ser "general-purpose" (NUNCA use "Explore")
- team_name DEVE ser o nome do time criado no passo 4.1
- name DEVE ser o nome kebab-case do teammate
- model DEVE ser haiku, sonnet ou opus
- run_in_background DEVE ser true

Se houver multiplos agentes sem dependencias, spawne TODOS em paralelo
(multiplos Task numa unica mensagem).

```
Task(
  subagent_type: "general-purpose",
  name: "{nome-kebab-case}",
  team_name: "{nome-do-time}",
  model: "{haiku|sonnet|opus}",
  run_in_background: true,
  description: "{3-5 palavras}",
  prompt: "Voce e o {PAPEL} do time {nome-do-time}.

## Seu papel
{descricao do papel}

## Contexto do projeto
- Objetivo: {objetivo}
- Diretorio: {path absoluto}
- Stack: {tecnologias}

## Responsabilidades
- {resp_1}
- {resp_2}
- {resp_3}

## Escopo de arquivos (APENAS estes sao seus)
- {pasta_1}
- {pasta_2}

## Entregaveis
{lista exata de arquivos/docs a produzir}

## Quando terminar
Use TaskUpdate(task_id: \"{task_id}\", status: \"completed\") para marcar como concluido."
)
```

### 4.5 — TaskUpdate para atribuir owner

Apos spawnar cada teammate, atribua a tarefa:

```
TaskUpdate(
  task_id: "{id}",
  owner: "{nome-do-teammate}",
  status: "in_progress"
)
```

### 4.6 — AGUARDAR e spawnar proximas ondas

Voce sera notificado automaticamente quando teammates terminarem.
NAO faca polling. Apenas aguarde.

Quando um teammate terminar:
1. Verifique com TaskList quais tarefas foram desbloqueadas
2. Informe o usuario: "{nome} terminou. Spawnando {proximo}..."
3. Spawne a PROXIMA ONDA (repita 4.4 + 4.5 para os novos teammates)
4. Continue ate TODAS as tarefas estarem completed

### 4.7 — Encerrar

Quando tudo terminar:
1. SendMessage type "shutdown_request" para cada teammate
2. Apresente resumo final ao usuario (entregaveis, status, proximos passos)
3. TeamDelete para limpar

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
- NUNCA use subagent_type "Explore" para teammates — SEMPRE "general-purpose"
- SEMPRE inclua team_name em cada Task call
- NUNCA gere texto descrevendo as tool calls — CHAME as ferramentas de verdade
