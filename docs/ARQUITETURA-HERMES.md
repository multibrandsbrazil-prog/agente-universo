# Arquitetura do Hermes Agent — Estrutura Completa por Partes

> **Data:** 2026-08-31
> **Fonte:** `~/.hermes/hermes-agent/AGENTS.md` (oficial, 1.435 linhas) + árvore de código real em `~/.hermes/hermes-agent/` (134 arquivos em `agent/`, 111 em `tools/`, 54 em `gateway/`, 28+ subcomandos CLI).
> **Objetivo:** Inventário EXAUSTIVO da arquitetura do Hermes, separado em 19 partes para posterior comparação com o `agente-universo`.
> **Escopo:** Apenas descrever. Sem análise, sem opinião, sem implicações.

---

## 📐 Camadas (de fora pra dentro)

```
┌──────────────────────────────────────────────────────────────┐
│ PARTE 1 — ENTRY POINTS (como o usuário alcança o agente)     │
│   CLI │ TUI │ Desktop │ ACP │ Web │ Gateway                  │
├──────────────────────────────────────────────────────────────┤
│ PARTE 2 — DISTRIBUTION (como mensagens viram turnos)          │
│   Slash command registry │ Message guards │ Delivery          │
├──────────────────────────────────────────────────────────────┤
│ PARTE 3 — AGENT LOOP (o coração síncrono)                    │
│   AIAgent │ Conversation loop │ Budget │ Interrupts          │
├──────────────────────────────────────────────────────────────┤
│ PARTE 4 — CAPABILITIES (o que o loop chama)                  │
│   Tools │ Toolsets │ Environments │ Skills │ Subagentes      │
├──────────────────────────────────────────────────────────────┤
│ PARTE 5 — CONTEXT LIFECYCLE (como o prompt é montado/limpo)   │
│   System prompt │ Coding context │ Compression │ References │
├──────────────────────────────────────────────────────────────┤
│ PARTE 6 — CONTEXT & MEMORY (o que o loop lê)                 │
│   Memory providers │ Context engines │ SessionDB │ Caching   │
├──────────────────────────────────────────────────────────────┤
│ PARTE 7 — INFERENCE (como chamadas saem pra LLM)              │
│   Transports │ Model providers │ Credentials │ Budget        │
├──────────────────────────────────────────────────────────────┤
│ PARTE 8 — VOICE & DESKTOP (surface multimodais)               │
│   TTS streaming │ Wake words │ Computer use                  │
├──────────────────────────────────────────────────────────────┤
│ PARTE 9 — CODE & IDE INTEGRATION                              │
│   LSP │ Codex │ Copilot ACP │ Checkpoints                    │
├──────────────────────────────────────────────────────────────┤
│ PARTE 10 — EXTENSIBILITY (como terceiros plugam)              │
│   Plugin ABCs │ Hooks │ Skill registry │ MCP │ Optional MCPs │
├──────────────────────────────────────────────────────────────┤
│ PARTE 11 — SECRETS & SECURITY                                 │
│   Secret sources (Bitwarden/1Password/command) │ Approvals    │
├──────────────────────────────────────────────────────────────┤
│ PARTE 12 — OBSERVABILITY & MONITORING                         │
│   OTLP exporter │ Health │ Events │ Redaction                 │
├──────────────────────────────────────────────────────────────┤
│ PARTE 13 — BACKGROUND WORK                                    │
│   Cron │ Kanban │ Delegation │ Terminal bg                   │
├──────────────────────────────────────────────────────────────┤
│ PARTE 14 — GATEWAY INFRASTRUCTURE                             │
│   Session lifecycle │ Turn lease │ Wake │ Scale-to-zero      │
├──────────────────────────────────────────────────────────────┤
│ PARTE 15 — RELAY & CONNECTOR                                  │
│   WebSocket transport │ Command manifest │ Media plane        │
├──────────────────────────────────────────────────────────────┤
│ PARTE 16 — PERSISTENCE & ISOLATION (onde tudo mora)           │
│   HERMES_HOME │ Profiles │ Config vs .env │ Logging           │
├──────────────────────────────────────────────────────────────┤
│ PARTE 17 — DISTRIBUTION & PACKAGING                           │
│   Docker │ Nix │ Locales │ Bootstrap installer               │
├──────────────────────────────────────────────────────────────┤
│ PARTE 18 — META (auto-manutenção)                            │
│   Curator │ Skin engine │ Pet/mascot │ Footprint Ladder       │
├──────────────────────────────────────────────────────────────┤
│ PARTE 19 — TESTS, DOCS & DOCS-SITE                            │
│   Pytest suite │ Docusaurus i18n │ AGENTS.md │ SOUL.md        │
└──────────────────────────────────────────────────────────────┘
```

---

## PARTE 1 — Entry Points (Surfaces)

| Surface | Arquivo principal | Engine | Como ativa |
|---|---|---|---|
| **CLI** | `cli.py` (~11k LOC) | Rich + prompt_toolkit | `hermes` |
| **TUI** | `ui-tui/src/` (Ink/React) ↔ `tui_gateway/` (Python) | Node stdio ↔ Python JSON-RPC | `hermes --tui` ou `HERMES_TUI=1` |
| **Desktop** | `apps/desktop/` + `apps/shared/` | Electron | App standalone |
| **ACP adapter** | `acp_adapter/` + `agent/copilot_acp_client.py` | VS Code / Zed / JetBrains / GitHub Copilot | Plugin IDE |
| **Web dashboard** | `web/` + `hermes_cli/web_routers/` + `plugins/web/` | SPA + auth (`plugins/dashboard_auth/`, `hermes_cli/dashboard_auth/`) | Browser |
| **Gateway** | `gateway/run.py` | Multi-plataforma mensageria | `hermes gateway start` |

Todas surfaces consomem o **mesmo core `AIAgent`** (`run_agent.py`). Cada surface pega um toolset base de `toolsets.py`.

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

### 2.4 Slash commands adicionais

- `gateway/slash_commands.py` — registry gateway-side.
- `gateway/slash_access.py` — controle de acesso por platform.
- `gateway/stream_consumer.py`, `stream_dispatch.py`, `stream_events.py` — streaming de respostas.

---

## PARTE 3 — Agent Loop (núcleo)

### 3.1 Classe `AIAgent`

- **Arquivo:** `run_agent.py` (~12k LOC).
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

### 3.4 Conversation loop (extraído)

- **Localização:** `agent/conversation_loop.py` — extracted de `run_agent.AIAgent` (declarado no docstring).
- Mantém o loop canônico separado da classe pra testabilidade.

### 3.5 Formato de mensagens

OpenAI schema: `{"role": "system/user/assistant/tool", ...}`. Reasoning fica em `assistant_msg["reasoning"]`.

