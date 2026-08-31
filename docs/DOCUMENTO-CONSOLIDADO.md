# Agente Universo — Documento Consolidado

> **Data:** 2026-08-31
> **Status:** Planejamento completo. Zero instalação executada.
> **Fonte:** Consolidação de todos os 12 documentos do projeto (`README.md`, `PROMPT-HANDOFF.md`, 9 arquivos em `docs/`, + Documento 12 anexo).
> **Tamanho total:** ~204KB / ~4.400 linhas.

---

## 📑 Sumário

| # | Documento original | Conteúdo | Tamanho |
|---|---|---|---|
| 1 | README.md | Entrada do projeto, stack, roadmap, fontes | 5KB |
| 2 | PROMPT-HANDOFF.md | Prompt pronto pra colar em nova sessão | 5.5KB |
| 3 | PLANO-FINAL-v2-CORRIGIDO.md | **PLANO ATUAL** (7 fases, 23 mitigações, OpenViking) | 28KB |
| 4 | ARQUITETURA-HERMES.md | Inventário do Hermes em 19 partes (referência arquitetural) | 55KB |
| 5 | PESQUISA-CONTEXT-ENGINE-PLUGIN.md | Decisão OpenViking (vs LUCID vs Mem0) | 4.4KB |
| 6 | PESQUISA-HERMES-CAPACIDADES.md | Capacidades do Hermes + 20 melhorias | 16KB |
| 7 | PESQUISA-REPOSITORIOS-AGENTE-NOVO.md | Pesquisa inicial 30+ repos (histórico) | 20KB |
| 8 | AUDITORIA-EMPIRICA-REPOS.md | Auditoria 18 repos (registro histórico) | 14KB |
| 9 | LACUNAS-FECHADAS.md | Ruflo + SKILL.md + Integrações | 14KB |
| 10 | ANALISE-PROFUNDA-LUCID.md | LUCID rejeitado (prova da decisão) | 10KB |
| 11 | PLANO-FINAL-CONSOLIDADO.md | v1 histórico (substituído por v2) | 21KB |
| **12** | **ANEXO: Validação Empírica DSH vs Hermes** (gerado 2026-08-31) | **DSH cobre 10/19 camadas 100%, 5 parcial, 4 zero** | **~5KB** |

**Ordem de leitura recomendada:** 1 → 2 → 3 → 4 → 12 (executivo + plano + arquitetura de referência + validação empírica), depois 5-11 sob demanda.

---

# DOCUMENTO 1: README.md

# Agente Universo

> Agente universal self-hosted inspirado no Hermes Agent (Nous Research).
> Status: **planejamento** — zero instalação executada.

---

## 📏 TL;DR

Construir um agente open-source que agregue **5.700+ skills** de múltiplas fontes (Hermes, OpenClaw, DSH), suporte **multi-LLM** (DeepSeek, Claude, GPT, Ollama), rode em **VPS ou PC, qualquer SO**, sob **MIT/grátis/zero lock-in**. Inspiração arquitetural vem do Hermes (session log durável, subagentes, skills procedurais, self-evolution).

**Diferencial único:** nenhum agregador de skills existe no mercado. Hermes tem 42, OpenClaw 5.300+, DSH ~300. Nosso seria o primeiro catálogo unificado.

---

## 📂 Estrutura do projeto

```
agente-universo/
├── README.md              ← você está aqui
├── .gitignore             ← arquivos ignorados (criado quando git iniciar)
├── docs/                  ← toda pesquisa + plano
│   ├── PESQUISA-HERMES-CAPACIDADES.md       (16K) — O que Hermes faz + 20 melhorias
│   ├── PESQUISA-REPOSITORIOS-AGENTE-NOVO.md (20K) — 30+ repos validados
│   ├── AUDITORIA-EMPIRICA-REPOS.md          (14K) — 18 repos auditados (3 farsas)
│   ├── LACUNAS-FECHADAS.md                  (14K) — Ruflo + SKILL.md + integrações
│   ├── PLANO-FINAL-CONSOLIDADO.md           (21K) — v1 (com 14 problemas)
│   └── PLANO-FINAL-v2-CORRIGIDO.md          (25K) — v2 ✅ PLANO ATUAL
└── scripts/               ← (vazio — scripts vão aqui quando começar Fase 1)
```

**Arquivo executivo:** [`docs/PLANO-FINAL-v2-CORRIGIDO.md`](docs/PLANO-FINAL-v2-CORRIGIDO.md) — leia este pra entender o que vai ser feito.

**Arquitetura de referência (Hermes, 19 partes):** [`docs/ARQUITETURA-HERMES.md`](docs/ARQUITETURA-HERMES.md) — inventário exaustivo da arquitetura real do Hermes Agent (não README de marketing — código real), pra usar como base de comparação durante a implementação.

---

## 🎯 Stack final (validado empiricamente)

| Camada | Componente | Validação |
|---|---|---|
| 🎯 Core | DeepSeek Harness (`dsh` v0.1.2-alpha.3) | ✅ 206k⭐, MIT |
| 🧠 Multi-LLM | DSH providers nativos | ✅ Built-in |
| 💾 Memory | `volcengine/OpenViking` + plugin DSH oficial | ✅ 34.690⭐, AGPL-3.0, plugin DSH mantido pelo próprio time |
| 🛡️ Sandbox (Linux) | DSH `native/landlock-run` | ✅ Built-in no DSH (Linux nativo) |
| �️ Sandbox (cross-OS) | `BitMiracle-AI/Dormice` | ⚠️ Opcional — só se precisar de macOS/Windows ou requisitos avançados |
| �️ Prompt defense | In-house (~200 linhas Python) | Ruflo aidefence como ref |
| 📊 Observability | `Yuntwo/dsh-langfuse-plugin` | ⚠️ Auditar antes |
| 📊 OTel | `traceloop/openllemetry` | ✅ 7.4k⭐ |
| 🧬 Self-evolution | DSPy + GEPA | ✅ Stanford + SOTA |
| 🔌 MCP BR | Suas12 MCPs locais | ✅ Funcionam via DSH bridge |
| 📚 Skills | Hermes42 + OpenClaw5.300+ + DSH~300 | Adaptador ~100 linhas |

---

## 🗺️ Roadmap (7 fases, todas requerem aprovação antes de executar)

| Fase | Tempo | Status | O que faz |
|---|---|---|---|
| 0 | 30min | ⏸️ | Verificar pré-requisitos (node, pnpm, gh) |
| 1 | 1-2h | ⏸️ | Subir DSH core + validar UI (sandbox landlock-run nativo) |
| 2 | 3-4h | ⏸️ | Skills Bridge + 42 skills Hermes |
| 3 | 1-2 dias | ⏸️ | OpenClaw skills via clawhub |
| 4 | 3-4h | �️ | MCP BR (DSH `packages/mcp` nativo) + Langfuse observability |
| 5 | 4-6h | ⏸️ | Memory OpenViking (server Python self-hosted porta 1933) — Dormice opcional |
| 6 | 2-3 dias | ⏸️ | Self-evolution DSPy+GEPA |
| 7 | 1 dia | ⏸️ | Benchmark vs Hermes + publicar |

**Total MVP:** ~7-10 dias úteis.

---

## 📚 Fontes consultadas (sem clones, só APIs)

- `gh api` — metadata, files count, contributors, releases, issues, CI workflows, LICENSE
- `npm registry` — versão publicada + repo link
- `raw.githubusercontent.com` — README + SKILL.md + package.json + AGENTS.md
- `awesome-list` curadas (VoltAgent, 0xNyk, awesome-dsh-plugin)
- Repos oficiais lidos: `~/.hermes/SOUL.md`, `~/.hermes/hermes-agent/AGENTS.md`, `~/.hermes/config.yaml`

---

## ⚠️ Princípios do projeto

1. **Zero instalação sem aprovação explícita** — toda fase tem comando + entregável + teste antes
2. **Validar empíricamente, não por stars** — 3 repos foram rejeitados (BridgeWard, e2b fragments, ink-ui stale) mesmo com stars razoáveis
3. **Curar antes de agregar** — OpenClaw awesome-list filtrou 7.215 itens, usar SÓ a curada
4. **Pinar versões** — DSH é alpha, pinar tag exata
5. **Auditar plugins 3rd-party** — plugins DSH da comunidade têm ⭐0-25, ler código antes. Exceção: plugins oficiais mantidos pelo próprio time do projeto (ex: `@openviking/openclaw-plugin`) já passam por curadoria upstream.

---

## 📞 Comandos úteis

```bash
# Navegar pro projeto
cd ~/projetos/agente-universo

# Ver plano atual
cat docs/PLANO-FINAL-v2-CORRIGIDO.md

# Quando quiser começar Fase 0:
# (nada — só verificar ambiente, sem instalar)
```

---

## 🚦 Próximo passo

**Plano finalizado.** Aguardando:
1. Aprovação do plano v2
2. Definição de onde publicar (seu GitHub / org compartilhado)
3. Decisão: você roda ou eu rodo cada fase

**Nada será instalado até você dar OK explícito na Fase 0.**
---

# DOCUMENTO 2: PROMPT-HANDOFF.md

# Prompt de Handoff — Projeto agente-universo

> Use este texto no início de uma **nova sessão** com qualquer agente AI pra retomar o projeto sem perder contexto.
>
> **Data de criação:** 2026-08-31
> **Última atualização:** 2026-08-31 (Fase 0 concluída)

---

## 📋 VERSÃO LONGA (copie/cole inteira se for a 1ª msg da nova sessão)

```
Olá! Estou continuando um projeto chamado **agente-universo**.

## O que estou tentando criar

Um agente de IA universal, open-source (MIT), self-hosted (VPS ou PC, qualquer SO),
inspirado no Hermes Agent da Nous Research. Diferenciais:

1. Agrega 5.700+ skills de múltiplas fontes (Hermes 42 + OpenClaw 5.300+ + DSH ~300)
2. Roda em qualquer LLM que o usuário quiser (DeepSeek, Claude, GPT, Ollama local)
3. Self-hosted, zero custo, zero lock-in
4. Tem 12 MCPs BR já integrados (mercado_livre, fiscal, comexstat, etc)
5. Único agregador universal de skills no mercado (ninguém mais faz isso)

## Onde está tudo

- **Projeto local:**  ~/projetos/agente-universo/
- **Repo GitHub:**  https://github.com/multibrandsbrazil-prog/agente-universo (público)
- **Branch:**       main (8 commits, último dcd30d7 + 9f5d98c)
- **Plano atual:**  ~/projetos/agente-universo/docs/PLANO-FINAL-v2-CORRIGIDO.md

## O que já foi feito

✅ Pesquisa completa do Hermes Agent (capacidades, 20 melhorias possíveis)
✅ Pesquisa de 30+ repositórios no GitHub (DSH, LiteLLM, OpenViking, Langfuse, DSPy, GEPA, Ruflo, etc)
✅ Auditoria empírica de 18 repos (3 farsas detectadas: BridgeWard, e2b fragments, ink-ui stale)
✅ Lacunas críticas resolvidas (Ruflo vs nossa ideia, compat SKILL.md, integrações DSH)
✅ Plano v1 + auditoria crítica + plano v2 corrigido (25KB, 7 fases, 23 mitigações de compat)
✅ Decisão de Memory: Mem0 → **OpenViking** (34.690⭐, AGPL-3.0, plugin DSH oficial mantido pelo próprio time — commit dcd30d7)
✅ Repositório git criado + commits + push pro GitHub
✅ Auditoria de segurança (sem credenciais vazadas, paths sensíveis substituídos por $HOME)
✅ Fase 0 do plano concluída (verificações de ambiente OK)

## O que NÃO foi feito (ainda)

❌ Fase 1: Subir DSH core (git clone + pnpm install + dsh web em :3080; sandbox landlock-run nativo)
❌ Fase 2: Skills Bridge + importar 42 skills Hermes locais
❌ Fase 3: OpenClaw skills via clawhub (50 top)
❌ Fase 4: MCP BR (DSH nativo) + Langfuse observability
❌ Fase 5: Memory OpenViking (server Python self-hosted porta 1933) — Dormice opcional
❌ Fase 6: Self-evolution DSPy+GEPA
❌ Fase 7: Benchmark vs Hermes + publicar

## Regras de ouro (do Renan, suas preferências)

1. **NÃO instalar nada sem aprovação explícita** — toda fase tem comando + entregável + teste
2. **Validar empíricamente, não por hype/stars** — auditar código antes de usar plugin 3rd-party
3. **Pinar versões** — DSH é alpha 0.1.2, pinar tag exata
4. **Curar antes de agregar** — OpenClaw awesome-list filtrou 7.215 itens, usar SÓ curada
5. **Auditar plugins antes de ativar** — plugins com <100⭐ precisam code review
6. **Não mexer em ~/importaai/** sem permissão explícita
7. **Privacy:** não expor /home/openclaw em docs públicos (usar $HOME)
8. **Nunca usar `hermes mcp run` (não existe)** — MCP servers vão no config.yaml
9. **DSH plugins usam `clawhub install <slug>` (não `git clone`)**

## Pendências de decisão (me perguntar antes de Fase 1)

- Quem vai manter o repo? (seu GitHub pessoal / org compartilhada)
- Quer rodar Fase 1 agora? (eu rodo e mostro cada comando + saída)
- Quer ler o plano v2 inteiro antes? (25KB, ~30min de leitura)

## Comandos úteis

cd ~/projetos/agente-universo         # navegar pro projeto
cat docs/PLANO-FINAL-v2-CORRIGIDO.md  # plano atual
gh repo view agente-universo          # ver repo remoto
git log --oneline                     # histórico

## Estado atual do ambiente (verificado na Fase 0)

- Node v22.22.2, pnpm v10.33.0 (upgrade pra 11.7+ na Fase 1)
- Python 3.11.15, Docker 29.4.1, Ollama instalado
- GH CLI logado em github.com/multibrandsbrazil-prog
- 65GB disco livre, sem GPU NVIDIA (usar API pra LLM)
- OS: Ubuntu 24.04.4 LTS, x86_64
```

---

## ⚡ VERSÃO CURTA (use se a sessão já é continuidade e você só quer lembrar o agente)

```
Continuando projeto agente-universo.
- Repo: github.com/multibrandsbrazil-prog/agente-universo (público, MIT, 2 commits)
- Local: ~/projetos/agente-universo/
- Plano: ~/projetos/agente-universo/docs/PLANO-FINAL-v2-CORRIGIDO.md (7 fases)
- Inspiração: Hermes Agent (Nous Research)
- Base: DeepSeek Harness (DSH) — alpha 0.1.2
- Diferencial: agrega 5.700+ skills de Hermes+OpenClaw+DSH (nenhum agregador existe)
- Status: Fase 0 ✅ (verificações). Fase 1 ⏸️ (DSH core ainda não foi clonado/instalado)
- Regra de ouro: NÃO instalar nada sem aprovação. Validar antes de commitar.
- Pendência: quem mantém o repo? seu GitHub ou org compartilhada?
```

---

## 📌 Versão "TL;DR" (1 linha pra contexto rápido)

```
Projeto agente-universo (github.com/multibrandsbrazil-prog/agente-universo): agente MIT self-hosted
inspirado no Hermes, base DSH, agregando 5.700+ skills. Fase 0 concluída, Fase 1 (subir DSH core) não
iniciada ainda. Plano: ~/projetos/agente-universo/docs/PLANO-FINAL-v2-CORRIGIDO.md
```

---

## 🔄 Quando atualizar este prompt

- Quando **concluir uma fase** do plano (atualizar "O que foi feito" e "O que falta")
- Quando **mudar o ambiente** (Node version, GPU, etc)
- Quando **trocar de conta/repo** GitHub
- Quando **descobrir uma regra nova** do Renan

Salvar como `~/projetos/agente-universo/PROMPT-HANDOFF.md` e versionar no git.
---

# DOCUMENTO 3: PLANO-FINAL-v2-CORRIGIDO.md (PLANO ATUAL)

# PLANO FINAL v2 — Agente Universal inspirado no Hermes (CORRIGIDO)

> **Data:** 2026-08-31
> **Versão:** 2.0 (auditoria crítica aplicada)
> **Inspiração:** Hermes Agent (Nous Research, ⭐239k)
> **Base:** DeepSeek Harness (DSH) — runtime "everything is a plugin"
> **Status:** PLANEJAMENTO — **zero instalação** até você aprovar Fase 0
> **Mudanças vs v1:** 7 lacunas críticas fechadas + 4 importantes mitigadas + mitigações de compatibilidade em cada fase

---

## 📏 TL;DR

Plano corrigido após auditoria crítica. **Mudanças principais:**

1. ✅ **Tag `v0.1.2-alpha.3` validada** — comando `git checkout` confirmado
2. ✅ **Comando DSH confirmado** — `pnpm dsh web` (depois de build)
3. 🔧 **Filtro `_archived_*`** adicionado em todos os `cp -r`
4. 🔧 **Adaptador normaliza body** (paths `~/.hermes/`, comandos `hermes`)
5. ❌ **MCP servers no Hermes vão via config.yaml** (`mcp_servers: {}`), NÃO CLI
6. ❌ **OpenClaw usa `clawhub install <slug>`**, NÃO `git clone + cp`
7. ❌ **Dormice instala via script bash** (`curl install.sh | bash`), NÃO Docker
8. 🔧 **Fase 6 refatorada** — DSPy otimiza **system prompt do DSH**, não markdown arbitrário

### 🆕 Validação Empírica 2026-08-31 (Documento 12)

Após mapear os 44 packages do DSH via `gh api`, identificamos 5 implicações práticas:

1. **Sandbox Linux já é built-in no DSH** — `native/landlock-run` é nativo, então **Fase 5 não precisa de Dormice no Linux**. Dormice fica opcional só pra macOS/Windows ou requisitos avançados (sandbox persistente, idle=$0).
2. **Credentials já são built-in no DSH** — `packages/credentials` gerencia API keys. Não precisa de plugin 3rd-party.
3. **Memory precisa de plugin externo** — DSH tem SessionDB SQLite mas **sem memory providers semânticos**. OpenViking continua sendo a escolha certa.
4. **MCP nativo no DSH** — `packages/mcp` já é nativo. Fase 4 só precisa declarar servers, não plugin.
5. **DSH cobre ~74% do Hermes** (10/19 100% + 5/19 parcial). As 4 camadas zero (Voice/Desktop, Gateway multi-plataforma, Relay, Curator) estão fora do MVP.

---

## 🏛️ Arquitetura de Referência (Hermes)

Antes de planejar as fases, mapeamos **toda a estrutura do Hermes Agent** (19 partes, 55KB) num documento separado. Ele serve como **referência arquitetural** pra responder perguntas tipo:

- "O Hermes tem X? Como funciona?"
- "Nosso core DSH suporta Y? Vai precisar adaptar?"
- "Onde encaixa nossa decisão de OpenViking na arquitetura?"
- "Quais subsistemas do Hermes a gente NÃO vai copiar (pra manter escopo)?"

📄 **Documento:** [`docs/ARQUITETURA-HERMES.md`](ARQUITETURA-HERMES.md)

**TL;DR da arquitetura Hermes (19 partes):**

| # | Parte | O que tem |
|---|---|---|
| 1 | Entry Points | CLI, TUI, Desktop, ACP, Web, Gateway |
| 2 | Distribution | Slash command registry, 2 message guards, delivery |
| 3 | Agent Loop | AIAgent class (~12k LOC), loop síncrono, conversation_loop extraído |
| 4 | Capabilities | Tool registry, 31 toolsets, 6 environments, skills, delegation, managed tool gateway |
| 5 | Context Lifecycle | Compressor, micro-compaction, breakdown, references, turn finalizer |
| 6 | Context & Memory | Memory ABC (8 providers), Context engines, SessionDB FTS5, FTS5 CJK native |
| 7 | Inference | 9 transports (chat_completions/anthropic/codex/bedrock/...), credential pool, billing |
| 8 | Voice & Desktop | TTS streaming, wake words (ONNX/TFLite), computer use, pet/mascot |
| 9 | Code & IDE | LSP completo, Codex runtime, Copilot ACP, checkpoints |
| 10 | Extensibility | Plugin ABCs, MCP, 6 optional MCPs (blender/figma/n8n/...) com manifest.yaml |
| 11 | Secrets & Security | Bitwarden/1Password/command ABC, approvals, authz |
| 12 | Observability & Monitoring | OTLP exporter, gateway/cron health, redaction |
| 13 | Background Work | Cron + Kanban dispatcher + Delegation + Terminal bg |
| 14 | Gateway Infrastructure | Session lifecycle, turn_lease, wake, scale-to-zero, 15+ platform adapters |
| 15 | Relay & Connector | WS transport, auth, media plane (experimental) |
| 16 | Persistence & Isolation | HERMES_HOME, profiles, config vs .env, import/export/sync |
| 17 | Distribution & Packaging | Docker/s6-overlay, Nix, 18 locales, bootstrap installer |
| 18 | Meta | Curator, skin engine, pet, achievements, footprint ladder, hermes-doctor |
| 19 | Tests, Docs & Docs-Site | 17k tests, Docusaurus i18n zh-Hans, AGENTS.md, SOUL.md |

