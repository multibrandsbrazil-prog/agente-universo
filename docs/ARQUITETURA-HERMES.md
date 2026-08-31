# Arquitetura do Hermes Agent — Estrutura por Partes

> **Data:** 2026-08-31
> **Fonte:** `~/.hermes/hermes-agent/AGENTS.md` (oficial, 1.435 linhas) + árvore de código real em `~/.hermes/hermes-agent/`.
> **Objetivo:** Inventário estruturado da arquitetura do Hermes, separado em partes para posterior comparação com o `agente-universo`.
> **Escopo:** Apenas descrever. Sem análise, sem opinião, sem implicações.

---

## 📐 Camadas (de fora pra dentro)

```
┌─────────────────────────────────────────────────────────────┐
│ PARTE 1 — ENTRY POINTS (como o usuário alcança o agente)    │
│   CLI │ TUI │ Desktop │ ACP │ Web │ Gateway                 │
├─────────────────────────────────────────────────────────────┤
│ PARTE 2 — DISTRIBUTION (como mensagens viram turnos)         │
│   Slash command registry │ Message guards │ Delivery         │
├─────────────────────────────────────────────────────────────┤
│ PARTE 3 — AGENT LOOP (o coração síncrono)                   │
│   AIAgent │ Conversation loop │ Budget │ Interrupts         │
├─────────────────────────────────────────────────────────────┤
│ PARTE 4 — CAPABILITIES (o que o loop chama)                 │
│   Tools │ Toolsets │ Environments │ Skills │ Subagentes     │
├─────────────────────────────────────────────────────────────┤
│ PARTE 5 — CONTEXT & MEMORY (o que o loop lê)                │
│   System prompt │ SessionDB │ Memory providers │ Context    │
│             engines │ Caching invariants                    │
├─────────────────────────────────────────────────────────────┤
│ PARTE 6 — INFERENCE (como as chamadas saem pra LLM)         │
│   Model providers │ Credential pool │ Fallback │ Budget    │
├─────────────────────────────────────────────────────────────┤
│ PARTE 7 — EXTENSIBILITY (como terceiros plugam)             │
│   Plugin ABCs │ Hooks │ Skill registry │ MCP                │
├─────────────────────────────────────────────────────────────┤
│ PARTE 8 — BACKGROUND (trabalho que não é request/response)  │
│   Cron │ Kanban │ Delegation │ Terminal bg                  │
├─────────────────────────────────────────────────────────────┤
│ PARTE 9 — PERSISTENCE & ISOLATION (onde tudo mora)          │
│   HERMES_HOME │ Profiles │ Config vs .env │ Logging         │
├─────────────────────────────────────────────────────────────┤
│ PARTE 10 — META (quem cuida da própria estrutura)           │
│   Curator (skill lifecycle) │ Skin engine │ Contribución    │
└─────────────────────────────────────────────────────────────┘
```

---

## PARTE 1 — Entry Points (Surfaces)

| Surface | Arquivo principal | Engine | Como ativa |
|---|---|---|---|
| **CLI** | `cli.py` (~11k LOC) | Rich + prompt_toolkit | `hermes` |
| **TUI** | `ui-tui/src/` (Ink/React) ↔ `tui_gateway/` (Python) | Node stdio ↔ Python JSON-RPC | `hermes --tui` ou `HERMES_TUI=1` |
| **Desktop** | `apps/desktop/` | Electron | App standalone |
| **ACP adapter** | `acp_adapter/` | VS Code / Zed / JetBrains integration | Plugin IDE |
| **Web dashboard** | `web/` + `plugins/web/` | SPA | Plugin |
| **Gateway** | `gateway/run.py` | Multi-plataforma mensageria | `hermes gateway start` |

Todas as surfaces consomem o **mesmo core `AIAgent`** (run_agent.py). Cada surface pega um toolset base de `toolsets.py`.

---

## PARTE 2 — Distribution

### 2.1 Slash command registry

- **Localização:** `hermes_cli/commands.py` — lista central `COMMAND_REGISTRY` de `CommandDef`.
- **Consumidores que derivam automaticamente:**
  - CLI (`process_command()`)
  - Gateway (`GATEWAY_KNOWN_COMMANDS` frozenset)
  - Telegram (`telegram_bot_commands()`)
  - Slack (`slack_subcommand_map()`)
  - Autocomplete (`SlashCommandCompleter`)
  - Help (`COMMANDS_BY_CATEGORY`)