### 3.6 File dependency chain

```
tools/registry.py         (no deps — importado por todos)
       ↑
tools/*.py                (cada um chama registry.register() no import)
       ↑
model_tools.py            (importa registry + dispara discover_builtin_tools())
       ↑
run_agent.py, cli.py, batch_runner.py, environments/
```

### 3.7 Interrupts

- `tools/interrupt.py` — interrupt handling infrastructure.
- `agent/checkpoint_manager.py` + `tools/checkpoint_manager.py` — checkpoints pra recovery após interrupt.

---

## PARTE 4 — Capabilities (Tools, Toolsets, Skills, Subagentes)

### 4.1 Tool registry

- **Localização:** `tools/registry.py`.
- **Auto-discovery:** cada `tools/<name>.py` chama `registry.register(...)` no import; `model_tools.py` chama `discover_builtin_tools()`.

### 4.2 Toolsets (bundles por surface)

- **Localização:** `toolsets.py` — dicionário `TOOLSETS`.
- **Padrão:** `_HERMES_CORE_TOOLS` é o bundle default.
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

#### 4.4.2 Categorias em `skills/` (built-in, 12)

`apple`, `autonomous-ai-agents`, `creative`, `email`, `github`, `media`, `mlops`, `note-taking`, `productivity`, `research`, `smart-home`, `social-media`, `software-development`.

#### 4.4.3 Categorias em `optional-skills/` (21)

`autonomous-ai-agents`, `blockchain`, `communication`, `creative`, `data-science`, `devops`, `dogfood`, `email`, `finance`, `gaming`, `health`, `mcp`, `migration`, `mlops`, `payments`, `productivity`, `research`, `security`, `software-development`, `web-development`, `yuanbao`.

#### 4.4.4 SKILL.md frontmatter (campos)

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

#### 4.4.5 Estrutura de diretórios por skill

```
skills/<category>/<skill>/
├── SKILL.md                  # corpo da skill
├── scripts/                  # helpers
├── references/               # docs longas
├── templates/                # arquivos modelo
└── tests/skills/test_*.py    # pytest, só stdlib + unittest.mock
```

#### 4.4.6 8 Padrões HARDLINE (rejeitam PR se violar)

1. `description` ≤ 60 chars, sem marketing.
2. Tools referenciadas no prose = tools nativas Hermes em backticks (NUNCA `grep`/`cat`/`sed`/`find`/`ls`).
3. `platforms:` auditado contra imports reais (POSIX-only → gate).
4. `author` credita humano primeiro.
5. Body: `# <Skill> Skill` → intro 2-3 frases → `## When to Use` / `## Prerequisites` / `## How to Run` / `## Quick Reference` / `## Procedure` / `## Pitfalls` / `## Verification`.
6. Scripts em `scripts/`, refs em `references/`, templates em `templates/`.
7. Tests em `tests/skills/test_<skill>_skill.py`, sem rede.
8. `.env.example` additions isoladas em bloco delimitado.

#### 4.4.7 Skill slash commands

`agent/skill_commands.py` scanneia `~/.hermes/skills/` e injeta skills como **user message** (não system prompt) — preserva prompt caching.

#### 4.4.8 Skills hub

- `tools/skills_hub.py` (`OptionalSkillSource`) — gerencia install de optional-skills.
- `tools/blueprints.py` — skill templates/blueprints.

### 4.5 Delegation (subagentes)

- **Tool:** `tools/delegate_tool.py`.
- **Live log:** `tools/delegation_live_log.py`.
- **Async queue:** `tools/async_delegation.py` — fila de completion de background delegations.
- **Delegation context:** `agent/delegation_context.py` — contexto passado pro subagent.
- **Shapes:**
  - `goal` (+ `context`, `toolsets`) — single.
  - `tasks: [...]` — batch paralelo, concurrency cap por `delegation.max_concurrent_children` (default **3**).
- **Roles:**
  - `leaf` (default) — não chama `delegate_task`, `clarify`, `memory`, `send_message`, `cronjob`. Retém `execute_code`.
  - `orchestrator` — retém `delegate_task`. Gated por `delegation.orchestrator_enabled` (default true), bounded por `delegation.max_spawn_depth` (default **2**).
- **Config knobs:** `max_concurrent_children`, `max_spawn_depth`, `child_timeout_seconds`, `orchestrator_enabled`, `subagent_auto_approve`, `inherit_mcp_toolsets`, `max_iterations`.
- **Durability:** background `delegate_task` é process-local. Pra sobreviver restart, usar `cronjob` ou `terminal(background=True, notify_on_complete=True)`.

### 4.6 Managed tool gateway

- `tools/managed_tool_gateway.py` — "generic managed-tool gateway helpers for Nous-hosted vendor passthroughs".
- Camada de proxy/restrição pra tools hospedados por terceiros.

---

## PARTE 5 — Context Lifecycle

### 5.1 System prompt

- Fixo por toda a conversação (cache-friendly).
- Composto por: MEMORY (do provider) + USER profile + tools + skills injetadas como user message.
- **Invariante:** byte-stable pelo tempo de vida da conversa. Única exceção: context compression.

### 5.2 Coding context

- `agent/coding_context.py` — "base Hermes, every interactive surface".
- Awareness de contexto pra code-related work (lê arquivos, edita código).

### 5.3 Context breakdown

- `agent/context_breakdown.py` — live session context-window breakdown pra UI surfaces.
- Funções: `_chars_to_tokens`, `_json_tokens`, `_tool_name`, `_split_tools`.
- Mostra quanto tokens cada parte do prompt está ocupando em tempo real.

### 5.4 Context compressor

- `agent/context_compressor.py` — automatic context window compression pra conversations longas.
- Função core: `_safe_int`.
- É a **única exceção permitida** à invariante de prompt caching (junto com PARTE 5.5).

### 5.5 Conversation compression

- `agent/conversation_compression.py` — extract dos métodos do AIAgent que dirigem summarisation.
- Docstring: "Context compression — extract the AIAgent methods that drive summarisation."

### 5.6 Manual compression feedback

- `agent/manual_compression_feedback.py` — user-facing summaries pra manual compression commands.
- Funções: `describe_compression_lock_skip`, `summarize_manual_compression`.
- Quando user dispara `/compress`, gera summary human-readable.

### 5.7 Context references

- `agent/context_references.py` — classe `ContextReference`.
- Tracking de referências no contexto (URLs, files, etc.) pra citation/recall.

### 5.8 Turn finalizer (micro-compaction)

- `agent/turn_finalizer.py` — post-turn micro-compaction.
- Lógica: após cada turno, roda micro-compaction pra absorver oldest uncompacted messages sem invalidar cache.
- Log: "Micro-compaction: %d -> %d messages".
- Defrag rewrites dos newest MICRO compacted rows.