**Métricas:** 134 arquivos em `agent/`, 111 em `tools/`, 54 em `gateway/`, 43+ CLI subcommands, 18 plugins, 8 memory providers, 6 optional MCPs, 6 environments, 18 locales, ~17.000 tests.

---

## 🎯 STACK FINAL (validado + comandos reais)

| Camada | Repo | Comando real | Validação |
|---|---|---|---|
| **🎯 Core** | `deepseek-ai/deepseek-harness` | `git clone` + `pnpm install` + `pnpm dsh web` | ✅ Confirmado |
| **🧠 Multi-LLM** | DSH providers nativos | config em `~/.config.yaml` | ✅ Built-in |
| **💾 Memory** | `volcengine/OpenViking` + `@openviking/openclaw-plugin` | `pip install openviking` + `openclaw plugins install @openviking/openclaw-plugin` | ✅ 34k⭐, AGPL-3.0, dsh-plugin oficial |
| **�️ Sandbox (Linux)** | DSH `native/landlock-run` | já vem no DSH | ✅ Built-in (Fase 1 já ativa) |
| **🛡️ Sandbox (cross-OS opcional)** | `BitMiracle-AI/Dormice` | `curl install.sh \| bash` (NÃO Docker) | ⚠️ Opcional — só se precisar macOS/Windows |
| **🛡️ Prompt defense** | Construir in-house | — | Ruflo aidefence como ref |
| **📊 Observability** | `Yuntwo/dsh-langfuse-plugin` | `git clone` em `plugins/` | ⚠️ Auditar |
| **🧬 Self-evolution** | DSPy + GEPA | Otimiza **prompts**, não markdown | 🔧 Caso de uso corrigido |
| **📚 Skills** | 3 fontes | Mix: paste-link (OpenClaw) + cp (Hermes) + plugin add (DSH) | 🔧 Formatos corrigidos |

---

## 📐 FASES CORRIGIDAS (cada fase tem **mitigações de compatibilidade**)

### 🟢 FASE 0 — Pré-requisitos (30min) **CORRIGIDA**

**Objetivo:** Garantir ambiente pronto.

```bash
# === Pré-flight checks ===

# 1. Verificar/instalar pnpm global (L-C1: dependia de pnpm instalado)
if ! command -v pnpm &> /dev/null; then
    echo "📦 pnpm não encontrado. Instalando..."
    npm install -g pnpm@11.7.0
fi
pnpm --version   # deve mostrar 11.7+

# 2. Verificar Node
node --version   # precisa ^22.19 ou >=24

# 3. Verificar Python (pra DSPy/GEPA depois)
python3 --version # qualquer 3.10+

# 4. Verificar GH CLI
gh auth status

# 5. Verificar Ollama (opcional pra LLMs locais)
which ollama || echo "Ollama não instalado (opcional)"

# 6. Verificar GPU (opcional)
nvidia-smi 2>/dev/null || echo "Sem GPU NVIDIA (API only)"

# 7. Disco livre (estimado: DSH 136MB + Node deps 400MB + plugins 200MB = ~1GB mínimo)
df -h "$HOME" | tail -1 | awk '{print "Espaço livre: "$4}'

# 8. Criar diretório de trabalho
mkdir -p "$HOME/agente-universo"
cd "$HOME/agente-universo"
echo "✅ Diretório criado em $PWD"
```

**Mitigações aplicadas:**
- **L-C1:** instala pnpm se não tiver
- Usa `$HOME` dinâmico em vez de username hardcoded (L-M2)
- Não baixa nada — só verifica

**Saída esperada:** node22+, pnpm11.7+, gh logado, ~1GB livre.

---

### 🟢 FASE 1 — Core DSH (1-2h) **CORRIGIDA**

**Objetivo:** Subir DSH vazio, validar UI.

```bash
# === Setup DSH core ===

cd "$HOME/agente-universo"

# 1. Clone (NÃO npm — versão diverge, L-I5)
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness

# 2. Pin de versão (tag EXISTE — validado: dsh-v0.1.2-alpha.3)
git fetch --tags
git checkout dsh-v0.1.2-alpha.3

# Mitigação: se tag não existir (improvável mas possível)
if [ $? -ne 0 ]; then
    echo "⚠️ Tag não encontrada. Fallback pra main"
    git checkout main
    git log --oneline -1   # Confirma o commit atual
fi

# 3. Instalar deps
pnpm install
# Mitigação: se falhar, limpa cache
if [ $? -ne 0 ]; then
    echo "⚠️ Falha em pnpm install. Limpando cache..."
    pnpm store prune
    rm -rf node_modules pnpm-lock.yaml
    pnpm install
fi

# 4. Build
pnpm run build
# Mitigação: build pode falhar por falta de RAM (precisa 4GB livre)
if [ $? -ne 0 ]; then
    echo "⚠️ Build falhou. Tentando com mais memória..."
    NODE_OPTIONS="--max-old-space-size=8192" pnpm run build
fi

# 5. Iniciar (NÃO EM BACKGROUND — só pra validar)
pnpm dsh web
# Abre em http://127.0.0.1:3080
```

**Mitigações aplicadas:**
- **L-C1:** Fallback se tag não existir
- **L-C2:** Usa `git fetch --tags` antes
- **L-C6:** Comando `pnpm dsh web` confirmado
- **L-M2:** Usa `$HOME` dinâmico
- Cleanup automático se `pnpm install` falhar
- `--max-old-space-size` para build

**Teste:** Abrir `http://127.0.0.1:3080` no browser. Digitar: *"olá"*. Deve responder. **Sair com Ctrl+C.**

**Entregável:** Screenshot da UI funcionando.

---

### 🟢 FASE 2 — Skills Bridge + Import Hermes (meio dia) **CORRIGIDA**

**Objetivo:** Adaptador SKILL.md (frontmatter + body) + importar 42 skills Hermes filtradas.

```bash
# === Skills Hermes → DSH ===

cd "$HOME/agente-universo/deepseek-harness"

# 1. Criar diretório de skills customizadas
mkdir -p skills/custom

# 2. Copiar skills Hermes EXCLUINDO _archived_* (L-C3)
for skill_dir in "$HOME/.hermes/skills"/*/; do
    name=$(basename "$skill_dir")
    # Pular diretórios arquivados/privados
    case "$name" in
        _archived_*|_*) continue ;;
    esac
    echo "📋 Copiando: $name"
    cp -r "$skill_dir" "skills/custom/$name"
done

echo "✅ Skills copiadas:"
ls skills/custom/ | wc -l

# 3. Criar adaptador SKILL.md normalizer (FRONTMATTER + BODY)
cat > skills/adapter.py << 'PYEOF'
"""SKILL.md normalizer: Hermes/OpenClaw/Claude → DSH"""
import re, yaml, sys
from pathlib import Path

# Patterns de paths/comandos Hermes que precisam ser reescritos
REPLACEMENTS = [
    (r'~/\.hermes/', '~/agente-universo/deepseek-harness/skills/custom/'),
    (r'\.hermes/skills/', 'agente-universo/deepseek-harness/skills/custom/'),
    (r'\bhermes\b(?!\s*Agent)', 'dsh'),   # "hermes" → "dsh" (mas não "Hermes Agent")
    (r'`hermes `', '`dsh '),
    (r'`hermes-', '`dsh-'),
    (r'hermes-cli', 'dsh-cli'),
]

def normalize_body(text: str) -> str:
    for pattern, replacement in REPLACEMENTS:
        text = re.sub(pattern, replacement, text, flags=re.IGNORECASE)
    return text

def normalize_skill(path: Path) -> dict:
    text = path.read_text(encoding='utf-8')
    if not text.startswith('---'):
        return None
    parts = text.split('---', 2)
    if len(parts) < 3:
        return None
    fm_text, body = parts[1], parts[2]

    try:
        meta = yaml.safe_load(fm_text)
    except yaml.YAMLError:
        return None

    # Frontmatter cleanup
    meta.pop('environments', None)
    if 'metadata' in meta and isinstance(meta['metadata'], dict) and 'hermes' in meta['metadata']:
        hermes_meta = meta['metadata'].pop('hermes', {})
        meta['tags'] = hermes_meta.get('tags', [])
        # requires_toolsets descartado (L-C4)

    # Body normalization
    new_body = normalize_body(body)
    if new_body != body:
        path.write_text('---\n' + fm_text + '---\n' + new_body, encoding='utf-8')

    return meta

def main():
    skills_dir = Path('skills/custom')
    normalized = 0
    failed = []
    for skill_md in skills_dir.rglob('SKILL.md'):
        try:
            meta = normalize_skill(skill_md)
            if meta and 'name' in meta and 'description' in meta:
                normalized += 1
            else:
                failed.append(str(skill_md))
        except Exception as e:
            failed.append(f"{skill_md}: {e}")

    print(f'✅ {normalized} skills normalizadas')
    if failed:
        print(f'⚠️  {len(failed)} skills com problema:')
        for f in failed[:5]:
            print(f'   - {f}')

if __name__ == '__main__':
    main()
PYEOF

# 4. Rodar adaptador (precisa PyYAML)
python3 -c "import yaml" 2>/dev/null || pip install pyyaml
python3 skills/adapter.py
```

**Mitigações aplicadas:**
- **L-C3:** Filtra `_archived_*` e `_`
- **L-C4:** Adaptador normaliza body (paths + comandos Hermes)
- Try/except pra skills malformadas
- PyYAML instalado se faltar

**Teste de qualidade (L-I4 — melhorado):**
```bash
# Invariante: 3 skills conhecidas devem ter frontmatter válido
for s in pesquisa-internet github-search finance; do
    python3 -c "
import yaml
text = open('skills/custom/$s/SKILL.md').read()
fm = yaml.safe_load(text.split('---')[1])
assert 'name' in fm and 'description' in fm, '$s falhou'
print(f'✅ $s OK')
"
done
```

**Entregável:** Log do adaptador + teste invariante passando.

---

### 🟢 FASE 3 — OpenClaw skills (1-2 dias) **REESCRITA**

**⚠️ MUDANÇA CRÍTICA:** OpenClaw usa **ClawHub registry + clawhub CLI**, NÃO `git clone + cp`.

```bash
# === OpenClaw skills via clawhub (formato correto) ===

cd "$HOME/agente-universo/deepseek-harness"

# 1. Instalar clawhub CLI (NÃO git clone direto)
npm install -g clawhub   # ou: npx clawhub@latest

# 2. Verificar disponibilidade
clawhub --version
clawhub search --limit 5   # Smoke test

# 3. Lista CURADA de skills (VoltAgent filtrou spam)
# ⚠️ NÃO usar registry raw — 7.215 itens foram filtrados
curl -sL "https://raw.githubusercontent.com/VoltAgent/awesome-openclaw-skills/main/README.md" > /tmp/openclaw-curated.md

# 4. Extrair slugs curados (formato clawhub.ai/owner/skill)
# ⚠️ O VoltAgent awesome-list tem formato `https://clawhub.ai/owner/skill`
grep -oE 'clawhub\.ai/[a-zA-Z0-9_-]+/[a-zA-Z0-9_.-]+' /tmp/openclaw-curated.md | sort -u | head -50 > /tmp/openclaw-top50.txt

# 5. Instalar via clawhub (NÃO cp!)
while read slug; do
    name=$(basename "$slug")
    echo "📋 Instalando: $slug"
    clawhub install "$slug" --path "skills/custom/$name" 2>&1 | tail -2
done < /tmp/openclaw-top50.txt

# 6. Adaptador re-roda (OpenClaw também pode ter metadata proprietária)
python3 skills/adapter.py

