# PESQUISA — Capacidades do Hermes Agent + Ideias de Melhorias

> **Data:** 2026-08-31
> **Fontes:** Oficiais (AGENTS.md 75KB, SOUL.md, config.yaml) + Curated (awesome-hermes-agent, 0xNyk) + Locais (`~/.hermes/`)
> **Objetivo:** Mapear **o que o Hermes faz hoje** + **onde dá pra ser melhor** no nosso agente novo.

---

## 📏 TL;DR

O Hermes Agent (Nous Research, ⭐239k) é um **agente open-source auto-melhorável** que roda em **CLI + TUI + Web Dashboard + Desktop (Electron) +20 plataformas de mensageria**. Capacidades core: **terminal real + browser + file system + tools modulares + memória persistente + skills procedurais + plugins + cron + subagentes + kanban multi-agent + self-evolution (DSPy/GEPA)**. Força única dele: **curador de skills** que cria/melhora skills automaticamente baseado em uso.

**Onde nosso agente pode ser MELHOR** (oportunidades reais):
1. **Catálogo agregado de skills/plugins** — Hermes tem ~42 skills; OpenClaw tem 5.300+; Claude Code tem ~50 marketplace; **nenhum agrega tudo**. Nosso pode ser o primeiro.
2. **Multi-LLM plugável** — Hermes suporta vários via plugins; DSH similar. **Vantagem nova**: roteamento inteligente automático (modelo barato p/ tarefas simples, top p/ difíceis).
3. **Self-evolution público** — Hermes tem DSPy/GEPA interno. **Vantagem nova**: evolução compartilhada — skills que a comunidade usa ficam públicas e melhoram com uso real.
4. **Multi-UI simultâneo** — Hermes tem CLI+Telegram+Desktop+Web separados. **Vantagem nova**: sessão única compartilhada entre UIs (começa no Telegram, continua no terminal).
5. **Sandbox padrão ON** — Hermes sandbox é opcional (precisa config). **Vantagem nova**: sandbox por padrão, opt-out (mais seguro por default).
6. **Aprovação explícita por tier** — Hermes tem approval para destructive, mas é "tudo ou nada". **Vantagem nova**: 3 tiers (read/write/delete) com allowlist por skill.
7. **Observability built-in** — Prometheus/OTel/Langfuse nativo, não como plugin.
8. **BYOK transparente** — Multi-provider via plug, mas sem rate-limit handling. **Vantagem nova**: failover automático entre providers quando rate-limited.

---

## 🧠 O QUE O HERMES FAZ HOJE (capacidades verificadas)

### 1. **Core Architecture**

| Aspecto | Implementação Hermes |
|---|---|
| Linguagem | Python (~12k LOC em `run_agent.py`) |
| Distribution | MIT (open-source), mas Nous Research comercializa |
| Stars | **239k** |
| Versão | v0.18.2 (v2026.7.7.2) |
| Loop principal | `AIAgent` em `run_agent.py`, síncrono, ~500 max |
| Message format | OpenAI chat completions (`{"role": "system/user/assistant/tool"}`) |
| Prompt caching | **Sagrado** — não quebra mid-conversation (exceto compress) |

### 2. **Surfaces (onde você usa o agente)**

| Surface | Como acessar | Stack |
|---|---|---|
| **CLI** | `hermes` no terminal | prompt_toolkit + Rich + KawaiiSpinner |
| **TUI** | `hermes --tui` | Ink (React) + Python JSON-RPC backend |
| **Web Dashboard** | `hermes dashboard` | Docusaurus + embedded TUI |
| **Desktop (Electron)** | `hermes-desktop` | Electron + @assistant-ui/react |
| **Messaging Gateway** | Telegram, Discord, Slack, WhatsApp, Signal, iMessage, Email, SMS, **+15 outras** | gateway/platforms/ |
| **ACP Server** | VS Code, Zed, JetBrains | `acp_adapter/` |
| **API Server** | `hermes serve` | JSON-RPC over WebSocket |
| **Cron daemon** | scheduler | jobs.py + scheduler.py |

### 3. **Tools nativos** (o agente tem DE FATO)

