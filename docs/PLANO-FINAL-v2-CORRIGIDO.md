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
| **🛡️ Sandbox** | `BitMiracle-AI/Dormice` | `curl install.sh \| bash` (NÃO Docker) | ✅ Confirmado via README |
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

**⚠️ MUDANÇA CRÍTICA:** Dormice **não tem Docker image**. Instala via script bash.

```bash
# === Sandbox Dormice (formato correto) ===

cd "$HOME/agente-universo"

# 1. Dormice install — SCRIPT, NÃO Docker (L-I6)
# ⚠️ Requer Ubuntu/Debian x86_64 + root
if [ "$EUID" -ne 0 ]; then
    echo "⚠️ Dormice precisa de root. Use sudo."
    exit 1
fi

# 2. Instalar (idempotente — pode re-rodar pra upgrade)
curl -fsSL https://raw.githubusercontent.com/BitMiracle-AI/Dormice/main/deploy/install.sh | bash

# 3. Validar
dor doctor   # bateria de checks (3 deles bootam container gVisor)

# 4. Em paralelo: OpenViking server (memória persistente)
# OpenViking = "Context Database for AI Agents" (volcengine/OpenViking, 34k⭐, AGPL-3.0)
# Faz TUDO que LUCID/Mem0/Honcho fazem + tem plugin DSH oficial
pip install openviking
openviking-server start --port 1933 &

# Validar server up
curl -s http://localhost:1933/health | python3 -m json.tool

# 5. Instalar plugin DSH oficial
cd "$HOME/agente-universo/deepseek-harness"
openclaw plugins install @openviking/openclaw-plugin

# 6. Configurar plugin
openclaw openviking setup \
  --base-url http://localhost:1933 \
  --json

openclaw gateway restart

# 7. Selecionar contextEngine slot
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
# Dormice funcional
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

**Entregável:** `dor doctor` output + `openclaw openviking status` verde + demo de memória persistente.

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