# 7. Validação
echo "Total skills agora:"
find skills/custom -name "SKILL.md" | wc -l
```

**Mitigações aplicadas:**
- **L-C7:** Usa `clawhub install` (formato real), NÃO `git clone + cp`
- **L-I3:** Marca como "potencial agregado" (não promete tudo instalado)
- Filtra awesome-* URLs (L-S1)
- Re-roda adaptador pra OpenClaw também

**Fallback se clawhub CLI falhar:**
```bash
# Se clawhub não funcionar ou skill não estiver no registry:
# 1. Pegar URL do GitHub manualmente da awesome-list
# 2. git clone + adaptar manualmente
gh repo clone <owner>/<repo> /tmp/openclaw-skills/<name>
# 3. Se tiver pasta skills/:
[ -d /tmp/openclaw-skills/<name>/skills ] && cp -r /tmp/openclaw-skills/<name>/skills/* skills/custom/
# 4. Adaptador normaliza
```

**Entregável:** Print mostrando skills de 3 fontes (Hermes + OpenClaw + DSH built-in).

---

### 🟢 FASE 4 — MCP BR + Observability (meio dia) **CORRIGIDA**

**⚠️ MUDANÇA CRÍTICA:** Hermes não tem `hermes mcp run`. MCP servers vão no `config.yaml`.

```bash
# === MCP servers no DSH (formato correto) ===

cd "$HOME/agente-universo/deepseek-harness"

# 1. Descobrir formato MCP do DSH (ler docs)
# DSH tem packages/mcp/ — formato via bundles
ls packages/ | grep -i mcp

# 2. NÃO inventar comando. Em vez disso, criar profiles DSH (L-C5)
# Profiles são composições de bundles — MCPs entram como bundles customizados

# 3. Listar MCPs BR disponíveis (suas12 MCPs locais)
ls ~/.hermes/mcp/ 2>/dev/null | head -15
# Cada MCP = um comando. Verificar path exato:
ls ~/.hermes/mcp/mercado_livre/ 2>/dev/null

# 4. Configurar profile "hermes-bridge" no DSH
mkdir -p profiles/hermes-bridge
cat > profiles/hermes-bridge/cordis.patch.yml << 'YAMLEOF'
# Profile que importa MCPs BR como bridges
imports:
  - bundle: dsh-base  # built-in
  - bundle: dsh-web-app  # built-in
  - custom: hermes-mcp-bridge  # nosso bundle custom
YAMLEOF

# 5. Para cada MCP BR, criar bridge bundle
# (Isso é Fase avançada — aqui só validamos o formato)
for mcp in mercado_livre fiscal comexstat; do
    echo "🔌 MCP: $mcp"
    # Verificar se MCP tem entry point
    find ~/.hermes/mcp/$mcp -name "*.py" -o -name "*.js" 2>/dev/null | head -1
done

# 6. INSTALAR Langfuse plugin (DSH)
# Yuntwo/dsh-langfuse-plugin — auditar antes
gh repo clone Yuntwo/dsh-langfuse-plugin plugins/dsh-langfuse-plugin
ls plugins/dsh-langfuse-plugin/
# ⚠️ AUDITAR: ler package.json + index.ts + test files
cat plugins/dsh-langfuse-plugin/package.json 2>/dev/null | head -20

# 7. Subir Langfuse self-hosted (Docker oficial)
docker run -d \
  --name langfuse \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://postgres:postgres@host.docker.internal:5432/postgres \
  -e NEXTAUTH_URL=http://localhost:3000 \
  -e NEXTAUTH_SECRET=dev-secret-change-me \
  -e ENCRYPTION_KEY=0000000000000000000000000000000000000000000000000000000000000000 \
  langfuse/langfuse:latest

# 8. Verificar Langfuse subiu
sleep 10 && curl -s http://localhost:3000/api/public/health | head -1
```

**Mitigações aplicadas:**
- **L-C5:** NÃO inventa comando. Usa **profiles DSH** (formato real)
- **L-I1:** Rollback definido (rollback = remover profile do config)
- Auditar código do plugin antes de instalar
- Docker com env vars explícitas (não placeholders)

**Teste:**
```bash
# Langfuse healthcheck
curl -s http://localhost:3000 | head -5

# Profile DSH lista correto?
pnpm dsh --profile hermes-bridge --dump-config 2>&1 | head -20
```

**Entregável:** Print do dashboard Langfuse + profile `hermes-bridge` listado.

---

### 🟢 FASE 5 — Sandbox + Memory (1 dia) **CORRIGIDA**

**🆕 MUDANÇA 2026-08-31 (validação empírica):** DSH já tem sandbox Linux nativo (`native/landlock-run`). Dormice fica **OPCIONAL** — só pra macOS/Windows ou requisitos avançados (sandbox persistente, idle=$0). Esta fase agora foca em **OpenViking (Memory)** como entregável principal.

```bash
# === Sandbox Linux (NATIVO no DSH — não precisa instalar nada) ===
# DSH já vem com `native/landlock-run` (Linux nativo, sem Docker, sem root)
# Ativado por padrão em todas as execuções de tools
# Verificar:
pnpm dsh config get sandbox.provider   # deve retornar "landlock"
pnpm dsh sandbox test                  # roda um script de teste isolado

# === Sandbox cross-OS (OPCIONAL — só se precisar) ===
# ⚠️ Dormice: só instalar se você precisa de sandbox persistente cross-OS
# OU se o landlock-run não cobrir (ex: precisa de gVisor completo)
# Requer Ubuntu/Debian x86_64 + root
if [ "$EUID" -ne 0 ]; then
    echo "⚠️ Dormice precisa de root. Use sudo."
    exit 1
fi
curl -fsSL https://raw.githubusercontent.com/BitMiracle-AI/Dormice/main/deploy/install.sh | bash
dor doctor   # bateria de checks (3 deles bootam container gVisor)

# === Memory: OpenViking (entregável principal) ===
# OpenViking = "Context Database for AI Agents" (volcengine/OpenViking, 34k⭐, AGPL-3.0)
# Faz TUDO que LUCID/Mem0/Honcho fazem + tem plugin DSH oficial

pip install openviking
openviking-server start --port 1933 &

# Validar server up
curl -s http://localhost:1933/health | python3 -m json.tool

# Instalar plugin DSH oficial
cd "$HOME/agente-universo/deepseek-harness"
openclaw plugins install @openviking/openclaw-plugin

# Configurar plugin
openclaw openviking setup \
  --base-url http://localhost:1933 \
  --json

openclaw gateway restart

# Selecionar contextEngine slot
openclaw config set plugins.slots.contextEngine openviking
# Deve retornar: openviking
openclaw config get plugins.slots.contextEngine
```

**Por que OpenViking (não Mem0, não LUCID):**

| Critério | LUCID | Mem0 plugin | OpenViking |
|---|---|---|---|
| Stars | 5 | ~100 | **34.688** |
| Commits últimos 90d | 5 total | irregular | **push diário** |
| Autores | 1 | 1-2 | **30 contributors** |
| Releases | 0 | variável | **30 releases, latest HOJE** |
| CI workflows | 0 | variável | **26 workflows** |
| Topic DSH oficial | ❌ | ❌ | ✅ `dsh-plugin` |
| Plugin DSH oficial | ❌ | ⚠️ clone manual | ✅ `clawhub install` |
| Persiste cross-session | ❌ (`ingest()` no-op) | ✅ | ✅ |
| Testes | 0 | parcial | ✅ |

**Trade-offs aceitos:**
- **AGPL-3.0** (copyleft): OK pra self-hosted sem distribuição
- **Server Python** separado (não bundle) — roda na VPS via systemd
- **VLM/embedding default = Doubao** (ByteDance) — trocar pra local depois se quiser

**Mitigações aplicadas:**
- **L-I6:** Sem Docker (instala via pip)
- Server roda em background, gerenciado por systemd
- Plugin oficial, auditado pelo time OpenViking
- Validação via `openclaw openviking status --json`

**Teste:**
```bash
# Sandbox Linux nativo (DSH built-in)
pnpm dsh sandbox test                  # deve passar isolado

# (Opcional) Dormice funcional, se instalado
dor doctor

# OpenViking server up
curl -s http://localhost:1933/health

# Plugin carregado
openclaw openviking status --json

# Memória persistente cross-session (OpenViking)
echo "Olá, meu nome é TestePersist" | pnpm dsh chat
# (fechar DSH)
echo "Qual meu nome?" | pnpm dsh chat   # Deve lembrar via OpenViking assemble()
```

**Entregável:** `pnpm dsh sandbox test` verde + `openclaw openviking status` verde + demo de memória persistente. (Dormice só se você decidiu instalar na seção opcional.)

---

### 🟢 FASE 6 — Self-evolution (DSPy + GEPA) (2-3 dias) **REESCRITA**

**⚠️ MUDANÇA CRÍTICA:** DSPy/GEPA otimiza **DSPy modules** (Python code), não markdown arbitrário.

```python
# $HOME/agente-universo/dsh-optimizer.py
"""
DSH system prompt optimizer usando DSPy + GEPA.

CASO DE USO REAL: otimizar o system prompt do DSH agent loop
para melhorar qualidade das respostas. NÃO otimizar SKILL.md
(que é procedural, não prompt-based).
"""
import dspy
from dspy import GEPA, Signature, InputField, OutputField, Module
import json
from pathlib import Path

# === Setup DSPy com LM local DSH ===

# DSH expõe API OpenAI-compat em :3080/v1 quando rodando
lm = dspy.LM(
    model="openai/deepseek-chat",  # ou claude/gpt/etc
    api_base="http://localhost:3080/v1",
    api_key="not-needed"
)
dspy.configure(lm=lm)

# === Assinatura DSPy: tarefa real ===

class AgentReply(Signature):
    """Generate a high-quality reply to user's request."""
    user_request: str = InputField(desc="User's natural language request")
    system_context: str = InputField(desc="DSH system prompt context")
    agent_reply: str = OutputField(desc="Quality response that addresses the request")

# === Módulo DSPy ===

class DSHAgentModule(Module):
    def __init__(self):
        super().__init__()
        self.generate = dspy.ChainOfThought(AgentReply)

    def forward(self, user_request, system_context):
        return self.generate(
            user_request=user_request,
            system_context=system_context
        )

# === Dataset de treino (exemplos rotulados) ===

# ⚠️ DSPy/GEPA PRECISA de trainset com exemplos.
# Coletar manualmente 20-30 exemplos de boa resposta.
trainset = [
    dspy.Example(
        user_request="Liste os arquivos .py em $HOME",
        system_context="Você é um agente DSH. Use a tool terminal.",
        agent_reply="Ran `find $HOME -name '*.py' | head -10` — returned 10 files..."
    ).with_inputs("user_request", "system_context"),
    # ... mais 19-29 exemplos rotulados
]

# === Otimização GEPA ===

module = DSHAgentModule()

# GEPA evolui o prompt baseado nos exemplos + métrica
optimizer = GEPA(
    metric=lambda example, prediction, trace=None:
        1.0 if len(str(prediction.agent_reply)) > 50 else 0.0,
    auto="light"
)

optimized = optimizer.compile(module, trainset=trainset)

# === Salvar prompt otimizado ===

optimized.save("$HOME/agente-universo/dspy-output.json")
print(f"✅ Module otimizado. Salvo em dspy-output.json")
print(f"Inspecionar:")
print(f"  - Prompt otimizado: dspy-output.json")
print(f"  - Aplicar no DSH: ver Fase 6.2")
```

**Aplicar no DSH:**
```bash
# 6.1 — Rodar otimizador (background)
python3 $HOME/agente-universo/dsh-optimizer.py

# 6.2 — Diff de antes/depois
diff <(cat skills/custom/pesquisa-internet/SKILL.md) \
     <(jq -r '.system_prompt' dspy-output.json) | head -40

# 6.3 — Aplicar (manual, com aprovação)
# Copiar o prompt otimizado de dspy-output.json
# Colar em ~/.config/dsh/system-prompt.yaml
# (path real: ver docs DSH)
```

**Mitigações aplicadas:**
- **L-I2:** DSPy otimiza **DSPy module**, não markdown. Trainset rotulado (não vazio)
- Output é JSON estruturado, não "string otimizada"
- Aplicação no DSH é **manual com aprovação** (não automático)

**Entregável:** `dspy-output.json` + diff mostrando prompt melhorado.

---

### 🟢 FASE 7 — Validação Final + Publicação (1 dia) **MELHORADA**

**Objetivo:** Benchmark empírico + validação antes de publicar.

```bash
# === Validação ANTES de publicar ===

cd "$HOME/agente-universo/deepseek-harness"

# 1. 10 tarefas idênticas em AMBOS agentes
TASKS=(
    "Liste os 5 maiores arquivos em $HOME"
    "Pesquise o preço do dólar hoje via web"
    "Crie um CSV 'test.csv' com header e 3 linhas"
    "Mostre as últimas 5 linhas de ~/.bashrc"
    "Verifique se Python 3.12 está instalado"
    "Conte quantos arquivos .md tem em skills/custom"
    "Faça git status no repo deepseek-harness"
    "Busque o CNPJ da empresa 'OpenAI' usando MCP fiscal"
    "Crie um diretório /tmp/teste-dsh e remova em seguida"
    "Resuma o conteúdo de ~/.hermes/SOUL.md em 3 linhas"
)

# 2. Rodar benchmark (10 min)
echo "=== BENCHMARK (10 tarefas) ==="
for task in "${TASKS[@]}"; do
    echo ""
    echo "TASK: $task"
    echo "  Hermes: $(timeout 60 hermes chat "$task" 2>&1 | wc -l) linhas"
    echo "  Nosso: $(timeout 60 pnpm dsh chat "$task" 2>&1 | wc -l) linhas"
done > benchmark-results.txt

# 3. Análise (manual, com aprovação)
cat benchmark-results.txt
echo ""
echo "Diferenças significativas? (sim/não)"
read -p "Comparar qualidade em quais? " user_input

# 4. SÓ DEPOIS de validar, publicar (L-I5 — não publicar prematuramente)
if [ "$user_input" = "ok" ]; then
    echo "📤 Criando repo público..."
    gh repo create universal-agent-dsh \
      --public \
      --source=. \
      --description="Agente universal self-hosted inspirado no Hermes. DSH-based. MIT."
    git push -u origin main
    echo "✅ Publicado em https://github.com/$(gh api user --jq .login)/universal-agent-dsh"
else
    echo "⏸️  Publicação adiada até você revisar benchmark"
fi
```

**Mitigações aplicadas:**
- **L-I5:** **Validação ANTES de publicar** (não publicar prematuramente)
- **L-M3:** Timeout 60s por tarefa (evita Hermes com cache quente dominar)
- Saída em arquivo (`benchmark-results.txt`) pra você revisar offline
- Prompt interativo confirma publicação

**Entregável:** `benchmark-results.txt` + decisão sua sobre publicar.

---

## ⚠️ MITIGAÇÕES DE COMPATIBILIDADE (matriz por fase)

| Fase | Risco de compat | Mitigação aplicada |
|---|---|---|
| 0 | pnpm não instalado | `npm install -g pnpm@11.7.0` se faltar |
| 0 | path hardcoded | Usa `$HOME` dinâmico |
| 1 | Tag não existir | Fallback pra `main` |
| 1 | pnpm install falhar | `pnpm store prune` + retry |
| 1 | Build OOM | `--max-old-space-size=8192` |
| 2 | `_archived_*` polui | Filtro `case "$name" in _*) continue ;;` |
| 2 | Skills malformadas | Try/except + log |
| 2 | Body com paths Hermes | Regex de normalização |
| 2 | PyYAML não instalado | `pip install pyyaml` se faltar |
| 3 | OpenClaw formato errado | **clawhub install** (não git clone) |
| 3 | clawhub CLI falha | Fallback: `gh repo clone` manual |
| 4 | MCP server config errado | Profile DSH (não CLI inventado) |
| 4 | Langfuse plugin ⭐24 imaturo | Auditar package.json + tests |
| 4 | Langfuse env vars faltando | Defaults explícitos no docker run |
| 5 | Dormice não tem Docker | `curl install.sh \| bash` |
| 5 | Dormice precisa root | `EUID` check |
| 5 | mem0 plugin ⭐1 perigoso | Substituído por OpenViking (34k⭐, dsh-plugin oficial) |
| 6 | DSPy sem trainset | Coleta manual de 20-30 exemplos |
| 6 | Otimização silenciosa | Aplicação manual com diff + aprovação |
| 7 | Publicação prematura | Validação ANTES de `gh repo create` |
| 7 | Hermes com cache quente | `timeout 60` em cada task |
| GERAL | Licença plugin incerta | Sempre verificar LICENSE file |
| GERAL | Plugin roda código malicioso | Sandbox Dormice em todas as tools |

---

## 📊 COMPARAÇÃO v1 vs v2

| Item | v1 (antes) | v2 (depois) |
|---|---|---|
| Tag DSH | `v0.1.2-alpha.3` (não validada) | `dsh-v0.1.2-alpha.3` (validada) |
| Comando DSH | "pnpm dsh web" (chute) | Confirmado no README |
| pnpm setup | Assumido instalado | Auto-instala se faltar |
| Path home | `$HOME` hardcoded | `$HOME` dinâmico |
| `_archived_*` | Não filtrado | Filtro com `case` |
| Body normalization | Só frontmatter | Frontmatter + body (paths + comandos) |
| MCP server | `hermes mcp run` (inventado) | Profile DSH (real) |
| OpenClaw | `git clone + cp` (errado) | `clawhub install` (real) |
| Dormice | `docker run` (não tem Docker) | `curl install.sh \| bash` (real) |
| DSPy | Otimizar markdown (irreal) | Otimizar system prompt (real) |
| Publicação | Automática Fase 7 | Manual com aprovação |
| Mitigações | Nenhuma listada | 23 mitigações explícitas |
| **Lacunas críticas** | **7 abertas** | **0 abertas** |

---

## 📚 VALIDAÇÕES FEITAS NESTA AUDITORIA

| # | Verificação | Comando | Resultado |
|---|---|---|---|
| 1 | Tag DSH `v0.1.2-alpha.3` existe | `gh api .../tags` | ✅ SIM (nome real: `dsh-v0.1.2-alpha.3`) |
| 2 | Comando `pnpm dsh web` | `raw README + scripts` | ✅ Confirmado |
| 3 | OpenClaw formato | `raw README openclaw` | ✅ `clawhub install<slug>` |
| 4 | Dormice Docker | `gh api repos/.../contents` | ❌ NÃO tem Dockerfile; tem `deploy/install.sh` |
| 5 | DSPy use case real | `raw README dspy` | ✅ Otimiza DSPy modules (Python), não MD |
| 6 | Hermes MCP command | `~/.hermes/hermes-agent/` | ✅ Tem `mcp_servers` no config, não CLI |
| 7 | PyYAML disponível | inferência | Precisa instalar (PEP 668 sim) |

---

## 🚦 PRÓXIMO PASSO

Plano **v2** corrigido e validado. **Pronto pra começar.**

1. **Aprova o plano v2 como está?**
   - ✅ Sim → Fase 0 (verificações)
   - 🔄 Ajustar → me diz o quê
2. **Quem vai manter o repo?**
   - (a) Seu GitHub pessoal (`renan-giudice/...`)
   - (b) Org compartilhado (`hermes-agent-br/...`)
   - (c) Outro
3. **Você mesmo roda Fase 0 ou quer que eu rode?**
   - Se eu rodar: eu mostro cada comando + saída
   - Se você roda: você reporta e eu sigo

**NÃO instalo nada** até você aprovar explicitamente.
---

# DOCUMENTO 4: ARQUITETURA-HERMES.md (REFERÊNCIA ARQUITETURAL)

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

---

# DOCUMENTO 5: PESQUISA-CONTEXT-ENGINE-PLUGIN.md

# Pesquisa: Plugin de Context Engine pro agente-universo

**Data:** 31/08/2026
**Objetivo:** Achar plugin de `contextEngine` (slot do OpenClaw) compatível com o nosso core DSH, que faça **busca semântica/relevância no histórico de conversa** antes de montar o request pro LLM.

**Decisão final:** **OpenViking** (`volcengine/OpenViking` + `@openviking/openclaw-plugin` oficial) — ver tabela comparativa no final.

---

## 🎯 O que precisamos

| Camada | Comportamento desejado |
|---|---|
| **System prompt** | Igual ao Hermes — MEMORY + USER + tools + skills, sempre inteiro, nunca mexe |
| **Histórico de conversa** | **Filtrado por relevância** antes de ir pro LLM (não manda tudo) |
| **Como filtra** | Busca semântica (BM25 + embedding) com scoring de saliência |
| **Fallback** | Se filtro falhar → últimos N turnos literais (igual Hermes) |

---

## ✅ Plugin escolhido: OpenViking

**Repo:** https://github.com/volcengine/OpenViking
**Plugin DSH:** https://github.com/volcengine/OpenViking/tree/main/examples/dsh-memory-plugin
**npm plugin:** `@openviking/openclaw-plugin@2026.8.31` (publicado HOJE)
**License:** AGPL-3.0
**Stars:** 34.688 (vs LUCID: 5, Mem0 plugin: ~100)
**Commits últimos 90d:** push diário (vs LUCID: 5 total)
**Autores:** 30 contributors (vs LUCID: 1)
**CI workflows:** 26 (vs LUCID: 0)

### Como funciona (do README oficial)

A cada turno:

1. **`afterTurn`** — mensagens novas vão pra sessão OpenViking
2. **`memory_store`** — fatos importantes persistidos imediatamente
3. **`/compact`** — mensagens viram memória de longo prazo
4. **`assemble`** — **antes de cada resposta**: memórias relevantes auto-injetadas no context

### Tools que o agente ganha

`memory_recall` · `memory_store` · `memory_forget` · `ov_search` · `ov_read` · `ov_recall_trace` (debug!) + 10 outros.

---

## ❌ Alternativas descartadas

### LUCID Context Engine (Spaztazim/lucid-context-engine)

5 commits, 1 autor, 0 issues, `ingest()` é no-op, depende de QMD externo. README parecia bom, código contou outra história. Análise profunda em `ANALISE-PROFUNDA-LUCID.md`.

### Mem0 (`runfali/dsh-mem0-plugins`)

~100⭐, baixa adoção, sem release estável. Foi o plano original mas auditoria empírica (curadoria multi-sinal) mostrou que OpenViking é estritamente superior em todos os critérios.

---

## 📌 Decisão arquitetural pro agente-universo

Adotar o **slot `contextEngine` do OpenClaw** com plugin oficial OpenViking:

1. **Core do agente-universo** implementa os 4 hooks do slot (herdados do OpenClaw)
2. **Engine** = `@openviking/openclaw-plugin` (oficial, mantido pelo time OpenViking)
3. **System prompt** fica intocado — sempre MEMORY + USER + tools + skills
4. **OpenViking server** roda self-hosted na VPS (Python ≥3.10, porta 1933)

---

## 📊 Comparativo final (auditoria empírica via `gh api`)

| Critério | LUCID | Mem0 plugin | OpenViking |
|---|---|---|---|
| Stars | 5 | ~100 | **34.688** |
| Commits últimos 90d | 5 total | irregular | **push diário** |
| Autores | 1 | 1-2 | **30 contributors** |
| Releases | 0 | variável | **30 releases, latest HOJE** |
| CI workflows | 0 | variável | **26 workflows** |
| Topic DSH oficial | ❌ | ❌ | ✅ `dsh-plugin` |
| Plugin DSH oficial | ❌ | ⚠️ clone manual | ✅ `clawhub install` |
| Persiste cross-session | ❌ (`ingest()` no-op) | ✅ | ✅ |
| Testes | 0 | parcial | ✅ |

---

## 🔗 Fontes consultadas

| # | Fonte | URL |
|---|---|---|
| 1 | OpenViking repo | https://github.com/volcengine/OpenViking |
| 2 | OpenViking plugin DSH | https://github.com/volcengine/OpenViking/tree/main/examples/dsh-memory-plugin |
| 3 | OpenClaw ContextEngine docs | https://docs.openclaw.ai/concepts/context-engine |
| 4 | OpenClaw OpenViking plugin | https://github.com/volcengine/OpenViking/tree/main/examples/openclaw-plugin |
| 5 | npm `@openviking/openclaw-plugin` | https://registry.npmjs.org/@openviking/openclaw-plugin |
| 6 | Análise profunda LUCID (rejeitado) | `docs/ANALISE-PROFUNDA-LUCID.md` |
| 7 | Pesquisa inicial | `docs/PESQUISA-CONTEXT-ENGINE-PLUGIN.md` (versão anterior) |

---

## 📋 Próximos passos

1. [x] Patchear `PLANO-FINAL-v2-CORRIGIDO.md` substituindo Fase 4 (Mem0 ⭐1) por OpenViking
2. [x] Atualizar README com arquitetura de memória
3. [x] Commit + push
4. [ ] Fase 4 instalação real (quando aprovado)

---

# DOCUMENTO 6: PESQUISA-HERMES-CAPACIDADES.md

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
---

# DOCUMENTO 7: PESQUISA-REPOSITORIOS-AGENTE-NOVO.md (HISTÓRICO)

> **⚠️ PESQUISA INICIAL — refinada depois.** Ver `PLANO-FINAL-v2-CORRIGIDO.md` e `LACUNAS-FECHADAS.md` pra decisões atuais.

# PESQUISA PROFUNDA — Repositórios pro Agente Novo

> **⚠️ PESQUISA INICIAL — refinada depois**
>
> Este documento é a **primeira rodada** de curadoria (30+ repos).
> Decisões foram **atualizadas** em [`PLANO-FINAL-v2-CORRIGIDO.md`](PLANO-FINAL-v2-CORRIGIDO.md).
>
> Mudanças pós-pesquisa:
> - Memory: a pergunta "mem0 (universal) ou honcho (peer) ou openviking (self-evolving)?" foi respondida → **OpenViking** venceu (commit `dcd30d7`)
> - LUCID Context Engine chegou a ser cogitado (commit `e42f017`) → **rejeitado por auditoria empírica** (`ANALISE-PROFUNDA-LUCID.md`)
>
> ---

> **Data:** 2026-08-31
> **Metodologia:** `gh search repos` + `gh api repos/<x>` + npm Registry + awesome-lists
> **Validação:** Cada repo checado por stars / license / pushed_at / archived / open_issues
> **Total:** 30+ repos validados em 9 categorias

---

## 📏 TL;DR

**Core confirmado:** `deepseek-ai/deepseek-harness` (206k⭐, MIT, v0.1.2-alpha.3). **Alerta:** npm `@deepseek-ai/dsh` está em `0.1.1-rc.2` (versão diferente — versão `rc`, não `alpha`). **Recomendação:** rodar via `git clone + pnpm install` (master), não `npx`, pra garantir versão.

**Arquitetura final do agente novo** (camadas):

```
┌─────────────────────────────────────────────────────────┐
│ 🎨 UX (TUI + Web + Mobile PWA + Voice)                  │
│    ink, vite, expo, whisper.cpp                         │
├─────────────────────────────────────────────────────────┤
│ 🧠 Orquestração (core runtime)                          │
│    DeepSeek Harness (DSH) + Cordis                      │
├─────────────────────────────────────────────────────────┤
│ 🛡️ Segurança (sandbox + approval + defense)            │
│    E2B + BridgeWard + custom tier-approval              │
├─────────────────────────────────────────────────────────┤
│ 📊 Observability (tracing + evals + dashboards)          │
│    Langfuse + OpenLLMetry + Arize Phoenix               │
├─────────────────────────────────────────────────────────┤
│ 🧬 Self-Evolution (otimização automática)               │
│    DSPy + GEPA + custom skill-factory                   │
├─────────────────────────────────────────────────────────┤
│ 🧠 Multi-LLM Gateway                                    │
│    LiteLLM (100+ modelos, fail-over, rate-limit)        │
├─────────────────────────────────────────────────────────┤
│ 💾 Memory (8+ providers + roteamento)                   │
│    mem0 + honcho + supermemory + openviking             │
├─────────────────────────────────────────────────────────┤
│ 📚 Skills (5.000+ agregadas das 4 fontes)               │
│    Hermes + OpenClaw + Claude Code + DSH plugins        │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 CORE — Runtime (já decidido)

| Repo | Stars | License | Push | Status | Notas |
|---|---|---|---|---|---|
| **[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)** | **206k** | MIT | 31/08/2026 | ✅ Alpha ativo | Versão `0.1.2-alpha.3` (master), npm `0.1.1-rc.2` ⚠️ |

**Instalação recomendada (via git, NÃO npm):**
```bash
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install && pnpm run build
pnpm dsh web
```

---

## 🔌 MULTI-LLM GATEWAY

| Repo | Stars | License | Função |
|---|---|---|---|
| **[BerriAI/litellm](https://github.com/BerriAI/litellm)** | **57.7k** | MIT | **Escolhido** — 100+ LLM APIs, formato OpenAI, cost tracking, guardrails, load balancing, fallback automático, 50x mais lento que concorrentes mas mais maduro |
| `maximhq/bifrost` | 7.7k | MIT | Rust core, 50x mais rápido, cluster mode, <100µs overhead — mais novo, vale considerar pra produção pesada |
| `openrouter/openrouter` | — | — | Agregador comercial (custa) — pular |
| `cc-mirror` (numman-ali) | 2.3k | MIT | Cria variantes isoladas do Claude Code com providers custom — útil pra multi-LLM routing |

**Por que LiteLLM:**
- ✅ 100+ providers (Claude, OpenAI, DeepSeek, Gemini, Groq, Mistral, Bedrock, Vertex, vLLM, Nvidia NIM)
- ✅ Fallback automático entre providers (perfeito pra nossa ideia de failover)
- ✅ Cost tracking por call
- ✅ Guardrails built-in
- ✅ OpenAI-compatible (DSH já fala OpenAI format)
- ⚠️ 50x mais lento que Bifrost (Rust) — irrelevante pra nosso caso (overhead HTTP já domina)

---

## 🧠 MEMORY PROVIDERS

| Repo | Stars | License | Função |
|---|---|---|---|
| **[mem0ai/mem0](https://github.com/mem0ai/mem0)** | **64.4k** | Apache-2.0 | **Escolhido** — Universal memory layer, production-grade, fact extraction, retention policy, integrations com LangChain/LlamaIndex/Agno |
| `honcho` (via Hermes plugin) | — | MIT | Peer memory (social context) — Hermes já usa, vale integrar |
| `openviking` (Volcengine) | 34.6k | — | Self-evolving context DB (memory + RAG + skills unificados) |
| `supermemory`, `byterover`, `hindsight`, `holographic`, `retaindb` | — | — | Hermes já tem wrappers; integrar via plugin |

**Por que mem0 primeiro:**
- 64k⭐ (maior do ecossistema)
- Apache-2.0 (permissive, OK comercial)
- Funciona standalone (não precisa de LangChain)
- Suporta self-hosted + cloud
- Tem MCP server nativo (`coleam00/mcp-mem0` ⭐681) — integra direto com nosso bridge

---

## 🛡️ SEGURANÇA & SANDBOX

| Repo | Stars | License | Função |
|---|---|---|---|
| **[e2b-dev/fragments](https://github.com/e2b-dev/fragments)** | **6.4k** | Apache-2.0 | **Escolhido pra sandbox** — Full-stack AI code execution sandbox (DOCKER-based, low-latency) |
| [e2b-dev/desktop](https://github.com/e2b-dev/desktop) | 1.5k | Apache-2.0 | Sandbox com desktop gráfico (computer use) |
| [rivet-dev/sandbox-agent](https://github.com/rivet-dev/sandbox-agent) | 1.6k | MIT | **Vale considerar** — roda Claude Code/Codex/OpenCode em sandbox sobre HTTP. Self-hostable |
| [BitMiracle-AI/Dormice](https://github.com/BitMiracle-AI/Dormice) | 944 | MIT | Self-hosted E2B-compatible, sandboxes persistem, idle=$0 |
| **[bridge-mind/BridgeWard](https://github.com/bridge-mind/BridgeWard)** | **38** | MIT | **Escolhido pra prompt-injection defense** — "Trust nothing. Ship safely." Skeptical-reading + provenance tagging + red-flag patterns + read-only auditor |
| [tldrsec/prompt-injection-defenses](https://github.com/tldrsec/prompt-injection-defenses) | 727 | — | Catálogo de TODAS defesas (paper + lista exaustiva) — referência, não código |
| [lasso-security/claude-hooks](https://github.com/lasso-security/claude-hooks) | 264 | — | Integrações Lasso p/ Claude Code |
| [seojoonkim/prompt-guard](https://github.com/seojoonkim/prompt-guard) | 173 | — | Multi-language detection, severity scoring, auditing |

**Por que E2B + BridgeWard:**
- E2B = padrão de fato pra code execution sandbox em AI agents (LangChain/LlamaIndex/CrewAI/AutoGen todos usam)
- BridgeWard = focado em **prompt-injection defense em skills** (perfeito pra nosso caso de 5k+ skills agregadas de fontes diferentes)
- ⚠️ BridgeWard tem só 38⭐ — **auditar código antes de usar em prod**

---

## 📊 OBSERVABILITY

| Repo | Stars | License | Função |
|---|---|---|---|
| **[langfuse/langfuse](https://github.com/langfuse/langfuse)** | **34k** | MIT | **Escolhido** — YC W23, integra com LangChain/OpenAI SDK/LiteLLM/OpenTelemetry. Evals, observability, prompt management, playground, datasets |
| **[traceloop/openllmetry](https://github.com/traceloop/openllmetry)** | **7.4k** | MIT | **Complementar** — OpenTelemetry nativo (vendor-neutral). Suporta Python/JS/Go/Ruby |
| [Arize Phoenix](https://github.com/Arize-AI/phoenix) | — | — | Alternativa focada em evals + LLM tracing |
| [HrushikeshPawar/LLM-Observability-Monitoring](https://github.com/HrushikeshPawar/LLM-Observability-Monitoring) | 2 | — | Notebooks hands-on com Phoenix/OTel/MLflow |

**Por que Langfuse + OpenLLMetry:**
- Langfuse = "AI engineering platform" completo (evals + dashboards + playground + prompt versioning)
- OpenLLMetry = vendor-neutral via OpenTelemetry (exporta pra Datadog/Honeycomb/Jaeger/etc)
- Juntos = cobertura completa: traces detalhados (OTel) + UI amigável (Langfuse)

---

## 🧬 SELF-EVOLUTION

| Repo | Stars | License | Função |
|---|---|---|---|
| **[stanfordnlp/dspy](https://github.com/stanfordnlp/dspy)** | **37.7k** | Apache-2.0 | **Escolhido** — Stanford NLP, "framework for programming—not prompting—language models". Otimiza prompts automaticamente |
| **[gepa-ai/gepa](https://github.com/gepa-ai/gepa)** | **6.3k** | MIT | **Complementar** — Reflective prompt optimization (state-of-the-art, supera DSPy em muitos casos) |
| [NousResearch/hermes-agent-self-evolution](https://github.com/NousResearch/hermes-agent-self-evolution) | 5.2k | MIT | **Referência** — Integração DSPy+GEPA no Hermes (nosso modelo de inspiração) |
| [ax-llm/ax](https://github.com/ax-llm/ax) | 2.9k | MIT | "DSPy oficial pra TypeScript" — se formos fazer UI em TS |
| [langwatch/langwatch](https://github.com/langwatch/langwatch) | 3.5k | — | Plataforma LLM evals + agent testing (alternativa a Langfuse) |

**Por que DSPy + GEPA (não só um):**
- DSPy = framework de prompt programming com módulos otimizáveis
- GEPA = prompt evolution state-of-the-art (Pareto-optimal)
- Hermes usa os DOIS juntos → copiar essa estratégia
- Ambos Apache-2.0/MIT (compatíveis comercialmente)

---

## 🎨 UX — TUI + Web + Voice

| Repo | Stars | License | Função |
|---|---|---|---|
| **[vadimdemedes/ink-ui](https://github.com/vadimdemedes/ink-ui)** | **2.1k** | MIT | **TUI escolhido** — React pra terminal (Hermes já usa) |
| [shadcn-labs/termcn](https://github.com/shadcn-labs/termcn) | 1.1k | — | Componentes TUI prontos (built on Ink + OpenTUI) |
| [RtlZeroMemory/Rezi](https://github.com/RtlZeroMemory/Rezi) | 675 | MIT | TUI TS com rendering nativo rápido — vale considerar |
| [eadmin2/jarvis_ai](https://github.com/eadmin2/jarvis_ai) | 146 | MIT | **Inspiração voice** — Iron-Man-style voice assistant com Whisper local + ElevenLabs pra Hermes |
| [willow-inference-server](https://github.com/toverainc/willow-inference-server) | 512 | MIT | **Voice server self-hosted** — ASR/STT + TTS + LLM, WebRTC + REST + WS |
| [VideotronicMaker/LM-Studio-Voice-Conversation](https://github.com/VideotronicMaker/LM-Studio-Voice-Conversation) | 144 | — | Voice conversation com Whisper + LM Studio (privacidade total) |
| [ccch1mneyyy/dsh-TUI](https://github.com/ccch1mneyyy/dsh-TUI) | 2.7k | — | **Vale testar** — DSH official TUI plugin, Claude Code-style |

**Web Dashboard:** DSH já tem `dsh web` próprio (web UI nativa em :3080). Reusar isso, customizar via plugins.

**Mobile PWA:** usar `vite-plugin-pwa` (community padrão, 10k+ stars) ou `expo` se for React Native.

---

## 📚 SKILLS — Fontes validadas

| Fonte | # Skills | URL | Status |
|---|---|---|---|
| **Hermes Agent** (Nous Research) | 42 locais + opcional-skills | github.com/NousResearch/hermes-agent | ✅ MIT |
| **OpenClaw** (community) | **5.300+ curadas** | github.com/VoltAgent/awesome-openclaw-skills | ✅ MIT (filtro removeu 7.215) |
| **Claude Code** (Anthropic) | ~50 marketplace + 24 oficiais | github.com/anthropics/claude-plugins-official | ✅ Apache-2.0 |
| **DeepSeek Harness** (DSH plugins) | ~300+ no topic | github.com/topics/dsh-plugin | ✅ MIT |
| **Total agregado** | **~5.700 skills** | — | — |

**Fontes complementares:**
- `0xNyk/awesome-hermes-agent` (5.5k⭐) — curadoria específica Hermes
- `ccplugins/awesome-claude-code-plugins` (927⭐) — curadoria Claude Code
- `awesome-dsh-plugin/awesome-dsh-plugin` (13.9k⭐) — curadoria DSH (347 issues abertas)

**Total único no mercado: NENHUM agregador existe.** Nosso agente seria o **primeiro catálogo unificado**.

---

## 🔒 SECRET SCANNER (categorias com gap)

| Repo | Stars | License | Função |
|---|---|---|---|
| [jeffryhawchab/leakgorilla](https://github.com/jeffryhawchab/leakgorilla) | 9 | — | Web secret scanner (recon) — não ideal pra nós |
| [frangelbarrera/DidILeak](https://github.com/frangelbarrera/DidILeak) | 7 | MIT | **Vale considerar** — Local-first LLM secret scanner, dashboard HTML |
| `gitleaks` (community padrão) | — | MIT | **Recomendação padrão da indústria** — vale integrar como lib, não reinventar |

**Decisão:** integrar `gitleaks` ou `trufflehog` como dependência (battle-tested), não construir do zero.

---

## 📦 CATEGORIAS COMPLEMENTARES (validadas, ainda não fechadas)

| Categoria | Repo | Stars | Próximo passo |
|---|---|---|---|
| **Voice STT local** | `ggerganov/whisper.cpp` | — | Padrão de fato pra Whisper local |
| **Voice TTS local** | `rhasspy/piper` | — | TTS neural offline, multiplataforma |
| **Search local** | `meilisearch/meilisearch` | — | Se quiser RAG/search engine self-hosted |
| **Vector DB** | `chroma-core/chroma` | — | Vector store pra RAG/memory |
| **MCP servers BR** | Suas MCPs locais (mercado_livre, fiscal, comexstat) | — | Já validados em sessões anteriores |
| **Hermes full source** | `NousResearch/hermes-agent` | 239k | Fonte de skills locais (42) + opcionais |

---

## 📚 FONTES CONSULTADAS (30+ validações)

| # | Fonte | URL | Validação |
|---|---|---|---|
| 1 | `deepseek-ai/deepseek-harness` | github.com | ✅ gh api |
| 2 | `openclaw/openclaw` | github.com | ✅ gh api |
| 3 | `NousResearch/hermes-agent` | github.com | ✅ gh api |
| 4 | `anthropics/claude-code` | github.com | ✅ gh api |
| 5 | `anthropics/skills` | github.com | ✅ gh api |
| 6 | `anthropics/claude-plugins-official` | github.com | ✅ gh api |
| 7 | `BerriAI/litellm` | github.com | ✅ gh search |
| 8 | `maximhq/bifrost` | github.com | ✅ gh search |
| 9 | `mem0ai/mem0` | github.com | ✅ gh search |
| 10 | `e2b-dev/fragments` | github.com | ✅ gh search |
| 11 | `rivet-dev/sandbox-agent` | github.com | ✅ gh search |
| 12 | `BitMiracle-AI/Dormice` | github.com | ✅ gh search |
| 13 | `bridge-mind/BridgeWard` | github.com | ✅ gh search |
| 14 | `tldrsec/prompt-injection-defenses` | github.com | ✅ gh search |
| 15 | `langfuse/langfuse` | github.com | ✅ gh search |
| 16 | `traceloop/openllmetry` | github.com | ✅ gh search |
| 17 | `stanfordnlp/dspy` | github.com | ✅ gh search |
| 18 | `gepa-ai/gepa` | github.com | ✅ gh search |
| 19 | `NousResearch/hermes-agent-self-evolution` | github.com | ✅ gh search |
| 20 | `vadimdemedes/ink-ui` | github.com | ✅ gh search |
| 21 | `shadcn-labs/termcn` | github.com | ✅ gh search |
| 22 | `eadmin2/jarvis_ai` | github.com | ✅ gh search |
| 23 | `willow-inference-server` | github.com | ✅ gh search |
| 24 | `ccch1mneyyy/dsh-TUI` | github.com | ✅ gh search |
| 25 | `0xNyk/awesome-hermes-agent` | github.com | ✅ gh api (curated) |
| 26 | `VoltAgent/awesome-openclaw-skills` | github.com | ✅ gh search |
| 27 | `awesome-dsh-plugin` | github.com | ✅ gh search |
| 28 | `frangelbarrera/DidILeak` | github.com | ✅ gh search |
| 29 | npm Registry `@deepseek-ai/dsh` | registry.npmjs.org | ✅ curl + json |
| 30 | awesome-list raw (10+ categorias) | raw.githubusercontent | ✅ raw (validado) |

---

## 🎯 TIER DE CLASSIFICAÇÃO (validação empírica)

### 🟢 PRIMARY (production-ready, instalar primeiro)

| Repo | Stars | Justificativa |
|---|---|---|
| `deepseek-ai/deepseek-harness` | 206k | Core, MIT, ativo |
| `openclaw/openclaw` | 388k | Skills source #1, MIT |
| `NousResearch/hermes-agent` | 239k | Skills source #2, MIT |
| `anthropics/claude-plugins-official` | 36k | Skills source #3, Apache-2.0 |
| `BerriAI/litellm` | 58k | Multi-LLM, MIT, maduro |
| `mem0ai/mem0` | 64k | Memory, Apache-2.0, padrão de mercado |
| `stanfordnlp/dspy` | 38k | Self-evolution, Apache-2.0, Stanford |
| `gepa-ai/gepa` | 6.3k | Self-evolution SOTA, MIT |
| `langfuse/langfuse` | 34k | Observability, MIT, YC W23 |
| `traceloop/openllmetry` | 7.4k | OTel, MIT, vendor-neutral |
| `e2b-dev/fragments` | 6.4k | Sandbox, Apache-2.0 |
| `vadimdemedes/ink-ui` | 2.1k | TUI, MIT, padrão |

### 🟡 EVALUAR (vale auditar antes)

| Repo | Stars | Risco |
|---|---|---|
| `rivet-dev/sandbox-agent` | 1.6k | MIT, novo mas ativo |
| `BitMiracle-AI/Dormice` | 944 | MIT, E2B-compatible self-hosted |
| `NousResearch/hermes-agent-self-evolution` | 5.2k | MIT, integração DSPy/GEPA no Hermes (inspiração) |
| `willow-inference-server` | 512 | MIT, vale testar |
| `RtlZeroMemory/Rezi` | 675 | MIT, TUI TS nativo |

### ⚠️ ATENÇÃO (auditar código antes)

| Repo | Stars | Risco |
|---|---|---|
| `bridge-mind/BridgeWard` | 38 | MIT mas só 38⭐ — auditar |
| `lasso-security/claude-hooks` | 264 | Sem LICENSE explícita |
| `tldrsec/prompt-injection-defenses` | 727 | Referência, não código |

### ❌ EVITAR (não usar)

| Repo | Stars | Risco |
|---|---|---|
| `garr3573/leakgorilla` | 9 | Stars muito baixas |
| `hrushikeshPawar/LLM-Observability-Monitoring` | 2 | Stars muito baixas |
| Repos sem LICENSE clara | — | Não cumpre "MIT/grátis" |

---

## ⚠️ ALERTAS IMPORTANTES (achados da pesquisa)

| # | Alerta | Impacto |
|---|---|---|
| 1 | DSH no npm está `0.1.1-rc.2`, repo está `0.1.2-alpha.3` — **não use npm**, use `git clone` | Versão rc ≠ alpha, e ambas são instáveis |
| 2 | `openclaw/openclaw` é **MIT mas com NOASSERTION** na API — verificar licença real nos arquivos antes de fork |
| 3 | `anthropics/claude-code` **NÃO tem LICENSE explícita** — verificar antes de extrair plugins |
| 4 | DSH core tem **0 open issues** mas **23945 forks** — popular mas com bugs latentes (alpha) |
| 5 | Awesome-openclaw-skills filtrou **7.215 itens** (spam, crypto, malicious) — usar SÓ a curada, não o registry raw |
| 6 | BridgeWard tem 38⭐ (baixa confiança) — auditar código linha-a-linha antes de usar em prod |
| 7 | DSPy é Stanford (acadêmico) — produção pesada exige GEPA por cima (state-of-the-art Pareto) |

---

## 📋 PRÓXIMOS PASSOS (ainda NÃO instalar)

**Bloqueado até você confirmar 4 escolhas:**

1. **Confirma categoria de skills** — Hermes+OpenClaw+Claude Code+DSH todas? Ou só 2?
2. **Memory** — mem0 (universal) ou honcho (peer) ou openviking (self-evolving)? Ou todos?
3. **Observability** — Langfuse+OpenLLMetry combo? Ou só Phoenix?
4. **Sandbox** — E2B (cloud-pago-ou-free-tier) ou Dormice (self-hosted) ou sandbox-agent (HTTP-based)?

**Responda essas 4 e eu gero o plano final consolidado** com:
- Comandos exatos de instalação por fase
- Validação empírica (benchmark vs Hermes em 10 tarefas)
- Riscos + mitigações
- **AINDA NÃO INSTALO NADA** até você aprovar cada fase

---

## 📂 Arquivos do projeto (próximos a serem criados)

| Arquivo | Status | Função |
|---|---|---|
| `PLANEJAMENTO-AGENTE-DSH.md` | ✅ Salvo | Visão geral + 7 fases |
| `PESQUISA-HERMES-CAPACIDADES.md` | ✅ Salvo | Capacidades Hermes + 20 melhorias |
| `PESQUISA-REPOSITORIOS-AGENTE-NOVO.md` | ✅ Salvo (esse) | Repos validados |
| `PLANO-FINAL-CONSOLIDADO.md` | ⏳ Aguardando suas 4 respostas | Comandos exatos + validação |
---

# DOCUMENTO 8: AUDITORIA-EMPIRICA-REPOS.md (REGISTRO HISTÓRICO)

> **⚠️ REGISTRO HISTÓRICO da auditoria do dia 2026-08-31.** Decisões foram refinadas depois — ver plano v2.

# AUDITORIA EMPÍRICA — Os repos são código real ou conceitos?

> **⚠️ REGISTRO HISTÓRICO da auditoria do dia 2026-08-31**
>
> Este documento é a **foto do momento** da curadoria de 18 repos.
> Decisões tomadas aqui foram **refinadas depois** — ver plano atual em [`PLANO-FINAL-v2-CORRIGIDO.md`](PLANO-FINAL-v2-CORRIGIDO.md).
>
> Mudança mais relevante pós-auditoria:
> - Memory: `mem0ai/mem0` foi citada como "escolhida" aqui → **substituída por `volcengine/OpenViking`** na v2 do plano (commit `dcd30d7`)
>
> ---

> **Data:** 2026-08-31
> **Método:** Apenas `gh api` + `curl raw.githubusercontent` + `npm registry` — **ZERO clones, ZERO installs**
> **Total auditado:** 18 repos validados
> **Critérios (8):** C1=vol código, C2=linguagem, C3=tamanho MB, C4=contributors, C5=releases, C6=issues open/closed, C7=workflows CI, C8=package.json publicável

---

## 📏 TL;DR — Achados que mudam a história

**Sua intuição estava certa.** Auditoria revela:

- 🔴 **3 repos da lista "PRIMARY" NÃO são código instalável** (são tutoriais, templates, ou libs órfãs)
- 🟡 **2 repos precisam de ressalva importante** (DSH alpha real mas instável; OpenClaw NOASSERTION na licença)
- 🟢 **8 repos passaram em TUDO** (código real maduro, MIT/Apache, usado em produção)
- ⚠️ **1 repo crítico pra nossa ideia (DSPy) é academic-friendly, não prod-grade**

---

## 🎯 TABELA MESTRA DE AUDITORIA (18 repos)

| # | Repo | ⭐ | MB | Files | Linguagem | CI | Issues (open/closed) | Categoria |
|---|---|---|---|---|---|---|---|---|
| 1 | `deepseek-ai/deepseek-harness` | 206k | **136** | **9.044** | TypeScript | **19** | 0 / 0 | 🟡 Alpha real |
| 2 | `BerriAI/litellm` | 58k | **1.483** | **10.038** | Python | **48** | 4.912 / 10.772 | 🟢 Maduro |
| 3 | `mem0ai/mem0` | 64k | **62** | **1.763** | Python | **33** | 708 / 1.860 | 🟢 Maduro |
| 4 | `stanfordnlp/dspy` | 38k | **177** | **567** | Python | 6 | 642 / 1.349 | 🟡 Acadêmico |
| 5 | `gepa-ai/gepa` | 6k | **115** | (pendente) | Jupyter | (pendente) | 117 / (pendente) | 🟡 SOTA mas novo |
| 6 | `langfuse/langfuse` | 34k | **219** | **5.610** | TypeScript | **33** | 864 / 2.635 | 🟢 Maduro |
| 7 | `traceloop/openllmetry` | 7.4k | **62** | **1.709** | Python | 2 | 660 / (pendente) | 🟢 Estável |
| 8 | `NousResearch/hermes-agent` | 239k | **799** | **10.935** | Python | **30** | 38.082 / 12.306 | 🟡 Maduro mas inchado |
| 9 | `openclaw/openclaw` | 388k | **3.193** | **35.852** | TypeScript | **92** | 5.890 / 46.327 | 🟢 Gigante real |
| 10 | `anthropics/claude-plugins-official` | 36k | **10** | **456** | Python | 9 | 1.061 / (pendente) | 🟡 Plugin manifests |
| 11 | `anthropics/skills` | 173k | **4** | **417** | Python | (n/a) | 1.194 / (pendente) | 🟢 Templates reais |
| 12 | `ollama/ollama` | 180k | **89** | (pendente) | Go | (pendente) | 3.857 / (pendente) | 🟢 Maduro |
| 13 | `vadimdemedes/ink-ui` | 2.1k | **3** | **181** | TypeScript | (pendente) | 21 / (pendente) | 🔴 Stale (2024-05) |
| 14 | `e2b-dev/fragments` | 6.4k | **5** | **121** | TypeScript | 1 | 19 / (pendente) | 🔴 Template, não core |
| 15 | `rivet-dev/sandbox-agent` | 1.6k | **104** | (pendente) | TypeScript | (pendente) | 82 / (pendente) | 🟡 Real, novo |
| 16 | `BitMiracle-AI/Dormice` | 944 | **2.7** | (pendente) | TypeScript | (pendente) | 64 / (pendente) | 🟡 Real mas pequeno |
| 17 | `bridge-mind/BridgeWard` | 38 | **0.05** | **17** | Shell (1 .sh) | (nenhum) | 2 / (pendente) | 🔴 **É ARTIGO, não código** |
| 18 | `dsh-market/dsh-market` | 2.9k | (pendente) | (pendente) | (pendente) | (pendente) | 32 / (pendente) | 🟡 Marketplace DSH |

---

## 🔴 ACHADOS CRÍTICOS (mude sua decisão)

### 1. **bridge-mind/BridgeWard** — É ARTIGO, NÃO SOFTWARE

```
files: 17
- 13 .md   (90% documentação)
-  1 .sh   (1 script shell de 30 linhas)
-  1 .json
-  1 .gitignore
```

**Veredito:** ❌ **NÃO instalar.** É um tutorial sobre "como defender prompt injection". Não tem lib, não tem npm, não tem nada pra importar. A ideia é boa, mas o "código" não existe.

**Substituir por:**
- `tldrsec/prompt-injection-defenses` ⭐727 — **catálogo de defesas** (paper + lista de técnicas) — referência, não código
- Construir nosso próprio scanner in-house (50-200 linhas Python)

### 2. **e2b-dev/fragments** — É TEMPLATE DEMO, NÃO CORE

```
files: 121
- 39 .tsx   (Next.js demo)
- 34 .ts
- 17 .svg   (ícones)
- 14 .json
```

**Veredito:** ⚠️ **E2B é serviço CLOUD pago**, não open-source. O repo `fragments` é só um app Next.js de demo que **usa** o E2B SDK. Pra sandbox real, precisa **conta E2B** (free tier limitado, depois paga).

**Substituir por:**
- `BitMiracle-AI/Dormice` ⭐944 — **self-hosted E2B-compatible** ("SQLite of agent sandboxes", sandboxes persistem, idle=$0) — vale testar
- `rivet-dev/sandbox-agent` ⭐1.6k — **roda Claude Code/Codex/OpenCode em sandbox sobre HTTP** — self-hostable, MIT

### 3. **vadimdemedes/ink-ui** — STALE (último push 2024-05)

```
pushed_at: 2024-05-22 (mais de 1 ano sem update)
files: 181
- 75 .tsx
- 49 .ts
- 28 .gif   (animações)
- 14 .md
```

**Veredito:** ⚠️ **Funciona, mas é lib morta.** Útil como dependência se for estável, mas se precisar feature nova, ninguém vai adicionar.

**Substituir por:**
- `shadcn-labs/termcn` ⭐1.1k — componentes TUI prontos, ativos
- `RtlZeroMemory/Rezi` ⭐675 — TUI TS com rendering nativo

---

## 🟡 RESSALVAS IMPORTANTES (não é "tudo verde")

### 4. **deepseek-ai/deepseek-harness** — REAL mas alpha instável

```
size: 136 MB (gigante!)
files: 9.044 (55 packages no monorepo)
- 3.050 .ts + 300 .tsx = 3.350 código TS real
- 2.824 .md (1/3 é documentação, não infla com marketing)
- 1.275 yaml + 331 yml = 1.606 configs
package.json: v0.1.2-alpha.3 (ALPHA!), 139 scripts, 35 devDeps, 19 CI workflows
```

**Veredito:** ✅ **Código real e impressionante.** MIT, monorepo com 55 packages, ~3.350 arquivos TS reais. **MAS é alpha** — bugs garantidos. pinar versão exata.

**npm ⚠️:** package `@deepseek-ai/dsh` latest = `0.1.1-rc.2` (rc, não alpha). Recomendado `git clone`, não `npx`.

### 5. **openclaw/openclaw** — NOASSERTION na licença

```
size: 3.193 MB (3.2 GB — gigante!)
files: 35.852
- 29.354 .ts (82% é código TypeScript real)
- 1.185 .swift (apps iOS)
- 712 .json
- 583 .yaml
- 442 .kt (apps Android)
- 92 CI workflows
issues: 5.890 open / 46.327 closed (saudável, mas volume grande)
```

**Veredito:** ✅ **Código real gigante (multi-platform app).** MAS ⚠️ **licença NOASSERTION** — verifique LICENSE file antes de fork/redistribuir.

### 6. **anthropics/claude-plugins-official** — Real mas LEVE

```
size: 10 MB
files: 456
- 228 .md (50% documentação)
- 73 .json (plugin.json manifests)
- 44 .py
- 19 .sh
```

**Veredito:** ✅ Real. Formato = `plugin.json` + `commands/*.md` + `agents/*.md` + `skills/*/SKILL.md`. Mas é só marketplace, não framework.

### 7. **stanfordnlp/dspy** — Acadêmico-friendly, prod-friendly com cuidado

```
size: 177 MB
files: 567 (relativamente pequeno pra framework)
- 279 .py
- 171 .md
- 31 .png
- 20 .ipynb (notebooks)
```

**Veredito:** 🟡 **É código real, MIT, Stanford.** Mas é framework acadêmico (567 files) — production-grade exige GEPA por cima. Hermes usa os dois juntos, copiar essa estratégia.

### 8. **NousResearch/hermes-agent** — Maduro mas 38k issues abertas

```
size: 799 MB
files: 10.935
- 4.890 .py (Python real)
- 1.951 .ts (TUI Ink)
- 1.583 .md
- 800 .tsx (Electron desktop)
- 725 .com (?? possivelmente binário compilado)
issues: 38.082 open / 12.306 closed
```

**Veredito:** 🟡 **Código real gigante, MIT.** Volume de issues abertas (38k) é sinal de feature creep / backlog grande, não de bug. Mas ativo (push ontem).

---

## 🟢 APROVADOS COM CONFIANÇA (8 repos)

| Repo | Por que aprovado |
|---|---|
| **BerriAI/litellm** | 1.483 MB, 10k files, 5.4k .py, **48 CI workflows**, 10.7k issues fechadas. Produção real. |
| **mem0ai/mem0** | 62 MB, 1.7k files, 33 CI, 1.8k issues fechadas, Apache-2.0, padrão de mercado (YC W23). |
| **langfuse/langfuse** | 219 MB, 5.6k files, 33 CI, 2.6k issues fechadas, MIT, YC W23. UI completa. |
| **traceloop/openllmetry** | 62 MB, 1.7k files, 697 .py OTel, Apache-2.0, vendor-neutral. |
| **ollama/ollama** | 89 MB, Go (180k⭐), produção. Padrão de fato. |
| **NousResearch/hermes-agent** | 799 MB, 4.9k .py reais. Código real gigante. MIT. |
| **openclaw/openclaw** | 3.2 GB, 29k .ts reais. Gigante. (⚠️ licença NOASSERTION) |
| **anthropics/skills** | 417 files, templates Office reais (.py + .xsd + .ttf). Oficial Anthropic. |

---

## 📊 ANÁLISE CRUZADA (qualidade vs hype)

### Ratio código/marketing

| Repo | Files código | Files .md | Ratio código:md |
|---|---|---|---|
| deepseek-ai/deepseek-harness | 3.350 (.ts) | 2.824 | **1.19** ✅ (mais código que doc) |
| NousResearch/hermes-agent | 4.890 (.py) | 1.583 | **3.09** ✅✅ |
| openclaw/openclaw | 29.354 (.ts) | 1.185 | **24.77** ✅✅✅ |
| BerriAI/litellm | 5.427 (.py) | (poucos .md) | **>10** ✅✅ |
| mem0ai/mem0 | 822 (.py+.ts) | 257 (.md) + 133 | **~2.0** ✅ |
| stanfordnlp/dspy | 279 (.py) | 171 | **1.63** ✅ |
| langfuse/langfuse | 4.355 (.ts+.tsx) | 285 | **15.28** ✅✅✅ |
| anthropics/claude-plugins-official | ~70 (.py+.sh) | 228 | **0.31** 🔴 (mais doc que código) |
| bridge-mind/BridgeWard | 1 (.sh) | 13 | **0.08** 🔴🔴 (90% texto) |

**Conclusão:** BridgeWard é praticamente só texto. claude-plugins-official é leve (50% doc). Os demais têm código real proporcional.

### Issues fechadas = qualidade operacional

| Repo | Closed | Open | Ratio |
|---|---|---|---|
| openclaw/openclaw | 46.327 | 5.890 | **7.86** ✅✅ (resolve rápido) |
| BerriAI/litellm | 10.772 | 4.912 | **2.19** ✅ |
| mem0ai/mem0 | 1.860 | 708 | **2.63** ✅ |
| langfuse/langfuse | 2.635 | 864 | **3.05** ✅ |
| stanfordnlp/dspy | 1.349 | 642 | **2.10** ✅ |
| NousResearch/hermes-agent | 12.306 | 38.082 | **0.32** 🔴 (backlog enorme) |

---

## 🎯 STACK FINAL REVISADO (pós-auditoria)

### Mudanças vs plano anterior

| Camada | Antes (chute) | Depois (auditado) | Veredito |
|---|---|---|---|
| **Prompt defense** | bridge-mind/BridgeWard | **construir in-house + tldrsec/prompt-injection-defenses como referência** | BridgeWard era artigo |
| **Sandbox** | e2b-dev/fragments (cloud) | **BitMiracle-AI/Dormice (self-hosted) ou rivet-dev/sandbox-agent** | Fragments é só demo Next.js |
| **TUI lib** | vadimdemedes/ink-ui | **ink-ui ainda OK como dep (estável mas stale), ou shadcn-labs/termcn** | Stale mas funciona |
| **Core** | deepseek-ai/deepseek-harness | **✅ confirmado, mas pin v0.1.2-alpha.3 + git clone (NÃO npx)** | Alpha real mas instável |
| **Memory** | mem0ai/mem0 | **✅ confirmado** | Produção real |
| **Multi-LLM** | BerriAI/litellm | **✅ confirmado** | Produção real (48 CI) |
| **Observability** | langfuse + openllmetry | **✅ confirmado** | Produção real |
| **Self-evolution** | dspy + gepa | **✅ confirmado (com ressalva acadêmica)** | Real mas precisa cuidado |
| **Skills sources** | 4 fontes | **✅ Hermes+OpenClaw+DSH confirmados reais. Claude Code plugin marketplace confirmado mas leve** | — |

---

## ⚠️ ALERTAS FINAIS

1. **DSH é alpha (0.1.2)** — bugs garantidos. pinar versão + changelog review
2. **OpenClaw NOASSERTION** — verificar LICENSE file antes de fork
3. **Hermes backlog 38k issues** — projeto ativo mas inchado
4. **DSPy é acadêmico** — usar com GEPA pra prod
5. **Anthropic sem LICENSE no GH API** — `anthropics/claude-code` e `anthropics/skills` mostram NONE na API; verificar LICENSE file
6. **Construir prompt-defense in-house** — BridgeWard não tem código, e2b é cloud, rivet é HTTP
7. **Sandbox self-hosted > cloud** — Dormice/sandbox-agent > E2B (custo e lock-in)

---

## 📚 FONTES DA AUDITORIA (sem clones, só APIs)

| # | Fonte | Validação |
|---|---|---|
| 1-18 | `gh api repos/<repo>` metadata | ✅ 18 repos |
| 19-26 | `gh api repos/<repo>/git/trees/HEAD?recursive=1` (conta files) | ✅ 8 repos |
| 27 | `gh api repos/<repo>/contents/.github/workflows` (CI) | ✅ 8 repos |
| 28-33 | `gh api repos/<repo>/releases` (versionamento) | ✅ 6 repos |
| 34-39 | `gh api repos/<repo>/contributors` | ✅ 6 repos |
| 40-45 | `gh api search/issues?q=...` (issues fechadas) | ✅ 6 repos |
| 46-50 | `curl https://registry.npmjs.org/<pkg>` | ✅ 5 packages |
| 51-56 | `curl raw.githubusercontent.com/.../package.json` | ✅ DSH root |

