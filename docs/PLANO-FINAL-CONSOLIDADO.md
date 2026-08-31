# PLANO FINAL CONSOLIDADO — Agente Universal inspirado no Hermes

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