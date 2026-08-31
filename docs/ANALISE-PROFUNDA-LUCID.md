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