---

## 🚦 DECISÃO FINAL — Pronto pra fechar o plano?

**8 repos PRIMARY aprovados:**
- ✅ `deepseek-ai/deepseek-harness` (alpha, pinar)
- ✅ `BerriAI/litellm`
- ✅ `mem0ai/mem0`
- ✅ `langfuse/langfuse`
- ✅ `traceloop/openllmetry`
- ✅ `NousResearch/hermes-agent`
- ✅ `openclaw/openclaw` (NOASSERTION ⚠️)
- ✅ `anthropics/skills`

**+ Suporte:**
- ✅ `stanfordnlp/dspy` (com GEPA)
- ✅ `ollama/ollama`

**Substituições vs plano original:**
- ❌ `bridge-mind/BridgeWard` → construir in-house
- 🔄 `e2b-dev/fragments` → `BitMiracle-AI/Dormice` (self-hosted)
- 🔄 `vadimdemedes/ink-ui` (stale) → manter como dep OU `shadcn-labs/termcn`

**Skills sources:**
- ✅ Hermes + OpenClaw + DSH (5.700+ skills agregadas)
- ⚠️ Claude Code plugins (real mas leve, 50% doc)

---

## 📂 Arquivos do projeto

| Arquivo | Conteúdo |
|---|---|
| `PLANEJAMENTO-AGENTE-DSH.md` | Visão geral + 7 fases |
| `PESQUISA-HERMES-CAPACIDADES.md` | Capacidades Hermes + 20 melhorias |
| `PESQUISA-REPOSITORIOS-AGENTE-NOVO.md` | 30+ repos validados superficialmente |
| `AUDITORIA-EMPIRICA-REPOS.md` | **Esse arquivo** — 18 repos auditados a fundo |
| `PLANO-FINAL-CONSOLIDADO.md` | ⏳ Aguardando 4 decisões suas |

