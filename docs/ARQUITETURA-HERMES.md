# Arquitetura do Hermes Agent — Tópicos Principais

> **Data:** 2026-08-31
> **Fonte primária:** [`AGENTS.md`](https://github.com/NousResearch/hermes-agent/blob/main/AGENTS.md) (1.435 linhas, oficial) + árvore de `~/.hermes/hermes-agent/` + plugins/ reais.
> **Objetivo:** Mapear a arquitetura real do Hermes em 9 tópicos, pra alimentar o `agente-universo` com decisões fundamentadas.
> **Regra:** tudo aqui veio do código real. Quando a fonte for README/marketing, marco como ⚠️.

---

## 📏 TL;DR — O que é o Hermes em uma frase

Hermes é um **agente pessoal que roda o MESMO core (AIAgent) em 4+ surfaces** (CLI, TUI, Desktop, Gateway de mensagens com ~20 plataformas), tem **memória persistente cross-session**, **delegação pra subagentes**, **skills procedurais**, **plugins plugáveis** e **jobs agendados** — tudo self-hosted, MIT, Python-first.

**Princípios de design não-negociáveis** (do AGENTS.md, linhas 13-22):
- **Per-conversation prompt caching é sagrado** — nada que muta contexto/tools mid-conversation (única exceção: context compression).
- **Core é "narrow waist"; capability vive nas bordas** — toda nova tool é enviada em TODO API call; bar é alta pra tool nova no core; preferir CLI+skill, plugin, ou MCP.

---

## 🗺️ Visão geral — 9 tópicos

```
┌──────────────────────────────────────────────────────────────────────┐
│                         SURFACES (entrada do usuário)                │
│  CLI │ TUI (Ink/React) │ Desktop (Electron) │ Gateway (~20 canais)  │
└─────────────────────────────┬────────────────────────────────────────┘
                              │
┌─────────────────────────────▼────────────────────────────────────────┐
│  🧠 AIAgent (run_agent.py)  ── core loop síncrono + tool calling    │
│       │                                                            │
│       ├──► Model Providers (plugins/model-providers/)              │
│       ├──► Toolsets (registry auto-discovery)                      │
│       ├──► Memory (plugins/memory/ — honcho/mem0/OpenViking/...)   │
│       ├──► Context Engine (plugins/context_engine/)                │
│       └──► Sessions (hermes_state.py — SQLite FTS5)                │
└──────────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼────────────────────────────────────────┐
│  EDGE: Skills │ Plugins │ MCP servers │ ACP │ Cron │ Kanban │ Subagentes│
└──────────────────────────────────────────────────────────────────────┘
```

---

## 1. 🧠 Núcleo — AIAgent & Tool Loop

| Arquivo | LOC | O que faz |
|---|---|---|
| `run_agent.py` | ~12k | Classe `AIAgent`, loop de conversação |
| `model_tools.py` | — | Orquestração de tools, `discover_builtin_tools()`, `handle_function_call()` |
| `toolsets.py` | — | `_HERMES_CORE_TOOLS` (default bundle) + dicionário `TOOLSETS` |
| `hermes_state.py` | — | `SessionDB` (SQLite com FTS5 search) |
| `hermes_constants.py` | — | `get_hermes_home()`, profile-aware paths |
| `hermes_logging.py` | — | `agent.log`, `errors.log`, `gateway.log` |

### Loop central (síncrono, com interrupt + budget tracking)

```python
# run_agent.py — agent loop simplificado
while (api_call_count < self.max_iterations
       and self.iteration_budget.remaining > 0) \
       or self._budget_grace_call:
    if self._interrupt_requested: break
    response = client.chat.completions.create(
        model=model, messages=messages, tools=tool_schemas)
    if response.tool_calls:
        for tc in response.tool_calls:
            result = handle_function_call(tc.name, tc.args, task_id)
            messages.append(tool_result_message(result))
        api_call_count += 1
    else:
        return response.content
```

**Mensagens:** formato OpenAI `{"role": "system/user/assistant/tool", ...}`. Reasoning fica em `assistant_msg["reasoning"]`.

**Interface `AIAgent`:**
- `chat(message)` — simples, retorna string final.
- `run_conversation(user_message, system_message, history, task_id)` — completa, retorna dict com `final_response + messages`.
- `__init__` aceita ~60 params (credenciais, routing, callbacks, session, budget, fallback_model, checkpoints, etc.).

### Dependency chain (ordem de imports)

```
tools/registry.py  (sem deps — importado por tudo)
       ↑
tools/*.py  (cada um chama registry.register() no import)
       ↑
model_tools.py  (importa registry + dispara discovery)
       ↑
run_agent.py, cli.py, batch_runner.py, environments/
```

---

## 2. 💾 Sistemas Centrais

### 2.1 Memory (plugins/memory/)

**Pluggable via ABC.** Built-in providers (jun/2026): `honcho`, `mem0`, `supermemory`, `byterover`, `hindsight`, `holographic`, **`openviking`**, `retaindb`.

Cada provider implementa `MemoryProvider` (em `agent/memory_provider.py`) e é orquestrado por `agent/memory_manager.py`. Hooks do lifecycle:

| Hook | Quando |
|---|---|
| `sync_turn(turn_messages)` | Depois de cada turno concluído |
| `prefetch(query)` | Background, antes do LLM responder |
| `post_setup(hermes_home, config)` | Setup wizard (integração) |
| `shutdown()` | Cleanup no exit |

**Policy crítica (mai/2026):** `plugins/memory/` é **closed set** — novos providers DEVEM ser **standalone plugin repos** em `~/.hermes/plugins/` ou via pip entry points. PRs que adicionam diretório em `plugins/memory/` são fechadas.

**Policy (mai/2026):** plugins **NÃO podem modificar core** (`run_agent.py`, `cli.py`, `gateway/run.py`). Se precisarem de capability nova, expandem a surface genérica (novo hook ou método `ctx`).

### 2.2 Context Engine (plugins/context_engine/)

Plugado em `agent/context_engine.py`. Mesmo padrão ABC + orchestrator + per-plugin directory. OpenViking, LUCID, lossless-claw são exemplos.

### 2.3 Sessions (hermes_state.py)

`SessionDB` — SQLite + **FTS5 index** pra full-text search no histórico. Cada mensagem é uma row. Suporta `session_search()` (a ferramenta que você usa pra mim: "find the session where X").

### 2.4 Credentials & Budget

- `credential_pool` injetado no AIAgent — rotação de API keys, fallback model.
- `iteration_budget` — cap de tool-calling iterations + `_budget_grace_call` (uma chamada extra de margem).
- `CreditsState` (em `agent/credits_tracker.py`) rastreia spend em USD, depleted fraction, age.

---

## 3. 🔧 Tools & Toolsets

### 3.1 Tool registry (auto-discovery)

Toda tool em `tools/<name>.py` chama `registry.register(...)` no import. Sem declaração manual em lugar nenhum.

```python
# tools/registry.py — base
# tools/web_search.py — chama registry.register("web_search", fn, schema)
# model_tools.py — chama discover_builtin_tools() após imports
```

### 3.2 Toolsets (bundle per surface)

Definidos em `toolsets.py` (dicionário `TOOLSETS`). **Cada platform adapter** (Telegram, Discord, CLI, etc.) escolhe um toolset base.

**Toolsets atuais:** `browser`, `clarify`, `code_execution`, `cronjob`, `debugging`, `delegation`, `discord`, `discord_admin`, `feishu_doc`, `feishu_drive`, `file`, `homeassistant`, `image_gen`, `kanban`, **`memory`**, **`messaging`**, `moa`, `rl`, `safe`, `search`, `session_search`, `skills`, `spotify`, `terminal`, `todo`, `tts`, `video`, `vision`, `web`, `yuanbao`.

**Enable/disable por platform:**
- `hermes tools` (UI curses)
- `tools.<platform>.enabled/disabled` no `config.yaml`

### 3.3 Environments (tool/terminal backends)

`tools/environments/` — backends onde o terminal roda: `local`, `docker`, `ssh`, `modal`, `daytona`, `singularity`. Trocar o environment muda onde comandos shell executam.

---

## 4. 📚 Skills (procedurais)

### 4.1 Duas superfícies paralelas

| Diretório | Ativação | Tamanho |
|---|---|---|
| `skills/` | Built-in, ativo por default | Leve-médio |
| `optional-skills/` | Shipped mas **NÃO** ativo; instala via `hermes skills install official/<cat>/<skill>` | Pesado/nicho |

Categorias em `optional-skills/`: `autonomous-ai-agents`, `blockchain`, `communication`, `creative`, `devops`, `email`, `health`, `mcp`, `migration`, `mlops`, `productivity`, `research`, `security`, `web-development`.

### 4.2 SKILL.md frontmatter (padrão)

```yaml
name: skill-name
description: ≤60 chars, uma frase, termina com ponto. (HARDLINE)
version: 1.0.0
author: Real Name <github>   # humano PRIMEIRO
license: MIT
platforms: [linux, macos]    # gating
metadata:
  hermes:
    tags: [t1, t2]
    category: mlops
    related_skills: [other-skill]
    config:                       # settings injetados no load
      api_key_env: MY_API_KEY
```

### 4.3 Padrões de skill (HARDLINE — reviewer rejeita PR se violar)

1. `description` ≤ 60 chars, sem marketing ("powerful", "seamless").
2. Tools referenciadas no prose = tools nativas Hermes **nomeadas em backticks** (`` `terminal` ``, `` `web_extract` ``, `` `read_file` ``, `` `delegate_task` ``) — NUNCA `grep`/`cat`/`sed`/`find`/`ls` que o agent já tem wrapped.
3. `platforms:` auditado contra imports reais (POSIX-only? `fcntl`, `termios`, `setsid`, `/proc`, `SIGKILL` → gate).
4. `author` credita humano primeiro (não "Hermes Agent").
5. Body: `# <Skill> Skill` → intro 2-3 frases → `## When to Use`, `## Prerequisites`, `## How to Run`, `## Quick Reference`, `## Procedure`, `## Pitfalls`, `## Verification`. ~200 linhas complexo, ~100 simples.
6. Scripts em `scripts/`, refs em `references/`, templates em `templates/`.
7. Tests em `tests/skills/test_<skill>_skill.py`, só stdlib + pytest + `unittest.mock` (sem rede).
8. `.env.example` additions isoladas em bloco delimitado.

### 4.4 Curator (lifecycle automático)

Background skill-maintenance em `agent/curator.py`:
- Trackea usage (sidecar `~/.hermes/skills/.usage.json`: `use_count`, `view_count`, `last_activity_at`, `state`).
- Auto-archives skills stale → `~/.hermes/skills/.archive/` (restorable).
- Backups pré-run: `agent/curator_backup.py` (tar.gz).
- CLI: `hermes curator <verb>` → `status`, `run`, `pause`, `resume`, `pin`, `unpin`, `archive`, `restore`, `prune`, `backup`, `rollback`.

---

## 5. 🔌 Plugins (3 superfícies)

### 5.1 Plugins gerais (`hermes_cli/plugins.py` + `plugins/<name>/`)

Discovery em `~/.hermes/plugins/`, `./.hermes/plugins/`, pip entry points. Cada plugin expõe `register(ctx)` que pode:

| Capability | Como |
|---|---|
| Lifecycle hooks | `ctx.pre_tool_call`, `post_tool_call`, `pre_llm_call`, `post_llm_call`, `on_session_start`, `on_session_end` |
| Tools | `ctx.register_tool(...)` |
| CLI subcommands | `ctx.register_cli_command(...)` — argparse wired no `hermes <pluginname> <subcmd>` |

**Pitfall:** `discover_plugins()` só roda como side-effect de importar `model_tools.py`. Code paths que leem plugin state sem esse import devem chamar `discover_plugins()` explicit (idempotente).

### 5.2 Model-provider plugins (`plugins/model-providers/`)

Todo inference backend (openrouter, anthropic, gmi, deepseek, nvidia...) é plugin aqui. Cada `__init__.py` chama `providers.register_provider(ProviderProfile(...))` no load.

**Discovery é lazy e separado:**
1. Bundled: `<repo>/plugins/model-providers/<name>/`
2. User: `$HERMES_HOME/plugins/model-providers/<name>/`
3. Legacy: `<repo>/providers/<name>.py` (back-compat)

User plugins de mesmo nome **override** bundled (last-writer-wins). Terceiros podem trocar qualquer built-in profile sem patch no repo.

### 5.3 Third-party products (POLICY jun/2026)

Plugins que integram produto de terceiro (observability backends, vendor SaaS, dashboards, paid services) **DEVEM ser standalone plugin repos** instalados em `~/.hermes/plugins/` ou via pip entry point. **Não** entram no tree. Razão: maintenance burden contra core em movimento rápido, pra backend que não é nosso. Promovidos no Discord `#plugins-skills-and-skins`.

---

## 6. 🖥️ Surfaces (entry points)

| Surface | Como | Engine |
|---|---|---|
| **CLI** (`hermes`) | `cli.py` (~11k LOC, `HermesCLI`) | Rich + prompt_toolkit, skin engine (`hermes_cli/skin_engine.py`), KawaiiSpinner (`agent/display.py`) |
| **TUI** (`hermes --tui`) | `ui-tui/` (Ink/React) | Node stdio ↔ Python `tui_gateway/` (JSON-RPC) |
| **Desktop** (Electron) | `apps/desktop/` | Electron + mesma engine via IPC |
| **ACP adapter** | `acp_adapter/` | VS Code / Zed / JetBrains integration |
| **Web** | `web/` | Dashboard plugin (`plugins/web/`) |
| **Gateway** | `gateway/` | Mensageria — ver seção 7 |

### Slash command registry

Central em `COMMAND_REGISTRY` (lista de `CommandDef`), em `hermes_cli/commands.py`. Todos consumidores derivam automaticamente:

- **CLI** → `process_command()` resolve aliases via `resolve_command()`.
- **Gateway** → `GATEWAY_KNOWN_COMMANDS` (frozenset) + dispatch.
- **Telegram** → `telegram_bot_commands()` gera BotCommand menu.
- **Slack** → `slack_subcommand_map()` gera `/hermes` routing.
- **Autocomplete** → `COMMANDS` dict alimenta `SlashCommandCompleter`.

`CommandDef` fields: `name`, `description`, `category` (Session / Configuration / Tools & Skills / Info / Exit), `aliases`, `args_hint`, `cli_only`, `gateway_only`, `gateway_config_gate`.

**Skill slash commands** são injetados como **user message** (NÃO system prompt) — preserve prompt caching.

---

## 7. 📡 Gateway (mensageria)

### 7.1 Arquivos principais

- `gateway/run.py` — runner principal, JSON-RPC style.
- `gateway/session.py` — session lifecycle.
- `gateway/platforms/` — **um adapter por plataforma**.
- `gateway/builtin_hooks/` — extension point (zero shipped hoje).

### 7.2 Plataformas suportadas (~20 adapters)

`telegram`, `discord`, `slack`, `whatsapp` (cloud + bridge), `homeassistant`, `signal`, `matrix`, `mattermost`, `email`, `sms`, `dingtalk`, `wecom`, `weixin`, `feishu`, `qqbot`, `bluebubbles`, `yuanbao`, `webhook`, `api_server`, `irc`, ...

Cada adapter herda `base.py` (helpers compartilhados).

### 7.3 Duas message guards (pitfall crítico)

Quando o agent tá rodando, mensagens passam por 2 guards sequenciais:

1. **`base.py` adapter** → fila `_pending_messages` quando `session_key in self._active_sessions`.
2. **`gateway/run.py` runner** → intercepta `/stop`, `/new`, `/queue`, `/status`, `/approve`, `/deny` ANTES de chegar em `running_agent.interrupt()`.

Qualquer command novo que precise chegar no runner enquanto o agent tá bloqueado (tipo approval prompt) **DEVE bypassar AMBOS guards** e ser dispatched inline, NÃO via `_process_message_background()` (racea session lifecycle).

---

## 8. ⏱️ Background work

### 8.1 Cron (scheduled jobs)

- Store: `cron/jobs.py`. Loop: `cron/scheduler.py`.
- **Schedules:** `"30m"`, `"2h"`, `"1d"`, `"every 2h"`, `"every monday 9am"`, `"0 9 * * *"`, ISO timestamp one-shot.
- **Per-job fields:** `skills` (load específicos), `model`/`provider` overrides, `script` (pre-run data-collection; `no_agent=True` → script É o job), `context_from` (chain output), `workdir` (carrega AGENTS.md/CLAUDE.md do dir), multi-platform delivery.
- **Hardening:** 3-min hard interrupt (anti runaway), catchup window (½ período clamped 120s–2h), grace window 120s one-shot, file lock `~/.hermes/cron/.tick.lock` previne duplicate ticks, `skip_memory=True` (memory providers não rodam em cron).
- **Cron deliveries NÃO espelham pra gateway session** — ficam em cron session própria com header/footer pra preservar message-role alternation.

### 8.2 Kanban (multi-agent work queue)

SQLite-backed board compartilhado entre múltiplos profiles/workers.

- **CLI:** `hermes kanban <verb>` → `init`, `create`, `list`, `show`, `assign`, `link`, `comment`, `attach`, `complete`, `block`, `archive`, `tail`, `dispatch`, `daemon`, `gc`, ...
- **Toolset:** `tools/kanban_tools.py` expõe `kanban_show`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, etc. Profiles fora de dispatcher-spawned task também ganham `kanban_list` e `kanban_unblock`.
- **Dispatcher:** loop long-lived (default 60s tick) que reclama claims stale, promove ready, atomic claim, spawna assigned profiles. Roda **inside gateway** por default (`kanban.dispatch_in_gateway: true`).
- **Plugin assets:** `plugins/kanban/dashboard/` (web UI) + `plugins/kanban/systemd/` (dispatcher standalone).
- **Isolation:** Board é hard boundary (workers spawnados com `HERMES_KANBAN_BOARD` pinned no env). Tenant é soft namespace within board. Após `kanban.failure_limit` (default 2) tentativas não-successful, auto-block pra prevenir spin loop.

### 8.3 Delegation (subagentes)

`tools/delegate_tool.py` spawna subagent com context + terminal isolados.

**Dois shapes:**
- **Single:** `goal` (+ `context`, `toolsets`).
- **Batch (parallel):** `tasks: [...]`. Concurrency cap por `delegation.max_concurrent_children` (default **3**).

**Roles:**
- `leaf` (default) — não pode chamar `delegate_task`, `clarify`, `memory`, `send_message`, `cronjob`. Retém `execute_code`.
- `orchestrator` — retém `delegate_task`. Gated por `delegation.orchestrator_enabled` (default true), bounded por `delegation.max_spawn_depth` (default **2**).

**Config knobs:** `max_concurrent_children`, `max_spawn_depth`, `child_timeout_seconds`, `orchestrator_enabled`, `subagent_auto_approve`, `inherit_mcp_toolsets`, `max_iterations`.

**Durability:** background `delegate_task` é process-local. Pra trabalho que sobreviva restart, usar `cronjob` ou `terminal(background=True, notify_on_complete=True)`.

### 8.4 Terminal background

`terminal(background=True, notify_on_complete=True)` — gateway tem watcher que detecta completion e dispara novo agent turn. Verbosity via `display.background_process_notifications`:
- `all` — running + final (default)
- `result` — só final
- `error` — só final quando exit != 0
- `off` — nada

---

## 9. 🔁 Cross-cutting

### 9.1 Profiles (multi-instance)

Hermes suporta **profiles** — múltiplas instâncias totalmente isoladas, cada uma com seu próprio `HERMES_HOME` (config, API keys, memory, sessions, skills, gateway).

**Mecanismo:** `_apply_profile_override()` em `hermes_cli/main.py` seta `HERMES_HOME` ANTES de qualquer import. Todos `get_hermes_home()` automaticamente escopam pro profile ativo.

**Regras profile-safe (HARD):**
1. **Usar `get_hermes_home()`** (NUNCA `Path.home() / ".hermes"` em code).
2. Usar `display_hermes_home()` pra mensagens user-facing.
3. Module-level constants OK (cacheiam `get_hermes_home()` no import, depois de `_apply_profile_override()`).
4. Tests que mockam `Path.home()` devem também setar `HERMES_HOME`.
5. **Gateway adapters devem usar token locks** — se conecta com credencial única (bot token, API key), chama `acquire_scoped_lock()` em `connect/start` e `release_scoped_lock()` em `disconnect/stop`. Pattern: `plugins/platforms/irc/adapter.py`.
6. **Profile ops são HOME-anchored, não HERMES_HOME-anchored** — `_get_profiles_root()` retorna `Path.home() / ".hermes" / "profiles"`, NÃO `get_hermes_home() / "profiles"`. Permite `hermes -p coder profile list` ver todos profiles.

### 9.2 Logging

- `~/.hermes/logs/agent.log` (INFO+).
- `~/.hermes/logs/errors.log` (WARNING+).
- `~/.hermes/logs/gateway.log` (quando gateway rodando).
- Profile-aware via `get_hermes_home()`.
- Browse: `hermes logs [--follow] [--level ...] [--session ...]`.

### 9.3 Config & .env

- `~/.hermes/config.yaml` — **TODAS** as settings comportamentais (timeouts, thresholds, feature flags, display prefs).
- `~/.hermes/.env` — **APENAS secrets** (API keys, tokens, passwords).
- **Policy:** Rejectar PRs que dizem "set X in your .env" a menos que X seja credencial. Bridge pra env var interno se a mechanism precisar, mas docs user-facing apontam `config.yaml`.

### 9.4 MCP (Model Context Protocol)

- Suporte MCP nativo em `packages/mcp/`.
- Tools podem ser MCP servers registrados no catálogo.
- Skill pode declarar dependência em MCP server no `## Prerequisites` e usar via tool wrapping.

### 9.5 ACP (Agent Client Protocol)

`acp_adapter/` — server ACP pra integração VS Code / Zed / JetBrains.

### 9.6 Observability

Plugin em `plugins/observability/` — metrics / traces / logs. **Policy jun/2026:** third-party products (Langfuse, Datadog, ...) NÃO entram no tree — viram standalone plugin repo.

### 9.7 Skin/Theme system

`hermes_cli/skin_engine.py` — data-driven CLI theming. Inicializado de `display.skin` no startup. Customiza: banner colors, spinner faces/verbs/wings, tool prefix, response box, branding text.

---

## 📊 Decisões de arquitetura que importam pro agente-universo

| # | Decisão do Hermes | Implicação pro nosso agente |
|---|---|---|
| 1 | Per-conv prompt caching sagrado | Slot `contextEngine` deve operar ANTES do LLM, NUNCA mid-conversation |
| 2 | Plugins NÃO modificam core | Nosso `agente-universo` precisa de plugin surface rica (hooks, ctx methods) desde o dia 1 |
| 3 | Memory providers = closed set + ABC | Memory vira plugin ABC, novos providers são standalone |
| 4 | Skills: 2 surfaces (built-in + optional) | Nosso catálogo unificado (Hermes 42 + OpenClaw 5k + DSH ~300) precisa do mesmo split |
| 5 | Toolsets per platform | Gateway do agente-universo precisa de toolset por canal (Telegram ≠ CLI ≠ Web) |
| 6 | Slash commands via registry central | 1 lugar pra adicionar comandos = 5 surfaces atualizam |
| 7 | Background work = cron + kanban + delegation + terminal bg | Vamos precisar dos 4 (não só cron) pra suportar multi-agent |
| 8 | Profiles via HERMES_HOME override | Multi-tenant / multi-instance = trivial, mesma base |
| 9 | Gateway com 2 message guards | Qualquer approval prompt precisa bypass inline |
| 10 | `.env` = só secrets, `config.yaml` = settings | Adopt essa divisão pra evitar 2 anos de debt |

---

## 📚 Fontes consultadas

| # | Fonte | Local |
|---|---|---|
| 1 | `AGENTS.md` (oficial, 1.435 linhas) | `~/.hermes/hermes-agent/AGENTS.md` |
| 2 | Árvore de código real | `~/.hermes/hermes-agent/{run_agent,cli,hermes_state,hermes_constants,model_tools,toolsets}.py` |
| 3 | Plugins/ (20+ diretórios) | `~/.hermes/hermes-agent/plugins/{memory,context_engine,model-providers,kanban,observability,...}/` |
| 4 | Gateway (60+ arquivos) | `~/.hermes/hermes-agent/gateway/{run,session,platforms/*}.py` |
| 5 | Docs user-facing (inglês) | `~/.hermes/hermes-agent/website/docs/user-guide/skills/bundled/autonomous-ai-agents/autonomous-ai-agents-hermes-agent.md` |
| 6 | Docs design (Docusaurus EN) | `~/.hermes/hermes-agent/website/docs/user-guide/skills/bundled/software-development/software-development-hermes-agent-skill-authoring.md` |

---

## 📋 Próximos passos (não fazer — só pesquisar)

1. [ ] Adicionar este arquivo ao PLANO-FINAL-v2-CORRIGIDO.md como referência arquitetural
2. [ ] Cruzar cada decisão da seção "Decisões que importam pro agente-universo" com as 7 fases do plano v2
3. [ ] Identificar gaps onde DSH NÃO cobre o que Hermes faz (provavelmente: skin engine, kanban dispatcher, ACP, multi-platform delivery do cron)