| Toolset | Tools dentro |
|---|---|
| `terminal` | `terminal`, `process` (manage background), `execute_code` |
| `file` | `read_file`, `write_file`, `patch`, `search_files` |
| `browser` | `browser_navigate`, `browser_click`, `browser_extract`, screenshot |
| `web` | `web_search`, `web_extract`, `vision_analyze` |
| `delegation` | `delegate_task` (single + batch parallel) |
| `cronjob` | Agendar jobs (`30m`, `every 2h`, `0 9 * * *`, ISO) |
| `kanban` | `kanban_show`, `kanban_complete`, `kanban_block`, etc |
| `todo` | List de tarefas do agent loop |
| `memory` | Read/write no memory provider |
| `session_search` | FTS5 search no SQLite de sessions |
| `clarify` | `clarify` tool (pergunta ao user sem travar) |
| `tts` | Text-to-speech provider plugável |
| `vision` | Análise de imagem |
| `image_gen` | Geração de imagem |
| `video` | Video processing |
| `messaging` | Mandar mensagem em plataforma |
| `discord`/`discord_admin` | Específicos Discord |
| `homeassistant` | Smart home |
| `spotify` | Spotify control |
| `skills` | Scan e load de skills |
| `safe` | Approval gates |
| `rl` | Reinforcement learning experiment |
| `moa` | Mixture of agents |
| `code_execution` | Executa código em sandbox |
| `debugging` | Debug helpers |
| `search` | Code search |
| `feishu_doc`/`feishu_drive` | Feishu integration |
| `yuanbao` | Yuanbao groups |

**Total: 30+ toolsets**, ~80 tools individuais.

### 4. **Memory providers** (8 built-in)

- `honcho` — peer memory (social/relationship context)
- `mem0` — facts extraction
- `supermemory` — long-term consolidation
- `byterover` — bytes-level recall
- `hindsight` — recall com context temporal
- `holographic` — distributed memory
- `openviking` — self-evolving context DB
- `retaindb` — retention-based

**Regra (policy May 2026):** novos providers NÃO entram in-tree — devem ser standalone plugins.

### 5. **Skills system** (procedural memory)

- **42 skills instaladas localmente** (`~/.hermes/skills/*`)
- Formato **padrão aberto** (agentskills.io) — compatível com Claude Code, Cursor, Codex
- Categorias locais: `autonomous-ai-agents`, `browser-components`, `cloudflare-quick-tunnel`, `cognitive`, `creative`, `data-science`, `email`, `finance`, `frontend`, `github`, `github-search`, `gps-location-lookup`, `importaai-arquitetura-juridica`, `mcp-comex-brasil-stack`, `media`, `mlops`, `note-taking`, `osint-pessoa-fisica-br`, `pesquisa-chinagate-id`, `pesquisa-internet`, `productivity`, `programming-platforms`, `research`, `scroll-driven-hero`, `security`, `simple-technical-explanations`, `skill-findability`, `smart-home`, `social-media`, `software-development`, `third-party-skill-validation`, `yuanbao`, mais 10+

**SKILL.md standard (HARDLINE):**
- `description` ≤ 60 chars, uma frase, com ponto final
- Body: `# Title` + intro + `## When to Use` + `## Prerequisites` + `## How to Run` + `## Quick Reference` + `## Procedure` + `## Pitfalls` + `## Verification`
- Target: ~200 linhas (complexo), ~100 (simples)
- Scripts em `scripts/`, refs em `references/`, templates em `templates/`
- Tests em `tests/skills/test_<skill>_skill.py`

### 6. **Plugins** (extensão modular)

| Tipo | Localização | Exemplos |
|---|---|---|
| **Gerais** | `~/.hermes/plugins/<name>/` | hooks (pre/post tool/LLM), new tools, CLI subcommands |
| **Memory providers** | `plugins/memory/` | honcho, mem0, supermemory... (closed set) |
| **Model providers** | `plugins/model-providers/` | openrouter, anthropic, gmi, deepseek, nvidia |
| **Context engines** | `plugins/context_engine/` | Estratégia de compressão de contexto |
| **Image-gen providers** | `plugins/image_gen/` | Stable Diffusion, FLUX, DALL-E |
| **Kanban dashboard** | `plugins/kanban/` | UI web do multi-agent board |

**Regra (Teknium, May 2026):** plugins NÃO modificam core files. Se precisam de hook, framework expõe; não hardcode plugin-specific.

### 7. **Self-evolution (único!)**