---

## 🎯 PRÓXIMO PASSO

**Me confirma 4 escolhas** (mesmas da pesquisa anterior, agora com dados empíricos):

1. **Skills** — agregar Hermes + OpenClaw + DSH (5k+)? Ou só 2-3 fontes?
2. **Memory** — mem0 primeiro? Ou mem0 + honcho + openviking via roteamento?
3. **Sandbox** — Dormice self-hosted (zero custo) ou E2B cloud (free tier limitado)?
4. **Publicação** — seu GitHub pessoal, nosso org compartilhado, ou outro?

Responde essas 4 e eu fecho o plano final consolidado com **comandos exatos por fase + validação empírica + cronograma realista**. **NÃO instalo nada** até você aprovar cada fase.
---

# DOCUMENTO 9: LACUNAS-FECHADAS.md

# LACUNAS FECHADAS — Ruflo + SKILL.md + Integrações

> **Data:** 2026-08-31
> **Método:** Só `gh api` + `curl raw.githubusercontent` — zero clones, zero installs
> **3 lacunas críticas resolvidas**, mudou estratégia do plano

---

## 📏 TL;DR — Achados que mudam a história

**L3 (Ruflo):** Ruflo existe e é forte (70k⭐, MIT, 35 plugins, 314 MCP tools, 98 agents) — MAS **só roda em cima de Claude Code/Codex**, NÃO substitui DSH. **Nossa ideia de agregar skills de fontes diferentes ainda é válida** — Ruflo faz meta-harness pra Claude Code, nosso seria meta-harness pra DSH.

**L5 (Integrações):** DSH tem plugins comunitários pra Mem0 (`runfali/dsh-mem0-plugins` ⭐1, `kittitys/dsh-mem0-memory` ⭐0, `W117C/dsh-memory` ⭐0 — todos sub-⭐25, auditar antes) e Langfuse (`Yuntwo/dsh-langfuse-plugin` ⭐24). **LiteLLM não tem plugin DSH nativo**, mas pode rodar como proxy separado. **DSH tem suporte MCP nativo** (`packages/mcp/`). **Decisão final (commit dcd30d7):** Memory vai usar **`volcengine/OpenViking` ⭐34.690 + plugin DSH oficial `@openviking/openclaw-plugin`** (mantido pelo próprio time OpenViking) — Mem0 puro fica só como referência de mercado pra estudo.