### 5.9 Turn context

- `agent/turn_context.py` / `gateway/turn_context.py` — per-turn context compartilhado entre GatewayRunner e AIAgent.

---

## PARTE 6 — Context & Memory (providers + engines)

### 6.1 Memory providers (plugins/memory/)

- **Padrão:** ABC `MemoryProvider` (em `agent/memory_provider.py`) + orchestrator `agent/memory_manager.py`.
- **Built-in providers (jun/2026, 8):** `honcho`, `mem0`, `supermemory`, `byterover`, `hindsight`, `holographic`, **`openviking`**, `retaindb`.
- **Lifecycle hooks:**
  - `sync_turn(turn_messages)` — após cada turno.
  - `prefetch(query)` — background antes do LLM.
  - `post_setup(hermes_home, config)` — wizard de setup.
  - `shutdown()` — cleanup.
- **Policy:** `plugins/memory/` é **closed set**. Novos providers DEVEM ser standalone plugin repo em `~/.hermes/plugins/` ou pip entry point.

### 6.2 Context engines (plugins/context_engine/)

- Plugado em `agent/context_engine.py`.
- Mesmo padrão ABC + orchestrator + per-plugin directory.
- Exemplos: OpenViking, LUCID, lossless-claw.

### 6.3 Session store

- **Localização:** `hermes_state.py` — `SessionDB` (SQLite + FTS5 full-text search).
- Cada mensagem vira row; suporta `session_search()` (full-text recall).

### 6.4 FTS5 CJK native

- `native/fts5_cjk/` — extensão SQLite nativa com tokenização CJK (chinês/japonês/coreano).
- Build: `build.sh`, vendor libs.
- Lê README para detalhes.

### 6.5 Prompt caching invariants (sacred)

- **NÃO** alterar contexto mid-conversation.
- **NÃO** trocar toolsets mid-conversation.
- **NÃO** reload memories ou rebuild system prompts mid-conversation.
- Única exceção: context compression (PARTE 5).
- Commands que mutam system-prompt state são **cache-aware** (default deferred invalidation, opt-in `--now`).

---

## PARTE 7 — Inference

### 7.1 Transports (camada de saída das chamadas LLM)

**Localização:** `agent/transports/`. Provider response normalization.

| Transport | API target |
|---|---|
| `chat_completions.py` | OpenAI Chat Completions |
| `anthropic.py` | Anthropic Messages API |
| `codex.py` | OpenAI Responses API (Codex) |
| `codex_app_server.py` | Codex app-server JSON-RPC client |
| `codex_app_server_session.py` | Session adapter pra codex app-server |
| `codex_event_projector.py` | Projeta codex events na lista de messages |
| `bedrock.py` | AWS Bedrock Converse API |
| `hermes_tools_mcp_server.py` | Hermes-tools-as-MCP server pro codex app-server |
| `types.py` | Shared types pra normalized responses |
| `base.py` | Abstract base class |

### 7.2 Model-provider adapters (legacy)

- `providers/base.py`, `providers/__init__.py` (legacy antes do split).
- **Built-in:** openrouter, anthropic, gmi, deepseek, nvidia.
- **Adapters específicos:** `agent/anthropic_adapter.py`, `agent/codex_responses_adapter.py`, `agent/codex_runtime.py`, `agent/bedrock_adapter.py`, `agent/azure_identity_adapter.py`, `agent/backend_identity.py`.

### 7.3 Model-provider plugins (plugins/model-providers/)

- Cada `__init__.py` chama `providers.register_provider(ProviderProfile(...))` no load.
- **Discovery é lazy e separado:**
  1. Bundled: `<repo>/plugins/model-providers/<name>/`
  2. User: `$HERMES_HOME/plugins/model-providers/<name>/`
  3. Legacy: `<repo>/providers/<name>.py`
- **User overrides bundled** (last-writer-wins).

### 7.4 API modes

`api_mode` aceita: `"chat_completions"` | `"codex_responses"` | ...

### 7.5 Credential pool

- `agent/credential_pool.py` + `agent/credential_sources.py` + `agent/credential_persistence.py` — gerencia rotação de API keys, fallback model.
- `agent/proxy_sources/iron_proxy.py` — iron-proxy integration pra credential-injecting egress control.
- `tools/credential_files.py` — file-based credential storage.

### 7.6 Credits tracking

- `agent/credits_tracker.py` — `CreditsState` rastreia spend em USD, depleted fraction, age.
- `agent/aux_accounting.py` + `agent/auxiliary_client.py` — accounting de chamadas auxiliary.
- `agent/billing_usage.py` + `agent/billing_view.py` + `agent/billing_links.py` — billing views.
- `agent/account_usage.py` — account-level usage.
- `agent/battery.py` — battery-related (se aplicável).
- `tools/budget_config.py` — config de budget.

---

## PARTE 8 — Voice & Desktop (surfaces multimodais)

### 8.1 TTS streaming

- `gateway/streaming_tts_consumer.py` — consumer de streaming TTS pro gateway.
- `docs/streaming-tts.md` — design doc.
- TTS é parte do toolset `tts`.

### 8.2 Wake words