- **Campos do `CommandDef`:** `name`, `description`, `category`, `aliases`, `args_hint`, `cli_only`, `gateway_only`, `gateway_config_gate`.
- **Categorias:** Session | Configuration | Tools & Skills | Info | Exit.

### 2.2 Message guards (gateway)

Sequência quando o agent está rodando:

1. **Base adapter** (`gateway/platforms/base.py`) — fila `_pending_messages` quando `session_key in self._active_sessions`.
2. **Gateway runner** (`gateway/run.py`) — intercepta `/stop`, `/new`, `/queue`, `/status`, `/approve`, `/deny` antes de chegar em `running_agent.interrupt()`.

Qualquer command que precise bypassar para approval prompts é dispatched inline, **não** via `_process_message_background()`.

### 2.3 Delivery & background notifications

- `gateway/delivery.py` + `gateway/delivery_ledger.py` — roteamento de respostas.
- `terminal(background=True, notify_on_complete=True)` — watcher detecta completion e dispara novo agent turn.
- Verbosidade: `display.background_process_notifications` (`all`/`result`/`error`/`off`).

---

## PARTE 3 — Agent Loop (núcleo)

### 3.1 Classe `AIAgent`

- **Arquivo:** `run_agent.py` (~12k LOC)
- **`__init__` aceita ~60 parâmetros:** `base_url`, `api_key`, `provider`, `api_mode` (`"chat_completions"` | `"codex_responses"` | ...), `model`, `max_iterations`, `enabled_toolsets`, `disabled_toolsets`, `quiet_mode`, `save_trajectories`, `platform`, `session_id`, `skip_context_files`, `skip_memory`, `credential_pool`, callbacks, `thread/user/chat IDs`, `iteration_budget`, `fallback_model`, `checkpoints_config`, `prefill_messages`, `service_tier`, `reasoning_config`.

### 3.2 Interface pública

```python
chat(message: str) -> str                  # simples
run_conversation(user_message, system_message=None,
                 conversation_history=None,
                 task_id=None) -> dict     # completa (final_response + messages)
```

### 3.3 Loop interno (`run_conversation`)

Síncrono, com interrupt checks e budget tracking.

```python
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

### 3.4 Formato de mensagens

OpenAI schema: `{"role": "system/user/assistant/tool", ...}`. Reasoning fica em `assistant_msg["reasoning"]`.

### 3.5 File dependency chain

```
tools/registry.py         (no deps — importado por todos)
       ↑