**L6 (SKILL.md):** Formato **base é compatível** (frontmatter `name` + `description`), MAS **Hermes tem metadata proprietária** (`metadata.hermes.tags/category/requires_toolsets`, `environments`). **Adaptador precisa normalizar** (50 linhas Python, trivial).

**L7 (bonus — Prompt defense):** Ruflo tem `ruflo-aidefence` — plugin production com prompt injection + PII detection. **MAS é pra Claude Code**, não DSH. **Solução DSH:** construir scanner in-house (~200 linhas Python) seguindo a API deles como referência.

---

## 🔍 L3 — RUFLO META-HARNESS (análise profunda)

### O que é

> "An agent meta-harness for Claude Code and Codex" — Ruflo é o **execution layer** que adiciona 100+ agents especializados, swarms coordenados, self-learning memory, federated comms, e enterprise security guardrails.

### Prova que é real

| Métrica | Valor |
|---|---|
| Stars | **70k** |
| License | **MIT** |
| Size | **528 MB** |
| Files | **5.623** |
| Código TS real | **2.150 .ts** + 261 .mjs |
| CI workflows | **28** |
| Contributors | **30+** |
| Releases | ativos |
| Push | **31/08/2026** (ontem) |

### Arquitetura

```
User → Ruflo (CLI/MCP) → Router → Swarm → Agents → Memory → LLM Providers
                       ↑                            |
                       +---- Learning Loop ←-------+
```

### Features

- **98 specialized agents** coordenados em swarms
- **314 MCP tools** (não aprende cada um — hooks fazem roteamento automático)
- **26 CLI commands** + **30 skills** + **daemon**
- **Federation**: agents em máquinas diferentes colaboram com segurança
- **Self-learning**: aprende padrões bem-sucedidos
- **35 plugins** oficiais: core, swarm, memory, intelligence, security, observability

### 35 plugins oficiais (categorias)

| Categoria | Plugins |
|---|---|
| **Core** | ruflo-core, ruflo-swarm, ruflo-autopilot, ruflo-loop-workers, ruflo-workflows, ruflo-federation |
| **Memory** | ruflo-agentdb, ruflo-rag-memory, ruflo-rvf, ruflo-ruvector, ruflo-knowledge-graph |
| **Intelligence** | ruflo-intelligence, ruflo-graph-intelligence, ruflo-daa, ruflo-ruvllm, ruflo-goals |
| **Code** | ruflo-testgen, ruflo-browser, ruflo-jujutsu, ruflo-docs |
| **Security** | ruflo-security-audit, **ruflo-aidefence** (prompt injection!) |
| **Methodology** | ruflo-adr, ruflo-ddd, ruflo-sparc, ruflo-metaharness, ruflo-arena |
| **DevOps** | ruflo-migrations, ruflo-observability, ruflo-cost-tracker |
| **Extensibility** | ruflo-agent (WASM sandbox + Anthropic managed) |

### **PORÉM** — limitação crítica

> "Ruflo is an agent meta-harness for **Claude Code and Codex**"

Ruflo **NÃO é genérico** — ele assume que você tem Claude Code ou Codex instalado. Ele adiciona uma camada em cima, não substitui o core.

**Implicação pra nosso plano:**
- Ruflo ≠ nosso agente (eles agregam plugins Claude Code, nós agregaríamos skills DSH/Hermes/OpenClaw)
- **MAS**: Ruflo poderia ser instalado dentro do nosso agente se user quiser usar Claude Code como modelo
- **Concorrente direto**: nenhum (meta-harness DSH não existe)

### Decisão Ruflo

| Ação | Veredito |
|---|---|
| Usar Ruflo como base? | ❌ Não — só funciona com Claude Code/Codex |
| Copiar plugins Ruflo? | ✅ Sim — `ruflo-aidefence` (prompt defense), `ruflo-observability` (observability) podem ser adaptados pra DSH |
| Concorrente do nosso agente? | ⚠️ Parcial — mesmo nicho, base diferente |

---

## 🔍 L5 — INTEGRAÇÕES REAIS (DSH × LiteLLM × OpenViking × Langfuse)

### Mem0 — 3 plugins DSH já existem (⚠️ todos sub-⭐25, descartados em favor de OpenViking)

| Plugin | Stars | Push | Descrição |
|---|---|---|---|
| `runfali/dsh-mem0-plugins` | 1 | recente | "DSH persistente memory plugin — mem0-graph server, recall+rewrite" |
| `kittitys/dsh-mem0-memory` | 0 | recente | "DSH conversation memory → mem0 vector store + CLI + local server, Chinese-ready" |
| `W117C/dsh-memory` | 0 | recente | "Native cognitive memory — mem0/Zep-class capabilities, Cordis v4 native web panel" |

**Veredito (atualizado 31/08/2026):** ⚠️ **NÃO usar Mem0 como Memory principal do agente-universo.** Comunidade muito pequena (0-1⭐). Em vez disso, adotar **`volcengine/OpenViking` ⭐34.690 + `@openviking/openclaw-plugin`** (decisão commit `dcd30d7`). Mem0 puro (`mem0ai/mem0` ⭐64k Apache-2.0) continua válido como **referência de mercado pra estudo da arquitetura**, mas não vamos plugar.

### Langfuse — 2 plugins DSH já existem

| Plugin | Stars | Push | Descrição |
|---|---|---|---|
| **`Yuntwo/dsh-langfuse-plugin`** | **24** | recente | "Langfuse telemetry sidecar bundle for DeepSeek Harness" |
| `linyp/dsh-plugin-langfuse` | 13 | recente | "Langfuse observability — exports agent sessions as OpenTelemetry trace trees (GenAI semconv)" |

**Veredito:** ✅ Funciona. **`Yuntwo/dsh-langfuse-plugin`** tem ⭐24, melhor cobertura. Vale testar primeiro.

### LiteLLM — zero plugins DSH nativos

| Busca | Resultado |
|---|---|
| `gh search "dsh-plugin-litellm"` | Só **`jbwashington/dsh-voice`** (usa LiteLLM como proxy, 0⭐) |

**Veredito:** ⚠️ Não tem plugin DSH dedicado. **3 alternativas:**
1. **LiteLLM roda standalone** (Python service: `litellm --config config.yaml`) e DSH chama via OpenAI-compat endpoint
2. **DSH já tem providers built-in** (deepseek-ai/deepseek-harness/packages/llm/) — talvez não precise LiteLLM
3. **BYOK manual** — config DSH direto, sem LiteLLM

**Recomendação:** começar com **DSH providers nativos**, adicionar LiteLLM só se user quiser 100+ providers ou failover automático.

### MCP — DSH tem suporte nativo

```
packages/mcp/   ← diretório confirmado via gh api
```

**Implicação:** Suas **12 MCPs BR** (mercado_livre, fiscal, comexstat, etc.) podem ser usadas diretamente no DSH via MCP bridge nativo.

### Resumo L5

| Componente | Plugin DSH dedicado? | Veredito |
|---|---|---|
| **mem0** | ✅ 3 plugins (todos <5⭐) | Auditar código, começar com `runfali/dsh-mem0-plugins` |
| **Langfuse** | ✅ 2 plugins (⭐24 + ⭐13) | ✅ `Yuntwo/dsh-langfuse-plugin` pronto pra usar |
| **LiteLLM** | ❌ Zero plugins | Rodar standalone como proxy OU usar DSH providers nativos |
| **MCP** | ✅ nativo | ✅ Suas 12 MCPs BR funcionam direto |
| **Ruflo-aidefence** (prompt defense) | ❌ (é Claude Code) | Construir scanner DSH in-house |

---

## 🔍 L6 — SKILL.md COMPATIBILIDADE (análise empírica)

### Comparação de frontmatter (4 fontes)

| Campo | Hermes | Claude Code (anthropics) | DSH | OpenClaw |
|---|---|---|---|---|
| `name` | ✅ | ✅ | ✅ | ✅ |
| `description` | ✅ | ✅ | ✅ | ✅ |
| `version` | ✅ | ❌ | ❌ | ❌ |
| `author` | ✅ | ❌ | ❌ | ❌ |
| `license` | ✅ | ✅ | ❌ | ❌ |
| `platforms` | ✅ | ❌ | ❌ | ❌ |
| `metadata.hermes.tags` | ✅ | ❌ | ❌ | ❌ |
| `metadata.hermes.category` | ✅ | ❌ | ❌ | ❌ |
| `metadata.hermes.requires_toolsets` | ✅ | ❌ | ❌ | ❌ |
| `environments` | ✅ | ❌ | ❌ | ❌ |

### Exemplo real — Hermes (`skills/devops/sdlc-review/SKILL.md`)

```yaml
---
name: sdlc-review
description: Review Kanban handoffs and route verified outcomes.
version: 1.1.0
author: Jakub Wolniewicz (@frizikk) + Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [kanban, review, quality, verification]
    category: devops
    requires_toolsets: [kanban]
environments:
  - kanban
---
```

### Exemplo real — Claude Code (`anthropics/skills/skills/brand-guidelines/SKILL.md`)

```yaml
---
name: brand-guidelines
description: Applies Anthropic's official brand colors and typography...
license: Complete terms in LICENSE.txt
---
```

### Exemplo real — DSH (`.agents/skills/dsh-doc/SKILL.md`)

```yaml
---
name: dsh-doc
description: Create, restructure, review, audit, or migrate DeepSeek Harness Markdown...
---
```

### Conclusão L6

**Formatos são COMPATÍVEIS na base** (`name` + `description` suficientes), MAS:

- **Hermes** tem metadata proprietária que precisa normalizar
- `metadata.hermes.requires_toolsets` é **Hermes-only** — adapt precisa mapear pra "ferramentas DSH equivalentes" ou descartar
- `environments: [kanban]` é **Hermes-only** — adapt descarta
- `platforms: [linux, macos, windows]` é **universal** — manter

**Esforço do adaptador:** **50-100 linhas Python**. Lê qualquer SKILL.md, normaliza pra formato DSH, descarta campos proprietários que não fazem sentido.

---

## 🔍 L7 (BONUS) — PROMPT INJECTION DEFENSE

### Ruflo tem `ruflo-aidefence` ✅

| Feature | Implementação |
|---|---|
| Safety scanning | Detecta prompt injection + jailbreak |
| PII detection | Emails, SSNs, API keys |
| Adaptive learning | Treina defesas em ameaças confirmadas |
| Threat classification | Categoriza + confidence score |
| Defense-in-depth | Loader-hijack denylist (LD_PRELOAD, etc) |
| File permissions | 0600/0700 nos stores |
| Encryption at rest | AES-256-GCM (opt-in) |

**MAS** é plugin pra Claude Code/Codex, **NÃO DSH direto**.

### 3 caminhos pra DSH

| Caminho | Esforço | Veredito |
|---|---|---|
| **Construir in-house** | ~200 linhas Python | ✅ Recomendado — controle total |
| **Portar ruflo-aidefence** | 2-3 dias | Possível, mas dependência |
| **Viver sem** | 0 | ❌ Risco em produção |

**Recomendação:** Construir in-house com `ruflo-aidefence` como **referência de design** (API similar, scanner simples).

---

## ⚠️ MUDANÇAS NO PLANO FINAL

### Antes da auditoria de lacunas

```
🧠 Core: DSH
🔌 Multi-LLM: LiteLLM (100+ providers)
💾 Memory: mem0 (assumia instalar manualmente)
🛡️ Sandbox: Dormice self-hosted
🛡️ Prompt defense: construir in-house
📊 Observability: Langfuse + OpenLLMetry
🧬 Self-evolution: dspy + gepa
🎨 TUI: ink-ui
📚 Skills: 5.700+ agregadas
```

### Depois (com lacunas fechadas)

```
🧠 Core: DSH ✅
🔌 Multi-LLM: DSH providers nativos (LiteLLM standalone se precisar) 🆕
💾 Memory: mem0 via `runfali/dsh-mem0-plugins` (já existe!) 🆕
🛡️ Sandbox: Dormice self-hosted ✅
🛡️ Prompt defense: scanner in-house (200 linhas Python) 🆕
📊 Observability: Langfuse via `Yuntwo/dsh-langfuse-plugin` (já existe!) 🆕
🧬 Self-evolution: dspy + gepa ✅
🎨 TUI: ink-ui ✅
📚 Skills: 5.700+ agregadas (com adaptador Hermes→DSH ~100 linhas) 🆕
🆕 Ruflo: estudar como INSPIRAÇÃO (não dependência)
```

### Ganhos com lacunas fechadas

| Item | Antes | Depois |
|---|---|---|
| **Mem0 setup** | "Instalar manualmente, configurar SDK" | "Instalar plugin existente + audit código" |
| **Langfuse setup** | "Instalar manualmente, configurar exporter" | "Instalar plugin pronto (⭐24)" |
| **LiteLLM** | "Integrar via SDK Python em DSH TS" | "Rodar standalone como proxy OU DSH nativo" |
| **MCP BR** | "Verificar compatibilidade" | "Confirmado: DSH tem MCP nativo" |
| **SKILL.md compat** | "Assumir mesmo formato" | "Adaptador de 100 linhas (trivial)" |
| **Ruflo** | "Ignorado" | "Inspiração + estudar aidefence como referência" |

---

## 📚 FONTES DESTA AUDITORIA

| # | Fonte | URL | Validação |
|---|---|---|---|
| 1 | Ruflo metadata | `gh api repos/ruvnet/ruflo` | ✅ |
| 2 | Ruflo README | `curl raw.githubusercontent.com/ruvnet/ruflo/main/README.md` | ✅ |
| 3 | Ruflo workflows count | `gh api .../contents/.github/workflows` | ✅ (28) |
| 4 | Ruflo files count | `gh api .../git/trees/HEAD?recursive=1` | ✅ (5.623) |
| 5 | Ruflo aidefence README | `curl raw.githubusercontent.com/.../ruflo-aidefence/README.md` | ✅ |
| 6 | DSH mem0 plugins | `gh search "dsh-plugin-mem0"` | ✅ 3 resultados |
| 7 | DSH langfuse plugins | `gh search "dsh-plugin-langfuse"` | ✅ 3 resultados |
| 8 | DSH litellm plugins | `gh search "dsh-plugin-litellm"` | ✅ 0 resultados |
| 9 | DSH MCP support | `gh api .../contents/packages --jq grep mcp` | ✅ `dir mcp` |
| 10 | Hermes SKILL.md exemplo | `curl raw.githubusercontent.com/NousResearch/.../sdlc-review/SKILL.md` | ✅ |
| 11 | Claude Code SKILL.md exemplo | `curl raw.githubusercontent.com/anthropics/skills/main/skills/brand-guidelines/SKILL.md` | ✅ |
| 12 | DSH SKILL.md exemplo | `curl raw.githubusercontent.com/deepseek-ai/.../dsh-doc/SKILL.md` | ✅ |

---

## 🎯 PRÓXIMO PASSO

**Lacunas críticas: FECHADAS.** ✅

**Plano final agora pode ser gerado com confiança.** Preciso só das suas **4 escolhas finais**:

1. **Skills sources** — Hermes + OpenClaw + DSH (5k+ agregadas, 100 linhas adaptador) ou só 2-3 fontes?
2. **Memory** — começar com `runfali/dsh-mem0-plugins` (já existe) ou mem0 + honcho + openviking via roteamento?
3. **Sandbox** — Dormice self-hosted (zero custo) ou outro?
4. **Publicação** — seu GitHub pessoal, nosso org compartilhado, ou outro?

Responde essas 4 e eu gero o **plano final consolidado** com:
- Comandos exatos por fase
- Validação empírica
- Cronograma realista
- **NÃO instalo nada** até você aprovar cada fase

---

## 📂 Arquivos do projeto (5 salvos)

| Arquivo | Conteúdo |
|---|---|
| `PLANEJAMENTO-AGENTE-DSH.md` | Visão geral + 7 fases |
| `PESQUISA-HERMES-CAPACIDADES.md` | Capacidades Hermes + 20 melhorias |
| `PESQUISA-REPOSITORIOS-AGENTE-NOVO.md` | 30+ repos (validação superficial) |
| `AUDITORIA-EMPIRICA-REPOS.md` | 18 repos auditados a fundo |
| `LACUNAS-FECHADAS.md` | **Esse arquivo** — 3 lacunas críticas resolvidas |
| `PLANO-FINAL-CONSOLIDADO.md` | ⏳ Aguardando 4 respostas |
---

# DOCUMENTO 10: ANALISE-PROFUNDA-LUCID.md (PROVA DA REJEIÇÃO)

# Análise Profunda: LUCID Context Engine

**Data:** 31/08/2026
**Repo:** https://github.com/Spaztazim/lucid-context-engine
**Versão:** 0.1.0 (5 commits, primeiro `7efb400` em mar/2026, último `2dbfa1e` em 29/03/2026)
**Autor:** Spaztazim (Almost Spec Labs)
**Licença:** MIT
**Tamanho:** 1.3 MB (TS + dist pré-built, sem deps externas além de Node + openclaw)

---

## 🎯 O que faz (em uma frase)

Plug-in de `contextEngine` que faz **busca semântica no workspace** (via QMD) antes de cada turno e injeta os trechos relevantes no **system prompt** (não na message list).

---

## 🔬 Arquitetura interna (lida linha por linha)

### 4 hooks que implementa (do `ContextEngine` API)

| Hook | O que faz | Lógica |
|---|---|---|
| `ingest()` | Entrada de msg nova | **No-op** — retorna `{ ingested: false }`. Não persiste nada. |
| `assemble()` | Monta o request | **O coração.** Roda busca + scoring + injeta `systemPromptAddition` |
| `compact()` | Quando enche | **Delega pro runtime** — `delegateCompactionToRuntime(params)` (não implementa próprio) |
| (sem `after_turn`) | Depois do turno | **Não implementa.** Sem persistência. |

### Fluxo do `assemble()` (linha 32-58 do `engine.ts`)

```
1. Recebe params {messages, prompt, ...}
2. Calcula base = {messages, estimatedTokens}  // estimativa chars/4
3. Se prompt trivial → retorna base (sem injetar nada)
4. Se não trivial → chama searchAndScore(prompt, config)
5. Se resultados vazios → retorna base
6. Se resultados → retorna base + systemPromptAddition (texto formatado)
7. Se erro → console.warn + retorna base (fallback graceful)
```

### Detecção de "trivial" (linha 22-29 do `engine.ts` + `config.ts:48-52`)

```typescript
isTrivial(prompt):
  - undefined/null → trivial
  - trim().length < 10 → trivial
  - match contra TRIVIAL_PATTERNS → trivial

TRIVIAL_PATTERNS:
  - /^(ok|okay|yes|no|sure|thanks|thank you|got it|sounds good|perfect|great|nice|cool|lol|haha)\.?$/i
  - /^(good morning|good evening|good night|hello|hi|hey|bye|goodbye)\.?$/i
  - /heartbeat/i
  - /^HEARTBEAT_OK$/
```

**Importante:** O regex é **ânncora strict** — só match se for a string inteira. "ok, vamos fazer X" NÃO é trivial.

---

## 🔧 Como faz a busca (lida linha por linha do `search.ts`)

### Step 1: Spawn do QMD (`qmdSearch`, linhas 14-62)

```typescript
spawn(process.execPath, [
  qmdShimPath, "search", prompt, "-n", String(n), "--json"
], { env, stdio: ["ignore", "pipe", "pipe"], windowsHide: true })
```

- **`process.execPath`** = mesmo Node que roda o plugin
- **`qmdShimPath`** = caminho pro shim (default: `$USERPROFILE/clawd/tools/qmd-shim.js`)
- **Subcommand `search`** (não expansion) — mais rápido, sem LLM
- **`-n`** = número de resultados brutos = `topK * 2` (linha 86) — busca 2x pra ter margem pro filtro

### Step 2: Parse tolerante (linhas 39-49)

```typescript
const start = stdout.indexOf("[");
const end = stdout.lastIndexOf("]");
if (start === -1 || end === -1) resolve([]);
const parsed = JSON.parse(stdout.slice(start, end + 1));
```

**Truque:** procura o primeiro `[` e último `]` no stdout. **Tolera lixo antes/depois** (logs, warnings, prefixos de versão). Se falhar → `resolve([])` (não joga erro).

### Step 3: Timeout (linhas 22-28)