- **Engine:** [openWakeWord](https://github.com/dscripka/openWakeWord) (Apache-2.0).
- **Models bundled:** `tools/wakewords/hey_hermes.onnx`, `hey_hermes.tflite`.
- **Label:** `hey_hermes` (matches filename).
- **Default phrase:** "Hey Hermes".
- **Configurable:** treinar próprio modelo e setar `wake_word.openwakeword.model` no config, ou usar built-in (`hey_jarvis`, `alexa`, `hey_mycroft`).
- Shared feature-extraction models (melspectrogram + embedding) NÃO bundled — fetched on first use.
- Docs user-facing: `website/docs/user-guide/features/wake-word.md`.

### 8.3 Audio container

- `tools/audio_container.py` — packaging de audio pra voice channels.

### 8.4 Computer use (desktop automation)

- **Localização:** `tools/computer_use/`.
- **Tool entry:** `tools/computer_use/tool.py` — tool entry point.
- **Schema:** `tools/computer_use/schema.py` — um tool consolidado com `action` discriminator (compacto, baixo token cost).
- **Backends:**
  - `tools/computer_use/backend.py` — Abstract `UIElement`.
  - `tools/computer_use/cua_backend.py` — Cua-driver backend (macOS, Windows, Linux).
  - `tools/computer_use/browser_route.py` — Session-scoped typed-browser routing.
  - `tools/computer_use/doctor.py` — health check.
  - `tools/computer_use/permissions.py` — permissions layer.
  - `tools/computer_use/vision_routing.py` — Vision-routing decisions pra capture results.
- **Tool wrapper:** `tools/computer_use_tool.py`.

### 8.5 Pet/mascot

- **Localização:** `agent/pet/`.
- **Modules:**
  - `agent/pet/store.py` — on-disk pet store (install/list/resolve).
  - `agent/pet/manifest.py` — manifest format.
  - `agent/pet/render.py` — rendering do pet.
  - `agent/pet/state.py` — state machine.
  - `agent/pet/generate/` — generation pipeline.
  - `agent/pet/constants.py` — constants.

---

## PARTE 9 — Code & IDE Integration

### 9.1 LSP (Language Server Protocol)

- **Localização:** `agent/lsp/` (10 arquivos).
- **Entry:** `agent/lsp/__init__.py` declara "Language Server Protocol (LSP) integration for Hermes Agent".
- **Components:**
  - `client.py` — async LSP client over stdin/stdout.
  - `protocol.py` — minimal LSP JSON-RPC 2.0 framer over async streams.
  - `manager.py` — service-level orchestration.
  - `servers.py` — server registry (per-language).
  - `install.py` — auto-installation of LSP server binaries.
  - `workspace.py` — workspace/project-root resolution.
  - `range_shift.py` — diff-aware line-shift map pra cross-edit LSP delta filtering.
  - `reporter.py` — format LSP diagnostics pra inclusion in tool output (severity-1 only by default — warnings/info/hints would flood).
  - `eventlog.py` — structured logging with steady-state silence.
  - `cli.py` — `hermes lsp` subcommand.

### 9.2 Codex integration

- `agent/codex_runtime.py` + `agent/codex_responses_adapter.py` — Codex runtime + adapter.
- `agent/copilot_acp_client.py` — Copilot ACP client.
- Subcommands: `hermes_cli/subcommands/acp.py`.

### 9.3 Checkpoints

- `tools/checkpoint_manager.py` + `agent/checkpoint_*` — feature de recovery.
- AIAgent.__init__ aceita `checkpoints_config`.
- Permite restaurar conversation state após interrupt/failure.

### 9.4 Background review

- `agent/background_review.py` — background review process (qualidade das respostas async).

---

## PARTE 10 — Extensibility

### 10.1 Plugins gerais (`hermes_cli/plugins.py` + `plugins/<name>/`)

- **Discovery:** `~/.hermes/plugins/`, `./.hermes/plugins/`, pip entry points.
- **Cada plugin expõe `register(ctx)`** que pode:
  - Hooks: `pre_tool_call`, `post_tool_call`, `pre_llm_call`, `post_llm_call`, `on_session_start`, `on_session_end`.
  - Tools: `ctx.register_tool(...)`.
  - CLI subcommands: `ctx.register_cli_command(...)`.
- **Pitfall:** `discover_plugins()` só roda como side-effect de importar `model_tools.py`.

### 10.2 Plugins diretórios (built-in)

Em `plugins/`: `browser`, `context_engine`, `cron_providers`, `dashboard_auth`, `disk-cleanup`, `google_meet`, `hermes-achievements`, `image_gen`, `kanban`, `memory`, `model-providers`, `observability`, `platforms`, `security-guidance`, `spotify`, `teams_pipeline`, `video_gen`, `web`.

### 10.3 Memory plugins

Ver PARTE 6.1.

### 10.4 Model-provider plugins

Ver PARTE 7.3.

### 10.5 MCP (Model Context Protocol)

- Suporte MCP nativo em `packages/mcp/`.
- Skills podem declarar dependência em MCP server no `## Prerequisites`.
- Roda via `mcp__*` toolset quando habilitado.
- `hermes_cli/subcommands/mcp.py` — `hermes mcp <verb>`.
- `mcp-research-data/` — research data para MCP planning.

### 10.6 Optional MCPs (catálogo oficial de MCP servers pesados)

**Localização:** `optional-mcps/`. Cada um tem `manifest.yaml` com:

| MCP server | Função |
|---|---|
| `blender/` | Drive live Blender (modeling, scenes, renders) via `ahujasid/blender-mcp` (MIT, PyPI: blender-mcp 1.6.4). Bridge stdio + addon em Blender. |
| `comfy-cloud/` | ComfyUI cloud. |
| `figma/` | Official Figma remote MCP (OAuth 2.1 + DCR em `https://mcp.figma.com/mcp`). Auto-registra client_name "Claude Code" (Figma allowlists exact names). |
| `linear/` | Linear project management. |
| `n8n/` | Manage/inspect n8n workflows via `CyberSamuraiX/hermes-n8n-mcp` (stdio bridge, API key). |
| `unreal-engine/` | Unreal Engine integration. |

**Manifest fields:** `manifest_version`, `name`, `description`, `source`, `transport` (type, command/args, version, env), `install` (opcional: git/url/ref/bootstrap), `auth` (api_key/oauth/none + env vars), `tools` (default_enabled list), `post_install` (instruções human-readable).

**Pinning policy:** exact version (Pypi/git SHA), release ≥ 2 semanas old at pin time (supply-chain cooldown). Catalog nunca auto-atualiza.

**Telemetry policy:** upstream telemetry desabilitada por default em todos os entries (no outbound telemetry sem opt-in explícito).

### 10.7 Third-party product policy (jun/2026)

Plugins que integram produto de terceiro (observability backends, vendor SaaS, dashboards) DEVEM ser standalone plugin repos instalados em `~/.hermes/plugins/`. **Não** entram no tree.

---

## PARTE 11 — Secrets & Security

### 11.1 Secret sources (plugins/secret_sources/)

- **Localização:** `agent/secret_sources/`.
- **Padrão:** ABC `base.py` — contract que todo backend implementa.
- **Registry:** `agent/secret_sources/registry.py` + `apply` orchestrator.
- **Cache:** `agent/secret_sources/_cache.py` — shared substrate.
- **Backends built-in:**
  - `bitwarden.py` — Bitwarden Secrets Manager (`bws` CLI).
  - `onepassword.py` — 1Password (`op` CLI).
  - `command.py` — `command` secret source (resolve via user-configured helper).

### 11.2 Approvals (humans in the loop)

- `tools/approval.py` — "Dangerous command approval — detection, prompting, and per-session state."
- `gateway/clarify_gateway.py` + `tools/clarify_tool.py` — clarify prompts entre agent e user.
- `hermes_cli/subcommands/approvals.py` — CLI: `hermes approvals`.

### 11.3 Authorization

- `gateway/authz_mixin.py` — "User-authorization methods for `GatewayRunner`."
- `gateway/pairing.py` — pairing flow (primeiro contato).
- `hermes_cli/subcommands/pairing.py` — CLI.

### 11.4 Security plugins & guidance

- `plugins/security-guidance/` — security guidance plugin.
- `hermes_cli/subcommands/security.py` — CLI security verbs.

### 11.5 Per-field caps em output

- `agent/lsp/reporter.py` documenta: "Per-field caps for diagnostic content sourced from the language server. These bound the length of any single attacker-controlled identifier that..." — bounded lengths em campos onde attacker control é possível.

---

## PARTE 12 — Observability & Monitoring

### 12.1 Monitoring core

- **Localização:** `agent/monitoring/`.
- **Entry:** `agent/monitoring/__init__.py` — "Hermes gateway monitoring".
- **Components:**
  - `events.py` — typed gateway monitoring events (classe `GatewayHealthEvent`).
  - `emitter.py` — fire-and-forget queue + background dispatcher.
  - `gateway_health.py` — gateway health e diagnostics signal producer.
  - `gateway_health_export.py` — gateway Health & Diagnostics OTLP export runtime.
  - `otlp_exporter.py` — export monitoring events to OpenTelemetry Collector over OTLP/HTTP.
  - `cron_health.py` — content-free cron service-health e execution telemetry projection.
  - `policy.py` — install identity (função `ensure_install_id`).
  - `redaction.py` — redaction aplicada a monitoring data antes de egress (PII redaction).

### 12.2 Observability plugin

- `plugins/observability/` — metrics / traces / logs plugin (third-party backend).
- `hermes_cli/observability/` — observability CLI integrations.
- `scripts/observability/` — observability scripts.
- `docs/observability/` — observability design docs.

### 12.3 Dashboard & health

- `hermes_cli/subcommands/monitoring.py` — `hermes monitoring` CLI.
- `hermes_cli/subcommands/insights.py` — insights view.
- `hermes_cli/subcommands/status.py` — `hermes status`.
- `hermes_cli/subcommands/dashboard.py` — dashboard CLI.

### 12.4 Logging

- `hermes_logging.py` — `setup_logging()` cria `agent.log`, `errors.log`, `gateway.log`.
- `hermes_cli/subcommands/logs.py` — `hermes logs [--follow] [--level ...] [--session ...]`.

---

## PARTE 13 — Background Work

### 13.1 Cron (scheduled jobs)

- **Arquivos:** `cron/jobs.py` (store) + `cron/scheduler.py` (tick loop) + `cron/scripts/` (job scripts) + `cron/suggestions.py`.
- **CLI:** `hermes_cli/subcommands/cron.py`.
- **Schedules suportados:**
  - Duration: `"30m"`, `"2h"`, `"1d"`
  - "every": `"every 2h"`, `"every monday 9am"`
  - 5-field cron: `"0 9 * * *"`
  - ISO timestamp one-shot: `"2026-06-01T09:00:00Z"`
- **Per-job fields:** `skills`, `model`/`provider` overrides, `script` (pre-run data-collection; `no_agent=True` → script É o job), `context_from` (chain output), `workdir` (carrega AGENTS.md/CLAUDE.md), multi-platform delivery.
- **Hardening:** 3-min hard interrupt, catchup window (½ período clamped 120s–2h), grace window 120s one-shot, file lock `~/.hermes/cron/.tick.lock`, `skip_memory=True` em cron sessions.
- **Delivery:** cron NÃO espelha pra gateway session — fica em cron session própria com header/footer.
- **Contracts:** `docs/chronos-managed-cron-contract.md`.

### 13.2 Kanban (multi-agent work queue)

- SQLite-backed board compartilhado entre múltiplos profiles/workers.
- **CLI verbs:** `hermes_cli/subcommands/kanban.py` (ou equivalente).
- **Toolset (`tools/kanban_tools.py`):** `kanban_show`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, `kanban_comment`, `kanban_create`, `kanban_link`, `kanban_attach`, `kanban_attach_url`, `kanban_attachments`; profiles com toolset `kanban` fora de dispatcher-spawned task também ganham `kanban_list`, `kanban_unblock`.
- **Dispatcher:** loop long-lived (default 60s tick) — reclama stale claims, promove ready, atomic claim, spawna assigned profiles. Roda **inside gateway** por default (`kanban.dispatch_in_gateway: true`).
- **Plugin assets:** `plugins/kanban/dashboard/` (web UI) + `plugins/kanban/systemd/` (`hermes-kanban-dispatcher.service`).
- **Isolation:** Board = hard boundary (`HERMES_KANBAN_BOARD` pinned no env dos workers). Tenant = soft namespace within board. `kanban.failure_limit` (default **2**) tentativas não-successful → auto-block.
- **Watcher integration:** `gateway/kanban_watchers.py`.
- **Docs:** `docs/kanban/`.

### 13.3 Delegation

Ver PARTE 4.5.

### 13.4 Terminal background

- `terminal(background=True, notify_on_complete=True)` — gateway watcher detecta completion e dispara novo agent turn.
- Verbosity: `all`/`result`/`error`/`off`.
- Hook spill: `tools/hook_output_spill.py`.

---

## PARTE 14 — Gateway Infrastructure

### 14.1 Runner

- `gateway/run.py` — runner principal, JSON-RPC style.
- `gateway/session.py` — session lifecycle top-level.

### 14.2 Session lifecycle

- `gateway/session_context.py` — session context.
- `gateway/session_state.py` — per-session gateway state consolidated into one container.
- `gateway/turn_lease.py` — per-session turn lease, serializes o region `[load history → run → flush]`.
- `gateway/turn_context.py` — per-turn context shared entre GatewayRunner._run_agent_inner e AIAgent.
- `gateway/mirror.py` — session mirroring (cross-instance?).
- `gateway/lifecycle_ledger.py` — lifecycle event ledger.
- `gateway/runtime_footer.py` — runtime footer.
- `gateway/rich_sent_store.py` — rich sent-store.
- `gateway/message_timestamps.py` — message timestamps.

### 14.3 Wake & scale-to-zero

- `gateway/wake.py` — wake an existing agent session from a background completion event.
- `gateway/scale_to_zero.py` — scale-to-zero idle detection + dormant-quiesce (Phase 0).
- `gateway/systemd_notify.py` — systemd notify integration.
- `gateway/readiness.py` — readiness signal.

### 14.4 Restart & watchdog

- `gateway/restart.py` — shared gateway restart constants e supervisor detection helpers.
  - EX_TEMPFAIL de sysexits.h — usado pra pedir pro service manager restart o gateway após graceful drain/reload.
- `gateway/restart_loop_guard.py` — restart loop guard.
- `gateway/shutdown_watchdog.py` — out-of-loop shutdown e event-loop liveness backstops (issues #66892, #69089).
- `gateway/shutdown_flush.py` — graceful shutdown flush.
- `gateway/shutdown_forensics.py` — shutdown forensics.
- `gateway/cgroup_cleanup.py` — cgroup cleanup.

### 14.5 Message routing

- `gateway/stream_consumer.py`, `stream_dispatch.py`, `stream_events.py` — streaming de respostas.
- `gateway/turn_context.py` — turn-level context.
- `gateway/drain_control.py` — drain control.
- `gateway/dead_targets.py` — dead target tracking.
- `gateway/code_skew.py` — code skew detection.
- `gateway/cwd_placeholder.py` — cwd placeholder substitution.
- `gateway/status.py` + `gateway/status_phrases.py` — status e status phrases.
- `gateway/channel_directory.py` — channel directory.

### 14.6 Platforms (adapters)

**Localização:** `gateway/platforms/`. Cada um herda `base.py` (helpers compartilhados).

| Platform | Arquivo |
|---|---|
| api_server | `api_server.py` |
| bluebubbles | `bluebubbles.py` |
| qqbot | `qqbot/` (subdir) |
| signal | `signal.py` + `signal_format.py` + `signal_rate_limit.py` |
| webhook | `webhook.py` + `webhook_filters.py` + `msgraph_webhook.py` |
| weixin | `weixin.py` |
| whatsapp | `whatsapp_cloud.py` + `whatsapp_common.py` |
| yuanbao | `yuanbao.py` + `yuanbao_media.py` + `yuanbao_proto.py` + `yuanbao_sticker.py` |
| shared helpers | `helpers.py`, `media_cache.py`, `_http_client_limits.py` |

### 14.7 Profile routing

- `gateway/profile_routing.py` — gateway-side profile routing logic.

### 14.8 Streaming TTS

- `gateway/streaming_tts_consumer.py` — consumer pra streaming TTS no gateway.

### 14.9 Hooks extension point

- `gateway/builtin_hooks/` — extension point pra always-registered gateway hooks (zero shipped hoje).
- `hermes_cli/subcommands/hooks.py` — CLI pro hooks management.

### 14.10 Memory monitor & stickers

- `gateway/memory_monitor.py` — memory monitoring.
- `gateway/sticker_cache.py` — sticker cache (Yuanbao-specific).

---

## PARTE 15 — Relay & Connector

### 15.1 Visão

Sistema de **relay/connector** que permite o gateway Hermes conectar a outras instâncias via WebSocket ou outro transport, com autenticação e media plane.

### 15.2 Componentes

**Localização:** `gateway/relay/`. Marcado como **EXPERIMENTAL** em vários arquivos.

| Componente | Função |
|---|---|
| `adapter.py` | `RelayAdapter` — generic gateway adapter fronted by the connector. |
| `auth.py` | Gateway-side relay authentication primitives. |
| `transport.py` | Relay transport protocol — gateway↔connector wire contract. |
| `ws_transport.py` | Production WebSocket RelayTransport — gateway's live link to the connector. |
| `media.py` | Relay media client — gateway↔connector media plane (Phase 2). |
| `command_manifest.py` | Gateway-declared slash-command manifest pra relay lane (Phase 4). |
| `descriptor.py` | `CapabilityDescriptor` — relay handshake payload. |

---

## PARTE 16 — Persistence & Isolation

### 16.1 HERMES_HOME & perfis

- **`HERMES_HOME`** — env var que define o diretório raiz.
- **Profile mechanism:** `_apply_profile_override()` em `hermes_cli/main.py` seta `HERMES_HOME` antes de qualquer import.
- **`get_hermes_home()`** (código) vs **`display_hermes_home()`** (user-facing messages).
- **Profile operations são HOME-anchored**, não HERMES_HOME-anchored — `_get_profiles_root()` retorna `Path.home() / ".hermes" / "profiles"`.

### 16.2 Config vs .env

- `~/.hermes/config.yaml` — **TODAS** as settings comportamentais.
- `~/.hermes/.env` — **APENAS secrets**.
- **Policy:** rejectar PRs que dizem "set X in .env" a menos que X seja credencial.

### 16.3 Logging

- `~/.hermes/logs/agent.log` (INFO+).
- `~/.hermes/logs/errors.log` (WARNING+).
- `~/.hermes/logs/gateway.log` (quando gateway rodando).

### 16.4 User home layout

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

### 16.5 Import/Export

- `hermes_cli/subcommands/import_agent.py`, `import_cmd.py` — import agent configs.
- `hermes_cli/subcommands/sync.py` — sync configs.
- `hermes_cli/subcommands/backup.py` — backup configs/skills.

---

## PARTE 17 — Distribution & Packaging

### 17.1 Docker

- **Localização:** `docker/`.
- **Layout:**
  - `SOUL.md` — embedded doc.
  - `entrypoint.sh`, `entrypoint-dispatch.sh`, `main-wrapper.sh`, `hermes-exec-shim.sh`, `stage2-hook.sh`, `tini-shim.sh`.
  - `cont-init.d/` — s6 init scripts.
  - `s6-rc.d/` — s6 service tree.

### 17.2 Nix

- **Localização:** `nix/`.
- **Files:** `checks.nix`, `configMergeScript.nix`, `desktop.nix`, `devShell.nix`, `hermes-agent.nix`, `lib.nix`, `nixosModules.nix`, `overlays.nix`, `packages.nix`, `python.nix`, `tui.nix`, `web.nix`.
- **Pinned node packages:** `node-gyp-11-4-0-package-lock.json`, `node-gyp-11-4-0.nix`, `npm-12-0-2.nix`.

### 17.3 Locales (i18n)

- **Localização:** `locales/` — 18 YAML files.
- **Idiomas:** `af`, `ar`, `de`, `en`, `es`, `fr`, `ga`, `hu`, `it`, `ja`, `ko`, `pt`, `ru`, `tr`, `uk`, `zh-hant`, `zh` (+ mais).

### 17.4 Bootstrap installer

- `apps/bootstrap-installer/` — installer standalone pra first-time setup.

### 17.5 Uninstall & update

- `hermes_cli/subcommands/uninstall.py` — `hermes uninstall`.
- `hermes_cli/subcommands/update.py` — `hermes update`.

---

## PARTE 18 — Meta (auto-manutenção)

### 18.1 Curator (skill lifecycle)

- **Localização:** `agent/curator.py` + `agent/curator_backup.py` + `tools/skill_usage.py` + `hermes_cli/curator.py` + `hermes_cli/subcommands/skills.py`.
- **CLI verbs:** `status`, `run`, `pause`, `resume`, `pin`, `unpin`, `archive`, `restore`, `prune`, `backup`, `rollback`.
- **Telemetry sidecar:** `~/.hermes/skills/.usage.json` com `use_count`, `view_count`, `patch_count`, `last_activity_at`, `state` (active/stale/archived), `pinned`.
- **Archives:** vão pra `~/.hermes/skills/.archive/`, restoráveis.

### 18.2 Skin/theme engine

- **`hermes_cli/skin_engine.py`** — data-driven CLI theming.
- Carregado de `display.skin` no `config.yaml` no startup.
- Customiza: banner colors, spinner faces/verbs/wings, tool prefix, response box, branding text.
- CLI: `hermes_cli/subcommands/skin.py`.

### 18.3 Pet/mascot

Ver PARTE 8.5.

### 18.4 Footprint Ladder

**Regras de contribution (extraídas do AGENTS.md):**
- **Core é "narrow waist"** — novos core tools têm bar alta (enviados em TODO API call).
- Preferência para capability nova (em ordem):
  1. Estender código existente
  2. CLI command + skill
  3. Service-gated tool (`check_fn`)
  4. Plugin
  5. MCP server no catálogo
  6. **Novo core tool (último recurso)**
- ABC + orchestrator pattern pra plugins da mesma categoria.

### 18.5 Achievements plugin

- `plugins/hermes-achievements/` — gamified achievement tracking.

### 18.6 Hermes-doctor

- `hermes_cli/subcommands/doctor.py` — `hermes doctor` health check.
- `tools/env_probe.py` — environment probing.
- `tools/env_passthrough.py` — env var passthrough management.
- `tools/lazy_deps.py` — lazy dependency loading.

---

## PARTE 19 — Tests, Docs & Docs-Site

### 19.1 Test suite

- **Localização:** `tests/` (~17.000 tests em ~900 files).
- **Subdirs:** `acp`, `acp_adapter`, `agent`, `ci`, `cli`, `computer_use`, `conformance`, `cron`, `dashboard`, `docker`, `e2e`, `fakes`, `fixtures`, `gateway`, `hermes_cli`, `hermes_state`, `honcho_plugin`, `integration`, `manual`, `monitoring`, `openviking_plugin`, `plugins`, `providers`, `run_agent`, `scripts`, `secret_sources`, `skills`, `state`, `stress`, `tools`, `tui_gateway`, `website`.
- **TestJS:** `tests-js/` — JS-side tests (pro TUI/web).
- **Scripts:** `scripts/run_tests.sh`, `scripts/run_tests_parallel.py`.

### 19.2 Conformance & E2E

- `tests/conformance/` — conformance test vectors.
- `tests/e2e/` — end-to-end tests.

### 19.3 Docs site (Docusaurus)

- **Localização:** `website/`.
- **Components:**
  - `website/docs/` — English docs (markdown).
  - `website/i18n/zh-Hans/docusaurus-plugin-content-docs/current/` — Chinese (Simplified) docs.
  - `website/src/`, `website/static/`, `website/scripts/` — Docusaurus theme + build.

### 19.4 Internal docs

- `docs/` (no repo root) — design docs:
  - `billing-lifecycle.md`
  - `chronos-managed-cron-contract.md`
  - `micro-compaction.md`
  - `profile-routing.md`
  - `relay-connector-contract.md`
  - `rca-ssl-cacert-post-git-pull.md`
  - `session-lifecycle.md`
  - `streaming-tts.md`
- Subdiretórios: `docs/design/`, `docs/kanban/`, `docs/middleware/`, `docs/observability/`, `docs/plans/`, `docs/security/`.
- `hermes-kanban-v1-spec.pdf` — kanban v1 spec PDF.

### 19.5 SOUL.md & AGENTS.md

- `~/.hermes/SOUL.md` — embedded soul doc.
- `~/.hermes/hermes-agent/AGENTS.md` — official dev guide (1.435 linhas).
- `~/.hermes/hermes-agent/Docker/SOUL.md` — Docker-embedded soul.

### 19.6 Contributor docs

- `contributors/` + `contributors/emails/` — contributor data.

### 19.7 Datagen config examples

- `datagen-config-examples/` — example configs pra data generation pipelines.

---

## 📚 Inventário de arquivos principais (referência)

| Categoria | Arquivos |
|---|---|
| **Core loop** | `run_agent.py`, `model_tools.py`, `toolsets.py`, `batch_runner.py` |
| **CLI core** | `cli.py`, `hermes_cli/main.py`, `hermes_cli/commands.py`, `hermes_cli/skin_engine.py` |
| **CLI subcommands** | `hermes_cli/subcommands/{acp,approvals,auth,backup,claw,config,console,cron,dashboard,debug,doctor,dump,gateway,gui,hooks,import_agent,import_cmd,insights,login,logout,logs,mcp,memory,model,monitoring,pairing,plugins,profile,prompt_size,security,setup,skills,skin,slack,status,sync,tools,uninstall,update,version,webhook,whatsapp}.py` |
| **TUI** | `ui-tui/src/{entry,app}.tsx`, `tui_gateway/server.py` |
| **Desktop** | `apps/desktop/`, `apps/shared/` |
| **Gateway runner** | `gateway/run.py`, `gateway/session.py`, `gateway/platforms/base.py`, `gateway/builtin_hooks/` |
| **Gateway infrastructure** | `gateway/{session_context,session_state,turn_lease,turn_context,wake,scale_to_zero,restart,shutdown_watchdog,kanban_watchers,streaming_tts_consumer,kanban_watchers,profile_routing,mirror,lifecycle_ledger,stream_consumer,stream_dispatch,stream_events,drain_control,dead_targets,code_skew,cwd_placeholder,status,status_phrases,channel_directory,memory_monitor,sticker_cache}.py` |
| **Gateway platforms** | `gateway/platforms/{api_server,bluebubbles,signal,signal_format,signal_rate_limit,webhook,webhook_filters,msgraph_webhook,weixin,whatsapp_cloud,whatsapp_common,yuanbao,yuanbao_media,yuanbao_proto,yuanbao_sticker,qqbot/__init__}.py` |
| **Gateway relay** | `gateway/relay/{adapter,auth,command_manifest,descriptor,media,transport,ws_transport}.py` |
| **Tools** | `tools/registry.py`, `tools/{terminal,file,web,browser,delegation,memory,approval,checkpoint,clarify,code_execution,interrupt,kanban,async_delegation,delegation_live_log,managed_tool_gateway,focus_pane,close_terminal,desktop_ui,computer_use_tool,image_generation_tool,flux3_video_tool,audio_container,browser_tool,browser_camofox,browser_cdp_tool,browser_dialog_tool,browser_supervisor,blueprints,credential_files,budget_config,hook_output_spill,image_source,daemon_pool,debug_helpers,env_probe,env_passthrough,lazy_deps,file_state,file_tools,file_operations,fuzzy_match,ansi_strip,binary_extensions,fal_common,blueprints}.py` |
| **Toolsets** | `toolsets.py`, `tools/environments/{local,docker,ssh,modal,daytona,singularity}/` |
| **Computer use** | `tools/computer_use/{tool,backend,cua_backend,doctor,permissions,schema,vision_routing,browser_route,__init__}.py` |
| **Wakewords** | `tools/wakewords/{hey_hermes.onnx,hey_hermes.tflite,README.md}` |
| **Agent internals** | `agent/{context_compressor,context_breakdown,context_references,context_engine,conversation_compression,conversation_loop,coding_context,manual_compression_feedback,turn_context,turn_finalizer,delegation_context,credential_pool,credential_sources,credential_persistence,browser_provider,browser_registry,aux_accounting,auxiliary_client,codex_runtime,codex_responses_adapter,copilot_acp_client,anthropic_adapter,azure_identity_adapter,bedrock_adapter,backend_identity,account_usage,billing_usage,billing_view,billing_links,battery,bounded_response,background_review,credits_tracker,display,error_classifier,memory_provider,memory_manager,curator,curator_backup,delegation_context,async_utils,agent_init,agent_runtime_helpers,chat_completion_helpers,codex_app_server_session,codex_event_projector,secret_sources/base,secret_sources/bitwarden,secret_sources/onepassword,secret_sources/command,secret_sources/registry,secret_sources/_cache,proxy_sources/iron_proxy,transports/base,transports/anthropic,transports/chat_completions,transports/codex,transports/codex_app_server,transports/codex_app_server_session,transports/codex_event_projector,transports/bedrock,transports/hermes_tools_mcp_server,transports/types,lsp/{__init__,cli,client,eventlog,install,manager,protocol,range_shift,reporter,servers,workspace}.py,monitoring/{__init__,cron_health,emitter,events,gateway_health,gateway_health_export,otlp_exporter,policy,redaction}.py,pet/{store,manifest,render,state,constants,generate/*}.py` |
| **Memory** | `agent/memory_provider.py`, `agent/memory_manager.py`, `plugins/memory/{honcho,mem0,supermemory,byterover,hindsight,holographic,openviking,retaindb}/` |
| **Context engine** | `agent/context_engine.py`, `plugins/context_engine/<provider>/` |
| **Models** | `providers/__init__.py`, `providers/base.py`, `plugins/model-providers/<provider>/` |
| **Sessions** | `hermes_state.py` (SQLite + FTS5) |
| **Cron** | `cron/jobs.py`, `cron/scheduler.py`, `cron/scripts/`, `cron/suggestions.py` |
| **Kanban** | `plugins/kanban/{dashboard,systemd}/`, `tools/kanban_tools.py`, `hermes_cli/kanban.py` |
| **Skills** | `skills/{apple,autonomous-ai-agents,creative,email,github,media,mlops,note-taking,productivity,research,smart-home,social-media,software-development}/`, `optional-skills/{21 categories}/`, `tools/skills_hub.py`, `agent/skill_commands.py` |
| **Plugins** | `hermes_cli/plugins.py`, `plugins/{browser,context_engine,cron_providers,dashboard_auth,disk-cleanup,google_meet,hermes-achievements,image_gen,kanban,memory,model-providers,observability,platforms,security-guidance,spotify,teams_pipeline,video_gen,web}/`, `plugin_utils.py` |
| **MCP** | `packages/mcp/`, `mcp-research-data/`, `optional-mcps/{blender,comfy-cloud,figma,linear,n8n,unreal-engine}/` |
| **ACP** | `acp_adapter/` |
| **Web** | `web/`, `plugins/web/`, `hermes_cli/web_routers/`, `hermes_cli/dashboard_auth/`, `plugins/dashboard_auth/` |
| **Observability** | `plugins/observability/`, `hermes_cli/observability/`, `scripts/observability/`, `agent/monitoring/` |
| **Logging** | `hermes_logging.py` |
| **Config** | `~/.hermes/config.yaml`, `~/.hermes/.env` |
| **Profile** | `_apply_profile_override()` em `hermes_cli/main.py`, `_get_profiles_root()` |
| **Curator** | `agent/curator.py`, `agent/curator_backup.py`, `hermes_cli/curator.py`, `tools/skill_usage.py` |
| **Distribution** | `docker/`, `nix/`, `locales/`, `apps/bootstrap-installer/` |
| **Documentation** | `website/docs/`, `website/i18n/`, `AGENTS.md`, `SOUL.md`, `docs/{billing-lifecycle,chronos-managed-cron-contract,micro-compaction,profile-routing,relay-connector-contract,rca-ssl-cacert-post-git-pull,session-lifecycle,streaming-tts}.md` |
| **Tests** | `tests/` (~17k tests em ~900 files), `tests-js/` |

---

## 🔢 Métricas de tamanho (referência para comparação)

| Componente | Métrica |
|---|---|
| `run_agent.py` (AIAgent class) | ~12.000 LOC |
| `cli.py` (HermesCLI) | ~11.000 LOC |
| AGENTS.md (dev guide) | 1.435 linhas |
| Skills built-in | 13 categorias (apple, autonomous-ai-agents, creative, email, github, media, mlops, note-taking, productivity, research, smart-home, social-media, software-development) |
| Skills optional | 21 categorias |
| Toolsets | 31 |
| Model providers built-in | openrouter, anthropic, gmi, deepseek, nvidia, + adapters |
| Transports | 9 (chat_completions, anthropic, codex, codex_app_server, codex_app_server_session, codex_event_projector, bedrock, hermes_tools_mcp_server, base) |
| Memory providers built-in | 8 (honcho, mem0, supermemory, byterover, hindsight, holographic, openviking, retaindb) |
| Context engine providers | multiple plugins |
| Secret sources built-in | 3 (Bitwarden, 1Password, command) |
| Platforms gateway | ~15+ (api_server, bluebubbles, qqbot, signal, webhook, weixin, whatsapp, yuanbao, + Telegram/Discord/Slack/Matrix/Mattermost/etc via plugins/platforms/) |
| Plugins directory | 18 subdiretórios |
| Optional MCPs | 6 (blender, comfy-cloud, figma, linear, n8n, unreal-engine) |
| CLI subcommands | 43+ |
| Environments (terminal backends) | 6 (local, docker, ssh, modal, daytona, singularity) |
| Locales | 18 (af, ar, de, en, es, fr, ga, hu, it, ja, ko, pt, ru, tr, uk, zh-hant, zh, + mais) |
| Tests | ~17.000 tests em ~900 files |
| Agent/ dir | 134 arquivos |
| Tools/ dir | 111 arquivos |
| Gateway/ dir | 54 arquivos |
| Cron jobs hard interrupt | 3 min |
| Kanban dispatcher tick | 60s default |
| Delegation max concurrency | 3 default |
| Delegation max spawn depth | 2 default |
| Kanban failure_limit | 2 default |
| FTS5 | SQLite full-text search (com CJK native extension) |

---

**Próximo passo:** usar este inventário como base de comparação com a arquitetura do `agente-universo`.