**Hermes é o ÚNICO agente open-source com closed learning loop público:**
- **[hermes-agent-self-evolution](https://github.com/NousResearch/hermes-agent-self-evolution)** — DSPy + GEPA (Genetic Evolution of Prompt Architectures)
- **[hermes-skill-factory](https://github.com/Romanescu11/hermes-skill-factory)** — meta-skill que auto-gera skills de workflows repetidos
- **[hermes-dojo](https://github.com/Yonkoo11/hermes-dojo)** — monitora performance, identifica skills fracas, itera automaticamente
- **[super-hermes](https://github.com/Cranot/super-hermes)** — meta-reasoning, agente gera melhores prompts pra si mesmo
- **Curator** — auto-archive skills stale, LLM review periódico

### 8. **Delegation / Multi-agent**

| Capacidade | Detalhe |
|---|---|
| `delegate_task` single | Spawn subagent com context isolado |
| `delegate_task` batch | Paralelo, até 3 concorrentes |
| Roles | `leaf` (não delega) / `orchestrator` (delega, max depth=2) |
| Config knobs | max_concurrent_children, child_timeout, subagent_auto_approve |
| Kanban | Board SQLite-backed, dispatcher 60s tick, isolamento por board+tenant |

### 9. **Messaging platforms (20+)**

Telegram, Discord, Slack, WhatsApp, Signal, iMessage, Email, SMS, HomeAssistant, Matrix, Mattermost, DingTalk, WeCom, WeiXin, Feishu, QQBot, BlueBubbles, Yuanbao, Webhook, API Server, Google Chat.

### 10. **Configuração**

```yaml
# ~/.hermes/config.yaml (real, atual)
model:
  default: minimax/MiniMax-M3
  provider: minimax
  max_tokens: 16384
agent:
  max_turns: 100
  reasoning_effort: high
  personalities: 14 (helpful, concise, technical, creative, teacher,
                     kawaii, catgirl, pirate, shakespeare, surfer,
                     noir, uwu, philosopher, hype)
terminal:
  backend: local
  container_cpu: 1, memory: 5120MB, disk: 51200MB
  docker_mount_cwd_to_workspace: false
```

**Personalities: 14内置** (kawaii, pirate, shakespeare, noir, etc).

---

## 📚 FONTES CONSULTADAS

| # | Fonte | Tipo | URL |
|---|---|---|---|
| 1 | `NousResearch/hermes-agent` AGENTS.md (75KB) | ✅ oficial | github.com |
| 2 | `~/.hermes/hermes-agent/AGENTS.md` | ✅ local | nosso filesystem |
| 3 | `~/.hermes/SOUL.md` | ✅ local | identidade do agente |
| 4 | `~/.hermes/config.yaml` | ✅ local | config real em uso |
| 5 | `0xNyk/awesome-hermes-agent` | ⚠️ comunidade | github.com |
| 6 | `~/.hermes/skills/` | ✅ local | 42 skills reais |
| 7 | Repos oficiais via `gh search` | ✅ oficial | hermes-agent-self-evolution, hermes-desktop, hermes-workspace |
| 8 | `lintsinghua/claude-code-book` | ⚠️ comunidade | análise 42万字 do Claude Code |
| 9 | `claude-code-system-prompts` | ⚠️ comunidade | system prompts leaked |

---

## 🎯 ONDE PODEMOS SER MELHORES (20 ideias concretas)

### 🔥 Categoria A — Catálogo agregado (única no mercado)

| # | Melhoria | Por que seria melhor que Hermes | Esforço |
|---|---|---|---|
| **A1** | **Importar TODAS as skills**: Hermes42 + OpenClaw5300+ + Claude Code ~50 + DSH ~300 | Hermes tem 42 skills próprias. **Nenhum agregador existe** no mercado que una. | Médio (2-3 dias) |
| **A2** | **Bridge universal SKILL.md → todos os formatos** (Hermes/DSH/Claude/Codex/OpenClaw) | Cada agente só lê formato próprio. Bridge = "instala1 skill, funciona em todos" | Médio |
| **A3** | **Skill marketplace federado** com rating, comments, audit, VirusTotal | OpenClaw tem ClawHub; Hermes tem skills hub. Mas **rating+audit+security review unificado** não tem | Alto |
| **A4** | **One-line install** de qualquer skill de qualquer fonte (`dsh plugin add <url>` já existe; unificar) | Hoje precisa saber de cada fonte separado. **Nosso: 1 comando, qualquer skill** | Baixo |

### 🚀 Categoria B — Multi-LLM inteligente

| # | Melhoria | Por que melhor | Esforço |
|---|---|---|---|
| **B1** | **Roteamento automático de modelo por tarefa** (tarefa simples=cheap, complexa=top) | Hermes permite mas user escolhe. **Nosso: escolhe sozinho** | Alto |
| **B2** | **Failover automático entre providers** quando rate-limited (DeepSeek→OpenAI→Anthropic fallback) | Hermes falha se provider cair. **Nosso: graceful degrade** | Médio |
| **B3** | **BYOK transparente + OAuth subscriptions** (igual `dsh-plugin-subscriptions` mas first-class) | Usar ChatGPT/Claude/Grok sem API key via OAuth — `vyhuhl/dsh-plugin-subscriptions` ⭐305 já existe, integrar | Baixo |
| **B4** | **Benchmark empírico por skill** — qual modelo melhor em qual tarefa | Nenhum agente faz. **Nosso: mede e recomenda** | Alto |

### 🧠 Categoria C — Self-evolution pública

| # | Melhoria | Por que melhor | Esforço |
|---|---|---|---|
| **C1** | **Skills que melhoram com uso real compartilhado** (vs Hermes que evolui por instância) | Cada Hermes evolui sozinho. **Nosso: evolução coletiva da rede** | Muito alto |
| **C2** | **DSPy/GEPA integration first-class** (não como plugin opcional) | Hermes tem como plugin. **Nosso: built-in** | Médio |
| **C3** | **Skill-factory open** — usuário descreve workflow repetido, agente cria skill | `hermes-skill-factory` já existe mas é experimental. **Nosso: production-ready** | Alto |
| **C4** | **Auto-archive de skills com heurística transparente** (vs Hermes curator LLM review) | Hermes: LLM decide o que arquivar ($$$). **Nosso: heurística + opt-in LLM** | Médio |

### 🛡️ Categoria D — Segurança por padrão

| # | Melhoria | Por que melhor | Esforço |
|---|---|---|---|
| **D1** | **Sandbox ON por padrão** (Hermes é OFF + opt-in) | Menos "agent went rogue" stories | Baixo |
| **D2** | **3-tier approval** (read/write/delete) com allowlist por skill | Hermes: binário (allow ou deny tudo). **Nosso: granular** | Médio |
| **D3** | **Secret scanner built-in** (anti-leak em prompts) | Nenhum agente tem. **Nosso: detecta + bloqueia** | Alto |
| **D4** | **Prompt-injection defense** (input filtering) | Crítico em mundo conectado. Skills hoje podem ser vetor | Alto |

### 🎨 Categoria E — UX

| # | Melhoria | Por que melhor | Esforço |
|---|---|---|---|
| **E1** | **Sessão única compartilhada entre UIs** (começa Telegram, continua terminal) | Hermes: cada plataforma = sessão separada. **Nosso: cross-session unified** | Alto |
| **E2** | **Built-in observability** (Prometheus/OTel/Langfuse) — não plugin | Hermes: sem observability nativa. **Nosso: dashboard built-in** | Médio |
| **E3** | **Voice I/O** (TTS + STT com wake-word) | Hermes tem TTS, STT só via skill. **Nosso: native voice** | Médio |
| **E4** | **Mobile-first responsive** (controlar pelo celular com mesma UX do desktop) | Hermes web não é mobile-friendly. **Nosso: PWA instalável** | Médio |

---

## 📊 Comparação final: Hermes vs Nosso Agente (target)

| Capacidade | Hermes | OpenClaw | DSH | **Nosso (target)** |
|---|---|---|---|---|
| Skills próprias | 42 | 5.300+ | ~300 | **5.000+ (agregadas)** |
| Plugins | ~30 (in-tree) | ClawHub registry | ~300 | **100+ curados** |
| Multi-LLM | ✅ 8+ providers | ✅ 25+ providers | ✅ multi | ✅ **+ roteamento automático** |
| Multi-UI | ✅ 8 surfaces | ✅ 7 channels | ✅ web+CLI+SDK | ✅ **+ sessão unificada** |
| Memory | ✅ 8 providers | ⚠️ básico | ✅ session log | ✅ **+ roteamento por contexto** |
| Self-evolution | ✅ DSPy/GEPA | ❌ | ❌ | ✅ **+ coletiva da rede** |
| Sandbox | ⚠️ opt-in | ⚠️ opt-in | ⚠️ opt-in | ✅ **ON por padrão** |
| Approval gates | ⚠️ binário | ⚠️ básico | ⚠️ básico | ✅ **3-tier granular** |
| Observability | ❌ plugin | ❌ plugin | ❌ | ✅ **built-in** |
| Mobile | ❌ | ❌ | ⚠️ | ✅ **PWA** |
| Voice I/O | ⚠️ parcial | ⚠️ parcial | ❌ | ✅ **built-in** |
| Open source | ✅ MIT | ✅ MIT | ✅ MIT | ✅ **MIT** |
| Idade | 2+ anos | ~1 ano | meses | 🆕 **novo, agregando melhor dos 3** |

---

## ✅ Próximo passo

**Me confirma 3 escolhas** e eu fecho o plano final:

1. **Quais categorias de melhoria priorizar?** (A=catálogo / B=multi-LLM / C=self-evolution / D=segurança / E=UX)

2. **Quanto tempo/dedicação você quer por fase?** (Rápido 1 semana / Médio 1 mês / Longo 3-6 meses)

3. **Você vai publicar como seu projeto** no GitHub ou **como projeto nosso** (Renan + Hermes Agent)?

Responde as3 e eu:
- Atualizo `PLANEJAMENTO-AGENTE-DSH.md` com escopo fechado
- Listo os **comandos exatos** de instalação por fase
- Defino **validação empírica** (benchmark vs Hermes em 10 tarefas idênticas)
- **AINDA NÃO INSTALO NADA** — você aprova cada fase