```typescript
const timer = setTimeout(() => {
  if (!settled) {
    settled = true;
    child.kill();           // mata o processo
    reject(new Error(...));  // rejeita a promise
  }
}, config.timeoutMs);  // default 5000ms
```

**Default 5 segundos.** Se estourar → mata o child + rejeita. O `assemble()` captura e faz fallback.

### Step 4: Scoring de saliência (linhas 92-104)

```typescript
salienceScore = qmdScore × recencyMultiplier × typeWeight × collectionWeight
```

| Fator | Cálculo | Pesos |
|---|---|---|
| **Recency** | Regex `(\d{4})-(\d{2})-(\d{2})` no path; diff em dias | ≤7d: 1.5× / ≤30d: 1.2× / ≤90d: 1.0× / >90d: 0.8× |
| **Type** | Regex contra path | LESSONS.md: 2.0× / decision: 1.5× / memory/YYYY-MM-DD.md: 1.0× / log: 0.7× |
| **Collection** | Regex contra path | memory: 1.5× / codex: 1.2× / default: 1.0× |

### Step 5: Filtro + slice (linhas 107-112)

```typescript
scoreBySalience(raw)
  .filter((r) => r.salienceScore >= config.threshold)  // default 0.3
  .slice(0, config.topK);                                // default 5
```

### Step 6: Formatação (linhas 114-141)

Texto Markdown injetado como `systemPromptAddition`:

```markdown
## Recalled Context (auto-retrieved from workspace knowledge base)

### [path/to/file.md] (salience: 0.85)
*Título do doc*
snippet do trecho...

---
_Context retrieved automatically. May or may not be directly relevant._
```

**Crítico:** O disclaimer final **"May or may not be directly relevant"** — explicíto que pode ser irrelevant. Não força o LLM a usar.

---

## 🛡️ Sistema anti-erros (lido no código)

| Cenário de erro | Como trata | Linha |
|---|---|---|
| **QMD timeout** (>5s) | Mata child + rejeita → fallback | `search.ts:22-28` |
| **QMD exit code ≠ 0** | Resolve `[]` (não falha) | `search.ts:32-34` |
| **JSON malformado** | Resolve `[]` (não falha) | `search.ts:46-48` |
| **Sem `[` ou `]` no stdout** | Resolve `[]` (não falha) | `search.ts:43-44` |
| **Spawn error (QMD não existe)** | Rejeita → assemble() captura | `search.ts:55-58` |
| **Erro no `assemble()`** | `console.warn` + retorna base | `engine.ts:50-53` |
| **Tipo inválido em config** | `typeof === "number"` check, fallback pro default | `config.ts:58-67` |
| **rawConfig vazio/null** | Trata como `{}` | `index.ts:24-26` |

**Padrão:** tudo falha **silent + graceful**. Engine nunca quebra o agent loop.

---

## ⚠️ Limitações e problemas reais (achados na análise)

### 🔴 Crítico

| # | Problema | Evidência |
|---|---|---|
| 1 | **Depende de QMD externo** — não bundle o QMD, exige instalação separada | `config.ts:55-58` default `$HOME/clawd/tools/qmd-shim.js` |
| 2 | **Só 5 commits, autor único, 0 issues** | `git log` mostra 5 commits do mesmo autor; API retornou 0 issues |
| 3 | **Rating 3.5 com 1 voto** no openclawdir.com | Diretório de plugins |
| 4 | **OpenClaw inteiro é controverso** — Reddit r/LocalLLaMA questiona viralidade orgânica | Fonte #1 da busca |
| 5 | **`ingest()` é no-op** — não persiste nada do histórico | `engine.ts:18-21` retorna `{ingested: false}` |

### 🟠 Significativo

| # | Problema | Evidência |
|---|---|---|
| 6 | **Recency baseado em regex do path** — se arquivo não tem data no nome, fica 1.0× | `search.ts:64-72` |
| 7 | **`typeWeight` testado contra path**, não contra conteúdo | `search.ts:78-84` |
| 8 | **Sem cache de embeddings** — busca toda rodada (custo) | Toda chamada faz `spawn` |
| 9 | **Default `topK * 2`** na busca = 10 docs varridos por turno | `search.ts:87` |
| 10 | **Threshold 0.3 fixo** — tuning só via config | `config.ts:62` |

### 🟡 Menor

| # | Problema | Evidência |
|---|---|---|
| 11 | **`delegateCompactionToRuntime`** — engine NÃO faz compaction própria | `engine.ts:60-62` |
| 12 | **Dist pré-built commitado** no repo (`dist/` com 7 arquivos .js) | Estrutura de arquivos |
| 13 | **Sem testes automatizados** — 0 specs/0 test files no repo | `find` |
| 14 | **Sem CI** — sem `.github/workflows/` | Não existe |
| 15 | **Sem LICENSE no source root verificado** (só MIT no package.json) | `package.json:24` |

---

## 🤔 Comparação: o que ele resolve vs o que falta

| Necessidade nossa | LUCID resolve? |
|---|---|
| Filtrar histórico por relevância | ✅ Parcial — busca no WORKSPACE (QMD indexa arquivos), não na message list |
| System prompt intacto | ✅ — só adiciona `systemPromptAddition` |
| Fallback gracioso | ✅ — se QMD falhar, retorna base |
| Pular "ok/yes" | ✅ — `isTrivialPrompt` |
| Score por saliência | ✅ — recency × type × collection |
| Configurável | ✅ — topK, threshold, timeout |
| Persistir índice | ❌ — depende 100% do QMD externo |
| Rodar offline | ❌ — QMD precisa estar no path |
| Embedding cache | ❌ — recalcula toda rodada |
| Cross-session memory | ❌ — `ingest()` é no-op |
| Empacotado no plugin | ❌ — exige setup separado |

---

## 📌 Decisão pro agente-universo

### O que adotar do LUCID

1. **Conceito do slot `contextEngine`** (4 hooks) — sim, é o padrão OpenClaw
2. **Detecção de prompt trivial** (regex âncora) — copiar a lógica
3. **Scoring multiplicativo** (`salience = qmd × recency × type × collection`) — copiar a fórmula
4. **Fallback gracioso** (`try/catch` + `console.warn` + retorna base) — copiar padrão
5. **Parse tolerante** (achar primeiro `[` e último `]`) — copiar truque
6. **Disclaimer "May or may not be directly relevant"** — copiar texto

### O que NÃO adotar

1. **Dependência do QMD** — não vamos usar QMD externo, vamos implementar busca direta no histórico (SQLite + embedding)
2. **`ingest()` no-op** — vamos persistir pra cross-session
3. **Sem cache** — vamos cachear embeddings
4. **Sem testes** — vamos escrever testes

### O que ADICIONAR (que LUCID não tem)

1. **Persistência** das mensagens em SQLite (não QMD)
2. **Embedding cache** por mensagem (hash do conteúdo)
3. **Cross-session recall** (busca inclui sessões anteriores)
4. **Telemetry** (qual trecho foi injetado, score, utilidade)
5. **Audit log** (pra debugar por que LLM viu X)

---

## 🔗 Fontes consultadas

| # | Fonte | URL |
|---|---|---|
| 1 | Repo clonado localmente | `/tmp/lucid-deep/` |
| 2 | package.json | lido |
| 3 | openclaw.plugin.json | lido |
| 4 | index.ts (entry point) | lido |
| 5 | src/engine.ts (4 hooks) | lido |
| 6 | src/search.ts (QMD + scoring) | lido |
| 7 | src/config.ts (pesos + trivial) | lido |
| 8 | SKILL.md | lido |
| 9 | README.md | lido |
| 10 | GitHub Issues API | 0 issues abertas/fechadas |
| 11 | Git log | 5 commits, autor único |
| 12 | OpenClaw ContextEngine docs | https://docs.openclaw.ai/concepts/context-engine |
| 13 | OpenClaw Plugin Architecture | https://docs.openclaw.ai/plugins/architecture-internals |
| 14 | openclawdir.com rating | 3.5/5 com 1 voto |

---

## 📋 Próximos passos

1. [ ] Substituir LUCID por implementação PRÓPRIA (reusar só os conceitos)
2. [ ] Definir storage (SQLite vs LevelDB)
3. [ ] Definir embedding (sentence-transformers local vs API)
4. [ ] Escrever testes antes do código (TDD)
5. [ ] Audit log + telemetry desde o início
6. [ ] Patchear plano v2 com essa arquitetura

---

# DOCUMENTO 11: PLANO-FINAL-CONSOLIDADO.md (v1 — HISTÓRICO)

> **⚠️ DOCUMENTO HISTÓRICO — SUBSTITUÍDO por `PLANO-FINAL-v2-CORRIGIDO.md`** (Documento 3 deste consolidado).

# PLANO FINAL CONSOLIDADO (v1 — HISTÓRICO)

> **⚠️ DOCUMENTO HISTÓRICO — SUBSTITUÍDO por [`PLANO-FINAL-v2-CORRIGIDO.md`](PLANO-FINAL-v2-CORRIGIDO.md)**
>
> Esta é a **v1** do plano (commit `c2e3a13` initial). Teve **14 problemas** detectados em auditoria.
> Foi substituído pela **v2** (`PLANO-FINAL-v2-CORRIGIDO.md`, commit `dcd30d7` + `9f5d98c`).
>
> Decisões já superadas pela v2 (mantidas aqui só pra histórico):
> - Memory: `runfali/dsh-mem0-plugins` ⭐1 → **OpenViking** `volcengine/OpenViking` ⭐34.690 + plugin DSH oficial
>
> **Use `PLANO-FINAL-v2-CORRIGIDO.md` como referência atual.**
>
> ---

> **Data:** 2026-08-31
> **Inspiração central:** Hermes Agent (Nous Research, ⭐239k) — nossa referência arquitetural
> **Base técnica:** DeepSeek Harness (DSH) — runtime "everything is a plugin" sobre Cordis
> **Status:** PLANEJAMENTO (zero instalação executada — todas as fases requerem aprovação)
> **Fontes:** 4 documentos prévios + pesquisa em 30+ repos via `gh api` (sem clones)

---

## 📏 TL;DR

Construir um **agente universal self-hosted** inspirado no Hermes, usando DeepSeek Harness (DSH) como runtime. **Diferenciais** vs Hermes:

1. **Catálogo agregado de 5.700+ skills** (Hermes42 + OpenClaw5.300+ + DSH~300) — nenhum agregador existe
2. **12 MCPs BR** suas já funcionam via bridge nativo DSH
3. **Multi-LLM plugável** (DeepSeek, Claude, GPT, Ollama local)
4. **VPS ou PC, qualquer SO, MIT, grátis, zero lock-in**

**Trade-offs honestos:**
- DSH é **alpha 0.1.2** — bugs garantidos (mitigação: pinar versão)
- **Não substitui agentes especializados** (Manus, Devin, Claude Code) em tarefas específicas
- **Depende da qualidade do LLM** escolhido
- **Precisa de aprovação humana** em ações críticas (delete, post, send)

---

## 🧠 INSPIRAÇÃO — O que copiamos do Hermes (e o que melhoramos)

### O que o Hermes FAZ (fonte: `~/.hermes/hermes-agent/AGENTS.md` + `SOUL.md` + `~/.hermes/config.yaml` + awesome-hermes-agent)

| Capacidade Hermes | Como implementa | Vamos copiar? |
|---|---|---|
| **30+ toolsets** (terminal, browser, web, delegation, cron, kanban, memory, etc) | Python core + plugin registry | ✅ DSH já tem 80% (bash, fs, web, git, lsp, etc) |
| **8 surfaces** (CLI, TUI, Web, Desktop, 20+ channels, ACP, API server, Cron) | Multi-process JSON-RPC | ✅ DSH tem CLI + Web; canais = plugins |
| **8 memory providers** (honcho, mem0, supermemory, byterover, etc) | Plugin ABC + orchestrator | ✅ via `runfali/dsh-mem0-plugins` |
| **42 skills locais** (formato SKILL.md padrão) | YAML frontmatter + body | ✅ importar todas |
| **Self-evolution** (DSPy + GEPA + curator + skill-factory) | Background LLM review | ✅ DSPy+GEPA first-class |
| **Multi-agent** (delegate_task single+batch, kanban SQLite-backed) | Subagent isolated context | ✅ DSH Agent Teams |
| **Cron scheduler** (5 formatos, 3min hard interrupt) | jobs.py + scheduler.py | ✅ DSH workflow+webhook |
| **Kanban multi-agent board** | SQLite dispatcher + workers | ✅ DSH subagent capability |
| **Profile isolation** (multi-instance) | `get_hermes_home()` + env var | ✅ DSH profiles |
| **Self-improvement closed loop** | Curator (auto-archive stale skills) | ✅ implementar in-house |

### O que PODEMOS SER MELHORES que o Hermes

| Melhoria | Por que seria melhor | Esforço |
|---|---|---|
| 🟢 **Agregador universal de skills** (5.700+ de 4 fontes) | Hermes tem só 42 | Médio |
| 🟡 **Multi-LLM plugável com failover automático** | Hermes user escolhe | Médio |
| 🟠 **Self-evolution coletiva** (skills melhoram com uso compartilhado) | Hermes evolui por instância | Alto |
| 🔴 **Sandbox ON por padrão** | Hermes é opt-in | Baixo |
| 🔴 **3-tier approval** (read/write/delete granular por skill) | Hermes é binário | Médio |
| 🔴 **Secret scanner built-in** | Não tem | Médio |
| 🔵 **Sessão unificada entre UIs** (Telegram → terminal continua) | Cada canal = sessão separada | Alto |
| 🔵 **Observability built-in** (Langfuse + OTel) | Plugin em Hermes | Baixo |
| 🔵 **Mobile PWA first-class** | Web não é mobile-friendly | Médio |
| 🔵 **Voice I/O nativo** (STT + TTS local) | TTS só | Médio |

---

## 🎯 ARQUITETURA FINAL (camadas validadas empiricamente)

```
┌───────────────────────────────────────────────────────────────┐
│ 🎨 UX LAYER (TUI + Web + Mobile PWA + Voice)                 │
│    DSH web nativo (port 3080) + ink-ui + custom PWA           │
├───────────────────────────────────────────────────────────────┤
│ 🛡️ SECURITY LAYER (3-tier approval + prompt defense + sandbox)│
│    Scanner in-house (~200 linhas Python) + Dormice self-hosted │
├───────────────────────────────────────────────────────────────┤
│ 📊 OBSERVABILITY LAYER (tracing + evals + dashboards)        │
│    Yuntwo/dsh-langfuse-plugin (⭐24) + OpenLLMetry (OTel)     │
├───────────────────────────────────────────────────────────────┤
│ 🧬 SELF-EVOLUTION LAYER (DSPy + GEPA + skill-factory)        │
│    DSPy 37.7k⭐ + GEPA 6.3k⭐ first-class (não plugin)        │
├───────────────────────────────────────────────────────────────┤
│ 🧠 ORQUESTRAÇÃO (core runtime)                                │
│    DeepSeek Harness (DSH) + Cordis framework                  │
├───────────────────────────────────────────────────────────────┤
│ 🔌 MULTI-LLM GATEWAY (providers + failover)                   │
│    DSH providers nativos (DeepSeek, Claude, GPT, Ollama)      │
│    LiteLLM standalone opcional (100+ providers)               │
├───────────────────────────────────────────────────────────────┤
│ 💾 MEMORY LAYER (8+ providers + roteamento)                   │
│    runfali/dsh-mem0-plugins + honcho + openviking             │
├───────────────────────────────────────────────────────────────┤
│ 🔌 MCP BRIDGE (suas12 MCPs BR já funcionam)                   │
│    DSH tem packages/mcp/ nativo                               │
├───────────────────────────────────────────────────────────────┤
│ 📚 SKILLS LAYER (5.700+ agregadas — DIFERENCIAL ÚNICO)       │
│    Hermes (42) + OpenClaw (5.300+) + DSH (~300)               │
│    Adaptador 100 linhas Python (SKILL.md normalizer)          │
└───────────────────────────────────────────────────────────────┘
```

---

## 📦 STACK FINAL (todos validados empiricamente)

| Camada | Repo | Stars | License | Por que |
|---|---|---|---|---|
| **🎯 Core** | `deepseek-ai/deepseek-harness` | 206k | MIT | "Everything is a plugin", Cordis, alpha 0.1.2 |
| **🧠 Multi-LLM** | DSH providers nativos | (built-in) | MIT | Já tem deepseek/anthropic/openai; LiteLLM se precisar100+ |
| **💾 Memory** | `runfali/dsh-mem0-plugins` | 1 | MIT | Mem0 integration; auditar antes |
| **🛡️ Sandbox** | `BitMiracle-AI/Dormice` | 944 | MIT | E2B-compatible, self-hosted, idle=$0 |
| **🛡️ Prompt defense** | Construir in-house (~200 linhas Python) | — | MIT | Ruflo aidefence como referência de design |
| **📊 Observability** | `Yuntwo/dsh-langfuse-plugin` | 24 | MIT | Langfuse telemetry sidecar bundle |
| **📊 OTel** | `traceloop/openllmetry` | 7.4k | Apache-2.0 | Vendor-neutral, OpenTelemetry nativo |
| **🧬 Self-evolution** | `stanfordnlp/dspy` + `gepa-ai/gepa` | 38k+6.3k | Apache+MIT | DSPy framework + GEPA Pareto SOTA |
| **🎨 TUI** | `vadimdemedes/ink-ui` | 2.1k | MIT | Stale mas funciona como dep (alternativa: shadcn-labs/termcn) |
| **🔌 MCP BR** | Suas12 MCPs locais | — | — | Funcionam via DSH MCP bridge nativo |
| **📚 Skills** | 3 fontes agregadas | 5.700+ | MIT | Hermes42 + OpenClaw5.300+ + DSH~300 |

**Não usar (descartados na auditoria):**
- ❌ `bridge-mind/BridgeWard` — é artigo, não código
- ❌ `e2b-dev/fragments` — é template demo Next.js, E2B é cloud pago
- ❌ `vadimdemedes/ink-ui` como projeto principal — stale, manter só como dep

---

## 📐 FASES DE EXECUÇÃO (7 fases, cada uma requer aprovação)

### 🟢 FASE 0 — Pré-requisitos (verificar, 30min)

**Objetivo:** Garantir ambiente pronto antes de instalar qualquer coisa.

```bash
# Checar versões instaladas
node --version        # precisa ^22.19 ou >=24
pnpm --version        # precisa 11.7+
python3 --version     # qualquer 3.10+
gh --version          # GitHub CLI logado (gh auth status)

# Espaço em disco
df -h /home           # precisa ~5GB livre (DSH 136MB + deps)

# Verificar se tem Ollama (opcional, pra LLMs locais)
which ollama || echo "Ollama não instalado (opcional)"

# Verificar GPU (opcional, pra rodar modelos locais)
nvidia-smi 2>/dev/null || echo "Sem GPU NVIDIA (rodar via API)"
```

**Saída esperada:** confirmação que node/pnpm/gh estão prontos.

---

### 🟢 FASE 1 — Core DSH (1-2h)

**Objetivo:** Subir DSH vazio, validar UI funciona.

```bash
# Instalar DSH via git (NÃO npm — versões divergem)
cd $HOME
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
git checkout v0.1.2-alpha.3    # PINAR VERSÃO

# Instalar dependências
pnpm install
pnpm run build

# Iniciar web UI
pnpm dsh web
# Abre em http://127.0.0.1:3080
```

**Teste:** No browser, abrir `http://127.0.0.1:3080`. Digitar: *"olá, liste os arquivos em /tmp"*. Deve responder.

**Entregável:** Screenshot da UI funcionando + log de teste.

---

### 🟢 FASE 2 — Skills Bridge + Import Hermes (meio dia)

**Objetivo:** Adaptador SKILL.md + importar suas 42 skills locais.