tools/*.py                (cada um chama registry.register() no import)
       ↑
model_tools.py            (importa registry + dispara discover_builtin_tools())
       ↑
run_agent.py, cli.py, batch_runner.py, environments/
```

---

## PARTE 4 — Capabilities (Tools, Toolsets, Skills, Subagentes)

### 4.1 Tool registry

- **Localização:** `tools/registry.py`
- **Auto-discovery:** cada `tools/<name>.py` chama `registry.register(...)` no import; `model_tools.py` chama `discover_builtin_tools()` para disparar.

### 4.2 Toolsets (bundles por surface)

- **Localização:** `toolsets.py` — dicionário `TOOLSETS`.
- **Padrão:** `_HERMES_CORE_TOOLS` é o bundle default que a maioria das platforms herda.
- **Lista atual (31):** `browser`, `clarify`, `code_execution`, `cronjob`, `debugging`, `delegation`, `discord`, `discord_admin`, `feishu_doc`, `feishu_drive`, `file`, `homeassistant`, `image_gen`, `kanban`, `memory`, `messaging`, `moa`, `rl`, `safe`, `search`, `session_search`, `skills`, `spotify`, `terminal`, `todo`, `tts`, `video`, `vision`, `web`, `yuanbao`.
- **Enable/disable por platform:** `hermes tools` (UI curses) ou `tools.<platform>.enabled/disabled` em `config.yaml`.

### 4.3 Environments (backends do terminal)

`tools/environments/`: `local`, `docker`, `ssh`, `modal`, `daytona`, `singularity`. Trocar o environment muda onde comandos shell executam.

### 4.4 Skills (procedurais)

#### 4.4.1 Duas superfícies paralelas

| Diretório | Ativação | Conteúdo típico |
|---|---|---|
| `skills/` | Ativo por default | Leve-médio |
| `optional-skills/` | `hermes skills install official/<cat>/<skill>` | Pesado/nicho |

#### 4.4.2 Categorias em `optional-skills/`

`autonomous-ai-agents`, `blockchain`, `communication`, `creative`, `devops`, `email`, `health`, `mcp`, `migration`, `mlops`, `productivity`, `research`, `security`, `web-development`.

#### 4.4.3 SKILL.md frontmatter (campos)

```yaml
name: skill-name
description: "≤60 chars, uma frase, termina com ponto"
version: 1.0.0
author: Real Name <github>          # humano primeiro
license: MIT
platforms: [linux, macos]           # OS gating
metadata:
  hermes:
    tags: [...]
    category: ...
    related_skills: [...]
    config:                          # settings injetados no load
      api_key_env: MY_API_KEY
```

#### 4.4.4 Estrutura de diretórios por skill

```
skills/<category>/<skill>/
├── SKILL.md                  # corpo da skill
├── scripts/                  # helpers
├── references/               # docs longas
├── templates/                # arquivos modelo
└── tests/skills/test_*.py    # pytest, só stdlib + unittest.mock
```

#### 4.4.5 8 Padrões HARDLINE (rejeitam PR se violar)

1. `description` ≤ 60 chars, sem marketing.
2. Tools referenciadas no prose = tools nativas Hermes em backticks (NUNCA `grep`/`cat`/`sed`/`find`/`ls`).
3. `platforms:` auditado contra imports reais (POSIX-only → gate).
4. `author` credita humano primeiro.
5. Body: `# <Skill> Skill` → intro 2-3 frases → `## When to Use` / `## Prerequisites` / `## How to Run` / `## Quick Reference` / `## Procedure` / `## Pitfalls` / `## Verification`.
6. Scripts em `scripts/`, refs em `references/`, templates em `templates/`.
7. Tests em `tests/skills/test_<skill>_skill.py`, sem rede.
8. `.env.example` additions isoladas em bloco delimitado.

#### 4.4.6 Skill slash commands

`agent/skill_commands.py` scanneia `~/.hermes/skills/` e injeta skills como **user message** (não system prompt) — preserva prompt caching.

### 4.5 Delegation (subagentes)

- **Tool:** `tools/delegate_tool.py`.
- **Shapes:**
  - `goal` (+ `context`, `toolsets`) — single.
  - `tasks: [...]` — batch paralelo, concurrency cap por `delegation.max_concurrent_children` (default **3**).
- **Roles:**
  - `leaf` (default) — não chama `delegate_task`, `clarify`, `memory`, `send_message`, `cronjob`. Retém `execute_code`.
  - `orchestrator` — retém `delegate_task`. Gated por `delegation.orchestrator_enabled` (default true), bounded por `delegation.max_spawn_depth` (default **2**).
- **Config knobs:** `max_concurrent_children`, `max_spawn_depth`, `child_timeout_seconds`, `orchestrator_enabled`, `subagent_auto_approve`, `inherit_mcp_toolsets`, `max_iterations`.
- **Durability:** background `delegate_task` é process-local. Pra sobreviver restart, usar `cronjob` ou `terminal(background=True, notify_on_complete=True)`.

---

## PARTE 5 — Context & Memory

### 5.1 System prompt

- Fixo por toda a conversação (cache-friendly).
- Composto por: MEMORY (do provider) + USER profile + tools + skills injetadas como user message.
- **Invariante:** byte-stable pelo tempo de vida da conversa. Única exceção: context compression.

### 5.2 Memory providers (plugins/memory/)

- **Padrão:** ABC `MemoryProvider` (em `agent/memory_provider.py`) + orchestrator `agent/memory_manager.py`.
- **Built-in providers (jun/2026):** `honcho`, `mem0`, `supermemory`, `byterover`, `hindsight`, `holographic`, **`openviking`**, `retaindb`.
- **Lifecycle hooks:**
  - `sync_turn(turn_messages)` — após cada turno.
  - `prefetch(query)` — background antes do LLM.
  - `post_setup(hermes_home, config)` — wizard de setup.
  - `shutdown()` — cleanup.
- **Policy:** `plugins/memory/` é **closed set**. Novos providers DEVEM ser standalone plugin repo em `~/.hermes/plugins/` ou pip entry point.

### 5.3 Context engines (plugins/context_engine/)

- Plugado em `agent/context_engine.py`.
- Mesmo padrão ABC + orchestrator + per-plugin directory.
- Exemplos: OpenViking, LUCID, lossless-claw.

### 5.4 Session store

- **Localização:** `hermes_state.py` — `SessionDB` (SQLite + FTS5 full-text search).
- Cada mensagem vira row; suporta `session_search()` (full-text recall).

### 5.5 Prompt caching invariants (sacred)

- **NÃO** alterar contexto mid-conversation.
- **NÃO** trocar toolsets mid-conversation.
- **NÃO** reload memories ou rebuild system prompts mid-conversation.
- Única exceção: context compression.
- Commands que mutam system-prompt state são **cache-aware** (default deferred invalidation, opt-in `--now`).

---

## PARTE 6 — Inference

### 6.1 Model providers (plugins/model-providers/)

Todo inference backend (openrouter, anthropic, gmi, deepseek, nvidia...) é plugin.

- Cada `__init__.py` chama `providers.register_provider(ProviderProfile(...))` no load.
- **Discovery é lazy e separado** do PluginManager geral:
  1. Bundled: `<repo>/plugins/model-providers/<name>/`
  2. User: `$HERMES_HOME/plugins/model-providers/<name>/`
  3. Legacy: `<repo>/providers/<name>.py`
- **User overrides bundled** (last-writer-wins).

### 6.2 API modes

`api_mode` aceita: `"chat_completions"` | `"codex_responses"` | ...

### 6.3 Credential pool

- `credential_pool` injetado no AIAgent — rotação de API keys, fallback model.
- `iteration_budget` — cap de tool-calling iterations + `_budget_grace_call`.

### 6.4 Credits tracking

- `agent/credits_tracker.py` — `CreditsState` rastreia spend em USD, depleted fraction, age.
- `display.background_process_notifications` controla verbosidade.

---

## PARTE 7 — Extensibility

### 7.1 Plugins gerais (`hermes_cli/plugins.py` + `plugins/<name>/`)

- **Discovery:** `~/.hermes/plugins/`, `./.hermes/plugins/`, pip entry points.
- **Cada plugin expõe `register(ctx)`** que pode:
  - Hooks: `pre_tool_call`, `post_tool_call`, `pre_llm_call`, `post_llm_call`, `on_session_start`, `on_session_end`.
  - Tools: `ctx.register_tool(...)`.
  - CLI subcommands: `ctx.register_cli_command(...)` — argparse wired no `hermes <pluginname> <subcmd>`.
- **Pitfall:** `discover_plugins()` só roda como side-effect de importar `model_tools.py`.

### 7.2 Memory plugins

Ver PARTE 5.2.

### 7.3 Model-provider plugins

Ver PARTE 6.1.

### 7.4 MCP (Model Context Protocol)

- Suporte MCP nativo em `packages/mcp/`.
- Skills podem declarar dependência em MCP server no `## Prerequisites`.
- Roda via `mcp__*` toolset quando habilitado.

### 7.5 Third-party product policy (jun/2026)

Plugins que integram produto de terceiro (observability backends, vendor SaaS, dashboards) DEVEM ser standalone plugin repos instalados em `~/.hermes/plugins/`. **Não** entram no tree.

---

## PARTE 8 — Background Work

### 8.1 Cron (scheduled jobs)

- **Arquivos:** `cron/jobs.py` (store) + `cron/scheduler.py` (tick loop).
- **Schedules suportados:**
  - Duration: `"30m"`, `"2h"`, `"1d"`
  - "every": `"every 2h"`, `"every monday 9am"`
  - 5-field cron: `"0 9 * * *"`
  - ISO timestamp one-shot: `"2026-06-01T09:00:00Z"`
- **Per-job fields:** `skills`, `model`/`provider` overrides, `script` (pre-run data-collection; `no_agent=True` → script É o job), `context_from` (chain output), `workdir` (carrega AGENTS.md/CLAUDE.md), multi-platform delivery.
- **Hardening:** 3-min hard interrupt, catchup window (½ período clamped 120s–2h), grace window 120s one-shot, file lock `~/.hermes/cron/.tick.lock`, `skip_memory=True` em cron sessions.
- **Delivery:** cron NÃO espelha pra gateway session — fica em cron session própria com header/footer.

### 8.2 Kanban (multi-agent work queue)

- SQLite-backed board compartilhado entre múltiplos profiles/workers.
- **CLI verbs:** `init`, `create`, `list`/`ls`, `show`, `assign`, `link`, `unlink`, `comment`, `attach`, `attachments`, `attach-rm`, `complete`, `block`, `unblock`, `archive`, `tail`, `watch`, `stats`, `runs`, `log`, `assignees`, `heartbeat`, `notify-*`, `dispatch`, `daemon`, `gc`.
- **Toolset (`tools/kanban_tools.py`):** `kanban_show`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, `kanban_comment`, `kanban_create`, `kanban_link`, `kanban_attach`, `kanban_attach_url`, `kanban_attachments`; profiles com toolset `kanban` fora de dispatcher-spawned task também ganham `kanban_list`, `kanban_unblock`.
- **Dispatcher:** loop long-lived (default 60s tick) — reclama stale claims, promove ready, atomic claim, spawna assigned profiles. Roda **inside gateway** por default (`kanban.dispatch_in_gateway: true`).
- **Plugin assets:** `plugins/kanban/dashboard/` (web UI) + `plugins/kanban/systemd/` (`hermes-kanban-dispatcher.service`).
- **Isolation:** Board = hard boundary (`HERMES_KANBAN_BOARD` pinned no env dos workers). Tenant = soft namespace within board. `kanban.failure_limit` (default **2**) tentativas não-successful → auto-block.

### 8.3 Delegation

Ver PARTE 4.5.

### 8.4 Terminal background

- `terminal(background=True, notify_on_complete=True)` — gateway watcher detecta completion e dispara novo agent turn.
- Verbosity: `all`/`result`/`error`/`off`.

---

## PARTE 9 — Persistence & Isolation

### 9.1 HERMES_HOME & perfis

- **`HERMES_HOME`** — env var que define o diretório raiz (config, API keys, memory, sessions, skills, gateway).
- **Profile mechanism:** `_apply_profile_override()` em `hermes_cli/main.py` seta `HERMES_HOME` antes de qualquer import. Todos `get_hermes_home()` escopam pro profile ativo.
- **`get_hermes_home()`** (código) vs **`display_hermes_home()`** (user-facing messages).
- **Profile operations são HOME-anchored**, não HERMES_HOME-anchored — `_get_profiles_root()` retorna `Path.home() / ".hermes" / "profiles"`, permitindo `hermes -p coder profile list` ver todos.

### 9.2 Config vs .env

- `~/.hermes/config.yaml` — **TODAS** as settings comportamentais (timeouts, thresholds, feature flags, display prefs).
- `~/.hermes/.env` — **APENAS secrets** (API keys, tokens, passwords).
- **Policy:** rejectar PRs que dizem "set X in .env" a menos que X seja credencial.

### 9.3 Logging

- `~/.hermes/logs/agent.log` (INFO+).
- `~/.hermes/logs/errors.log` (WARNING+).
- `~/.hermes/logs/gateway.log` (quando gateway rodando).
- Browse: `hermes logs [--follow] [--level ...] [--session ...]`.

### 9.4 User home layout

```
~/.hermes/
├── config.yaml
├── .env
├── logs/
├── sessions/                  # SQLite SessionDB
├── skills/                    # built-in + user
│   ├── .archive/
│   └── .usage.json
├── plugins/                   # user-installed
├── profiles/<name>/           # multi-instance
├── cron/
│   ├── .tick.lock
│   └── output/<job_id>/
└── memory/<provider>/         # provider-specific data
```

---

## PARTE 10 — Meta (auto-manutenção)

### 10.1 Curator (skill lifecycle)

- **Localização:** `agent/curator.py` (review loop, auto-transitions, LLM review prompt) + `agent/curator_backup.py` (pre-run tar.gz snapshots).
- **CLI verbs:** `status`, `run`, `pause`, `resume`, `pin`, `unpin`, `archive`, `restore`, `prune`, `backup`, `rollback`.
- **Telemetry sidecar:** `~/.hermes/skills/.usage.json` com `use_count`, `view_count`, `patch_count`, `last_activity_at`, `state` (active/stale/archived), `pinned`.
- **Archives:** vão pra `~/.hermes/skills/.archive/`, restoráveis.

### 10.2 Skin/theme engine

- **`hermes_cli/skin_engine.py`** — data-driven CLI theming.
- Carregado de `display.skin` no `config.yaml` no startup.
- Customiza: banner colors, spinner faces/verbs/wings, tool prefix, response box, branding text.

### 10.3 Permission/contribution model

- **Core é "narrow waist"** — novos core tools têm bar alta (enviados em TODO API call).
- Preferência para capability nova (em ordem):
  1. Estender código existente
  2. CLI command + skill
  3. Service-gated tool (`check_fn`)
  4. Plugin
  5. MCP server no catálogo
  6. **Novo core tool (último recurso)**
- Footprint Ladder: ABC + orchestrator pattern para plugins da mesma categoria.

---

## 📚 Inventário de arquivos principais (referência)

| Categoria | Arquivos |
|---|---|
| Core loop | `run_agent.py`, `model_tools.py`, `toolsets.py`, `batch_runner.py` |
| CLI | `cli.py`, `hermes_cli/main.py`, `hermes_cli/commands.py`, `hermes_cli/skin_engine.py` |
| TUI | `ui-tui/src/{entry,app}.tsx`, `tui_gateway/server.py` |
| Desktop | `apps/desktop/`, `apps/shared/` |
| Gateway | `gateway/run.py`, `gateway/session.py`, `gateway/platforms/base.py`, `gateway/builtin_hooks/` |
| Tools | `tools/registry.py`, `tools/{terminal,file,web,browser,delegation,memory,...}.py` |
| Toolsets | `toolsets.py`, `tools/environments/{local,docker,ssh,modal,daytona,singularity}/` |
| Memory | `agent/memory_provider.py`, `agent/memory_manager.py`, `plugins/memory/<provider>/` |
| Context engine | `agent/context_engine.py`, `plugins/context_engine/<provider>/` |
| Models | `providers/__init__.py`, `providers/base.py`, `plugins/model-providers/<provider>/` |
| Sessions | `hermes_state.py` (SQLite + FTS5) |
| Cron | `cron/jobs.py`, `cron/scheduler.py`, `cron/scripts/` |
| Kanban | `plugins/kanban/{dashboard,systemd}/`, `tools/kanban_tools.py`, `hermes_cli/kanban.py` |
| Skills | `skills/<cat>/<skill>/`, `optional-skills/<cat>/<skill>/`, `tools/skills_hub.py`, `agent/skill_commands.py` |
| Plugins | `hermes_cli/plugins.py`, `plugins/<name>/`, `plugin_utils.py` |
| MCP | `packages/mcp/` |
| ACP | `acp_adapter/` |
| Web | `web/`, `plugins/web/` |
| Observability | `plugins/observability/` |
| Logging | `hermes_logging.py` |
| Config | `~/.hermes/config.yaml`, `~/.hermes/.env` |
| Profile | `_apply_profile_override()` em `hermes_cli/main.py`, `_get_profiles_root()` |
| Curator | `agent/curator.py`, `agent/curator_backup.py`, `hermes_cli/curator.py`, `tools/skill_usage.py` |
| Documentation | `website/docs/`, `AGENTS.md`, `~/.hermes/SOUL.md`, `~/.hermes/config.yaml` |

---

## 🔢 Métricas de tamanho (referência para comparação)

| Componente | LOC aproximado |
|---|---|
| `run_agent.py` (AIAgent class) | ~12.000 |
| `cli.py` (HermesCLI) | ~11.000 |
| AGENTS.md (dev guide) | 1.435 |
| Skills built-in | ~42 skills |
| Skills optional | 14 categorias |
| Toolsets | 31 |
| Model providers built-in | 7+ (openrouter, anthropic, gmi, deepseek, nvidia, ...) |
| Memory providers built-in | 8 (honcho, mem0, supermemory, byterover, hindsight, holographic, openviking, retaindb) |
| Platforms gateway | ~20 |
| Plugins directory | 20+ subdiretórios |
| Tests | ~17.000 tests em ~900 files (mai/2026) |
| Cron jobs hard interrupt | 3 min |
| Kanban dispatcher tick | 60s default |
| Delegation max concurrency | 3 default |
| Delegation max spawn depth | 2 default |
| Kanban failure_limit | 2 default |

---

**Próximo passo:** usar este inventário como base de comparação com a arquitetura do `agente-universo`.