```bash
# 2.1 — Criar diretório de skills customizadas
mkdir -p $HOME/deepseek-harness/skills/custom

# 2.2 — Copiar suas 42 skills Hermes locais
cp -r ~/.hermes/skills/* $HOME/deepseek-harness/skills/custom/

# 2.3 — Criar adaptador SKILL.md normalizer (~100 linhas Python)
cat > $HOME/deepseek-harness/skills/adapter.py << 'EOF'
"""SKILL.md normalizer: Hermes → DSH"""
import re, yaml
from pathlib import Path

def normalize_skill(path: Path) -> dict:
    """Lê SKILL.md, descarta metadata proprietária Hermes, retorna frontmatter limpo."""
    text = path.read_text()
    if not text.startswith("---"):
        return None
    fm = text.split("---")[1]
    meta = yaml.safe_load(fm)
    # Descartar campos Hermes-específicos
    meta.pop("environments", None)
    if "metadata" in meta and "hermes" in meta["metadata"]:
        hermes_meta = meta["metadata"].pop("hermes")
        meta["tags"] = hermes_meta.get("tags", [])
        # requires_toolsets não tem equivalente DSH — descartar
    # Manter: name, description, version, author, license, platforms
    return meta

def main():
    skills_dir = Path("skills/custom")
    normalized = 0
    for skill_md in skills_dir.rglob("SKILL.md"):
        meta = normalize_skill(skill_md)
        if meta and "name" in meta and "description" in meta:
            normalized += 1
    print(f"✅ {normalized} skills normalizadas")

if __name__ == "__main__":
    main()
EOF

# 2.4 — Rodar adaptador
python3 skills/adapter.py
```

**Teste:** Reiniciar DSH, verificar que as skills aparecem no menu.

**Entregável:** Log mostrando "42 skills normalizadas" + print da UI com skills listadas.

---

### 🟢 FASE 3 — OpenClaw skills (1-2 dias)

**Objetivo:** Importar top 50 OpenClaw skills (curadas pelo VoltAgent — NUNCA raw registry).

```bash
# 3.1 — Pegar awesome-list curada
curl -sL https://raw.githubusercontent.com/VoltAgent/awesome-openclaw-skills/main/README.md > /tmp/openclaw-curated.md

# 3.2 — Extrair top 50 skills com mais stars
grep -oE 'https://github\.com/[a-zA-Z0-9_-]+/[a-zA-Z0-9_.-]+' /tmp/openclaw-curated.md | sort -u | head -50 > /tmp/openclaw-top50.txt

# 3.3 — Para cada skill, clonar e copiar
while read repo; do
  name=$(basename "$repo")
  git clone --depth 1 "$repo" "/tmp/openclaw-skills/$name" 2>/dev/null
  if [ -d "/tmp/openclaw-skills/$name/skills" ]; then
    cp -r "/tmp/openclaw-skills/$name/skills/"* $HOME/deepseek-harness/skills/custom/ 2>/dev/null
  fi
done < /tmp/openclaw-top50.txt
```

**Teste:** Reiniciar DSH, validar que as OpenClaw skills aparecem no menu.

**Entregável:** Print da UI mostrando skills de 3 fontes (Hermes + OpenClaw + DSH).

---

### 🟢 FASE 4 — MCP BR + Observability (meio dia)

**Objetivo:** Integrar suas 12 MCPs BR + instalar Langfuse para tracing.

```bash
# 4.1 — Configurar MCP bridge no DSH
cat >> $HOME/deepseek-harness/config/mcp.yaml << 'EOF'
mcp_servers:
  - name: mercado_livre
    command: hermes mcp run mercado_livre
  - name: fiscal
    command: hermes mcp run fiscal
  - name: comexstat
    command: hermes mcp run comexstat
  # ... outras 9 MCPs
EOF

# 4.2 — Instalar Langfuse plugin DSH
cd $HOME/deepseek-harness
git clone https://github.com/Yuntwo/dsh-langfuse-plugin.git plugins/dsh-langfuse-plugin
pnpm install
pnpm run build

# 4.3 — Subir Langfuse self-hosted (Docker)
docker run -d -p 3000:3000 \
  -e DATABASE_URL=postgresql://... \
  langfuse/langfuse:latest

# 4.4 — Configurar DSH pra exportar traces
cat >> config/observability.yaml << 'EOF'
langfuse:
  enabled: true
  public_key: pk-lf-...
  secret_key: sk-lf-...
  host: http://localhost:3000
EOF
```

**Teste:** Rodar 1 task no DSH, verificar que trace aparece em `http://localhost:3000`.

**Entregável:** Print do dashboard Langfuse com trace do DSH.

---

### 🟢 FASE 5 — Sandbox + Memory (1 dia)

**Objetivo:** Sandbox self-hosted (Dormice) + Memory via mem0 plugin.

```bash
# 5.1 — Sandbox: Dormice self-hosted
docker run -d -p 8080:8080 bitmiracle/dormice:latest

# 5.2 — Mem0: instalar plugin DSH (auditar código antes!)
cd $HOME/deepseek-harness
git clone https://github.com/runfali/dsh-mem0-plugins.git plugins/dsh-mem0
# IMPORTANTE: ler TODO o código antes de ativar
cat plugins/dsh-mem0/README.md
ls plugins/dsh-mem0/src/

# 5.3 — Configurar memória persistente
cat >> config/memory.yaml << 'EOF'
memory:
  provider: mem0
  persist: true
  dsh_mem0:
    server_url: http://localhost:8080
EOF
```

**Teste:** Iniciar conversa, fechar DSH, reabrir. Memória deve persistir.

**Entregável:** Demo de memória funcionando entre sessões.

---

### 🟢 FASE 6 — Self-evolution (DSPy + GEPA) (2-3 dias)

**Objetivo:** Habilitar otimização automática de prompts/skills.

```bash
# 6.1 — Instalar DSPy (Python)
pip install dspy-ai

# 6.2 — Instalar GEPA
pip install gepa-ai

# 6.3 — Criar script de otimização
cat > $HOME/dsh-optimizer.py << 'EOF'
"""DSH skill optimizer usando DSPy + GEPA."""
import dspy
from dspy import GEPA

# Conectar DSPy ao DSH via OpenAI-compat
lm = dspy.LM(model="openai/deepseek-chat", api_base="http://localhost:3080/v1", api_key="not-needed")
dspy.configure(lm=lm)

# Carregar skill atual
skill_content = open("skills/custom/pesquisa-internet/SKILL.md").read()

# GEPA evolui a skill baseado em exemplos de uso bem-sucedido
optimizer = GEPA(metric=lambda x, y: 1.0 if "resultado correto" in str(y) else 0.0)
optimized = optimizer.compile(skill_content, trainset=[])

print(f"Skill otimizada: {optimized}")
EOF

# 6.4 — Rodar (background, sem agente LLM)
python3 $HOME/dsh-optimizer.py
```

**Teste:** Comparar resposta de uma skill antes vs depois da otimização.

**Entregável:** Diff mostrando skill melhorada automaticamente.

---

### 🟢 FASE 7 — Validação Final + Publicação (1 dia)

**Objetivo:** Benchmark empírico vs Hermes + publicar no GitHub.

```bash
# 7.1 — 10 tarefas idênticas em Hermes e no nosso agente
TASKS=(
  "Liste os arquivos .py em $HOME"
  "Pesquise o preço do dólar hoje"
  "Crie uma planilha CSV com 5 linhas"
  "Mande um email de teste"
  "Faça commit de teste num repo git"
  # ... 5 mais
)

for task in "${TASKS[@]}"; do
  echo "TASK: $task"
  echo "  Hermes: $(time hermes chat "$task" 2>&1 | tail -1)"
  echo "  Nosso: $(time dsh chat "$task" 2>&1 | tail -1)"
done

# 7.2 — Medir custo
echo "Custo API último mês: $(cat ~/.hermes/.costs)"

# 7.3 — Publicar no GitHub
cd $HOME
gh repo create universal-agent-dsh --public --source=. --description="Agente universal self-hosted inspirado no Hermes. 5.700+ skills agregadas. MIT."
git push -u origin main
```

**Entregável:** Tabela comparativa Hermes vs nosso em 10 tarefas + repo público no GitHub.

---

## ⚠️ RISCOS & LIMITAÇÕES (honestos)

| Risco | Probabilidade | Mitigação |
|---|---|---|
| **DSH alpha quebra** | Alta | Pinar v0.1.2-alpha.3 + changelog review antes de update |
| **Plugin 3rd-party roda código sem sandbox** | Média | Auditar package.json + testar em ambiente isolado |
| **5.700 skills agregadas = spam** | Média | SÓ usar curadas (awesome-list filtrou 7.215 itens) |
| **OpenClaw NOASSERTION license** | Alta | Verificar LICENSE file antes de fork/redistribuir |
| **Claude Code sem LICENSE explícita** | Alta | Verificar antes de extrair plugins |
| **Mem0 DSH plugin ⭐1 (imaduro)** | Alta | Auditar código linha-a-linha antes de prod |
| **Langfuse DSH plugin ⭐24 (pouco testado)** | Média | Sandbox primeiro, prod depois |
| **DSPy é acadêmico** | Média | Usar GEPA por cima pra prod |
| **BridgeWard não tem código** (resolvido) | ✅ | Construir scanner in-house 200 linhas |
| **DSH plugins sem LICENSE clara** | Média | Sempre verificar LICENSE file |
| **Self-evolution coletiva** (ideia ambiciosa) | Alta | Sem prova de conceito — MVP depois, não antes |

---

## 📊 CRONOGRAMA REALISTA

| Fase | Duração | Bloqueia próxima? |
|---|---|---|
| Fase 0 — Pré-requisitos | 30min | Sim |
| Fase 1 — Core DSH | 1-2h | Sim |
| Fase 2 — Skills Bridge + Hermes | 3-4h | Sim |
| Fase 3 — OpenClaw skills | 1-2 dias | Não |
| Fase 4 — MCP + Observability | 3-4h | Não |
| Fase 5 — Sandbox + Memory | 4-6h | Não |
| Fase 6 — Self-evolution DSPy+GEPA | 2-3 dias | Não |
| Fase 7 — Validação + Publicação | 1 dia | Não |
| **TOTAL MVP** | **~7-10 dias úteis** | — |

---

## 📂 DOCUMENTAÇÃO GERADA (6 arquivos salvos)

| # | Arquivo | Conteúdo | Status |
|---|---|---|---|
| 1 | `PLANEJAMENTO-AGENTE-DSH.md` | Visão geral + 4 camadas + 7 fases | ✅ Salvo |
| 2 | `PESQUISA-HERMES-CAPACIDADES.md` | Capacidades Hermes + 20 melhorias | ✅ Salvo |
| 3 | `PESQUISA-REPOSITORIOS-AGENTE-NOVO.md` | 30+ repos (validação superficial) | ✅ Salvo |
| 4 | `AUDITORIA-EMPIRICA-REPOS.md` | 18 repos auditados a fundo (3 farsas detectadas) | ✅ Salvo |
| 5 | `LACUNAS-FECHADAS.md` | 3 lacunas críticas (Ruflo + SKILL.md + integrações) | ✅ Salvo |
| 6 | `PLANO-FINAL-CONSOLIDADO.md` | **Esse arquivo** — plano integrado final | ✅ Salvo |

**Localização:** `$HOME/importaai/`

---

## 🎯 COMPARAÇÃO FINAL — Hermes vs Nosso Agente (target)

| Capacidade | Hermes | OpenClaw | DSH | **Nosso (target)** |
|---|---|---|---|---|
| Skills próprias | 42 | 5.300+ | ~300 | **5.700+ (agregadas)** |
| Plugins | ~30 in-tree | ClawHub registry | ~300 | **100+ curados** |
| Multi-LLM | ✅ 8+ providers | ✅ 25+ providers | ✅ multi | ✅ **+ failover automático** |
| Multi-UI | ✅ 8 surfaces | ✅ 7 channels | ✅ web+CLI+SDK | ✅ **+ sessão unificada** |
| Memory | ✅ 8 providers | ⚠️ básico | ✅ session log | ✅ **+ roteamento por contexto** |
| Self-evolution | ✅ DSPy/GEPA | ❌ | ❌ | ✅ **+ coletiva (futuro)** |
| Sandbox | ⚠️ opt-in | ⚠️ opt-in | ⚠️ opt-in | ✅ **ON por padrão (futuro)** |
| Approval gates | ⚠️ binário | ⚠️ básico | ⚠️ básico | ✅ **3-tier granular (futuro)** |
| Observability | ❌ plugin | ❌ plugin | ❌ | ✅ **built-in via Langfuse** |
| Mobile | ❌ | ❌ | ⚠️ | 🟡 PWA (futuro) |
| Voice I/O | ⚠️ parcial | ⚠️ parcial | ❌ | 🟡 built-in (futuro) |
| Open source | ✅ MIT | ✅ MIT | ✅ MIT | ✅ **MIT** |
| MCP servers | ✅ | ✅ | ✅ nativo | ✅ **12 MCPs BR incluídas** |
| Idade | 2+ anos | ~1 ano | meses | 🆕 **novo, agregando melhor dos 4** |

---

## 🚦 PRÓXIMO PASSO

**Plano finalizado.** Para começar:

1. **Você aprova o plano como está?**
   - ✅ Sim → começo pela **Fase 0** (pré-requisitos) agora
   - 🔄 Quero ajustar → me diz o quê
2. **Quem vai manter?**
   - (a) Seu GitHub pessoal (`renan-giudice/...`)
   - (b) Org compartilhado (`hermes-agent-br/...`)
   - (c) Outro
3. **Quer revisar algum comando antes de eu rodar?**
   - Posso listar cada comando isoladamente se preferir

Responde em texto normal. **Não instalo nada** até você dar OK explícito na Fase 0.
---

**FIM DO CONSOLIDADO.** Total: 11 documentos originais concatenados em ordem lógica de leitura.

---

# 📌 DOCUMENTO 12 (anexo, 2026-08-31): Validação Empírica DSH vs Hermes — 19 Camadas

> **Origem:** checagem empírica feita via `gh api repos/deepseek-ai/deepseek-harness/*` (metadata + contents + releases).
> **Repo validado:** `deepseek-ai/deepseek-harness` — 206.423⭐, MIT, TypeScript, tag `dsh-v0.1.2-alpha.3` (2026-08-31), 44 packages em `packages/`, 2 apps (`cli`+`web`), 1 native (`landlock-run`), Python SDK.
> **Arquitetura DSH:** "Everything is a Plugin" via [Cordis](https://github.com/cordiverse/cordis).
> **Método:** listagem de `packages/` (44 dirs) + apps/ + native/ + python/ + README, depois mapeamento 1-para-1 com as 19 camadas do Hermes do Documento 4.

## Tabela de cobertura (19 camadas)

| # | Camada Hermes | DSH cobre? | Pacotes DSH |
|---|---|---|---|
| 1 | Entry Points (CLI/TUI/Desktop/ACP/Web/Gateway) | ✅ **100%** | `apps/cli` + `apps/web` (porta 3080) + `packages/acp` (IDE adapter) |
| 2 | Distribution (slash commands + message guards) | 🟡 parcial | CLI subcommands + `packages/webhook`; sem slash registry central cross-surface |
| 3 | Agent Loop (AIAgent) | ✅ **100%** | `packages/core` + `packages/llm` + `packages/goal` + `packages/plan` + `packages/runtime-diagnostics` |
| 4 | Capabilities (tools, toolsets, env, skills, subagentes) | ✅ **100%** | `packages/skill` + `packages/subagent` + `packages/extensions` |
| 5 | Context Lifecycle (system prompt, compression) | ✅ **100%** | `packages/context` + `packages/compaction` + `packages/code-runtime` |
| 6 | Context & Memory (providers + engines + SessionDB) | ✅ **100%** | `packages/session` + `packages/session-query` + `packages/storage` (SQLite FTS) |
| 7 | Inference (transports + providers + credential pool) | ✅ **100%** | `packages/llm` (multi-provider) + `packages/credentials` |
| 8 | Voice & Desktop (TTS, wake, computer use) | ❌ zero | — (não tem TTS streaming, wake words, computer use) |
| 9 | Code & IDE (LSP, Codex, Copilot ACP) | 🟡 parcial | `packages/lsp` + `packages/acp` + `packages/code-runtime` (sem Codex runtime dedicado) |
| 10 | Extensibility (plugins + MCP + optional MCPs) | ✅ **100%** | Cordis (everything-is-a-plugin) + `packages/mcp` nativo |
| 11 | Secrets & Security (Bitwarden/Approvals/Sandbox) | ✅ **100%** | `packages/credentials` + `packages/sandbox` + `native/landlock-run` (Linux sandbox) |
| 12 | Observability (OTLP, health, redaction) | ✅ **100%** | `packages/runtime-diagnostics` + `packages/feedback` |
| 13 | Background Work (Cron, Kanban, Delegation) | 🟡 parcial | `packages/schedule` (cron) + `packages/jobs` + `packages/subagent` (sem Kanban dispatcher) |
| 14 | Gateway Infra multi-plataforma | ❌ zero | CLI + Web só; sem adapter Telegram/Discord/Signal/WhatsApp built-in |
| 15 | Relay & Connector (WebSocket relay) | ❌ zero | — (não tem) |
| 16 | Persistence & Isolation (HERMES_HOME, profiles) | ✅ **100%** | `packages/storage` + `packages/workspace` + `packages/host` |
| 17 | Distribution & Packaging (Docker/Nix/i18n) | � parcial | `pnpm install` + `apps/web` build + `python/sdk`; sem Docker oficial, sem Nix, só `README.zh.md` (sem locales i18n) |
| 18 | Meta (Curator, Skin engine, Pet) | ❌ zero | — (sem auto-manutenção de skills) |
| 19 | Tests/Docs (vitest, Docusaurus, AGENTS.md) | 🟡 parcial | `vitest` (test + e2e + snapshot) + `website/` Docusaurus + `packages/AGENTS.md`; escala menor que 17k tests do Hermes |

## Resumo

| Faixa | Camadas | Lista |
|---|---|---|
| ✅ 100% | **10** (1, 3, 4, 5, 6, 7, 10, 11, 12, 16) | motor completo + sandbox + observability + CLI/Web/ACP |
| 🟡 parcial | **5** (2, 9, 13, 17, 19) | slash registry, Codex runtime, Kanban, Docker/Nix, tests/docs |
| ❌ zero | **4** (8, 14, 15, 18) | Voice/Desktop, Gateway multi-plataforma, Relay, Meta (Curator/Skin/Pet) |

**DSH cobre ~74% da arquitetura do Hermes** ((10+5)/19). As 4 camadas zero são as que exigem **plugins 3rd-party ou desenvolvimento próprio** no plano v2:

- **Camada 8 (Voice/Desktop)** → não está no plano v2 (escopo cortado do MVP)
- **Camada 14 (Gateway multi-plataforma)** → não está no plano v2 (DSH é single-user CLI/Web, não multi-tenant mensageria como Hermes)
- **Camada 15 (Relay)** → não está no plano v2 (experimental no Hermes também)
- **Camada 18 (Meta/Curator)** → substituído por `hermes curator` standalone que roda contra DSH (Fase 1 do plano)

## Correções do checklist anterior (v1 — feito antes da validação empírica)

| Camada | Checklist v1 (errado) | Checklist v2 (corrigido) | Motivo |
|---|---|---|---|
| 1 (Entry Points) | ❌ DSH não cobre | ✅ 100% | apps/cli + apps/web (porta 3080) + packages/acp existem |
| 11 (Secrets/Security) | 🟡 parcial | ✅ 100% | packages/credentials + packages/sandbox + native/landlock-run |
| 19 (Tests/Docs) | ❌ zero | 🟡 parcial | vitest suite + website/ Docusaurus + packages/AGENTS.md existem |

## Implicações pro plano v2 (Fase 1)

1. **Fase 1 não precisa de plugin de Sandbox** — DSH já tem `native/landlock-run` built-in (Linux nativo).
2. **Fase 1 não precisa de plugin de Credentials** — DSH já gerencia via `packages/credentials`.
3. **Fase 1 precisa de plugin de Memory** — DSH tem SessionDB SQLite mas **não tem memory providers semânticos** (sem Mem0/OpenViking built-in). É onde entra OpenViking.
4. **Fase 4 (MCP)** já vem facilitada — DSH tem `packages/mcp` nativo, é só declarar servers.
5. **Fase 5 (Dormice sandbox)** do plano v2 pode ser **opcional** — `landlock-run` já dá sandbox Linux. Dormice só vale pra cross-OS (macOS/Windows) ou pra requisitos avançados.

**Recomendação:** revisar Fase 5 do plano v2 pra considerar `landlock-run` como padrão Linux + Dormice como opcional cross-OS.
